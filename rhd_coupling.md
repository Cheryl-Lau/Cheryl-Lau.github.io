
[< BACK](codes.md)

## Instructions on how to couple Phantom to CMacIonize

Disclaimer: This method has been tested on three machines (one local machines and two HPCs), but there is no guarantee.  

The following instructions assume you use gfortran. 


#### Step 1: Deal with the HDF5 

Check if HDF5 has already been installed. If not, download it from source by running this on the command line:
```
wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.12/hdf5-1.12.2/src/hdf5-1.12.2.tar.gz 
gzip -cd hdf5-1.12.2.tar.gz | tar xvf -
```
(or replace 1.12.2 with any other latest version of HDF5).

Then, go to the top directory of your HDF5 and run:
```
./configure --enable-fortran --enable-cxx
make
make install
```

#### Step 2: Go to your .bashrc 

Put in the following lines, if it's not already there: 
```
# Phantom
export SYSTEM=gfortran
export PHANTOM_DIR=<path to phantom dir>
export OMP_SCHEDULE="dynamic"
export OMP_STACKSIZE=512M
ulimit -s unlimited
export HDF5=no
export HDF5ROOT=<path to HDF5 dir>
export LD_LIBRARY_PATH=<path to HDF5 dir>/hdf5/lib:$LD_LIBRARY_PATH

# CMacIonize
export CMI_DIR=<path to CMacIonize dir>
export HDF5_ROOT=<path to HDF5 dir>/hdf5

# Activate compilers
module load openmpi/5.0.5
module load gcclibs/11.4.1
module load ucx/1.16.0
```

#### Step 3: Go to CMacIonize top directory

Run the following lines:
```
mkdir build
cd build
cmake -DMAKE_BUILD_TYPE=Release -DMAX_NUMBER_OF_THREADS=192 -DACTIVATE_FORTRAN_BINDING=True -DOUTPUT_HEATING=True -DCMAKE_Fortran_COMPILER=/usr/bin/gfortran <path to CMacIonize dir>
make
```
Before running `make`, check that it has found HDF5 and the fortran compiler. 

Now you should see that, inside `CMacIonize/build/compilation`, it has generated two files, named `cmi_fortran_includes.txt` and `cmi_fortran_libs.txt`. 


#### Step 4: Go to Phantom 

Open the Makefile and create a new setup that looks like this: 
```
ifeq ($(SETUP), cmi)
#	Coupling Phantom to CMacIonize for adding ionizing radiation
    SETUPFILE=velfield_fromcubes.f90 setup_sphere_evolvedmc.f90
    FPPFLAGS= -DPHOTOION
    SRCPHOTOION=utils_cmi.f90 kdtree_cmi.f90 hnode_cmi.f90 heating_cooling_cmi.f90 photoionize_cmi.F90
    CMACIONIZE=yes
endif
``` 

Then, down below, add the following. Copy the ld flags listed in cmi_fortran_libs.txt to here: 
``` 
ifeq ($(CMACIONIZE), yes)
    LDFLAGS+= <whatever written in cmi_fortran_libs.txt>
endif
``` 
For example, it could look like
``` 
ifeq ($(CMACIONIZE), yes)
    LDFLAGS+= -L/$(CMI_DIR)/build/lib -lCMIFortranLibrary -lCMILibrary -lLegacyEngine -lSharedEngine $(HDF5ROOT)/lib/libhdf5.so /usr/lib64/libz.so /usr/lib64/libdl.a /usr/lib64/libm.so /software/MPI/openmpi-5.0.5/lib/libmpi.so -lstdc++ -lc
endif
```

Somewhere further down below, add the following. Copy the f flags listed in cmi_fortran_includes.txt to here: 
``` 
photoionize_cmi.o: photoionize_cmi.F90
	$(FC) -c $(FFLAGS) <whatever written in cmi_fortran_includes.txt> $< -o $@
``` 
For example, 
``` 
photoionize_cmi.o: photoionize_cmi.F90
	$(FC) -c $(FFLAGS) -fopenmp -I$(CMI_DIR)/build/include $< -o $@
``` 


#### Step 5: Finally, go to your run directory 

Compile everything.
``` 
make; make setup
``` 


</br>
Best of luck and happy trouble-shooting. 






