cmake_minimum_required(VERSION 3.4)

project(APCpp)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# for building with the geode action
set(USE_MBED_TLS ON)
set(CMAKE_C_SIMULATE_ID "Clang")

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

add_library(APCpp STATIC Archipelago.cpp Archipelago.h)

set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(ENABLE_TESTING OFF CACHE BOOL "" FORCE)
set(ENABLE_PROGRAMS OFF CACHE BOOL "" FORCE)

# Enable TLS. On Windows, build mbedTLS
set(USE_TLS ON CACHE BOOL "Enable TLS support" FORCE)
if (WIN32 AND USE_TLS)
    # geode embeds cpm
    CPMAddPackage("gh:Mbed-TLS/mbedtls@3.6.1")
    target_link_libraries(${PROJECT_NAME} mbedtls)
    
    set(USE_STATIC_MBEDTLS_LIBRARY ON CACHE BOOL "" FORCE)
    set(MBEDTLS_INCLUDE_DIRS ${mbedtls_SOURCE_DIR}/include)
    set(MBEDTLS_VERSION_GREATER_THAN_3 ${MBEDTLS_INCLUDE_DIRS})
    set(MBEDTLS_LIBRARY mbedtls)
    set(MBEDX509_LIBRARY mbedx509)
    set(MBEDCRYPTO_LIBRARY mbedcrypto)

    target_include_directories(${PROJECT_NAME} PUBLIC ${mbedtls_SOURCE_DIR}/include)
endif (WIN32 AND USE_TLS)

set(IXWEBSOCKET_INSTALL OFF CACHE BOOL "" FORCE)
add_subdirectory(IXWebSocket)
include_directories(IXWebSocket)
target_link_libraries(APCpp ixwebsocket)

if (MINGW)
    target_link_libraries(APCpp -static -static-libstdc++ -static-libgcc)
endif(MINGW)

# Attempt finding system version of JsonCpp
find_package(PkgConfig)
if (PKGCONFIG_FOUND AND (NOT MINGW))
    pkg_check_modules(JSONCPP jsoncpp)
    if (JSONCPP_FOUND)
        include_directories(${JSONCPP_INCLUDE_DIRS})
        target_link_libraries(APCpp ${JSONCPP_LDFLAGS})
    endif(JSONCPP_FOUND)
endif(PKGCONFIG_FOUND AND (NOT MINGW))

# No system version: Enable bundled
if((NOT JSONCPP_FOUND))
    set(JSONCPP_WITH_TESTS OFF CACHE BOOL "" FORCE)
    set(JSONCPP_WITH_POST_BUILD_UNITTEST OFF CACHE BOOL "" FORCE)
    add_subdirectory(jsoncpp)
    set_property(TARGET jsoncpp_static PROPERTY POSITION_INDEPENDENT_CODE ON)
    include_directories(jsoncpp/include)
    target_link_libraries(APCpp jsoncpp_static)
endif((NOT JSONCPP_FOUND))

if (WIN32)
    target_link_libraries(APCpp ws2_32)
endif(WIN32)
