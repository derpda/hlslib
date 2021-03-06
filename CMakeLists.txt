# Author:  Johannes de Fine Licht (definelicht@inf.ethz.ch)

cmake_minimum_required(VERSION 3.0)
project(hlslib)

option(BUILD_TESTS "Build tests for hlslib" ON)
option(BUILD_DOCUMENTATION "Build documentation for hlslib" ON)
option(HLSLIB_XILINX "Use Xilinx FPGA ecosystem" ON)
option(HLSLIB_INTEL "Use Intel FPGA ecosystem" ON)

# Include custom Find<Module>.cmake scripts to enable searching for
# Vitis/SDAccel and Intel FPGA OpenCL.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake)

# Without this variable set, CMake will build tests when running install
set(CMAKE_SKIP_INSTALL_ALL_DEPENDENCY ON)

# Generate Doxygen if available
if(${BUILD_DOCUMENTATION})
  find_package(Doxygen REQUIRED)
  configure_file(${CMAKE_SOURCE_DIR}/Doxyfile.in Doxyfile)
  add_custom_target(doxygen ALL
      COMMAND ${DOXYGEN_EXECUTABLE} Doxyfile
      WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
endif()

# Find Xilinx and Intel OpenCL software for enabling test targets
find_package(Threads REQUIRED)
find_package(OpenCL REQUIRED)
if(${HLSLIB_XILINX})
  find_package(Vitis REQUIRED)
endif()
if(${HLSLIB_INTEL})
  find_package(IntelFPGAOpenCL REQUIRED)
endif()

# hlslib target
add_library(hlslib INTERFACE)
target_include_directories(hlslib INTERFACE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    ${Vitis_INCLUDE_DIRS})
target_link_libraries(hlslib INTERFACE
    ${CMAKE_THREAD_LIBS_INIT}
    OpenCL::OpenCL)

if(${BUILD_TESTS})
  enable_testing()
  # Xilinx example setup and testing
  if(${HLSLIB_XILINX})
    add_subdirectory(xilinx_test)
  endif()
  if(${HLSLIB_INTEL})
    add_subdirectory(intel_test)
  endif()
endif()

# Installation
install(TARGETS hlslib
    EXPORT hlslibConfig
    LIBRARY DESTINATION lib COMPONENT Runtime
    ARCHIVE DESTINATION lib COMPONENT Development
    RUNTIME DESTINATION bin COMPONENT Runtime
    PUBLIC_HEADER DESTINATION include COMPONENT Development
    BUNDLE DESTINATION bin COMPONENT Runtime)
install(EXPORT hlslibConfig DESTINATION cmake)
export(TARGETS hlslib FILE hlslibConfig.cmake)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/hlslib DESTINATION include)
install(FILES ${PROJECT_SOURCE_DIR}/cmake/FindVitis.cmake DESTINATION cmake)
install(FILES ${PROJECT_SOURCE_DIR}/cmake/FindSDAccel.cmake DESTINATION cmake)
install(FILES ${PROJECT_SOURCE_DIR}/cmake/FindIntelFPGAOpenCL.cmake DESTINATION cmake)
