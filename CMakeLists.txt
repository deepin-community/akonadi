cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

set(PIM_VERSION "5.22.3")
project(Akonadi VERSION ${PIM_VERSION})


# ECM setup
set(KF5_MIN_VERSION "5.99.0")

find_package(ECM ${KF5_MIN_VERSION} CONFIG REQUIRED)

set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)

include(GenerateExportHeader)
include(ECMGenerateHeaders)
include(ECMGeneratePriFile)

include(ECMSetupVersion)
include(FeatureSummary)
include(KDEGitCommitHooks)
include(KDEClangFormat)
file(GLOB_RECURSE ALL_CLANG_FORMAT_SOURCE_FILES *.cpp *.h *.c)
kde_clang_format(${ALL_CLANG_FORMAT_SOURCE_FILES})

include(CheckAtomic)
include(CheckIncludeFiles)
include(ECMQtDeclareLoggingCategory)
include(ECMDeprecationSettings)
include(CheckSymbolExists)

include(KDEPackageAppTemplates)
include(ECMMarkNonGuiExecutable)
include(ECMAddTests)
include(ECMSetupQtPluginMacroNames)

include(AkonadiMacros)
include(ECMAddQch)


option(BUILD_QCH "Build API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)" OFF)
add_feature_info(QCH ${BUILD_QCH} "API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)")

set(QT_REQUIRED_VERSION "5.15.2")
if (QT_MAJOR_VERSION STREQUAL "6")
    set(QT_REQUIRED_VERSION "6.4.0")
endif()
set(RELEASE_SERVICE_VERSION "22.12.3")
set(AKONADI_FULL_VERSION "${PIM_VERSION} (${RELEASE_SERVICE_VERSION})")

configure_file(akonadifull-version.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/akonadifull-version.h @ONLY)


ecm_setup_version(PROJECT
    VARIABLE_PREFIX AKONADI
    VERSION_HEADER "${CMAKE_CURRENT_BINARY_DIR}/akonadi_version.h"
    PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/KF5AkonadiConfigVersion.cmake"
    SOVERSION 5)

# Find packages
find_package(Qt${QT_MAJOR_VERSION}Core ${QT_REQUIRED_VERSION} REQUIRED COMPONENTS Private)
find_package(Qt${QT_MAJOR_VERSION}Sql ${QT_REQUIRED_VERSION} REQUIRED COMPONENTS Private)
find_package(Qt${QT_MAJOR_VERSION}DBus ${QT_REQUIRED_VERSION} REQUIRED)
find_package(Qt${QT_MAJOR_VERSION}Network ${QT_REQUIRED_VERSION} REQUIRED)
find_package(Qt${QT_MAJOR_VERSION}Test ${QT_REQUIRED_VERSION} REQUIRED)
find_package(Qt${QT_MAJOR_VERSION}Widgets ${QT_REQUIRED_VERSION} REQUIRED)
find_package(Qt${QT_MAJOR_VERSION}Xml ${QT_REQUIRED_VERSION} REQUIRED)

find_package(KF5Config ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5ConfigWidgets ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5CoreAddons ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5I18n ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5IconThemes ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5ItemModels ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5ItemViews ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5KIO ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5WidgetsAddons ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5WindowSystem ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5XmlGui ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5Crash ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(Qt${QT_MAJOR_VERSION}Designer NO_MODULE)
set_package_properties(Qt${QT_MAJOR_VERSION}Designer PROPERTIES
   PURPOSE "Required to build the Qt Designer plugins"
   TYPE OPTIONAL
)

option(BUILD_DESIGNERPLUGIN "Build plugin for Qt Designer" ON)
add_feature_info(DESIGNERPLUGIN ${BUILD_DESIGNERPLUGIN} "Build plugin for Qt Designer")
if (QT_MAJOR_VERSION STREQUAL "6")
    find_package(Qt6Core5Compat)
endif()

set(AccountsQt5_MINIMUM_VERSION "1.16")
find_package(AccountsQt5 ${AccountsQt5_MINIMUM_VERSION})
set_package_properties(AccountsQt5 PROPERTIES
    DESCRIPTION "Qt bindings for the Accounts framework"
    URL "https://gitlab.com/accounts-sso/libaccounts-qt"
    TYPE OPTIONAL
)
set(KAccounts_MINIMUM_VERSION "19.08.0")
find_package(KAccounts ${KAccounts_MINIMUM_VERSION})
set_package_properties(KAccounts PROPERTIES
    DESCRIPTION "KDE library for Accounts framework integration"
    URL "https://cgit.kde.org/kaccounts-integration.git"
    TYPE OPTIONAL
)
if (${AccountsQt5_FOUND} AND ${KAccounts_FOUND})
    set(WITH_ACCOUNTS TRUE)
endif()

set(LibLZMA_MINIMUM_VERSION "5.0.0")
find_package(LibLZMA ${LibLZMA_MINIMUM_VERSION})
set_package_properties(LibLZMA PROPERTIES
    DESCRIPTION "LZMA compression library"
    URL "https://tukaani.org/xz/"
    TYPE REQUIRED
)


if(BUILD_TESTING)
    set(AKONADI_TESTS_EXPORT AKONADICORE_EXPORT)
    set(AKONADIWIDGET_TESTS_EXPORT AKONADIWIDGETS_EXPORT)
    add_definitions(-DBUILD_TESTING)
endif()

ecm_setup_qtplugin_macro_names(
    JSON_ARG2
        "AKONADI_AGENTCONFIG_FACTORY"
    CONFIG_CODE_VARIABLE
        PACKAGE_SETUP_AUTOMOC_VARIABLES
)

# Make sure the KF5Akonadi_DATA_DIR is absolute before passing it to KF5AkonadiConfig.cmake.in
# otherwise build fails either on OSX CI, or for normal users
if (IS_ABSOLUTE "${KDE_INSTALL_DATADIR_KF}")
    set(KF5Akonadi_DATA_DIR "${KDE_INSTALL_DATADIR_KF}/akonadi")
else()
    set(KF5Akonadi_DATA_DIR "${CMAKE_INSTALL_PREFIX}/${KDE_INSTALL_DATADIR_KF}/akonadi")
endif()

check_symbol_exists(malloc_trim "malloc.h" HAVE_MALLOC_TRIM)

############### Build Options ###############
option(AKONADI_BUILD_QSQLITE "Build the Sqlite backend." TRUE)
option(BUILD_TOOLS "Build and install tools for development and testing purposes." TRUE)
option(INSTALL_APPARMOR "Install AppArmor profiles" TRUE)

if(BUILD_TESTING)
   set(BUILD_TOOLS TRUE)
endif()

option(USE_UNITY_CMAKE_SUPPORT "Use UNITY cmake support (speedup compile time)" OFF)

set(COMPILE_WITH_UNITY_CMAKE_SUPPORT OFF)
if (USE_UNITY_CMAKE_SUPPORT)
    set(COMPILE_WITH_UNITY_CMAKE_SUPPORT ON)
    add_definitions(-DCOMPILE_WITH_UNITY_CMAKE_SUPPORT)
endif()
set(SMI_MIN_VERSION "1.3")
find_package(SharedMimeInfo ${SMI_MIN_VERSION} REQUIRED)

find_program(XSLTPROC_EXECUTABLE xsltproc)
if(NOT XSLTPROC_EXECUTABLE)
    message(FATAL_ERROR "\nThe command line XSLT processor program 'xsltproc'  could not be found.\nPlease install xsltproc.\n")
endif()

find_program(MYSQLD_EXECUTABLE NAMES mysqld
    PATHS /usr/sbin /usr/local/sbin /usr/libexec /usr/local/libexec /opt/mysql/libexec /usr/mysql/bin /opt/mysql/sbin
    DOC "The mysqld executable path. ONLY needed at runtime"
)

find_path(MYSQLD_SCRIPTS_PATH NAMES mysql_install_db
    DOC "Path to the mysql or mariadb installation scripts (mysql_install_db, mysql_upgrade...)"
)

if(MYSQLD_EXECUTABLE)
    message(STATUS "MySQL Server found: ${MYSQLD_EXECUTABLE}")
else()
    message(STATUS "MySQL Server wasn't found. it is required to use the MySQL backend.")
endif()

if(MYSQLD_SCRIPTS_PATH)
    message(STATUS "MySQL scripts location: ${MYSQLD_SCRIPTS_PATH}")
else()
    message(STATUS "MySQL scripts location was not found. Use -DMYSQLD_SCRIPTS_PATH to point to the install location.")
endif()

find_path(POSTGRES_PATH NAMES pg_ctl
    HINTS /usr/lib${LIB_SUFFIX}/postgresql/8.4/bin
          /usr/lib${LIB_SUFFIX}/postgresql/9.0/bin
          /usr/lib${LIB_SUFFIX}/postgresql/9.1/bin
          DOC "The pg_ctl executable path. ONLY needed at runtime by the PostgreSQL backend"
)

if(POSTGRES_PATH)
    message(STATUS "PostgreSQL Server found.")
else()
    message(STATUS "PostgreSQL wasn't found. it is required to use the Postgres backend.")
endif()


if("${DATABASE_BACKEND}" STREQUAL "SQLITE")
    set(SQLITE_TYPE "REQUIRED")
else()
    set(SQLITE_TYPE "OPTIONAL")
endif()

if(AKONADI_BUILD_QSQLITE)
    set(SQLITE_MIN_VERSION 3.6.23)
    find_package(Sqlite ${SQLITE_MIN_VERSION})
    set_package_properties(Sqlite PROPERTIES
        URL "https://www.sqlite.org"
        DESCRIPTION "The Sqlite database library"
        TYPE ${SQLITE_TYPE}
        PURPOSE "Required by the Sqlite backend"
    )
endif()

find_program(XMLLINT_EXECUTABLE xmllint)
if(NOT XMLLINT_EXECUTABLE)
    message(STATUS "xmllint not found, skipping akonadidb.xml schema validation")
endif()

check_include_files(unistd.h HAVE_UNISTD_H)

if (IS_ABSOLUTE "${KDE_INSTALL_DBUSINTERFACEDIR}")
    set(AKONADI_DBUS_INTERFACES_INSTALL_DIR "${KDE_INSTALL_DBUSINTERFACEDIR}")
else()
    set(AKONADI_DBUS_INTERFACES_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${KDE_INSTALL_DBUSINTERFACEDIR}")
endif()

if (IS_ABSOLUTE "${KDE_INSTALL_INCLUDEDIR_KF}")
    set(AKONADI_INCLUDE_DIR "${KDE_INSTALL_INCLUDEDIR_KF}")
else()
    set(AKONADI_INCLUDE_DIR "${CMAKE_INSTALL_PREFIX}/${KDE_INSTALL_INCLUDEDIR_KF}")
endif()

############### Build Options ###############

include(CTest)  # Calls enable_testing().

if(NOT DEFINED DATABASE_BACKEND)
    set(DATABASE_BACKEND "MYSQL" CACHE STRING "The default database backend to use for Akonadi. Can be either MYSQL, POSTGRES or SQLITE")
endif()

############### Macros ###############

macro(SET_DEFAULT_DB_BACKEND)
    set(_backend ${ARGV0})
    if("${_backend}" STREQUAL "SQLITE")
        set(AKONADI_DATABASE_BACKEND QSQLITE3)
        set(AKONADI_BUILD_QSQLITE TRUE)
    else()
        if("${_backend}" STREQUAL "POSTGRES")
          set(AKONADI_DATABASE_BACKEND QPSQL)
        else()
          set(AKONADI_DATABASE_BACKEND QMYSQL)
        endif()
    endif()

    message(STATUS "Using default db backend ${AKONADI_DATABASE_BACKEND}")
endmacro()

#### DB BACKEND DEFAULT ####
set_default_db_backend(${DATABASE_BACKEND})

############### Compilers flags ###############

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_C_COMPILER MATCHES "icc" OR (CMAKE_CXX_COMPILER_ID MATCHES "Clang"))
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -Wno-long-long -std=iso9899:1990 -Wundef -Wcast-align -Werror-implicit-function-declaration -Wchar-subscripts -Wall -Wextra -Wpointer-arith -Wwrite-strings -Wformat-security -Wmissing-format-attribute -fno-common")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wnon-virtual-dtor -Wundef -Wcast-align -Wchar-subscripts -Wall -Wextra -Wpointer-arith -Wformat-security -fno-common -pedantic")
    CHECK_CXX_COMPILER_FLAG(-Wno-deprecated-copy NO_DEPRECATED_COPY)
    if (NO_DEPRECATED_COPY)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-copy")
    endif()

    set(CMAKE_CXX_FLAGS_DEBUGFULL "-g3 -fno-inline" CACHE STRING "Flags used by the C++ compiler during debugfull builds." FORCE)
    set(CMAKE_C_FLAGS_DEBUGFULL   "-g3 -fno-inline" CACHE STRING "Flags used by the C compiler during debugfull builds." FORCE)
    mark_as_advanced(CMAKE_CXX_FLAGS_DEBUGFULL CMAKE_C_FLAGS_DEBUGFULL)
elseif (MSVC)
    # This sets the __cplusplus macro to a real value based on the version of C++ specified by
    # the /std switch. Without it MSVC keeps reporting C++98, so feature detection doesn't work.
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:__cplusplus")
endif()


if(MSVC)
    set(_ENABLE_EXCEPTIONS -EHsc)
else()
    set(_ENABLE_EXCEPTIONS -fexceptions)
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${_ENABLE_EXCEPTIONS}")

############### Configure files #############

configure_file(config-akonadi.h.cmake ${Akonadi_BINARY_DIR}/config-akonadi.h)

############### build targets ###############
set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/KF5Akonadi")

add_definitions(-DTRANSLATION_DOMAIN=\"libakonadi5\")

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    src
)


ecm_set_disabled_deprecation_versions(QT 5.15.2 KF 5.99.0)

add_subdirectory(src)
add_subdirectory(icons)

add_subdirectory(templates)

if(BUILD_TOOLS)
    # add testrunner (application for managing a self-contained
    # test environment)
    add_subdirectory(autotests/libs/testrunner)
    add_subdirectory(autotests/libs/testresource)
    add_subdirectory(autotests/libs/testsearchplugin)
endif()

if(BUILD_TESTING)
    add_subdirectory(autotests)
    add_subdirectory(tests)
endif()

if(INSTALL_APPARMOR)
   add_subdirectory(apparmor)
endif()

############### install stuff ###############

install(FILES akonadi-mime.xml DESTINATION ${KDE_INSTALL_MIMEDIR})
update_xdg_mimetypes(${KDE_INSTALL_MIMEDIR})

############### CMake Config Files ###############


configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/KF5AkonadiConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/KF5AkonadiConfig.cmake"
    INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR}
    PATH_VARS AKONADI_DBUS_INTERFACES_INSTALL_DIR
              AKONADI_INCLUDE_DIR
              KF5Akonadi_DATA_DIR
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/KF5AkonadiConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/KF5AkonadiConfigVersion.cmake"
    "${CMAKE_CURRENT_SOURCE_DIR}/KF5AkonadiMacros.cmake"
    DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
    COMPONENT Devel
)

install(EXPORT
    KF5AkonadiTargets
    DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
    FILE KF5AkonadiTargets.cmake
    NAMESPACE KF5::)

ecm_qt_install_logging_categories(
        EXPORT AKONADI
        FILE akonadi.categories
        DESTINATION ${KDE_INSTALL_LOGGINGCATEGORIESDIR}
        )

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/akonadi_version.h
    DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF}/AkonadiCore COMPONENT Devel
)

kde_configure_git_pre_commit_hook(CHECKS CLANG_FORMAT)
ki18n_install(po)
feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
