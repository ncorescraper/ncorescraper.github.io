# Synology Torrent Integration Setup

This guide sets up automatic torrent downloading: click → GitHub → Synology.

## Step 1: Create GitHub Repository

1. Go to [GitHub](https://github.com/new)
2. Create a **private** repository named `torrent-downloads`
3. Initialize with a README
4. Create a `torrents/` folder by:
   - Click "Add file" → "Create new file"
   - Name it `torrents/.gitkeep`
   - Commit

## Step 2: Generate GitHub Personal Access Token

1. Go to https://github.com/settings/tokens/new
2. Create a new token (classic) with name "Synology Torrent Sync"
3. **Check only:** `repo` scope (full control of private repos)
4. **Generate token**
5. **Copy and save** (you won't see it again!)

## Step 3: Configure Web App

1. Open the nCore Film Downloader web app
2. Click ⚙️ **Settings**
3. Paste your GitHub token
4. Enter repo as `username/torrent-downloads`
5. Branch: `main`
6. Click **Save**

Now when you click "Download", torrents will be pushed to your GitHub repo!

## Step 4: Set Up Synology Script

### Option A: Using Task Scheduler (Easiest)

1. SSH into your Synology or use Web Terminal
   ```bash
   sudo -i
   ```

2. Create the script file:
   ```bash
   nano /volume1/homes/admin/torrent-sync.sh
   ```

3. Copy the content from `synology-torrent-sync.sh` in this repo

4. Edit these lines:
   ```bash
   GITHUB_REPO="yourusername/torrent-downloads"  # Your repo
   GITHUB_TOKEN="ghp_your_token_here"             # Your token
   DOWNLOAD_STATION_PATH="/volume1/homes/admin/Downloads"  # Or your Download Station path
   ```

5. Save (Ctrl+X, Y, Enter)

6. Make it executable:
   ```bash
   chmod +x /volume1/homes/admin/torrent-sync.sh
   ```

7. Test it:
   ```bash
   /volume1/homes/admin/torrent-sync.sh
   ```

### Option B: Create a Cron Job (Auto-run every 30 min)

1. Create a new script in `Control Panel` → `Task Scheduler` → `Create` → `Triggered Task`
   - User: `root`
   - Event: `Repeat daily`
   - Schedule: Every 30 minutes

2. Or via terminal:
   ```bash
   crontab -e
   ```

3. Add this line:
   ```
   */30 * * * * /volume1/homes/admin/torrent-sync.sh
   ```

   This runs the script every 30 minutes.

## Step 5: Configure Download Station

1. Open Download Station on your Synology
2. Go to **Settings** → **General**
3. Set "Monitored Folder" to: `/volume1/homes/admin/Downloads` (or your chosen path)
4. Save

Now Download Station will automatically pick up any `.torrent` files placed there.

## Testing

1. Open the web app and click ⚙️ Settings
2. Fill in your GitHub details
3. Find a film and click "⬇️ Download"
4. You should see: `✓ Pushed to GitHub!`
5. Wait ~30 seconds or run the script manually:
   ```bash
   /volume1/homes/admin/torrent-sync.sh
   ```
6. Check Download Station - the torrent should appear!

## Troubleshooting

### "Error: Connection refused"
- Make sure SSH/Web Terminal is enabled on Synology
- Verify GitHub token is correct

### "Failed to push to GitHub"
- Token may have expired - generate a new one
- Check repo name format: `username/repo-name`
- Ensure token has `repo` scope permissions

### Torrents not appearing in Download Station
- Check the monitored folder path is correct
- Verify Download Station "Monitored Folder" is enabled
- Check Synology system logs for errors

### Need to manually test the sync script?
```bash
ssh admin@192.168.1.140  # Connect to Synology
/volume1/homes/admin/torrent-sync.sh  # Run the script
```

## Monitoring

Check the sync log on your Synology:
```bash
cat /volume1/homes/admin/torrent-sync/sync.log
```

## Files Organization

Your GitHub repo structure will look like:
```
torrent-downloads/
├── .gitkeep
├── sync.log              (created by script)
├── torrents/
│   ├── film_name_1.torrent
│   ├── film_name_2.torrent
│   └── ...
└── processed/            (created by script)
    ├── film_name_1.torrent.1234567890
    └── ...
```

The script moves processed torrents to `processed/` folder to avoid re-processing.
