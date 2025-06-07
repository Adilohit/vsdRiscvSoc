
# Phase 1 - Week 1: Hello RISC-V

---

## ✅ Task 1: Install & Sanity-Check the Toolchain

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>



I have downloaded riscv-toolchain-rv32imac-x86\_64-ubuntu.tar.gz. How exactly do I unpack it, add it to PATH, and confirm the gcc, objdump, and gdb binaries work?



</details>

<details>
<summary><strong>Steps Followed</strong></summary>

1. Extract the toolchain:

   ```bash
   tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
``

2. Navigate into the folder and locate `opt/riscv/bin`.

3. Add toolchain to PATH:

   ```bash
   export PATH=$HOME/Downloads/opt/riscv/bin:$PATH
   ```

4. Persist PATH in `~/.bashrc`:

   ```bash
   echo 'export PATH=$HOME/Downloads/opt/riscv/bin:$PATH' >> ~/.bashrc
   source ~/.bashrc
   ```

5. Verify binaries:

   ```bash
   riscv32-unknown-elf-gcc --version
   riscv32-unknown-elf-objdump --version
   riscv32-unknown-elf-gdb --version
   ```

> Note: GDB required installing `libpython3.10.so.1.0`.

</details>

![Toolchain Installation](https://github.com/user-attachments/assets/28a919ac-f21c-4227-9f98-95f79583b555)

---

## ✅ Task 2: Compile "Hello, RISC-V"

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
Show me a minimal C 'hello world' that cross-compiles for RV32IMC and the exact gcc flags to produce an ELF.
```

</details>

<details>
<summary><strong>Source Code: hello.c</strong></summary>

```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

</details>

<details>
<summary><strong>Compilation Command</strong></summary>

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
```

</details>

<details>
<summary><strong>Verify ELF</strong></summary>

```bash
file hello.elf
```

Expected output:

```
hello.elf: ELF 32-bit LSB executable, UCB RISC-V, ...
```

</details>

![ELF File Check](https://github.com/user-attachments/assets/3b155afa-0394-4fb7-ad1e-658ea24fcae6)

---

## ✅ Task 3: From C to Assembly

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
How do I generate the .s file and explain the prologue/epilogue of the main function?
```

</details>

<details>
<summary><strong>Generate Assembly</strong></summary>

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c -o hello.s
```

</details>

<details>
<summary><strong>Sample Assembly Snippet</strong></summary>

```asm
main:
    addi    sp, sp, -16      # Allocate stack space
    sw      ra, 12(sp)       # Save return address
    sw      s0, 8(sp)        # Save frame pointer
    addi    s0, sp, 16       # Set frame pointer

    # function body ...

    lw      ra, 12(sp)       # Restore return address
    lw      s0, 8(sp)        # Restore frame pointer
    addi    sp, sp, 16       # Deallocate stack space
    ret                     # Return from function
```

</details>

<details>
<summary><strong>Explanation</strong></summary>

* **Prologue:**

  * `addi sp, sp, -16` reserves 16 bytes on the stack for local variables and saved registers.
  * `sw ra, 12(sp)` saves the return address so it can be restored later.
  * `sw s0, 8(sp)` saves the frame pointer.
  * `addi s0, sp, 16` sets the frame pointer relative to the stack pointer.

* **Epilogue:**

  * `lw ra, 12(sp)` reloads the return address.
  * `lw s0, 8(sp)` reloads the frame pointer.
  * `addi sp, sp, 16` cleans up the stack.
  * `ret` returns control to the caller.

</details>

![Assembly Prologue/Epilogue](https://github.com/user-attachments/assets/a50b3cab-841f-4c98-8316-31c7048afa51)
![Assembly Full View](https://github.com/user-attachments/assets/f704598e-77d6-43e4-94be-bfa48e84b0e7)

---

## ✅ Task 4: Hex Dump & Disassembly

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
Show me how to turn my ELF into a raw hex and to disassemble it with objdump. What do each column mean?
```

</details>

<details>
<summary><strong>Commands</strong></summary>

```bash
# Disassemble ELF
riscv32-unknown-elf-objdump -d hello.elf > hello.dump

# Convert ELF to Intel HEX format
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

</details>

<details>
<summary><strong>Explanation of objdump output columns</strong></summary>

| Column       | Description                             |
| ------------ | --------------------------------------- |
| Address      | The memory address of the instruction   |
| Machine Code | Hexadecimal encoding of the instruction |
| Assembly     | The human-readable assembly instruction |

</details>

![Disassembly Output](https://github.com/user-attachments/assets/b3463459-fa61-4dca-9c83-0ef338dffc96)
![Intel HEX Output](https://github.com/user-attachments/assets/28f218ac-c4d5-4780-a881-36fb47f068a9)

---

## ✅ Task 5: ABI & Register Cheat-Sheet

<details>
<summary><strong>RV32 Integer Registers & ABI Names</strong></summary>

| Reg #   | ABI Name  | Role / Calling Convention          |
| ------- | --------- | ---------------------------------- |
| x0      | zero      | Hardwired zero (always 0)          |
| x1      | ra        | Return address (callee saved)      |
| x2      | sp        | Stack pointer                      |
| x3      | gp        | Global pointer                     |
| x4      | tp        | Thread pointer                     |
| x5-x7   | t0-t2     | Temporary (caller saved)           |
| x8-x9   | s0/fp, s1 | Saved registers (callee saved)     |
| x10-x17 | a0-a7     | Function arguments / return values |
| x18-x27 | s2-s11    | Saved registers (callee saved)     |
| x28-x31 | t3-t6     | Temporary (caller saved)           |

</details>

<details>
<summary><strong>Calling Convention Summary</strong></summary>

* **Arguments (a0–a7 / x10–x17):** Pass up to 8 function arguments and hold return values.
* **Temporary (t0–t6 / x5–x7, x28–x31):** Caller-saved, used for intermediate calculations.
* **Saved (s0–s11 / x8–x9, x18–x27):** Callee-saved, must be preserved by called functions.
* **Special:** `zero` (x0), `ra` (x1), `sp` (x2), `gp` (x3), `tp` (x4).

</details>

---

## ✅ Task 6: Stepping with GDB

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
How do I start riscv32-unknown-elf-gdb on my ELF, set a breakpoint at main, step, and inspect registers?
```

</details>

<details>
<summary><strong>Steps Followed</strong></summary>

1. Compile with debug symbols:

   ```bash
   riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -g -o hello.elf hello.c
   ```

2. Start GDB:

   ```bash
   riscv32-unknown-elf-gdb hello.elf
   ```

3. Connect to simulator (Spike/QEMU):

   ```gdb
   target sim
   ```

4. Set breakpoint at `main`:

   ```gdb
   break main
   ```

5. Run program:

   ```gdb
   run
   ```

6. Step through instructions:

   ```gdb
   step      # Step into
   next      # Step over
   ```

7. Inspect registers (e.g., a0):

   ```gdb
   info registers a0
   ```

8. Disassemble current function:

   ```gdb
   disassemble
   ```

</details>

![GDB Session](https://github.com/user-attachments/assets/e0c0c009-e472-40c0-add6-7981b7456df6)

---

## ✅ Task 7: Running Under an Emulator

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
Give me spike or QEMU commands to boot my bare-metal ELF and print to the ‘UART’ console.
```

</details>

<details>
<summary><strong>Steps Followed</strong></summary>

* Run on **Spike** with Proxy Kernel:

  ```bash
  spike --isa=rv32imc pk hello.elf
  ```

* Run on **QEMU** RISC-V 32-bit emulator with UART output:

  ```bash
  qemu-system-riscv32 -nographic -kernel hello.elf
  ```

* UART output appears directly in the terminal.

</details>

> **Notes:**
>
> * Spike requires `pk` as a proxy kernel for syscalls.
> * QEMU runs the ELF as a bare-metal kernel with no graphical output.

![Emulator UART Output](https://github.com/user-attachments/assets/be3ffbcd-7b6b-4adf-b47b-ff63d57db8aa)
---
Got it! Here's the updated markdown with **dropdown boxes** (using `<details>` tags) for **all code snippets and explanations except output images**, exactly as you requested:

---

## ✅ Task 8: Exploring GCC Optimisation

<details>
<summary>🛠️ Steps Followed (Click to expand)</summary>

```bash
# Compile without optimization (-O0)
riscv32-unknown-elf-gcc -S -O0 hello.c -o hello_O0.s

# Compile with optimization level 2 (-O2)
riscv32-unknown-elf-gcc -S -O2 hello.c -o hello_O2.s

# Compare both .s files
diff hello_O0.s hello_O2.s
```

</details>

<details>
<summary>🧾 Observations and Explanation (Click to expand)</summary>

| Optimization | Behavior                                                                                                                           |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `-O0`        | Generates verbose, unoptimized assembly. Includes redundant instructions and full stack frame setup.                               |
| `-O2`        | Optimizes for performance. Removes dead code, reduces stack usage, reuses registers, and may inline small functions like `printf`. |

</details>

---

### 🖼️ Output Images

![Assembly -O0 vs -O2 side by side](https://github.com/user-attachments/assets/398d24ea-c2c7-48c0-b566-9448ce2279fa)

---

## ✅ Task 9: Inline Assembly Basics

<details>
<summary>🛠️ Code Snippet (Click to expand)</summary>

```c
#include <stdint.h>

static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle" : "=r"(c));
    return c;
}

void main() {
    uint32_t cycles = rdcycle();

    while (1);  // Prevent program exit
}
```

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* `volatile`: Prevents the compiler from optimizing away the assembly block, ensuring it always executes.
* `"csrr %0, cycle"`: RISC-V CSR read instruction to read cycle counter CSR (0xC00).
* `: "=r"(c)`: Output operand; `=` means write-only, `r` means assign a general-purpose register to hold the result, storing it in variable `c`.

This inline assembly safely reads the hardware cycle counter directly into a C variable.

</details>

---

### 🖼️ Output Images

![Inline assembly explanation image](https://github.com/user-attachments/assets/30620f75-d5cd-4971-8453-9dc8697a7090)
![Cycle counter read example](https://github.com/user-attachments/assets/344ba642-b56d-4435-bbfe-f783d2ba0fd1)

---

## ✅ Task 10: Memory-Mapped I/O Demo

<details>
<summary>🛠️ Code Snippet (Click to expand)</summary>

```c
#include <stdint.h>

#define GPIO_ADDR 0x10012000
volatile uint32_t *gpio = (uint32_t *)GPIO_ADDR;

void main() {
    *gpio = 0x1;              // Set GPIO pin high
    for (volatile int i = 0; i < 100000; i++); // Simple delay loop
    *gpio = 0x0;              // Set GPIO pin low

    while (1);                // Infinite loop to hold state
}
```

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* `volatile`: Prevents compiler optimizations that might remove or reorder I/O memory accesses.
* `uint32_t *gpio`: Pointer to a 32-bit memory-mapped register at address `0x10012000`.
* Delay loop is volatile to ensure it isn’t optimized away, providing a time gap between toggles.
* Infinite loop keeps the program running to maintain output state.

</details>

---

### 🖼️ Output Images

![GPIO toggle code output](https://github.com/user-attachments/assets/a6e96d5d-c880-4263-91e0-0484784130c9)
![Memory-mapped I/O illustration](https://github.com/user-attachments/assets/67fa18c5-4455-4d33-bdec-6975a73c3ba9)

---
## ✅ Task 11: Linker Script 101

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
Provide a minimal linker script that places .text at 0x00000000 and .data at 0x10000000 for RV32IMC.
```

</details>
---

<details>
<summary>🛠️ Minimal Linker Script (Click to expand)</summary>

```ld
ENTRY(_start)

SECTIONS
{
    . = 0x00000000;

    .text : {
        *(.text*)
        *(.rodata*)
    }
S
    .data 0x10000000 : {
        *(.data*)
    }

    .bss (NOLOAD) : {
        *(.bss*)
        *(COMMON)
    }
}

````

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* `.text` section is placed at **0x00000000** — typically represents Flash memory where code is stored.
* `.data` section is placed at **0x10000000** — typically represents SRAM where variables are stored at runtime.
* The `*()` syntax selects all matching input sections (e.g., `*(.text*)` selects `.text`, `.text.main`, etc.).
* A linker script helps map program sections to specific physical memory regions.
* The separation of code and data is essential in embedded systems where code executes from Flash and data resides in RAM.

</details>

---

### 🖼️ Output Images

![image](https://github.com/user-attachments/assets/f18f7301-7a94-4fc3-a421-2ee1c85b9b60)
![image](https://github.com/user-attachments/assets/ff7e0808-e8e3-4e72-a12d-7799bf7ed711)
---
## ✅ Task 12: Start-up Code & crt0

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
What does crt0.S typically do in a bare-metal RISC-V program and where do I get one?
```

</details>
---
<details>
<summary>🛠️ Typical Start-up Code (crt0.S) (Click to expand)</summary>

```asm
.section .text
.globl _start

_start:
    # 1. Set stack pointer (example address; adjust as needed)
    la sp, _stack_top

    # 2. Copy .data from flash (_etext) to RAM (_sdata to _edata)
    la a0, _etext
    la a1, _sdata
    la a2, _edata

copy_data:
    beq a1, a2, clear_bss
    lw t0, 0(a0)
    sw t0, 0(a1)
    addi a0, a0, 4
    addi a1, a1, 4
    j copy_data

# 3. Zero out .bss from _sbss to _ebss
clear_bss:
    la a0, _sbss
    la a1, _ebss

clear_loop:
    beq a0, a1, call_main
    sw zero, 0(a0)
    addi a0, a0, 4
    j clear_loop

# 4. Call main()
call_main:
    call main

# 5. Infinite loop (if main returns)
hang:
    j hang
````

👉 A more complete version of `crt0.S` may include:

* Zero-initializing `.bss` section
* Copying initialized data from flash to SRAM
* Setting up frame pointers (if used)
* Eventually calling `main`

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* **Stack Setup:** The `la sp, _stack_top` sets up the stack pointer so that C functions can work properly.
* **.bss Initialization:** In real-world crt0.S, memory for uninitialized global variables (.bss) is usually set to zero.
* **Calling `main`:** The application starts by jumping to the `main` function.
* **Infinite Loop:** After `main` returns (in bare-metal), we typically loop indefinitely to prevent the CPU from executing garbage instructions.

### 📦 Where to Get crt0.S:

* **Newlib (RISC-V port)**: Common source for generic `crt0.S` files compatible with bare-metal toolchains.
* **Device-specific SDKs:** Vendors like SiFive and AndesTech provide their own customized `crt0.S` for booting on specific chips.
* **Minimal custom startup:** Can be written manually like above if targeting a simple emulator like QEMU or Spike.

</details>

---

### 🖼️ Output Images

![image](https://github.com/user-attachments/assets/9e4b55df-fa41-4736-8707-0bc10aa4c7d8)
![image](https://github.com/user-attachments/assets/8f340051-ee55-4f55-aa88-fd58490197a8)
![image](https://github.com/user-attachments/assets/78ba5d02-6e2f-4994-947e-da8ad4fddc9e)

---
## ✅ Task 13: Interrupt Primer

### 🧠 Prompt Asked to ChatGPT:
Demonstrate how to enable the machine-timer interrupt (MTIP) and write a simple handler in C/asm.
---
<details>
<summary>🛠️ Code Snippet (Click to expand)</summary>

```c
#include <stdint.h>
#define MTIMECMP_ADDR 0x2004000
#define MTIME_ADDR    0x200BFF8
#define MIE_MTIE      (1 << 7)
#define MSTATUS_MIE   (1 << 3)

volatile uint64_t* mtime    = (uint64_t*) MTIME_ADDR;
volatile uint64_t* mtimecmp = (uint64_t*) MTIMECMP_ADDR;

void timer_init() {
    uint64_t now = *mtime;
    *mtimecmp = now + 100000;        // Set timer interrupt for future
    asm volatile ("csrs mie, %0" :: "r"(MIE_MTIE));    // Enable machine timer interrupt
    asm volatile ("csrs mstatus, %0" :: "r"(MSTATUS_MIE)); // Global interrupt enable
}

// Timer Interrupt Handler (naked attribute)
void __attribute__((naked)) __attribute__((interrupt)) machine_timer_handler() {
    *mtimecmp = *mtime + 100000; // Reset the timer
}
````

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* **mtime / mtimecmp:** Registers for setting machine timer interrupt time.
* **MIE\_MTIE:** Bit mask to enable the timer interrupt in the Machine Interrupt Enable (MIE) CSR.
* **MSTATUS\_MIE:** Enables global interrupts via the Machine Status register.
* ****attribute**((interrupt)):** Ensures correct function prologue/epilogue for an interrupt handler.
* ****attribute**((naked)):** Tells compiler to skip standard function entry/exit; developer manages registers.

### 🧠 Notes:

* This example assumes you're running on a RISC-V target where `mtime` and `mtimecmp` are memory-mapped.
* On real hardware or full-system emulation (e.g., QEMU with CLINT), exact register addresses may differ.

</details>

---

### 🖼️ Output Images
---






