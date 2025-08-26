add_library(sdl.hdll
    sdl.c
    gl.c
)

if (ANDROID OR UNIX)
    set(SDL2_CMAKE_ARGS 
        -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
        -DBUILD_SHARED_LIBS=OFF
        -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
        -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
        -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    )
    if(ANDROID)
        list(APPEND SDL2_CMAKE_ARGS
            -DANDROID_PLATFORM=${ANDROID_PLATFORM}
            -DANDROID_ABI=${CMAKE_ANDROID_ARCH_ABI}
        )    
    endif()

	ExternalProject_Add(sdl2-project
        URL https://github.com/libsdl-org/SDL/releases/download/release-2.32.8/SDL2-2.32.8.tar.gz
        URL_HASH SHA256=0ca83e9c9b31e18288c7ec811108e58bac1f1bb5ec6577ad386830eac51c787e
        CMAKE_ARGS 
            -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
            ${SDL2_CMAKE_ARGS}
        # INSTALL_BYPRODUCTS in CMake 3.26+
        INSTALL_BYPRODUCTS <INSTALL_DIR>/${CMAKE_INSTALL_LIBDIR}/libSDL2.a
        DOWNLOAD_EXTRACT_TIMESTAMP true
    )
    ExternalProject_Get_Property(sdl2-project INSTALL_DIR)

    add_library(sdl2 SHARED IMPORTED)
    set_target_properties(sdl2 PROPERTIES IMPORTED_LOCATION ${INSTALL_DIR}/${CMAKE_INSTALL_LIBDIR}/libSDL2.a)

    set(SDL2_INCLUDE_DIR ${INSTALL_DIR}/${CMAKE_INSTALL_INCLUDEDIR}/SDL2 PARENT_SCOPE)
    set(SDL2_LIBRARY sdl2)

    target_include_directories(sdl2 INTERFACE ${SDL2_INCLUDE_DIR})

    add_dependencies(sdl2 sdl2-project)
else()
    if(WIN32)
        set(SDL2_PATH ${INCLUDES_BASE_DIR}/sdl)
    endif()

    find_package(SDL2 REQUIRED)
endif()

set_as_hdll(sdl)
target_include_directories(sdl.hdll
    PUBLIC
    ${SDL2_INCLUDE_DIR}
    ${INCLUDES_BASE_DIR}/gl
)
target_link_libraries(sdl.hdll
    PUBLIC
    libhl
    ${SDL2_LIBRARY}
)

if(WIN32)
    target_link_libraries(sdl.hdll
        PRIVATE
        winmm
        opengl32
    )
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
