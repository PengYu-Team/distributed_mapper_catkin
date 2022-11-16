cmake_minimum_required(VERSION 2.8.3)
project(distributed_mapper_catkin)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++11")

find_package(catkin REQUIRED COMPONENTS
  gtsam_catkin
)

catkin_package(
  INCLUDE_DIRS include
  DEPENDS 

  CATKIN_DEPENDS

  LIBRARIES distributed_mapper_catkin
)

# include directories
include_directories(
  include
  include/pairwise_consistency_maximization
  ${CMAKE_CURRENT_SOURCE_DIR}/src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/
  ${catkin_INCLUDE_DIRS}
)

# link directories
link_directories(
  include
)

# Graph utils library
add_library(graph_utils src/pairwise_consistency_maximization/graph_utils/graph_utils_functions.cpp)
target_link_libraries(graph_utils)

# Robot local map library
add_library(robot_measurements src/pairwise_consistency_maximization/robot_measurements/robot_measurements.cpp src/pairwise_consistency_maximization/robot_measurements/interrobot_measurements.cpp src/pairwise_consistency_maximization/robot_measurements/robot_local_map.cpp)
target_link_libraries(robot_measurements graph_utils)

# Paiwise Consistency computation library
add_library(pairwise_consistency src/pairwise_consistency_maximization/pairwise_consistency/pairwise_consistency.cpp)
target_link_libraries(pairwise_consistency robot_measurements)

# Maximum Clique Solver
add_library(fast_max-clique_finder src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/findClique.h src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/graphIO.h src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/findClique.cpp src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/findCliqueHeu.cpp  src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/utils.cpp  src/pairwise_consistency_maximization/third_parties/fast_max-clique_finder/src/graphIO.cpp
)
target_compile_options(fast_max-clique_finder PRIVATE -w)

# Global map solver library
add_library(global_map_solver src/pairwise_consistency_maximization/global_map/global_map.cpp)
target_link_libraries(global_map_solver graph_utils pairwise_consistency fast_max-clique_finder)

# Distributed PCM library
add_library(distributed_pcm src/pairwise_consistency_maximization/distributed_pcm/distributed_pcm.cpp)
target_link_libraries(distributed_pcm global_map_solver)

# Distributed Mapper
add_library(distributed_mapper_catkin src/distributed_mapper/distributed_mapper.cpp src/distributed_mapper/evaluation_utils.cpp include/distributed_mapper/distributed_mapper_utils.h src/distributed_mapper/distributed_mapper_utils.cpp src/distributed_mapper/run_distributed_mapper.cpp)
target_link_libraries(distributed_mapper_catkin graph_utils robot_measurements pairwise_consistency fast_max-clique_finder distributed_pcm global_map_solver)
