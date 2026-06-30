FROM rust:1.96.0-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates pkg-config libssl-dev libudev-dev \
    && rm -rf /var/lib/apt/lists/*

RUN cargo install --locked qmmd

CMD ["qmmd"]
