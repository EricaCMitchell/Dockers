FROM ericacmitchell/libint_derivs:latest

# Set non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN apt-get update\
    && apt-get -y install \
    lsb-release coreutils bash-completion \
    apt-transport-https software-properties-common ca-certificates gnupg \
    linux-tools-common build-essential \
    git make ccache \
    gdb valgrind \
    && apt-get -y upgrade

COPY --from=continuumio/miniconda3:latest /opt/conda /opt/conda

ENV PATH="/opt/conda/bin:/opt/conda/include:/opt/intel/oneapi/mkl/latest/bin:/opt/intel/oneapi/mpi/2021.12/bin:$PATH"
ENV PYTHONPATH="/opt/conda/lib:/opt/intel/oneapi/advisor/2024.1/pythonapi:$PYTHONPATH"
ENV MKL_ROOT="/opt/intel/oneapi/mkl/latest"
ENV LAPACK_LIBRARIES="/opt/intel/oneapi/mkl/latest/lib/libmkl_rt.so;/opt/intel/oneapi/mkl/latest/lib/libmkl_rt.so"
ENV LD_LIBRARY_PATH="/usr/lib64:/opt/intel/oneapi/compiler/2024.1/lib:/opt/intel/oneapi/compiler/2024.1/opt/compiler/lib:$LD_LIBRARY_PATH"

# Usage examples
RUN set -ex && \
    conda config --set always_yes yes --set changeps1 no && \
    conda info -a && \
    conda config --add channels conda-forge && \
    conda install --quiet --freeze-installed -c conda-forge mamba

RUN mamba install --quiet psi4 pybind11 \
    jax jaxlib \
    hdf5 h5py \
    pytest \
    && mamba clean --all

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Cleanup the image
RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=C.UTF-8
ENV PATH="/usr/local/psi4/bin:$PATH"
ENV KMP_DUPLICATE_LIB_OK=TRUE

# [Optional] Set the default user. Omit if you want to keep the default as root.
USER $USERNAME
