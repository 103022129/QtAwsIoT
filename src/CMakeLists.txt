
set(${TARGET_NAME}_SOURCES
    qawsiotclient.cpp
    qawsiotnetworkrequest.cpp
)

set(${TARGET_NAME}_PUBLIC_HEADERS
    qawsiotclient.h
    qawsiot_global.h
)

set(${TARGET_NAME}_PRIVATE_HEADERS
    qawsiotclient_p.h
    qawsiotnetworkrequest_p.h
)

add_definitions(-DQTAWSIOT_LIBRARY)

# will be a static or dynamic library based on the setting of BUILD_SHARED_LIBS
# see: https://cmake.org/cmake/help/v2.8.12/cmake.html#command%3aadd_library
# The header files are also included in the add_library command, so that the automoc functionality
# will work.
add_library(${TARGET_NAME}
            ${${TARGET_NAME}_SOURCES}
            ${${TARGET_NAME}_PUBLIC_HEADERS}
            ${${TARGET_NAME}_PRIVATE_HEADERS})
add_library(${PROJECT_NAME_PREFIX}${QT_MAJOR_VERSION_REQUIRED}::${TARGET_NAME} ALIAS ${TARGET_NAME})

# add the Qtlibraries to the list of linked libraries
# the Qt libraries are added as PUBLIC, so that projects using this module will transitively
# link to these libraries as well. Include paths and compiler settings will also be promoted.
target_link_libraries(${TARGET_NAME} PUBLIC Qt5::Core Qt5::Network Qt5::WebSockets Qt5::Mqtt)

set_target_properties(${TARGET_NAME} PROPERTIES
  VERSION ${FULL_VERSION}
  SOVERSION ${SO_VERSION}
  PUBLIC_HEADER "${${TARGET_NAME}_PUBLIC_HEADERS}"
  PRIVATE_HEADER "${${TARGET_NAME}_PRIVATE_HEADERS}"
)

##############################################
# Create, export and install config packages #
##############################################

# Add include directories of own project to include interface
target_include_directories(${TARGET_NAME} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)

# Create config file
configure_package_config_file(cmake/config.cmake.in
  ${CMAKE_BINARY_DIR}/${CMAKE_CONFIG_FILE_BASE_NAME}Config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_DIR}
  PATH_VARS INCLUDE_INSTALL_DIR
)

# Create a config version file
write_basic_package_version_file(
  ${CMAKE_BINARY_DIR}/${CMAKE_CONFIG_FILE_BASE_NAME}ConfigVersion.cmake
  VERSION ${FULL_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Create import targets
install(TARGETS ${TARGET_NAME} EXPORT ${TARGET_NAME}Targets
  RUNTIME DESTINATION ${BIN_INSTALL_DIR}
  LIBRARY DESTINATION ${LIB_INSTALL_DIR}
  ARCHIVE DESTINATION ${LIB_INSTALL_DIR}
  PUBLIC_HEADER DESTINATION ${INCLUDE_INSTALL_DIR}
  PRIVATE_HEADER DESTINATION ${INCLUDE_INSTALL_DIR}/private
)

# Export the import targets
install(EXPORT ${TARGET_NAME}Targets
  FILE "${CMAKE_CONFIG_FILE_BASE_NAME}Targets.cmake"
  NAMESPACE ${PROJECT_NAMESPACE}::
  DESTINATION ${CMAKE_INSTALL_DIR}
)

# Now install the 3 config files
install(FILES ${CMAKE_BINARY_DIR}/${CMAKE_CONFIG_FILE_BASE_NAME}Config.cmake
              ${CMAKE_BINARY_DIR}/${CMAKE_CONFIG_FILE_BASE_NAME}ConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_DIR}
)

# Create and install a global module include file
# This makes it possible to include all header files of the module by using
# #include <${PROJECT_NAME}>
set(GLOBAL_HEADER_FILE ${CMAKE_BINARY_DIR}/${PROJECT_NAME})
file(WRITE ${GLOBAL_HEADER_FILE} "//Includes all headers of ${PROJECT_NAME}\n\n")

foreach(header ${${TARGET_NAME}_PUBLIC_HEADERS})
  file(APPEND ${GLOBAL_HEADER_FILE} "#include \"${header}\"\n")
endforeach()

install(FILES ${GLOBAL_HEADER_FILE} DESTINATION ${INCLUDE_INSTALL_DIR})
