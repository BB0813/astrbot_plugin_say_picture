# astrbot_plugin_say_picture

当检测到群消息中 **@某人 说～xxx**（或 **@某人 说 xxx**）格式时，自动生成聊天气泡表情包并发送。「说」后必须有分隔符（空格或～）。

基于 [AstrBot](https://github.com/AstrBotDevs/AstrBot) 框架开发，兼容 OneBot v11 标准；在 QQ 官方机器人等非 OneBot 平台会优雅降级（见「平台兼容」）。

---

## 效果预览

发送以下任意一条消息：

```
@某人 说～你好呀
@某人 说 今天天气真好
```

机器人将生成一张聊天截图表情包：

- 🧑 圆形头像
- 🏷️ 等级徽章（LV<等级>）+ 群头衔（群主/管理员/段位或自定义头衔），底色随身份变化
- ✍️ 昵称（徽章右侧；归一化 + 字体回退，花体/特殊符号也不会空白，#59）
- 💬 白色聊天气泡（位置上移、文字随气泡跟随，#60），气泡内显示你说的话

---

## 安装

### 方式一：插件市场（推荐）

在 AstrBot WebUI 插件市场搜索 `astrbot_plugin_say_picture`，一键安装。

### 方式二：手动安装

```bash
# 克隆仓库
git clone https://github.com/mjy1113451/astrbot_plugin_say_picture.git

# 将内容放入 AstrBot 插件目录后重启
# 通常目录为：AstrBot/plugins/
```

---

## 配置

插件有一个开关配置项：

```json
// config.json
{
  "enabled": true   // true=开启，false=关闭
}
```

在 AstrBot WebUI 的插件配置页面也可以直接修改。

---

## 工作原理

```
用户发送 "@某人 说～内容" 或 "@某人 说 内容"
    ↓
插件拦截消息链，提取 At 组件 + "说" 分隔符后面的文字
    ↓
跨平台身份解析：OneBot get_group_member_info → QLogo CDN（仅数字 QQ）→ 占位头像（线程池异步拉取）
    ↓
PIL 渲染聊天截图（圆形头像 + 昵称 + LV 徽章/群头衔 + 气泡）
    ↓
以图片消息形式发送回聊天（通用 result 接口，跨平台）
```

### 技术细节

| 项目 | 说明 |
|------|------|
| 图片渲染 | PIL (Pillow)，`rendering/` 包（chat_screenshot / font_manager） |
| 中文字体 | CDN 下载 Noto Sans SC（manifest SHA256 校验）→ 内置 msyh/韩文子集字符级回退；NFKC 归一化 + 空段回退保证昵称不空白（#59） |
| 头像获取 | OneBot `get_group_member_info` → `q1.qlogo.cn`（仅数字 QQ、http(s) 协议校验）→ 占位头像兜底；线程池异步拉取不阻塞事件循环 |
| 输出格式 | JPEG（quality 90） |

### 平台兼容（issue #61）

渲染与发送走 AstrBot 通用消息接口，跨平台可用；仅「群成员信息（头像/昵称/等级/头衔）」依赖 OneBot，非 OneBot 平台优雅降级：

| 平台 | 头像 | 昵称 / 等级 / 头衔 |
|------|------|--------------------|
| OneBot v11（aiocqhttp 等） | 成员信息 avatar → QLogo CDN | 完整（card/nickname/level/title） |
| QQ 官方机器人等 | 占位头像（纯色圆） | 昵称回退 `用户<id>`，等级 LV0、无头衔 |

即在官方机器人上仍能正常出图，只是头像为占位圆、等级/头衔为默认值。

---

## 项目结构

```
astrbot_plugin_say_picture/
├── fonts/
│   ├── msyh-subset.ttf       # 内置中文字体（微软雅黑子集）
│   └── hangul-subset.ttf     # 内置韩文子集（字符级回退 #52）
├── rendering/
│   ├── __init__.py
│   ├── chat_screenshot.py    # 聊天截图渲染（昵称/LV 徽章/头衔/气泡）
│   └── font_manager.py       # 字体 CDN 下载（manifest SHA256 校验）
├── resources/
│   └── corner1.png ~ corner4.png  # 气泡圆角素材
├── .gitignore
├── _conf_schema.json          # 配置 schema
├── config.json               # 插件配置
├── LICENSE                   # AGPL-3.0
├── main.py                   # 插件主逻辑（触发/身份解析/发送）
├── metadata.yaml             # 插件元信息
└── README.md
```

---

## 依赖

- Python ≥ 3.10
- Pillow (`pip install Pillow`)
- astrbot ≥ 对应版本（由 AstrBot 框架提供）

---

## 许可证

[AGPL-3.0](./LICENSE)

> 修改后的代码必须开源，网络服务使用本项目也必须开源。

---

## 作者

[mjy1113451](https://github.com/mjy1113451)

[作者的插件群](https://qun.qq.com/universal-share/share?ac=1&authKey=LPNpKBYL2R3WqjbvADCCIA8XpZ%2Fqmitz2xpdkNNlOin%2BPz6ez8UCrbZDgneoR762&busi_data=eyJncm91cENvZGUiOiIxMDc1OTIwMzIzIiwidG9rZW4iOiJ0bG5BWjhZN2xkZXFlVk1WclZRNGNDVElCRW9CRG9ieFZzUEFwMEljaXNibzVOUUc1YkJSOC9NRnI4NTBOcEhkIiwidWluIjoiMTczMTUzODMzNCJ9&data=hV1Jra4PMbmVn17xmFmtJ-UBqIGZ7rBtO31vMQXm2h5RmBxeNr6YKfDa7FJAUe3MJS75Vl4LP7piI4YxWV2fOg&svctype=4&tempid=h5_group_info)

如有问题或建议，欢迎提交 Issue 或联系作者。
