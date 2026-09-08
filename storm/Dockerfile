# syntax=docker/dockerfile:1.4
FROM ubuntu:25.04

ENV DEBIAN_FRONTEND=noninteractive

# --- Parametri per pin/branch (puoi passare --build-arg STORM_REF=<sha> ecc.)
ARG STORM_REF=master
ARG STORMPY_REF=master

# 1) Toolchain & dipendenze native (Storm + stormpy) + GUI/X11
RUN apt-get update && apt-get install -y --no-install-recommends \
    # toolchain & build
    git cmake ninja-build build-essential g++ pkg-config \
    # dipendenze core Storm
    libboost-all-dev libgmp-dev libcln-dev libginac-dev libeigen3-dev \
    libhwloc-dev libtbb-dev bison flex zlib1g-dev \
    # python
    python3 python3-venv python3-pip python3-dev \
    # GUI/X11 e OpenGL (per PyQt/Tk)
    python3-tk x11-apps libgl1 libglib2.0-0 libx11-xcb1 libxcb1 libxcb-glx0 \
    libxcb-keysyms1 libxcb-image0 libxcb-shm0 libxcb-sync1 libxcb-xfixes0 \
    libxcb-randr0 libxcb-render0 libxcb-xinerama0 libxkbcommon-x11-0 \
    libqt5gui5 mesa-utils libosmesa6 \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# 2) Venv Python
RUN python3 -m venv /opt/venv \
 && /opt/venv/bin/pip install --upgrade pip wheel setuptools


RUN apt update && apt install -y libz3-dev z3 libglpk-dev

# 3) Clona e compila Storm (branch/commit = ${STORM_REF})
RUN git clone https://github.com/moves-rwth/storm.git \
 && cd storm && git checkout ${STORM_REF} \
 && cmake -S . -B build -G Ninja \
      -DCMAKE_BUILD_TYPE=Release \
      -DSTORM_USE_INT=ON \
      -DSTORM_USE_GMP=ON \
      -DSTORM_USE_CLN=ON \
      -DSTORM_USE_Z3=ON \
      -DBUILD_TESTING=OFF \
 && cmake --build build --parallel 4 --verbose \
 && cmake --install build \
 && ldconfig

# 4) stormpy da sorgente, compilato contro la Storm appena installata
#    - pybind11[global] espone i file CMake di pybind11 al sistema (utile in build C++)
RUN git clone https://github.com/moves-rwth/stormpy.git \
 && cd stormpy \
 && git checkout ${STORMPY_REF:-master} \
 && /opt/venv/bin/pip install --upgrade pip wheel setuptools \
    cmake "pybind11[global]" ninja scikit-build-core \
 && CMAKE_BUILD_PARALLEL_LEVEL=4 /opt/venv/bin/pip install --no-build-isolation .
 
# 5) Altri pacchetti Python utili nel tuo flusso
RUN . /opt/venv/bin/activate \
 && /opt/venv/bin/pip install --no-cache-dir --disable-pip-version-check --progress-bar off \
      networkx matplotlib ipykernel PyQt5 PySide2 tk

# 6) Cartella runtime come nel tuo Dockerfile
RUN mkdir -p /tmp/runtime-docker && chmod 700 /tmp/runtime-docker

# 7) Ambiente: usa il venv di default
ENV PATH="/opt/venv/bin:${PATH}"

# 8) Workdir, volume e cmd
WORKDIR /storm_ws
VOLUME ["/Scrivania/data"]
CMD ["/bin/bash"]