add_library(openal.hdll openal.c mojoal.c)
add_dependencies(openal.hdll sdl.hdll)

# option(WITH_STATIC_OPENAL "Link openal.hdll to static openal-soft" OFF) # GPL WARNING

set_as_hdll(openal)

message(STATUS "sdl2 include dir: ${SDL2_INCLUDE_DIR}")

target_link_libraries(openal.hdll
    libhl
    # ${OPENAL_LIBRARY}
    ${SDL2_LIBRARY}
)

target_include_directories(openal.hdll
    PRIVATE
    # ${OPENAL_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/AL
    ${SDL2_INCLUDE_DIR}
)

install(
    TARGETS
        openal.hdll
    DESTINATION ${HDLL_DESTINATION}
)
