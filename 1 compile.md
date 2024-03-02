1、安装Nvidia GPU toolkit
我WSL上没有GPU，但是编译需要链接CUDA的库和头文件，我安装了toolkit，但是没有安装驱动程序，安装程序为
cuda_12.3.2_545.23.08_linux.run
windows上有gforce显卡，居然nvidia-smi.exe程序可以跑。
2、编译llvm，支持CUDA
```
git clone --depth 1 https://github.com/llvm/llvm-project.git
cmake -S llvm -B build-debug -G Ninja  -DLLVM_ENABLE_RTTI=ON -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_TARGETS_TO_BUILD="host;NVPTX" -DCMAKE_INSTALL_PREFIX=/home/yhz/gpu -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_ASSERTIONS=ON -DLLVM_PARALLEL_LINK_JOBS=1 -DLLVM_ENABLE_TERMINFO=OFF -DLLVM_PARALLEL_COMPILE_JOBS=4
ninja -C build-debug
```
3、测试编译
axpy.cu
```
#include <iostream>

__global__ void axpy(float a, float* x, float* y) {
  y[threadIdx.x] = a * x[threadIdx.x];
}

int main(int argc, char* argv[]) {
  const int kDataLen = 4;

  float a = 2.0f;
  float host_x[kDataLen] = {1.0f, 2.0f, 3.0f, 4.0f};
  float host_y[kDataLen] = {0.0f};

  // Copy input data to device.
  float* device_x;
  float* device_y;
  cudaMalloc(&device_x, kDataLen * sizeof(float));
  cudaMalloc(&device_y, kDataLen * sizeof(float));
  cudaMemcpy(device_x, host_x, kDataLen * sizeof(float),
             cudaMemcpyHostToDevice);

  // Launch the kernel.
  axpy<<<1, kDataLen>>>(a, device_x, device_y);

  // Copy output data to host.
  cudaDeviceSynchronize();
  cudaMemcpy(host_y, device_y, kDataLen * sizeof(float),
             cudaMemcpyDeviceToHost);

  // Print the results.
  for (int i = 0; i < kDataLen; ++i) {
    std::cout << "y[" << i << "] = " << host_y[i] << "\n";
  }

  cudaDeviceReset();
  return 0;
}
```
编译命令
```
~/llvm-project/build-debug/bin/clang++ ./axpy.cu -o axpy  --cuda-gpu-arch=sm_60 -L /usr/local/cuda/lib64 -lcudart_static -ldl -lrt -pthread
```
我的WSL可以执行axpy程序，但是结果不对
```
yhz@DESKTOP-A2C7G7A:~/cuda$ ./axpy
y[0] = 0
y[1] = 0
y[2] = 0
y[3] = 0
```
检查二进制程序axpy
```
readelf -S ./axpy
```
```
There are 33 section headers, starting at offset 0xf2150:
Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align

  [16] .nv_fatbin        PROGBITS         0000000000093740  00093740
       0000000000001028  0000000000000000   A       0     0     8
```
4、dump bitcode
```
clang++ ./axpy.cu -emit-llvm -c  --cuda-gpu-arch=sm_60
```
输出2个文件
axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc  axpy.bc
第一个是nvptx后端文件，第二个是x86平台文件。


