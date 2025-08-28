add_library(sdl.hdll
    sdl.c
    gl.c
)

include(FetchContent)
FetchContent_Declare(
    SDL2
    URL https://github.com/libsdl-org/SDL/releases/download/release-2.32.8/SDL2-2.32.8.tar.gz
    URL_HASH SHA256=0ca83e9c9b31e18288c7ec811108e58bac1f1bb5ec6577ad386830eac51c787e
    EXCLUDE_FROM_ALL
    OVERRIDE_FIND_PACKAGE
)
option(SDL_SHARED "Build a shared version of the library" OFF)
option(SDL_STATIC "Build a static version of the library" ON)
option(SDL_STATIC_PIC "" ON)
option(SDL_TEST "Build the SDL2_test library" OFF)
FetchContent_MakeAvailable(SDL2)
find_package(SDL2 REQUIRED)

set_as_hdll(sdl)
target_include_directories(sdl.hdll
    PUBLIC
    ${INCLUDES_BASE_DIR}/gl
)
target_link_libraries(sdl.hdll
    PUBLIC
    libhl
    SDL2::SDL2-static
)

if(WIN32)
    target_link_libraries(sdl.hdll PRIVATE OpenGL32)
endif()

if(ANDROID)
    target_link_libraries(sdl.hdll PRIVATE m OpenSLES dl log android GLESv1_CM GLESv2 EGL GLESv3)
endif()

if((APPLE OR UNIX) AND NOT ANDROID)
    find_package(OpenGL REQUIRED)
    target_include_directories(sdl.hdll
        PRIVATE
        ${OPENGL_INCLUDE_DIR}
    )
    target_link_libraries(sdl.hdll
        PRIVATE
        ${OPENGL_gl_LIBRARY}
    )
endif()


install(
    TARGETS sdl.hdll
    DESTINATION ${HDLL_DESTINATION}
)
