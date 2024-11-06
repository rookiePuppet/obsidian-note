#### 引入

序列化是Unity的一个核心，许多特性都是基于序列化系统的：

- 保存存储在脚本中的数据。
- Inspector窗口：Inspector窗口并不是通过C# API来计算正在检查的属性值，而是让对象序列化自己，然后显示出序列化的数据。
- 预制体：在内部，预制体是游戏对象和组件的序列化数据流。一个预制体实例是应该在序列化的数据之上进行的修改列表。预制体的修改仅仅存在于编辑阶段。当Unity进行构建时，预制体的修改会被放入正常的序列化流中。当预制体被实例化之后，实例化出来的游戏对象并不知道它们原先是编辑器中的一个预制体。
- 实例化：当你调用Instantiate方法实例化预制体或其他（继承于UnityEngine.Object的对象就可以被序列化），我们序列化对象，然后创建一个新对象，之后再反序列化数据到新对象上。
- 保存：如果使用文本编辑器打开一个.unity后缀的场景文件，并在Unity中设置”fore text serialization“，那么序列化器会使用YAML后端运行。
- 加载：这似乎并不奇怪，但是向后兼容加载也是一个建立在序列化之上的系统。在编辑器中，yaml加载使用了序列化系统，同样在运行时的场景和资源加载，Assetbundle也使用了序列化系统。
- 编辑器模式的热重载：当你修改了一个编辑器脚本，我们会序列化所有的编辑器窗口（它们继承自UnityEngine.Object），然后销毁所有窗口，卸载旧的C#代码，加载新的C#代码，重新创建窗口，最后反序列化窗口的数据流到新窗口。
- Resource.GarbageCollectSharedAssets()：这是我们原生的垃圾回收器，与C#的垃圾回收器不同。当你加载了一个场景，这个方法所做的就是计算来自之前的场景中不再被引用的事物，然后卸载它们。原生的垃圾回收器运行序列化器，被我们用于报告所有对外部UnityEngine.Object引用的对象。这使被场景1使用的纹理，在加载场景2之后就会被卸载。

这个序列化系统是用C++写的，我们将它用于所有内部的对象类型（Textures，AnimationClip，Camera，等等）。序列化发生在UnityEngine.Object层次中，每个UnityEngine.Object总是被作为整体被序列化，它们可以包含对其他UnityEngine.Object的引用，这些引用会被正确地序列化。

现在你可能会说这对你无关紧要，只是对它能够工作感到高兴并想继续创建一些实际的内容。但是，这和你是有关的，因为我们使用同样的序列化器去序列化由你编写的MonoBehaviour组件。由于这个序列化器有着非常高的性能需求，在所有情况下，它的行为并不完全像C#开发者期望的那样。下面我们将描述序列化器是如何工作的，以及如何最好地利用它的最佳实践。

#### 什么样的字段在脚本中能够被序列化？

- `public`，或者有`[SerializeField]`特性
- 非`static`
- 非`const`
- 非`readonly`
- 必须是能够被序列化的类型

#### 什么类型能够序列化

- 有`[Serializable]`特性的自定义非抽象类、结构体
- 对继承自UnityEngine.Obejct类对象的引用（ 如**GameObject**、 **Component**、**Material**、**Texture**、**Mesh**、**Sprite**）
- 原始的数据类型（`int`, `float`, `double`, `bool`, `string`等等）
- 能够序列化的类型的数组和`List`

#### 序列化器在哪些情形中会和期望中表现的不同？

##### 表现为结构体的自定义类

```C#
[Serializable]
class Animal
{
    public string name;
}

class MyScript : MonoBehaviour
{
    public Animal[] animals;
}
```

如果你在animals数组中填充了三个引用，都指向同一个Animal对象，在序列化流中你会发现有3个对象，当它被反序列化时，就会有3个不同的对象了。如果需要序列化一个包含引用的复杂对象图，你不能依赖于Unity的序列化器自动为你完成，而是需要自己去实现序列化。

注意这只适用于自定义类，它们是内联序列化的，因为它们的数据会成为MonoBehaviour完整序列化数据的一部分。当你有对UnityEngine.Object派生类对象的字段引用时，例如”public Camema myCamera“，这个Camera中的数据不是被内联序列化的，对Camera实际的引用才会被序列化。

##### 不支持为null的自定义类

```C#
class Test : MonoBehaviour
{
    public Trouble t;
}

[Serializable]
class Trouble
{
    public Trouble t1;
    public Trouble t2;
    public Trouble t3;
}
```

当序列化Test时，会造成多少内存分配？

答案是729。

序列化器不支持null，如果序列化一个为null的对象，我们就实例化一个新的该类型的对象，然后再序列化它。显然这回导致无线循环，所以它会有一个深度的限制，最多7个层次。

由于有太多的子系统是基于序列化系统构建的了，对于Test MonoBehaviour的这个意料之外的巨大序列化流，会导致所有的子系统运行变得更慢。当我们调查客户项目的性能问题时，几乎总是发现有这个问题，所以在Unity 4.5中增加了这种情形的警告。

##### 不支持多态性

如果有一个`public Animal[] animals`字段，然后将一个狗，猫和一个长颈鹿的实例放进去，在序列化之后，你会有3个Animal的实例。

解决这种限制的一种方法就是认识到它仅适用于内联序列化的自定义类，引用其他UnityEngine.Object被序列化为实际引用，对于这些对象，多态性确实有效。你需要创建一个ScriptableObject的派生类，或者另一个MonoBehaviour派生类，然后引用它。这样做的负面效果就是，你需要在某处存储MonoBehaviour或ScriptableObject，并且不能将其很好地内联序列化。

#### 想要序列化Unity序列化器不支持的类型，该怎么做

在许多情况下，最好的方法是使用序列化回调，它们允许在序列化程序从字段中读取数据之前和完成对字段的写入之后通知你，你可以使用它在运行时拥有与实际序列化时不同的难以序列化数据的表示形式。你需要在Unity序列化之前，将你的数据转换成Unity能够理解的形式，你也可以用它来将序列化的数据转回需要在运行时使用的数据。

假如你想要一个树状的数据结构，如果你让Unity直接序列化它，”不支持null“的限制将会造成数据流变得非常大，导致许多系统的性能下降。

```C#
using UnityEngine;
using System.Collections.Generic;
using System;

public class VerySlowBehaviourDoNotDoThis : MonoBehaviour
{
    [Serializable]
    public class Node
    {
        public string interestingValue = "value";
        
        //The field below is what makes the serialization data become huge because
        //it introduces a 'class cycle'.
		public List<Node> children = new List<Node>();
    }
    
    //this gets serialized
    public Node root = new Node();
    
    void OnGUI()
    {
        Display (root);
    }
    
    void Display(Node node)
    {
        GUILayout.Label ("Value: ");
        node.interestingValue = GUILayout.TextField(node.interestingValue, GUILayout.Width(200));
        
        GUILayout.BeginHorizontal ();
        GUILayout.Space (20);
        GUILayout.BeginVertical ();
        
        foreach (var child in node.children)
            Display (child);
            
        if (GUILayout.Button ("Add child"))
            node.children.Add (new Node ());
            
        GUILayout.EndVertical ();
        GUILayout.EndHorizontal ();
    }
}

Instead, you tell Unity not to serialize the tree directly, and you make a seperate field to store the tree in a serialized format, suited for Unity’s serializer:

using UnityEngine;
using System.Collections.Generic;
using System;

public class BehaviourWithTree : MonoBehaviour, ISerializationCallbackReceiver
{
    //node class that is used at runtime
    public class Node
    {
        public string interestingValue = "value";
        public List<Node> children = new List<Node>();
    }

    //node class that we will use for serialization
    [Serializable]
    public struct SerializableNode
    {
        public string interestingValue;
        public int childCount;
        public int indexOfFirstChild;
    }
    
    //the root of what we use at runtime. not serialized.
    Node root = new Node();
    
    //the field we give unity to serialize.
    public List<SerializableNode> serializedNodes;
    
    public void OnBeforeSerialize()
    {
        //unity is about to read the serializedNodes field's contents. lets make sure
        //we write out the correct data into that field "just in time".
        serializedNodes.Clear();
        AddNodeToSerializedNodes(root);
    }
    
    void AddNodeToSerializedNodes(Node n)
    {
        var serializedNode = new SerializableNode () {
            interestingValue = n.interestingValue,
            childCount = n.children.Count,
            indexOfFirstChild = serializedNodes.Count+1
        };
        
        serializedNodes.Add (serializedNode);
        foreach (var child in n.children)
            AddNodeToSerializedNodes (child);
    }
    
    public void OnAfterDeserialize()
    {
        //Unity has just written new data into the serializedNodes field.
        //let's populate our actual runtime data with those new values.
        
        if (serializedNodes.Count > 0)
            root = ReadNodeFromSerializedNodes (0);
        else
            root = new Node ();
    }
    
    Node ReadNodeFromSerializedNodes(int index)
    {
        var serializedNode = serializedNodes [index];
        var children = new List<Node> ();
        for(int i=0; i!= serializedNode.childCount; i++)
            children.Add(ReadNodeFromSerializedNodes(serializedNode.indexOfFirstChild + i));
            
        return new Node() {
            interestingValue = serializedNode.interestingValue,
            children = children
        };
    }
    
    void OnGUI()
    {
        Display (root);
    }
    
    void Display(Node node)
    {
        GUILayout.Label ("Value: ");
        node.interestingValue = GUILayout.TextField(node.interestingValue, GUILayout.Width(200));
        
        GUILayout.BeginHorizontal ();
        GUILayout.Space (20);
        GUILayout.BeginVertical ();
        
        foreach (var child in node.children)
            Display (child);
            
        if (GUILayout.Button ("Add child"))
            node.children.Add (new Node ());
            
        GUILayout.EndVertical ();
        GUILayout.EndHorizontal ();
    }
}
```

注意，序列化器，包括这些回调函数，通常不会在主线程运行，所以你会非常受限于无法调用Unity API。

[1] I lied, the correct answer isn't actually 729. This is because in the very very old days before we had this 7 level depth limit, Unity would just endless loop, and then run out of memory if you created a script like the Trouble one I just wrote. Our very first fix for that 5 years ago was to just not serialize fieldtypes that were of the same type as the class itself. Obviously, this was not the most robust fix, as it's easy to create a cycle using Trouble1->Trouble2->Trouble1->Trouble2 class. So shortly afterwards we actually implemented the 7 level depth limit to catch those cases too. For the point I'm trying to make however it doesn't matter, what matters is that you realize that if there is a cycle you are in trouble.