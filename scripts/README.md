# Scripts

This directory is reserved for local helper scripts.

Recommended script pattern:

```text
run_<project_name>.m
```

Supported actions:

```matlab
run_project_name("build")    % write the initial Zemax file
run_project_name("analyze")  % quick focus and analyze
run_project_name("open")     % open the focused ZMX in OpticStudio
run_project_name("all")      % build + focus + analyze
```

Do not hard-code private project data into the public skill package. Keep reusable ZOS-API helpers here and put project-specific prescription data in the project folder.

