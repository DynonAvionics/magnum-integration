#
#   This file is part of Magnum.
#
#   Copyright © 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019,
#               2020, 2021, 2022, 2023, 2024, 2025
#             Vladimír Vondruš <mosra@centrum.cz>
#   Copyright © 2018 Jonathan Hale <squareys@googlemail.com>
#
#   Permission is hereby granted, free of charge, to any person obtaining a
#   copy of this software and associated documentation files (the "Software"),
#   to deal in the Software without restriction, including without limitation
#   the rights to use, copy, modify, merge, publish, distribute, sublicense,
#   and/or sell copies of the Software, and to permit persons to whom the
#   Software is furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included
#   in all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
#   DEALINGS IN THE SOFTWARE.
#

# Generate version header. If Git is found and this is a Git working copy,
# extract values from there, otherwise use just MAGNUMINTEGRATION_VERSION_YEAR
# / MONTH that are set in project root CMakeLists.
find_package(Git)
if(Git_FOUND)
    # Match only tags starting with `v`, always use the long format so we have
    # a commit hash also on a tagged version
    execute_process(COMMAND ${GIT_EXECUTABLE} describe --match "v*" --long
        OUTPUT_VARIABLE MAGNUMINTEGRATION_VERSION_STRING
        RESULT_VARIABLE _MAGNUMINTEGRATION_VERSION_RESULT
        ERROR_VARIABLE _MAGNUMINTEGRATION_VERSION_ERROR
        # Otherwise this gets executed in the build dir, which might be inside
        # a totally different Git working copy. But that's not enough, if it
        # would be run in ${CMAKE_CURRENT_SOURCE_DIR}, on Windows the
        # `--match "v*"` would get expanded to the `versionIntegration.h.cmake`
        # file, causing Git to obviously not find any such tag. The reason is
        # probably that on Windows the wildcard expansion is done on the
        # application side and not by the shell, thus being performed even
        # though CMake docs say `execute_process()` doesn't involve a shell.
        # The package/git directory is thus dedicated for that operation,
        # *guaranteed* to never contain any file starting with `v` (or `V` for
        # that matter because, again, HELLO WINDOWS).
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/package/git
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    if(MAGNUMINTEGRATION_VERSION_STRING MATCHES "^v([0-9][0-9][0-9][0-9])\\.0?([0-9][0-9]?)-([0-9]+)-g([a-f0-9]+)$")
        set(MAGNUMINTEGRATION_VERSION_YEAR ${CMAKE_MATCH_1})
        set(MAGNUMINTEGRATION_VERSION_MONTH ${CMAKE_MATCH_2})
        set(MAGNUMINTEGRATION_VERSION_COMMIT ${CMAKE_MATCH_3})
        # Otherwise if commit is 0, it would #undef the variable
        set(MAGNUMINTEGRATION_VERSION_COMMIT " ${MAGNUMINTEGRATION_VERSION_COMMIT}")
        set(MAGNUMINTEGRATION_VERSION_HASH ${CMAKE_MATCH_4})
    elseif(_MAGNUMINTEGRATION_VERSION_RESULT EQUAL 0)
        message(WARNING "Can't match Git version from ${MAGNUMINTEGRATION_VERSION_STRING}, generating a fallback versionIntegration.h instead")
    else()
        message(WARNING "Can't match Git version, generating a fallback versionIntegration.h instead: ${_MAGNUMINTEGRATION_VERSION_ERROR}")
    endif()
endif()
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/versionIntegration.h.cmake
               ${CMAKE_CURRENT_BINARY_DIR}/versionIntegration.h)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/versionIntegration.h DESTINATION ${MAGNUM_INCLUDE_INSTALL_DIR})

if(MAGNUM_WITH_BULLET)
    add_subdirectory(BulletIntegration)
endif()

if(MAGNUM_WITH_DART)
    add_subdirectory(DartIntegration)
endif()

if(MAGNUM_WITH_EIGEN)
    add_subdirectory(EigenIntegration)
endif()

if(MAGNUM_WITH_GLM)
    add_subdirectory(GlmIntegration)
endif()

if(MAGNUM_WITH_IMGUI)
    add_subdirectory(ImGuiIntegration)
endif()

if(MAGNUM_WITH_OVR)
    add_subdirectory(OvrIntegration)
endif()

if(MAGNUM_BUILD_TESTS)
    add_subdirectory(Test ${EXCLUDE_FROM_ALL_IF_TEST_TARGET})
endif()
