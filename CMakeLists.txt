cmake_minimum_required(VERSION 2.6)

project(cudaSift CUDA)
set(cudaSift_VERSION_MAJOR 2)
set(cudaSift_VERSION_MINOR 0)
set(cudaSift_VERSION_PATCH 0)

set(CPACK_PACKAGE_VERSION_MAJOR "${cudaSift_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${cudaSift_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${cudaSift_VERSION_PATCH}")
set(CPACK_GENERATOR "ZIP")
include(CPack)

find_package(OpenCV REQUIRED)
# find_package(CUDA)
# if (NOT CUDA_FOUND)
#   message(STATUS "CUDA not found. Project will not be built.")
# endif(NOT CUDA_FOUND)

if (WIN32)
  set(EXTRA_CXX_FLAGS "/DVERBOSE /D_CRT_SECURE_NO_WARNINGS ")
  list(APPEND CUDA_NVCC_FLAGS "-arch=sm_35;--compiler-options;-O2;-DVERBOSE") 
endif()
if (UNIX)
  if (APPLE)
    set(EXTRA_CXX_FLAGS "-DVERBOSE -msse2")
    list(APPEND CUDA_NVCC_FLAGS "-arch=sm_35;--compiler-options;-O2;-DVERBOSE") 
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -msse2 ")
    list(APPEND CUDA_NVCC_FLAGS "-lineinfo;-ccbin;/usr/bin/gcc-6;--compiler-options;-O2;-D_FORCE_INLINES;-DVERBOSE_NOT") 
  endif()
endif()

SET(INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/include/CudaSift" CACHE PATH "The directory the headers are installed in")
SET(LIB_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE STRING "Directory where lib will install")
SET(DOC_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/share/doc/${PROJECT_NAME}" CACHE PATH "Path to the documentation")

# TODO: Danny 08-07-2020 add CudaSiftConfig.cmake inside CudaSift
# IF(UNIX OR CYGWIN)
#     SET(_CMAKE_INSTALL_DIR "${LIB_INSTALL_DIR}/cmake/${PROJECT_NAME}")
# ELSEIF(WIN32)
#     SET(_CMAKE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/cmake")
# ENDIF()
# SET(CMAKE_INSTALL_DIR "${_CMAKE_INSTALL_DIR}" CACHE PATH "The directory cmake fiels are installed in")

set(cuda_sources
  cudaImage.cu  
  cudaImage.h  
  cudaSiftH.cu 
  cudaSiftH.h  
  matching.cu  
  cudaSiftD.h  
  cudaSift.h  
  cudautils.h
)  

set(sources
  geomFuncs.cpp  
  mainSift.cpp  
)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR} 
)

#SET(CUDA_SEPARABLE_COMPILATION ON)

add_library(cudasift STATIC
  ${cuda_sources} ${sources}
)

# cuda_add_executable(cudasift ${cuda_sources} ${sources} OPTIONS -arch=sm_35)

#cuda_add_executable(l2net l2netD.cu OPTIONS -arch=sm_35)

set_target_properties(cudasift PROPERTIES
  COMPILE_FLAGS "${EXTRA_CXX_FLAGS}"			   
)

target_link_libraries(cudasift ${CUDA_cudadevrt_LIBRARY} ${OpenCV_LIBS})
#  /usr/local/cuda/lib64/libcudadevrt.a ${OpenCV_LIBS} 
#)

export(PACKAGE ${PROJECT_NAME})
 
install(FILES 
  ${cuda_sources} 
  ${sources}
  cudaSiftD.cu
  CMakeLists.txt
  # Copyright.txt
  LICENSE
  DESTINATION .
)

install(FILES README.md LICENSE
        DESTINATION "${DOC_INSTALL_DIR}"
        COMPONENT doc)

install(FILES cudaImage.h
    # cudaSiftH.h cudaSiftD.h 
    cudaSift.h
    cudautils.h
    DESTINATION "${INCLUDE_INSTALL_DIR}"
    COMPONENT dev)

install(FILES data/left.pgm data/righ.pgm
  DESTINATION data
)

# FIXME: Danny 08-07-2020 add config (Debug/Release)
# install(FILES ${CMAKE_BINARY_DIR}/$<CONFIG>/cudasift.lib
install(FILES ${CMAKE_BINARY_DIR}/Release/cudasift.lib
  DESTINATION lib
)

# TODO: Danny 08-07-2020 add CudaSiftConfig.cmake inside CudaSift
# # cmake-modules
# CONFIGURE_FILE(${PROJECT_NAME}Config.cmake.in
#     ${PROJECT_NAME}Config.cmake
#     @ONLY)
# CONFIGURE_FILE(${PROJECT_NAME}ConfigVersion.cmake.in
#     ${PROJECT_NAME}ConfigVersion.cmake
#     @ONLY)
# INSTALL(FILES
#     ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
#     ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
#     DESTINATION "${CMAKE_INSTALL_DIR}"
#     COMPONENT dev)
