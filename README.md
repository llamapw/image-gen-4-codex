# Image Generation Skill for Codex

在 Codex 中直接生成 AI 图像，支持上下文感知的提示词优化。

## 安装

此 skill 已安装在 `~/.codex/skills/image-gen/` 目录下。

## 配置

编辑 `~/.codex/skills/image-gen/config.json`：

```json
{
  "apiEndpoint": "https://your-api-relay.com/v1/images/generations",
  "apiKey": "your-api-key",
  "defaultModel": "gpt-image-1.5",
  "defaultSize": "1024x1024",
  "outputDir": "~/Pictures/codex-generated"
}
```

### 配置说明

| 字段 | 说明 | 必填 |
|------|------|------|
| apiEndpoint | API 中转站的图像生成端点 | 是 |
| apiKey | 你的 API Key | 是 |
| defaultModel | 默认使用的模型 | 是 |
| defaultSize | 默认图像尺寸 | 否 |
| outputDir | 图像保存目录 | 否 |

### 支持的尺寸

- `1024x1024` — 正方形（默认）
- `1792x1024` — 横向宽幅
- `1024x1792` — 纵向竖幅

## 使用方式

在 Codex 对话中直接说：

- "生成一张图片：一只猫在阳光下"
- "帮我画一个网站的 hero 背景图"
- "generate an image of a mountain landscape"

Skill 会自动：
1. 根据对话上下文优化你的提示词
2. 调用 API 生成图像
3. 将图像保存到本地并在终端中展示

## 工作原理

1. **提示词优化**：不是简单传递用户输入，而是结合对话上下文（当前项目、讨论的主题、设计需求等）智能扩展提示词
2. **API 调用**：通过 PowerShell 发送 POST 请求到配置的中转站端点
3. **Base64 解码**：API 返回 base64 编码的图像数据，skill 解码后保存为 PNG 文件
4. **内联展示**：使用 Markdown 图片标签在 Codex 中展示图像

## 故障排除

- **图片未生成**：检查 config.json 中的 apiKey 是否正确
- **401 错误**：API Key 无效或过期
- **429 错误**：请求频率过高，稍后重试
- **图片质量不满意**：可以要求重新生成，并提供更具体的描述
