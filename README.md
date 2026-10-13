BodySlide and Outfit Studio
===========================

BodySlide and Outfit Studio, a tool to convert, create, and customize outfits and bodies for The Elder Scrolls, Fallout and Starfield.

[![CMake Release](https://github.com/ousnius/BodySlide-and-Outfit-Studio/actions/workflows/cmake-release.yml/badge.svg)](https://github.com/ousnius/BodySlide-and-Outfit-Studio/actions/workflows/cmake-release.yml)

## Documentation
* [Wiki Overview](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki)
* [Guides and Documentation](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Guides-and-Documentation)

## Build Instructions
* [Building on Windows (Visual Studio or CMake)](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Building-on-Windows-%28Visual-Studio-or-CMake%29)
* [Building on Linux](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Building-on-Linux)
* [Building for macOS / CrossOver (Wine)](#building-for-macos--crossover-wine)
* [Installation and Settings](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Installation-and-Settings)
* [Installation on Linux](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Installation-on-Linux)

## Building for macOS / CrossOver (Wine)

When running Windows versions of BodySlide and Outfit Studio under CrossOver or Wine on Apple Silicon macOS, the OpenGL context creation requires specific flags to work with macOS's CGL driver:

### Compatibility Fixes in this Fork
1. **Forward-Compatible Core Profile**:
   macOS CGL (Core OpenGL) strictly requires the `WGL_CONTEXT_FORWARD_COMPATIBLE_BIT_ARB` attribute flag for OpenGL 3.2+ Core Profile contexts. Without this flag, Wine's display driver (`winemac.so`) logs `OS X only supports forward-compatible 3.2+ contexts` and refuses to initialize the context, leaving the 3D viewport broken. This fork adds `.ForwardCompatible()` to `GLSurface::GetGLContextAttribs()`.
2. **Fixed-Function Shader Cleanup**:
   Legacy fixed-function OpenGL calls (`glShadeModel`) that are illegal in core profiles on macOS are guarded out.

### Build Methods for macOS Users

#### Option 1: Automated Build via GitHub Actions (Recommended)
Because BodySlide relies on Windows MSVC-compiled dependencies (wxWidgets, Bullet3, OpenEXR via vcpkg), the cleanest way to produce Windows binaries on macOS is using GitHub Actions:
1. Push this branch/tag to your GitHub fork.
2. In your GitHub repository, navigate to the **Actions** tab.
3. Run the **CMake RelWithDebInfo** workflow (`.github/workflows/cmake-release.yml`).
4. Download the generated `windows-x64-relwithdebinfo.zip` artifact.
5. Extract it directly into your Mod Organizer 2 mod folder (e.g. `CalienteTools/BodySlide/`).

#### Option 2: Cross-Compiling with Windows SDK & vcpkg on macOS
If cross-compiling locally on macOS using LLVM (`clang-cl` / `lld-link`):
1. Provision Microsoft CRT and Windows SDK sysroot using `xwin`:
   ```sh
   xwin --accept-license 17 splat --output ~/.xwin
   ```
2. Configure CMake with the Windows target triplet:
   ```sh
   cmake -B build -S . \
     -DCMAKE_SYSTEM_NAME=Windows \
     -DCMAKE_C_COMPILER=clang-cl \
     -DCMAKE_CXX_COMPILER=clang-cl \
     -DCMAKE_TOOLCHAIN_FILE="$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake" \
     -DVCPKG_TARGET_TRIPLET=x64-windows-static \
     -DBSOS_BUILD_TESTS=OFF \
     -DBSOS_ENABLE_FBXSDK=OFF
   cmake --build build --config Release --parallel
   ```

## Credits / Libraries
**Created by and/or with the help of:**
* Caliente
* ousnius
* jonwd7 for NIF and general help
* degenerated1123 for help with shaders
* NifTools team

**Relevant work:**
* [nifly](https://github.com/ousnius/nifly): C++ NIF library for the NetImmerse File Format (NetImmerse, Gamebryo, Creation Engine).
* [NiflySharp](https://github.com/ousnius/NiflySharp): Native C# / .NET version of nifly that uses source generation based on [nifxml](https://github.com/niftools/nifxml).
* [Faster HDT-SMP (FSMP)](https://github.com/DaymareOn/hdtSMP64): Skinned mesh physics for Skyrim SE/AE/VR, maintained by DaydreamingDay / DaymareOn, forked from [Karonar1](https://github.com/Karonar1/hdtSMP64) and [aers](https://github.com/aers/hdtSMP64), from the original [work](https://github.com/HydrogensaysHDT/hdt-skyrimse-mods) by HydrogensaysHDT.
  The physics preview in BodySlide and Outfit Studio is a port of its `hdtSkinnedMesh` core (GPL-3.0, like this project) so it simulates the same physics XML files with the same behavior. See [src/physics/hdt/README.md](src/physics/hdt/README.md) for the port notes.

**Libraries used:**
* [wxWidgets](https://wxwidgets.org/)
* [nifly](https://github.com/ousnius/nifly)
* [OpenGL](https://www.opengl.org/)
* [OpenGL Image (GLI)](https://github.com/g-truc/gli)
* [Simple OpenGL Image Library 2 (SOIL2)](https://github.com/SpartanJ/SOIL2)
* [half - IEEE 754-based half-precision floating point library](https://half.sourceforge.net/)
* [Miniball](https://github.com/hbf/miniball)
* [LZ4(F)](https://github.com/lz4/lz4)
* [TinyXML-2](https://github.com/leethomason/tinyxml2)
* [nlohmann/json](https://github.com/nlohmann/json)
* [fkYAML](https://github.com/fktn-k/fkYAML)
* [Catch2 v3](https://github.com/catchorg/Catch2) (optional tests)
* FSEngine (BSA/BA2 library)
* [Autodesk FBX SDK](https://aps.autodesk.com/developer/overview/fbx-sdk) (optional FBX import/export)
* [Bullet](https://github.com/bulletphysics/bullet3) (optional physics preview)
