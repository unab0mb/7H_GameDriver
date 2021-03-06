cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 NEW)

set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /NODEFAULTLIB:MSVCRT /NODEFAULTLIB:MSVCRTD /FORCE:MULTIPLE /MANIFEST:NO /DYNAMICBASE:NO /NXCOMPAT:NO /INCREMENTAL:NO /NOCOFFGRPINFO /LTCG:OFF /RELEASE /SUBSYSTEM:WINDOWS,\"5.01\"")
set(_DLL_VERSION "${_DLL_VERSION}")

if(CMAKE_BUILD_TYPE STREQUAL "Release")
	set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded")
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /NODEFAULTLIB:LIBCMTD")
else()
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /Zi")
	set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDebug")
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /NODEFAULTLIB:LIBCMT /DEBUG /OPT:REF /OPT:ICF")
endif()

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
option(SUPERBUILD "Build the project using a superbuild" ON)

if (SUPERBUILD)
	project(SUPERBUILD)
	set_directory_properties(PROPERTIES EP_BASE "${CMAKE_BINARY_DIR}/ep")
	add_subdirectory(third_party)
	include(ExternalProject)
	ExternalProject_Add(
		7H_GameDriver
		SOURCE_DIR	"${CMAKE_SOURCE_DIR}"
		CMAKE_ARGS
			"-DSUPERBUILD=OFF"
			"-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}"
			"-DCMAKE_PREFIX_PATH=${CMAKE_BINARY_DIR}/vendor"
			"-DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}"
			"-D_DLL_VERSION=${_DLL_VERSION}"
		DEPENDS
		glew
		ffmpeg
		libpng
		libconfuse
		vgmstream
	)
	ExternalProject_Add_Step(
		7H_GameDriver
		reconfigure
		COMMAND ${CMAKE_COMMAND} -E echo "Forcing a superbuild reconfigure"
		DEPENDEES download
		DEPENDERS configure
		ALWAYS ON
	)
	return()
endif()

project(7H_GameDriver)
add_subdirectory(src)

