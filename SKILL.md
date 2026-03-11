---
name: openclaw-deploy-ninja
description: >
  End-to-end one-shot OpenClaw installer for macOS, Windows, and Linux.
  Use when the user wants OpenClaw installed, onboarded, configured with models,
  channels, skills, hooks, and startup verification in one continuous guided flow.
  Triggers: "install openclaw", "one-click openclaw setup", "full openclaw onboarding",
  "setup openclaw with models/channels/skills/hooks".
---

# OpenClaw Deploy Ninja - Usage Guide

**Description:** End-to-end one-shot OpenClaw installer for macOS/Windows/Linux. This skill must complete install + onboard + model + channels + skills + hooks in one continuous flow (no "please run this command yourself" handoff).

## Triggers
- "install openclaw"
- "one-click openclaw setup"
- "full openclaw onboarding"
- "setup openclaw with models/channels/skills/hooks"

## Non-Negotiable Behavior
1. Never ask the user to manually run `openclaw onboard` in another terminal.
2. Keep the whole process in the same session: preflight -> install -> onboard -> channels -> hooks -> verification.
3. Ask questions only for values you cannot infer (install path, auth method, API keys/tokens, optional channel credentials).
4. Install required dependencies automatically (best effort) before OpenClaw install.
5. Use system Node.js + npm (do not use bundled Node installers).
6. Do not spawn a separate Terminal app for setup wizards (`openclaw china setup`, channel setup). Keep wizard interaction in the current assistant session.
7. Fail fast on errors and report exact failing step + command + stderr. Do not silently skip or auto-downgrade auth steps.
8. Ensure gateway autostart is enabled before finishing install so OpenClaw starts after OS reboot.

## Locked Default Workflow (mandatory, no mode selection)
This skill is hard-locked to one default workflow. Do not ask for default/custom mode and do not branch to delayed configuration.

Fixed execution order (must be completed in one continuous session):
1. Install OpenClaw.
2. Enter Codex authorization flow and set default model to `gpt-codex-5.3`.
3. Run non-interactive OpenClaw auto-configuration (hooks, skills, and security baseline).
4. Install OpenClaw China plugin from `https://github.com/BytePioneer-AI/openclaw-china` via:
   - `openclaw plugins install @openclaw-china/channels`
5. Run Telegram setup prompt with exactly one input item:
   - `输入你的机器人Token`
   - After input, auto-enable and configure Telegram plugin immediately.
6. Immediately enter OpenClaw China channel configuration (in the same install flow, never postponed):
   - When user selects a channel, show exactly one input item: `输入配置信息`.
   - Under the input item, show one concrete channel-specific example explaining the expected format/content.
7. Complete remaining verification and startup steps, open WebUI, then print install summary and command checklist.

Hard gates:
- It is forbidden to defer OpenClaw China configuration to the end with prompts like `是否需要继续配置openclaw-china插件`.
- Do not mark installation complete until Telegram + selected OpenClaw China channels are configured in-session.
- Keep all wizard interactions in the current assistant session.

## Telegram and OpenClaw China Input UX (single-field rules)
Telegram input UX (fixed):
- Only one prompt option: `输入你的机器人Token`.
- After user enters token, apply config in-session and verify by `openclaw channels status --probe`.

OpenClaw China input UX (fixed):
- After plugin install, enforce a strict two-question protocol in the same phase.
- Question 1 (channel selection, multi-select): show supported channels from `@openclaw-china/channels` only:
  - `dingtalk`
  - `qqbot`
  - `wecom`
  - `wecom-app`
  - `feishu-china`
- Question 2 (configuration input stage): for all channels selected in Question 1, ask `输入配置信息` for each selected channel in the same stage.
- Normalize common typo `webcom` -> `wecom` before applying config.
- Under this input, always display one example for that channel. Example formats:
  - `dingtalk`: `clientId=ding_xxx;clientSecret=xxx`
  - `qqbot`: `appId=1024;clientSecret=xxx`
  - `wecom`: `botId=ww_xxx;secret=xxx`
  - `wecom-app`: `token=xxx;encodingAESKey=xxx;corpId=wwxxx;corpSecret=xxx;agentId=1000002;webhookPath=/wecom/callback`
  - `feishu-china`: `appId=cli_xxx;appSecret=xxx`

Parse and apply rules:
- Parse key-value input by `;` delimiters and `=` pairs.
- Reject empty required keys and ask re-entry immediately.
- Apply config immediately, then run `openclaw channels status --probe`.
- Do not add extra questionnaire steps beyond the two-question protocol.

Default onboarding command (always use):
```bash
openclaw onboard \
  --accept-risk \
  --flow quickstart \
  --mode local \
  --auth-choice openai-codex \
  --skip-channels \
  --skip-skills \
  --skip-search \
  --skip-health \
  --skip-ui \
  --install-daemon \
  --node-manager npm
```

## Workflow

### Phase 0: Preflight Dependencies (auto-install)
Install missing base tools using system package manager.

Required bins:
- `bash`, `curl`, `git`, `jq`, `tar`, `node`, `npm`

Best-effort installer logic:
1. Detect missing bins via `command -v`.
2. If none are missing, continue.
3. If missing:
   - macOS + Homebrew: `brew install curl git jq`
   - Debian/Ubuntu: `sudo apt-get update && sudo apt-get install -y curl git jq ca-certificates`
   - Fedora/RHEL: `sudo dnf install -y curl git jq ca-certificates`
   - Arch: `sudo pacman -Sy --noconfirm curl git jq ca-certificates`
4. Re-check bins and fail with clear actionable output only if still missing.

### Phase 0.1: System Node policy (required)
Always use system Node runtime.

Rules:
1. Detect version:
```bash
node -v
```
2. If Node is missing, install Node LTS/latest using system method:
   - macOS (Homebrew): `brew install node`
   - Debian/Ubuntu: `sudo apt-get update && sudo apt-get install -y nodejs npm`
   - Fedora/RHEL: `sudo dnf install -y nodejs npm`
   - Arch: `sudo pacman -Sy --noconfirm nodejs npm`
   - Windows: `winget install OpenJS.NodeJS.LTS`
3. If major version `< 22`, update to latest supported (22+):
   - macOS (Homebrew): `brew upgrade node`
   - Debian/Ubuntu/Fedora/Arch: upgrade Node via package manager or official NodeSource workflow
   - Windows: `winget upgrade OpenJS.NodeJS.LTS`
4. Verify final version is `>=22` before installing OpenClaw.

### Phase 0.2: npm TLS/certificate preflight (required)
Before `npm install -g openclaw@latest`, always validate npm TLS connectivity.

Run:
```bash
npm ping
npm view openclaw version
```

If output contains certificate verification errors (for example `UNABLE_TO_GET_ISSUER_CERT`, `SELF_SIGNED_CERT_IN_CHAIN`, `unknown certificate verification error`), apply this remediation order:

0. Retry once first (transient network/CDN/proxy glitches are common):
```bash
npm ping
npm install -g openclaw@latest
```
Only proceed to certificate remediation if retry still fails with certificate verification error.

1. Verify system time and retry (TLS fails on bad clock):
```bash
date
```

2. Set a reliable registry and retry:
```bash
npm config set registry https://registry.npmjs.org
npm ping
```

3. If user is behind enterprise proxy/custom CA, ask for CA certificate path and configure npm + Node:
```bash
npm config set cafile "<CA_CERT_PATH>"
export NODE_EXTRA_CA_CERTS="<CA_CERT_PATH>"
npm ping
```

4. Last-resort temporary bypass (only when user explicitly accepts risk):
```bash
npm config set strict-ssl false
npm install -g openclaw@latest
npm config set strict-ssl true
```

Guardrails:
- Never silently disable SSL verification.
- Prefer CA-based fix over `strict-ssl=false`.
- If certificate issue persists, stop with actionable error and exact next command.

### Phase 1: Install OpenClaw (isolated prefix)
Let `<target_dir>` be selected install path.

Run (macOS/Linux):
```bash
npm install -g openclaw@latest
```

Run (Windows PowerShell):
```powershell
npm install -g openclaw@latest
```

Post-install:
```bash
openclaw --version
mkdir -p "<target_dir>/conf"
```

### Phase 1.1: Cross-platform environment variable persistence (auto-detect, no hardcoded assumptions)
Do not hardcode one fixed rc file path. Detect platform + shell first, choose system-default persistence target, then write idempotently.

Required env values:
- `OPENCLAW_STATE_DIR` should point to `<target_dir>/conf`
- `PATH` should include npm global bin if `openclaw` is not already resolvable

#### Detection algorithm (all Unix-like: macOS/Linux)
1. Detect login shell in this order:
   - `basename "$SHELL"`
   - `ps -p $$ -o comm=`
   - fallback: `sh`
2. For detected shell, choose write targets by precedence:
   - `zsh`: prefer existing file from `~/.zshrc`, `~/.zprofile`, `~/.profile`; if none exist, create `~/.zshrc`.
   - `bash`: prefer existing file from `~/.bashrc`, `~/.bash_profile`, `~/.profile`; if none exist, create `~/.bashrc`.
   - `fish`: prefer `~/.config/fish/config.fish` (create dirs/file if missing) and use fish-native syntax.
   - other shells (`sh`, `dash`, `ksh`, unknown): prefer existing `~/.profile`; create it if missing.
3. Write to exactly one primary file unless shell semantics require extra linkage.
4. If login/non-login linkage is needed (example bash login shells), add a minimal source bridge only when absent.

#### macOS specifics
- Follow detection algorithm above; do not assume `.profile` or `.bash_profile` exists.
- For zsh, default primary target is `~/.zshrc`.
- For bash, default primary target is `~/.bashrc`; add `~/.bash_profile` bridge only when needed.

#### Linux specifics
- Follow detection algorithm above; distro differences are expected.
- For `fish`: write universal vars and fish config entry:
   - `set -Ux OPENCLAW_STATE_DIR "<target_dir>/conf"`
   - ensure npm global bin path is available in fish PATH when needed
- Unknown shell fallback remains `~/.profile`.

#### Windows (system-default approach)
Prefer registry-backed user environment variables via PowerShell APIs (default Windows behavior), not shell profile files as primary persistence.

1. Persist user env with PowerShell (idempotent):
```powershell
[Environment]::SetEnvironmentVariable("OPENCLAW_STATE_DIR", "<target_dir>\\conf", "User")
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($userPath -notlike "*npm*") {
  # ensure npm global bin path is present when openclaw command is not found
  [Environment]::SetEnvironmentVariable("Path", $userPath, "User")
}
```
2. Also update current session immediately:
```powershell
$env:OPENCLAW_STATE_DIR = "<target_dir>\\conf"
```
3. Optionally update PowerShell profile for convenience only; this is secondary to registry persistence.

#### Implementation guardrails
- Never fail setup just because `~/.profile` / `~/.bash_profile` is missing.
- Missing file is a normal case: create it.
- Avoid duplicate lines: append only when exact line is absent.
- Never overwrite unrelated user config; perform minimal additive edits only.
- Record which file/target was selected and print it in setup output for transparency.
- After writing env config, run a quick validation command in the same shell session.

### Phase 1.2: State dir single-source-of-truth guard (required)
Prevent split config (`~/.openclaw/openclaw.json` vs `~/.openclaw/conf/openclaw.json`) by enforcing one state dir for both CLI and service.

Canonical state dir:
- `<target_dir>/conf` (default install = `~/.openclaw/conf`)

Hard rules:
1. During this skill run, every `openclaw ...` command MUST execute with `OPENCLAW_STATE_DIR` set to canonical dir.
   - Unix/macOS:
   ```bash
   export OPENCLAW_STATE_DIR="<target_dir>/conf"
   ```
   - Windows PowerShell:
   ```powershell
   $env:OPENCLAW_STATE_DIR = "<target_dir>\\conf"
   ```
2. Never mix commands with and without `OPENCLAW_STATE_DIR` in the same session.
3. Before onboarding, print both resolved config paths and require equality after startup persistence checks.

Required consistency checks:
```bash
openclaw gateway status
```

Pass criteria:
- `Config (cli)` and `Config (service)` both point to `<target_dir>/conf/openclaw.json`.

Auto-remediation when mismatch detected:
1. Re-assert env in current session (`OPENCLAW_STATE_DIR=<target_dir>/conf`).
2. Re-run gateway service install/start (`openclaw gateway install && openclaw gateway start`).
3. Re-check `openclaw gateway status` until `Config (cli)` equals `Config (service)`.
4. If `~/.openclaw/openclaw.json` exists and differs from canonical config, report as legacy split-config artifact and keep canonical config authoritative.

### Phase 2: Run Onboard Immediately (same session)
Do not hand off to user. Execute onboard now.

### Phase 2.4: Provider plugin dependency bootstrap (auto, before auth)
Before running auth/login commands, auto-install/enable provider-plugin dependencies when required by selected auth mode.

Rules:
1. Always run `openclaw plugins list` first.
2. If selected auth mode requires a provider plugin and it is disabled, run `openclaw plugins enable <id>`.
3. If plugin id is missing entirely, install it, then enable it:
   - `openclaw plugins install <npm-spec>`
   - `openclaw plugins enable <id>`
4. Verify with `openclaw plugins list` and fail early with actionable output if still unavailable.

Provider/auth mapping:
- `openai-codex`: no provider plugin bootstrap required (built-in onboarding auth flow).
- `gemini-cli`: plugin id `google-gemini-cli-auth` (npm spec `@openclaw/google-gemini-cli-auth`).
- `minimax-portal`: plugin id `minimax-portal-auth` (npm spec `@openclaw/minimax-portal-auth`).
- `qwen-portal`: plugin id `qwen-portal-auth` (npm spec `@openclaw/qwen-portal-auth`).
- `copilot-proxy`: plugin id `copilot-proxy` (npm spec `@openclaw/copilot-proxy`).

#### 2.0 OpenAI Codex one-step OAuth fast path
Goal: skip intermediate onboarding prompts (`Onboarding mode`, `Existing config detected`, `Config handling`) and jump directly to OAuth/browser authorization.

Usage policy:
- Locked default workflow: always use this fast path.

Use this command when user selected `openai-codex`:

```bash
openclaw onboard \
  --accept-risk \
  --flow quickstart \
  --mode local \
  --auth-choice openai-codex \
  --skip-channels \
  --skip-skills \
  --skip-search \
  --skip-health \
  --skip-ui \
  --install-daemon \
  --node-manager npm
```

Behavior notes:
- `--accept-risk` skips the security confirmation prompt.
- `--flow quickstart` skips the onboarding mode selection prompt.
- `--skip-search` skips the Web search provider prompt.
- In quickstart flow, existing config is reused unless explicitly overridden by flags.
- OAuth remains interactive by design (user still completes browser authorization), but pre-auth wizard prompts are minimized.
- For `openai-codex`, keep auth in onboarding flow; do not switch to `models auth login --provider openai-codex` as a fallback.

Safety rule:
- If non-interactive fast path fails due environment-specific constraints, fallback to standard interactive `openclaw onboard --auth-choice openai-codex --install-daemon`.

OAuth certificate-error fallback:
- If `openai-codex` onboarding fails with certificate/TLS errors (for example `unknown certificate verification error`, `UNABLE_TO_GET_ISSUER_CERT`, `SELF_SIGNED_CERT_IN_CHAIN`), stop immediately and report the exact failing step.
- Do not auto-skip auth with `auth-choice skip`.
- Print exact resume command for the same step after user/environment fix:
```bash
openclaw onboard --auth-choice openai-codex --skip-search --install-daemon --node-manager npm
```

### Step-level execution logging (required)
For each major command, print a stable step id before execution and include it in error output.

Format:
- `STEP 1: node preflight`
- `STEP 2: install openclaw`
- `STEP 3: onboard openai-codex`
- `STEP 4: telegram setup`
- `STEP 5: china setup`
- `STEP 6: verification + dashboard + checklist`

On error, output must include:
1. `Failed step id` (example: `STEP 3`)
2. Exact command that failed
3. Raw stderr excerpt (at least first actionable lines)
4. One retry command for that same step

#### 2.1 Auth command templates

- OpenAI Codex OAuth:
```bash
openclaw onboard --auth-choice openai-codex --skip-search --install-daemon
```

- OpenAI API key (non-interactive):
```bash
openclaw onboard --non-interactive --accept-risk \
  --mode local \
  --auth-choice openai-api-key \
  --openai-api-key "<OPENAI_API_KEY>" \
  --gateway-bind loopback \
  --gateway-port 18789 \
  --skip-search \
  --install-daemon \
  --node-manager npm
```

- Claude setup-token (non-interactive):
```bash
openclaw onboard --non-interactive --accept-risk \
  --mode local \
  --auth-choice setup-token \
  --token-provider anthropic \
  --token "<CLAUDE_SETUP_TOKEN>" \
  --gateway-bind loopback \
  --gateway-port 18789 \
  --skip-search \
  --install-daemon \
  --node-manager npm
```

- Claude/Anthropic API key (auth-choice `apiKey`):
```bash
openclaw onboard --non-interactive --accept-risk \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "<ANTHROPIC_API_KEY>" \
  --gateway-bind loopback \
  --gateway-port 18789 \
  --skip-search \
  --install-daemon \
  --node-manager npm
```

- Gemini API key:
```bash
openclaw onboard --non-interactive --accept-risk \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "<GEMINI_API_KEY>" \
  --gateway-bind loopback \
  --gateway-port 18789 \
  --skip-search \
  --install-daemon \
  --node-manager npm
```

- Gemini OAuth (plugin-backed):
```bash
openclaw plugins enable google-gemini-cli-auth || openclaw plugins install @openclaw/google-gemini-cli-auth
openclaw onboard --accept-risk --flow quickstart --mode local --auth-choice google-gemini-cli --skip-search --install-daemon --node-manager npm
```

- Custom proxy / OneAPI / NewAPI (OpenAI or Anthropic compatible):
```bash
openclaw onboard --non-interactive --accept-risk \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "<BASE_URL>" \
  --custom-api-key "<API_KEY>" \
  --custom-model-id "<MODEL_ID>" \
  --custom-compatibility "<openai_or_anthropic>" \
  --gateway-bind loopback \
  --gateway-port 18789 \
  --skip-search \
  --install-daemon \
  --node-manager npm
```

#### 2.2 Subscription auth special rule
If subscription auth opens browser/device flow (for example `openai-codex`, `claude-code`, `google-gemini-cli`, `kimi`, `glm`), keep user in current guided flow and ask only for callback value if needed. Never tell user to open a new terminal and run commands themselves.

#### 2.3 Subscription auth completion wait gate (required)
When subscription auth is required (OAuth/browser login/device-code style flows), the installer must wait for auth completion before declaring setup done.

Anti-stall rule:
- A long-running subscription-auth step must never appear frozen.
- If the shell/tool running `openclaw onboard` does not stream child-process output reliably, the agent MUST add its own visible progress updates while the command is still running.
- It is forbidden to launch a blocking auth command and then remain silent until it exits.

Required behavior:
1. Start subscription auth flow in the same guided process (no external Terminal app).
2. Before starting the auth command, print a clear waiting message that tells the user exactly what is happening, for example:
   - `STEP 3: starting subscription authorization (<provider>); your browser/device-code page may stay open for a while while authorization completes.`
3. Poll auth readiness every 3-5 seconds for up to 10 minutes.
4. Never stay silent while waiting: emit a progress heartbeat every 10-15 seconds (for example `STEP 3 waiting subscription auth... elapsed 45s; checking again in 5s`).
5. If no new signal for 60 seconds, print an actionable status update that includes all of the following:
   - current waiting step id,
   - elapsed time,
   - what the user should verify in the browser,
   - whether the auth tab can be left open,
   - next automatic check time.
6. Verification command (preferred):
```bash
openclaw models status --check
```
7. Success condition:
   - `openclaw models status --check` exits 0, and
   - default model provider matches configured auth target family.
   - Provider-family examples: `openai-codex/*`, `claude*`, `gemini*`, `kimi*`, `glm*`.
8. On timeout:
   - keep installation artifacts intact,
   - print exact resume command,
   - mark setup as "partially complete: awaiting subscription auth" (not fully complete),
   - include last heartbeat timestamp + last known OAuth state so user can see it was not hung.

Implementation rule for assistants:
- The waiting UX must be agent-driven, not delegated to chance shell buffering.
- Preferred pattern: run the OAuth command in a way the agent can keep reporting status concurrently, then poll readiness in a separate loop.
- If interactive subprocess support exists, keep the auth command attached there and send heartbeat messages from the main session.
- If only blocking shell execution is available, use a backgrounded command or equivalent mechanism so the agent can continue printing heartbeats and polling status until completion.

Reference wait-loop shape (adapt to the host assistant/tooling):
```bash
# start subscription-auth-capable onboarding without losing control of the session
openclaw onboard --auth-choice "<selected-auth-choice>" --skip-search --install-daemon --node-manager npm &
auth_pid=$!
start_ts=$(date +%s)
last_notice_ts=$start_ts

while kill -0 "$auth_pid" 2>/dev/null; do
  now_ts=$(date +%s)
  elapsed=$((now_ts - start_ts))

  if openclaw models status --check >/dev/null 2>&1; then
    wait "$auth_pid"
    break
  fi

  if [ $((now_ts - last_notice_ts)) -ge 15 ]; then
    printf 'STEP 3 waiting subscription auth... elapsed %ss; checking again in 5s\n' "$elapsed"
    last_notice_ts=$now_ts
  fi

  if [ "$elapsed" -ge 60 ] && [ $((elapsed % 60)) -lt 5 ]; then
    printf '%s\n' 'STEP 3 still waiting: please finish the browser/device authorization if it is still open; I will check again automatically in 5s.'
  fi

  if [ "$elapsed" -ge 600 ]; then
    printf '%s\n' 'STEP 3 timed out waiting for subscription authorization; installation state is preserved and can be resumed.'
    break
  fi

  sleep 5
done
```

Hard requirement:
- During any subscription authorization flow (`openai-codex`, `claude-code`, `google-gemini-cli`, `kimi`, `glm`, and equivalent providers), the transcript must show visible progress at least once every 15 seconds until success or timeout.
- If the assistant/tool cannot satisfy that requirement with one execution method, it must switch to another execution method that can (for example interactive session, background process, or explicit polling loop).

Immediately after subscription auth success:
- If `auth-choice` is `openai-codex`, run the codex default-model pin below.
- If `auth-choice` is not `openai-codex`, do not force-set `gpt-codex-5.3`; keep provider-consistent defaults.

Codex-only post-auth pin (required when `auth-choice=openai-codex`):
```bash
openclaw config set agents.defaults.model.primary "gpt-codex-5.3"
openclaw config get agents.defaults.model.primary
```

Model gate:
- Do not proceed to channel/plugin/final steps unless `agents.defaults.model.primary` equals `gpt-codex-5.3` when auth choice is `openai-codex`.

Resume command (openai-codex):
```bash
openclaw onboard --auth-choice openai-codex --skip-search --install-daemon --node-manager npm
```

### Phase 2.5: Agent count provisioning (same session, automatic)
After onboarding succeeds, provision requested agents immediately.

Rules:
1. `main` is the first/default agent.
2. Requested count `N` means total agents after setup should be `N`.
3. If `N=1`, do nothing extra.
4. For `N>1`, create `N-1` additional agents with `openclaw agents add`.

Non-interactive creation pattern (recommended):
```bash
openclaw agents add <agent_id> --workspace "<workspace_root>-<agent_id>" --non-interactive
```

Example (user requested total `3`, default names):
```bash
openclaw agents add agent-01 --workspace "~/.openclaw/workspace-agent-01" --non-interactive
openclaw agents add agent-02 --workspace "~/.openclaw/workspace-agent-02" --non-interactive
```

If user provided custom names, create each one with the same pattern and matching workspace suffix.

Post-create verification:
```bash
openclaw agents list --bindings
```

Optional (when user provided per-agent bindings):
```bash
openclaw agents bind --agent <agent_id> --bind <channel[:accountId]>
```

### Phase 2.6: Install-time per-agent routing prompt (recommended)
Always ask this after agent creation:
1. `Do you want to configure routing now?` (yes/no, default yes)
2. If yes, for each agent ask:
   - `Bindings for <agent_id> (comma-separated channel[:accountId], or skip)`
   - Example: `telegram:dev,discord:work`
3. Apply each binding immediately with `openclaw agents bind`.
4. Show final routing table with `openclaw agents list --bindings`.

If user chooses no, continue installation and print a post-install tip:
`openclaw agents bind --agent <agent_id> --bind <channel[:accountId]>`

### Phase 3: Channels / Skills / Hooks in same pass (hard-locked, no deferred china config)

1. Channels:
   - Always run Telegram setup in this phase with exactly one prompt item: `输入你的机器人Token`.
   - Apply with:
     - `openclaw channels add --channel telegram --token "<TELEGRAM_BOT_TOKEN>"`
   - `openclaw config set channels.telegram.groupPolicy open`
   - Verify with:
     - `openclaw config get channels.telegram.groupPolicy`
     - `openclaw channels status --probe`
   - Completion gate: Telegram must be configured/running before moving on.

2. Skills dependencies:
   - Always install clawhub via npm without asking user:
     - `npm install -g clawhub`
    - Install OpenClaw China plugin (unified package), then run setup wizard in the same assistant session:
       - `openclaw plugins install @openclaw-china/channels`
       - `openclaw china setup`
   - Never open a separate Terminal app to run `openclaw china setup`.
   - If direct interactive wizard IO is unavailable, collect required fields via `question` and apply equivalent `openclaw config set channels.<china-channel>...` commands in-session.
   - After plugin install, immediately run the strict two-question protocol in this same phase.
   - It is forbidden to postpone this picker to the end-of-install summary.
   - Question 1: multi-select channel picker with only `dingtalk`, `qqbot`, `wecom`, `wecom-app`, `feishu-china`.
   - Question 2: for all selected channels, collect `输入配置信息` (one entry per selected channel) and show the channel-specific example for each.
   - Parse and apply channel config in-session; do not defer.
   - Completion gate: OpenClaw China selected channels must be fully configured in-session before verification.
    - Do not clone openclaw-china source in this skill. Use plugin install command path only.
   - Run `openclaw skills check`.
   - If missing dependencies are reported, install them using npm/system package manager.

3. Hooks:
   - No hooks prompt. Always enable this fixed set:
      - `openclaw hooks enable boot-md`
      - `openclaw hooks enable bootstrap-extra-files`
      - `openclaw hooks enable command-logger`
      - `openclaw hooks enable session-memory`
   - Verify with `openclaw hooks check`.

### Phase 4: External volume launchd fix (macOS only, conditional)
Apply only when install path starts with `/Volumes/`:

```bash
sed -i '' 's|/Volumes/.*/logs/gateway.log|/tmp/openclaw-gateway.log|g' ~/Library/LaunchAgents/ai.openclaw.gateway.plist
sed -i '' 's|/Volumes/.*/logs/gateway.err.log|/tmp/openclaw-gateway.err.log|g' ~/Library/LaunchAgents/ai.openclaw.gateway.plist
launchctl bootout gui/$UID/ai.openclaw.gateway || true
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

### Phase 5: Verification and handoff
Run and report:
1. `openclaw status`
2. `openclaw channels status --probe`
3. `openclaw hooks check`
4. `openclaw models status --check`
5. `openclaw config get channels.telegram.groupPolicy` (must be `open` in default mode)
6. `openclaw config get agents.defaults.model.primary` (must be `gpt-codex-5.3` after codex auth)

### Phase 5.2: Startup persistence (required)
Before final completion, ensure startup-on-boot is configured.

Required actions:
1. Install/start gateway service (idempotent):
```bash
openclaw gateway install
openclaw gateway start
```
2. Verify autostart status:
```bash
openclaw gateway status
```
2.1 Verify state-dir consistency (required gate):
- Parse `openclaw gateway status` output and confirm:
  - `Config (cli): ~/.openclaw/conf/openclaw.json` (or `<target_dir>/conf/openclaw.json`)
  - `Config (service): ~/.openclaw/conf/openclaw.json` (or `<target_dir>/conf/openclaw.json`)
- If either side points to a different path (for example `~/.openclaw/openclaw.json`), DO NOT mark install complete.
- Apply Phase 1.2 auto-remediation and re-run status checks.
3. Platform checks:
   - macOS: verify LaunchAgent exists/loaded (`ai.openclaw.gateway`)
   - Linux (systemd): verify service enabled (`enabled`)
   - Windows: verify startup task/service installed and active
4. If verification fails, do not mark install complete; print exact fix command and rerun status check.

Output must include:
- Current status (healthy or actionable failures)
- Config path parity result (`Config (cli)` == `Config (service)`)
- Canonical `OPENCLAW_STATE_DIR` value used during setup
- Dashboard URL (`http://127.0.0.1:18789/#token=...`)
- What was auto-configured (model/auth mode, channels, skills deps, hooks)
- OAuth verification result (complete / pending with resume command)
- WebUI open result (opened automatically / fallback required)
- Telegram setup result (configured + groupPolicy=open)
- openclaw-china setup result (wizard completed / equivalent config completed)
- china picker choice and configured channel list (or explicit skip status)
- Install warning: Telegram still has a final manual authorization step:
  - User sends any message to the Telegram bot first to receive an authorization code.
  - User then enters that authorization code in WebUI and asks OpenClaw to complete Telegram authorization.
- If any step failed: include exact failed step id + command + stderr snippet + same-step retry command.

### Phase 5.1: WebUI open + smoke test guidance (always)
After installation completes, open WebUI only at the very end:

Ordering rule:
- Do not run `openclaw dashboard` before all install/config/setup/verification steps and final checklist output are complete.

1. If headless/no-GUI prevents open, fallback and print URL:
```bash
openclaw dashboard --no-open
```
2. Provide a minimal smoke test checklist:
   - Page loads without 4xx/5xx
   - Agent list is visible (at least `main`)
   - Start one test prompt in chat and confirm response
   - If model auth is pending (OAuth/API key), explicitly indicate auth is required before chat reply works

### Phase 6: Print post-install core command checklist (always, both default and custom modes)
At the end of setup, always print a complete command checklist for daily use.
Do not omit blocks.
Do not skip this step even if any earlier phase required retries.

Required checklist blocks (fixed canonical output):

```bash
# China - dingtalk（交互式填写 AppKey/AppSecret）
openclaw china setup
# 重启并复查
openclaw gateway restart
openclaw channels status --probe

# Health and status
openclaw status
openclaw status --deep
openclaw gateway status
openclaw channels status --probe
openclaw models status --check
openclaw dashboard
openclaw dashboard --no-open

# Gateway service lifecycle
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway status
openclaw gateway uninstall

# Onboarding and auth refresh
openclaw onboard
openclaw onboard --auth-choice openai-codex
openclaw onboard --auth-choice openai-api-key
openclaw onboard --skip-search
openclaw plugins list

# Agents and routing
openclaw agents list --bindings
openclaw agents add <id>
openclaw agents bind --agent <id> --bind <channel[:accountId]>
openclaw agents unbind --agent <id> --bind <channel[:accountId]>

# Channels
openclaw channels list
openclaw channels add --channel <name>
openclaw channels remove --channel <name> --delete
openclaw config set channels.telegram.groupPolicy open
openclaw config get channels.telegram.groupPolicy

# Skills and hooks
npm install -g clawhub
clawhub login
clawhub search "<query>"
clawhub install <slug>
clawhub list
clawhub update --all
openclaw skills list --eligible
openclaw skills check
openclaw hooks list
openclaw hooks enable boot-md
openclaw hooks enable bootstrap-extra-files
openclaw hooks enable command-logger
openclaw hooks enable session-memory
openclaw hooks check

# Plugins (provider dependencies)
openclaw plugins list
openclaw plugins doctor
openclaw plugins enable <plugin-id>
openclaw plugins install <npm-spec>
openclaw plugins install @openclaw-china/channels
openclaw china setup
openclaw channels status --probe

# Config and logs
export OPENCLAW_STATE_DIR="$HOME/.openclaw/conf"
openclaw config get gateway.port
openclaw config set gateway.port 18789
openclaw config get agents.defaults.model.primary
openclaw config set agents.defaults.model.primary "gpt-codex-5.3"
openclaw logs --follow
```

After printing the full checklist above, run final WebUI open step:
```bash
openclaw dashboard
```

If auto-open fails, run fallback and print URL:
```bash
openclaw dashboard --no-open
```

Formatting requirements:
- Use the fixed canonical checklist above exactly.
- Use copy-paste-ready commands only.
- Include only commands valid for the detected platform/context.

## Success Criteria
- User does not need to separately run `openclaw onboard`.
- Install + onboarding + key configuration are completed in one continuous guided session.
- Required dependencies are installed or explicit failures are reported with exact fix commands.
- If OAuth is selected, setup is only "complete" after auth verification passes.
- Telegram must be configured and `channels.telegram.groupPolicy` must be `open`.
- `openclaw china setup` (or in-session equivalent config application) must be completed before declaring done.
- OpenClaw China configuration must happen during main install flow (no deferred post-install "continue configure" prompt).
- Final summary must include Telegram manual final-step warning (authorization code from Telegram message -> input in WebUI).
- Final response must include the full post-install command checklist.
- Gateway autostart must be enabled and verified so OpenClaw starts automatically after reboot.
