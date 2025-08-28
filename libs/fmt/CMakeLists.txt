set(MINIMP3_INCLUDE_DIR ${INCLUDES_BASE_DIR}/minimp3)
set(MIKKTSPACE_INCLUDE_DIR ${INCLUDES_BASE_DIR}/mikktspace)

add_library(fmt.hdll
    fmt.c
    sha1.c
    dxt.c
    mikkt.c
    ${MIKKTSPACE_INCLUDE_DIR}/mikktspace.c
)

set(PNG_INCLUDE_DIRS ${INCLUDES_BASE_DIR}/png)
set(VORBIS_INCLUDE_DIR ${INCLUDES_BASE_DIR}/vorbis)
set(ZLIB_INCLUDE_DIR ${INCLUDES_BASE_DIR}/zlib)

add_library(zlib STATIC 
    ${INCLUDES_BASE_DIR}/zlib/adler32.c
    ${INCLUDES_BASE_DIR}/zlib/crc32.c
    ${INCLUDES_BASE_DIR}/zlib/deflate.c
    ${INCLUDES_BASE_DIR}/zlib/inffast.c
    ${INCLUDES_BASE_DIR}/zlib/inflate.c
    ${INCLUDES_BASE_DIR}/zlib/inftrees.c
    ${INCLUDES_BASE_DIR}/zlib/trees.c
    ${INCLUDES_BASE_DIR}/zlib/zutil.c
)
target_include_directories(zlib PUBLIC ${ZLIB_INCLUDE_DIR})
target_compile_definitions(zlib PUBLIC -DZ_SOLO)
set_property(TARGET zlib PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(png STATIC 
    ${INCLUDES_BASE_DIR}/png/png.c
    ${INCLUDES_BASE_DIR}/png/pngerror.c
    ${INCLUDES_BASE_DIR}/png/pngget.c
    ${INCLUDES_BASE_DIR}/png/pngmem.c
    ${INCLUDES_BASE_DIR}/png/pngpread.c
    ${INCLUDES_BASE_DIR}/png/pngread.c
    ${INCLUDES_BASE_DIR}/png/pngrio.c
    ${INCLUDES_BASE_DIR}/png/pngrtran.c
    ${INCLUDES_BASE_DIR}/png/pngrutil.c
    ${INCLUDES_BASE_DIR}/png/pngset.c
    ${INCLUDES_BASE_DIR}/png/pngtrans.c
    ${INCLUDES_BASE_DIR}/png/pngwio.c
    ${INCLUDES_BASE_DIR}/png/pngwrite.c
    ${INCLUDES_BASE_DIR}/png/pngwtran.c
    ${INCLUDES_BASE_DIR}/png/pngwutil.c
)
target_include_directories(png PUBLIC ${PNG_INCLUDE_DIRS})
target_link_libraries(png PUBLIC zlib)
set_property(TARGET png PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(vorbis STATIC
    ${INCLUDES_BASE_DIR}/vorbis/bitrate.c
    ${INCLUDES_BASE_DIR}/vorbis/bitwise.c
    ${INCLUDES_BASE_DIR}/vorbis/block.c
    ${INCLUDES_BASE_DIR}/vorbis/codebook.c
    ${INCLUDES_BASE_DIR}/vorbis/envelope.c
    ${INCLUDES_BASE_DIR}/vorbis/floor0.c
    ${INCLUDES_BASE_DIR}/vorbis/floor1.c
    ${INCLUDES_BASE_DIR}/vorbis/framing.c
    ${INCLUDES_BASE_DIR}/vorbis/info.c
    ${INCLUDES_BASE_DIR}/vorbis/lookup.c
    ${INCLUDES_BASE_DIR}/vorbis/lpc.c
    ${INCLUDES_BASE_DIR}/vorbis/lsp.c
    ${INCLUDES_BASE_DIR}/vorbis/mapping0.c
    ${INCLUDES_BASE_DIR}/vorbis/mdct.c
    ${INCLUDES_BASE_DIR}/vorbis/psy.c
    ${INCLUDES_BASE_DIR}/vorbis/registry.c
    ${INCLUDES_BASE_DIR}/vorbis/res0.c
    ${INCLUDES_BASE_DIR}/vorbis/sharedbook.c
    ${INCLUDES_BASE_DIR}/vorbis/smallft.c
    ${INCLUDES_BASE_DIR}/vorbis/synthesis.c
    ${INCLUDES_BASE_DIR}/vorbis/vorbisfile.c
    ${INCLUDES_BASE_DIR}/vorbis/window.c
)
target_include_directories(vorbis PUBLIC ${VORBIS_INCLUDE_DIR})
set_property(TARGET vorbis PROPERTY POSITION_INDEPENDENT_CODE ON)

set(TurboJpeg_CMAKE_ARGS
    -DENABLE_SHARED=OFF
    -DENABLE_STATIC=ON
    -DBUILD_TESTING=OFF
    -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
    -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
)

if(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|arm64")
    # TODO: enable
    target_compile_definitions(png PRIVATE PNG_ARM_NEON_OPT=0)
endif()

if (ANDROID)
    list(APPEND TurboJpeg_CMAKE_ARGS 
        -DANDROID_PLATFORM=${ANDROID_PLATFORM}
        -DANDROID_ABI=${CMAKE_ANDROID_ARCH_ABI}
    )
endif()

if (MSVC)
    set(TURBOJPEG_LIB_PATH lib/turbojpeg-static.lib)
else()
    set(TURBOJPEG_LIB_PATH libturbojpeg.a)
endif()

ExternalProject_Add(turbojpeg-project
    URL https://github.com/libjpeg-turbo/libjpeg-turbo/releases/download/3.1.1/libjpeg-turbo-3.1.1.tar.gz
    URL_HASH SHA256=aadc97ea91f6ef078b0ae3a62bba69e008d9a7db19b34e4ac973b19b71b4217c
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
        ${TurboJpeg_CMAKE_ARGS}
    # INSTALL_BYPRODUCTS in CMake 3.26+
    INSTALL_BYPRODUCTS <INSTALL_DIR>/${CMAKE_INSTALL_LIBDIR}/${TURBOJPEG_LIB_PATH}
    DOWNLOAD_EXTRACT_TIMESTAMP true
)

ExternalProject_Get_Property(turbojpeg-project INSTALL_DIR)

add_library(turbojpeg STATIC IMPORTED)
set_target_properties(turbojpeg PROPERTIES IMPORTED_LOCATION ${INSTALL_DIR}/${CMAKE_INSTALL_LIBDIR}/${TURBOJPEG_LIB_PATH})

set(TurboJPEG_INCLUDE_DIRS ${INSTALL_DIR}/${CMAKE_INSTALL_INCLUDEDIR})
set(TurboJPEG_LIBRARIES turbojpeg)

add_dependencies(turbojpeg turbojpeg-project)

set_as_hdll(fmt)

target_include_directories(fmt.hdll
    PRIVATE
    ${TurboJPEG_INCLUDE_DIRS}
    ${MINIMP3_INCLUDE_DIR}
    ${MIKKTSPACE_INCLUDE_DIR}
)

target_link_libraries(fmt.hdll
    PUBLIC
    libhl
    ${TurboJPEG_LIBRARIES}
    zlib
    png
    vorbis
)

install(
    TARGETS
        fmt.hdll
    DESTINATION ${HDLL_DESTINATION}
)
