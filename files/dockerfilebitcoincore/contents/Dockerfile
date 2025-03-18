# Build Stage
FROM {osName}:{osRelease} AS builder

ARG BITCOIN_VERSION={bitcionCoreVersion}

# Install dependencies
RUN apt-get update -y && \
        apt-get install -y wget gnupg

# Set working directory for downloads and extraction
WORKDIR /tmp

# Download Bitcoin Core tarball and signature
RUN wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS.asc \
        https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS \
        https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/bitcoin-${BITCOIN_VERSION}-aarch64-linux-gnu.tar.gz

# Download and extract guix.sigs builder-keys
RUN wget -O guix-sigs.tar.gz https://github.com/bitcoin-core/guix.sigs/archive/main.tar.gz && \
        mkdir -p builder-keys && \
        tar -xzf guix-sigs.tar.gz --strip-components=1 guix.sigs-main/builder-keys

# Import all keys from builder-keys
RUN gpg --import builder-keys/*

# Verify the signature
RUN gpg --verify SHA256SUMS.asc SHA256SUMS

# Verify the tarball checksum
RUN grep "bitcoin-${BITCOIN_VERSION}-aarch64-linux-gnu.tar.gz" SHA256SUMS | sha256sum -c

# Extract and install Bitcoin Core
RUN tar -xzf bitcoin-${BITCOIN_VERSION}-aarch64-linux-gnu.tar.gz

# Runtime stage
FROM {osName}:{osRelease}

# Copy only the binaries from the builder stage
COPY --from=builder /tmp/bitcoin-*/bin/* /usr/local/bin/

# Create a non-root user for running bitcoind
RUN useradd -r -m -d /bitcoin-data bitcoin

USER bitcoin

VOLUME ["/bitcoin-data"]

EXPOSE 8333 8332

ENTRYPOINT ["bitcoind", "-datadir=/bitcoin-data"]