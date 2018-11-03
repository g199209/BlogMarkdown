title: 使用Doxygen生成高质量代码参考文档
date: 2015-11-12 18:44:51
tags: [Doxygen]
categories: 工具之术

---

注释和文档是程序的重要组成部分，使用 [Doxygen](http://www.stack.nl/~dimitri/doxygen/index.html) 可以自动将程序中特定格式的注释提取出来，生成一份很漂亮的参考文档。这样只要在编程的时候遵循特定的格式书写注释，程序编好后文档也就自动完成了，不需要专门花时间来写文档，这无疑减少了很多的工作量。

Doxygen支持的语言有很多，常用的有C/C++、Java、Python等；可以生成的文档格式也很丰富，其中以HTML格式和Latex格式最为常用。

关于Doxygen的安装和使用，可参考其官方 [参考手册](http://www.stack.nl/~dimitri/doxygen/manual/index.html)， 这个参考手册本身也是使用Doxygen生成的。网上也可找到很多基础教程，在此不再累述。本文主要介绍在使用C语言编程时该如何写注释，以便使用Doxygen自动生成高质量的文档。

<!--more-->

下面以STM32F0系列单片机的标准库（Standard Peripheral Library）中的ADC模块为例进行分析，这是ST公司为其F0系列单片机提供的驱动库文件，注释的书写十分标准，很适合进行学习模仿。

## **定义功能模块**
STM32F0xx_StdPeriph_Driver是按照功能模块进行组织的，每个功能模块有其对应的头文件与源文件。功能模块列表如下：
![](http://gmf.shengnengjin.cn/Doxygen20151112222545.png)
为了让Doxygen能正确提取模块定义，需添加模块定义注释。在文件中加入：
``` C
/** @addtogroup ADC 
  * @brief ADC driver modules
  * @{
  */

//程序代码

/**
  * @}
  */ 
```
使用`@brief`对本模块的主要功能进行说明，如需进行更详细的说明，可使用`@details`。

## **定义宏定义子模块**
定义了功能模块后，一般还将宏定义单独定义为一个子模块，宏定义一般放在头文件中，加入以下注释：
``` C
/** @defgroup ADC_Exported_Constants
  * @{
  */ 
  
/** @defgroup ADC_JitterOff
  * @{
  */ 
#define ADC_JitterOff_PCLKDiv2                    ADC_CFGR2_JITOFFDIV2
#define ADC_JitterOff_PCLKDiv4                    ADC_CFGR2_JITOFFDIV4

#define IS_ADC_JITTEROFF(JITTEROFF) (((JITTEROFF) & 0x3FFFFFFF) == (uint32_t)RESET)
/**
  * @}
  */ 
  
/** @defgroup ADC_Resolution
  * @{
  */ 
#define ADC_Resolution_12b                         ((uint32_t)0x00000000)
#define ADC_Resolution_10b                         ADC_CFGR1_RES_0
#define ADC_Resolution_8b                          ADC_CFGR1_RES_1
#define ADC_Resolution_6b                          ADC_CFGR1_RES

#define IS_ADC_RESOLUTION(RESOLUTION) (((RESOLUTION) == ADC_Resolution_12b) || \
                                       ((RESOLUTION) == ADC_Resolution_10b) || \
                                       ((RESOLUTION) == ADC_Resolution_8b) || \
                                       ((RESOLUTION) == ADC_Resolution_6b))

/**
  * @}
  */ 
  
/**
  * @}
  */ 
```
以上代码段中定义了一个名为`ADC_Exported_Constants`的模块，此模块包含了ADC模块中所有的宏定义。在`ADC_Exported_Constants`模块中，又有若干个子模块，如`ADC_JitterOff`，`ADC_Resolution`等。它们的关系可以参考下图：
![](http://gmf.shengnengjin.cn/Doxygengroup___a_d_c___exported___constants.png)
这样组织代码可以让大的量宏定义结构更为清晰，而且有利于在其它地方进行交叉引用。

## **定义函数子模块**
与宏定义子模块类似，也可以将函数定义为一个子模块，并且按功能进行进一步分组归类。函数的定义与注释说明一般都全部放在源文件中，头文件中仅简单的进行函数声明。在源文件中加入以下注释：
``` C
/** @defgroup ADC_Private_Functions
  * @{
  */

/** @defgroup ADC_Group1 Initialization and Configuration functions
  * @brief   Initialization and Configuration functions 
  *
  * @{
  */
void ADC_DeInit(ADC_TypeDef* ADCx){}
void ADC_Init(ADC_TypeDef* ADCx, ADC_InitTypeDef* ADC_InitStruct){}
void ADC_StructInit(ADC_InitTypeDef* ADC_InitStruct){}
/**
  * @}
  */
  
/** @defgroup ADC_Group2 Power saving functions
  * @brief   Power saving functions 
  * @{
  */
void ADC_AutoPowerOffCmd(ADC_TypeDef* ADCx, FunctionalState NewState){}
void ADC_WaitModeCmd(ADC_TypeDef* ADCx, FunctionalState NewState){}
/**
  * @}
  */

/**
  * @}
  */ 
```
以上代码在实际代码的基础上进行了精简，仅保留了进行模块定义的基本结构。代码段中定义了一个名为`ADC_Private_Functions`的模块，此模块中包含了所有的函数定义，并且各函数按用途进行了分组，分为了若干个子模块，如`Initialization and Configuration functions`，`Power saving functions`等。关系示意图如下：
![](http://gmf.shengnengjin.cn/Doxygengroup___a_d_c___private___functions.png)

## **结构体注释说明**
对于结构体，一般使用如下形式的注释：
``` C
/** 
  * @brief  ADC Init structure definition
  */
typedef struct
{
  uint32_t ADC_Resolution;                  /*!< Selects the resolution of the conversion.
                                                 This parameter can be a value of @ref ADC_Resolution */

  FunctionalState ADC_ContinuousConvMode;   /*!< Specifies whether the conversion is performed in
                                                 Continuous or Single mode.
                                                 This parameter can be set to ENABLE or DISABLE. */

  uint32_t ADC_ExternalTrigConvEdge;        /*!< Selects the external trigger Edge and enables the
                                                 trigger of a regular group. This parameter can be a value
                                                 of @ref ADC_external_trigger_edge_conversion */

  uint32_t ADC_ExternalTrigConv;            /*!< Defines the external trigger used to start the analog
                                                 to digital conversion of regular channels. This parameter
                                                 can be a value of @ref ADC_external_trigger_sources_for_channels_conversion */

  uint32_t ADC_DataAlign;                   /*!< Specifies whether the ADC data alignment is left or right.
                                                 This parameter can be a value of @ref ADC_data_align */

  uint32_t  ADC_ScanDirection;              /*!< Specifies in which direction the channels will be scanned
                                                 in the sequence. 
                                                 This parameter can be a value of @ref ADC_Scan_Direction */
}ADC_InitTypeDef;
```
此代码段中对`ADC_InitTypeDef`这个结构体进行了较为详尽的注释，开头使用`@brief`说明此结构体的主要功能，之后对结构体中每个成员的意义进行了注释。使用`/*!< Comment */`这样的语法表示此注释对应的是注释前面的语句，这样可以使代码的排版更为美观。

另外，注意到其中`@ref`标签的使用，这代表交叉引用。如`@ref ADC_Resolution`，实际会生成一个超链接指向之前定义的`ADC_Resolution`模块。这里的交叉引用可以为模块名、函数名、结构体等。

以上代码段提取出的文档效果见下图：
![](http://gmf.shengnengjin.cn/Doxygen20151113211950.png)

枚举的注释形式与结构体完全相同，可参照以上示例进行注释。

## **函数注释说明**
函数的注释说明一般放在源文件中，头文件和源文件中最好不要重复添加注释，否则生成的文档会有重复。函数注释一般使用以下的形式：
``` C
/**
  * @brief  Enables or disables the ADC DMA request after last transfer (Single-ADC mode)
  * @param  ADCx: where x can be 1 to select the ADC1 peripheral.
  * @param  ADC_DMARequestMode: the ADC channel to configure. 
  *          This parameter can be one of the following values:
  *            @arg ADC_DMAMode_OneShot: DMA One Shot Mode 
  *            @arg ADC_DMAMode_Circular: DMA Circular Mode  
  * @retval None
  */
void ADC_DMARequestModeConfig(ADC_TypeDef* ADCx, uint32_t ADC_DMARequestMode){}

/**
  * @brief  Active the Calibration operation for the selected ADC.
  * @note   The Calibration can be initiated only when ADC is still in the 
  *         reset configuration (ADEN must be equal to 0).
  * @param  ADCx: where x can be 1 to select the ADC1 peripheral.
  * @retval ADC Calibration factor 
  */
uint32_t ADC_GetCalibrationFactor(ADC_TypeDef* ADCx){}
```
使用`@brief`简要说明函数的作用；使用`@param`说明输入参数，若输入参数是有限的几个值，可用`@arg`进行列举；使用`@retval`说明函数的返回值。另外，一些需要特别注意的地方可以使用`@note`，`@warning`进行说明。

以上代码段提取出的文档效果见下图：
![](http://gmf.shengnengjin.cn/Doxygen20151113212310.png)

## **文件头**
每个文件的开头部分一般都需要添加一个对此文件的说明，可使用如下格式：
``` C
/**
  **************************************************************
  * @file Example.c
  * @author 高明飞
  * @version V1.0
  * @date 2015-11-13
  *
  * @brief 程序的简要说明
  *
  * @details 
  * @verbatim
  * 程序的详细说明。
  *
  * 修改记录：
  * 2015-11-13 :
  *   - 修改记录
  *
  * @endverbatim
  ***************************************************************
  */
```

## **杂项**
- 使用`@todo`和`@bug`标签列出待办事项与Bug，Doxygen会自动汇总Todo与Bug列表。

- 可使用`@verbatim`与`@endverbatim`包含一段文本，这段文本就会按原样输出。







