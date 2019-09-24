# 多维数组（Multi-dimensional Arrays）

## 声明

```c
type name[size1][size2]...[sizeN];
```

## 二维数组（2D Arrays）

二维数组实质上是以一维数组为元素而组成的新数组。同理，三维数组是以二维数组为元素而组成的数组，以此类推。

* 声明

```c
type name[x][y];
```

* 数据结构

二维数组 `a[x][y]` 可以看作是具有 x 行 y 列的表。数组 `a[3][4]` 的数据结构如下：

```c
       Column0   Column1   Column2   Column3
     +---------+---------+---------+---------+
Row0 | a[0][0] | a[0][1] | a[0][2] | a[0][3] |
Row1 | a[1][0] | a[1][1] | a[1][2] | a[1][3] |
Row2 | a[2][0] | a[2][1] | a[2][2] | a[2][3] |
     +---------+---------+---------+---------+
```

* 初始化

```c
// 每一行使用嵌套的大括号 {}
// 若缺省元素会自动在相应行补默认值
int a[3][4] = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12},
};

// 省略行的嵌套大括号 {}
// 若缺省元素会自动在末尾补默认值
int a[3][4] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
```

* 访问数组元素

```c
#include <stdio.h>
#define LEN(arr) (sizeof(arr) / sizeof(arr[0]))

int main()
{
    // Row2 缺省两个元素
    int a[3][4] = {
        {1, 2, 3, 4},
        {5, 6},
        {9, 10, 11, 12},
    };

    size_t i, j;
    for (i = 0; i < LEN(a); i++)
        for (j = 0; j < LEN(a[i]); j++)
            printf("a[%zu][%zu] = %d\n", i, j, a[i][j]);

    return 0;
}
```