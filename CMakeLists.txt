cmake_minimum_required(VERSION 3.5)
project(quatro)

set(CMAKE_CXX_STANDARD 17)

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif ()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)

find_package(OpenCV REQUIRED)
find_package(OpenCV REQUIRED QUIET)
find_package(OpenMP)
find_package(PCL REQUIRED)
find_package(Boost REQUIRED)
find_package(Eigen3 QUIET REQUIRED NO_MODULE)

include_directories(
        ${catkin_INCLUDE_DIRS}
        ${OpenCV_INCLUDE_DIRS}
        ${PCL_INCLUDE_DIRS}
        include
)

include(3rdparty/find_dependencies.cmake)

#file(DOWNLOAD https://urserver.kaist.ac.kr/publicdata/quatro/000540.bin ${CMAKE_CURRENT_LIST_DIR}/materials/000540.bin)
#file(DOWNLOAD https://urserver.kaist.ac.kr/publicdata/quatro/001319.bin ${CMAKE_CURRENT_LIST_DIR}/materials/001319.bin)

if (OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif ()

####### Executable #######
add_library(quatro SHARED
  src/graph.cc
  src/teaser_utils/feature_matcher.cc
  src/teaser_utils/fpfh.cc
)
target_compile_features(quatro PUBLIC c_std_99 cxx_std_17)  # Require C99 and C++17
target_include_directories(quatro PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
#ament_target_dependencies(quatro
#  rclcpp
#  OpenCV
#  OpenMP
#  PCL
#  Boost
#  Eigen3
#)
target_link_libraries(quatro PRIVATE
  ${rclcpp_LIBRARIES}
  ${OpenCV_LIBRARIES}
  ${OpenMP_LIBRARIES}
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
  ${Eigen3_LIBRARIES}
  pmc::pmc
)
# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
target_compile_definitions(quatro PRIVATE "QUATRO_BUILDING_LIBRARY")

install(
  DIRECTORY include/
  DESTINATION include
)
install(
  TARGETS quatro PMCHelper
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

ament_export_dependencies(
  rclcpp
  OpenCV
  OpenMP
  PCL
  Boost
  Eigen3
)
ament_export_include_directories(
  include
  ${EIGEN3_INCLUDE_DIRS}
)
ament_export_libraries(
  quatro
  pmc
)
ament_export_targets(
  export_${PROJECT_NAME}
)
ament_package()
