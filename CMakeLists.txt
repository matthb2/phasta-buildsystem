CMAKE_MINIMUM_REQUIRED(VERSION 2.8.5)
#SET(CMAKE_FORTRAN_COMPILER gfortran gfortran)
PROJECT(PHASTA Fortran C CXX )
enable_language(Fortran)
enable_language(C)
enable_language(CXX)

list(APPEND CMAKE_MODULE_PATH ${PHASTA_SOURCE_DIR}/CMakeFiles)

#Handle Fortran name mangling
include(FortranCInterface)
FortranCInterface_VERIFY(CXX)
FortranCInterface_Header(FCMangle.h SYMBOLS write_restart write_error write_displ write_field write_hessian)
include_directories("${PHASTA_BINARY_DIR}")

#MESSAGE(ERROR ${CMAKE_Fortran_SOURCE_FILE_EXTENSIONS})
#SET(CMAKE_Fortran_SOURCE_FILE_EXTENSIONS f90 f F90 F)
# stuff for C/C++ calls to Fortran -- still needs to be done properly...
IF(CMAKE_Fortran_COMPILER_ID MATCHES GNU)
  SET(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -ffixed-form -ffixed-line-length-132 -x f95-cpp-input" )
ENDIF(CMAKE_Fortran_COMPILER_ID MATCHES GNU)

IF(CMAKE_Fortran_COMPILER_ID MATCHES Intel)

SET(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -W0 -xW -ip -cm -w95 -80 -fpp" )
message(STATUS "Found ifort")
message(STATUS ${CMAKE_Fortran_FLAGS})
ENDIF(CMAKE_Fortran_COMPILER_ID MATCHES Intel)

if(CMAKE_Fortran_COMPILER_ID MATCHES PGI)
	set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -Mextend")
endif(CMAKE_Fortran_COMPILER_ID MATCHES PGI)

if(CMAKE_Fortran_COMPILER_ID MATCHES XL)
	set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -qstrict -qnullterm -qfixed=132")
endif(CMAKE_Fortran_COMPILER_ID MATCHES XL)

IF(${CMAKE_BUILD_TYPE} MATCHES "Debug" )
  ADD_DEFINITIONS( -DDEBUG )
ENDIF(${CMAKE_BUILD_TYPE} MATCHES "Debug" )

# set archos "like" options for things like c calling fortran bindings
# later on we may want to do it dependent on behavior (e.g. 
# C_FORTRAN_CALL == uppercase, underline, nochange)
#OPTION(ARCHOS_OPTION "ArchOS dependent defines (e.g. -DLINUX)" )
SET(ARCHOS_OPTION "-DLINUX" CACHE STRING "ArchOS dependent defines (e.g. -DLINUX)" )
ADD_DEFINITIONS( ${ARCHOS_OPTION} )


SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY
  ${PHASTA_BINARY_DIR}/lib)

SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY
  ${PHASTA_BINARY_DIR}/bin)

OPTION(PHASTA_USE_MPI "Build MPI components" ON)
IF(PHASTA_USE_MPI)
  ADD_DEFINITIONS( -DPARALLEL -DMPI -DMPICH_SKIP_MPICXX )
  FIND_PACKAGE(MPI REQUIRED)
#  SET(CMAKE_MODULE_PATH ${PHASTA_SOURCE_DIR}/CMakeFiles)
  # bug in FIND_PACKAGE for Threads -- ignore for now (bug has been reported)...
  FIND_PACKAGE(Threads)
  IF(Threads_FOUND)
    INCLUDE(FindThreads)
  ENDIF(Threads_FOUND)
ENDIF(PHASTA_USE_MPI)

OPTION(PHASTA_BUILD_PHASTAIO "Build PhastaIO" ON)
if(PHASTA_BUILD_PHASTAIO)
	add_subdirectory(phastaIO)
	list(APPEND PHASTA_INCLUDES "${PHASTA_SOURCE_DIR}/phastaIO")
endif(PHASTA_BUILD_PHASTAIO)

OPTION(PHASTA_BUILD_METIS "Build phMetis" ON)
if(PHASTA_BUILD_METIS)
	add_subdirectory(phMetis)
	add_dependencies(phMetis common) #only for XLF?
endif(PHASTA_BUILD_METIS)

OPTION(PHASTA_BUILD_SHAPEFUNCTION "Build shapeFunction" ON)
if(PHASTA_BUILD_SHAPEFUNCTION)
	list(APPEND PHASTA_INCLUDES "${PHASTA_SOURCE_DIR}/shapeFunction/src")
	add_subdirectory(shapeFunction)
endif(PHASTA_BUILD_SHAPEFUNCTION)

if(EXISTS ${PHASTA_SOURCE_DIR}/phShape)
OPTION(PHASTA_BUILD_PHSHAPE "Build phShape" OFF)
if(PHASTA_BUILD_PHSHAPE)
	list(APPEND PHASTA_INCLUDES "${PHASTA_SOURCE_DIR}/phShape/src")
	add_subdirectory(phShape)
endif(PHASTA_BUILD_PHSHAPE)
endif(EXISTS ${PHASTA_SOURCE_DIR}/phShape)

OPTION(PHASTA_BUILD_SOLVERS "Build Solvers" ON)
if(PHASTA_BUILD_SOLVERS)
	set(PHASTA_BUILD_PHASTAIO ON)
	if(PHASTA_USE_MPI)
		set(PHASTA_BUILD_METIS ON)
	endif(PHASTA_USE_MPI)
	list(APPEND PHASTA_INCLUDES "${PHASTA_SOURCE_DIR}/phSolver/common")
	include_directories(${PHASTA_INCLUDES})
	add_subdirectory(phSolver)
#TODO
endif(PHASTA_BUILD_SOLVERS)

OPTION(PHASTA_BUILD_PHPOST "Build Post Processing Tools" ON)
if(PHASTA_BUILD_PHPOST)
	add_subdirectory(phPost)
endif(PHASTA_BUILD_PHPOST)


if(EXISTS ${PHASTA_SOURCE_DIR}/phNSpre)
	add_subdirectory(phNSpre)
endif(EXISTS ${PHASTA_SOURCE_DIR}/phNSpre)
