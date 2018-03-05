cmake_minimum_required(VERSION 3.2)
project(marketmaker C)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/;${CMAKE_MODULE_PATH}")

include(FindCURL)
find_package(CURL REQUIRED)

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

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    add_compile_options(-fdata-sections -ffunction-sections)
endif ()

if(WIN32)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_WINSOCK_DEPRECATED_NO_WARNINGS)
    add_definitions(-D_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING)
endif()

set(DEPS_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install)

include(DownloadProject.cmake)

download_project(PROJ nanomsg
        GIT_REPOSITORY  https://github.com/nanomsg/nanomsg.git
        GIT_TAG         1.1.2
        GIT_SHALLOW     1
        CMAKE_ARGS      "-DNN_ENABLE_DOC=OFF -DNN_TESTS=OFF -DNN_TOOLS=OFF -DNN_ENABLE_NANOCAT=OFF -DCMAKE_INSTALL_PREFIX=${DEPS_INSTALL_PREFIX}"
        UPDATE_DISCONNECTED 1
        )

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
include_directories(${DEPS_INSTALL_PREFIX}/include)
target_link_libraries(${PROJECT_NAME} crypto777 jpeg secp256k1 ${CURL_LIBRARIES})
if(UNIX)
    target_link_libraries(${PROJECT_NAME} m)
endif()
if(WIN32)
    target_link_libraries(${PROJECT_NAME} wsock32 ws2_32)
endif()


add_subdirectory(secp256k1)
include_directories(secp256k1/include)

add_subdirectory(jpeg)
include_directories(jpeg)

add_subdirectory(crypto777)
include_directories(crypto777/includes)

install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)

include_directories(src/main)

target_link_libraries(${PROJECT_NAME} nanomsg)

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

