cmake_minimum_required(VERSION 3.2)

if(NOT DEFINED ENV{TRAVIS_BUILD_NUMBER})
	set(ENV{TRAVIS_BUILD_NUMBER} 2)
endif()

project(cmake VERSION 0.1.$ENV{TRAVIS_BUILD_NUMBER})

if (DEFINED ENV{COMPUTERNAME})
	if (NOT DEFINED ENV{BOOST_ROOT} AND $ENV{COMPUTERNAME} STREQUAL "IZUBKOV")
		set(BOOST_ROOT "C:\\Program Files\\boost\\boost_1_70_0")
	endif()
endif()



find_package(Threads REQUIRED)
find_package(Boost COMPONENTS unit_test_framework REQUIRED)

configure_file(version.h.in ${PROJECT_SOURCE_DIR}/version.h)

add_executable(helloworld main.cpp)
add_library(helloworld_lib lib.cpp)
add_executable(test_version test_version.cpp)

set_target_properties(helloworld helloworld_lib test_version PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)

set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

target_link_libraries(helloworld
    helloworld_lib
    Threads::Threads
)

target_link_libraries(test_version
    ${Boost_LIBRARIES}
    helloworld_lib
)

if (MSVC)
    target_compile_options(helloworld PRIVATE
        /W4
    )
    target_compile_options(helloworld_lib PRIVATE
        /W4
    )
    target_compile_options(test_version PRIVATE
        /W4
    )
else ()
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld_lib PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()

install(TARGETS helloworld RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT roboter95@gmail.com)

set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)

include(CPack)

enable_testing()

add_test(helloworld_tests test_version)
