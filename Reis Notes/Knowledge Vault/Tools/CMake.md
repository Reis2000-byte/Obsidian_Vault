
## What is CMake?  

**CMake** is a build system generator. It does not compile code itself.  

Instead, CMake reads project configuration files (`CMakeLists.txt`) and generates build files for tools like:  

- Ninja  

- Make  

- Visual Studio  

- Eclipse  

Think of it like this:  

```text  

Source Code  

↓  

CMake  

↓  

Build System (Ninja/Make)  

↓  

Compiler (GCC, Clang, ARM GCC)  

↓  

Firmware / Executable  

```  

In embedded projects, CMake is commonly used with:  

- STM32  

- NXP  

- Zephyr  

- FreeRTOS  

- ESP-IDF  

- ARM GCC toolchains  

## Why Use CMake?  

Without CMake:  

```bash  

arm-none-eabi-gcc main.c motor.c adc.c ...  

```  

As projects grow, build commands become

# Common CMake Commands  

## Configure Build Directory  

```bash  

cmake -B build  

```  

Generate build files in the `build` folder.  
## Configure with Explicit Source Directory  

```bash  

cmake -S . -B build  

```  
- `-S` = source directory  

- `-B` = build directory  
## Build Project  

```bash  

cmake --build build  

```  

Compile the project.  
## Parallel Build  
```bash  

cmake --build build -j  

```  
Use multiple CPU cores.  
## Debug Build  
```bash  

cmake -B build -DCMAKE_BUILD_TYPE=Debug  

```  

  
Includes debugging symbols.  
## Release Build  
```bash  

cmake -B build -DCMAKE_BUILD_TYPE=Release  

```  

Optimized build.  
## Specify Toolchain  
```bash  

cmake -B build -DCMAKE_TOOLCHAIN_FILE=toolchain.cmake  

```  

Very common for embedded systems.  
## Install Build Artifacts  
```bash  

cmake --install build  

```  

Copies outputs to install locations.  
## Show Available Cache Variables  
```bash  

cmake -LAH  

```  

  

Useful for debugging build configuration.  

  

## View CMake Version  

  

```bash  

cmake --version  

```  

  

## Run Unit Tests  

  

```bash  

ctest  

```  

  

or  

  

```bash  

ctest --test-dir build  

```  

  

---  

  

# CMake Folder Structure  

  

A typical embedded project might look like:  

  

```text  

MotorController/  

│  

├── CMakeLists.txt  

├── toolchain.cmake  

│  

├── Src/  

│ ├── main.c  

│ ├── motor.c  

│ └── adc.c  

│  

├── Inc/  

│ ├── motor.h  

│ └── adc.h  

│  

├── Drivers/  

│ ├── STM32_HAL/  

│ └── CMSIS/  

│  

├── Middleware/  

│ └── FreeRTOS/  

│  

├── Tests/  

│  

└── build/  

```  

  

## What Each Folder Does  

  

### Src/  

  

Contains implementation files.  

  

```text  

main.c  

motor.c  

adc.c  

```  

  

### Inc/  

  

Contains headers.  

  

```text  

motor.h  

adc.h  

```  

  

### Drivers/  

  

Vendor and hardware drivers.  

  

Examples:  

  

```text  

HAL  

CMSIS  

BSP  

```  

  

### Middleware/  

  

Third-party software.  

  

Examples:  

  

```text  

FreeRTOS  

lwIP  

USB Stack  

```  

  

### build/  

  

Generated files.  

  

Never edit files here manually.  

  

Typically contains:  

  

```text  

Makefiles  

Ninja files  

Object files  

Executables  

Firmware  

```  

  

If things get weird:  

  

```bash  

rm -rf build  

```  

  

and regenerate.  

  

---  

  

# Multi-Level CMake Projects  

  

Most embedded projects have multiple CMakeLists files.  

  

Example:  

  

```text  

Project/  

│  

├── CMakeLists.txt  

│  

├── Drivers/  

│ └── CMakeLists.txt  

│  

├── App/  

│ └── CMakeLists.txt  

│  

└── Middleware/  

└── CMakeLists.txt  

```  

  

## Root CMakeLists.txt  

  

Usually contains:  

  

```cmake  

cmake_minimum_required(VERSION 3.20)  

  

project(MotorController)  

  

add_subdirectory(Drivers)  

add_subdirectory(App)  

add_subdirectory(Middleware)  

```  

  

Think of it as:  

  

```text  

Root Project  

│  

├── Drivers  

├── App  

└── Middleware  

```  

  

---  

  

# add_subdirectory()  

  

This is one of the most common commands you'll see.  

  

```cmake  

add_subdirectory(Drivers)  

```  

  

It tells CMake:  

  

> Go into that folder and process its CMakeLists.txt.  

  

---  

  

# Real Embedded Project Example  

  

Root:  

  

```cmake  

project(MotorController)  

  

add_subdirectory(Drivers)  

add_subdirectory(App)  

  

add_executable(firmware)  

```  

  

Drivers:  

  

```cmake  

add_library(drivers  

adc.c  

uart.c  

)  

```  

  

App:  

  

```cmake  

target_sources(firmware PRIVATE  

main.c  

motor_control.c  

)  

```  

  

Root:  

  

```cmake  

target_link_libraries(firmware drivers)  

```  

  

Result:  

  

```text  

firmware  

├── main.c  

├── motor_control.c  

├── adc.c  

└── uart.c  

```  

  

---  

  

# Frequently Seen Variables  

  

## Project Name  

  

```cmake  

${PROJECT_NAME}  

```  

  

## Source Directory  

  

```cmake  

${CMAKE_SOURCE_DIR}  

```  

  

Root of project.  

  

## Current Directory  

  

```cmake  

${CMAKE_CURRENT_SOURCE_DIR}  

```  

  

Current folder's source path.  

  

## Build Directory  

  

```cmake  

${CMAKE_BINARY_DIR}  

```  

  

Usually:  

  

```text  

build/  

```  

  

## Compiler  

  

```cmake  

${CMAKE_C_COMPILER}  

```  

  

Example:  

  

```text  

arm-none-eabi-gcc  

```  

  

---  

  

# Things You'll See All The Time  

  

## GLOB Source Files  

  

```cmake  

file(GLOB APP_SOURCES  

"*.c"  

)  

```  

  

Collects matching files*  

  

Many*teams*avoid this because CMake may not n*tice newly*added files without reconfiguratio**  

  

*--  

  

## Include Directories  

  

```cma*e  

target_include_directories(app P*IVATE  

Inc  

Drivers  

)  

```  

  

E*uivalent to:  

  

```bash  

-IInc -IDriv*rs  

```  

  

---  

  

## Compiler Defines  

  

*``cmake  

target_compile_definitions*app PRIVATE  

DEBUG  

STM32G43*xx  

)  

```  

  

Equivalent to:  

  

```bash  

*DDEBUG  

-DSTM32G431xx  

*``  

  

---  

  

## Compiler Options  

  

```c*ake  

target_compile_options(app PRI*ATE  

-Wall  

-Wextra  

)  

```  

  

E*uivalent to:  

  

```bash  

gcc -Wall -W*xtra  

```  

  

---  

  

## Linker Options  

  

*``cmake  

target_link_options(app PR*VATE  

-TSTM32G431.ld  

)  

```  

  

Com*on for embedded linker*scripts.  

  

---  

  

# How CMake Thinks  

*New*users think:  

  

```text  

Folder-based*```  

  

CMake thinks:  

  

```text  

Target*based  

```  

  

*eaning CMake cares about:  

  

-*Execut*bles  

- Libraries  

- Dependencies  

  

I* does*NOT really care about folder organ*zation.  

  

This is the single bigges* mindset shift.  

  

---  

  

# Typical Em*edded Build Flow  

  

```text  

CMakeLis*s.txt  

↓  

Configure  

↓  

*enerate Ninja/Make  

↓  

*ompile Sources  

↓  

Link*Objects* ↓  

.elf  

↓  

.bin / .hex* *↓*Flash MCU  

```  

  

---  

  

# Commands*You'll Use Most Often  

  

```bash  

cm*ke -B build  

  

cmake --build build  

  

*make --build build -j  

  

cmake -B bu*ld*-DCMAKE_TOOLCHAIN_FILE=toolchain.c*ake  

  

cmake -B build -DCMAKE_BUILD_*YPE=Debug  

  

cmake -LAH  

  

ctest  

  

rm -*f build  

```  

  

---  

  

# Embedded Engin*er Translation  

  

| I want to... | U*e |  

|--------------|------|  

| Add * source file | `target_sources()` *  

| Add an include folder | `target*include_directories()` |  

| Add a l*brary | `add_library()` |  

| Link a*library | `target_link_libraries()* |  

| Add a compiler flag | `target*compile_options()` |  

| Add a `#def*ne` | `target_compile_definitions(*` |  

| Process another folder | `ad*_subdirectory()` |  

| Build firmwar* | `cmake --build build` |  

| Start*over | Delete `build/` |  

| Find co*piler | Check toolchain file |  

```*