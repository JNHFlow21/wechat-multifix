# 微信双开 · macOS 一键安装工具

macOS 微信双开/多开自动化安装与修复工具，内置 **6种主题配色图标**，每个微信实例一看便知。

## 6种内置图标

<p align="center">
  <img src="icons/previews/black.png" width="60" alt="black"/> &nbsp;
  <img src="icons/previews/cyan.png" width="60" alt="cyan"/> &nbsp;
  <img src="icons/previews/purple.png" width="60" alt="purple"/> &nbsp;
  <img src="icons/previews/orange.png" width="60" alt="orange"/> &nbsp;
  <img src="icons/previews/navy.png" width="60" alt="navy"/> &nbsp;
  <img src="icons/previews/darkgray.png" width="60" alt="darkgray"/>
</p>

| 配色 | 图标文件名 | 主题感 |
|------|-----------|--------|
| ⬛ 黑底 | `AppIcon-black.icns` | 低调、暗黑模式首选 |
| 🔵 青底 | `AppIcon-cyan.icns` | 清新、科技感 |
| 🟣 紫底 | `AppIcon-purple.icns` | 优雅、创意 |
| 🟠 橙底 | `AppIcon-orange.icns` | 活力、温暖 |
| 🔷 深蓝底 | `AppIcon-navy.icns` | 沉稳、商务 |
| ⬜ 深灰底 | `AppIcon-darkgray.icns` | 低调、简洁 |

**与原版对比：** WeChat.app = 🟢 绿底白标（原版）

---

## Quick Start

把下面这段 prompt **完整复制**给 AI Agent（Claude Code、Openclaw、Hermes 等）：

```
帮我安装 macOS 微信双开工具。

步骤：
1. 读取这个 Skill 的 SKILL.md 文件，路径在当前目录的 .agents/skills/wechat-multifix/SKILL.md
2. 按照 SKILL.md 中的流程执行安装

安装前，Agent 会自动询问你想要哪种配色（黑/青/紫/橙/深蓝/深灰），你只需要回复数字或配色名即可。

完成后告诉我：
- WeChat 和 WeChat2 的 Bundle ID 分别是什么？
- 两者是否不同？
```

Agent 会自动：
1. 读取 Skill 说明
2. **询问你想要哪种配色**
3. 执行完整安装流程（复制 → Bundle ID → 签名 → 换图标）
4. 验证 Bundle ID 唯一性
5. 提示可以启动

---

## 目录结构

```
wechat-multifix/
├── README.md              ← 本文件
├── SKILL.md               ← Agent 技能定义（自动安装+修复）
└── icons/
    ├── AppIcon-black.icns
    ├── AppIcon-cyan.icns
    ├── AppIcon-purple.icns
    ├── AppIcon-orange.icns
    ├── AppIcon-navy.icns
    ├── AppIcon-darkgray.icns
    └── previews/          ← 预览图
```

---

## 手动安装

```bash
SKILL_DIR="$HOME/.../wechat-multifix"
ICON="AppIcon-black.icns"  # 换成其他配色

cp -R /Applications/WeChat.app /Applications/WeChat2.app
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist
xattr -cr /Applications/WeChat2.app
codesign --force --deep --sign - /Applications/WeChat2.app
chown -R $(whoami):admin /Applications/WeChat2.app
cp "$SKILL_DIR/icons/$ICON" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app
open -a /Applications/WeChat2.app
```

---

## 微信更新后修复

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
