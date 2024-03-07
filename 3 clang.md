先简单分析一下clang侧代码
## clang/lib/CodeGen/CGCUDANV.cpp
```
class CGNVCUDARuntime : public CGCUDARuntime {
```
生成很多运行时需要的函数。可以通过分析bitcode文件检查这些功能。
这些运行时函数运行在host侧，检查axpy-host-x86_64-unknown-linux-gnu.ll文件
```
@1 = private constant [4136 x i8] c"P\EDU\BA\ ...... section ".nv_fatbin", align 8
```
对应@1中存放了fatbin数据，对应数据放到.nv_fatbin段，可以通过
```
hexdump -C axpy.cu-cuda-nvptx64-nvidia-cuda.fatbin
```
对比两边的内容。
```
@__cuda_fatbin_wrapper = internal constant { i32, i32, ptr, ptr } { i32 1180844977, i32 1, ptr @1, ptr null }, section ".nvFatBinSegment", align 8
```
另外一个常量@__cuda_fatbin_wrapper定义在.nvFatBinSegment段，类型为constant { i32, i32, ptr, ptr }，其中第三个参数指向前面的@1
对应指向fatbin的数据地址？
```
@llvm.global_ctors = appending global [2 x { i32, ptr, ptr }] [{ i32, ptr, ptr } { i32 65535, ptr @_GLOBAL__sub_I_axpy.cu, ptr null }, { i32, ptr, ptr } { i32 65535, ptr @__cuda_module_ctor, ptr null }]
```
@llvm.global_ctors是系统定义的全局变量，包括了系统定义的constructor函数列表。这里定义了2个函数
@_GLOBAL__sub_I_axpy.cu和@__cuda_module_ctor




