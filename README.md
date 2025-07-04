# VASP package installation on macOS (Memorandum)
	Date: 2024.03.21
	Mac model: Apple M1 (2020)
	OS X version: Sonoma 14.7.1（23H222）
	

### 1. Basic instruction from offical website
https://www.vasp.at/wiki/index.php/Personal_computer_installation#Building_VASP_6.5.1_on_Mac_OS_X_(Apple_Silicon_M1/2/3/4)
- homebrew: https://brew.sh/
After install homebrew, install following packages via brew.
```
brew install gfortran gcc fftw hdf5 openmpi openblas scalapack qd
```
### 2. makefile.include
	# Default precompiler options
	CPP_OPTIONS = -DHOST=\"LinuxGNU\" \
              	-DMPI -DMPI_BLOCK=8000 -Duse_collective \
              	-DscaLAPACK \
              	-DCACHE_SIZE=4000 \
              	-Davoidalloc \
              	-Dvasp6 \
              	-Dtbdyn \
              	-Dfock_dblbuf

	CPP         = gcc-14 -E -C -w $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

	FC          = mpif90
	FCL         = mpif90

	FREE        = -ffree-form -ffree-line-length-none

	FFLAGS      = -w -ffpe-summary=none

	OFLAG       = -O2
	OFLAG_IN    = $(OFLAG)
	DEBUG       = -O0

	# For what used to be vasp.5.lib
	CPP_LIB     = $(CPP)
	FC_LIB      = $(FC)
	CC_LIB      = gcc-14
	CFLAGS_LIB  = -O
	FFLAGS_LIB  = -O1
	FREE_LIB    = $(FREE)

	OBJECTS_LIB = linpack_double.o

	# For the parser library
	CXX_PARS    = g++ -O2 -fno-common
	#LLIBS       = -lstdc++
	LLIBS       = -Lparser -lc++

	##
	## Customize as of this point! Of course you may change the preceding
	## part of this file as well if you like, but it should rarely be
	## necessary ...
	##

	# When compiling on the target machine itself, change this to the
	# relevant target when cross-compiling for another architecture
	VASP_TARGET_CPU ?= -march=native
	FFLAGS     += $(VASP_TARGET_CPU)

	# For gcc-10 and higher (comment out for older versions)
	FFLAGS     += -fallow-argument-mismatch

	# BLAS and LAPACK (mandatory)
	OPENBLAS_ROOT = /opt/homebrew/opt/openblas/
	BLASPACK    = -L$(OPENBLAS_ROOT)/lib -lopenblas

	# scaLAPACK (mandatory)
	SCALAPACK_ROOT = /opt/homebrew/opt/scalapack
	SCALAPACK   = -L$(SCALAPACK_ROOT)/lib -lscalapack

	LLIBS      += $(SCALAPACK) $(BLASPACK)

	# FFTW (mandatory)
	FFTW_ROOT  = /opt/homebrew/Cellar/fftw/3.3.10_2/
	LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3
	INCS       += -I$(FFTW_ROOT)/include

	# HDF5-support (optional but strongly recommended, and mandatory for some features)
	#CPP_OPTIONS+= -DVASP_HDF5
	#HDF5_ROOT  ?= /path/to/your/hdf5/installation
	#LLIBS      += -L$(HDF5_ROOT)/lib -lhdf5_fortran
	#INCS       += -I$(HDF5_ROOT)/include
	CPP_OPTIONS += -Dqd_emulate

	BREW_ROOT = /opt/homebrew/
	LLIBS       += -L$(BREW_ROOT)/lib -lqdmod
	INCS        += -I$(BREW_ROOT)/include/qd

	# For the VASP-2-Wannier90 interface (optional)
	#CPP_OPTIONS    += -DVASP2WANNIER90
	#WANNIER90_ROOT ?= /path/to/your/wannier90/installation
	#LLIBS          += -L$(WANNIER90_ROOT)/lib -lwannier

	# For machine learning library vaspml (experimental)
	#CPP_OPTIONS += -Dlibvaspml
	#CPP_OPTIONS += -DVASPML_USE_CBLAS
	#CPP_OPTIONS += -DVASPML_DEBUG_LEVEL=3
	#CXX_ML      = mpic++
	#CXXFLAGS_ML = -O3 -std=c++17 -pedantic-errors -Wall -Wextra
	#INCLUDE_ML  = -I$(OPENBLAS_ROOT)/include

NOTE: 
1. I have specified gcc-14 to use /opt/homebrew/bin/gcc-14 directly. We have two types of gcc on the mac, /usr/bin/gcc and /opt/homebrew/bin/gcc-14. I choosed gcc-14 installed from homebrew. I am still not sure this does matter.
2. I have specified OPENBLAS_ROOT, SCALAPACK_ROOT, and FFTW_ROOT, somehow this make linking libraries successful.
3. I failed complining VASP several times once I have tried intel one api. Although an official guide exist, one api likely no longer support mac os officially. I could download dmg files of oneapi 2023, and could install ifort, icc, and etc., however it does not support mpi versions.
   - https://stackoverflow.com/questions/77783400/intel-oneapi-for-mac-os-in-2024
   - https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-macos/2024-0/overview.html

### 3. Troubleshooting 
	ld: unsupported tapi file type '!tapi-tbd' in YAML file
Likely encoutered pretty often, could be related to path. In my case, I could resolve the problem by setting export PATH="/usr/bin:$PATH" on zsh. (Or type export PATH="/usr/bin:$PATH" directly). 
See links below. 
- https://lattician.exblog.jp/32917378/
- https://stackoverflow.com/questions/74318052/cant-compile-fortran-on-macos-monterey-ld-unsupported-tapi-file-type-tapi-t/75736450
- https://stackoverflow.com/questions/63592445/ld-unsupported-tapi-file-type-tapi-tbd-in-yaml-file

### 4. PATH information (my exmaple, could be more simplified)
	echo $PATH
	/opt/homebrew/bin:/usr/bin:/opt/homebrew/bin:/Users/username/opt/anaconda3/bin:/Users/username/opt/anaconda3/condabin:/usr/bin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin:/opt/X11/bin:/Library/Apple/usr/bin:/Library/TeX/texbin:/opt/homebrew/opt/llvm/bin:/usr/local/bin:/opt/homebrew/bin:/Users/username/bin:/opt/homebrew/opt/llvm/bin:/usr/local/bin:/opt/homebrew/bin/:/Users/username/bin

### 5. Execution 
	mpirun -np 8 vasp_std (8 cores, for example; M1 has 8 cpu cores)
