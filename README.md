# Alexandria

Docker Compose setup for Jellyfin media streaming and one-way Syncthing backups between two Windows 11 PCs over Tailscale. Uses Docker Desktop with Linux containers.

Run commands from this project's root folder. Keep both PCs connected to Tailscale with MagicDNS enabled.

**Local paths — once per PC**

Edit `.env` in the project root: set `root=D:/Alexandria` on desktop-41310 or `root=C:/Alexandria` on desktop-41700. All volumes use this directory. Compose requires a nonempty `root`.

The paths below assume these values. If moving an existing installation, move its data and settings into the corresponding folders under `root` before starting the containers.

**Jellyfin — desktop-41310**

```powershell
docker compose --env-file .env -f jellyfin/compose.yml up -d
```

Open http://localhost:8096. Media: `D:\Alexandria\Sync\Media`. Settings: `D:\Alexandria\Jellyfin\config`.

**Syncthing — desktop-41310**

```powershell
docker compose --env-file .env -f syncthing/compose.41310.yml up -d
```

**Syncthing — desktop-41700**

```powershell
docker compose --env-file .env -f syncthing/compose.41700.yml up -d
```

Open http://localhost:8384 on each PC. In **Actions → Settings → GUI**, set **GUI Authentication User** and **GUI Authentication Password**, then save.

Access either PC over Tailscale (replace `<tailnet>` with your tailnet name):

- `http://desktop-41310.<tailnet>.ts.net:8384`
- `http://desktop-41700.<tailnet>.ts.net:8384`

Allow incoming TCP 8384 on both PCs in Windows Firewall and any restrictive tailnet rules. This port also listens on LAN interfaces.

For existing installations, set **Settings → General → Device Name** to `desktop-41310` or `desktop-41700`. Fresh installations use the hostname from Compose.

| | desktop-41310 | desktop-41700 |
|---|---|---|
| Files | `D:\Alexandria\Sync` | `C:\Alexandria\Sync` |
| Settings | `D:\Alexandria\Syncthing\config` | `C:\Alexandria\Syncthing\config` |
| Folder mode | Send Only | Receive Only |

**Syncthing setup — once**

1. On both PCs, use **Actions → Show ID**, then **Add Remote Device** to add the other PC's ID.
2. On 41310, set 41700's address to `tcp://desktop-41700.<tailnet>.ts.net:22000`. Leave 41310's address as `dynamic` on 41700.
3. On both PCs, set **Connections → Sync Protocol Listen Addresses** to `tcp://0.0.0.0:22000`. Disable discovery, relaying and NAT traversal. Allow incoming TCP 22000 through Windows Firewall on 41700.
4. On 41310, share `/data` with folder ID `desktop-41310-data`, using **Send Only**. On 41700, accept it at `/data`, using **Receive Only**. Enable **Ignore Permissions** on both; set the full rescan interval to `300` seconds on 41310.
5. On 41700, enable **Staggered File Versioning**, maximum age **90 days**. Source deletions also sync; previous versions are kept in `C:\Alexandria\Sync\.stversions`.

Add a small file to `D:\Alexandria\Sync` and confirm it appears in `C:\Alexandria\Sync` on 41700.

**After reinstalling Windows on 41310**

Keep this project (including `.env`) and the data/settings folders on `D:`. Reinstall Docker Desktop and Tailscale, then run the 41310 commands above. Syncthing keeps its identity and pairing in `D:\Alexandria\Syncthing\config`.

Containers restart when Docker starts. Enable Docker Desktop's start-at-sign-in option on both PCs.
