# Claude Code Leak Threat Assessment (2026-04-01)

## Bottom line

This repository should be treated as **untrusted third-party code**. It is **not safe to run directly on your host machine with real credentials or sensitive files**.

A cautious path is possible (isolated VM/container, no secrets, locked dependency policy), but **"safe by default" is no**.

## Online-first threat directions to investigate

Use these directions first when evaluating any leaked or mirrored codebase:

1. **Provenance & ownership checks**
   - Is the repo under an official owner (Anthropic org) or a personal mirror?
   - Are there signed tags/releases and reproducible build metadata?
   - Are there known statements from vendor/security researchers about the leak timeline?

2. **Distribution poisoning checks**
   - Look for lookalike npm packages, typo-squats, and re-published binaries.
   - Compare package names/scopes used by source with what exists publicly.
   - Verify whether any install docs pipe remote scripts directly (`curl | bash` patterns).

3. **Dependency confusion checks**
   - Enumerate all internal/private-looking package names in imports.
   - Confirm whether those scopes are publicly claimable.
   - Inspect lockfiles for registry pinning/integrity fields and unexpected tarball hosts.

4. **Post-leak social-engineering indicators**
   - Fake "official" setup guides that ask for API keys, wallet keys, SSH keys.
   - Forks adding one-liner installers, hidden postinstall hooks, or obfuscated binaries.

5. **Runtime data-exposure paths**
   - Any local web/WS server defaulting to no auth.
   - Any default telemetry, remote uploads, remote task execution.
   - Any plugin/skill auto-install, shell execution, git hook execution.

6. **Environment hardening before first run**
   - Disposable environment only, no prod credentials.
   - Egress filtering and read-only mounts.
   - Disable auto-update, plugin installation, and unauthenticated web endpoints.

## Code findings in this repository snapshot

### 1) Repo is explicitly a leak mirror, not official
- Root package metadata labels itself as leaked/non-official (`0.0.0-leaked`, "Not an official release").
- README states it is an exploratory mirror from a leak event.

### 2) Supply-chain signals: mostly public registry, but risk remains
- `web/package-lock.json` and `mcp-server/package-lock.json` resolve to `registry.npmjs.org` tarballs with lockfile pinning.
- No obvious `preinstall/postinstall` hooks are defined in top-level package scripts.
- **However** source imports include internal/private-looking packages (e.g. `@ant/*`, `@anthropic-ai/sandbox-runtime`, `@anthropic-ai/mcpb`), creating dependency-confusion pressure if users try to make the source fully runnable by adding missing deps.

### 3) Explicit network + execution surfaces exist
- Local/remote task and shell execution capabilities exist by design.
- Upstream proxy/network allowlist logic includes external domains like Anthropic + npm + GitHub.
- MCP HTTP server can expose source browsing over network.

### 4) Security-aware code exists, but does not make repo trustworthy
- There are built-in mitigations against GH command exfiltration patterns.
- Git trust/worktree validation includes explicit security checks against malicious `.git` metadata tricks.
- Still, these mitigations don't establish package provenance or protect you from a poisoned mirror/fork.

### 5) Configuration foot-guns
- Web auth helper admits all connections if `AUTH_TOKEN` is unset.
- If someone runs provided servers without explicit auth/network boundaries, local data exposure risk increases.

## Practical safety verdict

- **Safe for reverse engineering / reading source?** Yes, with caution.
- **Safe to run normally on personal workstation?** **No (not recommended).**
- **Safe to run in hardened disposable environment?** **Conditionally yes**, if you enforce strict isolation and no secrets.

## Hardening checklist before any execution

1. Run in an ephemeral VM/container with no mounted home directory.
2. Use throwaway credentials only (or none).
3. Block outbound network except exact domains you need.
4. Set explicit auth tokens for any HTTP/WS endpoint.
5. Avoid one-liner installer commands from README/forks unless you independently verify each fetched artifact.
6. Pin dependency install to lockfiles, verify integrity/provenance where possible, and mirror dependencies internally.
7. Prefer static analysis first; execute only minimal commands needed.
8. Treat all forks and reposted bundles as potentially backdoored even if filenames match.

