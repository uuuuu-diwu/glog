cmake_minimum_required (VERSION 3.0)

if (POLICY CMP0042)
  cmake_policy (SET CMP0042 NEW)
endif (POLICY CMP0042)

if (POLICY CMP0063)
  cmake_policy (SET CMP0063 NEW)
endif (POLICY CMP0063)

project(glog VERSION 0.5.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 11)
set (CPACK_PACKAGE_NAME glog)
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "Google logging library")
set (CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set (CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set (CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set (CPACK_PACKAGE_VERSION ${PROJECT_VERSION})

option (BUILD_SHARED_LIBS "Build shared libraries" ON)
option (PRINT_UNSYMBOLIZED_STACK_TRACES
  "Print file offsets in traces instead of symbolizing" OFF)
option (WITH_CUSTOM_PREFIX "Enable support for user-generated message prefixes" OFF)
option (WITH_GFLAGS "Use gflags" ON)
option (WITH_GTEST "Use googletest" ON)
option (WITH_PKGCONFIG "Enable pkg-config support" ON)
option (WITH_SYMBOLIZE "Enable symbolize module" ON)
option (WITH_THREADS "Enable multithreading support" ON)
option (WITH_TLS "Enable Thread Local Storage (TLS) support" ON)
option (WITH_UNWIND "Enable libunwind support" ON)

if (NOT WITH_UNWIND)
  set (CMAKE_DISABLE_FIND_PACKAGE_Unwind ON)
endif (NOT WITH_UNWIND)

if (NOT WITH_GTEST)
  set (CMAKE_DISABLE_FIND_PACKAGE_GTest ON)
endif (NOT WITH_GTEST)

if (NOT WITH_THREADS)
  set (CMAKE_DISABLE_FIND_PACKAGE_Threads ON)
endif (NOT WITH_THREADS)

set (CMAKE_C_VISIBILITY_PRESET hidden)
set (CMAKE_CXX_VISIBILITY_PRESET hidden)
set (CMAKE_POSITION_INDEPENDENT_CODE ON)
set (CMAKE_VISIBILITY_INLINES_HIDDEN ON)

list (APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include (CheckCXXCompilerFlag)
include (CheckCXXSourceCompiles)
include (CheckCXXSourceRuns)
include (CheckCXXSymbolExists)
include (CheckFunctionExists)
include (CheckIncludeFileCXX)
include (CheckLibraryExists)
include (CheckStructHasMember)
include (CheckTypeSize)
include (CMakePackageConfigHelpers)
include (CMakePushCheckState)
include (CPack)
include (CTest)
include (DetermineGflagsNamespace)
include (GenerateExportHeader)
include (GetCacheVariables)
include (GNUInstallDirs)

set (CMAKE_DEBUG_POSTFIX d)
set (CMAKE_THREAD_PREFER_PTHREAD 1)

find_package (GTest)

if (GTest_FOUND)
  set (HAVE_LIB_GTEST 1)
endif (GTest_FOUND)

if (WITH_GFLAGS)
  find_package (gflags 2.2.0)

  if (gflags_FOUND)
    set (HAVE_LIB_GFLAGS 1)
    determine_gflags_namespace (gflags_NAMESPACE)
  endif (gflags_FOUND)
endif (WITH_GFLAGS)

find_package (Threads)
find_package (Unwind)

if (Unwind_FOUND)
  set (HAVE_LIB_UNWIND 1)
  set (HAVE_UNWIND_H 1)
endif (Unwind_FOUND)

check_include_file_cxx (dlfcn.h HAVE_DLFCN_H)
check_include_file_cxx (execinfo.h HAVE_EXECINFO_H)
check_include_file_cxx (glob.h HAVE_GLOB_H)
check_include_file_cxx (inttypes.h HAVE_INTTYPES_H)
check_include_file_cxx (memory.h HAVE_MEMORY_H)
check_include_file_cxx (pwd.h HAVE_PWD_H)
check_include_file_cxx (stdint.h HAVE_STDINT_H)
check_include_file_cxx (strings.h HAVE_STRINGS_H)
check_include_file_cxx (sys/stat.h HAVE_SYS_STAT_H)
check_include_file_cxx (sys/syscall.h HAVE_SYS_SYSCALL_H)
check_include_file_cxx (sys/time.h HAVE_SYS_TIME_H)
check_include_file_cxx (sys/types.h HAVE_SYS_TYPES_H)
check_include_file_cxx (sys/utsname.h HAVE_SYS_UTSNAME_H)
check_include_file_cxx (sys/wait.h HAVE_SYS_WAIT_H)
check_include_file_cxx (syscall.h HAVE_SYSCALL_H)
check_include_file_cxx (syslog.h HAVE_SYSLOG_H)
check_include_file_cxx (ucontext.h HAVE_UCONTEXT_H)
check_include_file_cxx (unistd.h HAVE_UNISTD_H)

check_include_file_cxx ("ext/hash_map" HAVE_EXT_HASH_MAP)
check_include_file_cxx ("ext/hash_set" HAVE_EXT_HASH_SET)
check_include_file_cxx ("ext/slist" HAVE_EXT_SLIST)
check_include_file_cxx ("tr1/unordered_map" HAVE_TR1_UNORDERED_MAP)
check_include_file_cxx ("tr1/unordered_set" HAVE_TR1_UNORDERED_SET)
check_include_file_cxx ("unordered_map" HAVE_UNORDERED_MAP)
check_include_file_cxx ("unordered_set" HAVE_UNORDERED_SET)

check_type_size ("unsigned __int16" HAVE___UINT16 LANGUAGE CXX)
check_type_size (u_int16_t HAVE_U_INT16_T LANGUAGE CXX)
check_type_size (uint16_t HAVE_UINT16_T LANGUAGE CXX)

check_function_exists (dladdr HAVE_DLADDR)
check_function_exists (fcntl HAVE_FCNTL)
check_function_exists (pread HAVE_PREAD)
check_function_exists (pwrite HAVE_PWRITE)
check_function_exists (sigaction HAVE_SIGACTION)
check_function_exists (sigaltstack HAVE_SIGALSTACK)

# NOTE gcc does not fail if you pass a non-existent -Wno-* option as an
# argument. However, it will happily fail if you pass the corresponding -W*
# option. So, we check whether options that disable warnings exist by testing
# the availability of the corresponding option that enables the warning. This
# eliminates the need to check for compiler for several (mainly Clang) options.

check_cxx_compiler_flag (-Wdeprecated HAVE_NO_DEPRECATED)
check_cxx_compiler_flag (-Wunnamed-type-template-args
    HAVE_NO_UNNAMED_TYPE_TEMPLATE_ARGS)

# NOTE: Cannot use check_function_exists here since >=vc-14.0 can define
# snprintf as an inline function
check_cxx_symbol_exists (snprintf cstdio HAVE_SNPRINTF)

check_library_exists (dbghelp UnDecorateSymbolName "" HAVE_DBGHELP)

check_cxx_source_compiles ("
#include <cstdlib>
static void foo(void) __attribute__ ((unused));
int main(void) { return 0; }
" HAVE___ATTRIBUTE__)

check_cxx_source_compiles ("
#include <cstdlib>
static void foo(void) __attribute__ ((visibility(\"default\")));
int main(void) { return 0; }
" HAVE___ATTRIBUTE__VISIBILITY_DEFAULT)

check_cxx_source_compiles ("
#include <cstdlib>
static void foo(void) __attribute__ ((visibility(\"hidden\")));
int main(void) { return 0; }
" HAVE___ATTRIBUTE__VISIBILITY_HIDDEN)

check_cxx_source_compiles ("
int main(void) { if (__builtin_expect(0, 0)) return 1; return 0; }
" HAVE___BUILTIN_EXPECT)

check_cxx_source_compiles ("
int main(void)
{
  int a; if (__sync_val_compare_and_swap(&a, 0, 1)) return 1; return 0;
}
" HAVE___SYNC_VAL_COMPARE_AND_SWAP)

check_cxx_source_compiles ("
#define _XOPEN_SOURCE 500
#include <pthread.h>
int main(void)
{
  pthread_rwlock_t l;
  pthread_rwlock_init(&l, NULL);
  pthread_rwlock_rdlock(&l);
  return 0;
}
" HAVE_RWLOCK)

check_cxx_source_compiles ("
__declspec(selectany) int a;
int main(void) { return 0; }
" HAVE___DECLSPEC)

check_cxx_source_compiles ("
#include <vector>
vector<int> t; int main() { }
" STL_NO_NAMESPACE)

check_cxx_source_compiles ("
#include <vector>
std::vector<int> t; int main() { }
" STL_STD_NAMESPACE)

check_cxx_source_compiles ("
#include <iostream>
std::ostream& operator<<(std::ostream&, struct s);
using ::operator<<;
int main() { }
" HAVE_USING_OPERATOR)

check_cxx_source_compiles ("
namespace Outer { namespace Inner { int i = 0; }}
using namespace Outer::Inner;;
int main() { return i; }
" HAVE_NAMESPACES)

check_cxx_source_compiles ("
__thread int tls;
int main() { }
" HAVE_GCC_TLS)

check_cxx_source_compiles ("
__declspec(thread) int tls;
int main() { }
" HAVE_MSVC_TLS)

check_cxx_source_compiles ("
thread_local int tls;
int main() { }
" HAVE_CXX11_TLS)

check_cxx_source_compiles ("
#include <type_traits>
std::aligned_storage<sizeof(char), alignof(char)>::type data;
int main() { }
" HAVE_ALIGNED_STORAGE)

check_cxx_source_compiles ("
#include <atomic>
std::atomic<int> i;
int main() { }
" HAVE_CXX11_ATOMIC)

check_cxx_source_compiles ("
#include <cstddef>
void foo(std::nullptr_t) {}
int main(void) { foo(nullptr); }
" HAVE_CXX11_NULLPTR_T)

if (WITH_TLS)
  # Cygwin does not support the thread attribute. Don't bother.
  if (HAVE_GCC_TLS)
    set (GLOG_THREAD_LOCAL_STORAGE "__thread")
  elseif (HAVE_MSVC_TLS)
    set (GLOG_THREAD_LOCAL_STORAGE "__declspec(thread)")
  elseif (HAVE_CXX11_TLS)
    set (GLOG_THREAD_LOCAL_STORAGE thread_local)
  endif (HAVE_GCC_TLS)
endif (WITH_TLS)

set (_PC_FIELDS
  "gregs[REG_PC]"
  "gregs[REG_EIP]"
  "gregs[REG_RIP]"
  "sc_ip"
  "uc_regs->gregs[PT_NIP]"
  "gregs[R15]"
  "arm_pc"
  "mc_eip"
  "mc_rip"
  "__gregs[REG_EIP]"
  "__gregs[REG_RIP]"
  "ss.eip"
  "__ss.__eip"
  "ss.rip"
  "__ss.__rip"
  "ss.srr0"
  "__ss.__srr0"
)

set (_PC_HEADERS ucontext.h signal.h)

if (HAVE_UCONTEXT_H AND NOT PC_FROM_UCONTEXT)
  foreach (_PC_FIELD ${_PC_FIELDS})
    foreach (_PC_HEADER ${_PC_HEADERS})
      set (_TMP
      ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeTmp/uctfield.cpp)
      file (WRITE ${_TMP} "
#define _GNU_SOURCE 1
#include <${_PC_HEADER}>
int main(void)
{
  ucontext_t u;
  return u.${_PC_FIELD} == 0;
}
")
      try_compile (HAVE_PC_FROM_UCONTEXT ${CMAKE_CURRENT_BINARY_DIR} ${_TMP}
        COMPILE_DEFINITIONS _GNU_SOURCE=1)

      if (HAVE_PC_FROM_UCONTEXT)
        set (PC_FROM_UCONTEXT ${_PC_FIELD} CACHE)
      endif (HAVE_PC_FROM_UCONTEXT)
    endforeach (_PC_HEADER)
  endforeach (_PC_FIELD)
endif  (HAVE_UCONTEXT_H AND NOT PC_FROM_UCONTEXT)

if (STL_STD_NAMESPACE)
  set (STL_NAMESPACE std)
else (STL_STD_NAMESPACE)
  set (STL_NAMESPACE "")
endif (STL_STD_NAMESPACE)

set (GOOGLE_NAMESPACE google)
set (_START_GOOGLE_NAMESPACE_ "namespace ${GOOGLE_NAMESPACE} {")
set (_END_GOOGLE_NAMESPACE_ "}")
set (ac_cv_have_glog_export 1)

if (HAVE___UINT16)
  set (ac_cv_have___uint16 1)
else (HAVE___UINT16)
  set (ac_cv_have___uint16 0)
endif (HAVE___UINT16)

if (HAVE_INTTYPES_H)
  set (ac_cv_have_inttypes_h 1)
else (HAVE_INTTYPES_H)
  set (ac_cv_have_inttypes_h 0)
endif (HAVE_INTTYPES_H)

if (HAVE_LIB_GFLAGS)
  set (ac_cv_have_libgflags 1)
else (HAVE_LIB_GFLAGS)
  set (ac_cv_have_libgflags 0)
endif (HAVE_LIB_GFLAGS)

if (HAVE_STDINT_H)
  set (ac_cv_have_stdint_h 1)
else (HAVE_STDINT_H)
  set (ac_cv_have_stdint_h 0)
endif (HAVE_STDINT_H)

if (HAVE_SYS_TYPES_H)
  set (ac_cv_have_systypes_h 1)
else (HAVE_SYS_TYPES_H)
  set (ac_cv_have_systypes_h 0)
endif (HAVE_SYS_TYPES_H)

if (HAVE_U_INT16_T)
  set (ac_cv_have_u_int16_t 1)
else (HAVE_U_INT16_T)
  set (ac_cv_have_u_int16_t 0)
endif (HAVE_U_INT16_T)

if (HAVE_UINT16_T)
  set (ac_cv_have_uint16_t 1)
else (HAVE_UINT16_T)
  set (ac_cv_have_uint16_t 0)
endif (HAVE_UINT16_T)

if (HAVE_UNISTD_H)
  set (ac_cv_have_unistd_h 1)
else (HAVE_UNISTD_H)
  set (ac_cv_have_unistd_h 0)
endif (HAVE_UNISTD_H)

set (ac_google_namespace ${GOOGLE_NAMESPACE})
set (ac_google_end_namespace ${_END_GOOGLE_NAMESPACE_})
set (ac_google_start_namespace ${_START_GOOGLE_NAMESPACE_})

if (HAVE___ATTRIBUTE__)
  set (ac_cv___attribute___noreturn "__attribute__((noreturn))")
  set (ac_cv___attribute___noinline "__attribute__((noinline))")
  set (ac_cv___attribute___printf_4_5 "__attribute__((__format__(__printf__, 4, 5)))")
elseif (HAVE___DECLSPEC)
  set (ac_cv___attribute___noreturn "__declspec(noreturn)")
  #set (ac_cv___attribute___noinline "__declspec(noinline)")
endif (HAVE___ATTRIBUTE__)

if (HAVE___BUILTIN_EXPECT)
  set (ac_cv_have___builtin_expect 1)
else (HAVE___BUILTIN_EXPECT)
  set (ac_cv_have___builtin_expect 0)
endif (HAVE___BUILTIN_EXPECT)

if (HAVE_USING_OPERATOR)
  set (ac_cv_cxx_using_operator 1)
else (HAVE_USING_OPERATOR)
  set (ac_cv_cxx_using_operator 0)
endif (HAVE_USING_OPERATOR)

if (HAVE_CXX11_NULLPTR_T)
  set (ac_cv_cxx11_nullptr_t 1)
else (HAVE_CXX11_NULLPTR_T)
  set (ac_cv_cxx11_nullptr_t 0)
endif (HAVE_CXX11_NULLPTR_T)

if (HAVE_EXECINFO_H)
  set (HAVE_STACKTRACE 1)
endif (HAVE_EXECINFO_H)

if (HAVE_CXX11_ATOMIC)
  set (ac_cv_cxx11_atomic 1)
else (HAVE_CXX11_ATOMIC)
  set (ac_cv_cxx11_atomic 0)
endif (HAVE_CXX11_ATOMIC)

if (WITH_SYMBOLIZE)
  if (WIN32 OR CYGWIN)
    cmake_push_check_state (RESET)
    set (CMAKE_REQUIRED_LIBRARIES DbgHelp)

    check_cxx_source_runs ([=[
    #include <windows.h>
    #include <dbghelp.h>
    #include <cstdlib>

    void foobar() { }

    int main()
    {
        HANDLE process = GetCurrentProcess();

        if (!SymInitialize(process, NULL, TRUE))
            return EXIT_FAILURE;

        char buf[sizeof(SYMBOL_INFO) + MAX_SYM_NAME];
        SYMBOL_INFO *symbol = reinterpret_cast<SYMBOL_INFO *>(buf);
        symbol->SizeOfStruct = sizeof(SYMBOL_INFO);
        symbol->MaxNameLen = MAX_SYM_NAME;

        void* const pc = reinterpret_cast<void*>(&foobar);
        BOOL ret = SymFromAddr(process, reinterpret_cast<DWORD64>(pc), 0, symbol);

        return ret ? EXIT_SUCCESS : EXIT_FAILURE;
    }
    ]=] HAVE_SYMBOLIZE)

    cmake_pop_check_state ()

    if (HAVE_SYMBOLIZE)
      set (HAVE_STACKTRACE 1)
    endif (HAVE_SYMBOLIZE)
  elseif (UNIX OR (APPLE AND HAVE_DLADDR))
    set (HAVE_SYMBOLIZE 1)
  endif (WIN32 OR CYGWIN)
endif (WITH_SYMBOLIZE)

# CMake manages symbolize availability. The definition is necessary only when
# building the library. Switch to add_compile_definitions once we drop support
# for CMake below version 3.12.
add_definitions (-DGLOG_NO_SYMBOLIZE_DETECTION)

check_cxx_source_compiles ("
#include <cstdlib>
#include <ctime>
int main()
{
    time_t timep;
    struct tm result;
    localtime_r(&timep, &result);
    return EXIT_SUCCESS;
}
" HAVE_LOCALTIME_R)

set (SIZEOF_VOID_P ${CMAKE_SIZEOF_VOID_P})

if (WITH_THREADS AND Threads_FOUND)
  if (CMAKE_USE_PTHREADS_INIT)
    set (HAVE_PTHREAD 1)
  endif (CMAKE_USE_PTHREADS_INIT)
else (WITH_THREADS AND Threads_FOUND)
  set (NO_THREADS 1)
endif (WITH_THREADS AND Threads_FOUND)

# fopen/open on Cygwin can not handle unix-type paths like /home/....
# therefore we translate TEST_SRC_DIR to windows-path.
if (CYGWIN)
  execute_process (COMMAND cygpath.exe -m ${CMAKE_CURRENT_SOURCE_DIR}
                   OUTPUT_STRIP_TRAILING_WHITESPACE
                   OUTPUT_VARIABLE TEST_SRC_DIR)
  set (TEST_SRC_DIR \"${TEST_SRC_DIR}\")
else (CYGWIN)
  set (TEST_SRC_DIR \"${CMAKE_CURRENT_SOURCE_DIR}\")
endif (CYGWIN)

configure_file (src/config.h.cmake.in config.h)
configure_file (src/glog/logging.h.in glog/logging.h @ONLY)
configure_file (src/glog/raw_logging.h.in glog/raw_logging.h @ONLY)
configure_file (src/glog/stl_logging.h.in glog/stl_logging.h @ONLY)
configure_file (src/glog/vlog_is_on.h.in glog/vlog_is_on.h @ONLY)

if (WITH_PKGCONFIG)
  set (VERSION ${PROJECT_VERSION})
  set (prefix ${CMAKE_INSTALL_PREFIX})
  set (exec_prefix ${CMAKE_INSTALL_FULL_BINDIR})
  set (libdir ${CMAKE_INSTALL_FULL_LIBDIR})
  set (includedir ${CMAKE_INSTALL_FULL_INCLUDEDIR})

  configure_file (
    "${PROJECT_SOURCE_DIR}/libglog.pc.in"
    "${PROJECT_BINARY_DIR}/libglog.pc"
    @ONLY
  )

  unset (VERSION)
  unset (prefix)
  unset (exec_prefix)
  unset (libdir)
  unset (includedir)
endif (WITH_PKGCONFIG)

set (GLOG_PUBLIC_H
  ${CMAKE_CURRENT_BINARY_DIR}/glog/export.h
  ${CMAKE_CURRENT_BINARY_DIR}/glog/logging.h
  ${CMAKE_CURRENT_BINARY_DIR}/glog/raw_logging.h
  ${CMAKE_CURRENT_BINARY_DIR}/glog/stl_logging.h
  ${CMAKE_CURRENT_BINARY_DIR}/glog/vlog_is_on.h
  src/glog/log_severity.h
)

set (GLOG_SRCS
  ${GLOG_PUBLIC_H}
  src/base/commandlineflags.h
  src/base/googleinit.h
  src/base/mutex.h
  src/demangle.cc
  src/demangle.h
  src/logging.cc
  src/raw_logging.cc
  src/symbolize.cc
  src/symbolize.h
  src/utilities.cc
  src/utilities.h
  src/vlog_is_on.cc
)

if (HAVE_PTHREAD OR WIN32 OR CYGWIN)
  list (APPEND GLOG_SRCS src/signalhandler.cc)
endif (HAVE_PTHREAD OR WIN32 OR CYGWIN)

if (CYGWIN OR WIN32)
  list (APPEND GLOG_SRCS
    src/windows/port.cc
    src/windows/port.h
  )
endif (CYGWIN OR WIN32)

add_compile_options ($<$<AND:$<BOOL:${HAVE_NO_UNNAMED_TYPE_TEMPLATE_ARGS}>,$<NOT:$<CXX_COMPILER_ID:GNU>>>:-Wno-unnamed-type-template-args>)

set (_glog_CMake_BINDIR ${CMAKE_INSTALL_BINDIR})
set (_glog_CMake_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR})
set (_glog_CMake_LIBDIR ${CMAKE_INSTALL_LIBDIR})
set (_glog_CMake_INSTALLDIR ${_glog_CMake_LIBDIR}/cmake/glog)

set (_glog_CMake_DIR glog/cmake)
set (_glog_CMake_DATADIR ${CMAKE_INSTALL_DATAROOTDIR}/${_glog_CMake_DIR})
set (_glog_BINARY_CMake_DATADIR
  ${CMAKE_CURRENT_BINARY_DIR}/${_glog_CMake_DATADIR})

# Add additional CMake find modules here.
set (_glog_CMake_MODULES)

if (Unwind_FOUND)
  # Copy the module only if libunwind is actually used.
  list (APPEND _glog_CMake_MODULES ${CMAKE_CURRENT_SOURCE_DIR}/cmake/FindUnwind.cmake)
endif (Unwind_FOUND)

# Generate file name for each module in the binary directory
foreach (_file ${_glog_CMake_MODULES})
  get_filename_component (_module "${_file}" NAME)

  list (APPEND _glog_BINARY_CMake_MODULES
    ${_glog_BINARY_CMake_DATADIR}/${_module})
endforeach (_file)

if (_glog_CMake_MODULES)
  # Copy modules to binary directory during the build
  add_custom_command (OUTPUT ${_glog_BINARY_CMake_MODULES}
    COMMAND ${CMAKE_COMMAND} -E make_directory
    ${_glog_BINARY_CMake_DATADIR}
    COMMAND ${CMAKE_COMMAND} -E copy ${_glog_CMake_MODULES}
    ${_glog_BINARY_CMake_DATADIR}
    DEPENDS ${_glog_CMake_MODULES}
    COMMENT "Copying find modules..."
  )
endif (_glog_CMake_MODULES)

add_library (glog
  ${GLOG_SRCS}
  ${_glog_BINARY_CMake_MODULES}
)

add_library (glog::glog ALIAS glog)

if (Unwind_FOUND)
  target_link_libraries (glog PUBLIC unwind::unwind)
  set (Unwind_DEPENDENCY "find_dependency (Unwind ${Unwind_VERSION})")
endif (Unwind_FOUND)

if (HAVE_DBGHELP)
   target_link_libraries (glog PUBLIC dbghelp)
endif (HAVE_DBGHELP)

if (HAVE_PTHREAD)
  target_link_libraries (glog PUBLIC ${CMAKE_THREAD_LIBS_INIT})
endif (HAVE_PTHREAD)

if (gflags_FOUND)
  target_link_libraries (glog PUBLIC gflags)

  target_compile_definitions (glog PRIVATE
    GFLAGS_DLL_DECLARE_FLAG=GOOGLE_GLOG_DLL_DECL
    GFLAGS_DLL_DEFINE_FLAG=GOOGLE_GLOG_DLL_DECL
  )
endif (gflags_FOUND)

if (ANDROID)
  target_link_libraries (glog PUBLIC log)
endif()

set_target_properties (glog PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties (glog PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR})

if (CYGWIN OR WIN32)
  target_compile_definitions (glog PUBLIC GLOG_NO_ABBREVIATED_SEVERITIES)
endif (CYGWIN OR WIN32)

if (WITH_CUSTOM_PREFIX)
  target_compile_definitions(glog PUBLIC GLOG_CUSTOM_PREFIX_SUPPORT)
endif (WITH_CUSTOM_PREFIX)

set_target_properties (glog PROPERTIES PUBLIC_HEADER "${GLOG_PUBLIC_H}")

target_include_directories (glog BEFORE PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
  "$<INSTALL_INTERFACE:${_glog_CMake_INCLUDE_DIR}>"
  PRIVATE ${CMAKE_CURRENT_BINARY_DIR}
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)

if (CYGWIN OR WIN32)
  target_include_directories (glog PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/windows>"
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/windows)
endif (CYGWIN OR WIN32)

set_target_properties (glog PROPERTIES DEFINE_SYMBOL GOOGLE_GLOG_IS_A_DLL)

generate_export_header (glog
  EXPORT_MACRO_NAME GOOGLE_GLOG_DLL_DECL
  EXPORT_FILE_NAME ${CMAKE_CURRENT_BINARY_DIR}/glog/export.h)

# Unit testing

if (BUILD_TESTING)
  set (_GLOG_TEST_LIBS glog::glog)

  if (HAVE_LIB_GTEST)
    list (APPEND _GLOG_TEST_LIBS GTest::GTest)
  endif (HAVE_LIB_GTEST)

  add_executable (logging_unittest
    src/logging_unittest.cc
  )

  target_link_libraries (logging_unittest PRIVATE ${_GLOG_TEST_LIBS})

  if (WITH_CUSTOM_PREFIX)
    add_executable (logging_custom_prefix_unittest
      src/logging_custom_prefix_unittest.cc
    )

    target_link_libraries (logging_custom_prefix_unittest PRIVATE ${_GLOG_TEST_LIBS})

    add_test (NAME logging_custom_prefix
              COMMAND logging_custom_prefix_unittest)
  endif (WITH_CUSTOM_PREFIX)

  add_executable (stl_logging_unittest
    src/stl_logging_unittest.cc
  )

  target_link_libraries (stl_logging_unittest PRIVATE ${_GLOG_TEST_LIBS})

  if (HAVE_NO_DEPRECATED)
    set_property (TARGET stl_logging_unittest APPEND PROPERTY COMPILE_OPTIONS
      -Wno-deprecated)
  endif (HAVE_NO_DEPRECATED)

  if (HAVE_UNORDERED_MAP AND HAVE_UNORDERED_SET)
    target_compile_definitions (stl_logging_unittest PRIVATE
      GLOG_STL_LOGGING_FOR_UNORDERED)
  endif (HAVE_UNORDERED_MAP AND HAVE_UNORDERED_SET)

  if (HAVE_TR1_UNORDERED_MAP AND HAVE_TR1_UNORDERED_SET)
    target_compile_definitions (stl_logging_unittest PRIVATE
      GLOG_STL_LOGGING_FOR_TR1_UNORDERED)
  endif (HAVE_TR1_UNORDERED_MAP AND HAVE_TR1_UNORDERED_SET)

  if (HAVE_EXT_HASH_MAP AND HAVE_EXT_HASH_SET)
    target_compile_definitions (stl_logging_unittest PRIVATE
      GLOG_STL_LOGGING_FOR_EXT_HASH)
  endif (HAVE_EXT_HASH_MAP AND HAVE_EXT_HASH_SET)

  if (HAVE_EXT_SLIST)
    target_compile_definitions (stl_logging_unittest PRIVATE
      GLOG_STL_LOGGING_FOR_EXT_SLIST)
  endif (HAVE_EXT_SLIST)

  if (HAVE_SYMBOLIZE)
    add_executable (symbolize_unittest
      src/symbolize_unittest.cc
    )

    target_link_libraries (symbolize_unittest PRIVATE ${_GLOG_TEST_LIBS})
  endif (HAVE_SYMBOLIZE)

  add_executable (demangle_unittest
    src/demangle_unittest.cc
  )

  target_link_libraries (demangle_unittest PRIVATE ${_GLOG_TEST_LIBS})

  if (HAVE_STACKTRACE)
    add_executable (stacktrace_unittest
      src/stacktrace_unittest.cc
    )

    target_link_libraries (stacktrace_unittest PRIVATE ${_GLOG_TEST_LIBS})
  endif (HAVE_STACKTRACE)

  add_executable (utilities_unittest
    src/utilities_unittest.cc
  )

  target_link_libraries (utilities_unittest PRIVATE ${_GLOG_TEST_LIBS})

  if (HAVE_STACKTRACE AND HAVE_SYMBOLIZE)
    add_executable (signalhandler_unittest
      src/signalhandler_unittest.cc
    )

    target_link_libraries (signalhandler_unittest PRIVATE ${_GLOG_TEST_LIBS})
  endif (HAVE_STACKTRACE AND HAVE_SYMBOLIZE)

  add_test (NAME demangle COMMAND demangle_unittest)
  add_test (NAME logging COMMAND logging_unittest)

  if (TARGET signalhandler_unittest)
    add_test (NAME signalhandler COMMAND signalhandler_unittest)
  endif (TARGET signalhandler_unittest)

  if (TARGET stacktrace_unittest)
    add_test (NAME stacktrace COMMAND stacktrace_unittest)
    set_tests_properties(stacktrace PROPERTIES TIMEOUT 30)
  endif (TARGET stacktrace_unittest)

  add_test (NAME stl_logging COMMAND stl_logging_unittest)

  if (TARGET symbolize_unittest)
    add_test (NAME symbolize COMMAND symbolize_unittest)
  endif (TARGET symbolize_unittest)

  # Generate an initial cache

  get_cache_variables (_CACHEVARS EXCLUDE CMAKE_MAKE_PROGRAM)

  set (_INITIAL_CACHE
    ${CMAKE_CURRENT_BINARY_DIR}/test_package_config/glog_package_config_initial_cache.cmake)

  # Package config test

  add_test (NAME cmake_package_config_init COMMAND ${CMAKE_COMMAND}
    -DTEST_BINARY_DIR=${CMAKE_CURRENT_BINARY_DIR}/test_package_config
    -DINITIAL_CACHE=${_INITIAL_CACHE}
    -DCACHEVARS=${_CACHEVARS}
    -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/TestInitPackageConfig.cmake
  )

  add_test (NAME cmake_package_config_generate COMMAND ${CMAKE_COMMAND}
    -DGENERATOR=${CMAKE_GENERATOR}
    -DGENERATOR_PLATFORM=${CMAKE_GENERATOR_PLATFORM}
    -DGENERATOR_TOOLSET=${CMAKE_GENERATOR_TOOLSET}
    -DINITIAL_CACHE=${_INITIAL_CACHE}
    -DPACKAGE_DIR=${CMAKE_CURRENT_BINARY_DIR}
    -DPATH=$ENV{PATH}
    -DSOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}/src/package_config_unittest/working_config
    -DTEST_BINARY_DIR=${CMAKE_CURRENT_BINARY_DIR}/test_package_config/working_config
    -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/TestPackageConfig.cmake
  )

  add_test (NAME cmake_package_config_build COMMAND
    ${CMAKE_COMMAND} --build ${CMAKE_CURRENT_BINARY_DIR}/test_package_config/working_config
  )

  add_test (NAME cmake_package_config_cleanup COMMAND ${CMAKE_COMMAND} -E
    remove_directory
    ${CMAKE_CURRENT_BINARY_DIR}/test_package_config
  )

  # Fixtures setup
  set_tests_properties (cmake_package_config_init PROPERTIES FIXTURES_SETUP
    cmake_package_config)
  set_tests_properties (cmake_package_config_generate PROPERTIES FIXTURES_SETUP
    cmake_package_config_working)

  # Fixtures cleanup
  set_tests_properties (cmake_package_config_cleanup PROPERTIES FIXTURES_CLEANUP
    cmake_package_config)

  # Fixture requirements
  set_tests_properties (cmake_package_config_generate PROPERTIES
    FIXTURES_REQUIRED cmake_package_config)
  set_tests_properties (cmake_package_config_build PROPERTIES
    FIXTURES_REQUIRED "cmake_package_config;cmake_package_config_working")
endif (BUILD_TESTING)

install (TARGETS glog
  EXPORT glog-targets
  RUNTIME DESTINATION ${_glog_CMake_BINDIR}
  PUBLIC_HEADER DESTINATION ${_glog_CMake_INCLUDE_DIR}/glog
  LIBRARY DESTINATION ${_glog_CMake_LIBDIR}
  ARCHIVE DESTINATION ${_glog_CMake_LIBDIR})

if (WITH_PKGCONFIG)
  install (
    FILES "${PROJECT_BINARY_DIR}/libglog.pc"
    DESTINATION "${_glog_CMake_LIBDIR}/pkgconfig"
  )
endif (WITH_PKGCONFIG)

set (glog_CMake_VERSION 3.0)

if (gflags_FOUND)
  # Ensure clients locate only the package config and not third party find
  # modules having the same name. This avoid cmake_policy PUSH/POP errors.
  if (CMAKE_VERSION VERSION_LESS 3.9)
    set (gflags_DEPENDENCY "find_dependency (gflags ${gflags_VERSION})")
  else (CMAKE_VERSION VERSION_LESS 3.9)
    # Passing additional find_package arguments to find_dependency is possible
    # starting with CMake 3.9.
    set (glog_CMake_VERSION 3.9)
    set (gflags_DEPENDENCY "find_dependency (gflags ${gflags_VERSION} NO_MODULE)")
  endif (CMAKE_VERSION VERSION_LESS 3.9)
endif (gflags_FOUND)

configure_package_config_file (glog-config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/glog-config.cmake
  INSTALL_DESTINATION ${_glog_CMake_INSTALLDIR}
  NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file (
  ${CMAKE_CURRENT_BINARY_DIR}/glog-config-version.cmake
  COMPATIBILITY SameMajorVersion)

export (TARGETS glog NAMESPACE glog:: FILE glog-targets.cmake)
export (PACKAGE glog)

get_filename_component (_PREFIX "${CMAKE_INSTALL_PREFIX}" ABSOLUTE)

# Directory containing the find modules relative to the config install
# directory.
file (RELATIVE_PATH glog_REL_CMake_MODULES
  ${_PREFIX}/${_glog_CMake_INSTALLDIR}
  ${_PREFIX}/${_glog_CMake_DATADIR}/glog-modules.cmake)

get_filename_component (glog_REL_CMake_DATADIR ${glog_REL_CMake_MODULES}
  DIRECTORY)

set (glog_FULL_CMake_DATADIR
  ${CMAKE_CURRENT_BINARY_DIR}/${_glog_CMake_DATADIR})

configure_file (glog-modules.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/glog-modules.cmake @ONLY)

install (CODE
"
set (glog_FULL_CMake_DATADIR \"\\\${CMAKE_CURRENT_LIST_DIR}/${glog_REL_CMake_DATADIR}\")
configure_file (\"${CMAKE_CURRENT_SOURCE_DIR}/glog-modules.cmake.in\"
  \"${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeTmp/glog-modules.cmake\" @ONLY)
file (INSTALL
  \"${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeTmp/glog-modules.cmake\"
  DESTINATION
  \"\${CMAKE_INSTALL_PREFIX}/${_glog_CMake_INSTALLDIR}\")
"
  COMPONENT Development
)

install (FILES
  ${CMAKE_CURRENT_BINARY_DIR}/glog-config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/glog-config-version.cmake
  DESTINATION ${_glog_CMake_INSTALLDIR})

# Find modules in share/glog/cmake
install (DIRECTORY ${_glog_BINARY_CMake_DATADIR}
  DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/glog
  COMPONENT Development
  FILES_MATCHING PATTERN "*.cmake"
)

install (EXPORT glog-targets NAMESPACE glog:: DESTINATION
  ${_glog_CMake_INSTALLDIR})
