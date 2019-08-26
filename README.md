# GRPC C++ Hello World Demo (ubuntu 18.04)

This demo is aimed to help guys who has trouble with generating HelloWorld project in GRPC using C++ on server & client side.

## Prerequisite

1. required packages (from https://github.com/grpc/grpc/blob/master/BUILDING.md)

```sh
sudo apt-get install build-essential autoconf libtool pkg-config libgflags-dev libgtest-dev clang libc++-dev libssl-dev cmake
```

2. install GO language

```sh
# download go install package from https://golang.org/doc/install?download=go1.12.9.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.12.9.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.bashrc
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib' >> $HOME/.bashrc
```



## build GRPC

### clone the GRPC repository

```sh
git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
```

### compile & install GRPC

```sh
mkdir build && cd build
cmake ..
make -j 4
sudo make install
```

### solve can't found **gRPCTargets.cmake** issue

follow the above instrcutions cmake will not automatically generate gRPCTargets.cmake, which makes gRPCConfig.cmake.in totally useless, and you will get the following error:

```
include could not find load file:

  /usr/local/lib/cmake/grpc/gRPCTargets.cmake
```



the current workaround is to rebuild the GRPC repository with different flags based on the first build & install.

```sh
cd .. # go back to grpc root folder
mkdir build_post && cd build_post
cmake -DgRPC_INSTALL=ON -DgRPC_ZLIB_PROVIDER=package -DgRPC_CARES_PROVIDER=package -DgRPC_PROTOBUF_PROVIDER=package -DgRPC_SSL_PROVIDER=package ..
make -j 4
sudo make install
```

now if you go to `/usr/local/lib/cmake` you will find gRPCTargets.cmake has been correctly generated.

```sh
cmake
├── benchmark
│   ├── benchmarkConfig.cmake
│   ├── benchmarkConfigVersion.cmake
│   ├── benchmarkTargets.cmake
│   └── benchmarkTargets-noconfig.cmake
├── c-ares
│   ├── c-ares-config.cmake
│   ├── c-ares-targets.cmake
│   └── c-ares-targets-noconfig.cmake
├── grpc
│   ├── gRPCConfig.cmake
│   ├── gRPCConfigVersion.cmake
│   ├── gRPCTargets.cmake
│   └── gRPCTargets-noconfig.cmake
└── protobuf
    ├── protobuf-config.cmake
    ├── protobuf-config-version.cmake
    ├── protobuf-module.cmake
    ├── protobuf-options.cmake
    ├── protobuf-targets.cmake
    └── protobuf-targets-noconfig.cmake
```

## Run HelloWorld Demo

### generate server & client side C++ code

```sh
protoc -I ./protos/ --grpc_out=./protos --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` helloworld.proto  ## generate C++ server side code
protoc -I ./protos/ --cpp_out=./protos helloworld.proto  ## generate C++ client side code
```

### build HelloWorld Demo

```sh
mkdir build && cd build
cmake ..
make -j 4
```

### run server & client

```sh
cd bin
./greeter_server &
./greeter_client
```


# [Appendix]: Use GRPC in your custom project

add the following code into your CMakeLists.txt

```sh
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf CONFIG REQUIRED)  # get protobuf needed by grpc
message(STATUS "Using protobuf ${protobuf_VERSION}")

find_package(gRPC CONFIG REQUIRED)  # get grpc
message(STATUS "Using gRPC ${gRPC_VERSION}")
```