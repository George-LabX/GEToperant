# Test Coverage Analysis for GEToperant

## Executive Summary

**Current Test Coverage: 0%**

The GEToperant codebase has **no automated tests**. There are:
- No `tests/` directory
- No test files (`test_*.py` or `*_test.py`)
- No test configuration (`pytest.ini`, `tox.ini`, `setup.cfg`)
- No CI/CD pipeline (`.github/workflows/`)

This document analyzes the codebase and proposes a comprehensive testing strategy.

---

## Codebase Overview

| File | Lines | Description |
|------|-------|-------------|
| `GEToperant.py` | 1041 | Core extraction logic |
| `GEToperantGUI.py` | 492 | Tkinter GUI interface |
| **Total** | **1533** | |

### Key Functions

1. **`convertMRP()`** - Converts MPC2XL Row Profiles to Excel profiles
2. **`GEToperant()`** - Main extraction function with 3 export modes (Main, Sheets, Books)

---

## Priority 1: Critical Areas Needing Tests

### 1.1 Profile Parsing (High Priority)

The code supports three profile formats:

| Format | Code Location | Risk Level |
|--------|---------------|------------|
| `.xlsx` (Excel 2007+) | Lines 121-153 | Medium |
| `.xls` (Legacy Excel) | Lines 155-193 | High |
| `.mrp` (MPC2XL Row Profile) | Lines 195-215 | High |

**Recommended Tests:**
```python
# test_profile_parsing.py
def test_parse_xlsx_profile():
    """Test parsing modern Excel profiles"""

def test_parse_xls_profile():
    """Test parsing legacy Excel profiles"""

def test_parse_mrp_profile():
    """Test parsing MPC2XL Row Profile format"""

def test_invalid_profile_format():
    """Test error handling for invalid profile files"""

def test_empty_profile():
    """Test handling of empty profile files"""

def test_profile_missing_columns():
    """Test profiles with missing required columns"""
```

### 1.2 Med-PC Data File Parsing (High Priority)

The parser extracts metadata from Med-PC files using string slicing:

| Field | Parsing Logic | Risk |
|-------|---------------|------|
| Filename | `line[6:-1]` | Medium |
| Start Date | Complex Y2K correction | **High** |
| End Date | Complex Y2K correction | **High** |
| Subject | `line[9:-1]` | Medium |
| Experiment | `line[12:-1]` | Medium |
| Group | `line[7:-1]` | Medium |
| Box | `line[5:-1]` + float conversion | **High** |
| Start Time | Conditional offset handling | High |
| End Time | Conditional offset handling | High |
| MSN | `line[5:-1]` | Medium |

**Recommended Tests:**
```python
# test_medpc_parsing.py
def test_parse_filename():
    """Test filename extraction from Med-PC files"""

def test_parse_date_y2k_compliant():
    """Test date parsing for Y2K-compliant files"""

def test_parse_date_non_y2k():
    """Test Y2K correction for older files"""

def test_parse_subject_info():
    """Test subject/experiment/group extraction"""

def test_parse_box_numeric():
    """Test box number extraction and conversion"""

def test_parse_box_non_numeric():
    """Test handling of non-numeric box values"""

def test_parse_time_with_leading_space():
    """Test time parsing when line has leading space"""

def test_parse_time_without_leading_space():
    """Test time parsing without leading space"""
```

### 1.3 Array Data Extraction (High Priority)

The code extracts data from 26 arrays (A-Z) with complex state machine logic:

**Recommended Tests:**
```python
# test_array_extraction.py
def test_extract_single_element():
    """Test extracting a single array element e.g., A(0)"""

def test_extract_array_range():
    """Test extracting a range from an array"""

def test_extract_full_array():
    """Test extracting entire array when StopElement is None"""

def test_extract_with_increment():
    """Test extracting every Nth element"""

def test_extract_empty_array():
    """Test handling of empty arrays"""

def test_extract_array_out_of_bounds():
    """Test handling when StartElement exceeds array length"""

def test_extract_comments():
    """Test comment extraction from data files"""

def test_extract_multiple_subjects():
    """Test extraction when file contains multiple subjects"""
```

### 1.4 Date Y2K Correction Logic (Critical)

**Current Code (GEToperant.py lines 285-296):**
```python
elif 'Start Date' in line:
    if len(line) < 22:
        Startdate.append("20"+line[18:-1]+"-"+line[12:14]+"-"+line[15:17])
    else:
        Startdate.append(line[18:-1]+"-"+line[12:14]+"-"+line[15:17])
```

**Potential Issues:**
- Magic number `22` for line length is fragile
- No validation of date format
- Assumes specific string positions

**Recommended Tests:**
```python
# test_date_handling.py
def test_date_2digit_year():
    """Test date correction for 2-digit year (pre-Y2K)"""

def test_date_4digit_year():
    """Test date parsing for 4-digit year"""

def test_date_malformed():
    """Test handling of malformed date strings"""

def test_date_boundary_cases():
    """Test dates at the boundary of Y2K (1999/2000)"""
```

---

## Priority 2: Export Mode Tests

### 2.1 Main Mode (Single Worksheet)

```python
# test_export_main.py
def test_export_main_single_file():
    """Test exporting single file to one worksheet"""

def test_export_main_multiple_files():
    """Test exporting multiple files to one worksheet"""

def test_export_main_header_selection():
    """Test that header checkboxes work correctly"""

def test_export_main_creates_valid_xlsx():
    """Test that output is a valid Excel file"""
```

### 2.2 Sheets Mode (Separate Worksheets)

```python
# test_export_sheets.py
def test_export_sheets_creates_worksheets():
    """Test that each file gets its own worksheet"""

def test_export_sheets_long_filename():
    """Test worksheet naming with 32+ char filenames"""

def test_export_sheets_illegal_characters():
    """Test handling of illegal worksheet name characters"""
```

### 2.3 Books Mode (Separate Workbooks)

```python
# test_export_books.py
def test_export_books_creates_files():
    """Test that each input creates a separate output file"""

def test_export_books_naming():
    """Test output file naming convention"""

def test_export_books_directory_handling():
    """Test output to specified directory"""
```

---

## Priority 3: MRP Conversion Tests

```python
# test_mrp_conversion.py
def test_convert_mrp_basic():
    """Test basic MRP to Excel conversion"""

def test_convert_mrp_with_comments():
    """Test MRP conversion preserving comment labels"""

def test_convert_mrp_output_format():
    """Test that converted profile has correct structure"""

def test_convert_mrp_invalid_file():
    """Test error handling for invalid MRP files"""
```

---

## Priority 4: Edge Cases and Error Handling

### 4.1 Known Potential Bugs

| Issue | Location | Recommended Test |
|-------|----------|------------------|
| `eval()` usage | Multiple lines | Security test |
| `float(Box[i])` | Lines 427, 706, 980 | Non-numeric input test |
| No `with` statements | File operations | Resource leak test |
| Index assumptions | Array access | Bounds checking test |

### 4.2 Recommended Edge Case Tests

```python
# test_edge_cases.py
def test_empty_data_file():
    """Test handling of empty Med-PC files"""

def test_data_file_missing_fields():
    """Test files with missing required fields"""

def test_large_data_file():
    """Test performance with large files"""

def test_unicode_in_labels():
    """Test handling of unicode characters"""

def test_special_characters_in_paths():
    """Test file paths with spaces/special chars"""

def test_concurrent_exports():
    """Test multiple simultaneous exports"""
```

---

## Priority 5: Integration Tests

```python
# test_integration.py
def test_end_to_end_xlsx_profile_main_mode():
    """Full workflow: xlsx profile -> Med-PC files -> Excel output"""

def test_end_to_end_mrp_profile_sheets_mode():
    """Full workflow: MRP profile -> Med-PC files -> Sheets output"""

def test_end_to_end_xls_profile_books_mode():
    """Full workflow: xls profile -> Med-PC files -> Books output"""

def test_converted_mrp_profile_usage():
    """Test using a converted MRP profile for extraction"""
```

---

## Recommended Test Infrastructure

### Directory Structure
```
GEToperant/
├── tests/
│   ├── __init__.py
│   ├── conftest.py              # Pytest fixtures
│   ├── fixtures/
│   │   ├── profiles/            # Sample profile files
│   │   │   ├── sample.xlsx
│   │   │   ├── sample.xls
│   │   │   └── sample.MRP
│   │   └── data/                # Sample Med-PC data files
│   │       ├── single_subject.txt
│   │       ├── multi_subject.txt
│   │       └── edge_cases/
│   ├── test_profile_parsing.py
│   ├── test_medpc_parsing.py
│   ├── test_array_extraction.py
│   ├── test_date_handling.py
│   ├── test_export_main.py
│   ├── test_export_sheets.py
│   ├── test_export_books.py
│   ├── test_mrp_conversion.py
│   ├── test_edge_cases.py
│   └── test_integration.py
├── pytest.ini
├── requirements-test.txt
└── .github/
    └── workflows/
        └── test.yml             # CI pipeline
```

### pytest.ini
```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short
```

### requirements-test.txt
```
pytest>=7.0.0
pytest-cov>=4.0.0
openpyxl>=3.0.0
xlrd>=2.0.0
xlsxwriter>=3.0.0
```

### GitHub Actions CI (.github/workflows/test.yml)
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10', '3.11']

    steps:
    - uses: actions/checkout@v4
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        pip install -r requirements-test.txt
    - name: Run tests
      run: pytest --cov=. --cov-report=xml
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

---

## Code Quality Observations

### Issues to Address During Testing

1. **Use of `eval()`** (Security Risk)
   - Lines: 324, 330, 339, 349, 354-355, 367-368, etc.
   - The code uses `eval(currentarray).append(values)` to dynamically access array variables
   - **Recommendation:** Refactor to use a dictionary instead of `eval()`

2. **Code Duplication**
   - The three modes (`Main`, `Sheets`, `Books`) contain ~270 lines of nearly identical code
   - **Recommendation:** Extract common parsing logic into helper functions

3. **File Handling**
   - Files are opened without `with` statements
   - **Recommendation:** Use context managers for proper resource cleanup

4. **Magic Numbers**
   - Date parsing uses hardcoded indices (`line[18:-1]`, `line[12:14]`)
   - **Recommendation:** Use named constants or regex with named groups

5. **No Input Validation**
   - Profile and data file inputs are not validated before processing
   - **Recommendation:** Add validation with clear error messages

---

## Test Implementation Priority

| Priority | Category | Estimated Tests | Effort |
|----------|----------|-----------------|--------|
| P0 | Date Y2K handling | 4 | Low |
| P0 | Array extraction | 8 | Medium |
| P1 | Profile parsing | 6 | Medium |
| P1 | Med-PC parsing | 8 | Medium |
| P2 | Export modes | 10 | High |
| P2 | MRP conversion | 4 | Low |
| P3 | Edge cases | 6 | Medium |
| P3 | Integration | 4 | High |
| **Total** | | **50** | |

---

## Next Steps

1. **Create test fixtures**: Sample Med-PC data files and profile files
2. **Set up pytest infrastructure**: `pytest.ini`, `conftest.py`, directory structure
3. **Implement P0 tests first**: Date handling and array extraction
4. **Add CI pipeline**: GitHub Actions for automated testing
5. **Refactor code**: Address `eval()` usage and code duplication to improve testability
6. **Achieve 80% coverage target**: Focus on core extraction logic

---

*Generated: January 2026*
