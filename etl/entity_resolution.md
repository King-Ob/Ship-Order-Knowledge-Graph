# Entity Resolution and Reconciliation Workflow

## Overview
This document describes the entity resolution (ER) strategy for linking and deduplicating ships across multiple data sources (NVR, WDQS, messy CSV) to create a golden ship list.

**Reference**: OpenRefine Reconciliation (https://openrefine.org/docs/manual/reconciling)

---

## Entity Resolution Strategy

### Goal
Create a unified, deduplicated "golden" list of ships by:
1. Identifying duplicate ships across sources
2. Resolving conflicts between sources
3. Merging complementary information
4. Maintaining provenance for all claims

---

## Blocking Keys (Primary Matching)

Blocking keys are used to efficiently group potential matches before detailed comparison.

### Priority Order

1. **Hull Symbol** (Primary for US Navy)
   - **Normalization**: Uppercase, remove spaces
   - **Example**: `CVN-68`, `DDG-51`
   - **Match**: Exact match required
   - **Coverage**: US Navy ships, some international

2. **IMO Number** (International Standard)
   - **Normalization**: 7 digits, no formatting
   - **Example**: `8700001`, `4907372`
   - **Match**: Exact match required
   - **Coverage**: International ships with IMO numbers

3. **MMSI** (Maritime Mobile Service Identity)
   - **Normalization**: 9 digits, no formatting
   - **Example**: `369970000`, `235110000`
   - **Match**: Exact match required
   - **Coverage**: Active ships with MMSI

4. **Pennant Number + Operator** (International Navies)
   - **Normalization**: Uppercase pennant, normalized operator code
   - **Example**: `R09` + `RN`, `R08` + `RN`
   - **Match**: Both must match
   - **Coverage**: Royal Navy and other international navies

### Blocking Strategy

```python
def create_blocking_key(ship_record):
    """
    Create blocking key for efficient entity resolution.
    Returns tuple: (blocking_key, confidence_level)
    """
    # Priority 1: Hull Symbol
    if ship_record.get('hull_symbol'):
        hull = normalize_hull_symbol(ship_record['hull_symbol'])
        return (f"hull:{hull}", "high")
    
    # Priority 2: IMO Number
    if ship_record.get('imo_number'):
        imo = normalize_imo(ship_record['imo_number'])
        return (f"imo:{imo}", "high")
    
    # Priority 3: MMSI
    if ship_record.get('mmsi'):
        mmsi = normalize_mmsi(ship_record['mmsi'])
        return (f"mmsi:{mmsi}", "high")
    
    # Priority 4: Pennant + Operator
    if ship_record.get('pennant_number') and ship_record.get('operator'):
        pennant = normalize_pennant(ship_record['pennant_number'])
        operator = normalize_operator(ship_record['operator'])
        return (f"pennant:{pennant}+operator:{operator}", "high")
    
    # Fallback: Name + Class + Year
    name = normalize_name(ship_record.get('name', ''))
    class_name = normalize_class(ship_record.get('class', ''))
    year = extract_year(ship_record.get('commission_date', ''))
    return (f"name:{name}+class:{class_name}+year:{year}", "medium")
```

---

## Secondary Matching (Fuzzy Matching)

When blocking keys don't match exactly, use secondary matching with similarity scoring.

### Matching Criteria

1. **Normalized Name Similarity**
   - Remove prefixes: USS, HMS, etc.
   - Normalize: lowercase, remove punctuation
   - Similarity: Levenshtein distance or Jaro-Winkler
   - **Threshold**: ≥ 0.85 similarity

2. **Class Match**
   - Normalize class names
   - Remove "-class" suffix
   - **Threshold**: Exact match or high similarity (≥ 0.9)

3. **Commission Year Match**
   - Extract year from commission date
   - **Threshold**: Exact match or ±1 year

4. **Composite Score**
   - Name similarity: 40% weight
   - Class match: 30% weight
   - Year match: 30% weight
   - **Overall Threshold**: ≥ 0.80

### Secondary Matching Algorithm

```python
def secondary_match(ship1, ship2):
    """
    Calculate similarity score for secondary matching.
    Returns: (similarity_score, match_confidence)
    """
    scores = {}
    
    # Name similarity
    name1 = normalize_name(ship1.get('name', ''))
    name2 = normalize_name(ship2.get('name', ''))
    scores['name'] = jaro_winkler(name1, name2)
    
    # Class similarity
    class1 = normalize_class(ship1.get('class', ''))
    class2 = normalize_class(ship2.get('class', ''))
    scores['class'] = 1.0 if class1 == class2 else jaro_winkler(class1, class2)
    
    # Year match
    year1 = extract_year(ship1.get('commission_date', ''))
    year2 = extract_year(ship2.get('commission_date', ''))
    if year1 and year2:
        year_diff = abs(int(year1) - int(year2))
        scores['year'] = 1.0 if year_diff == 0 else max(0, 1.0 - year_diff / 10)
    else:
        scores['year'] = 0.5  # Unknown year, neutral score
    
    # Weighted composite score
    composite = (
        scores['name'] * 0.4 +
        scores['class'] * 0.3 +
        scores['year'] * 0.3
    )
    
    # Confidence level
    if composite >= 0.90:
        confidence = "high"
    elif composite >= 0.80:
        confidence = "medium"
    else:
        confidence = "low"
    
    return (composite, confidence)
```

---

## Conflict Resolution Strategy

When multiple sources provide conflicting information for the same ship, use precedence rules.

### Precedence Rules

1. **NVR is Authoritative For**:
   - US Navy ship status (Active, Decommissioned, Museum)
   - Commission/decommission dates
   - Hull classification symbols
   - Homeport assignments
   - Official ship names
   - Fleet assignments

2. **WDQS is Authoritative For**:
   - International vessels (non-US Navy)
   - MMSI and IMO numbers
   - Technical specifications (when NVR doesn't have them)
   - Launch dates
   - Cross-references to other databases

3. **Conflict Resolution Rules**:
   - **Status conflicts**: Prefer NVR for US Navy ships
   - **Date conflicts**: Prefer NVR for US Navy lifecycle dates
   - **Name conflicts**: Prefer NVR for US Navy ships
   - **Technical specs**: Merge (prefer more recent or detailed)
   - **Missing data**: Fill from other sources

### Conflict Resolution Algorithm

```python
def resolve_conflicts(ship_records):
    """
    Resolve conflicts between multiple records of the same ship.
    Returns: merged ship record with provenance.
    """
    # Sort by source priority
    sorted_records = sorted(
        ship_records,
        key=lambda x: source_priority(x['source']),
        reverse=True
    )
    
    merged = {
        'golden_id': generate_golden_id(sorted_records[0]),
        'claims': {},
        'provenance': {}
    }
    
    # Process each field with precedence
    for record in sorted_records:
        source = record['source']
        
        for field, value in record.items():
            if field in ['source', 'provenance']:
                continue
            
            # Check if field already set by higher-priority source
            if field not in merged['claims']:
                merged['claims'][field] = value
                merged['provenance'][field] = source
            elif source_priority(source) > source_priority(merged['provenance'][field]):
                # Higher priority source overwrites
                merged['claims'][field] = value
                merged['provenance'][field] = source
            elif value and not merged['claims'][field]:
                # Fill missing data from lower priority source
                merged['claims'][field] = value
                merged['provenance'][field] = source
    
    return merged

def source_priority(source):
    """Return priority score for source (higher = more authoritative)."""
    priorities = {
        'NVR': 100,
        'WDQS': 50,
        'messy_csv': 10
    }
    return priorities.get(source, 0)
```

---

## OpenRefine Reconciliation Workflow

### Step 1: Prepare Data for OpenRefine

1. **Export Combined Data**:
   ```bash
   # Combine all sources into single CSV
   python scripts/combine_sources.py \
       data/raw/nvr_export.json \
       data/raw/wdqs_export.json \
       data/raw/messy_ship_data.csv \
       -o data/staging/combined_ships.csv
   ```

2. **Add Source Column**: Include source identifier in each row

3. **Normalize Identifiers**: Pre-normalize hull symbols, IMO, MMSI

### Step 2: Import into OpenRefine

1. Open OpenRefine
2. Create new project from `data/staging/combined_ships.csv`
3. Ensure proper column types (dates, numbers)

### Step 3: Create Blocking Keys

1. **Create Blocking Key Column**:
   ```
   if(isBlank(cells["hull_symbol"].value), 
      if(isBlank(cells["imo_number"].value),
         if(isBlank(cells["mmsi"].value),
            "name:" + value + "+class:" + cells["class"].value + "+year:" + toString(year(cells["commission_date"].value)),
            "mmsi:" + cells["mmsi"].value),
         "imo:" + cells["imo_number"].value),
      "hull:" + cells["hull_symbol"].value)
   ```

2. **Cluster by Blocking Key**: Use "Cluster and Edit" on blocking key column

### Step 4: Reconciliation

1. **Reconcile Against Golden List** (if exists):
   - Use "Reconcile" → "Start reconciling"
   - Point to reconciliation service or local golden list

2. **Manual Review**:
   - Review clusters with multiple records
   - Mark matches/non-matches
   - Merge records

3. **Create Links**:
   - Use "Add column based on this column" → "Reconcile"
   - Create `sameAs` links between matched records

### Step 5: Export Golden List

1. **Export Resolved Data**:
   - File → Export → Custom tabular exporter
   - Include: golden_id, all merged fields, provenance columns

2. **Save to**: `data/golden/ships.csv`

---

## Golden Ship List Structure

### Required Columns

- `golden_id`: Unique identifier for resolved ship
- `name`: Resolved ship name
- `hull_symbol`: Primary identifier (if available)
- `imo_number`: IMO number (if available)
- `mmsi`: MMSI (if available)
- `pennant_number`: Pennant number (if available)
- `class`: Ship class
- `operator`: Operator/navy
- `status`: Current status
- `commission_date`: Commission date
- `decommission_date`: Decommission date (if applicable)
- `homeport`: Homeport (if applicable)
- `source_nvr`: Boolean - has NVR data
- `source_wdqs`: Boolean - has WDQS data
- `source_csv`: Boolean - has CSV data
- `provenance_*`: Source for each field

### Example Golden Record

```csv
golden_id,name,hull_symbol,imo_number,mmsi,class,operator,status,commission_date,source_nvr,source_wdqs,provenance_name,provenance_status
ship-CVN-68,USS Nimitz,CVN-68,,,Nimitz,United States Navy,Active,1975-05-03,true,true,NVR,NVR
ship-DDG-51,USS Arleigh Burke,DDG-51,,369970051,Arleigh Burke,United States Navy,Active,1991-07-04,true,true,NVR,WDQS
ship-RN-R09,HMS Prince of Wales,R09,4907372,235110000,Queen Elizabeth,Royal Navy,Active,2019-12-10,false,true,WDQS,WDQS
```

---

## Quality Assurance

### Before/After Comparison

1. **Before ER**:
   - Count total records across all sources
   - Count unique ships (by blocking key)
   - Identify duplicates

2. **After ER**:
   - Count golden records
   - Count resolved duplicates
   - Count links created

### Metrics

- **Deduplication Rate**: (duplicates resolved / total records) × 100
- **Link Coverage**: (ships with links / total ships) × 100
- **Source Coverage**: (ships with data from N sources / total ships)

---

## Automation Script

### Python Script for ER

```python
#!/usr/bin/env python3
"""
Entity Resolution Script
Combines multiple sources and performs entity resolution.
"""

import json
import csv
from collections import defaultdict
from difflib import SequenceMatcher

def normalize_name(name):
    """Normalize ship name for matching."""
    if not name:
        return ""
    # Remove prefixes
    prefixes = ['USS', 'HMS', 'HMNZS', 'HMAS', 'HMCS']
    for prefix in prefixes:
        if name.upper().startswith(prefix + ' '):
            name = name[len(prefix)+1:].strip()
    return name.lower().strip()

def normalize_hull_symbol(hull):
    """Normalize hull symbol."""
    return hull.upper().strip().replace(' ', '') if hull else None

def create_blocking_key(record):
    """Create blocking key for record."""
    if record.get('hull_symbol'):
        return f"hull:{normalize_hull_symbol(record['hull_symbol'])}"
    if record.get('imo_number'):
        return f"imo:{record['imo_number']}"
    if record.get('mmsi'):
        return f"mmsi:{record['mmsi']}"
    # Fallback
    name = normalize_name(record.get('name', ''))
    class_name = record.get('class', '').lower()
    year = record.get('commission_date', '')[:4] if record.get('commission_date') else ''
    return f"name:{name}+class:{class_name}+year:{year}"

def resolve_entities(records):
    """Perform entity resolution on records."""
    # Group by blocking key
    blocks = defaultdict(list)
    for record in records:
        key = create_blocking_key(record)
        blocks[key].append(record)
    
    # Resolve conflicts within blocks
    golden_records = []
    for key, block_records in blocks.items():
        if len(block_records) == 1:
            # Single record, no resolution needed
            golden_records.append(create_golden_record(block_records[0]))
        else:
            # Multiple records, resolve conflicts
            golden_records.append(resolve_conflicts(block_records))
    
    return golden_records

def create_golden_record(record):
    """Create golden record from single source record."""
    return {
        'golden_id': generate_golden_id(record),
        **record,
        'source_nvr': record.get('source') == 'NVR',
        'source_wdqs': record.get('source') == 'WDQS',
        'source_csv': record.get('source') == 'CSV'
    }

def resolve_conflicts(records):
    """Resolve conflicts between multiple records."""
    # Sort by source priority
    sorted_records = sorted(records, key=lambda x: source_priority(x.get('source', '')), reverse=True)
    
    # Merge with precedence
    merged = {}
    provenance = {}
    
    for record in sorted_records:
        source = record.get('source', 'unknown')
        for field, value in record.items():
            if field == 'source':
                continue
            if field not in merged or source_priority(source) > source_priority(provenance.get(field, '')):
                merged[field] = value
                provenance[field] = source
    
    merged['golden_id'] = generate_golden_id(sorted_records[0])
    merged['source_nvr'] = any(r.get('source') == 'NVR' for r in records)
    merged['source_wdqs'] = any(r.get('source') == 'WDQS' for r in records)
    merged['source_csv'] = any(r.get('source') == 'CSV' for r in records)
    
    return merged

def source_priority(source):
    """Return priority score for source."""
    priorities = {'NVR': 100, 'WDQS': 50, 'CSV': 10}
    return priorities.get(source, 0)

def generate_golden_id(record):
    """Generate golden ID for record."""
    if record.get('hull_symbol'):
        return f"ship-{normalize_hull_symbol(record['hull_symbol'])}"
    if record.get('imo_number'):
        return f"ship-IMO-{record['imo_number']}"
    if record.get('mmsi'):
        return f"ship-MMSI-{record['mmsi']}"
    # Fallback
    name = normalize_name(record.get('name', 'unknown'))
    year = record.get('commission_date', '')[:4] if record.get('commission_date') else 'unknown'
    return f"ship-{name}-{year}"

# Main execution
if __name__ == "__main__":
    # Load sources
    with open('data/raw/nvr_export.json') as f:
        nvr_data = json.load(f)
        nvr_records = [{'source': 'NVR', **v} for v in nvr_data['vessels']]
    
    with open('data/raw/wdqs_export.json') as f:
        wdqs_data = json.load(f)
        wdqs_records = [{'source': 'WDQS', **v} for v in wdqs_data['vessels']]
    
    # Combine
    all_records = nvr_records + wdqs_records
    
    # Resolve entities
    golden_records = resolve_entities(all_records)
    
    # Export
    with open('data/golden/ships.csv', 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=golden_records[0].keys())
        writer.writeheader()
        writer.writerows(golden_records)
    
    print(f"Resolved {len(all_records)} records into {len(golden_records)} golden records")
```

---

## References

- **OpenRefine**: https://openrefine.org/
- **OpenRefine Reconciliation**: https://openrefine.org/docs/manual/reconciling
- **Entity Resolution**: https://en.wikipedia.org/wiki/Record_linkage
- **Blocking Strategies**: https://en.wikipedia.org/wiki/Blocking_(statistics)

---

## Change Log

- **2024-01-15**: Initial entity resolution documentation created
  - Defined blocking keys and secondary matching
  - Created conflict resolution strategy
  - Documented OpenRefine workflow
  - Created automation script
