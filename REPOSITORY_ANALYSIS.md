# Kore 3 Repository Analysis

## Executive Summary

**Kore 3** is a low-level toolkit for cross-platform game engine development. It's a modern rewrite of the original Kore library, focused on a new graphics API (similar to WebGPU) and a custom shader language called Kongruent.

---

## How It Works

### 1. Base Architecture

Kore 3 follows a layered architecture:

```
┌─────────────────────────────────────┐
│     User Application                │
│     (your game/engine)              │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     Kore 3 Public API               │
│     (headers in includes/kore3/)    │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     Core Implementation             │
│     (sources/ - common code)        │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     Specific Backends               │
│     (backends/ - per platform)      │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     Native System APIs              │
│     (Windows, macOS, Linux, etc.)   │
└─────────────────────────────────────┘
```

### 2. Build System (kmake)

Kore uses a meta-build-tool called **kmake** that:

1. **Reads `kfile.js`** - A JavaScript configuration file that defines:
   - Which files to include
   - Which backends to use
   - Dependencies and libraries

2. **Selects Backends** based on:
   - Target platform (Windows, Linux, macOS, Android, iOS, Web)
   - Graphics API chosen (Direct3D 11/12, Vulkan, Metal, OpenGL, WebGPU)
   - Audio API (WASAPI, DirectSound, CoreAudio, ALSA)

3. **Generates Projects** for IDEs:
   - Visual Studio (.vcxproj)
   - Xcode (.xcodeproj)
   - Makefiles
   - CMake

**Example usage:**
```bash
# Build for Windows with Direct3D 12
path/to/Kore/make windows -g direct3d12

# Build for Linux with Vulkan
path/to/Kore/make linux -g vulkan

# Build for macOS with Metal
path/to/Kore/make osx -g metal
```

### 3. Entry Point - `kickstart()`

Every Kore application starts with the `kickstart()` function:

```c
int kickstart(int argc, char **argv) {
    // Initialize system
    kore_init("My Game", 1280, 720, NULL, NULL);
    
    // Setup GPU
    kore_gpu_init(KORE_GPU_API_DIRECT3D11);
    
    // Register callbacks
    kore_set_update_callback(update_callback, NULL);
    
    // Start main loop
    kore_start();
    
    return 0;
}
```

### 4. Main Loop

Kore manages the main loop automatically after `kore_start()`:

```
┌──────────────────┐
│  Process Input   │ ← Keyboard, mouse, gamepad, touch
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Window Events    │ ← Resize, close, focus, PPI
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Update Callback  │ ← Your logic (physics, AI, etc.)
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Render Frame     │ → GPU commands
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Present/Swap     │ → Display on screen
└────────┬─────────┘
         ↓
    (repeats until kore_stop())
```

### 5. GPU Abstraction

Kore 3 provides a unified API for multiple graphics APIs:

```c
// THE SAME API works for all platforms
struct kore_gpu_command_list *list = kore_gpu_command_list_create();
kore_gpu_command_list_begin(list);

// Set pipeline, buffers, textures
kore_gpu_command_list_set_pipeline(list, pipeline);
kore_gpu_command_list_draw(list, vertex_count, instance_count);

kore_gpu_command_list_end(list);
kore_gpu_command_list_submit(list);
```

Internally, Kore translates this to:
- **Direct3D 11/12** on Windows
- **Metal** on macOS/iOS
- **Vulkan** on Linux/Android
- **OpenGL** as fallback
- **WebGPU** on Web

### 6. Directory Structure

```
/workspace/
├── includes/kore3/          # Public API headers
│   ├── gpu/                 # Graphics API
│   ├── input/               # Input (keyboard, mouse, gamepad)
│   ├── mixer/               # Audio
│   ├── io/                  # File system
│   ├── math/                # Math (vectors, matrices)
│   ├── threads/             # Threading
│   └── ...
│
├── sources/                 # Core implementation (common to all platforms)
│   ├── gpu/
│   ├── audio/
│   ├── input/
│   ├── io/
│   ├── math/
│   └── libs/                # Third-party libraries (stb_image, lz4, etc.)
│
├── backends/                # Platform-specific implementations
│   ├── system/
│   │   ├── windows/
│   │   ├── macos/
│   │   ├── linux/
│   │   ├── android/
│   │   └── ...
│   ├── gpu/
│   │   ├── direct3d11/
│   │   ├── direct3d12/
│   │   ├── vulkan/
│   │   ├── metal/
│   │   └── opengl/
│   └── audio/
│       ├── wasapi/
│       ├── directsound/
│       └── ...
│
├── shaders/                 # Kongruent Shaders (.kong)
├── tests/                   # Example tests
└── kfile.js                 # Build configuration
```

### 7. Main Modules

#### System/Windows (`system.h`, `window.h`)
- Window creation and management
- Fullscreen, resize, move
- Event callbacks
- Multi-window support

#### GPU (`gpu/*.h`)
- Device, command lists, pipelines
- Buffers (vertex, index, constant, storage)
- Textures and samplers
- Ray tracing (experimental)

#### Audio (`mixer/*.h`)
- Mixer with multiple channels
- Sound loading (.wav, .ogg via stb_vorbis)
- Music streaming
- Volume and pan control

#### Input (`input/*.h`)
- Keyboard (pressed/down states)
- Mouse (position, movement, buttons)
- Gamepad (up to 8 controllers)
- Touch surface
- Motion sensors (accelerometer, gyroscope)

#### I/O (`io/*.h`)
- File read/write
- Save path abstraction
- Asset packaging

#### Network (`network/*.h`)
- HTTP requests (GET/POST)
- TCP/UDP sockets

#### Math (`math/*.h`)
- Vectors (vec2, vec3, vec4)
- Matrices (mat4)
- Quaternions
- Random number generation

#### Threads (`threads/*.h`)
- Thread creation/join
- Mutex, semaphore, event
- Atomic operations

### 8. Included Third-Party Libraries

- **stb_image.h** - Image loading (PNG, JPG, etc.)
- **stb_vorbis.h** - Ogg Vorbis audio decoding
- **lz4** - Fast compression
- **offalloc** - Custom memory allocator
- **GLEW** - OpenGL extension loader (for OpenGL backend)

### 9. Supported Platforms

| Platform | System | GPU | Audio |
|----------|--------|-----|-------|
| Windows | Win32/UWP | D3D11, D3D12, Vulkan, OpenGL | WASAPI, DirectSound |
| macOS | Cocoa | Metal, OpenGL | CoreAudio |
| Linux | X11/Wayland | Vulkan, OpenGL | ALSA |
| Android | Android SDK | Vulkan, OpenGL ES | OpenSL ES |
| iOS | UIKit | Metal, OpenGL ES | CoreAudio |
| Web | Emscripten/WASM | WebGPU, WebGL | Web Audio API |

### 10. VR Support

Optional support for:
- **Oculus Rift** (via LibOVR)
- **SteamVR** (OpenVR)
- **HoloLens** (Windows Mixed Reality)

---

## Kore 3 vs Previous Versions Differences

| Feature | Kore v1 | Kore v2 | Kore 3 |
|---------|---------|---------|--------|
| Language | C++ | C++ | C |
| Graphics API | OpenGL/DirectX | OpenGL/DirectX/Vulkan | New API (WebGPU-like) |
| Shaders | GLSL/HLSL | GLSL/HLSL | Kongruent |
| Status | Legacy | Stable | Experimental |

---

## Typical Use Cases

1. **Game Engines**: Armored Pixel, Krom, Kha use Kore
2. **Graphics Tools**: Editors, 3D viewers
3. **Cross-Platform Applications**: Apps that need to run on desktop/mobile/web
4. **Rapid Prototyping**: Thanks to simple API and easy build system

---

## Minimal Complete Example

```c
#include <kore3/system.h>
#include <kore3/window.h>
#include <kore3/gpu/gpu.h>
#include <kore3/input/keyboard.h>
#include <kore3/log.h>

void update(void *data) {
    // Check input
    if (kore_keyboard_pressed(KORE_KEY_ESCAPE)) {
        kore_stop();
        return;
    }
    
    // Game logic here
    
    // Basic rendering
    // (setup pipeline, draw calls, etc.)
}

int kickstart(int argc, char **argv) {
    kore_log(KORE_LOG_LEVEL_INFO, "Starting application...");
    
    // Initialize with 1280x720 window
    kore_init("My App", 1280, 720, NULL, NULL);
    
    // Initialize GPU (auto-selects best API)
    kore_gpu_init(KORE_GPU_API_DEFAULT);
    
    // Setup update callback
    kore_set_update_callback(update, NULL);
    
    // Start main loop
    kore_start();
    
    kore_log(KORE_LOG_LEVEL_INFO, "Application closed.");
    return 0;
}
```

---

## Strengths

✅ **Truly cross-platform** - Same code for all platforms  
✅ **Unified GPU API** - Write once, run on D3D/Vulkan/Metal  
✅ **Smart build system** - kmake generates projects automatically  
✅ **Low-level** - Full control over GPU and resources  
✅ **Lightweight** - No unnecessary overhead  
✅ **Pure C** - Easy to integrate with any language  

## Points of Attention

⚠️ **Kore 3 is experimental** - API may change  
⚠️ **Limited documentation** - Few official examples  
⚠️ **Learning curve** - Requires graphics knowledge  
⚠️ **Few high-level features** - Not a complete engine  

---

## Additional Resources

- **Official Repository**: https://github.com/Kode/Kore
- **Samples**: https://github.com/Kode/Kore-Samples
- **Shader Language**: https://github.com/Kode/Kongruent
- **Kore v2 (stable)**: https://github.com/Kode/Kore/tree/v2
- **Kha Framework**: High-level framework based on Kore

