cmake_minimum_required(VERSION 3.2)
project(secp256k1 C)

set(CMAKE_C_STANDARD 90)

set(C_DEFS -DHAVE_CONFIG_H)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    set(C_OPTIONS -O3 -fdata-sections -ffunction-sections)
    set(C_OPTIONS ${C_OPTIONS} -pedantic -Wall -Wextra -Wcast-align -Wnested-externs -Wshadow -Wstrict-prototypes -Wno-unused-function -Wno-long-long -Wno-overlength-strings -fvisibility=hidden)
endif ()

if(WIN32)
    set(C_DEFS ${C_DEFS} -D_CRT_SECURE_NO_WARNINGS)
endif()

set(SOURCE_FILES
        src/secp256k1.c
        )

add_library(${PROJECT_NAME} ${SOURCE_FILES})
target_compile_definitions(${PROJECT_NAME} PRIVATE ${C_DEFS})
target_compile_options(${PROJECT_NAME} PRIVATE ${C_OPTIONS})

install(TARGETS ${PROJECT_NAME} 
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib/static
)

