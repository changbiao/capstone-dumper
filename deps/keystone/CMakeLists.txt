
include(ExternalProject)

# Keystone requires python3, so we first check whether python3 is installed as
# "python3", and if not, we suppose that `python` runs python3.
# Python is used to generate LLVMBuild.cmake file. Python2 generates this file
# with UNIX paths, which do not work. Python3 generates proper Windows paths.
find_program(PYTHON_EXECUTABLE "python3")
if(NOT PYTHON_EXECUTABLE)
	find_program(PYTHON_EXECUTABLE "python")
endif()

ExternalProject_Add(keystone-project
	GIT_REPOSITORY https://github.com/keystone-engine/keystone.git
	GIT_TAG d19579217c0ac71e2d63deb389f29fb23e5a9118
	CMAKE_ARGS
		# This does not work on MSVC, but may be useful on Linux.
		-DCMAKE_BUILD_TYPE=Release
		# Force python version set in this cmake file.
		-DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
		-DKEYSTONE_BUILD_STATIC_RUNTIME=OFF
	# Disable the update step.
	UPDATE_COMMAND ""
	# Disable the install step.
	INSTALL_COMMAND ""
	LOG_DOWNLOAD ON
	LOG_CONFIGURE ON
	LOG_BUILD ON
)

ExternalProject_Get_Property(keystone-project source_dir)
ExternalProject_Get_Property(keystone-project binary_dir)

add_library(keystone INTERFACE)
add_dependencies(keystone keystone-project)
target_include_directories(keystone SYSTEM INTERFACE ${source_dir}/include)
target_link_libraries(keystone INTERFACE debug ${binary_dir}/llvm/lib/${DEBUG_DIR}${CMAKE_FIND_LIBRARY_PREFIXES}keystone${CMAKE_STATIC_LIBRARY_SUFFIX})
target_link_libraries(keystone INTERFACE optimized ${binary_dir}/llvm/lib/${RELEASE_DIR}${CMAKE_FIND_LIBRARY_PREFIXES}keystone${CMAKE_STATIC_LIBRARY_SUFFIX})
