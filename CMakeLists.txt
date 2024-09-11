cmake_minimum_required(VERSION 3.14)
cmake_policy(VERSION 3.14)
if(POLICY CMP0135) # DOWNLOAD_EXTRACT_TIMESTAMP
  cmake_policy(SET CMP0135 OLD)
endif()

project(jwt-cpp)

option(JWT_DISABLE_BASE64 "Do not include the base64 implementation from this library" OFF)

set(JWT_INCLUDE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(JWT_HEADER_FILES 
    ${JWT_INCLUDE_PATH}/jwt-cpp/jwt.h
    ${JWT_INCLUDE_PATH}/jwt-cpp/traits/kazuho-picojson/defaults.h
    ${JWT_INCLUDE_PATH}/jwt-cpp/traits/kazuho-picojson/traits.h
)

if(NOT JWT_DISABLE_BASE64)
  list(APPEND JWT_HEADER_FILES ${JWT_INCLUDE_PATH}/jwt-cpp/base.h)
endif()

add_library(jwt-cpp INTERFACE)
add_library(jwt-cpp::jwt-cpp ALIAS jwt-cpp) # To match export
target_compile_features(jwt-cpp INTERFACE cxx_std_11)
if(JWT_DISABLE_BASE64)
  target_compile_definitions(jwt-cpp INTERFACE JWT_DISABLE_BASE64)
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
target_include_directories(jwt-cpp INTERFACE $<BUILD_INTERFACE:${JWT_INCLUDE_PATH}>
                                             $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

target_link_libraries(jwt-cpp INTERFACE libcrypto.a libssl.a)

set(JWT_CMAKE_FILES_INSTALL_DIR cmake)

write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config-version.cmake VERSION 0.7.0
                                 COMPATIBILITY ExactVersion)

install(TARGETS jwt-cpp EXPORT jwt-cpp-targets PUBLIC_HEADER 
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT jwt-cpp-targets NAMESPACE jwt-cpp:: FILE jwt-cpp-targets.cmake
        DESTINATION ${JWT_CMAKE_FILES_INSTALL_DIR})
install(DIRECTORY ${JWT_INCLUDE_PATH}/jwt-cpp 
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${JWT_INCLUDE_PATH}/picojson/picojson.h 
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/picojson)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config.cmake 
              ${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config-version.cmake
        DESTINATION ${JWT_CMAKE_FILES_INSTALL_DIR})
