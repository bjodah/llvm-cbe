IF (NOT DEFINED LLVM_VERSION_MAJOR)
  # Standalone build: requires both LLVM and Clang CMake packages
  cmake_minimum_required(VERSION 3.20.0)
  project(CBackendPlugin)
  find_package(LLVM REQUIRED CONFIG)
  if (LLVM_VERSION_MAJOR LESS 20 OR LLVM_VERSION_MAJOR GREATER 21)
    message(FATAL_ERROR "CBackendPlugin supports LLVM 20 and 21; found LLVM ${LLVM_PACKAGE_VERSION}")
  endif()
  find_package(Clang REQUIRED CONFIG)
  list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
  include(AddLLVM)
  set(CMAKE_CXX_STANDARD 17)
  include_directories(${LLVM_INCLUDE_DIRS} ${CLANG_INCLUDE_DIRS})
  add_definitions(${LLVM_DEFINITIONS})
else()
  find_package(Clang REQUIRED CONFIG)
  include_directories(${CLANG_INCLUDE_DIRS})
endif()

# Internal Clang CodeGen headers (ABIInfo.h, TargetInfo.h, etc.) are not
# exported; derive path from the LLVM source include directory.
if(DEFINED LLVM_PROJECT_SOURCE_DIR AND EXISTS "${LLVM_PROJECT_SOURCE_DIR}/clang/lib/CodeGen/ABIInfo.h")
  set(_clang_codegen_dir "${LLVM_PROJECT_SOURCE_DIR}/clang/lib/CodeGen")
else()
  get_filename_component(_llvm_src "${LLVM_MAIN_INCLUDE_DIR}" DIRECTORY)
  set(_clang_codegen_dir "${_llvm_src}/../clang/lib/CodeGen")
endif()
include_directories(${_clang_codegen_dir})

add_llvm_library(CBackendPlugin MODULE
  CBackend.cpp

  LINK_LIBS
  clangAST
  clangBasic
  clangCodeGen
  LLVMSupport
)

# Plugins must not have undefined symbols
set_target_properties(CBackendPlugin PROPERTIES
  CXX_VISIBILITY_PRESET hidden
)
