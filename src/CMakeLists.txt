# real32, real64 supported by HDF5 1.10 https://support.hdfgroup.org/HDF5/doc/RM/PredefDTypes.html
cmake_minimum_required (VERSION 3.2.2)
project(hdf5iface Fortran)

if(${CMAKE_Fortran_COMPILER_ID} STREQUAL Intel)
    list(APPEND FFLAGS -check all -traceback -warn -debug extended)
    set(F64 -r8)
elseif(${CMAKE_Fortran_COMPILER_ID} STREQUAL GNU) 
    if(${CMAKE_Fortran_COMPILER_VERSION} VERSION_LESS 5.4.1)
      message(ERROR " gfortran >= 5.4.1 required, you have " ${CMAKE_Fortran_COMPILER_VERSION})
    endif()
    list(APPEND FFLAGS -std=f2008ts -march=native -Wall -Wextra -Wpedantic -Werror=array-bounds -fbacktrace -fcheck=all)
    set(F64 -fdefault-real-8)
endif()

add_compile_options(-O3 -g ${FFLAGS})
#------------------------
find_package(HDF5 REQUIRED COMPONENTS Fortran Fortran_HL)

# need BOTH includes, for some older systems!
include_directories(${HDF5_INCLUDE_DIRS} ${HDF5_Fortran_INCLUDE_DIRS})


add_library(hdf5oo32 hdf5_interface.f90)
target_link_libraries(hdf5oo32 ${HDF5_Fortran_LIBRARIES} ${HDF5_Fortran_HL_LIBRARIES})
set_target_properties(hdf5oo32 PROPERTIES Fortran_MODULE_DIRECTORY mod.real32)

add_library(hdf5oo64 hdf5_interface.f90)
target_compile_options(hdf5oo64 PUBLIC ${F64})
target_link_libraries(hdf5oo64 ${HDF5_Fortran_LIBRARIES} ${HDF5_Fortran_HL_LIBRARIES})
set_target_properties(hdf5oo64 PROPERTIES Fortran_MODULE_DIRECTORY mod.real64)

# ========== test

add_executable(test32h5 test_hdf5_ifc.f90)
target_link_libraries(test32h5 hdf5oo32)
target_compile_options(test32h5 PRIVATE -Wno-compare-reals)
set_target_properties(test32h5 PROPERTIES Fortran_MODULE_DIRECTORY mod.real32)

add_executable(test64h5 test_hdf5_ifc.f90)
target_link_libraries(test64h5 hdf5oo64)
target_compile_options(test64h5 PRIVATE -Wno-compare-reals)
set_target_properties(test64h5 PROPERTIES Fortran_MODULE_DIRECTORY mod.real64)
#---------------------------
enable_testing()
add_test(NAME h5iface_32 COMMAND test32h5)

add_test(NAME h5iface_64 COMMAND test64h5)
