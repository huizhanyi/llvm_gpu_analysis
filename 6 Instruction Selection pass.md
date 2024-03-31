![image](https://github.com/huizhanyi/llvm_gpu_analysis/blob/main/SelectionDAG_Overview.png)
这里对NVPTX DAG->DAG Pattern Instruction Selection进行详细分析
```
llc -mcpu=sm_60 -mattr=+ptx83 axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc -debug-only=isel
llc -mcpu=sm_60 -mattr=+ptx83 axpy-cuda-nvptx64-nvidia-cuda-sm_60.bc -view-dag-combine1-dags
```
NVPTXISelDAGToDAG.cpp/NVPTXISelDAGToDAG.h
```
class LLVM_LIBRARY_VISIBILITY NVPTXDAGToDAGISel : public SelectionDAGISel {
```
