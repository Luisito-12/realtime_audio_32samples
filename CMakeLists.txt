cmake_minimum_required(VERSION 3.20)
project(RealTimeAudioEngine VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Banderas de optimización para Tiempo Real
if(MSVC)
    add_compile_options(/O2 /Oi /Ot /Gy /fp:fast /W4)
    add_compile_definitions(_CRT_SECURE_NO_WARNINGS)
else()
    add_compile_options(-O3 -march=native -ffast-math -Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif()

# Búsqueda de PortAudio
find_package(PkgConfig QUIET)
if(PkgConfig_FOUND)
    pkg_check_modules(PORTAUDIO portaudio-2.0)
endif()

if(NOT PORTAUDIO_FOUND)
    find_path(PORTAUDIO_INCLUDE_DIR portaudio.h)
    find_library(PORTAUDIO_LIBRARY NAMES portaudio portaudio_x64 portaudio_x86)
    if(PORTAUDIO_INCLUDE_DIR AND PORTAUDIO_LIBRARY)
        set(PORTAUDIO_FOUND TRUE)
        set(PORTAUDIO_INCLUDE_DIRS ${PORTAUDIO_INCLUDE_DIR})
        set(PORTAUDIO_LIBRARIES ${PORTAUDIO_LIBRARY})
    endif()
endif()

if(NOT PORTAUDIO_FOUND)
    find_package(PortAudio REQUIRED)
endif()

set(SOURCES
    src/main.cpp
    src/AudioEngine.cpp
)

set(HEADERS
    include/AudioEngine.h
    include/LockFreeRingBuffer.h
)

add_executable(RealTimeAudioEngine ${SOURCES} ${HEADERS})

target_include_directories(RealTimeAudioEngine PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${PORTAUDIO_INCLUDE_DIRS}
)

target_link_libraries(RealTimeAudioEngine PRIVATE 
    ${PORTAUDIO_LIBRARIES}
)

# Librerías de sistema operativo
if(WIN32)
    target_link_libraries(RealTimeAudioEngine PRIVATE winmm setupapi ole32 avrt)
elseif(APPLE)
    find_library(COREAUDIO_LIB CoreAudio REQUIRED)
    find_library(AUDIOUNIT_LIB AudioUnit REQUIRED)
    find_library(CORESERVICES_LIB CoreServices REQUIRED)
    target_link_libraries(RealTimeAudioEngine PRIVATE ${COREAUDIO_LIB} ${AUDIOUNIT_LIB} ${CORESERVICES_LIB})
elseif(UNIX)
    find_package(Threads REQUIRED)
    target_link_libraries(RealTimeAudioEngine PRIVATE Threads::Threads asound pthread)
endif()
