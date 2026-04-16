# Installer Functionality Audit for gh skill Migration

## Current Installer Responsibilities (4.6.0)

### What It Does Now

The 6,767 lines of installer code handle:

1. **Installation Scope Management**
   - Global scope (`~/.claude/agents/`) 
   - Project scope (`.claude/agents/` in current dir)
   - Multiple VS Code profiles (Stable, Insiders)
   - Multiple MCP profiles

2. **Agent/Skill File Management**
   - Copy agents to installation directory
   - Copy skills to installation directory
   - Create manifests
   - Verify file integrity
   - Handle version conflicts

3. **Configuration Management**
   - Role-based installation (developer/reviewer/author/full/custom)
   - Team config JSON support
   - Config merging
   - Environment variable setup

4. **Platform-Specific Setup**
   - Windows PowerShell detection
   - macOS/Linux shell detection
   - VS Code extension installation (Copilot)
   - Claude Desktop MCP profile setup
   - Codex CLI setup
   - Gemini CLI setup

5. **Runtime Validation & Checks**
   - Node.js version check (MCP server requirement)
   - Java version check (optional, for advanced features)
   - Playwright core availability
   - MCP health smoke tests
   - PDF validation readiness checks

6. **Git Hooks Installation**
   - Install pre-commit hooks
   - Register hooks globally
   - Validate hook execution

7. **Repair & Maintenance**
   - Post-install validation
   - Auto-repair functionality
   - Manifest regeneration
   - Version consistency checking

8. **User Education**
   - Setup wizard
   - Interactive prompts
   - Capability warnings
   - Readiness reports

---

## What `gh skill` Handles Automatically

| Functionality | gh skill | Notes |
|--------------|----------|-------|
| Cross-platform distribution | ✅ Yes | Windows/Mac/Linux unified |
| Version management | ✅ Yes | Semantic versioning enforced |
| Agent/skill file delivery | ✅ Yes | Downloaded to `~/.gh/`
| Installation verification | ✅ Yes | Checksum validation |
| Automatic updates | ✅ Yes | Built-in upgrade mechanism |
| Marketplace discovery | ✅ Yes | Listed in `gh skill search` |
| Security signing | ✅ Yes | GitHub cryptographic signatures |

---

## What We Still Need to Handle

### CRITICAL (Must Not Lose)

These features **cannot be handled by gh skill** and must be preserved:

1. **Git Hooks Installation** ❌ gh skill doesn't do this
   - Currently: `Install-GlobalHooks` function in install.ps1
   - Must preserve: Pre-commit hook setup for accessibility checks
   - Solution: Post-install script or separate `gh skill hook install` command

2. **Runtime Dependency Checks** ❌ gh skill doesn't validate runtimes
   - Currently: Node.js, Java, Playwright validation
   - Must preserve: Warn users if Node.js is missing (MCP server needs it)
   - Solution: First-run diagnostic script

3. **Configuration Management** ❌ gh skill doesn't support role-based config
   - Currently: Role-based installation (developer/reviewer/author/full/custom)
   - Must preserve: Team config JSON support, role selection
   - Solution: Interactive setup wizard after `gh skill install`

4. **MCP Profile Management** ❌ gh skill doesn't manage VS Code settings
   - Currently: Configure VS Code MCP settings for both profiles
   - Must preserve: Automatic Claude Desktop MCP setup
   - Solution: Post-install setup script for each platform

5. **Repair & Validation** ❌ gh skill doesn't provide repair commands
   - Currently: `./install.ps1 -Check` validates everything
   - Must preserve: `gh skill repair` or similar command
   - Solution: Separate CLI utility or `gh skill` subcommand

6. **Scope Management** ❌ gh skill installs to one location only
   - Currently: Project vs Global installation choice
   - Must preserve: Ability to install globally or per-project
   - Solution: `gh skill install --scope global|project` flag

---

## Migration Strategy: Preserve All Functionality

### Phase 1: Core gh skill Installation
```bash
gh skill install Community-Access/accessibility-agents
# Handles: agent/skill distribution, version management, cross-platform
```

### Phase 2: Post-Install Setup (Interactive)
```bash
gh skill setup Community-Access/accessibility-agents
# User prompted for:
#   - Role (developer/reviewer/author/full/custom)
#   - Scope (global/project)
#   - Platform preferences (VS Code stable/insiders, Claude Desktop, Codex, Gemini)
#   - Team config JSON (optional)
```

### Phase 3: Runtime Validation
```bash
gh skill health Community-Access/accessibility-agents
# Checks:
#   - Node.js version
#   - Java version
#   - Playwright status
#   - MCP server readiness
#   - Git hooks status
```

### Phase 4: Repair & Maintenance
```bash
gh skill repair Community-Access/accessibility-agents
# Actions:
#   - Reinstall Git hooks
#   - Regenerate manifests
#   - Fix broken installations
#   - Validate version consistency
```

### Phase 5: Git Hooks Management
```bash
gh skill hooks Community-Access/accessibility-agents install|uninstall
# Handles:
#   - Pre-commit hook installation
#   - Hook registration
#   - Global hook management
```

---

## New Architecture with gh skill

```
┌─────────────────────────────────────┐
│  GitHub Release (agent/skill files) │
└──────────────┬──────────────────────┘
               │
        ┌──────▼──────────┐
        │ gh skill CLI    │  1. Install distribution
        │ (GitHub's code) │
        └──────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Post-Install Setup  │  2. Role/scope/platform config
    │ (our CLI utility)   │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Git Hooks Setup     │  3. Install pre-commit hooks
    │ (our CLI utility)   │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Runtime Validation  │  4. Check Node.js, Java, Playwright
    │ (our CLI utility)   │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Ready to Use!       │
    │ Agents in editor    │
    └─────────────────────┘
```

---

## Files to Create

### 1. `.github/cli/setup.js` (Post-Install Setup)
Handles:
- Interactive role selection
- Scope management (global/project)
- Platform preferences
- Team config loading
- MCP profile configuration

```bash
node .github/cli/setup.js
# Or via gh skill: gh skill setup Community-Access/accessibility-agents
```

### 2. `.github/cli/health.js` (Runtime Validation)
Checks:
- Node.js version
- Java version
- Playwright core
- MCP server
- Git hooks status

```bash
node .github/cli/health.js
# Or via gh skill: gh skill health Community-Access/accessibility-agents
```

### 3. `.github/cli/repair.js` (Fix Broken Installations)
Actions:
- Reinstall Git hooks
- Regenerate manifests
- Fix configuration
- Validate consistency

```bash
node .github/cli/repair.js --auto-repair
# Or via gh skill: gh skill repair Community-Access/accessibility-agents
```

### 4. `.github/cli/hooks.js` (Git Hook Management)
Manages:
- Pre-commit hook installation
- Hook registration
- Global hook paths
- Uninstall cleanup

```bash
node .github/cli/hooks.js install|uninstall
# Or via gh skill: gh skill hooks Community-Access/accessibility-agents install
```

---

## Nothing Gets Lost: Feature Mapping

| Current Feature | 4.6.0 | 5.0.0+ | Handler |
|-----------------|-------|--------|---------|
| **Installation** | install.ps1/sh | `gh skill install` | GitHub |
| **Agent distribution** | Manual copy | Auto-download | GitHub |
| **Updates** | Manual script | `gh skill upgrade` | GitHub |
| **Version management** | Manual check | Semver enforced | GitHub |
| **Scope (global/project)** | install.ps1 flags | `gh skill setup` | Our CLI |
| **Role configuration** | install.ps1 flags | `gh skill setup` | Our CLI |
| **Team config JSON** | install.ps1 -Config | `gh skill setup` | Our CLI |
| **Git hooks** | install.ps1 -GlobalHooks | `gh skill hooks install` | Our CLI |
| **Runtime validation** | Built-in checks | `gh skill health` | Our CLI |
| **MCP setup** | Embedded in installer | `gh skill setup` | Our CLI |
| **Repair/fix** | Validation scripts | `gh skill repair` | Our CLI |
| **Platform detection** | install.ps1 logic | gh CLI handles | GitHub |
| **VS Code setup** | Copilot install logic | `gh skill setup` | Our CLI |

---

## Implementation: What Needs to Be Built

### Before 5.0.0 Release

- [x] Create migration guide ✅
- [x] Document adoption plan ✅
- [ ] Build `.github/cli/setup.js` (post-install wizard)
- [ ] Build `.github/cli/health.js` (runtime validation)
- [ ] Build `.github/cli/repair.js` (fix broken installs)
- [ ] Build `.github/cli/hooks.js` (git hook management)
- [ ] Update docs for `gh skill setup` workflow
- [ ] Create interactive setup tests
- [ ] Verify all functionality preserved in new CLI utilities

---

## User Experience: Side-by-Side Comparison

### 4.6.0 (Old Way)
```bash
# Download and run installer
irm https://raw.githubusercontent.com/.../install.ps1 | iex -Force

# Options passed via flags
.\install.ps1 -Global -Copilot -Role developer -Yes

# Updates are manual
.\install.ps1 -Force

# Check health
.\install.ps1 -Check

# Fix issues
.\scripts\repair-install.ps1
```

### 5.0.0+ (New Way)
```bash
# Install skill (GitHub handles distribution)
gh skill install Community-Access/accessibility-agents

# Interactive setup wizard (post-install)
gh skill setup Community-Access/accessibility-agents
# Prompts: role? scope? platforms? config file?

# Updates are automatic
gh skill upgrade Community-Access/accessibility-agents

# Check health
gh skill health Community-Access/accessibility-agents

# Fix issues
gh skill repair Community-Access/accessibility-agents

# Manage Git hooks
gh skill hooks Community-Access/accessibility-agents install
```

---

## Risk Assessment

### What Could Break

1. **Git Hooks** — Must rebuild in Node.js CLI
2. **MCP Configuration** — Must handle in setup wizard
3. **Role-based config** — Must preserve in setup script
4. **Runtime checks** — Must implement in health script

### Mitigation

All functionality is **preserved**, just moved from monolithic 2,000+ line installer to focused utility scripts:

- `setup.js` — Interactive configuration (~300 lines)
- `health.js` — Runtime validation (~200 lines)
- `repair.js` — Fix installations (~200 lines)
- `hooks.js` — Git hook management (~150 lines)

**Total: ~850 lines** (vs. 6,767 in old installer)

---

## Recommendation

### YES — Adopt gh skill, BUT:

1. **Build the CLI utilities first** (setup/health/repair/hooks)
2. **Test thoroughly** to ensure feature parity
3. **Document the new workflow** clearly
4. **Maintain backward compatibility** where possible

### What Gets Better

- ✅ Distribution (GitHub native)
- ✅ Updates (automatic)
- ✅ Cross-platform (unified)
- ✅ Security (signed releases)
- ✅ Discoverability (marketplace)

### What Stays the Same

- ✅ Installation scopes (global/project)
- ✅ Role-based config (developer/reviewer/author/full/custom)
- ✅ Team config support (JSON merge)
- ✅ Git hooks (now via `gh skill hooks`)
- ✅ Runtime validation (now via `gh skill health`)
- ✅ Repair functionality (now via `gh skill repair`)
- ✅ MCP setup (now via `gh skill setup`)

**Nothing gets lost. Everything gets simplified.**

---

## Next Steps

1. Build the 4 new CLI utilities (setup/health/repair/hooks)
2. Test feature parity with old installer
3. Document new user workflow
4. Package with 5.0.0 release
5. Deprecate old installers
