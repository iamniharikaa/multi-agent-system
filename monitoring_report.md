# Monitoring Report: ResearchMind Multi-Agent Research System

**Report date:** 26 November 2026
**Monitoring method:** Log-based review of six pipeline runs (trace_1.txt to trace_6.txt) and their summary file, traces.csv. No automated monitoring tool was active for these runs.
**Scope:** Terminal runs of the pipeline using openai/gpt-oss-120b on Groq's free tier.

---

## 1. Summary

| Metric | Result |
|---|---|
| Runs analysed | 6 |
| Search step completed | 4 of 6 (67%) |
| Reader step scraped a page | 0 of 6 (0%) |
| Writer produced a report | 3 of 6 (50%) |
| Complete reports (all required sections) | 2 of 6 (33%) |
| Critic ran | 3 of 6 (50%) |
| Average critic score (3 runs) | 7.3 / 10 (scores: 7, 8, 7) |
| Runs stopped by API limits | 3 of 6 (50%) |

**Key finding:** The reader step scraped no pages. The pipeline passes only the first 800 characters of the search output to the reader (`[:800]` in `pipeline.py` and `app.py`). The URLs appear later in that output, so the reader asked for a link instead of scraping. All three reports were written from search results alone.

---

## 2. Run-by-run log

| Run | Topic | Search | Reader | Writer | Critic | Outcome |
|---|---|---|---|---|---|---|
| 1 | Recent advances in solid-state batteries | Completed | Asked for URL | Report cut off before conclusion and sources | 7/10 | Partial report |
| 2 | CRISPR gene editing | Completed | Asked for URL | Complete | 8/10 | Complete report, no scraping |
| 3 | Fusion energy progress | Stopped: single request of 8,101 tokens exceeded 8,000 TPM limit (HTTP 413) | Not reached | Not reached | Not run | Failed |
| 4 | LLM agents 2025 | Completed | Asked for URL | Complete (sources list cut off at item 8) | 7/10 | Complete report, no scraping |
| 5 | Zorblax quantum fabric standards | Stopped: daily token limit (HTTP 429) | Not reached | Not reached | Not run | Failed |
| 6 | tips | Completed | Stopped: daily token limit (HTTP 429) | Not reached | Not run | Failed |

*Runs 3 and 5 are matched to their topics by loop order, because their trace files don't show the topic. Confirm these before submission.*

---

## 3. API limits observed

The free tier for openai/gpt-oss-120b on Groq sets a per-minute token limit (TPM) and a daily token limit (TPD). The table shows the values from the error messages.

| Run | Limit type | Limit | Usage at time of error | Request size | Retry wait given |
|---|---|---|---|---|---|
| 3 | Per-minute (TPM) | 8,000 | n/a | 8,101 tokens | Not given |
| 5 | Daily (TPD) | 200,000 | 194,654 | 6,311 tokens | About 7 minutes |
| 6 | Daily (TPD) | 200,000 | 198,421 | 2,036 tokens | About 3 minutes |

The usage figures are the total daily usage for the API key, including earlier test runs, not only these six. By runs 5 and 6, nearly the whole daily allowance was already used. Each full run uses a large share of that allowance, so the daily limit is the main constraint on repeated testing.

---

## 4. Incident history before this trace session

| Date | Environment | Error | Cause | Resolution |
|---|---|---|---|---|
| 26 Nov 2026 | Streamlit Cloud | OpenAI RateLimitError | OpenAI account had no available quota | Switched model provider to Groq |
| 26 Nov 2026 | Streamlit Cloud | Groq NotFoundError (HTTP 404) | Model `llama-3.3-70b-versatile` not available | Switched model to `openai/gpt-oss-120b` |
| 26 Nov 2026 | Local | StreamlitSecretNotFoundError | `st.secrets` read without a `secrets.toml` file | Wrapped the read in try/except |
| 26 Nov 2026 | Local (Windows) | UnicodeEncodeError | Console encoding (cp1252) could not print a special hyphen character | Set `PYTHONUTF8=1` |

---

## 5. Open issues

| # | Issue | Severity | Status |
|---|---|---|---|
| 1 | Reader never receives URLs because of the `[:800]` truncation in `pipeline.py` and `app.py` | High | Fix identified, not yet verified by a re-run |
| 2 | Daily token allowance (200,000) is used up after a few full runs | High for repeat testing | Open |
| 3 | A single search-agent request reached 8,101 tokens, above the 8,000 TPM limit | Medium | Open |
| 4 | Solid-state battery report cut off before its conclusion and sources; cause not confirmed | Medium | Open |
| 5 | Cited sources have not been verified against the pages they reference | Medium | Open |

---

## 6. Recommended alerts

These thresholds are proposals. They have not been tested against measured data.

| Signal | Threshold | Action |
|---|---|---|
| Scrape success rate | Below 80% over the last 10 runs | Check the reader step and how URLs are passed to it |
| Daily tokens used | Above 80% of 200,000 | Pause new runs until the daily limit resets |
| HTTP 413 or 429 error | Any occurrence | Log it, wait for the retry time in the error message, and reduce topic size or search output |
| Critic score | Below 6/10 | Review the report before using it |
| Incomplete report | Any occurrence | Check output length and whether sections are missing |

---

## 7. Not measured

- **Latency per step:** The logs have no timestamps.
- **Tokens per run:** Only error messages show token totals.
- **Cost:** Not tracked. The app runs on free tiers.
- **Live app traffic:** The Streamlit Cloud dashboard and its logs were not reviewed for this report.

---

## 8. Next steps

1. Apply the `[:800]` fix to `pipeline.py` and `app.py`, then re-run three topics and confirm the reader scrapes a page.
2. Turn on LangSmith tracing to record latency and token use for each step.
3. Add timestamps to the terminal output so future logs can measure latency.
4. Review the Streamlit Cloud logs after each deployment and record any errors in the incident history.
