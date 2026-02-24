# Building pico-dirtyJtag on Ubuntu

These instructions cover a complete build from scratch on Ubuntu without requiring sudo or modifying system packages.

## Prerequisites

The following tools must be available on the system:

- `git`
- `cmake` (3.13 or later)
- `ninja-build`
- `python3`

Verify they are present:

```bash
git --version
cmake --version
ninja --version
python3 --version
```

## 1. Create a Working Directory

```bash
mkdir -p ~/buildDirtyJtag
```

## 2. Download the ARM GNU Toolchain

Download the prebuilt ARM cross-compiler from the ARM developer website (version 14.2 Rel1, ~143 MB):

```bash
cd ~/buildDirtyJtag
wget -O arm-toolchain.tar.xz \
  "https://developer.arm.com/-/media/Files/downloads/gnu/14.2.rel1/binrel/arm-gnu-toolchain-14.2.rel1-x86_64-arm-none-eabi.tar.xz"
```

Extract it:

```bash
tar xf arm-toolchain.tar.xz
```

Verify the compiler works:

```bash
~/buildDirtyJtag/arm-gnu-toolchain-14.2.rel1-x86_64-arm-none-eabi/bin/arm-none-eabi-gcc --version
```

Expected output:
```
arm-none-eabi-gcc (Arm GNU Toolchain 14.2.Rel1 (Build arm-14.52)) 14.2.1 20241119
```

## 3. Clone the Pico SDK

Clone version 2.2.0 with the required submodules (TinyUSB, BTstack, cyw43-driver):

```bash
cd ~/buildDirtyJtag
git clone --depth=1 --branch 2.2.0 https://github.com/raspberrypi/pico-sdk.git pico-sdk
cd pico-sdk
git submodule update --init --depth=1 lib/tinyusb lib/cyw43-driver lib/btstack
```

## 4. Clone pico-dirtyJtag

```bash
cd ~/buildDirtyJtag
git clone https://github.com/phdussud/pico-dirtyJtag.git pico-dirtyJtag
```

> If you already have the source, skip this step.

## 5. Build

```bash
TOOLCHAIN=~/buildDirtyJtag/arm-gnu-toolchain-14.2.rel1-x86_64-arm-none-eabi/bin
PICO_SDK=~/buildDirtyJtag/pico-sdk

mkdir -p ~/buildDirtyJtag/pico-dirtyJtag/build
cd ~/buildDirtyJtag/pico-dirtyJtag/build

cmake .. \
  -DPICO_SDK_PATH=$PICO_SDK \
  -DCMAKE_C_COMPILER=$TOOLCHAIN/arm-none-eabi-gcc \
  -DCMAKE_CXX_COMPILER=$TOOLCHAIN/arm-none-eabi-g++ \
  -DCMAKE_ASM_COMPILER=$TOOLCHAIN/arm-none-eabi-gcc \
  -DPICO_TOOLCHAIN_PATH=$TOOLCHAIN \
  -G Ninja

ninja -j$(nproc)
```

### CMake notes

- CMake will automatically download and build `pioasm` and `picotool` from source during the first configure. This is expected and requires an internet connection on the first run.
- A warning about `picotool` not being installed is normal — it builds from source automatically.

## 6. Expected Build Output

A successful build produces the following files in `~/buildDirtyJtag/pico-dirtyJtag/build/`:

| File               | Size  | Description                              |
|--------------------|-------|------------------------------------------|
| `dirtyJtag.uf2`   | ~60K  | Flash image — drag-and-drop onto Pico    |
| `dirtyJtag.bin`   | ~30K  | Raw binary                               |
| `dirtyJtag.hex`   | ~84K  | Intel HEX format                         |
| `dirtyJtag.elf`   | ~782K | ELF with debug symbols                   |

The final ninja output should end with:

```
[115/115] Linking CXX executable dirtyJtag.elf
```

## 7. Flashing to the Pico

1. Hold the **BOOTSEL** button on the Pico while connecting it via USB.
2. It will appear as a USB mass storage device (e.g. `RPI-RP2`).
3. Copy `dirtyJtag.uf2` to the drive:

```bash
cp ~/buildDirtyJtag/pico-dirtyJtag/build/dirtyJtag.uf2 /media/$USER/RPI-RP2/
```

The Pico will reboot automatically and enumerate as a DirtyJTAG USB device:

```
Bus 003 Device 112: ID 1209:c0ca Generic Jean THOMAS DirtyJTAG
```

## Rebuilding After Changes

Subsequent builds do not need cmake to be re-run unless `CMakeLists.txt` changes:

```bash
cd ~/buildDirtyJtag/pico-dirtyJtag/build
ninja -j$(nproc)
```
