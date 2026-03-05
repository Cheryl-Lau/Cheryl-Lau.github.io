
[< BACK](codes.md)

## Instructions on how to couple Phantom to CMacIonize

Disclaimer: This method was found with Bert's help and has been tested on three machines (one PC and two HPCs), but there is no guarantee. 

The following instructions assume you use gfortran on a linux machine. 


#### Step 0: Download Phantom and CMacIonize 

Create a fork repo of Phantom and CMacIonize from 
`https://github.com/danieljprice/phantom` and `https://github.com/bwvdnbro/CMacIonize`.

Clone the fork into your computer by running the following commands: 
```
git clone https://github.com/<your git username>/phantom.git
git clone https://github.com/<your git username>/CMacIonize.git
```

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

Put in the following lines, if it is not already there: 
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
Don't forget to `source .bashrc`. 

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


#### Step 4: Deal with the Phantom Makefile 

Open the Makefile in `phantom/build`.

##### Step 4.1: Add in new setup 

Create a new setup that looks like this: 
```
ifeq ($(SETUP), cmi)
#	Coupling Phantom to CMacIonize for adding ionizing radiation
    SETUPFILE=velfield_fromcubes.f90 setup_sphere_evolvedmc.f90
    FPPFLAGS= -DPHOTOION
    SRCPHOTOION=utils_cmi.f90 kdtree_cmi.f90 hnode_cmi.f90 heating_cooling_cmi.f90 photoionize_cmi.F90
    CMACIONIZE=yes
endif
```

##### Step 4.2: Add in LD flags 

Underneath the `ifeq ($(MCFOST), yes)` block, add the following. Copy the ld flags listed in `cmi_fortran_libs.txt` to here: 
``` 
ifeq ($(CMACIONIZE), yes)
    LDFLAGS+= <whatever is written in cmi_fortran_libs.txt>
endif
``` 
For example, it could look like
``` 
ifeq ($(CMACIONIZE), yes)
    LDFLAGS+= -L/$(CMI_DIR)/build/lib -lCMIFortranLibrary -lCMILibrary -lLegacyEngine -lSharedEngine $(HDF5ROOT)/lib/libhdf5.so /usr/lib64/libz.so /usr/lib64/libdl.a /usr/lib64/libm.so /software/MPI/openmpi-5.0.5/lib/libmpi.so -lstdc++ -lc
endif
```
Important note! The specific compilation flags are machine-dependent. If you're running the code on different machines, you will have to create an extra block for each machine you use. For example, I have the following for St Andrews' HPC cluster Hypatia: 
```
ifeq ($(CMACIONIZE_HYPATIA), yes)
    LDFLAGS+= -L/$(CMI_DIR)/build/lib -lCMIFortranLibrary -lCMILibrary -lLegacyEngine -lSharedEngine $(HDF5ROOT)/lib/libhdf5.so /usr/lib64/libz.so /usr/lib64/libdl.a /usr/lib64/libm.so /software/MPI/openmpi-5.0.5/lib/libmpi.so -lstdc++ -lc
endif
```
and in the setup block, I'd now put `CMACIONIZE_HYPATIA=yes` instead of `CMACIONIZE=yes`. 

##### Step 4.3: Add in the source files 

Look for the line `SOURCES= physcon.f90 ${CONFIG} ${SRCKERNEL} io.F90`... in the Makefile. 

Add `${SRCPHOTOION}` into the list of source codes. Compilation order matters! It has to be after `kdtree.F90` and `linklist_kdtree.F90`, but before `force.F90`, `deriv.F90`, `step_leapfrog.F90` and `initial.F90`.

Do the same for `SRCDUMP= `. You could put it after `${SRCPOT} ${SRCPHOTO}`. 


##### Step 4.4: Add in the includes 

Underneath the `phantom: checksystem checkparams $(OBJECTS) phantom.o` block, add in the following. Copy the flags listed in `cmi_fortran_includes.txt` to here: 
``` 
photoionize_cmi.o: photoionize_cmi.F90
	$(FC) -c $(FFLAGS) <whatever is written in cmi_fortran_includes.txt> $< -o $@
``` 
For example, 
``` 
photoionize_cmi.o: photoionize_cmi.F90
	$(FC) -c $(FFLAGS) -fopenmp -I$(CMI_DIR)/build/include $< -o $@
``` 

##### Step 4.5: Download all the photoionization-relevant codes  

Checkout the following files from my phantom repo: 
1. `photoionize_cmi.F90`
2. `kdtree_cmi.f90`
3. `hnode_cmi.f90`
4. `heating_cooling_cmi.f90`
5. `utils_cmi.f90`  


#### Step 5: Compile everything 

Create a new work directory somewhere outside of the phantom repo. `cd` to your new repo and write a local Makefile for the new setup that you created in step 4.1. 
```
~/phantom/scripts/writemake.sh cmi > Makefile
```

Finally, compile everything. 
``` 
make; make setup
``` 


Best of luck and happy trouble-shooting. 






