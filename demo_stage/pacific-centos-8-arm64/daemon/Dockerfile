# CEPH DAEMON IMAGE
# 
# 主要目的是构建一个包含 Ceph 相关工具和配置的容器镜像，进行必要的软件安装和配置，同时进行清理和优化，以保持镜像的最小化
# 
# 使用 ceph/daemon-base:v6.0.10-annotated-pacific-centos-8-arm64 作为基础镜像, 该镜像就是 daemon-base 产出的镜像
# 添加了一系列标签，包括维护者信息、是否为 Ceph 容器、版本信息、Git 仓库信息、Git 分支、提交 ID、仓库是否干净、Ceph 版本等
# 使用 yum 安装了一些必要的软件包和工具，例如 wget、unzip、util-linux、python3-setuptools、udev、device-mapper、sharutils、lsof 和 s3cmd
# 从网络下载并安装了 confd、etcdctl 和 kubectl
# 清理了容器，包括删除不必要的文件和目录、压缩和剥离二进制文件、清理临时文件等
# 添加了 s3cfg 配置文件到 /root/.s3cfg
# 添加了 confd 模板和配置文件到 /etc/confd/templates/ 和 /etc/confd/conf.d/
# 添加了一些脚本文件、Ceph 默认配置文件、检查脚本以及入口脚本等到 /opt/ceph-container/bin/ 和 /opt/ceph-container/etc/
# 复制了 Sree-0.2.tar.gz 文件到 /opt/ceph-container/tmp/sree.tar.gz
# 运行生成入口脚本的脚本 generate_entrypoint.sh，并在完成后删除该脚本
# 使用 bash -n 检查所有脚本的语法
# 设置工作目录为根目录 /
# 设置入口点为 /opt/ceph-container/bin/entrypoint.sh

FROM ceph/daemon-base:v6.0.10-annotated-pacific-centos-8-arm64

# Who is the maintainer ?
LABEL maintainer="Guillaume Abrioux <gabrioux@redhat.com>"

# Is a ceph container ?
LABEL ceph="True"

# What is the actual release ? If not defined, this equals the git branch name
LABEL RELEASE="v6.0.10-annotated"

# What was the url of the git repository
LABEL GIT_REPO="git@github.com:bugwz/ceph-container.git"

# What was the git branch used to build this container
LABEL GIT_BRANCH="v6.0.10-annotated"

# What was the commit ID of the current HEAD
LABEL GIT_COMMIT="2924719a86a41e3c56961ecf4fd38f805ec138b6"

# Was the repository clean when building ?
LABEL GIT_CLEAN="False"

# What CEPH_POINT_RELEASE has been used ?
LABEL CEPH_POINT_RELEASE=""

#======================================================
# Install ceph and dependencies, and clean up
#======================================================


# Escape char after immediately after RUN allows comment in first line
RUN \
    # Install all components for the image, whether from packages or web downloads.
    # Typical workflow: add new repos; refresh repos; install packages; package-manager clean;
    #   download and install packages from web, cleaning any files as you go.
    echo 'Install packages' && \
      yum install -y --setopt=install_weak_deps=False wget unzip util-linux python3-setuptools udev device-mapper && \
      yum install -y --setopt=install_weak_deps=False --enablerepo=powertools \
          sharutils \
          lsof \
           \
           \
           \
          s3cmd && \
    # Centos 8 doesn't have confd/etcdctl/kubectl packages, so install them from web
    echo 'Web install confd' && \
      CONFD_VERSION=0.16.0 && \
      # Assume linux
      CONFD_ARCH=linux-arm64 && \
      wget -q -O /usr/local/bin/confd \
        "https://github.com/kelseyhightower/confd/releases/download/v${CONFD_VERSION}/confd-${CONFD_VERSION}-${CONFD_ARCH}" && \
      chmod +x /usr/local/bin/confd && mkdir -p /etc/confd/conf.d && mkdir -p /etc/confd/templates && \
    echo 'Web install etcdctl' && \
    ETCDCTL_VERSION=v3.2.18 && \
    # Assume linux
    ETCDCTL_ARCH=linux-arm64 && \
    wget -q -O- \
      "https://github.com/coreos/etcd/releases/download/${ETCDCTL_VERSION}/etcd-${ETCDCTL_VERSION}-${ETCDCTL_ARCH}.tar.gz" | \
      tar xfz - --no-same-owner -C/tmp/ etcd-${ETCDCTL_VERSION}-${ETCDCTL_ARCH}/etcdctl && \
    mv /tmp/etcd-${ETCDCTL_VERSION}-${ETCDCTL_ARCH}/etcdctl /usr/local/bin/etcdctl && \
    echo 'Web install kubectl' && \
      KUBECTL_VERSION=v1.8.11 && \
      # Assume linux
      KUBECTL_ARCH=arm64 && \
      wget -q -O /usr/local/bin/kubectl \
        "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/${KUBECTL_ARCH}/kubectl" && \
      chmod +x /usr/local/bin/kubectl && \
    # Clean container, starting with record of current size (strip / from end)
    INITIAL_SIZE="$(bash -c 'sz="$(du -sm --exclude=/proc /)" ; echo "${sz%*/}"')" && \
    #
    #
    # Perform any final cleanup actions like package manager cleaning, etc.
    yum clean all && \
    # Clean daemon-specific files
    # Let's remove easy stuff
    rm -f /usr/bin/{etcd-tester,etcd-dump-logs} && \
    # Let's compress fat binaries but keep them executable
    # As we don't run them often, the performance penalty isn't that big
    for binary in /usr/local/bin/{confd,kubectl} /usr/bin/etcdctl; do \
      if [ -f "$binary" ]; then gzexe $binary && rm -f ${binary}~; fi; \
    done && \
    # Remove etcd since all we need is etcdctl
    rm -f /usr/bin/etcd && \
    # Strip binaries
    bash -c ' \
      function ifstrip () { if compgen -g "$1"; then strip -s "$1"; fi } && \
      ifstrip /usr/local/bin/{confd,kubectl}' && \
    # Uncomment below line for more detailed debug info
    # find / -xdev -type f -exec du -c {} \; |sort -n && \
    echo "CLEAN DAEMON DONE!" && \
    # Clean common files like /tmp, /var/lib, etc.
    rm -rf \
        /etc/{selinux,systemd,udev} \
        /lib/{lsb,udev} \
        /tmp/* \
        /usr/lib{,64}/{locale,udev,dracut} \
        /usr/share/{doc,info,locale,man} \
        /usr/share/{bash-completion,pkgconfig/bash-completion.pc} \
        /var/log/* \
        /var/tmp/* && \
    find  / -xdev -name "*.pyc" -o -name "*.pyo" -exec rm -f {} \; && \
    # ceph-dencoder is only used for debugging, compressing it saves 10MB
    # If needed it will be decompressed
    # TODO: Is ceph-dencoder safe to remove as rook was trying to do?
    # rm -f /usr/bin/ceph-dencoder && \
    if [ -f /usr/bin/ceph-dencoder ]; then gzip -9 /usr/bin/ceph-dencoder; fi && \
    # TODO: What other ceph stuff needs removed/stripped/zipped here?
    # TODO: There was some overlap between this and the ceph clean? Where does it belong?
    #       If it's idempotent, it can *always* live here, even if it doesn't always apply
    # TODO: Should we even strip ceph libs at all?
    bash -c ' \
      function ifstrip () { if compgen -g "$1"; then strip -s "$1"; fi } && \
      ifstrip /usr/lib{,64}/ceph/erasure-code/* && \
      ifstrip /usr/lib{,64}/rados-classes/* && \
      ifstrip /usr/lib{,64}/python*/{dist,site}-packages/{rados,rbd,rgw}.*.so && \
      ifstrip /usr/bin/{crushtool,monmaptool,osdmaptool}' && \
    # Photoshop files inside a container ?
    rm -f /usr/lib/ceph/mgr/dashboard/static/AdminLTE-*/plugins/datatables/extensions/TableTools/images/psd/* && \
    # Some logfiles are not empty, there is no need to keep them
    find /var/log/ -type f -exec truncate -s 0 {} \; && \
    #
    #
    # Report size savings (strip / from end)
    FINAL_SIZE="$(bash -c 'sz="$(du -sm --exclude=/proc /)" ; echo "${sz%*/}"')" && \
    REMOVED_SIZE=$((INITIAL_SIZE - FINAL_SIZE)) && \
    echo "Cleaning process removed ${REMOVED_SIZE}MB" && \
    echo "Dropped container size from ${INITIAL_SIZE}MB to ${FINAL_SIZE}MB" && \
    #
    # Verify that the packages installed haven't been accidentally cleaned
    rpm -q \
          sharutils \
          lsof \
           \
           \
           \
          s3cmd && echo 'Packages verified successfully'

#======================================================
# Add ceph-container files
#======================================================

# Add s3cfg file
ADD s3cfg /root/.s3cfg

# Add templates for confd
ADD ./confd/templates/* /etc/confd/templates/
ADD ./confd/conf.d/* /etc/confd/conf.d/

# Add bootstrap script, ceph defaults key/values for KV store
ADD *.sh check_zombie_mons.py ./osd_scenarios/* entrypoint.sh.in disabled_scenario /opt/ceph-container/bin/
ADD ceph.defaults /opt/ceph-container/etc/
# ADD *.sh ceph.defaults check_zombie_mons.py ./osd_scenarios/* entrypoint.sh.in disabled_scenario /

# Copye sree web interface for cn
# We use COPY instead of ADD for tarball so that it does not get extracted automatically at build time
COPY Sree-0.2.tar.gz /opt/ceph-container/tmp/sree.tar.gz

# Modify the entrypoint
RUN bash "/opt/ceph-container/bin/generate_entrypoint.sh" && \
  rm -f /opt/ceph-container/bin/generate_entrypoint.sh && \
  bash -n /opt/ceph-container/bin/*.sh

# Execute the entrypoint
WORKDIR /
ENTRYPOINT ["/opt/ceph-container/bin/entrypoint.sh"]
