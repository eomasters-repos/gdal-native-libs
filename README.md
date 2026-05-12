# GDAL Native Libraries

This repository builds native GDAL libraries from official GDAL release tags:
<https://github.com/OSGeo/gdal/releases>

The workflow downloads the release source archive for the given tag and builds with CMake.

## Supported platforms

| OS      | x64 | ARM |
|---------|-----|-----|
| Windows | ✅  | ❌  |
| Linux   | ✅  | ✅  |
| macOS   | ❌  | ✅  |

## Workflow trigger

The build is started manually via GitHub Actions (`workflow_dispatch`) with:

- `gdal_tag` (example: `v3.13.0`)

Important:
- The workflow currently supports CMake builds for `GDAL >= 3.5.0`.
- Java bindings are always built.
- Python bindings are not built.
- C# bindings are not built.
- No additional GDAL drivers are enabled explicitly in this repository.

## Build dependencies

The workflow installs platform dependencies during the run:

- CMake + Ninja
- PROJ
- Java 17 + ANT (for Java bindings)

No GDAL test execution is performed.

## Release behavior

The workflow has three stages:

1. `build` (matrix for Linux/macOS/Windows)
2. `reset-release-tag` (runs only if build succeeded)
3. `publish`

`reset-release-tag` removes an existing GitHub release (and tag) for `gdal_tag`, so assets are always republished cleanly after a successful build.

## Artifact format

Each platform produces one archive:

- `gdal-<tag>-linux-x64.tar.gz`
- `gdal-<tag>-linux-arm64.tar.gz`
- `gdal-<tag>-macos-arm64.tar.gz`
- `gdal-<tag>-windows-x64.zip`

Each archive is repackaged as a minimal runtime bundle and contains only:

- `lib/` (native GDAL runtime and related library files from install `lib`)
- `lib/gdal.pc`

## Java runtime notes

For JVM usage, set at least:

- `-Djava.library.path=<bundle>/lib/jni`

Depending on your runtime environment, you may also need loader path setup for native dependencies:

- Windows: add native library directories to `PATH`
- Linux: add native library directories to `LD_LIBRARY_PATH`
- macOS: add native library directories to `DYLD_LIBRARY_PATH`
