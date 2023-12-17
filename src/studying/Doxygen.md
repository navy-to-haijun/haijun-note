# Doxygen 

详细描述

```c
javadoc风格
/**
 * ... text ...
 */
qt风格
/*!
 * ... text ...
 */
比较明显的注释
/************************************************
 *  ... text
 ***********************************************/
```

简短描述

```c
int var; ///< Brief description after the member
```



| 命令        | 字段名                                   | 语法                                                      |      |
| ----------- | ---------------------------------------- | --------------------------------------------------------- | ---- |
| @file       | 文件名                                   | file [< name >]                                           |      |
| @brief      | 简介                                     | brief { brief description }                               |      |
| @author     | 作者                                     | author { list of authors }                                |      |
| @mainpage   | 主页信息                                 | mainpage [(title)]                                        |      |
| @date       | 年-月-日                                 | date { date description }                                 |      |
| @version    | 版本号                                   | version { version number }                                |      |
| @copyright  | 版权                                     | copyright { copyright description }                       |      |
| @param      | 参数                                     | param [(dir)] < parameter-name> { parameter description } |      |
| @return     | 返回                                     | return { description of the return value }                |      |
| @retval     | 返回值                                   | retval <return value> { description }                     |      |
| @bug        | 漏洞                                     | bug { bug description }                                   |      |
| @details    | 细节                                     | details { detailed description }                          |      |
| @pre        | 前提条件                                 | pre { description of the precondition }                   |      |
| @see        | 参考                                     | see { references }                                        |      |
| @link       | 连接(与@see类库，{@link www.google.com}) | link < link-object>                                       |      |
| @throw      | 异常描述                                 | throw < exception-object> { exception description }       |      |
| @todo       | 待处理                                   | todo { paragraph describing what is to be done }          |      |
| @warning    | 警告信息                                 | warning { warning message }                               |      |
| @deprecated | 弃用说明。可用于描述替代方案，预期寿命等 | deprecated { description }                                |      |
| @example    | 弃用说明。可用于描述替代方案，预期寿命等 | deprecated { description }                                |      |

文件注释

```c
/**
 * @file 文件名
 * @brief 简介
 * @details 细节
 * @author 作者
 * @version 版本号
 * @date 年-月-日
 * @copyright 版权
 */
```

结构体注释

```c
 /**
 * @brief 类的详细描述
 */
```

函数描述

```c
 /**
  * @brief 函数描述
  * @param 参数描述
  * @return 返回描述
  * @retval 返回值描述
  */
```

