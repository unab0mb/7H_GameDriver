
find_package(GLEW REQUIRED)
find_package(ZLib REQUIRED)
find_package(PNG REQUIRED)
find_package(CONFUSE REQUIRED)
find_package(FFMPEG REQUIRED)
find_package(VGMSTREAM REQUIRED)

if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
	set(RELEASE_TYPE "DEBUG")
else()
	set(RELEASE_TYPE "RELEASE")
endif()

# Include all the source code files
file(GLOB_RECURSE source_files "*.h" "*.c")

include(generate_product_version)
generate_product_version(
    VersionFilesOutputVariable
    NAME "7H_GameDriver"
	ICON "${CMAKE_SOURCE_DIR}/misc/7H.ico"
    VERSION_MAJOR 1
    VERSION_MINOR 0
    VERSION_PATH 0
    VERSION_REVISION 3
)

add_library(7H_GameDriver SHARED ${source_files} ${VersionFilesOutputVariable})
target_include_directories(7H_GameDriver PRIVATE "${CMAKE_SOURCE_DIR}/src")
target_link_libraries(7H_GameDriver
	dbghelp
	GLEW::GLEW
	PNG::PNG
	CONFUSE::CONFUSE
	FFMPEG::AVUtil
	FFMPEG::SWResample
	FFMPEG::AVCodec
	FFMPEG::AVFormat
	FFMPEG::SWScale
	VGMSTREAM::VGMSTREAM
)
target_compile_options(7H_GameDriver
	PRIVATE /D_CRT_SECURE_NO_WARNINGS
	PRIVATE /D${RELEASE_TYPE}
)
target_link_options(7H_GameDriver PRIVATE /PDBALTPATH:7H_GameDriver.pdb)
if(_DLL_VERSION)
	target_compile_options(7H_GameDriver PRIVATE /DVERSION="${_DLL_VERSION}")
endif()

install(TARGETS 7H_GameDriver RUNTIME DESTINATION bin)

if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
	install(FILES $<TARGET_PDB_FILE:ff7_opengl> DESTINATION bin OPTIONAL)
endif()

install(FILES ../misc/7H_GameDriver.cfg DESTINATION bin OPTIONAL)