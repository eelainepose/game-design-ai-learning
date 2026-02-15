# Game Programming Patterns 读书笔记

> 来源: https://github.com/munificent/game-programming-patterns  
> 标签: #game-design #patterns

---

## 核心模式速查

### 1. Command 模式 ⭐⭐⭐⭐⭐

**问题**: 如何解耦输入处理与游戏逻辑？如何实现撤销/回放？

**解决方案**: 将请求封装为对象

```cpp
class Command {
public:
    virtual ~Command() {}
    virtual void execute(GameActor& actor) = 0;
    virtual void undo() = 0;
};

// 具体命令
class JumpCommand : public Command {
public:
    void execute(GameActor& actor) override {
        actor.jump();
    }
    void undo() override {
        // 撤销跳跃
    }
};
```

**游戏应用**:
- ✅ 输入映射：键盘/手柄 → 命令对象
- ✅ 撤销系统：命令历史栈
- ✅ 回放系统：记录命令序列
- ✅ AI 控制：AI 生成命令而非直接操作角色

---

### 2. Observer 模式 ⭐⭐⭐⭐⭐

**问题**: 一个对象状态变化，如何通知多个依赖者？

**解决方案**: 订阅-发布机制

```cpp
class Subject {
    vector<Observer*> observers;
public:
    void addObserver(Observer* o) { observers.push_back(o); }
    void notify() {
        for (auto o : observers) o->onNotify();
    }
};
```

**游戏应用**:
- ✅ 成就系统：监听击杀、收集事件
- ✅ 任务系统：条件变化自动更新
- ✅ UI 更新：数据变化刷新界面
- ✅ 音效触发：特定事件播放音效

---

### 3. State 模式 ⭐⭐⭐⭐⭐

**问题**: 对象行为随状态变化，如何避免大量 if-else？

**解决方案**: 将状态封装为类

```cpp
class HeroState {
public:
    virtual ~HeroState() {}
    virtual void handleInput(Hero& hero, Input input) = 0;
    virtual void update(Hero& hero) = 0;
};

class StandingState : public HeroState {
public:
    void handleInput(Hero& hero, Input input) override {
        if (input == PRESS_B) {
            hero.jump();
            hero.setState(new JumpingState());
        }
    }
};
```

**游戏应用**:
- ✅ 角色状态：Idle → Walk → Attack → Hurt
- ✅ 游戏流程：Menu → Playing → Paused → GameOver
- ✅ AI 状态：Patrol → Chase → Attack → Flee

---

### 4. Object Pool 模式 ⭐⭐⭐⭐

**问题**: 频繁创建/销毁对象导致性能问题（如子弹、粒子）

**解决方案**: 预分配对象池，重复使用

```cpp
class BulletPool {
    vector<Bullet*> available;
    vector<Bullet*> inUse;
public:
    Bullet* acquire() {
        if (available.empty()) {
            // 可选：扩展池或等待
            return nullptr;
        }
        Bullet* bullet = available.back();
        available.pop_back();
        inUse.push_back(bullet);
        return bullet;
    }
    
    void release(Bullet* bullet) {
        inUse.erase(remove(inUse.begin(), inUse.end(), bullet), inUse.end());
        available.push_back(bullet);
    }
};
```

**游戏应用**:
- ✅ 子弹系统：射击游戏大量子弹
- ✅ 粒子效果：爆炸、魔法特效
- ✅ 敌人刷新：波次敌人重用对象

---

## 学习检查清单

- [ ] 理解每个模式的意图和适用场景
- [ ] 能在自己的项目中识别使用机会
- [ ] 掌握 C++ 实现方式
- [ ] 了解模式的 trade-offs（何时不用）

---

## 相关笔记

- [[数值平衡方法论]] - 结合 State 模式做战斗状态数值
- [[WaveFunctionCollapse]] - 使用 Object Pool 管理瓦片对象

---

*Created: 2026-02-15*  
*Status: 🚧 持续更新*
