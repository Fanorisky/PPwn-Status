# PPwn-Status

Status / update feed for the **PocketPawn** Android app. The app fetches
[`status.json`](status.json) once per session to decide whether the installed
build is outdated and, if so, nags the user to update.

Fetch URL (raw):
```
https://raw.githubusercontent.com/Fanorisky/PPwn-Status/main/status.json
```

## How the app uses it

- The app compares its own `BuildConfig.VERSION_CODE` (which is the **git commit
  count**, so it only ever increases) against `versionCode` in this file.
- If `versionCode` here is **greater** than the installed one, the app shows the
  update dialog on **every launch** (even offline, from a cached copy) until the
  user updates.
- The fetch happens once per app session; if it fails (offline), the app retries
  once when a network becomes available.

## `status.json` fields

| Field | Type | Required | Meaning |
|---|---|---|---|
| `schemaVersion` | int | recommended | Format version of this file. The app only understands schema `1`; if this is **higher** than the app supports, the app ignores the file (forward-compatibility). Defaults to `1` if omitted. |
| `versionCode` | int | **yes** | The latest build's version code = its **git commit count**. Compared against the installed app. Must be monotonically increasing. |
| `versionName` | string | no | Human-readable label shown in the dialog (e.g. `1.3.0`). Cosmetic only. |
| `minSupportedVersionCode` | int | no | Force-update threshold. If installed `versionCode` is **below** this, the update is treated as **mandatory** (non-dismissable) regardless of `mandatory`. Use `0` to disable. |
| `downloadUrl` | string | yes (to update) | Direct URL to the latest APK. Downloaded via DownloadManager, then the installer is launched. A GitHub Releases asset URL works well. |
| `releaseNotes` | string | no | Text shown in the dialog (use `\n` for line breaks). |
| `mandatory` | bool | no | If `true`, the dialog is **non-dismissable** for everyone on an older version. Defaults to `false` (dismissable with "Later", reappears next launch). |

## Example

```json
{
  "schemaVersion": 1,
  "versionCode": 267,
  "versionName": "1.3.0",
  "minSupportedVersionCode": 0,
  "downloadUrl": "https://github.com/ninamuagza/PocketPawn/releases/latest/download/PocketPawn.apk",
  "releaseNotes": "- Custom indent guides\n- Symbol bar\n- Update checker",
  "mandatory": false
}
```

## Releasing an update

1. Build/release the new APK; note its **commit count** (the new `versionCode`).
2. Edit `status.json`: set `versionCode`, update `versionName`, `downloadUrl`,
   `releaseNotes`.
3. Commit & push to `main`. Every older install sees the dialog on next launch.

To force everyone below a build to update (blocking), set
`minSupportedVersionCode` to that build's code (or set `mandatory: true`).

## Notes

- Keep the file small (read on every session).
- Unknown/extra fields are ignored by the app; you can add fields later without
  breaking older builds.
- `versionCode` is the **only** value used for the comparison; never reuse or
  lower it.
