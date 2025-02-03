# Building CMSIS-DSP for STM32Cube IDE with ```arm-none-eabi-gcc```
This repository shows how to compile latest CMSIS-DSP for STM32 with arm-none-eabi-gcc.

## Creating folder
Create a folder where both CMSIS-6 and CMSIS-DSP will be stored.

## Clone CMSIS-6 and CMSIS-DSP into the folder
For the compilation process, you need the latest repositories from Git. You can either use the commands below or download the repositories as compressed archives and extract them into the folder you created:
git clone https://github.com/ARM-software/CMSIS_6
git clone https://github.com/ARM-software/CMSIS-DSP

## Create ```CMakeLists.txt``` File

Create a file named ```CMakeLists.txt``` with the following content:

```
cmake_minimum_required(VERSION 3.16)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/toolchain.cmake)

set(HOME $ENV{HOME})
set(CMSISDSP ${CMAKE_CURRENT_SOURCE_DIR}/CMSIS-DSP)
set(CMSISCORE ${CMAKE_CURRENT_SOURCE_DIR}/CMSIS_6/CMSIS/Core)

add_compile_options(
    -mcpu=cortex-m4
    -std=gnu11
    -ffunction-sections
    -fdata-sections
    #--specs=nano.specs
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
    -mthumb
    
    -Wsign-compare
    -Wdouble-promotion
    -Ofast -ffast-math
    -DNDEBUG
    -Wall -Wextra  -Werror
    -fshort-enums 
    #-fshort-wchar
    )
add_link_options(
    -mfloat-abi=hard    
    -mcpu=cortex-m4
    -Wl,--gc-sections
    -static
    -mfpu=fpv4-sp-d16
    -mthumb
    )
## Define the project
project (cmsis-dsp)

add_subdirectory(${CMSISDSP}/Source bin_dsp)
```

Also, we need to create ```toolchain.cmake``` file that we mentioned at ```set(CMAKE_TOOLCHAIN_FILE ${CMAKE_CURRENT_SOURCE_DIR}/toolchain.cmake)```. "toolchain.cmake" file can be the following:
```
# configuring the compiler and target spesific parameters
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)
set(CMAKE_CROSSCOMPILING 1)
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_SIZE_UTIL arm-none-eabi-size)
set(CMAKE_C_GDB arm-none-eabi-gdb-py)
set(CMAKE_AR arm-none-eabi-gcc-ar)
set(CMAKE_RANLIB arm-none-eabi-gcc-ranlib)

set(CMAKE_EXE_LINKER_FLAGS "--specs=nosys.specs" CACHE INTERNAL "")

# This should be safe to set for a bare-metal cross-compiler
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

## STM32Cube IDE Project Setup

### Launch STM32Cube IDE
Open the IDE and select the workspace where you want to create your project.

### Create a New Project
Choose the option for a new STM32 CMake project and select “Project with existing CMake sources.”

### Configure the Project Settings
- **Project Name:** Enter your desired project name.
- **Source Directory:** Set the source directory to the folder you created in first step.

### Select the Toolchain
Choose "MCU ARM GCC" as the toolchain and select your target MCU.

### Finish
Click the **Finish** button to create the project.

After creating the project, build it without any further configuration. The compilation process will generate a file named ```libCMSISDSP.a```. The console output should resemble the following:
```
...
[ 99%] Building C object bin_dsp/CMakeFiles/CMSISDSP.dir/WindowFunctions/arm_hft144d_f32.c.obj
[ 99%] Building C object bin_dsp/CMakeFiles/CMSISDSP.dir/WindowFunctions/arm_hft169d_f32.c.obj
[ 99%] Building C object bin_dsp/CMakeFiles/CMSISDSP.dir/WindowFunctions/arm_hft196d_f32.c.obj
[ 99%] Building C object bin_dsp/CMakeFiles/CMSISDSP.dir/WindowFunctions/arm_hft223d_f32.c.obj
[ 99%] Building C object bin_dsp/CMakeFiles/CMSISDSP.dir/WindowFunctions/arm_hft248d_f32.c.obj
[100%] Linking C static library libCMSISDSP.a
[100%] Built target CMSISDSP

21:18:15 Build Finished. 0 errors, 0 warnings. (took 1m:49s.617ms)
```

## Using the CMSIS-DSP Library
To integrate the CMSIS-DSP library into your project, follow these steps:

1. **Open Project Properties**
   - Right-click your project in the Project Explorer and select **Properties**.
   - Navigate to **C/C++ Build Settings**.

2. **Configure the Linker Settings**
   - Go to **MCU/MPU GCC Linker** → **Libraries**.
   - Under **Libraries (-l)**, add the library name ```CMSISDSP```.
   - Under **Library Search Path (-L)**, add the full path to the directory containing the `libCMSISDSP.a` file. For example:```/path/to/your/libdirectory```.

3. **Configure the Compiler Include Paths**
   - In the **C/C++ Build Settings**, navigate to **MCU GCC Compiler** → **Include Paths**.
   - Add the full path to the directory that contains the CMSIS-DSP header files. This is typically:```/path/to/CMSIS-DSP/CMSIS-DSP/Include/```
   - Make sure this path is correctly set so that the compiler can locate all required header files.

4. **Apply and Save Your Settings**
   - Click **Apply** and then **OK** to save your configuration changes.

5. **Build Your Project**
   - Rebuild your project to ensure that the new settings are applied and that the CMSIS-DSP library is correctly linked.
   - Upon a successful build, the integration of the CMSIS-DSP library will be complete.

> **Note:** Verify that the paths you provide match the actual locations on your system. Adjust them accordingly if your directory structure differs.
