# CEPH DAEMON IMAGE
# 
# 用于构建一个包含 Ceph 守护进程的容器镜像，通过安装必要的软件包、下载特定工具、清理不必要的文件、添加配置文件和脚本，并最终设置入口点脚本来完成容器的构建。
# 
# 基础镜像 ：
#   使用 ceph/daemon-base:v7.0.3-annotated-quincy-centos-8-arm64 作为基础镜像。
# 标签信息 ：
#   maintainer：由 Guillaume Abrioux gabrioux@redhat.com 维护。
#   ceph：标记此镜像为 Ceph 容器。
#   RELEASE：版本为 v7.0.3-annotated。
#   GIT_REPO：Git 仓库 URL 为 git@github.com:bugwz/ceph-container.git。
#   GIT_BRANCH：使用的 Git 分支为 v7.0.3-annotated。
#   GIT_COMMIT：当前 HEAD 的提交 ID 为 2e755540d9102cbf9bbeb5f2feb2053d08bb1645。
#   GIT_CLEAN：构建时仓库是否干净，值为 True。
#   CEPH_POINT_RELEASE：未定义。
# 安装 Ceph 及其依赖 ：
#   使用 yum 安装所需的软件包：wget、unzip、util-linux、python3-setuptools、udev、device-mapper、sharutils、lsof 和 s3cmd。
#   从网络安装 confd、etcdctl 和 kubectl。
#   清理不必要的文件和目录以减小镜像大小。
# 添加 Ceph 容器相关文件 ：
#   添加 s3cfg 文件到 /root/.s3cfg。
#   添加 confd 模板和配置文件到 /etc/confd/templates/ 和 /etc/confd/conf.d/。
#   添加各种脚本、配置文件和默认值到 /opt/ceph-container/bin/ 和 /opt/ceph-container/etc/。
#   复制 Sree-0.2.tar.gz 到 /opt/ceph-container/tmp/sree.tar.gz。
# 修改和执行入口点 ：
#   运行脚本生成入口点，并验证脚本语法。
#   设置工作目录为根目录 /。
#   设置入口点为 /opt/ceph-container/bin/entrypoint.sh

FROM ceph/daemon-base:v7.0.3-annotated-quincy-centos-8-arm64

# Who is the maintainer ?
LABEL maintainer="Guillaume Abrioux <gabrioux@redhat.com>"

# Is a ceph container ?
LABEL ceph="True"

# What is the actual release ? If not defined, this equals the git branch name
LABEL RELEASE="v7.0.3-annotated"

# What was the url of the git repository
LABEL GIT_REPO="git@github.com:bugwz/ceph-container.git"

# What was the git branch used to build this container
LABEL GIT_BRANCH="v7.0.3-annotated"

# What was the commit ID of the current HEAD
LABEL GIT_COMMIT="2e755540d9102cbf9bbeb5f2feb2053d08bb1645"

# Was the repository clean when building ?
LABEL GIT_CLEAN="True"

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
    # Remove etcd since all we need is etcdctl
    rm -f /usr/bin/etcd && \
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
    # ceph-dencoder 仅用于调试，压缩它可以节省 10MB
    # 如果需要，它将被解压缩
    # TODO：ceph-dencoder 是否可以像 rook 尝试的那样安全地删除？
    # rm -f /usr/bin/ceph-dencoder && \
    if [ -f /usr/bin/ceph-dencoder ]; then gzip -9 /usr/bin/ceph-dencoder; fi && \
    # TODO: What other ceph stuff needs removed/stripped/zipped here?
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
