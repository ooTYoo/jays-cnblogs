----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式里数据差错控制技术-和校验**。  

　　在系列前一篇文章里，痞子衡给大家介绍了比较简单的校验法-[奇偶校验](http://www.cnblogs.com/henjay724/p/8465229.html)，该校验法主要是针对byte传输校验而言，而在实际应用中我们不仅要保证byte的完整性，还需要校验由多个byte组成的数据包packet的完整性。今天痞子衡继续给大家介绍针对packet校验的最简单的校验法-即和校验法。  

### 一、和校验法基本原理

#### 1.1 校验依据
　　和校验法的校验依据就是判断一次传输的n bytes组成的packet的所有byte累加和结果（仅截取低byte）在传输前后是否一致。  

#### 1.2 和校验位
　　为了实现和校验，通常会在传输的这组n bytes数据最后插入一个额外的和校验字节(byte)，用它来记录这组数据累加和的低byte结果。  

#### 1.3 校验方法
　　前面讲到和校验位实际上是n bytes数据包的累加和，那么一包数据的长度n到底怎么确定呢？为了确定n，我们通常会在一包数据开始的时候额外插入一个信息位标明当前数据包长度。  
　　在实际应用中，数据包是一包一包连续发送的，如果传输过程中发生数据丢失，则会引起数据包的错位导致接下来一连串数据包的解析错误，如何及时发现数据包错位呢？我们通常还会在数据包最开始的时候再额外插入一个信息位标明一包数据的开始，这个信息位也叫作起始标志字节。  
　　所以最终完整的数据包变成如下格式：  

<table><tbody>
    <tr>
        <td><font color="Blue">起始标志字节(1 byte)</font></td>
        <td><font color="Blue">长度字节(1-2 byte)</font></td>
        <td>原始数据位(n bytes)</td>
        <td><font color="Blue">和校验字节(1 byte)</font></td>
    </tr>
</table>

　　有了上述前导信息位，我们便可以准确找到一包数据中的原始数据位进行累加计算得出和，然后与数据包中的校验和字节进行比较验证当前包数据的正确性。  
　　需要注意的是，对于校验和字节，有时候并不一定是数据位所有字节之和结果的原码，也有可能是反码或补码（关于三者区别，请参考痞子衡另一篇文章《[整数在计算机中的表示](http://www.cnblogs.com/henjay724/p/8506398.html)》），需要结合不同校验和应用标准区别对待，否则会导致验证结果有误。  

#### 1.4 C代码实现
　　实际中校验和字节为数据之和byte结果（认定被截断的bit9为1）的补码应用较多，因为在验证数据包时，直接将所有数据连同校验和字节直接相加得到byte结果为0，即表示数据包正确。此处示例代码以补码校验和为例：  

安装包:codeblocks-17.12mingw-setup.exe
集成环境:CodeBlocks 17.12 rev 11256
编译器:GNU GCC 5.1.0
调试器:GNU gdb (GDB) 7.9.1

```C
// checksum.c
//////////////////////////////////////////////////////////
#include <stdint.h>

enum _packet_constants
{
    kPacketStartByte = 0x5a
};

#pragma pack(1)
typedef struct _packet_header
{
    uint8_t startByte;
    uint8_t length;
} packet_header_t;
#pragma pack()

/*!
 * @brief 计算数据块的checksum（补码）
 *
 * @param src， 待处理的数据块.
 * @param lenInBytes， 待处理的数据块长度.
 */
uint8_t get_checksum(uint8_t *src,
                     uint32_t lenInBytes)
{
    uint8_t checksum = 0;
    // 计算数据和，丢弃高bytes
    while (lenInBytes--)
    {
        checksum += *src++;
    }
    // 转换为补码
    checksum = (~checksum) + 1;
    return checksum;
}

/*!
 * @brief 验证数据包的checksum
 *
 * @param src， 待处理的数据包.
 * @retval 0， 数据包checksum校验正确.
 * @retval 1， 数据包起始标志字节错误.
 * @retval 2， 数据包checksum校验错误.
 */
int32_t verify_packet(uint8_t *src)
{
    uint8_t sum = 0;
    packet_header_t *header = (packet_header_t *)src;
    // 校验数据包头
    if (header->startByte != kPacketStartByte)
    {
        return 1;
    }
    // 求所有数据及校验字节之和
    for (uint32_t i = 0; i < header->length; i++)
    {
        sum += *(src + sizeof(packet_header_t) + i);
    }
    // 结果为非0，则checksum错误
    if (sum)
    {
        return 2;
    }

    return 0;
}

// main.c
//////////////////////////////////////////////////////////
#include <stdio.h>
#include <stdlib.h>
#include "checksum.h"

int main(void)
{
    uint8_t packet[16];
    packet_header_t *header = (packet_header_t *)packet;
    // 填充包头
    header->startByte = kPacketStartByte;
    header->length = sizeof(packet) - sizeof(packet_header_t);
    // 填充数据
    for (uint32_t i = sizeof(packet_header_t); i < header->length - 1; i++)
    {
        packet[i] = rand();
    }
    // 填充checksum
    packet[sizeof(packet) - 1] = get_checksum(&packet[sizeof(packet_header_t)], header->length - 1);
    // 显示packet
    for (uint32_t i = 0; i < sizeof(packet); i++)
    {
        printf("packet[%d] = 0x%x\n", i, packet[i]);
    }

    // 校验checksum
    int32_t res = verify_packet(packet);
    printf("check res = %d\n", res);

    return 0;
}
```

#### 1.5 行业应用
　　和校验由于实现简单，检错性能也算理想，因此应用十分广泛，就嵌入式而言，比较典型的应用是在各种image格式中。做过编程器或者下载器的朋友肯定会比较了解，常用的image格式有hex、s19，这些image文件都是由多个数据包组成的，在下载image文件时需要对每一包进行和校验。关于image文件详情，可参考痞子衡的文章《[ARM开发中image文件详解](http://www.cnblogs.com/henjay724/p/8361693.html)》。  

### 二、和校验法失效分析
　　在数据包传输中，如果只是单byte发生bit错误（无论多少个bit错误），和校验法一定能够识别出错误。即使有多个byte发生bit出错，大部分情况下和校验法也能正常检出。但和校验法有如下3个明显的缺陷：  

> * 当多个byte发生的bit错误发生抵消现象（引起的增量和结果是0x100的倍数），无法识别错误。
> * 当packet中byte数据顺序发生调换时，无法识别错误。
> * 不能纠错，在发现错误后，只能要求重发。

　　和校验法虽然能够校验packet，且有一定的错误bit检测能力，但其是把packet当做无序数据包来处理的，有没有其他比和校验法更好且能够校验数据次序的检错方法呢？痞子衡在下篇会继续聊。  

　　至此，嵌入式里数据差错控制技术之和校验痞子衡便介绍完毕了，掌声在哪里~~~

