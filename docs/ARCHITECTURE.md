# Architecture

Printer Switcher is a two-application Windows system sharing a single, UI-independent core.
This document describes the high-level design. It leaves out implementation source, since the
code is proprietary.

## System overview

```
┌────────────────────────┐        ┌────────────────────────────┐
│   Config Editor (GUI)  │        │  Printer Switcher (service)│
│   - edit configs       │        │  - detect active client    │
│   - backups & logs     │        │  - apply printer mappings  │
└───────────┬────────────┘        └───────────────┬────────────┘
            │      both depend on the shared core  │
            └──────────────────┬───────────────────┘
                               ▼
                ┌──────────────────────────────┐
                │            core/             │
                │  configuration · printers ·  │
                │  registry monitor · backups ·│
                │  validation · logging         │
                └──────────────────────────────┘
                               │
                               ▼
              JSON configuration & backup files on disk
```

Both applications are built from the same `core/` services. The editor adds a Tkinter UI layer
on top, while the service runs headless. Keeping business logic free of UI dependencies means
the switching logic can run with no window, and the same services can be unit-tested in
isolation.

## Layered layout

```
main/                      Application entry points
  printer_switcher.py      Background service entry point
  config_editor.py         GUI application entry point

core/                      Business logic & data models (no UI)
  models.py                Configuration manager, app state, backup metadata, directory layout
  services.py              Configuration, printer mapping, registry, backup, validation,
                           monitoring, log-parsing, import-validation services
  constants.py             Constants, defaults, file operations, custom exceptions
  logging_system.py        Centralised logging

ui/                        Tkinter interface
  main_window.py           Window coordinator (grid layout, component wiring, file state)
  dialogs.py               Backup Manager, Log Viewer, Import dialogs
  styles.py                Theming / styling
  components/
    client_panel.py        Client list + real-time search
    mapping_panel.py       Port mapping grid + inline editing
    menu_bar.py            File / Client / Backup / Tools / Help menus
    status_bar.py          Real-time status display
```

## Core services

| Service | Responsibility |
|---|---|
| **ConfigurationService** | Load/save configuration files; current-file tracking, separate from backups. |
| **PrinterService** | Printer mapping and LPT/port management operations. |
| **RegistryService** | Windows registry access for client detection in Terminal Server / RDP. |
| **RegistryMonitorService** | Event-driven registry change notification (see below). |
| **BackupService** | Create, restore, and manage backups with metadata. |
| **ValidationService** | Input validation and system-state checks. |
| **SystemMonitoringService** | Process / service status detection. |
| **LogParsingService** | Parse and analyse log files for the viewer. |
| **ImportValidationService** | Validate imported configuration/backup formats. |

## Registry monitoring

The background service needs to know which client is active right now. In Terminal Server / RDP
environments, that's reflected in the Windows registry.

Rather than poll on a timer, the service uses an event-driven design:

- **`RegNotifyChangeKeyValue`** (Win32, via `ctypes`) signals the moment a watched key changes.
- A **dedicated background thread** waits on that notification and communicates with the main
  service thread using `threading.Lock` / `threading.Event` for safe coordination.
- A **callback** fires immediately on change, so the correct mappings are applied with typical
  latency under roughly 100 ms.
- **Graceful fallback.** If the API is unavailable or fails, the service reverts automatically to
  a 5-second polling loop, so it keeps working across all Windows environments.
- **Clean lifecycle.** Windows handles are released and the monitor thread is shut down cleanly
  on service termination.

The benefit over polling is near-instant response with no continuous CPU or registry overhead,
and zero work when nothing changes.

### Operating modes

- **Client mode** runs in Terminal Server / RDP setups. It tracks the active client via registry
  notifications, with polling as a fallback.
- **Local mode** runs on standalone workstations. It uses the computer name as the client
  identifier, so no registry monitoring is required.

## Data model

The system separates **active configuration files** from **backup files**.

A **configuration file** maps clients to port-to-printer mappings:

```json
{
  "Reception Desk": {
    "lpt1": "\\\\PrintServer\\HP_LaserJet",
    "usb001": "\\\\Reception\\Scanner_Printer",
    "comment": "Front desk setup"
  },
  "Manager Office": {
    "lpt1": "\\\\PrintServer\\Color_Printer",
    "comment": "High-quality output"
  }
}
```

A **backup file** holds the configuration plus metadata for the Backup Manager:

```json
{
  "metadata": {
    "backup_id": "2026-06-14_14-30-15",
    "backup_type": "manual",
    "created_date": "2026-06-14_14-30-15",
    "description": "Pre-migration backup",
    "client_count": 10,
    "version": "1.0"
  },
  "configuration": { "...": "..." }
}
```

## On-disk layout

The applications create and manage their own folder structure on first run:

```
configurations/                 Active configuration files
  config_YYYY-MM-DD_HH-MM-SS_[description].json
backups/
  automatic/                    Created on every save
  manual/                       User-created, with descriptions
  imports/                      Imported backups (incl. legacy migration)
logs/
  printer_switcher.log          Centralised application log
```

## Design principles

- **Separation of concerns.** UI, business logic, and OS integration stay in distinct layers.
- **Single responsibility.** Each module or service owns one well-defined area.
- **Reliability first.** Every OS-level operation has error handling and a fallback path.
- **Auditability.** All significant actions are logged with context.
