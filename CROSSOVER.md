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

3. **BC7 Textures (black meshes under MO2)**:
   Apple's OpenGL 4.1, which CrossOver exposes, has no `GL_ARB_texture_compression_bptc`. A BC7 DDS (DXGI 98/99, e.g. VRAMr output or most modern body textures) fails to upload and the mesh renders solid black. Run through MO2, the virtual file system resolves textures to those mod files, so meshes that look fine outside MO2 go black inside it. When BPTC is missing, `ResourceLoader::GLI_create_texture` now decodes BC7 to RGBA8 on the CPU (`lib/bcdec/bcdec.h`). Drivers with BPTC keep the compressed path.

4. **Viewport Lag (vertical sync)**:
   Under Wine on macOS `SwapBuffers` blocks until the display refresh (measured 11.6 ms of a 12 ms frame), and every mouse move renders a frame on the GUI thread. When `wine_get_version` is exported by `ntdll.dll`, `GLSurface::Initialize` calls `wglSwapIntervalEXT(0)`; measured swap time drops to 0.4 ms. Native Windows is unchanged.

5. **Local clang-cl Build**:
   `OutfitStudio.rc`, `BodySlide.rc` and `Resource.h` are plain ASCII (clang's resource preprocessor rejects UTF-16), and the manifests are added as CMake sources so `lld-link` embeds them (it rejects `/MANIFESTINPUT` without `/MANIFEST:EMBED`).

6. **BodySlide Slider Scroll Lag (WS_EX_COMPOSITED Bypass)**:
   In BodySlide, `leftPanel` (which hosts the slider scroll window containing over a thousand child controls) called `SetDoubleBuffered(true)`. In wxWidgets MSW, this applies the Win32 `WS_EX_COMPOSITED` extended style. Under Wine on macOS, `WS_EX_COMPOSITED` disables fast bitblt scrolling in `ScrollWindowEx` and forces Wine's display server to synchronously redraw and recomposite all 1,500+ controls on every scroll notch. This fork skips `SetDoubleBuffered(true)` on parent panels when running under Wine (`PlatformUtil::IsRunningOnWine()`), enabling instant hardware bitblt scrolling.

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
