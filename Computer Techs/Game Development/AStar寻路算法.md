---
tags: [GamesDevelopement, Algorithm, Computer]
date: 2023-08-31 18:20
---

文章参考自[Red Blob Games](https://www.redblobgames.com/)的[Introduction to the A* Algorithm](https://www.redblobgames.com/pathfinding/a-star/introduction.html)。
# 引入

在游戏中我们经常要寻找从某个位置到目标位置的一条路径，这条路径不仅要是最短的，还要花费最少的时间。

为了寻找这样的路径，我们就需要将游戏中的地图抽象成计算机中的 *图* 这种数据结构，再使用图的搜索算法。

A\* 是图搜索算法中比较受欢迎的一种。但我们先从一种最简单的图搜索算法—— *Breadth First Search* 开始讲起，逐步演变到最终的AStar算法。

# 表示地图

学习一种算法前需要做的第一件事是理解数据，输入是什么，输出是什么？

**Input：** 包括A\*在内的图搜索算法，都是以图作为输入。一张图由一组位置（节点）和它们之间的连接（边）组成。

![[20230831185630.png]]

图搜索算法并不清楚像上面这样地图实际的样子，也不知道某个位置在室内还是室外，或者某个区域的面积有多大，它只能从输入的图中获取信息。

**Output：** A\*找到的路径由图的节点和边组成，它只会告诉你从一个位置移动到另一个位置的整条路径，但不会告诉你如何进行移动。

**Tradeoffs：** 对于任何给定的游戏地图，有许多不同的制作路径查找图的方式，比如点和边、网格等等。路径查找图不必和游戏地图使用相同格式的图，比如网格地图可以使用非网格图进行寻路，反之亦然。

# 不同的算法

- **广度优先搜索**：在所有方向上平等地探索，不仅适用于常规路径查找，还适用于程序地图生成、流场寻路、距离图和其他类型的地图分析。
- **Dijkstra**：不是平等地探索所有可能的路径，而是倾向于低成本路径。可以分配更低的成本来鼓励在道路上移动，分配更高的成本来避开敌人，等等。
- **A\***：由Dijkstra改进而来，针对单个目的地进行优化。A* 查找指向一个位置或多个位置中最近的路径，它优先考虑似乎更接近目标的路径。

# 广度优先搜索

这几种图搜索算法的核心思想都是跟踪一个被称为 *frontier(边界)* 的扩展环，这个过程在网格上也被称为“洪水填充”。

![[20230831193429.png]]

![[20230831193443.png]]

从起点开始，每一轮循环都访问当前位置的相邻点即周围的四个位置。

如果相邻点没有被访问过，则将其加入待访问的队列中，并加入到已访问的集合中，避免重复访问。

循环直到待访问队列为空即访问完图中所有位置时结束。

下面使用Python代码表示上述过程：

```Python
frontier = Queue()
frontier.put(start)
reached = set()
reached.add(start)

while not frontier.empty():
	current = frontier.get()
	for next in graph.neighbors(current):
	    if next not in reached:
	        frontier.put(next)
	        reached.add(next)
```

上述循环是这几种图搜索算法的核心。但是要如何找到最短路径？

这个循环本身并不能构造出最短路径，它仅仅是将图上的所有位置遍历了一次。顺便一提，广度优先搜索并不仅仅用作寻路，还可以用于距离地图、随机地图的生成等等。在这里因为是用作寻路，所以接下来我们修改此循环来跟踪记录到达过的每个位置。

首先在循环外面声明一个字典 *cameFrom* ，键和值都是图中的位置，就用它替换 *reached* 集合，来记录我们遍历过的每一个位置。

```Python
frontier = Queue()
frontier.put(start)
cameFrom = dict() # path A->B is stored as cameFrom[B] == A
cameFrom[start] = None

while not frontier.empty():
	current = frontier.get()
	for next in graph.neighbors(current):
		if next not in cameFrom:
			frontier.put(next)
			cameFrom[next] = current
```

这样，我们通过 *cameFrom* 字典就可以知道某个位置来自何处，从终点一步步回溯，最终就能回到起点。

下面是回溯的逻辑：

```Python
current = goal 
path = []
while current != start: 
	path.append(current)
	current = cameFrom[current]
path.append(start) # optional
path.reverse() # optional
```

以上就是最简单的寻路算法，它不仅适用于此处所示的网格，也适用于任何类型的图形结构。

在地牢中，图形位置可以是房间，图形边缘是它们之间的门口。

在平台游戏中，图形位置可以是位置，图形边缘可能是可能的动作，例如向左移动、向右移动、向上跳跃、向下跳跃。

通常，将图形视为更改状态的状态和操作。

# 提前退出

通常我们不需要搜索完整个图，而是在一旦找到终点时就退出循环。

原始的搜索效果和使用提前退出的搜索效果对比如下：

![[20230831213947.png]]

代码修改如下：

```Python
frontier = Queue()
frontier.put(start)
cameFrom = dict()
cameFrom[start] = None

while not frontier.empty():
	current = frontier.get()
	
	if current == goal:
		break         
		
	for next in graph.neighbors(current):
		if next not in cameFrom:
			frontier.put(next)
	        cameFrom[next] = current
```

# 移动成本

在某些寻路场景中，选择不同路径进行移动所花费的成本是不同的。

例如在《文明》中，穿过平原或沙漠可能需要 1 个移动点，但穿过森林或丘陵可能需要 5 个移动点。又例如在网格的对角线上移动的成本高于轴向移动……

Dijkstra算法便能支持这种包含不同移动成本的寻路情景。

为了能够跟踪移动成本，我们需要添加一个新的字典来记录从起始位置开始的总移动成本。

其次，需要将原来的普通队列替换成优先级队列。

并且，一个位置可能会被多次访问，但成本不同。改变后的逻辑是：如果该位置从未到达过，或者是新路径的移动成本优于之前最佳路径的成本，就将位置添加进 *frontier* 中。

修改后的代码如下：

```Python
frontier = PriorityQueue()
frontier.put(start, 0)
cameFrom = dict()
costSoFar = dict()
cameFrom[start] = None
costSoFar[start] = 0

while not frontier.empty():
	current = frontier.get()
	
	if current == goal:
	    break
	    
	for next in graph.neighbors(current):
	    newCost = costSoFar[current] + graph.cost(current, next)
	    if next not in costSoFar or newCost < costSoFar[next]:
	        costSoFar[next] = newCost
	        priority = newCost
	        frontier.put(next, priority)
	        cameFrom[next] = current
```

# 启发式搜索

广度优先搜索和Dijkstra算法都是让探索的边界向所有方向扩散，这对于在一个图中搜索从一个位置到任意多个位置的路径来说确实是一个合理的做法。

但是，我们通常面临的是只搜寻一个目标点的寻路情景，这种情况下只需要让边界朝向目标扩散即可。

首先定义一个启发式函数，这个函数能够告诉我们现在离目标还有多远。

```Python
def heuristic(a, b):
	# Manhattan distance on a square grid
	return abs(a.x - b.x) + abs(a.y - b.y)
```

在Dijkstra算法中，我们使用了实际距离作为优先级队列的指标。

然而在 *贪婪最佳优先搜索* 中，我们会使用到目标位置的估计距离进行优先级队列排序。于是，和目标最近的位置会被优先遍历。

```Python
frontier = PriorityQueue()
frontier.put(start, 0)
cameFrom = dict()
cameFrom[start] = None

while not frontier.empty():
	current = frontier.get()
	
	if current == goal:
	    break
	    
	for next in graph.neighbors(current):
		if next not in cameFrom:
			priority = heuristic(goal, next)
			frontier.put(next, priority)
			cameFrom[next] = current
```

在简单的场景中，贪婪最佳优先搜索的速度非常快，远超于Dijkstra算法。

![[Pasted image 20230901102738.png]]

但是在较复杂场景中，贪婪最佳优先搜索得到的路径就可能不是最短路径了，不过速度依然很快。

![[Pasted image 20230901102928.png]]

# A\*算法

Dijkstra算法对于寻找最短路径来说表现很好，但它浪费了太多时间在探索不可能出现在最短路径的位置上；相反，贪婪最佳优先算法大大节省了在不必要的方向上的搜索时间，但它最终找到的不一定是最短路径。

A\*算法同时使用了从起点开始的实际距离和到目标点的估计距离，作为优先队列排序的依据。代码和Dijkstra算法非常相似：

```Python
frontier = PriorityQueue()
frontier.put(start, 0)
cameFrom = dict()
costSoFar = dict()
cameFrom[start] = None
costSoFar[start] = 0

while not frontier.empty():
	current = frontier.get()
	
	if current == goal:
	    break
	    
	for next in graph.neighbors(current):
	    newCost = costSoFar[current] + graph.cost(current, next)
	    if next not in costSoFar or newCost < costSoFar[next]:
	        costSoFar[next] = new_cost
	        priority = newCost + heuristic(goal, next)
	        frontier.put(next, priority)
	        cameFrom[next] = current
```

对比[[#^e15b49|Dijkstra]]，[[#^4594b6|贪婪最佳优先]]，[[#^d8b49f|A*]]。

![[Pasted image 20230901104634.png]]

可见，A\*不仅避免了在过多的错误的方向上探索，也找到了最短路径，无疑是一种相对于Dijkstra和贪婪最佳优先搜索两全其美的算法。

# 其他

在游戏中如何选择这几种寻路算法呢？

- 如果要搜索地图上所有的路径，使用广度优先搜索（移动成本相同时）或Dijkstra算法（移动成本不同时）。
- 如果要搜索到某个位置或几个目标中最接近的路径，使用贪婪最佳优先搜索或A\*（大多数情况使用）。

# C#实现

## 位置与无权图

```CS
public struct Location
{
    // Implementation notes: I am using the default Equals but it can
    // be slow. You'll probably want to override both Equals and
    // GetHashCode in a real project.
    
    public readonly int x, y;
    public Location(int x, int y)
    {
        this.x = x;
        this.y = y;
    }
}

public class Graph<Location>
{
    // NameValueCollection would be a reasonable alternative here, if
    // you're always using string location types
    public Dictionary<Location, Location[]> edges
	    = new Dictionary<Location, Location[]>();
	
    public Location[] Neighbors(Location id)
    {
	    return edges[id];
    }
};
```
## 广度优先搜索

```CS
class BreadthFirstSearch
{
    static void Search(Graph<string> graph, string start)
    {
        var frontier = new Queue<string>();
        frontier.Enqueue(start);
        
        var reached = new HashSet<string>();
        reached.Add(start);
        
        while (frontier.Count > 0)
        {
            var current = frontier.Dequeue();
            
            Console.WriteLine("Visiting {0}", current);
            foreach (var next in graph.Neighbors(current))
            {
                if (!reached.Contains(next)) {
                    frontier.Enqueue(next);
                    reached.Add(next);
                }
            }
        }
    }
    
    static void Main()
    {
        Graph<string> g = new Graph<string>();
        g.edges = new Dictionary<string, string[]>
            {
            { "A", new [] { "B" } },
            { "B", new [] { "A", "C", "D" } },
            { "C", new [] { "A" } },
            { "D", new [] { "E", "A" } },
            { "E", new [] { "B" } }
        };
        
        Search(g, "A");
    }
}
```

## 有权图与网格

```CS
// A* needs only a WeightedGraph and a location type L, and does *not*
// have to be a grid. However, in the example code I am using a grid.
public interface WeightedGraph<L>
{
    double Cost(Location a, Location b);
    IEnumerable<Location> Neighbors(Location id);
}

public class SquareGrid : WeightedGraph<Location>
{
    // Implementation notes: I made the fields public for convenience,
    // but in a real project you'll probably want to follow standard
    // style and make them private.
    
    public static readonly Location[] DIRS = new []
        {
            new Location(1, 0),
            new Location(0, -1),
            new Location(-1, 0),
            new Location(0, 1)
        };
        
    public int width, height;
    public HashSet<Location> walls = new HashSet<Location>();
    public HashSet<Location> forests = new HashSet<Location>();
    
    public SquareGrid(int width, int height)
    {
        this.width = width;
        this.height = height;
    }
    
    public bool InBounds(Location id)
    {
        return 0 <= id.x && id.x < width
            && 0 <= id.y && id.y < height;
    }
    
    public bool Passable(Location id)
    {
        return !walls.Contains(id);
    }
    
    public double Cost(Location a, Location b)
    {
        return forests.Contains(b) ? 5 : 1;
    }
    
    public IEnumerable<Location> Neighbors(Location id)
    {
        foreach (var dir in DIRS) {
            Location next = new Location(id.x + dir.x, id.y + dir.y);
            if (InBounds(next) && Passable(next)) {
                yield return next;
            }
        }
    }
}
```

## 优先队列

```CS
// When I wrote this code in 2015, C# didn't have a PriorityQueue<>
// class. It was added in 2020; see
// https://github.com/dotnet/runtime/issues/14032
//
// This is a placeholder PriorityQueue<> that runs inefficiently but
// will allow you to run the sample code on older C# implementations.
//
// If you're using a version of C# that doesn't have PriorityQueue<>,
// consider using one of these fast libraries instead of my slow
// placeholder:
//
// * https://github.com/BlueRaja/High-Speed-Priority-Queue-for-C-Sharp
// * https://visualstudiomagazine.com/articles/2012/11/01/priority-queues-with-c.aspx
// * http://xfleury.github.io/graphsearch.html
// * http://stackoverflow.com/questions/102398/priority-queue-in-net
public class PriorityQueue<TElement, TPriority>
{
    private List<Tuple<TElement, TPriority>> elements = new List<Tuple<TElement, TPriority>>();
    
    public int Count
    {
        get { return elements.Count; }
    }
    
    public void Enqueue(TElement item, TPriority priority)
    {
        elements.Add(Tuple.Create(item, priority));
    }
    
    public TElement Dequeue()
    {
        Comparer<TPriority> comparer = Comparer<TPriority>.Default;
        int bestIndex = 0;
        
        for (int i = 0; i < elements.Count; i++) {
            if (comparer.Compare(elements[i].Item2, elements[bestIndex].Item2) < 0) {
                bestIndex = i;
            }
        }
        
        TElement bestItem = elements[bestIndex].Item1;
        elements.RemoveAt(bestIndex);
        return bestItem;
    }
}
```

## A*

```CS
/* NOTE about types: in the main article, in the Python code I just
 * use numbers for costs, heuristics, and priorities. In the C++ code
 * I use a typedef for this, because you might want int or double or
 * another type. In this C# code I use double for costs, heuristics,
 * and priorities. You can use an int if you know your values are
 * always integers, and you can use a smaller size number if you know
 * the values are always small. */

public class AStarSearch
{
    public Dictionary<Location, Location> cameFrom
        = new Dictionary<Location, Location>();
    public Dictionary<Location, double> costSoFar
        = new Dictionary<Location, double>();
        
    // Note: a generic version of A* would abstract over Location and
    // also Heuristic
    static public double Heuristic(Location a, Location b)
    {
        return Math.Abs(a.x - b.x) + Math.Abs(a.y - b.y);
    }
    
    public AStarSearch(WeightedGraph<Location> graph, Location start, Location goal)
    {
        var frontier = new PriorityQueue<Location, double>();
        frontier.Enqueue(start, 0);
        
        cameFrom[start] = start;
        costSoFar[start] = 0;
        
        while (frontier.Count > 0)
        {
            var current = frontier.Dequeue();
            
            if (current.Equals(goal))
            {
                break;
            }
            
            foreach (var next in graph.Neighbors(current))
            {
                double newCost = costSoFar[current]
                    + graph.Cost(current, next);
                if (!costSoFar.ContainsKey(next)
                    || newCost < costSoFar[next])
                {
                    costSoFar[next] = newCost;
                    double priority = newCost + Heuristic(next, goal);
                    frontier.Enqueue(next, priority);
                    cameFrom[next] = current;
                }
            }
        }
    }
}

public class RunAStar
{
    static void DrawGrid(SquareGrid grid, AStarSearch astar) {
        // Print out the cameFrom array
        for (var y = 0; y < 10; y++)
        {
            for (var x = 0; x < 10; x++)
            {
                Location id = new Location(x, y);
                Location ptr = id;
                if (!astar.cameFrom.TryGetValue(id, out ptr))
                {
                    ptr = id;
                }
                if (grid.walls.Contains(id)) { Console.Write("##"); }
                else if (ptr.x == x+1) { Console.Write("\u2192 "); }
                else if (ptr.x == x-1) { Console.Write("\u2190 "); }
                else if (ptr.y == y+1) { Console.Write("\u2193 "); }
                else if (ptr.y == y-1) { Console.Write("\u2191 "); }
                else { Console.Write("* "); }
            }
            Console.WriteLine();
        }
    }
    
    static void Main()
    {
        // Make "diagram 4" from main article
        var grid = new SquareGrid(10, 10);
        for (var x = 1; x < 4; x++)
        {
            for (var y = 7; y < 9; y++)
            {
                grid.walls.Add(new Location(x, y));
            }
        }
        grid.forests = new HashSet<Location>
            {
                new Location(3, 4), new Location(3, 5),
                new Location(4, 1), new Location(4, 2),
                new Location(4, 3), new Location(4, 4),
                new Location(4, 5), new Location(4, 6),
                new Location(4, 7), new Location(4, 8),
                new Location(5, 1), new Location(5, 2),
                new Location(5, 3), new Location(5, 4),
                new Location(5, 5), new Location(5, 6),
                new Location(5, 7), new Location(5, 8),
                new Location(6, 2), new Location(6, 3),
                new Location(6, 4), new Location(6, 5),
                new Location(6, 6), new Location(6, 7),
                new Location(7, 3), new Location(7, 4),
                new Location(7, 5)
            };
            
        // Run A*
        var astar = new AStarSearch(grid, new Location(1, 4), new Location(8, 5));
        
        DrawGrid(grid, astar);
    }
}
```