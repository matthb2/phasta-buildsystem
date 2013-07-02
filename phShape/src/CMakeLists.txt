file(GLOB SRC_FILES *.cc *.f *.f90 *.c)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_library(phshape ${SRC_FILES})

option(PHSHAPE_USE_FMDB "Use FMDB with phShape" OFF)
if(PHSHAPE_USE_FMDB)
	find_package(FMDB REQUIRED)
	include_directories(${FMDB_INCLUDE_DIR})
	add_definitions(-DFMDB)
	target_link_libraries(phshape ${FMDB_LIBRARIES})
	#FMDB, SCORECUtil, SCORECModel?
endif(PHSHAPE_USE_FMDB)
option(PHSHAPE_USE_MESHSIM "Use MeshSim with phShape" ON)
if(PHSHAPE_USE_MESHSIM)
	find_package(MeshSim REQUIRED)
	include_directories(${MESHSIM_INCLUDE_DIRS})
	target_link_libraries(phshape ${MESHSIM_LIBRARY})
	add_definitions(-DSIM)
endif(PHSHAPE_USE_MESHSIM)
install(EXPORT phshape DESTINATION lib/phshape)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} DESTINATION include/phshape 
	FILES_MATCHING PATTERN "*.h")
install(TARGETS phshape EXPORT phshape RUNTIME DESTINATION bin 
	LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
