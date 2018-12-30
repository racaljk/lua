# lua fork for cmake

This fork removes redundant files and uses cmake to generate Makefile rather than maintaining that one manually.

# ref
https://www.lua.org/manual/5.3/

# 基于寄存器的LUA bytecode和基于栈的Java bytecode模型
本质都是一回事，不过基于栈的寄存器需要更多opcode，但是相对的基于栈的寄存器opcode更容易生成以及更接近源码形式。
Java字节码的栈帧由stack和local_variable组成，考虑下面代码：
```java
    public static void main(String[] args) {
        int a = 124;
        int b = a + 23;
        int c = b + 12;
        int d = c + 32;
        int e = d + 123;
        int f = e + 23;
        int g = f + 2;
        int h = g + 88;
        int i = 346 + h + 3;
        System.out.println(i);
    }
```
生成的字节码如下
```asm
public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 3 L0
    BIPUSH 124
    ISTORE 1
   L1
    LINENUMBER 4 L1
    ILOAD 1
    BIPUSH 23
    IADD
    ISTORE 2
   L2
    LINENUMBER 5 L2
    ILOAD 2
    BIPUSH 12
    IADD
    ISTORE 3
   L3
    LINENUMBER 6 L3
    ILOAD 3
    BIPUSH 32
    IADD
    ISTORE 4
   L4
    LINENUMBER 7 L4
    ILOAD 4
    BIPUSH 123
    IADD
    ISTORE 5
   L5
    LINENUMBER 8 L5
    ILOAD 5
    BIPUSH 23
    IADD
    ISTORE 6
   L6
    LINENUMBER 9 L6
    ILOAD 6
    ICONST_2
    IADD
    ISTORE 7
   L7
    LINENUMBER 10 L7
    ILOAD 7
    BIPUSH 88
    IADD
    ISTORE 8
   L8
    LINENUMBER 11 L8
    SIPUSH 346
    ILOAD 8
    IADD
    ICONST_3
    IADD
    ISTORE 9
   L9
    LINENUMBER 12 L9
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ILOAD 9
    INVOKEVIRTUAL java/io/PrintStream.println (I)V
   L10
    LINENUMBER 13 L10
    RETURN
   L11
    LOCALVARIABLE args [Ljava/lang/String; L0 L11 0
    LOCALVARIABLE a I L1 L11 1
    LOCALVARIABLE b I L2 L11 2
    LOCALVARIABLE c I L3 L11 3
    LOCALVARIABLE d I L4 L11 4
    LOCALVARIABLE e I L5 L11 5
    LOCALVARIABLE f I L6 L11 6
    LOCALVARIABLE g I L7 L11 7
    LOCALVARIABLE h I L8 L11 8
    LOCALVARIABLE i I L9 L11 9
    MAXSTACK = 2
    MAXLOCALS = 10
}
```
相比之下lua
```lua
local a=1
local b=3+a
local c=b+12
local d=c+34
local e=d+34
print(e)
```
生成的字节码
```asm
main <t.lua:0,0> (9 instructions at 00000000000d1e50)
0+ params, 7 slots, 1 upvalue, 5 locals, 5 constants, 0 functions
        1       [1]     LOADK           0 -1    ; 1
        2       [2]     ADD             1 -2 0  ; 3 -
        3       [3]     ADD             2 1 -3  ; - 12
        4       [4]     ADD             3 2 -4  ; - 34
        5       [5]     ADD             4 3 -4  ; - 34
        6       [6]     GETTABUP        5 0 -5  ; _ENV "print"
        7       [6]     MOVE            6 4
        8       [6]     CALL            5 2 1
        9       [6]     RETURN          0 1
```
把Java的`BIPUSH X,ISTORE X`合并为获取INDEX（隐含到字节码operand中）并直接从常量区读取；对局部变量的读取即`LOADI`只是隐含到字节码`ADD 1 -2 0`中，并没有什么改变。
因为要注意到，虚拟虚拟，只是虚拟的寄存器的概念，如果去掉这层抽象，两种都是读内存的，只是一个名字叫local_variable数组，一个是register数组。 