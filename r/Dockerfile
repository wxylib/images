# R with the wxy package from wxylib/r, built at a pinned commit.
# wxylib/r pins its own wxylib/core commit in src/rust/Cargo.toml.
#
#   docker build -t wxy-r r
#   docker run --rm wxy-r Rscript -e "wxy::wxy_affine(c(0, 10, 0, 5), c(10, 5))"

ARG R_REV=0b479dd4b94b83ad9c0abc389c8c421aa16a20f4
ARG R_VERSION=4.5

FROM rocker/r-ver:${R_VERSION} AS build
ARG R_REV
RUN apt-get update \
 && apt-get install -y --no-install-recommends git curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*
ENV RUSTUP_HOME=/opt/rustup CARGO_HOME=/opt/cargo PATH=/opt/cargo/bin:$PATH
RUN curl -sSf -o /tmp/rustup-init \
      https://static.rust-lang.org/rustup/dist/$(uname -m)-unknown-linux-gnu/rustup-init \
 && chmod +x /tmp/rustup-init \
 && /tmp/rustup-init -y -q --profile minimal --default-toolchain stable \
 && rm /tmp/rustup-init
WORKDIR /src
RUN git init -q r \
 && git -C r fetch -q --depth 1 https://github.com/wxylib/r "$R_REV" \
 && git -C r checkout -q FETCH_HEAD
RUN R CMD INSTALL r

# The package links Rust statically, so the runtime image needs no Rust
FROM rocker/r-ver:${R_VERSION}
ARG R_REV
LABEL org.opencontainers.image.source="https://github.com/wxylib/images" \
      org.opencontainers.image.description="R with the wxy package (wxylib/r)" \
      org.opencontainers.image.licenses="MIT OR Apache-2.0" \
      io.wxylib.r.rev="$R_REV"
COPY --from=build /usr/local/lib/R/site-library/wxy /usr/local/lib/R/site-library/wxy
WORKDIR /data
CMD ["R"]
