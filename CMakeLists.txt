cmake_minimum_required(VERSION 3.2)
project(marketmaker-cli)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/;${CMAKE_MODULE_PATH}")

include(cmake/GetGitRevisionDescription.cmake)
git_describe(VERSION "--dirty")
# parse the version information into pieces.
if (NOT VERSION MATCHES "^v?([0-9]+)\\.([0-9]+)\\.([0-9]+).*")
  message(FATAL_ERROR "Invalid version: " ${VERSION})
endif()
set(VERSION_MAJOR ${CMAKE_MATCH_1})
set(VERSION_MINOR ${CMAKE_MATCH_2})
set(VERSION_PATCH ${CMAKE_MATCH_3})
set(VERSION_SHORT "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set(PACKAGE_RELEASE "1")    # fixed
message(STATUS "Version set to: " ${VERSION})

configure_file(version.h.in include/version.h @ONLY)
configure_file(version_short.txt.in version_short.txt @ONLY)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    add_compile_options(-fdata-sections -ffunction-sections)
    link_libraries(--gc-sections --print-gc-sections)
    # add_compile_options(-Werror -Wall -Wno-sign-compare -Wmissing-declarations -Wcast-qual -Wwrite-strings)
    # add_compile_options(-Wstrict-prototypes -Wmissing-prototypes)
endif ()

if(WIN32)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_WINSOCK_DEPRECATED_NO_WARNINGS)
    add_definitions(-D_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING)
endif()

set(CFLAGS_LIST ${CMAKE_C_FLAGS})
separate_arguments(CFLAGS_LIST)

set(SOURCE_FILES
        src/main/mm.c
        src/main/cJSON.c
        src/main/groestl.c
        src/main/mini-gmp.c
        src/main/segwit_addr.c
        )

add_executable(${PROJECT_NAME} ${SOURCE_FILES})
target_compile_options(${PROJECT_NAME} PUBLIC ${C_WARNINGS})
if(WIN32)
    target_link_libraries(${PROJECT_NAME} wsock32 ws2_32)
endif()

install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)

include(CTest)

include(DownloadProject.cmake)
download_project(PROJ googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG release-1.8.0
        UPDATE_DISCONNECTED 1
        )

# Prevent GoogleTest from overriding our compiler/linker options
# when building with Visual Studio
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR} EXCLUDE_FROM_ALL)

include_directories("src/main" ${CMAKE_BINARY_DIR}/include)

set(CPACK_SET_DESTDIR ON)

set(CPACK_PACKAGE_NAME "marketmaker")
set(CPACK_PACKAGE_VENDOR "The SuperNET Developers")
set(CPACK_PACKAGE_CONTACT "Eliezio Oliveira <eliezio@protonmail.com>")
set(CPACK_PACKAGE_VERSION_MAJOR ${VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${VERSION_PATCH})
set(CPACK_RPM_PACKAGE_RELEASE ${PACKAGE_RELEASE})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "MarketMaker")
set(CPACK_RPM_SPEC_INSTALL_POST "/bin/true")
include(CPack)

