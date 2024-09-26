cmake_minimum_required(VERSION 3.10.0)
project(test_libtorch)

# Set the search path to the parent directory of your libtorch install
set(CMAKE_PREFIX_PATH /home/tom/.local/lib/libtorch/share/cmake) 

# find dependencies
find_package(PythonLibs 3.10 REQUIRED)
find_package(Eigen3 REQUIRED) 

# Find the Torch package
find_package(Torch REQUIRED)

# Output the location of libtorch for debugging
message(STATUS "Torch_DIR: ${Torch_DIR}") 



option(DOWNLOAD_MNIST "Download the MNIST dataset from the internet" ON)
if (DOWNLOAD_MNIST)
  message(STATUS "Downloading MNIST dataset")
  execute_process(
    COMMAND python ${CMAKE_CURRENT_LIST_DIR}/../tools/download_mnist.py
      -d ${CMAKE_BINARY_DIR}/data
    ERROR_VARIABLE DOWNLOAD_ERROR)
  if (DOWNLOAD_ERROR)
    message(FATAL_ERROR "Error downloading MNIST dataset: ${DOWNLOAD_ERROR}")
  endif()
endif()


# Create a dummy executable to verify linking
add_executable(mytest src/mytest.cpp)
target_link_libraries(mytest PRIVATE ${TORCH_LIBRARIES}) 

# Create the DCGAN executable
add_executable(dcgan src/dcgan.cpp)
target_link_libraries(dcgan "${TORCH_LIBRARIES}")
set_property(TARGET dcgan PROPERTY CXX_STANDARD 17)

# Create eigen torch test executable
add_executable(test_torch_eigen src/test_torch_eigen.cpp)

# Link the test executable against your library and Torch
target_link_libraries(test_torch_eigen cddp ${TORCH_LIBRARIES} ${PYTHON_LIBRARIES} Eigen3::Eigen)


if (MSVC)
  file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
  add_custom_command(TARGET dcgan
                     POST_BUILD
                     COMMAND ${CMAKE_COMMAND} -E copy_if_different
                     ${TORCH_DLLS}
                     $<TARGET_FILE_DIR:dcgan>)
endif (MSVC)

