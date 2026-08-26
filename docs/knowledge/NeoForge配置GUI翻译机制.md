# NeoForge 配置 GUI 翻译机制

## 背景

NeoForge 1.21.1 提供了两套配置 GUI 方案，本项目对两种方案均做了适配：

1. **NeoForge 内置** `ConfigurationScreen`：需在 mod 代码中注册 `IConfigScreenFactory` 扩展点，自动从 `ModConfigSpec` 生成界面。无额外依赖。
2. **Configured mod**（MrCrayfish）：第三方 mod，作为开发环境运行时依赖（`localRuntime`），自动为所有注册了 `ModConfigSpec` 的 mod 生成交互式配置界面。不打包到发布 jar，最终用户无需安装。

两种方案对翻译键的处理方式不同，本文档分别说明两套方案的翻译约定。

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

## NeoForge 内置 ConfigurationScreen 翻译键体系

NeoForge 内置 `ConfigurationScreen` 有自己独立的翻译键约定，与 Configured 完全不同。它通过 `TranslationChecker` 在开发环境下检查所有翻译键是否缺失，缺失时输出 WARN 日志。

### 翻译键生成规则（通过反编译字节码确认）

ConfigurationScreen 构造函数使用 `StringConcatFactory.makeConcatWithConstants` 拼接翻译键模板。关键模板：

| 模板 | 生成结果 | 用途 |
|------|----------|------|
| `\u0001.configuration.title` | `{modId}.configuration.title` | 配置界面整体标题 |
| `\u0001.configuration.section.\u0001\u0001` | `{modId}.configuration.section.{processedFileName}` | 配置文件分区名称 |
| 在 section key 后追加 `.title` | `{sectionKey}.title` | 分区标题 |
| `\u0001.button`（在 ConfigurationSectionScreen 中） | `{categoryTranslationKey}.button` | 分类按钮文本 |

其中 `{modId}` 取自 `ModContainer.getModId()`。

`{processedFileName}` 来自 `ModConfig.getFileName()`（如 `crystal_clear-client.toml`），经过以下处理：
```java
fileName.replaceAll("[^a-zA-Z0-9]+", ".")
        .replaceFirst("^\\.", "")
        .replaceFirst("\\.$", "")
        .toLowerCase();
```
即：所有非字母数字字符替换为 `.`，去掉首尾 `.`，转小写。例如 `crystal_clear-client.toml` → `crystal.clear.client.toml`。

`{categoryTranslationKey}` 取自配置分类在 `ModConfigSpec` 中 `.translation()` 设置的 key（如 `config.crystal_clear.creative`），追加 `.button` 后缀。

### 完整翻译键清单（以 crystal_clear 为例）

```json
{
  "crystal_clear.configuration.title": "晶莹剔透 配置",
  "crystal_clear.configuration.section.crystal.clear.client.toml": "crystal_clear-client.toml",
  "crystal_clear.configuration.section.crystal.clear.client.toml.title": "客户端配置",
  "config.crystal_clear.creative.button": "创造模式"
}
```

> **本项目未添加上述翻译键。** TranslationChecker 对这 4 个键使用 `check(key, fallback)` 带回退机制，缺失时自动回退到默认值（如标题用 mod 显示名、分区名用配置文件名），默认效果已足够好。这些键属于可选的个性化扩展，当前无定制需求，不添加。开发环境 runClient 时打开配置界面仍会看到 TranslationChecker WARN 日志，这是预期行为。

### TranslationChecker 行为

- `TranslationChecker` 是 `ConfigurationScreen` 的内部类，在配置界面被打开时实例化并检查所有翻译键。
- `check(key)` 方法通过 `I18n.exists(key)` 判断翻译键是否存在，不存在则加入 `untranslatables` 集合。
- `check(key, fallback)` 方法用于带回退值的检查，不存在则加入 `untranslatablesWithFallback` 集合。
- `finish()` 方法在界面关闭时调用，若存在未翻译的键且 `NeoForgeConfig.CLIENT.logUntranslatedConfigurationWarnings` 为 true（开发环境默认开启），则输出 WARN 日志。
- **注意**：TranslationChecker 只在打开配置界面时触发，游戏启动时不会输出警告。

### tooltip 行为

NeoForge 内置界面直接使用 `.comment()` 原文作为 tooltip，**不翻译**。这与 Configured 的 `.tooltip` 后缀约定不同。因此：
- `.comment()` 文本应该用目标语言书写（或英文），因为它会直接显示给用户。
- 不需要为内置界面额外添加 `.tooltip` 翻译键。

### 注意事项

- **modId 中的下划线不转换**：`{modId}` 直接使用 modId 原值（如 `crystal_clear`），不做下划线→点号转换。
- **文件名中的下划线会转换**：`processedFileName` 中 `[^a-zA-Z0-9]+` → `.` 的替换会把下划线转为点号（如 `crystal_clear` → `crystal.clear`）。
- **分类按钮 key 用 `.translation()` 值**：不是 modId 前缀，而是 `.translation()` 设置的完整 key + `.button`。

## 两种方案对比总结

| 对比项 | NeoForge ConfigurationScreen | Configured mod |
|--------|------------------------------|----------------|
| 条目名称 | `.translation()` | `.translation()` |
| tooltip | `.comment()` 原文（不翻译） | `translation_key + ".tooltip"`（可翻译） |
| 界面标题 | `{modId}.configuration.title` | 无（使用 mod 名称） |
| 分区名称 | `{modId}.configuration.section.{fileName}` | 无 |
| 分类按钮 | `{translation_key}.button` | `{translation_key}` |
| 需注册扩展点 | 是（`IConfigScreenFactory`） | 否（自动发现） |
| 依赖 | 无额外依赖 | 需安装 Configured（开发环境 `localRuntime`） |
| 翻译检查 | TranslationChecker（开发环境 WARN） | 无 |