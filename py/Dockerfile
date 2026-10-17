# Python with the wxy package from wxylib/py, built at a pinned commit.
# wxylib/py pins its own wxylib/core commit in its Cargo.toml.
#
#   docker build -t wxy-py py
#   docker run --rm wxy-py python -c "import wxy; print(wxy.affine((0, 10, 0, 5), (10, 5)))"

ARG PY_REV=d55f1bf68b997cc061db47948645a752a826e834
ARG PYTHON_VERSION=3.13

FROM python:${PYTHON_VERSION}-bookworm AS build
ARG PY_REV
ENV RUSTUP_HOME=/opt/rustup CARGO_HOME=/opt/cargo PATH=/opt/cargo/bin:$PATH
RUN curl -sSf -o /tmp/rustup-init \
      https://static.rust-lang.org/rustup/dist/$(uname -m)-unknown-linux-gnu/rustup-init \
 && chmod +x /tmp/rustup-init \
 && /tmp/rustup-init -y -q --profile minimal --default-toolchain stable \
 && rm /tmp/rustup-init \
 && pip install -q "maturin>=1.8,<2"
WORKDIR /src
RUN git init -q py \
 && git -C py fetch -q --depth 1 https://github.com/wxylib/py "$PY_REV" \
 && git -C py checkout -q FETCH_HEAD
# abi3 wheel, so it installs into any Python >= 3.9; the runtime image
# shares this image's Debian release, so the glibc it links against matches
RUN cd py && maturin build --release --locked --compatibility linux --out /wheels

FROM python:${PYTHON_VERSION}-slim-bookworm
ARG PY_REV
LABEL org.opencontainers.image.source="https://github.com/wxylib/images" \
      org.opencontainers.image.description="Python with the wxy package (wxylib/py)" \
      org.opencontainers.image.licenses="MIT OR Apache-2.0" \
      io.wxylib.py.rev="$PY_REV"
COPY --from=build /wheels /wheels
RUN pip install --no-cache-dir /wheels/*.whl && rm -rf /wheels
WORKDIR /data
CMD ["python"]
