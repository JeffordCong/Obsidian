

[官方文档](https://docs.unity3d.com/2022.3/Documentation/Manual/ProfilerWindow.html)

[Common Profiler markers](https://docs.unity3d.com/2022.3/Documentation/Manual/profiler-markers.html)


#### WaitForTargetFPS


**WaitForTargetFPS** 是 Unity 主线程（Main Thread）在当前帧的所有逻辑处理（Update、Physics、Animation）和渲染提交（Rendering）完成后，为了**强制对齐**设定的目标帧率（`Application.targetFrameRate`），而执行的一种**主动节流（Throttling）或休眠操作**。


*数学公式 (The Math):*


Unity 每一帧都会计算时间差：

$$T_{wait} = T_{target} - T_{actual}$$

- $T_{target}$：目标帧间隔（例如 60FPS $\approx$ 16.66ms）。
    
- $T_{actual}$：当前帧实际消耗的 CPU 时间。
    
- 如果 $T_{wait} > 0$，Unity 就会执行 `WaitForTargetFPS`。


**WaitForTargetFPS** 是**“为了不跑太快而故意浪费的时间”**。在 Profiler 中看到它，意味着你的 CPU 性能过剩，完全能毫无压力地跑满设定的帧率。



#### WaitForLastPresent和 WaitForPresent

这二者都属于渲染管线末端（Back-end）的同步等待，但它们的等待时机和根本原因不同。

##### 1. `Gfx.WaitForPresent` (最常见)

这是 CPU 主线程或渲染线程在一帧的最后，请求 GPU 展示画面时发生的等待。当 CPU 提交完命令缓冲区（CommandBuffer）后，调用底层图形 API（如 `d3dPresent`、`eglSwapBuffers`、`vkQueuePresentKHR`）。此时会发生两种情况：

1. **VSync 等待 (Normal):**
    
    - 如果你开启了垂直同步（VSync），显示器还没准备好刷新（例如 60Hz 屏幕每 16.6ms 刷新一次）。CPU 跑得太快，必须在这里**死等**显示器的 VSync 信号。
        
    - **判定：** 只要帧率稳定满帧，这个等待就是健康的（类似 `WaitForTargetFPS`）。
        
2. **GPU 拥塞等待 (Bad - GPU Bound):**
    
    - 如果你关闭了 VSync，或者开启了 VSync 但并未满帧。
        
    - 此时 CPU 提交了画面，但 **GPU 还没画完**（Back Buffer 被 GPU 锁住）。CPU 被驱动层阻塞，必须等 GPU 释放 Back Buffer 才能提交当前帧。
        
    - **判定：** 这是典型的 **GPU 瓶颈**。
        

**总结：** `WaitForPresent` = **"我（CPU）画完了，GPU 你好了没？或者屏幕你好了没？"**


### 2. `WaitForLastPresent` (移动端/Frame Pacing 特有)

**定义：** 这是 Android (尤其是开启了 Swappy / Google Frame Pacing) 或某些 Vulkan/Metal 实现下的特有标记。它通常发生在**一帧的开始阶段**或**提交渲染前**。

**底层原理 (The Pipeline Depth):** 为了减少**输入延迟 (Input Lag)**，现代引擎（如 Unity）会限制 CPU 能够“超前” GPU 多少帧。通常是 **Double Buffering (双重缓冲)** 或 **Triple Buffering (三重缓冲)**。

- **场景：** 假设 CPU 极快，GPU 极慢。
    
- **如果不加限制：** CPU 会连续提交第 100, 101, 102 帧的命令，存入驱动队列。玩家操作了第 103 帧，但要等前 3 帧渲染完才能看到反应，手感极“肉”（Latency 高）。
    
- **机制：** `WaitForLastPresent` 是一个**“栅栏 (Fence)”**。引擎会强制 CPU 等待，直到 GPU 彻底完成了**上一帧**（或上上帧）的显示操作。
    
- **目的：** 确保渲染队列（Render Queue）不会堆积太长，保持“CPU 只比 GPU 快一帧”，从而降低延迟。
    

**判定逻辑：**

- 如果在 Profiler 中看到 `WaitForLastPresent` 很高，说明 **GPU 处理上一帧花费了太长时间**，导致 CPU 被迫等待 GPU 腾出位置。
    
- **结论：** 依然是 **GPU 瓶颈**。
    

**总结：** `WaitForLastPresent` = **"队列满了，我（CPU）得等 GPU 把上一帧消化完，才能开始搞新的一帧。"**