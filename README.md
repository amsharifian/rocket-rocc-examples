Collection of example libraries and test programs for the existing Rocket Custom Coprocessor (RoCC) accelerators for [Rocket Chip](https://github.com/ucb-bar/rocket-chip).


## Getting Started on a Local Ubuntu Machine

1. Install Ubuntu packages

    sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev libusb-1.0-0-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev device-tree-compiler pkg-config libexpat-dev

2. Clone and build `riscv-gnu-toolchain`

    git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
    cd riscv-gnu-toolchain
    git submodule update --init --recursive
    ./configure --prefix=/opt/riscv
    sudo make -j8
    export PATH=/opt/riscv/bin:$PATH
    export LD_LIBRARY_PATH=/opt/riscv/lib:$LD_LIBRARY_PATH

To build the Newlib cross-compiler, pick an install path. If you choose, say, /opt/riscv, then add /opt/riscv/bin to your PATH now. Then, simply run the following command:

3. Clone and build `riscv-tools`

    git clone --recursive https://github.com/riscv/riscv-tools.git
    cd riscv-tools
    export RISCV=/path/to/install/riscv/toolchain
    ./build.sh

4. Clone Rocket-chip repo

    git clone --recursive https://github.com/chipsalliance/rocket-chip.git
    cd rocket-chip
    git submodule update --init
    export RISCV=/path/to/riscv/toolchain/installation
    export MAKEFLAGS="$MAKEFLAGS -jN" # Assuming you have N cores on your host system
    ./build.sh

5. Build emulation binary

    cd rocket-chip
    cd emulator
    make CONFIG=RoccExampleConfig

At this point you should have rocket-chip emulator: `emulator-freechips.rocketchip.system-RoccExampleConfig`


6. To run the bareMetal example:

    git clone --recursive git@github.com:amsharifian/riscv-tests.git
    cd riscv-tests
    git submodule update --init --recursive
    autoconf
    ./configure --prefix=$RISCV/target
    make
    make install

7. To run Baremetal test
    cd rocket-chip/emulator
    ./emulator-freechips.rocketchip.system-RoccExampleConfig ~/git/riscv-tests/benchmarks/rocc.riscv

You should expect the following output

```
This emulator compiled with JTAG Remote Bitbang client. To enable, use +jtag_rbb_enable=1.
Listening on port 46723
[INFO] Write R[1] = 0xdead
[INFO] Read R[1]
[INFO]   Received 0xdead (expected 0xdead)
[INFO] Accum R[1] with 0xffffffffffffe042
[INFO] Read R[1]
[INFO]   Received 0xbeef (expected 0xbeef)
[INFO] Load 0xbad (virt: 0x0x80021a30, phys: 0x0x9140ed79e0a30) via L1 virtual address
[INFO] Read R[1]
[INFO]   Received 0xbad (expected 0xbad)
```

## Usage

Install the RISC-V toolchain and make sure that it's on your path.
You need to pass the path to `riscv-tools` in as an argument to configure `--with-riscvtools=$PATH_TO_RISCV_TOOLS`.

```
autoconf
mkdir build && cd build
../configure --with-riscvtools=$PATH_TO_RISCV_TOOLS
make
```

This creates two classes of tests:
* `bareMetal` -- Bare metal tests like [RISCV Tests](https://github.com/riscv/riscv-tests)
* `pk` -- Tests that use the [Proxy Kernel](https://github.com/riscv/riscv-pk)

Bare Metal tests can be run directly on the emulator (for instructions on how to build this see the following section), e.g.:

```
emulator-freechips.rocketchip.system-RoccExampleConfig bareMetal/examples-bareMetal-p-accumulator
```

Proxy Kernel tests are ELFs and need the Proxy Kernel (or Linux).
You must first patch the Proxy Kernel so that the `XS` bits allow access to the "extension" (the RoCC).
You can change this manually or use the provided patch ([`patches/riscv-pk.patch`](patches/riscv-pk.patch)):
```
cd $RISCV_PK_DIR
git apply $THIS_REPO_DIR/patches/riscv-pk.patch

mkdir build
cd build
../configure --prefix=$RISCV/riscv64-unknown-elf --host=riscv64-unknown-elf
make
make install
```

Following that, you can run Proxy Kernel tests, e.g., :

```
emulator-freechips.rocketchip.system-RoccExampleConfig pk pk/examples-pk-accumulator
```

## Building a Rocket Chip Emulator

Build a rocket-chip emulator with the RoCC examples baked in and run the provided test program:
```
cd $ROCKETCHIP_DIR/emulator
make CONFIG=RoccExampleConfig
```

### Expected Run Time

The included test should run for ~5 million cycles over a wall clock time of ~5 minutes.

```
> time $ROCKETCHIP_DIR/emulator-freechips.rocketchip.system-RoccExampleConfig -c pk pk/examples-pk-accumulator
[INFO] Write R[1] = 0xdead
[INFO] Read R[1]
[INFO]   Received 0xdead (expected 0xdead)
[INFO] Accum R[1] with 0xffffffffffffe042
[INFO] Read R[1]
[INFO]   Received 0xbeef (expected 0xbeef)
[INFO] Load 0xbad (virt: 0x0xfee9ac0, phys: 0x0x8ffffac0) via L1 virtual address
[INFO] Read R[1]
[INFO]   Received 0xbad (expected 0xbad)
Completed after 3278954 cycles

real	5m2.738s
user	5m0.262s
sys	0m1.179s
```
