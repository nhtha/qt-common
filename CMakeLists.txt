set_env( CONFIG_URL "https://id.eesti.ee/config.json" CACHE STRING "Set Config URL" )
find_package( OpenSSL REQUIRED )
if( UNIX AND NOT APPLE )
	find_package(PCSCLite REQUIRED)
endif()

if( CONFIG_URL )
	set_env( LAST_CHECK_DAYS 7 CACHE STRING "How often check configuration changes" )
	if( LAST_CHECK_DAYS )
		set_source_files_properties( Configuration.cpp PROPERTIES COMPILE_DEFINITIONS "LAST_CHECK_DAYS=${LAST_CHECK_DAYS}" )
	endif()
	add_definitions( -DCONFIG_URL="${CONFIG_URL}" )
	list( APPEND SOURCES Configuration.cpp )
	file( DOWNLOAD ${CONFIG_URL} ${CMAKE_CURRENT_BINARY_DIR}/config.json )
	string( REPLACE ".json" ".rsa" RSA_URL ${CONFIG_URL} )
	file( DOWNLOAD ${RSA_URL} ${CMAKE_CURRENT_BINARY_DIR}/config.rsa )
	string( REPLACE ".json" ".pub" PUB_URL ${CONFIG_URL} )
	file( DOWNLOAD ${PUB_URL} ${CMAKE_CURRENT_BINARY_DIR}/config.pub )
	configure_file( config.qrc config.qrc COPYONLY )
	qt5_add_resources( SOURCES ${CMAKE_CURRENT_BINARY_DIR}/config.qrc )
endif()

configure_file( translations/common_tr.qrc common_tr.qrc COPYONLY )
qt5_add_translation( SOURCES  translations/qtbase_et.ts translations/qtbase_ru.ts
	translations/common_en.ts translations/common_et.ts translations/common_ru.ts )
qt5_add_resources( SOURCES images/common_images.qrc ${CMAKE_CURRENT_BINARY_DIR}/common_tr.qrc )
qt5_wrap_ui( SOURCES AboutDialog.ui CertificateWidget.ui )
add_library( qdigidoccommon STATIC ${SOURCES}
	AboutDialog.cpp
	CertificateWidget.cpp
	CliApplication.cpp
	ComboBox.cpp
	Common.cpp
	DateTime.cpp
	Diagnostics.cpp
	DiagnosticsTask.cpp
	IKValidator.cpp
	PinDialog.cpp
	QPCSC.cpp
	SOAPDocument.cpp
	Settings.h
	SslCertificate.cpp
	TokenData.cpp
	qasn1element.cpp
)
target_include_directories(qdigidoccommon PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} PRIVATE ${PCSCLITE_INCLUDE_DIR})
target_link_libraries(qdigidoccommon Qt5::Network Qt5::Widgets OpenSSL::SSL ${PCSCLITE_LIBRARY})
if(WIN32)
	target_compile_definitions(qdigidoccommon PUBLIC UNICODE WIN32_LEAN_AND_MEAN)
	target_sources(qdigidoccommon PRIVATE Diagnostics_win.cpp)
	target_link_libraries(qdigidoccommon Version SetupApi winscard)
else()
	target_sources(qdigidoccommon PRIVATE Diagnostics_unix.cpp)
endif()

if(APPLE)
	target_sources(qdigidoccommon PRIVATE MacMenuBar.cpp)
	target_link_libraries(qdigidoccommon "-framework Cocoa" "-framework PCSC" "-fobjc-arc")
else()
	add_subdirectory(qtsingleapplication)
	target_link_libraries(qdigidoccommon qtsingleapplication)
endif()
