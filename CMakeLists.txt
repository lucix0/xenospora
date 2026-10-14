cmake_minimum_required(VERSION 3.21)
project(Xenospora LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(SHADER_SOURCE ${CMAKE_SOURCE_DIR}/shaders)

include(FetchContent)

# --- bgfx ---
set(BGFX_BUILD_EXAMPLES OFF CACHE BOOL "Build bgfx examples" FORCE)
set(BGFX_BUILD_TOOLS    ON  CACHE BOOL "Build bgfx tools"    FORCE)
FetchContent_Declare(
        bgfx
        GIT_REPOSITORY https://github.com/bkaradzic/bgfx.cmake.git
        GIT_TAG v1.166.9546-589
)
FetchContent_MakeAvailable(bgfx)

# --- GLFW ---
FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw.git
    GIT_TAG        3.5.1
)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(glfw)

# --- BS::thread_pool ---
FetchContent_Declare(
        thread_pool
        GIT_REPOSITORY https://github.com/bshoshany/thread-pool.git
        GIT_TAG        v5.1.0
)
FetchContent_MakeAvailable(thread_pool)

# --- FastNoiseLite ---
FetchContent_Declare(
        FastNoiseLite
        GIT_REPOSITORY https://github.com/Auburn/FastNoiseLite.git
        GIT_TAG        v1.1.1
)
FetchContent_MakeAvailable(FastNoiseLite)

# Shaders
bgfx_compile_shaders(
        TYPE VERTEX
        SHADERS ${SHADER_SOURCE}/vs_composite.sc ${SHADER_SOURCE}/vs_gbuffer.sc
        VARYING_DEF ${SHADER_SOURCE}/varying.def.sc
        OUTPUT_DIR ${CMAKE_BINARY_DIR}/shaders/
)

bgfx_compile_shaders(
        TYPE FRAGMENT
        SHADERS ${SHADER_SOURCE}/fs_composite.sc ${SHADER_SOURCE}/fs_gbuffer.sc
        VARYING_DEF ${SHADER_SOURCE}/varying.def.sc
        OUTPUT_DIR ${CMAKE_BINARY_DIR}/shaders/
)

add_executable(${PROJECT_NAME}
        src/game/main.cpp
        src/engine/shader_program.cpp
        src/engine/material.cpp
        src/engine/texture.cpp
        src/engine/stb_image.cpp
        src/engine/Application.cpp
        include/engine/Application.h
        src/game/camera.cpp
        include/game/camera.h
        src/game/Chunk.cpp
        include/game/chunk.h
        src/game/chunk_mesher.cpp
        include/game/chunk_mesher.h
        include/game/block_type.h
        include/game/vertex.h
        src/game/world.cpp
        include/game/world.h
        src/game/chunk_streamer.cpp
        include/game/chunk_streamer.h
        src/game/chunk_renderer.cpp
        include/game/chunk_renderer.h
        src/game/chunk_generator.cpp
        include/game/chunk_generator.h
        src/engine/deferred_renderer.cpp
        include/engine/deferred_renderer.h
        ${SHADER_SOURCE}/vs_composite.sc
        ${SHADER_SOURCE}/vs_gbuffer.sc
        ${SHADER_SOURCE}/fs_composite.sc
        ${SHADER_SOURCE}/fs_gbuffer.sc
        include/engine/shader_manager.h
        src/engine/shader_manager.cpp
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_SOURCE_DIR}/src ${thread_pool_SOURCE_DIR}/include ${fastnoiselite_SOURCE_DIR}/Cpp include
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    bgfx
    bx
    bimg
    glfw
)
