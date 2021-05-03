# Technically most of this isn't needed
project(luajit_host C ASM)
include(TestBigEndian)

set(CMAKE_C_FLAGS_DEBUG "-fomit-frame-pointer ${LJ_MARCH}")
set(CMAKE_C_FLAGS_RELEASE "-O2 -fomit-frame-pointer ${LJ_MARCH}")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "-O2 -fomit-frame-pointer ${LJ_MARCH}")
set(CMAKE_C_FLAGS_PROFILE "-O2 -pg -fomit-frame-pointer ${LJ_MARCH}")

set(DASM_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../dynasm")
set(DASM "${DASM_DIR}/dynasm.lua")
set(DASM_ARCH "x64")
set(DASM_DASC "vm_${DASM_ARCH}.dasc")
set(DASM_FLAGS "-D JIT -D P64 -D HFABI -D FPU -D FFI")
set(IS_BE 0)
TEST_BIG_ENDIAN(IS_BE)

if (IS_BE matches 1)
  set(DASM_FLAGS "${DASM_FLAGS} -D ENDIAN_BE")
else ()
  set(DASM_FLAGS "${DASM_FLAGS} -D ENDIAN_LE")
endif ()

set(luajit_minilua_sources
  "${CMAKE_CURRENT_SOURCE_DIR}/minilua.c"
)

set(luajit_host_sources
  "${CMAKE_CURRENT_SOURCE_DIR}/buildvm.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/buildvm_asm.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/buildvm_fold.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/buildvm_lib.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/buildvm_peobj.c"
)

# build minilua
add_executable(minilua luajit_minilua_sources)

add_custom_target(buildvm_arch
  COMMAND minilua ${DASM} ${DASM_FLAGS} -o host/buildvm_arch.h ${DASM_DASC}
  DEPENDS minilua
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..
)

# build buildvm
add_executable(buildvm luajit_host_sources)
add_dependencies(buildvm buildvm_arch)
target_include_directories(buildvm PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../../dynasm)