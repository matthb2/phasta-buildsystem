find_package(MeshSim)
find_package(Threads)

file(GLOB SRC_FILES *.cc)

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${MESHSIM_INCLUDE_DIRS})

link_directories(${MESHSIM_LIBRARY_DIR})
add_executable(phNSpre-serial ${SRC_FILES})
add_definitions(-DSIM)
add_definitions(-DMODELER_PARASOLID)
target_link_libraries(phNSpre-serial SimAdvMeshing SimMeshing SimParasolid220 SimMeshTools SimModel)
target_link_libraries(phNSpre-serial ${PARASOLID_LIBRARY})
target_link_libraries(phNSpre-serial ${CMAKE_THREAD_LIBS_INIT})
target_link_libraries(phNSpre-serial phastaIO)
