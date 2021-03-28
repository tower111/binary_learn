#  base64

[^_^]: @Rudesa，@t0wer



## base64

​	Base64编码，是我们程序开发中经常使用到的编码方法。它是一种基于用64个可打印字符来表示二进制数据的表示方法。它通常用作存储、传输一些二进制数据编码方法！

### 编码原理

  它是用64个可打印字符表示二进制所有数据方法。由于2的6次方等于64，所以可以用每6个位元为一个单元，对应某个可打印字符。我们知道三个字节有24个位元，就可以刚好对应于4个Base64单元，即3个字节需要用4个Base64的可打印字符来表示。在Base64中的可打印字符包括字母A-Z、a-z、数字0-9  ，这样共有62个字符，此外两个可打印符号在不同的系统中一般有所不同。但是，我们经常所说的Base64另外2个字符是：“+/”。这64个字符。 

这样说会不会太抽象了？不怕，我们来看一个例子：

**转换前 aaaaaabb     ccccdddd     eeffffff** 

**转换后 00aaaaaa    00bbcccc    00ddddee    00ffffff** 

 

应该很清楚了吧？上面的三个字节是原文，下面的四个字节是转换后的Base64 编码，其前两位均为0 。

 

转换后，我们用一个码表来得到我们想要的字符串（也就是最终的Base64 编码），这个表是这样的：

### Base64编码表

​	　　![Base64算法原理简介（算法实现及例子）](http://www.elecfans.com/uploads/allimg/171114/2387123-1G114111323631.png)

​	　　



让我们再来看一个实际的例子，加深印象！

 

​		**转换前 10101101   10111010    01110110** 

​		**转换后 00101011   00011011   00101001   00110110** 

​		**十进制 43 27 41 54** 

​		**对应码表中的值 r b p 2** 

​		**所以上面的24 位编码，编码后的Base64 值为 rbp2**

​		**解码同理，把 rbq2 的二进制位连接上再重组得到三个8 位值，得出原码。**

### 编码

用更接近于编程的思维来说，编码的过程是这样的：

第一个字符通过右移2 位获得第一个目标字符的Base64 表位置，根据这个数值取到表上相应的字符，就是第一个目标字符。然后将第一个字符左移2 位加上第二个字符右移4 位，即获得第二个目标字符。

再将第二个字符左移2 位加上第三个字符右移4 位，获得第三个目标字符。

最后取第三个字符的右6 位即获得第四个目标字符。

 

在以上的每一个步骤之后，再把结果与 **0x3F（0x00111111）** 进行 AND 位操作，就可以得到编码后的字符了。

 

可是等等…… 聪明的你可能会问到，原文的字节数量应该是3 的倍数啊，如果这个条件不能满足的话，那该怎么办呢？

 

我们的解决办法是这样的：原文的字节不够的地方可以用全0 来补足，转换时Base64 编码用= 号来代替。这就是为什么有些Base64 编码会以一个或两个等号结束的原因，但等号最多只有两个。因为：

 

余数 = 原文字节数 MOD 3

 

所以余数任何情况下都只可能是0 ，1 ，2 这三个数中的一个。如果余数是0 的话，就表示原文字节数正好是3 的倍数（最理想的情况啦）。如果是1 的话，为了让Base64 编码是4 的倍数，就要补2 个等号；同理，如果是2 的话，就要补1 个等号。

### base64加密加密特点

#### 编码特征

> - 字符串的长度为4的整数倍。
> - 字符串的符号取值只能在A-Z, a-z, 0-9, +, /, =共计65个字符中，且=如果出现就必须在结尾出现。（一般加密的结果会有=）

#### 算法特征

上述的编码表在ida中找到可以确定base64

![image-20210326213906621](https://gitee.com/Rudesa/image/raw/master/img/20210326213906.png)

这些移位操作也能帮助判断base64

![image-20210326214105245](https://gitee.com/Rudesa/image/raw/master/img/20210326214105.png)

### 例子

​				接下来用一个实战例子来说明base64在逆向中的使用。**（PS：例子不作为重点，只是让初学者更容易理解）**

**[WUSTCTF2020]level3**

​			拿到题目先查壳，发现无壳，可以直接拉入ida查看主函数。

```
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v3; // ST0F_1
  const char *v4; // rax
  char v6; // [rsp+10h] [rbp-40h]
  unsigned __int64 v7; // [rsp+48h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  printf("Try my base64 program?.....\n>", argv, envp);
  __isoc99_scanf("%20s", &v6);
  v3 = time(0LL);
  srand(v3);
  if ( rand() & 1 )
  {
    v4 = (const char *)base64_encode(&v6);
    puts(v4);
    puts("Is there something wrong?");
  }
  else
  {
    puts("Sorry I think it's not prepared yet....");
    puts("And I get a strange string from my program which is different from the standard base64:");
    puts("d2G0ZjLwHjS7DmOzZAY0X2lzX3CoZV9zdNOydO9vZl9yZXZlcnGlfD==");
    puts("What's wrong??");
  }
  return 0;
}
```

​	根据上面的特征总结，有个字符串`d2G0ZjLwHjS7DmOzZAY0X2lzX3CoZV9zdNOydO9vZl9yZXZlcnGlfD==`**长度为4的倍数并且出现少于三个的等于号**，那么我们可以猜测是个base64加密，因为ctf逆向题**七分猜三分做**，很多题目大部分都是猜出来的

####  加密算法的过程

然后我们去看看主函数中一个重要的函数 `sub_4007A6`，为了让初学者更容易看懂，做了很多注释有助于理解，可以看到里面有很直观的base64加密的过程，如有不懂的地方可以再去看看base64加密原理，几乎是和这个函数的作用是一模一样的

<img src="https://gitee.com/Rudesa/image/raw/master/img/20210326211813.png" alt="image-20210326211812967"  />

![image-20210326214105245](https://gitee.com/Rudesa/image/raw/master/img/20210326214105.png)

看到上面的函数，就确定了这是base64加密，于是将这个字符串拿去直接拿去解密，但是发现解不出来，回来仔细看了看函数。然后发现base解密表被改了，默认的base转换表应该是

`"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"`如下图

###### ![image-20210326213906621](https://gitee.com/Rudesa/image/raw/master/img/20210326213906.png)

但是这道题有个函数把base转换表偷偷改了一下**（如下图）**，其实就是把转换表字符串的位置发生了变化，将第十九个与第一个交换位置，第十八个与第二个交换位置……以此类推，所以得到的是转换表是`"TSRQPONMLKJIHGFEDCBAUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"`

![image-20210326183643438](https://gitee.com/Rudesa/image/raw/master/img/20210326184236.png)

有了转换表，但是不知道怎么去转换**上网搜索发现python <u>Maketrans</u>以及<u>translate()函数</u>用来转换table方便很多。**

#### Maketrans以及translate()函数解释

​	[https://blog.csdn.net/xiangshangbashaonian/article/details/81031258?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160705244719726885840654%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160705244719726885840654&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-81031258.first_rank_v2_pc_rank_v29&utm_term=Maketrans%E5%87%BD%E6%95%B0](https://blog.csdn.net/xiangshangbashaonian/article/details/81031258?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160705244719726885840654%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160705244719726885840654&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-81031258.first_rank_v2_pc_rank_v29&utm_term=Maketrans%E5%87%BD%E6%95%B0)

​	直接写脚本就可以

```
import base64
import string

str1 = "d2G0ZjLwHjS7DmOzZAY0X2lzX3CoZV9zdNOydO9vZl9yZXZlcnGlfD=="

string1 = "TSRQPONMLKJIHGFEDCBAUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
string2 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

print (base64.b64decode(str1.translate(str.maketrans(string1,string2))))

```

就可以直接出flag

wctf2020{Base64_is_the_start_of_reverse}

​							

