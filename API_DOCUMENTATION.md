# Documentação da API Kore 3

## Visão Geral

Kore 3 é um toolkit de baixo nível para desenvolvimento de motores de jogos multiplataforma, fornecendo abstrações para:
- Gerenciamento de janelas e sistema
- Renderização GPU (múltiplas APIs)
- Áudio
- Input (teclado, mouse, gamepad, touch)
- Sistema de arquivos
- Rede
- Matemática (vetores, matrizes, quatérnions)
- Threads e sincronização

---

## Índice

1. [Inicialização do Sistema](#inicialização-do-sistema)
2. [Gerenciamento de Janelas](#gerenciamento-de-janelas)
3. [API GPU](#api-gpu)
4. [Áudio](#áudio)
5. [Input](#input)
6. [Sistema de Arquivos](#sistema-de-arquivos)
7. [Rede](#rede)
8. [Matemática](#matemática)
9. [Threads](#threads)
10. [Utilitários](#utilitários)

---

## Inicialização do Sistema

### `kore_init()`
```c
int kore_init(const char *name, int width, int height, 
              struct kore_window_parameters *win, 
              struct kore_framebuffer_parameters *frame);
```
Inicializa o aplicativo Kore e cria uma janela inicial.

**Parâmetros:**
- `name`: Nome do aplicativo
- `width`: Largura inicial da janela
- `height`: Altura inicial da janela
- `win`: Parâmetros da janela (pode ser NULL)
- `frame`: Parâmetros do framebuffer (pode ser NULL)

**Retorna:** ID da janela inicial (normalmente 0)

---

### `kore_start()`
```c
void kore_start(void);
```
Inicia o main loop do Kore. Deve ser chamado após configurar os callbacks.

---

### `kore_stop()`
```c
void kore_stop(void);
```
Para o main loop e retorna ao chamador de `kore_start()`.

---

### Callbacks do Sistema

```c
void kore_set_update_callback(void (*callback)(void *), void *data);
void kore_set_foreground_callback(void (*callback)(void *), void *data);
void kore_set_resume_callback(void (*callback)(void *), void *data);
void kore_set_pause_callback(void (*callback)(void *), void *data);
void kore_set_background_callback(void (*callback)(void *), void *data);
void kore_set_shutdown_callback(void (*callback)(void *), void *data);
```

---

## Gerenciamento de Janelas

### Estruturas

```c
typedef struct kore_window_parameters {
    const char *title;
    int x, y, width, height;
    int display_index;
    bool visible;
    int window_features;  // Combinação OR de KORE_WINDOW_FEATURE_*
    kore_window_mode mode; // KORE_WINDOW_MODE_WINDOW/FULLSCREEN/EXCLUSIVE_FULLSCREEN
} kore_window_parameters;

typedef struct kore_framebuffer_parameters {
    int frequency;
    bool vertical_sync;
    int color_bits;
    int depth_bits;
    int stencil_bits;
    int samples_per_pixel;
} kore_framebuffer_parameters;
```

### Funções

```c
int kore_window_create(kore_window_parameters *win, kore_framebuffer_parameters *frame);
void kore_window_destroy(int window);
void kore_window_resize(int window, int width, int height);
void kore_window_move(int window, int x, int y);
void kore_window_change_mode(int window, kore_window_mode mode);
void kore_window_change_features(int window, int features);
void kore_window_set_title(int window, const char *title);

// Queries
int kore_window_width(int window);
int kore_window_height(int window);
int kore_window_x(int window);
int kore_window_y(int window);
kore_window_mode kore_window_get_mode(int window);

// Callbacks
void kore_window_set_resize_callback(int window, void (*callback)(int x, int y, void *data), void *data);
void kore_window_set_close_callback(int window, bool (*callback)(void *data), void *data);
```

---

## API GPU

### Inicialização

```c
typedef enum kore_gpu_api {
    KORE_GPU_API_DIRECT3D11,
    KORE_GPU_API_DIRECT3D12,
    KORE_GPU_API_VULKAN,
    KORE_GPU_API_METAL,
    KORE_GPU_API_WEBGPU,
    KORE_GPU_API_OPENGL,
    KORE_GPU_API_KOMPJUTA,
    KORE_GPU_API_CONSOLE,
} kore_gpu_api;

void kore_gpu_init(kore_gpu_api api);
```

### Device

```c
#include <kore3/gpu/device.h>

struct kore_gpu_device *kore_gpu_device_create(void);
void kore_gpu_device_destroy(struct kore_gpu_device *device);
```

### Command List

```c
#include <kore3/gpu/commandlist.h>

struct kore_gpu_command_list *kore_gpu_command_list_create(void);
void kore_gpu_command_list_begin(struct kore_gpu_command_list *list);
void kore_gpu_command_list_end(struct kore_gpu_command_list *list);
void kore_gpu_command_list_submit(struct kore_gpu_command_list *list);
```

### Pipeline

```c
#include <kore3/gpu/pipeline.h>

struct kore_gpu_pipeline {
    // Configuração de vertex/fragment shaders
    // Blend states
    // Depth/stencil states
    // Rasterizer states
};
```

### Buffers

```c
#include <kore3/gpu/buffer.h>

typedef enum {
    KORE_GPU_BUFFER_TYPE_VERTEX,
    KORE_GPU_BUFFER_TYPE_INDEX,
    KORE_GPU_BUFFER_TYPE_CONSTANT,
    KORE_GPU_BUFFER_TYPE_STORAGE
} kore_gpu_buffer_type;

struct kore_gpu_buffer *kore_gpu_buffer_create(
    kore_gpu_buffer_type type,
    size_t size,
    const void *initial_data
);
void kore_gpu_buffer_destroy(struct kore_gpu_buffer *buffer);
void kore_gpu_buffer_lock(struct kore_gpu_buffer *buffer, void **data);
void kore_gpu_buffer_unlock(struct kore_gpu_buffer *buffer);
```

### Texturas

```c
#include <kore3/gpu/texture.h>

typedef enum {
    KORE_GPU_TEXTURE_FORMAT_R8G8B8A8,
    KORE_GPU_TEXTURE_FORMAT_R16G16B16A16_FLOAT,
    KORE_GPU_TEXTURE_FORMAT_R32G32B32A32_FLOAT,
    KORE_GPU_TEXTURE_FORMAT_D32_FLOAT,
    KORE_GPU_TEXTURE_FORMAT_BC1,
    KORE_GPU_TEXTURE_FORMAT_BC3,
    // ... mais formatos
} kore_gpu_texture_format;

struct kore_gpu_texture *kore_gpu_texture_create(
    int width,
    int height,
    int depth,
    kore_gpu_texture_format format,
    int mip_levels,
    const void *initial_data
);
void kore_gpu_texture_destroy(struct kore_gpu_texture *texture);
```

### Samplers

```c
#include <kore3/gpu/sampler.h>

typedef enum {
    KORE_GPU_FILTER_POINT,
    KORE_GPU_FILTER_LINEAR,
    KORE_GPU_FILTER_ANISOTROPIC
} kore_gpu_filter;

typedef enum {
    KORE_GPU_ADDRESS_WRAP,
    KORE_GPU_ADDRESS_CLAMP,
    KORE_GPU_ADDRESS_MIRROR
} kore_gpu_address_mode;

struct kore_gpu_sampler *kore_gpu_sampler_create(
    kore_gpu_filter min_filter,
    kore_gpu_filter mag_filter,
    kore_gpu_filter mip_filter,
    kore_gpu_address_mode address_u,
    kore_gpu_address_mode address_v,
    kore_gpu_address_mode address_w
);
```

### Ray Tracing (Experimental)

```c
#include <kore3/gpu/raytracing.h>

struct kore_gpu_acceleration_structure *kore_gpu_as_create(void);
struct kore_gpu_raytracing_pipeline *kore_gpu_rt_pipeline_create(void);
void kore_gpu_dispatch_rays(struct kore_gpu_command_list *list, 
                            struct kore_gpu_raytracing_pipeline *pipeline);
```

---

## Áudio

### Mixer

```c
#include <kore3/mixer/mixer.h>

void kore_mixer_init(int sample_rate);
void kore_mixer_update(void);
float *kore_mixer_buffer(void);
int kore_mixer_buffer_samples(void);
```

### Sons

```c
#include <kore3/mixer/sound.h>

struct kore_sound *kore_sound_load(const char *filename);
void kore_sound_play(struct kore_sound *sound, float volume, float pan);
void kore_sound_destroy(struct kore_sound *sound);
```

### Sound Streams

```c
#include <kore3/mixer/soundstream.h>

struct kore_sound_stream *kore_sound_stream_open(const char *filename);
void kore_sound_stream_play(struct kore_sound_stream *stream, float volume, float pan);
void kore_sound_stream_close(struct kore_sound_stream *stream);
```

---

## Input

### Teclado

```c
#include <kore3/input/keyboard.h>

typedef enum {
    KORE_KEY_A, KORE_KEY_B, KORE_KEY_C, /* ... */,
    KORE_KEY_0, KORE_KEY_1, /* ... */,
    KORE_KEY_F1, KORE_KEY_F2, /* ... */,
    KORE_KEY_RETURN, KORE_KEY_ESCAPE, KORE_KEY_SPACE,
    // ... mais teclas
} kore_key_code;

bool kore_keyboard_pressed(kore_key_code key);
bool kore_keyboard_down(kore_key_code key);
```

### Mouse

```c
#include <kore3/input/mouse.h>

bool kore_mouse_pressed(int button);  // 0 = left, 1 = right, 2 = middle
bool kore_mouse_down(int button);
int kore_mouse_x(void);
int kore_mouse_y(void);
int kore_mouse_movement_x(void);
int kore_mouse_movement_y(void);
void kore_mouse_lock(int window);
void kore_mouse_unlock(void);
bool kore_mouse_can_lock(void);
```

### Gamepad

```c
#include <kore3/input/gamepad.h>

#define KORE_GAMEPAD_MAX_COUNT 8

typedef enum {
    KORE_GAMEPAD_BUTTON_A,
    KORE_GAMEPAD_BUTTON_B,
    KORE_GAMEPAD_BUTTON_X,
    KORE_GAMEPAD_BUTTON_Y,
    KORE_GAMEPAD_BUTTON_LEFT_SHOULDER,
    KORE_GAMEPAD_BUTTON_RIGHT_SHOULDER,
    KORE_GAMEPAD_BUTTON_LEFT_TRIGGER,
    KORE_GAMEPAD_BUTTON_RIGHT_TRIGGER,
    KORE_GAMEPAD_BUTTON_BACK,
    KORE_GAMEPAD_BUTTON_START,
    KORE_GAMEPAD_BUTTON_LEFT_STICK,
    KORE_GAMEPAD_BUTTON_RIGHT_STICK,
    KORE_GAMEPAD_BUTTON_UP,
    KORE_GAMEPAD_BUTTON_DOWN,
    KORE_GAMEPAD_BUTTON_LEFT,
    KORE_GAMEPAD_BUTTON_RIGHT,
    KORE_GAMEPAD_BUTTON_GUIDE
} kore_gamepad_button;

bool kore_gamepad_present(int gamepad);
bool kore_gamepad_pressed(int gamepad, kore_gamepad_button button);
bool kore_gamepad_down(int gamepad, kore_gamepad_button button);
float kore_gamepad_axis(int gamepad, int axis);  // 0 = left X, 1 = left Y, 2 = right X, 3 = right Y
const char *kore_gamepad_name(int gamepad);
```

### Touch

```c
#include <kore3/input/surface.h>

typedef struct {
    int x, y;
    int id;
    bool started;
} kore_surface_touch;

kore_surface_touch *kore_surface_touches(int *count);
```

---

## Sistema de Arquivos

### Leitura

```c
#include <kore3/io/filereader.h>

struct kore_file_reader *kore_file_reader_open(const char *path);
size_t kore_file_reader_size(struct kore_file_reader *file);
size_t kore_file_reader_read(struct kore_file_reader *file, void *buffer, size_t size);
void kore_file_reader_close(struct kore_file_reader *file);
bool kore_file_reader_eof(struct kore_file_reader *file);
```

### Escrita

```c
#include <kore3/io/filewriter.h>

struct kore_file_writer *kore_file_writer_open(const char *path);
void kore_file_writer_write(struct kore_file_writer *file, const void *buffer, size_t size);
void kore_file_writer_close(struct kore_file_writer *file);
```

### Save Path

```c
const char *kore_internal_save_path(void);
```
Retorna o caminho apropriado para salvar dados do usuário.

---

## Rede

### HTTP

```c
#include <kore3/network/http.h>

typedef void (*kore_http_callback)(int status_code, const char *body, size_t body_size, void *data);

void kore_http_get(const char *url, kore_http_callback callback, void *data);
void kore_http_post(const char *url, const char *body, size_t body_size, 
                    kore_http_callback callback, void *data);
```

### Sockets

```c
#include <kore3/network/socket.h>

struct kore_socket *kore_socket_create(bool udp);
void kore_socket_connect(struct kore_socket *socket, const char *host, int port);
void kore_socket_send(struct kore_socket *socket, const void *data, size_t size);
void kore_socket_close(struct kore_socket *socket);

// Callbacks
void kore_socket_set_connect_callback(struct kore_socket *socket, 
                                      void (*callback)(struct kore_socket *, void *data), void *data);
void kore_socket_set_recv_callback(struct kore_socket *socket,
                                   void (*callback)(struct kore_socket *, uint8_t *data, size_t size, void *data), void *data);
void kore_socket_set_close_callback(struct kore_socket *socket,
                                    void (*callback)(struct kore_socket *, void *data), void *data);
```

---

## Matemática

### Vetores

```c
#include <kore3/math/vector.h>

typedef struct { float x, y; } kore_vec2;
typedef struct { float x, y, z; } kore_vec3;
typedef struct { float x, y, z, w; } kore_vec4;

kore_vec2 kore_vec2_add(kore_vec2 a, kore_vec2 b);
kore_vec2 kore_vec2_sub(kore_vec2 a, kore_vec2 b);
kore_vec2 kore_vec2_scale(kore_vec2 v, float s);
float kore_vec2_dot(kore_vec2 a, kore_vec2 b);
float kore_vec2_length(kore_vec2 v);
kore_vec2 kore_vec2_normalize(kore_vec2 v);

// Funções similares para vec3 e vec4
```

### Matrizes

```c
#include <kore3/math/matrix.h>

typedef struct {
    float m[4][4];
} kore_mat4;

kore_mat4 kore_mat4_identity(void);
kore_mat4 kore_mat4_translation(float x, float y, float z);
kore_mat4 kore_mat4_scaling(float x, float y, float z);
kore_mat4 kore_mat4_rotation_x(float radians);
kore_mat4 kore_mat4_rotation_y(float radians);
kore_mat4 kore_mat4_rotation_z(float radians);
kore_mat4 kore_mat4_perspective_projection(float fov, float aspect, float near, float far);
kore_mat4 kore_mat4_orthographic_projection(float left, float right, float bottom, float top, float near, float far);
kore_mat4 kore_mat4_multiply(kore_mat4 a, kore_mat4 b);
kore_vec4 kore_mat4_transform(kore_mat4 m, kore_vec4 v);
```

### Quatérnions

```c
#include <kore3/math/quaternion.h>

typedef struct {
    float x, y, z, w;
} kore_quaternion;

kore_quaternion kore_quaternion_identity(void);
kore_quaternion kore_quaternion_from_axis_angle(kore_vec3 axis, float angle);
kore_quaternion kore_quaternion_from_euler(float pitch, float yaw, float roll);
kore_quaternion kore_quaternion_multiply(kore_quaternion a, kore_quaternion b);
kore_quaternion kore_quaternion_normalize(kore_quaternion q);
kore_mat4 kore_quaternion_to_matrix(kore_quaternion q);
```

### Random

```c
#include <kore3/math/random.h>

void kore_random_seed(uint32_t seed);
float kore_random_float(void);      // [0, 1)
uint32_t kore_random_u32(void);
int32_t kore_random_s32(int32_t min, int32_t max);
```

---

## Threads

### Thread

```c
#include <kore3/threads/thread.h>

typedef struct kore_thread *kore_thread_ref;

kore_thread_ref kore_thread_create(void (*function)(void *), void *data);
void kore_thread_join(kore_thread_ref thread);
void kore_thread_destroy(kore_thread_ref thread);
```

### Mutex

```c
#include <kore3/threads/mutex.h>

typedef struct kore_mutex *kore_mutex_ref;

kore_mutex_ref kore_mutex_create(void);
void kore_mutex_lock(kore_mutex_ref mutex);
void kore_mutex_unlock(kore_mutex_ref mutex);
void kore_mutex_destroy(kore_mutex_ref mutex);
```

### Semaphore

```c
#include <kore3/threads/semaphore.h>

typedef struct kore_semaphore *kore_semaphore_ref;

kore_semaphore_ref kore_semaphore_create(int initial_count);
void kore_semaphore_wait(kore_semaphore_ref semaphore);
void kore_semaphore_signal(kore_semaphore_ref semaphore);
void kore_semaphore_destroy(kore_semaphore_ref semaphore);
```

### Event

```c
#include <kore3/threads/event.h>

typedef struct kore_event *kore_event_ref;

kore_event_ref kore_event_create(bool manual_reset);
void kore_event_wait(kore_event_ref event);
void kore_event_set(kore_event_ref event);
void kore_event_reset(kore_event_ref event);
void kore_event_destroy(kore_event_ref event);
```

### Atomic

```c
#include <kore3/threads/atomic.h>

typedef struct {
    volatile int32_t value;
} kore_atomic_i32;

int32_t kore_atomic_load_i32(kore_atomic_i32 *atomic);
void kore_atomic_store_i32(kore_atomic_i32 *atomic, int32_t value);
int32_t kore_atomic_exchange_i32(kore_atomic_i32 *atomic, int32_t value);
int32_t kore_atomic_compare_exchange_i32(kore_atomic_i32 *atomic, int32_t expected, int32_t desired);
int32_t kore_atomic_fetch_add_i32(kore_atomic_i32 *atomic, int32_t operand);
```

---

## Utilitários

### Log

```c
#include <kore3/log.h>

typedef enum {
    KORE_LOG_LEVEL_INFO,
    KORE_LOG_LEVEL_WARNING,
    KORE_LOG_LEVEL_ERROR
} kore_log_level;

void kore_log(kore_log_level level, const char *message, ...);
```

### Imagem

```c
#include <kore3/image.h>

typedef enum {
    KORE_IMAGE_FORMAT_RGBA32,
    KORE_IMAGE_FORMAT_RGB24,
    KORE_IMAGE_FORMAT_GRAY8
} kore_image_format;

struct kore_image {
    uint8_t *pixels;
    int width;
    int height;
    int stride;
    kore_image_format format;
};

struct kore_image *kore_image_load(const char *filename);
struct kore_image *kore_image_load_from_memory(const void *data, size_t size);
void kore_image_destroy(struct kore_image *image);
```

### CPU Compute (SIMD)

```c
#include <kore3/util/cpucompute.h>

// Operações SIMD otimizadas para diferentes arquiteturas
// SSE (x86), NEON (ARM), etc.
```

### Index Allocator

```c
#include <kore3/util/indexallocator.h>

struct kore_index_allocator *kore_index_allocator_create(int capacity);
int kore_index_allocator_alloc(struct kore_index_allocator *allocator);
void kore_index_allocator_free(struct kore_index_allocator *allocator, int index);
void kore_index_allocator_destroy(struct kore_index_allocator *allocator);
```

---

## Sistema

### Informações do Sistema

```c
const char *kore_system_id(void);      // Identificador da plataforma
const char *kore_language(void);        // Código de idioma (ex: "en", "pt")
int kore_cpu_cores(void);               // Número de cores físicos
int kore_hardware_threads(void);        // Número de threads de hardware
double kore_frequency(void);            // Frequência do timestamp
kore_ticks kore_timestamp(void);        // Timestamp atual
double kore_time(void);                 // Tempo atual em segundos
```

### Clipboard

```c
void kore_copy_to_clipboard(const char *text);
```

### URL

```c
void kore_load_url(const char *url);    // Abre URL no navegador padrão
```

### Safe Zone

```c
float kore_safe_zone(void);             // Retorna safe zone para TVs
bool kore_automatic_safe_zone(void);    // Se o sistema gerencia safe zone
void kore_set_safe_zone(float value);   // Define safe zone manualmente
```

### Debug

```c
void kore_debug_break(void);            // Breakpoint no debugger
bool kore_debugger_attached(void);      // Verifica se debugger está anexado
```

### Profiling

```c
void kore_marker_start(const char *name, uint32_t color);
void kore_marker_end(const char *name);
```
Suporta VTune e Superluminal quando compilado com as defines apropriadas.

---

## 2D Rendering

```c
#include <kore3/2d/2d.h>

// Painter abstrato para renderização 2D
struct kore_painter *kore_painter_create(void);
void kore_painter_begin(struct kore_painter *painter, int target_width, int target_height);
void kore_painter_end(struct kore_painter *painter);

// Desenho de primitivas
void kore_painter_draw_rect(struct kore_painter *painter, 
                            float x, float y, float width, float height,
                            kore_color color);
void kore_painter_draw_line(struct kore_painter *painter,
                            float x0, float y0, float x1, float y1,
                            kore_color color, float thickness);

// Imagens
void kore_painter_draw_image(struct kore_painter *painter,
                             struct kore_image *image,
                             float x, float y, float width, float height);

// Texto
void kore_painter_draw_text(struct kore_painter *painter,
                            const char *text, float x, float y,
                            kore_color color, int font_size);
```

---

## VR (Realidade Virtual)

```c
#include <kore3/vr/vr.h>

bool kore_vr_available(void);
void kore_vr_init(void);
void kore_vr_update(void);
void kore_vr_render(void);

// Posição e orientação do headset
kore_vec3 kore_vr_head_position(void);
kore_quaternion kore_vr_head_orientation(void);

// Controladores
bool kore_vr_controller_present(int controller);
kore_vec3 kore_vr_controller_position(int controller);
kore_quaternion kore_vr_controller_orientation(int controller);
```

---

## Exemplo de Uso Completo

```c
#include <kore3/system.h>
#include <kore3/window.h>
#include <kore3/gpu/gpu.h>
#include <kore3/gpu/device.h>
#include <kore3/input/keyboard.h>
#include <kore3/log.h>

static struct kore_gpu_device *device = NULL;

void update_callback(void *data) {
    // Limpar input
    if (kore_keyboard_pressed(KORE_KEY_ESCAPE)) {
        kore_stop();
        return;
    }
    
    // Lógica do jogo aqui
    
    // Renderizar
    struct kore_gpu_command_list *list = kore_gpu_command_list_create();
    kore_gpu_command_list_begin(list);
    
    // Comandos de renderização...
    
    kore_gpu_command_list_end(list);
    kore_gpu_command_list_submit(list);
}

int kickstart(int argc, char **argv) {
    // Inicializar sistema
    kore_window_parameters win_params;
    kore_window_options_set_defaults(&win_params);
    win_params.title = "Meu Jogo";
    win_params.width = 1280;
    win_params.height = 720;
    
    kore_framebuffer_parameters frame_params;
    kore_framebuffer_options_set_defaults(&frame_params);
    frame_params.vertical_sync = true;
    
    kore_init("Meu Jogo", 1280, 720, &win_params, &frame_params);
    
    // Inicializar GPU
    kore_gpu_init(KORE_GPU_API_DIRECT3D11);  // ou VULKAN, METAL, etc.
    device = kore_gpu_device_create();
    
    // Configurar callbacks
    kore_set_update_callback(update_callback, NULL);
    
    // Iniciar main loop
    kore_start();
    
    // Cleanup
    kore_gpu_device_destroy(device);
    
    return 0;
}
```

---

## Compilação

### Usando kmake

```bash
# Build para plataforma nativa
path/to/Kore/make

# Build para Windows
path/to/Kore/make windows

# Build com API gráfica específica
path/to/Kore/make windows -g direct3d12
path/to/Kore/make linux -g vulkan
path/to/Kore/make osx -g metal
```

### Defines de Plataforma

O kmake define automaticamente as seguintes macros baseadas na plataforma:
- `KORE_WINDOWS` / `KORE_WINDOWSAPP`
- `KORE_MACOS`
- `KORE_LINUX`
- `KORE_ANDROID`
- `KORE_IOS`
- `KORE_TVOS`
- `KORE_EMSCRIPTEN`
- `KORE_WASM`

### Defines de API Gráfica

- `KORE_DIRECT3D11`
- `KORE_DIRECT3D12`
- `KORE_VULKAN`
- `KORE_METAL`
- `KORE_OPENGL`
- `KORE_WEBGPU`

---

## Licença

Kore usa a licença zlib/libpng, uma das licenças mais permissivas disponíveis.

Alguns componentes incluídos têm licenças próprias:
- **stb_image.h**: Public Domain
- **stb_vorbis.h**: Public Domain ou MIT
- **lz4**: BSD 2-Clause (pode ser desabilitado com opção)
- **GLEW**: MIT/BSD

---

## Recursos Adicionais

- **Shaders Kongruent**: Linguagem de shader personalizada do Kore. Veja https://github.com/Kode/Kongruent
- **Amostras**: https://github.com/Kode/Kore-Samples
- **Versões estáveis**: 
  - [Kore v2](https://github.com/Kode/Kore/tree/v2) - Versão estável atual
  - [Kore v1](https://github.com/Kode/Kore/tree/v1) - API C++ original
