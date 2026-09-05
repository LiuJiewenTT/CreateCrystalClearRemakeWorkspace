# 开发者变更日志

> 本文档记录代码、构建、资源层面的内部变更，面向模组开发者/贡献者。
> 玩家可感知的变更请参阅 [player-changelog.md](player-changelog.md)。

---

## 2026-09-05

### 新功能：玻璃机壳包覆铜水管

- **方块类**：新增 `GlassEncasedPipeBlock`（继承 `EncasedPipeBlock`，透光属性）和 `IlluminationEncasedPipeBlock`（继承前者，`lightLevel(15)`）
- **BlockEntity**：`CPBlockEntities` 新增 `GLASS_ENCASED_PIPE`（覆盖 glass + clear_glass 8 方块）和 `ILLUMINATION_ENCASED_PIPE`（覆盖 illumination 4 方块），均复用 `FluidPipeBlockEntity`
- **方块注册**：`CPBlocks` 新增 3 组 `GlassBlockEntry`（glass / clear_glass / illumination），每组 4 材质 = 12 方块
- **Builder**：`GlassBlockBuilders` 新增 `glassEncasedPipe()` 和 `illuminationEncasedPipe()`，含 `PipeAttachmentModel::withAO` 注册、CT 注册、`casingConnectivity` 注册
- **包覆注册**：`CrystalClear.registerEncasingVariants()` 在 `onCommonSetup` 中注册 `EncasingRegistry.addVariant(AllBlocks.FLUID_PIPE.get(), ...)` × 3 组
- **资源文件**：12 blockstate JSON、6 父模型 + 24 材质变体模型、36 条语言翻译、12 个 mineable/pickaxe 条目、12 个战利品表（均掉落铜水管）
- **创造栏**：`CPCreativeTab` 的 `showEncasedVariants` 分支补上 3 组 pipe 变体

### 修复

- **PipeAttachmentModel 注册缺失**：两个 pipe builder 补上 `.onRegister(blockModel(() -> PipeAttachmentModel::withAO))`
- **创造栏缺 12 个水管变体**：`CPCreativeTab` 遗漏 pipe 变体，已补上

### 技术发现

- **原版铜机壳包覆水管不渲染内部铜管**：`EncasedPipeBlock` 不继承 `FluidPipeBlock`，`isPipe()` 返回 false，`StandardPipeFluidTransportBehaviour.getRenderedRimAttachment()` 对 `EncasedPipeBlock` 返回 `NONE`。这是原版设计，玻璃机壳透视内部铜管为未来计划

### 技术债务新增

| 编号 | 描述 | 状态 |
|------|------|------|
| #11 | 玻璃机壳水管内部铜管不渲染（原版一致行为，用户希望未来实现透视） | 已搁置 |

---

## v2.0.0（2026-08-27）

基于 LoneStarFateZero 修复版（CreatePrism）完全重建，更名回归 Create Crystal Clear。

### 命名重构

- **modId**：`createprism` → `crystal_clear`
- **包名**：`com.adonis.prism` → `io.liujiewentt.crystal_clear`
- **主类**：`CreatePrism` → `CrystalClear`
- **资源目录**：`assets/createprism/` → `assets/crystal_clear/`、`data/createprism/` → `data/crystal_clear/`
- **Mixins 配置**：`createprism.mixins.json` → `crystal_clear.mixins.json`，package 字段同步更新
- **neoforge.mods.toml**：删除 `[[mixins]]` 块多余的无效 `file=` 行
- **全局字符串替换**：342 个 JSON 文件中的 `createprism:` / `CreatePrism` 统一替换
- git 统计：398 文件 R100 rename + 7 文件原地修改

### 功能补全

- **铜照明包覆注册**：`CasingTypes.ILLUMINATION_ENCASED` 枚举补充 `copper` 条目，补齐 3 条中英文翻译
- **应力值注册**：`GlassBlockBuilders` 4 个 builder 方法添加 `BlockStressValues.IMPACTS.register(block, () -> 0)`
- **铜包覆系列资源**：补齐 6 条语言翻译、2 条 mineable 标签、9 个战利品表 JSON

### 贴图补全

- **铜系列 4 个缺失贴图**：以 `copper_casing` 为底图手绘补全 `copper_gearbox`、`copper_encased_cogwheel_side`、`large_copper_encased_cogwheel_side`、`copper_encased_cogwheel_side_connected`（32×32 CT 精灵表）
- **36 个模型 JSON 引用路径修正**：`create:block/copper_gearbox` → `crystal_clear:block/copper_gearbox`
- **copper_backing 贴图**：补全贴图文件，30 个 copper 系列 JSON backing 引用从 `minecraft:block/stripped_dark_oak_log_top` 改为 `crystal_clear:block/copper_backing`，使代码/JSON/贴图三者一致

### 代码-资源一致性修复

- **#6 大齿轮 blockFolder**：统一为 `encased_cogwheel`（后恢复 `large` 分叉，见下）
- **#9 illumination 侧面贴图路径**：创建 12 个 illumination 专用 siding 贴图（4 材质 × 3 形态），修改 32 个 illumination 齿轮模型 JSON 引用为独立路径，使代码与 JSON 一致。为未来侧透版机壳预留独立贴图通道
- **#10 copper_backing 引用不一致**：见上"贴图补全"

### 大齿轮模型修复（#12 延伸，三步）

1. **引入大齿轮专用模型**：从 Create 6.0.6 jar 提取 `encased_large_cogwheel/` 目录（5 个 JSON），60 个大齿轮模型 JSON parent 引用从 `encased_cogwheel/*` 改为 `encased_large_cogwheel/*`。`GlassBlockBuilders` 4 处 `blockFolder` 恢复 `large` 分叉
2. **父模型纹理变量统一**：`encased_large_cogwheel/` 5 个父模型 JSON 的 Create 原版变量名（`#side`/`#1`/`#4`）统一为本项目变量名（`#siding`/`#backing`/`#opening`），移除默认贴图值（item.json 保留 `#4` 为大齿轮本体贴图）
3. **UV 修正 + 安山照明变体修复**：
   - 上下壳 UV 修正（避开 16×16 贴图 y8-9 透明带，下壳选取完整区域，横向拉满）
   - 安山照明大齿轮 4 个模型文件 siding 引用 + parent 指向修正（#13 修复时的遗漏）

### 配置系统

- **新增 `CPConfig.java`**：CLIENT 类型配置 `show_encased_variants`（默认 false），控制创造栏包覆方块显示
- **`CrystalClear.java`**：注册 `ModConfigSpec` + `IConfigScreenFactory`（NeoForge 内置 ConfigurationScreen）
- **`CPCreativeTab.java`**：包覆轴/齿轮展示代码用配置条件包裹
- **语言文件**：en_us / zh_cn 各补充配置翻译条目
- **NeoForge 内置 GUI 翻译键**：反编译确认 4 个翻译键生成规则，TranslationChecker 有 fallback 机制，决定不添加

### 构建系统

- **移除无关依赖**：`build.gradle` 删除 `nether-depths-upgrade`（炽海生机）和 `geckolib` 两个 `localImplementation`，`gradle.properties` 删除对应版本号
- **JEI 版本升级**（开发环境）：19.21.0.247 → 19.24.0.317（满足 Configured 最低版本要求）
- **Configured 集成**（开发环境）：`localRuntime` 引用 Configured jar 用于配置 GUI 测试（发布 jar 不包含）
- **CI 修正**：GitHub Actions JDK 17 → 21

### 配置文件修复

- **Mixin package 路径**：修复 `crystal_clear.mixins.json` 中 package 字段与实际包路径不匹配
- **AccessTransformer**：清空不相关条目（FishingHook 相关，非本模组功能）
- **neoforge.mods.toml**：删除 `[[mixins]]` 块多余的 `file=` 行

### 文档与许可证

- **README.md**：从 UTF-16 LE BOM 单行错误标题重写为完整项目说明（简介、支持方块表、用法、技术栈、致谢、构建说明）
- **LICENSE.txt**：保留 Adonis (2025) 版权声明，新增 LiuJiewenTT (2026) 版权声明

### 已知技术债务

| 编号 | 描述 | 状态 |
|------|------|------|
| #2 | 7 处 `addLayer` 过时 API 警告 | 待定，当前编译通过 |
| #3 | 空 Mixin（`AllArmInteractionPointTypesMixin`）未注册 | 暂不处理 |
| #4 | 空 Ponder（`CPPonderPlugin`）无场景 | 暂不处理 |
| #7 | `mineable/axe` 标签与 `pickaxe` 内容相同 | 待定 |
| #8 | 5 个无关战利品表保留未清理 | 暂不处理 |