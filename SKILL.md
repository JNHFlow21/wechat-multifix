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

## ⚠️ 重要提醒（安装前必读）

### 🔄 微信自动更新会导致双开失效

**问题原因：**
微信每次自动更新（App Store 或自动更新）会覆盖 `WeChat2.app`，并把 Bundle ID 重置为与原版相同。此时 macOS 认定两个微信是"同一个应用"，导致**两者都无法启动**。

**表现：**
- 微信更新后，WeChat2 打不开了
- 或者原版微信和 WeChat2 都打不开
- 提示"应用已损坏"或无任何反应

**聊天记录不会丢失** —— 记录存在 `~/Library/Containers/com.tencent.xinWeChat2/`，与应用本身分离。

**解决方法超级简单 —— 运行「修复微信多开」即可：**

```
修复微信多开
```

Agent 会自动重建 WeChat2 并恢复图标，全程保留聊天记录。

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

### 安装前必须先询问配色

告诉用户有6种配色可选，请用户回复编号（1-6）或配色名。如果用户没有明确选择，使用默认的黑底白标。

### 安装后必须提醒用户两件事

安装完成后，必须告知用户以下内容：

> **✅ 安装完成！**
>
> **⚠️ 重要提醒：**
> 微信自动更新后，WeChat2 会失效打不开，这是正常现象（Bundle ID 被重置）。
>
> **解决方法：**
> 出现这个问题时，直接说「修复微信多开」，Agent 会自动重建，全程保留聊天记录。
>
> **建议：**
> 也可以在「系统设置 → 通用 → 自动更新」中关闭微信自动更新，需要时手动修复。
>
> ---
>
> **🔔 通知权限（必须手动开启）：**
> WeChat2 安装后需要手动开启通知权限，Agent 无法自动代劳。
>
> **开启方法：**
> 1. 打开 WeChat2，进入「设置 → 通知」
> 2. 确保「新消息通知」已开启
> 3. 同时在 macOS「系统设置 → 通知」中找到 WeChat2，确保允许通知
>
> **如果 Dock 没有红点提示**，大概率是这里没开。

### 安装命令

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

## 修复微信多开（打不开时用）

**触发词：「修复微信多开」**

当用户说「微信打不开了」「WeChat2 打不开」「微信更新后打不开」，直接执行此修复流程。

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
- 「修复微信多开」 ← 重点
- 「微信三开」
- 「微信多开图标」
- 「/wechat-multifix」
