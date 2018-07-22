----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔Kinetis系列MCU的KBOOT之ROM API特性**。  

　　KBOOT的ROM API特性主要存在于ROM Bootloader形态中，KBOOT内部集成了一些Kinetis内部IP模块driver，这些IP模块driver首要目的是用于实现KBOOT的功能，但由于这些IP模块driver会随着KBOOT一起被固化在ROM空间里，所以如果这些IP driver能够被外部（主要是运行于Flash中的Application）调用，那么肯定会节省Application代码空间，这么看起来将ROM Bootloader里的一些IP driver以API的形式export出去是很有意义的，这么有意义的事，KBOOT当然是会做的。下面痞子衡给大家介绍KBOOT里的ROM API特性：  

### 一、API tree原型
　　KBOOT中声明了如下一个名叫bootloader_tree_t的结构体来组织那些可被export出去的IP模块driver，结构体前4个成员记录了版本、版权以及Bootloader等信息，从第5个成员开始便是那些IP模块driver API，目前支持导出的模块API并不多，一共4个：Flash、AES、Kboot、USB。  

```C
//! @brief Structure of version property.
typedef union StandardVersion
{
    struct {
        uint8_t bugfix;     //!< bugfix version [7:0]
        uint8_t minor;      //!< minor version [15:8]
        uint8_t major;      //!< major version [23:16]
        char name;          //!< name [31:24]
    };
    uint32_t version;   //!< combined version numbers
} standard_version_t;

//! @brief Root of the bootloader API tree.
//!
//! An instance of this struct resides in read-only memory in the bootloader. It
//! provides a user application access to APIs exported by the bootloader.
//!
//! @note The order of existing fields must not be changed.
typedef struct BootloaderTree
{
    void (*runBootloader)(void *arg);            //!< Function to start the bootloader executing.
    standard_version_t version;                  //!< Bootloader version number.
    const char *copyright;                       //!< Copyright string.
    const bootloader_context_t *runtimeContext;  //!< Pointer to the bootloader's runtime context.
    const flash_driver_interface_t *flashDriver; //!< Flash driver API.
    const aes_driver_interface_t *aesDriver;     //!< AES driver API.
    const kb_interface_t *kbApi;                 //!< Bootloader API.
    const usb_driver_interface_t *usbDriver;     //!< USB driver API.
} bootloader_tree_t;
```

　　在所有导出的模块driver API中，Flash driver API是使用最广泛的，其API原型如下（不同芯片中Flash driver API版本可能不一致，下面是用于K80芯片上的F1.2.2版）：  

```C
//! @brief Interface for the flash driver.
typedef struct FlashDriverInterface {
    standard_version_t version;
    status_t (*flash_init)(flash_driver_t * driver);
    status_t (*flash_erase_all)(flash_driver_t * driver, uint32_t key);
    status_t (*flash_erase_all_unsecure)(flash_driver_t * driver, uint32_t key);
    status_t (*flash_erase)(flash_driver_t * driver, uint32_t start, uint32_t lengthInBytes, uint32_t key);
    status_t (*flash_program)(flash_driver_t * driver, uint32_t start, uint32_t * src, uint32_t lengthInBytes);
    status_t (*flash_get_security_state)(flash_driver_t * driver, flash_security_state_t * state);
    status_t (*flash_security_bypass)(flash_driver_t * driver, const uint8_t * backdoorKey);
    status_t (*flash_verify_erase_all)(flash_driver_t * driver, flash_margin_value_t margin);
    status_t (*flash_verify_erase)(flash_driver_t * driver, uint32_t start, uint32_t lengthInBytes, flash_margin_value_t margin);
    status_t (*flash_verify_program)(flash_driver_t * driver, uint32_t start, uint32_t lengthInBytes, const uint8_t * expectedData, flash_margin_value_t margin, uint32_t * failedAddress, uint32_t * failedData);
    status_t (*flash_get_property)(flash_driver_t * driver, flash_property_t whichProperty, uint32_t * value);
    status_t (*flash_register_callback)(flash_driver_t * driver, flash_callback_t callback);
    status_t (*flash_program_once)(flash_driver_t * driver, uint32_t index, uint32_t * src, uint32_t lengthInBytes);
    status_t (*flash_read_once)(flash_driver_t * driver, uint32_t index, uint32_t * dst, uint32_t lengthInBytes);
    status_t (*flash_read_resource)(flash_driver_t * driver, uint32_t start, uint32_t *dst, uint32_t lengthInBytes, flash_read_resource_option_t option);
} flash_driver_interface_t;
```

> * Note: 模块driver API设计必须满足几个条件：一、API里不能使用全局变量；二、模块IRQHandler不能直接当做API  

### 二、API tree位置
　　声明好了bootloader_tree_t结构体原型以及各IP模块driver API原型，下一步便是在KBOOT中定义如下常量g_bootloaderTree以创建对象分配内存（不同芯片中g_bootloaderTree版本可能不一致，下面是用于K80芯片上的K1.3.0版）。  

```C
//! @brief Static API tree.
const bootloader_tree_t g_bootloaderTree =
{
    .runBootloader = bootloader_user_entry,
    .version = {
            .name = kBootloader_Version_Name,
            .major = kBootloader_Version_Major,
            .minor = kBootloader_Version_Minor,
            .bugfix = kBootloader_Version_Bugfix
        },
    .copyright = bootloaderCopyright,
    .runtimeContext = &g_bootloaderContext,
    .flashDriver = &g_flashDriverInterface,
    .aesDriver = &g_aesInterface
};
```

　　只要找到g_bootloaderTree地址，便可以访问到那些IP模块driver API，现在的问题是如何找到g_bootloaderTree地址？我们知道在KBOOT工程中，如果不在链接文件里明确指定g_bootloaderTree地址，链接器会随机分配一个地址来存放g_bootloaderTree，这会导致在不同芯片中g_bootloaderTree地址是不一样的；但即使强制指定g_bootloaderTree链接地址，如果ROM空间起始地址不一定是从0x1c000000开始，那么还是难以做到g_bootloaderTree地址统一。到底该怎么解决这个问题？KBOOT使用了一个巧妙的方法，下面是KBOOT工程的startup文件（IAR版），KBOOT将g_bootloaderTree的地址放到了中断向量表第8个向量的位置处（该向量为ARM Cortex-M未定义的系统向量），因此只要知道了ROM空间的起始地址，那么偏移0x1c处开始的4bytes便是g_bootloaderTree地址。  

```C
        MODULE  ?cstartup

        ;; Forward declaration of sections.
        SECTION CSTACK:DATA:NOROOT(3)
        SECTION .intvec:CODE:NOROOT(2)

        EXTERN  g_bootloaderTree
        PUBLIC  __vector_table
        PUBLIC  __vector_table_0x1c

        DATA

__vector_table
        DCD     sfe(CSTACK)
        DCD     Reset_Handler

        DCD     NMI_Handler
        DCD     HardFault_Handler
        DCD     MemManage_Handler
        DCD     BusFault_Handler
        DCD     UsageFault_Handler
__vector_table_0x1c
        DCD     g_bootloaderTree
        DCD     0
        DCD     0
        DCD     0
        DCD     SVC_Handler
        DCD     DebugMon_Handler
        DCD     0
        DCD     PendSV_Handler
        DCD     SysTick_Handler
		;; ...
```

### 三、调用API的方法
　　KBOOT中的ROM API调用方法非常简单，以K80芯片中Flash driver API调用为例，首先需要按以下步骤定义s_flashInterface指针：  

```C
#define BOOTLOADER_TREE_LOCATION (0x1c00001cul)

#define BOOTLOADER_API_TREE_POINTER (*(bootloader_tree_t **)BOOTLOADER_TREE_LOCATION)

static const flash_driver_interface_t *s_flashInterface = BOOTLOADER_API_TREE_POINTER->flashDriver;
```

　　有了s_flashInterface指针便可以随意访问Flash driver API：  

```C
const uint32_t test_program_buffer[2] = { 0x01234567, 0x89abcdef };
flash_config_t flashInstance;

s_flashInterface->flash_init(&flashInstance);
s_flashInterface->flash_erase(&flashInstance, 0x8000, 0x1000);
s_flashInterface->flash_program(&flashInstance, 0x8000, (uint32_t *)test_program_buffer, 8);
```

　　至此，飞思卡尔Kinetis系列MCU的KBOOT之ROM API特性痞子衡便介绍完毕了，掌声在哪里~~~  

