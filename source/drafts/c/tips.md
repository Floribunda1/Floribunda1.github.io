这里记录一些学 c 的时候零碎的知识点

#### 为什么 `free` 需要额外释放 string? 不需要释放 int?

```c
typedef struct {
  char *name;
  int age;
  int height;
  int height;
} Person;
```

因为这里的 int 实际上是把值存到了 struct 的那段内存里，而不是存的引用，但是对于 name 来说，实际上存的是地址，所以需要额外把这里的 name 内存给释放掉

纠正：string 也不需要 free，只有 malloc 的才需要被 free

#### gcc 的一些参数(即 makefile CFLAGS)

`-Wall` 在编译过程中输出 warning 信息
`-g` 生成 debug 信息，用来给 gdb 用

#### Makefile

`CFLAGS` 是 makefile 里的 implicit rule，用来给 c compiler 传参数

一般可以在 makefile 里加上 clean 命令用来清除生成的产物

```makefile
clean:
  rm -f output
```

#### gdb 怎么用

一般我们在编译输出产物的时候带上 `-g` 表示输出 debug 信息

然后我们使用 `gdb ./output` 表示使用 gdb 打开文件

然后就可以 `run` 了

```bash
# 跑一遍程序，然后把报错信息和调用栈抛出来
gdb --batch --ex r --ex bt --ex q --args
```
