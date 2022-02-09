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
								  )
								  
set_target_properties(collada-dom PROPERTIES
  COMPILE_FLAGS "${COLLADA_COMPILE_FLAGS}"
  LINK_FLAGS "${libpcrecpp_LDFLAGS_OTHERS}")
  
if(CC_INSTALL_COLLADADOM)
	__install_directory_specif(${CMAKE_CURRENT_SOURCE_DIR}/include/ collada-dom)
	__install_directory_specif(${CMAKE_CURRENT_SOURCE_DIR}/include/1.4/ collada-dom)
endif()
