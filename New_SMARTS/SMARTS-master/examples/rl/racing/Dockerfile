FROM tensorflow/tensorflow:2.4.0-gpu

ARG DEBIAN_FRONTEND=noninteractive

# Install libraries.
RUN apt-get update --fix-missing && \
    apt-get install -y \
        software-properties-common && \
    add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y \
        ffmpeg \
        libspatialindex-dev \
        python3.8 \
        python3.8-venv \
        xorg && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

# Update default python version.
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3.8 1

# Setup virtual environment and install pip.
ENV VIRTUAL_ENV=/opt/.venv
RUN python3.8 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
RUN pip install --upgrade pip wheel

# Setup SUMO.
RUN pip install eclipse-sumo==1.10.0
ENV SUMO_HOME $VIRTUAL_ENV/lib64/python3.8/site-packages/sumo

# Install requirements.txt .
COPY ./examples/rl/racing/requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

# Copy source files and install.
COPY . /src
WORKDIR /src
RUN pip install --no-cache-dir -e ./examples/rl/racing

# Build scenarios and maps.
RUN scl scenario build-all --clean ./examples/rl/racing/scenarios

# For Envision.
EXPOSE 8081

# Optimizing compiler for machine learning. See https://www.tensorflow.org/xla .
ENV TF_XLA_FLAGS="--tf_xla_enable_xla_devices --tf_xla_auto_jit=2"

# Suppress message of missing /dev/input folder.
RUN echo "mkdir -p /dev/input" >> ~/.bashrc

SHELL ["/bin/bash", "-c", "-l"]