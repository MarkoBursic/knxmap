# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KNXmap is a Python 3.4+ security tool for scanning, auditing, and interacting with KNXnet/IP gateways on IP-driven networks. It enables discovery of KNX gateways, bus device scanning, message monitoring, group address writing, and authentication key bruteforcing.

## Commands

### Installation
```bash
sudo python setup.py install
# Or run directly without installing:
python main.py [command] [options]
```

### Running the Tool
```bash
knxmap scan <gateway> [bus_targets] [options]   # Scan gateways and bus devices
knxmap search -i <interface>                     # Multicast gateway discovery (requires root)
knxmap write <gateway> <group_addr> <value>      # Write to group address
knxmap apci <gateway> <device> <apci_type>       # Execute APCI functions
knxmap brute <gateway> <device>                  # Bruteforce authentication keys
knxmap monitor <gateway>                         # Monitor KNX bus messages
```

### Testing
Tests require a real KNX gateway:
```bash
python knxmap/tests/test_basic.py <gateway_ip>
```

### Debugging
```bash
PYTHONASYNCIODEBUG=1 knxmap -v scan 192.168.178.20 1.1.0-1.1.6 --bus-info
```

## Architecture

### Core Components

- **`knxmap/core.py`** - Main `KnxMap` orchestrator class. Manages asyncio workers and queues for concurrent scanning. Key methods: `scan()`, `search()`, `monitor()`, `brute()`, `apci()`, `group_writer()`.

- **`knxmap/bus/tunnel.py`** - `KnxTunnelConnection` implements asyncio.DatagramProtocol for tunnel connections with gateways. Handles TunnellingRequests, sequence counting, timeouts, TPCI connections, and APCI functions.

- **`knxmap/gateway.py`** - `KnxGatewaySearch` and `KnxGatewayDescription` protocols for multicast discovery and gateway identification.

- **`knxmap/messages/`** - Protocol message encoding/decoding across layers (APCI, TPCI, CEMI, tunnelling). Changes here affect multiple protocol layers.

- **`knxmap/targets.py`** - Target parsing for IP addresses (CIDR support) and KNX physical address ranges.

- **`knxmap/data/constants.py`** - KNX constants, device types, error codes.

### Message Flow
```
CLI (main.py) → KnxMap → Worker (asyncio task) → Protocol (DatagramProtocol) → Message encoding → Network
```

### Concurrency Model
- Asyncio event loop with queue-based worker distribution
- Default: 30 concurrent workers, 1 tunnel connection per gateway
- Configurable via `--workers` and `--connections` flags

## Development Notes

- **No external dependencies** - Keep it this way for deployment in restricted environments
- **Async-heavy** - Maintain asyncio patterns when modifying code
- **Protocol layers** - APCI/TPCI/CEMI changes cascade through message handling
- **Security tool** - Consider implications when modifying authentication, parsing, or connection logic
- **Hardware testing** - New features need testing against real KNX gateways
