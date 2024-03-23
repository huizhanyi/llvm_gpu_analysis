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
### MachineFunctionPass
MachineFunctionPass.cpp
```
 39 bool MachineFunctionPass::runOnFunction(Function &F) {
 93   bool RV = runOnMachineFunction(MF);
```
从这里可以看出，MachineFunctionPass的runOnFunction会调用runOnMachineFunction函数，因此MachineFunctionPass可以通过定义runOnMachineFunction改变自己的行为，
但是每个MachineFunctionPass都有一个公共的入口runOnFunction。
下面检查这个runOnFunction的实现，提供了那些公共功能？
```
 39 bool MachineFunctionPass::runOnFunction(Function &F) {
 40   // Do not codegen any 'available_externally' functions at all, they have
 41   // definitions outside the translation unit.
 42   if (F.hasAvailableExternallyLinkage())
 43     return false;
```
外部Linkage函数，不需要生成代码
```
 45   MachineModuleInfo &MMI = getAnalysis<MachineModuleInfoWrapperPass>().getMMI();
 46   MachineFunction &MF = MMI.getOrCreateMachineFunction(F);
```
这里通过Function生成MachineFunction类。
```
 48   MachineFunctionProperties &MFProps = MF.getProperties();
```
取MachineFunctionProperties。
```
 71   if (ShouldEmitSizeRemarks)
 72     CountBefore = MF.getInstructionCount();
```
收集指令数
```
 86   if (ShouldPrintChanged) {
 87     raw_svector_ostream OS(BeforeStr);
 88     MF.print(OS);
 89   }
```
打印MachineFunction
MachineFunction.cpp
```
 602 void MachineFunction::print(raw_ostream &OS, const SlotIndexes *Indexes) const {
 603   OS << "# Machine code for function " << getName() << ": ";
```
```
 93   bool RV = runOnMachineFunction(MF);
```
调用runOnMachineFunction
```
123     if (ShouldPrintChanged) {
124       raw_svector_ostream OS(AfterStr);
125       MF.print(OS);
126     }
```
Pass后打印
### 调试PASS
NVPTXISelDAGToDAG.cpp
```
  50 bool NVPTXDAGToDAGISel::runOnMachineFunction(MachineFunction &MF) {
MF中有一个TargetSubtargetInfo类型的指针，这里将对应类型转化为NVPTXSubtarget子类指针，保存在当前类的Subtarget指针。
  51   Subtarget = &MF.getSubtarget<NVPTXSubtarget>();
进入子类SelectionDAGISel的函数runOnMachineFunction
  52   return SelectionDAGISel::runOnMachineFunction(MF);
  53 }
```
llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp
```
 409 bool SelectionDAGISel::runOnMachineFunction(MachineFunction &mf) {
 410   // If we already selected that function, we do not need to run SDISel.
 411   if (mf.getProperties().hasProperty(
 412           MachineFunctionProperties::Property::Selected))
 413     return false;
如果已经Select函数，不需要再运行
 418   const Function &Fn = mf.getFunction();
 419   MF = &mf;
取Function，保存MachineFunction
 438   TM.resetTargetOptions(Fn);
复位函数的一些选项
 440   CodeGenOptLevel NewOptLevel = OptLevel;
 441   if (OptLevel != CodeGenOptLevel::None && skipFunction(Fn))
 442     NewOptLevel = CodeGenOptLevel::None;
 443   OptLevelChanger OLC(*this, NewOptLevel);
优化层次相关设置
 445   TII = MF->getSubtarget().getInstrInfo();
对应子类NVPTXInstrInfo
 446   TLI = MF->getSubtarget().getTargetLowering();
对应子类NVPTXTargetLowering
取目标相关信息
```


