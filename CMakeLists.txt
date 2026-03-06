cmake_minimum_required(VERSION 3.22)

project(message_logger)


find_package(ament_cmake REQUIRED)
find_package(ament_cmake_ros REQUIRED)
find_package(rclcpp REQUIRED)

include(CMakeDependentOption)
cmake_dependent_option(MELO_USE_COUT "Use std::cout" OFF
                       "DEFINED ENV{ROS_DISTRO}" ON)

# Avoid output on stderr by unsetting catkin variable not used - https://github.com/colcon/colcon-ros/issues/83#issue-504270285
unset(CATKIN_INSTALL_INTO_PREFIX_ROOT)

if(NOT MELO_USE_COUT)
  message(STATUS "Building message_logger using ros.")
else()
  message(STATUS "Building message_logger using std::cout.")
endif()

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/include/message_logger/log/log_messages_backend_config.hpp.in
  ${CMAKE_CURRENT_BINARY_DIR}/include/message_logger/log/log_messages_backend_config.hpp @ONLY
)

if(DEFINED MELO_FUNCTION_PRINTS)
  add_definitions(-DMELO_FUNCTION_PRINTS)
endif()


add_library(${PROJECT_NAME} SHARED
  src/log/globals.cpp
  src/time/Time.cpp
  src/time/TimeStd.cpp
)
target_include_directories(${PROJECT_NAME}
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
    
ament_target_dependencies(${PROJECT_NAME}
  rclcpp
)

target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic -fPIC)


# Install
install(TARGETS ${PROJECT_NAME} 
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
)
install(DIRECTORY include/ DESTINATION include/
)
install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/include/message_logger/log/log_messages_backend_config.hpp
  DESTINATION include/message_logger/log
)

ament_export_include_directories(include)
ament_export_libraries(
  ${PROJECT_NAME}
)
ament_export_targets(
  export_${PROJECT_NAME}
)

# Test
if(BUILD_TESTING)
  find_package(ament_cmake_gtest REQUIRED)
  ament_add_gtest(empty_test test/EmptyTests.cpp)
  target_link_libraries(empty_test)
endif()

ament_export_dependencies(rclcpp)

ament_package()
