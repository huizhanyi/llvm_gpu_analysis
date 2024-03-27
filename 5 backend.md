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
1585     FuncInfo->MBB = FuncInfo->MBBMap[LLVMBB];
设置当前处理的基本块。
1751       SelectBasicBlock(Begin, BI, HadTailCall);
处理FastISel没有处理的部分
```
处理单个BasicBlock
```
 715 void SelectionDAGISel::SelectBasicBlock(BasicBlock::const_iterator Begin,
 716                                         BasicBlock::const_iterator End,
 717                                         bool &HadTailCall) {
 718   // Allow creating illegal types during DAG building for the basic block.
 719   CurDAG->NewNodesMustHaveLegalTypes = false;
允许生成非法类型
 721   // Lower the instructions. If a call is emitted as a tail call, cease emitting
 722   // nodes for this block. If an instruction is elided, don't emit it, but do
 723   // handle any debug-info attached to it.
 724   for (BasicBlock::const_iterator I = Begin; I != End && !SDB->HasTailCall; ++I) {
 725     if (!ElidedArgCopyInstrs.count(&*I))
 726       SDB->visit(*I);
SDB是SelectionDAGBuilder结构
 727     else
处理指令调试信息
 728       SDB->visitDbgInfo(*I);
 729   }
Lower每个指令。
```
llvm/lib/CodeGen/SelectionDAG/SelectionDAGBuilder.cpp
```
 1288 void SelectionDAGBuilder::visit(const Instruction &I) {
 1289   visitDbgInfo(I);
 1290
 1291   // Set up outgoing PHI node register values before emitting the terminator.
 1292   if (I.isTerminator()) {
 1293     HandlePHINodesInSuccessorBlocks(I.getParent());
 1294   }
 1295
 1296   // Increase the SDNodeOrder if dealing with a non-debug instruction.
 1297   if (!isa<DbgInfoIntrinsic>(I))
 1298     ++SDNodeOrder;
 1299
 1300   CurInst = &I;
 1301
 1302   // Set inserted listener only if required.
 1303   bool NodeInserted = false;
 1304   std::unique_ptr<SelectionDAG::DAGNodeInsertedListener> InsertedListener;
 1305   MDNode *PCSectionsMD = I.getMetadata(LLVMContext::MD_pcsections);
 1306   if (PCSectionsMD) {
 1307     InsertedListener = std::make_unique<SelectionDAG::DAGNodeInsertedListener>(
 1308         DAG, [&](SDNode *) { NodeInserted = true; });
 1309   }
根据操作码处理指令，定义如后面。
 1311   visit(I.getOpcode(), I);
 1312
 1313   if (!I.isTerminator() && !HasTailCall &&
 1314       !isa<GCStatepointInst>(I)) // statepoints handle their exports internally
 1315     CopyToExportRegsIfNeeded(&I);
 1316
 1317   // Handle metadata.
 1318   if (PCSectionsMD) {
 1319     auto It = NodeMap.find(&I);
 1320     if (It != NodeMap.end()) {
 1321       DAG.addPCSections(It->second.getNode(), PCSectionsMD);
 1322     } else if (NodeInserted) {
 1323       // This should not happen; if it does, don't let it go unnoticed so we can
 1324       // fix it. Relevant visit*() function is probably missing a setValue().
 1325       errs() << "warning: loosing !pcsections metadata ["
 1326              << I.getModule()->getName() << "]\n";
 1327       LLVM_DEBUG(I.dump());
 1328       assert(false);
 1329     }
 1330   }
 1331
 1332   CurInst = nullptr;
 1333 }
```
llvm/lib/CodeGen/SelectionDAG/SelectionDAGBuilder.cpp
```
 1339 void SelectionDAGBuilder::visit(unsigned Opcode, const User &I) {
 1340   // Note: this doesn't use InstVisitor, because it has to work with
 1341   // ConstantExpr's in addition to instructions.
 1342   switch (Opcode) {
 1343   default: llvm_unreachable("Unknown instruction type encountered!");
 1344     // Build the switch statement using the Instruction.def file.
 1345 #define HANDLE_INST(NUM, OPCODE, CLASS) \
 1346     case Instruction::OPCODE: visit##OPCODE((const CLASS&)I); break;
根据指令的OPCODE，调用对应的visitOPCODE函数。
 1347 #include "llvm/IR/Instruction.def"
 1348   }
 1349 }
```
这里调用对应的visitOPCODE函数，例如visitRet函数，visitCallBr函数。这些函数都是“class SelectionDAGBuilder”的方法函数。

返回SelectionDAGISel::SelectBasicBlock函数
```
 731   // Make sure the root of the DAG is up-to-date.
 732   CurDAG->setRoot(SDB->getControlRoot());

 737   // Final step, emit the lowered DAG as machine code.
 738   CodeGenAndEmitDAG();
```
代码生成，直到生成机器码，流程图
![image](https://github.com/huizhanyi/llvm_gpu_analysis/blob/main/SelectionDAG_Overview.png)
来自：https://csstormq.github.io/blog/LLVM%20%E4%B9%8B%E5%90%8E%E7%AB%AF%E7%AF%87%EF%BC%884%EF%BC%89%EF%BC%9A%E7%90%86%E8%A7%A3%E6%8C%87%E4%BB%A4%E9%80%89%E6%8B%A9%E7%9A%84%20dump%20%E8%BE%93%E5%87%BA

llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp
```
 778 void SelectionDAGISel::CodeGenAndEmitDAG() {
```
依次生成各种DAG形式。
```
 805   ISEL_DUMP(dbgs() << "\nInitial selection DAG: "
 806                    << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 807                    << "'\n"
 808             CurDAG->dump());
dump初始的DAG形式
```
```
 822     CurDAG->Combine(BeforeLegalizeTypes, AA, OptLevel);

 825   ISEL_DUMP(dbgs() << "\nOptimized lowered selection DAG: "
 826                    << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 827                    << "'\n";
 828             CurDAG->dump());
```
dump经过Optimized lowered selection DAG的DAG
```
 844     Changed = CurDAG->LegalizeTypes();
 847   ISEL_DUMP(dbgs() << "\nType-legalized selection DAG: "
 848                    << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 849                    << "'\n";
 850             CurDAG->dump());
```
dump经过Type-legalized selection DAG的DAG
```
 868       CurDAG->Combine(AfterLegalizeTypes, AA, OptLevel);
 871     ISEL_DUMP(dbgs() << "\nOptimized type-legalized selection DAG: "
 872                      << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 873                      << "'\n";
 874               CurDAG->dump());
```
dump经过Optimized type-legalized selection DAG的DAG
```
 868       CurDAG->Combine(AfterLegalizeTypes, AA, OptLevel);

 871     ISEL_DUMP(dbgs() << "\nOptimized type-legalized selection DAG: "
 872                      << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 873                      << "'\n";
 874               CurDAG->dump());
```
dump经过Optimized type-legalized selection DAG的DAG
```
 885     Changed = CurDAG->LegalizeVectors();
 888   if (Changed) {
 889     ISEL_DUMP(dbgs() << "\nVector-legalized selection DAG: "
 890                      << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 891                      << "'\n";
 892               CurDAG->dump());
```
dump经过Vector-legalized selection DAG的DAG
这里因为DAG没有变化，不会Dump
```
 942     CurDAG->Legalize();
 945   ISEL_DUMP(dbgs() << "\nLegalized selection DAG: "
 946                    << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 947                    << "'\n";
 948             CurDAG->dump());
```
dump经过Legalized selection DAG的DAG
```
 986     DoInstructionSelection();
 989   ISEL_DUMP(dbgs() << "\nSelected selection DAG: "
 990                    << printMBBReference(*FuncInfo->MBB) << " '" << BlockName
 991                    << "'\n";
 992             CurDAG->dump());
```
完成Instruction selection
dump经过Selected selection DAG的DAG
```
 998   ScheduleDAGSDNodes *Scheduler = CreateScheduler();
1002     Scheduler->Run(CurDAG, FuncInfo->MBB);
1017     LastMBB = FuncInfo->MBB = Scheduler->EmitSchedule(FuncInfo->InsertPt);
```
完成调度？返回llvm::SelectionDAGISel::SelectBasicBlock
返回llvm::SelectionDAGISel::SelectAllBasicBlocks
返回llvm::SelectionDAGISel::runOnMachineFunction
```
 528   MachineRegisterInfo &MRI = MF->getRegInfo();
 529   for (DenseMap<Register, Register>::iterator I = FuncInfo->RegFixups.begin(),
 530                                               E = FuncInfo->RegFixups.end();
```
用理想的值取代前向声明的寄存器
```
 559   RegInfo->EmitLiveInCopies(EntryMBB, TRI, *TII);
```
```
 693   ISEL_DUMP(dbgs() << "*** MachineFunction at end of ISel ***\n");
 694   ISEL_DUMP(MF->print(dbgs()));
```
结束ISel，dump结构。
返回llvm::SelectionDAGISel::runOnMachineFunction
返回llvm::NVPTXDAGToDAGISel::runOnMachineFunction
返回llvm::MachineFunctionPass::runOnFunction

## Finalize ISel and expand pseudo-instructions
lib/CodeGen/FinalizeISel.cpp
```
 46 bool FinalizeISel::runOnMachineFunction(MachineFunction &MF) {
 47   bool Changed = false;
 48   const TargetLowering *TLI = MF.getSubtarget().getTargetLowering();
实际调用llvm::NVPTXSubtarget::getTargetLowering，返回NVPTXTargetLowering
 49
 50   // Iterate through each instruction in the function, looking for pseudos.
 51   for (MachineFunction::iterator I = MF.begin(), E = MF.end(); I != E; ++I) {
 52     MachineBasicBlock *MBB = &*I;
 53     for (MachineBasicBlock::iterator MBBI = MBB->begin(), MBBE = MBB->end();
 54          MBBI != MBBE; ) {
 55       MachineInstr &MI = *MBBI++;
 56
 57       // If MI is a pseudo, expand it.
 58       if (MI.usesCustomInsertionHook()) {
 59         Changed = true;
 60         MachineBasicBlock *NewMBB = TLI->EmitInstrWithCustomInserter(MI, MBB);
这是后端目标需要自定义的方法，但是NVPTX后端没有定义，是不涉及这样伪指令吗？
 61         // The expansion may involve new basic blocks.
 62         if (NewMBB != MBB) {
 63           MBB = NewMBB;
 64           I = NewMBB->getIterator();
 65           MBBI = NewMBB->begin();
 66           MBBE = NewMBB->end();
 67         }
 68       }
 69     }
 70   }
 71
 72   TLI->finalizeLowering(MF);
NVPTX后端没有特殊定义这个函数。
 73
 74   return Changed;
 75 }
```
## Local Stack Slot Allocation
CodeGen/LocalStackSlotAllocation.cpp
```
112 bool LocalStackSlotPass::runOnMachineFunction(MachineFunction &MF) {
113   MachineFrameInfo &MFI = MF.getFrameInfo();
114   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
115   unsigned LocalObjectCount = MFI.getObjectIndexEnd();
116
117   // If the target doesn't want/need this pass, or if there are no locals
118   // to consider, early exit.
119   if (LocalObjectCount == 0 || !TRI->requiresVirtualBaseRegisters(MF))
120     return false;
121
122   // Make sure we have enough space to store the local offsets.
123   LocalOffsets.resize(MFI.getObjectIndexEnd());
124
125   // Lay out the local blob.
126   calculateFrameObjectOffsets(MF);
127
128   // Insert virtual base registers to resolve frame index references.
129   bool UsedBaseRegs = insertFrameReferenceRegisters(MF);
130
131   // Tell MFI whether any base registers were allocated. PEI will only
132   // want to use the local block allocations from this pass if there were any.
133   // Otherwise, PEI can do a bit better job of getting the alignment right
134   // without a hole at the start since it knows the alignment of the stack
135   // at the start of local allocation, and this pass doesn't.
136   MFI.setUseLocalStackAllocationBlock(UsedBaseRegs);
137
138   return true;
139 }
```
