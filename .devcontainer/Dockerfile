ARG BASE_IMAGE
FROM ${BASE_IMAGE}
ARG USERNAME=devuser
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG SSH_PORT=22345

# Create the user
RUN groupadd --gid $USER_GID $USERNAME && \
    useradd --uid $USER_UID --gid $USER_GID -m $USERNAME && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

ADD . /.devcontainer

RUN apt-get update && \
    apt install -y --no-install-recommends \
        lsb-release \
        gcc-12 \
        dctrl-tools \
        cpp-12 \
        libgcc-12-dev \
        libasan8 \
        libtsan2 \
        distro-info-data \
        dkms \
        openssh-server \
        && \
    ls -al /.devcontainer && \
    dpkg -i /.devcontainer/*.deb && \
    apt-get autoremove && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/*

RUN curl -L -o nvshmem_src.txz https://developer.nvidia.com/downloads/assets/secure/nvshmem/nvshmem_src_3.2.5-1.txz && \
    tar -xvf nvshmem_src.txz && \
    cd nvshmem_src && \
    git apply /.devcontainer/nvshmem.patch && \
    CUDA_HOME=/usr/local/cuda \
        GDRCOPY_HOME=/tmp/gdrcopy \
        NVSHMEM_SHMEM_SUPPORT=0 \
        NVSHMEM_UCX_SUPPORT=0 \
        NVSHMEM_USE_NCCL=0 \
        NVSHMEM_MPI_SUPPORT=0 \
        NVSHMEM_IBGDA_SUPPORT=1 \
        NVSHMEM_PMIX_SUPPORT=0 \
        NVSHMEM_TIMEOUT_DEVICE_POLLING=0 \
        NVSHMEM_USE_GDRCOPY=1 \
        cmake -S . -B build/ -DCMAKE_INSTALL_PREFIX=/opt/nvshmem && \
    cd build && \
    make -j$(nproc) && \
    make install && \
    rm -rf /.devcontainer

RUN sed -i "s/^Port 22/Port ${SSH_PORT}/" /etc/ssh/sshd_config && \
    ssh-keygen -t rsa -f ~/.ssh/id_rsa -N "" -q && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

USER $USERNAME
