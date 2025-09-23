FROM ubuntu:24.04

# https://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-ubuntu2404.html

# Installing Dependencies
RUN apt update
RUN apt-get install -y  --no-install-recommends \
    git \
    autoconf \
    automake \
    libtool \
    make \
    libreadline-dev \
    texinfo \
    pkg-config \
    libpam0g-dev \
    libjson-c-dev \
    bison \
    flex \
    libc-ares-dev \
    python3-dev \
    python3-sphinx \
    install-info \
    build-essential \
    libsnmp-dev \
    perl \
    libcap-dev \
    libelf-dev \
    libunwind-dev \
    protobuf-c-compiler \
    libprotobuf-c-dev \
    libpcre2-dev

# Install libyang
RUN apt-get install -y cmake
RUN git clone https://github.com/CESNET/libyang.git
WORKDIR /libyang
RUN git checkout v2.1.128
RUN mkdir build
WORKDIR /libyang/build
RUN cmake --install-prefix /usr -D CMAKE_BUILD_TYPE:String="Release" ..
RUN make
RUN make install

# GRPC
RUN apt-get install -y libgrpc++-dev protobuf-compiler-grpc

# Config Rollbacks
RUN apt install libsqlite3-dev

# ZeroMQ (Optional)
RUN apt-get install -y libzmq5 libzmq3-dev

# Building & Installing FRR
RUN apt-get install -y adduser
RUN groupadd -r -g 92 frr
RUN groupadd -r -g 85 frrvty
RUN adduser --system --ingroup frr --home /var/run/frr/ --gecos "FRR suite" --shell /sbin/nologin frr
RUN usermod -a -G frrvty frr

# Compile
RUN git clone https://github.com/frrouting/frr.git /frr
WORKDIR /frr
RUN ./bootstrap.sh
RUN ./configure \
    --prefix=/usr \
    --includedir=\${prefix}/include \
    --bindir=\${prefix}/bin \
    --sbindir=\${prefix}/lib/frr \
    --libdir=\${prefix}/lib/frr \
    --libexecdir=\${prefix}/lib/frr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --with-moduledir=\${prefix}/lib/frr/modules \
    --enable-configfile-mask=0640 \
    --enable-logfile-mask=0640 \
    --enable-snmp \
    --enable-multipath=64 \
    --enable-user=frr \
    --enable-group=frr \
    --enable-vty-group=frrvty \
    --with-pkg-git-version \
    --with-pkg-extra-version=-MyOwnFRRVersion
RUN make
RUN make install

# Install FRR configuration files
RUN install -m 775 -o frr -g frr -d /var/log/frr
RUN install -m 775 -o frr -g frrvty -d /etc/frr
RUN install -m 640 -o frr -g frrvty tools/etc/frr/vtysh.conf /etc/frr/vtysh.conf
RUN install -m 640 -o frr -g frr tools/etc/frr/frr.conf /etc/frr/frr.conf
RUN install -m 640 -o frr -g frr tools/etc/frr/daemons.conf /etc/frr/daemons.conf
RUN install -m 640 -o frr -g frr tools/etc/frr/daemons /etc/frr/daemons
RUN install -m 640 -o frr -g frr tools/etc/frr/support_bundle_commands.conf /etc/frr/support_bundle_commands.conf

# Install systemd and systemctl
RUN install -m 644 tools/frr.service /etc/systemd/system/frr.service
RUN apt-get install -y --no-install-recommends \
    systemd \
    systemd-sysv

# Enable deamons
# RUN sed -i 's/bgpd=no/bgpd=yes/' /etc/frr/daemons \
#  && sed -i 's/zebra=no/zebra=yes/' /etc/frr/daemons
RUN systemctl enable frr

# Start FRRouting and Systemd
ENTRYPOINT ["/sbin/init"]


RUN apt-get install -y  --no-install-recommends iputils-ping
RUN apt-get install -y  --no-install-recommends netcat-openbsd
RUN apt-get install -y  --no-install-recommends traceroute
RUN apt-get install -y  --no-install-recommends tcpdump
RUN apt-get install -y  --no-install-recommends iproute2
