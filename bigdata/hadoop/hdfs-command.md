# HDFS ����

Hadoop 1.0 �� 2.0 ����������ͬ����Щ�Ѿ���ʱ��hadoop fsck, hadoop dfsadmin������Щ����ǿ�����ã�hadoop fs����

## namenode

```bash
$ hdfs namenode -format # ��ʽ�� namenode����һ������ namenode ʱӦ�ã�
```

## classpath

```bash
$ hdfs classpath
```

## dfs

[FileSystem shell](./fs-shell.md)

## fsck

```bash
$ hdfs fsck -move # ת��ȱ���ļ�������ĳ���ڵ���ˣ��� /lost+found
$ hdfs fsck -delete # ɾ��ȱ���ļ�
$ hdfs fsck -locations # 
```

## dfsadmin

```bash
$ hdfs dfsadmin -report # �鿴 namenode �� datanode ����Ϣ
$ hdfs dfsadmin -refreshNodes # ˢ�����нڵ�
$ hdfs dfsadmin -safemode [get | enter | leave | wait] # ��ȫģʽ��ֻ�����������д�����г���� hdfs ���ж�д����ʱ��hadoop ��Ⱥ�Զ����밲ȫģʽ��
```

## HDFS ��Ⱥ����

Hadoop 2.x ֧��ʹ�� `hdfs` ���������� HDFS ��Ⱥ��������������� docker ��������

> hadoop 2.7.2 ���������� namenode ���� bug������������ 2.7.3��

namenode
```bash
$ mkdir -p /data/hdfs/dfs/name
$ hdfs namenode -format -D dfs.namenode.name.dir=/data/hdfs/dfs/name # ���ﻹ���� bug��ָ����������Ȼ��Ч
$ hdfs namenode \
-D fs.defaultFS=hdfs://0.0.0.0:9000 \
-D hadoop.tmp.dir=/data/hdfs \
-D dfs.namenode.name.dir=/data/hdfs/dfs/name \
-D dfs.replication=3 \
-D dfs.namenode.datanode.registration.ip-hostname-check=false \
-D dfs.permissions.enabled=false
```

datanode
```bash
$ mkdir -p /data/hdfs/dfs/data
$ hdfs datanode \
-fs hdfs://xxx.xxx.xxx.xxx:9000 \
-D hadoop.tmp.dir=/data/hdfs \
-D dfs.datanode.data.dir=/data/hdfs/dfs/data \
-D dfs.permissions.enabled=false
```

## HDFS �߿��ü�Ⱥ

HDFS High Availability Using the Quorum Journal Manager��



## �ο�����

> [HDFS Commands Guide](http://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html)
> [Hadoop Commands Guide](http://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-common/CommandsManual.html)

