add_executable(greeter_server greeter_server.cc ${PROTO_GENERATED_FILES_DIR}/helloworld.pb.cc ${PROTO_GENERATED_FILES_DIR}/helloworld.grpc.pb.cc)
target_link_libraries(greeter_server
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF})

add_executable(greeter_client greeter_client.cc ${PROTO_GENERATED_FILES_DIR}/helloworld.pb.cc ${PROTO_GENERATED_FILES_DIR}/helloworld.grpc.pb.cc)
target_link_libraries(greeter_client
    ${_GRPC_GRPCPP_UNSECURE}
    ${_PROTOBUF_LIBPROTOBUF})
