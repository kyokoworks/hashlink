add_library(uv.hdll
    uv.c
)

set(LibUV_CMAKE_ARGS
    -DLIBUV_BUILD_SHARED=OFF
    -DLIBUV_BUILD_TESTS=OFF
    -DBUILD_TESTING=OFF
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
)

if(ANDROID)
    list(APPEND LibUV_CMAKE_ARGS
        -DANDROID_PLATFORM=${ANDROID_PLATFORM}
        -DANDROID_ABI=${CMAKE_ANDROID_ARCH_ABI}
    )    
endif()

if(CMAKE_TOOLCHAIN_FILE)
    list(APPEND LibUV_CMAKE_ARGS
        -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
    )
endif()

if(WIN32)
    set(LibUV_INSTALL_BYPRODUCTS <INSTALL_DIR>/${CMAKE_INSTALL_LIBDIR}/libuv.lib)
else()
    set(LibUV_INSTALL_BYPRODUCTS <INSTALL_DIR>/${CMAKE_INSTALL_LIBDIR}/libuv.a)
endif()

ExternalProject_Add(libuv-project
    URL https://github.com/libuv/libuv/archive/refs/tags/v1.51.0.tar.gz
    URL_HASH SHA256=27e55cf7083913bfb6826ca78cde9de7647cded648d35f24163f2d31bb9f51cd
    CMAKE_ARGS 
        -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
        ${LibUV_CMAKE_ARGS}
    # INSTALL_BYPRODUCTS in CMake 3.26+
    INSTALL_BYPRODUCTS ${LibUV_INSTALL_BYPRODUCTS}
    DOWNLOAD_EXTRACT_TIMESTAMP true
)
ExternalProject_Get_Property(libuv-project INSTALL_DIR)


add_library(libuv STATIC IMPORTED)
set_target_properties(libuv PROPERTIES IMPORTED_LOCATION ${INSTALL_DIR}/${CMAKE_INSTALL_LIBDIR}/libuv.a)

set(LibUV_INCLUDE_DIRS ${INSTALL_DIR}/${CMAKE_INSTALL_INCLUDEDIR})
set(LibUV_LIBRARIES libuv)

add_dependencies(libuv libuv-project)

set_as_hdll(uv)
target_link_libraries(uv.hdll
    libhl
)
target_include_directories(uv.hdll
    PRIVATE
    ${LibUV_INCLUDE_DIRS}
)
target_link_libraries(uv.hdll
    ${LibUV_LIBRARIES}
)

if(WIN32)
    target_link_libraries(uv.hdll
        psapi
        iphlpapi
        userenv
    )
endif()

install(
    TARGETS
        uv.hdll
    DESTINATION ${HDLL_DESTINATION}
)
