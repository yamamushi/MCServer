
cmake_minimum_required (VERSION 2.6)
project (MCServer)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++03")
endif()

include_directories (SYSTEM "${PROJECT_SOURCE_DIR}/../lib/")
include_directories (SYSTEM "${PROJECT_SOURCE_DIR}/../lib/jsoncpp/include")

set(FOLDERS OSSupport HTTPServer Bindings Items Blocks Protocol Generating) 
set(FOLDERS ${FOLDERS} WorldStorage Mobs Entities Simulator UI BlockEntities)



if (NOT MSVC)
	foreach(folder ${FOLDERS})
		add_subdirectory(${folder})
	endforeach(folder)

	file(GLOB SOURCE
		"*.cpp"
	)

	list(REMOVE_ITEM SOURCE "${PROJECT_SOURCE_DIR}/StackWalker.cpp" "${PROJECT_SOURCE_DIR}/LeakFinder.cpp")

else ()

	function(includefolder PATH)
		FILE(GLOB FOLDER_FILES
			"${PATH}/*.cpp"
			"${PATH}/*.h"
		)
		source_group("${PATH}" FILES ${FOLDER_FILES})
	endfunction(includefolder)

	foreach(folder ${FOLDERS})
		includefolder(${folder})
	endforeach(folder)

	file(GLOB_RECURSE SOURCE
		"*.cpp"
		"*.h"
	)

	include_directories("${PROJECT_SOURCE_DIR}")

	source_group("" FILES ${SOURCE})

	# Precompiled headers (1st part)
	SET_SOURCE_FILES_PROPERTIES(
		Globals.cpp PROPERTIES COMPILE_FLAGS "/Yc\"Globals.h\""
	)
	# CMake cannot "remove" the precompiled header flags, so we use a dummy precompiled header compatible with just this one file:
	SET_SOURCE_FILES_PROPERTIES(
		Bindings/Bindings.cpp PROPERTIES COMPILE_FLAGS "/Yc\"string.h\" /Fp\"$(IntDir)/Bindings.pch\""
	)
	SET_SOURCE_FILES_PROPERTIES(
		"StackWalker.cpp LeakFinder.h" PROPERTIES COMPILE_FLAGS "/Yc\"Globals.h\""
	)
endif()


set(EXECUTABLE MCServer)

add_executable(${EXECUTABLE} ${SOURCE})

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/MCServer)

if (MSVC)
	# MSVC generator adds a "Debug" or "Release" postfixes to the EXECUTABLE_OUTPUT_PATH, we need to cancel them:
	SET_TARGET_PROPERTIES(${EXECUTABLE} PROPERTIES PREFIX "../")
	SET_TARGET_PROPERTIES(${EXECUTABLE} PROPERTIES IMPORT_PREFIX "../")
endif()


# Make the debug executable have a "_debug" suffix
SET_TARGET_PROPERTIES(${EXECUTABLE} PROPERTIES DEBUG_POSTFIX "_debug")


# Precompiled headers (2nd part)
if (MSVC)
	SET_TARGET_PROPERTIES(
		${EXECUTABLE} PROPERTIES COMPILE_FLAGS "/Yu\"Globals.h\""
		OBJECT_DEPENDS "$(IntDir)/$(TargetName.pch)"
	)
endif ()


if (NOT MSVC)
	target_link_libraries(${EXECUTABLE} OSSupport HTTPServer Bindings Items Blocks)
	target_link_libraries(${EXECUTABLE} Protocol Generating WorldStorage)
	target_link_libraries(${EXECUTABLE} Mobs Entities Simulator UI BlockEntities)
endif ()
if (WIN32)
	target_link_libraries(${EXECUTABLE} expat tolualib ws2_32.lib Psapi.lib)
endif()
target_link_libraries(${EXECUTABLE} md5 luaexpat iniFile jsoncpp cryptopp zlib lua sqlite)
