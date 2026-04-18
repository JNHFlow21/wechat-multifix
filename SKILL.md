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

## 功能概览

| 功能 | 说明 |
|------|------|
| 一键安装 | 复制微信 → 修改 Bundle ID → 签名 → 替换图标全自动 |
| 6种配色 | 黑、青、紫、橙、深蓝、深灰，与原版绿色一眼区分 |
| 自动修复 | 微信更新后打不开？自动检测并重建 |
| 数据保全 | 聊天记录在 ~/Library/Containers/，与应用分离 |
| 多开支持 | WeChat2/3/4/5... 同样流程 |

---

## 内置图标

| 图标文件 | 配色 | 预览 |
|----------|------|------|
| AppIcon-black.icns | 黑底白标 | 🖤 |
| AppIcon-cyan.icns | 青底白标 | 🔵 |
| AppIcon-purple.icns | 紫底白标 | 🟣 |
| AppIcon-orange.icns | 橙底白标 | 🟠 |
| AppIcon-navy.icns | 深蓝底白标 | 🔷 |
| AppIcon-darkgray.icns | 深灰底白标 | ⬛ |

---

## 工作原理

**为什么微信多开会冲突？**
- 每个 macOS 应用有唯一的 Bundle ID
- 直接复制会导致 Bundle ID 相同，系统冲突，两者都无法启动
- 解决方法：修改克隆版的 Bundle ID，重新签名

**为什么更新后会坏？**
- 微信自动更新会覆盖 WeChat2.app 并重置 Bundle ID
- 修复只需重建，聊天记录不受影响

---

## 一键安装 WeChat2（全新安装）

```bash
SKILL_DIR="$HOME/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"
ICON_NAME="AppIcon-black.icns"  # 可选: AppIcon-cyan / AppIcon-purple / AppIcon-orange / AppIcon-navy / AppIcon-darkgray

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

# 7. 替换图标（默认黑底白标）
cp "$SKILL_DIR/icons/$ICON_NAME" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app

# 8. 验证
echo "WeChat:  $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat.app/Contents/Info.plist)"
echo "WeChat2: $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat2.app/Contents/Info.plist)"
```

---

## 快速启动

```bash
open -a /Applications/WeChat.app
sleep 0.5 && open -a /Applications/WeChat2.app
```

---

## 修复打不开的微信2（微信更新后）

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
    echo "修复完成！"
else
    echo "WeChat2 状态正常"
fi
```

---

## 多开（WeChat3/4/5...）

替换编号和图标即可：

```bash
NUM=3
ICON_NAME="AppIcon-cyan.icns"  # 每个实例用不同配色

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
