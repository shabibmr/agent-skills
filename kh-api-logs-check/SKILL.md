---
name: kh-api-logs-check
description: Read and analyze the live kh-api PM2 logs at `/root/.pm2/logs/kh-api-out.log` (stdout) and `/root/.pm2/logs/kh-api-error.log` (stderr). Use when the user asks to check backend logs, kh-api logs, PM2 logs, whether the API logged errors, or runs /kh-api-logs-check. Scan those live files for errors, report each finding with surrounding context, and reply "all good" when none are found.
---

# kh-api live log check

Check the live kh-api process logs and answer whether errors were logged.

## Source of truth

PM2 process `kh-api` (id 5) runs `/var/www/html/algo_cloud/kh_api/start.sh` → `node dist/main.js`. Nest Logger writes to stdout/stderr; PM2 captures it. There is no in-app log file.

Live files (default, always read these):

- stdout: `/root/.pm2/logs/kh-api-out.log`
- stderr: `/root/.pm2/logs/kh-api-error.log`

An empty current file means no new lines since the last `pm2-logrotate` cut (midnight). Treat that as no live content in that stream.

Read with:

```bash
tail -n 400 /root/.pm2/logs/kh-api-error.log
tail -n 400 /root/.pm2/logs/kh-api-out.log
```

If a live file is huge, take the last 400 lines of each. Confirm process health with `pm2 show kh-api` only when the files are missing or the process looks down.

Rotated copies (`/root/.pm2/logs/kh-api-{out,error}__YYYY-MM-DD_00-00-00.log`) and Apache `/var/log/apache2/` only when the user asks for history or HTTP proxy context (`https://algoray.cloud/kh_api`).

## Analyze

1. Scan stderr first, then stdout.
2. Treat as an error: Nest `ERROR` lines, uncaught exceptions, stack traces, `Error:` / `Exception`, 5xx handler dumps, Prisma/OCI/S3 failures, process crashes, restart loops.
3. Skip info/debug noise. Include `WARN` only when it sits next to an error or repeats enough to be a failure mode.
4. Deduplicate identical messages; keep the newest timestamp and a repeat count.
5. For each distinct error, capture: log file path, timestamp, the error line, 2–5 surrounding lines, and a one-line meaning.

## Respond

If any error remains after the scan, list findings (newest first) with that context.

If the live files have no error matches, reply exactly:

```
all good
```

One short line of coverage is allowed after that (files read, line window, process uptime) only if the user asked for a check with extra detail. For a bare check, stop at `all good`.
