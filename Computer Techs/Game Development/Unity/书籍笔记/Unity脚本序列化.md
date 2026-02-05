脚本序列化就是将脚本进行序列化和反序列化，从而将脚本中的数据保存。

在脚本中声明的可序列化数据类型且使用public修饰的字段，会被自动序列化。

## 查看数据

可以设置Unity资源编辑器中的保存格式。

- Mixed：混合。
- Binary：二进制。
- Force Text：所有资源都可用文本打开。

推荐使用`Force Text`，该设置不会影响最终游戏的发布效率，而且可以在编辑模式下更方便地查看资源。

## 序列化数据标签

前面提到public字段会被自动序列化，但是可能只有在运行时才需要使用这个字段，若将其序列化就会浪费内存。

为字段添加`[System.NonSerialized]`特性，可以使其不参与序列化。

若要让字段不显示在Inspector中，但是要参与序列化，则可以使用`[HideInInspector]`特性。

若希望字段是private的，但需要序列化并显示在Inspector中，则可以使用`[SerializeField]`特性。

对于编辑器拓展修改数据，若使用私有字段，则无法被访问，此时可以使用SerializedObject和SerializedProperty来获取和设置数据。

```C#
public class Sample: MonoBehaviour 
{
	[SerializedField] private int Id;
	[SerializedField] private string Name;
	[SerializedField] private GameObject Prefab;
	
#if UNITY_EDIOR
	[CustomEditor(typeof(Sample))]
	public class SampleInspector: Editor
	{
		public override void OnInspectorGUI()
		{
			// 更新最新数据
			serializedObject.Update();
			// 获取数据信息
			SerializedProperty property = serialiedObject.FindProperty(nameof(Id));
			// 赋值数据
			property.intValue = EditorGUILayout.IntField("主键", property.intValue);
			
			property = serialiedObject.FindProperty(nameof(Name));
			property.stringValue = EditorGUILayout.IntField("姓名", property.stringValue);
			
			property = serialiedObject.FindProperty(nameof(Prefab));
			property.objectReferenceValue = EditorGUILayout.ObjectField("游戏对象", property.objectReferenceValue, typeof(GameObject), true);
			
			// 保存全部数据
			serializedObject.ApplyModifiedProperties();
		}
	}
#endif
}
```

- OnInspectorGUI：对Inspector面板绘制的函数。
- serializedObject.Update：将所有参与序列化的数据更新。
- serializedObject.FindProperty(nameof(Id))：获取属性对象。
- serializedObject.ApplyModifiedProperties：修改变量后整体修改数据并保存。

## 监听数据更改

对于普通Monobehaviour脚本，可通过OnValidate生命周期函数监听脚本数据更改。

对于编辑器脚本，可以在需要监听的GUI元素外面包裹EditorGUI.BeginChangeCheck和EditorGUI.EndChangeCheck。

## 序列化/反序列化监听

继承ISerializationCallbackReceiver接口。