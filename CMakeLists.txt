cmake_minimum_required(VERSION 3.16)
project(libcsvcpp VERSION 0.1.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Legacy C library (untouched original implementation)
add_subdirectory(legacy)

# Modern C++ wrapper
add_subdirectory(csvcpp)

# Usage examples demonstrating modern API
add_subdirectory(examples)

# Test suite maintaining behavioral parity with legacy tests
add_subdirectory(tests)