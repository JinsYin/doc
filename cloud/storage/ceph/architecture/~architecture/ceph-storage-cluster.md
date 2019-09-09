# Ceph 存储集群

### 纠删码

```c


                      +-------------------+
                name  |       NYAN        |
                      +-------------------+
                      +-------------------+
             content  |     ABCDEFGHI     |
                      +-------------------+
                                |
                                v
                        +---------------+
    +-------------------|  encode(3,2)  |-------------------+
    |                   +---------------+                   |
    |                   |       |       |                   |
    |             +-----+       |       +-----+             |
    |             |             |             |             |
+-------+     +-------+     +-------+     +-------+     +-------+
|  NYAN |     |  NYAN |     |  NYAN |     |  NYAN |     |  NYAN |
+-------+     +-------+     +-------+     +-------+     +-------+
+-------+     +-------+     +-------+     +-------+     +-------+
|   1   |     |   2   |     |   3   |     |   4   |     |   5   |
+-------+     +-------+     +-------+     +-------+     +-------+
+-------+     +-------+     +-------+     +-------+     +-------+
|  ABC  |     |  DEF  |     |  GHI  |     |  YXY  |     |  QGC  |
+-------+     +-------+     +-------+     +-------+     +-------+
    |             |             |             |             |
    |             |             v             |             |
    |             |         +-------+         |             |
    |             |         |  OSD1 |         |             |
    |             |         +-------+         |             |
    |             |                           |             |
    |             |         +-------+         |             |
    |             +-------->|  OSD2 |         |             |
    |                       +-------+         |             |
    |                                         |             |
    |                       +-------+         |             |
    |                       |  OSD3 |<--------+             |
    |                       +-------+                       |
    |                                                       |
    |                       +-------+                       |
    |                       |  OSD4 |<----------------------+
    |                       +-------+
    |
    |                       +-------+
    +---------------------->|  OSD5 |
                            +-------+
```

```c


                                +-------------------+
                            name  |       NYAN        |
                                +-------------------+
                                +-------------------+
                        content  |     ABCDEFGHI     |
                                +-------------------+
                                            |
                                            v
                                    +---------------+
                +-------------------|  encode(3,2)  |<----+
                |                   +---------------+     |
                |                           ^             |
                |                           |             |
                |                           |             |
            +-------+     +-------+     +-------+     +-------+
    name    |  NYAN |     |  NYAN |     |  NYAN |     |  NYAN |
            +-------+     +-------+     +-------+     +-------+
            +-------+     +-------+     +-------+     +-------+
  shared    |   1   |     |   2   |     |   3   |     |   4   |
            +-------+     +-------+     +-------+     +-------+
            +-------+     +-------+     +-------+     +-------+
 content    |  ABC  |     |  DEF  |     |  GHI  |     |  YXY  |
            +-------+     +-------+     +-------+     +-------+
                ^             .             ^             ^
                |        Too  .             |             |
                |        Slow ^         +-------+         |
                |             |         |  OSD1 |         |
                |             |         +-------+         |
                |             |                           |
                |             |         +-------+         |
                |             +---------|  OSD2 |         |
                |                       +-------+         |
                |                                         |
                |                       +-------+         |
                |                       |  OSD3 |---------+
                |                       +-------+
                |
                |                       +-------+
                |                       |  OSD4 | OUT
                |                       +-------+
                |
                |                       +-------+
                +-----------------------|  OSD5 |
                                        +-------+
```

```c

   Primary OSD
+---------------+
|     OSD 1     |
|          log  |   Write Full   +-------------+
|  +----+       |<---------------| Ceph Client |
|  |D1v1|  1,1  |       v1       +-------------+
|  +----+       |
+---------------+
        |
        |                +---------------+
        |                |     OSD 2     |
        |                |          log  |
        |--------------> |  +----+       |
        |                |  |D2v1|  1,1  |
        |                |  +----+       |
        |                +---------------+
        |
        |                +---------------+
        |                |     OSD 3     |
        |                |          log  |
        +--------------> |  +----+       |
                         |  |C1v1|  1,1  |
                         |  +----+       |
                         +---------------+

```

```c

   Primary OSD
+---------------+
|     OSD 1     |
|          log  |
|  +----+       |
|  |D1v2|  1,2  |
|  +----+       |   Write Full   +-------------+
|               |<---------------| Ceph Client |
|  +----+       |       v1       +-------------+
|  |D1v1|  1,1  |
|  +----+       |
+---------------+
        |
        |                +---------------+
        |                |     OSD 2     |
        |                |          log  |
        |--------------> |  +----+       |
        |                |  |D2v1|  1,1  |
        |                |  +----+       |
        |                +---------------+
        |
        |                +---------------+
        |                |     OSD 3     |
        |                |          log  |
        +--------------> |  +----+       |
                         |  |C1v2|  1,2  |
                         |  +----+       |
                         |               |
                         |  +----+       |
                         |  |C1v1|  1,1  |
                         |  +----+       |
                         +---------------+
```

```c

   Primary OSD
+---------------+
|     OSD 1     |
|          log  |
|  +----+       |
|  |D1v2|  1,2  |
|  +----+       |   Write Full   +-------------+
|               |<---------------| Ceph Client |
|  +----+       |       v1       +-------------+
|  |D1v1|  1,1  |
|  +----+       |
+---------------+
        |
        |                +---------------+
        |                |     OSD 2     |
        |                |          log  |
        |--------------> |  +----+       |
        |                |  |D2v2|  1,2  |
        |                |  +----+       |
        |                |               |
        |                |  +----+       |
        |                |  |D2v1|  1,1  |
        |                |  +----+       |
        |                +---------------+
        |
        |                +---------------+
        |                |     OSD 3     |
        |                |          log  |
        +--------------> |  +----+       |
                         |  |D2v1|  1,2  |
                         |  +----+       |
                         |               |
                         |  +----+       |
                         |  |C1v1|  1,1  |
                         |  +----+       |
                         +---------------+
```

```c
   Primary OSD
+---------------+
|     OSD 1     |
|          log  |
|  +----+       |
|  |D1v2|  1,2  |
|  +----+       |
+---------------+
        |
        |                +---------------+
        |                |     OSD 2     |
        |                |          log  |
        |--------------> |  +----+       |
        |                |  |D2v2|  1,2  |
        |                |  +----+       |
        |                +---------------+
        |
        |                +---------------+
        |                |     OSD 3     |
        |                |          log  |
        +--------------> |  +----+       |
                         |  |C1v2|  1,2  |
                         |  +----+       |
                         +---------------+
```

```c
+---------------+
|     OSD 1     |
|    (down)     |
+---------------+
        |
        |                +---------------+
        |                |     OSD 2     |
        |                |          log  |
        |--------------> |  +----+       |
        |                |  |D2v1|  1,1  |
        |                |  +----+       |
        |                +---------------+
        |
        |                +---------------+
        |                |     OSD 3     |
        |                |          log  |
        +--------------> |  +----+       |
                         |  |C1v2|  1,2  |
                         |  +----+       |
                         |               |
                         |  +----+       |
                         |  |C1v1|  1,1  |
                         |  +----+       |
                         +---------------+

   Primary OSD
+---------------+
|     OSD 4     |
|          log  |
|               |
|          1,1  |
|               |
+---------------+
```

```c
   Primary OSD
+---------------+
|     OSD 4     |
|          log  |
|  +----+       |
|  |D1v1|  1,1  |
|  +----+       |
+---------------+
        |
        |                +---------------+
        |                |     OSD 2     |
        |                |          log  |
        |--------------> |  +----+       |
        |                |  |D2v1|  1,1  |
        |                |  +----+       |
        |                +---------------+
        |
        |                +---------------+
        |                |     OSD 3     |
        |                |          log  |
        +--------------> |  +----+       |
                         |  |C1v1|  1,1  |
                         |  +----+       |
                         +---------------+

+---------------+
|     OSD 1     |
|    (down)     |
+---------------+
```

```c
      +---------------+
      |  Ceph Client  |
      +---------------+
              ^                Faster I/O
  Tiering is  |             +--------------+
 Transparent  |   +-------->|  Cache Tier  |
     to Ceph  |   |         +--------------+
  Client Ops  |   |              |   ^
              v   v              |   |      Active Data in Cache Tier
      +---------------+          |   |
      |    Objecter   |          |   |
      +---------------+          |   |      Inactive Data in Storage Tier
                  ^              v   |
                  |         +--------------+
                  +-------->| Storage Tier |
                            +--------------+
                               Slower I/O
```