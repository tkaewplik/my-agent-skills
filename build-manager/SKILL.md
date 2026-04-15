---
name: build-manager
description: >
  Build, rebuild, and manage the Godot GDExtension C++ project using Conan and CMake.
  Use this skill whenever the user asks to build, compile, rebuild, clean, install dependencies,
  or troubleshoot build errors. Also trigger when the user says things like "build it",
  "compile the extension", "run conan", "fix build errors", "clean build", "rebuild",
  "install deps", or "update dependencies".
---

# Build Manager

Help the user build, clean, and manage the Godot GDExtension C++ project that uses Conan for dependency management and CMake as the build system.

## Project Structure

- `conanfile.py` — Conan recipe defining dependencies (asio, opencv) and build settings
- `CMakeLists.txt` — CMake build configuration
- `src/` — C++ source files for the GDExtension
- `godot-cpp/` — Godot C++ bindings (git submodule)
- `project/` — Godot project directory where the built `.so`/`.dll` lands

## Build Commands

### Full build (from clean state)

```bash
conan install . --build=missing -s build_type=Release
cd build/Release && cmake ../.. -DCMAKE_TOOLCHAIN_FILE=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release && cmake --build . -j$(nproc)
```

### Quick rebuild (after code changes only)

```bash
cd build/Release && cmake --build . -j$(nproc)
```

### Debug build

```bash
conan install . --build=missing -s build_type=Debug
cd build/Debug && cmake ../.. -DCMAKE_TOOLCHAIN_FILE=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Debug && cmake --build . -j$(nproc)
```

### Clean build

```bash
rm -rf build/
conan install . --build=missing -s build_type=Release
cd build/Release && cmake ../.. -DCMAKE_TOOLCHAIN_FILE=generators/conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Release && cmake --build . -j$(nproc)
```

## Steps

1. Determine what the user wants: full build, quick rebuild, clean, debug, or dependency update.
2. Check if `build/` directory exists to decide if `conan install` is needed.
3. Run the appropriate commands from the section above.
4. If the build fails, read the error output and:
   - For missing dependencies: re-run `conan install . --build=missing`
   - For CMake errors: check `CMakeLists.txt` and `conanfile.py`
   - For C++ compilation errors: read the relevant source file and fix the issue
5. Report the result using the output format below.

## Output Format

Use this structure every time:

```
## Build Result — <build_type>

### Status: <SUCCESS or FAILED>

### Summary
- <what was built or what failed>
- <library output location if successful>

### Errors (if any)
- <error description and suggested fix>
```

## Notes

- The project requires C++17 minimum (`check_min_cppstd(self, 17)` in conanfile.py).
- The built shared library goes into the Godot project's `bin/` directory for the GDExtension to load.
- Always use `-j$(nproc)` for parallel builds.
- If `godot-cpp/` submodule is not initialized, run `git submodule update --init --recursive` first.
- Conan profile should be detected automatically, but if issues arise check with `conan profile show`.
