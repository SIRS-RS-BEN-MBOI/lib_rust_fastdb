name: CI

on:
  push:
    branches:
      - main
      - master
    tags:
      - '*'
  pull_request:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  linux:
    runs-on: ${{ matrix.platform.runner }}
    strategy:
      matrix:
        python-version: ["3.12"]
        platform:
          - runner: ubuntu-22.04
            target: x86_64
          - runner: ubuntu-22.04
            target: x86
          - runner: ubuntu-22.04
            target: aarch64
          - runner: ubuntu-22.04
            target: armv7
          - runner: ubuntu-22.04
            target: s390x
          - runner: ubuntu-22.04
            target: ppc64le
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y pkg-config libssl-dev build-essential curl cmake
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - name: Build wheels
        uses: PyO3/maturin-action@v1
        env:
          RUSTFLAGS: '-C target-feature=+crt-static'
        with:
          target: ${{ matrix.platform.target }}
          args: --release --out dist --find-interpreter --verbose
          sccache: ${{ !startsWith(github.ref, 'refs/tags/') }}
          manylinux: 2014
      - name: Upload wheels
        uses: actions/upload-artifact@v4
        with:
          name: wheels-linux-${{ matrix.platform.target }}-py${{ matrix.python-version }}
          path: dist

  musllinux:
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        python-version: ["3.12"]
        platform:
          - target: x86_64
          - target: x86
          - target: aarch64
          - target: armv7
    steps:
      - uses: actions/checkout@v4
      - name: Install OpenSSL
        run: |
          sudo apt-get update
          sudo apt-get install -y pkg-config libssl-dev
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - name: Build wheels
        uses: PyO3/maturin-action@v1
        with:
          target: ${{ matrix.platform.target }}
          args: --release --out dist --find-interpreter --verbose
          sccache: ${{ !startsWith(github.ref, 'refs/tags/') }}
          manylinux: musllinux_1_2
      - name: Upload wheels
        uses: actions/upload-artifact@v4
        with:
          name: wheels-musllinux-${{ matrix.platform.target }}-py${{ matrix.python-version }}
          path: dist

  windows:
    runs-on: ${{ matrix.platform.runner }}
    strategy:
      matrix:
        python-version: ["3.12"]
        platform:
          - runner: windows-latest
            target: x64
          - runner: windows-latest
            target: x86
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          architecture: ${{ matrix.platform.target }}
      - name: Build wheels
        uses: PyO3/maturin-action@v1
        with:
          target: ${{ matrix.platform.target }}
          args: --release --out dist --find-interpreter --verbose
          sccache: ${{ !startsWith(github.ref, 'refs/tags/') }}
      - name: Upload wheels
        uses: actions/upload-artifact@v4
        with:
          name: wheels-windows-${{ matrix.platform.target }}
          path: dist

  macos:
    runs-on: ${{ matrix.platform.runner }}
    strategy:
      matrix:
        python-version: ["3.12"]
        platform:
          - runner: macos-13
            target: x86_64
          - runner: macos-14
            target: aarch64
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - name: Build wheels
        uses: PyO3/maturin-action@v1
        with:
          target: ${{ matrix.platform.target }}
          args: --release --out dist --find-interpreter --verbose
          sccache: ${{ !startsWith(github.ref, 'refs/tags/') }}
      - name: Upload wheels
        uses: actions/upload-artifact@v4
        with:
          name: wheels-macos-${{ matrix.platform.target }}-py${{ matrix.python-version }}
          path: dist

  sdist:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: 3.12
      - name: Build sdist
        uses: PyO3/maturin-action@v1
        with:
          command: sdist
          args: --out dist
      - name: Upload sdist
        uses: actions/upload-artifact@v4
        with:
          name: wheels-sdist
          path: dist

  release:
    name: Release
    runs-on: ubuntu-latest
    if: ${{ startsWith(github.ref, 'refs/tags/') || github.event_name == 'workflow_dispatch' }}
    needs: [linux, musllinux, windows, macos, sdist]
    permissions:
      id-token: write
      contents: write
      attestations: write
    steps:
      - uses: actions/download-artifact@v4
      - name: Generate artifact attestation
        uses: actions/attest-build-provenance@v2
        with:
          subject-path: 'wheels-*/*'
      - name: Publish to PyPI
        if: ${{ startsWith(github.ref, 'refs/tags/') }}
        uses: PyO3/maturin-action@v1
        env:
          MATURIN_PYPI_TOKEN: ${{ secrets.PYPI_API_TOKEN }}
        with:
          command: upload
          args: --non-interactive --skip-existing wheels-*/*
