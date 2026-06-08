# Zemax/OpticStudio 操作知识库

这是从实际 starting-point 建模中整理出来的 Zemax 操作备忘。它不是官方手册的替代品，重点记录那些会让初始结构看似能打开、但实际上不可信的细节。

## ZMX 文件格式

### 编码

- `.zmx` 通常使用 `UTF-16LE` 编码，带 BOM。
- 直接文本修改时要保持编码，否则 OpticStudio 可能无法识别文件。
- 如果用 Python 处理，流程应是：`decode('utf-16-le')`，修改文本，再 `encode('utf-16-le')`。

### GLAS 行

```text
GLAS <name> <formula> <melt> <Nd> <Vd> <dPgF> <TCE> <density> <cost> <status> <comment>
```

- `formula=0` 表示目录玻璃，Zemax 从已加载的 AGF 目录查找色散数据。
- `formula=1` 表示模型玻璃，Zemax 使用该行给出的 `Nd/Vd`。
- 只改玻璃名但不改 `formula`，会导致材料仍按模型玻璃处理。

```text
# 模型玻璃，不适合在有目录匹配时使用
GLAS ___BLANK 1 0 1.5348 55.7 0 0 0 0 0 0

# 目录玻璃
GLAS K26R 0 0 1.53504 55.7107 0 0 0 0 0 0
```

### COMM 行

```text
COMM surface note text
```

不要用引号包裹注释（`COMM "surface note"`）。本地 Zemax 样本文件都使用不带引号的注释，带引号可能被解析器误读。

### 文件结尾

ZMX 文件在最后一行数据后自然结束即可，**不要**额外添加 `END` 行。本地 Zemax 样本文件以 `MOFF`（多重组态时）或最后一个 surface 行结尾，不出现独立 `END`。

### 视场和波长一致性

```text
FTYP 0 0 3 5 0 0 0 3
XFLN 0 0 0
YFLN 0 4.363 6.170
FWGN 1 1 1
```

- `FTYP` 声明几个视场，`XFLN/YFLN/FWGN` 就要给几个值。
- 多余或缺失的 `WAVM` 行会让分析结果难以解释，应保持波长数量、主波长 `PWAV` 和权重一致。
- 如果用户没有明确指定主波长，默认选择波长列表的中间项：`PWAV = ceil(N / 2)`，其中 `N` 是 `WAVM` 行数量，且 `PWAV` 是 Zemax 的 1-based 序号。例如 3 个波长默认 `PWAV 2`，5 个波长默认 `PWAV 3`。不要默认使用第一条 `WAVM`。

## ZOS-API 建模规则

### 初始化

MATLAB 脚本应把这些路径做成可编辑参数：

```matlab
ctx.helperDll = 'C:\ProgramData\Zemax\ZOS-API\Libraries\ZOSAPI_NetHelper.dll';
ctx.zemaxRoot = 'D:\Program Files\Ansys Zemax OpticStudio 2024 R1.00';
```

常见 helper DLL 位置：

```text
C:\ProgramData\Zemax\ZOS-API\Libraries
C:\ProgramData\Zemax\ZOS-API\Extensions
C:\Program Files\Ansys Zemax OpticStudio*\ZOS-API
```

### ZOS-API LicenseStatus Unknown recovery

If `CreateNewApplication()` reports:

```text
IsValidLicenseForAPI = false
LicenseStatus = Unknown
Could not start OpticStudio or API license is not valid.
```

do not assume that restarting the OpticStudio GUI is enough, and do not keep reusing the same failed MCP or long-lived MATLAB session. This state often means the current API process has a stale or missing license session, even when the GUI can open normally.

Use this recovery order:

1. Close or discard the stale MCP / MATLAB session.
2. Start a fresh `matlab -batch` process for the ZOS-API check or build.
3. Set the license environment variables explicitly before the batch process starts.
4. For standalone validation, use `CreateNewApplication()`; use `ConnectToApplication()` only when OpticStudio was already launched in an API-compatible session.

PowerShell example:

```powershell
$env:ANSYSLMD_LICENSE_FILE = "1055@localhost"
$env:ANSYSLI_SERVERS = "2325@localhost"
matlab -batch "cd('<project-folder>'); run_zosapi_check"
```

The MATLAB initialization should still use the standard helper/root path pattern:

```matlab
NET.addAssembly(ctx.helperDll);
success = ZOSAPI_NetHelper.ZOSAPI_Initializer.Initialize();
if success ~= 1
    success = ZOSAPI_NetHelper.ZOSAPI_Initializer.Initialize(ctx.zemaxRoot);
end
zemaxDir = char(ZOSAPI_NetHelper.ZOSAPI_Initializer.GetZemaxDirectory());
NET.addAssembly(fullfile(zemaxDir, 'ZOSAPI.dll'));
NET.addAssembly(fullfile(zemaxDir, 'ZOSAPI_Interfaces.dll'));
connection = ZOSAPI.ZOSAPI_Connection();
app = connection.CreateNewApplication();
```

Record success only after the fresh process reports a valid API license, for example:

```text
IsValidLicenseForAPI = true
LicenseStatus = PremiumEdition
```

If the fresh batch process still fails, report only static ZMX generation, prescription audit, and first-order/paraxial checks. Do not claim Quick Focus, Spot, MTF, CRA, relative illumination, or layout verification.

### 目录玻璃优先

```matlab
% 目录玻璃
surface.Material = 'H-K9L';

% 模型玻璃只作为无目录匹配时的 fallback
solver = surface.MaterialCell.CreateSolveType(ZOSAPI.Editors.SolveType.MaterialModel);
solver.IndexNd = nd;
solver.AbbeVd = vd;
surface.MaterialCell.SetSolveData(solver);
```

有可用 AGF 匹配时，优先使用目录玻璃。模型玻璃会削弱后续色差、材料可得性和优化结果的可信度。

### 半口径

```matlab
solver = surface.SemiDiameterCell.CreateSolveType(ZOSAPI.Editors.SolveType.Automatic);
surface.SemiDiameterCell.SetSolveData(solver);
```

- 普通透镜面优先使用自动净口径求解。
- 光阑、像面、明确机械口径面可以固定。
- 2D layout 里镜片巨大、交叉或边缘光线异常时，先检查是否把直径当成半口径。

### 保护玻璃

保护玻璃必须用两个面建模：

```text
cover front: material = cover glass, thickness = fixed cover thickness
cover back:  material = air, thickness = focus air gap
image plane
```

不要用单面保护玻璃，否则 Quick Focus 可能改掉保护玻璃厚度。

### Quick Focus

Quick Focus 是初步对焦，不是优化。报告里必须说明它修改了哪个空气间隔，以及对焦后 BFL/TTL 是否仍满足约束。

## Optical Bench Hub 转换

### 口径列

Optical Bench 的口径列经常是直径，不是 Zemax semi-diameter。写入 Zemax 前要除以 2，并在审查报告里记录判断依据。

### AS/FS 行

AS/FS 行通常表示在前一个空气间隔内插入特殊面，而不是额外增加厚度。

```text
# 原始
6     13.258   6.561        19.48
6AS   AS       2.779        15.974
7    -25.121   1.00  1.67648 17.86

# 正确处理
S6 -> AS = 2.779
AS -> S7 = 6.561 - 2.779
```

## 非球面缩放

均匀缩放倍率为 `s` 时：

```text
radius'    = radius * s
thickness' = thickness * s
conic'     = conic
A4'        = A4  / s^3
A6'        = A6  / s^5
A8'        = A8  / s^7
A10'       = A10 / s^9
```

Zemax Even Asphere 常用列：

- `GetCellAt(9)`: Conic
- `GetCellAt(13)`: 4 阶系数
- `GetCellAt(14)`: 6 阶系数
- `GetCellAt(15)`: 8 阶系数
- `GetCellAt(16)`: 10 阶系数

## AGF 玻璃库

- CDGM/PLASTIC 常见编码：`UTF-16LE with BOM`。
- HOYA 常见编码：ASCII。
- 关键记录包括 `NM`、`CD`、`MD`、`IT`、`LD`。
- 玻璃替换时同时看 `Nd` 和 `Vd`，不要只按折射率匹配。
- 如果状态位表示停产、受限或不再接新单，应在 `glass_substitution_report.md` 中标注。

## 专利处理规则

- 专利表面顺序可能和使用方向相反，要检查 `R/f`、BFL、WD 和光线传播方向。
- 专利表格可能缺行、OCR 错误、单位混淆或混用多个实施例。
- 折叠系统展开后不等于真实机械结构，必须单独标注。
- 原始复现、反向使用、缩放版本、优化版本要分开保存和命名。
- **专利"视场角"通常是全视场角**，与 Zemax 半视场 `YFLN` 不同。交叉验证：`tan(half_FOV) ≈ image_semi_height / EFL`。如专利报 190° 全场，Zemax 最大半场为 95°。

## 排查清单

| 症状 | 常见原因 |
|---|---|
| ZMX 打不开 | 编码不是 UTF-16LE，或关键头部/行格式损坏 |
| 材料显示为模型玻璃 | `GLAS` 行 `formula=1`，或目录未加载 |
| 只有轴上视场 | `FTYP` 声明数与 `YFLN/FWGN` 数量不一致 |
| 2D layout 镜片巨大或交叉 | 口径用了直径而非半口径，或厚度符号错误 |
| 保护玻璃厚度被改 | 单面建模，Quick Focus 改了玻璃厚度 |
| 总长多出几毫米 | AS/FS 行被当成额外距离插入 |
| EFL 与来源差异大 | 缩放系数、材料匹配、传播方向或缺面错误 |
| Spot 出不来 | 视场、光阑、净口径或边缘光线追迹失败 |
| AGF 解析乱码 | 用错编码读取玻璃库 |
| `IsValidLicenseForAPI = false` | 未设 `ANSYSLMD_LICENSE_FILE` / `ANSYSLI_SERVERS` 环境变量 |
| 设了环境变量仍 license 失败 | MATLAB session 是旧的（之前失败过），需全新进程 |
| GUI 能用但 API 不行 | GUI 有自己的 license 查找路径；API 进程需要单独配置 |
| ZMX 加载后 surface count 不对 | 文件中有 quoted COMM 或多余 END 导致解析异常 |
| 全场角写成半场角 | 专利"视场角"可能是全视场角，Zemax YFLN 是半视场角 |
| license 环境正确但仍 Unknown | 检查端口号：`lmutil lmstat -a -c 1055@localhost` 验证 server 可达性 |
