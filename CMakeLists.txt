cmake_minimum_required (VERSION 3.19)
project (Project)

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(WINDOWS TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(LINUX TRUE)
else()
    set(ANDROID TRUE)
endif()

#os specific
if (WINDOWS OR ANDROID)
    if (CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(SFML_DEBUG TRUE)
    else()
        set(SFML_DEBUG FALSE)
    endif()
elseif(LINUX)
    set(SFML_DEBUG FALSE)
endif()

#SFML SHARED LIBRARY
if (WINDOWS)
    set(SFML_SHARED FALSE)
else(LINUX OR ANDROID)
    set(SFML_SHARED TRUE)
endif()

#FILES
file(GLOB_RECURSE Project_SOURCES "src/*.cpp")
if (NOT EXISTS "assets/")
    file(MAKE_DIRECTORY "assets/")
endif()
if (NOT EXISTS "SFML/")
    file(MAKE_DIRECTORY "SFML/")
endif()
if (NOT EXISTS "versions/")
    file(MAKE_DIRECTORY "versions/")
endif()

if (NOT ANDROID)
    file(COPY "assets/" DESTINATION "${CMAKE_BINARY_DIR}")
    if (WINDOWS)
        file(COPY "SFML/bin/openal32.dll" DESTINATION "${CMAKE_BINARY_DIR}/")
    endif()
endif()

#OPTIMIZATION FLAGS
if (NOT ANDROID)
    set(CMAKE_CXX_FLAGS_RELEASE "-O2")
endif()


if (WINDOWS)
    #enable_language("RC")
    #set (WIN32_RESOURCES "src/icon.rc")
    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES} #[[${WIN32_RESOURCES}]])
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES LINK_FLAGS_DEBUG "/SUBSYSTEM:CONSOLE")
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")
    target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE SFML/include/)
    target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE SFML/lib/)
elseif(LINUX)
    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES})
elseif(ANDROID)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")
    set(CMAKE_MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)
    set(PATH_TO_SFML ${ANDROID_NDK}/sources/third_party/sfml)
    set(SFML_DIR ${PATH_TO_SFML}/lib/${ANDROID_ABI}/cmake/SFML)
    find_package(SFML 2 COMPONENTS system)

    add_library(${CMAKE_PROJECT_NAME} SHARED ${Project_SOURCES})

    add_library(openal SHARED IMPORTED)
    set_target_properties(openal PROPERTIES IMPORTED_LOCATION_RELEASE ${PATH_TO_SFML}/extlibs/lib/${ANDROID_ABI}/libopenal.so)
    set_target_properties(openal PROPERTIES IMPORTED_LOCATION_DEBUG ${PATH_TO_SFML}/extlibs/lib/${ANDROID_ABI}/libopenal.so)
endif()


#CPP VERSION
target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_11)

#STATIC MACRO
if (NOT SFML_SHARED)
    target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_STATIC)
endif()


#sfml specific
if (SFML_DEBUG AND NOT SFML_SHARED)
    set (LINK_SFML
        winmm
        sfml-system-s-d
        opengl32
        gdi32
        sfml-window-s-d
        freetype
        sfml-graphics-s-d
        openal32
        flac
        vorbisenc
        vorbisfile
        vorbis
        ogg
        sfml-audio-s-d
        ws2_32
        sfml-network-s-d
        )
elseif(SFML_DEBUG AND SFML_SHARED)
    set (LINK_SFML
        sfml-system-d
        sfml-window-d
        sfml-graphics-d
        sfml-audio-d
        sfml-network-d
        )
elseif(NOT SFML_DEBUG AND SFML_SHARED)
    set (LINK_SFML
        sfml-system
        sfml-window
        sfml-graphics
        sfml-audio
        sfml-network
        )
elseif(NOT SFML_DEBUG AND NOT SFML_SHARED)
    set (LINK_SFML
        winmm
        sfml-system-s
        opengl32
        gdi32
        sfml-window-s
        freetype
        sfml-graphics-s
        openal32
        flac
        vorbisenc
        vorbisfile
        vorbis
        ogg
        sfml-audio-s
        ws2_32
        sfml-network-s
        )
endif()

if (WINDOWS)
    if (SFML_DEBUG)
        target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} sfml-main-d)
    else()
        target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} sfml-main)
    endif()
elseif(LINUX)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML})
elseif(ANDROID)
    target_link_libraries(${CMAKE_PROJECT_NAME}
        log
        android
        sfml-system
        sfml-window
        sfml-graphics
        sfml-audio
        sfml-activity
        sfml-network
        -Wl,--whole-archive sfml-main -Wl,--no-whole-archive
        openal
        EGL
        GLESv1_CM
)
endif()
