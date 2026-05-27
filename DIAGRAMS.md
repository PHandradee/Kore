# Diagramas de Arquitetura do Kore 3

## Visão Geral da Arquitetura

```mermaid
graph TB
    subgraph "Aplicação do Usuário"
        App[kickstart / main]
        Update[Update Callback]
        Render[Renderização]
        Logic[Lógica do Jogo]
    end
    
    subgraph "Kore 3 Core"
        System[Sistema / Window Management]
        GPU[GPU API Abstraction]
        Audio[Audio System]
        Input[Input System]
        IO[File I/O]
        Network[Network]
        Math[Math Library]
        Threads[Threading]
    end
    
    subgraph "Backends - Sistema"
        Win[Windows]
        Mac[macOS / iOS]
        Lin[Linux / Android]
        Web[Emscripten / WASM]
    end
    
    subgraph "Backends - GPU"
        D3D11[Direct3D 11]
        D3D12[Direct3D 12]
        VK[Vulkan]
        MTL[Metal]
        GL[OpenGL / WebGL]
        WGPU[WebGPU]
    end
    
    subgraph "Backends - Audio"
        WASAPI[WASAPI]
        DS[DirectSound]
        CoreAudio[CoreAudio]
        ALSA[ALSA]
    end
    
    App --> System
    App --> GPU
    App --> Audio
    App --> Input
    App --> IO
    App --> Network
    App --> Math
    App --> Threads
    
    Update --> Logic
    Logic --> Render
    Render --> GPU
    
    System --> Win
    System --> Mac
    System --> Lin
    System --> Web
    
    GPU --> D3D11
    GPU --> D3D12
    GPU --> VK
    GPU --> MTL
    GPU --> GL
    GPU --> WGPU
    
    Audio --> WASAPI
    Audio --> DS
    Audio --> CoreAudio
    Audio --> ALSA
    
    style App fill:#e1f5fe
    style System fill:#fff3e0
    style GPU fill:#f3e5f5
    style Audio fill:#e8f5e9
```

---

## Fluxo de Inicialização

```mermaid
sequenceDiagram
    participant User as Aplicação
    participant Kore as Kore Core
    participant Win as Window System
    participant GPU as GPU Backend
    participant Audio as Audio Backend
    participant Input as Input Backend
    
    User->>Kore: kore_init(name, width, height, win_params, frame_params)
    activate Kore
    
    Kore->>Win: Criar janela inicial
    activate Win
    Win-->>Kore: Window ID (0)
    deactivate Win
    
    Kore->>GPU: kore_gpu_init(api_type)
    activate GPU
    GPU-->>Kore: Device criado
    deactivate GPU
    
    Kore->>Audio: Inicializar mixer
    activate Audio
    Audio-->>Kore: Mixer pronto
    deactivate Audio
    
    Kore->>Input: Inicializar dispositivos
    activate Input
    Input-->>Kore: Dispositivos prontos
    deactivate Input
    
    Kore-->>User: Sistema inicializado
    deactivate Kore
    
    User->>Kore: kore_set_update_callback(callback, data)
    Kore-->>User: Callback registrado
    
    User->>Kore: kore_start()
    activate Kore
    Note over Kore: Main Loop inicia
    Kore-->>User: Controle transferido
    deactivate Kore
```

---

## Main Loop

```mermaid
stateDiagram-v2
    [*] --> Iniciando : kore_init()
    
    state MainLoop {
        [*] --> ProcessInput
        
        ProcessInput --> ProcessWindowEvents : Checar eventos
        ProcessWindowEvents --> UpdateCallback : Chamar update
        UpdateCallback --> RenderFrame : Renderizar
        RenderFrame --> SwapBuffers : Apresentar frame
        SwapBuffers --> ProcessInput : Próximo frame
        
        note right of ProcessInput
            - Keyboard
            - Mouse
            - Gamepad
            - Touch
        end note
        
        note right of ProcessWindowEvents
            - Resize
            - Close
            - Focus
            - PPI change
        end note
        
        note right of UpdateCallback
            - Lógica do jogo
            - Física
            - IA
            - Animação
        end note
        
        note right of RenderFrame
            - Command lists
            - Draw calls
            - Compute shaders
        end note
    }
    
    Iniciando --> MainLoop : kore_start()
    MainLoop --> Finalizando : kore_stop()
    Finalizando --> [*] : Cleanup
    
    MainLoop --> Shutdown : Evento de fechar
    Shutdown --> [*]
```

---

## Pipeline de Renderização GPU

```mermaid
graph LR
    subgraph "Preparação"
        A[Criar Command List] --> B[Begin Command List]
        B --> C[Set Viewport]
        C --> D[Set Scissor]
        D --> E[Clear Render Target]
    end
    
    subgraph "Pipeline Setup"
        E --> F[Set Pipeline State]
        F --> G[Set Vertex Buffer]
        G --> H[Set Index Buffer]
        H --> I[Set Constant Buffers]
        I --> J[Set Textures]
        J --> K[Set Samplers]
    end
    
    subgraph "Draw/Dispatch"
        K --> L{Tipo?}
        L -->|Draw| M[Draw Indexed/Instanced]
        L -->|Dispatch| N[Dispatch Compute]
        L -->|Ray Trace| O[Dispatch Rays]
    end
    
    subgraph "Finalização"
        M --> P[End Command List]
        N --> P
        O --> P
        P --> Q[Submit to Queue]
        Q --> R[Signal Fence]
        R --> S[Present]
    end
    
    style A fill:#e3f2fd
    style F fill:#fff3e0
    style L fill:#f3e5f5
    style Q fill:#e8f5e9
```

---

## Hierarquia de Classes/Structs

```mermaid
classDiagram
    class kore_window_parameters {
        +const char* title
        +int x, y, width, height
        +int display_index
        +bool visible
        +int window_features
        +kore_window_mode mode
    }
    
    class kore_framebuffer_parameters {
        +int frequency
        +bool vertical_sync
        +int color_bits
        +int depth_bits
        +int stencil_bits
        +int samples_per_pixel
    }
    
    class kore_gpu_device {
        +Criar command lists
        +Criar pipelines
        +Criar buffers
        +Criar texturas
        +Criar samplers
    }
    
    class kore_gpu_command_list {
        +begin()
        +end()
        +set_pipeline()
        +set_buffers()
        +draw()
        +dispatch()
    }
    
    class kore_gpu_buffer {
        +lock()
        +unlock()
        +destroy()
    }
    
    class kore_gpu_texture {
        +upload()
        +generate_mipmaps()
        +destroy()
    }
    
    class kore_sound {
        +play(volume, pan)
        +stop()
        +destroy()
    }
    
    class kore_file_reader {
        +open(path)
        +read(buffer, size)
        +size()
        +close()
    }
    
    kore_window_parameters --o kore_init
    kore_framebuffer_parameters --o kore_init
    kore_gpu_device --* kore_gpu_command_list
    kore_gpu_device --* kore_gpu_buffer
    kore_gpu_device --* kore_gpu_texture
```

---

## Sistema de Input

```mermaid
graph TB
    subgraph "Hardware"
        KB[Teclado]
        MS[Mouse]
        GP[Gamepad]
        TS[Touch Screen]
        PEN[Pen/Stylus]
        ACC[Acelerômetro]
        GYR[Giroscópio]
    end
    
    subgraph "Backend de Input"
        WinInput[Windows Input]
        MacInput[macOS Input]
        LinuxInput[Linux Input]
        AndroidInput[Android Input]
        WebInput[Web Input]
    end
    
    subgraph "Kore Input Layer"
        Keyboard[Keyboard API]
        Mouse[Mouse API]
        Gamepad[Gamepad API]
        Surface[Touch Surface API]
        Motion[Motion API]
    end
    
    subgraph "Aplicação"
        GameLogic[Lógica do Jogo]
        UI[Sistema de UI]
        Camera[Controle de Câmera]
    end
    
    KB --> WinInput
    KB --> MacInput
    KB --> LinuxInput
    KB --> WebInput
    
    MS --> WinInput
    MS --> MacInput
    MS --> LinuxInput
    MS --> WebInput
    
    GP --> WinInput
    GP --> MacInput
    GP --> LinuxInput
    GP --> AndroidInput
    
    TS --> AndroidInput
    TS --> WebInput
    
    WinInput --> Keyboard
    WinInput --> Mouse
    WinInput --> Gamepad
    
    MacInput --> Keyboard
    MacInput --> Mouse
    MacInput --> Gamepad
    
    LinuxInput --> Keyboard
    LinuxInput --> Mouse
    LinuxInput --> Gamepad
    
    AndroidInput --> Gamepad
    AndroidInput --> Surface
    AndroidInput --> Motion
    
    WebInput --> Keyboard
    WebInput --> Mouse
    WebInput --> Gamepad
    WebInput --> Surface
    
    Keyboard --> GameLogic
    Keyboard --> UI
    
    Mouse --> GameLogic
    Mouse --> UI
    Mouse --> Camera
    
    Gamepad --> GameLogic
    Gamepad --> UI
    
    Surface --> GameLogic
    Surface --> UI
    
    Motion --> GameLogic
```

---

## Sistema de Áudio

```mermaid
graph TB
    subgraph "Fontes de Áudio"
        SoundFiles[Arquivos .wav/.ogg]
        Streams[Música/.ogg streams]
        Procedural[Áudio Procedural]
    end
    
    subgraph "Kore Audio System"
        Mixer[Mixer Principal]
        Voices[Canais/Voices]
        Effects[Efeitos (pan, volume)]
    end
    
    subgraph "Backends"
        WASAPI[WASAPI - Windows]
        DirectSound[DirectSound - Windows]
        CoreAudio[CoreAudio - macOS/iOS]
        ALSA[ALSA - Linux]
        OpenSL[OpenSL ES - Android]
        WebAudio[Web Audio API - Web]
    end
    
    subgraph "Output"
        Speakers[Alto-falantes]
        Headphones[Fones de ouvido]
        HDMI[Áudio HDMI]
    end
    
    SoundFiles --> Mixer
    Streams --> Mixer
    Procedural --> Mixer
    
    Mixer --> Voices
    Voices --> Effects
    
    Effects --> WASAPI
    Effects --> DirectSound
    Effects --> CoreAudio
    Effects --> ALSA
    Effects --> OpenSL
    Effects --> WebAudio
    
    WASAPI --> Speakers
    DirectSound --> Speakers
    CoreAudio --> Speakers
    ALSA --> Speakers
    OpenSL --> Speakers
    WebAudio --> Speakers
    
    WASAPI --> Headphones
    CoreAudio --> Headphones
    
    WASAPI --> HDMI
```

---

## Build System (kmake)

```mermaid
graph TB
    subgraph "Configuração"
        Platform[Plataforma Alvo]
        Graphics[API Gráfica]
        Audio[API de Áudio]
        Options[Opções Extras]
    end
    
    subgraph "kmake"
        ParseKFile[Parse kfile.js]
        ResolveFiles[Resolver arquivos]
        SelectBackends[Selecionar backends]
        GenerateProject[Gerar projeto IDE]
    end
    
    subgraph "Projetos Gerados"
        VS[Visual Studio .vcxproj]
        Xcode[Xcode .xcodeproj]
        Make[Makefile]
        CMake[CMakeLists.txt]
    end
    
    subgraph "Compilação"
        Compiler[Compilador C/C++]
        Linker[Linker]
        ShaderComp[Compiler de Shaders]
    end
    
    subgraph "Binário Final"
        Executable[.exe / .app / .apk]
        Assets[Assets empacotados]
        Libs[Bibliotecas necessárias]
    end
    
    Platform --> ParseKFile
    Graphics --> ParseKFile
    Audio --> ParseKFile
    Options --> ParseKFile
    
    ParseKFile --> ResolveFiles
    ResolveFiles --> SelectBackends
    SelectBackends --> GenerateProject
    
    GenerateProject --> VS
    GenerateProject --> Xcode
    GenerateProject --> Make
    GenerateProject --> CMake
    
    VS --> Compiler
    Xcode --> Compiler
    Make --> Compiler
    CMake --> Compiler
    
    Compiler --> Linker
    ShaderComp --> Linker
    
    Linker --> Executable
    Executable --> Assets
    Executable --> Libs
    
    style ParseKFile fill:#fff3e0
    style GenerateProject fill:#e3f2fd
    style Executable fill:#e8f5e9
```

---

## Dependências entre Módulos

```mermaid
graph TD
    Global[global.h]
    System[system.h]
    Window[window.h]
    Display[display.h]
    Log[log.h]
    
    GPU[gpu/gpu.h]
    Device[gpu/device.h]
    CmdList[gpu/commandlist.h]
    Buffer[gpu/buffer.h]
    Texture[gpu/texture.h]
    Sampler[gpu/sampler.h]
    Pipeline[gpu/pipeline.h]
    RayTracing[gpu/raytracing.h]
    
    Audio[mixer/mixer.h]
    Sound[mixer/sound.h]
    SoundStream[mixer/soundstream.h]
    
    Keyboard[input/keyboard.h]
    Mouse[input/mouse.h]
    Gamepad[input/gamepad.h]
    Surface[input/surface.h]
    
    FileReader[io/filereader.h]
    FileWriter[io/filewriter.h]
    
    HTTP[network/http.h]
    Socket[network/socket.h]
    
    Vector[math/vector.h]
    Matrix[math/matrix.h]
    Quaternion[math/quaternion.h]
    Random[math/random.h]
    
    Thread[threads/thread.h]
    Mutex[threads/mutex.h]
    Semaphore[threads/semaphore.h]
    
    Image[image.h]
    Color[color.h]
    Video[video.h]
    
    VR[vr/vr.h]
    
    Global --> System
    Global --> Window
    Global --> Display
    Global --> Log
    Global --> GPU
    
    System --> Window
    System --> Display
    System --> Log
    
    GPU --> Device
    Device --> CmdList
    Device --> Buffer
    Device --> Texture
    Device --> Sampler
    Device --> Pipeline
    Pipeline --> RayTracing
    
    Audio --> Sound
    Audio --> SoundStream
    
    Image --> Color
    Video --> Image
    
    style Global fill:#ffcdd2
    style GPU fill:#bbdefb
    style Audio fill:#c8e6c9
    style Image fill:#fff9c4
```

---

## Ciclo de Vida de Recursos GPU

```mermaid
stateDiagram-v2
    [*] --> Alocado : create()
    
    state "Em Uso" como EmUso {
        Locked --> Mapped : lock()/map()
        Mapped --> Modificado : Escrever dados
        Modificado --> Unmapped : unlock()/unmap()
        Unmapped --> Ready : flush()
    }
    
    Alocado --> EmUso
    EmUso --> Ready
    
    Ready --> Bound : bind to pipeline
    Bound --> Drawing : draw/dispatch
    Drawing --> Ready : complete
    
    Ready --> Alocado : destroy()
    Bound --> Alocado : destroy()
    EmUso --> Alocado : destroy()
    
    note right of Alocado
        Memória alocada
        na GPU
    end note
    
    note right of Locked
        CPU espera GPU
        liberar recurso
    end note
    
    note right of Bound
        Recurso ativo
        no command buffer
    end note
    
    note right of Drawing
        GPU processando
        comandos
    end note
```

---

## Multi-Janela e Multi-Display

```mermaid
graph TB
    subgraph "Sistema de Displays"
        D0[Display 0<br/>1920x1080 @ 60Hz]
        D1[Display 1<br/>2560x1440 @ 144Hz]
        D2[Display 2<br/>3840x2160 @ 30Hz]
    end
    
    subgraph "Janelas"
        W0[Janela 0<br/>Principal]
        W1[Janela 1<br/>Secundária]
        W2[Janela 2<br/>Overlay]
    end
    
    subgraph "Framebuffers"
        FB0[Framebuffer 0<br/>Triple buffered]
        FB1[Framebuffer 1<br/>Double buffered]
        FB2[Framebuffer 2<br/>Single buffered]
    end
    
    subgraph "Eventos"
        E0[Resize Event]
        E1[Move Event]
        E2[PPI Changed]
        E3[Focus Changed]
    end
    
    D0 --> W0
    D1 --> W1
    D2 --> W2
    
    W0 --> FB0
    W1 --> FB1
    W2 --> FB2
    
    W0 --> E0
    W0 --> E1
    W0 --> E2
    W0 --> E3
    
    W1 --> E0
    W1 --> E1
    W1 --> E2
    
    W2 --> E0
    W2 --> E3
    
    style D0 fill:#e3f2fd
    style D1 fill:#e3f2fd
    style D2 fill:#e3f2fd
    style W0 fill:#fff3e0
    style W1 fill:#fff3e0
    style W2 fill:#fff3e0
```

---

## Thread Safety e Sincronização

```mermaid
sequenceDiagram
    participant Main as Main Thread
    participant Render as Render Thread
    participant Worker as Worker Thread
    participant GPU as GPU Driver
    participant Mutex as Mutex/Semaphore
    
    Main->>Mutex: lock()
    Main->>Main: Acessar recurso compartilhado
    Main->>Mutex: unlock()
    
    par Frame Processing
        Main->>Worker: Dispatch task
        Worker->>Worker: Processar dados
        Worker->>Main: Signal completion
    and
        Main->>Render: Submit command list
        Render->>GPU: Execute commands
        GPU-->>Render: Fence signal
    end
    
    Main->>Mutex: wait()
    Worker->>Main: Task completa
    
    Render->>Main: Frame completo
    Main->>Main: Swap buffers
    
    Note over Main,GPU: Fences garantem que<br/>a GPU terminou antes<br/>de reutilizar recursos
```
