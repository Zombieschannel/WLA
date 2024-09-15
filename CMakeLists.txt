cmake_minimum_required (VERSION 3.15)
project (Project)

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(WINDOWS TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(LINUX TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(ANDROID TRUE)
endif()

#SFML SHARED LIBRARY
if (WINDOWS)
    set(SFML_STATIC_LIBRARIES TRUE)
else(LINUX OR ANDROID)
    set(SFML_STATIC_LIBRARIES FALSE)
endif()

######################################################
# SET YOUR SFML DIR FOR EACH ARCHITECTURE ON DESKTOP #
######################################################
if (WINDOWS)
    if (${BUILD_ARCH} STREQUAL "x86")
        set(SFML_LOC "<insert>/SFML-2.6.1-x86")
    elseif(${BUILD_ARCH} STREQUAL "x64")
        set(SFML_LOC "<insert>/SFML-2.6.1-x64")
    elseif(${BUILD_ARCH} STREQUAL "ARM64")
        set(SFML_LOC "<insert>/SFML-2.6.1-ARM64")
    endif()
elseif(LINUX)
    set(SFML_LOC "<insert>/SFML-2.6.1")
endif()

#FILES
file(GLOB_RECURSE Project_SOURCES "src/*.cpp")
if (NOT EXISTS "assets/")
    file(MAKE_DIRECTORY "assets/")
endif()
if (NOT EXISTS "versions/")
    file(MAKE_DIRECTORY "versions/")
endif()

if (NOT ANDROID)
    file(COPY "assets/" DESTINATION "${CMAKE_BINARY_DIR}")
    if (WINDOWS)
        file(COPY "${SFML_LOC}/bin/openal32.dll" DESTINATION "${CMAKE_BINARY_DIR}/")
    endif()
endif()

if (WINDOWS)

    #enable_language("RC") #uncoment if using icons
    #set (WIN32_RESOURCES "src/icon.rc")

    set(SFML_DIR "${SFML_LOC}/lib/cmake/SFML")
    find_package(SFML 2.6 COMPONENTS graphics audio window system network REQUIRED)

    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES} #[[${WIN32_RESOURCES}]])
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES LINK_FLAGS_DEBUG "/SUBSYSTEM:CONSOLE")
    set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup")

elseif(LINUX)
    
    set(SFML_DIR "${SFML_LOC}/lib/cmake/SFML")
    find_package(SFML 2.6 COMPONENTS graphics audio window system network REQUIRED)
    add_executable (${CMAKE_PROJECT_NAME} ${Project_SOURCES})

elseif(ANDROID)

    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")
    set(CMAKE_MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)
    set(PATH_TO_SFML ${ANDROID_NDK}/sources/third_party/sfml)
    set(SFML_DIR ${PATH_TO_SFML}/lib/${ANDROID_ABI}/cmake/SFML)
    find_package(SFML 2.6 COMPONENTS graphics audio window system network REQUIRED)

    add_library(${CMAKE_PROJECT_NAME} SHARED ${Project_SOURCES})

    add_library(openal SHARED IMPORTED)
    set_target_properties(openal PROPERTIES IMPORTED_LOCATION_RELEASE ${PATH_TO_SFML}/extlibs/lib/${ANDROID_ABI}/libopenal.so)
    set_target_properties(openal PROPERTIES IMPORTED_LOCATION_DEBUG ${PATH_TO_SFML}/extlibs/lib/${ANDROID_ABI}/libopenal.so)

endif()

#C++ version
target_compile_features(${CMAKE_PROJECT_NAME} PRIVATE cxx_std_17)

#uncomment if you'd like to check which architecture is active on Windows from the c++ side
#[[
if (WINDOWS)
    if (${BUILD_ARCH} STREQUAL "x86")
        target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_X86)
    elseif(${BUILD_ARCH} STREQUAL "x64")
        target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_X64)
    elseif(${BUILD_ARCH} STREQUAL "ARM64")
        target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE SFML_ARM64)
    endif()
endif()
]]

set (LINK_SFML sfml-system sfml-window sfml-graphics sfml-audio sfml-network)

if (WINDOWS)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} sfml-main)
elseif(LINUX)
    target_link_libraries(${CMAKE_PROJECT_NAME} ${LINK_SFML} -lGL -lX11)
elseif(ANDROID)
    target_link_libraries(${CMAKE_PROJECT_NAME}
        log
        android
        ${LINK_SFML}
        -Wl,--whole-archive sfml-main -Wl,--no-whole-archive
        openal
        EGL
        GLESv1_CM
    )
endif()
