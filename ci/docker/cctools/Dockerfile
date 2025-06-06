# docker build -t clickhouse/cctools .

# This is a hack to significantly reduce the build time of the clickhouse/binary-builder
# It's based on the assumption that we don't care of the cctools version so much
# It even does not depend on the clickhouse/fasttest in the `docker/images.json`
ARG FROM_TAG=latest
FROM clickhouse/fasttest:$FROM_TAG AS builder

ENV CC=clang-${LLVM_VERSION}
ENV CXX=clang++-${LLVM_VERSION}

RUN git clone https://github.com/tpoechtrager/apple-libtapi.git \
    && cd apple-libtapi \
    && git checkout 15dfc2a8c9a2a89d06ff227560a69f5265b692f9 \
    && INSTALLPREFIX=/cctools NINJA=1 ./build.sh \
    && ./install.sh \
    && cd .. \
    && rm -rf apple-libtapi

# Build and install tools for cross-linking to Darwin (x86-64)
# Build and install tools for cross-linking to Darwin (aarch64)
RUN git clone https://github.com/tpoechtrager/cctools-port.git --branch 986-ld64-711 \
    && apt-get update \
    && apt-get install --yes \
        make \
    && apt-get clean \
    && cd cctools-port/cctools \
    && ./configure --prefix=/cctools --with-libtapi=/cctools \
        --target=x86_64-apple-darwin \
    && make install -j$(nproc) \
    && make clean \
    && ./configure --prefix=/cctools --with-libtapi=/cctools \
        --target=aarch64-apple-darwin \
    && make install -j$(nproc) \
    && cd ../.. \
    && rm -rf cctools-port

#
# GDB
#
# ld from binutils is 2.38, which has the following error:
#
#     DWARF error: invalid or unhandled FORM value: 0x23
#
ENV LD=ld.lld-${LLVM_VERSION}
ARG GDB_VERSION=16.2
RUN apt-get update \
    && apt-get install --yes \
        libgmp-dev \
        libmpfr-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /var/cache/debconf /tmp/*
RUN wget https://sourceware.org/pub/gdb/releases/gdb-$GDB_VERSION.tar.gz \
    && tar -xvf gdb-$GDB_VERSION.tar.gz \
    && cd gdb-$GDB_VERSION \
    && ./configure --prefix=/opt/gdb --with-separate-debug-dir=/usr/lib/debug \
    && make -j $(nproc) \
    && make install \
    && rm -fr gdb-$GDB_VERSION gdb-$GDB_VERSION.tar.gz

# just in case
RUN /opt/gdb/bin/gdb --version

FROM scratch
COPY --from=builder /cctools /cctools
COPY --from=builder /opt/gdb /opt/gdb

# NOTE: this file should be kept in sync with other docker images, since they use COPY from this image
