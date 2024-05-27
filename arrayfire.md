# Compiling PLUMED with ARRAYFIRE support
The purpose of this tutorial is to guide the user step by step through the process of compiling PLUMED with ARRAYFIRE, an open-source library that supports a wide range of hardware accelerators for parallel computing.

In this specific case, the library is used to take advantage of CUDA-based GPUs to speed up the SAS calculation (ATOMISTIC / MARTINI / PARAMETERS / ONEBEAD representations). Furthermore, in order to show a limit case, the installation will be performed on Leonardo, an HPC hosted by CINECA and managed by the SLURM scheduler.

In this specific scenario, the library is used to take advantage of CUDA-based GPUs to speed up the SAS calculation (ATOMISTIC / MARTINI / PARAMETERS / ONEBEAD representations). Furthermore, in order to show a real-life limit case, the installation will be performed on Leonardo, an HPC hosted by CINECA and managed by the SLURM scheduler.

To keep the working environment clean and compatible with other setups, the modules are not automatically loaded via .bashrc/.bash_aliases, but are invoked during installation and then managed via the SBATCH script used to submit jobs on the HPC.

## 1. Loading essential modules

To install ArrayFire, certain dependencies are required. Often, these dependencies are already available on HPC systems and can be loaded through modules. It is important to note that dependencies may also have their own dependencies, such as compilers or essential libraries. In general, to view the available modules on a SLURM-based HPC, you can use the command:
``` 
module avail
```
For a comprehensive list of requirements and additional installation suggestions, please visit the [official ARRAYFIRE GitHub page](https://github.com/arrayfire/arrayfire/wiki/Build-Instructions-for-Linux). Continuing with the Leonardo example, here is the list of basic modules that can be inserted directly into the shell:
``` 
module load profile/lifesc
module load gmp/6.2.1
module load mpfr/4.1.0
module load mpc/1.2.1
module load gcc/11.3.0
module load zlib/1.2.13--gcc--11.3.0
module load openmpi/4.1.4--gcc--11.3.0-cuda-11.8
module load openblas/0.3.21--gcc--11.3.0
module load cblas/2015-06-06--gcc--11.3.0
module load gsl/2.7.1--gcc--11.3.0
module load cuda/11.8
```
Additional dependencies are available on Leonardo, but for general purposes will be installed manually in the next steps. It is assumed that downloads are stored in the home folder.

## 2. Building dependencies from source
Let's start by creating a folder where all the codes will be installed:
``` 
mkdir $HOME/build
``` 
### FFTW
Download the "Fastest Fourier Transform in the West" package version 3.3.10:
``` 
wget https://www.fftw.org/fftw-3.3.10.tar.gz
``` 

Extract the archive:
``` 
tar -xzvf fftw-3.3.10.tar.gz
``` 
Copy the extracted folder to fftw-3.3.10_f and move into it:
```
cp -r fftw-3.3.10 fftw-3.3.10_f && cd fftw-3.3.10_f
```
The FFTW libraries must be built separately with single and double precision support. The next command configures the installation with 32-bit floating point support:
```
./configure --prefix=$HOME/build/fftw/ --enable-float --enable-shared CFLAGS="-march=native" --enable-avx2 --enable-sse2
```
Configuration details:
```
--prefix: Specifies the installation directory.
--enable-float: Enables single precision.
--enable-shared: Enables the compilation of shared libraries.
--enable-avx2 and --enable-sse2: Enable the use of AVX2 and SSE2 instructions respectively, which can improve the performance of operations on CPUs that support these instructions.
CFLAGS="-march=native": Sets compiler options to optimize the code for the specific architecture of the machine on which it is being compiled.
```
Compile and install the source code:
```
make && make install
```
Move to the fftw-3.3.10 folder and configure new instructions with the 64-bit floating point support:
```
./configure --prefix=$HOME/build/fftw/ --enable-shared CFLAGS="-march=native" --enable-avx2 --enable-sse2
```
Compile and install the source code:
```
make && make install
```
Set the LD_LIBRARY_PATH environment variable directly in the shell:
```
export LD_LIBRARY_PATH="$HOME/build/fftw/lib/:$LD_LIBRARY_PATH"
```
### BOOST
Download the "Boost C++" libraries version 1.85.0:
``` 
wget https://boostorg.jfrog.io/artifactory/main/release/1.85.0/source/boost_1_85_0.tar.gz
```
Extract the archive:
```
tar -xzvf boost_1_85_0.tar.gz
```
Move into the boost folder:
```
cd boost_1_85_0
```
Initialise the Boost build system:
```
./bootstrap.sh
```
Install Boost libraries specifying the directory where Boost will be installed:
```
./b2 install --prefix=$HOME/build/boost
```
Add the Boost library path to the LD_LIBRARY_PATH and the include path to the PATH:
```
export PATH="$HOME/boost/include/:$PATH"
export LD_LIBRARY_PATH="$HOME/build/boost/lib/:$LD_LIBRARY_PATH"
```
### SPDLOG
