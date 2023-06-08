# daisy-template

A simple development template for the daisy platform.

## REQUIREMENTS

You need the **libDaisy** and **DaisySP** somewhere on your system, I suggest adding a folder for all the audio related libraries you use and add it to your environment path.

* [Daisy Toolchain](https://github.com/electro-smith/DaisyWiki/wiki/1.-Setting-Up-Your-Development-Environment)
* [libDaisy](https://github.com/electro-smith/libDaisy)
* [DaisySP](https://github.com/electro-smith/DaisySP)

**In CMakeLists.txt there are a few points where you need to adapt the paths to your system**

## DEVELOPMENT

Based on a tip from (Nick // Infrasonic Audio) I've activated optimizations on the Debug build of libDaisy, so that the flash memory used doesn't get exhausted.

Add the following when needing debug information in libDaisy. 

```
#pragma GCC push_options
#pragma GCC optimize ("O0")

your code

#pragma GCC pop_options
```
