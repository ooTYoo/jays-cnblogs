----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式里数据差错控制技术-重复校验**。  

　　在嵌入式应用里，除了最核心的数据处理外，我们还会经常和数据传输打交道。数据传输需要硬件传输接口的支持，串行接口由于占用引脚少的优点目前应用比并行接口广泛，常用的串行接口种类非常多，比如UART,SPI,I2C,USB等，在使用这些接口传输数据时避不可免会遇到一个问题，如果传输过程中遇到未知硬件干扰发生bit错误怎么办？  

　　痞子衡今天给大家讲的就是数据传输过程中用于差错检测的最简单的方法，即重复校验法。  

### 一、重复校验法基本原理

#### 1.1 校验依据
　　重复校验法的校验依据就是判断重复传输的q组n bits二进制数据是否一致。

#### 1.2 重复校验位
　　为了实现重复校验，就是不断重复传输这组n bits原始数据q次即可，一次校验的q\*n bits数据块中，仅有n bits数据是原始有效数据，校验位就是那些重复的(q-1)*n bits数据。是不是觉得简单又粗暴？

#### 1.3 校验方法
　　假设原始数据块是X[n-1:0]共n bits，重复次数为q（q一般为奇数），按重复传输方式，可分为两个子类：  
> * 按bit重复：发送数据序列为，q个X<sub>0</sub>（X<sub>0</sub>X<sub>0</sub>...），q个X<sub>1</sub>（X<sub>1</sub>X<sub>1</sub>...)...，q个X<sub>n-1</sub>（X<sub>n-1</sub>X<sub>n-1</sub>...)  
> * 按block重复：发送数据序列为，第1个X[n-1:0]，第2个X[n-1:0]...，第q个X[n-1:0]。

　　接受端收到数据后，逐次比较q个重复位，如完全一致，则认为没有错差；如不一致，则存在错误bit。如需纠错的话，原理也很简单，判断q个重复位里哪种数据位出现的次数多（这里解释了q为何应是奇数）则为原始正确数据位。  

#### 1.4 C代码实现
　　实际中按block重复校验法应用比较多，此处示例代码以此为例：  

安装包:codeblocks-17.12mingw-setup.exe
集成环境:CodeBlocks 17.12 rev 11256
编译器:GNU GCC 5.1.0
调试器:GNU gdb (GDB) 7.9.1

```C
// repetition_code.c
//////////////////////////////////////////////////////////
#include <stdint.h>
#include <assert.h>

/*!
 * @brief 处理按block重复的数据块
 *
 * @param src， 待处理的数据块.
 * @param dest， 处理完成的原始数据.
 * @param lenInBytes， 待处理的数据块长度.
 * @param repeatTimes， 数据重复次数（假定为奇数）.
 * @retval 0， 数据无错误位.
 * @retval 1， 数据有错误位且已纠正.
 */
uint32_t verify_correct_repetition_block(uint8_t *src,
                                         uint8_t *dest,
                                         uint32_t lenInBytes,
                                         uint32_t repeatTimes)
{
    assert(repeatTimes % 2);
    assert(!(lenInBytes % repeatTimes));

    uint32_t result = 0;
    uint32_t blockBytes = lenInBytes / repeatTimes;

    // 遍历一个block长度里每个byte
    for (uint32_t i = 0; i < blockBytes; i++)
    {
        // 遍历当前byte的每个bit
        uint8_t correctByte = 0;
        for (uint32_t j = 0; j < 8; j++)
        {
            // 遍历当前byte的所有重复byte
            uint32_t bit1Count = 0;
            for (uint32_t k = 0; k < repeatTimes; k++)
            {
                // 记录所有重复byte中当前bit为1的个数
                uint8_t countByte = *(src + i + k * blockBytes);
                bit1Count += (countByte & (0x1u << j)) >> j;
            }
            // 当bit1出现半数则将当前bit认定为1
            if (bit1Count > (repeatTimes / 2))
            {
                correctByte |= 0x1u << j;
            }
            // 首次发现错误bit时，置位result
            if ((!result) && (bit1Count !=0) && bit1Count != repeatTimes)
            {
                result = 1;
            }
        }
        // 将校验后的byte存入dest
        *(dest + i) = correctByte;
    }

    return result;
}

// main.c
//////////////////////////////////////////////////////////
#include "repetition_code.h"
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    uint8_t src[3][4] = {{0x32, 0x33, 0x04, 0x08},
                         {0x32, 0x83, 0x04, 0xd8},
                         {0x31, 0x33, 0x04, 0xe8}};
    uint8_t dest[4];
    uint32_t result = verify_correct_repetition_block((uint8_t *)src, dest, sizeof(src), sizeof(src) / sizeof(src[0]));

    printf("result = %d\n", result);
    for (uint32_t i = 0; i < sizeof(dest); i++)
    {
        printf("dest[%d] = 0x%x\n", i, dest[i]);
    }
    return 0;
}
```

#### 1.5 行业应用
　　实际上本文所讲的单纯的重复校验法行业因为效率的原因，行业里较少应用，其改进版的实现RA Codes应用在了FlexRay协议里。  

### 二、重复校验法失效分析
　　重复校验实现非常简单，具有比较理想的检错能力，但效率太低，并未得到广泛使用。即便牺牲了效率，但重复校验法也存在如下2个缺陷，导致其检错纠错并不可靠：  

> * 当重复bit全部发生错误时，会被误认为没有错误bit发生。
> * 当错误bit出现概率大于原始bit时，在纠错时会认定错误bit是原始bit。

　　有没有其他比重复校验法更高效的检错方法？痞子衡在下篇会继续聊。  

　　至此，嵌入式里数据差错控制技术之重复校验痞子衡便介绍完毕了，掌声在哪里~~~

