# ⚙️ FPGA-Augmented RISC-V LLVM Backend — Full System Integration

This repository represents the **top-level integration** of the FPGA-augmented RISC-V compiler and hardware system.  
It connects a **custom LLVM backend** (software side) with a **FPGA accelerator** (hardware side) for real-time **YUV→RGB image conversion**, running on the **BeagleV-Fire** development platform.

---

## 🧭 Project Overview

The **FPGA-Augmented RISC-V LLVM Backend** project demonstrates how a RISC-V processor can be extended through:
- a **compiler-level pseudo-instruction**, generated automatically by LLVM/Clang, and  
- a **hardware accelerator** implemented in Verilog and deployed on an FPGA fabric.

Together, these layers create a **hardware/software co-design framework** that enables transparent offloading of computation from CPU to FPGA — entirely managed by the compiler.

---

## 🧠 What We Want to Achieve

Our goal is to build a **compiler-driven heterogeneous platform** where:
- Hardware accelerators can be triggered directly by compiler intrinsics, without OS drivers.
- The RISC-V instruction set can be **dynamically augmented** with new pseudo-instructions, each mapped to a specific FPGA function.
- The entire flow (software to hardware) is **reproducible and open-source**, serving as an educational and research reference for **compiler design, hardware acceleration, and co-optimization**.

The **first case study** focuses on **YUV422 → RGB conversion**, but the same mechanism can later be extended to other image-processing and ML workloads.

---

## 🖥️ Platform Details

The project runs on the **BeagleV-Fire** platform featuring the **Microchip PolarFire SoC (MPFS025T)**:
- 1× E51 RISC-V monitor core  
- 4× U54 application cores (RV64GC)  
- FPGA fabric integrated on-chip (approx. 23k logic elements, 68 DSP blocks)  
- AXI/APB bus interconnect between CPU and FPGA  
- Linux support via Buildroot and Yocto images  
- External interfaces: DDR4, PCIe, GPIO, UART, I2C, SPI, Ethernet  

The SoC allows **memory-mapped I/O** communication between Linux user-space software and custom FPGA logic — making it ideal for this co-design.

---

## 🔩 Project Structure

```
FPGA-Augmented-RISCV-LLVM-Backend/
│
├── llvm/      → Custom LLVM backend & Clang builtin for YUV→RGB pseudo-instruction  
│                (mapped to RISC-V pseudo-op, MMIO expansion)  
│                ↳ [View commits](https://github.com/meetzaa/llvm-wYUVRGB/tree/f146e5c13c6cde08d916f97605ea381ac8b1cca0)
│
├── fpga/      → FPGA accelerator (Verilog), connected to SoC APB/AXI bus  
│                Implements YUV→RGB pipeline, optional DMA mode, FSM control logic  
│                ↳ [View commits](https://git.beagleboard.org/Luciana.lm11/testing-gateware/-/tree/optimized_apb_conversion?ref_type=heads)
│
└── runtime-yuvrgb/  → User-space runtime for testing and invoking the builtin  
                      Reads `.yuv` images, triggers FPGA, and writes RGB output  
```

---

## 🧩 Software Architecture (LLVM Layer)

### 🧱 Compiler Modifications
- Added a **new Clang builtin**:  
  ```c
  __builtin_riscv_yuvrgb(yuv, rgb, H, W);
  ```
- Declared a corresponding LLVM intrinsic:
  ```llvm
  declare void @llvm.riscv.yuvrgb(i8* %yuv, i8* %rgb, i32 %H, i32 %W)
  ```
- Introduced a **pseudo-instruction** in the RISC-V backend, expanded in  
  `RISCVExpandPseudoInsts.cpp`, which performs:
  1. Mapping the physical address of the FPGA registers via `mmap()`
  2. Writing configuration parameters (image width, height, base addresses)
  3. Polling FPGA status bits for completion

### ⚙️ Runtime Layer
A simple Linux user-space runtime application:
- Allocates input/output image buffers in memory  
- Calls the builtin (`__builtin_riscv_yuvrgb`)  
- Saves the result to `bands_1080p.yuv_out.rgb`

---

## 🔌 Hardware Architecture (FPGA Layer)

### 🧠 YUV-RGB Conversion Core
- Implemented in Verilog using modular FSMs and AXI-Lite interface.  
- Supports both **non-DMA** and **DMA-driven** operation.  
- Converts four pixels in parallel.  
- Synchronization and completion flags exposed through memory-mapped registers.

### 📡 Communication Path
The pseudo-instruction emitted by LLVM triggers:
1. A **write sequence** to specific FPGA MMIO registers.  
2. The FPGA begins conversion autonomously.  
3. A **polling sequence** from CPU detects when the accelerator signals “done”.  
4. The runtime copies the resulting RGB frame back to system memory.

---

## 🧱 Platform Integration Diagram

```
 ┌───────────────────────────────┐
 │        User Application       │
 │   (calls builtin function)    │
 └───────────────┬───────────────┘
                 │
                 ▼
 ┌───────────────┴───────────────┐
 │         LLVM/Clang            │
 │  (intrinsic + pseudo-op)      │
 └───────────────┬───────────────┘
                 │
                 ▼
 ┌───────────────┴───────────────┐
 │     RISC-V Instruction Set    │
 │  + MMIO sequence generation   │
 └───────────────┬───────────────┘
                 │
                 ▼
 ┌───────────────┴───────────────┐
 │        FPGA Accelerator       │
 │ (Verilog YUV→RGB pipeline)    │
 └───────────────────────────────┘
```

---

## 🧩 Repositories & References

| Layer | Description | Repository / Commit |
|--------|--------------|---------------------|
| **LLVM Backend** | Custom RISC-V pseudo-instruction and builtin for YUV→RGB | [github.com/meetzaa/llvm-wYUVRGB/tree/f146e5c13c6cde08d916f97605ea381ac8b1cca0](https://github.com/meetzaa/llvm-wYUVRGB/tree/f146e5c13c6cde08d916f97605ea381ac8b1cca0) |
| **FPGA Accelerator** | Verilog design (optimized APB conversion, BeagleV-Fire) | [git.beagleboard.org/Luciana.lm11/testing-gateware/-/tree/optimized_apb_conversion](https://git.beagleboard.org/Luciana.lm11/testing-gateware/-/tree/optimized_apb_conversion?ref_type=heads) |

---

## 🧠 Keywords
`RISC-V` · `LLVM` · `Clang` · `FPGA` · `BeagleV-Fire` · `Compiler Backend` · `Hardware Acceleration` · `YUV-RGB Conversion` · `MOLEN Architecture` · `Co-Design`


