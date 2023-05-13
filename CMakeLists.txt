cmake_minimum_required(VERSION 3.15)

project(ADLplug-AE VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

### from ADLMIDI.cmake, modified a bit
set(WITH_MIDI_SEQUENCER OFF CACHE STRING "" FORCE)
set(WITH_MUS_SUPPORT OFF CACHE STRING "" FORCE)
set(WITH_XMI_SUPPORT OFF CACHE STRING "" FORCE)
set(WITH_EMBEDDED_BANKS OFF CACHE STRING "" FORCE)

set(libADLMIDI_STATIC ON CACHE STRING "" FORCE)
set(libADLMIDI_SHARED OFF CACHE STRING "" FORCE)
add_subdirectory("external/ADLplug/thirdparty/libADLMIDI" EXCLUDE_FROM_ALL)
target_compile_definitions(ADLMIDI_static PRIVATE "ADLMIDI_EXPORT=")
target_compile_definitions(ADLMIDI_static PUBLIC "ADLMIDI_UNSTABLE_API=")

set(libOPNMIDI_STATIC ON CACHE STRING "" FORCE)
set(libOPNMIDI_SHARED OFF CACHE STRING "" FORCE)
add_subdirectory("external/ADLplug/thirdparty/libOPNMIDI" EXCLUDE_FROM_ALL)
target_compile_definitions(OPNMIDI_static PRIVATE "OPNMIDI_EXPORT=")
target_compile_definitions(OPNMIDI_static PUBLIC "OPNMIDI_UNSTABLE_API=")
### end of copy


if(ADLplug_RT_CHECKER)
    add_definitions("-DADLplug_RT_CHECKER=1")
    set(ADLplug_RT_CHECKER_LINK_FLAGS "-Wl,--wrap=malloc -Wl,--wrap=realloc -Wl,--wrap=free -Wl,--wrap=memalign")
endif()

if(ADLplug_ASSERTIONS)
    add_definitions("-UNDEBUG")
endif()

# deps

if(ADLplug_USE_SYSTEM_FMT)
    find_package(fmt 5.0.0)
    if(NOT fmt_FOUND)
        message(FATAL_ERROR "Unable to find fmt 5.x on system.")
    endif()
    add_library(ADLplug_fmt INTERFACE)
    target_compile_definitions(ADLplug_fmt INTERFACE "FMT_USE_USER_DEFINED_LITERALS=0")
    target_link_libraries(ADLplug_fmt INTERFACE fmt)
else()
    add_library(ADLplug_fmt STATIC
            "external/ADLplug/thirdparty/fmt/src/format.cc"
            "external/ADLplug/thirdparty/fmt/src/posix.cc")
    target_include_directories(ADLplug_fmt PUBLIC "external/ADLplug/thirdparty/fmt/include")
    target_compile_definitions(ADLplug_fmt PUBLIC "FMT_USE_USER_DEFINED_LITERALS=0")
    set_property(TARGET ADLplug_fmt PROPERTY POSITION_INDEPENDENT_CODE ON)
endif()

add_library(simple-ini STATIC "external/ADLplug/thirdparty/simpleini/ConvertUTF.cpp")
target_include_directories(simple-ini INTERFACE "external/ADLplug/thirdparty/simpleini")

add_library(wopl STATIC "external/ADLplug/thirdparty/wopl/wopl/wopl_file.c")
target_include_directories(wopl PUBLIC "thirdparty/wopl")
set_property(TARGET wopl PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(wopn STATIC "external/ADLplug/thirdparty/wopn/wopn/wopn_file.c")
target_include_directories(wopn PUBLIC "thirdparty/wopn")
set_property(TARGET wopn PROPERTY POSITION_INDEPENDENT_CODE ON)

add_subdirectory(external/JUCE)
add_subdirectory(external/clap-juce-extensions EXCLUDE_FROM_ALL)



# common build fragment using macro

function(ADLAE_ADD_PLUGIN BaseTargetName ADLplug_CHIP)

message("Building Plugin: ${BaseTargetName} ...")

juce_add_plugin("${BaseTargetName}"
        ICON_BIG ${ADLAE_ICON_BIG}
        ICON_SMALL ${ADLAE_ICON_SMALL}
        COMPANY_NAME "Atsushi Eno"
        IS_SYNTH TRUE
        NEEDS_MIDI_INPUT TRUE
        NEEDS_MIDI_OUTPUT FALSE
        IS_MIDI_EFFECT FALSE
        EDITOR_WANTS_KEYBOARD_FOCUS TRUE
        COPY_PLUGIN_AFTER_BUILD TRUE
        PLUGIN_MANUFACTURER_CODE AENO
        PLUGIN_CODE ${ADLAE_PLUGIN_CODE}
        FORMATS AU AUv3 VST3 LV2 Standalone
        LV2URI ${ADLAE_LV2_URI}
        PRODUCT_NAME ${ADLAE_PLUGIN_NAME})

clap_juce_extensions_plugin(TARGET ${BaseTargetName}
        CLAP_ID ${ADLAE_CLAP_ID}
        CLAP_FEATURES instrument)

## deps

target_link_libraries(${BaseTargetName} PUBLIC ADLplug_fmt)
target_link_libraries(${BaseTargetName} PUBLIC simple-ini)
target_include_directories(${BaseTargetName} PUBLIC "external/ADLplug/thirdparty/folly")

## JUCE stuff

target_compile_definitions(${BaseTargetName}
        PUBLIC
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_CAN_REPLACE_VST2=0)

target_link_libraries(${BaseTargetName} PRIVATE
        juce_audio_utils
        juce_recommended_config_flags
        juce_recommended_lto_flags
        juce_recommended_warning_flags)

target_sources(${BaseTargetName}
        PRIVATE
        external/ADLplug/sources/bank_manager.cc
        external/ADLplug/sources/configuration.cc
        #external/ADLplug/sources/jack_main.cc
        external/ADLplug/sources/messages.cc
        external/ADLplug/sources/midi/insnames.cc
        external/ADLplug/sources/nothing.cc
        external/ADLplug/sources/pch/ADLplug_core.cc
        external/ADLplug/sources/plugin_editor.cc
        external/ADLplug/sources/plugin_processor.cc
        external/ADLplug/sources/plugin_state.cc
        external/ADLplug/sources/resources.cc
        external/ADLplug/sources/ui/components/algorithm_component.cc
        external/ADLplug/sources/ui/components/indicator_NxM.cc
        external/ADLplug/sources/ui/components/knob_component.cc
        external/ADLplug/sources/ui/components/midi_keyboard_ex.cc
        external/ADLplug/sources/ui/components/new_program_editor.cc
        external/ADLplug/sources/ui/components/program_name_editor.cc
        external/ADLplug/sources/ui/components/styled_knobs.cc
        external/ADLplug/sources/ui/components/vu_meter.cc
        external/ADLplug/sources/ui/components/wave_label.cc
        external/ADLplug/sources/ui/look_and_feel.cc
        external/ADLplug/sources/ui/utility/graphics.cc
        external/ADLplug/sources/ui/utility/image.cc
        external/ADLplug/sources/ui/utility/key_maps.cc
        external/ADLplug/sources/ui/utility/knobman_skin.cc
        external/ADLplug/sources/utility/midi.cc
        external/ADLplug/sources/utility/pak.cc
        external/ADLplug/sources/utility/rt_checker.cc
        external/ADLplug/sources/utility/simple_fifo.cc
        external/ADLplug/sources/worker.cc
        )

target_include_directories(${BaseTargetName}
        PRIVATE
        external/ADLplug/sources
        )

juce_generate_juce_header(${BaseTargetName})

# chip-specifics (had to specify here after all...)

if (${ADLplug_CHIP} STREQUAL "OPL3")
    ## OPL3 specifics

    target_compile_definitions(${BaseTargetName} PUBLIC "ADLPLUG_OPL3")

    target_link_libraries(${BaseTargetName} PUBLIC wopl)

    target_link_libraries(${BaseTargetName} PUBLIC ADLMIDI_static)

    target_sources(${BaseTargetName} PRIVATE
            external/ADLplug/sources/opl3/adl/chip_settings.cc
            external/ADLplug/sources/opl3/adl/instrument.cc
            external/ADLplug/sources/opl3/adl/instrument_properties.cc
            external/ADLplug/sources/opl3/adl/player.cc
            external/ADLplug/sources/opl3/parameter_block.cc
            external/ADLplug/sources/opl3/ui/about_component.cc
            external/ADLplug/sources/opl3/ui/components/algorithm_help.cc
            external/ADLplug/sources/opl3/ui/components/opl3_waves.cc
            external/ADLplug/sources/opl3/ui/main_component.cc
            external/ADLplug/sources/opl3/ui/operator_editor.cc
            external/ADLplug/sources/opl3/adl/measurer/chips/dosbox/dbopl.cpp
            external/ADLplug/sources/opl3/adl/measurer/chips/dosbox_opl3.cpp
            external/ADLplug/sources/opl3/adl/measurer/measurer.cpp
            )
    target_include_directories(${BaseTargetName}
            PRIVATE
            external/ADLplug/sources/opl3
            external/ADLplug/thirdparty/libADLMIDI/include
            external/ADLplug/thirdparty/wopl
            )
    target_compile_definitions(${BaseTargetName}
            PUBLIC
            ADLPLUG_OPL3=1
            )

elseif (${ADLplug_CHIP} STREQUAL "OPN2")
    ## OPNplug specifics

    target_compile_definitions(${BaseTargetName} PUBLIC "ADLPLUG_OPN2")

    target_link_libraries(${BaseTargetName} PUBLIC wopn)

    target_link_libraries(${BaseTargetName} PUBLIC OPNMIDI_static)

    target_sources(${BaseTargetName} PRIVATE
            external/ADLplug/sources/opn2/adl/chip_settings.cc
            external/ADLplug/sources/opn2/adl/instrument.cc
            external/ADLplug/sources/opn2/adl/instrument_properties.cc
            external/ADLplug/sources/opn2/adl/player.cc
            external/ADLplug/sources/opn2/parameter_block.cc
            external/ADLplug/sources/opn2/ui/about_component.cc
            external/ADLplug/sources/opn2/ui/components/algorithm_help.cc
            external/ADLplug/sources/opn2/ui/components/ssgeg_waves.cc
            external/ADLplug/sources/opn2/ui/main_component.cc
            external/ADLplug/sources/opn2/ui/operator_editor.cc
            external/ADLplug/sources/opn2/adl/measurer/chips/mame/mame_ym2612fm.cpp
            external/ADLplug/sources/opn2/adl/measurer/chips/mame_opn2.cpp
            external/ADLplug/sources/opn2/adl/measurer/measurer.cpp
            )
    target_include_directories(${BaseTargetName}
            PRIVATE
            external/ADLplug/sources/opn2
            external/ADLplug/thirdparty/libOPNMIDI/include
            external/ADLplug/thirdparty/wopn
            )
    target_compile_definitions(${BaseTargetName}
            PUBLIC
            ADLPLUG_OPN2=1
            )
else ()
    message(FATAL_ERROR "unsupported chip ${ADLplug_CHIP}")
endif()

endfunction()

# end of common stuff


# build ADLplug (OPL3)

set(ADLAE_PLUGIN_NAME "ADLplug-AE")
set(ADLAE_PLUGIN_CODE "ADLA")
set(ADLAE_ICON_BIG "external/ADLplug/resources/application/ADLplug-96.png")
set(ADLAE_ICON_SMALL "external/ADLplug/resources/application/ADLplug-32.png")
set(ADLAE_LV2_URI "https://github.com/atsushieno/ADLplug-AE?chip=OPL3")
set(ADLAE_CLAP_ID "dev.atsushieno.ports.adlplug")
set(BaseTargetName ${ADLAE_PLUGIN_NAME})

ADLAE_ADD_PLUGIN("ADLplug-AE" "OPL3")


# build OPNplug (OPN2)

set(ADLAE_PLUGIN_NAME "OPNplug-AE")
set(ADLAE_PLUGIN_CODE "OPNA")
set(ADLAE_ICON_BIG "external/ADLplug/resources/application/OPNplug-96.png")
set(ADLAE_ICON_SMALL "external/ADLplug/resources/application/OPNplug-32.png")
set(ADLAE_LV2_URI "https://github.com/atsushieno/ADLplug-AE?chip=OPN2")
set(ADLAE_CLAP_ID "dev.atsushieno.ports.opnplug")
set(BaseTargetName ${ADLAE_PLUGIN_NAME})

ADLAE_ADD_PLUGIN("OPNplug-AE" "OPN2")
