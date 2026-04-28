cmake_minimum_required( VERSION 3.15 )
project( Havoc )

##
## cmake sets
##
set( QT_VERSION 5 )
set( CMAKE_CXX_STANDARD 20 )
set( CMAKE_AUTOMOC ON )
set( CMAKE_AUTORCC ON )
set( CMAKE_AUTOUIC ON )
set( REQUIRED_LIBS Core Gui Widgets Network WebSockets Sql )
set( REQUIRED_LIBS_QUALIFIED Qt5::Core Qt5::Gui Qt5::Widgets Qt5::Network Qt5::WebSockets Qt5::Sql )
set( APP_ICON_RESOURCE_WINDOWS "data/Havoc.qrc" )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY .. )
set( CMAKE_INCLUDE_CURRENT_DIR ON )

##
## Ensure 3.10 is used if present.
## If not, modules will not work within the client.
##
set( Python_ADDITIONAL_VERSIONS 3.10 )

##
## import includes
##
include_directories( include )

##
## find qt packages
##
find_package( Qt${QT_VERSION} COMPONENTS ${REQUIRED_LIBS} REQUIRED )

##
## external libraries
##
# On Windows, require vcpkg-provided packages (CI uses vcpkg toolchain).
# On other platforms, try package managers quietly; vendored submodule headers
# serve as a fallback when the submodules have been checked out.
if(WIN32)
  find_package( nlohmann_json CONFIG REQUIRED )
  find_package( spdlog CONFIG REQUIRED )
  find_package( toml11 CONFIG REQUIRED )
else()
  find_package( nlohmann_json CONFIG QUIET )
  find_package( spdlog CONFIG QUIET )
  find_package( toml11 CONFIG QUIET )
  if(NOT TARGET nlohmann_json::nlohmann_json)
    include_directories( external/json/include )
  endif()
  if(NOT TARGET spdlog::spdlog)
    include_directories( external/spdlog/include )
  endif()
  if(NOT TARGET toml11::toml11)
    include_directories( external/toml )
  endif()
endif()

##
## Python support
##
# Use modern CMake Python package when available (especially on Windows, to provide Python.h).
# Fall back to legacy FindPythonLibs on older UNIX setups.
if(APPLE)
  find_package( Python 3 COMPONENTS Interpreter Development REQUIRED )
  set( PYTHON_MAJOR $ENV{Python_VERSION_MAJOR} )
  set( PYTHON_MINOR $ENV{Python_VERSION_MINOR} )
  set( PYTHONLIBS_VERSION_STRING ${Python_VERSION} )
  set( PYTHON_INCLUDE_DIR ${Python_INCLUDE_DIRS} )
  set( PYTHON_LIBRARIES ${Python_LIBRARIES} )
  message( "Apple - Using Python:${Python_VERSION_MAJOR} - Libraries:${PYTHON_LIBRARIES} - includeDirs: ${PYTHON_INCLUDE_DIR}" )
elseif(WIN32)
  find_package( Python 3.10 COMPONENTS Development REQUIRED )
  set( PYTHON_LIBRARIES Python::Python )
elseif(UNIX)
  find_package( PythonLibs 3 REQUIRED )
else()
  set( PYTHONLIBS_VERSION_STRING $ENV{PY_VERSION} )
endif()

if(APPLE)
    execute_process( COMMAND brew --prefix OUTPUT_VARIABLE BREW_PREFIX ) #this because brew install location differs Intel/Apple Silicon macs
    string( STRIP ${BREW_PREFIX} BREW_PREFIX ) #for some reason this happens: https://gitlab.kitware.com/cmake/cmake/-/issues/22404
    include_directories( "${BREW_PREFIX}/bin/python3.10" )
    include_directories( "${BREW_PREFIX}/Frameworks/Python.framework/Headers" )
elseif(WIN32)
    include_directories( ${Python_INCLUDE_DIRS} )
elseif(UNIX)
    include_directories( ${PYTHON_INCLUDE_DIRS} )
endif()

set( HAVOC_INCLUDE

        # Misc
        include/Havoc/Connector.hpp
        include/Havoc/DBManager/DBManager.hpp
        include/Havoc/DemonCmdDispatch.h
        include/Havoc/Packager.hpp
        include/Havoc/Havoc.hpp
        include/Havoc/Service.hpp
        include/Havoc/CmdLine.hpp

        # Python Api
        include/Havoc/PythonApi/PythonApi.h
        include/Havoc/PythonApi/Event.h
        include/Havoc/PythonApi/HavocUi.h
        include/Havoc/PythonApi/PyAgentClass.hpp
        include/Havoc/PythonApi/UI/PyDialogClass.hpp
        include/Havoc/PythonApi/UI/PyLoggerClass.hpp
        include/Havoc/PythonApi/UI/PyTreeClass.hpp
        include/Havoc/PythonApi/UI/PyWidgetClass.hpp

        # Dialogs
        include/UserInterface/Dialogs/Payload.hpp
        include/UserInterface/Dialogs/About.hpp
        include/UserInterface/Dialogs/Connect.hpp
        include/UserInterface/Dialogs/Listener.hpp

        # small widgets
        include/UserInterface/SmallWidgets/EventViewer.hpp
        include/UserInterface/SmallWidgets/EventViewer.hpp

        # widgets
        include/UserInterface/Widgets/Chat.hpp
        include/UserInterface/Widgets/DemonInteracted.h
        include/UserInterface/Widgets/ListenerTable.hpp
        include/UserInterface/Widgets/ProcessList.hpp
        include/UserInterface/Widgets/PythonScript.hpp
        include/UserInterface/Widgets/SessionTable.hpp
        include/UserInterface/Widgets/SessionGraph.hpp
        include/UserInterface/Widgets/TeamserverTabSession.h
        include/UserInterface/Widgets/ScriptManager.h
        include/UserInterface/Widgets/LootWidget.h
        include/UserInterface/Widgets/FileBrowser.hpp
        include/UserInterface/Widgets/Teamserver.hpp
        include/UserInterface/Widgets/Store.hpp

        include/UserInterface/HavocUI.hpp
)

# src for UI
set(
        HAVOC_UI

        src/UserInterface/HavocUi.cc
        src/global.cc

        # Dialogs
        src/UserInterface/Dialogs/About.cc
        src/UserInterface/Dialogs/Connect.cc
        src/UserInterface/Dialogs/Listener.cc
        src/UserInterface/Dialogs/Payload.cc

        # Widgets
        src/UserInterface/Widgets/SessionTable.cc
        src/UserInterface/Widgets/SessionGraph.cc
        src/UserInterface/Widgets/Chat.cc
        src/UserInterface/Widgets/ListenersTable.cc
        src/UserInterface/Widgets/DemonInteracted.cc
        src/UserInterface/Widgets/TeamserverTabSession.cc
        src/UserInterface/Widgets/PythonScript.cc
        src/UserInterface/Widgets/ScriptManager.cc
        src/UserInterface/Widgets/LootWidget.cc
        src/UserInterface/Widgets/FileBrowser.cc
        src/UserInterface/Widgets/Teamserver.cc
        src/UserInterface/Widgets/Store.cc
        src/UserInterface/Widgets/ProcessList.cc

        # SmallWidgets
        src/UserInterface/SmallWidgets/EventViewer.cc
)

set(
        HAVOC_UTIL

        src/Util/ColorText.cpp
        src/Util/Base64.cpp
        src/Util/Base.cpp
)

# src for Havoc
set( HAVOC_SOURCE
        src/Main.cc

        src/Havoc/Packager.cc
        src/Havoc/Connector.cc
        src/Havoc/Service.cc

        src/Havoc/DBManger/DBManager.cc
        src/Havoc/DBManger/Teamserver.cc
        src/Havoc/DBManger/Scripts.cc

        src/Havoc/Demon/ConsoleInput.cc
        src/Havoc/Demon/CommandSend.cc
        src/Havoc/Demon/CommandOutput.cc
        src/Havoc/Demon/Commands.cc

        src/Havoc/PythonApi/PythonApi.cc
        src/Havoc/PythonApi/Havoc.cc
        src/Havoc/PythonApi/HavocUi.cc
        src/Havoc/PythonApi/UI/PyDialogClass.cc
        src/Havoc/PythonApi/UI/PyLoggerClass.cc
        src/Havoc/PythonApi/UI/PyTreeClass.cc
        src/Havoc/PythonApi/UI/PyWidgetClass.cc
        src/Havoc/PythonApi/PyDemonClass.cc
        src/Havoc/PythonApi/Event.cc
        src/Havoc/PythonApi/PyAgentClass.cc

        src/Havoc/Havoc.cc

    ${HAVOC_UI}
)

add_executable( ${PROJECT_NAME} ${HAVOC_INCLUDE} ${HAVOC_SOURCE} ${HAVOC_UTIL} ${APP_ICON_RESOURCE_WINDOWS} data/Havoc.rc )
add_definitions( -DQT_NO_DEBUG_OUTPUT )

target_link_libraries(
    ${PROJECT_NAME}
    ${REQUIRED_LIBS_QUALIFIED}
    ${PYTHON_LIBRARIES}
)

# Link optional external deps when found (e.g., via vcpkg)
if(TARGET nlohmann_json::nlohmann_json)
  target_link_libraries(${PROJECT_NAME} nlohmann_json::nlohmann_json)
endif()
if(TARGET spdlog::spdlog)
  target_link_libraries(${PROJECT_NAME} spdlog::spdlog)
endif()
if(TARGET toml11::toml11)
  target_link_libraries(${PROJECT_NAME} toml11::toml11)
endif()
