cmake_minimum_required (VERSION 3.20)
project(SoLoud)

list(APPEND CMAKE_MODULE_PATH  ${PROJECT_SOURCE_DIR}/cmake)

include(OptionDependentOnPackage)
include(PrintOptionStatus)

option(SOLOUD_DYNAMIC "Set to ON to build dynamic SoLoud" OFF)
print_option_status (SOLOUD_DYNAMIC "Build dynamic library")

option(SOLOUD_STATIC "Set to ON to build static SoLoud" ON)
print_option_status (SOLOUD_STATIC "Build static library")

option(SOLOUD_BUILD_DEMOS "Set to ON for building demos" OFF)
option(SOLOUD_ENABLE_BACKEND_NULL "Set to ON for building NULL backend" ON)
option(SOLOUD_ENABLE_BACKEND_SDL2 "Set to ON for building SDL2 backend" OFF)
option(SOLOUD_ENABLE_BACKEND_SDL3 "Set to ON for building SDL3 backend" OFF)
option(SOLOUD_ENABLE_BACKEND_ALSA "Set to ON for building the ALSA backend" OFF)
option(SOLOUD_ENABLE_BACKEND_COREAUDIO "Set to ON for building CoreAudio backend" OFF)
option(SOLOUD_ENABLE_BACKEND_OPENSLES "Set to ON for building OpenSLES backend" OFF)
option(SOLOUD_ENABLE_BACKEND_XAUDIO2 "Set to ON for building XAudio2 backend" OFF)
option(SOLOUD_ENABLE_BACKEND_WINMM "Set to ON for building WINMM backend" OFF)
option(SOLOUD_ENABLE_BACKEND_WASAPI "Set to ON for building WASAPI backend" OFF)
option(SOLOUD_AUTO_DETECT_BACKENDS "Attempt to enable backends based on what is available" ON)

if(SOLOUD_AUTO_DETECT_BACKENDS)
	find_package(SDL2)
	set(SOLOUD_ENABLE_BACKEND_SDL2 ${SDL2_FOUND})
	find_package(SDL3 CONFIG COMPONENTS SDL3-shared)
	set(SOLOUD_ENABLE_BACKEND_SDL3 ${SDL3_FOUND})
	find_package(ALSA)
	set(SOLOUD_ENABLE_BACKEND_ALSA ${ALSA_FOUND})

	if(APPLE)
		set(SOLOUD_BACKEND_COREAUDIO ON)
	endif()

	find_package(OpenSLES)
	set(SOLOUD_ENABLE_BACKEND_OPENSLES ${OPENSLES_FOUND})
endif()

if(SOLOUD_BUILD_DEMOS)
	set(SOLOUD_STATIC ON)
endif()

print_option_status (SOLOUD_BUILD_DEMOS "Build demos")
print_option_status (SOLOUD_AUTO_DETECT_BACKENDS "Auto Detect Backends")
print_option_status (SOLOUD_ENABLE_BACKEND_NULL "NULL backend")
print_option_status (SOLOUD_BACKEND_SDL2 "SDL2 backend")
print_option_status (SOLOUD_ENABLE_BACKEND_SDL3 "SDL3 backend")
print_option_status (SOLOUD_ENABLE_BACKEND_ALSA "ALSA backend")
print_option_status (SOLOUD_BACKEND_COREAUDIO "CoreAudio backend")
print_option_status (SOLOUD_BACKEND_OPENSLES "OpenSLES backend")
print_option_status (SOLOUD_BACKEND_XAUDIO2 "XAudio2 backend")
print_option_status (SOLOUD_BACKEND_WINMM "WINMM backend")
print_option_status (SOLOUD_BACKEND_WASAPI "WASAPI backend")

#Establish some basic mappings to filesystem directories
get_filename_component(SOLOUD_ROOT_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/.." ABSOLUTE)

include (src.cmake)

# Demos
if(SOLOUD_BUILD_DEMOS)
	include (demos.cmake)
endif ()

IF (SOLOUD_GENERATE_GLUE)
	include (gen_glue.cmake)
endif ()

# ------------------------------
# Install rules for SoLoud
# ------------------------------
include(GNUInstallDirs)

install(
	TARGETS soloud
	EXPORT SoLoudTargets
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Install headers (adjust path if needed)
install(
    DIRECTORY ${SOLOUD_ROOT_DIRECTORY}/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h"
)

# Export CMake targets
install(
    EXPORT SoLoudTargets
    FILE SoLoudTargets.cmake
    NAMESPACE SoLoud::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/SoLoud
)

# Create Config & Version files for find_package()
include(CMakePackageConfigHelpers)

configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/SoLoudConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/SoLoudConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/SoLoud
)

# write_basic_package_version_file(
#     ${CMAKE_CURRENT_BINARY_DIR}/SoLoudConfigVersion.cmake
#     VERSION ${PROJECT_VERSION}
#     COMPATIBILITY AnyNewerVersion
# )

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/SoLoudConfig.cmake
        #${CMAKE_CURRENT_BINARY_DIR}/SoLoudConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/SoLoud
)
