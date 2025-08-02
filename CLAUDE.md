# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Thrifty is proof-of-concept SDR (Software Defined Radio) software for TDOA (Time Difference of Arrival) positioning using inexpensive SDR hardware such as the RTL-SDR. The project enables wildlife tracking and positioning using hyperbolic positioning systems.

## Development Commands

### Installation and Setup
```bash
# Install in development mode (creates symbolic link)
make dev

# Setup virtual environment
make venv

# Install requirements
make init
pip install -r requirements.txt
```

### Testing and Quality
```bash
# Run unit tests
make test
py.test tests

# Run specific test
py.test tests/test_specific.py

# Lint code
make lint
flake8 thrifty/ tests/
pylint -rn thrifty/*.py tests/*.py
```

### Documentation
```bash
# Generate documentation
make docs
cd docs && make html
```

## Core Architecture

### Main Modules (thrifty/)
- **cli.py**: Central command-line interface routing to all modules
- **detect.py**: Signal detection and SoA (Start of Arrival) estimation
- **identify.py**: Transmitter ID identification and duplicate filtering
- **matchmaker.py**: Match detections across multiple receivers
- **tdoa_est.py**: TDOA estimation using beacon synchronization
- **pos_est.py**: Position estimation from TDOA data

### Signal Processing Pipeline
1. **Capture**: Raw SDR data capture via `fastcard_capture.py`
2. **Detection**: Signal detection via `detect.py` → produces .toad files
3. **Identification**: ID extraction via `identify.py`
4. **Matching**: Cross-receiver matching via `matchmaker.py`
5. **TDOA**: Time difference calculation via `tdoa_est.py`
6. **Position**: Final positioning via `pos_est.py`

### Data Flow
- Raw data: `.card` files (captured SDR data)
- Detections: `.toad` files (Time of Arrival Detections)
- Matches: `.match` files
- Clock models and position estimates

### Analysis and Utilities
- **Analysis tools**: `*_analysis.py` modules for data examination
- **Templates**: `template_generate.py`, `template_extract.py` for signal templates
- **Experimental**: `experimental/` directory contains research implementations
- **Scripts**: `scripts/` directory for standalone analysis tools

### Configuration
- Uses configuration files (e.g., `detector.cfg`)
- Settings managed via `settings.py` with custom parsers in `setting_parsers.py`
- Pylint configuration in `pylintrc` with project-specific rules

## Typical Workflow

### Single Receiver Processing
```bash
thrifty capture rx0.card
thrifty detect rx0.card -o rx0.toad
```

### Multi-Receiver TDOA Pipeline
```bash
# At each receiver
thrifty capture rx0.card
thrifty detect rx0.card -o rx0.toad

# At processing server
thrifty identify rx0.toad rx1.toad
thrifty match
thrifty tdoa
thrifty pos
```

### Using Makefiles
The `example/` directory contains a Makefile that automates the full pipeline:
```bash
cd example/
# Edit detector.cfg as needed
thrifty capture cards/rxX.card
make  # Runs full pipeline: detect → identify → match → tdoa → pos
```

## Code Style and Standards

- Python 3.3+ compatible (converted from Python 2.7)
- Uses numpy/scipy for numerical computing
- Pylint configuration enforces snake_case naming
- Line length limit: 100 characters
- Tests use pytest framework
- Dependencies: numpy, scipy (core), matplotlib (analysis), pytest/flake8/pylint (dev)

## External Components

- **fastcard/**: C implementation for fast SDR data capture
- **fastdet/**: C++ implementation for accelerated detection
- **rpi/**: Raspberry Pi configuration files and scripts

The codebase integrates with external hardware (RTL-SDR) and relies on compiled components (fastcard/fastdet) for performance-critical operations.