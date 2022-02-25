# Define here the needed parameters
set (COLLADA_DOM_VERSION_MAJOR 2)
set (COLLADA_DOM_VERSION_MINOR 4)
set (COLLADA_DOM_VERSION_PATCH 0)
set (COLLADA_DOM_VERSION ${COLLADA_DOM_VERSION_MAJOR}.${COLLADA_DOM_VERSION_MINOR}.${COLLADA_DOM_VERSION_PATCH})
set (COLLADA_DOM_SOVERSION ${COLLADA_DOM_VERSION_MAJOR}.${COLLADA_DOM_VERSION_MINOR})

message(STATUS "Compiling Collada DOM Version ${COLLADA_DOM_VERSION}")
message(STATUS "Using cmake version ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" )

# http://www.cmake.org/cmake/help/cmake-2.6.html#policy:CMP0002
cmake_policy(SET CMP0002 NEW)
# http://www.cmake.org/cmake/help/cmake-2.6.html#policy:CMP0003
cmake_policy(SET CMP0003 NEW)

# Use, i.e. don't skip the full RPATH for the build tree
set(CMAKE_SKIP_BUILD_RPATH  FALSE)

# When building, don't use the install RPATH already
# (but later on when installing)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

# The RPATH to be used when installing
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}")

# Add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

message(STATUS "installing to ${CMAKE_INSTALL_PREFIX}")

##############################################################################
# Custom CMake options
##############################################################################

option(OPT_COLLADA15 "Enable collada 1.5 compilation" OFF)
option(OPT_COLLADA14 "Enable collada 1.4 compilation" ON)
option(OPT_COMPILE_VIEWER "Enable collada viewer compilation" OFF)
option(OPT_COMPILE_FX "Enable collada fx compilation" OFF)
option(OPT_COMPILE_RT "Enable collada rt compilation" OFF)
option(OPT_BUILD_PACKAGES "Set to ON to generate CPack configuration files and packaging targets" OFF)
option(OPT_BUILD_PACKAGE_DEFAULT "Set to ON to generate a default openrave package that creates symlinks" OFF)
option(OPT_DOUBLE_PRECISION "Use double precision for everything (daeFloat included)" ON)

set(COLLADA_DOM_EXTERNAL_FLAGS)
if( OPT_COLLADA14 )
  set(COLLADA_DOM_EXTERNAL_FLAGS "${COLLADA_DOM_EXTERNAL_FLAGS} -DCOLLADA_DOM_SUPPORT141")
endif()
if( OPT_COLLADA15 )
  set(COLLADA_DOM_EXTERNAL_FLAGS "${COLLADA_DOM_EXTERNAL_FLAGS} -DCOLLADA_DOM_SUPPORT150")
endif()

if( OPT_COMPILE_VIEWER )
  message(WARNING "compiling collada viewer currently not supported")
endif()
if( OPT_COMPILE_FX )
  message(WARNING "compiling collada fx currently not supported")
endif()
if( OPT_COMPILE_RT )
  message(WARNING "compiling collada rt currently not supported")
endif()

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake-modules")

if( CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX )
  add_definitions("-fno-strict-aliasing -Wall")
endif()

if( CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX )
  set(CMAKE_CXX_FLAGS_RELEASE "-O2")# -DNDEBUG -DBOOST_DISABLE_ASSERTS -D_SECURE_SCL=0") # this practically removes all checks making it a very dangerous options to play with
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
  set(CMAKE_CXX_FLAGS_DEBUG "-g -D_DEBUG")
endif()

if( UNIX OR CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX )
  set(STDC_LIBRARY stdc++)
else()
  set(STDC_LIBRARY)
endif()

if( APPLE OR ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  # apple doesn't have 64bit versions of file opening functions, so add them
  add_definitions("-Dfopen64=fopen -Dfseeko64=fseeko -Dfseek64=fseek -Dftell64=ftell -Dftello64=ftello")
endif()

if(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  set(EXTRA_COMPILE_FLAGS "${COLLADA_DOM_EXTERNAL_FLAGS} -DCOLLADA_DOM_NAMESPACE -fPIC")
else()
  set(EXTRA_COMPILE_FLAGS "${COLLADA_DOM_EXTERNAL_FLAGS} -DCOLLADA_DOM_NAMESPACE")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

__assert_target(minizip)
__assert_target(LibXml2)

set(COLLADA_LIBS zlib minizip boost_file_system boost_system pcrecpp_local LibXml2)
set(COLLADA_CORE_LIBS boost_file_system boost_system)

file(GLOB dae_files ${CMAKE_CURRENT_SOURCE_DIR}/src/dae/*.cpp)
file(GLOB libxmlplugin_files ${CMAKE_CURRENT_SOURCE_DIR}/src/modules/LIBXMLPlugin/*.cpp)
file(GLOB stddatabase_files ${CMAKE_CURRENT_SOURCE_DIR}/src/modules/STLDatabase/*.cpp)
file(GLOB stderrplugin_files ${CMAKE_CURRENT_SOURCE_DIR}/src/modules/stdErrPlugin/*.cpp)

set(COLLADA_BASE_SOURCES ${dae_files} ${libxmlplugin_files} ${stddatabase_files} ${stderrplugin_files})
set(COLLADA_COMPILE_FLAGS "${EXTRA_COMPILE_FLAGS}")
# create dynamic libraries
set(COLLADA_COMPILE_FLAGS "${COLLADA_COMPILE_FLAGS} -DDOM_DYNAMIC -DDOM_EXPORT")

if( OPT_COLLADA15 )
  add_subdirectory(src/1.5)
  set(COLLADA_LIBS ${COLLADA_LIBS} colladadom150)
endif()
if( OPT_COLLADA14 )
  add_subdirectory(src/1.4)
  set(COLLADA_LIBS ${COLLADA_LIBS} colladadom141)
endif()

set(INCS ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(LIBS ${COLLADA_LIBS})
set(DEFS DOM_INCLUDE_LIBXML PCRE_STATIC)
__add_real_target(collada-dom dll SOURCE ${COLLADA_BASE_SOURCES}
								  INC ${INCS}
								  LIB ${LIBS}
								  DEF ${DEFS}
								  INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include/ ${CMAKE_CURRENT_SOURCE_DIR}/include/1.4/
								  )
								  
set_target_properties(collada-dom PROPERTIES
  COMPILE_FLAGS "${COLLADA_COMPILE_FLAGS}"
  LINK_FLAGS "${libpcrecpp_LDFLAGS_OTHERS}")
  
if(CC_INSTALL_COLLADADOM)
	__install_directory_specif(${CMAKE_CURRENT_SOURCE_DIR}/include/ collada-dom)
	__install_directory_specif(${CMAKE_CURRENT_SOURCE_DIR}/include/1.4/ collada-dom)
endif()
