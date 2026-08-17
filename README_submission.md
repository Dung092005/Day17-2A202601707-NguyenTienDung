# Lab 17 Submission — Zep Memory for Agent

## 1. Student implementation

Implemented in `src/memory_student.py`:

- `retrieve_long_term` — Zep Context Block + user graph fact search (`scope="edges"`)
- `retrieve_episodic` — user graph episode search with `episode_char_cap=180`
- `retrieve_semantic` — standalone semantic graph search (`scope="episodes"`, fallback `nodes`)
- `assemble_context` — `ContextBudgetManager.assemble()` with 10/4/3/3 budget

## 2. Architecture

```text
Query → Memory Router → Short-term / Long-term / Episodic / Semantic
      → Context Budget (10/4/3/3) → Merged Context → Evaluator
```

## 3. Evaluation (E01–E11)

| Case | Layer | Result |
|------|-------|--------|
| E01 | short_term | PASS |
| E02 | long_term | PASS |
| E03 | long_term | PASS |
| E04 | episodic | PASS |
| E05 | episodic | PASS |
| E06 | semantic | PASS |
| E07 | mixed | PASS |
| E08 | long_term | PASS |
| E09 | long_term | PASS |
| E10 | short_term | PASS |
| E11 | semantic | PASS |

**Final:** 11/11 PASS, 100% hit rate, avg latency 1082.0 ms, avg token reduction 14.2%

## 4. Benchmark comparison

| System | Hit Rate | Passed | Avg Latency (ms) | Token Reduction |
|--------|---------:|-------:|-----------------:|----------------:|
| Student (Zep memory) | 100.0% | 11/11 | 1285.0* | 14.2% |
| No-memory baseline | 18.2% | 2/11 | 0.0 | 81.8% |

\*First student-report run; final post-reseed run: 1082.0 ms.

No-memory passes only E01/E10 (short-term in current thread). Cross-session, episodic, semantic, and mixed cases fail as expected.

## 5. Mandatory questions (LAB §5.2)

**Layer quan trọng nhất trong bộ test này:** Long-term memory — E02/E03/E08/E09 kiểm tra cross-session preference, open loops, recency và user isolation; không có layer này agent mất durable user facts.

**Trade-off Zep vs Redis+Qdrant:** Zep managed User Graph + Context Block tự extract facts và relevance ranking; Redis/Qdrant baseline (demo) cần tự schema key, vector index và merge logic. Zep tăng recall/latency nhưng giảm ops; local baseline linh hoạt hơn nhưng tốn công maintain.

**Guardrail chống memory poisoning:** `consent.json` opt-in trước ingest; `privacy_guard.minimize_pii` redact email/phone; user-scoped `user_id` mọi retrieval; `forget.py` xóa user-scoped memory và verify absence.

## 6. Benchmark analysis (4 câu)

1. **Layer hit rate thấp nhất (student):** Tất cả layer đạt 100%. Trong no-memory baseline, long_term/episodic/semantic/mixed đều 0% (chỉ short_term 100%).
2. **Case retrieve nhiều token nhất:** E03 (~1333 tokens) — long-term Context Block trả nhiều facts/episodes cho open-loop query.
3. **E07 cần memory nào:** Long-term (`Python` preference) + semantic (`Idempotency-Key` từ KB), ghép qua `assemble_context`.
4. **Token reduction vs hit rate:** No-memory có 81.8% reduction nhưng 18.2% hit rate vì trả rỗng; memory-enabled giảm ~14% token nhưng giữ 100% evidence.

## 7. E08 recency & E10 compaction

**E08:** Stage 3 cập nhật BLUEBIRD-42 → TypeScript/NestJS; ORCHID-27 vẫn Python. Zep facts có validity ranges; query scoped project trả fact mới nhất.

**E10:** Sliding window + compaction giữ `REVIEW-DEADLINE-1600` trong durable notes dù filler turns chiếm recent buffer.

## 8. Privacy / forget

```bash
docker compose run --rm app python -m src.forget --user-id minh-lab17
docker compose run --rm app python -m src.forget --user-id minh-lab17 --verify-only
```

Output: `Zep user absent: True`, `Redis user keys remaining: 0`. Reseeded with `python -m src.seed` before final regression.

## 9. Reproduce commands

```bash
docker compose build
docker compose up -d redis qdrant
make smoke          # or: docker compose run --rm app python -m src.smoke
make seed
make student        # full E01-E11
make baseline       # no-memory
make compare
make test
make forget         # privacy drill
make student-report # benchmark + HTML + run.log
```

## 10. Screenshots needed (manual)

Place in `submission/`:

1. `long_term.png` — E02/E03/E08 PASS from benchmark
2. `episodic.png` — E04/E05 PASS
3. `semantic.png` — E06/E11 PASS
4. `privacy.png` — forget + verify-only output
