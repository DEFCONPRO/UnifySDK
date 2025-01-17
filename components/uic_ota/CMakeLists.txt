# Find nlohmann_json, unfortunately the find_package doesn't work for this when
# cross compiling, thus we use the simplified method find_path instead
find_path(nlohmann_json_include nlohmann/json.hpp REQUIRED)

# List of source files for uic_ota
set(uic_ota_source_files
    platform/${COMPATIBLE_PLATFORM}/ota_time.cpp src/ota.cpp src/ota_cache.cpp
    src/ota_mqtt.cpp src/ota_download.cpp)

# Library
add_library(uic_ota OBJECT ${uic_ota_source_files})

target_include_directories(
  uic_ota
  PRIVATE src ${nlohmann_json_include}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/platform/${COMPATIBLE_PLATFORM}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>)

target_link_libraries(uic_ota PUBLIC uic_definitions uic_contiki uic_mqtt
                                     uic_log)

install_headers(uic_ota uic libuic-dev)

# GCC 8.3 define the std::filesystem as a separate library
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_link_libraries(uic_ota PRIVATE stdc++fs)
endif()

if(BUILD_TESTING)
  # Mock. We make a uic_ota_mock version with the MQTT Mock library.
  add_library(uic_ota_mock ${uic_ota_source_files})

  target_include_directories(
    uic_ota_mock
    PRIVATE src
    PUBLIC platform/${COMPATIBLE_PLATFORM} include)

  target_link_libraries(uic_ota_mock PUBLIC uic_definitions uic_contiki
                                            uic_mqtt_mock uic_log)

  # Test
  add_subdirectory(test)
endif()
