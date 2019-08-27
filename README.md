# GRPC C++ Hello World Using cmake (ubuntu 18.04)

This demo is aimed to help guys who has trouble with generating HelloWorld project in GRPC using C++ on server & client side.

## Prerequisite

1. required packages (from https://github.com/grpc/grpc/blob/master/BUILDING.md)

```sh
sudo apt-get install build-essential autoconf libtool pkg-config libgflags-dev libgtest-dev clang libc++-dev libssl-dev cmake
```

2. install GO language

```sh
# download go install package of linux-amd64 version
sudo tar -C /usr/local -xzf go*linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.bashrc
echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib' >> $HOME/.bashrc
source $HOME/.bashrc
```

## build GRPC

### clone the GRPC repository

```sh
git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
```

### Install zlib / cares / protbuf 
preinstall these three packages will allow grpc to automatically generate *targets.cmake, otherwise you will receive the following error if you follow the official build guide.

```
include could not find load file:

  /usr/local/lib/cmake/grpc/gRPCTargets.cmake
```

#### install zlib

```sh
# in grpc/third_party/zlib dir
mkdir build && cd build
cmake ..
make -j 4
sudo make install

```

#### install cares

```sh
# in grpc/third_party/cares/cares dir
mkdir build && cd build
cmake ..
make -j 4
sudo make install
```

#### install protobuf

```sh
# in grpc/third_party/protobuf/cmake dir
mkdir build && cd build
cmake -Dprotobuf_BUILD_TESTS=OFF ..
make -j 4
sudo make install
```

### install GRPC

```sh
# in grpc root dir
mkdir build && cd build
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
PROTO_SRC_DIR=./protos
protoc -I $PROTO_SRC_DIR --grpc_out=./protos --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` $PROTO_SRC_DIR/helloworld.proto  ## generate C++ server side code
protoc -I $PROTO_SRC_DIR --cpp_out=./protos $PROTO_SRC_DIR/helloworld.proto  ## generate C++ client side code
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


## [Appendix]: Use GRPC in your custom project

add the following code into your CMakeLists.txt

```sh
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf CONFIG REQUIRED)  # get protobuf needed by grpc
message(STATUS "Using protobuf ${protobuf_VERSION}")

find_package(gRPC CONFIG REQUIRED)  # get grpc
message(STATUS "Using gRPC ${gRPC_VERSION}")
```