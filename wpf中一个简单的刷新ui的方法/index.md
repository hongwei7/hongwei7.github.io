# WPF 中一个简单刷新 UI 的方法




以前写一个WPF的游戏时曾经用过`Inoke`方法来实现UI的实时更新,但是听说WinForm有更加简单的`App.DoEvents()`方法来更新UI. 后来找到了在WPF中实现类似函数的方法.

```c++
private static DispatcherOperationCallback exitFrameCallback = new DispatcherOperationCallback(ExitFrame);
public static void DoEvents()
{
  DispatcherFrame nestedFrame = new DispatcherFrame();
  DispatcherOperation exitOperation = Dispatcher.CurrentDispatcher.BeginInvoke(DispatcherPriority.Background,exitFrameCallback, nestedFrame);
  Dispatcher.PushFrame(nestedFrame);
  if (exitOperation.Status !=DispatcherOperationStatus.Completed)
  {
    exitOperation.Abort();
  }
}
private static Object ExitFrame(Object state)
{
    DispatcherFrame frame = state as
    DispatcherFrame;
    frame.Continue = false;
    return null;
}
```

这样在需要的地方运行`DoEvent()`即可以实现UI的刷新.性能赶不上使用`Inoke()`方法的多线程刷新UI,但这个方法胜在实现上较为简单,不需要`TimerTick()`函数.

