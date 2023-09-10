# docker-qemu-riscv
🛠️ Docker image with QEMU configured for RISC-V emulation. Perfectly suited for cross-debugging RISC-V.

![cross-debugging!!!](https://raw.githubusercontent.com/paulopacitti/docker-qemu-riscv/main/docs/screenshot.png)

### Problem
I created this image while I was developing cryptographic optimizations of know algorithms for RISC-V. I installed the `riscv-gnu-toolchain` in my MacBook Pro M1 using `brew install riscv-tools`. While I succeed installing the cross-compilation toolchain, I couldn't achieve cross-debugging with QEMU in Apple Silicon (building the source didn't work).

### Solution
 Docker image based on Ubuntu with QEMU configured for RISC-V emulation (both system and user space).

## Tools included
- `qemu-system-riscv32`: System emulation of RISC-V 32 bits;
- `qemu-system-riscv64`: System emulation of RISC-V 64 bits;
- `qemu-riscv32`: User space emulation of RISC-V 32 bits;
- `qemu-riscv64`: User space emulation of RISC-V 64 bits;

## Roadmap

1. [x] QEMU for RISC-V for cross debugging
2. [ ] `riscv-gnu-toolchain` for cross-compilation  

## Cross-debugging example
This image is very useful to create a _light linux VM_ for cross-debugging RISC-V, using `qemu` and `gdb` (the `gdb` required is the one installed together with `riscv-gnu-toolchain`). The `example/` folder in this repo contain a simple C code used this tutorial:

1. Cross-compile your C code for debugging:
```bash
riscv64-unknown-elf-gcc example/main.c -static -o -g main.elf
```
2. Run the `qemu-riscv` container sharing the directory the code is being built as a volume, port mapping the port `1234``, and, run bash thru it:
```bash
docker run --name=qemu-riscv -dit -v $(pwd):/root -p 1234:1234 paulopacitti/qemu-riscv:latest
docker exec -it qemu-riscv bash
```
3. Inside the container, run QEMU in debug mode with the `-g` flag in port `1234` passing your the ELF built:
```bash
qemu-riscv64 -g 1234 /root/main.elf
```
4. While QEMU is hanging, in your local machine run `gdb` from the `riscv-gnu-toolchain`:
```bash
riscv64-unknown-elf-gdb main.elf
```
5. Add breaks, enable features and then add the container as a remoter target:
```bash
(gdb) b main
(gdb) tui enable
(gdb) tui reg all
(gdb) target extended-remote :1234
```
6. Now you can debug with `gdb` as you normally would :)