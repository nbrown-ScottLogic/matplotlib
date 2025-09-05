# Matplotlib Development

Matplotlib is a comprehensive Python library for creating static, animated, and interactive visualizations. It uses a Meson build system with C++ extensions and has extensive test coverage.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Initial Setup
- Install system dependencies required for compilation:
  - `sudo apt-get update`
  - `sudo apt-get install -y build-essential pkg-config libfreetype6-dev python3-dev python3-venv python3-pip`
- Create and activate a virtual environment:
  - `python3 -m venv venv`
  - `source venv/bin/activate` (Linux/macOS) or `venv\Scripts\activate` (Windows)
- Install development dependencies:
  - `pip install -r requirements/dev/dev-requirements.txt` -- takes 2-5 minutes
- Install matplotlib in editable mode:
  - `python -m pip install --verbose --no-build-isolation --editable ".[dev]"` -- takes 15-45 minutes. NEVER CANCEL. Set timeout to 60+ minutes.
  - Note: This downloads and compiles external dependencies like FreeType which requires internet access

### Build and Test Process
- **Full development build**: Takes 15-45 minutes due to C++ extension compilation. NEVER CANCEL builds - they are CPU intensive but normal.
- **Run tests**: `pytest` or `python -m pytest` -- takes 15-30 minutes for full suite. NEVER CANCEL. Set timeout to 45+ minutes.
- **Quick test**: `pytest lib/matplotlib/tests/test_basic.py` -- takes 1-2 minutes
- **Linting**: `python -m ruff check lib/matplotlib/` -- takes < 1 minute
- **Format code**: `python -m ruff format lib/matplotlib/` -- takes < 1 minute

### Common Development Commands
- `python -c "import matplotlib; print(matplotlib.__version__, matplotlib.__file__)"` -- verify installation
- `pytest --pyargs matplotlib.tests -v` -- run all tests with verbose output
- `pytest lib/matplotlib/tests/test_FILENAME.py::test_function_name` -- run specific test
- `python -m ruff check --fix lib/matplotlib/` -- auto-fix linting issues
- `pre-commit run --all-files` -- run all pre-commit hooks (if installed)

## Validation Scenarios

ALWAYS manually validate changes by running these complete scenarios:

### Basic Plotting Validation
```python
import matplotlib
matplotlib.use('Agg')  # Non-interactive backend for testing
import matplotlib.pyplot as plt
import numpy as np

# Test basic plotting
x = np.linspace(0, 10, 100)
y = np.sin(x)
plt.figure(figsize=(8, 6))
plt.plot(x, y, label='sin(x)')
plt.title('Test Plot')
plt.xlabel('X values')
plt.ylabel('Y values')
plt.legend()
plt.grid(True)
plt.savefig('test_plot.png', dpi=150, bbox_inches='tight')
print('Basic plotting works!')
```

### Test New Features End-to-End
- Create a simple script that exercises the new functionality
- Verify it produces correct output/plots
- Test with different backends if relevant: `matplotlib.use('TkAgg')`, `matplotlib.use('Qt5Agg')`
- Always test with the Agg backend for headless environments

## Critical Timing and Timeout Information

- **NEVER CANCEL BUILD COMMANDS** - Compilation of C++ extensions takes significant time
- **pip install editable build**: 15-45 minutes (set 60+ minute timeout)
- **Full test suite**: 15-30 minutes (set 45+ minute timeout)  
- **Single test file**: 1-5 minutes (set 10+ minute timeout)
- **Linting entire codebase**: < 1 minute (set 5+ minute timeout)
- **Documentation build**: 10-20 minutes (set 30+ minute timeout)

## Repository Structure

### Key Directories
- `lib/matplotlib/` - Main Python source code
- `lib/matplotlib/tests/` - Test suite (thousands of tests)
- `src/` - C++ extension modules  
- `extern/` - External dependencies (FreeType, etc.)
- `doc/` - Documentation source
- `galleries/` - Examples and tutorials
- `requirements/` - Dependency specifications

### Important Files
- `pyproject.toml` - Main project configuration and dependencies
- `meson.build` - Build system configuration  
- `tox.ini` - Test automation configuration
- `.pre-commit-config.yaml` - Code quality hooks
- `environment.yml` - Conda environment specification

## Build System Details

Matplotlib uses **Meson** (not setuptools) as its build system:
- Requires Python 3.11+
- Uses pybind11 for C++ bindings
- Downloads external dependencies during build (requires internet)
- Compiles C++ extensions which is time-intensive
- **Editable installs** require `--no-build-isolation` flag

## Testing Infrastructure

- Uses **pytest** framework exclusively
- Image comparison tests store results in `result_images/` directory
- Baseline images are in `lib/matplotlib/tests/baseline_images/`
- Tests require additional dependencies: see `requirements/testing/all.txt`
- Use `@image_comparison` decorator for visual regression tests
- Use `@check_figures_equal` decorator to compare two plotting methods

## Linting and Code Quality

- **Ruff** for linting and formatting (replaces flake8/black)
- Configuration in `pyproject.toml` under `[tool.ruff]`
- **mypy** for type checking: `tox -e stubtest`
- **pre-commit hooks** available: `pre-commit install` then `pre-commit run --all-files`

## Network Dependencies

Build process downloads external dependencies:
- FreeType font rendering library
- May require system packages: `libfreetype6-dev`, `pkg-config`
- If internet access is limited, install system packages first: `sudo apt-get install python3-matplotlib` for basic functionality

## CI Validation Requirements

Before committing changes, ALWAYS run:
1. `python -m ruff check lib/matplotlib/` -- must pass with no errors
2. `python -m ruff format lib/matplotlib/` -- apply auto-formatting  
3. `pytest lib/matplotlib/tests/` -- run relevant test subset
4. Manual validation of your specific changes using the scenarios above

## Common Troubleshooting

- **Build fails downloading FreeType**: Install system FreeType development packages
- **Import errors after installation**: Verify virtual environment is activated and check `matplotlib.__file__` path
- **Test failures with baseline images**: Visual tests may fail on different systems - this is normal for image comparison tests
- **Meson build issues**: Ensure `--no-build-isolation` flag is used for editable installs
- **Permission errors**: Don't use `sudo pip install` - use virtual environments instead

## Development Workflow Notes

- Always work in a virtual environment
- Install in editable mode for development: `pip install -e ".[dev]"`
- The first build takes longest due to compilation and dependency downloads
- Subsequent builds are incremental and much faster
- Use `pytest -n auto` for parallel test execution (requires pytest-xdist)
- Image tests create large output files - use `tools/visualize_tests.py` to review them