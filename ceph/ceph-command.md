# Ceph ����

����
```bash
# �޸����ú���Ҫ��������ceph-node�� (ceph-admin)
$ ceph-deploy --overwrite-conf config push {node(s)}
```

��������
```bash
$ systemctl restart ceph-osd@3.service # �ڶ�Ӧ�ڵ������� osd.3
$ systemctl restart ceph-mon@centos191.service # �� centos191 �ڵ������� ceph-mon
$ systemctl restart��ceph-radosgw@rgw.centos192.service # �� centos192 �ڵ������� ceph-radosgw
```


## ��Ⱥ״̬

```bash
$ ceph health
$ ceph status # ceph -s
$ ceph osd stat
$ ceph osd dump
$ ceph mon dump
$ ceph quorum_status -f json-pretty
```

## MON 

```bash
$ ceph mon rm centos203 # �Ƴ� monitor
```


## OSD (Object Storage Device)

```bash
$ ceph osd tree��# �鿴 osd Ȩ�غ�����״̬���洢·���� osd.1 --> /var/lib/ceph/osd/ceph-1��
$ ceph osd dump # �鿴 osd �� pool ����ϸ��Ϣ
$ ceph osd pool create mypool 64 64 # �����洢�أ����ֱַ��ʾ pg_num���͡�pgp_num��
```

```bash
# �鿴����Ĵ洢λ����Ϣ
# obj1���洢�ڡ�pg 2.7 �ϣ��� /var/lib/ceph/osd/ceph-[2,0]/current/2.7_head 
$ ceph osd map mypool obj1
osdmap e45 pool 'mypool' (2) object 'obj1' -> pg 2.6cf8deff (2.7) -> up ([2,0], p2) acting ([2,0], p2)
```

```bash
$ ceph osd down 0 # down ��һ�� osd
$ ceph osd rm 0 # ɾ��һ�� osd���Ƴ�һ�� osd ֮ǰӦ���� down ����ȷ������ͬ���������ڵ����ˣ�
$ ceph osd crush rm osd.0 # ɾ��һ�� osd ���̵� crush map
$ ceph osd crush rm centos194��# ɾ��һ�� osd �ġ�host �ڵ�
$ ceph osd out osd.3 # �����Ⱥ
$ ceph osd int osd.3 #�����뼯Ⱥ
```


## RBD (Rados Block Device)

docker ʹ�á�ceph ��洢�洢 docker volume������Ӧ ceph rbd �еġ�image��

```bash
$ rbd create myimage --size 1024 --pool mypool # ��������Ĭ�ϳ��� rbd����λ��MB��
$ rbd ls --pool mypool # �鿴���о�����д: rbd ls mypool
$ rbd info mypool/myimage # ������Ϣ��ͬ��rbd info (--image) myimage --pool mypool
$ rbd rm mypool/myimage # ɾ������
$ rbd cp mypool/myimage mypool/newimage # ��������
$ rbd rename mypool/myimage mypool/img # ������
```

```bash
$ rbd resize mypool/myimage --size 2048 # ���ݣ�
$ rbd resize mypool/myimage --size 512 --allow-shrink # ����
```

```bash
#
$ rbd snap create mypool/myimage@mysnap # �������գ��ɶ����
$ rbd snap ls mypool/myimage # �鿴�������
$ rbd snap rm mypool/myimage@mynap # ɾ�� myimage �����ĳ������
$ rbd snap purge mypool/myimage # ɾ����myimage ��������п���
```

```bash
$ rbd showmapped # �鿴��������ӳ�䣬�� lsblk ��鿴��
$ rbd unmap /dev/rbd1 # ȡ�������� /dev/rbd1����ӳ��
```


## RADOS

```bash
$ rados mkpool mypool # pg_num��pgp_num ΪĬ�ϣ�ceph osd pool create mypool 64 64
$ rados rmpool mypool # ɾ���洢�أ�ceph osd pool rm mypool
$ rados lspools��# �鿴���д洢�أ�ceph osd pool ls
```

```bash
$ rados ls --pool mypool # ��ѯָ�����е����� objects
$ rados df # ��������object����������Ϣ
$ rados put obj1 /etc/hosts --pool mypool # �ϴ��ļ���Ϊ mypool ���еĶ��� obj1
$ rados stat obj1 --pool mypool # �鿴������Ϣ
```


## RGW (Rados Gateway)
  
��

## PG

```bash
$ ceph pg dump
$ bash ./pg_num_per_osd.sh # ��ѯÿ�� osd �ϵġ�pg ������
```

## CRUSH map

```bash
$��ceph osd getcrushmap -o crush.map��# ������ǰ��Ⱥ�� crush map
$ crushtool -d crush.map -o crush.txt # �����루crush.map�����ɶ���crush.txt���ɶ���
```

```bash
$��crushtool -c new_crush.txt -o new_crush.map��# ���±��루���޸ĺ����±��롡crush map��
$ ceph osd setcrushmap -i new_crush.map # �� crush map ���õ���Ⱥ��ȥ
```
