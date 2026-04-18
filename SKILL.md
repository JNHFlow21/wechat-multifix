---
name: wechat-multifix
description: |
  macOS 微信双开/多开一键安装与修复工具。
  自动完成：复制微信 → 修改 Bundle ID → 重新签名 → 替换黑底白标图标。
  触发词：「微信双开」「安装微信多开」「WeChat2」「微信双开安装」「配置微信双开」「修复微信多开」。
  支持微信更新后的自动修复，聊天记录零丢失。
allowed-tools:
  - Bash(cp) Bash(rm) Bash(xattr) Bash(codesign) Bash(chown) Bash(open) Bash(ls) Bash(stat) Bash(mkdir) Bash(echo) Bash(tee) Bash(killall) Bash(lsregister) Bash(subprocess.run) Bash(sudo)
---

# 微信双开 · macOS 一键安装工具

> 原版微信 + 黑底白标微信2，同时在线，图标一看便知。

---

## 功能概览

| 功能 | 说明 |
|------|------|
| 一键安装 | 复制微信、修改 Bundle ID、签名、替换图标全自动 |
| 区分配色 | WeChat2 使用黑底白标图标，与原版绿色一眼区分 |
| 自动修复 | 微信更新后 Bundle ID 被重置导致打不开？一键修复 |
| 数据保全 | 聊天记录在 ~/Library/Containers/，与应用本身分离 |
| 多开支持 | 支持 WeChat3、WeChat4... 同样流程 |
| 图标可选 | 内置黑底白标图标（默认），也可用原版绿色图标 |

---

## 工作原理

**为什么微信多开会冲突？**
- 每个 macOS 应用有唯一的 Bundle ID
- 微信官方不支持多开，直接复制会导致 Bundle ID 相同
- macOS 遇到相同 Bundle ID 的两个应用，会产生冲突，两者都无法启动
- 解决方法：修改克隆副本的 Bundle ID，让系统识别为不同应用

**为什么更新后会坏？**
- 微信自动更新会覆盖 WeChat2.app 并重置 Bundle ID
- 修复只需重建 WeChat2，聊天记录不受影响

---

## 一键安装 WeChat2（全新安装）

```bash
SKILL_DIR="/Users/fujunhao/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"

# 步骤 1：复制微信应用
cp -R /Applications/WeChat.app /Applications/WeChat2.app

# 步骤 2：修改 Bundle ID
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist

# 步骤 3：修改显示名称
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist

# 步骤 4：清除扩展属性
xattr -cr /Applications/WeChat2.app

# 步骤 5：重新签名
codesign --force --deep --sign - /Applications/WeChat2.app

# 步骤 6：修复所有权
chown -R $(whoami):admin /Applications/WeChat2.app

# 步骤 7：替换图标为黑底白标（内置图标）
cp "$SKILL_DIR/icons/AppIcon-black.icns" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app

# 步骤 8：验证
echo "=== 安装验证 ==="
echo "WeChat Bundle ID: $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat.app/Contents/Info.plist)"
echo "WeChat2 Bundle ID: $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat2.app/Contents/Info.plist)"
echo "Bundle ID 不同 → 安装成功！"
```

---

## 快速启动

```bash
# 启动原版微信
open -a /Applications/WeChat.app

# 启动微信2（延迟 0.5 秒避免冲突）
sleep 0.5 && open -a /Applications/WeChat2.app
```

---

## 一键启动脚本（双开同时启动）

```bash
cat > ~/start-wechat-dual.sh << 'EOF'
#!/bin/bash
open -a /Applications/WeChat.app
sleep 0.5
open -a /Applications/WeChat2.app
EOF
chmod +x ~/start-wechat-dual.sh
~/start-wechat-dual.sh
```

---

## 修复打不开的微信2（微信更新后必用）

微信更新后 WeChat2 打不开，是因为 Bundle ID 被重置了。运行以下命令修复：

```bash
SKILL_DIR="/Users/fujunhao/Library/Mobile Documents/com~apple~CloudDocs/Claude/Content_Creation/.agents/skills/wechat-multifix"

# 检查当前状态
echo "=== 诊断 ==="
echo "原版: $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat.app/Contents/Info.plist)"
[ -f /Applications/WeChat2.app/Contents/Info.plist ] && echo "WeChat2: $(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat2.app/Contents/Info.plist)"

# 如果 Bundle ID 冲突，重建 WeChat2
ORIG_ID=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat.app/Contents/Info.plist)
[ -f /Applications/WeChat2.app/Contents/Info.plist ] && W2_ID=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' /Applications/WeChat2.app/Contents/Info.plist)

if [ "$ORIG_ID" = "$W2_ID" ] || [ ! -f /Applications/WeChat2.app/Contents/Info.plist ]; then
    echo "检测到冲突或损坏，开始修复..."
    rm -rf /Applications/WeChat2.app
    cp -R /Applications/WeChat.app /Applications/WeChat2.app
    /usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat2" /Applications/WeChat2.app/Contents/Info.plist
    /usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat2" /Applications/WeChat2.app/Contents/Info.plist
    xattr -cr /Applications/WeChat2.app
    codesign --force --deep --sign - /Applications/WeChat2.app
    chown -R $(whoami):admin /Applications/WeChat2.app
    cp "$SKILL_DIR/icons/AppIcon-black.icns" /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
    codesign --force --deep --sign - /Applications/WeChat2.app
    echo "修复完成！"
else
    echo "WeChat2 状态正常，无需修复"
fi
```

---

## 安装后 Dock 效果

| 应用 | 配色 |
|------|------|
| WeChat.app | 绿底白标（原版） |
| WeChat2.app | 黑底白标（克隆） |

---

## 安装后提示

首次打开 WeChat2 时，macOS 可能提示「应用已损坏」，这是因为签名改变了。**右键 → 打开** 即可正常运行。

---

## 多开（WeChat3/4/...）

按相同流程，只需替换编号：

```bash
# 以 WeChat3 为例
NUM=3
cp -R /Applications/WeChat.app /Applications/WeChat${NUM}.app
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.tencent.xinWeChat${NUM}" /Applications/WeChat${NUM}.app/Contents/Info.plist
/usr/libexec/PlistBuddy -c "Set :CFBundleName WeChat${NUM}" /Applications/WeChat${NUM}.app/Contents/Info.plist
xattr -cr /Applications/WeChat${NUM}.app
codesign --force --deep --sign - /Applications/WeChat${NUM}.app
chown -R $(whoami):admin /Applications/WeChat${NUM}.app
# 图标替换同理，复制 AppIcon-black.icns 到对应位置
```

---

## 卸载（清理干净）

```bash
# 删除应用
rm -rf /Applications/WeChat2.app

# 删除数据（注意：会删除聊天记录，不可逆）
rm -rf ~/Library/Containers/com.tencent.xinWeChat2

echo "清理完成"
```

---

## 图标说明

内置的 `icons/AppIcon-black.icns` 是**黑底白标**微信图标：
- 从原版微信图标提取形状
- 绿色背景替换为纯黑
- Logo 白色保留
- 与原版轮廓完全一致，配色区分

如果想恢复**原版绿色图标**，运行：
```bash
cp /Applications/WeChat.app/Contents/Resources/AppIcon.icns /Applications/WeChat2.app/Contents/Resources/AppIcon.icns
codesign --force --deep --sign - /Applications/WeChat2.app
```

---

## 触发词

- 「微信双开」
- 「安装微信多开」
- 「WeChat2」
- 「微信双开安装」
- 「配置微信双开」
- 「修复微信多开」
- 「/wechat-multifix」
