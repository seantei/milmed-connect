# MilMed Connect PitchBook — API Key Discovery Check

Before Phase 2, run this on the MILMED Connect iMac to confirm which credentials the platform can use. It separates "configured," "authenticated," and "usable."

## Explicit credential contract

For each provider, the check looks for an exact, approved source, in this order:

1. **Project `.env`** (read via `dotenv_values`, does not mutate process environment)
2. **Current process environment** (already set)
3. **Project-specific Keychain item** — `security find-generic-password -s "milmed-connect-<provider>" -w`

Do not recursively scan unrelated CLI config. Do not execute shell profiles. Do not assume Claude Code or Codex credentials are general-purpose API keys.

## How to run

From the repo root on the iMac:

```bash
python3 << 'EOF'
import os, subprocess, json

PROVIDERS = {
    "anthropic": {"env": "ANTHROPIC_API_KEY", "keychain": "milmed-connect-anthropic"},
    "openai":    {"env": "OPENAI_API_KEY",    "keychain": "milmed-connect-openai"},
    "typesafe":  {"env": "TYPESAFE_API_KEY",  "keychain": "milmed-connect-typesafe"},
}

results = {}

# Explicit .env read (does not mutate os.environ)
try:
    from dotenv import dotenv_values
    env_file = dotenv_values(".env")
except ImportError:
    env_file = None
    print("WARNING: python-dotenv not installed; .env will not be read. Install with: pip install python-dotenv")
except Exception as e:
    env_file = None
    print(f"WARNING: could not read .env: {e}")

for name, spec in PROVIDERS.items():
    status = {
        "configured": False,
        "source": None,
        "authenticated": "not tested",
        "usable": "not tested"
    }

    # 1. Project .env (highest precedence)
    if env_file is not None and env_file.get(spec["env"]):
        status["configured"] = True
        status["source"] = ".env"

    # 2. Current process environment
    elif os.environ.get(spec["env"]):
        status["configured"] = True
        status["source"] = "environment"

    # 3. Project-specific Keychain
    else:
        try:
            r = subprocess.run(
                ["security", "find-generic-password", "-s", spec["keychain"], "-w"],
                capture_output=True, text=True, timeout=5)
            if r.returncode == 0 and r.stdout.strip():
                status["configured"] = True
                status["source"] = f"Keychain ({spec['keychain']})"
            elif r.returncode != 0 and "could not be found" not in (r.stderr or "").lower():
                status["source"] = f"Keychain access denied or error: {r.stderr.strip()[:100]}"
        except Exception as e:
            status["source"] = f"Keychain error: {e}"

    results[name] = status

print(json.dumps(results, indent=2))
print()
print("Next steps:")
print("- Configured = credential exists in an approved source.")
print("- Authenticated = a minimal API call succeeds (test separately).")
print("- Usable = account has billing and permissions for this project.")
print("- Missing optional credentials disable that feature; the core product continues.")
print("- Do not print key values.")
EOF
```

## Interpreting results

| Status | Meaning |
|--------|---------|
| configured | Env var or project-specific Keychain item exists |
| authenticated | Minimal API call succeeds (test separately) |
| usable | Account has billing and permissions |

Missing optional credentials disable the AI feature. The rules-only path continues.

## Security rules

- Never print or log key values.
- `.env` is gitignored.
- Do not recursively scan unrelated CLI configuration.
- Do not execute shell profiles.
- Jev's official SDK reads `TYPESAFE_API_KEY`.
- The check is safe to run anytime — it never modifies anything.
