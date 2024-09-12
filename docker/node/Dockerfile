FROM golang:1.23-bookworm as genesis-builder
USER root
RUN apt-get update && apt-get install -y \
    make \
    curl \
    git \
    && apt-get clean

WORKDIR /root

# Install Cosmovisor
RUN go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
# Make Genesis binaries
RUN git clone -b v0.33.0 https://github.com/lavanet/lava.git
WORKDIR lava
RUN LAVA_BINARY=lavad make build

##build v0.34.0 binary
FROM golang:1.23-bookworm as upgrade-v0_34_0
USER root
RUN apt-get update && apt-get install -y \
    make \
    curl \
    git \
    && apt-get clean
WORKDIR /root
# Make binaries
RUN git clone -b v0.34.0 https://github.com/lavanet/lava.git
WORKDIR lava
RUN LAVA_BINARY=lavad make build


##build v0.35.0 binary
FROM golang:1.23-bookworm as upgrade-v0_35_0
USER root
RUN apt-get update && apt-get install -y \
    make \
    curl \
    git \
    && apt-get clean
WORKDIR /root
# Make binaries
RUN git clone -b v0.35.0 https://github.com/lavanet/lava.git
WORKDIR lava
RUN LAVA_BINARY=lavad make build


##build v1.0.1 binary
FROM golang:1.23-bookworm as upgrade-v1_0_1
USER root
RUN apt-get update && apt-get install -y \
    make \
    curl \
    git \
    && apt-get clean
WORKDIR /root
# Make binaries
RUN git clone -b v1.0.1 https://github.com/lavanet/lava.git
WORKDIR lava
RUN LAVA_BINARY=lavad make build


##build v2.2.0 binary
FROM golang:1.23-bookworm as upgrade-v2_2_0
USER root
RUN apt-get update && apt-get install -y \
    make \
    curl \
    git \
    && apt-get clean
WORKDIR /root
# Make binaries
RUN git clone -b v2.2.0 https://github.com/lavanet/lava.git
WORKDIR lava
RUN LAVA_BINARY=lavad make build

##build v3.1.0 binary
FROM golang:1.23-bookworm as upgrade-v3_1_0
USER root
RUN apt-get update && apt-get install -y \
    make \
    curl \
    git \
    && apt-get clean
WORKDIR /root
# Make binaries
RUN git clone -b v3.1.0 https://github.com/lavanet/lava.git
WORKDIR lava
RUN LAVA_BINARY=lavad make build


FROM debian:bookworm-slim AS runtime
RUN apt-get update && apt-get install -y \
    curl \
    nano \
    jq \
    dnsutils \
    wget \
    unzip \
    lz4 \
    aria2 \
    pv \
    git \
    iputils-ping \
    iproute2 \
    && apt-get clean

# Create user operator
RUN useradd -m pilot
USER pilot
WORKDIR /home/pilot

# Set Cosmovisor env variables
ENV DAEMON_NAME=lavad
ENV DAEMON_HOME=/home/pilot/.lava
ENV DAEMON_ALLOW_DOWNLOAD_BINARIES=true
ENV DAEMON_RESTART_AFTER_UPGRADE=true
ENV UNSAFE_SKIP_BACKUP=true

RUN mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
RUN ln -s $DAEMON_HOME/cosmovisor/genesis $DAEMON_HOME/cosmovisor/current

ENV PATH=$PATH:$DAEMON_HOME/cosmovisor/current/bin

COPY --chown=pilot:pilot --from=genesis-builder   /go/bin/cosmovisor /usr/local/bin/
COPY --chown=pilot:pilot --from=genesis-builder   /root/lava/build/lavad $DAEMON_HOME/cosmovisor/genesis/bin/
COPY --chown=pilot:pilot --from=upgrade-v0_34_0   /root/lava/build/lavad $DAEMON_HOME/cosmovisor/upgrades/v0.34.0/bin/
COPY --chown=pilot:pilot --from=upgrade-v0_35_0   /root/lava/build/lavad $DAEMON_HOME/cosmovisor/upgrades/v0.35.0/bin/
COPY --chown=pilot:pilot --from=upgrade-v1_0_1    /root/lava/build/lavad $DAEMON_HOME/cosmovisor/upgrades/v1.0.1/bin/
COPY --chown=pilot:pilot --from=upgrade-v2_2_0    /root/lava/build/lavad $DAEMON_HOME/cosmovisor/upgrades/v2.2.0/bin/
COPY --chown=pilot:pilot --from=upgrade-v3_1_0    /root/lava/build/lavad $DAEMON_HOME/cosmovisor/upgrades/v3.1.0/bin/

# workaround to https://github.com/cosmos/cosmos-sdk/issues/20947
RUN mkdir -p $DAEMON_HOME/data $DAEMON_HOME/config $DAEMON_HOME/keyring-test $DAEMON_HOME/keyring-file

ENTRYPOINT ["/usr/local/bin/cosmovisor"]
CMD ["--help"]