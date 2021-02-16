---
title: ART中编译器相关基础知识记录
keywords: art, compiler
last_updated: Aug 20, 2020
created: 2020-08-20
tags: [art,default]
summary: "本文用于记录ART中基本的一些编译器数据结构等"
sidebar: mydoc_sidebar
permalink: 2020-08-20-art_compiler_basics.html
folder: vee/art
---

## 基础数据结构简介

vim art/compiler/optimizing/nodes.h +312
```
// Control-flow graph of a method. Contains a list of basic blocks.
class HGraph : public ArenaObject<kArenaAllocGraph> {
 public:
  HGraph(ArenaAllocator* allocator,
         ArenaStack* arena_stack,
         const DexFile& dex_file,
         uint32_t method_idx,
         InstructionSet instruction_set,
         InvokeType invoke_type = kInvalidInvokeType,
         bool dead_reference_safe = false,
         bool debuggable = false,
         bool osr = false,
         int start_instruction_id = 0)
      : allocator_(allocator),
        arena_stack_(arena_stack),
        blocks_(allocator->Adapter(kArenaAllocBlockList)),
        reverse_post_order_(allocator->Adapter(kArenaAllocReversePostOrder)),
        linear_order_(allocator->Adapter(kArenaAllocLinearOrder)),
        entry_block_(nullptr),
        exit_block_(nullptr),
        maximum_number_of_out_vregs_(0),
        number_of_vregs_(0),
        number_of_in_vregs_(0),
        temporaries_vreg_slots_(0),
        has_bounds_checks_(false),
        has_try_catch_(false),
        has_simd_(false),
        has_loops_(false),
        has_irreducible_loops_(false),
        dead_reference_safe_(dead_reference_safe),
        debuggable_(debuggable),
        current_instruction_id_(start_instruction_id),
        dex_file_(dex_file),
        method_idx_(method_idx),
        invoke_type_(invoke_type),
        in_ssa_form_(false),
        number_of_cha_guards_(0),
        instruction_set_(instruction_set),
        cached_null_constant_(nullptr),
        cached_int_constants_(std::less<int32_t>(), allocator->Adapter(kArenaAllocConstantsMap)),
        cached_float_constants_(std::less<int32_t>(), allocator->Adapter(kArenaAllocConstantsMap)),
        cached_long_constants_(std::less<int64_t>(), allocator->Adapter(kArenaAllocConstantsMap)),
        cached_double_constants_(std::less<int64_t>(), allocator->Adapter(kArenaAllocConstantsMap)),
        cached_current_method_(nullptr),
        art_method_(nullptr),
        inexact_object_rti_(ReferenceTypeInfo::CreateInvalid()),
        osr_(osr),
        cha_single_implementation_list_(allocator->Adapter(kArenaAllocCHA)) {
    blocks_.reserve(kDefaultNumberOfBlocks);
  }
```
呐，头部的注释就很清楚的说明了这个类HGraph的作用，即方法粒度的控制流图，内含一列基本块

vim art/compiler/optimizing/nodes.h +969
```
/ A block in a method. Contains the list of instructions represented
// as a double linked list. Each block knows its predecessors and
// successors.

class HBasicBlock : public ArenaObject<kArenaAllocBasicBlock> {
 public:
  explicit HBasicBlock(HGraph* graph, uint32_t dex_pc = kNoDexPc)
      : graph_(graph),
        predecessors_(graph->GetAllocator()->Adapter(kArenaAllocPredecessors)),
        successors_(graph->GetAllocator()->Adapter(kArenaAllocSuccessors)),
        loop_information_(nullptr),
        dominator_(nullptr),
        dominated_blocks_(graph->GetAllocator()->Adapter(kArenaAllocDominated)),
        block_id_(kInvalidBlockId),
        dex_pc_(dex_pc),
        lifetime_start_(kNoLifetime),
        lifetime_end_(kNoLifetime),
        try_catch_information_(nullptr) {
    predecessors_.reserve(kDefaultNumberOfPredecessors);
    successors_.reserve(kDefaultNumberOfSuccessors);
    dominated_blocks_.reserve(kDefaultNumberOfDominatedBlocks);
  }
```


vim art/compiler/optimizing/block_builder.h +28

```
class HBasicBlockBuilder : public ValueObject {
 public:
  HBasicBlockBuilder(HGraph* graph,
                     const DexFile* const dex_file,
                     const CodeItemDebugInfoAccessor& accessor,
                     ScopedArenaAllocator* local_allocator);

  // Creates basic blocks in `graph_` at branch target dex_pc positions of the
  // `code_item_`. Blocks are connected but left unpopulated with instructions.
  // TryBoundary blocks are inserted at positions where control-flow enters/
  // exits a try block.
  bool Build();

  // Creates basic blocks in `graph_` for compiling an intrinsic.
  void BuildIntrinsic();

  size_t GetNumberOfBranches() const { return number_of_branches_; }
  HBasicBlock* GetBlockAt(uint32_t dex_pc) const { return branch_targets_[dex_pc]; }

  size_t GetQuickenIndex(uint32_t dex_pc) const;

 private:
  // Creates a basic block starting at given `dex_pc`.
  HBasicBlock* MaybeCreateBlockAt(uint32_t dex_pc);

  // Creates a basic block for bytecode instructions at `semantic_dex_pc` and
  // stores it under the `store_dex_pc` key. This is used when multiple blocks
  // share the same semantic dex_pc, e.g. when building switch decision trees.
  HBasicBlock* MaybeCreateBlockAt(uint32_t semantic_dex_pc, uint32_t store_dex_pc);

  bool CreateBranchTargets();
  void ConnectBasicBlocks();
  void InsertTryBoundaryBlocks();

  // Helper method which decides whether `catch_block` may have live normal
  // predecessors and thus whether a synthetic catch block needs to be created
  // to avoid mixing normal and exceptional predecessors.
  // Should only be called during InsertTryBoundaryBlocks on blocks at catch
  // handler dex_pcs.
  bool MightHaveLiveNormalPredecessors(HBasicBlock* catch_block);

  ArenaAllocator* const allocator_;
  HGraph* const graph_;

  const DexFile* const dex_file_;
  CodeItemDataAccessor code_item_accessor_;  // null code item for intrinsic graph.

  ScopedArenaAllocator* const local_allocator_;
  ScopedArenaVector<HBasicBlock*> branch_targets_;
  ScopedArenaVector<HBasicBlock*> throwing_blocks_;
  size_t number_of_branches_;

  // A table to quickly find the quicken index for the first instruction of a basic block.
  ScopedArenaSafeMap<uint32_t, uint32_t> quicken_index_for_dex_pc_;

  static constexpr size_t kDefaultNumberOfThrowingBlocks = 2u;

  DISALLOW_COPY_AND_ASSIGN(HBasicBlockBuilder);
};

}  // namespace art
```

vim art/compiler/optimizing/optimizing_compiler.cc  +667
```
OptimizationDef optimizations[] = {
    // Initial optimizations.
    OptDef(OptimizationPass::kConstantFolding),
    OptDef(OptimizationPass::kInstructionSimplifier),
    OptDef(OptimizationPass::kDeadCodeElimination,
           "dead_code_elimination$initial"),
    // Inlining.
    OptDef(OptimizationPass::kInliner),
    // Simplification (only if inlining occurred).
    OptDef(OptimizationPass::kConstantFolding,
           "constant_folding$after_inlining",
           OptimizationPass::kInliner),
    OptDef(OptimizationPass::kInstructionSimplifier,
           "instruction_simplifier$after_inlining",
           OptimizationPass::kInliner),
    OptDef(OptimizationPass::kDeadCodeElimination,
           "dead_code_elimination$after_inlining",
           OptimizationPass::kInliner),
    // GVN.
    OptDef(OptimizationPass::kSideEffectsAnalysis,
           "side_effects$before_gvn"),
    OptDef(OptimizationPass::kGlobalValueNumbering),
    // Simplification (TODO: only if GVN occurred).
    OptDef(OptimizationPass::kSelectGenerator),
    OptDef(OptimizationPass::kConstantFolding,
           "constant_folding$after_gvn"),
    OptDef(OptimizationPass::kInstructionSimplifier,
           "instruction_simplifier$after_gvn"),
    OptDef(OptimizationPass::kDeadCodeElimination,
           "dead_code_elimination$after_gvn"),
    // High-level optimizations.
    OptDef(OptimizationPass::kSideEffectsAnalysis,
           "side_effects$before_licm"),
    OptDef(OptimizationPass::kInvariantCodeMotion),
    OptDef(OptimizationPass::kInductionVarAnalysis),
    OptDef(OptimizationPass::kBoundsCheckElimination),
    OptDef(OptimizationPass::kLoopOptimization),
    // Simplification.
    OptDef(OptimizationPass::kConstantFolding,
           "constant_folding$after_bce"),
    OptDef(OptimizationPass::kInstructionSimplifier,
           "instruction_simplifier$after_bce"),
    // Other high-level optimizations.
    OptDef(OptimizationPass::kSideEffectsAnalysis,
           "side_effects$before_lse"),
    OptDef(OptimizationPass::kLoadStoreAnalysis),
    OptDef(OptimizationPass::kLoadStoreElimination),
    OptDef(OptimizationPass::kCHAGuardOptimization),
    OptDef(OptimizationPass::kDeadCodeElimination,
           "dead_code_elimination$final"),
    OptDef(OptimizationPass::kCodeSinking),
    // The codegen has a few assumptions that only the instruction simplifier
    // can satisfy. For example, the code generator does not expect to see a
    // HTypeConversion from a type to the same type.
    OptDef(OptimizationPass::kInstructionSimplifier,
           "instruction_simplifier$before_codegen"),
    // Eliminate constructor fences after code sinking to avoid
    // complicated sinking logic to split a fence with many inputs.
    OptDef(OptimizationPass::kConstructorFenceRedundancyElimination)
  };
```












