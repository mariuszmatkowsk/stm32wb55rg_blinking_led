cmake_minimum_required(VERSION 3.27)
project(blinking_led LANGUAGES C CXX ASM)

add_compile_definitions(STM32WB55xx)

set(CMAKE_EXPORT_COMPILE_COMMANDS On)

# MCU-specific flags
set(MCU_FLAGS -mcpu=cortex-m4 -mthumb)
set(FPU_FLAGS -mfloat-abi=softfp -mfpu=fpv4-sp-d16)

# Common compiler flags for both C and C++
set(COMMON_FLAGS
    ${MCU_FLAGS}
    ${FPU_FLAGS}
    -nostdlib
    -nostartfiles
    -ffast-math
    -fno-common
    -Wall
    -Wextra
    -Wno-strict-aliasing
    -Werror
    -Wno-unused-parameter
    -fsingle-precision-constant
    -ffunction-sections
    -fdata-sections
)

# Only use C-compatible flags globally
add_compile_options(${COMMON_FLAGS})

# ASM flags
string(JOIN " " CMAKE_ASM_FLAGS ${MCU_FLAGS})

# Linker script
set(LD_FILE "${CMAKE_CURRENT_LIST_DIR}/linker/stm32wb55xx_flash_cm4.ld")
set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} -Wl,--gc-sections)
set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} -fno-use-cxa-atexit -fno-exceptions -fno-rtti -Wl,-Map=${PROJECT_NAME}.map,-cref,--print-memory-usage)
set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} -T${LD_FILE})
string(JOIN " " CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS})

# Source files
set(SRC
    ${CMAKE_CURRENT_LIST_DIR}/main.cpp
    ${CMAKE_CURRENT_LIST_DIR}/system_stm32wbxx.c
    ${CMAKE_CURRENT_LIST_DIR}/startup_stm32wb55xx_cm4.s
)

# Create the executable
add_executable(${PROJECT_NAME}.elf ${SRC})

# Includes
target_include_directories(${PROJECT_NAME}.elf PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}
    ${CMAKE_CURRENT_LIST_DIR}/Include
    ${CMAKE_CURRENT_LIST_DIR}/CMSIS/Core/Include
)

# Apply C++-only flags to just C++ sources
target_compile_options(${PROJECT_NAME}.elf PRIVATE
    $<$<COMPILE_LANGUAGE:CXX>:
        -fno-exceptions
        -fno-rtti
        -fno-use-cxa-atexit
        -fno-threadsafe-statics
    >
)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Obinary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMENT "Generationg firmware .bin file"
)





