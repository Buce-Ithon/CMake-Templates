# OpenGL-Project

[cite_start]A modern, platform-agnostic OpenGL graphics rendering project template[cite: 1]. This project provides a robust CMake configuration that seamlessly integrates with multi-toolchains (MSVC, MinGW, Clang/GCC) and development environments like Neovim and Visual Studio.

---

## 📂 Recommended Project Structure

To ensure that the dependencies and source files are correctly scanned by CMake, please organize your project files according to the following layout:

```text
Your-OpenGL-Project/
├── CMakeLists.txt              # The modern Target-based CMake configuration
├── CMakePresets.json           # Unified build configuration presets
├── src/                        # Application source directory
│   └── main.cpp                # App entry point
├── include/                    # Private header directory
│   └── custom_shader.hpp       # Your project headers
└── ext/                        # Third-party dependencies
    ├── glad/
    │   ├── include/glad/glad.h
    │   └── src/glad.c          # Automatically built into the executable
    ├── stb/
    │   └── include/stb_image.h
    ├── glfw-3.4_win64_msvc17/  # Pre-compiled GLFW for MSVC (Visual Studio)
    │   ├── Debug/include/
    │   ├── Debug/lib/
    │   ├── Release/include/
    │   └── Release/lib/
    └── glfw-3.4_win64_mingw-w64/ # Pre-compiled GLFW for MinGW (GCC/Clang)
        ├── include/
        └── lib/libglfw3.a
```

---

## ⚙️ Development Environment Setup

1. Neovim / VS Code (Recommended)

This template is optimized for Language Server Protocols (LSP) such as **Clangd**:

- `compile_commands.json`: CMake is configured to automatically generate and mirror this file into your project root. This ensures that your LSP can resolve paths for auto-completion, diagnostics, and navigation instantly.  
- **Note**: It explicitly avoids copying this file under native MSVC Visual Studio environments to prevent generation conflicts.

2. Multi-Config vs Single-Config Generators

- **MinGW Makefiles / Ninja**: These are Single-Config generators. Debug and Release targets are built into separate dedicated build directories.
- **Visual Studio 17 2022**: This is a Multi-Config generator. A single configuration step generates a unified solution (`.sln`), and you switch between `Debug` or `Release` directly at the build stage.

## 🛠️ Building the Project via CLI

With **CMake Presets (v4)**, you no longer need to type long, messy paths and flags. All options are wrapped into standard preset names.  

### Option A: Using Ninja (Highly Recommended for Speed)

Perfect for Neovim workflows on Windows.

```Bash
# 1. Configure the project (Pick debug or release)
cmake --preset ninja-debug
# OR
cmake --preset ninja-release

# 2. Build the target
cmake --build --preset ninja-debug
# OR
cmake --build --preset ninja-release
```

### Option B: Using MinGW Makefiles

Alternative for environments without Ninja installed.

```Bash
# 1. Configure
cmake --preset mingw-debug
# OR
cmake --preset mingw-release

# 2. Build
cmake --build --preset mingw-debug
# OR
cmake --build --preset mingw-release
```

### Option C: Using Visual Studio 2022 (MSVC)

For standard Windows MSVC developers.

```Bash
# 1. Configure once (Generates the unified VS Solution)
cmake --preset msvc

# 2. Build specifying the dynamic configuration 
cmake --build --preset msvc-debug
# OR
cmake --build --preset msvc-release
```

## 🎛️ Advanced / Optional Arguments

If you want to manually override options or tweak behaviors without touching `CMakePresets.json`, you can pass standard CMake arguments to the `--preset` pipeline.

1. Specifying a Custom Installation Path

The presets define default installation paths (`${sourceDir}/install/...`). You can dynamically override it via:

```Bash
cmake --preset ninja-release -DCMAKE_INSTALL_PREFIX="D:/My/Custom/Install/Prefix"
```

2. Verbose Compilation Output
To see the exact compilation and linking commands executed by the compiler (helpful for debugging link errors):

```Bash
cmake --build --preset ninja-debug --verbose
```

3. Parallel Building
Accelerate your compilation times by utilizing multiple CPU cores:

```Bash
# Use 4 parallel jobs to build
cmake --build --preset ninja-debug --parallel 4
# Or
cmake --build --preset ninja-debug -j 4
```