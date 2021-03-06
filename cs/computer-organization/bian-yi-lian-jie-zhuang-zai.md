# 编译、链接、装载

* 编译和查看汇编代码

```bash
# 使用 gcc 编译程序，输出 *.o 文件
$ gcc -g -c add_lib.c link_example.c

# 使用 objdump 查看汇编代码
$ objdump -d -M intel -S add_lib.o
$ objdump -d -M intel -S link_example.o
```

使用 `gcc -g -c addlib.c link_example.c` 编译出来的是目标文件，并不是可执行文件，目标文件通过链接器把多个目标文件以及调用的各种函数库链接起来，才能得到一个可执行文件。

```bash
# 链接多个目标文件
$ gcc -o link_example add_lib.o link_example.o
```

当我们在编写 C 语言后，然后到可执行的文件，之间经历了几个步骤：

1. 由编译、汇编、链接三个阶段，将代码转成可执行文件
2. 由装载器将可执行文件装载到内存中，CPU 从内存中读取指令和数据，来开始真正的执行

![&#x4ECE;&#x6E90;&#x4EE3;&#x7801;&#x5230;&#x7A0B;&#x5E8F;&#x6267;&#x884C;&#x7684;&#x8FC7;&#x7A0B;](../../.gitbook/assets/image%20%28124%29.png)

### Linux 下的可执行文件格式：ELF

对于可执行文件，我们可以通过 objdump 命令查看具体的内容，在 Linux 中可执行文件和目标文件使用的都是一种叫 ELF （**可执行与可链接文件格式**） 格式的文件.  在 ELF 文件里有一个符号表，它相当于一个地址簿，把名字和地址关联起来，main 函数里调用 add 的跳转地址，不再是下一条指令的地址了，而是 add 函数的入口地址。

![](../../.gitbook/assets/image%20%28122%29.png)

ELF 文件格式把各种信息，分成一个一个的 Section 保存起来。ELF 有一个基本的文件头（File Header），用来表示这个文件的基本属性，比如是否是可执行文件，对应的 CPU、操作系统等等。除了这些基本属性之外，大部分程序还有这么一些 Section：

1. 首先是 .text Section，也叫作代码段或者指令段（Code Section），用来保存程序的代码和指令；
2. 接着是 .data Section，也叫作数据段（Data Section），用来保存程序里面设置好的初始化数据信息；
3. 然后就是 .rel.text Secion，叫作重定位表（Relocation Table）。重定位表里，保留的是当前的文件里面，哪些跳转地址其实是我们不知道的。比如上面的 link\_example.o 里面，我们在 main 函数里面调用了 add 和 printf 这两个函数，但是在链接发生之前，我们并不知道该跳转到哪里，这些信息就会存储在重定位表里；
4. 最后是 .symtab Section，叫作符号表（Symbol Table）。符号表保留了我们所说的当前文件里面定义的函数名称和对应地址的地址簿。

链接器会扫描所有输入的目标文件，然后把所有符号表里的信息收集起来，构成一个全局的符号表。然后再根据重定位表，把所有不确定要跳转地址的代码，根据符号表里面存储的地址，进行一次修正。最后，把所有的目标文件的对应段进行一次合并，变成了最终的可执行代码。这也是为什么，可执行文件里面的函数调用的地址都是正确的。

![](../../.gitbook/assets/image%20%28125%29.png)

装载器不再需要考虑地址跳转的问题，只需要解析 ELF 文件，把对应的指令和数据，加载到内存里面供 CPU 执行就可以了。



### 程序装载

通过装载器可以运行 ELF 格式的文件或者 PE 格式的文件，所以也为在 Linux 下执行Windows 程序提供了可能性；因为装载器会把对应的指令和数据加载到内存里面来，让 CPU 去执行：

1. 可执行程序加载后占用的内存空间应该是连续的
2. 需要同时加载很多个程序，并且不能让程序自己规定在内存中加载的位置

要解决上边两个问题，需要在内存里面，找到一段连续的内存空间，然后分配给装载的程序，然后把这段连续的内存空间地址，和整个程序指令里指定的内存地址做一个映射。把指令里用到的内存地址叫作虚拟内存地址（Virtual Memory Address），实际在内存硬件里面的空间地址，叫物理内存地址（Physical Memory Address）。

程序里有指令和各种内存地址，我们只需要关心虚拟内存地址就行了。对于任何一个程序来说，它看到的都是同样的内存地址。我们维护一个虚拟内存到物理内存的映射表，这样实际程序指令执行的时候，会通过虚拟内存地址，找到对应的物理内存地址，然后执行。因为是连续的内存地址空间，所以我们只需要维护映射关系的起始地址和对应的空间大小就可以了。

这种找出一段连续的物理内存和虚拟内存地址进行映射的方法，我们叫分段（Segmentation）。这里的段，就是指系统分配出来的那个连续的内存空间。

![](../../.gitbook/assets/image%20%28123%29.png)

分段的方法虽然解决了程序不需要关心具体物理地址的问题，但是也产生了内存碎片问题。解决内存碎片的一个办法就是使用内存交换。**虚拟内存、分段，再加上内存交换，看起来似乎已经解决了计算机同时装载运行很多个程序的问题。不过，你千万不要大意，这三者的组合仍然会遇到一个性能瓶颈。硬盘的访问速度要比内存慢很多，而每一次内存交换，我们都需要把一大段连续的内存数据写到硬盘上。所以，如果内存交换的时候，交换的是一个很占内存空间的程序，这样整个机器都会显得卡顿**。

既然问题出在内存碎片和内存交换的空间太大上，那么解决问题的办法就是，少出现一些内存碎片，我们可以采取内存分页的方式。和分段这样分配一整段连续的空间给到程序相比，分页是把整个物理内存空间切成一段段固定尺寸的大小。而对应的程序所需要占用的虚拟内存空间，也会同样切成一段段固定尺寸的大小。这样一个连续并且尺寸固定的内存空间，我们叫页（Page）。从虚拟内存到物理内存的映射，不再是拿整段连续的内存的物理地址，而是按照一个一个页来的。页的尺寸一般远远小于整个程序的大小。在 Linux 下，我们通常只设置成 4KB。



### 动态链接

我们的内存空间是有限的，我们的很多个程序都需要通过装载器装载到内存里，而这些程序有时候会共用一些相同的代码，比如 C 标准库的代码，如果每个程序的共享代码在装载时都重新装载一遍，并且占用一部分内存，就会非常浪费内存空间，实际上是没有必要这样的。那解决这个问题的思路是引入一种新的链接代码的方法，我们称之为动态链接，这种链接方式是程序在运行时链接的已经被加载到内存的共享库，这个加载到内存的共享库会被很多个程序的指令调用到。在 Windows 下是 dll 文件，在 Linux 下是 so 文件。

此外，应用程序使用共享库，有一定的要求，就是这些机器码必须是“地址无关”的。也就是说，我们编译出来的共享库文件的指令代码，是地址无关码（Position-Independent Code）。换句话说就是，这段代码，无论加载在哪个内存地址，都能够正常执行。如果不是这样的代码，就是地址相关的代码。

对于所有动态链接共享库的程序来讲，虽然我们的共享库用的都是同一段物理内存地址，但是在不同的应用程序里，它所在的虚拟内存地址是不同的。我们没办法、也不应该要求动态链接同一个共享库的不同程序，必须把这个共享库所使用的虚拟内存地址变成一致。如果这样的话，我们写的程序就必须明确地知道内部的内存地址分配。

动态代码库内部的变量和函数调用都很容易解决，我们只需要使用相对地址（Relative Address）就好了。各种指令中使用到的内存地址，给出的不是一个绝对的地址空间，而是一个相对于当前指令偏移量的内存地址。因为整个共享库是放在一段连续的虚拟内存地址中的，无论装载到哪一段地址，不同指令之间的相对地址都是不变的。



