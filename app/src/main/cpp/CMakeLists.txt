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
        "${CMAKE_CXX_FLAGS} -ffast-math -Ofast -Os"
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

# 使用动态库 (如果存在)
set(PADDLE_LITE_LIB "${PaddleLite_DIR}/cxx/libs/${ANDROID_ABI}/libpaddle_light_api_shared.so")
set(PADDLE_LITE_LIB_STATIC "${PaddleLite_DIR}/cxx/libs/${ANDROID_ABI}/libpaddle_api_light_bundled.a")

if(EXISTS ${PADDLE_LITE_LIB})
    add_library(paddle_light_api_shared SHARED IMPORTED)
    set_target_properties(paddle_light_api_shared PROPERTIES IMPORTED_LOCATION ${PADDLE_LITE_LIB})
    set(PADDLE_LITE_LIBRARIES paddle_light_api_shared)
elseif(EXISTS ${PADDLE_LITE_LIB_STATIC})
    # 使用静态库
    set(PADDLE_LITE_LIBRARIES "${PADDLE_LITE_LIB_STATIC}")
else()
    message(FATAL_ERROR "Paddle Lite library not found in ${PaddleLite_DIR}/cxx/libs/${ANDROID_ABI}/")
endif()

target_link_libraries(
        Native
        ${PADDLE_LITE_LIBRARIES}
        ${OpenCV_LIBS}
        GLESv2
        EGL
        jnigraphics
        ${log-lib}
)
