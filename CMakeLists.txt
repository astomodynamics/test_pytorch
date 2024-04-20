cmake_minimum_required(VERSION 3.10.0)
project(test_libtorch)

# Set the search path to the parent directory of your libtorch install
set(CMAKE_PREFIX_PATH /home/tom/.local/lib/libtorch/share/cmake) 

# Find the Torch package
find_package(Torch REQUIRED)

# Output the location of libtorch for debugging
message(STATUS "Torch_DIR: ${Torch_DIR}") 

# Create a dummy executable to verify linking
add_executable(mytest main.cpp)
target_link_libraries(mytest PRIVATE ${TORCH_LIBRARIES}) 

