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
```
由-O2调整为-O0，打开FastISel
```
Changing optimization level for Function _Z8multiplyff
        Before: -O2 ; After: -O0
        FastISel is enabled
```

```
 445   TII = MF->getSubtarget().getInstrInfo();
对应子类NVPTXInstrInfo
 446   TLI = MF->getSubtarget().getTargetLowering();
对应子类NVPTXTargetLowering
取目标相关信息
 461   ISEL_DUMP(dbgs() << "\n\n\n=== " << FuncName << "\n");
打印当前函数名
```
```
=== _Z8multiplyff
```
```
 466   CurDAG->init(*MF, *ORE, this, LibInfo, UA, PSI, BFI, FnVarLocs);
当前类中包括SelectionDAG *CurDAG定义。
```
init定义llvm/lib/CodeGen/SelectionDAG/SelectionDAG.cpp,根据定义，用给定的MF准备SelectionDAG，生成代码。
```
 455   /// Prepare this SelectionDAG to process code in the given MachineFunction.

 1326 void SelectionDAG::init(MachineFunction &NewMF,
 1327                         OptimizationRemarkEmitter &NewORE, Pass *PassPtr,
 1328                         const TargetLibraryInfo *LibraryInfo,
 1329                         UniformityInfo *NewUA, ProfileSummaryInfo *PSIin,
 1330                         BlockFrequencyInfo *BFIin,
 1331                         FunctionVarLocs const *VarLocs) {
 1332   MF = &NewMF;
 1333   SDAGISelPass = PassPtr;
 1334   ORE = &NewORE;
 1335   TLI = getSubtarget().getTargetLowering();
 1336   TSI = getSubtarget().getSelectionDAGInfo();
 1337   LibInfo = LibraryInfo;
 1338   Context = &MF->getFunction().getContext();
 1339   UA = NewUA;
 1340   PSI = PSIin;
 1341   BFI = BFIin;
 1342   FnVarLocs = VarLocs;
 1343 }
```
```
 467   FuncInfo->set(Fn, *MF, CurDAG);
 468   SwiftError->setFunction(*MF);
```
初始化FunctionLoweringInfo结构和SwiftErrorValueTracking结构
```
 485   SDB->init(GFI, AA, AC, LibInfo);
```
初始化SelectionDAGBuilder
```
 511   MachineBasicBlock *EntryMBB = &MF->front();
```
取MachineBasicBlock入口块,MachineBasicBlock定义如下，是BasicBlock的一个wrapper类或者扩展类
```
 101 class MachineBasicBlock
 102     : public ilist_node_with_parent<MachineBasicBlock, MachineFunction> {
 119   const BasicBlock *BB;
```
llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp
```
 516   SelectAllBasicBlocks(Fn);
```
遍历所有基本块，处理之
```
1489 void SelectionDAGISel::SelectAllBasicBlocks(const Function &Fn) {
1506   FuncInfo->MBB = FuncInfo->MBBMap[&Fn.getEntryBlock()];
设置当前处理的基本块
1507   FuncInfo->InsertPt = FuncInfo->MBB->begin();
当前的插入点？
1509   CurDAG->setFunctionLoweringInfo(FuncInfo.get());
设置当前的FunctionLoweringInfo
1511   if (!FastIS) {
1512     LowerArguments(Fn);
调用SelectionDAGISel::LowerArguments，考虑lower arguments
1557   // Iterate over all basic blocks in the function.
1558   StackProtector &SP = getAnalysis<StackProtector>();
1559   for (const BasicBlock *LLVMBB : RPOT) {
迭代处理所有基本块
1580     BasicBlock::const_iterator const Begin =
1581         LLVMBB->getFirstNonPHI()->getIterator();
1582     BasicBlock::const_iterator const End = LLVMBB->end();
取当前块的开始和结束位置
```


