# Análise do Repositório Kore 3

## Resumo Executivo

**Kore 3** é um toolkit de baixo nível para desenvolvimento de motores de jogos multiplataforma. É uma reescrita moderna da biblioteca Kore original, focada em uma nova API gráfica (similar ao WebGPU) e uma linguagem de shader personalizada chamada Kongruent.

---

## Como Funciona

### 1. Arquitetura Base

O Kore 3 segue uma arquitetura em camadas:

```
┌─────────────────────────────────────┐
│     Aplicação do Usuário            │
│     (seu jogo/engine)               │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     API Pública do Kore 3           │
│     (headers em includes/kore3/)    │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     Implementação Core              │
│     (sources/ - código comum)       │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     Backends Específicos            │
│     (backends/ - por plataforma)    │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│     APIs Nativas do Sistema         │
│     (Windows, macOS, Linux, etc.)   │
└─────────────────────────────────────┘
```

### 2. Sistema de Build (kmake)

O Kore usa um meta-build-tool chamado **kmake** que:

1. **Lê o `kfile.js`** - Um arquivo de configuração JavaScript que define:
   - Quais arquivos incluir
   - Quais backends usar
   - Dependências e bibliotecas

2. **Seleciona Backends** baseado em:
   - Plataforma alvo (Windows, Linux, macOS, Android, iOS, Web)
   - API gráfica escolhida (Direct3D 11/12, Vulkan, Metal, OpenGL, WebGPU)
   - API de áudio (WASAPI, DirectSound, CoreAudio, ALSA)

3. **Gera Projetos** para IDEs:
   - Visual Studio (.vcxproj)
   - Xcode (.xcodeproj)
   - Makefiles
   - CMake

**Exemplo de uso:**
```bash
# Build para Windows com Direct3D 12
path/to/Kore/make windows -g direct3d12

# Build para Linux com Vulkan
path/to/Kore/make linux -g vulkan

# Build para macOS com Metal
path/to/Kore/make osx -g metal
```

### 3. Entry Point - `kickstart()`

Todo aplicativo Kore começa com a função `kickstart()`:

```c
int kickstart(int argc, char **argv) {
    // Inicializar sistema
    kore_init("Meu Jogo", 1280, 720, NULL, NULL);
    
    // Configurar GPU
    kore_gpu_init(KORE_GPU_API_DIRECT3D11);
    
    // Registrar callbacks
    kore_set_update_callback(update_callback, NULL);
    
    // Iniciar main loop
    kore_start();
    
    return 0;
}
```

### 4. Main Loop

O Kore gerencia o main loop automaticamente após `kore_start()`:

```
┌──────────────────┐
│  Process Input   │ ← Teclado, mouse, gamepad, touch
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Window Events    │ ← Resize, close, focus, PPI
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Update Callback  │ ← Sua lógica (física, IA, etc.)
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Render Frame     │ → GPU commands
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Present/Swap     │ → Mostra na tela
└────────┬─────────┘
         ↓
    (repete até kore_stop())
```

### 5. Abstração de GPU

O Kore 3 fornece uma API unificada para múltiplas APIs gráficas:

```c
// A MESMA API funciona para todas as plataformas
struct kore_gpu_command_list *list = kore_gpu_command_list_create();
kore_gpu_command_list_begin(list);

// Set pipeline, buffers, textures
kore_gpu_command_list_set_pipeline(list, pipeline);
kore_gpu_command_list_draw(list, vertex_count, instance_count);

kore_gpu_command_list_end(list);
kore_gpu_command_list_submit(list);
```

Internamente, o Kore traduz isso para:
- **Direct3D 11/12** no Windows
- **Metal** no macOS/iOS
- **Vulkan** no Linux/Android
- **OpenGL** como fallback
- **WebGPU** na Web

### 6. Estrutura de Diretórios

```
/workspace/
├── includes/kore3/          # Headers públicos da API
│   ├── gpu/                 # API gráfica
│   ├── input/               # Input (teclado, mouse, gamepad)
│   ├── mixer/               # Áudio
│   ├── io/                  # Sistema de arquivos
│   ├── math/                # Matemática (vetores, matrizes)
│   ├── threads/             # Threading
│   └── ...
│
├── sources/                 # Implementação core (comum a todas plataformas)
│   ├── gpu/
│   ├── audio/
│   ├── input/
│   ├── io/
│   ├── math/
│   └── libs/                # Bibliotecas third-party (stb_image, lz4, etc.)
│
├── backends/                # Implementações específicas por plataforma
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
├── shaders/                 # Shaders Kongruent (.kong)
├── tests/                   # Testes de exemplo
└── kfile.js                 # Configuração do build
```

### 7. Principais Módulos

#### Sistema/Janelas (`system.h`, `window.h`)
- Criação e gerenciamento de janelas
- Fullscreen, resize, move
- Callbacks de eventos
- Multi-window support

#### GPU (`gpu/*.h`)
- Device, command lists, pipelines
- Buffers (vertex, index, constant, storage)
- Textures e samplers
- Ray tracing (experimental)

#### Áudio (`mixer/*.h`)
- Mixer com múltiplos canais
- Carregamento de sons (.wav, .ogg via stb_vorbis)
- Streaming de música
- Controle de volume e pan

#### Input (`input/*.h`)
- Teclado (pressed/down states)
- Mouse (posição, movimento, botões)
- Gamepad (até 8 controllers)
- Touch surface
- Sensores de movimento (accelerometer, gyroscope)

#### I/O (`io/*.h`)
- Leitura/escrita de arquivos
- Save path abstraction
- Empacotamento de assets

#### Rede (`network/*.h`)
- HTTP requests (GET/POST)
- Sockets TCP/UDP

#### Matemática (`math/*.h`)
- Vetores (vec2, vec3, vec4)
- Matrizes (mat4)
- Quatérnions
- Random number generation

#### Threads (`threads/*.h`)
- Thread creation/join
- Mutex, semaphore, event
- Atomic operations

### 8. Third-Party Libraries Incluídas

- **stb_image.h** - Carregamento de imagens (PNG, JPG, etc.)
- **stb_vorbis.h** - Decodificação de áudio Ogg Vorbis
- **lz4** - Compressão rápida
- **offalloc** - Alocador de memória customizado
- **GLEW** - OpenGL extension loader (para backend OpenGL)

### 9. Plataformas Suportadas

| Plataforma | Sistema | GPU | Audio |
|------------|---------|-----|-------|
| Windows | Win32/UWP | D3D11, D3D12, Vulkan, OpenGL | WASAPI, DirectSound |
| macOS | Cocoa | Metal, OpenGL | CoreAudio |
| Linux | X11/Wayland | Vulkan, OpenGL | ALSA |
| Android | Android SDK | Vulkan, OpenGL ES | OpenSL ES |
| iOS | UIKit | Metal, OpenGL ES | CoreAudio |
| Web | Emscripten/WASM | WebGPU, WebGL | Web Audio API |

### 10. VR Support

Suporte opcional para:
- **Oculus Rift** (via LibOVR)
- **SteamVR** (OpenVR)
- **HoloLens** (Windows Mixed Reality)

---

## Diferenças do Kore 3 vs Versões Anteriores

| Feature | Kore v1 | Kore v2 | Kore 3 |
|---------|---------|---------|--------|
| Linguagem | C++ | C++ | C |
| API Gráfica | OpenGL/DirectX | OpenGL/DirectX/Vulkan | Nova API (WebGPU-like) |
| Shaders | GLSL/HLSL | GLSL/HLSL | Kongruent |
| Status | Legacy | Estável | Experimental |

---

## Casos de Uso Típicos

1. **Motores de Jogo**: Armored Pixel, Krom, Kha usam Kore
2. **Ferramentas Gráficas**: Editores, visualizadores 3D
3. **Aplicações Multiplataforma**: Apps que precisam rodar em desktop/mobile/web
4. **Prototipagem Rápida**: Graças à API simples e build system fácil

---

## Exemplo Mínimo Completo

```c
#include <kore3/system.h>
#include <kore3/window.h>
#include <kore3/gpu/gpu.h>
#include <kore3/input/keyboard.h>
#include <kore3/log.h>

void update(void *data) {
    // Checar input
    if (kore_keyboard_pressed(KORE_KEY_ESCAPE)) {
        kore_stop();
        return;
    }
    
    // Lógica do jogo aqui
    
    // Renderização básica
    // (configurar pipeline, draw calls, etc.)
}

int kickstart(int argc, char **argv) {
    kore_log(KORE_LOG_LEVEL_INFO, "Iniciando aplicativo...");
    
    // Inicializar com janela 1280x720
    kore_init("Meu App", 1280, 720, NULL, NULL);
    
    // Inicializar GPU (auto-seleciona melhor API)
    kore_gpu_init(KORE_GPU_API_DEFAULT);
    
    // Configurar callback de update
    kore_set_update_callback(update, NULL);
    
    // Iniciar main loop
    kore_start();
    
    kore_log(KORE_LOG_LEVEL_INFO, "Aplicativo fechado.");
    return 0;
}
```

---

## Pontos Fortes

✅ **Verdadeiramente multiplataforma** - Mesmo código para todas plataformas  
✅ **API unificada de GPU** - Escreva uma vez, rode em D3D/Vulkan/Metal  
✅ **Build system inteligente** - kmake gera projetos automaticamente  
✅ **Baixo nível** - Controle total sobre GPU e recursos  
✅ **Leve** - Sem overhead desnecessário  
✅ **C puro** - Fácil de integrar com qualquer linguagem  

## Pontos de Atenção

⚠️ **Kore 3 é experimental** - API pode mudar  
⚠️ **Documentação limitada** - Poucos exemplos oficiais  
⚠️ **Curva de aprendizado** - Requer conhecimento de gráficos  
⚠️ **Poucas features high-level** - Não é um engine completo  

---

## Recursos Adicionais

- **Repositório Oficial**: https://github.com/Kode/Kore
- **Amostras**: https://github.com/Kode/Kore-Samples
- **Linguagem de Shader**: https://github.com/Kode/Kongruent
- **Kore v2 (estável)**: https://github.com/Kode/Kore/tree/v2
- **Kha Framework**: Framework high-level baseado em Kore
