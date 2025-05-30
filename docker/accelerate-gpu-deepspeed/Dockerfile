# Builds GPU docker image of PyTorch specifically
# Uses multi-staged approach to reduce size
# Stage 1
# Use base conda image to reduce time
FROM continuumio/miniconda3:latest AS compile-image
# Specify py version
# Note: DeepSpeed beyond v0.12.6 requires py 3.10
ENV PYTHON_VERSION=3.10
# Install apt libs
RUN apt-get update && \
    apt-get install -y curl git wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists*

# Create our conda env
RUN conda create --name accelerate python=${PYTHON_VERSION} ipython jupyter pip
# We don't install pytorch here yet since CUDA isn't available
# instead we use the direct torch wheel
ENV PATH /opt/conda/envs/accelerate/bin:$PATH
# Activate our bash shell
RUN chsh -s /bin/bash
SHELL ["/bin/bash", "-c"]
# Activate the conda env, install mpy4pi, and install torch + accelerate
RUN source activate accelerate && conda install -c conda-forge mpi4py
RUN source activate accelerate && \
    python3 -m pip install --no-cache-dir \
    git+https://github.com/huggingface/accelerate#egg=accelerate[testing,test_trackers,deepspeed] \
    --extra-index-url https://download.pytorch.org/whl/cu126

RUN python3 -m pip install --no-cache-dir bitsandbytes

# Stage 2
FROM nvidia/cuda:12.6.3-cudnn-devel-ubuntu22.04 AS build-image
COPY --from=compile-image /opt/conda /opt/conda
ENV PATH /opt/conda/bin:$PATH

# Install apt libs
RUN apt-get update && \
    apt-get install -y curl git wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists*

RUN echo "source activate accelerate" >> ~/.profile

# Activate the virtualenv
CMD ["/bin/bash"]