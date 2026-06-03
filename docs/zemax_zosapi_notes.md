# Zemax ZOS-API Notes

## Initialization

The local script should allow the user to edit:

```matlab
ctx.helperDll = 'C:\ProgramData\Zemax\ZOS-API\Libraries\ZOSAPI_NetHelper.dll';
ctx.zemaxRoot = 'D:\Program Files\Ansys Zemax OpticStudio 2024 R1.00';
```

If the helper DLL is not found, search common locations:

```text
C:\ProgramData\Zemax\ZOS-API\Libraries
C:\ProgramData\Zemax\ZOS-API\Extensions
C:\Program Files\Ansys Zemax OpticStudio*\ZOS-API
```

## Semi-Diameter

When uncertain, use automatic solves for normal lens surfaces. Keep stops and image surfaces fixed.

```matlab
solver = surface.SemiDiameterCell.CreateSolveType(ZOSAPI.Editors.SolveType.Automatic);
surface.SemiDiameterCell.SetSolveData(solver);
```

## Cover Glass

Use two surfaces:

```text
cover front: thickness = 0.3 mm, material = cover glass
cover back: thickness = focus air, material = air
image plane
```

This prevents Quick Focus from changing the cover-glass thickness.

## Even Asphere Mapping

For a common patent equation:

```text
z = cr^2 / (1 + sqrt(1 - (1 + K)c^2r^2)) + A r^4 + B r^6 + C r^8 + D r^10
```

Map to Zemax Even Asphere:

```text
K -> conic
A -> 4th order
B -> 6th order
C -> 8th order
D -> 10th order
```

Uniform scale `s`:

```text
A4'  = A4  / s^3
A6'  = A6  / s^5
A8'  = A8  / s^7
A10' = A10 / s^9
```

## Quick Focus

Quick Focus is useful for a first pass, but it is not optimization. Always report what thickness changed.

