cmake_minimum_required(VERSION 3.5)
project(BioSemi VERSION 1.12.0
	LANGUAGES CXX)
include(Findliblsl.cmake)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
find_package(Qt5 REQUIRED COMPONENTS Widgets)

# Boost
find_package(Boost REQUIRED COMPONENTS thread)

SET_SOURCE_FILES_PROPERTIES(
  "DLL/bsif_libusb_linux64.o"
  PROPERTIES
  EXTERNAL_OBJECT true
  GENERATED true
)

# APPLICATION #
set(target BioSemi)
add_executable(${target}
	MACOSX_BUNDLE
	WIN32
	main.cpp
	mainwindow.cpp
	mainwindow.h
	mainwindow.ui
	biosemi_io.cpp
	biosemi_io.h
	Resampler.h
	DLL/labview_dll.cpp
	DLL/labview_dll.h
	DLL/bsif.h
	DLL/bsif_libusb_linux64.o
)

target_link_libraries(${target}
	PRIVATE
	Qt5::Widgets
	Boost::thread
	LSL::lsl
)

if(APPLE)
	target_link_libraries(${target} PRIVATE "DLL/liblabview_dll.0.0.1.dylib")
elseif(UNIX)
	find_library(LIBUSB usb-1.0)
	if(NOT LIBUSB)
		message(FATAL_ERROR "Didn't find libusb. Please install libusb-1.0.0-dev")
	endif()
	target_link_libraries(${target} PRIVATE ${LIBUSB})
	#target_link_libraries(${target} PRIVATE "DLL/bsif_libusb.o")
endif()

# Deployment # 
installLSLApp(${target})  # Will handle Qt and LSL libraries
installLSLAuxFiles(${target} biosemi_config.cfg)
installLSLAuxFiles(${target} "directory" DLL)
LSLGenerateCPackConfig()
