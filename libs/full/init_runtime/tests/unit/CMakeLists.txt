# Copyright (c) 2020-2026 The STE||AR-Group
#               2011 Bryce Adelstein-Lelbach
#
# SPDX-License-Identifier: BSL-1.0
# Distributed under the Boost Software License, Version 1.0. (See accompanying
# file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

set(tests runtime_type parcel_pool shutdown_suspended_thread
          start_stop_callbacks
)

if(HPX_WITH_DISTRIBUTED_RUNTIME)
  set(tests ${tests} handled_exception unhandled_exception)
endif()

if(HPX_WITH_SUPERVISION)
  set(tests ${tests} force_disconnect)

  # cmake-format: off
  set(force_disconnect_PARAMETERS LOCALITIES 3)
  set(force_disconnect_FLAGS
      COMPILE_FLAGS -DHPX_HAVE_RUN_MAIN_EVERYWHERE
      DEPENDENCIES supervision_dispatch_component
  )
  # cmake-format: on
endif()

set(unhandled_exception_PARAMETERS FAILURE_EXPECTED)

foreach(test ${tests})
  set(sources ${test}.cpp)

  set(${test}_PARAMETERS ${${test}_PARAMETERS} THREADS_PER_LOCALITY 4)

  source_group("Source Files" FILES ${sources})

  set(folder_name "Tests/Unit/Modules/Full/InitRuntime")

  add_hpx_executable(
    ${test}_test INTERNAL_FLAGS
    SOURCES ${sources} ${${test}_FLAGS}
    EXCLUDE_FROM_ALL
    HPX_PREFIX ${HPX_BUILD_PREFIX}
    FOLDER ${folder_name}
  )

  add_hpx_unit_test("modules.init_runtime" ${test} ${${test}_PARAMETERS})

  # MSVC V1951 and V1952 expose ICE compiling this in C++20 module mode
  if(HPX_WITH_CXX_MODULES
     AND (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
     AND (MSVC_VERSION LESS_EQUAL 1952)
     AND (test STREQUAL "force_disconnect")
  )
    target_compile_definitions(
      ${test}_test PRIVATE HPX_HAVE_FORCE_NO_CXX_MODULES
    )
  endif()
endforeach()
