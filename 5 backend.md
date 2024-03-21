## NVPTX DAG->DAG Pattern Instruction Selection
```
llc -mcpu=sm_60 -mattr=+ptx83 axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc -debug-only=isel
llc -mcpu=sm_60 -mattr=+ptx83 axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc -view-dag-combine1-dags
```
NVPTXISelDAGToDAG.cpp/NVPTXISelDAGToDAG.h
```
class LLVM_LIBRARY_VISIBILITY NVPTXDAGToDAGISel : public SelectionDAGISel {
```
这是一个MachineFunctionPass，也是一个FunctionPass。所以MachineFunctionPass也可以类似FunctionPass登记在遍管理器里。
MachineFunctionPass.cpp
```
 39 bool MachineFunctionPass::runOnFunction(Function &F) {
 93   bool RV = runOnMachineFunction(MF);
```
从这里可以看出，MachineFunctionPass的runOnFunction会调用runOnMachineFunction函数，因此MachineFunctionPass可以通过定义runOnMachineFunction改变自己的行为，
但是每个MachineFunctionPass都有一个公共的入口runOnFunction。
