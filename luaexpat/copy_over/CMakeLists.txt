CMAKE_MINIMUM_REQUIRED(VERSION 3.22)

PROJECT("com.github.lunarmodules-luaexpat")

ENABLE_TESTING()
FIND_PACKAGE(Python3 REQUIRED COMPONENTS Interpreter)
SET(PYTHON_EXECUTABLE "${Python3_EXECUTABLE}")

FIND_PACKAGE(lua5.4 CONFIG REQUIRED)
GET_TARGET_PROPERTY(LUA_INCLUDE_DIR org.lua.lua::lua5.4 INTERFACE_INCLUDE_DIRECTORIES)

# Build the Lua module.
ADD_LIBRARY(TARGET_luaexpat MODULE
            src/lxplib.c)
TARGET_INCLUDE_DIRECTORIES(TARGET_luaexpat
                           PRIVATE ${LUA_INCLUDE_DIR})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_luaexpat
	                      PRIVATE lua5.4)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Add the expat includes and library.
TARGET_INCLUDE_DIRECTORIES(TARGET_luaexpat
                           PRIVATE ${LIBEXPAT_INCLUDE_DIRECTORIES})
TARGET_LINK_LIBRARIES(TARGET_luaexpat
                      PRIVATE ${LIBEXPAT_LIBRARY})

# On MinGW link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_luaexpat PROPERTY LINK_FLAGS "-static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "lxp" without any prefix like "lib".
SET_TARGET_PROPERTIES(TARGET_luaexpat PROPERTIES PREFIX "" OUTPUT_NAME "lxp")


#-----------------------------------------------------------------------------
#
# Install the LUA modules.
#
INSTALL(TARGETS TARGET_luaexpat
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES})
INSTALL(FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/lxp/lom.lua ${CMAKE_CURRENT_SOURCE_DIR}/src/lxp/threat.lua ${CMAKE_CURRENT_SOURCE_DIR}/src/lxp/totable.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS}/lxp/)

#-----------------------------------------------------------------------------
#
# Add tests for the modules.
#
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME luaexpat_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 -u lua5.4 $<TARGET_FILE:TARGET_luaexpat>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
