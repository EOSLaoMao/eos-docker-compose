FROM ubuntu:16.04

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install openssl git && rm -rf /var/lib/apt/lists/*

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get -y install curl clang-4.0 lldb-4.0 libclang-4.0-dev cmake make libbz2-dev libssl-dev \
        libgmp3-dev autotools-dev build-essential libbz2-dev libicu-dev python3-dev autoconf libtool

RUN git clone -b master --depth 1 https://github.com/EOSIO/eos.git --recursive \
    && cd eos \
    && ./eosio_build.sh \
    && cd build \
    && make install
