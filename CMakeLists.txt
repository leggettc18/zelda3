cmake_minimum_required(VERSION 3.19)
project(zelda3 VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 11 CACHE STRING "The C++ Standard to use")

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY VS_STARTUP_PROJECT zelda3)

set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_SYSTEM_NAME MATCHES "NintendoSwitch")
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

option(USE_AUTO_VCPKG "Whether or not to automatically download and bootstrap Vcpkg and install required dependencies.
When OFF (default), you will need to supply your own copy of Vcpkg, or supply the dependencies some other way (such as
a different package manager or manually supplied paths).")

# Vcpkg does not work for Nintendo Switch, just Windows, Mac, and Linux.
if (USE_AUTO_VCPKG AND NOT CMAKE_SYSTEM_NAME MATCHES "NintendoSwitch")
	include(cmake/automate-vcpkg.cmake)
	
	vcpkg_bootstrap()
	vcpkg_install_packages(SDL2)
endif()

find_package(SDL2)

set(ZELDA
	"ancilla.c"
    "attract.c"
    "audio.c"
    "config.c"
    "dungeon.c"
    "ending.c"
    "glsl_shader.c"
    "hud.c"
    "load_gfx.c"
    "main.c"
    "messaging.c"
    "misc.c"
    "nmi.c"
    "opengl.c"
    "overlord.c"
    "overworld.c"
    "player.c"
    "player_oam.c"
    "poly.c"
    "select_file.c"
    "spc_player.c"
    "sprite.c"
    "sprite_main.c"
    "tagalong.c"
    "tile_detect.c"
    "util.c"
    "zelda_cpu_infra.c"
    "zelda_rtl.c"
	"ancilla.h"
	"assets.h"
	"attract.h"
	"audio.h"
	"config.h"
	"dungeon.h"
	"ending.h"
	"features.h"
	"glsl_shader.h"
	"hud.h"
	"load_gfx.h"
	"messaging.h"
	"misc.h"
	"nmi.h"
	"overlord.h"
	"overworld.h"
	"player.h"
	"player_oam.h"
	"poly.h"
	"select_file.h"
	"spc_player.h"
	"sprite.h"
	"sprite_main.h"
	"tagalong.h"
	"tile_detect.h"
	"types.h"
	"util.h"
	"variables.h"
	"zelda_cpu_infra.h"
	"zelda_rtl.h"
)

source_group("Zelda" FILES ${ZELDA})

set(SNES
	"snes/apu.h"
	"snes/cart.h"
	"snes/cpu.h"
	"snes/dma.h"
	"snes/dsp.h"
	"snes/dsp_regs.h"
	"snes/input.h"
	"snes/ppu.h"
	"snes/saveload.h"
	"snes/snes_regs.h"
	"snes/spc.h"
	"snes/tracing.h"
	"snes/apu.c"
    "snes/cart.c"
    "snes/cpu.c"
    "snes/dma.c"
    "snes/dsp.c"
    "snes/input.c"
    "snes/ppu.c"
    "snes/snes.c"
    "snes/snes_other.c"
    "snes/spc.c"
    "snes/tracing.c"
)

source_group("Snes" FILES ${SNES})

set(THIRDPARTY
	"third_party/gl_core/gl_core_3_1.h"
	"third_party/gl_core/gl_core_3_1.c"
)

source_group("third_party" FILES ${THIRDPARTY})

set(THIRDPARTY_OPUS
	"third_party/opus-1.3.1-stripped/opus_decoder_amalgam.c"
)

source_group("third_party\\Opus" FILES ${THIRDPARTY_OPUS})

if(CMAKE_SYSTEM_NAME MATCHES "Windows")
	set(PLATFORM
		"platform/win32/volume_control.h"
		"platform/win32/volume_control.c"
	)
	source_group("Platform" FILES ${PLATFORM})
	set(RESOURCES
		"platform/win32/zelda3.rc"
		"platform/win32/resource.h" 
		"platform/win32/triforce.ico"
	)
	source_group("Resource Files" FILES ${RESOURCES})
endif()
	

add_executable(zelda3 
	${ZELDA}
	${SNES}
	${THIRDPARTY}
	${THIRDPARTY_OPUS}
	${PLATFORM}
	${RESOURCES}
)

configure_file(zelda3.ini zelda3.ini COPYONLY)
configure_file(tables/zelda3_assets.dat zelda3_assets.dat COPYONLY)

target_link_libraries(zelda3 SDL2::SDL2 $<$<NOT:$<PLATFORM_ID:Windows>>:m>)

if(CMAKE_SYSTEM_NAME MATCHES "NintendoSwitch")
    nx_generate_nacp(zelda3.nacp
        NAME "The Legend of Zelda: A Link to the Past"
        AUTHOR "snesrev"
        VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")
    nx_create_nro(zelda3
        NACP zelda3.nacp
        ICON ${CMAKE_CURRENT_SOURCE_DIR}/platform/switch/icon.jpg)
    set_target_properties(zelda3 PROPERTIES LINKER_LANGUAGE CXX)
endif()


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
