cmake_minimum_required(VERSION 3.5)
project(rospack)

# Add support for C++11
if(NOT WIN32)
  add_definitions(-std=c++11)
endif()

add_definitions(-fPIC)

find_package(ament_cmake REQUIRED)
find_package(cmake_modules REQUIRED)
find_package(python_cmake_module REQUIRED)
find_package(PythonExtra MODULE)

set(Boost_USE_STATIC_LIBS ON)
find_package(Boost REQUIRED COMPONENTS filesystem program_options system)
set(Python_ADDITIONAL_VERSIONS "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")
find_package(PythonLibs "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}"
  REQUIRED)

find_package(TinyXML2 REQUIRED HINTS "*.lib") # add hint seems to allow debug lib to be found when appropriate

set(INCLUDE_DIRS include ${ament_cmake_INCLUDE_DIRS} ${cmake_modules_INCLUDE_DIRS}
  ${TinyXML2_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS} ${PythonExtra_INCLUDE_DIRS} ${PythonLibs_INCLUDE_DIRS})
include_directories(${INCLUDE_DIRS})

set(LIBRARY_DIRS ${ament_cmake_LIBRARY_DIRS} ${cmake_modules_LIBRARY_DIRS}
  ${TinyXML2_LIBRARY_DIRS} ${Boost_LIBRARY_DIRS} ${PythonExtra_LIBRARY_DIRS} ${PythonLibs_LIBRARY_DIRS})

link_directories(${LIBRARY_DIRS})

if(CMAKE_BUILD_TYPE MATCHES Debug)
set(BOOST_DEPENDENCIES
  $ENV{BOOST_LIBRARYDIR}/libboost_date_time-vc140-mt-gd-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_thread-vc140-mt-gd-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_system-vc140-mt-gd-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_chrono-vc140-mt-gd-1_61.lib
)
else()
set(BOOST_DEPENDENCIES
  $ENV{BOOST_LIBRARYDIR}/libboost_date_time-vc140-mt-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_thread-vc140-mt-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_system-vc140-mt-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_chrono-vc140-mt-1_61.lib
)
endif()

set(LIBS ${ament_cmake_LIBRARIES} ${cmake_modules_LIBRARIES}
  ${TinyXML2_LIBRARIES} ${BOOST_DEPENDENCIES} ${PythonExtra_LIBRARIES} ${PythonLibs_LIBRARIES})

#add_definitions(-Wall)

set(API_BACKCOMPAT_V1 "YES"
  CACHE
  BOOL "Whether to enable backwards compatibility with old C++ API")
if(API_BACKCOMPAT_V1)
  add_definitions(-DROSPACK_API_BACKCOMPAT_V1)
  set(backcompat_source src/rospack_backcompat.cpp)
endif()

add_library(rospack SHARED src/rospack.cpp ${backcompat_source} src/rospack_cmdline.cpp
  src/utils.cpp)
target_link_libraries(rospack ${LIBS} ${PYTHON_LIBRARIES})

add_executable(rospackexe src/rospack_main.cpp)
# Set the name, and make it a "global" executable
set_target_properties(rospackexe
  PROPERTIES
  OUTPUT_NAME rospack
  RUNTIME_OUTPUT_DIRECTORY bin)
target_link_libraries(rospackexe rospack)
add_executable(rosstackexe src/rosstack_main.cpp)
target_link_libraries(rosstackexe rospack)
# Set the name, and make it a "global" executable
set_target_properties(rosstackexe
  PROPERTIES
  OUTPUT_NAME rosstack
  RUNTIME_OUTPUT_DIRECTORY bin)

install(TARGETS rospack rospackexe rosstackexe
  ARCHIVE
  DESTINATION lib
  LIBRARY
  DESTINATION lib
  RUNTIME
  DESTINATION bin)
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION include/${PROJECT_NAME}
  FILES_MATCHING
  PATTERN "*.h")

# uninstall target
#configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
#  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
#  IMMEDIATE @ONLY)
#add_custom_target(uninstall
#  COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

# TODO: fix tests
#if(BUILD_TESTING)
#  add_subdirectory(test)
#endif()

if(DOXYGEN_FOUND)
  add_custom_target(rospack-docs
    COMMAND doxygen Doxyfile
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})
endif()

ament_export_dependencies(ament_cmake)
ament_export_dependencies(cmake_modules)
ament_export_include_directories(${INCLUDE_DIRS})
ament_export_libraries(rospack)

ament_package()
