add_library(uv.hdll
    uv.c
)

include(FetchContent)
FetchContent_Declare(
    libuv
    URL https://github.com/libuv/libuv/archive/refs/tags/v1.51.0.tar.gz
    URL_HASH SHA256=27e55cf7083913bfb6826ca78cde9de7647cded648d35f24163f2d31bb9f51cd
    EXCLUDE_FROM_ALL
    OVERRIDE_FIND_PACKAGE
)
set(BUILD_SHARED_LIBS OFF)
if(LINUX)
    list(APPEND uv_cflags -fPIC)
endif()
option(LIBUV_BUILD_SHARED "" OFF)
option(LIBUV_BUILD_TESTS "" OFF)
FetchContent_MakeAvailable(libuv)
find_package(libuv REQUIRED)
if(WIN32)
    target_compile_definitions(uv_a
        PRIVATE
        BUILDING_UV_SHARED=1
    )
endif()

set_as_hdll(uv)
target_link_libraries(uv.hdll
    PRIVATE
    libhl
    libuv::libuv
)

if(WIN32)
    target_link_libraries(uv.hdll
        PRIVATE
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
