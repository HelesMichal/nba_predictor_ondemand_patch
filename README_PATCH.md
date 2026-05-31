# Patch – on-demand Telegram commands + multi-device

Drop these files into your repo (overwriting existing ones with the same path):

```
nba_predictor/notify/telegram.py        # multi-chat broadcast + retries
nba_predictor/predict.py                # --mode today|tomorrow|week, --chat-id
.github/workflows/daily-predict.yml     # 17:00 CET digest (DST-safe)
.github/workflows/weekly-retrain.yml    # Monday 06:00 UTC, self-healing
.github/workflows/on-demand.yml         # NEW – handles /today etc.
cloudflare-worker/worker.js             # NEW – free always-on Telegram webhook
cloudflare-worker/wrangler.toml         # NEW
DEPLOYMENT.md                           # NEW – full setup, step by step
```

Then follow **DEPLOYMENT.md** end-to-end. It takes ~10 min total and your
laptop never has to be on again afterwards.

## What changed in 1 paragraph

- `TELEGRAM_CHAT_IDS` (comma-separated) replaces single-chat `TELEGRAM_CHAT_ID`
  (the old name still works as a fallback) so the daily digest reaches as
  many devices/users as you want.
- A new Cloudflare Worker receives Telegram messages and, on `/today`,
  `/tomorrow` or `/week`, fires a `repository_dispatch` event that runs the
  new `on-demand.yml` workflow. That workflow predicts and replies to the
  exact chat that asked.
- `predict.py` learned `--mode {today,tomorrow,week}` and `--chat-id`.
- Daily cron runs at both 15:00 and 16:00 UTC so 17:00 CET works year-round
  (CET vs CEST).
- Retrain workflow only uploads the model + notifies on success, and the
  notify step has `continue-on-error: true` so Telegram outages can't fail
  the job.

## Cleanup checklist (optional but recommended)

Remove from the repo if present – none of these are imported anywhere:

- `notebooks/`, `scratch/`, `.ipynb_checkpoints/`
- `Dockerfile*`, `docker-compose*.yml`
- `scripts/run_local.sh`, `scripts/serve.py`
- any `tests/fixtures/*.parquet` (rebuilt from the NBA API)
- `__pycache__/` and `*.pyc` (always)
