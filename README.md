# Live Sync Companion

Sync your live WordPress site with [Local](https://localwp.com) — pull it into Local for development and push changes back over the REST API. No SSH or FTP needed.

Live Sync Companion is the **server-side half** of the WP Live Sync add-on for Local. Install this plugin on your live WordPress site, connect from the Local add-on with an Application Password, and you can:

- **Pull** your entire live site (database + files) into a Local site
- **Create a brand-new Local site** directly from your live site in one click
- **Push** your local changes (database + files) back to the live server

## Built for real-world hosting

- **Stepped exports** — database dumps and file archiving run in short resumable steps (~15 s each), so exports never trip proxy timeouts (e.g. Cloudflare's 100-second limit)
- **Multi-part archives** — files are zipped into ~100 MB parts instead of one giant archive
- **Chunked uploads** — large database imports arrive in 8 MB chunks, fitting even restrictive `upload_max_filesize` limits
- **Memory-safe streaming** — downloads stream in 1 MB chunks; nothing is ever buffered whole in memory

## Requirements

- WordPress 5.6+ (Application Passwords built in)
- PHP 7.4+ with the ZipArchive extension
- The WP Live Sync add-on installed in Local on your computer

## Installation

1. Download this repository (or the latest [release](https://github.com/rambozindia/live-sync-companion/releases)) and upload the `live-sync-companion` folder to `/wp-content/plugins/`
2. Activate **Live Sync Companion** in WP Admin → Plugins
3. Create an Application Password: **Users → Profile → Application Passwords** → name it (e.g. "Local Sync") → **Add New** → copy the password
4. In Local, open the **WP Live Sync** panel, enter your site URL, username, and the Application Password, then connect

## REST API

All endpoints live under `wp-json/wp-sync/v1` and require an **administrator** authenticated via Application Passwords.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/status` | Health check, plugin/WP/PHP versions |
| GET | `/site-info` | WP version, theme, plugins, disk usage |
| POST | `/export/database` | DB export; with `stepped=1` runs one ~15 s slice per request — pass back `token` until `complete: true` |
| POST | `/export/files` | File archive export; same stepped protocol, produces ~100 MB ZIP parts |
| GET | `/download/{token}` | Stream an exported file (`?part=N` for multi-part exports) |
| POST | `/import/database` | Upload & import a SQL file (small files) |
| POST | `/import/database/chunk` | Upload one 8 MB chunk of a large SQL file; imports once the last chunk arrives |
| POST | `/import/files` | Upload & extract a ZIP archive |
| DELETE | `/cleanup/{token}` | Remove temporary export files |
| GET | `/log` | Tail of the plugin's log (for remote debugging) |

## Security

- Every endpoint requires `manage_options` (administrator) via Application Passwords
- Export files use cryptographically random filenames inside a protected directory (`.htaccess` deny + `index.php` guard); the log filename is derived from the site's auth salt
- `wp-config.php` is never included in exports and never overwritten by imports
- Uploaded archives are validated against path traversal before extraction
- Temporary files are removed after each sync, on deactivation, and on uninstall

**Always use HTTPS** — Application Passwords over plain HTTP expose credentials.

## Changelog

See [readme.txt](readme.txt) for the full changelog.

## License

[GPLv2 or later](LICENSE)

---

Made by [Ramkumar R](https://24gb.uk) · [BlueBurn Technologies](https://blueburn.in)
