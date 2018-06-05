INCLUDE_DIRECTORIES(
	${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_BINARY_DIR}
)

MACRO(BUILD_PY_SHELL PY_V)
	INCLUDE_DIRECTORIES(
		${PYTHON${PY_V}_INCLUDE_PATH}
	)

	ADD_LIBRARY (py-shell${PY_V} SHARED
		GenericShell
		PythonShell
		PythonShellModule
	)

	TARGET_LINK_LIBRARIES(py-shell${PY_V}
		server
		${ATOMSPACE_smob_LIBRARY}
		${ATOMSPACE_LIBRARY}
		${COGUTIL_LIBRARY}
		${Boost_SYSTEM_LIBRARY}
		${PYTHON${PY_V}_LIBRARIES}
	)
	INSTALL (TARGETS py-shell${PY_V}
			LIBRARY DESTINATION "lib${LIB_DIR_SUFFIX}/opencog/modules")
ENDMACRO()

IF (PY_2_BINDINGS)
    BUILD_PY_SHELL(2)
ENDIF()
IF (PY_3_BINDINGS)
    BUILD_PY_SHELL(3)
ENDIF()


ADD_LIBRARY (scheme-shell SHARED
	GenericShell
	SchemeShell
	SchemeShellModule
)

TARGET_LINK_LIBRARIES(scheme-shell
	server # setting the guile atomspace needs atomspace of server.
	${ATOMSPACE_smob_LIBRARY}
	${ATOMSPACE_LIBRARY}
	${COGUTIL_LIBRARY}
	${Boost_SYSTEM_LIBRARY}
)

INSTALL (TARGETS scheme-shell
	LIBRARY DESTINATION "lib${LIB_DIR_SUFFIX}/opencog/modules")

INSTALL (FILES
	GenericShell.h
	DESTINATION "include/${PROJECT_NAME}/cogserver/shell"
)
