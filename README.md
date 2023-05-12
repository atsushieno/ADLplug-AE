# ADLplug-AE

## What is this?

ADLplug-AE is a modernized builds of [jpcima/ADLplug](https://github.com/jpcima/ADLplug/), namely:

- based on JUCE7.
- revamped CMake build script so that we can easily integrate others'
  JUCE efforts that works with the official JUCE CMake scripts.
  - supports LV2 in JUCE7 way (uses LV2 Patch and Atom for parameters)
  - supports CLAP build.
- eliminated modal dialogs for Android (for Standalone and AAP plugin).

Everything else is left as is, so far.

Everything is still subject to changes. Do not expect it as a stable synth plugin yet.


## LICENSES

ADLplug-AE is released under the GPLv3 license.

ADLplug codebase is released under the Boost license.

JUCE is released under the GPLv3 license.

clap-juce-extensions is released under the MIT license.

