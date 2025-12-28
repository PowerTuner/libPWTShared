cmake_minimum_required(VERSION 3.23)

project(PWTShared VERSION 1.1 DESCRIPTION "Shared library for PowerTuner clients and daemon" LANGUAGES CXX)

set(PROJECT_AUTHOR "kylon")
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)

find_package(Qt6 6.10 REQUIRED COMPONENTS Core Network)

configure_file(pwtShared/Resources/version.h.in ${CMAKE_CURRENT_SOURCE_DIR}/version.h)

set(PROJECT_SOURCES
	pwtShared/sharedExport.h
	pwtShared/DaemonSettings.h
	pwtShared/DaemonSettings.cpp
	pwtShared/Utils.h
	pwtShared/Utils.cpp

	pwtShared/Include/LogLevel.h
	pwtShared/Include/Feature.h
	pwtShared/Include/Features.h
	pwtShared/Include/SystemInfo.h
	pwtShared/Include/DynamicSystemInfo.h
	pwtShared/Include/OSType.h
	pwtShared/Include/DaemonCMD.h
	pwtShared/Include/DaemonError.h
	pwtShared/Include/PacketError.h

	pwtShared/Include/Types/MinMax.h
	pwtShared/Include/Types/ROData.h
	pwtShared/Include/Types/RWData.h

	pwtShared/Include/OS/Linux/CPUFrequencyLimits.h
	pwtShared/Include/OS/Linux/CPUScalingAvailableGovernors.h
	pwtShared/Include/OS/Linux/BlockDeviceQueSched.h
	pwtShared/Include/OS/Linux/MiscPMDevice.h
	pwtShared/Include/OS/Linux/Intel/GPURPSLimits.h
	pwtShared/Include/OS/Linux/AMD/AMDPStateData.h
	pwtShared/Include/OS/Linux/AMD/GPUDPMForcePerfLevel.h
	pwtShared/Include/OS/Linux/AMD/GPUODRanges.h

	pwtShared/Include/OS/Windows/PowerScheme.h
	pwtShared/Include/OS/Windows/PowerSchemesData.h
	pwtShared/Include/OS/Windows/PowerSchemeSetting.h
	pwtShared/Include/OS/Windows/PowerSchemeSettingData.h
	pwtShared/Include/OS/Windows/PowerSettingValue.h

	pwtShared/Include/CPU/Intel/FIVRControlUV.h
	pwtShared/Include/CPU/Intel/HWPCapabilities.h
	pwtShared/Include/CPU/Intel/HWPRequest.h
	pwtShared/Include/CPU/Intel/HWPRequestPkg.h
	pwtShared/Include/CPU/Intel/MiscProcFeatures.h
	pwtShared/Include/CPU/Intel/MiscPwrMgmt.h
	pwtShared/Include/CPU/Intel/PkgPowerLimit.h
	pwtShared/Include/CPU/Intel/PowerCtl.h
	pwtShared/Include/CPU/Intel/PP1CurrentConfig.h
	pwtShared/Include/CPU/Intel/TurboPowerCurrentLimit.h
	pwtShared/Include/CPU/Intel/TurboRatioLimit.h
	pwtShared/Include/CPU/Intel/VRCurrentConfig.h
	pwtShared/Include/CPU/Intel/PkgCstConfigControl.h
	pwtShared/Include/CPU/Intel/TemperatureTarget.h
	pwtShared/Include/CPU/Intel/PkgThermalStatusInfo.h
	pwtShared/Include/CPU/Intel/MCHBARPkgRaplLimit.h

	pwtShared/Include/CPU/AMD/CPPCCapability1.h
	pwtShared/Include/CPU/AMD/CPPCRequest.h
	pwtShared/Include/CPU/AMD/PStateCurrentLimit.h

	pwtShared/Include/CPU/CpuInfo.h
	pwtShared/Include/CPU/CPUVendor.h

	pwtShared/Include/GPU/GpuInfo.h
	pwtShared/Include/GPU/GPUVendor.h

	pwtShared/Include/Data/FAN/FanData.h

	pwtShared/Include/Packets/DeviceInfoPacket.h
	pwtShared/Include/Packets/ClientPacket.h
	pwtShared/Include/Packets/DaemonPacket.h

	pwtShared/Include/Data/OS/Linux/LinuxData.h
	pwtShared/Include/Data/OS/Linux/LinuxThreadData.h
	pwtShared/Include/Data/OS/Linux/LinuxAMDData.h
	pwtShared/Include/Data/OS/Linux/LinuxAMDThreadData.h
	pwtShared/Include/Data/OS/Linux/LinuxIntelGPUData.h
	pwtShared/Include/Data/OS/Linux/LinuxAMDGPUData.h

	pwtShared/Include/Data/OS/Windows/WindowsData.h

	pwtShared/Include/Data/Vendor/Intel/IntelData.h
	pwtShared/Include/Data/Vendor/Intel/IntelCoreData.h
	pwtShared/Include/Data/Vendor/Intel/IntelThreadData.h

	pwtShared/Include/Data/Vendor/AMD/AMDData.h
	pwtShared/Include/Data/Vendor/AMD/AMDCoreData.h
	pwtShared/Include/Data/Vendor/AMD/AMDThreadData.h
)

if (WIN32)
	configure_file(pwtShared/Resources/Windows/win.rc.in ${CMAKE_CURRENT_SOURCE_DIR}/win.rc)

	list(APPEND PROJECT_SOURCES
		win.rc
	)
endif ()

add_library(${PROJECT_NAME} SHARED ${PROJECT_SOURCES})
add_library("PWT::Shared" ALIAS ${PROJECT_NAME})

target_link_libraries(${PROJECT_NAME} PRIVATE Qt::Core Qt::Network)
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

if (WIN32)
	target_compile_definitions(${PROJECT_NAME} PUBLIC PWTSHARED_LIBRARY)
else ()
	target_compile_definitions(${PROJECT_NAME} PRIVATE PWTSHARED_LIBRARY)
endif ()

include(CheckIPOSupported)
check_ipo_supported(RESULT has_ipo OUTPUT ipo_error)
if (has_ipo)
	message(STATUS "${PROJECT_NAME}: IPO/LTO enabled")
	set_property(TARGET ${PROJECT_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
endif ()

include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

set_target_properties(${PROJECT_NAME} PROPERTIES
	VERSION ${PROJECT_VERSION}
	SOVERSION ${PROJECT_VERSION}
	CXX_VISIBLITY_PRESET hidden
	VISIBILITY_INLINE_HIDDEN TRUE
)
