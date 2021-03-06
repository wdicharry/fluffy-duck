cmake_minimum_required( VERSION 2.8 )

project(arm-kernel C ASM)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O1 -nostartfiles")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS} -O0 -g")

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(SOURCES
    main.c
    start.S
    cstartup.c
    sbrk.c
    gpio.c
    timer.c
    interrupt.c
    rpi_interrupt.c
    arm_timer.c
)

set(HEADERS
    gpio.h
    gpio_defs.h
    timer.h
    rpi_interrupt.h
    arm_timer.h
)

set(CMAKE_EXE_LINKER_FLAGS
    "${CMAKE_EXE_LINKER_FLAGS} -Wl,-T,${CMAKE_CURRENT_SOURCE_DIR}/rpi.x"
)

add_executable(
    arm-kernel
    ${SOURCES}
    ${HEADERS}
)

add_custom_command(
    TARGET arm-kernel
    COMMAND ${CMAKE_OBJCOPY} ./arm-kernel${EXECUTABLE_SUFFIX} -O binary kernel.img
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Convert ELF output to binary image"
)

set(KERNEL_ROOT_PARTITION "" CACHE PATH "FAT32 parition to deploy kernel to")
if (KERNEL_ROOT_PARTITION)
    add_custom_target(
        deploykernel
        COMMAND cmake -E copy kernel.img ${KERNEL_ROOT_PARTITION}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Copying kernel.img to ${KERNEL_ROOT_PARTITION}"
    )
endif()
