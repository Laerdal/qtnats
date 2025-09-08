cmake_minimum_required(VERSION 3.22.3)

project(QtNats VERSION 0.1.0)

option(BUILD_QMLNATS "Build the QML NATS plugin (Qt6-only)" OFF)
option(BUILD_TESTING "Build unit tests" OFF)

set(default_build_type "Release")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_AUTOMOC ON)


file(GLOB SOURCES "src/*.cpp")
file(GLOB HEADERS "src/*.h")
add_library(${PROJECT_NAME} SHARED ${SOURCES} ${HEADERS})

find_package(Qt5 5.15 REQUIRED COMPONENTS Core)
target_link_libraries(${PROJECT_NAME} PRIVATE Qt::Core)

find_library(nats_library nats REQUIRED)
target_link_libraries(${PROJECT_NAME} PUBLIC ${nats_library})
get_filename_component(NATS_LIB_DIR  "${nats_library}" DIRECTORY)
get_filename_component(NATS_LIB_NAME "${nats_library}" NAME)
file(GLOB NATS_LIB_VARIANTS "${NATS_LIB_DIR}/${NATS_LIB_NAME}*")
install(FILES ${NATS_LIB_VARIANTS} DESTINATION ${vs_lib_path} PERMISSIONS ${install_permissions})

target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_LIST_DIR}/src)

# this has no effect on Windows due to https://gitlab.kitware.com/cmake/cmake/-/issues/19618
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION})

include(GenerateExportHeader)
GENERATE_EXPORT_HEADER(${PROJECT_NAME})

install(TARGETS ${PROJECT_NAME} DESTINATION ${vs_lib_path} PERMISSIONS ${install_permissions})

# ------------- unit tests ----------------------
if (BUILD_TESTING)
    enable_testing()

    add_executable(test_core test/test_core.cpp)
    add_test(NAME test_core COMMAND test_core)
    target_link_libraries(test_core PRIVATE qtnats Qt::Test)

    add_executable(test_jetstream test/test_jetstream.cpp)
    add_test(NAME test_jetstream COMMAND test_jetstream)
    target_link_libraries(test_jetstream PRIVATE qtnats Qt::Test)
endif()

if (BUILD_QMLNATS)
    if (${QT_VERSION_MAJOR} EQUAL 6)
        add_subdirectory(qml)
    else()
        message(FATAL_ERROR "NATS QML is supported only with Qt6")
    endif()
endif()
