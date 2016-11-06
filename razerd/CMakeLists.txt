include(${razer_SOURCE_DIR}/scripts/cmake.global)
include(GNUInstallDirs)

# Add an RPATH to the installed binary.
# Do not build with the install RPATH, use the local path.
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
# Include a specific RPATH when installing the binary.
# This allows it install the librazer outside of the normal system search path.
# TODO: Determine a better way to do this. It must match where the librazer.so
# will be installed.
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")

add_executable(razerd
	       razerd.c)

set_target_properties(razerd PROPERTIES COMPILE_FLAGS ${GENERIC_COMPILE_FLAGS})

include_directories("${razer_SOURCE_DIR}/librazer")

target_link_libraries(razerd razer)
install(TARGETS razerd DESTINATION bin)

if (NOT DEFINED ENV{RPM_BUILD_ROOT} AND NOT DEFINED ENV{RAZERCFG_PKG_BUILD})
	if(SYSTEMD_UNIT_DIR)
		install_exec_cmd("systemctl --system daemon-reload"
				 "If you use systemd, please reload systemd manually or reboot the system")
		install_exec_cmd("systemctl --system --force enable razerd.service"
				 "If you use systemd, enable razerd.service manually")
	endif(SYSTEMD_UNIT_DIR)
	if(UDEV_DIR)
		install_exec_cmd("udevadm control --reload-rules"
				 "Please reload udev manually or reboot the system")
	endif(UDEV_DIR)
endif (NOT DEFINED ENV{RPM_BUILD_ROOT} AND NOT DEFINED ENV{RAZERCFG_PKG_BUILD})
