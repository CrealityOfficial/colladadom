file(GLOB dom_files ${CMAKE_CURRENT_SOURCE_DIR}/dom/*.cpp)
__add_real_target(colladadom150 lib SOURCE ${dom_files}
									LIB ${COLLADA_CORE_LIBS}
									)

set_target_properties(colladadom150 PROPERTIES COMPILE_FLAGS "${COLLADA_COMPILE_FLAGS}")
