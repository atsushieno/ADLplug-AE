# ADLplug-AE

## What is this?

ADLplug-AE is a modernized builds of [jpcima/ADLplug](https://github.com/jpcima/ADLplug/), namely:

- based on JUCE7.
- revamped CMake build script so that we can easily integrate others'
  JUCE efforts that work well with the official JUCE CMake scripts.
  - supports LV2 in JUCE7 way (uses LV2 Patch and Atom for parameters)
  - supports CLAP build.
- eliminated modal dialogs for Android (for Standalone and AAP plugin).
- build both ADLplug and OPNplug at a time, without cmake arguments.

Everything else is left as is, so far.

Everything is still subject to changes. Do not expect it as a stable synth plugin yet.

## Building

This repository keeps changes to the dependencies as minimum as possible.
We do not make changes into a fork branch, but has just a `.patch` file to ADLplug.

After checking out from this git repository, run the following commands:

```
git submodule update --init --recursive # in case you did not
cd external/ADLplug
patch -i ../../adlplug-ae.patch -p1
cd ../..
cmake -B build # or any name instead of `build`
cmake --build build # ditto
```

... and the build artifacts would show up under `build` (or whatever you specified). The build artifacts would be dependent on each platform.

## LICENSES

ADLplug-AE is released under the GPLv3 license.

ADLplug codebase is released under the Boost license.

JUCE is released under the GPLv3 license.

clap-juce-extensions is released under the MIT license.

