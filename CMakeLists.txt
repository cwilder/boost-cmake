cmake_minimum_required(VERSION 2.8)
project (libboost)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(SELF_CFLAGS "")
set(SELF_CPPFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

#if(NOT TARGET build_libz)
#	add_subdirectory(libz-cmake)
#endif(NOT TARGET build_libz)
#set(SELF_DEPENDS "${SELF_DEPENDS};build_libz")

set(TOOL_TARGET_NAME "build_libboost")
set(TOOL_TARGET_BUILD_OUTPUT "BUILD_libboost")
set(TOOL_TARGET_COMMENT "build libboost")
set(TOOL_TARGET_SOURCE "boost_1_54_0.tar.gz")
set(TOOL_TARGET_DIR "boost_1_54_0")
set(TOOL_TARGET_PATCH "boost_1_54_0.patch")
set(TOOL_TARGET_BASE_URL "http://downloads.sourceforge.net/project/boost/boost/1.54.0")
set(TOOL_TARGET_INSTALL_PREFIX ${CMAKE_BINARY_DIR})

set(SELF_CPPFLAGS "-I${TOOL_TARGET_INSTALL_PREFIX}/include")
set(SELF_LDFLAGS "-L${TOOL_TARGET_INSTALL_PREFIX}/lib")

set(CONFIG_ADD_OPTIONS "--prefix=\"${TOOL_TARGET_INSTALL_PREFIX}\";cxxflags=\"${SELF_CPPFLAGS};-fPIC;-g\";cflags=\"${SELF_CFLAGS};-fPIC;-g\";include=\"${TOOL_TARGET_INSTALL_PREFIX}/include\";library-path=\"${TOOL_TARGET_INSTALL_PREFIX}/lib\"")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		set(CONFIG_ADD_OPTIONS "${CONFIG_ADD_OPTIONS};--without-context;--without-graph;--without-graph_parallel;--without-mpi;--without-python;--without-wave;variant=release;threading=multi;link=shared;runtime-link=shared;")
		message(${CONFIG_ADD_OPTIONS})
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget ${TOOL_TARGET_BASE_URL}/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_PATCH} && cd ${TOOL_TARGET_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_PATCH} && echo "use patch file: ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_PATCH}" || echo "no patch file"
			COMMAND echo "using gcc : arm : ${target_platform}-g++ \;" >> ${TOOL_TARGET_DIR}/tools/build/v2/user-config.jam
			COMMAND cd ${TOOL_TARGET_DIR} && ./bootstrap.sh
			COMMAND cd ${TOOL_TARGET_DIR} && ./b2 ${CONFIG_ADD_OPTIONS} install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT ${TOOL_TARGET_BUILD_OUTPUT}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE} || wget ${TOOL_TARGET_BASE_URL}/${TOOL_TARGET_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND tar -xzvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_SOURCE}
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_PATCH} && cd ${TOOL_TARGET_DIR} && patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_PATCH} && echo "use patch file: ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_TARGET_PATCH}" || echo "no patch file"
			COMMAND cd ${TOOL_TARGET_DIR} && ./bootstrap.sh
			COMMAND cd ${TOOL_TARGET_DIR} && ./b2 ${CONFIG_ADD_OPTIONS} install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT ${TOOL_TARGET_COMMENT}
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(${TOOL_TARGET_NAME} ALL DEPENDS ${SELF_DEPENDS} ${TOOL_TARGET_BUILD_OUTPUT} )
