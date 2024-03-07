先简单分析一下clang侧代码
## clang/lib/CodeGen/CGCUDANV.cpp
```
class CGNVCUDARuntime : public CGCUDARuntime {
```
生成很多运行时需要的函数。可以通过分析bitcode文件检查这些功能。

