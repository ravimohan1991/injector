#[[
    Abstractions and Models are NOT at WAR!
                                            - Cobwoy's Talisman
    But Abstractions don't care and Models can't understand!!
                                            - Lul, Practicality
 ]]

cmake_minimum_required (VERSION 3.23)

# Agnostic Injector
project(agInjector)

if(LINUX)
file(GLOB AGIN_HEADERS src/linux/injector_internal.h)
file(GLOB AGIN_SOURCE src/linux/*.c)
file(GLOB AGIN_MISC src/linux/shellcode.S)
# Solution from https://stackoverflow.com/q/15132185 thread
enable_language(C ASM)
SET(ASM_OPTIONS "-x assembler-with-cpp")
SET(CMAKE_ASM_FLAGS "-Wall -fPIC ${ASM_OPTIONS}")
endif()

# First test with executable
add_library(agInjector SHARED ${AGIN_HEADERS} ${AGIN_SOURCE}
${AGIN_MISC})

# INTERFACE (Something that Consumers require but Producers don't)
# Since I am a library coder here, I am the Producer
# Rest of the Engine is Consumer here

target_include_directories(agInjector
    PRIVATE
        # where the library itself will look for its internal headers
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    PUBLIC
        # where top-level project will look for the library's public headers
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        # where external projects will look for the library's public headers
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_compile_definitions(agInjector PUBLIC agInjector)

project(cmdInject)

add_executable(cmdInject cmd/main.c)

target_include_directories(cmdInject
    PRIVATE
        # where the library itself will look for its internal headers
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    PUBLIC
        # where top-level project will look for the library's public headers
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        # where external projects will look for the library's public headers
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_link_libraries(cmdInject PUBLIC agInjector)