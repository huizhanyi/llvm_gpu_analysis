参考：https://jia.je/software/2023/10/17/clang-cuda-support/
增加选项-v，检查编译过程,增加选项--save-temps保存临时中间文件。
```
clang++ ./axpy.cu -o axpy  --cuda-gpu-arch=sm_60 -L /usr/local/cuda/lib64 -lcudart_static -ldl -lrt -pthread -v
```
## Device侧代码
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -E -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include/cuda_wrappers -include __clang_cuda_runtime_wrapper.h -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /usr/local/cuda-12.3/include -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -fdeprecated-macro -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -cuid=805820bbe493b9aa -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.cui -x cuda ./axpy.cu
```
生成axpy-cuda-nvptx64-nvidia-cuda-sm_60.cui文件，其中是设备侧预处理的源代码。
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -emit-llvm-bc -emit-llvm-uselists -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fdeprecated-macro -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -disable-llvm-passes -cuid=805820bbe493b9aa -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc -x cuda-cpp-output axpy-cuda-nvptx64-nvidia-cuda-sm_60.cui
```
生成文件axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc，包括设备侧bitcode文件。
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple nvptx64-nvidia-cuda -aux-triple x86_64-unknown-linux-gnu -S -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model static -mframe-pointer=all -fno-rounding-math -no-integrated-as -aux-target-cpu x86-64 -fcuda-is-device -mllvm -enable-memcpyopt-without-libcalls -fcuda-allow-variadic-functions -mlink-builtin-bitcode /usr/local/cuda-12.3/nvvm/libdevice/libdevice.10.bc -target-sdk-version=12.3 -target-cpu sm_60 -target-feature +ptx83 -debugger-tuning=gdb -fno-dwarf-directory-asm -fdebug-compilation-dir=/home/yhz/cuda -v -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fno-autolink -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcolor-diagnostics -cuid=805820bbe493b9aa -o axpy-cuda-nvptx64-nvidia-cuda-sm_60.s -x ir axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc
```
生成文件axpy-cuda-nvptx64-nvidia-cuda-sm_60.s，这里是NVPTX文件
```
"/usr/local/cuda-12.3/bin/ptxas" -m64 -O0 -v --gpu-name sm_60 --output-file axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin axpy-cuda-nvptx64-nvidia-cuda-sm_60.s
```
生成axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin,使用cuobjdump生成汇编代码
```
cuobjdump --dump-sass ./axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin
```
或者使用
```
nvdisasm ./axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin
```
从host的二进制提取cubin文件
```
cuobjdump ./axpy -xelf all
```
查看资源使用情况
```
cuobjdump ./axpy.1.sm_60.cubin -res-usage
```
关于SASS参考:

https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html CUDA Binary Utilities

https://zhuanlan.zhihu.com/p/161624982

https://zhuanlan.zhihu.com/p/163865260

https://zhuanlan.zhihu.com/p/166180054

https://github.com/NervanaSystems/maxas/wiki/Control-Codes
```
"/usr/local/cuda-12.3/bin/fatbinary" -64 --create axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin --image=profile=sm_60,file=axpy-cuda-nvptx64-nvidia-cuda-sm_60.cubin --image=profile=compute_60,file=axpy-cuda-nvptx64-nvidia-cuda-sm_60.s
```
生成axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin

## host侧代码
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple x86_64-unknown-linux-gnu -target-sdk-version=12.3 -fcuda-allow-variadic-functions -aux-triple nvptx64-nvidia-cuda -E -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yhz/cuda -v -fcoverage-compilation-dir=/home/yhz/cuda -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include/cuda_wrappers -include __clang_cuda_runtime_wrapper.h -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/x86_64-linux-gnu/c++/9 -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../include/c++/9/backward -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /home/yhz/llvm-project/build-debug/lib/clang/18/include -internal-isystem /usr/local/include -internal-isystem /usr/lib/gcc/x86_64-linux-gnu/9/../../../../x86_64-linux-gnu/include -internal-externc-isystem /usr/include/x86_64-linux-gnu -internal-externc-isystem /include -internal-externc-isystem /usr/include -internal-isystem /usr/local/cuda-12.3/include -fdeprecated-macro -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -cuid=805820bbe493b9aa -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-host-x86_64-unknown-linux-gnu.cui -x cuda ./axpy.cu
```
生成axpy-host-x86_64-unknown-linux-gnu.cui
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple x86_64-unknown-linux-gnu -target-sdk-version=12.3 -fcuda-allow-variadic-functions -aux-triple nvptx64-nvidia-cuda -emit-llvm-bc -emit-llvm-uselists -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yhz/cuda -v -fcoverage-compilation-dir=/home/yhz/cuda -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -fdeprecated-macro -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcxx-exceptions -fexceptions -fcolor-diagnostics -disable-llvm-passes -fcuda-include-gpubinary axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin -cuid=805820bbe493b9aa -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-host-x86_64-unknown-linux-gnu.bc -x cuda-cpp-output axpy-host-x86_64-unknown-linux-gnu.cui
```
生成axpy-host-x86_64-unknown-linux-gnu.bc，注意这里增加参数-fcuda-include-gpubinary axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin，把设备侧代码插入到host侧。查看bitcode文件，可以看到常量__cuda_fatbin_wrapper定义了fatbin文件数据。
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1 -triple x86_64-unknown-linux-gnu -target-sdk-version=12.3 -fcuda-allow-variadic-functions -aux-triple nvptx64-nvidia-cuda -S -dumpdir axpy- -save-temps=cwd -disable-free -clear-ast-before-backend -main-file-name axpy.cu -mrelocation-model pic -pic-level 2 -pic-is-pie -mframe-pointer=all -fmath-errno -ffp-contract=on -fno-rounding-math -mconstructor-aliases -funwind-tables=2 -target-cpu x86-64 -tune-cpu generic -debugger-tuning=gdb -fdebug-compilation-dir=/home/yhz/cuda -v -fcoverage-compilation-dir=/home/yhz/cuda -resource-dir /home/yhz/llvm-project/build-debug/lib/clang/18 -ferror-limit 19 -pthread -fgnuc-version=4.2.1 -fcolor-diagnostics -cuid=805820bbe493b9aa -faddrsig -D__GCC_HAVE_DWARF2_CFI_ASM=1 -o axpy-host-x86_64-unknown-linux-gnu.s -x ir axpy-host-x86_64-unknown-linux-gnu.bc
```
生成axpy-host-x86_64-unknown-linux-gnu.s
```
"/home/yhz/llvm-project/build-debug/bin/clang-18" -cc1as -triple x86_64-unknown-linux-gnu -filetype obj -main-file-name axpy.cu -target-cpu x86-64 -fdebug-compilation-dir=/home/yhz/cuda -dwarf-version=5 -mrelocation-model pic -mrelax-all -o axpy-host-x86_64-unknown-linux-gnu.o axpy-host-x86_64-unknown-linux-gnu.s
```
生成axpy-host-x86_64-unknown-linux-gnu.o
```
"/usr/bin/ld" -z relro --hash-style=gnu --eh-frame-hdr -m elf_x86_64 -pie -dynamic-linker /lib64/ld-linux-x86-64.so.2 -o axpy /lib/x86_64-linux-gnu/Scrt1.o /lib/x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/9/crtbeginS.o -L/usr/local/cuda/lib64 -L/usr/lib/gcc/x86_64-linux-gnu/9 -L/usr/lib/gcc/x86_64-linux-gnu/9/../../../../lib64 -L/lib/x86_64-linux-gnu -L/lib/../lib64 -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib64 -L/lib -L/usr/lib axpy-host-x86_64-unknown-linux-gnu.o -lcudart_static -ldl -lrt -lstdc++ -lm -lgcc_s -lgcc -lpthread -lc -lgcc_s -lgcc /usr/lib/gcc/x86_64-linux-gnu/9/crtendS.o /lib/x86_64-linux-gnu/crtn.o
```
生成axpy
## host代码分析
https://jia.je/software/2023/10/17/clang-cuda-support/#host-%E4%BB%A3%E7%A0%81
对kernel的调用
```
axpy<<<1, kDataLen>>>(a, device_x, device_y);
```
展开为
```
  %call4 = call i32 @__cudaPushCallConfiguration(i64 %2, i32 %4, i64 %6, i32 %8, i64 noundef 0, ptr noundef null)
  %tobool = icmp ne i32 %call4, 0
  br i1 %tobool, label %kcall.end, label %kcall.configok

kcall.configok:                                   ; preds = %entry
  %9 = load float, ptr %a, align 4
  %10 = load ptr, ptr %device_x, align 8
  %11 = load ptr, ptr %device_y, align 8
  call void @_Z19__device_stub__axpyfPfS_(float noundef %9, ptr noundef %10, ptr noundef %11) #10
  br label %kcall.end
```
这里由./clang/lib/CodeGen/CGCUDARuntime.cpp的EmitCUDAKernelCallExpr生成。
其中__cudaPushCallConfiguration为cudart_static的内部定义函数，传入grid和block参数。

_Z19__device_stub__axpyfPfS_函数原型为
```
__device_stub__axpy(float, float *, float *)
```
其中的参数为a, device_x, device_y。
__device_stub__axpy函数由lib/CodeGen/CGCUDANV.cpp内部CGNVCUDARuntime::emitDeviceStubBodyNew生成的函数
对应的bitcode函数表示形式在bitcode文件中可以看到。
```
; Function Attrs: mustprogress noinline norecurse optnone uwtable
define dso_local void @_Z19__device_stub__axpyfPfS_(float noundef %a, ptr noundef %x, ptr noundef %y) #4 {
entry:
  %a.addr = alloca float, align 4
  %x.addr = alloca ptr, align 8
  %y.addr = alloca ptr, align 8
  %grid_dim = alloca %struct.dim3, align 8
  %block_dim = alloca %struct.dim3, align 8
  %shmem_size = alloca i64, align 8
  %stream = alloca ptr, align 8
  %grid_dim.coerce = alloca { i64, i32 }, align 8
  %block_dim.coerce = alloca { i64, i32 }, align 8
  store float %a, ptr %a.addr, align 4
  store ptr %x, ptr %x.addr, align 8
  store ptr %y, ptr %y.addr, align 8
  %kernel_args = alloca ptr, i64 3, align 16
  %0 = getelementptr ptr, ptr %kernel_args, i32 0
  store ptr %a.addr, ptr %0, align 8
  %1 = getelementptr ptr, ptr %kernel_args, i32 1
  store ptr %x.addr, ptr %1, align 8
  %2 = getelementptr ptr, ptr %kernel_args, i32 2
  store ptr %y.addr, ptr %2, align 8
  %3 = call i32 @__cudaPopCallConfiguration(ptr %grid_dim, ptr %block_dim, ptr %shmem_size, ptr %stream)
  %4 = load i64, ptr %shmem_size, align 8
  %5 = load ptr, ptr %stream, align 8
  call void @llvm.memcpy.p0.p0.i64(ptr align 8 %grid_dim.coerce, ptr align 8 %grid_dim, i64 12, i1 false)
  %6 = getelementptr inbounds { i64, i32 }, ptr %grid_dim.coerce, i32 0, i32 0
  %7 = load i64, ptr %6, align 8
  %8 = getelementptr inbounds { i64, i32 }, ptr %grid_dim.coerce, i32 0, i32 1
  %9 = load i32, ptr %8, align 8
  call void @llvm.memcpy.p0.p0.i64(ptr align 8 %block_dim.coerce, ptr align 8 %block_dim, i64 12, i1 false)
  %10 = getelementptr inbounds { i64, i32 }, ptr %block_dim.coerce, i32 0, i32 0
  %11 = load i64, ptr %10, align 8
  %12 = getelementptr inbounds { i64, i32 }, ptr %block_dim.coerce, i32 0, i32 1
  %13 = load i32, ptr %12, align 8
  %call = call noundef i32 @cudaLaunchKernel(ptr noundef @_Z19__device_stub__axpyfPfS_, i64 %7, i32 %9, i64 %11, i32 %13, ptr noundef %kernel_args, i64 noundef %4, ptr noundef %5)
  br label %setup.end

setup.end:                                        ; preds = %entry
  ret void
}
```
真正启动 CUDA Kernel 的函数 cudaLaunchKernel
```
__host__​cudaError_t cudaLaunchKernel ( const void* func, dim3 gridDim, dim3 blockDim, void** args, size_t sharedMem, cudaStream_t stream )
```
其中 const void* func 指向了 device stub function 本身，args 是 device stub 把自己的参数保存下来放到数组中的，其余的参数是经过 __cudaPushCallConfiguration 保存又用 __cudaPopCallConfiguration 取出的启动配置。
这时候你可能觉得很奇怪，为什么要传 function 本身的指针，按理说要启动 GPU 上的 CUDA Kernel，不应该给一个 CUDA Kernel 的指针吗？CUDA Runtime 怎么能通过 device stub function 的指针，判断对应的 CUDA Kernel 是哪个呢？下面来讨论这个问题。
## CUDA Runtime 注册
llvm-project/clang/lib/CodeGen/CGCUDANV.cpp
```
 685 llvm::Function *CGNVCUDARuntime::makeModuleCtorFunction() {
```
编译器自动生成contructor函数，用于运行时注册。constructor函数注册到@llvm.global_ctors变量，在程序启动时调用初始化。
```
 668 /// For CUDA:
 669 /// \code
 670 /// void __cuda_module_ctor() {
 671 ///     Handle = __cudaRegisterFatBinary(GpuBinaryBlob);
 672 ///     __cuda_register_globals(Handle);
 673 /// }
 674 /// \endcode
```
这个函数调用__cudaRegisterFatBinary，注册fatbin的地址。
__cuda_register_globals函数是自动生成的函数
```
define internal void @__cuda_register_globals(ptr %0) {
entry:
  %1 = call i32 @__cudaRegisterFunction(ptr %0, ptr @_Z19__device_stub__axpyfPfS_, ptr @0, ptr @0, i32 -1, ptr null, ptr null, ptr null, ptr null, ptr null)
  ret void
}
```
这里调用__cudaRegisterFunction登记了stub函数，建立了stub和fatbin的kernel函数之间的关系。
fatbin文件可以使用cuobjdump查看
```
cuobjdump --dump-elf axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin
cuobjdump --dump-sass axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin
```
