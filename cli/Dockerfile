# The wxy command line tool from wxylib/core, built at a pinned commit.
#
#   docker build -t wxy cli
#   docker run --rm -v "$PWD":/data wxy --help

ARG CORE_REV=f57cdce0eaeaecb73f518b76610e2d58534e8b85

FROM rust:1-bookworm AS build
ARG CORE_REV
WORKDIR /src
RUN git init -q core \
 && git -C core fetch -q --depth 1 https://github.com/wxylib/core "$CORE_REV" \
 && git -C core checkout -q FETCH_HEAD
RUN cargo install --locked --path core/crates/wxy-cli --root /out

FROM debian:bookworm-slim
ARG CORE_REV
LABEL org.opencontainers.image.source="https://github.com/wxylib/images" \
      org.opencontainers.image.description="The wxy command line tool (wxylib/core)" \
      org.opencontainers.image.licenses="MIT OR Apache-2.0" \
      io.wxylib.core.rev="$CORE_REV"
# CA certificates for reading over HTTPS, without apt in this stage
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=build /out/bin/wxy /usr/local/bin/wxy
WORKDIR /data
ENTRYPOINT ["wxy"]
CMD ["--help"]
