# API Key Discovery Check — Run on the MILMED iMac

Before starting Phase 2, run this check on the MILMED Connect iMac to confirm which API keys the platform will actually find. It tells you whether Phase 2 starts with the LLM layer, the Jev layer, or rule-based scoring only.

## What it does

The check mirrors `api/config.py`'s discovery order without printing any secret values:

1. **Project `.env`** — if present, load it (gitignored, never committed).
2. **Shell profile** — `~/.zshrc` and `~/.zprofile` exports (e.g., `export ANTHROPIC_API_KEY=...`).
3. **macOS Keychain** — `security find-generic-password -s "<service>" -w` for services like `anthropic`, `openai`, `typesafe`.
4. **Standard CLI config files** — `~/.config/<tool>/` directories (Claude Code, Codex, etc.).

For each key, it logs **which source supplied it** — never the value itself. It fails loudly naming any missing key and where to put it.

## How to run

From the repo root on the iMac:

```bash
source .venv/bin/activate   # if the venv exists
python -m api.config_check
```

If `api/config_check.py` doesn't exist yet (Phase 1 may not have built it), run this inline:

```bash
python3 << 'EOF'
import os, subprocess, json

SERVICES = {
    "ANTHROPIC_API_KEY": ["anthropic", "claude"],
    "OPENAI_API_KEY":    ["openai", "gpt"],
    "JEV_API_KEY":      ["typesafe", "jev"],
}

results = {"found": {}, "missing": [], "sources": {}}

# 1. .env
try:
    from dotenv import load_dotenv
    load_dotenv()
    for k in SERVICES:
        if os.environ.get(k) and k not in results["found"]:
            results["found"][k] = True
            results["sources"][k] = ".env"
except ImportError:
    pass

# 2. Shell profile (already loaded into os.environ if sourced)
for k in SERVICES:
    if os.environ.get(k) and k not in results["found"]:
        results["found"][k] = True
        results["sources"][k] = "shell profile"

# 3. macOS Keychain
for k, svcs in SERVICES.items():
    if k in results["found"]:
        continue
    for s in svcs:
        try:
            r = subprocess.run(
                ["security", "find-generic-password", "-s", s, "-w"],
                capture_output=True, text=True, timeout=5)
            if r.returncode == 0 and r.stdout.strip():
                results["found"][k] = True
                results["sources"][k] = f"Keychain (service: {s})"
                break
        except Exception:
            pass

# 4. CLI config dirs
for k in SERVICES:
    if k in results["found"]:
        continue
    for tool in ["claude", "codex", "openai", "anthropic"]:
        p = os.path.expanduser(f"~/.config/{tool}")
        if os.path.isdir(p):
            for root, dirs, files in os.walk(p):
                for f in files:
                    try:
                        txt = open(os.path.join(root, f)).read()
                        if k.split("_")[0].lower() in txt.lower():
                            results["found"][k] = True
                            results["sources"][k] = f"~/.config/{tool}/"
                            break
                    except Exception:
                        pass

for k in SERVICES:
    if k not in results["found"]:
        results["missing"].append(k)

print(json.dumps(results, indent=2))
print()
if results["missing"]:
    print("MISSING KEYS — add them to .env, your shell profile, or Keychain:")
    for k in results["missing"]:
        print(f"  - {k}")
else:
    print("All checked keys found. Phase 2 can start with the LLM and/or Jev layers.")
EOF
```

## Interpreting results

| Found | Phase 2 starts with |
|-------|---------------------|
| `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` | LLM scoring layer (evidence citations, summaries) |
| `JEV_API_KEY` | Jev scoring layer (rubric scores, ER borderline queue, contradiction flags) |
| Both | Full stack: Jev decides, LLM explains |
| Neither | Rule-based keyword/tag scoring only — Jev and LLM layers wait on key signup |

## Security rules

- Never print or log key values — only source names.
- `.env` is gitignored; never commit it.
- The suppression list for MTEC contact emails lives in the DB, not in env.
- All ingested data is unclassified public information.

## After the check

- If keys are missing, add them to `~/.zshrc` (persistent across terminals) or Keychain via Keychain Access.app.
- Re-run the check to confirm before starting Phase 2.
- The check is safe to run anytime — it never modifies anything.