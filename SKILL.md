---
name: wechat-multifix
description: |
  macOS 微信双开/多开一键安装与修复工具。
  自动完成：复制微信 → 修改 Bundle ID → 重新签名 → 替换主题配色图标（黑/青/紫/橙/深蓝/深灰）。
  触发词：「微信双开」「安装微信多开」「WeChat2」「微信双开安装」「配置微信双开」「修复微信多开」「微信三开」「微信多开图标」。
  支持微信更新后的自动修复，聊天记录零丢失。
allowed-tools:
  - Bash(cp) Bash(rm) Bash(xattr) Bash(codesign) Bash(chown) Bash(open) Bash(ls) Bash(stat) Bash(mkdir) Bash(echo) Bash(tee) Bash(killall) Bash(lsregister) Bash(subprocess.run) Bash(sudo) Bash(iconutil)
---

# 微信双开 · macOS 一键安装工具

> 原版微信 + 6种主题配色图标，一眼看清楚用的是哪个账号。

---

## ⚠️ 安装前必读

**这是安装流程，修复流程在下方「修复打不开的微信2」。**

安装 WeChat2 之前，需要先询问用户想要哪个配色。

---

## 内置图标（6种）

| 编号 | 图标文件 | 配色 | 主题感 |
|------|----------|------|--------|
| 1 | `AppIcon-black.icns` | ⬛ 黑底白标 | 低调、暗黑模式首选 |
| 2 | `AppIcon-cyan.icns` | 🔵 青底白标 | 清新、科技感 |
| 3 | `AppIcon-purple.icns` | 🟣 紫底白标 | 优雅、创意 |
| 4 | `AppIcon-orange.icns` | 🟠 橙底白标 | 活力、温暖 |
| 5 | `AppIcon-navy.icns` | 🔷 深蓝底白标 | 沉稳、商务 |
| 6 | `AppIcon-darkgray.icns` | ⬜ 深灰底白标 | 低调、简洁 |

**默认：编号 1（黑底白标）**，如果用户没有指定配色，则使用默认值。

---

## 一键安装 WeChat2（全新安装）

**安装前必须先询问配色**：告诉用户有6种配色可选，请用户回复编号（1-6）或配色名。如果用户没有明确选择，使用默认的黑底白标。

安装命令：

```bash
SKILL_DIR="$HOME/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"
ICON_NAME="AppIcon-black.icns"  # 根据用户选择替换：cyan / purple / orange / navy / darkgray

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

# 7. 替换图标
cp "$SKILL_DIR/icons/$ICON_NAME" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app

# 8. 验证
echo "=== 安装完成 ==="
echo "WeChat:  $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat.app/Contents/Info.plist)"
echo "WeChat2: $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat2.app/Contents/Info.plist)"
echo ""
echo "✅ 可以运行以下命令启动："
echo "open -a /Applications/WeChat2.app"
```

---

## 快速启动

```bash
open -a /Applications/WeChat.app
sleep 0.5 && open -a /Applications/WeChat2.app
```

---

## 修复打不开的微信2（微信更新后）

**修复时也需要询问配色**（如果用户没有指定，默认为黑底白标）：

```bash
SKILL_DIR="$HOME/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"
ICON_NAME="AppIcon-black.icns"

ORIG_ID=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat.app/Contents/Info.plist)
[ -f /Applications/WeChat2.app/Contents/Info.plist ] && W2_ID=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat2.app/Contents/Info.plist)

if [ "$ORIG_ID" = "$W2_ID" ] || [ ! -f /Applications/WeChat2.app/Contents/Info.plist ]; then
    rm -rf /Applications/WeChat2.app
    cp -R /Applications/WeChat.app /Applications/WeChat2.app
    /usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist
    /usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist
    xattr -cr /Applications/WeChat2.app
    codesign --force --deep --sign - /Applications/WeChat2.app
    chown -R $(whoami):admin /Applications/WeChat2.app
    cp "$SKILL_DIR/icons/$ICON_NAME" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
    codesign --force --deep --sign - /Applications/WeChat2.app
    echo "✅ 修复完成！运行 open -a /Applications/WeChat2.app 启动"
else
    echo "WeChat2 状态正常，无需修复"
fi
```

---

## 多开（WeChat3/4/5...）

替换编号，**询问用户想要哪个配色**：

```bash
NUM=3
ICON_NAME="AppIcon-cyan.icns"  # 询问用户选择

rm -rf /Applications/WeChat${NUM}.app
cp -R /Applications/WeChat.app /Applications/WeChat${NUM}.app
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat${NUM}" /Applications/WeChat${NUM}.app/Contents/Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat${NUM}" /Applications/WeChat${NUM}.app/Contents/Info.plist
xattr -cr /Applications/WeChat${NUM}.app
codesign --force --deep --sign - /Applications/WeChat${NUM}.app
chown -R $(whoami):admin /Applications/WeChat${NUM}.app
cp "$SKILL_DIR/icons/$ICON_NAME" /Applications/WeChat${NUM}.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat${NUM}.app
```

---

## 安装后提示

首次打开时 macOS 提示「应用已损坏」→ **右键 → 打开** 即可。

---

## 卸载

```bash
rm -rf /Applications/WeChat2.app
rm -rf ~/Library/Containers/com.tencent.xinWeChat2  # 聊天记录（不可逆）
```

---

## 工作原理

- **Bundle ID**：每个 macOS 应用唯一标识，直接复制会导致冲突
- **解决方法**：修改克隆版的 Bundle ID + 重新签名
- **数据存储**：`~/Library/Containers/com.tencent.xinWeChat2/`（与应用分离，修复不丢数据）

---

## 触发词

- 「微信双开」
- 「安装微信多开」
- 「WeChat2」
- 「微信双开安装」
- 「配置微信双开」
- 「修复微信多开」
- 「微信三开」
- 「微信多开图标」
- 「/wechat-multifix」
