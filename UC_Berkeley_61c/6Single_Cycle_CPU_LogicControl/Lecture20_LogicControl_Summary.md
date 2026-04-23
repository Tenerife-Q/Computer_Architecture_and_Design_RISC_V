# UC Berkeley 61C - Lecture 20: Single-Cycle CPU Logic Control (单周期 CPU 控制逻辑) 总结

本章主要基于 UC Berkeley CS61C 第20讲的内容，详细分析了在单周期 RISC-V 处理器中如何实现“控制逻辑 (Logic Control)”。控制逻辑是 CPU 的“大脑”，它负责解析从指令内存中提取的机器指令，并为整个数据通路（Datapath）生成相匹配的操作与选择信号。

## 1. 控制逻辑 (Control Logic) 的核心作用

数据通路 (Datapath) 负责数据流的运算与传输，而 **控制逻辑 (Control Logic)** 决定了这套通路该具体进行“哪种操作”。
控制单元（Control Unit）将输入的 **32位指令** 截取其中的关键字段：
- **`opcode`** (指令操作码: bits [6:0])
- **`funct3`** (3位功能码: bits [14:12])
- **`funct7`** (7位功能码: bits [31:25])

基于这些字段，控制单元输出一系列开关信号（控制信号），驱动各类多路选择器 (MUX) 和功能组件如 ALU 或存储器。

## 2. 关键的控制信号 (Control Signals)

常见的控制信号（如截图中的真值表与原理图所示）分为几个主要的类别：

### MUX 选择控制 (Multiplexer Selection)
*   **`ALUSrc`**: 控制 ALU 第二个输入源。`0` 代表来自寄存器 (Register 2)，`1` 代表来自立即数生成器 (Imm Gen)。
*   **`MemtoReg`**: 控制写回寄存器堆的数据来源。`0` 代表来自 ALU 的计算结果，`1` 代表来自数据存储器 (Data Memory)。
*   **`PCSrc` (由 Branch 和 ALU 标志产生)**: 判断是否进行分支跳转。

### 读/写使能控制 (Read/Write Enables)
*   **`RegWrite`**: 寄存器堆写使能信号。`1` 代表允许将结果写入目标寄存器 (`rd`)。
*   **`MemRead`**: 数据存储器读使能信号。Load 指令专用。
*   **`MemWrite`**: 数据存储器写使能信号。Store 指令专用。

### 运算操作控制 (Operation Selection)
*   **`ALUOp`** / **ALU Control**: 决定 ALU 内部进行哪种运算（如加法 Add、减法 Sub、与 And、或 Or、小于置位 Slt 等）。

## 3. 两级控制架构 (Two-Level Control Structure)

为了简化复杂性，通常将控制逻辑划分为两级：

1.  **Main Control (主控制单元)**
    *   **输入**: 仅 `opcode` (bits [6:0])。
    *   **输出**: 大部分 MUX 和读写使能信号（ALUSrc, MemtoReg, RegWrite, MemRead, MemWrite, Branch），以及一个粗略的 **ALUOp**（如 2-bit 的信号，用来指示这是 Load/Store、Branch 还是 R-type 操作）。

2.  **ALU Control (ALU 控制单元)**
    *   **输入**: 主控制输出的 `ALUOp` 加上指令内的 `funct3` 和 `funct7`。
    *   **输出**: 最终送入 ALU 的 4-bit 实际操作信号。
    *   **优势**: 降低解码逻辑的耦合度和硬件设计的复杂度。

## 4. ROM 与真值表实现 (ROM implementation)

在截图中可以看到真值表 (Truth Table) 会将指令对应的控制信号列出：
*   **R-Format** (`add`, `sub` 等): RegWrite=1, ALUSrc=0, MemRead=0, MemWrite=0, MemtoReg=0。
*   **Load** (`lw` 等): RegWrite=1, ALUSrc=1, MemRead=1, MemWrite=0, MemtoReg=1。
*   **Store** (`sw` 等): RegWrite=0, ALUSrc=1, MemRead=0, MemWrite=1, MemtoReg=X (Don't Care)。
*   **Branch** (`beq` 等): RegWrite=0, ALUSrc=0, MemRead=0, MemWrite=0, Branch=1。

这些真值表逻辑可以直接通过只读存储器（ROM - Read Only Memory）或者组合逻辑门（AND/OR/Inverter）电路来实现，从而完成了从固化的 Software 到动态运行的 Hardware 的映射转换。

## 总结

第6章的内容成功将“指令集架构 (ISA)”与上一章的“数据通路 (Datapath)”缝合起来。通过控制逻辑的精密调度，单周期 CPU 能够在一个时钟周期内完整执行每一条不同格式的 RISC-V 指令，为后续学习流水线 (Pipelining) 打下了基础。
