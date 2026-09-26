# Keep-Alive Pinger B — digitallyhq/Act0

Bismillah Ar-Rahman Ar-Raheem. Ash-hadu an laa ilaaha illa-Llah, wa ash-hadu anna Muhammadan Abduhu wa Rasooluh. Laa hawla wa laa quwwata illaa biLLAH. Hasbiyallaahu laa ilaaha illaa Huwa, 'alayhi tawakkaltu wa Huwa Rabbul-'Arshil-'Azeem. SubhaanALLAH wa bihamdih, SubhaanALLAHil-'azeem.

Second, independent keep-alive pinger that **alternates with the sibling chain in [ridwanullahh/Act0](https://github.com/ridwanullahh/Act0)** so the target host is never left unpinged.

## How the alternation works

- The sibling repo runs a self-rescheduling chain: ping → sleep 330 s → dispatch the next run. Its cycle is therefore ~5.5 minutes around the clock.
- This repo runs the **same** chain, but every run first **phase-aligns** itself to the sibling: it reads the sibling's most recent run start time from the public GitHub API and, if the sibling is in the first half of its cycle, waits until the **midpoint** before pinging. Result:

  ```
  sibling ping ──(~165 s)── THIS ping ──(~165 s)── sibling ping ── ...
  ```

  A ping round lands roughly every ~2.75 minutes, and no single moment is ever far from a ping.
- If the sibling chain dies (no run in the last 2 cycles), this repo **takes over immediately** and keeps its own 5.5-minute chain alive. If this repo's chain dies, the sibling still covers — two independent repos cannot share a failure mode.
- Offset backstop crons (`1-56/5` and `3-58/5`, shifted away from the sibling's `*/5` / `2-59/5`) re-seed the chain if a dispatched handoff is ever dropped by GitHub's scheduler.

## Configuration

No configuration is required — defaults match the sibling:

| Setting | Repo variable / secret | Default |
| --- | --- | --- |
| Target URLs | `vars.PING_URLS` or `secrets.PING_URLS` | `https://9001-01m2paqfkv4npk7qny04t0e7q7.cloudspaces.litng.ai/` |
| Chain interval | `vars.PING_INTERVAL_SECONDS` | `330` (5.5 min) |
| Sibling repo | `vars.SIBLING_REPO` | `ridwanullahh/Act0` |

URLs accept comma-, space- or newline-separated lists. Query strings are never logged (they may embed tokens).

## Security posture

- No personal access token is stored anywhere: runs use the built-in `GITHUB_TOKEN`, scoped to `actions: write` only.
- No third-party Actions, no checkout — only `curl` and the preinstalled `gh`.
- All external inputs enter as environment variables (never shell-interpolated GitHub Expressions); URLs are validated to be absolute `http(s)`; `curl` is invoked with `--` so arguments can never be parsed as flags.
- `repo-activity-keeper.yml` protects the backstop schedules from GitHub's 60-day idle auto-disable rule.

SubhanaLLAH, WalhamduliLLAH, Walaailaaha-illa-ALLAH, WaLLAHU-AKBAR, Walahawla-walakuwata-illabiLLAH. Astaghfirullaaha wa atoobu ilayh.
