----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式里数据差错控制技术-奇偶校验**。  

　　在系列第一篇文章里，痞子衡给大家介绍了最简单的校验法-[重复校验](http://www.cnblogs.com/henjay724/p/8457391.html)，该校验法实现简单，检错纠错能力都还不错，但传输效率实在是不高，在效率至上的大背景下，这种方法是不能容忍的。今天痞子衡继续给大家介绍另一种也非常简单但效率较高的校验法-即奇偶校验法。  

### 一、奇偶校验法基本原理

#### 1.1 校验依据
　　奇偶校验法的校验依据就是判断一次传输的一组二进制数据中bit "1"的奇偶性(奇数个还是偶数个)在传输前后是否一致，所以其实奇偶检验法有两个子类：  
> * 奇校验：如果以二进制数据中1的个数是奇数为依据，则是奇校验
> * 偶校验：如果以二进制数据中1的个数是偶数为依据，则是偶校验

　　一般在同步传输方式中常采用奇校验，而在异步传输方式中常采用偶校验。

#### 1.2 奇偶校验位
　　为了实现奇偶校验，通常会在传输的这组二进制数据中插入一个额外的奇偶校验位(bit)，用它来确保发送出去的这组二进制数据中“1”的个数为奇数或偶数。  
　　划重点，奇偶校验位并不是用来标记原始传输数据中1的个数是奇数还是偶数，而是用来确保原始数据加上奇偶校验位后的合成数据中1的个数是奇数或者偶数。  

#### 1.3 校验方法
　　常用的奇偶校验共有三种：水平奇偶校验，垂直奇偶校验校验和水平垂直奇偶校验。以对32位数据：10100101 10111001 10000100 00011010进行校验为例讲解：  
> * 水平奇偶校验：对每一种数据的编码添加校验位，使信息位与校验位处于同一行。

<table><tbody>
    <tr>
        <th>原始数据</th>
        <th>水平奇校验位</th>
        <th>水平偶校验位</th>
    </tr>
    <tr>
        <td>10100101</td>
        <td><font color="Green">1</font></td>
        <td><font color="Blue">0</font></td>
    </tr>
    <tr>
        <td>10111001</td>
        <td><font color="Green">0</font></td>
        <td><font color="Blue">1</font></td>
    </tr>
    <tr>
        <td>10000100</td>
        <td><font color="Green">1</font></td>
        <td><font color="Blue">0</font></td>
    </tr>
    <tr>
        <td>00011010</td>
        <td><font color="Green">0</font></td>
        <td><font color="Blue">1</font></td>
    </tr>
</table>

　　所以加上水平偶校验位后应传输的数据是：10100101<font color="Blue" size=4>0</font> 10111001<font color="Blue" size=4>1</font> 10000100<font color="Blue" size=4>0</font> 00011010<font color="Blue" size=4>1</font>  

> * 垂直奇偶校验：将数据分为若干组，一组一行，再加上一行校验位，针对每一列采样奇校验或偶校验。

<table><tbody>
    <tr>
        <th>编码分类</th>
        <th>垂直奇校验</th>
        <th>垂直偶校验</th>
    </tr>
    <tr>
        <td rowspan="4">原始数据</td>
        <td>10100101</td>
        <td>10100101</td>
    </tr>
    <tr>
        <td>10111001</td>
        <td>10111001</td>
    </tr>
    <tr>
        <td>10000100</td>
        <td>10000100</td>
    </tr>
    <tr>
        <td>00011010</td>
        <td>00011010</td>
    </tr>
    <tr>
        <td>校验位</td>
        <td><font color="Green">01111101</font></td>
        <td><font color="Blue">10000010</font></td>
    </tr>
</table>

　　所以加上垂直偶校验位后应传输的数据是：10100101 10111001 10000100 00011010<font color="Blue" size=4>10000010</font>  

> * 水平垂直奇偶校验：也叫Hamming Code，其是在水平和垂直方向上进行双校验，其不仅可以检测2bit错误的具体位置，还可纠正1bit错误，常用于NAND Flash里。这部分不属于本文要讨论的内容，痞子衡后续会专门介绍Hamming Code。

#### 1.4 C代码实现
　　实际中水平校验法应用比较多，此处示例代码以水平奇校验为例：  

安装包:codeblocks-17.12mingw-setup.exe
集成环境:CodeBlocks 17.12 rev 11256
编译器:GNU GCC 5.1.0
调试器:GNU gdb (GDB) 7.9.1

```C
// parity_check.c
//////////////////////////////////////////////////////////
#include <stdbool.h>
#include <stdint.h>

/*!
 * @brief 判断当前byte的极性是否为奇
 *
 * @param byte， 待计算奇偶性的数据.
 * @retval ture， byte极性(含1的个数)为奇数.
 * @retval false， byte极性(含1的个数)为偶数.
 */
bool is_byte_odd_parity(uint8_t byte)
{
    bool parity = false;
    // 普通算法-byte逐位异或（需循环8次）
    /*
    for (uint8_t i = 0; i < 8; i++)
    {
        parity ^= byte & 0x01u;
        byte >>= 1;
    }
    */
    // 效率较高算法-计数byte中1的个数（需循环n次，n为byte中1的个数）
    while (byte)
    {
        parity = !parity;
        byte &= byte - 1;
    }
    return parity;
}

/*!
 * @brief 获取给定data的水平奇校验位
 *
 * @param src， 待计算奇偶性的数据块.
 * @param lenInBytes， 待计算奇偶性的数据块长度.
 * @retval 0， data极性(含1的个数)为奇数.
 * @retval 1， data极性(含1的个数)为偶数.
 */
uint32_t get_data_parity(uint8_t *src,
                         uint32_t lenInBytes)
{
    uint32_t result = 0;
    // 水平校验法
    // isDataOddParity用于判断所有data bits的行极性是否为奇
    bool isDataOddParity = false;
    while (lenInBytes--)
    {
        isDataOddParity ^= is_byte_odd_parity(*src++);
    }
    // result为所有data bits的奇校验位
    result = !isDataOddParity;

    return result;
}

// main.c
//////////////////////////////////////////////////////////
#include <stdio.h>
#include <stdlib.h>
#include "parity_check.h"

int main(void)
{
    uint8_t data[4] = {0x31, 0x33, 0x04, 0x08};
    uint32_t parity = get_data_parity(data, sizeof(data));

    printf("parity = %d\n", parity);
    return 0;
}
```

#### 1.5 行业应用
　　奇偶检验比较典型的应用是在串口UART上，玩过UART的朋友肯定了解串口奇偶检验位的作用，包括下位机MCU UART驱动的编写，上位机串口调试助手的设置都需要注意奇偶校验位。下图是UART传输时序图，奇偶校验位是可选位，仅当使能时才会生效。不过作为嵌入式开发者，倒不必关注奇偶校验的具体实现，因为MCU的UART模块已经在硬件上支持了奇偶检验，我们只需要操作UART对应寄存器的控制位去使能奇偶检验功能即可。  

![](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/UART-Timing.PNG)

### 二、奇偶校验法失效分析
　　在现实数据传输中，偶尔1位出错的机会最多，2位及以上发生错误的概率比较低，且由于奇偶校验实现简单，具有相对理想的检错能力，因此得到广泛使用。但奇偶校验法有如下2个明显的缺陷：  

> * 奇数位误码能检出，偶数位误码不能检出
> * 不能纠错，在发现错误后，只能要求重发。

　　前面讲的两种校验法实际上更多是针对byte传输校验，而在实际应用中我们校验的对象往往是数据包packet，有没有其他比奇偶校验法更好且针对packet的检错方法呢？痞子衡在下篇会继续聊。  

　　至此，嵌入式里数据差错控制技术之奇偶校验痞子衡便介绍完毕了，掌声在哪里~~~

