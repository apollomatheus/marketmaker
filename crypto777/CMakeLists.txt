cmake_minimum_required(VERSION 3.2)
project(crypto777 C)

set(CMAKE_C_STANDARD 99)

set(C_DEFS -DLIQUIDITY_PROVIDER=1)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    set(C_OPTIONS -O2 -fdata-sections -ffunction-sections)
endif ()

if(WIN32)
    set(C_DEFS ${C_DEFS} -D_CRT_SECURE_NO_WARNINGS -D_WINSOCK_DEPRECATED_NO_WARNINGS)
endif()

set(SOURCE_FILES
        OS_nonportable.c
        OS_portable.c
        OS_time.c
        SaM.c
        bitcoind_RPC.c
        cJSON.c
        curve25519-donna.c
        curve25519.c
        hmac_sha512.c
        iguana_OS.c
        iguana_serdes.c
        iguana_utils.c
        inet.c
        ramcoder.c
        scrypt.c
        tweetnacl.c
        )

add_library(${PROJECT_NAME} ${SOURCE_FILES})
target_compile_definitions(${PROJECT_NAME} PRIVATE ${C_DEFS})
target_compile_options(${PROJECT_NAME} PRIVATE ${C_OPTIONS})
target_include_directories(${PROJECT_NAME} PRIVATE includes ${nanomsg_SOURCE_DIR}/src)

install(TARGETS ${PROJECT_NAME} 
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib/static
)

