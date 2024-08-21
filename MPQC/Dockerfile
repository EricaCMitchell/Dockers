ARG VARIANT="jammy"
FROM buildpack-deps:${VARIANT}-curl

ENV DEBIAN_FRONTEND=noninteractive
LABEL dev.containers.features="common"

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ENV LANGUAGE en_US:en
ENV LANG en_US.UTF-8 
ENV LC_ALL en_US.UTF-8

RUN mkdir -p /home/ValeevGroup/
COPY ./Makefile /home/ValeevGroup/
WORKDIR /home/ValeevGroup/

RUN apt-get -yq update && \
    apt-get -yq install make wget curl

# main, cmake
RUN make install/main && \
    make install/cmake && \
    apt-get clean -y

# Add MKL repo, download
RUN make apt-add-repository/intel-mkl && \
    make download/intel-mkl && \
    make /home/ValeevGroup/ install/intel-mkl

# libint
RUN make install/libint2 && \
    rm -fr /home/ValeevGroup/libint2 \
    /home/ValeevGroup/libint-${libint2_version}-mpqc4.tgz \
    /home/ValeevGroup/libint-${libint2_version}

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

# Set links to solve ninja dependencies when compiling
RUN ln -s /usr/include /include

# Set Intel oneAPI paths
ENV CMAKE_PREFIX_PATH='/opt/intel/oneapi/tbb/2021.13/env/..:/opt/intel/oneapi/mkl/2024.2/lib/cmake:/opt/intel/oneapi/compiler/2024.2:/usr/local'
ENV CPATH='/opt/intel/oneapi/tbb/2021.13/env/../include:/opt/intel/oneapi/mkl/2024.2/include'
ENV PATH='/opt/intel/oneapi/mkl/2024.2/bin/:/opt/intel/oneapi/debugger/2024.2/opt/debugger/bin:/opt/intel/oneapi/compiler/2024.2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
ENV MKLROOT='/opt/intel/oneapi/mkl/2024.2'
ENV TBBROOT='/opt/intel/oneapi/tbb/2021.13/env/..'
ENV LD_LIBRARY_PATH='/opt/intel/oneapi/tbb/2021.13/env/../lib/intel64/gcc4.8:/opt/intel/oneapi/mkl/2024.2/lib:/opt/int:/opt/intel/oneapi/debugger/2024.2/opt/debugger/lib'

# [Optional] Set the default user. Omit if you want to keep the default as root.
USER $USERNAME
