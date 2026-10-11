# MilMed Connect PitchBook — API Key Discovery Check

Before starting Phase 2, run this check on the MILMED Connect iMac to confirm which credentials the platform can use. It separates "configured," "authenticated," and "usable" — not just "found."

## Explicit credential contract

For each provider, the check looks for an exact, approved source:

1. **Project `.env`** (gitignored) — the var is set.
2. **Current process environment** — the var is already set (from an explicitly sourced shell profile). Do not execute shell profiles.
3. **Project-specific Keychain item** — `security find-generic-password -s "milmed-connect-<provider>" -w`. Not a broad scan.

Do not recursively scan unrelated CLI config directories. Do not assume Claude Code or Codex login credentials are general-purpose API keys.

## How to run

From the repo root on the iMac:

```bash
source .venv/bin/activate   # if the venv exists
python -m api.config_check
```

If `api/config_check.py` doesn't exist yet, run this inline:

```bash
python3 << 'EOF'
import os, subprocess, json

# Explicit contract: named provider → exact env var or project-specific Keychain item
PROVIDERS = {
    "anthropic": {"env": "ANTHROPIC_API_KEY", "keychain": "milmed-connect-anthropic"},
    "openai":    {"env": "OPENAI_API_KEY",    "keychain": "milmed-connect-openai"},
    "typesafe":  {"env": "TYPESAFE_API_KEY",  "keychain": "milmed-connect-typesafe"},
}

results = {}

for name, spec in PROVIDERS.items():
    status = {"configured": False, "source": None, "authenticated": "not tested", "usable": "not tested"}

    # 1. .env (via dotenv if present)
    try:
        from dotenv import load_dotenv
        load_dotenv()
    except ImportError:
        pass

    # 2. Current process environment (already set)
    if os.environ.get(spec["env"]):
        status["configured"] = True
        status["source"] = "environment"

    # 3. Project-specific Keychain item only
    if not status["configured"]:
        try:
            r = subprocess.run(
                ["security", "find-generic-password", "-s", spec["keychain"], "-w"],
                capture_output=True, text=True, timeout=5)
            if r.returncode == 0 and r.stdout.strip():
                status["configured"] = True
                status["source"] = f"Keychain ({spec['keychain']})"
        except Exception:
            pass

    results[name] = status

print(json.dumps(results, indent=2))
print()
print("Next steps:")
print("- Configured = the credential exists. Authenticated = a call succeeds. Usable = billing and permissions are set.")
print("- Test authentication separately with a minimal API call. Do not print key values.")
print("- Missing optional credentials disable that feature; the core product continues.")
EOF
```

## Interpreting results

| Status | Meaning |
|--------|---------|
| configured | The env var or project-specific Keychain item exists |
| authenticated | A minimal API call succeeds (test separately) |
| usable | The account has billing and permissions for this project |

| Found | Phase 2 starts with |
|-------|---------------------|
| Anthropic or OpenAI (usable) | LLM scoring layer |
| TypeSafe/Jev (usable) | Jev scoring layer |
| Both | Benchmark one first, then add the second |
| Neither | Rule-based scoring only |

## Security rules

- Never print or log key values — only source names and status.
- `.env` is gitignored; never commit it.
- Do not recursively scan unrelated CLI configuration.
- Do not execute shell profiles.
- Jev's official SDK reads `TYPESAFE_API_KEY`, not `JEV_API_KEY`.
- The check is safe to run anytime — it never modifies anything.