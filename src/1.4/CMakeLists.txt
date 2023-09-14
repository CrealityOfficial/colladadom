file(GLOB dom_files ${CMAKE_CURRENT_SOURCE_DIR}/dom/*.cpp)
__add_real_target(colladadom141 lib SOURCE ${dom_files}
									LIB ${COLLADA_CORE_LIBS} colladadom_static
									)
