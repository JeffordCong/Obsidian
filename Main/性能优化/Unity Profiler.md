

[官方文档](https://docs.unity3d.com/2022.3/Documentation/Manual/ProfilerWindow.html)

[Common Profiler markers](https://docs.unity3d.com/2022.3/Documentation/Manual/profiler-markers.html)


#### WaitForTargetFPS

**WaitForTargetFPS** 是 Unity 主线程（Main Thread）在当前帧的所有逻辑处理（Update、Physics、Animation）和渲染提交（Rendering）完成后，为了**强制对齐**设定的目标帧率（`Application.targetFrameRate`），而执行的一种**主动节流（Throttling）或休眠操作**。



