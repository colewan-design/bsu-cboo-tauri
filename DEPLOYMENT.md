# BSU CBOO Tauri App — Deployment & Update Guide

## How the Auto-Updater Works

When a client opens the installed app, it checks this endpoint for a newer version:
```
https://github.com/colewan-design/bsu-cboo-tauri/releases/latest/download/latest.json
```
If the version there is higher than what's installed, it prompts the user to update automatically.

The GitHub Actions workflow (`.github/workflows/release.yml`) handles the build and release — it is triggered only when a version tag (e.g. `v1.0.2`) is pushed.

---

## Updating the Server IP Address

**1. Edit `src-tauri/tauri.conf.json`**

Change the `url` field to the new IP:
```json
"url": "http://<new-ip>/login"
```

**2. Bump the version** in the same file (must be higher than the currently installed version):
```json
"version": "1.0.2"
```

**3. Commit and push:**
```powershell
git add src-tauri/tauri.conf.json
git commit -m "update server IP"
git push
```

**4. Tag and push to trigger the build:**
```powershell
git tag v1.0.2
git push origin v1.0.2
```

**5. Verify on GitHub:**
- Go to `https://github.com/colewan-design/bsu-cboo-tauri/actions`
- Confirm the workflow run is green
- Go to `https://github.com/colewan-design/bsu-cboo-tauri/releases` and confirm `latest.json` is listed as a release asset

Client apps will auto-update on next launch.

---

## Any Future Update (General Rule)

Same steps apply for any change to the app:

1. Make your changes
2. Bump `version` in `src-tauri/tauri.conf.json`
3. `git add`, `git commit`, `git push`
4. `git tag v<new-version>` then `git push origin v<new-version>`
5. Wait for Actions to go green

---

## Troubleshooting

### Actions page shows "Get started with GitHub Actions" (no workflows)
The `.github/` folder was never committed. Fix:
```powershell
git add .github/
git commit -m "add release workflow"
git push
```
Then delete and re-push the tag (see below).

### Tag was pushed before the workflow was in the repo
Delete the tag and re-push it after committing the workflow:
```powershell
git push origin :refs/tags/v1.0.x   # delete remote tag
git tag -d v1.0.x                   # delete local tag
git tag v1.0.x                      # re-create
git push origin v1.0.x              # push again — now triggers Actions
```

### Clients not updating after a successful release
- Confirm the version in `tauri.conf.json` is **higher** than what's installed on client machines
- Open `https://github.com/colewan-design/bsu-cboo-tauri/releases/latest/download/latest.json` in a browser — it should return JSON with the new version number
