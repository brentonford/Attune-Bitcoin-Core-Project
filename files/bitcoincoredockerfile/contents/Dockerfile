# Build Stage
FROM ${osName}:${osRelease} AS builder

ARG BITCOIN_VERSION=${bitcoinCoreVersion}

# Install dependencies
RUN apt-get update -y && \
        apt-get install -y wget gnupg

# Set working directory for downloads and extraction
WORKDIR /tmp

# Open MAKO raw text tag
<%text>

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

</%text>

# Runtime stage
FROM ${osName}:${osRelease}

# Copy only the binaries from the builder stage
COPY --from=builder /tmp/bitcoin-*/bin/* /usr/local/bin/

# Create bitcoin user with proper /etc/passwd and /etc/group entries
RUN useradd -r -m -d /bitcoin-data -u 1000 bitcoin && \
        echo "bitcoin:x:1000:1000:Bitcoin User:/bitcoin-data:/bin/bash" > /etc/passwd && \
        echo "bitcoin:x:1000:" > /etc/group

USER bitcoin

VOLUME ["/bitcoin-data"]

EXPOSE 8333 8332

ENTRYPOINT ["bitcoind", "-datadir=/bitcoin-data"]