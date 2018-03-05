cmake_minimum_required(VERSION 3.2)
project(jpeg C)

set(CMAKE_C_STANDARD 99)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    set(C_OPTIONS -O2 -fdata-sections -ffunction-sections)
endif ()

if(WIN32)
    set(C_DEFS ${C_DEFS} -D_CRT_SECURE_NO_WARNINGS)
endif()

file(GLOB SOURCE_FILES "*.c")
if(WIN32)
    set(OS_SUBDIR "win")
else()
    set(OS_SUBDIR "unix")
endif()

file(GLOB PLAT_SOURCE_FILES "${OS_SUBDIR}/*.c")

set(SOURCE_FILES
        ${SOURCE_FILES}
        ${PLAT_SOURCE_FILES}
        )

add_library(${PROJECT_NAME} ${SOURCE_FILES})
target_compile_definitions(${PROJECT_NAME} PRIVATE ${C_DEFS})
target_compile_options(${PROJECT_NAME} PRIVATE ${C_OPTIONS})

install(TARGETS ${PROJECT_NAME} 
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib/static
)

