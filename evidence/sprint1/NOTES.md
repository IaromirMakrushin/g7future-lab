# Sprint 1 evidence notes

- **`filsrv01-static-ip-set.png` shows the original plan, not the current addressing.** It was taken before the re-IP on 10 Nov 2026 and shows `10.7.10.20/24` with gateway `10.7.10.1`. FILSRV01 now uses `10.7.1.3/24` with gateway `10.7.1.1` (see `README.md`). The screenshot is kept as history.
- Other screenshots taken before the re-IP may also show old addresses. Only `filsrv01-static-ip-set.png` and `dc01-dns-set.png` were checked (the latter shows only DNS = `127.0.0.1`, no subnet). The rest are **not re-checked**.
- No screenshot yet shows the final `10.7.1.x` configuration of all four VMs. Planned: `ipconfig /all` on DC01, FILSRV01, CA01 and CLIENT01, saved here with the date. Until then the current addressing is **not verified** by evidence.
