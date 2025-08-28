add_library(openal.hdll openal.c mojoal.c)
add_dependencies(openal.hdll sdl.hdll)

# option(WITH_STATIC_OPENAL "Link openal.hdll to static openal-soft" OFF) # GPL WARNING

set_as_hdll(openal)

find_package(SDL2 REQUIRED)
target_link_libraries(openal.hdll
    libhl
    SDL2::SDL2-static
)

target_include_directories(openal.hdll
    PRIVATE
    # ${OPENAL_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/AL
)

target_compile_definitions(openal.hdll PRIVATE -DOPENAL_STATIC)

install(
    TARGETS
        openal.hdll
    DESTINATION ${HDLL_DESTINATION}
)
