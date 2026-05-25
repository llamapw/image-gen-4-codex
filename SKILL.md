---
name: image-gen
description: "Use this skill when the user wants to generate, create, or produce an image/picture/illustration/artwork. Trigger on requests like: 'generate an image of...', 'draw me a...', 'create a picture...', 'make an illustration...', '生成图片', '画一个...', '生成一张...'. Also trigger when the user references creating visual content that requires AI image generation (not diagrams or charts from code). This skill calls an OpenAI-compatible image generation API and displays the result inline."
---

# Image Generation Skill

## Overview

Generates images via an OpenAI-compatible API (base64 response) and displays them inline in Codex.

## Configuration

The user must have a config file at `~/.codex/skills/image-gen/config.json`:

```json
{
  "apiEndpoint": "https://coding.rexai.top/v1/images/generations",
  "apiKey": "your-api-key-here",
  "defaultModel": "gpt-image-1.5",
  "defaultSize": "1024x1024",
  "outputDir": "~/Pictures/codex-generated"
}
```

If config.json does not exist, guide the user to create it before proceeding.

## Workflow

### Step 1: Read Configuration

```powershell
$config = Get-Content "$env:USERPROFILE\.codex\skills\image-gen\config.json" | ConvertFrom-Json
```

If the file doesn't exist, stop and ask the user to configure it first.

### Step 2: Optimize the Prompt

**This is the key differentiator.** Do NOT just pass the user's raw text as the image prompt.

Use the full conversation context to craft an optimized, detailed image prompt:

1. **Extract intent**: What does the user actually want to see?
2. **Add detail from context**: If the conversation discusses a specific project, UI design, character, scene — incorporate those details.
3. **Enhance for quality**: Add style descriptors, composition hints, lighting, atmosphere — but only when they don't conflict with user intent.
4. **Preserve language**: If the user described in Chinese, the final prompt can be in English (better generation quality) but must faithfully represent their intent.

Example transformation:
- User says: "给我画一个登录页面的背景图"
- Context: conversation is about a fintech app with dark theme
- Optimized prompt: "Abstract dark gradient background for a fintech login page, deep navy and midnight blue tones, subtle geometric patterns, modern minimalist style, professional and trustworthy atmosphere, 4K quality"

### Step 3: Call the API

Use PowerShell to make the POST request:

```powershell
$body = @{
    model = $config.defaultModel
    prompt = $optimizedPrompt
    n = 1
    size = $config.defaultSize
} | ConvertTo-Json -Compress

$headers = @{
    "Authorization" = "Bearer $($config.apiKey)"
    "Content-Type" = "application/json"
}

$response = Invoke-RestMethod -Uri $config.apiEndpoint -Method POST -Headers $headers -Body $body
```

### Step 4: Decode Base64 and Save

The API returns base64-encoded image data in `data[0].b64_json`:

```powershell
$b64 = $response.data[0].b64_json
$bytes = [System.Convert]::FromBase64String($b64)

# Generate output path
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$outputDir = $config.outputDir -replace '~', $env:USERPROFILE
if (-not (Test-Path $outputDir)) { New-Item -ItemType Directory -Path $outputDir -Force }
$outputPath = Join-Path $outputDir "image_$timestamp.png"

[System.IO.File]::WriteAllBytes($outputPath, $bytes)
```

### Step 5: Display the Image

Display the saved PNG file inline with a Markdown image tag that uses the absolute file path:

```markdown
![Generated image](C:\absolute\path\to\image.png)
```

Then tell the user:
- The image has been generated and saved to `$outputPath`
- Briefly describe what was generated
- Offer to regenerate with different parameters if needed

## Parameters

| Parameter | Default | Options |
|-----------|---------|---------|
| model | gpt-image-1.5 | As configured |
| size | 1024x1024 | 1024x1024, 1792x1024, 1024x1792 |
| n | 1 | 1-4 |
| quality | standard | standard, hd |

## Error Handling

- **No config file**: Guide user to create `~/.codex/skills/image-gen/config.json`
- **API error (401)**: API key is invalid, ask user to check their key
- **API error (429)**: Rate limited, wait and retry or inform user
- **API error (400)**: Prompt may be rejected (content policy), inform user and suggest modification
- **Empty b64_json**: API may have returned URL format instead; check for `data[0].url` as fallback

## Notes

- Always show the optimized prompt to the user before/after generation so they understand what was sent
- The generated image file persists on disk for the user to use later
- If the user wants multiple variations, increment n or call multiple times
- Respect content policies — do not generate prompts that would violate API terms
