"# Kryten WebUI

Kryten web interface service - provides web dashboard and control panel for CyTube.

## Features

- Real-time chat message monitoring
- User tracking and management
- Event-driven architecture using NATS
- Extensible moderation rules

## Installation

### Prerequisites

- Python 3.10 or higher
- Poetry
- NATS server running
- kryten-py library

### Setup

1. Install dependencies:
```bash
uv sync
```

2. Copy the example configuration:
```bash
cp config.example.json config.json
```

3. Edit `config.json` with your settings:
```json
{
  "nats_url": "nats://localhost:4222",
  "nats_subject_prefix": "cytube",
  "service_name": "kryten-webui"
}
```

## Usage

### Running the Service

Using Poetry:
```bash
uv run kryten-webui --config config.json
```

Using the startup script (PowerShell):
```powershell
.\start-webui.ps1
```

Using the startup script (Bash):
```bash
./start-webui.sh
```

### Command Line Options

- `--config PATH`: Path to configuration file (default: `/etc/kryten/webui/config.json`)
- `--log-level LEVEL`: Set logging level (DEBUG, INFO, WARNING, ERROR)

## Event Handling

The service currently listens for:

- **chatMsg**: Chat messages for dashboard display
- **usercount**: User count updates
- **playlist**: Playlist updates for dashboard

## Development

### Running Tests

```bash
uv run pytest
```

### Linting

```bash
uv run ruff check .
```

### Formatting

```bash
uv run black .
```

## License

MIT License - see LICENSE file for details
" 
