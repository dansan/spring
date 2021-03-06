# Place executables and shared libs under "build-dir/",
# instead of under "build-dir/rts/"
# This way, we have the build-dir structure more like the install-dir one,
# which makes testing spring in the builddir easier, eg. like this:
# cd build-dir
# SPRING_DATADIR=$(pwd) ./spring
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")

SET(ENGINE_SRC_ROOT_DIR "${CMAKE_SOURCE_DIR}/rts")


### Assemble defines
SET(USE_MMGR FALSE CACHE BOOL "Use memory manager?")
if (USE_MMGR)
	ADD_DEFINITIONS(-DUSE_MMGR)
endif (USE_MMGR)

SET(TRACE_SYNC FALSE CACHE BOOL "Enable sync tracker")
if (TRACE_SYNC)
	ADD_DEFINITIONS(-DTRACE_SYNC)
endif (TRACE_SYNC)

SET(SYNCDEBUG FALSE CACHE BOOL "Enable sync debugger (needs SYNCCHECK=true)")
if (SYNCDEBUG)
	ADD_DEFINITIONS(-DSYNCDEBUG)
	If     (NOT "${CMAKE_BUILD_TYPE}" STREQUAL "DEBUG2" AND NOT "${CMAKE_BUILD_TYPE}" STREQUAL "DEBUG3")
		Message(FATAL_ERROR "You need CMAKE_BUILD_TYPE set to either DEBUG2 or DEBUG3 for a SYNCDEBUG build")
	EndIf  ()
	If     (NOT SYNCCHECK)
		Message(FATAL_ERROR "You need SYNCCHECK=TRUE for a SYNCDEBUG build")
	EndIf  ()
	If     (NOT TRACE_SYNC)
		Message(WARNING "It is recommended to use TRACE_SYNC=TRUE for a SYNCDEBUG build")
	EndIf  ()
endif (SYNCDEBUG)

# Only used by GML build, but used in builds/GML and lib/gml
SET(USE_GML_DEBUG FALSE CACHE BOOL "Use GML call debugging?")



### Assemble common incude dirs
INCLUDE_DIRECTORIES(BEFORE lib/lua/include)
INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR}/include/AL)
INCLUDE_DIRECTORIES(${SPRING_MINIZIP_INCLUDE_DIR})
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

### Assemble common libraries
Add_Subdirectory(System/Sound)
if    (NO_SOUND)
	ADD_DEFINITIONS(-DNO_SOUND)
endif (NO_SOUND)


FIND_PACKAGE_STATIC(DevIL REQUIRED)
LIST(APPEND engineCommonLibraries ${DEVIL_IL_LIBRARY})

IF    (PREFER_STATIC_LIBS)
	# dependencies of DevIL
	FIND_PACKAGE_STATIC(JPEG REQUIRED)
	FIND_PACKAGE_STATIC(TIFF REQUIRED)
	FIND_PACKAGE_STATIC(PNG REQUIRED)
	LIST(APPEND engineCommonLibraries ${JPEG_LIBRARY} ${TIFF_LIBRARY} ${PNG_LIBRARY})
ENDIF (PREFER_STATIC_LIBS)


### Find include directories and add platform specific libraries
LIST(APPEND engineCommonLibraries ${Boost_REGEX_LIBRARY} ${Boost_THREAD_LIBRARY} ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_SYSTEM_LIBRARY} ${Boost_SIGNALS_LIBRARY})

LIST(APPEND engineCommonLibraries lua luasocket 7zip oscpack ${SPRING_MINIZIP_LIBRARY} streflop lobby assimp)

IF    (PREFER_STATIC_LIBS)
	# dependency of minizip
	FIND_PACKAGE_STATIC(ZLIB REQUIRED)
	LIST(APPEND engineCommonLibraries ${ZLIB_LIBRARY})
ENDIF (PREFER_STATIC_LIBS)

IF    (UNIX)
	# Needed for dynamically loading shared libraries (on some OS)
	LIST(APPEND engineCommonLibraries dl)
ENDIF (UNIX)

IF (MINGW)
	LIST(APPEND engineCommonLibraries ${WIN32_LIBRARIES} mingw32)
ENDIF (MINGW)


### Assemble engine sources
Add_Subdirectory(Game)
Add_Subdirectory(Lua)
Add_Subdirectory(ExternalAI)
Add_Subdirectory(Rendering)
Add_Subdirectory(aGui)
Add_Subdirectory(Menu)
Add_Subdirectory(Map)
Add_Subdirectory(Sim)
#Add_Subdirectory(System) # this is already added in ../

SET(engineSources
		${sources_engine_Game}
		${sources_engine_Lua}
		${sources_engine_Map}
		${sources_engine_Rendering}
		${sources_engine_aGui}
		${sources_engine_Menu}
		${sources_engine_Sim}
		${sources_engine_System}
		${sources_engine_ExternalAI}
		${nedmalloc_obj} # TODO remove? (seems never to be set)
	)

### Add headers for generated project files (e.g. Code::Blocks)
file(GLOB_RECURSE engineHeaders "*.h" "*.hpp")


Add_Subdirectory(builds)
