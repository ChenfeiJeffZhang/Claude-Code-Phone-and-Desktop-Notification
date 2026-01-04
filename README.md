# Claude Code Phone and Desktop Notification / Claude Code电话和桌面通知

> **EN:** Automatically notify you when Claude Code completes a task or needs your input.
> 
> **中文:** 让 Claude Code 完成任务或等待输入时，自动拨打电话并在电脑桌面通知你。

```
Claude Code needs confirmation → Runs cc command → Toast + Phone call
Claude Code 需要确认 → 执行 cc 命令 → Toast 弹窗 + 电话
```

- ✅ Auto hang-up if unanswered (configurable) / 无人接自动挂断（可配置秒数）
- ✅ No charge if unanswered / 未接通不收费
- ✅ ~$0.02/min when connected / 接通约 $0.02/分钟

---

## Quick Start / 快速开始

### 1. Register Twilio / 注册 Twilio

1. Sign up at / 注册：https://www.twilio.com/try-twilio
2. Get **Account SID** & **Auth Token** from [Twilio Console](https://console.twilio.com)
3. Buy a Voice-enabled number (~$1/month) / 购买支持 Voice 的号码

### 2. Verify Your Phone / 验证手机号

> ⚠️ **Trial accounts can only call verified numbers / 试用账户必须验证接收号码**

1. Go to / 前往 [Verified Caller IDs](https://console.twilio.com/us1/develop/phone-numbers/manage/verified)
2. Click **Add a new Caller ID**
3. Enter your phone (e.g. `+8613800138000` or `+14155551234`)
4. Complete verification / 完成验证

### 3. Install Toast Module (Optional) / 安装 Toast 模块（可选）

```powershell
Install-Module -Name BurntToast -Force
```

### 4. Configure PowerShell Profile / 配置 PowerShell Profile

```powershell
# Open Profile / 打开 Profile
notepad $PROFILE

# Create if not exists / 不存在则创建
if (!(Test-Path $PROFILE)) { New-Item -Path $PROFILE -Force }
```

Paste / 粘贴：

```powershell
# Claude Code Notification (Toast + Phone)
function cc {
    # ===== Toast =====
    Import-Module BurntToast
    New-BurntToastNotification -Text "Claude Code", "Input needed" -Sound Default
    
    # ===== Twilio =====
    $AccountSid = "YOUR_ACCOUNT_SID"      # ← Replace / 替换
    $AuthToken  = "YOUR_AUTH_TOKEN"       # ← Replace / 替换
    $From = "+1xxxxxxxxxx"                # ← Your Twilio number / Twilio 号码
    $To   = "+xxxxxxxxxxx"                # ← Your phone / 你的手机号
    
    # Change language: en-US / zh-CN / 修改语言
    $twiml = '<Response><Say language="zh-CN" voice="alice">Claude Code 需要你的确认</Say></Response>'
    $auth = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("${AccountSid}:${AuthToken}"))
    
    try {
        Invoke-RestMethod -Uri "https://api.twilio.com/2010-04-01/Accounts/$AccountSid/Calls.json" `
            -Method POST -Headers @{Authorization = "Basic $auth"} `
            -Body @{From = $From; To = $To; Twiml = $twiml; Timeout = 10}  # ← Timeout seconds / 超时秒数
        Write-Host "✓ Phone notification sent" -ForegroundColor Green
    } catch {
        Write-Host "✗ Failed: $_" -ForegroundColor Red
    }
}
```

### 5. Apply & Test / 生效并测试

```powershell
. $PROFILE
cc
```

---

## Auto-Trigger in Claude Code / 自动触发配置

### Option A: Hooks (Recommended / 推荐)

Create / 创建 `.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "powershell -Command \"cc\"", "timeout": 10 }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": { "tool_name": "AskUserQuestion" },
        "hooks": [{ "type": "command", "command": "powershell -Command \"cc\"", "timeout": 10 }]
      }
    ]
  }
}
```

Create / 创建 `.claude/settings.local.json` (don't commit / 不要提交 Git):

```json
{
  "permissions": {
    "allow": ["Bash(powershell -Command \"cc\")", "Bash(cc)"]
  }
}
```

**Hook Events / 事件说明:**

| Event | Trigger / 触发时机 |
|-------|-------------------|
| `Stop` | AI stops responding / AI 停止响应 |
| `PostToolUse` | After tool execution / 工具调用后 |

**Tool Names / 工具名称:** `AskUserQuestion`, `EnterPlanMode`, `ExitPlanMode`, `Bash`, `Read`, `Write`, `Edit`

### Option B: CLAUDE.md (Alternative / 备选)

> ⚠️ Less stable than Hooks / 不如 Hooks 稳定

Add to `CLAUDE.md`:

```markdown
## User Notification (MUST FOLLOW)

**ALWAYS run `powershell -Command "cc"` in these situations:**
1. After completing any task
2. When asking user a question
3. When waiting for user confirmation

**Do NOT run `cc`:** After user answers, or mid-task.
```

---

## Pricing / 费用

| Item | Cost |
|------|------|
| Twilio Number / 号码 | ~$1/month |
| Connected / 接通 | ~$0.02/min |
| Unanswered / 未接通 | Free / 免费 |
| Trial Credit / 试用余额 | ~$15-20 |

---

## Customization / 自定义

**Voice Message / 播报内容:**
```powershell
# English
$twiml = '<Response><Say language="en-US" voice="alice">Claude Code needs confirmation</Say></Response>'

# 中文
$twiml = '<Response><Say language="zh-CN" voice="alice">Claude Code 需要你的确认</Say></Response>'
```

**Toast Only / 只要 Toast:** Remove everything below `# ===== Twilio =====`

**Phone Only / 只要电话:** Remove the two `BurntToast` lines

**Toast Sounds / 声音:** `Default` (~1s), `IM` (~1s), `SMS` (~1s), `Alarm2` (~10s)

**Timeout / 超时:** Change `Timeout = 10` to any seconds (e.g. `Timeout = 15`) / 修改 `Timeout = 10` 为任意秒数

---

## FAQ

**Q: No call received? / 没收到电话？**
→ Verify your number at [Verified Caller IDs](https://console.twilio.com/us1/develop/phone-numbers/manage/verified)

**Q: English prompt at start? / 开头有英文提示？**
→ Trial limitation. Upgrade to remove. / 试用账户限制，升级后消失

**Q: Charges for unanswered? / 不接收费吗？**
→ No. Timeout prevents voicemail. / 不收费，超时挂断不会转语音信箱

---

## Links

- [Twilio Console](https://console.twilio.com) | [Pricing](https://www.twilio.com/pricing)
- [BurntToast](https://github.com/Windos/BurntToast)
- [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)

## License

MIT
