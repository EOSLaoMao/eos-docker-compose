FROM ubuntu:16.04

RUN WORK_DIR=$PWD \
TEMP_DIR=/tmp \
DOXYGEN=false \
CXX_COMPILER=clang++-4.0 \
C_COMPILER=clang-4.0 \
MONGOD_CONF=${HOME}/opt/mongodb/mongod.conf

RUN BUILD_DIR=${WORK_DIR}/build

RUN export PATH=${HOME}/opt/mongodb/bin:$PATH
RUN export BOOST_ROOT=${HOME}/opt/boost_1_66_0
RUN export OPENSSL_ROOT_DIR=/usr/include/openssl
RUN export WASM_ROOT=${HOME}/opt/wasm


RUN MEM_MEG=$( free -m | grep Mem | tr -s ' ' | cut -d\  -f2 )
RUN CPU_CORE=$( lscpu | grep "^CPU(s)" | tr -s ' ' | cut -d\  -f2 )

RUN apt-get update && DEBIAN_FRONTEND=noninteractive && apt-get install -y clang-4.0 lldb-4.0 libclang-4.0-dev cmake make libbz2-dev libssl-dev \
        libgmp3-dev autotools-dev build-essential libbz2-dev libicu-dev python3-dev autoconf libtool curl lcov

RUN apt-get update && DEBIAN_FRONTEND=noninteractive && apt-get install -y python-dev libxml2-dev libxslt-dev

RUN curl -LO -w '%{http_code}' --connect-timeout 30 https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.bz2 \
        &&tar xf ${TEMP_DIR}/boost_1_66_0.tar.bz2 \
        &&cd ${TEMP_DIR}/boost_1_66_0/ \
        &&./bootstrap.sh --prefix=${BOOST_ROOT} \
        &&./b2 install

RUN mkdir ${HOME}/opt \
        &&cd ${HOME}/opt \
        &&curl -LO -w '%{http_code}' --connect-timeout 30 https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.3.tgz \
        &&tar xf mongodb-linux-x86_64-3.6.3.tgz \
        &&ln -s ${HOME}/opt/mongodb-linux-x86_64-3.6.3/ mongodb \
        &&mkdir ${HOME}/opt/mongodb/data \
        &&mkdir ${HOME}/opt/mongodb/log \
        &&touch ${HOME}/opt/mongodb/log/mongodb.log

RUN cd ${TEMP_DIR} \
        &&curl -LO -w '%{http_code}' --connect-timeout 30 https://github.com/mongodb/mongo-c-driver/releases/download/1.9.3/mongo-c-driver-1.9.3.tar.gz \
        &&tar xf mongo-c-driver-1.9.3.tar.gz \
        &&cd mongo-c-driver-1.9.3 \
        &&./configure --enable-static --with-libbson=bundled --enable-ssl=openssl --disable-automatic-init-and-cleanup --prefix=/usr/local \
        &&make -j \
        &&make install

RUN apt-get update && DEBIAN_FRONTEND=noninteractive && apt-get install -y git

RUN cd ${TEMP_DIR} \
        &&git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1 \
        &&cd mongo-cxx-driver/build \
        &&cmake -DBUILD_SHARED_LIBS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local .. \
        &&make -j \
        &&make install

RUN cd ${TEMP_DIR} \
        &&git clone https://github.com/cryptonomex/secp256k1-zkp.git \
        &&cd secp256k1-zkp \
        &&./autogen.sh \
        &&./configure \
        &&make -j \
        &&make install

RUN cd ${TEMP_DIR} \
        &&mkdir llvm-compiler  2>/dev/null \
        &&cd llvm-compiler \
        &&git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git \
        &&cd llvm/tools \
        &&git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git \
        &&cd .. \
        &&mkdir build \
        &&cd build \

RUN cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=${HOME}/opt/wasm -DLLVM_TARGETS_TO_BUILD= \
        -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../ \
        &&make -j install \


RUN COMPILE_EOS=1 \
COMPILE_CONTRACTS=1 \
CMAKE_BUILD_TYPE=Release

RUN cd ${WORK_DIR} \
        &&mkdir -p ${BUILD_DIR} \
        &&cd ${BUILD_DIR} \

RUN cmake -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_CXX_COMPILER=${CXX_COMPILER} \
        -DCMAKE_C_COMPILER=${C_COMPILER} -DWASM_ROOT=${WASM_ROOT} \
        -DOPENSSL_ROOT_DIR=${OPENSSL_ROOT_DIR} -DBUILD_MONGO_DB_PLUGIN=true \
        -DBUILD_DOXYGEN=${DOXYGEN} ..



RUN make -j${CPU_CORE}

