# Claude plugin marketplace: verified facts for the HVN marketplace (2026-10-21)

## Bottom line
- **Recommended setup:** one public GitHub repo with `.claude-plugin/marketplace.json` at the root and 7 plugins under `./plugins/<name>`, each using a relative-path `source`.
- **What works where:**
  - Skills and a remote `type: http` MCP entry load in Claude Code, claude.ai chat, Desktop and Cowork.
  - A top-level `bin/` makes claude.ai and Cowork refuse the plugin.
  - Plugin `dependencies` are Claude Code-only.
- **Shared files:** copy the common file into each plugin at build time. This is the only route verified to work on every surface and to pass directory validation, because the directory blocks symlinks.
- **Directory:** the names that carry third-party brands (`-claude`, `-chatgpt`, `-gemini`, `-google-workspace`, `-microsoft-365`) are held for brand review. Content that is mainly promotional is excluded by policy (§4C).

Source credibility: everything in sections 1–7 comes from official Anthropic docs (code.claude.com, claude.com/docs, support.claude.com), fetched 2026-10-21. Section 8 uses each vendor's own docs.

## 1. Schemas
**marketplace.json** (https://code.claude.com/docs/en/plugins/marketplace-reference)
- **Required fields:** `name`, `owner` (`owner.name` is required; `email` and `url` are optional) and `plugins[]`.
- **Optional fields:** `$schema`, `description` (validate warns if it is missing), `version`, `metadata.{description,version,pluginRoot}`, `forceRemoveDeletedPlugins`, `allowCrossMarketplaceDependenciesOn`, `renames`.
- **`name` rules:** letters, digits, `.` `_` `-`; must start alphanumeric; no `..`; no non-ASCII. Reserved names include `claude-plugins-official`, `agent-skills`, `claudeai-*`, `npm`/`github`, and anything that impersonates "official claude".
- **Plugin entry, required:** `name` and `source`.
- **Plugin entry, optional:** `description`, `version`, `category`, `tags`, `strict` (default true), `dependencies`, `defaultEnabled`, `displayName`, `metadata`, `relevance`, `headers`/`headersHelper` (archive only). It also accepts any plugin.json field except the directory-listing fields.
- **Same-repo source:** `"./plugins/x"`, which must start with `./` and must not contain `..`. It resolves from the repo root, not from `.claude-plugin/`. A bare `"x"` works only with `metadata.pluginRoot`, and org-sync rejects that form.
- **Other source types:** `github{repo,ref,sha}`, `url`, `git-subdir{url,path}`, `npm`, `archive{url,sha256}`, `command`.
- **Name match:** keep each entry `name` identical to the `name` in that plugin's plugin.json.

```json
{ "name": "hvn-marketplace", "description": "HVN GROUP guides (Vietnamese)",
  "owner": { "name": "HVN GROUP", "url": "https://hvn.vn" },
  "plugins": [ { "name": "hvn-advisor", "source": "./plugins/hvn-advisor",
                 "description": "..." } ] }
```
**plugin.json** (https://code.claude.com/docs/en/plugins/manifest-reference)
- The file is optional. `name` is its only required field: no spaces, `@`, `:` or `/`; kebab-case.
- **Optional fields:** `displayName`, `version`, `description`, `author{name,email,url}`, `homepage` (must parse as a URL), `repository`, `license`, `keywords`, `metadata`, `defaultEnabled`, `dependencies`, `userConfig`, and component paths (`skills`, `commands`, `agents`, `hooks`, `mcpServers`, …).
- **Path rule:** component paths must start with `./` and stay inside the plugin root.
- **Directory-only fields:** `icon`, `documentationUrl`, `supportUrl`, `privacyPolicyUrl` and `termsOfServiceUrl` are read only by the directory.
- **Reserved names:**
  - Error: a name that starts with `claude-`/`anthropic-`, or puts `official` beside `claude`/`anthropic`.
  - Warning: `claude` or `anthropic` as a whole word anywhere else. **`hvn-claude` therefore warns, and `--strict` fails on it.**

```json
{ "name": "hvn-advisor", "displayName": "HVN Advisor", "version": "1.0.0",
  "description": "...", "author": { "name": "HVN GROUP" }, "license": "MIT",
  "homepage": "https://hvn.vn" }
```
**Versioning:** if `version` is set, users get nothing new until you bump it. If it is omitted, the git commit SHA is used as the version (https://code.claude.com/docs/en/plugins/loading).

## 2. Layout and SKILL.md
**Default plugin layout** (manifest reference, "Standard layout"):
- `.claude-plugin/plugin.json` (only the manifest goes inside `.claude-plugin/`)
- `skills/<name>/SKILL.md`, `commands/*.md`, `agents/*.md`, `hooks/hooks.json`
- `.mcp.json`, `.lsp.json`, `output-styles/`, `bin/`, `settings.json`

A `CLAUDE.md` at the plugin root is not loaded, and validate warns about it.

**SKILL.md frontmatter by surface:**

| Surface | Required | Limits |
|---|---|---|
| Agent Skills spec (https://agentskills.io/specification) | `name`, `description` | name 1–64 chars, `a-z0-9-`, no leading, trailing or double hyphen, **must equal the folder name**; description 1–1024 chars; `compatibility` ≤500 |
| claude.ai / Cowork uploads (https://claude.com/docs/skills/how-to) | both | name ≤64 and must match the directory; description ≤1,024. Only these fields are allowed: `name, description, license, compatibility, metadata, allowed-tools`. Any other field is a hard error (https://code.claude.com/docs/en/skills) |
| Claude Code (https://code.claude.com/docs/en/skills) | none (description recommended) | `description` + `when_to_use` are truncated at 1,536 chars; keep SKILL.md under 500 lines |

**Rule to use:** write to the spec subset. That means an ASCII kebab `name` that matches the folder, and a Vietnamese `description` of 1,024 characters or fewer. Non-ASCII text in the description is not prohibited by any source I found.

## 3. Installing on each surface
- **Claude Code CLI** (https://code.claude.com/docs/en/plugins/install):
  - Add the marketplace with `/plugin marketplace add hvn-group/<repo>` (or `claude plugin marketplace add …`), then run `claude plugin install hvn-advisor@hvn-marketplace`.
  - Or do both in one step: `claude plugin install hvn-advisor --marketplace hvn-group/<repo>` (v2.1.292+).
  - Auto-update is **off by default** for third-party marketplaces. Users turn it on in `/plugin` > Marketplaces.
- **Desktop, Code tab:** **+ > Add plugins**. It shares settings files with the CLI.
- **claude.ai web, Desktop chat and Cowork** (https://claude.com/docs/plugins/overview):
  - Go to **Customize > Plugins > Add > Add marketplace** and enter `owner/repo` or a GitHub URL. **Yes, a non-developer can add a public third-party GitHub marketplace.** Upload of a `.zip` is also supported.
  - Installs belong to the account. They appear in Cowork, and in Claude Code as `<name>@synced`.
  - Limits: 25 self-added marketplaces per account per org; 5,000 files / 200 MB per plugin (https://claude.com/docs/plugins/platform-support).
  - On Team and Enterprise, members only see the Add options if the Owner left the **User-created skills** policy on (https://claude.com/docs/plugins/admin).
- **Org push, claude.ai** (https://claude.com/docs/plugins/admin, https://claude.com/docs/plugins/org-sync):
  - An Owner uses Organization settings > Plugins & skills to sync from GitHub/GitLab or upload a zip, then sets availability to Available, Installed by default, or Required. On Enterprise this can also be set per group.
  - **Catch:** org sync requires the marketplace repo on github.com to be **private or internal**. HVN's public repo can't be org-synced directly. A customer admin would have to mirror it into a private repo or upload the zips.
- **Org push, Claude Code fleets:** managed settings `extraKnownMarketplaces` + `enabledPlugins` (https://code.claude.com/docs/en/plugins/host-marketplace#roll-out-to-a-whole-company).

## 4. Dependencies and shared files
- **Dependencies are supported in Claude Code.** In plugin.json: `"dependencies": ["hvn-core", {"name":"x","version":"^1.0"}]`. Bare names resolve inside the same marketplace, and Claude Code auto-installs them (https://code.claude.com/docs/en/plugins/dependencies).
  - On claude.ai and Cowork, "dependencies between plugins" is listed as a Claude Code-only component (https://claude.com/docs/plugins/build).
  - A synced plugin's dependency is satisfied only if the user also enables that dependency.
- **No runtime file sharing.** Cached plugins are copied, and `../shared` is neither copied nor allowed. A component path that escapes the plugin root is rejected (https://code.claude.com/docs/en/plugins/loading).
- **Symlinks:** a symlink inside a plugin that points to another folder in the same marketplace is dereferenced (copied in) at Claude Code install (https://code.claude.com/docs/en/plugins/host-marketplace#share-files-within-a-marketplace-with-symlinks). However:
  - The directory **blocks** symlinks (https://claude.com/docs/plugins/pre-submission-checklist).
  - `claude plugin validate` doesn't follow them.
  - claude.ai "Add marketplace" handling of symlinks is **UNVERIFIED**.

**Options, ranked:**
1. **Build-time copy** of `shared/*.md` into each `plugins/*/skills/<skill>/references/`, with a CI check for drift. Works on all surfaces and with the directory.
2. **Symlink.** Claude Code only.
3. **`hvn-core` plugin + `dependencies`.** Claude Code only, and the skill still can't read the other plugin's path reliably.

## 5. Validator
- `claude plugin validate <path> [--strict] [--json]`. The path can be a plugin dir, the marketplace root or a manifest file.
  - Exit codes: 0 = passed (possibly with warnings), 1 = failed, 2 = validator error.
  - In a session: `/plugin validate <path>`.
  - A marketplace run does **not** open the component files of listed plugins, so validate each plugin dir as well (https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate).
- `skills-ref validate ./skill` checks SKILL.md against the spec (https://agentskills.io/specification).
- The directory portal's **Validate** button adds directory rules the CLI doesn't check.

## 6. Anthropic directory
- **Where to submit:** the developer portal at https://claude.ai/directory/manage, on a paid plan (Pro, Max, or a Team/Enterprise Owner). The plugin repo must be public before the listing goes live. Each plugin folder is submitted separately (https://claude.com/docs/directory/publish).
- **Review:** every version gets validation and a security scan, and a person reviews each new listing.
- **Blocking checks (pre-submission checklist):**
  - README of 40+ words and a LICENSE
  - no symlinks, submodules or LFS files
  - no `.DS_Store`
  - remote MCP URLs must be `https://`
  - no secrets
  - pinned `npx`/`uvx` versions
- **Held-for-review checks (same checklist):**
  - "Name matches a known brand"
  - name collides with an existing listing
  - names made only of generic words
- **Policy** (https://support.claude.com/en/articles/13145358-anthropic-software-directory-policy):
  - §4C excludes software that serves ads, sponsored content or paid placement, or that is "primarily an advertising or promotional vehicle".
  - §2B: descriptions must match what the plugin actually does.
  - §2D: a plugin must not coerce Claude into calling other software.
  - §2F: a plugin must not pull instructions from external sources at runtime.
  - §3A: a privacy policy link is required if the plugin handles user data or makes remote connections.
  - The policy has no explicit rule on affiliate links or localization.
- **Official marketplace:** `claude-plugins-official` takes no portal submissions (https://code.claude.com/docs/en/plugins/publish).

## 7. Remote MCP without auth
- **Plugin `.mcp.json`:** `{"mcpServers":{"hvn":{"type":"http","url":"https://mcp.hvn.vn/mcp"}}}`. Headers are optional, and `type` is required; without it the entry is treated as stdio and skipped (https://code.claude.com/docs/en/mcp).
  - Validate warns on `http://` URLs to non-loopback hosts, and the directory blocks non-https URLs.
- **Behavior on claude.ai and Cowork:** the server shows on the plugin's Connectors tab. Each user must add or connect it there; on Team/Enterprise only an Owner can add it. Claude Code connects directly (https://claude.com/docs/plugins/build).
- **Custom connectors:** the dialog offers **"No sign-in"** as an authentication option.
  - Free plan: one custom connector. Pro and Max: users add their own.
  - Team/Enterprise: the Owner adds the connector (https://claude.com/docs/connectors/custom/add-unlisted).
  - The support article https://support.claude.com/en/articles/11175166 also says "authless and OAuth" servers are supported.
- **Directory:** submit the server separately as an "MCP connector".

## 8. Agent Skills in other tools
| Tool | Project path | User path | Source |
|---|---|---|---|
| Gemini CLI | `.gemini/skills/` or `.agents/skills/` | `~/.gemini/skills/` or `~/.agents/skills/`; `gemini skills install <git-url>` | https://geminicli.com/docs/cli/skills/ (replaced by Antigravity CLI for free and Google One users on 2026-06-18) |
| Antigravity | `.agents/skills/` (legacy `.agent/skills/`) | App/IDE: `~/.gemini/config/skills/`; CLI: `~/.gemini/antigravity-cli/skills/` | https://antigravity.google/docs/skills |
| OpenAI Codex | `.agents/skills` from the cwd up to the repo root | `~/.agents/skills`; admin: `/etc/codex/skills` | https://learn.chatgpt.com/docs/build-skills |
| Cursor | `.agents/skills/` or `.cursor/skills/`; also reads `.claude/skills`, `.codex/skills` | `~/.agents/skills/`, `~/.cursor/skills/` (+ `~/.claude`, `~/.codex`) | https://cursor.com/docs/context/skills |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` | https://code.claude.com/docs/en/skills |

**Shared convention:** `.agents/skills/` is the cross-client path (https://agentskills.io/client-implementation/adding-skills-support.md). Shipping the same skill folders works across all of these tools; only the install location differs. Claude Code is the only one in this table that does not list `.agents/skills`.

## Trade-offs
| Option | Surfaces | Effort | Risk |
|---|---|---|---|
| Own public marketplace (recommended) | CLI, claude.ai, Desktop, Cowork | Low | Users must find it; auto-update is off in Claude Code |
| Anthropic directory | All, discoverable | Medium (portal, README, LICENSE, review) | Brand names and promotional content are likely to be held or rejected |
| Org sync at customer | claude.ai/Cowork, required/default | Customer admin work | Needs a private mirror repo |

**Adoption risk:** the plugin system is moving fast. The docs cite features up to v2.1.29x and the field set changes often. Pin docs to a date and run `claude plugin validate` in CI.

## Limitations and open questions
1. Whether claude.ai "Add marketplace" (user or Owner) dereferences symlinks. **UNVERIFIED.**
2. Whether the Free plan can use **Add marketplace** on claude.ai. Not stated (**UNVERIFIED**); the directory is listed for Pro+ only.
3. Whether a Team/Enterprise Owner can add a *public* third-party GitHub marketplace through **Add > Add marketplace**, as opposed to Sync from GitHub. **UNVERIFIED**: the docs describe that option as "e.g. Anthropic's catalogs".
4. The agentskills.io client showcase renders client-side, so the full adopter list was not enumerated.
5. Naming: decide whether to keep `hvn-claude` (validator warning; directory brand hold) or rename it, for example to `hvn-anthropic-guide`, which would also warn. Any name with `claude` as a whole word warns.

Status: DONE_WITH_CONCERNS
Summary: All 8 questions answered from official Anthropic docs (and each vendor's docs for item 8). Plugin dependencies and symlinks are Claude Code-only, so shared files should be copied at build time; the brand-named plugins and the promotional-content policy are risks for a directory listing.
Concerns/Blockers: Three points UNVERIFIED (claude.ai symlink handling, Free-plan marketplace add, Owner adding a public third-party marketplace); the public repo can't be org-synced directly.
