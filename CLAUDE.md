# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AKShare is a comprehensive financial data interface library for Python that provides access to various financial markets including stocks, futures, options, funds, bonds, economic indicators, cryptocurrencies, and more. The library acts as a unified interface to scrape and fetch data from multiple Chinese and international financial websites.

## Development Setup

```bash
# Install with development dependencies
pip install -e ".[dev]"

# Install pre-commit hooks (recommended)
pre-commit install
```

## Common Commands

### Code Quality

```bash
# Run linter (fixes issues automatically)
ruff check --fix .

# Format code
ruff format .

# Run pre-commit hooks manually
pre-commit run --all-files
```

### Testing

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_func.py
```

### Building and Distribution

```bash
# Build package
python -m build

# Install locally in editable mode
pip install -e .
```

## Architecture

### Module Organization by Data Category

The library is organized by **financial data categories** under `akshare/`:

```
akshare/
├── stock/           # Stock market data (A-shares, US stocks, HK stocks)
├── fund/            # Mutual funds, ETFs, LOFs
├── futures/         # Futures market data
├── option/          # Options market data
├── bond/            # Bond market data
├── economic/        # Economic indicators
├── currency/        # Exchange rates
├── crypto/          # Cryptocurrency data
├── forex/           # Forex data
├── index/           # Market indices
├── energy/          # Energy commodities
├── bank/            # Banking data
├── news/            # News and articles
├── utils/           # Shared utilities
└── __init__.py      # Main interface that exports all public functions
```

### Standard Module Pattern

Each data category (e.g., `stock/`, `fund/`) follows this structure:

- `__init__.py` - Empty marker file
- `cons.py` - **Configuration file** containing:
  - API endpoint URLs
  - Request parameters/payloads
  - Authentication tokens
  - Constant values
- `*.py` - Individual implementation files, each containing specific data fetching functions

### Function Architecture

Typical data fetching function pattern:

1. **Import** configuration from `module/cons.py`
2. **Make HTTP request** using `requests` or `curl_cffi` (for anti-scraping protection)
3. **Parse response** (HTML via BeautifulSoup, JSON, or JavaScript via `mini-racer`/`akracer`)
4. **Return pandas DataFrame** - All public functions return DataFrames
5. **Error handling** - Use custom exceptions from `akshare.exceptions`

### Key Utility Modules

- `utils/request.py` - `request_with_retry()` - HTTP requests with retry logic and exponential backoff
- `utils/func.py` - `fetch_paginated_data()` - Handle paginated API responses with progress bars
- `utils/tqdm.py` - Progress bar wrapper
- `utils/datasets.py` - Path utilities for static data files in `akshare/data/`
- `exceptions.py` - Custom exception hierarchy (`AkshareException`, `APIError`, `DataParsingError`, etc.)

### Entry Points

The main `akshare/__init__.py` imports and re-exports all public functions from submodules. This is the only file excluded from Ruff linting due to its size (60,000+ lines).

### JavaScript Execution

Some websites require JavaScript execution for data extraction. The library uses:
- `mini-racer` on macOS/Windows
- `py-mini-racer` on Linux
- `akracer` on Linux (alternative)

These are used to execute JavaScript code embedded in HTML responses or to decrypt obfuscated data.

## Code Style

- **Linter/Formatter**: Ruff (configured in `pyproject.toml`)
- **Line length**: 88 characters
- **Target Python**: 3.13 (supports 3.8-3.14)
- **Quote style**: Double quotes
- **Import style**: No unused imports, sorted

### Commit Messages

The project uses **conventional commits** (enforced via pre-commit hooks):
- `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `test:`, `chore:`

## Adding New Data Sources

When adding new data fetching functions:

1. **Choose the appropriate category directory** (e.g., `stock/` for stock-related data)
2. **Create a new file** or use an existing one in that category
3. **Add URLs and constants to `module/cons.py`** - Don't hardcode URLs
4. **Use utility functions** from `utils/` when applicable:
   - `request_with_retry()` for HTTP requests
   - `fetch_paginated_data()` for paginated APIs
5. **Always return a pandas DataFrame**
6. **Handle errors** with appropriate custom exceptions
7. **Export the function** in `akshare/__init__.py`
8. **Add documentation** in `docs/data/` directory

## Testing

Test coverage is currently limited. The main test file is `tests/test_func.py` which primarily tests dataset utility functions.

When adding tests:
- Use pytest
- Place tests in `tests/`
- Mock HTTP responses to avoid hitting real APIs
- Test both success and error cases

## CI/CD

- **Multi-platform**: Ubuntu, macOS, Windows
- **Python versions**: 3.11, 3.12, 3.13, 3.14
- **Workflows**:
  - `checks.yml` - Runs on push/PR to main/dev branches (Ruff + pytest)
  - `release_and_deploy.yml` - Builds and publishes to PyPI on release tags

## Important Notes

- The main `__init__.py` is excluded from linting due to its size
- Static data files are stored in `akshare/data/` (e.g., `.json`, `.pk`, `.zip` files)
- Some functions may require API tokens stored in `cons.py` files
- The library scrapes public websites - interfaces may break if source sites change
- All data is for academic research purposes only
- Be mindful of rate limiting when adding new data sources - use delays and retry logic
