cmake_minimum_required(VERSION 3.16)

# Pull in SDK (must be before project)
include(cmake/pico_sdk_import.cmake)

project(applevga)
set(CMAKE_C_STANDARD 11)

# Set default build type to Release if not specified
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

set(APPLE_MODEL "IIPLUS" CACHE STRING "The model of Apple II to support")
message("Target Apple model is ${APPLE_MODEL}")
add_definitions(-DAPPLE_MODEL_${APPLE_MODEL}=1)

# Set the output program name based on APPLE_MODEL
if(APPLE_MODEL STREQUAL "IIPLUS")
    set(PROGRAM_NAME "apple-iiplus-vga")
elseif(APPLE_MODEL STREQUAL "IIE")
    set(PROGRAM_NAME "apple-iie-vga")
else()
    message(FATAL_ERROR "Invalid APPLE_MODEL: ${APPLE_MODEL}. Must be IIPLUS or IIE.")
endif()

pico_sdk_init()

add_compile_options(-Wall -Wextra -Wnull-dereference -Wno-unused-parameter -Wno-unused-function)

add_executable(${PROGRAM_NAME})
pico_set_binary_type(${PROGRAM_NAME} copy_to_ram)
pico_set_linker_script(${PROGRAM_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/memmap_copy_to_ram_custom.ld)

pico_generate_pio_header(${PROGRAM_NAME}
    ${CMAKE_CURRENT_SOURCE_DIR}/vga.pio)
pico_generate_pio_header(${PROGRAM_NAME}
    ${CMAKE_CURRENT_SOURCE_DIR}/abus.pio)

file(GLOB TEXTFONT_SRC_FILES textfont/*.c)
target_sources(${PROGRAM_NAME} PUBLIC
    main.c
    abus.c
    buffers.c
    colors.c
    config.c
    device_regs.c
    render.c
    render_hires.c
    render_lores.c
    render_testpat.c
    render_text.c
    ${TEXTFONT_SRC_FILES}
    vga.c
    videx_vterm.c
)
target_include_directories(${PROGRAM_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(${PROGRAM_NAME} PUBLIC
    pico_multicore
    pico_stdlib
    hardware_dma
    hardware_flash
    hardware_pio
)

pico_add_extra_outputs(${PROGRAM_NAME})