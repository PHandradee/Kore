# Kore 3 API Documentation

## Table of Contents

1. [System Initialization](#1-system-initialization)
2. [Window Management](#2-window-management)
3. [GPU API](#3-gpu-api)
4. [Audio System](#4-audio-system)
5. [Input System](#5-input-system)
6. [File I/O](#6-file-io)
7. [Networking](#7-networking)
8. [Math Library](#8-math-library)
9. [Threading](#9-threading)
10. [Utilities](#10-utilities)

---

## 1. System Initialization

### `kore_init()`

Initialize the Kore system and create a window.

```c
void kore_init(const char *title, int width, int height, 
               struct kore_window_options *options,
               struct kore_framebuffer_options *framebuffer);
```

**Parameters:**
- `title` - Window title
- `width` - Window width in pixels
- `height` - Window height in pixels
- `options` - Optional window configuration (NULL for defaults)
- `framebuffer` - Optional framebuffer configuration (NULL for defaults)

**Example:**
```c
kore_init("My Game", 1280, 720, NULL, NULL);
```

---

### `kore_start()`

Start the main loop. This function blocks until `kore_stop()` is called.

```c
void kore_start(void);
```

**Example:**
```c
kore_set_update_callback(update, NULL);
kore_start();
```

---

### `kore_stop()`

Stop the main loop and exit the application.

```c
void kore_stop(void);
```

**Example:**
```c
if (kore_keyboard_pressed(KORE_KEY_ESCAPE)) {
    kore_stop();
}
```

---

### `kore_set_update_callback()`

Set the callback function called every frame.

```c
void kore_set_update_callback(void (*callback)(void*), void* data);
```

**Parameters:**
- `callback` - Function to call every frame
- `data` - User data passed to callback

**Example:**
```c
void update(void* data) {
    // Game logic here
}

kore_set_update_callback(update, NULL);
```

---

### `kore_set_drop_files_callback()`

Set callback for file drop events (drag & drop).

```c
void kore_set_drop_files_callback(void (*callback)(const char*, void*), void* data);
```

---

### `kore_set_foreground_callback()`

Set callback when window gains focus.

```c
void kore_set_foreground_callback(void (*callback)(void*), void* data);
```

---

### `kore_set_background_callback()`

Set callback when window loses focus.

```c
void kore_set_background_callback(void (*callback)(void*), void* data);
```

---

### `kore_shutdown()`

Cleanup and shutdown the Kore system.

```c
void kore_shutdown(void);
```

---

## 2. Window Management

### `kore_window_create()`

Create an additional window.

```c
struct kore_window* kore_window_create(int x, int y, int width, int height,
                                       struct kore_window_options *options);
```

**Parameters:**
- `x`, `y` - Window position
- `width`, `height` - Window dimensions
- `options` - Window options (can be NULL)

**Returns:** Pointer to created window, or NULL on failure

---

### `kore_window_destroy()`

Destroy a window.

```c
void kore_window_destroy(struct kore_window *window);
```

---

### `kore_window_resize()`

Resize a window.

```c
void kore_window_resize(struct kore_window *window, int width, int height);
```

---

### `kore_window_move()`

Move a window.

```c
void kore_window_move(struct kore_window *window, int x, int y);
```

---

### `kore_window_change_features()`

Change window features (fullscreen, resizable, etc.).

```c
void kore_window_change_features(struct kore_window *window, 
                                 int features);
```

**Features flags:**
- `KORE_WINDOW_FULLSCREEN`
- `KORE_WINDOW_RESIZABLE`
- `KORE_WINDOW_BORDERLESS`

---

### `kore_window_set_title()`

Set window title.

```c
void kore_window_set_title(struct kore_window *window, const char *title);
```

---

### `kore_window_get_width()` / `kore_window_get_height()`

Get window dimensions.

```c
int kore_window_get_width(struct kore_window *window);
int kore_window_get_height(struct kore_window *window);
```

---

### `kore_window_get_presentation_interval()`

Get the presentation interval (vsync).

```c
int kore_window_get_presentation_interval(struct kore_window *window);
```

---

### `kore_display_count()`

Get number of connected displays.

```c
int kore_display_count(void);
```

---

### `kore_display_by_id()`

Get display by ID.

```c
struct kore_display* kore_display_by_id(int id);
```

---

### `kore_display_width()` / `kore_display_height()`

Get display dimensions.

```c
int kore_display_width(struct kore_display *display);
int kore_display_height(struct kore_display *display);
```

---

## 3. GPU API

### Device Management

#### `kore_gpu_init()`

Initialize the GPU with specified API.

```c
void kore_gpu_init(enum kore_gpu_api api);
```

**API types:**
- `KORE_GPU_API_DEFAULT` - Auto-select best API
- `KORE_GPU_API_DIRECT3D11`
- `KORE_GPU_API_DIRECT3D12`
- `KORE_GPU_API_VULKAN`
- `KORE_GPU_API_METAL`
- `KORE_GPU_API_OPENGL`
- `KORE_GPU_API_WEBGPU`

---

#### `kore_gpu_device_create()`

Create a GPU device.

```c
struct kore_gpu_device* kore_gpu_device_create(struct kore_window *window);
```

---

#### `kore_gpu_device_destroy()`

Destroy a GPU device.

```c
void kore_gpu_device_destroy(struct kore_gpu_device *device);
```

---

### Command Lists

#### `kore_gpu_command_list_create()`

Create a command list for recording GPU commands.

```c
struct kore_gpu_command_list* kore_gpu_command_list_create(void);
```

---

#### `kore_gpu_command_list_begin()`

Begin recording commands.

```c
void kore_gpu_command_list_begin(struct kore_gpu_command_list *list);
```

---

#### `kore_gpu_command_list_end()`

End recording commands.

```c
void kore_gpu_command_list_end(struct kore_gpu_command_list *list);
```

---

#### `kore_gpu_command_list_submit()`

Submit command list to GPU queue.

```c
void kore_gpu_command_list_submit(struct kore_gpu_command_list *list);
```

---

#### `kore_gpu_command_list_set_pipeline()`

Set the pipeline state for rendering.

```c
void kore_gpu_command_list_set_pipeline(
    struct kore_gpu_command_list *list,
    struct kore_gpu_pipeline *pipeline);
```

---

#### `kore_gpu_command_list_draw()`

Draw primitives.

```c
void kore_gpu_command_list_draw(struct kore_gpu_command_list *list,
                                int vertex_count,
                                int instance_count);
```

---

#### `kore_gpu_command_list_draw_indexed()`

Draw indexed primitives.

```c
void kore_gpu_command_list_draw_indexed(
    struct kore_gpu_command_list *list,
    int index_count,
    int instance_count);
```

---

#### `kore_gpu_command_list_dispatch()`

Dispatch compute shader.

```c
void kore_gpu_command_list_dispatch(struct kore_gpu_command_list *list,
                                    int x, int y, int z);
```

---

### Buffers

#### `kore_gpu_buffer_create()`

Create a GPU buffer.

```c
struct kore_gpu_buffer* kore_gpu_buffer_create(
    enum kore_gpu_buffer_type type,
    size_t size,
    enum kore_gpu_buffer_usage usage);
```

**Buffer types:**
- `KORE_GPU_BUFFER_TYPE_VERTEX`
- `KORE_GPU_BUFFER_TYPE_INDEX`
- `KORE_GPU_BUFFER_TYPE_CONSTANT`
- `KORE_GPU_BUFFER_TYPE_STORAGE`

**Usage flags:**
- `KORE_GPU_BUFFER_USAGE_STATIC`
- `KORE_GPU_BUFFER_USAGE_DYNAMIC`
- `KORE_GPU_BUFFER_USAGE_READ`
- `KORE_GPU_BUFFER_USAGE_WRITE`

---

#### `kore_gpu_buffer_map()`

Map buffer for CPU access.

```c
void* kore_gpu_buffer_map(struct kore_gpu_buffer *buffer);
```

---

#### `kore_gpu_buffer_unmap()`

Unmap buffer after CPU access.

```c
void kore_gpu_buffer_unmap(struct kore_gpu_buffer *buffer);
```

---

#### `kore_gpu_buffer_upload()`

Upload data to buffer.

```c
void kore_gpu_buffer_upload(struct kore_gpu_buffer *buffer,
                            const void *data,
                            size_t size);
```

---

#### `kore_gpu_buffer_destroy()`

Destroy a buffer.

```c
void kore_gpu_buffer_destroy(struct kore_gpu_buffer *buffer);
```

---

### Textures

#### `kore_gpu_texture_create()`

Create a texture.

```c
struct kore_gpu_texture* kore_gpu_texture_create(
    int width,
    int height,
    enum kore_gpu_texture_format format,
    enum kore_gpu_texture_usage usage);
```

**Texture formats:**
- `KORE_GPU_TEXTURE_FORMAT_RGBA8`
- `KORE_GPU_TEXTURE_FORMAT_RGB8`
- `KORE_GPU_TEXTURE_FORMAT_DEPTH32`
- `KORE_GPU_TEXTURE_FORMAT_DEPTH24_STENCIL8`
- `KORE_GPU_TEXTURE_FORMAT_RGBA16F`
- `KORE_GPU_TEXTURE_FORMAT_RGBA32F`

---

#### `kore_gpu_texture_upload()`

Upload pixel data to texture.

```c
void kore_gpu_texture_upload(struct kore_gpu_texture *texture,
                             const void *pixels,
                             int stride);
```

---

#### `kore_gpu_texture_generate_mipmaps()`

Generate mipmaps for texture.

```c
void kore_gpu_texture_generate_mipmaps(struct kore_gpu_texture *texture);
```

---

#### `kore_gpu_texture_destroy()`

Destroy a texture.

```c
void kore_gpu_texture_destroy(struct kore_gpu_texture *texture);
```

---

### Samplers

#### `kore_gpu_sampler_create()`

Create a texture sampler.

```c
struct kore_gpu_sampler* kore_gpu_sampler_create(
    enum kore_gpu_filter min_filter,
    enum kore_gpu_filter mag_filter,
    enum kore_gpu_mipmap_filter mipmap_filter,
    enum kore_gpu_address_mode u_address,
    enum kore_gpu_address_mode v_address,
    enum kore_gpu_address_mode w_address);
```

**Filter modes:**
- `KORE_GPU_FILTER_NEAREST`
- `KORE_GPU_FILTER_LINEAR`

**Address modes:**
- `KORE_GPU_ADDRESS_MODE_CLAMP`
- `KORE_GPU_ADDRESS_MODE_REPEAT`
- `KORE_GPU_ADDRESS_MODE_MIRROR`

---

### Pipelines

#### `kore_gpu_pipeline_create()`

Create a render pipeline.

```c
struct kore_gpu_pipeline* kore_gpu_pipeline_create(
    struct kore_gpu_shader *vertex_shader,
    struct kore_gpu_shader *fragment_shader,
    struct kore_gpu_input_layout *input_layout,
    struct kore_gpu_blend_state *blend_state,
    struct kore_gpu_depth_state *depth_state,
    struct kore_gpu_rasterizer_state *rasterizer_state);
```

---

#### `kore_gpu_pipeline_destroy()`

Destroy a pipeline.

```c
void kore_gpu_pipeline_destroy(struct kore_gpu_pipeline *pipeline);
```

---

### Shaders

#### `kore_gpu_shader_create()`

Create a shader from compiled bytecode.

```c
struct kore_gpu_shader* kore_gpu_shader_create(
    const void *bytecode,
    size_t size,
    enum kore_gpu_shader_stage stage);
```

**Shader stages:**
- `KORE_GPU_SHADER_STAGE_VERTEX`
- `KORE_GPU_SHADER_STAGE_FRAGMENT`
- `KORE_GPU_SHADER_STAGE_COMPUTE`

---

#### `kore_gpu_shader_destroy()`

Destroy a shader.

```c
void kore_gpu_shader_destroy(struct kore_gpu_shader *shader);
```

---

### Render Targets

#### `kore_gpu_framebuffer_create()`

Create a framebuffer (render target).

```c
struct kore_gpu_framebuffer* kore_gpu_framebuffer_create(
    struct kore_gpu_texture *color,
    struct kore_gpu_texture *depth);
```

---

#### `kore_gpu_command_list_set_framebuffer()`

Set the active framebuffer for rendering.

```c
void kore_gpu_command_list_set_framebuffer(
    struct kore_gpu_command_list *list,
    struct kore_gpu_framebuffer *framebuffer);
```

---

#### `kore_gpu_command_list_viewport()`

Set the viewport.

```c
void kore_gpu_command_list_viewport(
    struct kore_gpu_command_list *list,
    int x, int y, int width, int height);
```

---

#### `kore_gpu_command_list_scissor()`

Set the scissor rectangle.

```c
void kore_gpu_command_list_scissor(
    struct kore_gpu_command_list *list,
    int x, int y, int width, int height);
```

---

#### `kore_gpu_command_list_clear()`

Clear a render target.

```c
void kore_gpu_command_list_clear(
    struct kore_gpu_command_list *list,
    struct kore_gpu_framebuffer *framebuffer,
    float r, float g, float b, float a,
    float depth, int stencil);
```

---

### Ray Tracing (Experimental)

#### `kore_gpu_acceleration_structure_create()`

Create a ray tracing acceleration structure.

```c
struct kore_gpu_acceleration_structure* 
kore_gpu_acceleration_structure_create(void);
```

---

#### `kore_gpu_command_list_trace_rays()`

Dispatch ray tracing shader.

```c
void kore_gpu_command_list_trace_rays(
    struct kore_gpu_command_list *list,
    struct kore_gpu_ray_tracing_pipeline *pipeline,
    int width, int height, int depth);
```

---

## 4. Audio System

### Mixer

#### `kore_mixer_init()`

Initialize the audio mixer.

```c
void kore_mixer_init(int sample_rate);
```

---

#### `kore_mixer_sound_play()`

Play a sound.

```c
void kore_mixer_sound_play(struct kore_mixer_sound *sound,
                           float volume,
                           bool loop);
```

---

#### `kore_mixer_sound_stop()`

Stop a playing sound.

```c
void kore_mixer_sound_stop(struct kore_mixer_sound *sound);
```

---

#### `kore_mixer_set_volume()`

Set master volume.

```c
void kore_mixer_set_volume(float volume);
```

**Parameters:**
- `volume` - Volume level (0.0 to 1.0)

---

### Sounds

#### `kore_mixer_sound_load()`

Load a sound from file.

```c
struct kore_mixer_sound* kore_mixer_sound_load(const char *filename);
```

**Supported formats:** .wav, .ogg

---

#### `kore_mixer_sound_destroy()`

Destroy a loaded sound.

```c
void kore_mixer_sound_destroy(struct kore_mixer_sound *sound);
```

---

### Music Streaming

#### `kore_mixer_music_play()`

Play music from file (streaming).

```c
void kore_mixer_music_play(const char *filename, bool loop);
```

---

#### `kore_mixer_music_stop()`

Stop music playback.

```c
void kore_mixer_music_stop(void);
```

---

#### `kore_mixer_music_pause()`

Pause/resume music.

```c
void kore_mixer_music_pause(bool pause);
```

---

## 5. Input System

### Keyboard

#### `kore_keyboard_pressed()`

Check if key was pressed this frame.

```c
bool kore_keyboard_pressed(enum kore_key key);
```

---

#### `kore_keyboard_down()`

Check if key is currently held down.

```c
bool kore_keyboard_down(enum kore_key key);
```

---

#### Key codes:

```c
enum kore_key {
    KORE_KEY_A, KORE_KEY_B, KORE_KEY_C, ..., KORE_KEY_Z,
    KORE_KEY_0, KORE_KEY_1, ..., KORE_KEY_9,
    KORE_KEY_F1, ..., KORE_KEY_F12,
    KORE_KEY_ESCAPE,
    KORE_KEY_RETURN,
    KORE_KEY_SPACE,
    KORE_KEY_LEFT, KORE_KEY_RIGHT, KORE_KEY_UP, KORE_KEY_DOWN,
    // ... and more
};
```

---

### Mouse

#### `kore_mouse_x()` / `kore_mouse_y()`

Get mouse position.

```c
int kore_mouse_x(void);
int kore_mouse_y(void);
```

---

#### `kore_mouse_moved_x()` / `kore_mouse_moved_y()`

Get mouse movement delta.

```c
int kore_mouse_moved_x(void);
int kore_mouse_moved_y(void);
```

---

#### `kore_mouse_pressed()`

Check if mouse button was pressed.

```c
bool kore_mouse_pressed(enum kore_mouse_button button);
```

**Button codes:**
- `KORE_MOUSE_BUTTON_LEFT`
- `KORE_MOUSE_BUTTON_RIGHT`
- `KORE_MOUSE_BUTTON_MIDDLE`

---

#### `kore_mouse_down()`

Check if mouse button is held.

```c
bool kore_mouse_down(enum kore_mouse_button button);
```

---

#### `kore_mouse_show()` / `kore_mouse_hide()`

Show/hide mouse cursor.

```c
void kore_mouse_show(void);
void kore_mouse_hide(void);
```

---

### Gamepad

#### `kore_gamepad_count()`

Get number of connected gamepads.

```c
int kore_gamepad_count(void);
```

---

#### `kore_gamepad_pressed()`

Check if gamepad button was pressed.

```c
bool kore_gamepad_pressed(int gamepad, enum kore_gamepad_button button);
```

**Button codes (17 buttons):**
- `KORE_GAMEPAD_BUTTON_A`
- `KORE_GAMEPAD_BUTTON_B`
- `KORE_GAMEPAD_BUTTON_X`
- `KORE_GAMEPAD_BUTTON_Y`
- `KORE_GAMEPAD_BUTTON_LEFT_SHOULDER`
- `KORE_GAMEPAD_BUTTON_RIGHT_SHOULDER`
- `KORE_GAMEPAD_BUTTON_LEFT_TRIGGER`
- `KORE_GAMEPAD_BUTTON_RIGHT_TRIGGER`
- `KORE_GAMEPAD_BUTTON_BACK`
- `KORE_GAMEPAD_BUTTON_START`
- `KORE_GAMEPAD_BUTTON_LEFT_STICK`
- `KORE_GAMEPAD_BUTTON_RIGHT_STICK`
- `KORE_GAMEPAD_BUTTON_DPAD_UP`
- `KORE_GAMEPAD_BUTTON_DPAD_DOWN`
- `KORE_GAMEPAD_BUTTON_DPAD_LEFT`
- `KORE_GAMEPAD_BUTTON_DPAD_RIGHT`
- `KORE_GAMEPAD_BUTTON_GUIDE`

---

#### `kore_gamepad_axis()`

Get gamepad axis value.

```c
float kore_gamepad_axis(int gamepad, enum kore_gamepad_axis axis);
```

**Axes:**
- `KORE_GAMEPAD_AXIS_LEFT_X`
- `KORE_GAMEPAD_AXIS_LEFT_Y`
- `KORE_GAMEPAD_AXIS_RIGHT_X`
- `KORE_GAMEPAD_AXIS_RIGHT_Y`

**Returns:** Value between -1.0 and 1.0

---

### Touch

#### `kore_touch_surface_count()`

Get number of touch points.

```c
int kore_touch_surface_count(void);
```

---

#### `kore_touch_surface_x()` / `kore_touch_surface_y()`

Get touch point position.

```c
float kore_touch_surface_x(int index);
float kore_touch_surface_y(int index);
```

---

### Sensors

#### `kore_sensor_available()`

Check if sensor is available.

```c
bool kore_sensor_available(enum kore_sensor_type type);
```

**Sensor types:**
- `KORE_SENSOR_ACCELEROMETER`
- `KORE_SENSOR_GYROSCOPE`

---

#### `kore_sensor_read()`

Read sensor value.

```c
float kore_sensor_read(enum kore_sensor_type type, int axis);
```

---

## 6. File I/O

### Reading Files

#### `kore_file_read()`

Read entire file into memory.

```c
void* kore_file_read(const char *filename, size_t *out_size);
```

**Returns:** Pointer to allocated buffer (must be freed), or NULL on error

---

#### `kore_file_read_async()`

Read file asynchronously.

```c
void kore_file_read_async(const char *filename,
                          void (*callback)(void*, size_t),
                          void *user_data);
```

---

### Writing Files

#### `kore_file_write()`

Write data to file.

```c
bool kore_file_write(const char *filename,
                     const void *data,
                     size_t size);
```

---

#### `kore_file_write_async()`

Write file asynchronously.

```c
void kore_file_write_async(const char *filename,
                           const void *data,
                           size_t size,
                           void (*callback)(bool),
                           void *user_data);
```

---

### Save Path

#### `kore_save_path()`

Get platform-specific save path.

```c
const char* kore_save_path(void);
```

**Returns:** Path like:
- Windows: `C:\Users\Username\AppData\Roaming\GameName\`
- macOS: `/Users/Username/Library/Application Support/GameName/`
- Linux: `/home/username/.local/share/GameName/`

---

### File Existence

#### `kore_file_exists()`

Check if file exists.

```c
bool kore_file_exists(const char *filename);
```

---

## 7. Networking

### HTTP

#### `kore_http_get()`

Perform HTTP GET request.

```c
void kore_http_get(const char *url,
                   void (*callback)(int status, const char *body, size_t size),
                   void *user_data);
```

---

#### `kore_http_post()`

Perform HTTP POST request.

```c
void kore_http_post(const char *url,
                    const char *content_type,
                    const void *body,
                    size_t body_size,
                    void (*callback)(int status, const char *response, size_t size),
                    void *user_data);
```

---

### Sockets

#### TCP Sockets

```c
struct kore_socket_tcp* kore_socket_tcp_create(void);
void kore_socket_tcp_connect(struct kore_socket_tcp *socket,
                             const char *host,
                             int port,
                             void (*callback)(bool success));
void kore_socket_tcp_send(struct kore_socket_tcp *socket,
                          const void *data,
                          size_t size);
void kore_socket_tcp_close(struct kore_socket_tcp *socket);
```

---

#### UDP Sockets

```c
struct kore_socket_udp* kore_socket_udp_create(void);
void kore_socket_udp_bind(struct kore_socket_udp *socket, int port);
void kore_socket_udp_send(struct kore_socket_udp *socket,
                          const char *host,
                          int port,
                          const void *data,
                          size_t size);
void kore_socket_udp_close(struct kore_socket_udp *socket);
```

---

## 8. Math Library

### Vectors

#### 2D Vector

```c
struct kore_vec2 {
    float x, y;
};

struct kore_vec2 kore_vec2_add(struct kore_vec2 a, struct kore_vec2 b);
struct kore_vec2 kore_vec2_sub(struct kore_vec2 a, struct kore_vec2 b);
struct kore_vec2 kore_vec2_scale(struct kore_vec2 v, float s);
float kore_vec2_dot(struct kore_vec2 a, struct kore_vec2 b);
float kore_vec2_length(struct kore_vec2 v);
struct kore_vec2 kore_vec2_normalize(struct kore_vec2 v);
```

---

#### 3D Vector

```c
struct kore_vec3 {
    float x, y, z;
};

struct kore_vec3 kore_vec3_add(struct kore_vec3 a, struct kore_vec3 b);
struct kore_vec3 kore_vec3_sub(struct kore_vec3 a, struct kore_vec3 b);
struct kore_vec3 kore_vec3_scale(struct kore_vec3 v, float s);
float kore_vec3_dot(struct kore_vec3 a, struct kore_vec3 b);
struct kore_vec3 kore_vec3_cross(struct kore_vec3 a, struct kore_vec3 b);
float kore_vec3_length(struct kore_vec3 v);
struct kore_vec3 kore_vec3_normalize(struct kore_vec3 v);
```

---

#### 4D Vector

```c
struct kore_vec4 {
    float x, y, z, w;
};

// Similar operations as vec2/vec3
```

---

### Matrices

#### 4x4 Matrix

```c
struct kore_mat4 {
    float m[4][4];
};

struct kore_mat4 kore_mat4_identity(void);
struct kore_mat4 kore_mat4_translate(float x, float y, float z);
struct kore_mat4 kore_mat4_rotate(float angle, float x, float y, float z);
struct kore_mat4 kore_mat4_scale(float x, float y, float z);
struct kore_mat4 kore_mat4_perspective(float fov, float aspect,
                                       float near, float far);
struct kore_mat4 kore_mat4_ortho(float left, float right,
                                 float bottom, float top,
                                 float near, float far);
struct kore_mat4 kore_mat4_look_at(struct kore_vec3 eye,
                                   struct kore_vec3 center,
                                   struct kore_vec3 up);
struct kore_mat4 kore_mat4_multiply(struct kore_mat4 a, struct kore_mat4 b);
struct kore_vec4 kore_mat4_transform(struct kore_mat4 m, struct kore_vec4 v);
```

---

### Quaternions

```c
struct kore_quaternion {
    float x, y, z, w;
};

struct kore_quaternion kore_quaternion_from_axis_angle(float angle,
                                                        float x, float y, float z);
struct kore_quaternion kore_quaternion_from_euler(float pitch, float yaw, float roll);
struct kore_quaternion kore_quaternion_multiply(struct kore_quaternion a,
                                                 struct kore_quaternion b);
struct kore_quaternion kore_quaternion_normalize(struct kore_quaternion q);
struct kore_mat4 kore_quaternion_to_matrix(struct kore_quaternion q);
```

---

### Random Numbers

```c
void kore_random_seed(unsigned int seed);
float kore_random_float(void);           // Returns 0.0 to 1.0
int kore_random_int(int max);            // Returns 0 to max-1
float kore_random_range(float min, float max);
```

---

## 9. Threading

### Thread Management

#### `kore_thread_create()`

Create a new thread.

```c
struct kore_thread* kore_thread_create(void (*func)(void*), void *arg);
```

---

#### `kore_thread_join()`

Wait for thread to complete.

```c
void kore_thread_join(struct kore_thread *thread);
```

---

#### `kore_thread_destroy()`

Destroy a thread handle.

```c
void kore_thread_destroy(struct kore_thread *thread);
```

---

### Synchronization

#### Mutex

```c
struct kore_mutex* kore_mutex_create(void);
void kore_mutex_lock(struct kore_mutex *mutex);
void kore_mutex_unlock(struct kore_mutex *mutex);
void kore_mutex_destroy(struct kore_mutex *mutex);
```

---

#### Semaphore

```c
struct kore_semaphore* kore_semaphore_create(int count);
void kore_semaphore_acquire(struct kore_semaphore *semaphore);
void kore_semaphore_release(struct kore_semaphore *semaphore);
void kore_semaphore_destroy(struct kore_semaphore *semaphore);
```

---

#### Event

```c
struct kore_event* kore_event_create(void);
void kore_event_signal(struct kore_event *event);
void kore_event_wait(struct kore_event *event);
void kore_event_reset(struct kore_event *event);
void kore_event_destroy(struct kore_event *event);
```

---

### Atomic Operations

```c
int kore_atomic_add(volatile int *value, int addend);
int kore_atomic_sub(volatile int *value, int subtrahend);
int kore_atomic_inc(volatile int *value);
int kore_atomic_dec(volatile int *value);
int kore_atomic_load(volatile int *value);
void kore_atomic_store(volatile int *value, int desired);
int kore_atomic_compare_exchange(volatile int *value, int expected, int desired);
```

---

## 10. Utilities

### Logging

#### `kore_log()`

Log a message.

```c
void kore_log(enum kore_log_level level, const char *format, ...);
```

**Log levels:**
- `KORE_LOG_LEVEL_DEBUG`
- `KORE_LOG_LEVEL_INFO`
- `KORE_LOG_LEVEL_WARNING`
- `KORE_LOG_LEVEL_ERROR`

**Example:**
```c
kore_log(KORE_LOG_LEVEL_INFO, "Player score: %d", score);
```

---

### Image Loading

#### `kore_image_load()`

Load image from file.

```c
struct kore_image* kore_image_load(const char *filename);
```

**Returns:** Image with `width`, `height`, `pixels` fields

**Supported formats:** PNG, JPG, BMP, TGA (via stb_image)

---

#### `kore_image_destroy()`

Free loaded image.

```c
void kore_image_destroy(struct kore_image *image);
```

---

### CPU Compute (SIMD)

```c
// Vectorized operations (platform-specific SIMD)
void kore_cpu_float4_add(float *result, const float *a, const float *b);
void kore_cpu_float4_mul(float *result, const float *a, const float *b);
// ... more SIMD operations
```

---

### Index Allocator

```c
struct kore_index_allocator* kore_index_allocator_create(int max_indices);
int kore_index_allocator_allocate(struct kore_index_allocator *alloc);
void kore_index_allocator_free(struct kore_index_allocator *alloc, int index);
void kore_index_allocator_destroy(struct kore_index_allocator *alloc);
```

Useful for managing entity IDs, resource handles, etc.

---

## Compilation Guide

### Basic Setup

1. **Include headers:**
```c
#include <kore3/system.h>
#include <kore3/window.h>
#include <kore3/gpu/gpu.h>
```

2. **Compile with Kore:**
```bash
# Using kmake
path/to/Kore/make windows -g direct3d11

# Or manually compile sources
gcc -I/path/to/Kore/includes main.c \
    /path/to/Kore/sources/*.c \
    /path/to/Kore/backends/system/windows/*.c \
    /path/to/Kore/backends/gpu/direct3d11/*.c \
    -ld3d11 -ldxgi -o mygame.exe
```

### Platform-Specific Libraries

| Platform | Required Libraries |
|----------|-------------------|
| Windows | d3d11.lib, dxgi.lib, user32.lib |
| macOS | Metal.framework, Cocoa.framework |
| Linux | -lvulkan, -lX11, -lGL |
| Android | -landroid, -lEGL, -lGLESv3 |
| Web | Link with Emscripten |

---

## Error Handling

Most Kore functions return NULL or false on error. Check return values:

```c
struct kore_gpu_buffer *buffer = kore_gpu_buffer_create(...);
if (!buffer) {
    kore_log(KORE_LOG_LEVEL_ERROR, "Failed to create buffer");
    return;
}
```

Use `kore_log()` for debugging and error reporting.

---

## Best Practices

1. **Initialize once**: Call `kore_init()` only once at startup
2. **Resource cleanup**: Destroy all GPU resources before shutdown
3. **Thread safety**: GPU calls must be made from main thread
4. **Memory management**: Free all allocated memory (images, files, etc.)
5. **Error checking**: Always check return values
6. **VSync**: Enable vsync for smooth framerate
7. **Asset loading**: Load assets asynchronously when possible

---

## Version Information

This documentation covers Kore 3 (experimental). API may change between versions.

- **Repository**: https://github.com/Kode/Kore
- **License**: zlib/libpng License
- **Language**: Pure C

