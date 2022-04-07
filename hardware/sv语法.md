<!--
 * @Author: weiwei
 * @Date: 2022-04-04 22:49:53
 * @LastEditors: WeiWei
 * @LastEditTime: 2022-04-05 22:43:06
 * @FilePath: /notes/hardware/sv语法.md
 * @Description: system verilog 语法
-->

# System Verilog 语法总结

## Subroutines 子程序(task and function)

![sub](../assets/Screen%20Shot%202022-04-04%20at%2010.52.20%20pm.png)

- task与function最大的区别有两点

  - task 没有返回值，function 有返回值
  - Tasks can block, Functions cannot block
  - task可以添加消耗时间的语句，而function不可以消耗时间 (这一点与verilog相同)。
  - task可以调用task和function，而function仅能调用function。

- task和function中是不能使用initial和always的

- 如果不消耗时间（require time passage），尽量把 subroutines 写成 function, 保证在 task和 function 中可用

- 可以 value 或者 ref 传参

### 传参位置

![argument](../assets/Screen%20Shot%202022-04-05%20at%2012.08.11%20am.png)

- input     -copy value in at beginning -default
- output    -copy value out at end
- inout     -copy in at beginning and out at end
- ref       – pass by reference, makes the argument variable the same as the calling variable

  - Changes to argument variable will change calling variable immediately

- const ref     –pass by reference but read only
  - Saves time and memory for passing arrays to tasks & functions

![test](../assets/Screen%20Shot%202022-04-05%20at%2012.12.23%20am.png)

## 生命周期

![lifetime](../assets/Screen%20Shot%202022-04-05%20at%205.47.01%20pm.png)

### automatic 和 static（指 lifetime qualifier 修饰符）

- 变量被声明为automatic，那么进入该方法后，就会自动创建，离开该方法后，就会被销毁；
而static则是在仿真开始时就会被创建，直到仿真结束，可以被多个方法/进程共享。

- 在 module, interface 和 program内的task, function 或 block声明的变量的lifetime默认是 static, 可以用 automatic 修饰符声明

### 变量的生命周期

- automatic 和 static 指 static, global/local 指 scope

- 在 module, interface, task, 和 function 外声明的变量是 static 和 global的

- tasks and functions 可以声明为 automatic, 并且内部的变量默认都是 automatic 的

- 全局变量n 只能被非连续赋值，n is static and local to my_test()

## subroutines 的生命周期

- program 块中的 subroutines 默认是 static 的，可以命名为 automatic
- class 块中的subroutines默认是 automatic

## Array 数组

- Recommended Usage Model

  - Fixed-size Arrays 定宽数组:
    - Size known at compile time
    - Contiguous data
    - Multi-dimensional arrays

  - Dynamic Arrays 动态数组:
    - Size unknown at compile time
    - Contiguous data

  - Queues 队列:
    - FIFO/Stack

  - Associative Arrays 关联数组:
    - Sparse data – memories
    - Index by number or string

![packed array](../assets/Screen%20Shot%202022-04-05%20at%2010.32.18%20pm.png)

### Array Querying System Functions

- $dimensions (array_name)
  - Returns the # of dimensions in the array
- $left (array_name, dimension)
  - Returns MSB of specified dimension
- $right (array_name, dimension)
  - Returns LSB of specified dimension
- $low (array_name, dimension)
  - Returns the min of $left and $right
- $high (array_name, dimension)
  - Returns the max of $left and $right
- $increment (array_name, dimension) 􏰁 returns 1 if: $left is >= $right
  - returns –1 if: $left is < $right.
- $size (array_name, dimension)
  - Returns the total # of elements in the specified dimension ($high - $low +1)

- 例子

```systemverilog
int c[2], a[2][2];
bit[31:0] b[0:2][0:1]={{3,7},{5,1},{0,4}};
$display($dimensions(a));
for (int i=1; i<=$dimensions(a); i++) begin
  $display("a dimension %0d size is %0d", i, $size(a, i));
  $display("a dimension %0d left is %0d", i, $left(a, i));
  $display("a dimension %0d right is %0d", i, $right(a, i));
  $display("a dimension %0d low is %0d", i, $low(a, i));
  $display("a dimension %0d high is %0d", i, $high(a, i));
  $display("a dimension %0d increment is %0d", i, signed'($increment(a, i)));
end
$display($dimensions(b));
for (int i=1; i<=$dimensions(b); i++) begin
  $display("b dimension %0d size is %0d", i, $size(b, i));
  $display("b dimension %0d left is %0d", i, $left(b, i));
  $display("b dimension %0d right is %0d", i, $right(b, i));
  $display("b dimension %0d low is %0d", i, $low(b, i));
  $display("b dimension %0d high is %0d", i, $high(b, i));
  $display("b dimension %0d increment is %0d", i, signed'($increment(b, i)));
end
```

结果：

a dimension 1 size is 2, b dimension 1 size is 3

a dimension 1 left is 0, b dimension 1 left is 0

a dimension 1 right is 1, b dimension 1 right is 2

a dimension 1 low is 0, b dimension 1 low is 0

a dimension 1 high is 1, b dimension 1 high is 2

a dimension 1 increment is -1 a dimension 2 size is 2

a dimension 2 left is 0, b dimension 1 increment is -1

a dimension 2 right is 1, b dimension 2 size is 2

a dimension 2 low is 0, b dimension 2 left is 0

a dimension 2 high is 1, b dimension 2 right is 1

a dimension 2 increment is -1, b dimension 2 increment is -1

a dimension 3 size is X, b dimension 3 size is 32

a dimension 3 left is X, b dimension 3 left is 31

a dimension 3 right is X, b dimension 3 right is 0

a dimension 3 low is X, b dimension 3 low is 0

a dimension 3 high is X, b dimension 3 high is 31

a dimension 3 increment is X, b dimension 3 increment is 1
