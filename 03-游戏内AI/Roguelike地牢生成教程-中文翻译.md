# Roguelike 地牢生成教程（中文翻译）

> 来源：https://rogueliketutorials.com/tutorials/tcod/2019/part-3/  
> 标签: #procedural-generation #roguelike #tutorial

---

## 概述

欢迎来到 Roguelike 教程修订版！在本教程中，我们将向拥有真正可运行的游戏迈出**非常**重要的一步：**创建程序化生成的地牢**！

---

## 核心概念

### 1. 地牢生成的基本思路

**关键原则**：从完全封闭的地图开始，然后"挖掘"出房间和通道

```python
# 初始化：所有瓦片都设为阻挡（True）
tiles = [[Tile(True) for y in range(self.height)] for x in range(self.width)]
```

为什么这样做？
- 大多数地牢生成算法都采用这种方式
- 先创建封闭空间，再"雕刻"出可行走区域
- 比从开放空间开始更容易控制

---

### 2. 房间类（Rect）

创建一个矩形类来表示房间：

```python
# map_objects/rectangle.py
class Rect:
    def __init__(self, x, y, w, h):
        self.x1 = x          # 左上角 X
        self.y1 = y          # 左上角 Y
        self.x2 = x + w      # 右下角 X
        self.y2 = y + h      # 右下角 Y
```

**为什么需要 +1 偏移？**

为了保持房间之间的墙壁：

```
# 不加 +1 的问题：
0 1 2 3 4 5 6 7 8 9
# # # # # # # # # #
# . . . . . . . . #   ← 房间1 (1-7)
# . . . . . . . . #
# # # # # # # # # #
# . . . . . . . . #   ← 房间2 (紧邻，无墙！)
# . . . . . . . . #
# # # # # # # # # #

# 加 +1 后的正确做法：
0 1 2 3 4 5 6 7
# # # # # # # #
# # # # # # # #
# # . . . . # #   ← 内部可行走
# # . . . . # #
# # . . . . # #
# # # # # # # #
```

通过 `x1 + 1` 和 `y1 + 1`，我们确保房间周围有一圈墙壁。

---

### 3. 创建房间

```python
def create_room(self, room):
    """在矩形区域内创建可行走的房间"""
    for x in range(room.x1 + 1, room.x2):      # +1 保持墙壁
        for y in range(room.y1 + 1, room.y2):  # +1 保持墙壁
            self.tiles[x][y].blocked = False
            self.tiles[x][y].block_sight = False
```

---

### 4. 创建通道（隧道）

连接两个房间需要水平隧道和垂直隧道：

```python
def create_h_tunnel(self, x1, x2, y):
    """创建水平隧道"""
    for x in range(min(x1, x2), max(x1, x2) + 1):
        self.tiles[x][y].blocked = False
        self.tiles[x][y].block_sight = False

def create_v_tunnel(self, y1, y2, x):
    """创建垂直隧道"""
    for y in range(min(y1, y2), max(y1, y2) + 1):
        self.tiles[x][y].blocked = False
        self.tiles[x][y].block_sight = False
```

---

### 5. 完整的地图生成算法

```python
def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player):
    """生成随机地牢地图"""
    
    rooms = []
    num_rooms = 0
    
    for r in range(max_rooms):
        # 随机生成房间尺寸
        w = randint(room_min_size, room_max_size)
        h = randint(room_min_size, room_max_size)
        
        # 随机生成房间位置
        x = randint(0, map_width - w - 1)
        y = randint(0, map_height - h - 1)
        
        new_room = Rect(x, y, w, h)
        
        # 检查是否与其他房间重叠
        failed = False
        for other_room in rooms:
            if new_room.intersect(other_room):
                failed = True
                break
        
        if not failed:
            # 创建房间
            self.create_room(new_room)
            
            # 获取房间中心坐标
            (new_x, new_y) = new_room.center()
            
            if num_rooms == 0:
                # 第一个房间：放置玩家
                player.x = new_x
                player.y = new_y
            else:
                # 后续房间：用隧道连接
                (prev_x, prev_y) = rooms[num_rooms-1].center()
                
                # 随机选择连接顺序
                if randint(0, 1) == 1:
                    # 先水平后垂直
                    self.create_h_tunnel(prev_x, new_x, prev_y)
                    self.create_v_tunnel(prev_y, new_y, new_x)
                else:
                    # 先垂直后水平
                    self.create_v_tunnel(prev_y, new_y, prev_x)
                    self.create_h_tunnel(prev_x, new_x, new_y)
            
            rooms.append(new_room)
            num_rooms += 1
```

---

## 完整代码示例

### rectangle.py
```python
class Rect:
    def __init__(self, x, y, w, h):
        self.x1 = x
        self.y1 = y
        self.x2 = x + w
        self.y2 = y + h

    def center(self):
        center_x = int((self.x1 + self.x2) / 2)
        center_y = int((self.y1 + self.y2) / 2)
        return (center_x, center_y)

    def intersect(self, other):
        """检查两个房间是否重叠"""
        return (self.x1 <= other.x2 and self.x2 >= other.x1 and
                self.y1 <= other.y2 and self.y2 >= other.y1)
```

---

## 关键设计决策

### 1. 房间不重叠
- 使用 `intersect()` 方法检查碰撞
- 如果重叠就放弃这个房间，尝试下一个

### 2. 隧道连接策略
- 从上一个房间的中心连接到新房间的中心
- 随机决定先水平还是先垂直
- 这样会产生自然的"L"形通道

### 3. 玩家起始位置
- 放在第一个生成的房间中心
- 确保玩家不会被困在墙里

---

## 扩展思路

### 可以添加的功能：
1. **敌人/NPC 生成**：在每个房间随机放置
2. **物品生成**：在房间内放置宝箱、药水
3. **门**：在房间入口处添加门
4. **不同类型的房间**：大厅、走廊、密室
5. **预定义房间模板**：从预设中随机选择

### 与其他算法结合：
- **WFC (WaveFunctionCollapse)**：用于更复杂的地图生成
- **柏林噪声**：用于自然地形
- **细胞自动机**：用于洞穴式地图

---

## 学习检查清单

- [ ] 理解为什么要从封闭地图开始
- [ ] 理解 +1 偏移的作用
- [ ] 能实现房间类（Rect）
- [ ] 能实现隧道连接逻辑
- [ ] 能处理房间重叠检测
- [ ] 能放置玩家到第一个房间

---

## 参考资源

- [原文教程](https://rogueliketutorials.com/tutorials/tcod/2019/part-3/)
- [你的 bracket-lib 项目](https://github.com/amethyst/bracket-lib)
- [PythonPlantsVsZombies](https://github.com/marblexu/PythonPlantsVsZombies) - 游戏实现参考

---

*翻译整理: 2026-02-15*  
*原文作者: Roguelike Tutorials*
