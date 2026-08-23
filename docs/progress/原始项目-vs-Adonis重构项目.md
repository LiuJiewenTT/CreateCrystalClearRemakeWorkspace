# 区别报告：原始项目 vs Adonis 重构项目

> 日期：2026-08-23
> 对比对象：
> - 原始项目：c0nnor263 修复版（基于 Cyvack 的 Crystal-Clear-Arch，ARR 协议，MC 1.20.1 + Create 6.0.4，Architectury 多平台）
> - Adonis 重构项目：Adonis 的 CreatePrism（MIT 协议，MC 1.21.1 + Create 6.0.6，NeoForge 单平台）
> - 末尾附 LoneStarFateZero 修复版与 Adonis 原版的区别

---

## 一、项目元信息对比

| 维度 | 原始项目（c0nnor263 版） | Adonis 重构版 |
|------|--------------------------|---------------|
| 协议 | ARR（Cyvack 原始仓库） | MIT |
| MC 版本 | 1.20.1 | 1.21.1 |
| Create 版本 | 6.0.4-79 | 6.0.6-98 |
| 加载器 | Forge 47.3.33 + Fabric 0.16.9（Architectury 多平台） | NeoForge 21.1.152（单平台） |
| modId | `crystal_clear` | `createprism` |
| mod 名 | Crystal-Clear | Create: Prismatic Shine |
| 包名 | `com.cyvack.crystal_clear` | `com.adonis.prism` |
| 主类名 | `CrystalClear` | `CreatePrism` |
| 版本号 | 2.0-Beta | 1.1.0 |
| 作者 | Cyvack | Adonis |
| Flywheel | 1.0.1 | 1.0.0-9 |
| Registrate | MC1.20-1.3.3 | MC1.21-1.3.0+62 |

## 二、架构差异

### 2.1 多平台 vs 单平台

原始项目使用 Architectury 框架，代码分为三层：
- `common/` — 平台无关代码（19 个 Java 文件）
- `forge/` — Forge 平台适配
- `fabric/` — Fabric 平台适配

Adonis 重构版完全放弃多平台，仅支持 NeoForge：
- `src/main/java/` — 全部代码（26 个 Java 文件）
- 无平台分层，直接使用 NeoForge API

### 2.2 源码包结构对比

**原始项目**（19 个 Java 文件）：
```
com.cyvack.crystal_clear/
├── CCCPlatform.java           — 平台抽象入口
├── CCCTabPlatform.java        — 创造栏平台抽象
├── RegistrationGenPlatform.java — 注册平台抽象
└── common/
    ├── CrystalClear.java      — 主类
    ├── ModSetup.java          — 模组初始化
    ├── CCCRegistrate.java     — Registrate 封装
    ├── content/blocks/        — 方块类（3个：GlassCasing, GlassEncasedCogwheel, GlassEncasedShaft）
    ├── creative_tab/          — ItemVisibilityHandler
    ├── generation/            — RegistrateGenHelper（方块构建器+资源引用）
    ├── glass_ct_behaviours/   — CT 行为（2个）
    ├── registry/              — 注册（CCCBlocks, CCCBlockEntities, CCSpriteShifts）
    └── util/                  — 工具（CasingHolder, CasingTypes, GlassBlockList）
```

**Adonis 重构版**（26 个 Java 文件）：
```
com.adonis.prism/
├── CreatePrism.java            — 主类（含 EncasingRegistry 注册逻辑）
├── block/
│   ├── glass/                  — 玻璃系列方块（5个：Casing, EncasedShaft, EncasedCogwheel, 2个CTBehaviour）
│   ├── illumination/           — 照明系列方块（5个：IlluminationCasing, IlluminationEncasedShaft, IlluminationEncasedCogwheel, 2个CTBehaviour）
│   └── package-info.java
├── mixin/                      — AllArmInteractionPointTypesMixin（空实现）
├── ponder/                     — CPPonderPlugin（空实现）
├── registry/
│   ├── CPBlocks.java           — 方块注册
│   ├── CPBlockEntities.java    — 方块实体注册
│   ├── CPItems.java            — 物品注册
│   ├── CPCreativeTab.java      — 创造栏
│   ├── CPClient.java           — 客户端注册
│   ├── CPPartialModels.java    — 部分模型
│   ├── CPRegistrate.java       — Registrate 封装
│   ├── CPSpriteShifts.java     — 精灵位移
│   └── builders/
│       └── GlassBlockBuilders.java — 方块构建器（所有 builder 集中于此）
└── util/                       — CasingHolder, CasingTypes, GlassBlockList
```

### 2.3 关键架构变化

1. **平台抽象层消除**：原始项目通过 `CCCPlatform`、`CCCTabPlatform`、`RegistrationGenPlatform` 三个接口抽象平台差异；Adonis 直接使用 NeoForge API，不再需要平台抽象。

2. **方块构建器重构**：原始项目将构建逻辑分散在 `RegistrateGenHelper` 中（含 builder + 资源引用工具方法）；Adonis 集中到 `GlassBlockBuilders` 类，且资源引用方法（`getOpening`、`getBacking`、`getSiding` 等）改为私有方法。

3. **EncasingRegistry 注册时机**：原始项目通过 `RegistrationGenPlatform` 平台接口在构建时注册 encasing 变体；Adonis 改在 `FMLCommonSetupEvent` 中统一注册（`CreatePrism.registerEncasingVariants()`）。

## 三、功能差异

### 3.1 照明系列（Illumination）— Adonis 新增

原始项目**没有**照明系列。Adonis 新增了完整的照明系列：
- `IlluminationCasing` — 照明机壳方块（基于 GlowStone，发光等级 15）
- `IlluminationEncasedShaft` — 照明包覆轴
- `IlluminationEncasedCogwheel` — 照明包覆齿轮
- `IlluminationEncasedCTBehaviour` / `IlluminationEncasedCogCTBehaviour` — CT 连接纹理行为
- `ILLUMINATION_CASINGS` 和 `ILLUMINATION_ENCASED` 枚举（CasingTypes.java）

照明系列的渲染层使用 `translucent`（半透明），而玻璃系列使用 `cutout`（镂空）。

### 3.2 Copper 材质扩展

| 系列 | 原始项目 | Adonis 重构版 |
|------|----------|---------------|
| Copper glass casing | 有 | 有 |
| Copper clear glass casing | 有 | 有 |
| Copper glass scaffolding | 有 | 有 |
| Copper glass encased shaft | **无** | 有 |
| Copper glass encased cogwheel | **无** | 有 |
| Copper glass encased large cogwheel | **无** | 有 |
| Copper illumination casing | **无** | 有 |
| Copper illumination encased shaft/cogwheel | **无** | 有（资源有，代码注册**缺失** — 本次重建已修复） |

原始项目中 Copper 的 `GENERAL_ENCASED` 枚举**不含** copper（只有 andesite、brass、train），而 Adonis 加入了 copper。但 Adonis 的 `ILLUMINATION_ENCASED` 枚举漏了 copper（只有 andesite、brass、train），这是原作者遗留的未完成功能。

### 3.3 应力值注册

原始项目（c0nnor263 版）在 `RegistrateGenHelper` 中为所有包覆轴和齿轮注册了应力值为 0：
```java
.onRegister(block -> BlockStressValues.IMPACTS.register(block, () -> 0))
```
- `glassEncasedShaft` 方法中（第130行）
- `glassEncasedCogwheelBase` 方法中（第201行）

Adonis 重构版的 `GlassBlockBuilders` 中**缺少**此注册。本次重建已补上。

### 3.4 Copper 贴图缺失（两者共同问题）

Copper 材质的包覆轴/齿轮引用了以下贴图，但两个版本中**均不存在**：
- `copper_gearbox`（轴模型 opening 贴图）
- `copper_backing`（齿轮模型 backing 贴图）
- `encased_cogwheels/copper_encased_cogwheel_side`（齿轮 siding 贴图）
- `encased_cogwheels/copper_encased_cogwheel_side_connected`（CT 连接贴图）
- `encased_cogwheels/large_copper_encased_cogwheel_side`（大齿轮 siding 贴图）

原始项目因为没有 Copper 包覆轴/齿轮，所以不存在此问题。Adonis 添加了 Copper 包覆系列但未提供对应贴图。

### 3.5 Copper 包覆管道（仅原始项目）

原始项目有 `copper_glass_encased_pipe` 的模型文件（`block_flat.json`、`block_open.json`），Adonis 重建时未包含此功能。

### 3.6 Steel 材质（仅原始项目）

原始项目有 steel 相关贴图（`steel_gearbox.png`、`steel_glass_casing.png`、`steel_glass_casing_connected.png`、`steel_tinted_glass_casing.png`、`steel_tinted_glass_casing_connected.png`），Adonis 重建时未包含。

### 3.7 Mixin 和 Ponder（仅 Adonis）

Adonis 新增了两个空实现：
- `AllArmInteractionPointTypesMixin` — 已编写但未注册到 mixins.json，代码仅反射获取 `register` 方法，空 catch 块
- `CPPonderPlugin` — 已注册到 PonderIndex，但 `registerScenes()` 和 `registerTags()` 均为空

原始项目没有这两个文件。

### 3.8 可疑战利品表（仅 Adonis）

Adonis 的项目中存在与本模组无关的战利品表文件：
- `centrifugal_pump.json`
- `copper_tap.json`
- `fluid_interface.json`
- `pipette.json`
- `smart_fluid_interface.json`

这些文件名指向其他 Create 附属模组的功能，疑似从其他项目残留。原始项目无此问题。

### 3.9 old_copper 贴图（仅 Adonis）

Adonis 包含 6 个 `old_copper_*.png` 贴图（旧版铜色调），与对应的 `copper_*.png` 像素内容不同。代码未引用，但已确认是独立旧版贴图。原始项目无此文件。

## 四、资源文件对比

| 资源类型 | 原始项目 | Adonis 重构版 |
|----------|----------|---------------|
| blockstates | 32（自动生成） | 53（手写） |
| models/block | 74（自动生成）+ 11（手写模板） | 137（手写） |
| models/item | 32（自动生成） | 53（手写） |
| textures/block | 32 | 50 |
| lang | 2（en_us, zh_cn） | 2（en_us, zh_cn） |
| loot_tables | 32（自动生成） | 53（手写，含 5 个可疑） |
| recipes | 14（自动生成） | 12（手写 item_application + 8 stonecutting） |
| advancements | 14（自动生成） | 无 |

原始项目使用 Registrate 的 data 生成机制，blockstate/model/loot/recipe/advancement 均为自动生成（存放在 `src/generated/`）。Adonis 改为全部手写资源文件。

## 五、 CT 行为差异

原始项目使用 `SimpleCTBehaviour`（单方向连接纹理）处理 glass casing 的 CT，而 Adonis 改用 `EncasedCTBehaviour`（Create 自带的包覆 CT 行为），处理方式不同。

---

## 附：LoneStarFateZero 修复版与 Adonis 原版的区别

LoneStarFateZero 的仓库仅有一个初始提交（`0287466 修复缺失文件`），包含完整的 419 个文件。由于 Adonis 的原始仓库不完整（缺少部分文件导致无法编译），LoneStarFateZero 的修复本质上是补全了 Adonis 项目缺失的文件，使其可以正常编译。

经比对本项目的重建子模块与 LoneStarFateZero 版，两者代码完全一致（LoneStarFateZero 版仅做了编译修复，无功能性变更）。具体表现为：

1. **单一提交**：仓库只有一个 commit，无法从中分离出"修复了哪些文件"的增量信息
2. **无功能性变更**：与 Adonis 原始设计完全一致，仅确保项目可编译
3. **保留所有 Adonis 特征**：包括空 Mixin、空 Ponder、可疑战利品表、old_copper 贴图、Copper 系列贴图缺失等所有 Adonis 遗留问题

LoneStarFateZero 的贡献在于：将 Adonis 的不完整项目整理为可编译、可发布的完整状态，为后续重建工作提供了可靠的基础。