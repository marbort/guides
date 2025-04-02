###Install DeePMD Python

module reset
module load FFTW.MPI/3.3.10-gompi-2022a
module load FFTW/3.3.10-GCC-13.3.0 
module load Python/3.12.3-GCCcore-13.3.0
module load CUDA/12.6.0
module load CMake/3.29.3-GCCcore-13.3.0
module load GCC/13.3.0


START_DIR=$PWD
export LAMMPS_ROOT="/projects/ec12/marcobo/lammps/lammps-29Aug2024" #change with the path of your LAMMPS source
export deepmd_root="${START_DIR}/deepmd-kit-install" #change with the path of  where you want deepmd plugins to be installed

python3 -m venv deepmd-env
source deepmd-env/bin/activate
pip install tensorflow[and-cuda]
pip install numpy
pip install setuptools
pip install mpi4py


export HOROVOD_WITHOUT_MXNET=1
export HOROVOD_WITHOUT_PYTORCH=1
export HOROVOD_GPU=CUDA
export HOROVOD_WITHOUT_GLOO=1
export HOROVOD_WITH_TENSORFLOW=1

git clone --recursive https://github.com/horovod/horovod.git
cd horovod/
pip install --verbose .
cd ..


git clone https://github.com/deepmodeling/deepmd-kit.git 
cd deepmd-kit
export DP_VARIANT=cuda
pip install . --verbose

cd build
mkdir plugins
cd plugins

cmake -DCMAKE_INSTALL_PREFIX=$deepmd_root -DUSE_TF_PYTHON_LIBS=TRUE -DLAMMPS_SOURCE_ROOT=$LAMMPS_ROOT  -DCMAKE_BUILD_TYPE=Release -DDP_VARIANT=cude ../../source

make -j 8 
make install
cd $START_DIR
PATH=$deepmd_root/bin:$PATH
PATH=$deepmd_root/lib:$PATH

mkdir lammps-install
LAMMPS_INSTALL=$(realpath  lammps-install)
cd  lammps/lammps-29Aug2024
mkdir build
cd build

cmake  -D BUILD_MPI=yes -D PKG_PLUMED=yes -D PLUMED_MODE=runtime -D DOWNLOAD_PLUMED=NO  -D PKG_OPENMP=yes   -D PKG_PLUGIN=ON -D PKG_KSPACE=ON -D PKG_RIGID=ON -D PKG_EXTRA-FIX=ON -D PKG_EXTRA-DUMP=ON -D LAMMPS_INSTALL_RPATH=ON -DBUILD_SHARED_LIBS=yes -D CMAKE_INSTALL_PREFIX=$LAMMPS_INSTALL  -D PKG_GPU=ON  -D GPU_API=CUDA  -D GPU_ARCH=sm_86 ../cmake

make -j 8
make install
cd $START_DIR
