# Re-tag the upstream pansou-web image into the lazycat registry.
# This wrapper does not vendor pansou-web source (upstream has no LICENSE
# file, so we avoid redistributing source); we only repackage the pre-built
# OCI image the upstream author publishes to GHCR.
#
# Pinned by sha256 digest of pansou-web:latest at the time the wrapper version
# was cut, so re-cutting the wrapper is reproducible until the maintainer
# bumps the pin.
FROM ghcr.io/fish2018/pansou-web@sha256:1857ad4c48a6569410bf019d771b3494a6a379a55bbf81a7345f3f33be093e48
