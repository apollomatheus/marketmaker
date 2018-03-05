cmake_minimum_required(VERSION 3.2 FATAL_ERROR)
project(marketmaker LANGUAGES C)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/;${CMAKE_MODULE_PATH}")

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

# VERSION value is set based on semantic version reported by "git describe"
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

configure_file(version_short.txt.in version_short.txt @ONLY)

set(CMAKE_C_STANDARD 99)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    add_compile_options(-fdata-sections -ffunction-sections)
endif ()

if(WIN32)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

set(DEPS_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install)

include(cmake/DownloadProject.cmake)

# Download and install nanomsg at CMake configure time
download_project(PROJ nanomsg
        GIT_REPOSITORY  https://github.com/nanomsg/nanomsg.git
        GIT_TAG         1.1.2
        GIT_SHALLOW     1
        GIT_PROGRESS    1
        CMAKE_ARGS      "-DNN_STATIC_LIB=ON -DNN_ENABLE_DOC=OFF -DNN_TESTS=OFF -DNN_TOOLS=OFF -DNN_ENABLE_NANOCAT=OFF -DCMAKE_INSTALL_PREFIX=${DEPS_INSTALL_PREFIX}"
        UPDATE_DISCONNECTED 1
        )

find_library(NANOMSG_LIBRARY NAMES nanomsg PATHS ${DEPS_INSTALL_PREFIX}/lib)

# Download and install cURL at CMake configure time
if(APPLE)
    set(CURL_CMAKE_ARGS "-DCMAKE_USE_OPENSSL=OFF -DCMAKE_USE_DARWINSSL=ON")
endif()
if(WIN32)
    set(CURL_CMAKE_ARGS "-DCMAKE_USE_OPENSSL=OFF -DCMAKE_USE_WINSSL=ON")
endif()
download_project(PROJ curl
        GIT_REPOSITORY  https://github.com/curl/curl.git
        GIT_TAG         curl-7_58_0
        GIT_SHALLOW     1
        GIT_PROGRESS    1
        CMAKE_ARGS      "-DCMAKE_INSTALL_PREFIX=${DEPS_INSTALL_PREFIX} ${CURL_CMAKE_ARGS}"
        UPDATE_DISCONNECTED 1
        )

find_library(CURL_LIBRARY NAMES curl PATHS ${DEPS_INSTALL_PREFIX}/lib)

# dependencies as submodules
add_subdirectory(secp256k1)
add_subdirectory(jpeg)
add_subdirectory(crypto777)

set(SOURCE_FILES
        src/main/mm.c
        src/main/groestl.c
        src/main/groestl.h
        src/main/mini-gmp.c
        src/main/mini-gmp.h
        src/main/segwit_addr.c
        src/main/segwit_addr.h
        )

add_executable(${PROJECT_NAME} ${SOURCE_FILES})
target_include_directories(${PROJECT_NAME} PRIVATE
        ${DEPS_INSTALL_PREFIX}/include
        secp256k1/include
        jpeg
        crypto777/includes)
target_link_libraries(${PROJECT_NAME} ${NANOMSG_LIBRARY} crypto777 jpeg secp256k1 ${CURL_LIBRARY})
target_link_libraries(${PROJECT_NAME} Threads::Threads)
if(UNIX)
    target_link_libraries(${PROJECT_NAME} m)
endif()
if(WIN32)
    target_link_libraries(${PROJECT_NAME} wsock32 ws2_32)
endif()

install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)

set(CPACK_SET_DESTDIR ON)

set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "The SuperNET Developers")
set(CPACK_PACKAGE_CONTACT "Eliezio Oliveira <eliezio@protonmail.com>")
set(CPACK_PACKAGE_VERSION_MAJOR ${VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${VERSION_PATCH})
set(CPACK_RPM_PACKAGE_RELEASE ${PACKAGE_RELEASE})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "MarketMaker")
set(CPACK_RPM_SPEC_INSTALL_POST "/bin/true")
include(CPack)

