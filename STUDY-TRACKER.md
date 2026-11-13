# Study Tracker (1 hour/day)

Revision of everything up to **term saturation** runs 6–16 Nov 2026 (re-planned on 13 Nov after the plan slipped). New learning starts 17 Nov 2026 with the rest of Week 5–6 (scoring fundamentals).
The roadmap lives in [README.md](README.md). This file is the daily plan and the log.

---

## How every 1-hour session works

| Minutes | Step | What to do |
|---|---|---|
| 0–5 | Recall | Without looking at notes, answer yesterday's 3 questions out loud or on paper. Mark each right or wrong in the log. |
| 5–20 | Learn | Read the one topic of the day (your notes for revision days, the official docs for new topics). Stop at 15 minutes even if unfinished. |
| 20–50 | Hands-on | Kibana Dev Tools. **Predict first** (hits, order, rough score), then run, then explain any difference. Use `_analyze`, `_explain`, `_profile`. |
| 50–60 | Write and log | Save the practical as a `.txt` file in your existing style. Write 3 questions for tomorrow's recall. Fill in today's log row. |

Rules:
- Hands-on is at least half the hour. Reading alone doesn't count as done.
- If a prediction was wrong, that is the most valuable minute of the day. Write down why.
- Never start a new topic until yesterday's questions have been answered.
- If you miss a day, push every later day back by one day. Don't double up two topics in one hour.

## Done means

A topic is done only when all of these are true:
1. You wrote a practical file with queries you ran yourself.
2. You predicted at least one result before running it.
3. You can explain it in 3 sentences without notes.
4. Its README checkbox is ticked.

## Weekly review (last day of each 7-day block, replaces the hands-on step)

1. Count days done out of 7.
2. Tick the README items that meet "Done means".
3. Re-answer 5 random questions from earlier days. Anything missed goes back on the plan for next week.
4. Send me the log. I'll adjust the next block.

---

## Block 1: revision (6–16 Nov 2026, re-planned 13 Nov)

Method: close your notes and write the answer from memory first, then open the notes to check. Re-run the practical queries cold.

| Day | Date | Topic | Notes to use | Hands-on |
|---|---|---|---|---|
| R1 | Fri 6 Nov | Document model and mapping: text vs keyword, arrays vs nested, multi-fields, dynamic mapping risks | note 1–4 | Redo the task mappings from note 4 from memory. Spot the errors yourself, since we found 3 in note 4. |
| R2 | Sat 7 Nov | Analyzers, tokenizers, filters, index vs query time | advanced/Tokenizers_*, TokenFilters_*, Analyzers_Practical, Index_Vs_Query_Analyzers | `_analyze` on emails, IDs, hyphenated words. Predict the tokens first. |
| R3 | Sun 8 Nov | Multi-fields, bool query, query vs filter context, filter caching | Multi-Fields*, Bool, Filter_Caching_And_Perf | Write a bool query with must, filter, should and must_not from memory. |
| R4 | Fri 13 Nov | Nested: why, false positives, filter pitfalls | why_nested, nested_facts, nested_filter_pitfalls | Reproduce the object-vs-nested false positive. |
| R5 | Sat 14 Nov | Nested scoring, sorting, inner_hits | nested_scoring*, nested_sort, sort*, multiple_nested_practical | Predict `score_mode` effects, then verify. |
| R6 | Sun 15 Nov | TF and IDF | term_frequency, idf | Hand-compute IDF for a 5-doc index, then compare with `_explain`. |
| R7 | Mon 16 Nov | BM25, field length normalization, term saturation. **Weekly review.** | BM25, BM25_calculations | Redo one score from the BM25 challenge by hand. |

---

## Block 2: Week 5–6 remaining, scoring fundamentals (17–26 Nov 2026)

Save practicals in `Phase 2/Week 5 and 6/` using the same naming style as `BM25.txt`.

| Day | Date | Topic | Key question to answer | Hands-on |
|---|---|---|---|---|
| S1 | Tue 17 Nov | Phrase vs term scoring | Why does `match_phrase` score differently from `match`? What are positions and slop? | Same 5 docs, compare `match`, `match_phrase`, and slop 1 and 2. |
| S2 | Wed 18 Nov | Why exact matches rank higher, why long docs rank lower | How do norms and length normalization produce this? | Same term in a 3-word and a 60-word field. Read `_explain`. Try `b` set to 0 vs 0.75. |
| S3 | Thu 19 Nov | How bool queries affect scoring | Which clauses add to score and which don't? What does `minimum_should_match` change? | Predict the sum of two `should` clauses, then check. Move one clause to `filter` and compare. Optional 10 min: repeat the `cache_lab` must vs filter cache-stats test (see the "still to verify" list in `Filter_Caching_And_Perf.txt`). |
| S4 | Fri 20 Nov | How filters affect scoring; `match_all` scoring; score = 0 scenarios | Why is the `match_all` score 1.0? When can a hit have score 0? | `match_all`, `constant_score`, filter-only bool, `boost` of 0. |
| S5 | Sat 21 Nov | Shard-level scoring differences | Why can the same doc score differently with 3 shards than with 1? | Small index with 3 shards, then `search_type=dfs_query_then_fetch` and compare. |
| S6 | Sun 22 Nov | Explain API deep dive | Can you rebuild the number from the tree by hand? | `_explain` on 2 docs, reproduce both scores in a spreadsheet or on paper. |
| S7 | Mon 23 Nov | Profile API. **Weekly review.** | What does profile tell you about cost, and what does it not tell you about scoring? | `profile: true` on a bool query. Find the slowest clause. |
| S8 | Tue 24 Nov | Dark corners 1: identical scores, ranking instability | Why do ties reorder between requests? | Index docs with identical text, run the same query repeatedly, test with replicas. |
| S9 | Wed 25 Nov | Dark corners 2: pagination score drift, why results reorder | How do you make paging stable? | Page through ties with `from/size`, then add a tiebreaker sort, then `preference`. |
| S10 | Thu 26 Nov | Challenge | Can you predict a full ranking before running it? | 6 docs, 3 queries. Write predicted order and scores first. Verify and tick README items. |

After S10, Week 7–8 (multi-field ranking and boosting) starts on 27 Nov. I'll write that block when we get there.

---

## Daily log

Status: `✅` done, `½` partial, `❌` skipped. Recall is right answers out of 3.

| Day | Date | Status | Hands-on done? | Recall (x/3) | Prediction right? | What surprised me / open questions |
|---|---|---|---|---|---|---|
| R1 | 6 Nov | ✅ | Yes: practical file written, queries run | n/a | Yes | Meets all "Done means" checks (README Week 1 already ticked) |
| R2 | 7 Nov | ✅ (finished 9 Nov, synonym extension 10 Nov) | Tokenizers and analyzers practicals done, incl. log_store task; synonym vs synonym_graph practical added to TokenFilters_Practical.txt | | Yes | Pattern tokenizer `+` (split vs extract mode), char_filter order, stemmer turns happy into happi, tokenizer must keep accented letters (`\p{L}`) before `asciifolding` can fold them. 10 Nov: query-time synonyms via `search_analyzer`; flat `synonym` vs `synonym_graph` differ only by `positionLength`, which tells ES that `ny` covers the same span as `new york`; `operator: and` is needed to expose the flat problem (default `or` hides it); a query on a missing field returns 0 hits and no error |
| R3 | 8 Nov | ✅ (finished 13 Nov) | Filter caching run in Kibana (`cache_lab`), practical added to `Filter_Caching_And_Perf.txt` | | Partly: expected caching after ~2 uses, it was 5 | Filter cache = node query cache. Cached only after 5 uses. `term` filters never cached. Stats count clause lookups, not searches. A refresh adding a segment doesn't invalidate old entries |
| R4 | 13 Nov | | | | | |
| R5 | 14 Nov | | | | | |
| R6 | 15 Nov | | | | | |
| R7 | 16 Nov | | | | | |
| S1 | 17 Nov | | | | | |
| S2 | 18 Nov | | | | | |
| S3 | 19 Nov | | | | | |
| S4 | 20 Nov | | | | | |
| S5 | 21 Nov | | | | | |
| S6 | 22 Nov | | | | | |
| S7 | 23 Nov | | | | | |
| S8 | 24 Nov | | | | | |
| S9 | 25 Nov | | | | | |
| S10 | 26 Nov | | | | | |

## Question bank

Add the 3 questions you write each day here, with the answer you gave. Re-ask them 1 day and 7 days later.

| Day | Question | My answer | Right? |
|---|---|---|---|
| R2 | In what order do char filters, the tokenizer and token filters run? | | |
| R2 | With the `pattern` tokenizer, what does `group: 0` change, and when does `+` alter the tokens? | | |
| R2 | Why did `happy` become `happi`, and why does a search for `happy` still match? | | |
| R2 (10 Nov) | What is the only difference between the flat `synonym` and `synonym_graph` output for `ny`, and what does it tell Elasticsearch? | | |
| R2 (10 Nov) | Why do you need `operator: and` (or `match_phrase`) to see flat synonyms break? | | |
| R2 (10 Nov) | Which synonym filter goes at search time, and which at index time? Why? | | |
| R3 (13 Nov) | After how many uses does a filter clause get cached, and which kind of filter is never cached? | After 5 uses. `term` filter is never cached. | ✅ |
| R3 (13 Nov) | Why is `now-5m` a poor cache candidate? | `now` changes per request, so the cache key never repeats. | ✅ |
| R3 (13 Nov) | Does a refresh that adds a segment invalidate older segments' entries, and which segments never get cached? | No, the old segment stayed. Segments under 3% are not cached. | ½ (a segment needs >= 10k docs AND >= 3% of the whole shard, not 3% of the previous segment) |
