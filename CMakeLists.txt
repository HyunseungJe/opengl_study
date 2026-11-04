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

# GLM: 벡터/행렬 수학 (glm::vec3, glm::mat4, glm::translate 등)
set(GLM_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLM_BUILD_LIBRARY OFF CACHE BOOL "" FORCE)
set(GLM_BUILD_INSTALL OFF CACHE BOOL "" FORCE)
FetchContent_Declare(
    glm
    GIT_REPOSITORY https://github.com/g-truc/glm.git
    GIT_TAG 1.0.1
    GIT_SHALLOW TRUE
)

# glad v0.1.36은 cmake_minimum_required(VERSION 3.0)이라 CMake 4.x에서 거부됨
set(CMAKE_POLICY_VERSION_MINIMUM 3.5)
FetchContent_MakeAvailable(glfw glad spdlog glm)

# stb_image: 이미지 로더 (헤더 파일 하나만 받아옴, 커밋과 해시를 고정해 항상 같은 파일)
set(STB_COMMIT d14d46f6b44456eb88beac58cba833a4135d4c89)
set(STB_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_deps/stb)
file(DOWNLOAD
    https://raw.githubusercontent.com/nothings/stb/${STB_COMMIT}/stb_image.h
    ${STB_INCLUDE_DIR}/stb_image.h
    EXPECTED_HASH SHA256=594c2fe35d49488b4382dbfaec8f98366defca819d916ac95becf3e75f4200b3
)
add_library(stb_image INTERFACE)
target_include_directories(stb_image INTERFACE ${STB_INCLUDE_DIR})

add_executable(${PROJECT_NAME} 
    src/main.cpp
    src/common.cpp src/common.h
    src/shader.cpp src/shader.h
    src/program.h src/program.cpp
    src/context.h src/context.cpp
    src/buffer.h src/buffer.cpp
    src/vertex_layout.h src/vertex_layout.cpp
    src/image.h src/image.cpp
    src/texture.h src/texture.cpp
    )
target_link_libraries(${PROJECT_NAME} PRIVATE glfw glad spdlog::spdlog stb_image glm::glm)

# 소스 코드에서 쓸 매크로 (#define WINDOW_NAME "..." 와 같은 효과)
target_compile_definitions(${PROJECT_NAME} PRIVATE
    WINDOW_NAME="${WINDOW_NAME}"
    WINDOW_WIDTH=${WINDOW_WIDTH}
    WINDOW_HEIGHT=${WINDOW_HEIGHT}
)

# 빌드할 때마다 shader, image 폴더를 실행 파일 옆으로 복사 (리소스만 고쳐도 반영되도록 별도 타깃으로 매번 실행)
add_custom_target(copy_resources ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_SOURCE_DIR}/shader
        $<TARGET_FILE_DIR:${PROJECT_NAME}>/shader
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_SOURCE_DIR}/image
        $<TARGET_FILE_DIR:${PROJECT_NAME}>/image
    COMMENT "Copying shaders and images"
)
add_dependencies(${PROJECT_NAME} copy_resources)

# 한국어 Windows에서 MSVC는 소스를 CP949로 읽으므로 UTF-8로 읽도록 지정
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)
endif()
