# Java基础知识复习
## Java数据结构之稀疏数组
### 稀疏数组概念
+ 当一个数组中大部分元素为0，或者为同一值时，可以使用稀疏数组来保存该数组。
+ 稀疏数组的处理方式是：
    - 记录数组一共有几行几列，有多少个不同值
    - 把具有不同元素的行列及值记录在一个小规模的数组中，从而缩小程序的规模
### 代码
```java
// 创建一个二维数组 11*11 0：没棋  1：黑棋    2：白棋
    int[][] array1 = new int[11][11];
    array1[1][2] = 1;
    array1[2][3] = 2;
    // 输出原始数组
    System.out.println("输出原始数组");

    for (int[] ints : array1) {
        for (int anInt : ints) {
            System.out.print(anInt + "\t");
        }
        System.out.println();
    }

    System.out.println("=============================");
    // 转换为稀疏数组来保存
    // 获取有效值的个数
    int sum = 0;
    for (int i = 0; i < array1.length; i++) {
        for (int j = 0; j < array1[i].length ; j++) {
            if (array1[i][j] != 0){
                sum++;
            }
        }
    }
    System.out.println("有效值的个数为" + sum);

    // 创建一个稀疏数组
    int[][] array2 = new int[sum+1][3];
    array2[0][0] = array1.length;   // 原始数组的行数
    array2[0][1] = array1[0].length;    // 原始数组的列数
    array2[0][2] = sum; // 原始数组中有效值的个数

    // 遍历原始数组，将非0值，存放在稀疏数组中
    int count = 0;
    for (int i = 0; i < array1.length; i++) {
        for (int j = 0; j < array1[i].length; j++) {
            if(array1[i][j] != 0){
                count++;
                array2[count][0] = i;   // 存放当前有效值在第几行
                array2[count][1] = j;   // 存放当前有效值在第几列
                array2[count][2] = array1[i][j];   // 存放当前有效值
            }
        }
    }
    // 输出稀疏数组
    System.out.println("输出稀疏数组");
    for (int i = 0; i < array2.length; i++) {
        System.out.println(array2[i][0] + "\t"
                + array2[i][1] + "\t"
                + array2[i][2]);
    }
    System.out.println("=============================");
    System.out.println("还原原始数组");
    // 读取稀疏数组
    int[][] array3 = new int[array2[0][0]][array2[0][1]];
    // 给其中的元素还原它的值
    for (int i = 1; i < array2.length; i++) {
        array3[array2[i][0]][array2[i][1]] = array2[i][2];
    }
    System.out.println("输出还原的原始数组");
    for (int[] ints : array3) {
        for (int anInt : ints) {
            System.out.print(anInt+"\t");
        }
        System.out.println();
    }
```
## 面向对象三大特性
### 封装
**高内聚，低耦合。属性私有，get/set**
+ 提高程序安全性，保护数据
+ 隐藏代码的实现细节
+ 统一接口
+ 系统可维护性增加
### 继承
**Java中类只有单继承，没有多继承**
#### super详解
**super注意点：**
1. super调用父类的构造方法，必须在构造方法的第一个
2. super只能出现在子类的方法或者构造方法中
3. super和this不能同时调用构造方法

**super和this的区别**
1. 代表的对象不同
    + this代表调用者本身这个对象
    + super代表父类对象的引用
2. 前提
    + this没有继承也可以使用
    + super只有在继承条件下才能使用
3. 构造方法
    + this() 本类的构造
    + super() 父类的构造



















