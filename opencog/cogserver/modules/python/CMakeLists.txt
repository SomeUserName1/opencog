# ----------------------------------------
# Python module to be loaded by CogServer

MACRO(BUILD_PY_MODULE PY_V)
INCLUDE_DIRECTORIES(
   ${PYTHON${PY_V}_INCLUDE_DIRS}
   ${CMAKE_CURRENT_SOURCE_DIR}
   ${CMAKE_CURRENT_BINARY_DIR}
)

ADD_LIBRARY (PythonModule${PY_V} SHARED
	PythonModule.cc 
	PyMindAgent.cc
	PyRequest.cc
)

TARGET_LINK_LIBRARIES(PythonModule${PY_V}
	server
	agent_finder${PY_V}
	${ATOMSPACE_LIBRARIES}
	${PYTHON${PY_V}_LIBRARIES}
)
ADD_DEPENDENCIES(PythonModule${PY_V} agent_finder${PY_V})

INSTALL (TARGETS PythonModule${PY_V} DESTINATION
	"lib${LIB_DIR_SUFFIX}/opencog/modules")
ENDMACRO()

if (PY_2_BINDINGS)
	ADD_DEFINITIONS(-DCOMPLILING_WITH_PY2)
	SET(COMPILING_WITH_PY2 TRUE)
	BUILD_PY_MODULE(2) # add define to import propper agent_finder_api${PY_V}.h
endif()
if (PY_3_BINDINGS)
	SET(COMPILING_WITH_PY2 FALSE)
	BUILD_PY_MODULE(3)
endif()
REMOVE_DEFINITIONS(COMPILING_WITH_PY2 TRUE) # clean up