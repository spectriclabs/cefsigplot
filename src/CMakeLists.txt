# Copyright (c) 2014 The Chromium Embedded Framework Authors. All rights
# reserved. Use of this source code is governed by a BSD-style license that
# can be found in the LICENSE file.

#
# Source files.
#

set(CEFSIGPLOT_APP_SRCS
  app/app.html
  app/app.js
  )
source_group(cefsigplot\\\\app FILES ${CEFSIGPLOT_APP_SRCS})

# cefsigplot sources.
set(CEFSIGPLOT_SRCS
  sigplot_app.cc
  sigplot_app.h
  sigplot_handler.cc
  sigplot_handler.h
  )
set(CEFSIGPLOT_SRCS_LINUX
  cefsigplot_linux.cc
  sigplot_handler_linux.cc
  )
set(CEFSIGPLOT_SRCS_MACOSX
  cefsigplot_mac.mm
  sigplot_handler_mac.mm
  )
set(CEFSIGPLOT_SRCS_WINDOWS
  cefsigplot.exe.manifest
  cefsigplot.rc
  cefsigplot_win.cc
  resource.h
  sigplot_handler_win.cc
  )
APPEND_PLATFORM_SOURCES(CEFSIGPLOT_SRCS)
source_group(cefsigplot FILES ${CEFSIGPLOT_SRCS})


set(CEFSIGPLOT_RES_SRCS_WINDOWS
  res/cefsigplot.ico
  res/small.ico
  )
APPEND_PLATFORM_SOURCES(CEFSIGPLOT_RES_SRCS)
source_group(cefsigplot\\\\res FILES ${CEFSIGPLOT_RES_SRCS})

set(CEFSIGPLOT_SRCS
  ${CEFSIGPLOT_SRCS}
  ${CEFSIGPLOT_RES_SRCS}
  ${CEFSIGPLOT_APP_SRCS}
  )

# cefsigplot helper sources.
set(CEFSIGPLOT_HELPER_SRCS_MACOSX
  process_helper_mac.cc
  )
APPEND_PLATFORM_SOURCES(CEFSIGPLOT_HELPER_SRCS)
source_group(cefsigplot FILES ${CEFSIGPLOT_HELPER_SRCS})

# cefsigplot resources.
set(CEFSIGPLOT_RESOURCES_MAC_SRCS_MACOSX
  mac/Info.plist
  mac/cefsigplot.icns
  )
APPEND_PLATFORM_SOURCES(CEFSIGPLOT_RESOURCES_MAC_SRCS)
source_group(cefsigplot\\\\mac FILES ${CEFSIGPLOT_RESOURCES_MAC_SRCS})

set(CEFSIGPLOT_RESOURCES_MAC_ENGLISH_LPROJ_SRCS_MACOSX
  mac/English.lproj/InfoPlist.strings
  mac/English.lproj/MainMenu.xib
  )
APPEND_PLATFORM_SOURCES(CEFSIGPLOT_RESOURCES_MAC_ENGLISH_LPROJ_SRCS)
source_group(cefsigplot\\\\mac\\\\English.lproj FILES ${CEFSIGPLOT_RESOURCES_MAC_ENGLISH_LPROJ_SRCS})

set(CEFSIGPLOT_RESOURCES_SRCS
  ${CEFSIGPLOT_RESOURCES_MAC_SRCS}
  ${CEFSIGPLOT_RESOURCES_MAC_ENGLISH_LPROJ_SRCS}
  )


#
# Shared configuration.
#

# Target executable names.
set(CEF_TARGET "cefsigplot")
if(OS_MACOSX)
  set(CEF_HELPER_TARGET "cefsigplot_Helper")
  set(CEF_HELPER_OUTPUT_NAME "cefsigplot Helper")
else()
  # Logical target used to link the libcef library.
  ADD_LOGICAL_TARGET("libcef_lib" "${CEF_LIB_DEBUG}" "${CEF_LIB_RELEASE}")
endif()

# Determine the target output directory.
SET_CEF_TARGET_OUT_DIR()


#
# Linux configuration.
#

if(OS_LINUX)
  # Executable target.
  add_executable(${CEF_TARGET} ${CEFSIGPLOT_SRCS})
  SET_EXECUTABLE_TARGET_PROPERTIES(${CEF_TARGET})
  add_dependencies(${CEF_TARGET} libcef_dll_wrapper)
  target_link_libraries(${CEF_TARGET} libcef_lib libcef_dll_wrapper ${CEF_STANDARD_LIBS})

  # Set rpath so that libraries can be placed next to the executable.
  set_target_properties(${CEF_TARGET} PROPERTIES INSTALL_RPATH "$ORIGIN")
  set_target_properties(${CEF_TARGET} PROPERTIES BUILD_WITH_INSTALL_RPATH TRUE)
  set_target_properties(${CEF_TARGET} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CEF_TARGET_OUT_DIR})

  # Copy binary and resource files to the target output directory.
  COPY_FILES("${CEF_TARGET}" "${CEF_BINARY_FILES}" "${CEF_BINARY_DIR}" "${CEF_TARGET_OUT_DIR}")
  COPY_FILES("${CEF_TARGET}" "${CEF_RESOURCE_FILES}" "${CEF_RESOURCE_DIR}" "${CEF_TARGET_OUT_DIR}")
  COPY_FILES("${CEF_TARGET}" "${CEFSIGPLOT_APP_SRCS}" "${CMAKE_CURRENT_SOURCE_DIR}" "${CEF_TARGET_OUT_DIR}/app")

  message("${CEF_TARGET}")
  message("${CEF_RESOURCE_FILES}")
  message("${CEF_RESOURCE_DIR}")
  message("${CEF_TARGET_OUT_DIR}")

  # Set SUID permissions on the chrome-sandbox target.
  SET_LINUX_SUID_PERMISSIONS("${CEF_TARGET}" "${CEF_TARGET_OUT_DIR}/chrome-sandbox")
endif()


#
# Mac OS X configuration.
#

if(OS_MACOSX)
  # Output paths for the app bundles.
  set(CEF_APP "${CEF_TARGET_OUT_DIR}/${CEF_TARGET}.app")
  set(CEF_HELPER_APP "${CEF_TARGET_OUT_DIR}/${CEF_HELPER_OUTPUT_NAME}.app")

  # Variable referenced from Info.plist files.
  set(PRODUCT_NAME "${CEF_TARGET}")

  # Helper executable target.
  add_executable(${CEF_HELPER_TARGET} MACOSX_BUNDLE ${CEFSIGPLOT_HELPER_SRCS})
  SET_EXECUTABLE_TARGET_PROPERTIES(${CEF_HELPER_TARGET})
  add_dependencies(${CEF_HELPER_TARGET} libcef_dll_wrapper)
  target_link_libraries(${CEF_HELPER_TARGET} libcef_dll_wrapper ${CEF_STANDARD_LIBS})
  set_target_properties(${CEF_HELPER_TARGET} PROPERTIES
    MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/mac/helper-Info.plist
    OUTPUT_NAME ${CEF_HELPER_OUTPUT_NAME}
    )

  if(USE_SANDBOX)
    # Logical target used to link the cef_sandbox library.
    ADD_LOGICAL_TARGET("cef_sandbox_lib" "${CEF_SANDBOX_LIB_DEBUG}" "${CEF_SANDBOX_LIB_RELEASE}")
    target_link_libraries(${CEF_HELPER_TARGET} cef_sandbox_lib)
  endif()

  # Main executable target.
  add_executable(${CEF_TARGET} MACOSX_BUNDLE ${CEFSIGPLOT_RESOURCES_SRCS} ${CEFSIGPLOT_SRCS})
  SET_EXECUTABLE_TARGET_PROPERTIES(${CEF_TARGET})
  add_dependencies(${CEF_TARGET} libcef_dll_wrapper "${CEF_HELPER_TARGET}")
  target_link_libraries(${CEF_TARGET} libcef_dll_wrapper ${CEF_STANDARD_LIBS})
  set_target_properties(${CEF_TARGET} PROPERTIES
    MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/mac/Info.plist
    )

  # Copy files into the main app bundle.
  add_custom_command(
    TARGET ${CEF_TARGET}
    POST_BUILD
    # Copy the helper app bundle into the Frameworks directory.
    COMMAND ${CMAKE_COMMAND} -E copy_directory
            "${CEF_HELPER_APP}"
            "${CEF_APP}/Contents/Frameworks/${CEF_HELPER_OUTPUT_NAME}.app"
    # Copy the CEF framework into the Frameworks directory.
    COMMAND ${CMAKE_COMMAND} -E copy_directory
            "${CEF_BINARY_DIR}/Chromium Embedded Framework.framework"
            "${CEF_APP}/Contents/Frameworks/Chromium Embedded Framework.framework"
    VERBATIM
    )

  # Manually process and copy over resource files.
  # The Xcode generator can support this via the set_target_properties RESOURCE
  # directive but that doesn't properly handle nested resource directories.
  # Remove these prefixes from input file paths.
  set(PREFIXES "mac/")
  COPY_MACOSX_RESOURCES("${CEFSIGPLOT_RESOURCES_SRCS}" "${PREFIXES}" "${CEF_TARGET}" "${CMAKE_CURRENT_SOURCE_DIR}" "${CEF_APP}")
endif()


#
# Windows configuration.
#

if(OS_WINDOWS)
  # Executable target.
  add_executable(${CEF_TARGET} WIN32 ${CEFSIGPLOT_SRCS})
  add_dependencies(${CEF_TARGET} libcef_dll_wrapper)
  SET_EXECUTABLE_TARGET_PROPERTIES(${CEF_TARGET})
  target_link_libraries(${CEF_TARGET} libcef_lib libcef_dll_wrapper ${CEF_STANDARD_LIBS})

  if(USE_SANDBOX)
    # Logical target used to link the cef_sandbox library.
    ADD_LOGICAL_TARGET("cef_sandbox_lib" "${CEF_SANDBOX_LIB_DEBUG}" "${CEF_SANDBOX_LIB_RELEASE}")
    target_link_libraries(${CEF_TARGET} cef_sandbox_lib ${CEF_SANDBOX_STANDARD_LIBS})
  endif()

  # Add the custom manifest files to the executable.
  ADD_WINDOWS_MANIFEST("${CMAKE_CURRENT_SOURCE_DIR}" "${CEF_TARGET}" "exe")

  # Copy binary and resource files to the target output directory.
  COPY_FILES("${CEF_TARGET}" "${CEF_BINARY_FILES}" "${CEF_BINARY_DIR}" "${CEF_TARGET_OUT_DIR}")
  COPY_FILES("${CEF_TARGET}" "${CEF_RESOURCE_FILES}" "${CEF_RESOURCE_DIR}" "${CEF_TARGET_OUT_DIR}")
endif()
