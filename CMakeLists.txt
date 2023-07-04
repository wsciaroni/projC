# CMakeList.txt : Top-level CMake project file, do global configuration
# and include sub-projects here.
#
cmake_minimum_required (VERSION 3.25)

project(ProjC
    VERSION 0.0.0
)

# GoogleTest requires at least C++14
set(CMAKE_CXX_STANDARD 17)

# To organize projects in VisualStudio:
if(MSVC)
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)
endif(MSVC)

include(cmake/EnableGTest.cmake)
enable_gtest()

# Include sub-projects.
include(cmake/IncludeProjA.cmake)
include(cmake/IncludeProjB.cmake)

##################### Order Dependent Calls
# Initialize A first at Rev 0.0.2
# Initialize B second at Rev 0.0.1
# include_proj_a()
# include_proj_b()
# This results in the following output:
#     Utility Lib A Version 0.0.2.
#     Core Lib A Version 0.0.2.
#     Utility Lib B Version 0.0.1.
#     Core Lib B Version 0.0.1.

# Initialize B first at Rev 0.0.1
# Initialize A second at Rev 0.0.2
# include_proj_b()
# include_proj_a()
# This results in the following output:
#     Utility Lib A Version 0.0.0.
#     Core Lib A Version 0.0.0.
#     Utility Lib B Version 0.0.1.
#     Core Lib B Version 0.0.1.

# Note that Rev 0.0.1 of project b specifies rev 0.0.0 of project A
# Because the FetchContent for A within B is reached first, the later call within the superbuild is ignored.

########## Order dependent but doing all of the declares first
include(FetchContent)

FetchContent_Declare(
    ProjB
    GIT_REPOSITORY https://github.com/wsciaroni/projB.git
    GIT_TAG 0.0.1
)

FetchContent_Declare(
    ProjA
    GIT_REPOSITORY https://github.com/wsciaroni/projA.git
    GIT_TAG 0.0.2
)

FetchContent_GetProperties(ProjB)
if(NOT ProjB_POPULATED)
    FetchContent_MakeAvailable(ProjB)
endif()

FetchContent_GetProperties(ProjA)
if(NOT ProjA_POPULATED)
    FetchContent_MakeAvailable(ProjA)
endif()

# This results in the following output:
#     Utility Lib A Version 0.0.2.
#     Core Lib A Version 0.0.2.
#     Utility Lib B Version 0.0.1.
#     Core Lib B Version 0.0.1.

# Notice how this solves the problem from below.
# Therefore, if you want explicit control of all of your sub builds, you should specify all
# the FetchContent_Declare calls in the superbuild before calling any of the MakeAvailable calls
