# Kore 3 - Mermaid Diagrams

## 1. General Architecture Overview

```mermaid
graph TB
    subgraph "User Application Layer"
        A[Your Game/Engine Code]
    end
    
    subgraph "Kore 3 Public API"
        B[System & Window API]
        C[GPU API]
        D[Audio API]
        E[Input API]
        F[I/O API]
        G[Network API]
        H[Math API]
        I[Threads API]
    end
    
    subgraph "Core Implementation"
        J[Common Code - sources/]
    end
    
    subgraph "Platform Backends"
        K[Windows Backend]
        L[macOS Backend]
        M[Linux Backend]
        N[Android Backend]
        O[iOS Backend]
        P[Web Backend]
    end
    
    subgraph "Graphics Backends"
        Q[Direct3D 11]
        R[Direct3D 12]
        S[Vulkan]
        T[Metal]
        U[OpenGL]
        V[WebGPU]
    end
    
    subgraph "Native APIs"
        W[Win32/UWP]
        X[Cocoa]
        Y[X11/Wayland]
        Z[Android SDK]
        AA[UIKit]
        AB[Emscripten]
    end
    
    A --> B
    A --> C
    A --> D
    A --> E
    A --> F
    A --> G
    A --> H
    A --> I
    
    B --> J
    C --> J
    D --> J
    E --> J
    F --> J
    G --> J
    H --> J
    I --> J
    
    J --> K
    J --> L
    J --> M
    J --> N
    J --> O
    J --> P
    
    K --> Q
    K --> R
    K --> S
    K --> U
    L --> T
    L --> U
    M --> S
    M --> U
    N --> S
    N --> U
    O --> T
    O --> U
    P --> V
    P --> U
    
    K --> W
    L --> X
    M --> Y
    N --> Z
    O --> AA
    P --> AB
```

---

## 2. Initialization Flow

```mermaid
sequenceDiagram
    participant App as Application
    participant Kick as kickstart()
    participant Init as kore_init()
    participant GPU as kore_gpu_init()
    participant Callback as kore_set_update_callback()
    participant Start as kore_start()
    participant Loop as Main Loop
    participant Backend as Platform Backend
    
    App->>Kick: kickstart(argc, argv)
    activate Kick
    
    Kick->>Init: kore_init(title, width, height)
    activate Init
    Init->>Backend: Create window
    Backend-->>Init: Window handle
    Init-->>Kick: Success
    deactivate Init
    
    Kick->>GPU: kore_gpu_init(api_type)
    activate GPU
    GPU->>Backend: Initialize graphics context
    Backend-->>GPU: Device & swapchain
    GPU-->>Kick: GPU ready
    deactivate GPU
    
    Kick->>Callback: kore_set_update_callback(fn, data)
    activate Callback
    Callback-->>Kick: Callback registered
    deactivate Callback
    
    Kick->>Start: kore_start()
    activate Start
    Start->>Loop: Enter main loop
    activate Loop
    
    loop Every Frame
        Loop->>Backend: Process input events
        Backend-->>Loop: Input state
        
        Loop->>Backend: Process window events
        Backend-->>Loop: Window events
        
        Loop->>App: Call update callback
        App-->>Loop: Update complete
        
        Loop->>Loop: Render frame (GPU commands)
        Loop->>Backend: Present/Swap
        Backend-->>Loop: Frame displayed
    end
    
    Note over Loop: Continues until kore_stop()
    
    Loop-->>Start: Exit loop
    deactivate Loop
    Start-->>Kick: Return
    deactivate Start
    Kick-->>App: Return 0
    deactivate Kick
```

---

## 3. Main Loop State Diagram

```mermaid
stateDiagram-v2
    [*] --> Initializing: kore_start()
    
    Initializing --> Running: Initialization complete
    
    state Running {
        [*] --> ProcessInput
        
        ProcessInput --> ProcessWindowEvents: Input handled
        ProcessWindowEvents --> CheckCloseRequest: Events processed
        
        CheckCloseRequest --> UpdateCallback: No close requested
        CheckCloseRequest --> Stopping: Close requested
        
        UpdateCallback --> RenderFrame: Update complete
        RenderFrame --> PresentFrame: Rendering complete
        PresentFrame --> ProcessInput: Next frame
        
        note right of ProcessInput
            Keyboard
            Mouse
            Gamepad
            Touch
        end note
        
        note right of ProcessWindowEvents
            Resize
            Close
            Focus
            PPI change
        end note
        
        note right of UpdateCallback
            User logic
            Physics
            AI
            Animation
        end note
        
        note right of RenderFrame
            GPU commands
            Draw calls
            Compute shaders
        end note
    }
    
    Running --> Stopping: kore_stop() called
    Stopping --> Cleanup: Stop requested
    Cleanup --> [*]: Resources freed
```

---

## 4. GPU Rendering Pipeline

```mermaid
flowchart TD
    A[Application Frame Start] --> B[Create Command List]
    B --> C[Begin Command List]
    
    C --> D{Render Pass Setup}
    D --> E[Set Render Target]
    E --> F[Set Viewport]
    D --> G[Set Scissor Rect]
    
    F --> H[Clear Render Target]
    G --> H
    
    H --> I[Set Pipeline State]
    I --> J[Set Vertex Buffers]
    J --> K[Set Index Buffer]
    K --> L[Set Constant Buffers]
    L --> M[Set Textures]
    M --> N[Set Samplers]
    
    N --> O{Draw or Dispatch?}
    O -->|Draw| P[Set Primitive Type]
    O -->|Dispatch| Q[Compute Shader]
    
    P --> R[Draw Indexed/Non-Indexed]
    Q --> R
    
    R --> S{More Draw Calls?}
    S -->|Yes| I
    S -->|No| T
    
    T --> U[End Command List]
    U --> V[Submit to Queue]
    V --> W[Wait for Fence]
    W --> X[Present Swapchain]
    X --> Y[Frame Complete]
```

---

## 5. Class/Struct Hierarchy

```mermaid
classDiagram
    class kore_window {
        +int id
        +int width
        +int height
        +bool fullscreen
        +void* platform_data
        +kore_framebuffer framebuffer
    }
    
    class kore_display {
        +int id
        +int width
        +int height
        +int frequency
        +bool is_primary
    }
    
    class kore_gpu_device {
        +kore_gpu_adapter adapter
        +void* native_device
        +create_command_list()
        +create_pipeline()
        +create_buffer()
        +create_texture()
    }
    
    class kore_gpu_command_list {
        +begin()
        +end()
        +set_pipeline()
        +set_buffers()
        +draw()
        +dispatch()
        +submit()
    }
    
    class kore_gpu_pipeline {
        +vertex_shader
        +fragment_shader
        +input_layout
        +blend_state
        +depth_state
        +rasterizer_state
    }
    
    class kore_gpu_buffer {
        +type
        +size
        +usage
        +map()
        +unmap()
        +upload()
    }
    
    class kore_gpu_texture {
        +width
        +height
        +format
        +usage
        +upload()
        +generate_mipmaps()
    }
    
    class kore_audio_mixer {
        +channels
        +volume
        +play_sound()
        +stop_sound()
        +stream_music()
    }
    
    class kore_keyboard {
        +pressed(key)
        +down(key)
        +get_text()
    }
    
    class kore_mouse {
        +x
        +y
        +moved_x
        +moved_y
        +pressed(button)
        +down(button)
    }
    
    class kore_gamepad {
        +id
        +pressed(button)
        +down(button)
        +axis(axis)
    }
    
    kore_window *-- kore_display : belongs to
    kore_window *-- kore_gpu_device : uses
    kore_gpu_device *-- kore_gpu_command_list : creates
    kore_gpu_device *-- kore_gpu_pipeline : creates
    kore_gpu_device *-- kore_gpu_buffer : creates
    kore_gpu_device *-- kore_gpu_texture : creates
    kore_gpu_command_list --> kore_gpu_pipeline : sets
    kore_gpu_command_list --> kore_gpu_buffer : binds
    kore_gpu_command_list --> kore_gpu_texture : samples
```

---

## 6. Input System Architecture

```mermaid
graph LR
    subgraph "Hardware Layer"
        KB[Keyboard]
        MS[Mouse]
        GP[Gamepad]
        TS[Touch Screen]
        AC[Accelerometer]
        GY[Gyroscope]
    end
    
    subgraph "Platform Input Backends"
        WIN[Windows Input]
        MAC[macOS Input]
        LIN[Linux Input]
        AND[Android Input]
        IOS[iOS Input]
        WEB[Web Input]
    end
    
    subgraph "Kore Input API"
        KEY[kore_keyboard_*]
        MOU[kore_mouse_*]
        PAD[kore_gamepad_*]
        TOU[kore_touch_*]
        SEN[kore_sensor_*]
    end
    
    subgraph "User Application"
        GAME[Game Logic]
    end
    
    KB --> WIN
    KB --> MAC
    KB --> LIN
    
    MS --> WIN
    MS --> MAC
    MS --> LIN
    
    GP --> WIN
    GP --> MAC
    GP --> AND
    GP --> IOS
    
    TS --> AND
    TS --> IOS
    TS --> WEB
    
    AC --> AND
    AC --> IOS
    
    GY --> AND
    GY --> IOS
    
    WIN --> KEY
    WIN --> MOU
    WIN --> PAD
    
    MAC --> KEY
    MAC --> MOU
    MAC --> PAD
    
    LIN --> KEY
    LIN --> MOU
    
    AND --> KEY
    AND --> MOU
    AND --> PAD
    AND --> TOU
    AND --> SEN
    
    IOS --> KEY
    IOS --> MOU
    IOS --> PAD
    IOS --> TOU
    IOS --> SEN
    
    WEB --> KEY
    WEB --> MOU
    WEB --> PAD
    
    KEY --> GAME
    MOU --> GAME
    PAD --> GAME
    TOU --> GAME
    SEN --> GAME
```

---

## 7. Audio System Architecture

```mermaid
graph TB
    subgraph "Audio Sources"
        WAV[.wav Files]
        OGG[.ogg Files]
        STREAM[Music Stream]
        PROC[Procedural Audio]
    end
    
    subgraph "Audio Decoding"
        STB[stb_vorbis]
        WAV_DEC[WAV Decoder]
    end
    
    subgraph "Kore Audio Mixer"
        VOICES[Voice Pool]
        CHANNELS[Channels]
        EFFECTS[Effects]
        MIXER[Mixer Core]
    end
    
    subgraph "Platform Audio Backends"
        WASAPI[WASAPI]
        DS[DirectSound]
        CA[CoreAudio]
        ALSA[ALSA]
        OPENSL[OpenSL ES]
        WEB_AUDIO[Web Audio API]
    end
    
    subgraph "Output"
        SPEAKERS[Speakers]
        HEADPHONES[Headphones]
    end
    
    WAV --> WAV_DEC
    OGG --> STB
    STREAM --> STB
    PROC --> VOICES
    
    WAV_DEC --> VOICES
    STB --> VOICES
    
    VOICES --> CHANNELS
    CHANNELS --> EFFECTS
    EFFECTS --> MIXER
    
    MIXER --> WASAPI
    MIXER --> DS
    MIXER --> CA
    MIXER --> ALSA
    MIXER --> OPENSL
    MIXER --> WEB_AUDIO
    
    WASAPI --> SPEAKERS
    DS --> SPEAKERS
    CA --> SPEAKERS
    ALSA --> SPEAKERS
    OPENSL --> SPEAKERS
    WEB_AUDIO --> SPEAKERS
```

---

## 8. Build System (kmake) Flow

```mermaid
flowchart TD
    A[kfile.js Configuration] --> B[kmake Tool]
    
    B --> C{Parse Configuration}
    C --> D[Identify Source Files]
    C --> E[Identify Dependencies]
    C --> F[Select Backends]
    
    D --> G{Target Platform?}
    E --> G
    F --> G
    
    G -->|Windows| H[Visual Studio Generator]
    G -->|macOS| I[Xcode Generator]
    G -->|Linux| J[Makefile/CMake Generator]
    G -->|Android| K[Gradle Generator]
    G -->|iOS| I
    G -->|Web| L[Emscripten Generator]
    
    H --> M[.vcxproj Files]
    I --> N[.xcodeproj Files]
    J --> O[Makefile / CMakeLists.txt]
    K --> P[build.gradle]
    L --> Q[shell.html + JS]
    
    M --> R[MSBuild]
    N --> S[xcodebuild]
    O --> T[make / cmake --build]
    P --> U[gradle build]
    Q --> V[emcc]
    
    R --> W[Executable/Binary]
    S --> W
    T --> W
    U --> W[APK]
    V --> X[HTML + WASM + JS]
```

---

## 9. Module Dependencies Graph

```mermaid
graph LR
    subgraph "Core Modules"
        SYS[system.h]
        WIN[window.h]
        LOG[log.h]
    end
    
    subgraph "GPU Modules"
        GPU[gpu.h]
        DEV[device.h]
        CMD[command_list.h]
        PIPE[pipeline.h]
        BUF[buffer.h]
        TEX[texture.h]
        SHADER[shader.h]
    end
    
    subgraph "Input Modules"
        KEY[keyboard.h]
        MOU[mouse.h]
        PAD[gamepad.h]
        TOU[touch.h]
    end
    
    subgraph "Audio Modules"
        MIX[mixer.h]
        SND[sound.h]
        STR[stream.h]
    end
    
    subgraph "Utility Modules"
        IO[io.h]
        NET[network.h]
        MATH[math.h]
        THR[threads.h]
        IMG[image.h]
    end
    
    SYS --> WIN
    WIN --> GPU
    WIN --> KEY
    WIN --> MOU
    
    GPU --> DEV
    DEV --> CMD
    DEV --> PIPE
    DEV --> BUF
    DEV --> TEX
    PIPE --> SHADER
    CMD --> BUF
    CMD --> TEX
    
    KEY --> SYS
    MOU --> SYS
    PAD --> SYS
    TOU --> SYS
    
    MIX --> SND
    MIX --> STR
    SND --> IO
    
    IO --> SYS
    NET --> SYS
    THR --> SYS
    
    MATH --> SYS
    IMG --> IO
```

---

## 10. GPU Resource Lifecycle

```mermaid
stateDiagram-v2
    [*] --> NotCreated: Application start
    
    NotCreated --> Creating: create_buffer/texture()
    Creating --> Created: Resource allocated
    
    state Created {
        [*] --> Unmapped
        
        Unmapped --> Mapping: map()
        Mapping --> Mapped: Memory accessible
        
        Mapped --> Uploading: write data
        Uploading --> Mapped: Write complete
        
        Mapped --> Unmapping: unmap()
        Unmapping --> Unmapped: Sync complete
        
        Unmapped --> InUse: bind to command list
        InUse --> Unmapped: Command submitted
    }
    
    Created --> Destroying: destroy()
    Destroying --> Destroyed: Freed
    Destroyed --> [*]
    
    note right of Creating
        GPU memory allocated
        Platform-specific handle created
    end note
    
    note right of Mapped
        CPU can read/write
        GPU access restricted
    end note
    
    note right of InUse
        GPU is using resource
        CPU must wait or use sync
    end note
```

---

## 11. Multi-Window and Multi-Display Support

```mermaid
graph TB
    subgraph "Physical Displays"
        DISP1[Display 1<br/>1920x1080 @ 60Hz]
        DISP2[Display 2<br/>2560x1440 @ 144Hz]
    end
    
    subgraph "Kore Display Objects"
        KDISP1[kore_display #0<br/>Primary]
        KDISP2[kore_display #1<br/>Secondary]
    end
    
    subgraph "Kore Windows"
        WIN1[kore_window #1<br/>Main Game Window]
        WIN2[kore_window #2<br/>Editor Panel]
        WIN3[kore_window #3<br/>Debug Console]
    end
    
    subgraph "Framebuffers"
        FB1[Framebuffer 1<br/>Color + Depth]
        FB2[Framebuffer 2<br/>Color Only]
        FB3[Framebuffer 3<br/>Color + Depth]
    end
    
    DISP1 --> KDISP1
    DISP2 --> KDISP2
    
    KDISP1 --> WIN1
    KDISP1 --> WIN3
    KDISP2 --> WIN2
    
    WIN1 --> FB1
    WIN2 --> FB2
    WIN3 --> FB3
    
    FB1 --> GPU1[GPU Render Target 1]
    FB2 --> GPU2[GPU Render Target 2]
    FB3 --> GPU3[GPU Render Target 3]
    
    note right of WIN1
        Fullscreen possible
        Can span displays
    end note
    
    note right of WIN2
        Secondary display
        Different refresh rate
    end note
```

---

## 12. Thread Safety and Synchronization

```mermaid
sequenceDiagram
    participant Main as Main Thread
    participant Worker as Worker Thread
    participant GPU as GPU Driver
    participant Mutex as Mutex
    participant Fence as GPU Fence
    
    Main->>Mutex: lock()
    Mutex-->>Main: Acquired
    
    Main->>Worker: Start thread
    activate Worker
    
    Worker->>Mutex: lock()
    Mutex-->>Worker: Blocked
    
    Main->>Main: Update shared data
    Main->>Mutex: unlock()
    Mutex-->>Worker: Acquired
    
    Worker->>Worker: Process data
    Worker->>Fence: Create signal
    
    par Parallel Execution
        Main->>GPU: Submit command list
        GPU-->>Main: Queued
        GPU->>Fence: Signal when done
    and
        Worker->>Worker: Continue work
        Worker->>Fence: Wait for signal
    end
    
    GPU->>GPU: Execute commands
    GPU->>Fence: Signal complete
    Fence-->>Worker: Signaled
    
    Worker->>Mutex: unlock()
    Worker-->>Main: Join complete
    deactivate Worker
    
    Main->>Main: Read results
```

