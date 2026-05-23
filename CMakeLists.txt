cmake_minimum_required(VERSION 3.25)
project(RealTimePhysicsSandbox LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Export compile commands for clangd LSP
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ─── Build type default ───────────────────────────────────────────────────────
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

# ─── System dependencies (apt) ───────────────────────────────────────────────
find_package(glfw3 3.3 REQUIRED)
find_package(glm REQUIRED)

# ─── GLAD (vendored static library target) ────────────────────────────────────
add_library(glad STATIC
    thirdparty/glad/src/glad.c
)
target_include_directories(glad PUBLIC
    thirdparty/glad/include
)
# GLAD is C code — no C++ warnings needed
set_target_properties(glad PROPERTIES LINKER_LANGUAGE C)

# ─── ImGui (vendored static library target) ───────────────────────────────────
add_library(imgui STATIC
    thirdparty/imgui/imgui.cpp
    thirdparty/imgui/imgui_draw.cpp
    thirdparty/imgui/imgui_tables.cpp
    thirdparty/imgui/imgui_widgets.cpp
    thirdparty/imgui/imgui_demo.cpp
    thirdparty/imgui/backends/imgui_impl_glfw.cpp
    thirdparty/imgui/backends/imgui_impl_opengl3.cpp
)
target_include_directories(imgui PUBLIC
    thirdparty/imgui
    thirdparty/imgui/backends
)
target_link_libraries(imgui PUBLIC glfw)

# ─── Engine executable ────────────────────────────────────────────────────────
add_executable(PhysicsSandbox
    src/main.cpp
)

target_include_directories(PhysicsSandbox PRIVATE
    src
    thirdparty/glad/include
    thirdparty/imgui
    thirdparty/imgui/backends
)

target_link_libraries(PhysicsSandbox PRIVATE
    glad
    imgui
    glfw
    GL
)

# ─── Compiler flags ───────────────────────────────────────────────────────────
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(PhysicsSandbox PRIVATE
        -Wall -Wextra -Werror
        -g3 -O0
        -fsanitize=address,undefined
        -fno-omit-frame-pointer
    )
    target_link_options(PhysicsSandbox PRIVATE
        -fsanitize=address,undefined
    )
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_options(PhysicsSandbox PRIVATE
        -Wall -Wextra -Werror
        -O2
    )
endif()
