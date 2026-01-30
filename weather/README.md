# Weather MCP Server

A simple Model Context Protocol (MCP) server that provides weather data to MCP-compatible clients such as Claude for Desktop.

This server exposes two tools powered by the National Weather Service (NWS) API:

- `get_alerts` – Get active weather alerts for a U.S. state
- `get_forecast` – Get a short-term weather forecast for a given latitude/longitude

---

## Features

- Built with the Python MCP SDK (`mcp.server.fastmcp.FastMCP`)
- Uses `httpx` for async HTTP requests

---

## Prerequisites

- Python **3.10+**
- [`uv`](https://astral.sh) installed and available on your `PATH`
- Claude for Desktop (for testing this server as a client)

> Note: This project requires **Python MCP SDK 1.2.0 or higher** via the `mcp[cli]` extra.

---

## Installation & Setup

### 1. Clone this repository

```bash
git clone https://github.com/Justin-Crawford/mcp-server.git
cd weather-mcp
```

### 2. Create the project and virtual environment

If you haven’t already initialized the project using `uv`, you can follow:

```bash
uv init weather
cd weather

# Create virtual environment
uv venv
source .venv/bin/activate  # On macOS/Linux
# For Windows PowerShell:
# .venv\Scripts\Activate.ps1
```

If you already initialized the project, just activate your virtual environment.

### 3. Install dependencies

```bash
uv add "mcp[cli]" httpx
```

This pulls in the MCP Python SDK and `httpx` for making requests to the NWS API.

---

## Project Structure

A minimal structure looks like:

```text
.
├── weather.py      # MCP server implementation
├── README.md
└── uv.lock / pyproject.toml (created by uv)
```

- `weather.py` contains:
  - Server initialization with `FastMCP("weather")`
  - Helper functions for calling the NWS API
  - `get_alerts` and `get_forecast` tools
  - A `main()` function that runs the MCP server over STDIO

---

## Usage

### Running the MCP server

From the project directory:

```bash
uv run weather.py
```

The server will start and listen for JSON-RPC messages over STDIO from an MCP host (e.g., Claude for Desktop). It does **not** expose an HTTP port.

### Available Tools

#### `get_alerts(state: str) -> str`

Returns active weather alerts for a given U.S. state.

- **Argument**:
  - `state`: Two-letter U.S. state code (e.g. `CA`, `NY`, `TX`)
- **Behavior**:
  - Returns formatted alerts, one or more alerts separated by `---`
  - Returns a message if no alerts are found or if the API fails

#### `get_forecast(latitude: float, longitude: float) -> str`

Returns the short-term forecast for a given location.

- **Arguments**:
  - `latitude`: Latitude of the location (e.g. `38.5816`)
  - `longitude`: Longitude of the location (e.g. `-121.4944`)
- **Behavior**:
  - Uses NWS `/points/{lat},{lon}` to resolve a forecast URL
  - Fetches the forecast and returns the next ~5 forecast periods
  - Includes temperature, wind, and a detailed forecast description

---

## Logging

Because this server uses STDIO for communication:

- **Do not** use `print()` for logging in `weather.py` — it writes to stdout and will corrupt the MCP protocol stream.
- Instead, configure Python’s `logging` module or another logging framework to log to:
  - `stderr`, or
  - a separate logfile

Example pattern:

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.info("Server starting up…")
```

---

## Integrating with Claude for Desktop

To use this MCP server with Claude for Desktop:

1. Make sure Claude for Desktop is installed and up to date.
2. Locate the configuration file:

   ```text
   ~/Library/Application Support/Claude/claude_desktop_config.json
   ```

   Create it if it doesn’t exist.

3. Add or update the `mcpServers` section to include your `weather` server:

   ```json
   {
     "mcpServers": {
       "weather": {
         "command": "uv",
         "args": [
           "--directory",
           "/ABSOLUTE/PATH/TO/PARENT/FOLDER/weather",
           "run",
           "weather.py"
         ]
       }
     }
   }
   ```

   - Replace `/ABSOLUTE/PATH/TO/PARENT/FOLDER/weather` with the absolute path to your project directory.
   - You may need the full path to the `uv` executable (e.g., `/usr/local/bin/uv`), which you can find via `which uv` (macOS/Linux) or `where uv` (Windows).

4. Restart Claude for Desktop.

5. In a Claude chat, click the “Add files, connectors, and more /” plus icon, then hover over **Connectors**. You should see the `weather` MCP server listed.

---

## Example Prompts (in Claude)

Once connected, you can ask Claude things like:

- “What’s the weather in Sacramento?”
- “What are the active weather alerts in Texas?”
- “Give me the forecast for latitude 38.5816, longitude -121.4944.”

Claude will decide when and how to call:

- `get_alerts` for state-level alerts
- `get_forecast` for specific coordinates

---

## Extending This Server

Ideas for extending the functionality:

- Add a tool that:
  - Accepts city names and geocodes them to lat/lon
  - Returns hourly forecasts or radar imagery links
- Expose **resources** (e.g. latest raw JSON data from NWS)
- Add **prompts** that guide the model on how to best use the weather tools (e.g. for travel planning, event planning, etc.)

---

## License

Specify your chosen license here (e.g. MIT, Apache 2.0, etc.).

```text
MIT License

Copyright (c) 2026 Justin Crawford

##ADD LICENSE##
...
```