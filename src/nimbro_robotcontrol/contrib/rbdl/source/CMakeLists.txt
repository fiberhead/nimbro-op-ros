PROJECT (RBDL)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

LIST( APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake )

INCLUDE_DIRECTORIES ( 
	${CMAKE_CURRENT_SOURCE_DIR}/include
	${CMAKE_CURRENT_BINARY_DIR}/include
)

SET_TARGET_PROPERTIES ( ${PROJECT_EXECUTABLES} PROPERTIES
		LINKER_LANGUAGE CXX
	)

# Find and use the system's Eigen3 library
FIND_PACKAGE (Eigen3 3.0.0)

IF (NOT EIGEN3_FOUND AND NOT RBDL_USE_SIMPLE_MATH)
	MESSAGE (WARNING "Could not find Eigen3 on your system. Install it or use the slower SimpleMath library by enabling RBDL_USE_SIMPLE_MATH.")
ENDIF (NOT EIGEN3_FOUND AND NOT RBDL_USE_SIMPLE_MATH)

IF (EIGEN3_FOUND AND NOT RBDL_USE_SIMPLE_MATH)
	INCLUDE_DIRECTORIES (${EIGEN3_INCLUDE_DIR})
ENDIF (EIGEN3_FOUND AND NOT RBDL_USE_SIMPLE_MATH)

# Perform the proper linking
SET (CMAKE_SKIP_BUILD_RPATH FALSE)
SET (CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
SET (CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
SET (CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# Options
OPTION (BUILD_STATIC "Build the statically linked library" OFF)
OPTION (BUILD_TESTS "Build the test executables" OFF)
OPTION (RBDL_ENABLE_LOGGING "Enable logging (warning: major impact on performance!)" OFF)
OPTION (RBDL_USE_SIMPLE_MATH "Use slow math instead of the fast Eigen3 library (faster compilation)" OFF)
OPTION (RBDL_STORE_VERSION "Enable storing of version information in the library (requires build from valid repository)" OFF)
OPTION (BUILD_ADDON_URDFREADER "Build the (experimental) urdf reader" OFF)
OPTION (BUILD_ADDON_BENCHMARK "Build the benchmarking tool" OFF)
OPTION (BUILD_ADDON_LUAMODEL "Build the lua model reader" OFF)

SET ( RBDL_SOURCES 
	src/rbdl_version.cc
	src/rbdl_mathutils.cc
	src/rbdl_utils.cc
	src/Contacts.cc
	src/Dynamics.cc
	src/Logging.cc
	src/Joint.cc
	src/Model.cc
	src/Kinematics.cc
	)

ADD_LIBRARY ( rbdl SHARED ${RBDL_SOURCES} )

IF (BUILD_ADDON_URDFREADER)
  ADD_SUBDIRECTORY ( addons/urdfreader )
ENDIF (BUILD_ADDON_URDFREADER)

IF (BUILD_ADDON_BENCHMARK)
  ADD_SUBDIRECTORY ( addons/benchmark )
ENDIF (BUILD_ADDON_BENCHMARK)

IF (BUILD_ADDON_LUAMODEL)
  ADD_SUBDIRECTORY ( addons/luamodel )
ENDIF (BUILD_ADDON_LUAMODEL)

IF (BUILD_TESTS)
	ADD_SUBDIRECTORY ( vendor/UnitTest++ )
	INCLUDE_DIRECTORIES ( vendor/UnitTest++/src )
 ADD_SUBDIRECTORY ( tests )
ENDIF (BUILD_TESTS)

IF (BUILD_STATIC)
  ADD_LIBRARY ( rbdl-static STATIC ${RBDL_SOURCES} )
  SET_TARGET_PROPERTIES ( rbdl-static PROPERTIES PREFIX "lib")
  SET_TARGET_PROPERTIES ( rbdl-static PROPERTIES OUTPUT_NAME "rbdl")

	IF (BUILD_ADDON_LUAMODEL)
		TARGET_LINK_LIBRARIES ( rbdl-static
			rbdl_luamodel-static
			)
	ENDIF (BUILD_ADDON_LUAMODEL)

	INSTALL (TARGETS rbdl-static
	  LIBRARY DESTINATION lib
  	ARCHIVE DESTINATION lib
	)
ENDIF (BUILD_STATIC)

# Installing
INSTALL (TARGETS rbdl
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
	)

IF (RBDL_STORE_VERSION)
	# Set versioning information that can be queried during runtime
	EXEC_PROGRAM("hg" ${CMAKE_CURRENT_SOURCE_DIR} ARGS "id -i"
		OUTPUT_VARIABLE RBDL_BUILD_REVISION)
	EXEC_PROGRAM("hg" ${CMAKE_CURRENT_SOURCE_DIR} ARGS "branch"
		OUTPUT_VARIABLE RBDL_BUILD_BRANCH)
	SET (RBDL_BUILD_TYPE ${CMAKE_BUILD_TYPE})
ELSE (RBDL_STORE_VERSION)
	SET (RBDL_BUILD_REVISION "unknown")
	SET (RBDL_BUILD_BRANCH "unknown")
	SET (RBDL_BUILD_TYPE "unknown")
ENDIF (RBDL_STORE_VERSION)

CONFIGURE_FILE (
	"${CMAKE_CURRENT_SOURCE_DIR}/include/rbdl/rbdl_config.h.cmake" 
	"${CMAKE_CURRENT_BINARY_DIR}/include/rbdl/rbdl_config.h"
	)

FILE ( GLOB headers 
	${CMAKE_CURRENT_SOURCE_DIR}/include/rbdl/*.h
	${CMAKE_CURRENT_BINARY_DIR}/include/rbdl//rbdl_config.h
	)

INSTALL ( FILES ${headers} DESTINATION include/rbdl )

# Setup of SimpleMath install settings
IF (RBDL_USE_SIMPLE_MATH)
	INSTALL ( DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/rbdl/SimpleMath"
		DESTINATION include/rbdl
	)
ENDIF (RBDL_USE_SIMPLE_MATH)

