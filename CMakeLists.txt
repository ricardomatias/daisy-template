cmake_policy(SET CMP0048 NEW)
cmake_minimum_required(VERSION 3.20)

file(TO_CMAKE_PATH $ENV{HOME} USER_HOME)
file(TO_CMAKE_PATH $ENV{AUDIO_LIBS_DIR} AUDIO_LIBS_DIR)

# IMPORTANT - CHANGE THIS PATH TO YOUR GCC INSTALLATION
set(TOOLCHAIN_PREFIX "${USER_HOME}/scoop/apps/gcc-arm-none-eabi/current")
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/cmake/stm32h750xx.cmake)

project(daisy_example VERSION 0.1 LANGUAGES C CXX ASM)
# This is the target name of the firmware binary
set(FIRMWARE_NAME daisy_program)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# OPENOCD
set(CHIPSET stm32h7x)
set(OCD openocd)

# IMPORTANT - CHANGE THIS PATH TO YOUR OPENOCD SCRIPTS FOLDER
set(OCD_DIR "${USER_HOME}/scoop/apps/openocd/current/share/openocd/scripts")
set(PGM_DEVICE interface/stlink.cfg)
set(OCDFLAGS -f ${PGM_DEVICE} -f target/${CHIPSET}.cfg)
set(OCD_PROGRAM ${OCD} -s ${OCD_DIR} ${OCDFLAGS} -c "program ${FIRMWARE_NAME}.elf verify reset exit")

add_subdirectory(${AUDIO_LIBS_DIR}/libDaisy ${CMAKE_CURRENT_BINARY_DIR}/libDaisy)
add_subdirectory(${AUDIO_LIBS_DIR}/DaisySP ${CMAKE_CURRENT_BINARY_DIR}/DaisySP)

set(LINKER_SCRIPT ${AUDIO_LIBS_DIR}/libDaisy/core/STM32H750IB_flash.lds)

add_executable(${FIRMWARE_NAME} src/main.cpp)

target_compile_options(daisy
    PUBLIC
    -Os
    -flto
)

target_link_libraries(${FIRMWARE_NAME}
    PRIVATE
    daisy
    DaisySP
)

set_target_properties(${FIRMWARE_NAME} PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
    SUFFIX ".elf"
    LINK_DEPENDS ${LINKER_SCRIPT}
)

target_link_options(${FIRMWARE_NAME} PUBLIC
    -T ${LINKER_SCRIPT}
    -Wl,-Map=${FIRMWARE_NAME}.map,--cref
    -Wl,--check-sections
    -Wl,--unresolved-symbols=report-all
    -Wl,--warn-common
    -Wl,--warn-section-align
    -Wl,--print-memory-usage
    -Wl,--gc-sections
)

target_compile_options(${FIRMWARE_NAME}
    PUBLIC
    -fno-builtin
    -fno-exceptions
    -ffunction-sections
    -fdata-sections
    -fomit-frame-pointer
    -finline-functions
    -Wall
    -Wno-attributes
    -Wno-strict-aliasing
    -Wno-maybe-uninitialized
    -Wno-missing-attributes
    -Wno-stringop-overflow
    $<$<COMPILE_LANGUAGE:CXX>:-Wno-register>
    $<$<COMPILE_LANGUAGE:ASM>:-x assembler-with-cpp>
)

add_custom_command(TARGET ${FIRMWARE_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY}
    ARGS -O ihex
    -S ${FIRMWARE_NAME}.elf
    ${FIRMWARE_NAME}.hex
    BYPRODUCTS
    ${FIRMWARE_NAME}.hex
    COMMENT "Generating HEX image"
    VERBATIM)

add_custom_command(TARGET ${FIRMWARE_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY}
    ARGS -O binary
    -S ${FIRMWARE_NAME}.elf
    ${FIRMWARE_NAME}.bin
    BYPRODUCTS
    ${FIRMWARE_NAME}.bin
    COMMENT "Generating binary image"
    VERBATIM)

add_custom_target(upload
    COMMAND ${OCD_PROGRAM}
)