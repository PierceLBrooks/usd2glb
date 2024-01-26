cmake_minimum_required(VERSION 3.21)
project(usd2glb)

set(SOURCES
${CMAKE_CURRENT_LIST_DIR}/crc64/crc64.cpp
${CMAKE_CURRENT_LIST_DIR}/main.cpp
${CMAKE_CURRENT_LIST_DIR}/Image.h
)

set(INCLUDE_DIR
${CMAKE_CURRENT_LIST_DIR}/tinyusdz/src
${CMAKE_CURRENT_LIST_DIR}/tinyusdz/src/external
${CMAKE_CURRENT_LIST_DIR}/tinyusdz/src/external/jsonhpp/nlohmann
${CMAKE_CURRENT_LIST_DIR}/glm/glm
${CMAKE_CURRENT_LIST_DIR}/crc64
${CMAKE_CURRENT_LIST_DIR}
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if (WIN32)
set(DEFINES ${DEFINES}
-D"_CRT_SECURE_NO_DEPRECATE"
-D"_SCL_SECURE_NO_DEPRECATE"
-D"_CRT_SECURE_NO_WARNINGS"
)
else()
add_compile_options(-fPIC)
endif()

include_directories(${INCLUDE_DIR})
add_definitions(${DEFINES})
add_executable(usd2glb ${SOURCES})
add_custom_command(TARGET usd2glb
PRE_BUILD COMMENT "Building tinyusdz"
COMMAND ${CMAKE_COMMAND} -G ${CMAKE_GENERATOR} -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} ${CMAKE_CURRENT_LIST_DIR}/tinyusdz
COMMAND ${CMAKE_COMMAND} --build ${CMAKE_CURRENT_LIST_DIR}/tinyusdz-build --target install
WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/tinyusdz-build
)
install(TARGETS usd2glb)
target_compile_features(usd2glb PRIVATE cxx_std_17)
file(GLOB LIBRARIES ${CMAKE_INSTALL_PREFIX}/lib/*${CMAKE_STATIC_LIBRARY_SUFFIX})
list(LENGTH LIBRARIES SIZE)
if (SIZE GREATER 0)
list(GET LIBRARIES 0 LIBRARY)
add_library(tinyusdz_static STATIC IMPORTED)
set_target_properties(tinyusdz_static PROPERTIES IMPORTED_LOCATION ${LIBRARY})
target_link_libraries(usd2glb tinyusdz_static)
else()
message(WARNING "No tinyusdz yet")
endif()

