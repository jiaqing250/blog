### 1、前期准备
[缺氧MOD开发 从小白到入门（一、工具及环境）](https://www.bilibili.com/read/cv23530974?from=search)
### 2、move the this 使用报错，更新版本
在通过和chatgpt确认的过程中，发现是由于父类增加了一个类，而当前版本引用的模块没有这个类导致的问题
```csharp
Error in HaulingPointComplete.HaulingPoint.OnPrefabInit 
System.MissingFieldException: Field 'TreeFilterable.OnFilterChanged' not found.
  at MoveThisHere.HaulingPoint.Initialize (System.Boolean use_logic_meter) [0x00046] in <077ebee5e8f54c05a2d3c625162ea29d>:0 
  at MoveThisHere.HaulingPoint.OnPrefabInit () [0x00001] in <077ebee5e8f54c05a2d3c625162ea29d>:0 
  at KMonoBehaviour.InitializeComponent () [0x00068] in <0f4b778ef79c497e89f6ee18303840cf>:0 

  at UnityEngine.Debug.LogError (System.Object message, UnityEngine.Object context) [0x00000] in <72b60a3dd8cd4f12a155b761a1af9144>:0 
  at Debug.LogError (System.Object obj, UnityEngine.Object context) [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at DebugUtil.LogErrorArgs (UnityEngine.Object context, System.Object[] objs) [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at DebugUtil.LogException (UnityEngine.Object context, System.String errorMessage, System.Exception e) [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KMonoBehaviour.InitializeComponent () [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KMonoBehaviour.Awake () [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at UnityEngine.GameObject.SetActive (System.Boolean value) [0x00000] in <72b60a3dd8cd4f12a155b761a1af9144>:0 
  at BuildingDef.Create (UnityEngine.Vector3 pos, Storage resource_storage, System.Collections.Generic.IList`1[T] selected_elements, Recipe recipe, System.Single temperature, UnityEngine.GameObject obj) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at BuildingDef.Build (System.Int32 cell, Orientation orientation, Storage resource_storage, System.Collections.Generic.IList`1[T] selected_elements, System.Single temperature, System.Boolean playsound, System.Single timeBuilt) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at MoveThisHere.MoveThisHere_Patch+MoveThisHerePatches+BuildingDef_Instantiate_Patch.Prefix (UnityEngine.Vector3 pos, Orientation orientation, System.Collections.Generic.IList`1[T] selected_elements, System.Int32 layer, BuildingDef __instance, UnityEngine.GameObject& __result) [0x00000] in <077ebee5e8f54c05a2d3c625162ea29d>:0 
  at BuildingDef.BuildingDef.Instantiate_Patch1 (BuildingDef , UnityEngine.Vector3 , Orientation , System.Collections.Generic.IList`1[T] , System.Int32 ) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at BuildingDef.TryPlace (UnityEngine.GameObject src_go, UnityEngine.Vector3 pos, Orientation orientation, System.Collections.Generic.IList`1[T] selected_elements, System.Int32 layer) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at BuildingDef.TryPlace (UnityEngine.GameObject src_go, UnityEngine.Vector3 pos, Orientation orientation, System.Collections.Generic.IList`1[T] selected_elements, System.String facadeID, System.Int32 layer) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at BuildTool.TryBuild (System.Int32 cell) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at BuildTool.OnDragTool (System.Int32 cell, System.Int32 distFromOrigin) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at DragTool.AddDragPoint (UnityEngine.Vector3 cursorPos) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at DragTool.OnLeftClickDown (UnityEngine.Vector3 cursor_pos) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at BuildTool.OnLeftClickDown (UnityEngine.Vector3 cursor_pos) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at PlayerController.OnKeyDown (KButtonEvent e) [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
  at KInputHandler.HandleKeyDown (KButtonEvent e) [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KInputHandler.HandleKeyDown (KButtonEvent e) [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KInputHandler.HandleEvent (KInputEvent e) [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KInputController.Dispatch () [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KInputManager.Dispatch () [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at KInputManager.Update () [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at GameInputManager.Update () [0x00000] in <0f4b778ef79c497e89f6ee18303840cf>:0 
  at Global.Update () [0x00000] in <cd4b9bd5aa6c4ec38ec00dca1dc79105>:0 
Build: U47-562984-SD
```
解决方案: 其实模块是不需要修改代码，只需要再重新打包一份<br />使用Microsoft Visual Studio 开发工具打开项目后 快捷键 ctrl+b 就能生成ddl了<br />从其他地方复制一份mod文件夹，替换ddl文件即可<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1688217624311-0e84438e-be97-4ca1-b0ef-5beafe836edd.png#averageHue=%23fdfdfc&clientId=u6f270ef0-5c9c-4&from=paste&height=298&id=ucb41547c&originHeight=372&originWidth=1023&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=15238&status=done&style=none&taskId=u8e94040f-9e73-4a24-888b-99a98e9c3bd&title=&width=818.4)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/2923644/1688217663900-9eba72f1-750b-4d64-a852-8a4ba6d6afb0.png#averageHue=%23fcfbfb&clientId=u6f270ef0-5c9c-4&from=paste&height=357&id=u176553ac&originHeight=446&originWidth=987&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=32282&status=done&style=none&taskId=u8412b221-087e-47dc-bd84-3abbfc65d62&title=&width=789.6)<br />启动游戏-勾选mod 发现可以正常使用了
