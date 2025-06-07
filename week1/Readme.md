
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

![image](https://github.com/user-attachments/assets/842b638d-346c-49e5-a649-dabbf084523a)
![image](https://github.com/user-attachments/assets/5bc63d97-95db-4daf-ac6a-dcc52dd68378)
![image](https://github.com/user-attachments/assets/7eea83d2-828f-430f-b729-29207121be58)

---
Here is the **reformatted README file** for **Task 14** and **Task 15** following your requested structure and format:

---
## ✅ Task 14: rv32imac vs rv32imc – What’s the “A”?

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```

Explain the ‘A’ (atomic) extension in rv32imac. What instructions are added and why are they useful?

````

</details>
---

<details>
<summary>✅ Answer (Click to expand)</summary>

The **'A' extension** in RISC-V adds **atomic instructions** for handling shared memory safely in concurrent environments.

### Key Instructions:
- `lr.w` – Load Reserved
- `sc.w` – Store Conditional
- `amoadd.w`, `amoswap.w`, `amoxor.w`, `amoand.w`, etc.

### Why it matters:
- Enables **lock-free data structures**
- Allows implementation of **spinlocks**
- Crucial for **multithreading** and **OS kernel** synchronization
- Ensures **atomic read-modify-write** operations on memory

</details>

<details>
<summary>📘 Example Usage (Click to expand)</summary>

```assembly
again:
    lr.w t0, (a0)         # load-reserved from address in a0
    addi t0, t0, 1        # increment
    sc.w t1, t0, (a0)     # store-cond back
    bnez t1, again        # retry if store failed
````

This pattern is used to **safely increment** a shared memory location using atomic primitives.

</details>

---

### 🖼️ Output Diagram (Optional)

*Insert a diagram showing LR/SC handshake and memory consistency if needed.*

---

## ✅ Task 15: Atomic Test Program

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```
Provide a two-thread mutex example (pseudo-threads in main) using lr/sc on RV32.
```

</details>
---

<details>
<summary>🛠️ Spin-lock Mutex Implementation in C with Inline Assembly (Click to expand)</summary>

```c
#include <stdio.h>
#include <stdatomic.h>

volatile int lock = 0;

void acquire_lock(volatile int *lock) {
    int tmp;
    asm volatile (
        "1: lr.w %0, %1\n"
        "   bnez %0, 1b\n"
        "   li %0, 1\n"
        "   sc.w %0, %0, %1\n"
        "   bnez %0, 1b\n"
        : "=&r" (tmp), "+A" (*lock)
        :
        : "memory"
    );
}

void release_lock(volatile int *lock) {
    *lock = 0;
}

void thread_func(const char *name, volatile int *lock) {
    printf("%s: Waiting to acquire lock...\n", name);
    acquire_lock(lock);
    printf("%s: Lock acquired! Critical section...\n", name);
    for (volatile int i = 0; i < 1000000; i++); // simulate work
    printf("%s: Releasing lock.\n", name);
    release_lock(lock);
}

int main() {
    thread_func("Thread 1", &lock);
    thread_func("Thread 2", &lock);
    return 0;
}
```

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* Implements a **spin-lock mutex** using RISC-V `lr.w` / `sc.w` instructions.
* `acquire_lock()`:

  * Performs `lr.w` to read the lock state.
  * If already locked, it retries.
  * Sets the lock with `sc.w`, which only succeeds if no one else changed the value.
* `release_lock()` resets the lock to `0`.
* Simulates two pseudo-threads in main, sequentially acquiring and releasing the lock.
* Demonstrates **bare-metal synchronization** without OS support.

</details>

---

### 🖼️ Sample Output

```
Thread 1: Waiting to acquire lock...
Thread 1: Lock acquired! Critical section...
Thread 1: Releasing lock.
Thread 2: Waiting to acquire lock...
Thread 2: Lock acquired! Critical section...
Thread 2: Releasing lock.
```

---

### 🖼️ Output Images

![image](https://github.com/user-attachments/assets/39a4bb3a-6c38-4289-af18-1e74ec1aef49)

---


## ✅ Task 16: Using Newlib printf Without an OS

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```

How do I retarget \_write so that printf sends bytes to my memory-mapped UART?

````

</details>
---

<details>
<summary>🛠️ Retargeting _write for UART Output (Click to expand)</summary>

```c
#include <unistd.h>

// Define your UART transmit register address (example)
#define UART_TX (*(volatile char*)0x10000000)

int _write(int fd, char *buf, int len) {
    (void)fd; // File descriptor unused

    for (int i = 0; i < len; i++) {
        // Wait for UART ready if needed (depends on hardware)
        UART_TX = buf[i];    // Send byte to UART TX register
    }
    return len;
}
````

* Compile your program with:

```bash
riscv32-unknown-elf-gcc -nostartfiles -o hello.elf hello.c syscalls.c
```

* This replaces the default `_write` syscall used by `printf` so it outputs bytes to your UART hardware.

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* The `printf` function in Newlib ultimately calls `_write` to output characters.
* By default, `_write` expects an OS to handle system calls, but in bare-metal environments, you must provide your own.
* Implementing `_write` to loop over the buffer and send each character to the **memory-mapped UART transmit register** enables `printf` to work without an OS.
* Using `-nostartfiles` disables standard startup files, allowing you to provide custom system call implementations in `syscalls.c`.
* This method is common in embedded bare-metal programming to get console output over UART.

</details>

---

### 🖼️ Example Output

```
Hello, RISC-V!
```

(Output appears on UART serial terminal connected to your hardware/emulator.)

---

### 🖼️ (Optional) Diagram or UART Pinout Illustration

![image](https://github.com/user-attachments/assets/eabac2fc-cf3e-481b-8df0-d25aab220b39)

![image](https://github.com/user-attachments/assets/a5afed11-f599-4c25-9fd3-29c8e0f2f0b6)
![image](https://github.com/user-attachments/assets/af3340dc-261e-4aab-927d-9d46185f0112)
![image](https://github.com/user-attachments/assets/a5f1f8eb-d575-41cf-8908-5e95da86fa07)

---

## ✅ Task 17: Endianness & Struct Packing

<details>
<summary><strong>Prompt Asked to ChatGPT</strong></summary>

```

Is RV32 little-endian by default? Show me how to verify byte ordering with a union trick in C.

````

</details>
---

<details>
<summary>🛠️ Code to Verify Endianness Using a Union (Click to expand)</summary>

```c
#include <stdio.h>
#include <stdint.h>

int main() {
    union {
        uint32_t value;
        uint8_t bytes[4];
    } test;

    test.value = 0x01020304;

    printf("Bytes: %02x %02x %02x %02x\n",
           test.bytes[0], test.bytes[1], test.bytes[2], test.bytes[3]);

    if (test.bytes[0] == 0x04) {
        printf("System is Little Endian.\n");
    } else if (test.bytes[0] == 0x01) {
        printf("System is Big Endian.\n");
    } else {
        printf("Unknown Endianness.\n");
    }

    return 0;
}
````

</details>

<details>
<summary>🧾 Explanation (Click to expand)</summary>

* RV32 **is little-endian by default**, meaning the least significant byte is stored at the lowest memory address.
* The union overlays a 32-bit integer with a 4-byte array.
* Assigning `0x01020304` to the integer and printing the individual bytes shows the byte order.
* On little-endian systems, the bytes print as `04 03 02 01`.
* On big-endian systems, bytes print as `01 02 03 04`.
* This technique is a simple, portable way to verify endianness at runtime.

</details>

---

### 🖼️ Sample Output

```
Bytes: 04 03 02 01
System is Little Endian.
```

---

# RISC-V Cross-Compilation & Related Tasks - Summary WEEK -1

This repository contains solutions and explanations for various RISC-V related programming and toolchain tasks, focusing on cross-compilation, assembly, disassembly, atomic operations, system calls, and architecture-specific topics.

---

## ✅ Summary of Completed Tasks

1. **Minimal RISC-V “Hello, World” C Program**
   - Cross-compiled for RV32IMC using:
     ```bash
     riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
     ```
   - Verified ELF file type with `file hello.elf`.

2. **Generating Assembly (.s) from C**
   - Used `-S` flag to generate `hello.s`.
   - Explained function prologue/epilogue instructions (`addi sp, sp, -16`, `sw ra, 12(sp)`, etc.).

3. **Hex Dump & Disassembly**
   - Disassembled ELF using:
     ```bash
     riscv32-unknown-elf-objdump -d hello.elf > hello.dump
     ```
   - Created raw hex file:
     ```bash
     riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
     ```
   - Explained disassembly columns: address, opcode, mnemonic, operands.

4. **Atomic Extension ‘A’ in RV32IMAC**
   - Explained atomic instructions: `lr.w`, `sc.w`, `amoadd.w`, etc.
   - Highlighted their use in lock-free data structures and OS kernels.

5. **Atomic Test Program (Spin-lock Mutex)**
   - Provided C code with inline RISC-V assembly using `lr.w` and `sc.w` to implement a spin-lock mutex.
   - Demonstrated pseudo-thread locking in main function.

6. **Using Newlib printf Without an OS**
   - Retargeted `_write` syscall to send bytes to a memory-mapped UART register.
   - Enabled `printf` output on bare-metal hardware by linking with custom `syscalls.c`.

7. **Endianness & Struct Packing**
   - Verified that RV32 is little-endian by default using a union trick in C.
   - Printed byte order of a 32-bit value to determine system endianness.

---

## ⚡ Key Takeaways

- **Cross-compiling** for RV32 requires proper GCC multilib support and correct flags (`-march=rv32imc`, `-mabi=ilp32`).
- Generating assembly and understanding function prologue/epilogue is essential for low-level debugging.
- Disassembling ELF files with `objdump` helps analyze compiled instructions and memory layout.
- The atomic extension provides crucial synchronization primitives for concurrency on RISC-V.
- Retargeting standard library syscalls enables `printf` and other I/O without a full OS.
- Endianness affects how multi-byte data is stored and interpreted, impacting low-level data handling.

---

This collection of tasks provides a foundational workflow for embedded development on RISC-V processors, covering everything from toolchain setup to hardware-aware programming.

---




