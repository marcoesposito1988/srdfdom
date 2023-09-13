cmake_minimum_required(VERSION 3.0.2)
project(srdfdom)

find_package(Boost REQUIRED)

find_package(console_bridge REQUIRED)
find_package(urdfdom_headers REQUIRED)
find_package(urdfdom REQUIRED)

find_package(catkin QUIET COMPONENTS cmake_modules urdf)

if(NOT catkin_FOUND)
  # use local copies of FindTinyXML.cmake and FindTinyXML2.cmake from
  # 'cmake_modules' (https://github.com/ros/cmake_modules)
  list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)
endif()

find_package(TinyXML2 REQUIRED)

include_directories(
  include
  ${Boost_INCLUDE_DIR}
  ${TinyXML_INCLUDE_DIR}
  ${TinyXML2_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${console_bridge_INCLUDE_DIRS}
  ${urdfdom_headers_INCLUDE_DIRS}
  ${urdfdom_INCLUDE_DIRS}
  )

if (catkin_FOUND)
catkin_python_setup()

catkin_package(
  LIBRARIES ${PROJECT_NAME}
  INCLUDE_DIRS include
  DEPENDS TinyXML2 console_bridge urdfdom_headers
)
else()
  set(CATKIN_PACKAGE_INCLUDE_DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME})
  set(CATKIN_PACKAGE_LIB_DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
  set(CATKIN_PACKAGE_BIN_DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
  set(CATKIN_GLOBAL_BIN_DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
  set(${PROJECT_NAME}_VERSION 0.6.4)
endif()

add_library(${PROJECT_NAME}
  src/model.cpp
  src/srdf_writer.cpp
)
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${${PROJECT_NAME}_VERSION})
target_link_libraries(${PROJECT_NAME} ${TinyXML2_LIBRARIES} ${catkin_LIBRARIES} ${console_bridge_LIBRARIES} ${urdfdom_headers_LIBRARIES})


install(TARGETS ${PROJECT_NAME}
  EXPORT srdfdomTargets
  DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

install(PROGRAMS
  scripts/display_srdf
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

if(CATKIN_ENABLE_TESTING)
  find_package(rostest REQUIRED)
  add_rostest(test/srdf_parser.test)

  add_definitions(-DTEST_RESOURCE_LOCATION="${CMAKE_SOURCE_DIR}/test/resources")
  execute_process(COMMAND bash -c "locale -a | grep -q ^nl_NL"
    RESULT_VARIABLE TEST_LOCALE
    OUTPUT_QUIET ERROR_QUIET)
  if (TEST_LOCALE)
    message(STATUS "Locale nl_NL not available. Locale test will not be meaningful.")
  endif()

  add_rostest_gtest(test_cpp test/srdf_parser_cpp.test test/test_parser.cpp)
  target_link_libraries(test_cpp ${catkin_LIBRARIES} ${PROJECT_NAME})
endif()

if(NOT catkin_FOUND)
  install(
          EXPORT srdfdomTargets
          FILE srdfdomTargets.cmake
          DESTINATION "${CMAKE_INSTALL_PREFIX}/CMake"
          NAMESPACE srdfdom::
  )

  include(CMakePackageConfigHelpers)
  configure_package_config_file(
          cmake/srdfdomConfig.cmake.in
          "${CMAKE_CURRENT_BINARY_DIR}/srdfdomConfig.cmake"
          INSTALL_DESTINATION "${CMAKE_INSTALL_PREFIX}/CMake"
  )

  install(
          FILES "${CMAKE_CURRENT_BINARY_DIR}/srdfdomConfig.cmake"
          DESTINATION "${CMAKE_INSTALL_PREFIX}/CMake"
  )
endif()

