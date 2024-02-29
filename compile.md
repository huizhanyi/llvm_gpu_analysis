1、安装Nvidia GPU toolkit
我WSL上没有GPU，但是编译需要链接CUDA的库和头文件，我安装了toolkit，但是没有安装驱动程序，安装程序为
cuda_12.3.2_545.23.08_linux.run
2、编译llvm，支持CUDA
```
cmake -S llvm -B build-debug -G Ninja  -DLLVM_ENABLE_RTTI=ON -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_TARGETS_TO_BUILD="host;NVPTX" -DCMAKE_INSTALL_PREFIX=/home/yhz/gpu -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_ASSERTIONS=ON -DLLVM_PARALLEL_LINK_JOBS=1 -DLLVM_ENABLE_TERMINFO=OFF -DLLVM_PARALLEL_COMPILE_JOBS=4
```

