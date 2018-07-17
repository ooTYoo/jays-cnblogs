----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔Kinetis系列MCU的KBOOT架构**。  

　　Bootloader是嵌入式MCU开发里很常见的一种专用的应用程序，在一个没有Bootloader的嵌入式系统里如果要更新Application，只能通过外部硬件调试器/下载器，而如果有了Bootloader，我们可以轻松完成Application的更新升级以及加载启动，除此以外在Bootloader中还可以引入更多高级特性，比如Application完整性检测、可靠升级、加密特性等。  
　　KBOOT是设计运行于Kinetis芯片上的一种Bootloader，KBOOT由飞思卡尔（现恩智浦）官方推出，其功能非常全面，今天痞子衡就为你揭开KBOOT的神秘面纱：  

### 一、KBOOT由来
　　飞思卡尔Kinetis系列MCU是从2010年开始推出的，早期的Kinetis产品比如MK60, MKL25并没有配套标准Bootloader功能，不过可以从飞思卡尔官网上找到很多风格迥异的Bootloader参考设计，比如AN2295（UART型）、AN4655（I2C型）、AN4379（USB-MSD型）等，这些Bootloader参考方案都是不同的飞思卡尔应用工程师设计的，因此所用的通信协议以及上位机工具都不相同，虽然这些AN一定程度上能解决客户使用Bootloader的需求，但是在Bootloader后续维护升级以及拓展性方面有一定缺陷。  
　　飞思卡尔也逐渐意识到了这一点，为了完善软件生态建设与服务质量，于是在2013年初组建了一支专门开发Kinetis Bootloader的软件团队，即KBOOT Team，这个Team成立的目的就是要开发出一个Unified Kinetis Bootloader（简称KBOOT），这个bootloader必须拥有良好的架构，易于扩展和维护，功能全面且经过完善的验证。  
　　KBOOT项目发展至今（2017）已近5年，目前被广泛应用于主流Kinetis芯片上，是Kinetis芯片集成Bootloader的首选，其官方主页是 [www.nxp.com/kboot](www.nxp.com/kboot)  

### 二、KBOOT架构
　　从架构角度来分析KBOOT，抛开各种附加特性，其实KBOOT最核心的就是这三大组件：Peripheral Interface、Command & Data Processor、Memory Interface，如下图所示：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_arch.PNG" style="zoom:100%" />

#### 2.1 Peripheral Interface
　　KBOOT首要功能是能够与Host进行数据传输，我们知道数据传输接口种类有很多，KBOOT设计上可同时支持多种常见传输接口（UART, SPI, I2C, USB-HID, CAN），为此KBOOT在Peripheral Interface组件中抽象了Peripheral的行为（byte/packet层传输等），使得在Peripheral种类拓展上更容易。  
　　KBOOT中使用一个名叫g_peripherals[]的结构体数组来集合所有外设，下面示例仅包含UART, USB：  
```C
//! @brief Peripheral array.
const peripheral_descriptor_t g_peripherals[] = {
#if BL_CONFIG_LPUART_0
    // LPUART0
    {.typeMask = kPeripheralType_UART,
     .instance = 0,
     .pinmuxConfig = uart_pinmux_config,
     .controlInterface = &g_lpuartControlInterface,
     .byteInterface = &g_lpuartByteInterface,
     .packetInterface = &g_framingPacketInterface },
#endif // BL_CONFIG_LPUART_0

#if BL_CONFIG_USB_HID
    // USB HID
    {.typeMask = kPeripheralType_USB_HID,
     .instance = 0,
     .pinmuxConfig = NULL,
     .controlInterface = &g_usbHidControlInterface,
     .byteInterface = NULL,
     .packetInterface = &g_usbHidPacketInterface },
#endif    // BL_CONFIG_USB_HID

    { 0 } // Terminator
};
```

　　如下便是用于抽象外设行为的Peripheral descriptor原型，该原型可以描述所有类型的peripheral：  
```C
//! @brief Peripheral descriptor.
//!
//! Instances of this struct describe a particular instance of a peripheral that is
//! available for bootloading.
typedef struct PeripheralDescriptor
{
    //! @brief Bit mask identifying the peripheral type.
    //! See #_peripheral_types for a list of valid bits.
    // 外设的类型名，KBOOT用于识别当前外设的类型
    uint32_t typeMask;

    //! @brief The instance number of the peripheral.
    // 外设的编号，KBOOT可以支持同一外设的多个实例
    uint32_t instance;

    //! @brief Configure pinmux setting for the peripheral.
    // 外设的I/O初始化
    void (*pinmuxConfig)(uint32_t instance, pinmux_type_t pinmux);

    //! @brief Control interface for the peripheral.
    // 外设的行为控制
    const peripheral_control_interface_t *controlInterface;

    //! @brief Byte-level interface for the peripheral.
    //! May be NULL since not all periperhals support this interface.
    // 外设的byte级别传输控制
    const peripheral_byte_inteface_t *byteInterface;

    //! @brief Packet level interface for the peripheral.
    // 外设的packet级别传输控制
    const peripheral_packet_interface_t *packetInterface;
} peripheral_descriptor_t;

//! @brief Peripheral control interface.
typedef struct _peripheral_control_interface
{
    // 检测是否外设是否被激活
    bool (*pollForActivity)(const peripheral_descriptor_t *self);
    // 外设IP底层初始化
    status_t (*init)(const peripheral_descriptor_t *self, serial_byte_receive_func_t function);
    // 外设IP底层恢复
    void (*shutdown)(const peripheral_descriptor_t *self);
    // 特殊外设pump控制（比如USB-MSC, DFU等）
    void (*pump)(const peripheral_descriptor_t *self);
} peripheral_control_interface_t;

//! @brief Peripheral abstract byte interface.
typedef struct _peripheral_byte_inteface
{
    // byte传输初始化，一般为NULL
    status_t (*init)(const peripheral_descriptor_t *self);
    // byte发送
    status_t (*write)(const peripheral_descriptor_t *self, const uint8_t *buffer, uint32_t byteCount);
} peripheral_byte_inteface_t;

//! @brief Peripheral Packet Interface.
typedef struct _peripheral_packet_interface
{
    // packet传输初始化
    status_t (*init)(const peripheral_descriptor_t *self);
    // 接收一包packet
    status_t (*readPacket)(const peripheral_descriptor_t *self, uint8_t **packet, uint32_t *packetLength, packet_type_t packetType);
    // 发送一包packet
    status_t (*writePacket)(const peripheral_descriptor_t *self, const uint8_t *packet, uint32_t byteCount, packet_type_t packetType);
    // 立即终止当前packet
    void (*abortDataPhase)(const peripheral_descriptor_t *self);
    // 完成当前packet
    status_t (*finalize)(const peripheral_descriptor_t *self);
    // 获取最大packet包长
    uint32_t (*getMaxPacketSize)(const peripheral_descriptor_t *self);
    // byte接收callback
    void (*byteReceivedCallback)(uint8_t byte);
} peripheral_packet_interface_t;
```

#### 2.2 Memory Interface
　　KBOOT其次功能是能够读写存储空间，Kinetis上涉及的存储空间包括内部SRAM, Flash，Register、I/O以及外部QuadSPI NOR Flash（可以映射在MCU内部存储空间），为此KBOOT在Memory Interface组件中抽象了Memory的行为（read/write/erase等），使得在Memory种类拓展上更容易。  
　　KBOOT中使用一个名叫g_memoryMap[]的结构体数组来集合所有存储空间，下面示例包含了典型的存储空间（Flash、RAM、Register、I/O、QSPI NOR Flash）：  
```C
//! @brief Memory map.
//!
//! This map is not const because it is updated at runtime with the actual sizes of
//! flash and RAM for the chip we're running on.
//! @note Do not change the index of Flash, SRAM, or QSPI (see memory.h).
memory_map_entry_t g_memoryMap[] = {
    { 0x00000000, 0x0003ffff, kMemoryIsExecutable, &g_flashMemoryInterface },    // Flash array (256KB)
    { 0x1fff0000, 0x2002ffff, kMemoryIsExecutable, &g_normalMemoryInterface },   // SRAM (256KB)
    { 0x68000000, 0x6fffffff, kMemoryNotExecutable, &g_qspiMemoryInterface },    // QSPI memory
    { 0x04000000, 0x07ffffff, kMemoryNotExecutable, &g_qspiAliasAreaInterface }, // QSPI alias area
    { 0x40000000, 0x4007ffff, kMemoryNotExecutable, &g_deviceMemoryInterface },  // AIPS0 peripherals
    { 0x40080000, 0x400fefff, kMemoryNotExecutable, &g_deviceMemoryInterface },  // AIPS1 peripherals
    { 0x400ff000, 0x400fffff, kMemoryNotExecutable, &g_deviceMemoryInterface },  // GPIO
    { 0xe0000000, 0xe00fffff, kMemoryNotExecutable, &g_deviceMemoryInterface },  // M4 private peripherals
    { 0 }                                                                        // Terminator
};
```

　　如下便是用于抽象存储器操作的memory_map_entry原型，该原型可以描述所有类型的memory：  
```C
//! @brief Structure of a memory map entry.
typedef struct _memory_map_entry
{
    // 存储空间起始地址
    uint32_t startAddress;
    // 存储空间结束地址
    uint32_t endAddress;
    // 存储空间属性（Flash/RAM，是否能XIP）
    uint32_t memoryProperty;
    // 存储空间操作接口
    const memory_region_interface_t *memoryInterface;
} memory_map_entry_t;

typedef struct _memory_region_interface
{
    // 存储空间（IP控制器）初始化
    status_t (*init)(void);
    // 从存储空间指定范围内读取数据
    status_t (*read)(uint32_t address, uint32_t length, uint8_t *buffer);
    // 将数据写入存储空间指定范围内
    status_t (*write)(uint32_t address, uint32_t length, const uint8_t *buffer);
    // 将pattern填充入存储空间指定范围内
    status_t (*fill)(uint32_t address, uint32_t length, uint32_t pattern);
    // 对于支持page/section编程的存储器做一次page/section数据写入
    status_t (*flush)(void);
    // 将存储空间指定范围内容擦除
    status_t (*erase)(uint32_t address, uint32_t length);
} memory_region_interface_t;
```

#### 2.3 Command & Data Processor
　　KBOOT核心功能便是与Host之间的命令交互，KBOOT主要工作于Slave模式，实时监听来自Host的命令并做出响应，KBOOT仅能识别事先规定好的命令格式，因此KBOOT必须配套一个专用上位机工具使用。你可能会疑问，为什么这个组件又叫Data Processor？因为有些命令是含有Data phase的（比如read memory, write memory），对于这些命令时除了基本的命令交互响应之后，还必须有数据传输交互响应。  
　　KBOOT中使用如下名叫g_commandInterface和g_commandHandlerTable[]的结构变量来实现核心命令交互，KBOOT中一共实现了19条命令：  
```C
// See bl_command.h for documentation on this interface.
command_interface_t g_commandInterface = 
{
    bootloader_command_init,
    bootloader_command_pump,
    (command_handler_entry_t *)&g_commandHandlerTable,
    &g_commandData
};

//! @brief Command handler table.
const command_handler_entry_t g_commandHandlerTable[] = {
    // cmd handler              // data handler or NULL
    { handle_flash_erase_all, NULL },              // kCommandTag_FlashEraseAll = 0x01
    { handle_flash_erase_region, NULL },           // kCommandTag_FlashEraseRegion = 0x02
    { handle_read_memory, handle_data_producer },  // kCommandTag_ReadMemory = 0x03
    { handle_write_memory, handle_data_consumer }, // kCommandTag_WriteMemory = 0x04
    { handle_fill_memory, NULL },                  // kCommandTag_FillMemory = 0x05
    { handle_flash_security_disable, NULL },       // kCommandTag_FlashSecurityDisable = 0x06
    { handle_get_property, NULL },                    // kCommandTag_GetProperty = 0x07
    { handle_receive_sb_file, handle_data_consumer }, // kCommandTag_ReceiveSbFile = 0x08
    { handle_execute, NULL },                         // kCommandTag_Execute = 0x09
    { handle_call, NULL },                            // kCommandTag_Call = 0x0a
    { handle_reset, NULL },                           // kCommandTag_Reset = 0x0b
    { handle_set_property, NULL },                    // kCommandTag_SetProperty = 0x0c
    { handle_flash_erase_all_unsecure, NULL },        // kCommandTag_FlashEraseAllUnsecure = 0x0d
    { handle_flash_program_once, NULL },              // kCommandTag_ProgramOnce = 0x0e
    { handle_flash_read_once, NULL },                 // kCommandTag_ReadOnce = 0x0f
    { handle_flash_read_resource, handle_data_producer }, // kCommandTag_ReadResource = 0x10
    { handle_configure_memory, NULL },                    // kCommandTag_ConfigureMemory = 0x11
    { handle_reliable_update, NULL },                     // kCommandTag_ReliableUpdate = 0x12
    { handle_generate_key_blob, handle_key_blob_data },   // kCommandTag_GenerateKeyBlob = 0x13
};
```

　　如下便是用于核心命令交互的Command interface原型：  
```C
//! @brief Interface to command processor operations.
typedef struct CommandInterface
{
    // command处理控制单元初始化
    status_t (*init)(void);
    // command处理控制单元pump
    status_t (*pump)(void);
    // command服务函数查找表
    const command_handler_entry_t *handlerTable;
    // command处理控制单元状态数据
    command_processor_data_t *stateData;
} command_interface_t;

//! @brief Format of command handler entry.
typedef struct CommandHandlerEntry
{
    // command服务函数
    void (*handleCommand)(uint8_t *packet, uint32_t packetLength);
    // command的data级处理函数（只有少部分command有此函数）
    status_t (*handleData)(bool *hasMoreData);
} command_handler_entry_t;

//! @brief Command processor data format.
typedef struct CommandProcessorData
{
    // command处理控制状态机当前状态（command/data两种状态）
    int32_t state;
    // 指向当前处理的packet地址
    uint8_t *packet;
    // 当前处理的packet长度
    uint32_t packetLength;
    // command的data级处理控制状态数据
    struct DataPhase
    {
        uint8_t *data;               //!< Data for data phase
        uint32_t count;              //!< Remaining count to produce/consume
        uint32_t address;            //!< Address for data phase
        uint32_t memoryId;           //!< ID of the target memory
        uint32_t dataBytesAvailable; //!< Number of bytes available at data pointer
        uint8_t commandTag;          //!< Tag of command running data phase
        uint8_t option;              //!< option for special command
    } dataPhase;
    // 指向command服务函数查找表地址
    const command_handler_entry_t *handlerEntry; //! Pointer to handler table entry for packet in process
} command_processor_data_t;
```

　　至此，飞思卡尔Kinetis系列MCU的KBOOT架构痞子衡便介绍完毕了，掌声在哪里~~~ 

