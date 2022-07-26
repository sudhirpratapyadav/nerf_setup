- Link
	- https://github.com/NVlabs/instant-ngp

## SETUP
### DGX Setup

``

nvidia-docker run -itd --name sudhir_docker -v /raid/home/sudhir:/workspace/sudhir nvcr.io/nvidia/pytorch:21.06-py3
docker exec -it sudhir_docker /bin/bash

apt-get update
apt install software-properties-common
```


### Nerf setup
#### Requirements
1. Install below packages
```
apt-get install build-essential git python3-dev python3-pip libopenexr-dev libxi-dev libglfw3-dev libglew-dev libomp-dev libxinerama-dev libxcursor-dev

pip install numpy scipy pillow matplotlib commentjson

apt-get install ffmpeg
```

2. Install CUDA (if not installed)
	- to check cuda version use `nvidia-smi`
	- Add this to `.bashrc`
```
export PATH="/usr/local/cuda-11.3/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-11.3/lib64:$LD_LIBRARY_PATH"
```

3. Install OptiX
	- Download NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64.sh file (or something .sh file)
		- Download Link: https://developer.nvidia.com/designworks/optix/download
		- Note: You to sign-in in nvidia account (make it using any email)
	- copy this file to /usr/local`
	- install using
		- `./NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64.sh`
		- Note: make it executable by `chmod 777 -R NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64.sh`
	- Add following to `.bashrc`
		- `export OptiX_INSTALL_DIR=/usr/local/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64`

#### Compilation
```
git clone --recursive https://github.com/nvlabs/instant-ngp
cd instant-ngp

cmake . -B build -DNGP_BUILD_WITH_GUI=off
cmake --build build --config RelWithDebInfo -j 250

```
- use `nproc` command to find out number of processors

## Usage
- setting GPU
	- export CUDA_VISIBLE_DEVICES=1
- Testing
```
mkdir nerf_results

python3 instant-ngp/scripts/run.py --scene instant-ngp/data/nerf/fox --mode nerf --screenshot_dir nerf_results --width 640 --height 480 --save_snapshot nerf_results/snapshot_1 --save_mesh nerf_results/mesh_fox_1.obj --marching_cubes_res 100 --n_steps 1000 --video_camera_path instant-ngp/data/nerf/fox/base_cam.json --video_output nerf_results/video.mp4


docker exec -it sudhir_docker python3 /workspace/sudhir/nerf/instant-ngp/scripts/run.py --scene /workspace/sudhir/nerf/instant-ngp/data/nerf/fox --mode nerf --screenshot_dir /workspace/sudhir/nerf/nerf_results --width 640 --height 480 --save_snapshot snapshot_1 --save_mesh mesh_fox_1.obj --marching_cubes_res 100 --n_steps 1000


```

# Commands
```
chown -R 1004 sudhir/

scp ~/Downloads/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64.sh sudhir@172.25.0.216:~/nerf/

scp ~/Downloads/iitj_data sudhir@172.25.0.216:~/nerf/

scp -r sudhir@172.25.0.216:~/nerf/nerf_results ~/Downloads

scp -r sudhir@172.25.0.216:~/nerf/nerf_results/mesh_fox_1.obj ~/Downloads

scp -r sudhir@172.25.0.216:~/nerf/nerf_results/video.mp4 ~/Downloads


find /opt -iname libmkl_intel_lp64.so


```

# Data preparation
- Link
	- https://github.com/NVlabs/instant-ngp/blob/master/docs/nerf_dataset_tips.md

## COLMAP installtion
- link
	- https://colmap.github.io/install.html#installation

### Steps
```
apt-get install git cmake build-essential libboost-program-options-dev libboost-filesystem-dev libboost-graph-dev libboost-system-dev libboost-test-dev libeigen3-dev libsuitesparse-dev libfreeimage-dev libmetis-dev libgoogle-glog-dev libgflags-dev libglew-dev qtbase5-dev libqt5opengl5-dev libcgal-dev


apt-get install libatlas-base-dev libsuitesparse-dev

git clone https://ceres-solver.googlesource.com/ceres-solver
cd ceres-solver
git checkout $(git describe --tags) # Checkout the latest release
mkdir build
cd build
cmake .. -DBUILD_TESTING=OFF -DBUILD_EXAMPLES=OFF
make -j 250
make install
```

```
/workspace/sudhir/nerf/ceres-solver/build/build_20220715-1_amd64.deb

 You can remove it from your system anytime using: 

      dpkg -r build

```

```
git clone https://github.com/colmap/colmap
cd colmap
git checkout dev
mkdir build
cd build
cmake ..
make -j 250
make install
```

- /opt/conda/lib/
	- add this to LD_LIBRARY_PATH
- Set GPU
- export CUDA_VISIBLE_DEVICES=7
```
export PATH="/usr/local/cuda-11.3/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-11.3/lib64:/opt/conda/lib:$LD_LIBRARY_PATH"
export OptiX_INSTALL_DIR="/usr/local/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64"
export CUDA_VISIBLE_DEVICES=7

cd sudhir/nerf/iitj_data/
python /workspace/sudhir/nerf/instant-ngp/scripts/colmap2nerf.py --colmap_matcher exhaustive --run_colmap --aabb_scale 16


./workspace/sudhir/launch_files/col2nerf.sh
```

## Converting images to dataset
- execute below from data folder
	- note **data** folder should contain sub folder named **images** with images inside it
```
python /workspace/sudhir/nerf/instant-ngp/scripts/colmap2nerf.py --colmap_matcher exhaustive --run_colmap --aabb_scale 16


docker exec -it sudhir_docker python3 /workspace/sudhir/nerf/instant-ngp/scripts/colmap2nerf.py --colmap_matcher exhaustive --run_colmap --aabb_scale 16 --images /workspace/sudhir/nerf/iitj_data/images

```

## Issues
- The most common issue with datasets is an **incorrect scale** or **offset in the camera positions**; more details below.
- The next most common issue is **too few images**, or **images with inaccurate camera parameters** (for example, if COLMAP fails). In that case, you may need to acquire more images, or tune the process by which the camera positions are calculated. This is outside the scope of the **instant-ngp** implementation. 
