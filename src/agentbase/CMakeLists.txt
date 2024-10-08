# base install include dir
set(AkonadiAgentBase_INSTALL_INCLUDEDIR "${KDE_INSTALL_INCLUDEDIR_KF}/AkonadiAgentBase")

add_library(KF5AkonadiAgentBase)
add_library(KF5::AkonadiAgentBase ALIAS KF5AkonadiAgentBase)

ecm_qt_declare_logging_category(KF5AkonadiAgentBase HEADER akonadiagentbase_debug.h IDENTIFIER AKONADIAGENTBASE_LOG CATEGORY_NAME org.kde.pim.akonadiagentbase
        DESCRIPTION "akonadi (Akonadi AgentBase Library)"
        OLD_CATEGORY_NAMES akonadiagentbase_log
        EXPORT AKONADI
    )

ecm_generate_headers(AkonadiAgentBase_CC_HEADERS
    HEADER_NAMES
    AccountsIntegration
    AgentBase
    AgentSearchInterface
    PreprocessorBase
    ResourceBase
    ResourceSettings
    TransportResourceBase
    REQUIRED_HEADERS AkonadiAgentBase_HEADERS
    PREFIX Akonadi
)


KCONFIG_ADD_KCFG_FILES(KF5AkonadiAgentBase resourcebasesettings.kcfgc)

qt_add_dbus_interfaces(akonadiagentbase_SRCS ${Akonadi_SOURCE_DIR}/src/interfaces/org.freedesktop.Akonadi.Tracer.xml )

qt_add_dbus_adaptor(akonadiagentbase_SRCS ${Akonadi_SOURCE_DIR}/src/interfaces/org.freedesktop.Akonadi.Resource.xml
    resourcebase.h Akonadi::ResourceBase resourceadaptor Akonadi__ResourceAdaptor)
qt_add_dbus_adaptor(akonadiagentbase_SRCS ${Akonadi_SOURCE_DIR}/src/interfaces/org.freedesktop.Akonadi.Preprocessor.xml
    preprocessorbase_p.h Akonadi::PreprocessorBasePrivate preprocessoradaptor Akonadi__PreprocessorAdaptor)
qt_add_dbus_adaptor(akonadiagentbase_SRCS ${Akonadi_SOURCE_DIR}/src/interfaces/org.freedesktop.Akonadi.Agent.Status.xml
    agentbase.h Akonadi::AgentBase statusadaptor Akonadi__StatusAdaptor)
qt_add_dbus_adaptor(akonadiagentbase_SRCS ${Akonadi_SOURCE_DIR}/src/interfaces/org.freedesktop.Akonadi.Agent.Control.xml
    agentbase.h Akonadi::AgentBase controladaptor Akonadi__ControlAdaptor)
qt_add_dbus_adaptor(akonadiagentbase_SRCS ../interfaces/org.freedesktop.Akonadi.Resource.Transport.xml
    transportresourcebase_p.h Akonadi::TransportResourceBasePrivate transportadaptor Akonadi__TransportAdaptor)
qt_add_dbus_adaptor(akonadiagentbase_SRCS ../interfaces/org.freedesktop.Akonadi.Agent.Search.xml
                     agentsearchinterface_p.h Akonadi::AgentSearchInterfacePrivate searchadaptor Akonadi__SearchAdaptor )
qt_add_dbus_adaptor(akonadiagentbase_SRCS ../interfaces/org.kde.Akonadi.Accounts.xml
    accountsintegration.h Akonadi::AccountsIntegration accountsadaptor Akonadi__AccountsAdaptor)

target_sources(KF5AkonadiAgentBase PRIVATE
    accountsintegration.cpp
    agentbase.cpp
    agentsearchinterface.cpp
    preprocessorbase.cpp
    preprocessorbase_p.cpp
    recursivemover.cpp
    resourcebase.cpp
    resourcescheduler.cpp
    resourcesettings.cpp
    transportresourcebase.cpp
    accountsintegration.h
    agentbase.h
    agentsearchinterface.h
    preprocessorbase.h
    preprocessorbase_p.h
    recursivemover_p.h
    resourcebase.h
    resourcescheduler_p.h
    resourcesettings.h
    transportresourcebase.h
    ${akonadiagentbase_SRCS}
)



if (COMPILE_WITH_UNITY_CMAKE_SUPPORT)
    set_target_properties(KF5AkonadiAgentBase PROPERTIES UNITY_BUILD ON)
endif()

generate_export_header(KF5AkonadiAgentBase BASE_NAME akonadiagentbase)


target_include_directories(KF5AkonadiAgentBase INTERFACE "$<INSTALL_INTERFACE:${AkonadiAgentBase_INSTALL_INCLUDEDIR}>")

target_link_libraries(KF5AkonadiAgentBase
PUBLIC
    Qt::DBus
    Qt::Widgets # for QApplication
    KF5::AkonadiCore
    KF5::ConfigCore
    KF5::ConfigGui # for KConfigSkeleton
PRIVATE
    KF5::AkonadiPrivate
    KF5::AkonadiWidgets
    KF5::I18n
    Qt::Network
    akonadi_shared
)

if (WITH_ACCOUNTS)
    target_link_libraries(KF5AkonadiAgentBase PRIVATE KAccounts ${ACCOUNTSQT_LIBRARIES})
    target_include_directories(KF5AkonadiAgentBase PRIVATE ${ACCOUNTSQT_INCLUDE_DIRS})
endif()

set_target_properties(KF5AkonadiAgentBase PROPERTIES
    VERSION ${AKONADI_VERSION}
    SOVERSION ${AKONADI_SOVERSION}
    EXPORT_NAME AkonadiAgentBase
)

ecm_generate_pri_file(BASE_NAME AkonadiAgentBase
    LIB_NAME KF5AkonadiAgentBase
    DEPS "AkonadiCore AkonadiPrivate KConfigCore KConfigGui" FILENAME_VAR PRI_FILENAME
)

install(TARGETS
    KF5AkonadiAgentBase
    EXPORT KF5AkonadiTargets
    ${KF_INSTALL_TARGETS_DEFAULT_ARGS}
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/akonadiagentbase_export.h
    ${CMAKE_CURRENT_BINARY_DIR}/resourcebasesettings.h
    ${AkonadiAgentBase_HEADERS}
    DESTINATION ${AkonadiAgentBase_INSTALL_INCLUDEDIR}/akonadi
    COMPONENT Devel
)

install(FILES
    ${AkonadiAgentBase_CC_HEADERS}
    DESTINATION ${AkonadiAgentBase_INSTALL_INCLUDEDIR}/Akonadi
    COMPONENT Devel
)

install(FILES
    resourcebase.kcfg
    DESTINATION ${KDE_INSTALL_KCFGDIR}
)

install(FILES
    ${PRI_FILENAME}
    DESTINATION ${ECM_MKSPECS_INSTALL_DIR}
)
