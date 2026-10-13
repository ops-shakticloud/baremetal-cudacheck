# Test CUDA functionality on Baremetal servers 

## 1. Check the driver version

```bash
nvidia-smi
```

```text
| NVIDIA-SMI 570.172.08    Driver Version: 570.172.08    CUDA Version: 12.8 |
```

## 2. Find the compatible CUDA Toolkit (CTK) version

According to the [CUDA Toolkit 13.4 Update 1 – Release Notes](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/), **Table 3**: for driver `570.172.08`, the CTK version can be **12.x**.

<img width="661" height="293" alt="image" src="https://github.com/user-attachments/assets/0d0e024b-1612-45f1-8125-58b493e090ae" />


Referr to **Table 4**, CTK **12.8** or **12.8 Update 1** should be compatible:

<img width="940" height="289" alt="image" src="https://github.com/user-attachments/assets/4de37655-1573-4895-9ae4-29d467fc5851" />


## 3. Download toolkit

Download page: [CUDA Toolkit 12.8 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-12-8-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local)

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.8.0/local_installers/cuda_12.8.0_570.86.10_linux.run
```

## 4. Install the toolkit

> [!NOTE]
> Run the following steps as a **non-admin user** only.

```bash
./cuda_12.8.0_570.86.10_linux.run --installpath=$PWD/12.8.0
```



Select **Continue**:

<img width="743" height="437" alt="image" src="https://github.com/user-attachments/assets/e34c2fbd-9c2d-4ff9-b703-52152d00c557" />

<img width="743" height="437" alt="image" src="https://github.com/user-attachments/assets/ecd5fbc3-5d58-41e3-88b3-f790882adfdc" />



<img width="702" height="416" alt="image" src="https://github.com/user-attachments/assets/5351a523-dc0a-4894-9e52-f496a4f689c6" />


In **Toolkit Options**:

<img width="768" height="442" alt="image" src="https://github.com/user-attachments/assets/5a1f7b89-dec1-44ac-a7da-ad4814faae9e" />


In **Library install path**:

<img width="734" height="436" alt="image" src="https://github.com/user-attachments/assets/22ba022b-3640-4177-8cd1-a426da6b85d6" />


<img width="770" height="425" alt="image" src="https://github.com/user-attachments/assets/dcb0d802-69fc-47ae-a5ef-e7445e931f8c" />


Start the installation:

<img width="725" height="457" alt="image" src="https://github.com/user-attachments/assets/8331b7b8-f77f-47c0-ab83-3f874be58b21" />


ensure you see following 

<img width="1652" height="261" alt="image" src="https://github.com/user-attachments/assets/d49688b8-2341-49ec-9180-e1deb9d72d75" />


## 5. Set up the environment

```bash
cat > env.src << EOF
export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:$PWD/12.8.0/lib:$PWD/12.8.0/lib64
export LIBRARY_PATH=\$LIBRARY_PATH:$PWD/12.8.0/lib:$PWD/12.8.0/lib64
export PATH=\$PATH:$PWD/12.8.0/bin
EOF

source env.src
nvcc --version
```

## 6. Build and run basic CUDA sample codes

### Clone and check out the matching version

```bash
git clone https://github.com/NVIDIA/cuda-samples.git
cd cuda-samples
git tag -l "v12*"
git checkout v12.8
```

```text
$ git branch
* (HEAD detached at v12.8)
  master
```

### Configure

Check the GPU compute capability:

```bash
nvidia-smi --query-gpu=name,compute_cap --format=csv
```

Then configure the build (use the compute capability from above, e.g. `9.0` → `90`):

```bash
rm -rf build && mkdir build && cd build
cmake -DCMAKE_CUDA_ARCHITECTURES=90 ..
```

Ensure the compilers are detected:

<img width="1022" height="348" alt="image" src="https://github.com/user-attachments/assets/bef8f00a-b179-4007-8084-66f8076bd8b2" />


### Build

```bash
make -j$(nproc)
```
<img width="1411" height="447" alt="image" src="https://github.com/user-attachments/assets/edbce2ff-d263-4751-832f-725a90db9236" />


## 7. Verify

### Device query

```bash
Samples/1_Utilities/deviceQueryDrv/deviceQueryDrv
```

<img width="1167" height="508" alt="image" src="https://github.com/user-attachments/assets/31ac1b66-936c-43ec-9b08-4dda63f0f808" />


### Performance samples

```bash
CUDA_VISIBLE_DEVICES=1 Samples/6_Performance/UnifiedMemoryPerf/UnifiedMemoryPerf
```

<img width="1407" height="298" alt="image" src="https://github.com/user-attachments/assets/6757e1f2-1162-4cca-9e31-17a6fc66eddf" />


The process shows up on GPU 1 in `nvidia-smi`:

<img width="749" height="284" alt="image" src="https://github.com/user-attachments/assets/e5872e94-5097-40f8-9737-bc2ec40b9f8d" />

