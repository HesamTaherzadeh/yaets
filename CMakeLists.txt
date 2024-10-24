cmake_minimum_required(VERSION 3.5)
project(yaets VERSION 0.1)

set(CMAKE_CXX_STANDARD 17)

include(GNUInstallDirs)

option(BUILD_WITH_ROS "Build with ROS support using ament_cmake" ON)

if(BUILD_WITH_ROS)
  find_package(ament_cmake REQUIRED)
  ament_export_include_directories(include)
  ament_export_libraries(${PROJECT_NAME})
endif()

include_directories(include)

add_library(${PROJECT_NAME} SHARED
  src/yaets/tracing.cpp
)

if(BUILD_WITH_ROS)
  message(STATUS "Installing with ROS")
  install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

  install(PROGRAMS
    scripts/gantt.py
    scripts/histogram.py
    DESTINATION ${CMAKE_INSTALL_BINDIR}
  )

  install(TARGETS
    ${PROJECT_NAME}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )

else()
  message(STATUS "Installing without ROS")

  install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

  install(PROGRAMS
    scripts/gantt.py
    scripts/histogram.py
    DESTINATION ${CMAKE_INSTALL_BINDIR}
  )

  install(TARGETS
    ${PROJECT_NAME}
    EXPORT yaetsTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )
endif()

if(BUILD_TESTING)
  enable_testing()

  if(BUILD_WITH_ROS)
    find_package(ament_lint_auto REQUIRED)
    ament_lint_auto_find_test_dependencies()
    find_package(ament_cmake_gtest REQUIRED)
  else()
    find_package(GTest REQUIRED)
    include_directories(${GTEST_INCLUDE_DIRS})
    add_subdirectory(tests)
  endif()
endif()

if(BUILD_WITH_ROS)
  ament_package()
else()
  export(PACKAGE ${PROJECT_NAME})
endif()

if(NOT BUILD_WITH_ROS)

  include(CMakePackageConfigHelpers)

  write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/yaetsConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
  )

  configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/yaetsConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/yaetsConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/yaets
  )

  install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/yaetsConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/yaetsConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/yaets
  )

  install(EXPORT yaetsTargets
    FILE yaetsTargets.cmake
    NAMESPACE yaets::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/yaets
  )
endif()
