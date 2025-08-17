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

### 2. SQLAlchemy Join Ambiguity Issue (Python 3.11)

**Problem**: SQLAlchemy 1.4+ is stricter about ambiguous joins. Complex queries with multiple joins caused errors like "Can't determine which FROM clause to join from".

**Solution**: Fixed the complex query in `montage/rdb.py` around line 2725 in the `JurorDAO.get_all_rounds_task_counts()` method:
- Replaced the ambiguous `user_rounds_join` reference with an explicit join clause
- Made the FROM clause structure more explicit to avoid ambiguity

### 3. Missing pkg_resources

**Problem**: `python-graph-core` dependency required `pkg_resources` which is no longer included by default in Python 3.12+.

**Solution**: Added `setuptools` to `requirements-dev.txt` to ensure `pkg_resources` is available.

### 4. CFFI Compilation Issues

**Problem**: CFFI 1.16.0 couldn't compile on Python 3.13 due to deprecated C API usage.

**Solution**: 
- Added explicit `cffi>=1.17.0` constraint to `requirements.in`
- Updated cryptography from `==40.0.2` to `>=41.0.0` for better Python 3.13 support

### 5. Development Dependencies

**Problem**: Old versions of development tools were incompatible with Python 3.11+.

**Solutions**:
- Updated tox from `<3.15.0` to `>=4.0.0`
- Updated pytest from `==4.6.9` to `>=7.0`
- Updated coverage from `==5.0.2` to `>=6.0`

### 6. Tox Configuration Issues

**Problem**: Tox was failing to find the montage module path, causing "file or directory not found: /montage" errors.

**Solution**: Updated `tox.ini` to run tests from the source directory instead of trying to use the `{envsitepackagesdir}` variable:
- Changed the test command to use `{toxinidir}/montage/tests/` instead of `{envsitepackagesdir}/montage`
- Removed the `changedir = .tox` setting that was causing path issues

## Files Modified

### Core Dependencies
- `requirements.in`: Updated SQLAlchemy, cryptography, added cffi constraint, added setuptools
- `requirements.txt`: Regenerated with updated dependencies
- `requirements-dev.txt`: Updated development tools, added setuptools

### Configuration
- `tox.ini`: Updated to support py311, py312, py313 and modern tox version, fixed test path

### Code Changes
- `montage/check_rdb.py`: Fixed SQLAlchemy imports and registry access for 1.4+ compatibility
- `montage/rdb.py`: Fixed SQLAlchemy join ambiguity in `JurorDAO.get_all_rounds_task_counts()` method

## Testing

The upgrade has been tested with Python 3.13 (the available version in the environment). The fixes address:
- ✅ Import errors resolved
- ✅ SQLAlchemy compatibility issues fixed
- ✅ Tox path resolution issues resolved
- ✅ SQLAlchemy join ambiguity issues resolved

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

## Testing Commands

To test the fixes:

```bash
# Test with Python 3.11 (if available)
tox -e py311

# Test with Python 3.12 (if available)  
tox -e py312

# Test with Python 3.13 (if available)
tox -e py313

# Run all available Python versions
tox
```

## Next Steps (Optional)

For future improvements, consider:
1. Upgrading to SQLAlchemy 2.0 to remove compatibility warnings
2. Finding a replacement for `python-graph-core` that doesn't use deprecated `pkg_resources`
3. Updating other deprecated datetime usage in the codebase
4. Consider modernizing the query patterns to use SQLAlchemy 2.0 style queries