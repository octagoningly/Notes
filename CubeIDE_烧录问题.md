# STM32CubeIDE 烧录报错
## 一、报错日志
### 1. 报错信息
在使用 STM32CubeIDE 启动调试（`LED_IDE Debug`）时，出现以下弹窗错误：
> 'Launching LED_IDE Debug' has encountered a problem.
> Error in final launch sequence:
> Failed to start GDB server

### 2. 控制台完整日志
```log
Failed to bind to port 61235, error code -1: No error
Failure starting SWV server on TCP port 61235
Failed to bind to port 61234, error code -1: No error
Failure starting GDB server: TCP port 61234 not available.
Shutting down...
Exit.
```
### 3. 问题根本原因
调试所需的 **TCP 端口 61234/61235** 被其他进程**占用**，导致新的 GDB 服务无法启动，调试会话初始化失败。


## 二、解决
### 步骤1：确认端口占用情况
cmd执行以下命令，查询占用端口的进程：
```powershell
netstat -ano | findstr :61234
```
执行结果示例（PID 为 5180 的进程占用端口）：
```log
TCP    0.0.0.0:61234           0.0.0.0:0              LISTENING       5180
```

### 步骤2：强制结束占用端口的进程
使用 `taskkill` 命令终止残留进程，释放端口：
```powershell
taskkill /F /PID 5180
```


### 步骤3：验证端口是否成功释放
再次执行端口查询命令，确认无进程占用：
```powershell
netstat -ano | findstr :61234
```
如果无任何输出，说明端口已成功释放。

### 步骤4：重启 STM32CubeIDE 并重新调试


