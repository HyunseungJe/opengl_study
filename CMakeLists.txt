cmake_minimum_required(VERSION 3.20)
project(opengl_study LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(WINDOW_NAME "OpenGL Study")
set(WINDOW_WIDTH 960)
set(WINDOW_HEIGHT 540)

include(FetchContent)

# GLFW: 창 생성, 입력 처리
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw.git
    GIT_TAG 3.4
    GIT_SHALLOW TRUE
)

# glad: OpenGL 함수 로더 (빌드할 때 Python으로 생성됨)
set(GLAD_PROFILE "core" CACHE STRING "" FORCE)
set(GLAD_API "gl=3.3" CACHE STRING "" FORCE)
set(GLAD_GENERATOR "c" CACHE STRING "" FORCE)
FetchContent_Declare(
    glad
    GIT_REPOSITORY https://github.com/Dav1dde/glad.git
    GIT_TAG v0.1.36
    GIT_SHALLOW TRUE
)

# spdlog: 로깅 (SPDLOG_INFO, SPDLOG_ERROR 등)
FetchContent_Declare(
    spdlog
    GIT_REPOSITORY https://github.com/gabime/spdlog.git
    GIT_TAG v1.15.3
    GIT_SHALLOW TRUE
)

# glad v0.1.36은 cmake_minimum_required(VERSION 3.0)이라 CMake 4.x에서 거부됨
set(CMAKE_POLICY_VERSION_MINIMUM 3.5)
FetchContent_MakeAvailable(glfw glad spdlog)

add_executable(${PROJECT_NAME} 
    src/main.cpp
    src/common.cpp src/common.h
    src/shader.cpp src/shader.h
    src/program.h src/program.cpp
    src/context.h src/context.cpp
    )
target_link_libraries(${PROJECT_NAME} PRIVATE glfw glad spdlog::spdlog)

# 소스 코드에서 쓸 매크로 (#define WINDOW_NAME "..." 와 같은 효과)
target_compile_definitions(${PROJECT_NAME} PRIVATE
    WINDOW_NAME="${WINDOW_NAME}"
    WINDOW_WIDTH=${WINDOW_WIDTH}
    WINDOW_HEIGHT=${WINDOW_HEIGHT}
)

# 빌드할 때마다 shader 폴더를 실행 파일 옆으로 복사 (셰이더만 고쳐도 반영되도록 별도 타깃으로 매번 실행)
add_custom_target(copy_shaders ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_SOURCE_DIR}/shader
        $<TARGET_FILE_DIR:${PROJECT_NAME}>/shader
    COMMENT "Copying shaders"
)
add_dependencies(${PROJECT_NAME} copy_shaders)

# 한국어 Windows에서 MSVC는 소스를 CP949로 읽으므로 UTF-8로 읽도록 지정
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)
endif()
