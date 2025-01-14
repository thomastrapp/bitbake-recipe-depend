cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(bb-depends-dot VERSION 0.2.0)

# Add the top-level cmake module directory to CMAKE_MODULE_PATH
list(INSERT CMAKE_MODULE_PATH 0 ${PROJECT_SOURCE_DIR}/cmake)

find_package(Boost COMPONENTS graph program_options REQUIRED)

configure_file(
  "${PROJECT_SOURCE_DIR}/bbrd/bbrd/Version.cpp.in"
  "Version.cpp"
  # Restrict variable replacement to references of the form @VAR@
  @ONLY)

add_executable(
  bb-depends-dot
  "${CMAKE_CURRENT_BINARY_DIR}/Version.cpp"
  "${PROJECT_SOURCE_DIR}/bbrd/bbrd/DependencyGraph.cpp"
  "${PROJECT_SOURCE_DIR}/bbrd/bbrd/ErrorOutput.cpp"
  "${PROJECT_SOURCE_DIR}/bbrd/bbrd/File.cpp"
  "${PROJECT_SOURCE_DIR}/bbrd/bbrd/ProgramOptions.cpp"
  "${PROJECT_SOURCE_DIR}/ragel/Dependencies.cpp"
  "${PROJECT_SOURCE_DIR}/bbrd/main.cpp")

include(EnableWarnings)
enable_warnings(bb-depends-dot PUBLIC)

target_link_libraries(bb-depends-dot Boost::graph Boost::program_options)

target_include_directories(
  bb-depends-dot PRIVATE
  "${PROJECT_SOURCE_DIR}/bbrd")
target_compile_features(bb-depends-dot PRIVATE cxx_std_17)

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
  add_subdirectory("test")
endif()

include(GNUInstallDirs)
install(TARGETS bb-depends-dot RUNTIME
        DESTINATION ${CMAKE_INSTALL_BINDIR})

