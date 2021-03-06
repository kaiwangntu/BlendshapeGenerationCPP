cmake_minimum_required(VERSION 2.8.11)

project(BlendshapeGeneration)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Boost
find_package(Boost COMPONENTS filesystem timer program_options REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
link_libraries(${Boost_LIBRARIES} -lboost_filesystem -lboost_system)

# OpenMP
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")

# OpenCV
find_package( OpenCV REQUIRED )
message("-- ${OPENCV_LIBS}")
link_libraries(${OpenCV_LIBS})

# OpenGL
find_package(OpenGL REQUIRED)
find_package(GLUT REQUIRED)
include_directories( ${OPENGL_INCLUDE_DIRS}  ${GLUT_INCLUDE_DIRS} )
link_libraries(${OPENGL_LIBRARIES} ${GLUT_LIBRARY})

find_package(GLEW REQUIRED)
if (GLEW_FOUND)
    include_directories(${GLEW_INCLUDE_DIRS})
    link_libraries(${GLEW_LIBRARIES})
endif()

# GLI
#find_package(GLI REQUIRED)
#if (GLI_FOUND)
#    message([] ${GLI_INCLUDE_DIRS})
#    include_directories(${GLI_INCLUDE_DIRS})
#endif()

# Eigen
find_package(Eigen)
include_directories(${EIGEN3_INCLUDE_DIRS})
message("--   Eigen include path is ${EIGEN3_INCLUDE_DIRS}")

# SuiteSparse
set(SUITESPARSE_INCLUDE_DIR "/usr/include/suitesparse")
include_directories(${SUITESPARSE_INCLUDE_DIR})

# MKL
set(MKL_INCLUDE_DIRS "/opt/intel/mkl/include")
message([] ${MKL_INCLUDE_DIRS} )
include_directories(${MKL_INCLUDE_DIRS})
set(MKLROOT "/opt/intel/mkl")
set(MKLLIBS_DIRS "${MKLROOT}/lib/intel64/" "/opt/intel/lib/intel64_lin")
link_directories(${MKLLIBS_DIRS})
set(MKLLIBS "-Wl,--start-group -lmkl_intel_lp64 -lmkl_core -lmkl_intel_thread -Wl,--end-group -liomp5 -ldl -lpthread -lm")

# PhGLib
include_directories("$ENV{HOME}/SDKs/PhGLib/include")
link_directories("$ENV{HOME}/SDKs/PhGLib/lib")
set(PhGLib "-lPhGLib")

# Qt5
find_package(Qt5Core)
find_package(Qt5Widgets)
find_package(Qt5OpenGL)

# Ceres solver
find_package(Ceres REQUIRED)
include_directories(${CERES_INCLUDE_DIRS})
link_libraries(${CERES_LIBRARIES})

include_directories("${CMAKE_CURRENT_LIST_DIR}/MultilinearReconstruction/third_party/json/include")


add_library(meshdeformer meshdeformer.cpp)
target_link_libraries(meshdeformer
  Qt5::Core
  Qt5::Widgets
  Qt5::OpenGL
  CGAL
  ${MKLLIBS}
  ${OPENCV_LIBS}
  ${PhGLib}
  basicmesh
  ioutilities
  multilinearmodel
  reporter
  offscreenmeshvisualizer
)

add_library(meshtransferer meshtransferer.cpp)
target_link_libraries(meshtransferer
  Qt5::Core
  Qt5::Widgets
  Qt5::OpenGL
  CGAL
  ${MKLLIBS}
  ${OPENCV_LIBS}
  ${PhGLib}
  basicmesh
  ioutilities
  multilinearmodel
  reporter
  offscreenmeshvisualizer
)

# Face shape from shading program
add_executable(BlendshapeGeneration main.cpp
               blendshapegeneration.cpp
               interactiverigging.cpp
               blendshapesweightswidget.cpp
               blendshapesweightswidget.h
               pointcloud.cpp blendshaperefiner.cpp blendshaperefiner.h blendshaperefiner_old.cpp blendshaperefiner_old.h)

target_link_libraries(BlendshapeGeneration
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      CGAL
                      ${MKLLIBS}
                      ${OPENCV_LIBS}
                      ${PhGLib}
                      basicmesh
                      ioutilities
                      multilinearmodel
                      reporter
                      meshdeformer
                      meshtransferer)

add_executable(compute_details compute_details.cpp)
target_link_libraries(compute_details
        Qt5::Core
        Qt5::Widgets
        Qt5::OpenGL
        CGAL
        ${MKLLIBS}
        ${PhGLib}
        basicmesh
        ioutilities
        multilinearmodel
        meshdeformer)

add_executable(laplaciandeformation laplaciandeformation.cpp)
target_link_libraries(laplaciandeformation
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      CGAL
                      ${MKLLIBS}
                      ${PhGLib}
                      basicmesh
                      ioutilities
                      multilinearmodel
                      meshdeformer)

add_executable(deformationtransfer deformationtransfer.cpp)
target_link_libraries(deformationtransfer
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      CGAL
                      ${MKLLIBS}
                      ${PhGLib}
                      basicmesh
                      ioutilities
                      multilinearmodel
                      meshtransferer)

add_executable(expressiontransfer expressiontransfer.cpp)
target_link_libraries(expressiontransfer
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      CGAL
                      ${MKLLIBS}
                      ${PhGLib}
                      basicmesh
                      ioutilities
                      multilinearmodel
                      meshdeformer
                      meshtransferer
                      offscreenmeshvisualizer
                      meshvisualizer)

add_executable(meshfitter meshfitter.cpp)
target_link_libraries(meshfitter
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      CGAL
                      ${MKLLIBS}
                      ${PhGLib}
                      basicmesh
                      ioutilities
                      multilinearmodel
                      meshdeformer
                      meshtransferer
                      offscreenmeshvisualizer
                      meshvisualizer)

add_executable(blendshapes_driver blendshapes_driver.cpp)
target_link_libraries(blendshapes_driver
                      Qt5::Core
                      Qt5::Widgets
                      Qt5::OpenGL
                      CGAL
                      ${MKLLIBS}
                      ${PhGLib}
                      basicmesh
                      ioutilities
                      multilinearmodel
                      meshdeformer
                      meshtransferer
                      offscreenmeshvisualizer
                      meshvisualizer)

link_directories(MultilinearReconstruction)

add_subdirectory(MultilinearReconstruction)
