cmake_minimum_required (VERSION 3.0)

project (iwasm)

set (CMAKE_VERBOSE_MAKEFILE ON)

set (WAMR_BUILD_PLATFORM "freertos")

set (WAMR_BUILD_TARGET "THUMB")

set (WAMR_BUILD_INTERP 1)
set (WAMR_BUILD_AOT 0)
set (WAMR_BUILD_LIBC_BUILTIN 1)
set (WAMR_BUILD_LIBC_WASI 0)

set (WAMR_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/wasm-micro-runtime)

set (CMAKE_C_COMPILER "arm-none-eabi-gcc")
set (TOOLCHAIN_PREFIX "arm-none-eabi-")

include (${WAMR_ROOT_DIR}/build-scripts/runtime_lib.cmake)

add_library(iwasm_static STATIC ${WAMR_RUNTIME_LIB_SOURCE})
set_target_properties (iwasm_static PROPERTIES OUTPUT_NAME vmlib)
target_include_directories(iwasm_static INTERFACE ${WAMR_ROOT_DIR}/core/iwasm/include)
target_link_libraries (iwasm_static INTERFACE ${LLVM_AVAILABLE_LIBS} ${UV_A_LIBS} -lm -ldl -lpthread)
if (WAMR_BUILD_WASM_CACHE EQUAL 1)
  target_link_libraries(iwasm_static INTERFACE boringssl_crypto)
endif ()

install (TARGETS iwasm_static ARCHIVE DESTINATION lib)

set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16 -g3")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-math-errno -DARM_MATH_CM4 -D__FPU_PRESENT=1 -mfp16-format=ieee")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-array-bounds -Wno-stringop-overread")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-stringop-overflow")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DSTM32F4XX -DSTM32F40_41xxx -DHSE_VALUE=8000000 -DUSE_STDPERIPH_DRIVER")

set (CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16 -g3")
set (CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -fno-math-errno -DARM_MATH_CM4 -D__FPU_PRESENT=1 -mfp16-format=ieee")

set (CRAZYFLIE_FIRMWARE "crazyflie-firmware-wasm")
set (FREERTOS "${CRAZYFLIE_FIRMWARE}/vendor/FreeRTOS")

set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${FREERTOS}/include")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${FREERTOS}/portable/GCC/ARM_CM4F")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${CRAZYFLIE_FIRMWARE}/src/config")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${CRAZYFLIE_FIRMWARE}/src/drivers/interface")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${CRAZYFLIE_FIRMWARE}/src/hal/interface")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I${CRAZYFLIE_FIRMWARE}/src/utils/interface")
