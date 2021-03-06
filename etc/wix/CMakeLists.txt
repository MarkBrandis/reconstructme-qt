# @file
# @copyright Copyright (c) 2013 PROFACTOR GmbH. All rights reserved. 
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
# 
#     * Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
#     * Redistributions in binary form must reproduce the above
# copyright notice, this list of conditions and the following disclaimer
# in the documentation and/or other materials provided with the
# distribution.
#     * Neither the name of Profactor GmbH nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# 
# @authors Christoph Heindl
# 

# To generate the installer we currently use proprietary CMake functionality
# by PROFACTOR which is currently not part of the open-source distribution.

# Check for required PROFACTOR CMake functionality
set(ROBVIS_CMAKE_DIRECTORY ${CMAKE_SOURCE_DIR}/robvis/cmake CACHE PATH "Directory that contains the robvis cmake directory.")
if (NOT IS_DIRECTORY ${ROBVIS_CMAKE_DIRECTORY})
	message(STATUS "Required PROFACTOR CMake was not found. It is required for installer. Skipping installer")
	return()
endif ()
include(${ROBVIS_CMAKE_DIRECTORY}/functions/util_creators.cmake)

# Create installer target
robvis_create_wix_installer(ReconstructMeQtInstaller)
add_dependencies(ReconstructMeQtInstaller ReconstructMeQt)

set(REME_SDK_ROOT ${REME_ROOT_DIRECTORY})	

# Collect necessary ReconstructMe SDK binaries
set(HEAT_OPTS -dr INSTALLDIR 
			  -cg ReconstructMeSDKRuntimeFiles 
			  -var var.SDKRuntimeFilesDirectory
			  -gg -scom -sreg -sfrag -srd)
	
robvis_wix_installer_heat_directory(
	ReconstructMeQtInstaller 
	"${REME_SDK_ROOT}/bin"
	CANDLE_SOURCES
	HEAT_OPTS)
	
	
	
# Collect configurations that are shipped with the SDK
set(HEAT_OPTS -dr INSTALLDIR 
			  -cg ReconstructMeSDKConfigFiles 
			  -var var.SDKConfigFilesDirectory
			  -gg -scom -sreg -sfrag)
	
robvis_wix_installer_heat_directory(
	ReconstructMeQtInstaller 
	"${REME_SDK_ROOT}/cfg"
	CANDLE_SOURCES
	HEAT_OPTS)

# Compile relevant files	
set(CANDLE_OPTS -dSDKRuntimeFilesDirectory="${REME_SDK_ROOT}/bin" -dSDKConfigFilesDirectory="${REME_SDK_ROOT}/cfg")
GET_TARGET_PROPERTY(RECONSTRUCTMEQT_EXE ReconstructMeQt RELEASE_LOCATION)
list(APPEND CANDLE_SOURCES Main.wxs QtRuntimes.wxs ReconstructMeQtRuntimes.wxs)

robvis_wix_installer_compile(
	ReconstructMeQtInstaller
	CANDLE_SOURCES
	CANDLE_OBJS
	CANDLE_OPTS)
	
# Generate final .msi
set(LIGHT_OPTS -ext WixUIExtension)
set(RECONSTRUCTMEQTINSTALLER_OUTPUT_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/ReconstructMe Setup-${RECONSTRUCTMEQT_VERSION_MAJOR}.${RECONSTRUCTMEQT_VERSION_MINOR}.${RECONSTRUCTMEQT_VERSION_BUILD}.msi")
robvis_wix_installer_link(
	ReconstructMeQtInstaller
	CANDLE_OBJS
	${RECONSTRUCTMEQTINSTALLER_OUTPUT_LOCATION}	
	LIGHT_OPTS)