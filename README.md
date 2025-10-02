# rsoc-week2 
 [part 1](#System-on-Chip-(SoC)-Design-Fundamentals)<br>
 [Part 2](part2.md)
# System-on-Chip (SoC) Design Fundamentals

## 1. What is a System-on-Chip (SoC)?

Think of an SoC as the ultimate integration project. Instead of having a bunch of separate chips (CPU, memory controller, I/O) scattered across a motherboard, we smash them all onto a **single piece of silicon**. This creates one high-power, efficient Integrated Circuit (IC).  

**Why do we do this?** For the famous **PPA** trifecta :  

- **Power:** Lower consumption, crucial for phones and IoT.  
  
- **Performance:** Faster interaction between blocks.
  
- **Area:** Smaller physical size (miniaturization).  
  

This high-level integration, however, means if one part has a bug, the whole chip might fail. This is why verification is such a big deal.

![SoC Architecture  MATLAB & Simulink](https://in.mathworks.com/discovery/soc-architecture/_jcr_content/mainParsys/image.adapt.full.medium.jpg/1758716841937.jpg)

## 2. The Internal Architecture: What's Inside an SoC?

Every functional system needs standard components, and an SoC packages them as Intellectual Property (IP) blocks.

### 2.1. The Brains and the Data (Processing & Memory)

The heart is the **Processing Unit**—this could be one CPU core, or maybe a mix of CPUs, GPUs, and other accelerators, depending on the job. This is often called a heterogeneous architecture.  

To feed the processor, we rely on a fast memory hierarchy:

- **Registers (Level 1):** The fastest on-chip storage, right next to the CPU.  
  
- **Cache (L2/L3):** Quick access memory to store frequently used data.  
  

While high-density memory (like LPDDR) is sometimes stacked externally, the controllers that manage it are integrated right onto the SoC.  

### 2.2. Talking to Each Other (Peripherals and Interconnect)

- **Peripherals:** These blocks handle external stuff—USB, Wi-Fi, video output, etc.  
  
- **Interconnect Fabric:** This is the communication highway inside the chip. It lets the CPU talk to memory and peripherals. For complex, high-speed designs, people use protocols like AMBA AXI, but these are complicated to manage. For simpler, portable projects like ours, open-source buses like  
  
  **WISHBONE (WB)** are often preferred because they are easier to implement and verify (fewer arbitration issues to worry about).  
  

## 3. Why Functional Modelling Comes First

The VLSI design cycle is super structured, moving from high-level abstract ideas to physical silicon :  

- **Phase 1:** Specification & Architecture (Define goals, blocks).  
  
- **Phase 2: Functional/Logic Design (Our Focus):** Define *what* the logic should do.  
  
- **Phase 3:** RTL Design: Code *how* data flows, synchronized cycle-by-cycle (using Verilog).  
  
- **Phase 4:** Physical Design: Placement, routing, timing closure, and finally, Tapeout (sending files to the factory).
  

### 3.1. Verification is Non-Negotiable

Functional Verification happens in Phase 2. We use simulation to confirm the design logic is correct.  

**Why is this so important?** Finding a logical bug here is cheap and fast. Finding a bug *after* manufacturing (a "silicon respin") is disastrous—it costs millions and adds months to the schedule. Functional modelling ensures we nail the intended behavior before we worry about the physical realities of timing and layout.  

## 4. BabySoC: Our Simple RISC-V Testbench

We use the **BabySoC** as a minimal working system to teach the core concepts without overwhelming complexity.  

- **The CPU:** We are using the RVMYTH core, which is based on **RISC-V**.  
  
- **Why RISC-V?** It's a Reduced Instruction Set Computing (RISC) architecture, emphasizing simplicity and efficiency. It’s also an open standard, meaning no proprietary licenses, which is perfect for education and transparency.  
  

BabySoC forces us to handle real interfaces, like the **Analog PLL (Phase-Locked Loop)**, which is necessary to generate the stable, reliable clock (CLK) signal that drives the digital RVMYTH CPU. The 10 bit DAC allows real world communication with the outside world via analog signals. this can be used to interact with oscilloscope, tv, audio interfaces and more. Our goal in simulation is to watch the CPU execute instructions from the Instruction Memory (IMEM) and confirm that key outputs, like the final value in register  `r17`, are correct.  

![GitHub  ireneann713/VSDBABYSoCICC2](https://user-images.githubusercontent.com/55539862/189318328-db0fbdfe-fd84-432b-9262-a8171f91658c.png)

## 5. The Functional Modelling Toolchain (Icarus Verilog + GTKWave)

To do practical functional modeling, we use two powerful open-source tools.

### 5.1. Running the Simulation: Icarus Verilog

This toolchain involves dynamic simulation:

1. **Testbench:** We set up a testbench (written in Verilog) to generate the necessary clock and reset inputs (stimulus) for our chip (the DUT).  
  
2. **Compilation:** We use `iverilog` to compile the design and the testbench into an executable `.vvp` file.  
  
3. **Execution:** We run the compiled file with `vvp`. Crucially, we include Verilog system tasks (`$dumpfile`, `$dumpvars`) in the testbench to capture all the signal activity over time into a **Value Change Dump (VCD) file**.  
  

Since we are only simulating logic (behavior) and not physical delays, this process is very fast.  

### 5.2. Analyzing the Results: GTKWave

The VCD file is the proof. We load it into **GTKWave** to visually inspect the waveforms.  

This is the debugging part: you can select signals (like the clock, instruction address, or `r17` register value) and trace their state changes cycle-by-cycle. This visual confirmation is how we functionally verify that the BabySoC is doing exactly what it was designed to do.