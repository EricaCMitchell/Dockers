ubuntu_codename ?= $(shell cat /etc/os-release | grep UBUNTU_CODENAME= | sed 's/UBUNTU_CODENAME=//g' )
ubuntu_release ?= $(shell cat /etc/os-release | grep VERSION_ID= | sed 's/VERSION_ID=//g' | sed 's/"//g' )
clang_version ?= 17
libint2_version ?= 2.9.0

export DEBIAN_FRONTEND=noninteractive

ifeq ($(origin gcc_version), undefined)
  	ifeq (${ubuntu_codename}, jammy)
    	gcc_version = 12
  	else
    	gcc_version = 10
  	endif
endif

# N.B. important to install devel package to make the libraries discoverable by the wfn91 linalg cmake kit
intel_mkl := intel-oneapi-mkl-devel
intel_debugger := intel-oneapi-dpcpp-debugger

install/main: install/libboost-latest install/clang
	apt-get update
	apt-get -yq install \
	lsb-release coreutils bash-completion \
	apt-transport-https software-properties-common ca-certificates gnupg \
	linux-tools-common time pciutils \
	build-essential wget curl \
	git make ninja-build \
	gdb valgrind \
	libeigen3-dev \
	libblas-dev liblapack-dev liblapacke-dev \
	libunwind-dev libtbb-dev \
	libopenmpi-dev openmpi-bin libscalapack-openmpi-dev \
	vim emacs-nox ccache

install/gcc-%:
	apt-get -yq install gcc-$* g++-$* gfortran-$*

install/gcc: install/gcc-${gcc_version}

# LLVM
install/clang-%: install/add-apt-repository
	wget https://apt.llvm.org/llvm-snapshot.gpg.key -O - | apt-key add -
	add-apt-repository "deb http://apt.llvm.org/${ubuntu_codename}/ llvm-toolchain-${ubuntu_codename}-$* main"
	apt-get -yq update
	apt-get -yq install clang-$* libomp-$*-dev lldb-$* lld-$*

install/clang: install/clang-${clang_version}

install/add-apt-repository:
	apt-get update
	apt-get -yq install software-properties-common ca-certificates

# VRG stack requires Boost 1.81 or later now, so stock Boost no longer suffices
install/libboost-latest: install/add-apt-repository
	add-apt-repository -y ppa:mhier/libboost-latest
	apt-get update
	apt-get -yq install libboost1.81-all-dev

# CMake
install/cmake: install/add-apt-repository
	wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc | apt-key add -
	apt-add-repository "deb https://apt.kitware.com/ubuntu/ ${ubuntu_codename} main"
	apt-get -yq update
	apt-get -yq install cmake cmake-curses-gui

# Intel
apt-add-repository/intel-mkl:
	wget -O - https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null
	echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | tee /etc/apt/sources.list.d/oneAPI.list
	apt-get -yq update

download/intel-mkl:
	apt-get install -yq --download-only ${intel_mkl} ${intel_debugger}

install/intel-mkl:
	apt-get -yq install ${intel_mkl} ${intel_debugger}
	test -x /opt/intel/oneapi/mkl/latest/env/vars.sh

# Libint2
libint2:
	wget --no-verbose https://github.com/evaleev/libint/releases/download/v${libint2_version}/libint-${libint2_version}-mpqc4.tgz
	tar xzf libint-${libint2_version}-mpqc4.tgz
	mkdir libint2
	cd libint2 && cmake -GNinja ../libint-${libint2_version} -DCMAKE_POSITION_INDEPENDENT_CODE=ON
	cd libint2 && ninja -j6

.PHONY: install/libint2
install/libint2: libint2
	ninja -j8 -Clibint2 install
