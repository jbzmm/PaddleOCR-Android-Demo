# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.22.1)

# Creates and names a library, sets it as either STATIC or SHARED, and provides
# the relative paths to its source code. You can define multiple libraries, and
# CMake builds them for you. Gradle automatically packages shared libraries with
# your APK.

set(PaddleLite_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../PaddleLite")
include_directories(${PaddleLite_DIR}/cxx/include)

set(OpenCV_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../OpenCV/sdk/native/jni")
message(STATUS "opencv dir: ${OpenCV_DIR}")
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
include_directories(${OpenCV_INCLUDE_DIRS})
aux_source_directory(. SOURCES)
set(CMAKE_CXX_FLAGS
        "${CMAKE_CXX_FLAGS} -ffast-math -Ofast -Os -fopenmp"
        )
set(CMAKE_CXX_FLAGS
        "${CMAKE_CXX_FLAGS} -fvisibility=hidden -fvisibility-inlines-hidden -fdata-sections -ffunction-sections"
        )
set(CMAKE_SHARED_LINKER_FLAGS
        "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--gc-sections -Wl,-z,nocopyreloc")

add_library(
        Native
        SHARED
        ${SOURCES})

find_library(
        log-lib
        log)

# 使用静态库（动态库有符号表问题）
set(PADDLE_LITE_LIB_STATIC "${PaddleLite_DIR}/cxx/libs/${ANDROID_ABI}/libpaddle_api_light_bundled.a")

if(EXISTS ${PADDLE_LITE_LIB_STATIC})
    # 使用静态库
    set(PADDLE_LITE_LIBRARIES "${PADDLE_LITE_LIB_STATIC}")
else()
    message(FATAL_ERROR "Paddle Lite static library not found in ${PaddleLite_DIR}/cxx/libs/${ANDROID_ABI}/")
endif()

# Find OpenMP (needed by Paddle Lite static lib)
find_package(OpenMP)
if(OpenMP_CXX_FOUND)
    message(STATUS "OpenMP found: ${OpenMP_CXX_FLAGS}")
else()
    # Fallback: link against the NDK OpenMP library directly
    set(OpenMP_CXX_FLAGS "-fopenmp")
    find_library(OpenMP_LIB NAMES omp gomp HINTS
        "${ANDROID_NDK}/sources/cpufeatures"
        "${ANDROID_NDK}/toolchains/llvm/prebuilt/linux-x86_64/lib64/clang/${ANDROID_PLATFORM_LEVEL}"
    )
    if(NOT OpenMP_LIB)
        # Try to find libomp.so in NDK
        find_library(OpenMP_LIB omp)
    endif()
endif()

target_link_libraries(
        Native
        ${PADDLE_LITE_LIBRARIES}
        ${OpenCV_LIBS}
        GLESv2
        EGL
        jnigraphics
        ${log-lib}
        -fopenmp
        -lm
)
