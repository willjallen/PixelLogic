cmake_minimum_required(VERSION 3.12)
project(pixellogic)

# include all files in the CMake directory
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake;${CMAKE_MODULE_PATH}")

# set global C++ options
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_C_STANDARD 11)
# set(CMAKE_VERBOSE_MAKEFILE ON)

# include_directories(shared/include)

# # compile in the git version file
# include(GetGitRevisionDescription)
# git_describe(VERSION --tags --dirty=-d)

# get_git_head_revision(GIT_REFSPEC GIT_HASH)

# string(REGEX REPLACE "^v([0-9]+)\\..*" "\\1" VERSION_MAJOR "${VERSION}")
# string(REGEX REPLACE "^v[0-9]+\\.([0-9]+).*" "\\1" VERSION_MINOR "${VERSION}")
# string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" VERSION_PATCH "${VERSION}")
# string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.[0-9]+(.*)" "\\1" VERSION_SHA1 "${VERSION}")
# set(VERSION_SHORT "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

# execute_process(COMMAND git describe --tags --dirty=d OUTPUT_VARIABLE VERSION_TAG OUTPUT_STRIP_TRAILING_WHITESPACE)

# configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/version.cpp.in
#     ${CMAKE_CURRENT_BINARY_DIR}/version.cpp)
# set(version_file "${CMAKE_CURRENT_BINARY_DIR}/version.cpp")

# include(CMakeRC)

# find_package(CURL COMPONENTS HTTPS SSL IPv6)

###################################################################################################
#### sources for app
# build main executable and all modules
add_executable(pixellogic
    src/main.cpp
# resources and other misc stuff
    ${version_file}
)

target_compile_definitions(pixellogic PRIVATE CUBE_CLIENT)

# optimize to have at least AVX
SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -mavx")

# warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-anon-enum-enum-conversion -Wno-defaulted-function-deleted -Wno-gnu-zero-variadic-macro-arguments -Wno-reorder-ctor -Wno-c99-extensions")

# shared code
target_link_libraries(pixellogic PRIVATE shared shared_platform)

target_include_directories(pixellogic PUBLIC src)

###################################################################################################
#### libraries
# do NOT build shared libraries. thanks DLL hell (I wonder what DLL ever did to deserve this)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)

# glbinding (loading OpenGL functions)
target_link_directories(pixellogic PRIVATE libs/glbinding/build)
target_link_libraries(pixellogic PRIVATE glbinding)
target_include_directories(pixellogic PRIVATE libs/glbinding/source/glbinding/include)
target_include_directories(pixellogic PRIVATE libs/glbinding/build/source/glbinding/include)
target_include_directories(pixellogic PRIVATE libs/glbinding/build/source/include)
target_include_directories(pixellogic PRIVATE libs/glbinding/3rdparty/KHR/include)

# SDL2 (we build a dynamic version on macOS, static elsewhere)
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(SDL_STATIC OFF CACHE BOOL "" FORCE)
    set(SDL_SHARED ON CACHE BOOL "" FORCE)

    # do not link yet. we'll do this further down when we build the bundle to get the correct rpath
    target_link_libraries(pixellogic PUBLIC SDL2main SDL2)
else()
    set(SDL_STATIC ON CACHE BOOL "" FORCE)
    set(SDL_SHARED OFF CACHE BOOL "" FORCE)

    target_link_libraries(pixellogic PUBLIC SDL2main SDL2 SDL2-static)
endif()

add_subdirectory(libs/sdl2)
include_directories(libs/sdl2/include)

###################################################################################################
