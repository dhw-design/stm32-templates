#Минимальная версия cmake 3.8 необходима для поддержки стандарта C++17
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

#Путь к папке с компилятором (слеш в конце обязателен)
SET(TOOLCHAIN_PREFIX "C:/gnuARM/ArmGCC/")

#Путь к папке make (слеш в конце обязателен)
SET(MAKE_PREFIX "C:/gnuARM/BuildTools/bin/")

#Скрипт для линковки. В makefile, создаваемом кубом, значение указано в переменной LDSCRIPT
set(LINKER_SCRIPT "STM32F103VETx_FLASH.ld")

SET(TARGET_TRIPLET "arm-none-eabi")

#Папка для сборки Release
SET(RELEASE_DIR "Release")

#Папка для сборки Debug
SET(DEBUG_DIR "Debug")

include("cmake/stm32-gcc.cmake")

#Название проекта
project(cmake-template)

enable_language(C ASM)

# Флаги контроллера. В makefile, создаваемом кубом, значение берется из переменной CPU
set(CPU_OPTIONS
    -mthumb 
    -mcpu=cortex-m3
    )

#Флаги FPU (при наличии). В makefile, создаваемом кубом, значение берется из переменных FPU и FLOAT-ABI	
set(FPU_OPTIONS
	
	)

# Варианты оптимизации (-O0 -O1 -O2 -O3 -Os -Ofast -Og)
#Дополнительные флаги компиляции сборки Debug
set(CMAKE_C_FLAGS_DEBUG "-Og -g -DDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "-Og -g")

#Дополнительные флаги компиляции сборки Release
set(CMAKE_C_FLAGS_RELEASE "-O2")
set(CMAKE_CXX_FLAGS_RELEASE "-O2")

#В makefile, создаваемом кубом, дефайны описываются в переменной C_DEFS
add_definitions(
    -DUSE_FULL_LL_DRIVER
    -DSTM32F103xE
)

#Список путей заголовочных файлов
include_directories(
    ${CMAKE_SOURCE_DIR}
)


link_directories(
    ${CMAKE_SOURCE_DIR}
)

#Флаги компиляции
add_compile_options(
    ${CPU_OPTIONS}
    -Wall
    -Wcast-align
    -Wunused-parameter
    -Wmissing-declarations
    -fdata-sections
    -ffunction-sections
    $<$<COMPILE_LANGUAGE:CXX>:-fno-exceptions>
    $<$<COMPILE_LANGUAGE:CXX>:-fno-rtti>
)

#Флаги линковщика
add_link_options(
    ${CPU_OPTIONS}
    -T${LINKER_SCRIPT}
    -Wl,-Map=${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.map,--cref
    -Wl,--gc-sections,--print-memory-usage
    -specs=nano.specs
)

#Пример добавления ассемблерного файла в проект
set_source_files_properties(startup_stm32f103xe.s PROPERTIES COMPILE_FLAGS "-x assembler-with-cpp")
# list of modules to build final firmware (without extension .c or .cpp)

#Добавление .c файлов для компиляции. Каждый новый файл указывается в новой строке. В makefile, создаваемом кубом, данные хранятся в переменной C_SOURCES
set(USER_SRC
    Core/Src/main.c
    
)

add_executable(${PROJECT_NAME}
    ${USER_SRC}
    startup_stm32f103xe.s
)



#Добавляем дополнительные библиотеки
target_link_libraries(${PROJECT_NAME}
    m
    c
    nosys
)

#Добавляем генерацию hex файла
add_custom_command(TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Oihex ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.hex
)

#Добавляем генерацию bin файла
add_custom_command(TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -Obinary ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.bin
)

#Добавляем генерацию листинга
add_custom_command(TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_OBJDUMP} -D ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME} > ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.lst
)


add_custom_command(OUTPUT ${PROJECT_NAME}.sym
    DEPENDS ${PROJECT_NAME}
    COMMAND ${CMAKE_NM} -C -l -n -S ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME} > ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${PROJECT_NAME}.sym
)

add_custom_target(sym
    DEPENDS ${PROJECT_NAME}.sym
)
