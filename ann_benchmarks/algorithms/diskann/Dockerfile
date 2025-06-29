FROM ann-benchmarks

# 1. 基础依赖安装（合并层）
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        wget git cmake g++ libaio-dev libgoogle-perftools-dev \
        clang-format libboost-dev python3 python3-setuptools python3-pip  curl apt-utils\
    && pip3 install pybind11 numpy \
    && rm -rf /var/lib/apt/lists/*

# 2. 安全添加Intel MKL仓库
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB \
        | gpg --dearmor -o /etc/apt/keyrings/intel-mkl.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/intel-mkl.gpg] https://apt.repos.intel.com/mkl all main" \
        > /etc/apt/sources.list.d/intel-mkl.list

# 3. 安装MKL并配置环境
RUN apt-get update \
    && apt-get install -y --no-install-recommends intel-mkl-64bit-2020.0-088 \
    && rm -rf /var/lib/apt/lists/* \
    && update-alternatives --install /usr/lib/x86_64-linux-gnu/libblas.so.3 libblas.so.3 /opt/intel/mkl/lib/intel64/libmkl_rt.so 150 \
    && update-alternatives --install /usr/lib/x86_64-linux-gnu/liblapack.so.3 liblapack.so.3 /opt/intel/mkl/lib/intel64/libmkl_rt.so 150 \
    && echo "/opt/intel/mkl/lib/intel64" > /etc/ld.so.conf.d/mkl.conf \
    && ldconfig \
    && echo "MKL_THREADING_LAYER=GNU" >> /etc/environment

# 4. 构建DiskANN（使用WORKDIR）
RUN git clone --single-branch --branch python_bindings https://github.com/microsoft/diskann \
    && mkdir -p diskann/build \
    && cd diskann/build \
    && cmake -DCMAKE_BUILD_TYPE=Release .. \
    && make -j$(nproc) \
    && cd ../python \
    && pip install -e . \
    && python3 -c "import vamanapy"
