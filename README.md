# Agent Zero: Beginner's Guide to Common Errors

**Last Updated:** February 11, 2026  
**Agent Zero Version:** v0.9.8 (February 10, 2025)  
**Last Verified Against GitHub Issues:** February 11, 2026  
**Target Audience:** Non-expert beginners using self-hosted Agent Zero on Windows/Linux/macOS

---

## 🚨 CRITICAL: v0.9.8 Breaking Changes

**If upgrading from v0.9.7 or earlier, READ THIS FIRST:**

### What Changed
1. **Skills replaced Instruments** - Old "Instruments" no longer exist
2. **Volume mapping MUST use `/a0/usr`** - Mapping `/a0` will cause issues
3. **WebSocket-based UI** - Real-time updates, not polling
4. **User data moved to `/usr` structure** - Cleaner separation
5. **Resource requirements increased** - 8GB RAM minimum (was 6GB)

### Migration Required
```bash
# 1. Backup FIRST via Settings → Backup & Restore
# 2. Stop old container
docker stop agent-zero && docker rm agent-zero

# 3. Start v0.9.8 with NEW volume mapping
docker run -d -p 50080:80 \
  -v ~/agent-zero-data:/a0/usr \
  --ulimit nofile=65536:65536 \
  --name agent-zero agent0ai/agent-zero:latest

# 4. On first start, automatic migration runs
# 5. Verify your data in Settings
```

### If Migration Fails
- Restore from backup (Settings → Backup & Restore)
- Check logs: `docker logs agent-zero | tail -100`
- Report with your OS and error message

---

## 🚀 Fix 90% of Issues in 5 Minutes

**Try these before reading the full guide:**

### 1. Correct Docker Setup (v0.9.8)
```bash
# Stop and recreate with proper configuration
docker stop agent-zero && docker rm agent-zero

# Linux/macOS - NOTE: /a0/usr (not /a0)
docker run -d -p 50080:80 \
  -v ~/agent-zero-data:/a0/usr \
  --add-host=host.docker.internal:host-gateway \
  --ulimit nofile=65536:65536 \
  --name agent-zero agent0ai/agent-zero

# Windows (PowerShell) - NOTE: /a0/usr
docker run -d -p 50080:80 -v C:\agent-zero-data:/a0/usr --ulimit nofile=65536:65536 --name agent-zero agent0ai/agent-zero
```

### 2. Fix Connection Issues
- **API Base URL:** `http://host.docker.internal:11434` (not localhost)
- **LM Studio:** Load a model first, then enable local server
- **Test:** `curl http://localhost:11434/api/tags` (from host)

### 3. Fix Memory Errors
```bash
docker exec agent-zero rm -rf /a0/memory/*
docker restart agent-zero
```

### 4. Fix Most Other Issues
```bash
docker restart agent-zero
```

### 5. Use Built-in Backup (v0.9.8)
- Settings → Backup & Restore → Create Backup
- Do this BEFORE troubleshooting
- Store backup file outside Docker

---

## 📦 Known Safe Defaults (Updated for v0.9.8)

| Component | Recommended Value | Why |
|-----------|------------------|-----|
| Agent Zero version | v0.9.8 | Latest stable |
| Docker RAM | **8GB minimum** | WebSocket + Skills overhead |
| Ollama version | Latest (0.5+) | Works with v0.9.8 |
| Chat model | `llama3.2:3b` or `qwen2.5:3b` | Best balance |
| Embedding | `mxbai-embed-large` | Standard, reliable |
| File descriptors | `--ulimit nofile=65536:65536` | Prevents crashes |
| Volume mapping | `-v ~/data:/a0/usr` | **MUST be /usr** |
| Backup method | Settings → Backup & Restore | Built-in feature (v0.9+) |

---

## 🔍 Quick Symptom Finder

| What You See | Jump To Section |
|-------------|-----------------|
| `Cannot connect to host` / Connection timeout | [#1 Local LLM Connection](#1-local-llm-connection-failures) |
| `OSError: Socket is closed` / WebSocket disconnected | [#2 Socket & WebSocket Errors](#2-socket--websocket-errors) |
| Chat history missing / Corrupted backup | [#3 Data Loss](#3-data-loss-on-crashrestart) |
| `KeyError` / `AssertionError: d == self.d` | [#4 Memory FAISS Errors](#4-memory-system-faiss-errors) |
| Skills won't import / `SKILL.md` errors | [#5 Skills System Errors](#5-skills-system-errors-new-in-v098) |
| Git clone failed / Authentication error | [#6 Git Project Errors](#6-git-project-errors-new-in-v098) |
| Settings not saving / Config conflicts | [#7 Configuration Conflicts](#7-configuration-conflicts-new) |
| `OSError: [Errno 24] Too many open files` | [#8 File Descriptor Exhaustion](#8-file-descriptor-exhaustion) |
| Container won't start / Initialization loop | [#9 Startup Failures](#9-startup-failures--initialization-loops) |
| Commands run but no output | [#10 Terminal Output Empty](#10-terminal-output-empty) |
| Can't access remotely / Tunnel errors | [#11 Dev Tunnels Issues](#11-dev-tunnels--remote-access-new) |

---

## 📊 Error Priority Matrix (v0.9.8)

| Error | Frequency | Severity | Status | v0.9.8 Notes |
|-------|-----------|----------|--------|--------------|
| Configuration Conflicts | Very High | High | New Issue | `A0_SET_*` vs UI settings |
| Skills Import Failures | High | Medium | New in v0.9.8 | Legacy Instrument incompatibility |
| Socket/WebSocket Drops | High | Critical | Improved | New WebSocket infrastructure |
| Memory FAISS Errors | Medium | Critical | Improved | Deferred tasks help |
| Git Project Auth | Medium | High | New in v0.9.8 | SSH key setup |
| Data Loss | Low | Critical | Preventable | Built-in backup since v0.9 |
| Connection Issues | Low | High | Mostly Solved | Better docs |
| Startup Loops | Low | Critical | Ongoing | Attribute errors |

---

## 🔴 Critical Errors

### 1. Local LLM Connection Failures

**What you'll see:**
```
Connection timeout
Cannot connect to host
litellm.APIConnectionError: Cannot connect to host 127.0.0.1:11434
```

**Why it happens:**
- Docker network isolation (`localhost` inside ≠ `localhost` on host)
- Missing `/v1` in API path (LM Studio only)
- LM Studio server enabled but **no model loaded** (very common)
- Ollama not running
- Firewall blocking connections

**Status (v0.9.8):** LiteLLM integration stable. 4 new providers added (CometAPI, Z.AI, Moonshot AI, AWS Bedrock).

**Fix step-by-step:**

#### For Ollama:
1. **Verify Ollama running on HOST:**
   ```bash
   ollama list
   # Should show downloaded models
   ```

2. **Pull models if needed:**
   ```bash
   ollama pull llama3.2:3b
   ollama pull mxbai-embed-large
   ```

3. **Test API from host:**
   ```bash
   curl http://localhost:11434/api/tags
   # Should return JSON with model list
   ```

4. **Configure in Agent Zero UI:**
   - Settings → Agent Settings → Chat Model
   - Provider: `Ollama`
   - Model: `llama3.2:3b` (exact name from `ollama list`)
   - API Base URL: `http://host.docker.internal:11434`
   
   - Utility Model: Same settings
   - Embedding Model:
     - Provider: `Ollama`
     - Model: `mxbai-embed-large`
     - API Base URL: `http://host.docker.internal:11434`

#### For LM Studio:
1. **Load a model in LM Studio first** (critical step)
2. **Enable Local Server** (Developer → Local Server)
3. **Configure in Agent Zero:**
   - API Base URL: `http://host.docker.internal:1234/v1`
   - Note the `/v1` at the end!
   - Model: Copy exact name from LM Studio

#### For New Providers (v0.9.8):
- **CometAPI:** Set `COMETML_API_KEY` in Settings → API Keys
- **Z.AI:** Configure via custom provider
- **Moonshot AI:** Chinese model provider
- **AWS Bedrock:** Requires AWS credentials

**What NOT to do:**
- ❌ Using `localhost` or `127.0.0.1` from Docker
- ❌ Enabling LM Studio server without loading a model
- ❌ Forgetting `/v1` for LM Studio
- ❌ Typos in model names

**Prevention:**
- Always test with `curl` from host first
- Copy exact model names, don't type from memory
- Keep notes of working configurations

**Sources:** [GitHub Issues](https://github.com/agent0ai/agent-zero/issues?q=connection), Official docs

**Status:** ✅ Mostly Solved | **Priority:** Medium

---

### 2. Socket & WebSocket Errors

**What you'll see:**
```
OSError: Socket is closed
WebSocket connection dropped
UI not updating in real-time
Agent unresponsive after long task
```

**Why it happens:**
- **SSH socket issues:** Terminal connection to container times out
- **WebSocket drops (NEW in v0.9.8):** Real-time UI sync fails
- Long-running operations (>5 minutes)
- Docker networking drops idle connections

**Status (v0.9.8):** Improved with WebSocket infrastructure, but SSH socket issues persist. Now two separate error types.

**Fix step-by-step:**

#### For SSH Socket Errors:
1. **Immediate recovery:**
   - In Agent Zero UI, type: `reset terminal`
   - This restarts the terminal session

2. **For long operations:**
   - Break tasks into <4 minute chunks
   - Ask agent to save progress frequently

3. **Container restart (if unresponsive):**
   ```bash
   docker restart agent-zero
   ```

#### For WebSocket Connection Drops (NEW):
1. **Check browser console:**
   - Press F12 → Console tab
   - Look for WebSocket errors

2. **Refresh page:**
   - Hard refresh: Ctrl+F5 (Windows) / Cmd+Shift+R (Mac)

3. **Check firewall:**
   - Ensure port 50080 allows WebSocket upgrades
   - Some antivirus blocks WebSocket connections

4. **Verify container logs:**
   ```bash
   docker logs agent-zero | grep -i websocket
   ```

**What NOT to do:**
- ❌ Ignoring "socket closed" warnings
- ❌ Running operations >5 minutes without checkpoints
- ❌ Using VPN that blocks WebSockets
- ❌ Continuing to send commands after error appears

**Prevention:**
- Split large tasks into smaller subtasks
- Use process groups to track progress (v0.9.8 UI feature)
- Restart container periodically during heavy use
- Monitor logs: `docker logs -f agent-zero`

**Sources:** [GitHub #648](https://github.com/agent0ai/agent-zero/issues/648), [#680](https://github.com/agent0ai/agent-zero/issues/680)

**Status:** ⚠️ SSH: Unresolved | WebSocket: New system | **Priority:** Critical

---

### 3. Data Loss on Crash/Restart

**What you'll see:**
- Chat history disappeared
- Projects missing after restart
- Files you created are gone
- Corrupted backup files

**Why it happens:**
- No persistent volume mapped (data only in ephemeral container)
- Wrong volume mapping (using old `/a0` instead of `/a0/usr`)
- Crashes during file writes

**Status (v0.9.8):** **Fully preventable** with built-in Backup & Restore + correct volume mapping.

**Fix step-by-step:**

1. **Use built-in backup (PRIMARY METHOD):**
   - Settings → Backup & Restore
   - Click "Create Backup" before major work
   - Download backup file (stores outside container)
   - Test restore occasionally

2. **Setup persistent storage (REQUIRED):**
   ```bash
   # Stop container
   docker stop agent-zero && docker rm agent-zero
   
   # Create directory on host
   mkdir -p ~/agent-zero-data
   
   # Start with CORRECT volume mapping (v0.9.8)
   docker run -d -p 50080:80 \
     -v ~/agent-zero-data:/a0/usr \
     --ulimit nofile=65536:65536 \
     --name agent-zero agent0ai/agent-zero
   ```

3. **Manual backup (secondary):**
   ```bash
   # Backup entire user directory
   docker cp agent-zero:/a0/usr ~/backup-$(date +%Y%m%d)
   ```

4. **Restore from backup:**
   - Settings → Backup & Restore → Upload backup file
   - Or manual: `docker cp ~/backup agent-zero:/a0/usr`

**What NOT to do:**
- ❌ Running without any volume mapping
- ❌ Mapping old `/a0` path (use `/a0/usr` in v0.9.8)
- ❌ Trusting auto-save alone
- ❌ Deleting container before verifying backup
- ❌ Storing backups only inside Docker

**Prevention:**
- **ALWAYS** map `/a0/usr` volume
- Use Settings → Backup before updates
- Follow 3-2-1 rule: 3 copies, 2 media types, 1 offsite
- Test restore process monthly

**Sources:** [GitHub #923](https://github.com/agent0ai/agent-zero/issues/923), Official migration docs

**Status:** ✅ Fully Preventable (was Critical) | **Priority:** High

---

### 4. Memory System FAISS Errors

**What you'll see:**
```
KeyError: 3687
AssertionError: d == self.d (1024 != 768)
InvalidKeyException: Invalid characters in key
Memory search fails
```

**Why it happens:**
- Changed embedding model without clearing old index
- Different embedding dimensions (old: 768, new: 1024)
- FAISS index and document store out of sync
- Special characters in model names (rare)

**Status (v0.9.8):** Improved - memory operations now use deferred tasks, reducing errors under load.

**Fix step-by-step:**

1. **Use Memory Management Dashboard (RECOMMENDED):**
   - Settings → Memory Management
   - Review stored memories
   - Clear specific problematic entries
   - More surgical than wiping everything

2. **If changed embedding model:**
   ```bash
   # Clear all memories (deletes learned data)
   docker exec agent-zero rm -rf /a0/memory/*
   docker restart agent-zero
   ```

3. **For dimension mismatch:**
   - Check current model dimensions:
     - `mxbai-embed-large`: 1024
     - `nomic-embed-text`: 768
     - `text-embedding-3-small`: 1536
   - If switched models, must clear memory

4. **Complete reset (last resort):**
   ```bash
   docker exec agent-zero rm -rf /a0/memory/*
   docker restart agent-zero
   ```

**What NOT to do:**
- ❌ Changing embedding model without clearing memory
- ❌ Using model names with unusual characters
- ❌ Mixing different embedding dimensions
- ❌ Ignoring dimension mismatch errors

**Prevention:**
- Stick with one embedding model long-term
- Before changing: Backup → Clear → Switch → Rebuild
- Use Memory Management Dashboard weekly
- Document your model configuration

**Note:** Many experienced users periodically clear `/a0/memory/` (weekly or monthly) to prevent index corruption. This is a known trade-off between persistence and stability.

**Sources:** [GitHub #615](https://github.com/agent0ai/agent-zero/issues/615), [#759](https://github.com/agent0ai/agent-zero/issues/759)

**Status:** ⚠️ Improved but unresolved | **Priority:** Critical

---

### 5. Skills System Errors (NEW in v0.9.8)

**What you'll see:**
```
Skill import failed
Invalid SKILL.md format
Legacy Instrument not compatible
Skill not found in registry
```

**Why it happens:**
- **Skills replaced Instruments** in v0.9.8 (breaking change)
- Trying to import old Instrument files
- Invalid `SKILL.md` format
- Missing required skill fields
- Skill dependencies not met

**Status (v0.9.8):** New system, teething issues expected.

**Fix step-by-step:**

1. **Import skills via UI (NEW):**
   - Settings → Skills → Import Skill
   - Upload `.skill` file or `SKILL.md`
   - Or use built-in skills library

2. **Convert legacy Instruments:**
   - Old Instruments won't work directly
   - Must be rewritten as Skills with `SKILL.md` format
   - See official Skills documentation

3. **Validate SKILL.md format:**
   ```markdown
   # Required fields:
   ---
   name: skill_name
   description: Brief description
   version: 1.0.0
   author: your_name
   ---
   
   # Skill content follows
   ```

4. **Check skill dependencies:**
   - Some skills require specific tools
   - Verify all dependencies installed
   - Check logs: `docker logs agent-zero | grep -i skill`

5. **List active skills:**
   - Settings → Skills → Installed Skills
   - Verify skill loaded successfully

**What NOT to do:**
- ❌ Trying to use old Instrument files
- ❌ Manually editing system skill files
- ❌ Installing untrusted skills (security risk)
- ❌ Skipping skill validation

**Prevention:**
- Use official skills from library
- Test new skills in isolated projects
- Back up before installing custom skills
- Keep skills updated

**Sources:** [v0.9.8 Release Notes](https://github.com/agent0ai/agent-zero/releases/tag/v0.9.8), Skills documentation

**Status:** 🆕 New System | **Priority:** Medium

---

### 6. Git Project Errors (NEW in v0.9.8)

**What you'll see:**
```
Git clone failed
Authentication required
Permission denied (publickey)
Failed to initialize submodules
Repository not found
```

**Why it happens:**
- Missing SSH keys for private repos
- GitHub token not configured
- Wrong repository URL
- Insufficient permissions
- Submodule authentication failures

**Status (v0.9.8):** New Git-based projects feature.

**Fix step-by-step:**

#### For Public Repositories:
1. **Use HTTPS URL:**
   ```
   https://github.com/username/repo.git
   ```

2. **Clone via UI:**
   - Projects → New Project → Git Repository
   - Paste URL and clone

#### For Private Repositories:
1. **Setup SSH key (recommended):**
   ```bash
   # Generate SSH key on host
   ssh-keygen -t ed25519 -C "your_email@example.com"
   
   # Add to GitHub: Settings → SSH Keys
   cat ~/.ssh/id_ed25519.pub
   ```

2. **Or use Personal Access Token:**
   - GitHub → Settings → Developer Settings → Personal Access Tokens
   - Generate token with `repo` scope
   - Use URL: `https://TOKEN@github.com/username/repo.git`

3. **Configure in Agent Zero:**
   - Settings → Git Configuration
   - Add SSH key or token
   - Test connection

#### For Submodule Errors:
```bash
# Inside container
docker exec -it agent-zero bash
cd /a0/projects/your_project
git submodule update --init --recursive
```

**What NOT to do:**
- ❌ Using SSH URL without SSH key setup
- ❌ Hardcoding tokens in project files (security risk)
- ❌ Cloning to wrong directory
- ❌ Ignoring submodule errors

**Prevention:**
- Set up SSH keys once, use everywhere
- Use Deploy Keys for specific repos
- Test git access from host first
- Document authentication setup

**Sources:** [v0.9.8 Release Notes](https://github.com/agent0ai/agent-zero/releases/tag/v0.9.8)

**Status:** 🆕 New Feature | **Priority:** Medium

---

### 7. Configuration Conflicts (NEW)

**What you'll see:**
```
Settings not saving
Configuration keeps reverting
API keys disappearing
Different settings in UI vs actual behavior
```

**Why it happens:**
- **NEW in v0.9.8:** `A0_SET_*` environment variables override UI settings
- Conflicts between `.env` file and UI configuration
- Multiple configuration sources fighting each other
- Migration from older versions

**Status (v0.9.8):** New configuration system causes confusion.

**Fix step-by-step:**

1. **Understand configuration hierarchy:**
   ```
   Priority (highest to lowest):
   1. A0_SET_* environment variables (.env or docker -e)
   2. Settings UI
   3. Default values
   ```

2. **Check for A0_SET_* variables:**
   ```bash
   # View all environment variables
   docker exec agent-zero env | grep A0_SET
   ```

3. **Remove conflicting env vars:**
   - Edit `.env` file or docker run command
   - Remove any `A0_SET_*` variables you don't need
   - Restart container

4. **Use UI for most settings:**
   - Settings → Agent Settings
   - Settings → API Keys
   - Changes persist in `/a0/usr/config.json`

5. **Use A0_SET_* only for:**
   - Docker deployment automation
   - Secrets management
   - Read-only production configs

**Example conflict:**
```bash
# .env file has:
A0_SET_CHAT_MODEL=gpt-4

# UI shows llama3.2, but agent uses gpt-4
# Fix: Remove A0_SET_CHAT_MODEL from .env
```

**What NOT to do:**
- ❌ Setting same config in multiple places
- ❌ Using A0_SET_* for everything
- ❌ Editing config files while agent running
- ❌ Ignoring "settings not saving" warnings

**Prevention:**
- Choose ONE configuration method
- Document which settings use env vars
- Test settings changes immediately
- Check logs: `docker logs agent-zero | grep -i config`

**Sources:** [v0.9.8 Release Notes](https://github.com/agent0ai/agent-zero/releases/tag/v0.9.8), Configuration docs

**Status:** 🆕 New System | **Priority:** High

---

### 8. File Descriptor Exhaustion

**What you'll see:**
```
OSError: [Errno 24] Too many open files
Browser tool fails to launch
System becomes unstable
```

**Why it happens:**
- Browser tool leaks file handles (confirmed bug)
- Default limit (1024) too low for prolonged use
- Long-running sessions
- SQLite connections not released

**Status (v0.9.8):** Still present, same fix as before.

**Fix step-by-step:**

1. **Immediate relief:**
   ```bash
   docker exec agent-zero pkill -9 chrome
   docker exec agent-zero pkill -9 chromium
   ```

2. **Permanent fix (restart with ulimit):**
   ```bash
   docker stop agent-zero && docker rm agent-zero
   
   docker run -d -p 50080:80 \
     --ulimit nofile=65536:65536 \
     --ulimit nproc=8192:8192 \
     -v ~/agent-zero-data:/a0/usr \
     --name agent-zero agent0ai/agent-zero
   ```

3. **Monitor usage:**
   ```bash
   docker exec agent-zero sh -c "ls /proc/1/fd | wc -l"
   # If >800: restart soon
   ```

**What NOT to do:**
- ❌ Running without `--ulimit` settings
- ❌ Sessions >24 hours without restart
- ❌ Running multiple browser instances
- ❌ Ignoring warnings

**Prevention:**
- **Always** set ulimit when creating container
- Schedule daily restarts for heavy browser use
- Clean temp files: `docker exec agent-zero find /tmp -type f -mtime +1 -delete`
- Monitor: `docker stats agent-zero`

**Sources:** [GitHub #906](https://github.com/agent0ai/agent-zero/issues/906), [#681](https://github.com/agent0ai/agent-zero/issues/681)

**Status:** ⚠️ Known Bug | **Priority:** High

---

### 9. Startup Failures & Initialization Loops

**What you'll see:**
```
Container restarts repeatedly
Stuck on "Initializing..."
AttributeError on startup
Import errors
Migration failed
```

**Why it happens:**
- Migration from <v0.9 to v0.9.8 failed
- Corrupted config files
- Missing required files
- Permission errors on `/a0/usr`
- Attribute errors (recent GitHub issues #474, #569)

**Status (v0.9.8):** Migration issues during upgrades.

**Fix step-by-step:**

1. **Check logs:**
   ```bash
   docker logs agent-zero | tail -100
   # Look for error messages
   ```

2. **Common AttributeError fix:**
   ```bash
   # Often caused by corrupted config
   docker exec agent-zero rm /a0/usr/config.json
   docker restart agent-zero
   # Will rebuild config from defaults
   ```

3. **Clean migration:**
   ```bash
   # Backup first
   docker cp agent-zero:/a0/usr ~/backup-$(date +%Y%m%d)
   
   # Remove and recreate
   docker stop agent-zero && docker rm agent-zero
   docker pull agent0ai/agent-zero:latest
   docker run -d -p 50080:80 \
     -v ~/agent-zero-data:/a0/usr \
     --name agent-zero agent0ai/agent-zero
   ```

4. **Permission fix (Linux):**
   ```bash
   # If permission denied errors
   sudo chown -R $USER:$USER ~/agent-zero-data
   docker restart agent-zero
   ```

5. **Check port conflicts:**
   ```bash
   # Ensure 50080 is free
   lsof -i :50080
   # Or use different port: -p 50081:80
   ```

**What NOT to do:**
- ❌ Force-killing container repeatedly
- ❌ Editing files during startup
- ❌ Mapping volumes from different versions
- ❌ Running as root unnecessarily

**Prevention:**
- Read migration notes before upgrading
- Backup before major version jumps
- Test migrations on separate instance
- Keep logs of successful configurations

**Sources:** [GitHub #569](https://github.com/agent0ai/agent-zero/issues/569), [#474](https://github.com/agent0ai/agent-zero/issues/474)

**Status:** ⚠️ Ongoing | **Priority:** Critical

---

### 10. Terminal Output Empty

**What you'll see:**
- Commands run but return no output
- "No output returned" message
- Exit code 0 (success) but empty results

**Why it happens:**
- SSH session buffer not flushing
- Output capture timing issues
- Environment variable conflicts

**Status (v0.9.8):** Improved with new local terminal interface and PowerShell support (Windows).

**Fix step-by-step:**

1. **Restart container:**
   ```bash
   docker restart agent-zero
   ```

2. **Use explicit markers:**
   - Ask agent to add: `pwd && echo "__END__"`
   - Ensures output is visible

3. **Use full paths:**
   - Instead of `pwd`, use `/bin/pwd`
   - Instead of `ls`, use `/bin/ls -la`

4. **For Windows (NEW in v0.9.4+):**
   - PowerShell now supported
   - Try switching terminal type in settings

**What NOT to do:**
- ❌ Assuming command failed (check exit code)
- ❌ Running same command repeatedly
- ❌ Modifying SSH config without backup

**Prevention:**
- Use verbose flags: `ls -la` vs `ls`
- Verify exit codes, not just output
- Set `SSH_TERM=xterm-256color`

**Sources:** [GitHub #106](https://github.com/agent0ai/agent-zero/issues/106)

**Status:** ⚠️ Improved | **Priority:** Medium

---

### 11. Dev Tunnels & Remote Access (NEW)

**What you'll see:**
```
Tunnel authentication failed
Can't access Agent Zero remotely
Connection refused from external IP
Certificate errors
```

**Why it happens:**
- Microsoft Dev Tunnels not configured (new in v0.9.8)
- Firewall blocking tunnel
- Authentication token missing
- Security restrictions

**Status (v0.9.8):** New remote access feature.

**Fix step-by-step:**

1. **Setup Dev Tunnel (Windows/VS Code):**
   - Install devtunnel CLI
   - Create tunnel: `devtunnel create -a`
   - Start tunnel: `devtunnel start -p 50080`

2. **Verify tunnel:**
   ```bash
   devtunnel show
   # Note the public URL
   ```

3. **Configure Agent Zero:**
   - Settings → Dev Tunnels
   - Enter authentication token
   - Enable remote access

4. **Security settings:**
   - Settings → Authentication
   - Enable login requirement for remote access
   - Set strong password

**What NOT to do:**
- ❌ Exposing without authentication
- ❌ Using weak passwords for remote access
- ❌ Exposing to public internet without firewall
- ❌ Sharing tunnel URLs publicly

**Prevention:**
- Always require authentication
- Use VPN for sensitive work
- Monitor access logs
- Rotate credentials regularly

**Sources:** [v0.9.8 Release Notes](https://github.com/agent0ai/agent-zero/releases/tag/v0.9.8)

**Status:** 🆕 New Feature | **Priority:** Medium

---

## 🟡 Medium Priority Errors

### 12. Subagent Communication Failures (NEW)

**What you'll see:**
- Subagent not responding
- Agent number conflicts
- Memory not shared between agents
- Crossed communication

**Why it happens:**
- New subagents system in v0.9.8
- Agent profiles not configured
- Memory isolation issues

**Fix:**
- Settings → Subagents → Configure Profiles
- Ensure unique agent numbers
- Check agent communication logs

**Status:** 🆕 New Feature | **Priority:** Medium

---

### 13. UI File Editor Corruption (NEW)

**What you'll see:**
- File corrupted after browser edit
- Save failed
- Encoding issues

**Why it happens:**
- New in-browser file editor (v0.9.8)
- Concurrent edits
- Large files

**Fix:**
- Use external editor for critical files
- Keep backups before editing
- Avoid editing >10MB files in browser

**Status:** 🆕 New Feature | **Priority:** Low

---

### 14. MCP Server Improvements

**Status (v0.9.8):** Improved - streamable HTTP MCP servers now supported. Remote sessions more reliable.

**What changed:**
- Better session management
- HTTP streaming support
- Reduced timeout issues

**Still problematic:**
- Complex remote MCP setups
- Some legacy MCP tools

**Sources:** [GitHub #859](https://github.com/agent0ai/agent-zero/issues/859)

---

## 🖥️ Platform-Specific Setup (Updated for v0.9.8)

### 💻 Low-Spec / Minimal System Setup

**UPDATED Resource Requirements:**
- **Absolute Minimum:** 2 cores, **8GB RAM** (was 6GB)
- **Recommended:** 4 cores, 16GB RAM
- **Why increased:** WebSocket infrastructure + Skills system overhead

**Model Selection:**

**For 8GB RAM systems:**
```bash
# Smallest viable models
ollama pull qwen2.5:0.5b    # 395MB
ollama pull phi3:mini       # 2.3GB
ollama pull nomic-embed-text  # 274MB (embedding)
```

**For 12GB RAM systems:**
```bash
ollama pull llama3.2:3b     # ~2GB
ollama pull qwen2.5:3b      # ~2GB  
ollama pull mxbai-embed-large  # 670MB
```

**Docker Limits (8GB System):**
```bash
docker run -d -p 50080:80 \
  -v ~/agent-zero-data:/a0/usr \
  --memory="4g" \
  --memory-swap="6g" \
  --cpus="2" \
  --add-host=host.docker.internal:host-gateway \
  --name agent-zero agent0ai/agent-zero
```

---

### Windows Quick Start

**Prerequisites:**
- Windows 10/11 (64-bit)
- 8GB RAM minimum (16GB recommended)
- 25GB free disk space

**Setup:**

1. **Install Docker Desktop:**
   - Download from https://docker.com
   - Settings → Resources:
     - CPUs: 4+
     - Memory: 8GB+
   - Apply & Restart

2. **Install Ollama:**
   ```powershell
   # Download from ollama.com
   ollama pull llama3.2:3b
   ollama pull mxbai-embed-large
   ```

3. **Start Agent Zero (v0.9.8):**
   ```powershell
   docker run -d -p 50080:80 `
     -v C:\agent-zero-data:/a0/usr `
     --ulimit nofile=65536:65536 `
     --name agent-zero agent0ai/agent-zero
   ```

4. **Configure:**
   - Open http://localhost:50080
   - Settings → Agent Settings
   - API Base URL: `http://host.docker.internal:11434`
   - Models: Use exact names from `ollama list`

---

### Linux Quick Start

**Prerequisites:**
- Ubuntu 20.04+ / Debian 11+
- 8GB RAM minimum
- 25GB free space

**Setup:**

1. **Install Docker:**
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   # Log out and back in
   ```

2. **Install Ollama:**
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   ollama pull llama3.2:3b
   ollama pull mxbai-embed-large
   ```

3. **Start Agent Zero (v0.9.8):**
   ```bash
   docker run -d -p 50080:80 \
     -v ~/agent-zero-data:/a0/usr \
     --add-host=host.docker.internal:host-gateway \
     --ulimit nofile=65536:65536 \
     --name agent-zero agent0ai/agent-zero
   ```

4. **Configure:**
   - Settings → Agent Settings
   - API Base URL: `http://host.docker.internal:11434`

---

### macOS Quick Start

**Prerequisites:**
- macOS 11+ (Intel or Apple Silicon)
- 8GB RAM minimum

**Setup:**

1. **Install Docker Desktop:**
   - Download from docker.com
   - Settings → Resources:
     - Memory: 8GB+
     - CPUs: 4+

2. **Install Ollama:**
   ```bash
   brew install ollama
   ollama pull llama3.2:3b
   ollama pull mxbai-embed-large
   ```

3. **Start Agent Zero:**
   ```bash
   docker run -d -p 50080:80 \
     -v ~/agent-zero-data:/a0/usr \
     --ulimit nofile=65536:65536 \
     --name agent-zero agent0ai/agent-zero
   ```

---

## 🚨 Emergency Fix Cards

### Card #1: Can't Connect to Ollama

```bash
docker stop agent-zero && docker rm agent-zero

docker run -d -p 50080:80 \
  -v ~/agent-zero-data:/a0/usr \
  --add-host=host.docker.internal:host-gateway \
  --ulimit nofile=65536:65536 \
  --name agent-zero agent0ai/agent-zero
```

**Then:** API Base URL = `http://host.docker.internal:11434`

---

### Card #2: Memory Errors

```bash
docker exec agent-zero rm -rf /a0/memory/*
docker restart agent-zero
```

**Warning:** Deletes all learned memories

---

### Card #3: Complete Fresh Start

```bash
# 1. Backup via Settings → Backup & Restore

# 2. Remove everything
docker stop agent-zero && docker rm agent-zero
docker pull agent0ai/agent-zero:latest

# 3. Fresh start (v0.9.8)
docker run -d -p 50080:80 \
  -v ~/agent-zero-data:/a0/usr \
  --add-host=host.docker.internal:host-gateway \
  --ulimit nofile=65536:65536 \
  --name agent-zero agent0ai/agent-zero

# 4. Restore from backup via UI
```

---

### Card #4: Skills Won't Import

```bash
# Remove legacy instruments
docker exec agent-zero rm -rf /a0/instruments/

# Restart
docker restart agent-zero

# Re-import as Skills via UI
```

---

### Card #5: Settings Not Saving

```bash
# Check for conflicting env vars
docker exec agent-zero env | grep A0_SET

# Remove from .env file, then:
docker restart agent-zero
```

---

## 📚 Verified Issues (v0.9.8)

### ✅ Confirmed Bugs

| Error | GitHub Issue | Status | Severity |
|-------|-------------|--------|----------|
| Socket Closed (SSH) | [#648](https://github.com/agent0ai/agent-zero/issues/648), [#680](https://github.com/agent0ai/agent-zero/issues/680) | Open | Critical |
| FAISS Memory Errors | [#615](https://github.com/agent0ai/agent-zero/issues/615), [#759](https://github.com/agent0ai/agent-zero/issues/759) | Open | Critical |
| File Descriptor Leak | [#906](https://github.com/agent0ai/agent-zero/issues/906) | Open | High |
| Terminal Output Empty | [#106](https://github.com/agent0ai/agent-zero/issues/106) | Improved | Medium |
| Startup Loops | [#569](https://github.com/agent0ai/agent-zero/issues/569), [#474](https://github.com/agent0ai/agent-zero/issues/474) | Open | Critical |
| Browser Tool Issues | [#723](https://github.com/agent0ai/agent-zero/issues/723) | Open | High |

### 🆕 New in v0.9.8
- Skills import failures
- Git project authentication
- WebSocket connection drops
- Configuration conflicts (`A0_SET_*`)
- Migration issues
- Subagent communication

---

## 🎓 What Improved in v0.9.8

**Resolved/Improved:**
- ✅ Built-in Backup & Restore (Settings)
- ✅ User data in `/usr` (cleaner separation)
- ✅ Memory operations deferred (better performance)
- ✅ Real-time WebSocket UI (no more polling)
- ✅ Process groups (better error visibility)
- ✅ MCP improvements (HTTP streaming)
- ✅ PowerShell support (Windows terminals)

**Still Problematic:**
- ❌ SSH socket closed during long operations
- ❌ FAISS corruption when switching models
- ❌ Browser file descriptor leaks
- ❌ Startup initialization loops

---

## 📖 Appendix

### Quick Reference Commands

**Diagnostic:**
```bash
docker stats agent-zero
docker logs agent-zero | tail -50
docker exec agent-zero ls -lh /a0/memory/
```

**Backup:**
```bash
# Primary: Use Settings → Backup & Restore
# Manual:
docker cp agent-zero:/a0/usr ~/backup-$(date +%Y%m%d)
```

**Emergency:**
```bash
docker restart agent-zero
docker logs agent-zero | grep -i error
```

---

**Last Updated:** February 11, 2026  
**Guide Version:** 2.0 (v0.9.8)  
**Maintenance:** Review after each Agent Zero release

**Remember:** Always backup before troubleshooting. Use Settings → Backup & Restore as your primary protection.