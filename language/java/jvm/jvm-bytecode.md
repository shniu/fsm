# JVM Bytecode

### 分析字节码的步骤：

1. 预备好字节码文件参照表，这里有定义好的一些结构，比如字面量、常量池类型和它相关的数据结构、接口相关的数据结构、类相关的等等，可以找官方的  [Java Virtual Machine Specification / The class file format](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html).
2. 编译源码得到 class 文件，用 16 进制查看器可以看到二进制内容
3. 根据字节码文件的结构定义，按顺序进行分析，不懂的就查手册

### 字节码

字节码是 JVM 虚拟机规范的一部分，它是 Java 实现一次编译到处运行的关键，它连接了 Java 源码和机器码，所以掌握字节码对于理解 JVM 虚拟机的运行机制非常有帮助。

在 JVM 的虚拟机规范中有关于字节码类文件的定义，参看 [Java Virtual Machine Specification / The class file format](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html). 字节码文件结构是一组以 8 字节为基础的二进制流，各数据严格按照顺序紧凑的排列在文件中，中间并没有特殊的分隔符，如何识别不同的组（或者说不同的部分）？使用的是长度标识的方法。下面是字节码文件结构的大致组成部分：

![&#x5B57;&#x8282;&#x7801;&#x6587;&#x4EF6;&#x7EC4;&#x6210;&#x90E8;&#x5206;](../../../.gitbook/assets/image%20%28129%29.png)

通过一段程序来分析:

```java
// Java 代码如下
package io.github.shniu.toolbox;

public class BytecodeTest {
    private static final int MAX_COUNT = 8;
    private Reader reader;

    public BytecodeTest() {
        this.reader = new BytecodeReader();
    }

    public void startup() {
        byte[] bytes = new byte[MAX_COUNT];
        reader.read(bytes);
        System.out.println(new String(bytes));
        reader.read(bytes, 1, 4);
        System.out.println(new String(bytes));
    }

    public static void main(String[] args) {
        BytecodeTest bytecodeTest = new BytecodeTest();
        bytecodeTest.startup();
        System.out.println("Bytecode finished.");
    }
}

class BytecodeReader implements Reader {
    @Override
    public void read(byte[] bytes) {
        if (bytes == null) {
            return;
        }

        for (int i = 0; i < bytes.length; i++) {
            bytes[i] = (byte) (i + 1);
        }
    }

    @Override
    public void read(byte[] bytes, int start, int length) {
        if (bytes == null) {
            return;
        }

        if (bytes.length - 1 < start) {
            throw new RuntimeException("start position must be less than bytes.length.");
        }

        for (int i = 0; i < length; i++) {
            if (i + start >= bytes.length) {
                break;
            }

            bytes[i + start] = (byte) (i + start);
        }
    }

}

interface Reader {
    void read(byte[] bytes);
    void read(byte[] bytes, int start, int length);
}
```

将上面这段代码编译后，会产生 3 个 `.class` 文件，分别是：`BytecodeTest.class`, `BytecodeReader.class`, `Reader.class`，当我们运行 `BytecodeTest.java` 中的 main 方法时，JVM 的类加载系统会把这三个字节码文件都加载到内存的方法区（这个区域在 1.8 及以后被叫做元空间 Metaspace），类加载的过程也是读取字节码文件，然后在内存中构建 Java 可使用的对象的过程。

来详细分析一下这些字节码文件，分析最复杂的 `BytecodeTest.class` 文件，用 16 进制查看器查看这个文件如下：

```text
// BytecodeTest.java
  Offset: 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 	
00000000: CA FE BA BE 00 00 00 34 00 39 0A 00 0F 00 1F 07    J~:>...4.9......
00000010: 00 20 0A 00 02 00 1F 09 00 05 00 21 07 00 22 0B    ...........!..".
00000020: 00 23 00 24 09 00 25 00 26 07 00 27 0A 00 08 00    .#.$..%.&..'....
00000030: 28 0A 00 29 00 2A 0B 00 23 00 2B 0A 00 05 00 1F    (..).*..#.+.....
00000040: 0A 00 05 00 2C 08 00 2D 07 00 2E 01 00 09 4D 41    ....,..-......MA
00000050: 58 5F 43 4F 55 4E 54 01 00 01 49 01 00 0D 43 6F    X_COUNT...I...Co
00000060: 6E 73 74 61 6E 74 56 61 6C 75 65 03 00 00 00 08    nstantValue.....
00000070: 01 00 06 72 65 61 64 65 72 01 00 20 4C 69 6F 2F    ...reader...Lio/
00000080: 67 69 74 68 75 62 2F 73 68 6E 69 75 2F 74 6F 6F    github/shniu/too
00000090: 6C 62 6F 78 2F 52 65 61 64 65 72 3B 01 00 06 3C    lbox/Reader;...<
000000a0: 69 6E 69 74 3E 01 00 03 28 29 56 01 00 04 43 6F    init>...()V...Co
000000b0: 64 65 01 00 0F 4C 69 6E 65 4E 75 6D 62 65 72 54    de...LineNumberT
000000c0: 61 62 6C 65 01 00 07 73 74 61 72 74 75 70 01 00    able...startup..
000000d0: 04 6D 61 69 6E 01 00 16 28 5B 4C 6A 61 76 61 2F    .main...([Ljava/
000000e0: 6C 61 6E 67 2F 53 74 72 69 6E 67 3B 29 56 01 00    lang/String;)V..
000000f0: 0A 53 6F 75 72 63 65 46 69 6C 65 01 00 11 42 79    .SourceFile...By
00000100: 74 65 63 6F 64 65 54 65 73 74 2E 6A 61 76 61 0C    tecodeTest.java.
00000110: 00 16 00 17 01 00 26 69 6F 2F 67 69 74 68 75 62    ......&io/github
00000120: 2F 73 68 6E 69 75 2F 74 6F 6F 6C 62 6F 78 2F 42    /shniu/toolbox/B
00000130: 79 74 65 63 6F 64 65 52 65 61 64 65 72 0C 00 14    ytecodeReader...
00000140: 00 15 01 00 24 69 6F 2F 67 69 74 68 75 62 2F 73    ....$io/github/s
00000150: 68 6E 69 75 2F 74 6F 6F 6C 62 6F 78 2F 42 79 74    hniu/toolbox/Byt
00000160: 65 63 6F 64 65 54 65 73 74 07 00 2F 0C 00 30 00    ecodeTest../..0.
00000170: 31 07 00 32 0C 00 33 00 34 01 00 10 6A 61 76 61    1..2..3.4...java
00000180: 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 0C 00 16 00    /lang/String....
00000190: 31 07 00 35 0C 00 36 00 37 0C 00 30 00 38 0C 00    1..5..6.7..0.8..
000001a0: 1A 00 17 01 00 12 42 79 74 65 63 6F 64 65 20 66    ......Bytecode.f
000001b0: 69 6E 69 73 68 65 64 2E 01 00 10 6A 61 76 61 2F    inished....java/
000001c0: 6C 61 6E 67 2F 4F 62 6A 65 63 74 01 00 1E 69 6F    lang/Object...io
000001d0: 2F 67 69 74 68 75 62 2F 73 68 6E 69 75 2F 74 6F    /github/shniu/to
000001e0: 6F 6C 62 6F 78 2F 52 65 61 64 65 72 01 00 04 72    olbox/Reader...r
000001f0: 65 61 64 01 00 05 28 5B 42 29 56 01 00 10 6A 61    ead...([B)V...ja
00000200: 76 61 2F 6C 61 6E 67 2F 53 79 73 74 65 6D 01 00    va/lang/System..
00000210: 03 6F 75 74 01 00 15 4C 6A 61 76 61 2F 69 6F 2F    .out...Ljava/io/
00000220: 50 72 69 6E 74 53 74 72 65 61 6D 3B 01 00 13 6A    PrintStream;...j
00000230: 61 76 61 2F 69 6F 2F 50 72 69 6E 74 53 74 72 65    ava/io/PrintStre
00000240: 61 6D 01 00 07 70 72 69 6E 74 6C 6E 01 00 15 28    am...println...(
00000250: 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E    Ljava/lang/Strin
00000260: 67 3B 29 56 01 00 07 28 5B 42 49 49 29 56 00 21    g;)V...([BII)V.!
00000270: 00 05 00 0F 00 00 00 02 00 1A 00 10 00 11 00 01    ................
00000280: 00 12 00 00 00 02 00 13 00 02 00 14 00 15 00 00    ................
00000290: 00 03 00 01 00 16 00 17 00 01 00 18 00 00 00 30    ...............0
000002a0: 00 03 00 01 00 00 00 10 2A B7 00 01 2A BB 00 02    ........*7..*;..
000002b0: 59 B7 00 03 B5 00 04 B1 00 00 00 01 00 19 00 00    Y7..5..1........
000002c0: 00 0E 00 03 00 00 00 0B 00 04 00 0C 00 0F 00 0D    ................
000002d0: 00 01 00 1A 00 17 00 01 00 18 00 00 00 64 00 04    .............d..
000002e0: 00 02 00 00 00 38 10 08 BC 08 4C 2A B4 00 04 2B    .....8..<.L*4..+
000002f0: B9 00 06 02 00 B2 00 07 BB 00 08 59 2B B7 00 09    9....2..;..Y+7..
00000300: B6 00 0A 2A B4 00 04 2B 04 07 B9 00 0B 04 00 B2    6..*4..+..9....2
00000310: 00 07 BB 00 08 59 2B B7 00 09 B6 00 0A B1 00 00    ..;..Y+7..6..1..
00000320: 00 01 00 19 00 00 00 1A 00 06 00 00 00 10 00 05    ................
00000330: 00 11 00 0F 00 12 00 1D 00 13 00 29 00 14 00 37    ...........)...7
00000340: 00 15 00 09 00 1B 00 1C 00 01 00 18 00 00 00 39    ...............9
00000350: 00 02 00 02 00 00 00 15 BB 00 05 59 B7 00 0C 4C    ........;..Y7..L
00000360: 2B B6 00 0D B2 00 07 12 0E B6 00 0A B1 00 00 00    +6..2....6..1...
00000370: 01 00 19 00 00 00 12 00 04 00 00 00 18 00 08 00    ................
00000380: 19 00 0C 00 1A 00 14 00 1B 00 01 00 1D 00 00 00    ................
00000390: 02 00 1E   
```

上面有一张字节码文件的大致组成部分的图，依次来分析

* 最开始的前 4 个字节，是魔数，用来快速判断这是不是一个 Java 的字节码文件，如果不是直接退出或者报错；`CAFEBABE` 这 4 个字节是 Java 的魔数
* 紧接着的 4 个字节 `0000 0034` , 前两个字节是此版本号，后两个字节是主版本号，`0034` 表示是 JDK 1.8
* 接着就是常量池的部分，由常量池个数和常量池表组成
  * 第 9～10 这 2 字节表示常量池中常量的个数，这里是 `0036` ，10 进制是 54，那么常量池表中的数据项就是 53 个，为啥需要 -1，因为常量池中的第0个位置被我们的jvm占用了表示为null 所以我们通过编译出来的常量池索引是从1开始的
  * 紧接着后面的就是常量池表了，逐项分析，在这个常量池表中的每一项都以1个字节长度的 tag 来标记常量的类型
  * 常量池表中的第 1 项，`0A` 表示 `CONSTANT_Methodref_info`, 这种类型的常量后面有 2 字节的类索引 \(`CONSTANT_Class_info`\)和 2 字节的名称及类型描述符\(`CONSTANT_NameAndType_info` \); 所以 `00 0E` 是 14，表示指向常量池中位置是 14 的常量； `00 1E` 是 30, 表示指向常量池中位置是 30 的常量，而且这个常量的类型是 `NameAndType_info` 
  * 第二个常量 07 00 1F, 07 表示 `CONSTANT_Class_info` , 接着的 2 个字节指向权限定类名，指向索引 31，索引 31 处其实是 `31 = Utf8               io/github/shniu/toolbox/BytecodeReader` , 它是一个 Utf8 的字面量
  * 第三个常量 0A 00 02 00 1E，表示是一个 `CONSTANT_Methodref_info` , 分别指向索引 2 和 30，也就是指向 `BytecodeReader` 的无参构造方法
  * 以此类推进行分析即可
* 常量池后紧接着是访问标志符号，2 个字节，我们的文件里是 `00 21` ，表示是 0x0021 是通过位运算 & 计算出来的，这个 class 的访问权限是 `ACC_PUBLIC` _和 `ACC_SUPER`_

![](../../../.gitbook/assets/image%20%28134%29.png)

* 紧接着是 this class name, 表示当前所属类，2 个字节，值为 `00 05` ，表示索引指向为 5 的常量池中的元素，\#5 位置是一个 class，指向了 `BytecodeTest` 
* 再接着是 super class name, 表示当前类继承的父类名称，占用 2 个字节，值为 `00 0F` ，表示指向 \#15，而这个位置是 Object 类，也就是当前类的父类是 Object
* 再接着是接口的相关信息，2 个字节表示接口数量，后面跟着的是接口表；接口数量是 `00 00` ，说明当前类没有实现接口；如果当前类实现了接口，比如代码中的 `BytecodeReader` 类实现了一个接口 `Reader` ，那么这里的值就是 `00 01` ，紧接着的 2 个字节就是该接口指向的常量池中的位置，为 `00 07` ，\#7 指向的是 `Class_info` 
* 接口信息之后是字段表的信息，它表示的是类变量或者实例变量，但不包括方法的局部变量，接着的 2 个字节是字段的个数，值为 `00 02` ；
  * 每个字段表按如下方式组成
  * \`00 1A 00 10 00 11 00 01 00 12 00 00 

![&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x636E;&#x7ED3;&#x6784;](../../../.gitbook/assets/image%20%28132%29.png)

![](../../../.gitbook/assets/image%20%28130%29.png)

* 方法表

```text
00 03 
# 这是第一个方法的信息
      00 01 00 16 00 17 00 01 00 18 00 00 00 30   
00 03 00 01 00 00 00 10 2A B7 00 01 2A BB 00 02    
59 B7 00 03 B5 00 04 B1 00 00 00 01 00 19 00 00    
00 0E 00 03 00 00 00 0B 00 04 00 0C 00 0F 00 0D
# 这是第二个方法的信息
00 01 00 1A 00 17 00 01 00 18 00 00 00 64 00 04    
00 02 00 00 00 38 10 08 BC 08 4C 2A B4 00 04 2B    
B9 00 06 02 00 B2 00 07 BB 00 08 59 2B B7 00 09    
B6 00 0A 2A B4 00 04 2B 04 07 B9 00 0B 04 00 B2    
00 07 BB 00 08 59 2B B7 00 09 B6 00 0A B1 00 00    
00 01 00 19 00 00 00 1A 00 06 00 00 00 10 00 05    
00 11 00 0F 00 12 00 1D 00 13 00 29 00 14 00 37    
00 15
# 这是第三个方法的信息
      00 09 00 1B 00 1C 00 01 00 18 00 00 00 39    
00 02 00 02 00 00 00 15 BB 00 05 59 B7 00 0C 4C    ........;..Y7..L
2B B6 00 0D B2 00 07 12 0E B6 00 0A B1 00 00 00    +6..2....6..1...
01 00 19 00 00 00 12 00 04 00 00 00 18 00 08 00    ................
19 00 0C 00 1A 00 14 00 1B
```

`00 03` 表示有 3 个方法，在我们的类中是 无参构造方法、startup 方法、main 方法



* 最后是 .class 文件属性: `00 01 00 1D 00 00 00 02 00 1E` , 表示有 1\(00 01\) 个属性，属性指向的位置是 00 1D，也就是 \#29，属性的长度是 00  00 00 02 也就是长度为 2，00 1E 就是属性的值，指向 \#30

![&#x6587;&#x4EF6;&#x5C5E;&#x6027;&#x7ED3;&#x6784;](../../../.gitbook/assets/image%20%28128%29.png)

字节码文件内容如下

```text
// 字节码文件内容
// BytecodeReader.java
  Offset: 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 	
00000000: CA FE BA BE 00 00 00 34 00 1A 0A 00 06 00 12 07    J~:>...4........
00000010: 00 13 08 00 14 0A 00 02 00 15 07 00 16 07 00 17    ................
00000020: 07 00 18 01 00 06 3C 69 6E 69 74 3E 01 00 03 28    ......<init>...(
00000030: 29 56 01 00 04 43 6F 64 65 01 00 0F 4C 69 6E 65    )V...Code...Line
00000040: 4E 75 6D 62 65 72 54 61 62 6C 65 01 00 04 72 65    NumberTable...re
00000050: 61 64 01 00 05 28 5B 42 29 56 01 00 0D 53 74 61    ad...([B)V...Sta
00000060: 63 6B 4D 61 70 54 61 62 6C 65 01 00 07 28 5B 42    ckMapTable...([B
00000070: 49 49 29 56 01 00 0A 53 6F 75 72 63 65 46 69 6C    II)V...SourceFil
00000080: 65 01 00 11 42 79 74 65 63 6F 64 65 54 65 73 74    e...BytecodeTest
00000090: 2E 6A 61 76 61 0C 00 08 00 09 01 00 1A 6A 61 76    .java........jav
000000a0: 61 2F 6C 61 6E 67 2F 52 75 6E 74 69 6D 65 45 78    a/lang/RuntimeEx
000000b0: 63 65 70 74 69 6F 6E 01 00 2E 73 74 61 72 74 20    ception...start.
000000c0: 70 6F 73 69 74 69 6F 6E 20 6D 75 73 74 20 62 65    position.must.be
000000d0: 20 6C 65 73 73 20 74 68 61 6E 20 62 79 74 65 73    .less.than.bytes
000000e0: 2E 6C 65 6E 67 74 68 2E 0C 00 08 00 19 01 00 26    .length........&
000000f0: 69 6F 2F 67 69 74 68 75 62 2F 73 68 6E 69 75 2F    io/github/shniu/
00000100: 74 6F 6F 6C 62 6F 78 2F 42 79 74 65 63 6F 64 65    toolbox/Bytecode
00000110: 52 65 61 64 65 72 01 00 10 6A 61 76 61 2F 6C 61    Reader...java/la
00000120: 6E 67 2F 4F 62 6A 65 63 74 01 00 1E 69 6F 2F 67    ng/Object...io/g
00000130: 69 74 68 75 62 2F 73 68 6E 69 75 2F 74 6F 6F 6C    ithub/shniu/tool
00000140: 62 6F 78 2F 52 65 61 64 65 72 01 00 15 28 4C 6A    box/Reader...(Lj
00000150: 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B    ava/lang/String;
00000160: 29 56 00 20 00 05 00 06 00 01 00 07 00 00 00 03    )V..............
00000170: 00 00 00 08 00 09 00 01 00 0A 00 00 00 1D 00 01    ................
00000180: 00 01 00 00 00 05 2A B7 00 01 B1 00 00 00 01 00    ......*7..1.....
00000190: 0B 00 00 00 06 00 01 00 00 00 1E 00 01 00 0C 00    ................
000001a0: 0D 00 01 00 0A 00 00 00 57 00 04 00 03 00 00 00    ........W.......
000001b0: 1B 2B C7 00 04 B1 03 3D 1C 2B BE A2 00 10 2B 1C    .+G..1.=.+>"..+.
000001c0: 1C 04 60 91 54 84 02 01 A7 FF F0 B1 00 00 00 02    ..`.T...'.p1....
000001d0: 00 0B 00 00 00 1A 00 06 00 00 00 21 00 04 00 22    ...........!..."
000001e0: 00 05 00 25 00 0D 00 26 00 14 00 25 00 1A 00 28    ...%...&...%...(
000001f0: 00 0E 00 00 00 0A 00 03 05 FC 00 01 01 FA 00 12    .........|...z..
00000200: 00 01 00 0C 00 0F 00 01 00 0A 00 00 00 8C 00 04    ................
00000210: 00 05 00 00 00 3E 2B C7 00 04 B1 2B BE 04 64 1C    .....>+G..1+>.d.
00000220: A2 00 0D BB 00 02 59 12 03 B7 00 04 BF 03 36 04    "..;..Y..7..?.6.
00000230: 15 04 1D A2 00 20 15 04 1C 60 2B BE A1 00 06 A7    ...".....`+>!..'
00000240: 00 14 2B 15 04 1C 60 15 04 1C 60 91 54 84 04 01    ..+...`...`.T...
00000250: A7 FF E0 B1 00 00 00 02 00 0B 00 00 00 2A 00 0A    '.`1.........*..
00000260: 00 00 00 2C 00 04 00 2D 00 05 00 30 00 0D 00 31    ...,...-...0...1
00000270: 00 17 00 34 00 20 00 35 00 29 00 36 00 2C 00 39    ...4...5.).6.,.9
00000280: 00 37 00 34 00 3D 00 3B 00 0E 00 00 00 0C 00 05    .7.4.=.;........
00000290: 05 11 FC 00 02 01 11 FA 00 10 00 01 00 10 00 00    ..|....z........
000002a0: 00 02 00 11                                        ....

// Reader.java
  Offset: 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 	
00000000: CA FE BA BE 00 00 00 34 00 0A 07 00 08 07 00 09    J~:>...4........
00000010: 01 00 04 72 65 61 64 01 00 05 28 5B 42 29 56 01    ...read...([B)V.
00000020: 00 07 28 5B 42 49 49 29 56 01 00 0A 53 6F 75 72    ..([BII)V...Sour
00000030: 63 65 46 69 6C 65 01 00 11 42 79 74 65 63 6F 64    ceFile...Bytecod
00000040: 65 54 65 73 74 2E 6A 61 76 61 01 00 1E 69 6F 2F    eTest.java...io/
00000050: 67 69 74 68 75 62 2F 73 68 6E 69 75 2F 74 6F 6F    github/shniu/too
00000060: 6C 62 6F 78 2F 52 65 61 64 65 72 01 00 10 6A 61    lbox/Reader...ja
00000070: 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74 06 00    va/lang/Object..
00000080: 00 01 00 02 00 00 00 00 00 02 04 01 00 03 00 04    ................
00000090: 00 00 04 01 00 03 00 05 00 00 00 01 00 06 00 00    ................
000000a0: 00 02 00 07                                        ....

```



#### **LineNumberTable属性**

LineNumberTable是Code属性中的一个子属性，用来描述java源文件行号与字节码文件偏移量之间的对应关系。当程序运行抛出异常时，异常堆栈中显示出错的行号就是根据这个对应关系来显示的

### 总结

![&#x5B57;&#x8282;&#x7801;&#x7EC4;&#x7EC7;&#x7ED3;&#x6784;](../../../.gitbook/assets/image%20%28133%29.png)

![&#x5E38;&#x91CF;&#x6C60;&#x7C7B;&#x578B;&#x5206;&#x7C7B;](../../../.gitbook/assets/image%20%28131%29.png)

### Reference

* [JVM 字节码指令表](https://segmentfault.com/a/1190000008722128)
* 字节码文件结构：[https://www.cnblogs.com/chanshuyi/p/jvm\_serial\_05\_jvm\_bytecode\_analysis.html](https://www.cnblogs.com/chanshuyi/p/jvm_serial_05_jvm_bytecode_analysis.html)
* [Java Virtual Machine Specification / The class file format](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)


