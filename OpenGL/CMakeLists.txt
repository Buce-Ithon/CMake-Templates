cmake_minimum_required(VERSION 3.16)
project(OpenGL-Project 
VERSION 1.0.0 
LANGUAGES C CXX
DESCRIPTION "A modern OpenGL graphics rendering project."
)

# ===========================================================
# 1. Compilation options and toolchain support (LSP friendly)
# ===========================================================
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Automatically copy compile_commands.json so that LSPs such as Clangd can function correctly.
# Do not copy compile_commands.json in MSVC to prevent CMake errors.
if (NOT CMAKE_GENERATOR MATCHES "Visual Studio")
	if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
		add_custom_target(
			CopyCompileCommands ALL
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
				${CMAKE_BINARY_DIR}/compile_commands.json
				${CMAKE_CURRENT_SOURCE_DIR}/compile_commands.json
			DEPENDS ${CMAKE_BINARY_DIR}/compile_commands.json
			COMMENT "Copy compile_commands.json to the project source directory."
		)
	endif()
endif()

# ==================================
# 2. Define external libraries paths
# ==================================
set(DEP_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ext)

set(GLMSVC_DIR ${DEP_DIR}/glfw-3.4_win64_msvc17)
set(GLMINGW_DIR ${DEP_DIR}/glfw-3.4_win64_mingw-w64)
set(GLAD_DIR ${DEP_DIR}/glad)
set(STB_DIR ${DEP_DIR}/stb)

# =======================
# 3. Collect source files
# =======================
# === GLAD source ===
set(GLAD_SOURCES ${GLAD_DIR}/src/glad.c)

# === Application source ===
file(GLOB_RECURSE APP_SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp"
) 

# ===========================
# 4. Create executable target
# ===========================
add_executable(${PROJECT_NAME} ${APP_SOURCES} ${GLAD_SOURCES})

# ==========================================
# 5. Include directories (Target-based in modern CMake)
# ==========================================
target_include_directories(${PROJECT_NAME} PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
    "${GLAD_DIR}/include"
    "${STB_DIR}/include"
)

# ==============================================================================
# 6. Conditional compilation and linking (handling multiple compilers/platforms)
# ==============================================================================
# === Platform-specific GLFW setup ===
if(MSVC)
    # --- Visual Studio / MSVC ---
    target_include_directories(${PROJECT_NAME} PRIVATE "${GLMSVC_DIR}/Debug/include")
    
    target_link_libraries(${PROJECT_NAME} PRIVATE
        "${GLMSVC_DIR}/$<$<CONFIG:Debug>:Debug>$<$<CONFIG:Release>:Release>/lib/glfw3.lib"
        opengl32
    )
	# Quoted paths ensure that spaces in the path are correctly recognized by CMake.
else()
    # --- MinGW / GCC / Clang ---
    target_include_directories(${PROJECT_NAME} PRIVATE "${GLMINGW_DIR}/include")
    
    target_link_libraries(${PROJECT_NAME} PRIVATE
        "${GLMINGW_DIR}/lib/libglfw3.a"
        opengl32
        gdi32
    )
endif()