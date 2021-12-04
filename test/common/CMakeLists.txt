file(GLOB_RECURSE CE_TEST_COMMON_SRC "src/**.c" "src/**.cpp")
list(LENGTH CE_TEST_COMMON_SRC CE_TEST_COMMON_SRC_LEN)

#Local build variables
set(CE_TEST_COMMON_TARGET constemu_test_common)

#If there areno sources, set up a header-only library
#Otherwise, set up a shared library
if(CE_TEST_COMMON_SRC_LEN EQUAL 0)
 #Set up header-only library
 add_library(${CE_TEST_COMMON_TARGET} INTERFACE)
 set(CE_TEST_COMMON_OPTION_SCOPE INTERFACE)
else()
 #Set up shared library
 add_library(${CE_TEST_COMMON_TARGET} SHARED ${CE_TEST_COMMON_SRC})
 set(CE_TEST_COMMON_OPTION_SCOPE PUBLIC)
endif()

target_include_directories(
 ${CE_TEST_COMMON_TARGET} ${CE_TEST_COMMON_OPTION_SCOPE} 
 include
 ${GOOGLE_TEST_INCLUDE_DIRS}
)
target_link_libraries(
 ${CE_TEST_COMMON_TARGET} ${CE_TEST_COMMON_OPTION_SCOPE}
 constemu
 ${GOOGLE_TEST_TARGET}
)
target_compile_options(
 ${CE_TEST_COMMON_TARGET} ${CE_TEST_COMMON_OPTION_SCOPE} 
 ${CE_COMPILE_FLAGS}
)
target_link_options(
 ${CE_TEST_COMMON_TARGET} ${CE_TEST_COMMON_OPTION_SCOPE} 
 ${CE_LINK_FLAGS}
)

#Global build variables, required for dependents
set(CE_TEST_COMMON_TARGET ${CE_TEST_COMMON_TARGET} PARENT_SCOPE) 
set(CE_TEST_COMMON_INCLUDE_DIRS "${${CE_TEST_COMMON_TARGET}_SOURCE_DIR}/include" PARENT_SCOPE)
