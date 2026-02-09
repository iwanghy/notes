# Systrace 使用 Demo

## 前置准备

### 1. 确保设备已连接
```bash
adb devices
```

### 2. 确保设备已 root
```bash
adb root
adb remount
```

---

## 基础用法

### 方式一：使用 systrace.py（推荐）

#### 1. 进入 systrace 目录
```bash
cd /path/to/android-sdk/platform-tools/systrace
```

#### 2. 查看可用的 categories
```bash
python systrace.py --list
```

输出示例：
```
  gfx       - Graphics
  input     - Input
  view      - View System
  webview   - WebView
  wm        - Window Manager
  am        - Activity Manager
  audio     - Audio
  video     - Video
  camera    - Camera
  hal       - Hardware Modules
  res       - Resource Loading
  dalvik    - Dalvik VM
  rs        - RenderScript
  bionic    - Bionic C Library
  power     - Power Management
  sched     - CPU Scheduling
  freq      - CPU Frequency
  idle      - CPU Idle
  disk      - Disk I/O
  sync      - Synchronization
  binder    - Binder IPC
  binder_driver - Binder Driver
```

#### 3. 抓取 trace（常用场景）

##### 场景 1：分析 UI 卡顿问题
```bash
# 抓取 5 秒，关注图形渲染相关
python systrace.py --time=5 -o ui_trace.html gfx input view wm am sched
```

##### 场景 2：分析应用启动性能
```bash
# 抓取 10 秒，关注 Activity 和进程启动
python systrace.py --time=10 -o app_launch_trace.html am wm dalvik power sched freq
```

##### 场景 3：全面分析（所有 categories）
```bash
# 抓取 5 秒，记录所有信息
python systrace.py --time=5 -o full_trace.html gfx input view webview wm am audio video camera hal res dalvik rs bionic power sched freq idle disk sync binder binder_driver
```

##### 场景 4：指定 buffer 大小（长时间抓取）
```bash
# 抓取 30 秒，设置 8MB buffer
python systrace.py --time=30 --buf-size=8192 -o long_trace.html gfx input view wm sched
```

#### 4. 抓取时自动触发操作
```bash
# 终端 1：开始抓取
python systrace.py --time=10 -o test_trace.html gfx input view wm sched

# 终端 2：立即执行要测试的操作（如滑动屏幕、启动应用等）
```

---

## 方式二：使用 atrace 原生命令

### 1. 抓取 trace
```bash
# 开启抓取
adb shell atrace --async_start -b 8192 gfx input view wm am sched

# 执行测试操作...

# 停止抓取并保存
adb shell atrace --async_stop > atrace_output.txt
```

### 2. 转换为 HTML
```bash
python systrace.py --from-file=atrace_output.txt -o atrace_trace.html
```

---

## 高级用法

### 1. 指定应用 tag
```bash
# 只追踪特定应用的 trace
python systrace.py --time=5 -o app_trace.html --app=com.example.myapp gfx input view
```

### 2. 指定设备
```bash
# 多设备连接时指定目标设备
python systrace.py --time=5 --serial=XXXXXXXX -o trace.html gfx input view
```

### 3. 不压缩输出
```bash
# 生成更小的 HTML 文件（不含原始数据）
python systrace.py --time=5 --no-compress -o trace.html gfx input view
```

---

## 分析 trace 文件

### 1. 打开 HTML 文件
```bash
# 用浏览器打开生成的 HTML
google-chrome ui_trace.html
# 或
firefox ui_trace.html
```

### 2. 常用分析技巧

#### 查看 Frame Drop（掉帧）
1. 在 HTML 中按 **Ctrl+F** 搜索 `Frame`
2. 查看绿色的帧条，黄色/红色表示有掉帧
3. 点击帧可查看详细信息

#### 查看 CPU 调度
1. 查看 `Sched` 轨道
2. 找出长时间 Running 的线程
3. 分析 CPU 负载是否均衡

#### 查看锁竞争
1. 搜索 `lock` 或 `contention`
2. 点击查看调用栈
3. 找出持有锁的线程

---

## 代码中添加自定义 Trace

### Java/Kotlin
```java
import android.os.Trace;

// 开始 trace
Trace.beginSection("MyOperation");

// 你的代码逻辑
doSomething();

// 结束 trace
Trace.endSection();
```

### C++ (Native)
```cpp
#include <cutils/trace.h>

// 开始 trace
ATRACE_BEGIN("NativeOperation");

// 你的代码逻辑
doNativeWork();

// 结束 trace
ATRACE_END();
```

### 计数器 trace
```java
// 记录数值变化
Trace.traceCounter("MyCounter", value);
```

---

## 常见问题

### 1. Buffer 溢出
```
Error: Trace buffer overflow
```
**解决**：增大 buffer
```bash
python systrace.py --buf-size=16384 ...
```

### 2. 权限不足
```
Error: Permission denied
```
**解决**：确保设备已 root
```bash
adb root
adb remount
```

### 3. 设备未连接
```
Error: No device found
```
**解决**：检查 USB 连接
```bash
adb kill-server
adb start-server
adb devices
```

---

## 完整工作流示例

```bash
#!/bin/bash
# systrace_demo.sh - 完整的抓取脚本

# 配置变量
TIME=5
OUTPUT_FILE="demo_trace_$(date +%Y%m%d_%H%M%S).html"
CATEGORIES="gfx input view wm am sched freq"

# 检查设备连接
echo "1. 检查设备连接..."
adb devices

# Root 设备
echo "2. Root 设备..."
adb root
adb remount

# 等待用户准备
echo "3. 准备开始抓取，请在 5 秒内执行要测试的操作..."
sleep 5

# 开始抓取
echo "4. 开始抓取 ${TIME} 秒..."
cd /path/to/android-sdk/platform-tools/systrace
python systrace.py --time=$TIME --buf-size=8192 -o $OUTPUT_FILE $CATEGORIES

# 完成
echo "5. 抓取完成！文件保存至: $OUTPUT_FILE"
echo "6. 用浏览器打开查看"
```

---

## 推荐阅读

- [Android Systrace 官方文档](https://developer.android.com/topic/performance/tracing)
- [Systrace 系列文章](https://www.androidperformance.com/2019/05/28/Android-Systrace-About/)
