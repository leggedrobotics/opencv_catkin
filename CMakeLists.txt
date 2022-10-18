cmake_minimum_required(VERSION 3.1)
project(opencv_catkin)

find_package(catkin_simple REQUIRED)

catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

SET(CONTRIB_NAME "opencv_contrib")
ExternalProject_Add(${CONTRIB_NAME}
  # Uncomment below for Melodic
  #URL https://github.com/opencv/opencv_contrib/archive/3.4.11.zip
  # Uncomment below for Noetic
  URL https://github.com/opencv/opencv_contrib/archive/refs/tags/4.6.0.zip
  SOURCE_DIR ${CONTRIB_NAME}_src
  UPDATE_COMMAND ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)

SET(OPENCV_SRC_PATH "opencv_src")
SET(OPENCV_BUILD_PATH "opencv_build")

ExternalProject_Add(opencv_src
  DEPENDS ${CONTRIB_NAME}
  # Uncomment below for Melodic
  #URL https://github.com/opencv/opencv/archive/3.4.11.zip
  # Uncomment below for Noetic
  URL https://github.com/opencv/opencv/archive/refs/tags/4.6.0.zip
  UPDATE_COMMAND ""
  SOURCE_DIR ${OPENCV_SRC_PATH}
  BINARY_DIR ${OPENCV_BUILD_PATH}
  BUILD_ALWAYS 1 # to ensure that changes in the library source are rebuilt
  CONFIGURE_COMMAND cd ../${OPENCV_BUILD_PATH} && cmake
    -DOPENCV_EXTRA_MODULES_PATH=../${CONTRIB_NAME}_src/modules .
    -DBUILD_JAVA=OFF
    -DBUILD_opencv_python2=OFF
    -DBUILD_opencv_python3=ON
    -DWITH_GTK=OFF # OFF for Jetson
    -DWITH_GTK_2_X=ON  # Can't use GTK3 as it links against system protobuf
    -DWITH_PROTOBUF=OFF # Disable custom build of protobuf, since starting with OpenCV 3.4.2 a custom version of
                        # protobuf is always pulled and compiled, even if no enabled modules use it.
    -DWITH_V4L=ON
    -DINSTALL_C_EXAMPLES=OFF
    -DINSTALL_PYTHON_EXAMPLES=OFF
    -DBUILD_EXAMPLES=OFF
    -DWITH_QT=OFF # Needed by clang under Ubuntu 14.04 and GTK_WIDGET(cvGetWindowHandle(...)) with gcc (image_view)
    -DWITH_OPENGL=OFF # OFF for Jetson
    -DWITH_VTK=OFF # OFF for Jetson
    -DWITH_WEBP=OFF
    -DENABLE_PRECOMPILED_HEADERS=OFF
    -DCMAKE_INSTALL_PREFIX=${CATKIN_DEVEL_PREFIX}
    -DBUILD_SHARED_LIBS=ON
    # optimization flags
    -DCV_DISABLE_OPTIMIZATION=OFF
    -DWITH_TBB=OFF
    -DWITH_OPENCL=ON
    -DWITH_OPENMP=ON
    # CUDACMAKE_ARGS "${CMAKE_ARGS}
    -DWITH_CUDA=ON
    -DCUDA_HOST_COMPILER=${CMAKE_C_COMPILER} # this is required to use the chosen compiler
    -DCUDA_ARCH_BIN=${CUDA_ARCH_BIN}
    # tests
    -DBUILD_TESTS=OFF
    -DBUILD_opencv_ts=OFF
    -DBUILD_TESTS=OFF
    # Non free
    -DOPENCV_ENABLE_NONFREE=ON
    # opencv_contrib packages
    -DBUILD_opencv_apps=OFF
    -DBUILD_opencv_dnn=OFF # Pulls in the system protobuf as a dependency!
    -DBUILD_opencv_dnns_easily_fooled=OFF
    -DBUILD_opencv_cnn_3dobj=OFF
    -DBUILD_opencv_aruco=OFF
    -DBUILD_opencv_bgsegm=OFF
    -DBUILD_opencv_bioinspired=OFF
    -DBUILD_opencv_ccalib=OFF
    -DBUILD_opencv_cudaarithm=ON
    -DBUILD_opencv_cudafilters=ON
    -DBUILD_opencv_cudalegacy=ON
    -DBUILD_opencv_cudev=ON
    -DBUILD_opencv_contrib_world=OFF
    -DBUILD_opencv_datasets=OFF
    -DBUILD_opencv_dpm=OFF
    -DBUILD_opencv_face=OFF
    -DBUILD_opencv_fuzzy=OFF
    -DBUILD_opencv_freetype=OFF
    -DBUILD_opencv_hdf=OFF
    -DBUILD_opencv_line_descriptor=OFF
    -DBUILD_opencv_matlab=OFF
    -DBUILD_opencv_optflow=OFF
    -DBUILD_opencv_plot=OFF
    -DBUILD_opencv_reg=OFF
    -DBUILD_opencv_rgbd=OFF
    -DBUILD_opencv_saliency=OFF
    -DBUILD_opencv_sfm=OFF
    -DBUILD_opencv_stereo=OFF
    -DBUILD_opencv_structured_light=OFF
    -DBUILD_opencv_surface_matching=OFF
    -DBUILD_opencv_text=OFF
    -DBUILD_opencv_tracking=OFF
    -DBUILD_opencv_xfeatures2d=ON
    -DBUILD_opencv_ximgproc=ON
    -DBUILD_opencv_xobjdetect=OFF
    -DBUILD_opencv_nonfree=ON
    -DBUILD_opencv_xphoto=ON ../${OPENCV_SRC_PATH}
  BUILD_COMMAND cd ../${OPENCV_BUILD_PATH} && make -j8
  INSTALL_COMMAND cd ../${OPENCV_BUILD_PATH} && make install -j8
)

install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/opencv
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
        FILES_MATCHING PATTERN "*.h"
                       PATTERN "*.hpp")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/opencv2
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
        FILES_MATCHING PATTERN "*.h"
                       PATTERN "*.hpp")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/lib/
        DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        FILES_MATCHING PATTERN "libopencv*")
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/share/OpenCV
        DESTINATION ${CATKIN_GLOBAL_SHARE_DESTINATION})
set(OPENCV_INSTALLED_EXECUTABLES
    opencv_annotation
    opencv_createsamples
    opencv_traincascade
    opencv_version
    opencv_visualisation
)

foreach(installed_file ${OPENCV_INSTALLED_EXECUTABLES})
  # Need to rename binaries as they will conflict with the binaries from
  # the ROS OpenCV package otherwise.
  string(REPLACE opencv opencv_catkin new_file_name ${installed_file} )
  install(FILES ${CATKIN_DEVEL_PREFIX}/bin/${installed_file}
          RENAME ${new_file_name}
          PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
          DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
endforeach()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
          LIBRARIES opencv_core
                    opencv_calib3d
                    opencv_cudaarithm
                    opencv_cudafilters
                    opencv_cudalegacy
                    opencv_cudev
                    opencv_features2d
                    opencv_flann
                    opencv_highgui
                    opencv_imgcodecs
                    opencv_imgproc
                    #opencv_aruco
                    opencv_line_descriptor
                    opencv_ml
                    opencv_objdetect
                    #opencv_phase_unwrapping
                    opencv_photo
                    opencv_rgbd
                    #opencv_saliency
                    opencv_shape
                    opencv_stitching
                    opencv_superres
                    opencv_video
                    opencv_videoio
                    opencv_videostab
                    opencv_viz
                    opencv_xfeatures2d
                    opencv_ximgproc
                    opencv_xphoto
)
