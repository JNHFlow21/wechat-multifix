# 微信双开 · macOS 一键安装工具

macOS 微信双开/多开自动化安装与修复工具，内置 **6种主题配色图标**，每个微信实例一看便知。

## 6种内置图标

| 图标 | 配色 | 主题感 |
|------|------|--------|
| ![Black](icons/previews/black.png) | **黑底白标** | 低调、暗黑模式首选 |
| ![Cyan](icons/previews/cyan.png) | **青底白标** | 清新、科技感 |
| ![Purple](icons/previews/purple.png) | **紫底白标** | 优雅、创意 |
| ![Orange](icons/previews/orange.png) | **橙底白标** | 活力、温暖 |
| ![Navy](icons/previews/navy.png) | **深蓝底白标** | 沉稳、商务 |
| ![Dark Gray](icons/previews/darkgray.png) | **深灰底白标** | 低调、简洁 |

**与原版对比：**

| 应用 | 配色 |
|------|------|
| WeChat.app | 🟢 绿底白标（原版） |
| WeChat2.app | ⬛ 黑底白标 |
| WeChat3.app | 🔵 青底白标 |
| WeChat4.app | 🟣 紫底白标 |
| WeChat5.app | 🟠 橙底白标 |
| WeChat6.app | 🔷 深蓝底白标 |

---

## Quick Start

把下面这段 prompt **完整复制**给 AI Agent（Claude Code、Openclaw、Hermes 等），它会自动完成所有配置：

```
帮我安装 macOS 微信双开工具。

步骤：
1. 读取这个 Skill 的 SKILL.md 文件，路径在当前目录的 .agents/skills/wechat-multifix/SKILL.md
2. 按照 SKILL.md 中的「一键安装 WeChat2」流程执行
3. 安装完成后，执行 open -a /Applications/WeChat2.app 启动微信2

完成后告诉我：
- WeChat 和 WeChat2 的 Bundle ID 分别是什么？
- 两者是否不同？
```

Agent 会自动：
1. 读取 Skill 说明
2. 执行完整安装流程（复制 → Bundle ID → 签名 → 换图标）
3. 验证 Bundle ID 唯一性
4. 提示可以启动

**想用其他配色？** 在步骤2之前告诉 Agent 你想要哪个配色（黑/青/紫/橙/深蓝/深灰），它会在安装时使用对应的图标文件。

---

## 目录结构

```
wechat-multifix/
├── README.md              ← 本文件
├── SKILL.md               ← Agent 技能定义（自动安装+修复）
└── icons/
    ├── AppIcon-black.icns      ← 黑底白标
    ├── AppIcon-cyan.icns       ← 青底白标
    ├── AppIcon-purple.icns     ← 紫底白标
    ├── AppIcon-orange.icns     ← 橙底白标
    ├── AppIcon-navy.icns       ← 深蓝底白标
    ├── AppIcon-darkgray.icns   ← 深灰底白标
    └── previews/               ← 预览图
        ├── black.png
        ├── cyan.png
        ├── purple.png
        ├── orange.png
        ├── navy.png
        └── darkgray.png
```

---

## 手动安装（不用 Agent）

```bash
SKILL_DIR="$HOME/.../wechat-multifix"  # 下载后解压缩的路径
ICON_NAME="AppIcon-black.icns"         # 换成其他配色

cp -R /Applications/WeChat.app /Applications/WeChat2.app
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist
xattr -cr /Applications/WeChat2.app
codesign --force --deep --sign - /Applications/WeChat2.app
chown -R $(whoami):admin /Applications/WeChat2.app
cp "$SKILL_DIR/icons/$ICON_NAME" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app
open -a /Applications/WeChat2.app
```

---

## 微信更新后修复

微信自动更新会重置 Bundle ID。运行以下命令修复：

```
修复微信多开。微信更新后 WeChat2 打不开了。
读取 .agents/skills/wechat-multifix/SKILL.md，按「修复打不开的微信2」的步骤执行。
```

---

## 技术原理

- **Bundle ID**：每个 macOS 应用唯一标识，直接复制会导致冲突
- **解决方法**：修改克隆版的 Bundle ID + 重新签名
- **数据存储**：`~/Library/Containers/com.tencent.xinWeChat2/`（与应用分离，修复不丢数据）

---

## License

MIT
