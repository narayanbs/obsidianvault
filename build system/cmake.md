
## Compilation & Build Management

If you’ve ever tried to build a C++ project with multiple dependencies and cross- platform support, you know the pain. Makefiles become unwieldy, Visual Studio solutions don’t work on Linux, and don’t even get me started on trying to distribute your library for others to use.

 Today, we're going deep with a complex project - a complete game engine called ColumbaEngine (source code available here) with 80+ source files, multiple platforms (including web compilation), and a sophisticated build system.

### What Makes This Project Interesting?

#### Before we dive into CMake, let’s understand what we’re building:

- 80+ C++ source files organized in modules (ECS, Renderer, Audio, UI, etc.)
- Cross-platform: Windows, Linux, and Web (via Emscripten)
- Multiple dependencies: SDL2, OpenGL, FreeType, custom libraries
- Installable library: Other projects can use it with find_package()
- Example applications: Several games and tools


- Package generation: Creates .deb, .rpm, and other installers

#### This isn’t a toy project — it’s a real game engine that needs a robust build system.

### Project Structure: The Foundation

#### Here’s how our game engine is organized:

```
ColumbaEngine/
├── CMakeLists.txt # Main build file
├── src/
│ ├── Engine/ # Core engine code
│ │ ├── ECS/ # Entity-Component-System
│ │ ├── Renderer/ # Graphics rendering
│ │ └── Audio/ # Sound system
│ └── main.cpp # Editor application
├── examples/ # Example games
├── import/ # Vendored dependencies
├── cmake/ # CMake modules
└── test/ # Unit tests
```
#### The CMakeLists.txt is our blueprint. Let's build it step by step.

### Step 1: Project Setup and Configuration

#### Key concepts here:

```cmake
cmake_minimum_required(VERSION 3.18)
project(ColumbaEngine VERSION 1.0)
```
```cmake
# User-configurable options
option(BUILD_EXAMPLES "Build example applications" ON)
option(BUILD_STATIC_LIB "Build static library only" OFF)
option(ENABLE_TIME_TRACE "Add -ftime-trace to Clang builds" OFF)
```
```cmake
# Global settings
set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # For IDE support
set(CMAKE_CXX_STANDARD 17 )
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

- cmake_minimum_required: We use 3.18+ for modern features like precompiled

#### headers

- option(): Creates user-configurable boolean flags. Users can run cmake -
    DBUILD_EXAMPLES=OFF ..
- CMAKE_EXPORT_COMPILE_COMMANDS: Generates compile_commands.json for IDEs and

#### tools like clangd

### Step 2: The Dependency Challenge

```cmake
# Dependency paths - everything is self-contained
set(SDL2_DIR "import/SDL2-2.28.5")
set(SDL2MIXER_DIR "import/SDL2_mixer-2.6.3")
set(GLM_DIR "import/glm")
set(TASKFLOW_DIR "import/taskflow-3.6.0")
set(GTEST_DIR "import/googletest-1.14.0")
```
```cmake
# Configure dependencies before building them
set(TF_BUILD_EXAMPLES OFF) # Don't build taskflow examples
set(TF_BUILD_TESTS OFF) # Don't build taskflow tests
set(BUILD_SHARED_LIBS OFF) # Force static linking
set(SDL2_DISABLE_INSTALL ON) # We'll handle installation ourselves
```

Real projects have dependencies. Lots of them. Our game engine needs SDL2 for windowing, FreeType for text rendering, OpenGL for graphics, and more.

 We use a vendoring approach — including dependencies directly in our source tree: Why vendor dependencies?
 * `Pros`: Exact version control, works offline, simplified build
 * `Cons`: Larger repo size, manual updates

 For a game engine where stability is crucial, vendoring makes sense. For web services that need frequent security updates, you might prefer package managers.

### Deep Dive: Dependency Management Strategies

Choosing how to handle dependencies is one of the most critical architectural decisions in C++. Let’s compare the approaches:

### 1. Vendoring (Our Current Approach)

```cmake
# Everything is self-contained in import/
set(SDL2_DIR "import/SDL2-2.28.5")
set(GLM_DIR "import/glm")
add_subdirectory(${SDL2_DIR})
add_subdirectory(${GLM_DIR})
```
#### Advantages:

- Reproducible builds: Exact same versions everywhere
- Offline builds: No internet required after initial clone
- Control: Can patch dependencies if needed
- Simplicity: No external tools or package managers

#### Disadvantages:

- Repository size: Our import/ folder is 500MB+
- Update overhead: Manual process to update dependencies
- Security: Must manually track and update vulnerable dependencies
- License complexity: Must distribute all licenses

### 2. Package Managers (Conan, vcpkg)


```cmake
# With Conan
include(conan)
conan_cmake_run(REQUIRES
SDL2/2.28. 5
glm/0.9.9.
BASIC_SETUP CMAKE_TARGETS
BUILD missing
)
target_link_libraries(ColumbaEngine PRIVATE CONAN_PKG::SDL2 CONAN_PKG::glm)
```
#### Advantages:

- Smaller repos: Dependencies downloaded on-demand
- Easy updates: conan install --update
- Binary packages: Pre-built binaries for faster builds
- Ecosystem: Thousands of packages available

#### Disadvantages:

- External dependency: Requires internet and package manager
- Version conflicts: Diamond dependency problems
- Platform support: Not all packages support all platforms
- Learning curve: Another tool to learn and maintain

### 3. FetchContent (Modern CMake)

```cmake
include(FetchContent)
```
```cmake
FetchContent_Declare(glm
GIT_REPOSITORY https://github.com/g-truc/glm.git
GIT_TAG 0.9.9.
```

###### GIT_SHALLOW TRUE

###### )

```cmake
FetchContent_Declare(SDL
URL https://github.com/libsdl-org/SDL/releases/download/release-2.28. 5 /SDL2-
URL_HASH SHA256= 332 cb37d0be20cb9541739c61f79bae5a477427d79ae85e352089afdaf6666e
)
```
```cmake
FetchContent_MakeAvailable(glm SDL2)
target_link_libraries(ColumbaEngine PRIVATE glm SDL2::SDL2)
```
#### Advantages:

- CMake native: No external tools required
- Flexible sources: Git repos, URLs, local paths
- Version pinning: Exact commits/tags
- Cross-platform: Works everywhere CMake works

#### Disadvantages:

- Build time: Downloads and builds on first configure
- Internet required: At least for first build
- No binary caching: Always builds from source

### Hybrid Approach: The Best of Both Worlds

#### For ColumbaEngine, we could evolve to a hybrid approach:

```cmake
# Critical, stable dependencies: Vendor them
set(SDL2_DIR "import/SDL2-2.28.5")
add_subdirectory(${SDL2_DIR})
```
```cmake
# Development/testing dependencies: FetchContent
if(BUILD_TESTS)
include(FetchContent)
FetchContent_Declare(googletest
```

```cmake
GIT_REPOSITORY https://github.com/google/googletest.git
GIT_TAG v1.14.
GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(googletest)
endif()
```
```cmake
# Optional dependencies: find_package with fallback
find_package(Doxygen)
if(NOT Doxygen_FOUND AND ENABLE_DOCS)
FetchContent_Declare(doxygen
URL https://github.com/doxygen/doxygen/releases/download/Release_1_9_8/doxygen-
)
FetchContent_MakeAvailable(doxygen)
endif()
```
### Step 3: Cross-Platform Reality Check

#### Our engine supports three platforms, with different flags and need:

- Native (Windows/Linux): Build everything from source
- Emscripten (Web): Use system-provided libraries

```cmake
if(${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
# Web build - use Emscripten's built-in libraries
set(USE_FLAGS "-O3 -sUSE_SDL=2 -sUSE_SDL_MIXER=2 -sUSE_FREETYPE=1 -fwasm-exceptions"
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${USE_FLAGS}")
else()
# Native build - compile dependencies from source
add_subdirectory(${SDL2_DIR})
add_subdirectory(${SDL2MIXER_DIR})
add_subdirectory(${GLEW_DIR})
add_subdirectory(${TTF_DIR})
endif()
```
```cmake
# GLM is header-only, works everywhere
add_subdirectory(${GLM_DIR})
```

#### Platform detection patterns:

-  CMAKE_SYSTEM_NAME: Target platform (Windows, Linux, Darwin, Emscripten)
-  CMAKE_HOST_SYSTEM: Build machine platform
-  Use generator expressions for conditional compilation:
    $<$<PLATFORM_ID:Windows>:WIN32_CODE>

### Step 4: Creating the Main Target

#### Now for the heart of our build — the engine library itself:

```cmake
if(${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
# Web build - use Emscripten's built-in libraries
set(USE_FLAGS "-O3 -sUSE_SDL=2 -sUSE_SDL_MIXER=2 -sUSE_FREETYPE=1 -fwasm-exceptions"
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${USE_FLAGS}")
else()
# Native build - compile dependencies from source
add_subdirectory(${SDL2_DIR})
add_subdirectory(${SDL2MIXER_DIR})
add_subdirectory(${GLEW_DIR})
add_subdirectory(${TTF_DIR})
endif()
```
```cmake
# GLM is header-only, works everywhere
add_subdirectory(${GLM_DIR})
```

 Precompiled headers can cut build times by 50%+ in large projects. They compile commonly used headers once and reuse the result.

### Precompiled Headers — The Build Speed Game Changer

Precompiled headers (PCH) are one of the most impactful optimizations for C++ build times, yet they’re often overlooked. Let’s understand how they work:
#### The Problem: Header Parsing Overhead

```cmake
// Every .cpp file typically includes these
#include<iostream> // ~15,000 lines when fully expanded
#include<vector> // ~8,000 lines
#include<string> // ~12,000 lines
#include<memory> // ~6,000 lines
#include<SDL2/SDL.h> // ~25,000 lines
// Total: ~66,000 lines to parse per source file!
```
 
 With 80 source files, that’s 5.2 million lines of redundant header parsing!

#### The Solution: Precompiled Headers

```make
target_precompile_headers (ColumbaEngine PUBLIC src/Engine/stdafx.h)
```
#### Our stdafx.h contains the most commonly used headers:

```make
// stdafx.h - precompiled header
#pragma once
```
```cmake
// Standard library
#include<iostream>
#include<vector>
```

```cmake
#include<string>
#include<memory>
#include<unordered_map>
#include<algorithm>
```
```cmake
// Third-party libraries
#include<SDL2/SDL.h>
#include<glm/glm.hpp>
#include<glm/gtc/matrix_transform.hpp>
```
```cmake
// Engine fundamentals used everywhere
#include"types.h"
#include"logger.h"
#include"configuration.h"
```
#### How It Works:

#### 1. Compilation: CMake compiles stdafx.h once into stdafx.h.gch (GCC) or

#### stdafx.pch (MSVC)

#### 2. Reuse: Every source file automatically uses the precompiled version

#### 3. Speed: 66,000 lines → 0 lines to parse per file

#### Build Time Results (80 source files):

- Without PCH: ~8 minutes clean build
- With PCH: ~3 minutes clean build
- Incremental: Single file changes drop from 15s to 3s

#### PUBLIC vs PRIVATE PCH:

```cmake
# PUBLIC: Users of your library get the PCH benefits too
target_precompile_headers(ColumbaEngine PUBLIC stdafx.h)
```
```
# PRIVATE: Only internal compilation uses PCH
target_precompile_headers(ColumbaEngine PRIVATE stdafx.h)
```

#### Best Practices for PCH:

```cmake
// Good PCH content: Stable, frequently used headers
#include<vector> // Used in 90% of files
#include<SDL2/SDL.h> // Core dependency
```
```
// Bad PCH content: Frequently changing headers
#include"GameLogic.h" // Changes often, invalidates PCH
#include"debug_temp.h" // Temporary debugging code
```
#### PCH Pitfalls:

#### 1. Overuse: Adding changing headers defeats the purpose

#### 2. Platform differences: MSVC vs GCC handle PCH differently

#### 3. Dependencies: PCH creates implicit dependencies between files

### Step 5: Usage Requirements — The Secret Sauce

This is where modern CMake really shines. Instead of users manually figuring out  include paths and linking, we specify usage requirements:

```cmake
target_include_directories(ColumbaEngine PUBLIC
# When building this library
$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/Engine>
$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/GameElements>
# When using installed version
$<INSTALL_INTERFACE:include/ColumbaEngine>
$<INSTALL_INTERFACE:include/ColumbaEngine/GameElements>
)
```
#### Generator expressions ($<...>) are evaluated during build generation:

- BUILD_INTERFACE: Only when building this project
- INSTALL_INTERFACE: Only when using the installed version
- $<CONFIG:Debug>: Only in Debug builds
- $<PLATFORM_ID:Windows>: Only on Windows

### Why Generator Expressions Matter

Let’s understand why this is so powerful. Consider this traditional approach:

```cmake
# The old, problematic way
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDEBUG_MODE")
endif()
```

 Problems with this approach:

 1. Build-time evaluation: The condition is checked when CMake configures, not when you build
 2. Global pollution: Affects ALL targets in the project
 3. Multi-config generators: Breaks with Visual Studio, Xcode which support

### multiple configs

#### Generator expressions solve this:

```cmake
# Modern, correct approach
target_compile_definitions(MyTarget PRIVATE
$<$<CONFIG:Debug>:DEBUG_MODE>
$<$<CONFIG:Release>:NDEBUG>
$<$<PLATFORM_ID:Windows>:WIN32_LEAN_AND_MEAN>
)
```

 This is evaluated per-target, per-configuration, at build time. Much more flexible and robust.

### Common Generator Expression Patterns

```cmake
# Conditional compilation flags
target_compile_options(MyTarget PRIVATE
$<$<CXX_COMPILER_ID:GNU,Clang>:-Wall -Wextra>
$<$<CXX_COMPILER_ID:MSVC>:/W4>
)
```
```cmake
# Debug vs Release libraries
target_link_libraries(MyTarget PRIVATE
$<$<CONFIG:Debug>:MyLib_d>
$<$<CONFIG:Release>:MyLib>
)
```
```cmake
# Platform-specific linking
target_link_libraries(MyTarget PRIVATE
$<$<PLATFORM_ID:Windows>:ws2_32>
$<$<PLATFORM_ID:Linux>:pthread>
)
```
### Step 6: Dependency Linking — Getting Scope Right

#### Here’s a crucial concept: PUBLIC vs PRIVATE vs INTERFACE

```cmake
if (${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
# Web build
target_link_libraries(ColumbaEngine PUBLIC glm)
else()
# Native build - note the scoping!
target_link_libraries(ColumbaEngine PUBLIC
SDL2::SDL2-static # Users need SDL2 headers
glm # Header-only math library
OpenGL::GL # Graphics API
)
```
```cmake
target_link_libraries(ColumbaEngine PRIVATE
libglew_static # Internal OpenGL extension loading
```

```cmake
freetype # Internal text rendering
)
endif()
```
#### Scope meanings:

- PUBLIC: “I use this, and my users will need it too”
- PRIVATE: “I use this internally, but my users don’t need to know”
- INTERFACE: “I don’t use this, but my users will need it”

Get this right, and users of your library automatically get the right dependencies.
 Get it wrong, and you’ll have angry developers filing issues.

### Understanding Transitive Dependencies

Let’s look at a real example from our engine. Imagine this dependency chain:

```cmake
MyGame → ColumbaEngine → SDL2 → OpenGL
```
 If we declare SDL2 as PRIVATE to ColumbaEngine:

```cmake
target_link_libraries (ColumbaEngine PRIVATE SDL2::SDL2-static)
```

**Problem: MyGame won't be able to use SDL2 types in the public API. If  ColumbaEngine's headers include <SDL2/SDL.h>, users get compile errors.

#### If we declare SDL2 as PUBLIC:

```cmake
target_link_libraries (ColumbaEngine PUBLIC SDL2::SDL2-static)
```

 Result: MyGame automatically gets SDL2 headers and libraries. Perfect for our engine where users need SDL2 types.

### The INTERFACE Scope Mystery

INTERFACE is the trickiest to understand. It means "I don't use this, but my users will."

#### Real-world example: Header-only libraries with dependencies

```cmake
# HeaderOnlyMath library depends on Eigen, but is itself header-only
add_library(HeaderOnlyMath INTERFACE)
target_link_libraries(HeaderOnlyMath INTERFACE Eigen3::Eigen)
```
```cmake
# Users of HeaderOnlyMath automatically get Eigen
add_executable(MyApp main.cpp)
target_link_libraries(MyApp PRIVATE HeaderOnlyMath) # Gets Eigen too!
```
### Mixing Scopes: The Real World

```cmake
target_link_libraries(ColumbaEngine
# Users need these APIs
PUBLIC
SDL2::SDL2-static # Window/input handling in public API
glm # Math types in public headers
OpenGL::GL # Graphics API exposed to users
```
```cmake
# Internal implementation details
PRIVATE
libglew_static # OpenGL extension loading (wrapped)
freetype # Text rendering (abstracted away)
${CMAKE_DL_LIBS} # Dynamic library loading (platform detail)
)
```

#### Most real projects use mixed scopes:

### Step 7: Multiple Executables and Examples

#### A game engine isn’t just a library — it includes tools and examples:

```cmake
if(NOT BUILD_STATIC_LIB AND BUILD_EXAMPLES)
# Main editor application
add_executable(ColumbaEngineEditor src/main.cpp)
target_sources(ColumbaEngineEditor PRIVATE
src/application.cpp
src/Editor/Gui/inspector.cpp
src/Editor/Gui/projectmanager.cpp
)
target_link_libraries(ColumbaEngineEditor PRIVATE ColumbaEngine)
```
```cmake
# Game examples
add_executable(GameOff examples/GameOff/main.cpp)
target_sources(GameOff PRIVATE
examples/GameOff/application.cpp
examples/GameOff/character.cpp
examples/GameOff/inventory.cpp
)
target_link_libraries(GameOff PRIVATE ColumbaEngine)
endif()
```
 Pattern: Create multiple executables that all link to your main library. This way, the library code is compiled once and reused.

### Step 8: Testing Infrastructure

#### Professional projects need tests:

```cmake
if(NOT BUILD_STATIC_LIB)
enable_testing()
add_subdirectory(${GTEST_DIR})
```
```
add_executable(t1 test/maintest.cc)
target_sources(t1 PRIVATE
test/collision2d.cc
```

```cmake
test/ecssystem.cc
test/interpreter.cc
test/renderer.cc
)
target_link_libraries(t1 PRIVATE gtest gtest_main ColumbaEngine)
```
```cmake
# Auto-discover tests
include(GoogleTest)
gtest_discover_tests(t1)
endif()
```

gtest_discover_tests() automatically finds all test cases and creates individual CTest entries. Run with ctest or make test.

### Common Build Pitfalls

### 1. Global Variables vs Target Properties

```cmake
# BAD: Affects everything globally
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDEBUG")
```
```cmake
# GOOD: Only affects specific target
target_compile_definitions(MyTarget PRIVATE DEBUG)
```
### 2. Not Using Generator Expressions

```cmake
# BAD: Debug symbols in release builds
target_compile_options(MyTarget PRIVATE -g)
```
```cmake
# GOOD: Only in debug builds
target_compile_options(MyTarget PRIVATE $<$<CONFIG:Debug>:-g>)
```
### 3. Wrong Dependency Scopes


```cmake
# If users of your library need OpenGL headers:
target_link_libraries(MyLib PUBLIC OpenGL::GL)
```
```cmake
# If only your implementation uses OpenGL:
target_link_libraries(MyLib PRIVATE OpenGL::GL)
```
### What We’ve Achieved: Build System Results

 After implementing all these concepts, here’s what we achieved:  Developer Experience:

```cmake
# Simple build
git clone https://github.com/user/ColumbaEngine
cd ColumbaEngine
mkdir build && cd build
cmake ..
make -j
```
```cmake
# Custom configuration
cmake -DBUILD_EXAMPLES=OFF -DCMAKE_BUILD_TYPE=Release ..
```
```
# Run tests
ctest
```
#### Cross-platform Support:

- Same CMakeLists.txt works on Windows, Linux, and web
- Automatic dependency resolution per platform
- Platform-specific optimizations where needed

#### Build Performance:

- Precompiled headers cut build times by 60%


- Parallel compilation across all targets
- Incremental builds under 10 seconds for single-file changes

