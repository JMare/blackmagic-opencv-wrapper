#
#    Copyright 2019 Yasuhiro Sugimoto <yas@mech.eng.osaka-u.ac.jp>
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.

cmake_minimum_required(VERSION 2.8)

project(blackmagic-opencv-wrapper)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules")

if (APPLE)
    set (DeckLink_OS_PREFIX "Mac")
    find_library(COCOA_LIBRARY Cocoa)
    set(EXTRA_LIBS ${COCOA_LIBRARY} ${EXTRA_LIBS})
	set(CMAKE_CXX_FLAGS "-O2 -g -arch x86_64 -F/Library/Frameworks -Wno-format -std=c++11 ")
else (APPLE)
    set (DeckLink_OS_PREFIX "Linux")
endif (APPLE)

set(DeckLink_DIR "bmdsdk/")
set(DeckLink_INCLUDE_DIR "${DeckLink_DIR}/${DeckLink_OS_PREFIX}/include")
find_file (DeckLink_EXTRA_CODE_CPP DeckLinkAPIDispatch.cpp PATHS ${DeckLink_INCLUDE_DIR})

if (DeckLink_EXTRA_CODE_CPP)
    set (DeckLink_LIBRARIES BlackMagicDeckLink)
    include_directories (${DeckLink_INCLUDE_DIR})
    add_library (BlackMagicDeckLink STATIC ${DeckLink_EXTRA_CODE_CPP})
endif (DeckLink_EXTRA_CODE_CPP)


set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Boost REQUIRED COMPONENTS thread)

set(OpenCV_SHARED ON)
set(OpenCV_STATIC OFF)
find_package(OpenCV REQUIRED)
find_package(Threads REQUIRED)
set_target_properties(${OpenCV_LIBS} PROPERTIES MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)
set_target_properties(${OpenCV_LIBS} PROPERTIES MAP_IMPORTED_CONFIG_MINSIZEREL RELEASE)

link_directories(${Boost_LIBRARY_DIR_RELEASE} ${Boost_LIBRARY_DIR_DEBUG})

include_directories(${Boost_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} ${DeckLink_INCLUDE_DIR})


add_executable(blackmagic_test
    src/DeckLinkCapture.cpp
    src/DeckLinkInputCallback.cpp
    src/DeckLinkOpenCv.cpp
    src/DeckLinkCreateInstance.cpp
    src/main.cpp
)

target_link_libraries(blackmagic_test
${OpenCV_LIBS}
${DeckLink_LIBRARIES}
${BOOST_Libraries}
${EXTRA_LIBS}
${CMAKE_DL_LIBS}
Threads::Threads)



