

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

