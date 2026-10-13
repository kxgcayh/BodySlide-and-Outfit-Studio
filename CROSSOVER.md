# Building BodySlide and Outfit Studio for macOS (CrossOver / Wine)

This guide is for developers building BodySlide and Outfit Studio for macOS running under CrossOver or Wine. If you only want to install pre-built binaries, extract the release package into your Mod Organizer 2 mod directory.

---

## Overview

When running Windows versions of BodySlide and Outfit Studio under CrossOver or Wine on macOS, the 3D viewports (Outfit Studio main canvas and BodySlide 3D preview) require a forward-compatible OpenGL core profile context.

This fork provides the necessary OpenGL core profile initialization fixes (`.ForwardCompatible()`) so that the application viewports render without errors under Wine on macOS.

---

## Build Requirements

- macOS with Apple Silicon or Intel
- Homebrew
- CMake 3.20 or newer
- LLVM / Clang toolchain (`clang-cl`, `lld-link`, `llvm-rc`, `llvm-lib`)
- NASM (assembler needed for `libjpeg-turbo`)
- Python 3 and `xwin` (Microsoft CRT and Windows SDK sysroot generator)
- vcpkg for package management
- Optional: CrossOver or Wine for testing the resulting binaries

Install host utilities via Homebrew:

```sh
brew install cmake llvm ninja nasm pkg-config
cargo install xwin || pip install xwin
```

Ensure LLVM binaries are accessible in your PATH:

```sh
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
```

---

## Preferred: GitHub Actions CI

Because BodySlide relies on Windows libraries (`wxWidgets`, `Bullet3`, `OpenEXR`), building via GitHub Actions provides pre-compiled Windows x64 binaries:

1. Push your changes to your repository branch:
   ```sh
   git push origin dev
   ```
2. In GitHub, go to the **Actions** tab.
3. Select the **CMake Release** or **CMake RelWithDebInfo** workflow (`.github/workflows/cmake-release.yml`) and click **Run workflow**.
4. Download the compiled zip artifact and extract it into your Mod Organizer 2 mod folder (e.g. `CalienteTools/BodySlide/`).

---

## Local Cross-Compilation with LLVM and vcpkg

Local cross-compilation produces native Windows x64 PE executables directly on macOS using `clang-cl` and `lld-link`.

### 1. Provision Windows SDK and MSVC Sysroot

Use `xwin` to fetch the Microsoft CRT and Windows 10/11 SDK headers and import libraries:

```sh
xwin --accept-license 17 splat --output ~/.xwin
```

### 2. Setup vcpkg and LLVM Cross-Triplet

Clone and bootstrap vcpkg:

```sh
git clone https://github.com/microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh -disableMetrics
```

Create a custom toolchain file `triplets/x64-windows-llvm-toolchain.cmake`:

```cmake
set(CMAKE_SYSTEM_NAME Windows)
set(CMAKE_SYSTEM_PROCESSOR AMD64)

set(WINSYSROOT "$ENV{HOME}/.xwin")

set(CMAKE_C_COMPILER "/opt/homebrew/opt/llvm/bin/clang-cl")
set(CMAKE_CXX_COMPILER "/opt/homebrew/opt/llvm/bin/clang-cl")
set(CMAKE_LINKER "/opt/homebrew/bin/lld-link")
set(CMAKE_AR "/opt/homebrew/opt/llvm/bin/llvm-lib")
set(CMAKE_RANLIB "/opt/homebrew/opt/llvm/bin/llvm-ranlib")
set(CMAKE_RC_COMPILER "/opt/homebrew/opt/llvm/bin/llvm-rc")
set(CMAKE_MT "/opt/homebrew/opt/llvm/bin/llvm-mt")

set(MSVC_COMMON_FLAGS "--target=x86_64-pc-windows-msvc -msse4.1 -fuse-ld=lld-link -imsvc ${WINSYSROOT}/crt/include -imsvc ${WINSYSROOT}/sdk/include/ucrt -imsvc ${WINSYSROOT}/sdk/include/shared -imsvc ${WINSYSROOT}/sdk/include/um -imsvc ${WINSYSROOT}/sdk/include/winrt /D_CRT_SECURE_NO_WARNINGS /DNOMINMAX")

set(CMAKE_C_FLAGS_INIT "${MSVC_COMMON_FLAGS}")
set(CMAKE_CXX_FLAGS_INIT "${MSVC_COMMON_FLAGS} /EHsc")
set(CMAKE_RC_FLAGS_INIT "-I ${WINSYSROOT}/sdk/include/um -I ${WINSYSROOT}/sdk/include/shared -I ${WINSYSROOT}/sdk/include/ucrt -I ${WINSYSROOT}/crt/include")

cmake_policy(SET CMP0091 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0091 NEW)

set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>" CACHE STRING "")
set(CMAKE_C_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG" CACHE STRING "" FORCE)
set(CMAKE_C_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1" CACHE STRING "" FORCE)

set(MSVC_LINK_DIRS "/LIBPATH:${WINSYSROOT}/crt/lib/x86_64 /LIBPATH:${WINSYSROOT}/sdk/lib/ucrt/x86_64 /LIBPATH:${WINSYSROOT}/sdk/lib/um/x86_64")

set(CMAKE_EXE_LINKER_FLAGS_INIT "${MSVC_LINK_DIRS}")
set(CMAKE_SHARED_LINKER_FLAGS_INIT "${MSVC_LINK_DIRS}")
set(CMAKE_MODULE_LINKER_FLAGS_INIT "${MSVC_LINK_DIRS}")
```

Create the triplet file `triplets/x64-windows-llvm.cmake`:

```cmake
set(VCPKG_TARGET_ARCHITECTURE x64)
set(VCPKG_CRT_LINKAGE static)
set(VCPKG_LIBRARY_LINKAGE static)

set(VCPKG_C_FLAGS_RELEASE "/MT")
set(VCPKG_CXX_FLAGS_RELEASE "/MT")
set(VCPKG_C_FLAGS_DEBUG "/MTd")
set(VCPKG_CXX_FLAGS_DEBUG "/MTd")

unset(VCPKG_CMAKE_SYSTEM_NAME)
set(VCPKG_CHAINLOAD_TOOLCHAIN_FILE "${CMAKE_CURRENT_LIST_DIR}/x64-windows-llvm-toolchain.cmake")
```

### 3. Install Dependencies with vcpkg

Install the required packages using the LLVM cross-triplet:

```sh
./vcpkg install --triplet x64-windows-llvm \
  "wxwidgets[debug-support]" \
  bullet3 \
  openexr \
  directxmath \
  fmt \
  nlohmann-json \
  rapidcsv \
  simpleini \
  spdlog \
  xbyak
```

### 4. Configure and Build BodySlide and Outfit Studio

From the repository root:

```sh
cmake -S . -B build-win \
  -DCMAKE_SYSTEM_NAME=Windows \
  -DCMAKE_SYSTEM_PROCESSOR=AMD64 \
  -DCMAKE_TOOLCHAIN_FILE="/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake" \
  -DVCPKG_CHAINLOAD_TOOLCHAIN_FILE="/path/to/vcpkg/triplets/x64-windows-llvm-toolchain.cmake" \
  -DVCPKG_TARGET_TRIPLET=x64-windows-llvm \
  -DCMAKE_BUILD_TYPE=Release \
  -DBSOS_BUILD_TESTS=OFF \
  -DBSOS_ENABLE_FBXSDK=OFF

cmake --build build-win --config Release --parallel
```

### 5. Build Outputs and Local Testing

The compiled executables are produced in the build directory:

- `build-win/BodySlide.exe`
- `build-win/OutfitStudio.exe`

To test locally with CrossOver:

```sh
WINEPREFIX="$HOME/Library/Application Support/CrossOver/Bottles/MacGaming" \
CX_BOTTLE="MacGaming" \
/Applications/CrossOver.app/Contents/SharedSupport/CrossOver/bin/wine ./build-win/BodySlide.exe
```

For deployment to Mod Organizer 2, ensure the executable files are placed beside the application directories:

- `BodySlide.exe`
- `OutfitStudio.exe`
- `res/`
- `lang/`
- `Config.xml`
- `BodySlide.xml`
- `OutfitStudio.xml`
