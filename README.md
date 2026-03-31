# sous-vide-cooker

A Raspberry Pi–based sous-vide cooking controller. It reads a DS18B20 temperature sensor over GPIO, stores readings in a custom persistent object-storage server (`storserv`), and serves a live web dashboard (via the SnakeCharmer web server) with Highcharts graphs so you can monitor your cook from a browser.

## Architecture

| Component | File(s) | Role |
|---|---|---|
| **Storage server** | `storserv.py`, `storserv_config.py` | Persistent object-storage daemon (replaces a SQL database with a file-system–backed object hierarchy) |
| **Client library** | `pstorage.py` | Python 3 client that makes remote storage objects look like local Python objects |
| **Web server** | `SnakeCharmer.py`, `SnakeCharmer_config.py` | Standalone HTTP/HTTPS server (no Apache required) that serves the dashboard |
| **Sensor reader** | `temp_reg.py` | Polls a 1-Wire DS18B20 temperature sensor every 0.25 s and writes readings to storage |
| **Admin shell** | `stosh.py` | Interactive REPL for navigating and managing the storage tree |
| **Bootstrap / helpers** | `bootstrap.py`, `compile_methods.py`, `fix_do_su.py`, `psnew.py`, `NewUser.py` | Initialization, method compilation, and user-management utilities |

## Building & Running

There is no formal build system. The project is pure Python 3 and was designed to run directly on a Raspberry Pi.

```bash
# 1. Start the storage server
python3 storserv.py

# 2. Bootstrap the storage tree (first run only)
python3 bootstrap.py

# 3. Start the web server (HTTP on port 8000, HTTPS on port 8001)
python3 SnakeCharmer.py

# 4. Start reading temperature (requires RPi.GPIO and a DS18B20 sensor)
python3 temp_reg.py
```

### Running the tests

```bash
python3 test.py        # or python3 new_test.py
```

Tests use Python's `unittest` framework and exercise storage connection, authentication, session management, and access control. A running `storserv` instance is required.

## Features

- Reads temperature from a DS18B20 1-Wire sensor on the Raspberry Pi
- Persistent, hierarchical object storage with user authentication and permissions
- Standalone HTTPS web server with cookie-based sessions
- Live web dashboard with Highcharts temperature graphs
- Interactive admin shell (`stosh`) with `cd`, `ls`, `rm`, `edit`, `make`, `compile`, and `su` commands
- SystemV init scripts for `storserv` and `SnakeCharmer`

## Limitations

- Hardcoded to a single DS18B20 sensor address (`28-000004611932`)
- No `requirements.txt`, `setup.py`, or packaging; dependencies (`RPi.GPIO`, `ssl`, etc.) must be installed manually
- Web server uses a self-signed PEM certificate (`server.pem`) checked into the repo
- No GPIO-based heater control loop — the sensor is read-only; actual sous-vide relay control is not included
- Tests require a live `storserv` instance and a manually set password variable
- Targets Python 3.3 on Raspbian; may need adjustments for newer Python versions

## History

Development occurred entirely on 2013-07-14:

- **2013-07-14** — Initial commit created the repository with an empty README
- **2013-07-14** — Bulk "Initial Commit" added the full stack: persistent storage server (`storserv.py`), client library (`pstorage.py`), SnakeCharmer web server, temperature sensor reader (`temp_reg.py`), admin shell (`stosh.py`), unit tests, bootstrap scripts, init scripts, and a Highcharts-based web dashboard
- **2013-07-14** — Final "update" commit adjusted config paths from `/usr/local/SnakeCharmer/6.0/` to local paths, and added `temp_reg.py` for Raspberry Pi GPIO temperature reading
