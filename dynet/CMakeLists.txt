# ########## dynet library ##########
# Sources:
set(dynet_library_SRCS
    cfsm-builder.cc
    dynet.cc
    deep-lstm.cc
    devices.cc
    dict.cc
    dim.cc
    exec.cc
    expr.cc
    fast-lstm.cc
    globals.cc
    grad-check.cc
    graph.cc
    gru.cc
    hsm-builder.cc
    init.cc
    lstm.cc
    mem.cc
    model.cc
    mp.cc
    nodes.cc
    nodes-common.cc
    nodes-contract.cc
    nodes-conv.cc
    param-nodes.cc
    pretrain.cc
    rnn.cc
    rnn-state-machine.cc
    saxe-init.cc
    shadow-params.cc
    tensor.cc
    training.cc
    weight-decay.cc
)

# Headers:
set(dynet_library_HDRS
    aligned-mem-pool.h
    cfsm-builder.h
    c2w.h
    dynet.h
    cuda.h
    devices.h
    dict.h
    dim.h
    exec.h
    expr.h
    fast-lstm.h
    functors.h
    globals.h
    gpu-kernels.h
    gpu-ops.h
    graph.h
    gru.h
    hsm-builder.h
    init.h
    lstm.h
    mem.h
    model.h
    mp.h
    nodes.h
    nodes-contract.h
    nodes-conv.h
    param-nodes.h
    rnn-state-machine.h
    rnn.h
    saxe-init.h
    shadow-params.h
    simd-functors.h
    tensor.h
    timing.h
    training.h
    weight-decay.h
)

file(GLOB TEST_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} tests/*.cc)

#foreach(test_src ${TEST_SRCS})
  #Extract the filename without an extension (NAME_WE)
#  get_filename_component(testName ${test_src} NAME_WE)

  #Add compile target
#  add_executable(${testName} ${test_src})

  #link to Boost libraries AND your targets and dependencies
#  target_link_libraries(${testName} dynet ${LIBS})

#  set_target_properties(${testName} PROPERTIES
#      RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/tests.bin)

  #Finally add it to test execution -
  #Notice the WORKING_DIRECTORY and COMMAND
#  add_test(NAME ${testName}
#     WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/tests.bin
#     COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/tests.bin/${testName} )
#endforeach(test_src)

# Build cpu library
add_library(dynet SHARED ${dynet_library_SRCS} ${dynet_library_HDRS})
target_link_libraries(dynet ${LIBS})
if(WITH_CUDA_BACKEND)
  # Build main gpu library
  list(APPEND dynet_library_SRCS cuda.cc)
	add_library(gdynet SHARED ${dynet_library_SRCS} ${dynet_library_HDRS})
	target_link_libraries(gdynet ${LIBS})
  set_target_properties(gdynet PROPERTIES
                        COMPILE_DEFINITIONS HAVE_CUDA)
  # Build nvcc-compiled gpu library
  set(CUDA_SEPARABLE_COMPILATION ON)
  list(APPEND CUDA_NVCC_FLAGS "-gencode;arch=compute_30,code=sm_30;-gencode;arch=compute_35,code=sm_35;-gencode;arch=compute_37,code=sm_37;-gencode;arch=compute_50,code=sm_50;-gencode;arch=compute_52,code=sm_52;-gencode;arch=compute_52,code=compute_52;-std=c++11;-O2;-DVERBOSE;-DEIGEN_USE_GPU;-DHAVE_CUDA")
  SET(CUDA_PROPAGATE_HOST_FLAGS OFF)
  cuda_add_library(dynetcuda SHARED gpu-ops.cu gpu-nodes.cu gpu-nodes-contract.cu gpu-nodes-conv.cu gpu-param-nodes.cu gpu-training.cu gpu-model.cu OPTIONS --compiler-options "-fPIC")
endif(WITH_CUDA_BACKEND)

install(FILES ${dynet_library_HDRS} DESTINATION include/dynet)
install(TARGETS dynet DESTINATION lib)
if(WITH_CUDA_BACKEND)
  install(TARGETS gdynet DESTINATION lib)
endif(WITH_CUDA_BACKEND)

# target_compile_features(dynet PRIVATE cxx_range_for)

