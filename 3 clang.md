## host侧bitcode文件生成
clang/lib/CodeGen/CGCUDANV.cpp
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
对应指向fatbin的数据地址。这里变量由clang代码CGCUDANV.cpp生成：
```
 795   // Create initialized wrapper structure that points to the loaded GPU binary
 796   ConstantInitBuilder Builder(CGM);
 797   auto Values = Builder.beginStruct(FatbinWrapperTy);
 798   // Fatbin wrapper magic.
 799   Values.addInt(IntTy, FatMagic);
 800   // Fatbin version.
 801   Values.addInt(IntTy, 1);
 802   // Data.
 803   Values.add(FatBinStr);
 804   // Unused in fatbin v1.
 805   Values.add(llvm::ConstantPointerNull::get(PtrTy));
 806   llvm::GlobalVariable *FatbinWrapper = Values.finishAndCreateGlobal(
 807       addUnderscoredPrefixToName("_fatbin_wrapper"), CGM.getPointerAlign(),
 808       /*constant*/ true);
 809   FatbinWrapper->setSection(FatbinSectionName);
```
```
@llvm.global_ctors = appending global [2 x { i32, ptr, ptr }] [{ i32, ptr, ptr } { i32 65535, ptr @_GLOBAL__sub_I_axpy.cu, ptr null }, { i32, ptr, ptr } { i32 65535, ptr @__cuda_module_ctor, ptr null }]
```
@llvm.global_ctors是系统定义的全局变量，包括了系统定义的constructor函数列表。这里定义了2个函数
@_GLOBAL__sub_I_axpy.cu和@__cuda_module_ctor
@_GLOBAL__sub_I_axpy.cu函数是C++启动相关的初始化函数，不分析。
@__cuda_module_ctor函数定义如下：
```
define internal void @__cuda_module_ctor() {
entry:
__cudaRegisterFatBinary定义在libcudart_static.a，参数@__cuda_fatbin_wrapper
  %0 = call ptr @__cudaRegisterFatBinary(ptr @__cuda_fatbin_wrapper)
返回指针保存到全局变量@__cuda_gpubin_handle
  store ptr %0, ptr @__cuda_gpubin_handle, align 8
登记全局变量和函数
  call void @__cuda_register_globals(ptr %0)
定义在libcudart_static.a
  call void @__cudaRegisterFatBinaryEnd(ptr %0)
登记程序退出的dtor函数
  %1 = call i32 @atexit(ptr @__cuda_module_dtor)
  ret void
}
```
```
define internal void @__cuda_register_globals(ptr %0) {
entry:
定义在libcudart_static.a，按之前的分析，登记核函数和stub函数的对应关系。
  %1 = call i32 @__cudaRegisterFunction(ptr %0, ptr @_Z19__device_stub__axpyfPfS_, ptr @0, ptr @0, i32 -1, ptr null, ptr null, ptr null, ptr null, ptr null)
  ret void
}
```
每个__global__函数定义了一个stub函数,host侧直接调用这个stub函数实现对device侧的调用。
```
define dso_local void @_Z19__device_stub__axpyfPfS_(float noundef %a, ptr noundef %x, ptr noundef %y) #4 {
使用@__cudaPopCallConfiguration函数得到保存的运行配置参数，这些配置参数是在stub调用前@__cudaPushCallConfiguration保存的。
  %3 = call i32 @__cudaPopCallConfiguration(ptr %grid_dim, ptr %block_dim, ptr %shmem_size, ptr %stream)
根据配置结果，使用@cudaLaunchKernel调用device侧函数。
  %call = call noundef i32 @cudaLaunchKernel(ptr noundef @_Z19__device_stub__axpyfPfS_, i64 %7, i32 %9, i64 %11, i32 %13, ptr noundef %kernel_args, i64 noundef %4, ptr noundef %5)
```
对于cudaMalloc这样的函数
```
  %device_x = alloca ptr, align 8
  %call = call noundef i32 @_ZL10cudaMallocIfE9cudaErrorPPT_m(ptr noundef %device_x, i64 noundef 16)
```
直接调用了运行时实现，对应的device_x指针应该指向底层运行时的结构，能够通过运行时找到Device的地址并完成数据传递。

## device侧bitcode文件生成
```
%struct.__cuda_builtin_threadIdx_t = type { i8 }
这是在内部头文件定义的Struct类型，只有函数没有数据
$_ZN26__cuda_builtin_threadIdx_t17__fetch_builtin_xEv = comdat any
comdata key，提示这个函数可能在多个目标文件出现，链接器碰到如何处理这样的函数。
@threadIdx = extern_weak dso_local addrspace(1) global %struct.__cuda_builtin_threadIdx_t, align 1
这是一个外部全局变量@threadIdx

这是实际的设备侧和函数，会生成Fatbin文件放到section中，为什么没有看到地址空间类型的指示？
define dso_local void @_Z4axpyfPfS_(float noundef %a, ptr noundef %x, ptr noundef %y) #0 {
entry:
  %a.addr = alloca float, align 4
  %x.addr = alloca ptr, align 8
  %y.addr = alloca ptr, align 8
  store float %a, ptr %a.addr, align 4, !tbaa !8
  store ptr %x, ptr %x.addr, align 8, !tbaa !12
  store ptr %y, ptr %y.addr, align 8, !tbaa !12
  %0 = load float, ptr %a.addr, align 4, !tbaa !8
  %1 = load ptr, ptr %x.addr, align 8, !tbaa !12
  %call = call noundef i32 @_ZN26__cuda_builtin_threadIdx_t17__fetch_builtin_xEv() #3
  %idxprom = zext i32 %call to i64
  %arrayidx = getelementptr inbounds float, ptr %1, i64 %idxprom
  %2 = load float, ptr %arrayidx, align 4, !tbaa !8
  %mul = fmul contract float %0, %2
  %3 = load ptr, ptr %y.addr, align 8, !tbaa !12
  %call1 = call noundef i32 @_ZN26__cuda_builtin_threadIdx_t17__fetch_builtin_xEv() #3
  %idxprom2 = zext i32 %call1 to i64
  %arrayidx3 = getelementptr inbounds float, ptr %3, i64 %idxprom2
  store float %mul, ptr %arrayidx3, align 4, !tbaa !8
  ret void
}
前面的函数
define linkonce_odr dso_local noundef i32 @_ZN26__cuda_builtin_threadIdx_t17__fetch_builtin_xEv() #1 comdat align 2 {
entry:
  %0 = call i32 @llvm.nvvm.read.ptx.sreg.tid.x()
  ret i32 %0
}
```




