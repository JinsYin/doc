# 安装 Ceph 客户端

## RHEL

### 安装

```sh
# 安装和集群相同的版本
$ export CEPH_RELEASE="mimic"
$ export CEPH_VERSION=13.2.8

$ yum install -y yum-utils \
  && yum-config-manager --add-repo https://dl.fedoraproject.org/pub/epel/7/x86_64/ \
  && yum install --nogpgcheck -y epel-release \
  && rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 \
  && rm -f /etc/yum.repos.d/dl.fedoraproject.org*

# Install ceph-release and ceph-common (cn.ceph.com)
$ rpm --import 'https://download.ceph.com/keys/release.asc' \
  && rpm -Uvh http://download.ceph.com/rpm-${CEPH_RELEASE}/el7/noarch/ceph-release-1-1.el7.noarch.rpm \
  && yum install -y ceph-common-${CEPH_VERSION} libradosstriper1-${CEPH_VERSION} librgw2-${CEPH_VERSION}
```

### 配置

```sh
# 从 Monitor 节点拷贝 Ceph 配置文件和 admin.keyring 到客户端节点
$ scp /etc/ceph/<cluster_name>.conf <user>@<client_host>:/etc/ceph/
$ scp /etc/ceph/<cluster_name>.client.admin.keyring <user>@<client_host:/etc/ceph/
```

## DEBIAN

```sh
# 安装和集群相同的版本
$ CEPH_RELEASE="mimic"

$ wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -
  && echo deb http://cn.ceph.com/debian-${CEPH_RELEASE}/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
  && sudo apt-get update && sudo apt-get install ceph
```
