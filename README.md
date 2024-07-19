# AutoPatch Artifact

Welcome to the artifact for the CCS 2024 submission of our paper, titled "AutoPatch: Automated Generation of Hotpatches for Real-Time Embedded Devices". 

This artifact includes the implementation of *AutoPatch* and the source code of its LLVM passes. 
To evaluate *AutoPatch* and obtain results similar to those presented in the paper, access to one of the following **three** boards is required: 

**nRF52840, STM32-F446RE, or ESP-WROOM32**

**It is important to note that due to university policies, external access to the university's internal internet is not allowed. Therefore, we cannot provide the AE committee with access to the boards. However, we have included three photos of the board setup and overheads for AutoPatch in the screencast folder.**

As explained in the paper, the LLVM passes (i.e., **Instrumentation and Analysis components**) do not necessitate a board and can be executed on general-purpose computers. 

Although **AutoPatch** has been evaluated on different RTOSes and boards, in this artifact we explain its evaluation on the **nrf52840** board and **Zephyr OS**.

#### The structure of the artifact is as follows:
```
AutoPatchMain    <--- Main Project

Passes
|
└─── AutoPatchFirstPass    <--- Instrumentation Pass
└─── AutoPatchSecondPass   <--- Analysis Pass

LLVM Modifications
|
└─── BasicBlock.cpp
│   │   BasicBlock.h
│   │   ...

Scripts

Testcases
|
└─── CVE Dataset
│   │   Results

PCEvaluation
```

## How To Run AutoPatch on a Board

### Configure the working directory

- Install the following packages (any modern version):
  - cmake
  - ninja
  - J-Link
- Run the following commands:  
```shell
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install build-essential curl libcap-dev git libncurses5-dev python2-minimal python3-pip unzip libtcmalloc-minimal4 libgoogle-perftools-dev libsqlite3-dev doxygen ninja-build gcc g++ clang-9 llvm-9 llvm-9-dev llvm-9-tools autoconf  
```
- Download and install CMake 3.26.0 https://cmake.org/download/

- Install Ninja build `sudo apt-get install ninja-build`
  
### Download Software Applications

Download the following software applications:

- [VSCode](https://code.visualstudio.com/)
- [Nordic Semiconductor SDK (full suite)](https://www.nordicsemi.com/Products/Development-software/nrf5-sdk)
- [ZephyrOS](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)

### Setting up and Configuring LLVM

Please check [LLVM Passes](Passes) folder.

### VSCode Extensions

Download the following VSCode extensions for ZephyrOS and Nordic Semiconductor (found in the VSCode extension marketplace):

- DeviceTree for the Zephyr Project
- Kconfig for the Zephyr Project
- nRF Connect for VSCode
- nRF Terminal

Note that these extensions are required to run on the *nRF52840* board.

### Connecting Nordic Semiconductor ARM Board

Connect the Nordic Semiconductor ARM board (nRF52840-DK) to the desktop computer. If the board does not automatically show up, make sure you have installed J-Link.

Follow the instructions [here](https://www.zephyrproject.org/getting-started-with-nrf-connect-for-visual-studio-code/) to create a project, build, and flash example code onto the board.

### Run LLVM Passes Using Scripts

To streamline development work, we have written [scripts](Scripts) for executing the instrumentation and analysis components of AutoPatch.  

Before running the scripts, make sure to:
- Change the file path constants within the shell scripts to match the local file structure.
- Set **LLVM_BUILD_DIR**.
- Choose a C program from the example CVEs in [Testcases](Testcases).

Run `instrument.sh` by passing the chosen C file to it, and then answer the questions in Terminal (e.g., line numbers of official patch). Note that if you use our testcases, you should answer **true** to the **Is it patched?**. Then, It will generate a `.bc` file, which contains the instrumented llvm IR (i.e., trampolines in different locations of function). 

For generating the hotpatch (a functional equivalent patch that can be generated by one of the inserted trampolines), use the generated `.bc` file from prior step (`patchedFunc-inst.bc`) to run `analysis.sh` to get the `.o` file (see [Scripts](Scripts)), which represents the executable hotpatch, capable on running on the targeted hardware.


### Adding AutoPatch Worksapce

When you install Zephyr OS on your system according to [this link](https://docs.zephyrproject.org/latest/develop/getting_started/index.html), a new directory will be created (e.g., \~/zephyrproject). You need to copy the [AutoPatchMain](AutoPatchMain) folder to this directory: **~/zephyrproject/zephyr/samples/**. 



### Flash and Execute

To run the generated hotpatch on the board running Zephyr OS you need to do the following steps:

- Open AutoPatchMain project with VSCode.
- Copy hotpatch object file (located in \~/AutoPatchWorkspace/Testcases/Results/Hotpatch_CVE_{CVEid}.o) (see [Scripts](Scripts)) to AutoPatchMain source code folder (e.g., src/LocalPatches/) (see [AutoPatchMain Project](https://github.com/Moh3nsalehi/AutoPatchCode/tree/main/AutoPatchMain)).
- Change CMakeLists.txt (e.g., \~/zephyrproject/zephyr/samples/AutoPatchMain/) to add the hotpatch information.
  - add_library(myac_obj OBJECT `hotpatch directory in AutoPatchMain folder`)
  - target_sources(app PRIVATE `hotpatch directory in AutoPatchMain folder`)
-  Change `main` function in `main.c` in the `src` folder. For the hotpatches we generated, you only need to call one of the 36 `test_c()` functions.
-  Use **nRF Connect** extention in VSCode to flash and execute AutoPatchMain project to the board.
    - Forward the result to a serial port to see printed messages in the terminal.
 

## How To Run AutoPatch on PC

If you do not have access to the mentioned boards, you will need to perform the previous steps a bit differently. In the following, we will explain how the hotpatch execution process differs from the previous part. For this part, we used **Ubuntu 22.04**. 

*Note that AutoPatch is implemented to run on board and this part of the implementations is completely simplified and written for one CVE (CVE-2020-10021) to show that the generated hotpatches are executable.*

The process is almost the same as before and only the two final steps (i.e., *Adding AutoPatch Workspace* and *Flash and Execute*) are different. 

To run the hotpatch generated by AutoPatch on the desktop computer, we provided a folder named [PCEvaluation](PCEvaluation), which contains **4 main files**:

-  **test.c:** This is a simplified version of the main program from [AutoPatchMain](AutoPatchMain/src/main.c), designed to run on a PC.

-  **Hotpatch_CVE_10021.bc:** This hotpatch is generated by [analysis.sh](Scripts) (AutoPatchSecond Pass).


-  **Script.sh:** To link and generate an executable file, we have provided a script to simplify this process. Finally, the script executes the generated program.

  
-  **Script_withoutlink.sh:** To demonstrate that this process works correctly, the script generates an executable program without linking the hotpatch (`Hotpatch_CVE_10021.bc`) to the main program (`test.c`). When it tries to run the executable file, it throws an error because the main program lacks the function that the file expects.

  


