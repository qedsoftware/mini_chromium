cmake_minimum_required(VERSION 3.0)
project(mini_chromium VERSION 0.0.1)

set(
    sources
    base/atomicops.h
    base/atomicops_internals_atomicword_compat.h
    base/atomicops_internals_portable.h
    base/atomicops_internals_x86_msvc.h
    base/auto_reset.h
    base/bit_cast.h
    base/compiler_specific.h
    base/debug/alias.cc
    base/debug/alias.h
    base/files/file_path.cc
    base/files/file_path.h
    base/files/file_util.h
    base/files/scoped_file.cc
    base/files/scoped_file.h
    base/format_macros.h
    base/lazy_instance.cc
    base/lazy_instance.h
    base/logging.cc
    base/logging.h
    base/macros.h
    base/memory/aligned_memory.cc
    base/memory/aligned_memory.h
    base/memory/ptr_util.h
    base/memory/scoped_policy.h
    base/numerics/safe_conversions.h
    base/numerics/safe_conversions_impl.h
    base/numerics/safe_math.h
    base/numerics/safe_math_impl.h
    base/rand_util.cc
    base/rand_util.h
    base/scoped_clear_errno.h
    base/scoped_generic.h
    base/stl_util.h
    base/strings/string16.cc
    base/strings/string16.h
    base/strings/string_number_conversions.cc
    base/strings/string_number_conversions.h
    base/strings/string_piece.h
    base/strings/string_util.h
    base/strings/stringprintf.cc
    base/strings/stringprintf.h
    base/strings/sys_string_conversions.h
    base/strings/utf_string_conversion_utils.cc
    base/strings/utf_string_conversion_utils.h
    base/strings/utf_string_conversions.cc
    base/strings/utf_string_conversions.h
    base/synchronization/condition_variable.h
    base/synchronization/lock.cc
    base/synchronization/lock.h
    base/synchronization/lock_impl.h
    base/sys_byteorder.h
    base/template_util.h
    base/third_party/icu/icu_utf.cc
    base/third_party/icu/icu_utf.h
    base/threading/thread_local_storage.cc
    base/threading/thread_local_storage.h
    build/build_config.h
)

if(WIN32)
  list(
      APPEND
      sources
      base/strings/string_util_win.cc
      base/strings/string_util_win.h
      base/synchronization/lock_impl_win.cc
      base/threading/thread_local_storage_win.cc
  )
else()
  list(
      APPEND
      sources
      base/files/file_util_posix.cc
      base/posix/eintr_wrapper.h
      base/posix/safe_strerror.cc
      base/posix/safe_strerror.h
      base/strings/string_util_posix.h
      base/synchronization/condition_variable_posix.cc
      base/synchronization/lock_impl_posix.cc
      base/threading/thread_local_storage_posix.cc
  )
endif()

if(APPLE)
  list(
      APPEND
      sources
      base/atomicops_internals_mac.h
      base/mac/close_nocancel.cc
      base/mac/foundation_util.h
      base/mac/foundation_util.mm
      base/mac/mach_logging.cc
      base/mac/mach_logging.h
      base/mac/scoped_cftyperef.h
      base/mac/scoped_ioobject.h
      base/mac/scoped_launch_data.h
      base/mac/scoped_mach_port.cc
      base/mac/scoped_mach_port.h
      base/mac/scoped_mach_vm.cc
      base/mac/scoped_mach_vm.h
      base/mac/scoped_nsautorelease_pool.h
      base/mac/scoped_nsautorelease_pool.mm
      base/mac/scoped_nsobject.h
      base/mac/scoped_typeref.h
      base/strings/sys_string_conversions_mac.mm
  )
endif()

add_library(base ${sources})
target_include_directories(
    base PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>"
)

if(MSVC)
  target_compile_definitions(
      base
      PUBLIC
      "_HAS_EXCEPTIONS=0"
      "_CRT_SECURE_NO_WARNINGS"
      "NOMINMAX"
      "WIN32_LEAN_AND_MEAN"
      "_UNICODE"
      "UNICODE"
  )
endif()

string(COMPARE EQUAL "${CMAKE_SYSTEM_NAME}" "Linux" is_linux)
if(is_linux)
  target_compile_definitions(base PUBLIC "_FILE_OFFSET_BITS=64")
endif()

if(APPLE)
  target_link_libraries(base PUBLIC "-framework CoreFoundation")
  target_link_libraries(base PUBLIC "-framework CoreGraphics")
  target_link_libraries(base PUBLIC "-framework CoreText")
  target_link_libraries(base PUBLIC "-framework Foundation")
  target_link_libraries(base PUBLIC "-framework Security")
endif()

####
# Installation (https://github.com/forexample/package-example)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS base
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY "./base"
    DESTINATION "${include_install_dir}"
    FILES_MATCHING PATTERN "*.h"
)

install(
    FILES "build/build_config.h"
    DESTINATION "${include_install_dir}/build"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
