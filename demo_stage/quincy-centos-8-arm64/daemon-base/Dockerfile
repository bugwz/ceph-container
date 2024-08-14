# CEPH DAEMON BASE IMAGE
# 
# 主要功能是构建一个包含 Ceph 及其依赖的优化过的容器镜像，同时包含了详细的版本控制和清理步骤，以减少最终镜像的大小。
# 
# 基础镜像 ：
#   使用 centos:8 作为基础镜像。需要注意 centos8 官方的镜像版本只到 8.4.2105
# 环境变量 ：
#   设置 I_AM_IN_A_CONTAINER 为 1，标识在容器中运行。
#   定义了一系列与 Ceph 相关的环境变量，如 CEPH_VERSION（默认值为 quincy）、CEPH_POINT_RELEASE、CEPH_DEVEL 等。
# 标签（Labels） ：
#   标记维护者信息。
#   标记该镜像是一个 Ceph 容器。
#   记录构建镜像时的 Git 仓库信息、分支、提交 ID 等。
# 安装依赖和 Ceph：
#   安装 epel-release 和 jq。
#   根据 CEPH_VERSION 的不同，配置相应的 NFS-Ganesha 和 Ceph-iscsi 的 Yum 仓库。
#   更新系统并导入 Ceph 官方的 GPG 密钥。
#   安装 Ceph 及其依赖组件，包括 ceph-mon、ceph-osd、ceph-mgr 等，以及其他工具如 nfs-ganesha、tcmu-runner、ceph-iscsi 等。
# 清理和优化 ：
#   移除不必要的文件和目录（如 /etc/selinux、/var/log/* 等）。
#   压缩 ceph-dencoder，以减少镜像大小。
#   清理日志文件，移除 Python 编译文件等。
#   打印清理前后的镜像大小变化。
# 验证安装 ：
#   通过 rpm -q 验证已安装的包，确保清理过程中没有误删除。

FROM centos:8

ENV I_AM_IN_A_CONTAINER 1

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

ENV CEPH_VERSION quincy
ENV CEPH_POINT_RELEASE ""
ENV CEPH_DEVEL false
ENV CEPH_REF quincy
ENV OSD_FLAVOR default

#======================================================
# Install ceph and dependencies, and clean up
#======================================================



# Escape char after immediately after RUN allows comment in first line
RUN \
    # Install all components for the image, whether from packages or web downloads.
    # Typical workflow: add new repos; refresh repos; install packages; package-manager clean;
    #   download and install packages from web, cleaning any files as you go.
    # Installs should support install of ganesha for luminous
    yum install -y epel-release && \
yum install -y jq && \
bash -c ' \
  if [ -n "nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client" ]; then \
    if [[ "${CEPH_VERSION}" == master || "${CEPH_VERSION}" == main ]]; then \
      curl -s -L "https://shaman.ceph.com/api/repos/nfs-ganesha/next/latest/centos/8/flavors/ceph_main/repo" -o /etc/yum.repos.d/ganesha.repo ; \
    elif  [[ "${CEPH_VERSION}" == quincy ]]; then \
      echo "[ganesha]" > /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://buildlogs.centos.org/centos/\$releasever/storage/\$basearch/nfsganesha-4/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=0" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    elif [[ "${CEPH_VERSION}" == pacific ]]; then \
      echo "[ganesha]" > /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.5-stable/$CEPH_VERSION/el\$releasever/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
      echo "[ganesha-noarch]" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha-noarch" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.5-stable/$CEPH_VERSION/el\$releasever/noarch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    elif [[ "${CEPH_VERSION}" == octopus ]]; then \
      echo "[ganesha]" > /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.3-stable/$CEPH_VERSION/el\$releasever/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
      echo "[ganesha-noarch]" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha-noarch" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.3-stable/$CEPH_VERSION/el\$releasever/noarch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    elif [[ "${CEPH_VERSION}" == nautilus ]]; then \
      echo "[ganesha]" > /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V2.8-stable/$CEPH_VERSION/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    else \
      echo "[ganesha]" > /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V2.7-stable/$CEPH_VERSION/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    fi ; \
  fi ; \
  if [ -n "tcmu-runner ceph-iscsi python3-rtslib" ]; then \
    curl -s -L https://shaman.ceph.com/api/repos/tcmu-runner/main/latest/centos/8/repo?arch=$(arch) -o /etc/yum.repos.d/tcmu-runner.repo ; \
    if [[ "${CEPH_VERSION}" =~ master || "${CEPH_VERSION}" == main ]]; then \
      curl -s -L https://shaman.ceph.com/api/repos/ceph-iscsi/main/latest/centos/8/repo -o /etc/yum.repos.d/ceph-iscsi.repo ; \
    elif [[ "${CEPH_VERSION}" =~ nautilus|octopus|pacific|quincy ]]; then \
      curl -s -L https://download.ceph.com/ceph-iscsi/3/rpm/el8/ceph-iscsi.repo -o /etc/yum.repos.d/ceph-iscsi.repo ; \
    else \
      curl -s -L https://download.ceph.com/ceph-iscsi/2/rpm/el8/ceph-iscsi.repo -o /etc/yum.repos.d/ceph-iscsi.repo ; \
    fi ; \
  fi' && \
yum update -y --setopt=install_weak_deps=False && \
rpm --import 'https://download.ceph.com/keys/release.asc' && \
if [[ "${CEPH_VERSION}" == nautilus ]]; then \
  CEPH_MGR_K8SEVENTS="ceph-mgr-k8sevents"; \
  if [[ -n "" ]]; then \
    CPR= ; \
    if [[ ${CPR:1:2} -eq 14 ]] && [[ ${CPR:4:1} -eq 2 ]] && [[ ${CPR:6} -lt 5 ]]; then \
      CEPH_MGR_K8SEVENTS="" ; \
    fi ; \
  fi ; \
fi && \
bash -c ' \
  if [[ "${CEPH_VERSION}" =~ master  || "${CEPH_VERSION}" == main ]] || ${CEPH_DEVEL}; then \
    ARCH=$(arch); \
    if [[ "${ARCH}" == "aarch64" ]]; then \
      ARCH="arm64"; \
    fi ; \
    REPO_URL=$(curl -s "https://shaman.ceph.com/api/search/?project=ceph&distros=centos/8/${ARCH}&flavor=${OSD_FLAVOR}&ref=${CEPH_REF}&sha1=latest" | jq -r .[0].url); \
    RELEASE_VER=0 ;\
    if [[ "${OSD_FLAVOR}" == "crimson" ]]; then \
     CRIMSON_PACKAGES="ceph-crimson-osd";\
    fi ;\
  else \
    RELEASE_VER=1 ;\
    REPO_URL="http://download.ceph.com/rpm-${CEPH_VERSION}/el8/"; \
  fi && \
  rpm -Uvh "$REPO_URL/noarch/ceph-release-1-${RELEASE_VER}.el8.noarch.rpm" && \
  if [[ 8 -eq 8 ]]; then \
    yum install -y dnf-plugins-core ; \
    yum copr enable -y tchaikov/python-scikit-learn ; \
    yum copr enable -y tchaikov/python3-asyncssh ; \
  fi ' && \
yum install -y --setopt=install_weak_deps=False \
        ca-certificates \
        e2fsprogs \
        ceph-common  \
        ceph-mon  \
        ceph-osd \
        cephfs-top \
        ceph-mds \
cephfs-mirror \
        rbd-mirror  \
        ceph-mgr \
ceph-mgr-cephadm \
ceph-mgr-dashboard \
ceph-mgr-diskprediction-local \
ceph-mgr-k8sevents \
ceph-mgr-rook \
python3-saml\
        ceph-grafana-dashboards \
        kmod \
        lvm2 \
        gdisk \
        smartmontools \
        nvme-cli \
        libstoragemgmt \
        systemd-udev \
        sg3_utils \
        procps-ng \
        hostname \
        ceph-radosgw libradosstriper1 \
        nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client \
        tcmu-runner ceph-iscsi python3-rtslib \
        attr \
ceph-fuse \
rbd-nbd \
        ${CRIMSON_PACKAGES} \
        ceph-immutable-object-cache \
        python3-scikit-learn \
        ceph-volume && \
    # Clean container, starting with record of current size (strip / from end)
    INITIAL_SIZE="$(bash -c 'sz="$(du -sm --exclude=/proc /)" ; echo "${sz%*/}"')" && \
    #
    #
    # Perform any final cleanup actions like package manager cleaning, etc.
    echo 'Postinstall cleanup' && \
 (rm -rf "/usr/bin/hyperkube /usr/bin/etcd /usr/bin/systemd-analyze /usr/share/hwdata/{iab.txt,oui.txt} /etc/profile.d/lang.sh" && \
   yum clean all && \
   rpm -q \
        ca-certificates \
        e2fsprogs \
        ceph-common  \
        ceph-mon  \
        ceph-osd \
        cephfs-top \
        ceph-mds \
cephfs-mirror \
        rbd-mirror  \
        ceph-mgr \
ceph-mgr-cephadm \
ceph-mgr-dashboard \
ceph-mgr-diskprediction-local \
ceph-mgr-k8sevents \
ceph-mgr-rook \
python3-saml\
        ceph-grafana-dashboards \
        kmod \
        lvm2 \
        gdisk \
        smartmontools \
        nvme-cli \
        libstoragemgmt \
        systemd-udev \
        sg3_utils \
        procps-ng \
        hostname \
        ceph-radosgw libradosstriper1 \
        nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client \
        tcmu-runner ceph-iscsi python3-rtslib \
        attr \
ceph-fuse \
rbd-nbd \
        ${CRIMSON_PACKAGES} \
        ceph-immutable-object-cache \
        python3-scikit-learn \
        ceph-volume ) && \
    # Tweak some configuration files on the container system
    # disable sync with udev since the container can not contact udev
sed -i -e 's/udev_rules = 1/udev_rules = 0/' -e 's/udev_sync = 1/udev_sync = 0/' -e 's/obtain_device_list_from_udev = 1/obtain_device_list_from_udev = 0/' /etc/lvm/lvm.conf && \
# validate the sed command worked as expected
grep -sqo "udev_sync = 0" /etc/lvm/lvm.conf && \
grep -sqo "udev_rules = 0" /etc/lvm/lvm.conf && \
grep -sqo "obtain_device_list_from_udev = 0" /etc/lvm/lvm.conf && \
mkdir -p /var/run/ganesha && \
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
        ca-certificates \
        e2fsprogs \
        ceph-common  \
        ceph-mon  \
        ceph-osd \
        cephfs-top \
        ceph-mds \
cephfs-mirror \
        rbd-mirror  \
        ceph-mgr \
ceph-mgr-cephadm \
ceph-mgr-dashboard \
ceph-mgr-diskprediction-local \
ceph-mgr-k8sevents \
ceph-mgr-rook \
python3-saml\
        ceph-grafana-dashboards \
        kmod \
        lvm2 \
        gdisk \
        smartmontools \
        nvme-cli \
        libstoragemgmt \
        systemd-udev \
        sg3_utils \
        procps-ng \
        hostname \
        ceph-radosgw libradosstriper1 \
        nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client \
        tcmu-runner ceph-iscsi python3-rtslib \
        attr \
ceph-fuse \
rbd-nbd \
        ${CRIMSON_PACKAGES} \
        ceph-immutable-object-cache \
        python3-scikit-learn \
        ceph-volume && echo 'Packages verified successfully'
