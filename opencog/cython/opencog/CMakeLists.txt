
# Need to use -fno-strict-aliasing when compiling cython code, in order
# to avoid nasty compiler warnings about aliasing.  Cython explicitly
# performs aliasing, in order to emulate python object inheritance.
# See, for example,
# https://groups.google.com/forum/#!topic/cython-users/JV1-KvIUeIg
#

MACRO(BUILD_CYTHON PY_BUILD_VERSION)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-strict-aliasing")

	INCLUDE_DIRECTORIES(
			${ATOMSPACE_INCLUDE_DIR}
			${PYTHON${PY_BUILD_VERSION}_INCLUDE_DIRS}
			${CMAKE_CURRENT_SOURCE_DIR}
			${CMAKE_CURRENT_BINARY_DIR}
	)

	# Cythonizes one .pyx file into a .cpp file
	# Additional arguments are dependencies
	MACRO(CYTHON_ADD_MODULE_PYX name)
		IF(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pxd)
			SET(DEPENDS ${name}.pyx ${name}.pxd)
		ENDIF(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pxd)
		# Allow the user to specify dependencies as optional arguments
		SET(DEPENDS ${DEPENDS} ${ARGN})
		ADD_CUSTOM_COMMAND(
				OUTPUT ${name}${PY_BUILD_VERSION}.cpp
				COMMAND ${CYTHON_EXECUTABLE}
				ARGS -f
					-${PY_BUILD_VERSION}
					-I ${PROJECT_BINARY_DIR}
					-I ${ATOMSPACE_INCLUDE_DIR}/opencog/cython
					-o ${name}${PY_BUILD_VERSION}.cpp
					--cplus ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pyx
				DEPENDS ${DEPENDS}
				COMMENT "Cythonizing ${name}.pyx using Python ${PY_BUILD_VERSION} Libs")
	ENDMACRO(CYTHON_ADD_MODULE_PYX)

	# Use this as a guide:
	# https://github.com/OpenKinect/libfreenect/blob/master/wrappers/python/CMakeLists.txt

	IF (HAVE_ATOMSPACE)
		############################## type constructors #####################
		CYTHON_ADD_MODULE_PYX(cogserver_type_constructors)

		list(APPEND ADDITIONAL_MAKE_CLEAN_FILES
				"cogserver_type_constructors${PY_BUILD_VERSION}.cpp")

		ADD_LIBRARY(cogserver_type_constructors${PY_BUILD_VERSION} SHARED
			cogserver_type_constructors${PY_BUILD_VERSION}.cpp
		)

		# XXX FIXME Cython should not static link the atom-types, but
		# but should instead dynamic-load them, as needed.
		ADD_DEPENDENCIES(cogserver_type_constructors${PY_BUILD_VERSION} nlp_atom_types)
		ADD_DEPENDENCIES(cogserver_type_constructors${PY_BUILD_VERSION} attention_atom_types)
		ADD_DEPENDENCIES(cogserver_type_constructors${PY_BUILD_VERSION} spacetime_atom_types)
		ADD_DEPENDENCIES(cogserver_type_constructors${PY_BUILD_VERSION} patternminer_atom_types)

		TARGET_LINK_LIBRARIES(cogserver_type_constructors${PY_BUILD_VERSION}
			attention-types
			nlp-types
			spacetime-types
			patternminer-types
			${ATOMSPACE_LIBRARIES}
			${PYTHON${PY_BUILD_VERSION}_LIBRARIES}
		)

		SET_TARGET_PROPERTIES(cogserver_type_constructors${PY_BUILD_VERSION} PROPERTIES
			PREFIX ""
			OUTPUT_NAME cogserver_type_constructors
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_BUILD_VERSION}
		)
		INSTALL (TARGETS cogserver_type_constructors${PY_BUILD_VERSION}
				LIBRARY DESTINATION opencog
				DESTINATION ${PYTHON${PY_BUILD_VERSION}_SITE_PACKAGES})

	ENDIF (HAVE_ATOMSPACE)


	IF (HAVE_SERVER)
		############################ cogserver #########################
		CYTHON_ADD_MODULE_PYX(cogserver
			"${ATOMSPACE_INCLUDE_DIR}/opencog/cython/opencog/atomspace.pxd"
			"request.pyx" "agent.pyx" "agent_finder_types.h"
			"../../cogserver/server/Agent.h" "../../cogserver/server/Request.h"
		)

		list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "cogserver${PY_BUILD_VERSION}.cpp")

		# opencog.cogserver Python bindings
		ADD_LIBRARY(cogserver_cython${PY_BUILD_VERSION} SHARED
			cogserver${PY_BUILD_VERSION}.cpp
		)

		TARGET_LINK_LIBRARIES(cogserver_cython${PY_BUILD_VERSION}
			server
			${ATOMSPACE_LIBRARIES}
			${COGUTIL_LIBRARY}
			${PYTHON${PY_BUILD_VERSION}_LIBRARIES}
		)

		SET_TARGET_PROPERTIES(cogserver_cython${PY_BUILD_VERSION} PROPERTIES
			PREFIX ""
			OUTPUT_NAME cogserver
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_BUILD_VERSION})

		INSTALL (TARGETS cogserver_cython${PY_BUILD_VERSION}
				LIBRARY DESTINATION opencog
				DESTINATION ${PYTHON${PY_BUILD_VERSION}_SITE_PACKAGES})

		####################### agent finder ########################
		CYTHON_ADD_MODULE_PYX(agent_finder
			"${ATOMSPACE_INCLUDE_DIR}/opencog/cython/opencog/atomspace.pxd"
			"cogserver.pxd"
			"../../cogserver/server/Agent.h" "../../cogserver/server/Request.h"
		)

		list(APPEND ADDITIONAL_MAKE_CLEAN_FILES
			"agent_finder${PY_BUILD_VERSION}.cpp" "agent_finder_api${PY_BUILD_VERSION}.h")

		# utility library for PythonModule
		ADD_LIBRARY(agent_finder${PY_BUILD_VERSION} SHARED
			agent_finder${PY_BUILD_VERSION}.cpp
		)

		TARGET_LINK_LIBRARIES(agent_finder${PY_BUILD_VERSION}
			${ATOMSPACE_LIBRARIES}
			${PYTHON${PY_BUILD_VERSION}_LIBRARIES}
			${Boost_THREAD_LIBRARY}
			${Boost_SYSTEM_LIBRARY}
		)

		SET_TARGET_PROPERTIES(agent_finder${PY_BUILD_VERSION} PROPERTIES
			PREFIX ""
			OUTPUT_NAME agent_finder
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_BUILD_VERSION})

		INSTALL (TARGETS agent_finder${PY_BUILD_VERSION}
				LIBRARY DESTINATION opencog
				DESTINATION ${PYTHON${PY_BUILD_VERSION}_SITE_PACKAGES})
	ENDIF(HAVE_SERVER)

	IF (HAVE_STATISTICS)
		CYTHON_ADD_MODULE_PYX(statistics
			"../../learning/statistics/DataProvider.h"
			"../../learning/statistics/Probability.h"
			"../../learning/statistics/Entropy.h"
			"../../learning/statistics/InteractionInformation.h"
		)

		list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "statistics${PY_BUILD_VERSION}.cpp")

		# opencog.statistics Python bindings
		ADD_LIBRARY(statistics_cython${PY_BUILD_VERSION} SHARED
			statistics${PY_BUILD_VERSION}.cpp
		)

		TARGET_LINK_LIBRARIES(statistics_cython${PY_BUILD_VERSION}
			statistics
			${PYTHON${PY_BUILD_VERSION}_LIBRARIES}
		)

		SET_TARGET_PROPERTIES(statistics_cython${PY_BUILD_VERSION} PROPERTIES
			PREFIX ""
			OUTPUT_NAME statistics
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_BUILD_VERSION})

		INSTALL (TARGETS statistics_cython${PY_BUILD_VERSION}
				LIBRARY DESTINATION opencog
				DESTINATION ${PYTHON${PY_BUILD_VERSION}_SITE_PACKAGES})
	ENDIF (HAVE_STATISTICS)

	INSTALL (FILES
				__init__.py
			DESTINATION
				${PYTHON${PY_BUILD_VERSION}_SITE_PACKAGES})
ENDMACRO()

if (PY_2_BINDINGS)
	BUILD_CYTHON(2)
endif()
if (PY_3_BINDINGS)
	BUILD_CYTHON(3)
endif()

#
## How to Cython the .pyx file
#add_custom_command(OUTPUT freenect${Python_BUILD_VERSION}.c
#		COMMAND
#		${CYTHON_EXECUTABLE}
#		-${Python_BUILD_VERSION}
#		-o freenect${Python_BUILD_VERSION}.c
#		"${CMAKE_CURRENT_SOURCE_DIR}/freenect.pyx")
#list(APPEND ADDITIONAL_MAKE_CLEAN_FILES freenect${Python_BUILD_VERSION}.c)
#
## Compile the extension

#add_library(cython${Python_BUILD_VERSION}_freenect MODULE freenect${Python_BUILD_VERSION}.c)

#set_target_properties(cython${Python_BUILD_VERSION}_freenect PROPERTIES
#		PREFIX ""
#		OUTPUT_NAME "freenect"
#		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_BUILD_VERSION})

#target_link_libraries(cython${Python_BUILD_VERSION}_freenect
#		freenect_sync
#		${PYTHON${Python_BUILD_VERSION}_LIBRARIES})

#target_include_directories(cython${Python_BUILD_VERSION}_freenect PRIVATE
#		${PYTHON${Python_BUILD_VERSION}_INCLUDE_PATH}
#		../c_sync/
#		${PYTHON${Python_BUILD_VERSION}_NUMPY_INCLUDE_PATH})
#
## Install the extension

#install(TARGETS cython${Python_BUILD_VERSION}_freenect
#		DESTINATION ${PYTHON${Python_BUILD_VERSION}_SITE_PACKAGES})
#
## TODO: decide on what to do with demo_ scripts and were to install
##       them