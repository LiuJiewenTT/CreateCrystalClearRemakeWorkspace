# Create Crystal Clear Remake Workspace

Doc Lang: 简体中文 | [English](README-en_us.md)

这是一个计划重建 *Create Crystal Clear* 模组的工作区。由于原作者Cyvack（消极->积极）拒绝维护，且后续Minecraft JE 1.21.1版本仅有Adonis重建的版本且已经11个月没有更新了（此刻是2026年8月21日），所以我决定重建这个模组（With the POWER of AI!）。

## 项目规划 

Cyvack的仓库曾经是MIT协议的，后来转ARR了；Adonis的仓库是MIT协议的。这个仓库，决定继续使用MIT协议。

> 由于本人不擅长Mod开发，此项目不保证成功，纯看AI是否大力！另外，欢迎各位大佬接手维护和更新！

## 项目结构

当前工作区的结构：

- 文档存储在`docs`中。
- 参考代码库存储在`referring_repos`中，AI可以自行将想要参考的仓库克隆到此目录下。
- 重建项目独立一个仓库，存储在`repo`中，并作为一个完整的独立项目。

相关项目：

- Mod依附的主Mod *Create*（中文名：*机械动力*）：https://github.com/Creators-of-Create/Create

- 原始项目的前任项目（截止到1.19.2, MIT协议）：https://github.com/Cyvack/Create-Crystal-Clear
- 原始项目（ARR协议）：https://github.com/Cyvack/Crystal-Clear-Arch
- 原始项目的支持Create 6.0+的cinnamondev修复版：https://github.com/cinnamondev/Crystal-Clear-Arch/tree/1.20.1-create6
- 原始项目的支持Create 6.0+的c0nnor263修复版：https://github.com/c0nnor263/Crystal-Clear-Arch/tree/mc1.20.1/6.0.4-c0nnor263
- Adonis的重构项目：https://github.com/adonis-baffin/CreatePrism
- Adonis的重构项目的LoneStarFateZero修复版：https://github.com/LoneStarFateZero/CreatePrism

- 本项目的重建项目基于LoneStarFateZero修复版，比照借鉴原始项目以进行AI查错，但不应当使用原始项目的代码，避免违反ARR。
- 原始项目仅支持Minecraft 1.20.1，并且不支持Create 6.0+。
- Create 6.0+主要参考c0nnor263修复的版本。
- Create 6.0.6+又有一次破坏性变更（可以查看Modrinth上的更新日志或Github上发布页的描述），那是后话了，可以先以兼容6.0.4的版本为目标。

## AI工作指导

AI启动提示词：

```
读取文档编写指导和计划，读取必要的相关文档，然后向用户确认计划，然后开始工作。注意许可问题，避免直接使用不兼容MIT协议的代码。
```

## Credits

感谢此项目上游相关开发者的贡献：

 - [Cyvack](https://github.com/Cyvack)
 - [cinnamondev](https://github.com/cinnamondev)
 - [Oleh Boichuk](https://github.com/c0nnor263)
 - [adonis-baffin](https://github.com/adonis-baffin)
 - [Lonely Star](https://github.com/LoneStarFateZero)

