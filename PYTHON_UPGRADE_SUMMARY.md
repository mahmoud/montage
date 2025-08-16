# Python 3.11+ Upgrade Summary

This document summarizes the changes made to upgrade Montage from Python 3.9 to Python 3.11+ compatibility.

## Issues Identified and Fixed

### 1. SQLAlchemy Compatibility Issues

**Problem**: SQLAlchemy 1.2.19 was incompatible with Python 3.11+ and used deprecated imports.

**Solutions**:
- Upgraded SQLAlchemy from `==1.2.19` to `>=1.4.0,<2.0.0`
- Fixed import path for `_ModuleMarker` in `montage/check_rdb.py`:
  - Added fallback from `sqlalchemy.ext.declarative.clsregistry` to `sqlalchemy.orm.clsregistry`
- Fixed registry access pattern for SQLAlchemy 1.4+:
  - Added compatibility for both `_decl_class_registry` (old) and `registry._class_registry` (new)

### 2. Missing pkg_resources

**Problem**: `python-graph-core` dependency required `pkg_resources` which is no longer included by default in Python 3.12+.

**Solution**: Added `setuptools` to `requirements-dev.txt` to ensure `pkg_resources` is available.

### 3. CFFI Compilation Issues

**Problem**: CFFI 1.16.0 couldn't compile on Python 3.13 due to deprecated C API usage.

**Solution**: 
- Added explicit `cffi>=1.17.0` constraint to `requirements.in`
- Updated cryptography from `==40.0.2` to `>=41.0.0` for better Python 3.13 support

### 4. Development Dependencies

**Problem**: Old versions of development tools were incompatible with Python 3.11+.

**Solutions**:
- Updated tox from `<3.15.0` to `>=4.0.0`
- Updated pytest from `==4.6.9` to `>=7.0`
- Updated coverage from `==5.0.2` to `>=6.0`

## Files Modified

### Core Dependencies
- `requirements.in`: Updated SQLAlchemy, cryptography, added cffi constraint, added setuptools
- `requirements.txt`: Regenerated with updated dependencies
- `requirements-dev.txt`: Updated development tools, added setuptools

### Configuration
- `tox.ini`: Updated to support py311, py312, py313 and modern tox version

### Code Changes
- `montage/check_rdb.py`: Fixed SQLAlchemy imports and registry access for 1.4+ compatibility

## Testing

The upgrade has been tested with Python 3.13 (the available version in the environment). All tests pass with only deprecation warnings, which are expected and don't affect functionality.

## Warnings

The following deprecation warnings appear but don't affect functionality:
- `pkg_resources` deprecation warnings from `python-graph-core`
- SQLAlchemy 2.0 compatibility warnings (expected since we're using 1.4.x)
- `datetime.utcfromtimestamp()` deprecation in boltons

## Compatibility

The codebase now supports:
- Python 3.11+
- SQLAlchemy 1.4.x (with 2.0 compatibility warnings)
- Modern development tools (tox 4.x, pytest 7.x+, coverage 6.x+)

## Next Steps (Optional)

For future improvements, consider:
1. Upgrading to SQLAlchemy 2.0 to remove compatibility warnings
2. Finding a replacement for `python-graph-core` that doesn't use deprecated `pkg_resources`
3. Updating other deprecated datetime usage in the codebase