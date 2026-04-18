# 微信双开 · macOS 一键安装工具

macOS 微信双开/多开自动化安装与修复工具，内置**黑底白标图标**，一眼区分原版与克隆。

## 功能特点

- **一键安装**：复制微信 → 修改 Bundle ID → 签名 → 替换图标全自动
- **图标区分**：WeChat2 使用黑底白标，与原版绿色一眼区分
- **自动修复**：微信更新后打不开？一键检测并重建
- **数据保全**：聊天记录在 `~/Library/Containers/`，与应用分离
- **多开支持**：WeChat3、WeChat4... 同理可加

## 安装效果

| 应用 | 图标配色 |
|------|---------|
| WeChat.app | 绿底白标（原版） |
| WeChat2.app | 黑底白标（克隆） |

---

## Quick Start

把下面这段 prompt **完整复制**给你的 AI Agent（Claude Code、Openclaw、Hermes 等），它会自动完成所有配置：

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
2. 执行完整安装流程
3. 替换为内置的黑底白标图标
4. 验证 Bundle ID 唯一性
5. 提示可以启动

---

## 手动安装（不用 Agent）

如果你想自己操作，在终端运行：

```bash
SKILL_DIR="$HOME/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"

# 1. 复制微信
cp -R /Applications/WeChat.app /Applications/WeChat2.app

# 2. 修改 Bundle ID
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist

# 3. 修改显示名称
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist

# 4. 清除扩展属性
xattr -cr /Applications/WeChat2.app

# 5. 重新签名
codesign --force --deep --sign - /Applications/WeChat2.app

# 6. 修复所有权
chown -R $(whoami):admin /Applications/WeChat2.app

# 7. 替换黑底白标图标
cp "$SKILL_DIR/icons/AppIcon-black.icns" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app

# 8. 启动
open -a /Applications/WeChat2.app
```

---

## 微信更新后修复

微信自动更新会重置 Bundle ID，导致 WeChat2 打不开。运行以下命令修复：

```bash
SKILL_DIR="$HOME/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"

# 检测并修复
rm -rf /Applications/WeChat2.app
cp -R /Applications/WeChat.app /Applications/WeChat2.app
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist
xattr -cr /Applications/WeChat2.app
codesign --force --deep --sign - /Applications/WeChat2.app
chown -R $(whoami):admin /Applications/WeChat2.app
cp "$SKILL_DIR/icons/AppIcon-black.icns" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app
```

或者把这段 prompt 扔给 Agent：

```
修复微信多开。微信更新后 WeChat2 打不开了。
读取 .agents/skills/wechat-multifix/SKILL.md，按「修复打不开的微信2」的步骤执行。
```

---

## 目录结构

```
wechat-multifix/
├── SKILL.md              ← Skill 说明文件（给 Agent 看的）
└── icons/
    └── AppIcon-black.icns ← 内置黑底白标图标
```

---

## 技术原理

macOS 要求每个应用有唯一的 `CFBundleIdentifier`。直接复制微信会导致 Bundle ID 相同，系统产生冲突。解决方法：

1. 复制微信应用
2. 修改克隆版的 Bundle ID（如 `com.tencent.xinWeChat2`）
3. 重新签名，让系统接受这个"新"应用
4. 替换图标区分

聊天记录存在 `~/Library/Containers/com.tencent.xinWeChat2/`，与应用本身分离，修复流程不丢失数据。

---

## License

MIT
