# macOS / CrossOver (Wine) Compatibility & Build Guide

> Documentation for running and building BodySlide and Outfit Studio for macOS via CrossOver / Wine.

---

## 1. Overview

When running Windows versions of BodySlide and Outfit Studio under CrossOver or Wine on Apple Silicon macOS, the application starts normally and can batch build outfits, but the 3D viewports (Outfit Studio main canvas and BodySlide 3D preview) fail to initialize with:
```text
"Outfit Studio: OpenGL context is not OK."
```

---

## 2. Compatibility Fixes in this Fork

1. **Forward-Compatible OpenGL Core Profile**:
   macOS CGL (Core OpenGL) strictly requires the `WGL_CONTEXT_FORWARD_COMPATIBLE_BIT_ARB` attribute flag for OpenGL 3.2+ Core Profile contexts. Without this flag, Wine's display driver (`winemac.so`) logs:
   ```text
   OS X only supports forward-compatible 3.2+ contexts
   ```
   and refuses to create the context. This fork adds `.ForwardCompatible()` to `GLSurface::GetGLContextAttribs()`.

2. **Fixed-Function Shader Cleanup**:
   Legacy fixed-function OpenGL calls (`glShadeModel`) that are illegal in core profiles on macOS are guarded out.

---

## 3. Build Instructions

### Recommended: Automated Build via GitHub Actions
Because BodySlide relies on Windows MSVC-compiled dependencies (`wxWidgets`, `Bullet3`, `OpenEXR` via `vcpkg`), the cleanest way to produce Windows binaries without maintaining a full Windows toolchain locally on macOS is using GitHub Actions:

1. Push your branch/tag to your GitHub fork:
   ```sh
   git push origin dev
   ```
2. Navigate to your repository on GitHub $\rightarrow$ **Actions** tab.
3. Run the **CMake RelWithDebInfo** workflow (`.github/workflows/cmake-release.yml`).
4. Download the generated `windows-x64-relwithdebinfo.zip` artifact.
5. Extract it directly into your Mod Organizer 2 mod directory (e.g. `CalienteTools/BodySlide/`).

### Local Cross-Compilation (Advanced)
If you prefer to cross-compile locally on macOS using LLVM (`clang-cl` / `lld-link`):
1. Provision the Microsoft CRT and Windows SDK sysroot using `xwin`:
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
