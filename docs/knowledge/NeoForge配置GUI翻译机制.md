# NeoForge 配置 GUI 翻译机制

## 背景

NeoForge 1.21.1 提供了两套配置 GUI 方案：

1. **NeoForge 内置** `ConfigurationScreen`：需在 mod 代码中注册 `IConfigScreenFactory` 扩展点，自动从 `ModConfigSpec` 生成界面。
2. **Configured mod**（MrCrayfish）：第三方 mod，作为开发环境运行时依赖（`localRuntime`），自动为所有注册了 `ModConfigSpec` 的 mod 生成交互式配置界面。不打包到发布 jar，最终用户无需安装。

两种方案对翻译键的处理方式不同，本文档说明 Configured mod 的翻译约定。

## ModConfigSpec 翻译键

在 `ModConfigSpec.Builder` 中，每个配置条目和分类（push/pop 块）可以设置 `.translation("key")` 和 `.comment("text")`：

```java
builder.comment("Creative Mode display options")
       .translation("config.modid.creative")    // 分类名称翻译键
       .push("creative");

showEncasedVariants = builder
        .comment("Show encased shafts, cogwheels, and large cogwheels in the creative tab.")
        .translation("config.modid.creative.show_encased_variants")  // 条目名称翻译键
        .define("show_encased_variants", false);

builder.pop();
```

- `.translation()` 设置的 key 对应配置界面中该条目/分类的**显示名称**。
- `.comment()` 的文本写入 TOML 配置文件作为注释，同时作为 tooltip 的回退内容。

## Configured 的 tooltip 翻译约定

Configured 自创了 `.tooltip` 后缀约定：在 `.translation()` 的 key 后追加 `.tooltip`，即为该条目的鼠标悬停 tooltip 翻译键。

### 翻译键格式

| 用途 | 翻译键 |
|------|--------|
| 条目/分类显示名称 | `.translation("key")` 的值 |
| 条目/分类 tooltip | `.translation("key")` 的值 + `.tooltip` |

### 优先级链

1. 检查 lang 文件中是否存在 `key.tooltip` → 存在则显示翻译文本，并自动追加 `"Range: "` / `"Allowed Values: "` 部分（如有）
2. 不存在 → 回退显示 `.comment()` 原始文本（不翻译）

### 示例

```json
{
  "config.crystal_clear.creative": "创造模式",
  "config.crystal_clear.creative.tooltip": "创造模式显示选项",
  "config.crystal_clear.creative.show_encased_variants": "显示包覆方块变体",
  "config.crystal_clear.creative.show_encased_variants.tooltip": "在创造栏中显示包覆轴、齿轮和大齿轮。这些是将机壳应用于 Create 的传动杆/齿轮后产生的次级状态。默认关闭以保持栏面整洁。"
}
```

### 注意事项

- **必须调用 `.translation()`**：未调用时 `getTranslationKey()` 返回 `null`，Configured 无法构造 `.tooltip` 键，只能回退到 `.comment()` 原文（不翻译）。
- **`.comment()` 仍需保留**：即使有了 `.tooltip` 翻译，`.comment()` 会写入 TOML 配置文件作为注释；且 `"Range: "` / `"Allowed Values: "` 部分会被自动追加到翻译后的 tooltip 后面。
- **`defineInRange` 和 `defineEnum`**：NeoForge builder 会自动在 comment 中追加 `" Default: "` 和 `" Range: "`（数值范围）或 `"Allowed Values: "`（枚举），这部分会附加到 tooltip 翻译文本之后，不需要手动翻译。

## 与 NeoForge 原生 ConfigurationScreen 的区别

| 对比项 | NeoForge ConfigurationScreen | Configured mod |
|--------|------------------------------|----------------|
| 条目名称 | `.translation()` | `.translation()` |
| tooltip | `.comment()` 原文（不翻译） | `translation_key + ".tooltip"`（可翻译） |
| 需注册扩展点 | 是（`IConfigScreenFactory`） | 否（自动发现） |
| 依赖 | 无额外依赖 | 需安装 Configured（开发环境 `localRuntime`） |