# Minecraft 模型 UV 与 JSON 继承机制

> 本文档记录 Minecraft 模型 JSON 中 UV 坐标的概念、Create 原版大小齿轮包覆箱的 UV 区域设定差异、以及模型 JSON 继承机制的核心知识点。

---

## 一、UV 是什么

UV 是 3D 模型面（face）与 2D 贴图（texture）之间的映射坐标。每个面的 `uv` 字段定义了该面从贴图上取哪个矩形区域来显示。

### 格式

```json
"uv": [x1, y1, x2, y2]
```

- `(x1, y1)` 是贴图上的左上角坐标
- `(x2, y2)` 是贴图上的右下角坐标
- 该面将显示贴图上 `[x1,y1]` 到 `[x2,y2]` 的矩形区域

### 坐标系

Minecraft 方块模型 UV 坐标的原点 `(0,0)` 在贴图的**左上角**，x 向右增大，y 向下增大。这与 Minecraft 贴图 PNG 文件的像素布局一致。

**UV 数值范围固定为 0–16，与贴图实际尺寸无关。** 16 代表"贴图的 100% 宽度/高度"，而不是像素上限。渲染时引擎按比例换算：

```
实际像素 = UV 值 / 16 × 贴图尺寸
```

| 贴图尺寸 | 1 个 UV 单位 = ? 像素 | UV 16 = ? 像素 |
|----------|---------------------|----------------|
| 16×16 | 1 像素 | 16 像素（满宽） |
| 32×32 | 2 像素 | 32 像素（满宽） |

例如原版大齿轮贴图是 32×32，UV `[8,13,16,16]` 实际取的是像素 `(16,26)` 到 `(32,32)`——正好是右下角区域。同样写 16，在 16×16 贴图上是满宽 16 像素，在 32×32 贴图上是满宽 32 像素。

这意味着：**同一套 UV 值用在不同尺寸的贴图上，实际选取的区域不同。** 这正是本项目大齿轮换贴图后 UV 不适用的根本原因。

### 与 3D 面的关系

每个 3D 立方体元素（element）有 6 个面（north/east/south/west/up/down），每个面可以独立设置 UV 和贴图变量。面的 3D 尺寸由 `from`/`to` 决定，UV 只决定"从贴图上取哪块"来贴到这个面上。

例如一个 `from: [0,0,0]` `to: [16,6,16]` 的立方体（16×6×16），其 north 面是 16 宽 × 6 高的 3D 面，UV `[0,10,16,16]` 表示从贴图的 `(0,10)` 到 `(16,16)` 区域（16 宽 × 6 高）贴上去——恰好匹配面的宽高比。

### UV 可以与面尺寸不等

UV 区域不必与 3D 面尺寸 1:1 对应。如果 UV 区域比面大或小，贴图会被拉伸或压缩以铺满整个面。例如 3D 面 16 宽 × 6 高，UV `[0,10,16,16]`（16×16 贴图上取 16 宽 × 6 高），1:1 映射无拉伸；如果 UV 改为 `[0,13,16,16]`（16×16 贴图上取 16 宽 × 3 高），贴图会被纵向拉伸 3 行铺满 6px 高的面。

---

## 二、Create 原版大小齿轮包覆箱的 UV 区域设定

Create 原版对大小齿轮包覆箱使用了不同的侧面贴图和不同的 UV 区域设定。这是理解本项目为何需要调整 UV 的关键。

### 小齿轮（encased_cogwheel）

**贴图**：`andesite_encased_cogwheel_side.png`（16×16）

小齿轮侧面贴图被纵向划分为 3 个区域，对应机壳的 3 层结构：

```
贴图 (16×16)
┌──────────┐ y=0
│  上壳    │ 6px  ← UV: [0, 0, 16, 6]
├──────────┤ y=6
│ 中间齿轮 │ 4px  ← UV: [0, 6, 16, 10]
├──────────┤ y=10
│  下壳    │ 6px  ← UV: [0, 10, 16, 16]
└──────────┘ y=16
```

- 横向 0→16 拉满整个贴图宽度
- 纵向三段：上壳 y0-6、中间齿轮 y6-10、下壳 y10-16
- 每段高度恰好等于对应 3D 面的高度（6px/4px/6px），1:1 映射无拉伸

### 大齿轮（encased_large_cogwheel）

**贴图**：`andesite_encased_cogwheel_side_connected.png`（32×32 CT 精灵表）

大齿轮原版没有使用独立的侧面贴图，而是复用了 32×32 的 CT 连接精灵表（`side_connected`），从其右下角区域取一小块来用：

```
32×32 精灵表
┌────────────┬────────────┐
│            │            │
│  左上象限   │  右上象限   │
│ (CT 单块    │ (CT 单块    │
│  连接状态)  │  连接状态)  │
├────────────┼────────────┤
│            │            │
│  左下象限   │  右下象限   │
│ (CT 连接    │ (大齿轮    │
│  状态)      │  使用区域)  │
│            │            │
└────────────┴────────────┘
              ↑
        大齿轮取右下角区域的局部
```

原版大齿轮 UV 设定（各变体）：

| 变体 | 下壳侧面 UV | 上壳侧面 UV |
|------|------------|------------|
| block / item | `[8, 13, 16, 16]` | `[8, 8, 16, 11]` |
| block_top | `[8, 13, 16, 16]` | `[8, 8.5, 16, 11]` |
| block_bottom | `[8, 13, 16, 15.5]` | `[8, 8, 16, 11]` |
| block_top_bottom | `[8, 13, 16, 15.5]` | `[8, 8.5, 16, 11]` |

- 横向只取 `x8→16`，不拉满
- 纵向只取约 3 行（下壳 y13-16 或 y13-15.5、上壳 y8-11 或 y8.5-11），远小于 6 行的面高度
- 贴图会被纵向拉伸以铺满 6px 高的 3D 面

这是因为 32×32 精灵表中预留给大齿轮的区域只有约 3 行高度，Create 原版接受这种拉伸。

> **注意**：这里 UV 数值虽然是 0–16 范围，但因为贴图是 32×32，1 个 UV 单位 = 2 像素。所以 UV `[8,13,16,16]` 实际取的是像素 `(16,26)` 到 `(32,32)`，即 32×32 贴图的右下角 16×6 像素区域。

### 本项目的改造

本项目为大齿轮制作了 16×16 独立侧面贴图（`large_*_encased_cogwheel_side.png`），不再使用 32×32 精灵表。原版 UV 是从 32×32 贴图上选区的坐标，换了 16×16 贴图后这些选区不再适用：

1. 原版横向只取 `x8→16` → 本项目 16×16 贴图应横向拉满 `x0→16`
2. 原版纵向 y8-11 命中 16×16 贴图中央 y6-9 的透明带 → 侧面显示空白，需避开
3. 原版纵向只取 3 行 → 本项目有完整 6 行可用，应取完整区域

本项目最终 UV 设定（5 个父模型，上下壳横向拉满）：

| 面位置 | 本项目 UV | 说明 |
|--------|----------|------|
| 下壳侧面 | `[0, 10, 16, 16]` | 横向拉满，纵向取完整 6 行（y10-16） |
| 上壳侧面 | `[0, 0, 16, 6]` | 横向拉满，纵向取完整 6 行（y0-6） |

（`block_top`/`block_top_bottom` 变体中上壳高度为 5px，UV 为 `[0, 0, 16, 5]`；下壳同理为 `[0, 10, 16, 15]`，纵向少取 1 行以匹配缩短的面。）

### 大小齿轮 UV 对比总结

| 对比项 | 小齿轮 | 大齿轮（原版） | 大齿轮（本项目） |
|--------|--------|---------------|-----------------|
| 侧面贴图 | 16×16 独立贴图 | 32×32 CT 精灵表 | 16×16 独立贴图 |
| UV 横向 | `0→16` 拉满 | `8→16` 不拉满 | `0→16` 拉满 |
| 上壳 UV 纵向 | `0→6`（6 行） | `8→11`（3 行） | `0→6`（6 行） |
| 下壳 UV 纵向 | `10→16`（6 行） | `13→16`（3 行） | `10→16`（6 行） |
| 纵向拉伸 | 无（1:1） | 有（3 行拉到 6px） | 无（1:1） |

---

## 三、模型 JSON 继承机制

Minecraft 模型 JSON 通过 `parent` 字段实现继承。子模型继承父模型的 elements（3D 几何）、textures（纹理变量定义）等，并可覆盖或补充。

### 纹理变量（Texture Variables）

模型 JSON 的 `textures` 块定义纹理变量名到贴图路径的映射。有两种形式：

```json
"textures": {
    "siding": "crystal_clear:block/encased_cogwheels/andesite_encased_cogwheel_side",
    "particle": "#casing"
}
```

- `"siding": "crystal_clear:block/..."` — 定义变量 `siding` 指向具体贴图路径
- `"particle": "#casing"` — 定义变量 `particle` 引用另一个变量 `#casing` 的值

在 elements 的 face 中，通过 `"texture": "#siding"` 引用变量（注意 `#` 前缀）。

### 继承规则

#### 1. 子模型继承父模型的一切

子模型自动继承父模型的 `elements`（3D 几何）、`textures`（纹理变量定义）。如果子模型不定义自己的 elements，就使用父模型的几何。

#### 2. 纹理变量覆盖

子模型 `textures` 块中定义的变量会**覆盖**父模型中同名的变量。这是实现材质差异化的核心机制。

```json
// 父模型 encased_large_cogwheel/block.json
{
    "textures": {
        "particle": "#particle"
        // 无 siding/casing/backing 的默认值——由子模型提供
    },
    "elements": [
        { "faces": { "north": {"uv": [0,10,16,16], "texture": "#siding"} } }
    ]
}

// 子模型 andesite_glass_encased_large_cogwheel.json
{
    "parent": "crystal_clear:block/encased_large_cogwheel/block",
    "textures": {
        "backing": "minecraft:block/stripped_spruce_log_top",
        "casing": "crystal_clear:block/andesite_glass_casing",
        "opening": "create:block/gearbox",
        "siding": "crystal_clear:block/encased_cogwheels/large_andesite_encased_cogwheel_side",
        "particle": "crystal_clear:block/andesite_glass_casing"
    }
    // 无 elements——继承父模型的 3D 几何
}
```

子模型传入 `siding`/`casing`/`backing`/`opening`/`particle` 五个变量值，父模型的 faces 通过 `#siding` 等引用它们，从而实现不同材质的视觉差异。

#### 3. 父模型"出现就算"——无需提前声明

父模型的 `textures` 块中**不需要**提前声明所有子模型可能传入的变量名。父模型只需声明自己 elements 中实际引用的变量。

- 如果父模型 elements 中某个 face 引用了 `#siding`，但父模型 `textures` 块中没有定义 `siding` 的默认值，那么这个变量的值**必须由子模型提供**，否则该面会显示为缺失贴图（紫黑方块）。
- 如果父模型 `textures` 块中定义了 `siding` 的默认值，子模型没有覆盖它，则使用默认值。
- 子模型传入的变量如果父模型 elements 中没有引用，则被忽略（无效果）。

这就是"父模型不用单独声明变量，出现就算"的含义：父模型 elements 中写了 `"texture": "#siding"` 就算使用了 `siding` 变量，不需要在 `textures` 块中先声明一个空壳。

#### 4. 子模型必须使用父模型有的变量才能赋值生效

子模型 `textures` 块中定义的变量，**只有父模型 elements 中实际引用的才会生效**。

如果子模型定义了 `"my_var": "some:texture"`，但父模型没有任何 face 引用 `#my_var`，则这个赋值完全无效——不会被传递、不会被使用、不会报错，纯粹被忽略。

因此，子模型能覆盖的变量 = 父模型 elements 中实际引用的变量集合。子模型不能"新增"父模型不认识的变量并期望它生效。

#### 5. particle 的特殊性

`particle` 变量用于方块破坏时的粒子贴图。它通常通过 `"particle": "#casing"` 引用另一个变量，使其与机壳贴图一致。如果父模型定义了 `particle` 的默认值，子模型可以覆盖；如果父模型用 `#casing` 引用，则子模型覆盖 `casing` 即可同时影响 `particle`。

### 本项目的继承结构

```
Create 原版模型（供参考，不直接继承）
  create:block/encased_large_cogwheel/block   ← 原版父模型（变量名 #side/#1/#4）
  create:block/large_wheels                    ← 原版 item 父模型

本项目独立模型（从 Create 原版提取并改造）
  crystal_clear:block/encased_large_cogwheel/block        ← 本项目父模型（变量名 #siding/#backing/#opening）
  crystal_clear:block/encased_large_cogwheel/block_top     ← 变体（上壳有开口）
  crystal_clear:block/encased_large_cogwheel/block_bottom  ← 变体（下壳有开口）
  crystal_clear:block/encased_large_cogwheel/block_top_bottom ← 变体（上下都有开口）
  crystal_clear:block/encased_large_cogwheel/item         ← item 模型（parent: create:block/large_wheels）

本项目子模型（12 系列 × 4 变体 = 48 个 block + 12 个 item）
  andesite_glass_encased_large_cogwheel.json               ← parent: .../block
  andesite_glass_encased_large_cogwheel_top.json           ← parent: .../block_top
  andesite_glass_encased_large_cogwheel_bottom.json        ← parent: .../block_bottom
  andesite_glass_encased_large_cogwheel_top_bottom.json    ← parent: .../block_top_bottom
  ...（brass/copper/train × glass/clear_glass/illumination）
```

父模型定义 3D 几何和 UV（使用 `#siding`/`#casing`/`#backing`/`#opening` 变量名），子模型只传入材质贴图值。这种"接口层"设计将几何与材质解耦——如果 Create 原版将来改变量名，只需改本项目父模型一次，所有 60 个子模型不受影响。

---

## 四、本项目 UV 修复要点

本次修复（已处理问题 #12 延伸）的核心改动：

1. **贴图更换导致 UV 不适用**：原版 UV `[8,13,16,16]`/`[8,8,16,11]` 是从 32×32 精灵表上选区的坐标，本项目改用 16×16 独立贴图后这些选区不再适用，需重新设定
2. **避开透明带**：16×16 贴图中央 y6-9 有 4 行全透明带，上壳 UV 需避开此区域
3. **横向拉满**：用户改默认侧面纹理后，要求上下壳侧面横向 `x0→16` 拉满整个贴图宽度
4. **纵向完整取区域**：下壳 UV 从原版的 3 行改为完整的 6 行，与上壳对称

修复后的 UV 与小齿轮的 UV 设定方式一致（横向拉满、纵向按区域取完整高度），实现了大小齿轮风格统一。