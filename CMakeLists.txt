cmake_minimum_required(VERSION 3.16)

# If we are compiling for Mac OS we want to target OS versions down to 10.9
if (APPLE)
    set (CMAKE_OSX_DEPLOYMENT_TARGET "13.0" CACHE INTERNAL "")
endif()

project(Circle VERSION 0.0.1)

# Add JUCE
add_subdirectory(Ext/JUCE)

juce_add_module(Ext/foleys_gui_magic)

#set(FormatsToBuild VST3)
# set(AAX_SDK_PATH "AAX_SDK_2p3p2")

# comment or delete this line if VST2 SDK is not present
#set(VST2_SDK_PATH "VSTSDK/VST2_SDK")

# On MacOs build AU
if (APPLE)
    list(APPEND FormatsToBuild AU)
endif ()

# if path for AAX is present, build for AAX
#if (AAX_SDK_PATH)
#    juce_set_aax_sdk_path (${AAX_SDK_PATH})
#    list (APPEND FormatsToBuild AAX)
#endif()

# if path for VST2 is present, build for VST2
#if (VST2_SDK_PATH)
#    juce_set_vst2_sdk_path (${VST2_SDK_PATH})
#    list (APPEND FormatsToBuild VST)
#endif()

# Copy plugin in root directory only on MacOs
set(Copy_Plugin_After_Build FALSE)
if (APPLE)
    set(Copy_Plugin_After_Build TRUE)
endif ()

# Add plugin target
juce_add_plugin(Circle
        COMPANY_NAME ovolovo
        PLUGIN_MANUFACTURER_CODE Ovol
        PLUGIN_CODE Circ
        FORMATS ${FormatsToBuild}
        PRODUCT_NAME "Circle"
        IS_SYNTH FALSE
        NEEDS_MIDI_INPUT FALSE
        NEEDS_MIDI_OUTPUT FALSE
        IS_MIDI_EFFECT FALSE
#        AAX_IDENTIFIER com.KarotaSound.CutTheMooog
#        VST3_CATEGORIES Fx Distortion
        AU_MAIN_TYPE kAudioUnitType_Effect
#        AAX_CATEGORY AAX_ePlugInCategory_Harmonic
        COPY_PLUGIN_AFTER_BUILD ${Copy_Plugin_After_Build}
        AU_COPY_DIR ${/Users/dave/Library/Audio/Plug-Ins}
)

# compile with C++14
target_compile_features(Circle PRIVATE cxx_std_20)

# create JUCE header
juce_generate_juce_header(Circle)

# add plugin source file directory
add_subdirectory(Source)

# add plugin binary data files
juce_add_binary_data(BinaryData
        SOURCES
            Resources/Circle.xml
            Resources/Circle.png
            Resources/FuturaBoldFont.ttf
            Resources/FuturaBookFont.ttf
            Resources/FuturaHeavyfont.ttf
            Resources/powerswitch.svg
)

target_compile_definitions(Circle
        PUBLIC
        # JUCE flags
        JUCE_DISPLAY_SPLASH_SCREEN=0
        JUCE_REPORT_APP_USAGE=0
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_CAN_REPLACE_VST2=0

        # FOLEYS flags
        FOLEYS_SHOW_GUI_EDITOR_PALLETTE=0
        FOLEYS_ENABLE_BINARY_DATA=1
        FOLEYS_ENABLE_OPEN_GL_CONTEXT=1

        # plugin setup
        JucePlugin_Manufacturer="ovolovo"
        JucePlugin_VersionString="${CMAKE_PROJECT_VERSION}"
        JucePlugin_Name="Circle"
)

target_link_libraries(Circle PRIVATE
        BinaryData
        juce::juce_audio_utils
        juce::juce_audio_plugin_client
        foleys_gui_magic
        juce::juce_dsp
        PUBLIC
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags
        juce::juce_recommended_warning_flags
)

set_target_properties(BinaryData PROPERTIES # replace target name
        POSITION_INDEPENDENT_CODE TRUE)


