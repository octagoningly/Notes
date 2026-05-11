# 环境搭建
先在ST官网下载STM32CCubeCLT，解压后安装，安装的文件包含有很多工具链
## 一、vscode+CubeMX
### 1. vscode插件安装：EIDE 和 Cortex-Debug
### 2. CubeMX中生成工具链为CubeIDE类型
### 3. 配置gcc工具链
* 进入在左侧EIDE ，下方操作，安装GNU工具链（或者导入本地已有地址）  
* GNU 即 GNU Arm Embedded Toolchain (arm-none-eabi-gcc)
### 4. 导入工程
* 在左侧EIDE插件点击导入工程
* 点开工程项目中.cproject 文件
* 点击打开工作区
### 5. EIDE--构建配置
* 可改芯片类型等，一般不用改
* 链接脚本为`STM32F103C8TX FLASH.ld`文件，一般已自动导入
### 6.改烧录配置STlink
### 7.屏蔽部分文件
* 编译时部分源文件会报错，因为EIDE认为这是多余的文件
* 在左侧文件资源管理器中查看有红点的报错文件
* 回到EIDE--项目资源，将刚刚有红点的系统文件屏蔽（不参与编译）
* 包括：Drivers\CMSIS\Core\Template ；Drivers\CMSIS\Core_A ;Drivers\CMSIS\Core\DSP ; Drivers\CMSIS\Core\NN ; Drivers\CMSIS\Core\RTOS2

### 8.编译、下载、调试
* 三个按键均在右上角
* 调试设置：
    * 先在EIDE--项目设置--调试器
    * 选为刚刚安装的插件 Cortex-Debug