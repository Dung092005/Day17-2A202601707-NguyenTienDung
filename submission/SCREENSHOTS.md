# Screenshot checklist (manual capture)

Capture from terminal or IDE — do **not** include API keys.

| File | What to capture |
|------|-----------------|
| `long_term.png` | `make student --only-layer long_term` or benchmark table showing E02/E03/E08 PASS |
| `episodic.png` | E04/E05 PASS |
| `semantic.png` | E06/E11 PASS |
| `privacy.png` | Output of `forget --user-id minh-lab17` and `--verify-only` |

Optional evidence:

- `docker compose ps` — redis + qdrant healthy
- `make smoke` — all [OK]
- Full E01–E11 summary table (11/11 PASS)
