Before launch (when you make repos private again):
Rotate all secrets (GitHub tokens, Stripe keys, API keys) — they're permanently in git history
Use git filter-repo or BFG Repo Cleaner to remove secrets from history if needed
Update any hardcoded references to old secret values

---
