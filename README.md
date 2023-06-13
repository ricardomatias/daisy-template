# daisy-template

A simple development template for the daisy platform.

## REQUIREMENTS

You need the **libDaisy** and **DaisySP** somewhere on your system, I suggest adding a folder for all the audio related libraries you use and add it to your environment path.

* [Daisy Toolchain](https://github.com/electro-smith/DaisyWiki/wiki/1.-Setting-Up-Your-Development-Environment)
* [libDaisy](https://github.com/electro-smith/libDaisy)
* [DaisySP](https://github.com/electro-smith/DaisySP)

**IMPORTANT: In CMakeLists.txt there are a few points where you need to adapt the paths to your system**

## USAGE

If you use Visual Studio Code, then everything is already setup. Just make sure you have the recommended extensions installed and enabled. When CMake prompts you for choosing a kit, select [Unspecified], the toolchain file will take care of the configuration.

There are 3 available tasks:
* CMake: build - Builds the project
* CMake: build-flash-dfu (Default) - Builds and flashes the daisy platform via DFU (USB)
* CMake: build-flash-st-link - Builds and flashes the daisy platform via ST-Link

## DEBUGGING

There's a debugging task called "Cortex Debug". It works out of the box if you didn't change the `FIRMWARE_NAME` variable inside CMakeLists.txt. If you do, make sure you update `"executable": "${workspaceRoot}/build/daisy_firmware.elf"` in `.vscode/launch.json` to the right `.elf` target.

### libDaisy
Based on a tip from (Nick // Infrasonic Audio) I've activated optimizations on the Debug build of libDaisy, so that the flash memory used doesn't get exhausted.

Add the following when needing debug information in libDaisy. 

```
#pragma GCC push_options
#pragma GCC optimize ("O0")

your code

#pragma GCC pop_options
```
