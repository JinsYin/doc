# Ceph ��������

Ceph�������������ڴ洢��docker image��  
Ceph ��������mĬ�������� Civetweb��Ĭ�϶˿�7480����, �������ǡ�Apache���͡�FastCGI��֮�ϡ�

>[����Ceph���ã�ʹ��CivetWeb���ٴRGW](https://www.ustack.com/blog/civetweb/?belong=industry-news)


## ��װ

ʵ�ʷ��� jewel �汾�� `ceph-deploy install` ��װ ceph ��ʱ��Ĭ���Ѿ���װ�� ceph-radosgw �������ֱ��ʹ�� ceph Դ����װ ceph ʱ�����Զ���װ ceph-radosgw ���������ʹ�� `rpm -qa | grep ceph` ��������°�װ�˵��װ�װ����Щ ceph �����

```bash
# ceph-deploy �ڵ�
$ cd /ceph-cluster
$ ceph-deploy install --rgw <gateway-node1> [<gateway-node2> ...] # ruguo
$ ceph-deploy admin <gateway-node1> [<gateway-node2> ...]��# ���Ϊ����ڵ�
```

```bash
# radosgw �ڵ�
$ rpm -qa | grep ceph-radosgw # ����Ƿ�װ���
$ yum install -y ceph-radosgw # ��ѡ�����û�а�װ�ɹ�������ʹ�ø����װ
```


## �½�����ʵ��

```bash
# ceph-deploy �ڵ�
$ cd /ceph-cluster
$ ceph-deploy rgw create <gateway-node1> [<gateway-node2> ...] # ���� rgw ʵ��
```

```bash
# rgw ����Ĭ�϶˿ڡ�7480
http://gateway-node1:7480
```

```bash
# rgw �ڵ�
$ netstat -tpln | grep radosgw
```


# �޸�Ĭ�϶˿� (��ʱ���޸�)

```bash
# ceph-deploy �ڵ�
$ cd /ceph-cluster
$ cat ceph.conf
...
[client.rgw.centos192]
rgw_frontends = "civetweb port=8000"
```

```bash
# �������õ� rgw �ڵ�
$ ceph-deploy --overwrite-conf config push <gateway-node> [<other-nodes>]
```

```bash
# rgw �ڵ�
$ systemctl restart ceph-radosgw@rgw.centos192.service��# ���� rgw ����
$ systemctl status ceph-radosgw@rgw.centos192.service
```


## ʹ������

Ϊʹ�� REST �ӿڣ�������ҪΪ S3 �ӿڴ���һ����ʼ Ceph ���������û���Ȼ��Ϊ Swift �ӿڴ���һ�����û��������Ҫ��֤�������û��Ƿ���Է������ء�
����
���²������� ceph ����ڵ��ϲ�����
����
1. Ϊ��S3�����ʴ�����radosgw���û�
```bash
$ radosgw-admin user create --uid="registry" --display-name="Docker Registry User" # keys.access_key �� keys.secret_key ��������ʱ����֤
$ radosgw-admin user info --uid="registry" # �鿴rgw �û���Ϣ
$ rados lspools # �����û������������ pool
```

2. Ϊ Swift �ӿڴ���һ�����û�
```bash
# ʹ�����ַ�ʽ���ʼ�Ⱥ����Ҫ�½�һ�� Swift ���û��������û�����������������
$ radosgw-admin subuser create --uid="registry" --subuser=registry:swift --access=full��# �½� Swift �û�
$ radosgw-admin key create --subuser=registry:swift --key-type=swift --gen-secret # ������secret key�����ظ�ִ�л���� secret_key��
```

3. ������֤

����S3����
```bash
# ���Խű������� radosgw, �½�һ���µ� bucket ���г����е� buckets
$ yum install -y python-boto # ��װ python-boto ��
$ vi s3test.py # �޸�access_key, secret_key �� host
$ python s3test.py # ���: my-new-bucket 2015-02-16T17:09:10.000Z
```

���� Swift ����
```bash
# ��װswift�ͻ��� (CentOS)
$ yum install -y python-setuptools
$ easy_install pip
$ pip install --upgrade setuptools
$ pip install --upgrade python-swiftclient
```

```bash
# ��װswift�ͻ��� (Ubuntu)
$ apt-get install -y python-setuptools
$ easy_install pip
$ pip install --upgrade setuptools
$ pip install --upgrade python-swiftclient
```

```bash
# ���: my-new-bucket)
# swift -A http://{IP ADDRESS}:{port}/auth/1.0 -U registry:swift -K '{swift_secret_key}' list
$ swift -A http://192.168.1.192:7480/auth/1.0 -U registry:swift -K 'fcq..' list
$ swift -A http://192.168.1.192:7480/auth/1.0 -U registry:swift -K 'fcq..' stat -v
```

## ���� registry

�һ��ڹٷ��� registry �������¹�����һ������registry-ceph���������޸Ķ˿ں�/��ʹ�á�ceph���Ķ���洢���洢���񣬶����Ǵ洢�ڱ��ء�ͬʱ���������Ҳ������Ϊ��ͨ registry �á�

ʹ�� ceph ����洢
```bash
$ docker run -itd --name registry-ceph -p 80:80 -e HTTP_PORT=80 \
-e STORAGE_SWIFT_AUTHURL="http://192.168.111.192:7480/auth/v1" \ 
-e STORAGE_SWIFT_USERNAME="registry:swift" \
-e STORAGE_SWIFT_PASSWORD="NyINaRDvoFiZUAaCUHsNz6Nzu6glbn729rfDqh7r" \
registry-ceph:2.6.0
```

����ͨ������
```bash
$ docker run -itd --name registry-ceph -p 80:80 -e HTTP_PORT=80 registry-ceph:2.6.0
```

���
```bash
# web
curl http://localhost/v2/_catalog
```

```bash
# �����ϴ�����docker daemon ���� --insecure-registry 192.168.111.0/24 ��������
$ docker tag alpine:3.5 192.168.111.192/alpine:3.5
$ docker push 192.168.111.192/alpine:3.5
$ docker rmi 192.168.111.192/alpine:3.5 && docker pull 192.168.111.192/alpine:3.5
```

```bash
# ���һ�������� /var/lib/registry ��Ӧ�� volume �Ƿ�Ϊ�գ� ����Ӧ�ö����ڡ�ceph���вŶ�
$ rados ls --pool default.rgw.buckets.data | grep repositories/alpine | grep 3.5
```

����
```bash
# ɾ������ ��ͨ��ɾ��������ɾ������ 
$ rados ls --pool default.rgw.buckets.data | grep repositories/alpine
$ obj=$(rados ls --pool default.rgw.buckets.data | grep repositories/alpine)
$ if [ -n "${obj}" ]; then rados rm ${obj} --pool default.rgw.buckets.data; fi
$ rados ls --pool default.rgw.buckets.data | grep repositories/alpine # ���
```

