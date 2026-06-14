# Features

A detailed tour of what Printer Switcher does. Screenshots are in
[screenshots/](screenshots/).

## Automatic printer switching

- **Active-client detection** in Terminal Server / RDP environments via event-driven registry
  monitoring, with automatic fallback to polling.
- **Local mode** for standalone workstations, keyed on the computer name.
- **Silent background operation.** The service runs with no window and applies the correct
  mappings the instant the active client changes.
- **Registry-backed mapping** of LPT/USB/COM ports to network printers.

## Configuration management

- **Client model.** Each client (workstation, user, or RDP session) holds its own set of
  port-to-printer mappings.
- **Flexible port names.** LPT1, LPT2, LPT3, USB001+, COM1+, or any custom label that fits the
  site.
- **Network, local, and shared printer paths.** For example `\\Server\Printer`, a local name, or
  `\\Computer\SharedPrinter`.
- **Per-client comments** for documentation and handover notes.
- **Default configuration** is created automatically on first run, with sample data, so there's
  no blank slate to fight.
- **Validation** prevents duplicate clients/ports and invalid names.

## File management

It works like professional desktop software:

- **New / Open / Save / Save As** for configuration files.
- **Current-file tracking** shown in the title bar and status bar.
- **Descriptive, timestamped filenames** (`config_YYYY-MM-DD_HH-MM-SS_description.json`).
- **Export as backup** and **Import from backup** for moving configs between machines.

## Backup & restore

- **Automatic backups** on every save, timestamped.
- **Manual backups** with custom descriptions for milestones.
- **Three categories:** automatic, manual, and imported.
- **Backup Manager** dialog with sortable columns (type, date, description, client count, size)
  for restore, import, and cleanup.
- **One-click restore** of any previous configuration, with validation.

## Client & port editing

- **Real-time search** filters the client list as you type.
- **Add / delete / rename / duplicate** clients. Duplicate copies the full mapping set.
- **Inline editing** of fields, with double-click to rename port names.
- **Sortable columns.** Click a header to sort, click again to reverse.
- **Right-click context menus** throughout for fast, discoverable actions.

## Logging & monitoring

- **Centralised log** of lifecycle events, connections, configuration changes, and errors.
- **Built-in Log Viewer** with filtering (All / Connections / Errors / Config Changes), a day
  range selector, and sortable entries.
- **Export** logs for troubleshooting or auditing.
- **Service status indicators** next to each client:
  - 🔴 **Red:** service not running / inactive
  - 🟢 **Green:** active and mappings working
  - 🟡 **Yellow:** running but a mapping failed (check printer/server)
  - ⚪ **Gray:** configured but not currently monitored

## Interface & usability

- **Menu-driven design** across File, Client, Backup, Tools, and Help.
- **Quick Actions toolbar** with Add Client, Add Port, Remove, and Save All.
- **Responsive grid layout** that adapts to window resizing.
- **Comprehensive focus management** so every field accepts a direct click.

### Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+S` | Save all configurations |
| `Ctrl+N` | Add new client |
| `Ctrl+Shift+N` | New client (menu) |
| `Ctrl+O` | Open configuration |
| `F2` | Rename selected client |
| `Ctrl+D` | Duplicate selected client |
| `Delete` | Delete selected client |
| `Alt+F4` | Exit |

## Deployment

- **Standalone executables** built with PyInstaller, with no Python runtime or dependencies on
  target machines.
- **Self-initialising.** It creates its `configurations/`, `backups/`, and `logs/` folders on
  first run.
- **Two binaries:** `config_editor.exe` (GUI) and `printer_switcher.exe` (background service).

## Typical use cases

- **Offices** with different printer sets for reception, management, and meeting rooms.
- **Multi-location businesses** with per-site printer profiles.
- **Shared computers** running per-shift or per-user setups that switch automatically.
- **Terminal Server / RDP** where the correct printers follow each active session.
