# [Choice] Ubuntu version (use jammy on local arm64/Apple Silicon): jammy, focal
ARG VARIANT="jammy"
FROM buildpack-deps:${VARIANT}-curl

# Set non-root user
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    # Install C++ tools
    && apt-get -y install \
    lsb-release build-essential software-properties-common \
    ca-certificates ccache gpg-agent wget \
    && apt-get -y upgrade

RUN wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | tee /usr/share/keyrings/intel-oneapi-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/intel-oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main " > /etc/apt/sources.list.d/oneAPI.list

# Install software dependencies
RUN apt-get update \
    # Install C++ tools
    && apt-get -y install \
    cmake ninja-build \
    libboost-all-dev libeigen3-dev \
    intel-basekit intel-hpckit \
    && apt-get -y upgrade

ARG LIBINT_VERSION=2.9.0

WORKDIR /home

RUN wget https://github.com/EricaCMitchell/libint/releases/download/v0.1/libint-${LIBINT_VERSION}-maxderiv2-maxam5_mm4f12ob2.tgz \
    && tar -xzf libint-${LIBINT_VERSION}-maxderiv2-maxam5_mm4f12ob2.tgz

WORKDIR /home/libint2

ENV CMAKE_PREFIX_PATH='/opt/intel/oneapi/tbb/2021.12/env/..:/opt/intel/oneapi/mkl/2024.1/lib/cmake:/opt/intel/oneapi/ipp/2021.11/lib/cmake/ipp:/opt/intel/oneapi/dpl/2022.5/lib/cmake/oneDPL:/opt/intel/oneapi/dnnl/2024.1/lib/cmake:/opt/intel/oneapi/dal/2024.2:/opt/intel/oneapi/compiler/2024.1'
ENV LD_LIBRARY_PATH='/opt/intel/oneapi/tbb/2021.12/env/../lib/intel64/gcc4.8:/opt/intel/oneapi/mpi/2021.12/opt/mpi/libfabric/lib:/opt/intel/oneapi/mpi/2021.12/lib:/opt/intel/oneapi/mkl/2024.1/lib:/opt/intel/oneapi/ippcp/2021.11/lib/:/opt/intel/oneapi/ipp/2021.11/lib:/opt/intel/oneapi/dpl/2022.5/lib:/opt/intel/oneapi/dnnl/2024.1/lib:/opt/intel/oneapi/debugger/2024.1/opt/debugger/lib:/opt/intel/oneapi/dal/2024.2/lib:/opt/intel/oneapi/compiler/2024.1/opt/oclfpga/host/linux64/lib:/opt/intel/oneapi/compiler/2024.1/opt/compiler/lib:/opt/intel/oneapi/compiler/2024.1/lib:/opt/intel/oneapi/ccl/2021.12/lib/'
ENV MKL_ROOT='/opt/intel/oneapi/mkl/2024.1'
ENV PATH='/opt/intel/oneapi/vtune/2024.1/bin64:/opt/intel/oneapi/mpi/2021.12/bin:/opt/intel/oneapi/mkl/2024.1/bin/:/opt/intel/oneapi/dpcpp-ct/2024.1/bin:/opt/intel/oneapi/dev-utilities/2024.1/bin:/opt/intel/oneapi/debugger/2024.1/opt/debugger/bin:/opt/intel/oneapi/compiler/2024.1/opt/oclfpga/bin:/opt/intel/oneapi/compiler/2024.1/bin:/opt/intel/oneapi/advisor/2024.1/bin64:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
ENV TBB_ROOT='/opt/intel/oneapi/tbb/2021.12/env/..'
ENV GDB_INFO='/opt/intel/oneapi/debugger/2024.1/share/info/'

RUN cmake -GNinja ../libint-${LIBINT_VERSION}-derivs -DBUILD_SHARED_LIBS=ON -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
    -DCMAKE_C_COMPILER=/opt/intel/oneapi/compiler/2024.1/bin/icx -DCMAKE_CXX_COMPILER=/opt/intel/oneapi/compiler/2024.1/bin/icpx \
    && ninja -j8 install

WORKDIR /

# Cleanup the image
RUN rm -r /home/libint-${LIBINT_VERSION}-maxderiv2-maxam5_mm4f12ob2.tgz /home/libint-${LIBINT_VERSION}-derivs /home/libint2

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

RUN apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=C.UTF-8
ENV ACL_BOARD_VENDOR_PATH='/opt/Intel/OpenCLFPGA/oneAPI/Boards'
ENV ADVISOR_2024_DIR='/opt/intel/oneapi/advisor/2024.1'
ENV APM='/opt/intel/oneapi/advisor/2024.1/perfmodels'
ENV CCL_CONFIGURATION='cpu_gpu_dpcpp'
ENV CCL_CONFIGURATION_PATH=''
ENV CCL_ROOT='/opt/intel/oneapi/ccl/2021.12'
ENV CLASSPATH='/opt/intel/oneapi/mpi/2021.12/share/java/mpi.jar'
ENV CMPLR_ROOT='/opt/intel/oneapi/compiler/2024.1'
ENV CPATH='/opt/intel/oneapi/tbb/2021.12/env/../include:/opt/intel/oneapi/mpi/2021.12/include:/opt/intel/oneapi/mkl/2024.1/include:/opt/intel/oneapi/ippcp/2021.11/include:/opt/intel/oneapi/ipp/2021.11/include:/opt/intel/oneapi/dpl/2022.5/include:/opt/intel/oneapi/dpcpp-ct/2024.1/include:/opt/intel/oneapi/dnnl/2024.1/include:/opt/intel/oneapi/dev-utilities/2024.1/include:/opt/intel/oneapi/dal/2024.2/include/dal:/opt/intel/oneapi/compiler/2024.1/opt/oclfpga/include:/opt/intel/oneapi/ccl/2021.12/include'
ENV DALROOT='/opt/intel/oneapi/dal/2024.2'
ENV DAL_MAJOR_BINARY='2'
ENV DAL_MINOR_BINARY='0'
ENV DIAGUTIL_PATH='/opt/intel/oneapi/dpcpp-ct/2024.1/etc/dpct/sys_check/sys_check.sh:/opt/intel/oneapi/debugger/2024.1/etc/debugger/sys_check/sys_check.py:/opt/intel/oneapi/compiler/2024.1/etc/compiler/sys_check/sys_check.sh'
ENV DNNLROOT='/opt/intel/oneapi/dnnl/2024.1'
ENV DPL_ROOT='/opt/intel/oneapi/dpl/2022.5'
ENV FI_PROVIDER_PATH='/opt/intel/oneapi/mpi/2021.12/opt/mpi/libfabric/lib/prov:/usr/lib/x86_64-linux-gnu/libfabric'
ENV FPGA_VARS_ARGS=''
ENV FPGA_VARS_DIR='/opt/intel/oneapi/compiler/2024.1/opt/oclfpga'
ENV INFOPATH='/opt/intel/oneapi/debugger/2024.1/opt/debugger/lib'
ENV INTELFPGAOCLSDKROOT='/opt/intel/oneapi/compiler/2024.1/opt/oclfpga'
ENV INTEL_PYTHONHOME='/opt/intel/oneapi/debugger/2024.1/opt/debugger'
ENV IPPCP_TARGET_ARCH='intel64'
ENV IPPCRYPTOROOT='/opt/intel/oneapi/ippcp/2021.11'
ENV IPPROOT='/opt/intel/oneapi/ipp/2021.11'
ENV IPP_TARGET_ARCH='intel64'
ENV I_MPI_ROOT='/opt/intel/oneapi/mpi/2021.12'
ENV LIBRARY_PATH='/opt/intel/oneapi/tbb/2021.12/env/../lib/intel64/gcc4.8:/opt/intel/oneapi/mpi/2021.12/lib:/opt/intel/oneapi/mkl/2024.1/lib/:/opt/intel/oneapi/ippcp/2021.11/lib/:/opt/intel/oneapi/ipp/2021.11/lib:/opt/intel/oneapi/dpl/2022.5/lib:/opt/intel/oneapi/dnnl/2024.1/lib:/opt/intel/oneapi/dal/2024.2/lib:/opt/intel/oneapi/compiler/2024.1/lib:/opt/intel/oneapi/ccl/2021.12/lib/'
ENV MANPATH='/opt/intel/oneapi/mpi/2021.12/share/man:/opt/intel/oneapi/debugger/2024.1/share/man:/opt/intel/oneapi/compiler/2024.1/share/man:'
ENV NLSPATH='/opt/intel/oneapi/mkl/2024.1/share/locale/%l_%t/%N:/opt/intel/oneapi/compiler/2024.1/lib/compiler/locale/%l_%t/%N'
ENV OCL_ICD_FILENAMES='libintelocl_emu.so:libalteracl.so:/opt/intel/oneapi/compiler/2024.1/lib/libintelocl.so'
ENV ONEAPI_ROOT='/opt/intel/oneapi'
ENV PKG_CONFIG_PATH='/opt/intel/oneapi/vtune/2024.1/include/pkgconfig/lib64:/opt/intel/oneapi/tbb/2021.12/env/../lib/pkgconfig:/opt/intel/oneapi/mpi/2021.12/lib/pkgconfig:/opt/intel/oneapi/mkl/2024.1/lib/pkgconfig:/opt/intel/oneapi/ippcp/2021.11/lib/pkgconfig:/opt/intel/oneapi/dpl/2022.5/lib/pkgconfig:/opt/intel/oneapi/dnnl/2024.1/lib/pkgconfig:/opt/intel/oneapi/dal/2024.2/lib/pkgconfig:/opt/intel/oneapi/compiler/2024.1/lib/pkgconfig:/opt/intel/oneapi/ccl/2021.12/lib/pkgconfig/:/opt/intel/oneapi/advisor/2024.1/include/pkgconfig/lib64:'
ENV PYTHONPATH='/opt/intel/oneapi/advisor/2024.1/pythonapi'
ENV SETVARS_COMPLETED='1'
ENV VTUNE_PROFILER_2024_DIR='/opt/intel/oneapi/vtune/2024.1'
ENV VTUNE_PROFILER_DIR='/opt/intel/oneapi/vtune/2024.1'

# [Optional] Set the default user. Omit if you want to keep the default as root.
USER $USERNAME
