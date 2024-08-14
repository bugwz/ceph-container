# CEPH DAEMON BASE 
# 
# 主要用于创建一个包含 Ceph 相关组件的 CentOS 8 容器镜像，并进行了必要的配置和清理，以确保容器镜像的精简和功能完整
# 
# 基于 centos:8 镜像：需要注意 centos 的官方镜像中，centos8 的最高系列为 centos 8.4.2105
# 设置了多个环境变量，如 I_AM_IN_A_CONTAINER, CEPH_VERSION, CEPH_POINT_RELEASE, CEPH_DEVEL, CEPH_REF, OSD_FLAVOR 等
# 添加了多个标签来描述镜像的元数据，如 maintainer, ceph, RELEASE, GIT_REPO, GIT_BRANCH, GIT_COMMIT, GIT_CLEAN, CEPH_POINT_RELEASE
# 使用 yum 包管理器安装了一些基本的软件包和依赖，如 epel-release, jq
# 根据 CEPH_VERSION 的不同，配置和设置了不同的 Ganesha 和 Ceph 仓库
# 更新系统包并导入 Ceph 的 GPG 公钥
# 安装了 Ceph 的各种组件和插件，如 ceph-common, ceph-mon, ceph-osd, ceph-mds, ceph-mgr, ceph-radosgw, nfs-ganesha 等
# 清理了一些不必要的文件和目录，如 /usr/bin/hyperkube, /usr/bin/etcd, /usr/share/hwdata, /etc/profile.d/lang.sh, /tmp/*, /var/log/* 等
# 压缩了一些调试工具如 ceph-dencoder，以节省空间
# 清理了一些日志文件和 .pyc, .pyo 文件
# 修改 /etc/lvm/lvm.conf 文件，禁用对 udev 的同步和规则
# 创建目录 /var/run/ganesha
# 验证安装的软件包是否存在
# 打印清理过程节省的空间

FROM centos:8

ENV I_AM_IN_A_CONTAINER 1

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

ENV CEPH_VERSION pacific
ENV CEPH_POINT_RELEASE ""
ENV CEPH_DEVEL false
ENV CEPH_REF pacific
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
    echo "[ganesha]" > /etc/yum.repos.d/ganesha.repo ; \
    echo "name=ganesha" >> /etc/yum.repos.d/ganesha.repo ; \
    if [[ "${CEPH_VERSION}" == master ]]; then \
      echo "baseurl=https://buildlogs.centos.org/centos/\$releasever/storage/\$basearch/nfsganesha-3/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=0" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    elif [[ "${CEPH_VERSION}" == pacific ]]; then \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.5-stable/$CEPH_VERSION/el\$releasever/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
      echo "[ganesha-noarch]" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha-noarch" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.5-stable/$CEPH_VERSION/el\$releasever/noarch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    elif [[ "${CEPH_VERSION}" == octopus ]]; then \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.3-stable/$CEPH_VERSION/el\$releasever/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
      echo "[ganesha-noarch]" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "name=ganesha-noarch" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V3.3-stable/$CEPH_VERSION/el\$releasever/noarch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    elif [[ "${CEPH_VERSION}" == nautilus ]]; then \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V2.8-stable/$CEPH_VERSION/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    else \
      echo "baseurl=https://download.ceph.com/nfs-ganesha/rpm-V2.7-stable/$CEPH_VERSION/\$basearch/" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "gpgcheck=1" >> /etc/yum.repos.d/ganesha.repo ; \
      echo "enabled=1" >> /etc/yum.repos.d/ganesha.repo  ; \
    fi ; \
  fi ; \
  if [ -n "tcmu-runner ceph-iscsi python3-rtslib" ]; then \
    curl -s -L https://shaman.ceph.com/api/repos/tcmu-runner/main/latest/centos/8/repo?arch=$(arch) -o /etc/yum.repos.d/tcmu-runner.repo ; \
    if [[ "${CEPH_VERSION}" =~ master ]]; then \
      curl -s -L https://shaman.ceph.com/api/repos/ceph-iscsi/master/latest/centos/8/repo -o /etc/yum.repos.d/ceph-iscsi.repo ; \
    elif [[ "${CEPH_VERSION}" =~ nautilus|octopus|pacific ]]; then \
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
  if [[ "${CEPH_VERSION}" =~ master ]] || ${CEPH_DEVEL}; then \
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
  fi ' && \
yum install -y --setopt=install_weak_deps=False \
        ca-certificates \
        e2fsprogs \
        ceph-common  \
        ceph-mon  \
        ceph-osd \
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
        ceph-radosgw libradosstriper1 \
        nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client \
        tcmu-runner ceph-iscsi python3-rtslib \
        attr \
ceph-fuse \
rbd-nbd \
      	${CRIMSON_PACKAGES} \
        ceph-immutable-object-cache \
        python3-scikit-learn && \
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
        ceph-radosgw libradosstriper1 \
        nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client \
        tcmu-runner ceph-iscsi python3-rtslib \
        attr \
ceph-fuse \
rbd-nbd \
      	${CRIMSON_PACKAGES} \
        ceph-immutable-object-cache \
        python3-scikit-learn ) && \
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
    # ceph-dencoder 仅用于调试，压缩它可以节省 10MB
    # 如果需要，它将被解压缩
    # TODO: ceph-dencoder 是否可以安全移除，正如 rook 试图做的那样？
    # rm -f /usr/bin/ceph-dencoder && \
    if [ -f /usr/bin/ceph-dencoder ]; then gzip -9 /usr/bin/ceph-dencoder; fi && \
    # TODO: What other ceph stuff needs removed/stripped/zipped here?
    # TODO: There was some overlap between this and the ceph clean? Where does it belong?
    #       If it's idempotent, it can *always* live here, even if it doesn't always apply
    # TODO: Should we even strip ceph libs at all?
    # TODO: 这里还需要移除/剥离/压缩其他的 ceph 内容吗？
    # TODO: 这个和 ceph 清理有一些重叠吗？它属于哪里？
    #       如果它是幂等的，它可以*始终*存在于这里，即使它并不总是适用
    # TODO: 我们是否需要剥离 ceph 库？
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
        ca-certificates \
        e2fsprogs \
        ceph-common  \
        ceph-mon  \
        ceph-osd \
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
        ceph-radosgw libradosstriper1 \
        nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw nfs-ganesha-rados-grace nfs-ganesha-rados-urls sssd-client \
        tcmu-runner ceph-iscsi python3-rtslib \
        attr \
ceph-fuse \
rbd-nbd \
      	${CRIMSON_PACKAGES} \
        ceph-immutable-object-cache \
        python3-scikit-learn && echo 'Packages verified successfully'
