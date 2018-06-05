MACRO(BUILD_PY_RESTAPI PY_V)
	INSTALL (FILES
		web/__init__.py
		DESTINATION "${PYTHON${PY_V}_SITE_PACKAGES}/web")

	INSTALL (FILES
		web/api/apiatomcollection.py
		web/api/apimain.py
		web/api/apischeme.py
		web/api/apishell.py
		web/api/apitypes.py
		web/api/__init__.py
		web/api/mappers.py
		web/api/restapi.py
		web/api/utilities.py
		DESTINATION "${PYTHON${PY_V}_SITE_PACKAGES}/web/api")

	INSTALL (FILES
		web/socketio/atomspace_publisher.py
		web/socketio/__init__.py
		DESTINATION "${PYTHON${PY_V}_SITE_PACKAGES}/web/socketio")
ENDMACRO()

if (PY_2_BINDINGS)
	BUILD_PY_RESTAPI(2)
endif()
if (PY_3_BINDINGS)
	BUILD_PY_RESTAPI(3)
endif()


# TODO PORT WITH __FUTURE__