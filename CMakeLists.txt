CMAKE_MINIMUM_REQUIRED(VERSION 3.0)
PROJECT(GilViewer)

# SET(CMAKE_BUILD_TYPE Debug)

INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_SOURCE_DIR}/src)

#set(BUILD_SHARED_LIBS ON)
IF(WIN32)
  SET(BUILD_SHARED_LIBS OFF CACHE BOOL "build lib shared/static")
ELSE(WIN32)
  SET(BUILD_SHARED_LIBS ON CACHE BOOL "build lib shared/static")
ENDIF(WIN32)

MESSAGE(STATUS "Building in " ${CMAKE_BUILD_TYPE} " mode")
 
IF(UNIX) 
	add_definitions(-Wall -fPIC)
    SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D_GLIBCXX_DEBUG" CACHE STRING "Flags used by the compiler during debug builds." FORCE)
    #add_definitions(-pedantic)
ENDIF(UNIX)

SET(CMAKE_CONFIG_DIR cmake)

####
#### On recherche tous les fichiers ".h", ".hpp", ".hxx" et "*.inl"
####
SET(SRC_DIR src/GilViewer)
FILE( GLOB ALL_CONFIG_HEADER_FILES ${SRC_DIR}/config/*.hpp )
FILE( GLOB ALL_CONVENIENT_HEADER_FILES ${SRC_DIR}/convenient/*.hpp )
FILE( GLOB_RECURSE ALL_GUI_HEADER_FILES ${SRC_DIR}/gui/*.hpp ${SRC_DIR}/gui/*.fbp )
FILE( GLOB ALL_IO_HEADER_FILES ${SRC_DIR}/io/*.hpp )
FILE( GLOB ALL_LAYERS_HEADER_FILES ${SRC_DIR}/layers/*.hpp )
FILE( GLOB ALL_TOOLS_HEADER_FILES ${SRC_DIR}/tools/*.hpp )
FILE( GLOB ALL_PLUGINS_HEADER_FILES ${SRC_DIR}/plugins/*.hpp )
FILE( GLOB ALL_RESOURCES_HEADER_FILES ${SRC_DIR}/gui/resources/*.xpm )

# Creates subdirectories in project for MSVC
if(WIN32 AND MSVC)
	source_group( "Header Files\\config" FILES ${ALL_CONFIG_HEADER_FILES} )
        source_group( "Header Files\\convenient" FILES ${ALL_CONVENIENT_HEADER_FILES} )
        source_group( "Header Files\\gui" FILES ${ALL_GUI_HEADER_FILES} )
        source_group( "Header Files\\gui\\resources" FILES ${ALL_RESOURCES_HEADER_FILES} )
	source_group( "Header Files\\io" FILES ${ALL_IO_HEADER_FILES} )
	source_group( "Header Files\\layers" FILES ${ALL_LAYERS_HEADER_FILES} )
	source_group( "Header Files\\tools" FILES ${ALL_TOOLS_HEADER_FILES} )
	source_group( "Header Files\\plugins" FILES ${ALL_PLUGINS_HEADER_FILES} )
endif()

# Une variable regroupant tous les headers
SET( ALL_VIEWER_HEADERS ${ALL_CONFIG_HEADER_FILES}
                        ${ALL_CONVENIENT_HEADER_FILES}
                        ${ALL_GUI_HEADER_FILES}
                        ${ALL_IO_HEADER_FILES}
                        ${ALL_LAYERS_HEADER_FILES}
                        ${ALL_TOOLS_HEADER_FILES}
                        ${ALL_PLUGINS_HEADER_FILES} )

#### Definition des dossiers contenant les sources
#### ATTENTION : toutes les sources de ces dossiers seront incluses
#### ATTENTION : si un fichier est ajoute dans un de ces dossiers mais que le CMakeLists.txt n'est pas modifie, il faut relancer le cmake
####  a la main avant de compiler par make (ca, c'est pas vrai sous windows ! en fait je n'en suis pas si sur ...)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/config SRC_CONFIG)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/convenient SRC_CONVENIENT)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/gui SRC_GUI)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/gui/wxfb SRC_GUI_WXFB)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/io SRC_IO)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/layers SRC_CALQUES)
AUX_SOURCE_DIRECTORY(${SRC_DIR}/tools SRC_OUTILS) 
AUX_SOURCE_DIRECTORY(${SRC_DIR}/plugins SRC_PLUGINS)

# Creates subdirectories in project for MSVC
if(WIN32 AND MSVC)
	source_group( "Source Files\\config" FILES ${SRC_CONFIG} )
	source_group( "Source Files\\convenient" FILES ${SRC_CONVENIENT} )
	source_group( "Source Files\\gui" FILES ${SRC_GUI} )
	source_group( "Source Files\\gui\\wxfb" FILES ${SRC_GUI_WXFB} )
	source_group( "Source Files\\gui\\resources" FILES ${SRC_DIR}/gui/resources/resources.cpp ${SRC_DIR}/gui/resources/resources.cpp ${SRC_DIR}/gui/resources/resources.xrc )
	source_group( "Source Files\\io" FILES ${SRC_IO} )
	source_group( "Source Files\\layers" FILES ${SRC_CALQUES} )
	source_group( "Source Files\\tools" FILES ${SRC_OUTILS} )
	source_group( "Source Files\\plugins" FILES ${SRC_PLUGINS} )
endif()

# Une variable regroupant toutes les sources ...
SET( ALL_VIEWER_SOURCES ${SRC_CONFIG} ${SRC_CONVENIENT} ${SRC_GUI} ${SRC_GUI_WXFB} ${SRC_IO} ${SRC_CALQUES} ${SRC_OUTILS} ${SRC_PLUGINS} )

if(WIN32)
    if(MSVC)
        # Remove gil iterator related warnings...
        add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
        add_definitions(-D_SCL_SECURE_NO_WARNINGS)
        #Visual C++, 32-bit, version 6.0         1200
        #Visual C++, 32-bit, version .net 2002   1300
        #Visual C++, 32-bit, version .net 2003   1310
        #Visual C++, 32-bit, version 2005        1400 (vc80)
        #Visual C++, 32-bit, version 2005 SP1    14?? (vc80_sp1)
        #Visual C++, 32-bit, version 2008        1500 (vc90)
        #MSVC90  (Visual C++ 2008 SP1) MSVC_VERSION=1500 (Compiler  Version 15.00.30729.01)
        #MSVC100 (Visual C++ 2010) MSVC_VERSION=1600 (Compiler  Version 16.00.30319.01)
		#MESSAGE(STATUS " MSVC 71  ${MSVC71}" )
		if(MSVC_VERSION)
		    message(STATUS " MSVC_VERSION = ${MSVC_VERSION}" )
			if(${MSVC_VERSION} EQUAL 1310 OR ${MSVC_VERSION} EQUAL 1400 OR ${MSVC_VERSION} EQUAL 1500 OR ${MSVC_VERSION} EQUAL 1600 )
			else()
				aux_source_directory(${SRC_DIR}/gui/msvc SRC_MSVC)
				set( ALL_VIEWER_SOURCES ${ALL_VIEWER_SOURCES} ${SRC_MSVC} )
			endif()
		endif()
        if(${MSVC10})
            #add_definitions( -D_HAS_CPP0X=0)
        endif()
    endif(MSVC)
endif(WIN32)


###########################################################
### Recherche des packages necessaires a la compilation ###
###########################################################
if(WIN32)
    set(CMAKE_PREFIX_PATH "C:/Program Files/MATIS/")
endif(WIN32)

find_package(JPEG REQUIRED)
if(JPEG_FOUND)
    include_directories(${JPEG_INCLUDE_DIR})
else()
    message(FATAL_ERROR "JPEG not found ! Please set JPEG path ...")
endif()


if(NOT WIN32)
    find_package(PNG REQUIRED)
    if(PNG_FOUND)
        include_directories(${PNG_INCLUDE_DIR})
    else()
        message(FATAL_ERROR "PNG not found ! Please set PNG path ...")
    endif()

    find_package(ZLIB REQUIRED)
    if(ZLIB_FOUND)
        include_directories(${ZLIB_INCLUDE_DIR})
    else()
        message(FATAL_ERROR "ZLIB not found ! Please set ZLIB path ...")
    endif()
endif(NOT WIN32)
	
set(wxWidgets_USE_STATIC OFF CACHE BOOL "use wxWidgets static lib")

if(NOT WIN32)
    #Find wxWidgets
    set(wxWidgets_EXCLUDE_COMMON_LIBRARIES TRUE)
        find_package(wxWidgets REQUIRED adv base core xml aui xrc html)
    if(wxWidgets_FOUND)
        include(${wxWidgets_USE_FILE})
    else()
        message(FATAL_ERROR "wxWidgets not found ! Please set wxWidgets path ...")
    endif()
else(NOT WIN32)
    #Find wxWidgets
    set(wxWidgets_EXCLUDE_COMMON_LIBRARIES TRUE)
    #set(wxWidgets_USE_STATIC OFF)
    find_package(wxWidgets REQUIRED adv base core xml aui xrc html zlib png expat)
    if(wxWidgets_FOUND)
        #add_definitions(${wxWidgets_DEFINITIONS})
        # not set definitions like that because on MSVC 2010 this add wrong terme on AdditionalOptions flags
        include(${wxWidgets_USE_FILE})
        include_directories( ${wxWidgets_ROOT_DIR}/src/png )
        include_directories( ${wxWidgets_ROOT_DIR}/src/zlib )
    else()
        message(FATAL_ERROR "wxWidgets not found ! Please set wxWidgets path ...")
    endif()
endif(NOT WIN32)

# Find BOOST


# SET( Boost_ADDITIONAL_VERSIONS "1.67" "1.67.0" "1.71" "1.71.0")

set(Boost_USE_STATIC_LIBS OFF CACHE BOOL "use boost static lib")
set(Boost_USE_MULTITHREAD ON CACHE BOOL "use boost multi thread lib")
set(BOOST_ROOT "" CACHE PATH "path to boost root directory")
set(Boost_NO_SYSTEM_PATHS FALSE CACHE BOOL "use boost path env variable")

FIND_PACKAGE(Boost 1.71.0 COMPONENTS filesystem system thread serialization)
IF(Boost_FOUND)
	INCLUDE_DIRECTORIES( ${Boost_INCLUDE_DIRS})
    link_directories( ${Boost_LIBRARY_DIRS})
	if( WIN32 )
		add_definitions( -DBOOST_ALL_NO_LIB )
	endif()
ELSE(Boost_FOUND)
	MESSAGE(FATAL_ERROR "Boost not found ! Please set Boost path ...")
ENDIF()




####
#### Ajout du fichier contenant les regles de construction des libs externes : tinyxml
####
INCLUDE( ${CMAKE_CONFIG_DIR}/GilViewer_build_external_lib.cmake )

#####################################
###                     recherche des packages optionnels                        ###
#####################################

find_package(ImageIO QUIET)
option(USE_ImageIO "Build GilViewer with ImageIO" ImageIO_FOUND)
if(USE_ImageIO)
    set( GILVIEWER_USE_IMAGEIO 1 )
else()
    set( GILVIEWER_USE_IMAGEIO 0 )
endif()

#####################################
###                     Fin recherche des packages                         ###
#####################################

### Generation du fichier de resources
if( WIN32 )
    set(wxrc_EXECUTABLE ${wxWidgets_wxrc_EXECUTABLE})
else()
    set(wxrc_EXECUTABLE wxrc)
endif()
add_custom_command( OUTPUT ${CMAKE_SOURCE_DIR}/${SRC_DIR}/gui/resources/resources.cpp
                    COMMAND ${wxrc_EXECUTABLE} ${CMAKE_SOURCE_DIR}/${SRC_DIR}/gui/resources/resources.xrc -c -o ${CMAKE_SOURCE_DIR}/${SRC_DIR}/gui/resources/resources.cpp
                    DEPENDS ${CMAKE_SOURCE_DIR}/${SRC_DIR}/gui/resources/resources.xrc )

SET( ALL_VIEWER_SOURCES ${ALL_VIEWER_SOURCES} ${CMAKE_SOURCE_DIR}/${SRC_DIR}/gui/resources/resources.cpp )

####
#### Construction de la librarie
####
INCLUDE( ${CMAKE_CONFIG_DIR}/GilViewer_build_library.cmake )

####
#### Construction du sample
####
INCLUDE( ${CMAKE_CONFIG_DIR}/GilViewer_build_sample.cmake )

# Ajout du repertoire d'include des lib externes (pour l'instant,  tinyxml)
#INCLUDE_DIRECTORIES( BEFORE "extern/" )
# Ajout du repertoire d'include des sources du viewer
#INCLUDE_DIRECTORIES( BEFORE  "src/" )

# Ajout d'une dependance du viewer a tinyxml. Ca permet de compiler tinyxml AVANT le viewer ...
#ADD_DEPENDENCIES( GilViewer tinyxml )
# Et enfin, le viewer depend de la lib ...
#ADD_DEPENDENCIES( GilViewerApp GilViewer )
# De meme que les samples ...
#ADD_DEPENDENCIES( sample_vector_layer GilViewer )
#ADD_DEPENDENCIES( sample_subimage GilViewer )

####
#### Choice for install mode.
####     - None : no install target
####     - Install : create a install target
####     - Package : create a packake
SET(INSTALL_MODE "Install" CACHE STRING "Choose install mode between : None, Install, Package")
IF(${INSTALL_MODE} STREQUAL "Install")
    # If an install target is selected, propose to install langage files
    set( LANGAGES_SUPPORT_FILES "" )
    # Currently available langages are: French
    option( INSTALL_LANGAGE_FRENCH "Install french langage support" ON )
        set( LANGAGES_FR_SUPPORT_FILES ${CMAKE_SOURCE_DIR}/languages/fr/libGilViewer.mo ${CMAKE_SOURCE_DIR}/languages/fr/GilViewerApp.mo )
    INCLUDE( ${CMAKE_CONFIG_DIR}/GilViewer_install.cmake )
    MESSAGE( STATUS " Install include files" )
ENDIF(${INSTALL_MODE} STREQUAL "Install")
IF(${INSTALL_MODE} STREQUAL "Package")
    INCLUDE( ${CMAKE_CONFIG_DIR}/GilViewer_package.cmake )
    MESSAGE( STATUS " Install file package" )
ENDIF(${INSTALL_MODE} STREQUAL "Package")



####
#### Construction des plugins
####
INCLUDE( ${CMAKE_CONFIG_DIR}/GilViewer_build_plugins.cmake )

###
### Sum up found and not found dependencies
###
message( STATUS "**************" )
message( STATUS "With GDAL     --> " ${USE_GDAL_OGR} )
message( STATUS "With GDAL_JP2 --> " ${USE_GDAL_JP2} )
message( STATUS "**************" )