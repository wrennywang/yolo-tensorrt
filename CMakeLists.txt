cmake_minimum_required(VERSION 3.1) 
project(yolov3-trt)            

file(GLOB_RECURSE sources     	modules/*.hpp modules/*.cpp modules/*.h modules/*.cu extra/*.h)
add_executable(yolov3-trt samples/sample_detector.cpp)

#find_Package(gflags REQUIRED)
#if(gflags_FOUND)
#message("-- gflags found")
#endif()

set(CMAKE_CXX_COMPILIER "/usr/bin/g++")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wno-write-strings -lstdc++fs")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath -Wl,$ORIGIN")

#cuda
find_package(CUDA REQUIRED)

message("-- CUDA version:${CUDA_VERSION}")

if(CUDA_VERSION_MAJOR GREATER 9)
	message("-- CUDA ${CUDA_VERSION_MAJOR} detected")
		set(
		CUDA_NVCC_FLAGS
		${CUDA_NVCC_FLAGS}; 
		-gencode arch=compute_50,code=sm_50;
		-gencode arch=compute_52,code=sm_52;
		-gencode arch=compute_53,code=sm_53;
		-gencode arch=compute_60,code=sm_60;
		-gencode arch=compute_61,code=sm_61;
		-gencode arch=compute_62,code=sm_62;
		-gencode arch=compute_70,code=sm_70; 
		-gencode arch=compute_72,code=sm_72;
		-gencode arch=compute_75,code=sm_75
		)
endif()

find_package(OpenCV REQUIRED)
if(Opencv_FOUND)
	message("found opencv")
endif()

#generate detector lib
cuda_add_library(detector SHARED ${sources})
target_include_directories(detector PRIVATE extra/ modules/ ${OpenCV_INCLUDE_DIRS} ${CUDA_TOOLKIT_ROOT_DIR}/include ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})
target_link_libraries(detector nvinfer nvinfer_plugin nvcaffe_parser gflags "stdc++fs")
				  
target_include_directories(yolov3-trt PRIVATE modules/ extra/)
target_link_libraries(yolov3-trt detector opencv_highgui opencv_imgproc opencv_core opencv_calib3d opencv_dnn)



