project(zxing)
cmake_minimum_required(VERSION 2.8.0)

set(CMAKE_LIBRARY_PATH /opt/local/lib ${CMAKE_LIBRARY_PATH})


# Check for polluted source tree.
if(EXISTS ${CMAKE_SOURCE_DIR}/CMakeCache.txt OR
    EXISTS ${CMAKE_SOURCE_DIR}/CMakeFiles)
    message(FATAL_ERROR
        "Source directory is polluted:"
        "\n  * remove CMakeCache.txt"
        "\n  * remove CMakeFiles directory")
endif()

# Suppress in-source builds.
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    message(FATAL_ERROR
        "CMake generation is not allowed within the source directory:"
        "\n  * mkdir build"
        "\n  * cd build"
        "\n  * Unix-like: cmake -G \"Unix Makefiles\" .."
        "\n  * Windows: cmake -G \"Visual Studio 10\" ..")
endif()

# Adjust CMake's module path.
set(CMAKE_MODULE_PATH ./src/main/cpp/cmake/)


# Add libzxing library.
file(GLOB_RECURSE LIBZXING_FILES
    "./src/main/cpp/zxing/*.cpp"
    "./src/main/cpp/zxing/*.h"
    "./src/main/cpp/zxing/*.cc"
    "./src/main/cpp/zxing/*.hh"
)

# OpenCV classes
find_package(OpenCV)
if(OpenCV_FOUND)
    file(GLOB_RECURSE LIBZXING_OPENCV_FILES
        "./src/main/cpp/opencv/src/*.cpp"
        "./src/main/cpp/opencv/src/*.h"
    )
    set(LIBZXING_FILES ${LIBZXING_FILES} ${LIBZXING_OPENCV_FILES})
    include_directories(${OpenCV_INCLUDE_DIRS})
    include_directories(".src/main/cpp/opencv/src/")
endif()

include_directories("./src/main/cpp/")
add_library(libzxing SHARED ${LIBZXING_FILES})
set_target_properties(libzxing PROPERTIES PREFIX "")

find_package(Iconv)
if(ICONV_FOUND)
    include_directories(${ICONV_INCLUDE_DIR})
    target_link_libraries(libzxing ${ICONV_LIBRARIES})
else()
    add_definitions(-DNO_ICONV=1)
endif()

# Add cli executable.
file(GLOB_RECURSE ZXING_FILES
    "./src/main/cpp/cli/src/*.cpp"
    "./src/main/cpp/cli/src/*.h"
)

add_executable(zxing ${ZXING_FILES})

target_link_libraries(zxing libzxing)
install(TARGETS zxing libzxing
	LIBRARY DESTINATION lib
	RUNTIME DESTINATION bin
	ARCHIVE DESTINATION lib)
install(DIRECTORY src/main/cpp/zxing/ DESTINATION include/zxing FILES_MATCHING PATTERN "*.h")