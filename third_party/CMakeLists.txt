function(download_and_extract_library NAME EXT URL)
  if(NOT IS_DIRECTORY ${CMAKE_SOURCE_DIR}/third_party/${NAME})
    message("Downloading ${NAME}")
    file(DOWNLOAD
      ${URL}
      ${CMAKE_SOURCE_DIR}/third_party/${NAME}${EXT}
      SHOW_PROGRESS
      STATUS status
      LOG log)
    list(GET status 0 status_code)
    list(GET status 1 status_string)
    if(NOT status_code EQUAL 0)
      if(UNIX OR APPLE)
        message("Failed to download ${NAME}. Try to download with wget.")
        execute_process(
          COMMAND wget ${URL} -O ${CMAKE_SOURCE_DIR}/third_party/${NAME}${EXT})
      else()
        message(FATAL_ERROR "Failed to download ${NAME}. Please check network connection and proxy settings.
        status_code: ${status_code}
        status_string: ${status_string}
        log: ${log}
        ")
        error() # Place Unknown command to stop cmake process here.
      endif()
      
    endif()
    execute_process(
      COMMAND ${CMAKE_COMMAND} -E
      chdir ${CMAKE_SOURCE_DIR}/third_party
      ${CMAKE_COMMAND} -E tar xf ${NAME}${EXT})
  endif()
endfunction()

if(NNABLA_UTILS_WITH_HDF5)
  download_and_extract_library(hdf5-master .zip https://github.com/mokus0/hdf5/archive/master.zip)
  set(HDF5_EXTERNALLY_CONFIGURED, 1)
  set(HDF5_EXPORTED_TARGETS "nbla_utils-hdf5-targets")
  set(BUILD_SHARED_LIBS ON CACHE BOOL "Build Shared Libraries" FORCE)
  set(HDF5_BUILD_HL_LIB  ON CACHE BOOL "Build HIGH Level HDF5 Library" FORCE)
  add_subdirectory(hdf5-master)
  install(EXPORT nbla_utils-hdf5-targets DESTINATION "trash")
endif()

download_and_extract_library(eigen-git-mirror-3.3.5 .zip https://github.com/eigenteam/eigen-git-mirror/archive/3.3.5.zip)
download_and_extract_library(spdlog-0.16.3 .zip https://github.com/gabime/spdlog/archive/v0.16.3.zip)

if(BUILD_TEST)
  download_and_extract_library(googletest-release-1.7.0 .zip https://github.com/google/googletest/archive/release-1.7.0.zip)
  add_subdirectory(googletest-release-1.7.0)
endif()

if(BUILD_CPP_UTILS)
  download_and_extract_library(cmdline-master .zip https://github.com/tanakh/cmdline/archive/master.zip)
endif()
  
