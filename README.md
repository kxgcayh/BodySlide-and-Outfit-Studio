BodySlide and Outfit Studio (macOS CrossOver / Wine Edition)
============================================================

A tool to convert, create, and customize outfits and bodies for The Elder Scrolls, Fallout, and Starfield.

> **macOS Compatibility Fork**: This repository is a dedicated fork of the official [ousnius/BodySlide-and-Outfit-Studio](https://github.com/ousnius/BodySlide-and-Outfit-Studio) specifically tailored and maintained to run seamlessly on Apple Silicon macOS via **CrossOver / Wine** (including virtual filesystem execution through **Mod Organizer 2**).
>
> This fork actively tracks and integrates updates from upstream so macOS players can enjoy the latest releases and features (such as 6.1.0's MCP server, add-on/toggle sliders, 10-bit color, and sharper HDRi) with out-of-the-box Wine graphics and performance compatibility.

[![Release](https://img.shields.io/github/v/release/kxgcayh/BodySlide-and-Outfit-Studio?label=CrossOver%20Release)](https://github.com/kxgcayh/BodySlide-and-Outfit-Studio/releases)

---

## macOS / CrossOver Features in this Fork

* **Forward-Compatible OpenGL Core Profile**: Requests forward-compatible 3.2+ contexts required by Apple CGL / Wine `winemac.so`.
* **BC7 CPU Fallback**: Automatically decompresses BC7 (DXGI 98/99) DDS textures to RGBA8 on the CPU when `GL_ARB_texture_compression_bptc` is unavailable, fixing black meshes when textures are resolved through MO2 on Apple Silicon.
* **Eliminated Viewport Lag**: Disables vsync via `wglSwapIntervalEXT(0)` under Wine, dropping buffer swap wait from 11.6 ms to 0.4 ms per frame during mouse interaction.
* **Smooth BodySlide Slider Scrolling**: Bypasses `WS_EX_COMPOSITED` on parent panels under Wine, enabling instant bitblt scrolling instead of redrawing 1,500+ child controls on every wheel tick.
* **Fixed-Function Shader Cleanup**: Removes deprecated `glShadeModel` calls that violate OpenGL Core Profile on macOS.
* **Local Clang Cross-Compilation**: Clean `clang-cl` and `lld-link` build setup directly on macOS without requiring a Windows VM. See [CROSSOVER.md](CROSSOVER.md) for full details.

---

## Upstream & Documentation
* **Official Upstream Repository**: [ousnius/BodySlide-and-Outfit-Studio](https://github.com/ousnius/BodySlide-and-Outfit-Studio)
* [Upstream Wiki Overview](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki)
* [Guides and Documentation](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Guides-and-Documentation)
* [macOS CrossOver Guide & Build Instructions](CROSSOVER.md)
* [Building on Windows (Visual Studio or CMake)](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Building-on-Windows-%28Visual-Studio-or-CMake%29)
* [Building on Linux](https://github.com/ousnius/BodySlide-and-Outfit-Studio/wiki/Building-on-Linux)

---

## Credits / Libraries
**Upstream Authors & Contributors:**
* **[ousnius](https://github.com/ousnius)** — Lead maintainer and developer of BodySlide and Outfit Studio, creator of [nifly](https://github.com/ousnius/nifly), and author of the modern 6.x architecture.
* **[Caliente](https://www.nexusmods.com/skyrimspecialedition/users/3330686)** — Original creator of BodySlide and Caliente's Beautiful Bodies Enhancer (CBBE).
* **jonwd7** — NIF file structure and general architectural guidance.
* **degenerated1123** — Shaders and rendering implementation.
* **NifTools team** — Community reverse-engineering of the NetImmerse/Gamebryo format.
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
* [bcdec](https://github.com/iOrange/bcdec) (BC7 decoding where the GPU driver has no BPTC)
* [half - IEEE 754-based half-precision floating point library](https://half.sourceforge.net/)
* [Miniball](https://github.com/hbf/miniball)
* [LZ4(F)](https://github.com/lz4/lz4)
* [TinyXML-2](https://github.com/leethomason/tinyxml2)
* [nlohmann/json](https://github.com/nlohmann/json)
* [fkYAML](https://github.com/fktn-k/fkYAML)
* [cpp-httplib](https://github.com/yhirose/cpp-httplib) (MCP server)
* [Catch2 v3](https://github.com/catchorg/Catch2) (optional tests)
* FSEngine (BSA/BA2 library)
* [Autodesk FBX SDK](https://aps.autodesk.com/developer/overview/fbx-sdk) (optional FBX import/export)
* [Bullet](https://github.com/bulletphysics/bullet3) (optional physics preview)
