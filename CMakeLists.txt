cmake_minimum_required(VERSION 3.25)
project(zelda3 VERSION 1.0 LANGUAGES C)
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()
find_package(SDL2)
add_executable(zelda3 
    ancilla.c
    attract.c
    audio.c
    config.c
    dungeon.c
    ending.c
    glsl_shader.c
    hud.c
    load_gfx.c
    main.c
    messaging.c
    misc.c
    nmi.c
    opengl.c
    overlord.c
    overworld.c
    player.c
    player_oam.c
    poly.c
    select_file.c
    spc_player.c
    sprite.c
    sprite_main.c
    tagalong.c
    tile_detect.c
    util.c
    zelda_cpu_infra.c
    zelda_rtl.c
    snes/apu.c
    snes/cart.c
    snes/cpu.c
    snes/dma.c
    snes/dsp.c
    snes/input.c
    snes/ppu.c
    snes/snes.c
    snes/snes_other.c
    snes/spc.c
    snes/tracing.c
    third_party/gl_core/gl_core_3_1.c
    third_party/opus-1.3.1-stripped/opus_decoder_amalgam.c
)

target_link_libraries(zelda3 PUBLIC SDL2::SDL2 PRIVATE m)

find_package(PythonInterp REQUIRED)
add_custom_command(OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/tables/dialogue.txt"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/tables"
    COMMAND "${PYTHON_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/tables/extract_resources.py" "${CMAKE_CURRENT_SOURCE_DIR}/tables/zelda3.sfc")
add_custom_command(OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/tables/zelda3_assets.dat"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/tables"
    COMMAND "${PYTHON_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/tables/compile_resources.py" "${CMAKE_CURRENT_SOURCE_DIR}/tables/zelda3.sfc"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/tables/dialogue.txt"
)
add_custom_target(generate_assets ALL
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/tables/zelda3_assets.dat" "${CMAKE_CURRENT_SOURCE_DIR}/tables/dialogue.txt")
