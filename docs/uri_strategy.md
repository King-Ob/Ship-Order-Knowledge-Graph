# URI Strategy for Naval Vessel Knowledge Graph

## Overview
This document defines the strategy for minting stable, collision-resistant URIs for entities in the Ship Order Knowledge Graph. The strategy handles dirty inputs, same-name ships, and missing data while ensuring URIs remain stable across data updates.

**Base URI**: `https://example.org/ship-ontology/entity/`

---

## Design Principles

### 1. Stability
- URIs should not change when data is updated or corrected
- URIs should be based on stable identifiers (hull symbols, IMO numbers)
- Avoid URIs based on mutable attributes (status, homeport)

### 2. Collision Resistance
- Handle same-name ships (e.g., multiple ships named "America" or "Barry")
- Handle ships with missing identifiers
- Handle international ships with different identification schemes

### 3. Human Readability
- URIs should be readable and meaningful
- Include entity type in URI path
- Use normalized, URL-safe identifiers

### 4. Uniqueness
- Each entity must have a unique URI
- No two ships should share the same URI
- Handle edge cases (same name, same date, etc.)

---

## URI Patterns

### Base URI Structure
```
https://example.org/ship-ontology/entity/{entity-type}/{identifier}
```

**Entity Types**:
- `ship/` - Individual ships
- `class/` - Ship classes
- `operator/` - Operators/navies
- `homeport/` - Homeports
- `builder/` - Builders/shipyards

---

## Ship URI Strategy

### Priority Order (Most Stable to Least Stable)

1. **Hull Symbol** (Primary for US Navy ships)
2. **IMO Number** (International standard)
3. **MMSI** (Maritime Mobile Service Identity)
4. **Pennant Number + Operator** (International navies)
5. **Normalized Name + Commission Date** (Fallback)
6. **Normalized Name + Launch Date** (Last resort)

### 1. Hull Symbol (Primary Strategy)

**Pattern**: `ship/{normalized-hull-symbol}`

**Normalization Rules**:
- Convert to uppercase
- Remove spaces
- Replace hyphens with underscores (or keep hyphens, both are URL-safe)
- Remove special characters

**Examples**:
- `CVN-68` → `ship/CVN-68` → `https://example.org/ship-ontology/entity/ship/CVN-68`
- `DDG-51` → `ship/DDG-51` → `https://example.org/ship-ontology/entity/ship/DDG-51`
- `LHA-6` → `ship/LHA-6` → `https://example.org/ship-ontology/entity/ship/LHA-6`

**Advantages**:
- Unique and stable (hull symbols are official identifiers)
- Human-readable
- Already normalized in most data sources

**When to Use**:
- US Navy ships (always have hull symbols)
- Ships with official hull classification symbols

### 2. IMO Number (International Standard)

**Pattern**: `ship/IMO-{imo-number}`

**Normalization Rules**:
- Use IMO number as-is (7 digits)
- Prefix with "IMO-" for clarity

**Examples**:
- `8700001` → `ship/IMO-8700001` → `https://example.org/ship-ontology/entity/ship/IMO-8700001`

**Advantages**:
- International standard
- Unique across all ships
- Stable identifier

**When to Use**:
- Ships with IMO numbers (especially commercial vessels or international ships)
- When hull symbol is not available

### 3. MMSI (Maritime Mobile Service Identity)

**Pattern**: `ship/MMSI-{mmsi-number}`

**Normalization Rules**:
- Use MMSI as-is (9 digits)
- Prefix with "MMSI-" for clarity
- Validate format (must be 9 digits)

**Examples**:
- `369970000` → `ship/MMSI-369970000` → `https://example.org/ship-ontology/entity/ship/MMSI-369970000`

**Advantages**:
- Unique identifier
- Used for active ships

**When to Use**:
- Ships with MMSI but no hull symbol or IMO
- Active ships (MMSI may not be available for decommissioned ships)

### 4. Pennant Number + Operator (International Navies)

**Pattern**: `ship/{operator-code}-{normalized-pennant}`

**Normalization Rules**:
- Normalize operator to code (see Operator URI Strategy)
- Normalize pennant number (uppercase, remove spaces)
- Combine with hyphen

**Examples**:
- `R09` + `Royal Navy` → `ship/RN-R09` → `https://example.org/ship-ontology/entity/ship/RN-R09`
- `R08` + `Royal Navy` → `ship/RN-R08` → `https://example.org/ship-ontology/entity/ship/RN-R08`

**Operator Codes**:
- `USN` - United States Navy
- `RN` - Royal Navy
- `RAN` - Royal Australian Navy
- `RCN` - Royal Canadian Navy
- `JMSDF` - Japan Maritime Self-Defense Force

**Advantages**:
- Handles international ships
- Combines operator context with pennant

**When to Use**:
- International ships with pennant numbers but no hull symbol
- Royal Navy ships (use pennant numbers)

### 5. Normalized Name + Commission Date (Fallback)

**Pattern**: `ship/{normalized-name}-{commission-year}`

**Normalization Rules**:
- **Name Normalization**:
  - Remove prefixes: "USS", "HMS", "HMNZS", "HMAS", etc.
  - Convert to lowercase
  - Replace spaces with hyphens
  - Remove special characters (keep only alphanumeric and hyphens)
  - Remove articles: "the", "a", "an"
  - Limit to 50 characters
- **Date Normalization**:
  - Use 4-digit year from commission date
  - If no commission date, use launch date year
  - If no dates, use "UNKNOWN"

**Examples**:
- `USS Nimitz` + `1975-05-03` → `ship/nimitz-1975` → `https://example.org/ship-ontology/entity/ship/nimitz-1975`
- `HMS Prince of Wales` + `2019-12-10` → `ship/prince-of-wales-2019` → `https://example.org/ship-ontology/entity/ship/prince-of-wales-2019`
- `USS America` + `2014-10-11` → `ship/america-2014` → `https://example.org/ship-ontology/entity/ship/america-2014`

**Handling Same-Name Ships**:
- If two ships have same normalized name and same commission year:
  - Add month: `ship/nimitz-1975-05`
  - If still collision, add day: `ship/nimitz-1975-05-03`
  - If still collision, add sequence number: `ship/nimitz-1975-05-03-1`

**Advantages**:
- Works when official identifiers are missing
- Human-readable
- Uses stable attributes (name + date)

**When to Use**:
- When hull symbol, IMO, MMSI, and pennant are all unavailable
- Historical ships with incomplete records

### 6. Normalized Name + Launch Date (Last Resort)

**Pattern**: `ship/{normalized-name}-launched-{launch-year}`

**Normalization Rules**:
- Same name normalization as above
- Use launch date year
- Prefix with "launched-" to distinguish from commission-based URIs

**Examples**:
- `USS Nimitz` + launch `1972-05-13` → `ship/nimitz-launched-1972` → `https://example.org/ship-ontology/entity/ship/nimitz-launched-1972`

**When to Use**:
- When commission date is also unavailable
- Very rare edge case

---

## Ship Class URI Strategy

**Pattern**: `class/{normalized-class-name}`

**Normalization Rules**:
- Convert to lowercase
- Replace spaces with hyphens
- Remove "class" suffix if present
- Remove special characters
- Remove articles

**Examples**:
- `Nimitz` → `class/nimitz` → `https://example.org/ship-ontology/entity/class/nimitz`
- `Arleigh Burke` → `class/arleigh-burke` → `https://example.org/ship-ontology/entity/class/arleigh-burke`
- `Ticonderoga` → `class/ticonderoga` → `https://example.org/ship-ontology/entity/class/ticonderoga`
- `Queen Elizabeth` → `class/queen-elizabeth` → `https://example.org/ship-ontology/entity/class/queen-elizabeth`

**Handling Variations**:
- `Nimitz-class` → `class/nimitz`
- `Nimitz Class` → `class/nimitz`
- `Nimitz-class aircraft carrier` → `class/nimitz`

---

## Operator URI Strategy

**Pattern**: `operator/{operator-code}`

**Normalization Rules**:
- Use standard operator codes (see list below)
- Convert to uppercase
- Use abbreviations when standard

**Standard Operator Codes**:
- `USN` - United States Navy
- `RN` - Royal Navy
- `USCG` - United States Coast Guard
- `USMC` - United States Marine Corps (for operational context)
- `RAN` - Royal Australian Navy
- `RCN` - Royal Canadian Navy
- `JMSDF` - Japan Maritime Self-Defense Force
- `FRN` - French Navy (Marine Nationale)
- `GN` - German Navy (Deutsche Marine)
- `IN` - Indian Navy

**Examples**:
- `United States Navy` → `operator/USN` → `https://example.org/ship-ontology/entity/operator/USN`
- `Royal Navy` → `operator/RN` → `https://example.org/ship-ontology/entity/operator/RN`

**Handling Unknown Operators**:
- Normalize name: lowercase, replace spaces with hyphens
- `operator/{normalized-name}`

---

## Homeport URI Strategy

**Pattern**: `homeport/{normalized-location}`

**Normalization Rules**:
- Extract city/state/country
- Convert to lowercase
- Replace spaces with hyphens
- Remove special characters
- Remove "Naval Base", "Naval Station" prefixes (keep for disambiguation if needed)
- Format: `{city}-{state}-{country}` or `{city}-{country}`

**Examples**:
- `Naval Base San Diego, CA` → `homeport/san-diego-ca-us` → `https://example.org/ship-ontology/entity/homeport/san-diego-ca-us`
- `Naval Station Norfolk, VA` → `homeport/norfolk-va-us` → `https://example.org/ship-ontology/entity/homeport/norfolk-va-us`
- `Portsmouth, UK` → `homeport/portsmouth-uk` → `https://example.org/ship-ontology/entity/homeport/portsmouth-uk`
- `Bremerton, WA` → `homeport/bremerton-wa-us` → `https://example.org/ship-ontology/entity/homeport/bremerton-wa-us`

**Handling Variations**:
- `San Diego, CA` → `homeport/san-diego-ca-us`
- `Naval Base San Diego` → `homeport/san-diego-ca-us`
- `San Diego (Museum)` → `homeport/san-diego-ca-us` (ignore parenthetical notes)

**Country Codes**:
- Use ISO 3166-1 alpha-2 codes (US, UK, AU, CA, etc.)

---

## Builder URI Strategy

**Pattern**: `builder/{normalized-builder-name}`

**Normalization Rules**:
- Convert to lowercase
- Replace spaces with hyphens
- Remove "Shipbuilding", "Shipyard", "Systems" suffixes (keep if needed for disambiguation)
- Remove special characters
- Use full company name if standard

**Examples**:
- `Newport News Shipbuilding` → `builder/newport-news-shipbuilding` → `https://example.org/ship-ontology/entity/builder/newport-news-shipbuilding`
- `BAE Systems` → `builder/bae-systems` → `https://example.org/ship-ontology/entity/builder/bae-systems`
- `Ingalls Shipbuilding` → `builder/ingalls-shipbuilding` → `https://example.org/ship-ontology/entity/builder/ingalls-shipbuilding`

**Note**: Builder data may not be present in all data sources. URIs should be minted when builder information becomes available.

---

## URI Minting Algorithm

### For Ships

```python
def mint_ship_uri(ship_data):
    """
    Mint a stable, collision-resistant URI for a ship.
    
    Priority order:
    1. Hull symbol (if available)
    2. IMO number (if available)
    3. MMSI (if available and valid)
    4. Pennant + Operator (if available)
    5. Normalized name + commission date
    6. Normalized name + launch date
    """
    base_uri = "https://example.org/ship-ontology/entity/ship/"
    
    # Priority 1: Hull Symbol
    if ship_data.get('hull_symbol'):
        hull = normalize_hull_symbol(ship_data['hull_symbol'])
        return base_uri + hull
    
    # Priority 2: IMO Number
    if ship_data.get('imo_number'):
        imo = ship_data['imo_number'].strip()
        if validate_imo(imo):
            return base_uri + f"IMO-{imo}"
    
    # Priority 3: MMSI
    if ship_data.get('mmsi'):
        mmsi = ship_data['mmsi'].strip()
        if validate_mmsi(mmsi):
            return base_uri + f"MMSI-{mmsi}"
    
    # Priority 4: Pennant + Operator
    if ship_data.get('pennant_number') and ship_data.get('operator'):
        pennant = normalize_pennant(ship_data['pennant_number'])
        operator_code = get_operator_code(ship_data['operator'])
        return base_uri + f"{operator_code}-{pennant}"
    
    # Priority 5: Name + Commission Date
    if ship_data.get('name') and ship_data.get('commission_date'):
        name = normalize_name(ship_data['name'])
        year = extract_year(ship_data['commission_date'])
        uri = base_uri + f"{name}-{year}"
        
        # Check for collisions and add disambiguation if needed
        if uri_exists(uri):
            uri = add_disambiguation(uri, ship_data['commission_date'])
        
        return uri
    
    # Priority 6: Name + Launch Date
    if ship_data.get('name') and ship_data.get('launch_date'):
        name = normalize_name(ship_data['name'])
        year = extract_year(ship_data['launch_date'])
        return base_uri + f"{name}-launched-{year}"
    
    # Last resort: Name only (with warning)
    if ship_data.get('name'):
        name = normalize_name(ship_data['name'])
        return base_uri + f"{name}-UNKNOWN"
    
    raise ValueError("Cannot mint URI: insufficient ship identification data")
```

### Normalization Functions

```python
def normalize_hull_symbol(hull):
    """Normalize hull symbol for URI."""
    return hull.upper().strip().replace(' ', '')

def normalize_name(name):
    """Normalize ship name for URI."""
    # Remove prefixes
    prefixes = ['USS', 'HMS', 'HMNZS', 'HMAS', 'HMCS', 'JDS', 'FS']
    for prefix in prefixes:
        if name.upper().startswith(prefix + ' '):
            name = name[len(prefix)+1:].strip()
    
    # Convert to lowercase, replace spaces with hyphens
    name = name.lower()
    name = re.sub(r'[^a-z0-9\s-]', '', name)  # Remove special chars
    name = re.sub(r'\s+', '-', name)  # Spaces to hyphens
    name = name[:50]  # Limit length
    
    return name

def normalize_pennant(pennant):
    """Normalize pennant number for URI."""
    return pennant.upper().strip().replace(' ', '')

def get_operator_code(operator):
    """Get standard operator code."""
    operator_map = {
        'United States Navy': 'USN',
        'Royal Navy': 'RN',
        'US Navy': 'USN',
        'U.S. Navy': 'USN',
        # ... more mappings
    }
    return operator_map.get(operator, normalize_name(operator).upper())
```

---

## Collision Detection and Resolution

### Same-Name Ships

**Problem**: Multiple ships can have the same name (e.g., USS America, HMS America).

**Solution**:
1. Use hull symbols when available (most stable)
2. Use IMO numbers (unique international standard)
3. Use commission date for disambiguation
4. If still collision, add sequence number

**Example**:
- `USS America (LHA-6)` → `ship/LHA-6` (hull symbol)
- `USS America (CV-66)` → `ship/CV-66` (hull symbol)
- If no hull symbol: `ship/america-2014` vs `ship/america-1965` (different commission years)

### Missing Data

**Problem**: Some ships may have incomplete identification data.

**Solution**:
1. Use best available identifier (follow priority order)
2. Document missing data in provenance metadata
3. Flag URIs based on fallback strategies for review

### Data Quality Issues

**Problem**: Dirty data (inconsistent formatting, typos, missing fields).

**Solution**:
1. Normalize all inputs before URI minting
2. Validate identifiers (IMO format, MMSI format)
3. Use data cleaning pipeline before URI generation
4. Maintain mapping table for known data quality issues

---

## Stability Guarantees

### What Makes URIs Stable

1. **Hull Symbols**: Official identifiers, rarely change
2. **IMO Numbers**: International standard, permanent
3. **MMSI**: Stable for active ships
4. **Commission Dates**: Historical facts, never change

### What to Avoid

1. **Status-based URIs**: Status can change (Active → Decommissioned)
2. **Homeport-based URIs**: Homeports can change
3. **Operator-based URIs**: Ships can be transferred
4. **Mutable Attributes**: Any attribute that can change

### URI Updates

**When URIs Should NOT Change**:
- Data corrections (typos, missing fields)
- Status changes
- Homeport changes
- Operator changes

**When URIs MAY Change** (with migration):
- Discovery of official identifier (e.g., find hull symbol for ship previously using name+date)
- Major data model changes (rare)

**Migration Strategy**:
- Maintain `owl:sameAs` links between old and new URIs
- Deprecate old URIs with `owl:deprecated` annotation
- Provide redirects or mappings

---

## Examples

### Example 1: US Navy Ship with Hull Symbol
```turtle
ship-entity:ship/CVN-68 a ship:Ship ;
    ship:hullSymbol "CVN-68" ;
    ship:shipName "USS Nimitz" ;
    ship:hasClass ship-entity:class/nimitz ;
    ship:operatedBy ship-entity:operator/USN ;
    ship:homeportedAt ship-entity:homeport/bremerton-wa-us .
```

### Example 2: Royal Navy Ship with Pennant
```turtle
ship-entity:ship/RN-R09 a ship:Ship ;
    ship:pennantNumber "R09" ;
    ship:shipName "HMS Prince of Wales" ;
    ship:hasClass ship-entity:class/queen-elizabeth ;
    ship:operatedBy ship-entity:operator/RN ;
    ship:homeportedAt ship-entity:homeport/portsmouth-uk .
```

### Example 3: Ship with IMO Number
```turtle
ship-entity:ship/IMO-8700001 a ship:Ship ;
    ship:imoNumber "8700001" ;
    ship:shipName "USS Theodore Roosevelt" ;
    ship:hullSymbol "CVN-71" ;
    ship:hasClass ship-entity:class/nimitz .
```

### Example 4: Fallback (Name + Date)
```turtle
ship-entity:ship/nimitz-1975 a ship:Ship ;
    ship:shipName "USS Nimitz" ;
    ship:commissionDate "1975-05-03"^^xsd:date ;
    ship:hullSymbol "CVN-68" .
```

---

## Testing and Validation

### Test Cases

1. **Same Name, Different Ships**: Verify unique URIs
2. **Missing Hull Symbol**: Verify fallback to IMO/MMSI/name+date
3. **International Ships**: Verify pennant+operator strategy
4. **Data Quality Issues**: Verify normalization handles typos, inconsistent formatting
5. **50+ Ships**: Verify no collisions in large dataset

### Validation Rules

1. **Uniqueness**: No two ships should have the same URI
2. **Stability**: URIs should not change for same ship across data updates
3. **Readability**: URIs should be human-readable
4. **Completeness**: All ships should have mintable URIs (even with missing data)

---

## References

- **OWL 2 Overview**: https://www.w3.org/TR/owl2-overview/
- **URI Best Practices**: https://www.w3.org/TR/cooluris/
- **SKOS URI Strategy**: https://www.w3.org/TR/skos-reference/
- **Navy Vessel Register**: https://www.nvr.navy.mil

---

## Change Log

- **2024-01-15**: Initial URI strategy document created
  - Defined priority order for ship URI minting
  - Established normalization rules
  - Documented collision resolution strategies
  - Created examples and validation rules
