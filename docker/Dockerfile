FROM ubuntu:19.04 AS nuttx-tools

RUN apt-get update && apt-get install -y \
  curl \
  flex \
  bison \
  gperf \
  libncurses5-dev \
  make \
  && rm -rf /var/lib/apt/lists/*


RUN mkdir /tools
WORKDIR /tools

RUN mkdir -p /tools/nuttx-tools
RUN curl -s -L https://bitbucket.org/nuttx/tools/get/9ad3e1ee75c7.tar.gz \
  | tar -C nuttx-tools --strip-components=1 -xz

RUN cd nuttx-tools/kconfig-frontends \
  && ./configure --enable-mconf --disable-gconf --disable-qconf --enable-static --prefix=/tools/kconfig-frontends \
  && make install

RUN cd nuttx-tools \
  && mkdir genromfs \
  && tar -C genromfs --strip-components=1 -xf genromfs-0.5.2.tar.gz \
  && cd genromfs \
  && make install PREFIX=/tools/genromfs

CMD [ "/bin/bash" ]


FROM ubuntu:19.04
MAINTAINER Brennan Ashton<bashton@brennanashton.com>

RUN dpkg --add-architecture i386
RUN apt-get update && apt-get install -y \
  git \
  automake \
  build-essential \
  gperf \
  wget \
  libx11-dev \
  libxext-dev \
  u-boot-tools \
  lib32z1-dev \
  libc6-dev-i386 \
  libx11-dev:i386 \
  libxext-dev:i386 \
  curl \
  gettext \
  libcurl4-openssl-dev \
  xxd \
  unzip \
  && rm -rf /var/lib/apt/lists/*

RUN mkdir /tools
WORKDIR /tools

# Pull in the tools we just built for nuttx
COPY --from=nuttx-tools /tools/genromfs/ /tools/genromfs/
ENV PATH="/tools/genromfs/usr/bin:$PATH"
COPY --from=nuttx-tools /tools/kconfig-frontends/ kconfig-frontends/
ENV PATH="/tools/kconfig-frontends/bin:$PATH"

# Download the latest ARM GCC toolchain prebuilt by ARM
RUN mkdir gcc-arm-none-eabi && \
  curl -s -L  "https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2?revision=108bd959-44bd-4619-9c19-26187abf5225&la=en&hash=E788CE92E5DFD64B2A8C246BBA91A249CB8E2D2D" \
  | tar -C gcc-arm-none-eabi --strip-components 1 -xj
ENV PATH="/tools/gcc-arm-none-eabi/bin:$PATH"

# Download the pinguino compilers. Note we remove the 8-bit tools.
# If the 8 bit tools are needed just remove the last line
RUN mkdir pinguino-compilers && \
  curl -s -L "https://github.com/PinguinoIDE/pinguino-compilers/archive/62db5158d7f6d41c6fadb05de81cc31dd81a1958.tar.gz" \
  | tar -C pinguino-compilers --strip-components=2 --wildcards -xz */linux64 && \
  rm -rf pinguino-compilers/p8
ENV PATH="/tools/pinguino-compilers/p32/bin:$PATH"
#ENV PATH="/tools/pinguino-compilers/p8/bin:$PATH"

# Download the latest RISCV GCC toolchain prebuilt by SiFive
RUN mkdir riscv64-unknown-elf-gcc && \
  curl -s -l "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz" \
  | tar -C riscv64-unknown-elf-gcc --strip-components 1 -xz
ENV PATH="/prebuilt/riscv64-unknown-elf-gcc/bin:$PATH"
 

CMD [ "/bin/bash" ]
