# 从「微信双开打不开了」到一键修复工具：我的开源项目诞生记

## 那个让人崩溃的周一早上

事情是这样的。

某个普通的周一早上，我像往常一样打开微信准备处理消息，却发现自己再也无法同时登录两个账号了——工作号和家庭号，必须来回切换。那一刻我意识到：我需要微信双开。

按照 [@jinchenma94](https://x.com/jinchenma94/status/2030152054572994591) 的这篇教程，我成功配置了微信双开，一切看起来都很美好。直到微信自动更新之后——

**两个微信都打不开了。**

## 问题的根源：Bundle ID 冲突

微信每次自动更新会覆盖我手动创建的 `WeChat2.app`，并把它的 Bundle ID 重置为与原版相同：`com.tencent.xinWeChat`。

在 macOS 的世界里，Bundle ID 是应用的"身份证号"——每个应用必须唯一。当两个微信拥有相同的身份证号，系统就会困惑，然后拒绝运行两者中的任何一个。

```
WeChat.app        → com.tencent.xinWeChat    ← 原版
WeChat2.app       → com.tencent.xinWeChat    ← 被微信更新覆盖了！
```

解决方案很简单：**修改克隆版的 Bundle ID 并重新签名**。

但问题是——每次微信更新都要手动操作一遍，这太烦了。

## 从手动修复到一键自动化

作为一个 AI 爱好者，我立刻想到：能不能让 AI Agent 来帮我做这件事？

于是我找到了 Claude Code，说：

> "帮我安装 macOS 微信双开工具。"

Claude 自动读取 Skill 定义、执行完整安装流程（复制 → 修改 Bundle ID → 签名 → 换图标），5 分钟后我就有了两个可以同时运行的微信。

图标还是原版绿色？那就换一个——我选**黑底白标**，低调又专业：

```
WeChat.app        → 🟢 绿底白标（原版）
WeChat2.app       → ⬛ 黑底白标（一目了然）
```

## 不只是安装：自动修复

最大的痛点不是安装，而是**微信更新后又要重来一遍**。

于是我又对 Claude 说："能不能把整个修复流程固化成一个 Skill，以后一键修复？"

Claude 照做了。新的 Skill 不仅能安装，还能检测 Bundle ID 冲突、自动重建 WeChat2、并恢复我选的主题图标。整个过程无需我手动干预，聊天记录零丢失。

```
触发词：「修复微信多开」
```

Agent 自动完成一切。

## 六种配色，总有一款适合你

为了让多个微信实例一目了然，我预设了 6 种主题配色图标：

| 配色 | 预览 | 主题感 |
|------|------|--------|
| ⬛ 黑底 | ![](icons/previews/black.png) | 低调、暗黑模式首选 |
| 🔵 青底 | ![](icons/previews/cyan.png) | 清新、科技感 |
| 🟣 紫底 | ![](icons/previews/purple.png) | 优雅、创意 |
| 🟠 橙底 | ![](icons/previews/orange.png) | 活力、温暖 |
| 🔷 深蓝底 | ![](icons/previews/navy.png) | 沉稳、商务 |
| ⬜ 深灰底 | ![](icons/previews/darkgray.png) | 低调、简洁 |

安装时只需要告诉 Agent 你想要哪个配色，剩下的全部自动完成。

## 技术原理

macOS 应用签名机制的核心是 Bundle ID 和代码签名：

1. **复制应用**：`cp -R /Applications/WeChat.app /Applications/WeChat2.app`
2. **修改 Bundle ID**：`/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" ...`
3. **清除扩展属性**：`xattr -cr`（解决"resource fork"签名错误）
4. **重新签名**：`codesign --force --deep --sign - WeChat2.app`
5. **替换图标**：复制自定义 `.icns` 文件

聊天记录存储在 `~/Library/Containers/com.tencent.xinWeChat2/`，与应用本身分离，所以修复过程不会丢失任何数据。

## 发布开源

项目地址：[github.com/JNHFlow21/wechat-multifix](https://github.com/JNHFlow21/wechat-multifix)

如果你也在用 macOS 和微信双开，这个工具应该能帮你省不少麻烦。

## 致谢

本文的核心思路来源于 [@jinchenma94](https://x.com/jinchenma94/status/2030152054572994591) 的推文——正是他那篇「在 Mac 上双开/多开微信指南」让我第一次成功配置了微信双开。本项目在此基础上增加了**一键自动修复**和**多配色图标**功能，将手动流程升级为可复用的 AI Agent Skill。

---

*有问题？欢迎提 Issue。微信更新后打不开？直接说「修复微信多开」，让 Agent 来帮你。*
