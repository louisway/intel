# ubuntu 12.04 LTS cmake version 2.8.7
# ubuntu 14.04 LTS cmake version 2.8.12.2
# ubuntu 16.04 LTS cmake version 3.5.1
cmake_minimum_required(VERSION 2.8.3)

project(RealsenseExamples)

# Save the command line compile commands in the build output
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
# View the makefile commands during build
#set(CMAKE_VERBOSE_MAKEFILE on)

# This parameter is meant for disabling graphical examples when building for
# headless targets.
option(BUILD_GRAPHICAL_EXAMPLES "Build graphical examples." ON)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

if(ANDROID)
    set(DEPENDENCIES realsense2)

    add_executable(cpp-enumerate-devices cpp-enumerate-devices.cpp)
    target_link_libraries(cpp-enumerate-devices ${DEPENDENCIES})
    include_directories(cpp-enumerate-devices third_party/tclap/include)

    set_target_properties (cpp-enumerate-devices PROPERTIES
        FOLDER Examples
    )
else()
    if(BUILD_GRAPHICAL_EXAMPLES)
        find_package(OpenGL REQUIRED)
        set(DEPENDENCIES realsense2 ${OPENGL_LIBRARIES})

        if(WIN32)
            add_subdirectory(third_party/glfw)
            list(APPEND DEPENDENCIES glfw3)
        else()
            # Find glfw header
            find_path(GLFW_INCLUDE_DIR NAMES GLFW/glfw3.h
                PATHS /usr/X11R6/include
                      /usr/include/X11
                      /opt/graphics/OpenGL/include
                      /opt/graphics/OpenGL/contrib/libglfw
                      /usr/local/include
                      /usr/include/GL
                      /usr/include
            )
            # Find glfw library
            find_library(GLFW_LIBRARIES NAMES glfw glfw3
                    PATHS /usr/lib64
                          /usr/lib
                          /usr/lib/${CMAKE_LIBRARY_ARCHITECTURE}
                          /usr/local/lib64
                          /usr/local/lib
                          /usr/local/lib/${CMAKE_LIBRARY_ARCHITECTURE}
                          /usr/X11R6/lib
            )
            list(APPEND DEPENDENCIES m ${GLFW_LIBRARIES} ${LIBUSB1_LIBRARIES})
            include_directories(${GLFW_INCLUDE_DIR})
        endif()
    else()
        set(DEPENDENCIES realsense2)
        if(NOT WIN32)
            list(APPEND DEPENDENCIES m ${LIBUSB1_LIBRARIES})
        endif()
    endif()


    # C/C++ Headless Tutorials
    find_package( OpenCV REQUIRED )

    # c-tutorial-1-depth
    add_executable(c-tutorial-1-depth c-tutorial-1-depth.c)
    target_link_libraries(c-tutorial-1-depth ${DEPENDENCIES})
    set_target_properties (c-tutorial-1-depth PROPERTIES
        FOLDER "C Tutorials"
    )

    # cpp-tutorial-1-depth
    add_executable(cpp-tutorial-1-depth cpp-tutorial-1-depth.cpp)
    target_link_libraries(cpp-tutorial-1-depth ${DEPENDENCIES})
    set_target_properties (cpp-tutorial-1-depth PROPERTIES
        FOLDER "C++ Tutorials"
    )

    # data-collect
    add_executable(cpp-data-collect cpp-data-collect.cpp)
    target_link_libraries(cpp-data-collect ${DEPENDENCIES})
    include_directories(cpp-data-collect third_party/tclap/include)
    set_target_properties (cpp-data-collect PROPERTIES
        FOLDER "Examples"
    )

    # headless
    add_executable(cpp-headless cpp-headless.cpp)
    target_link_libraries(cpp-headless ${DEPENDENCIES})
    include_directories(cpp-headless third_party/tclap/include)
    set_target_properties (cpp-headless PROPERTIES
        FOLDER "Examples"
    )

    # enumerate-devices
    add_executable(cpp-enumerate-devices cpp-enumerate-devices.cpp)
    target_link_libraries(cpp-enumerate-devices ${DEPENDENCIES})
    include_directories(cpp-enumerate-devices third_party/tclap/include)
    set_target_properties (cpp-enumerate-devices PROPERTIES
        FOLDER Examples
    )

    # terminal
    add_executable(cpp-terminal cpp-terminal.cpp cpp-terminal-helpers/auto-complete/auto-complete.cpp cpp-terminal-helpers/auto-complete/auto-complete.h)
    if(WIN32)
        target_link_libraries(cpp-terminal ${DEPENDENCIES})
    else()
        target_link_libraries(cpp-terminal -lpthread ${DEPENDENCIES})
    endif()
    include_directories(cpp-terminal third_party/tclap/include cpp-terminal-helpers/auto-complete)
    set_target_properties (cpp-terminal PROPERTIES
        FOLDER Examples
    )

    # fw-logger
    add_executable(cpp-fw-logger cpp-fw-logger.cpp cpp-fw-logger-helpers/fw-log-data.cpp
        cpp-fw-logger-helpers/fw-log-data.h
        cpp-fw-logger-helpers/fw-logs-formating-options.cpp
        cpp-fw-logger-helpers/fw-logs-formating-options.h
        cpp-fw-logger-helpers/fw-logs-parser.cpp
        cpp-fw-logger-helpers/fw-logs-parser.h
        cpp-fw-logger-helpers/fw-logs-xml-helper.cpp
        cpp-fw-logger-helpers/fw-logs-xml-helper.h
        cpp-fw-logger-helpers/string-formatter.cpp
        cpp-fw-logger-helpers/string-formatter.h)
    if(WIN32)
        target_link_libraries(cpp-fw-logger ${DEPENDENCIES})
    else()
        target_link_libraries(cpp-fw-logger -lpthread ${DEPENDENCIES})
    endif()
    include_directories(cpp-fw-logger third_party/tclap/include cpp-fw-logger-helpers)
    set_target_properties (cpp-fw-logger PROPERTIES
        FOLDER Examples
    )

    install(
        TARGETS

        c-tutorial-1-depth
        cpp-tutorial-1-depth

        cpp-enumerate-devices
        cpp-terminal
        cpp-fw-logger
        cpp-data-collect
        cpp-headless

        RUNTIME DESTINATION
        ${CMAKE_INSTALL_PREFIX}/bin
    )

    if(BUILD_GRAPHICAL_EXAMPLES)
        # C/C++ Tutorials

        # c-tutorial-2-streams
        add_executable(c-tutorial-2-streams c-tutorial-2-streams.c)
        target_link_libraries(c-tutorial-2-streams ${DEPENDENCIES})
        set_target_properties (c-tutorial-2-streams PROPERTIES
            FOLDER "C Tutorials"
        )

        # config-ui
        add_executable(cpp-config-ui cpp-config-ui.cpp third_party/imgui/imgui.cpp third_party/imgui/imgui_draw.cpp third_party/imgui/imgui_impl_glfw.cpp example.hpp)
        find_library(GLOG_LIB glog /usr/local/lib)
        find_library(CUDA_LIB cuda /usr/local/cuda/lib64)
        find_library(CAFFE_LIB caffe ~/caffe/build/lib)
        find_library(THREAD_LIB pthread /lib32)
        find_library(BOOST_LIB boost_system  /usr/lib/x86_64-linux-gnu)
        target_link_libraries(cpp-config-ui ${DEPENDENCIES}; ${OpenCV_LIBS}; ${CAFFE_LIB};${GLOG_LIB};${THREAD_LIB};${CUDA_LIB};${BOOST_LIB})
        include_directories(cpp-config-ui third_party/imgui; ~/caffe/include/)
        link_directories(cpp-config-ui ~/caffe/lib)
        set_target_properties (cpp-config-ui PROPERTIES
            FOLDER Examples
        )

        # capture
        add_executable(cpp-capture cpp-capture.cpp)
        target_link_libraries(cpp-capture ${DEPENDENCIES})
        include_directories(cpp-capture third_party/tclap/include)
        set_target_properties (cpp-capture PROPERTIES
            FOLDER "Examples"
        )

        # tutorial-2-streams
        add_executable(cpp-tutorial-2-streams cpp-tutorial-2-streams.cpp)
        target_link_libraries(cpp-tutorial-2-streams ${DEPENDENCIES})
        set_target_properties (cpp-tutorial-2-streams PROPERTIES
            FOLDER "C++ Tutorials"
        )

        # multicam
        add_executable(cpp-multicam cpp-multicam.cpp)
        target_link_libraries(cpp-multicam ${DEPENDENCIES})
        set_target_properties (cpp-multicam PROPERTIES
            FOLDER Examples
        )

        # pointcloud
        add_executable(cpp-pointcloud cpp-pointcloud.cpp third_party/imgui/imgui.cpp third_party/imgui/imgui_draw.cpp third_party/imgui/imgui_impl_glfw.cpp)
        target_link_libraries(cpp-pointcloud ${DEPENDENCIES})
        include_directories(cpp-config-ui third_party/imgui)
        set_target_properties (cpp-pointcloud PROPERTIES
            FOLDER Examples
        )

        install(
            TARGETS

            c-tutorial-2-streams
            cpp-tutorial-2-streams

            cpp-config-ui
            cpp-capture
            cpp-multicam
            cpp-pointcloud

            RUNTIME DESTINATION
            ${CMAKE_INSTALL_PREFIX}/bin
        )
    endif()
endif()
