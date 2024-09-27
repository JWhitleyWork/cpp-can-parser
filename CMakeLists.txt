cmake_minimum_required(VERSION 3.7.0)

project(CPP-CAN-Parser
  VERSION 1.0.1
  LANGUAGES C CXX
)

include(CTest)

set(CPP_CAN_PARSER_INCLUDE_DIRECTORY_PRIVATE 
  ${CMAKE_CURRENT_LIST_DIR}/src/parsing
)

set(CPP_CAN_PARSER_SRC_FILES
  src/models/CANDatabase.cpp
  src/models/CANFrame.cpp
  src/models/CANSignal.cpp
  src/parsing/DBCParser.cpp
  src/parsing/ParsingUtils.cpp
  src/parsing/Tokenizer.cpp
  src/analysis/CANFrameAnalysis.cpp
)

set(CPP_CAN_PARSER_COMPILATION_TYPE SHARED)

if(CPP_CAN_PARSER_USE_STATIC)
  set(CPP_CAN_PARSER_COMPILATION_TYPE STATIC)
endif()

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS True)

add_library(cpp-can-parser ${CPP_CAN_PARSER_COMPILATION_TYPE} 
  ${CPP_CAN_PARSER_SRC_FILES}
)
target_compile_features(cpp-can-parser PUBLIC cxx_std_17)
set_target_properties(cpp-can-parser PROPERTIES CXX_EXTENSIONS OFF)
target_include_directories(cpp-can-parser 
  PUBLIC
    # Different paths for includes for build and install phase supported
    # via INSTALL_INTERFACE and BUILD_INTERFACE generator expressions.
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
    $<INSTALL_INTERFACE:include/cpp-can-parser>
  PRIVATE
    ${CPP_CAN_PARSER_INCLUDE_DIRECTORY_PRIVATE}
)
set_target_properties(cpp-can-parser PROPERTIES
  VERSION ${cpp-can-parser_VERSION}
  SOVERSION ${cpp-can-parser_VERSION_MAJOR}
  PUBLIC_HEADER "include/cpp-can-parser/cpp-can-parser.h"
)

add_executable(can-parse 
  utils/can-parse/can-parse.cpp
  utils/can-parse/print-frame.cpp
  utils/can-parse/print-single-frame.cpp
  utils/can-parse/check-frame.cpp
)
target_compile_features(can-parse PUBLIC cxx_std_17)
set_target_properties(can-parse PROPERTIES CXX_EXTENSIONS OFF)
target_link_libraries(can-parse cpp-can-parser)

set(CMAKE_INSTALL_CMAKEDIR share/cpp-can-parser/cmake)

install(DIRECTORY include/cpp-can-parser/
  DESTINATION include/cpp-can-parser
)

install(
  TARGETS cpp-can-parser can-parse
  EXPORT cpp-can-parserTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
  PUBLIC_HEADER DESTINATION include
)

install(EXPORT cpp-can-parserTargets
  FILE cpp-can-parserTargets.cmake
  NAMESPACE CPPCANParser::
  DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(cpp-can-parserConfigVersion.cmake
  COMPATIBILITY AnyNewerVersion
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/cpp-can-parserConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
)

install(FILES cpp-can-parserConfig.cmake
  DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
)

CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY
)

ADD_CUSTOM_TARGET(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
)

if(BUILD_TESTING)
  file(COPY tests/dbc-files/
    DESTINATION dbc-files/
  )
  
  add_executable(cpc-test-parsing
    tests/test-parsing.cpp
  )
  target_link_libraries(cpc-test-parsing PUBLIC cpp-can-parser)

  add_test(NAME cpc-test-parsing
    COMMAND cpc-test-parsing
  )

  add_test(NAME cpc-checkframe-1
    COMMAND can-parse checkframe dbc-files/single-frame-1.dbc
  )

  add_test(NAME cpc-checkframe-2
    COMMAND can-parse checkframe dbc-files/single-frame-2.dbc
  )

  add_test(NAME cpc-checkframe-big-endian-1
    COMMAND can-parse checkframe 294 dbc-files/big-endian-1.dbc
  )

  add_test(NAME cpc-checkframe-big-endian-2
    COMMAND can-parse checkframe 1807 dbc-files/big-endian-1.dbc
  )
  
  add_test(NAME cpc-checkframe-big-endian-3
    COMMAND can-parse checkframe 1800 dbc-files/big-endian-1.dbc
  )
  
  add_test(NAME cpc-checkframe-decimal-scale-1
    COMMAND can-parse checkframe dbc-files/decimal-scale-1.dbc
  )
endif()
