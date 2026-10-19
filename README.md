# wxylib/images

Container images for [wxylib](https://github.com/wxylib), published to the
GitHub Container Registry. Each one is built from a pinned commit of its
source repository, so an image tag names exactly the code inside it.

| Image | What is in it | Built from |
|-------|---------------|------------|
| `ghcr.io/wxylib/wxy` | The `wxy` command line tool: `warp`, `points`, `zonal` | [wxylib/core](https://github.com/wxylib/core), `CORE_REV` in [cli/Dockerfile](cli/Dockerfile) |
| `ghcr.io/wxylib/wxy-py` | Python 3.13 with the `wxy` package and numpy | [wxylib/py](https://github.com/wxylib/py), `PY_REV` in [py/Dockerfile](py/Dockerfile) |
| `ghcr.io/wxylib/wxy-r` | R 4.5 (rocker/r-ver) with the `wxy` package | [wxylib/r](https://github.com/wxylib/r), `R_REV` in [r/Dockerfile](r/Dockerfile) |

The Python and R packages pin their own wxylib/core commit in their
`Cargo.toml`, so moving them to newer core means a commit in wxylib/py or
wxylib/r first, then a new `PY_REV` or `R_REV` here.

None of the images contain GDAL or PROJ: everything is pure Rust.

## Tags

- `latest`: the current build of `main`.
- `src-<rev>`: the first seven characters of the pinned source commit, for
  example `ghcr.io/wxylib/wxy:src-f57cdce`.
- `sha-<rev>`: the commit of this repository that built the image.

## Running

The working directory inside each image is `/data`; mount the current
directory there to read and write local files.

### wxy

```sh
docker run --rm ghcr.io/wxylib/wxy --help

# warp a COG to Antarctic polar stereographic, written as GeoZarr
git clone --depth 1 https://github.com/wxylib/fixtures
docker run --rm -v "$PWD":/data ghcr.io/wxylib/wxy \
  warp fixtures/cog/world-int16-deflate.tif out.zarr \
  --crs EPSG:3031 --extent -4e6 -4e6 4e6 4e6 --size 145 145 -r bilinear

# values at lon/lat points, x,y lines on stdin (note -i)
printf '147.3,-42.9\n-70.6,-33.4\n' |
  docker run --rm -i -v "$PWD":/data ghcr.io/wxylib/wxy \
  points fixtures/cog/world-int16-deflate.tif

# statistics over GeoJSON polygons, weighted by exact cell coverage
echo '{"type":"Polygon","coordinates":[[[140,-45],[150,-45],[150,-38],[140,-38],[140,-45]]]}' > box.geojson
docker run --rm -v "$PWD":/data ghcr.io/wxylib/wxy \
  zonal fixtures/cog/world-int16-deflate.tif box.geojson --coverage

# the plan as JSON, reading no data
docker run --rm -v "$PWD":/data ghcr.io/wxylib/wxy \
  zonal fixtures/cog/world-int16-deflate.tif box.geojson --dry-run
```

On Linux, add `--user "$(id -u):$(id -g)"` so files written to the mount
belong to you rather than root.

### Python

```sh
docker run --rm -it ghcr.io/wxylib/wxy-py

docker run --rm ghcr.io/wxylib/wxy-py python -c "
import numpy as np, wxy
lat = np.arange(89.5, -90, -1.0)
z = np.repeat(lat[:, None], 360, axis=1)
w = wxy.warp(z, (-180, 180, -90, 90), 'EPSG:4326',
             dst_extent=(-5e6, 5e6, -5e6, 5e6), dst_dim=(200, 200),
             dst_crs='+proj=stere +lat_0=-90 +lat_ts=-71 +datum=WGS84',
             resample='bilinear')
print(w.shape)
"
```

### R

```sh
docker run --rm -it ghcr.io/wxylib/wxy-r

docker run --rm ghcr.io/wxylib/wxy-r Rscript -e '
library(wxy)
z <- outer(seq(89.5, -89.5), seq(-179.5, 179.5), function(lat, lon) lat)
w <- wxy_warp(z, c(-180, 180, -90, 90), "EPSG:4326",
              dst_extent = c(-5e6, 5e6, -5e6, 5e6), dst_dim = c(200, 200),
              dst_crs = "+proj=stere +lat_0=-90 +lat_ts=-71 +datum=WGS84",
              resample = "bilinear")
dim(w)
'
```

## Building locally

```sh
docker build -t wxy cli
docker build -t wxy-py py
docker build -t wxy-r r

# a different source commit
docker build --build-arg CORE_REV=<commit> -t wxy cli
```

## Publishing

[.github/workflows/publish.yml](.github/workflows/publish.yml) builds all
three images on every pull request and runs a smoke test in each. On a push
to `main` (or a manual run) it also pushes them to `ghcr.io/wxylib`. Images
are linux/amd64 for now.

To move an image to a newer source commit, change its `*_REV` default in
the Dockerfile and open a pull request.

Documentation: **https://wxylib.org/docs/**, one place for what works, how-tos,
explainers, the design documents and how to help.

License: MIT OR Apache-2.0.
