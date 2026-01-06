# Data Quality Rules - Naval Vessel Knowledge Graph

## Overview
This document defines the data quality validation rules for the Ship Order Knowledge Graph. These rules are implemented using SHACL (Shapes Constraint Language) and ensure data integrity, completeness, and consistency.

**SHACL Shapes File**: `shapes/ship_core.shacl.ttl`  
**Reference**: SHACL Specification (https://www.w3.org/TR/shacl/)

---

## Severity Levels

### Violation (sh:Violation) - **FAIL**
Critical errors that prevent data from being valid. These must be fixed before data can be ingested.

**Examples**:
- Missing required fields (name, class, operator)
- Invalid data types (wrong date format, non-numeric values)
- Pattern violations (invalid hull symbol, MMSI format)
- Logical inconsistencies (decommission before commission)

### Warning (sh:Warning) - **WARN**
Issues that should be addressed but don't prevent data processing. Data may be incomplete or have quality concerns.

**Examples**:
- Missing recommended fields (homeport for active ships)
- Pattern suggestions (pennant number format)
- Logical recommendations (launch date should be before commission)

### Info (sh:Info) - **PASS/INFO**
Recommendations for data completeness. These are informational and don't indicate errors.

**Examples**:
- Missing optional but recommended fields (length, displacement)
- Data completeness suggestions

---

## Core Validation Rules

### 1. Required Fields for All Ships

**Rule**: Every ship must have:
- `shipName` (exactly one)
- `hasClass` (exactly one - links to ShipClass)
- `operatedBy` (exactly one - links to Operator)
- `commissionDate` (valid xsd:date)
- At least one identifier: `hullSymbol` OR `imoNumber` OR `mmsi` OR `pennantNumber`

**Severity**: Violation (FAIL)

**Example Failures**:
```turtle
# Missing name
ship-entity:ship/CVN-68 a ship:Ship ;
    ship:hullSymbol "CVN-68" .
    # Missing ship:shipName - FAIL

# Missing class
ship-entity:ship/CVN-68 a ship:Ship ;
    ship:shipName "USS Nimitz" .
    # Missing ship:hasClass - FAIL

# Missing all identifiers
ship-entity:ship/test a ship:Ship ;
    ship:shipName "Test Ship" .
    # Missing hullSymbol, imoNumber, mmsi, pennantNumber - FAIL
```

---

### 2. Active Ship Requirements (Stricter)

**Rule**: Ships with status "Active" must have:
- `hasClass` (required)
- `operatedBy` (required)
- At least one identifier (required)
- `homeportedAt` (warning if missing)

**Severity**: 
- Missing class/operator/identifier: Violation (FAIL)
- Missing homeport: Warning (WARN)

**Rationale**: Active ships are operational and should have complete assignment information.

**Example**:
```turtle
# Active ship without homeport - WARNING
ship-entity:ship/CVN-68 a ship:Ship ;
    ship:shipName "USS Nimitz" ;
    ship:status "Active" .
    # Missing ship:homeportedAt - WARN (should have homeport)
```

---

## Identifier Pattern Validation

### 3. Hull Symbol Pattern

**Rule**: Hull symbols must match pattern: `^[A-Z]{1,4}-[0-9]{1,4}$`

**Pattern**: 1-4 uppercase letters, hyphen, 1-4 digits

**Valid Examples**:
- `CVN-68` ✅
- `DDG-51` ✅
- `LHA-6` ✅
- `CG-47` ✅
- `SSN-688` ✅

**Invalid Examples**:
- `cvn-68` ❌ (lowercase)
- `CVN68` ❌ (missing hyphen)
- `CVN-ABC` ❌ (letters after hyphen)
- `CVN-` ❌ (missing number)

**Severity**: Violation (FAIL)

---

### 4. MMSI Pattern

**Rule**: MMSI must be exactly 9 digits: `^[0-9]{9}$`

**Valid Examples**:
- `369970000` ✅
- `235110000` ✅

**Invalid Examples**:
- `36997000` ❌ (8 digits)
- `3699700000` ❌ (10 digits)
- `36997000A` ❌ (contains letter)
- `369-970-000` ❌ (contains hyphens)

**Severity**: Violation (FAIL)

**Reference**: ITU (International Telecommunication Union) standard for Maritime Mobile Service Identity.

---

### 5. IMO Number Pattern

**Rule**: IMO number must be exactly 7 digits: `^[0-9]{7}$`

**Valid Examples**:
- `8700001` ✅
- `4907372` ✅

**Invalid Examples**:
- `870001` ❌ (6 digits)
- `87000010` ❌ (8 digits)
- `870000A` ❌ (contains letter)

**Severity**: Violation (FAIL)

**Reference**: IMO (International Maritime Organization) ship identification number standard.

---

### 6. Pennant Number Pattern

**Rule**: Pennant numbers should match pattern: `^[A-Z0-9]{1,3}[-]?[0-9]{1,4}$`

**Pattern**: 1-3 alphanumeric characters, optional hyphen, 1-4 digits

**Valid Examples**:
- `R09` ✅
- `R08` ✅
- `D-123` ✅
- `A1` ✅

**Invalid Examples**:
- `R-09-` ❌ (trailing hyphen)
- `R` ❌ (no numbers)

**Severity**: Warning (WARN)

**Note**: Pennant number formats vary by navy, so this is a warning rather than a violation.

---

## Controlled Vocabulary Validation

### 7. Ship Type Vocabulary

**Rule**: Ship type must be from the SKOS ship type taxonomy.

**Valid Values** (from SKOS taxonomy):
- `Aircraft Carrier`
- `Carrier`
- `Nuclear-Powered Aircraft Carrier`
- `Conventional Aircraft Carrier`
- `Cruiser`
- `Guided Missile Cruiser`
- `Destroyer`
- `Guided Missile Destroyer`
- `Frigate`
- `Battleship`
- `Amphibious Assault Ship`
- `Submarine`
- `Attack Submarine`
- `Nuclear-Powered Attack Submarine`
- `Surface Combatant`

**Invalid Examples**:
- `carrier` ❌ (lowercase - use "Aircraft Carrier")
- `Warship` ❌ (not in taxonomy)
- `Boat` ❌ (not in taxonomy)

**Severity**: Violation (FAIL)

**Reference**: `taxonomy/ship_type.skos.ttl`

---

### 8. Status Vocabulary

**Rule**: Status must be from the SKOS status taxonomy.

**Valid Values** (from SKOS taxonomy):
- `Active`
- `In Service`
- `Commissioned`
- `Operational`
- `Reserve`
- `Reserve Fleet`
- `In Reserve`
- `Mothballed`
- `Decommissioned`
- `Decom`
- `Out of Service`
- `Stricken`
- `Museum`
- `Museum Ship`
- `Preserved`
- `Memorial`
- `Under Construction`
- `Building`
- `In Build`
- `Launched`
- `Scrapped`
- `Disposed`
- `Recycled`
- `Sunk`
- `Lost`
- `Transferred`

**Invalid Examples**:
- `active` ❌ (lowercase - use "Active")
- `Inactive` ❌ (not in taxonomy)
- `Retired` ❌ (not in taxonomy - use "Decommissioned")

**Severity**: Violation (FAIL)

**Reference**: `taxonomy/status.skos.ttl`

---

## Data Type Validation

### 9. Date Format Validation

**Rule**: All date fields must be valid `xsd:date` format: `YYYY-MM-DD`

**Fields**:
- `commissionDate`
- `decommissionDate`
- `launchDate`

**Valid Examples**:
- `1975-05-03` ✅
- `1986-10-25` ✅
- `1945-09-10` ✅

**Invalid Examples**:
- `1975/05/03` ❌ (wrong separator)
- `05-03-1975` ❌ (wrong order)
- `1975-5-3` ❌ (not zero-padded)
- `1975-13-03` ❌ (invalid month)
- `1975-05-32` ❌ (invalid day)

**Severity**: Violation (FAIL)

---

### 10. Numeric Type Validation

**Rule**: Numeric fields must be valid numeric types with appropriate constraints.

**Fields and Types**:
- `length`, `waterlineLength`, `beam`, `draft`: `xsd:decimal` (≥ 0)
- `displacementLight`, `displacementStandard`, `displacementFullLoad`: `xsd:decimal` (≥ 0)
- `crewComplement`: `xsd:integer` (≥ 0)
- `maxSpeed`: `xsd:decimal` (≥ 0)

**Valid Examples**:
- `332.8` ✅ (decimal)
- `3200` ✅ (integer)
- `0.0` ✅ (zero allowed)

**Invalid Examples**:
- `"332.8"` ❌ (string, not number)
- `-10.5` ❌ (negative)
- `abc` ❌ (not numeric)

**Severity**: Violation (FAIL)

---

## Logical Constraints

### 11. Date Ordering Constraints

**Rule 11a**: Decommission date must be after commission date.

**Example**:
```turtle
# Valid
ship:commissionDate "1975-05-03"^^xsd:date .
ship:decommissionDate "2004-09-30"^^xsd:date .  # After commission ✅

# Invalid
ship:commissionDate "1975-05-03"^^xsd:date .
ship:decommissionDate "1970-01-01"^^xsd:date .  # Before commission ❌
```

**Severity**: Violation (FAIL)

---

**Rule 11b**: Launch date should be before commission date.

**Example**:
```turtle
# Valid
ship:launchDate "1972-05-13"^^xsd:date .
ship:commissionDate "1975-05-03"^^xsd:date .  # After launch ✅

# Warning
ship:launchDate "1980-01-01"^^xsd:date .
ship:commissionDate "1975-05-03"^^xsd:date .  # Before launch ⚠️ (unusual but possible)
```

**Severity**: Warning (WARN)

**Note**: Ships are typically launched before commissioning, but exceptions exist (e.g., ships commissioned before launch in rare cases).

---

### 12. Length Consistency

**Rule**: Waterline length should not exceed overall length.

**Example**:
```turtle
# Valid
ship:length 332.8 .
ship:waterlineLength 317.0 .  # Less than length ✅

# Invalid
ship:length 300.0 .
ship:waterlineLength 350.0 .  # Exceeds length ❌ (data quality issue)
```

**Severity**: Violation (FAIL)

**Rationale**: This indicates a data quality issue. Waterline length is measured at the waterline, which is always less than or equal to overall length.

---

## Relationship Validation

### 13. Valid Class Relationship

**Rule**: `hasClass` must link to a valid `ShipClass` instance (IRI).

**Valid**:
```turtle
ship-entity:ship/CVN-68 ship:hasClass ship-entity:class/nimitz .
ship-entity:class/nimitz a ship:ShipClass .
```

**Invalid**:
```turtle
# Missing ShipClass instance
ship-entity:ship/CVN-68 ship:hasClass ship-entity:class/nimitz .
# ship-entity:class/nimitz not defined as ShipClass ❌

# Wrong type
ship-entity:ship/CVN-68 ship:hasClass ship-entity:operator/USN .  # Wrong type ❌
```

**Severity**: Violation (FAIL)

---

### 14. Valid Operator Relationship

**Rule**: `operatedBy` must link to a valid `Operator` instance (IRI).

**Valid**:
```turtle
ship-entity:ship/CVN-68 ship:operatedBy ship-entity:operator/USN .
ship-entity:operator/USN a ship:Operator .
```

**Invalid**:
```turtle
# Missing Operator instance
ship-entity:ship/CVN-68 ship:operatedBy ship-entity:operator/USN .
# ship-entity:operator/USN not defined as Operator ❌
```

**Severity**: Violation (FAIL)

---

### 15. Valid Homeport Relationship

**Rule**: `homeportedAt` must link to a valid `Homeport` instance (IRI) if present.

**Valid**:
```turtle
ship-entity:ship/CVN-68 ship:homeportedAt ship-entity:homeport/bremerton-wa-us .
ship-entity:homeport/bremerton-wa-us a ship:Homeport .
```

**Invalid**:
```turtle
# Missing Homeport instance
ship-entity:ship/CVN-68 ship:homeportedAt ship-entity:homeport/bremerton-wa-us .
# ship-entity:homeport/bremerton-wa-us not defined as Homeport ❌
```

**Severity**: Violation (FAIL)

**Note**: Homeport is optional, but if provided, it must be valid.

---

## Data Completeness Recommendations

### 16. Recommended Fields

**Rule**: The following fields are recommended for data completeness but not required.

**Fields**:
- `length` (Info)
- `displacementFullLoad` (Info)
- `beam` (Info)
- `draft` (Info)
- `crewComplement` (Info)

**Severity**: Info (PASS/INFO)

**Rationale**: These fields enhance data quality and enable more comprehensive queries, but their absence doesn't prevent data ingestion.

---

## Validation Report Structure

### Report Format

A SHACL validation report should include:

1. **Summary**:
   - Total violations (FAIL)
   - Total warnings (WARN)
   - Total info messages (INFO)

2. **Per-Entity Results**:
   - Entity IRI
   - Shape that was validated
   - Constraint violations with:
     - Severity level
     - Constraint path
     - Error message
     - Focus node (the entity that failed)

3. **Example Report Structure**:
```json
{
  "conforms": false,
  "results": [
    {
      "focusNode": "ship-entity:ship/CVN-68",
      "resultSeverity": "http://www.w3.org/ns/shacl#Violation",
      "sourceShape": "ship:ShipShape",
      "resultPath": "ship:shipName",
      "message": "Ship must have exactly one name (shipName property)"
    },
    {
      "focusNode": "ship-entity:ship/CVN-68",
      "resultSeverity": "http://www.w3.org/ns/shacl#Warning",
      "sourceShape": "ship:ActiveShipShape",
      "resultPath": "ship:homeportedAt",
      "message": "Active ship should have a homeport assignment"
    }
  ]
}
```

---

## Common Validation Failures

### Missing Identifier
**Error**: "Ship must have at least one identifier: hullSymbol, imoNumber, mmsi, or pennantNumber"

**Fix**: Add at least one identifier property.

### Invalid Hull Symbol Format
**Error**: "Hull symbol must match pattern: 1-4 letters, hyphen, 1-4 numbers"

**Fix**: Ensure hull symbol matches pattern (e.g., `CVN-68`, not `cvn68`).

### Invalid Date Format
**Error**: "Commission date must be valid xsd:date format (YYYY-MM-DD)"

**Fix**: Use ISO 8601 date format: `YYYY-MM-DD`.

### Invalid Ship Type
**Error**: "Ship type must be a valid value from SKOS ship type taxonomy"

**Fix**: Use a value from the SKOS taxonomy (see `taxonomy/ship_type.skos.ttl`).

### Waterline Length Exceeds Overall Length
**Error**: "Waterline length cannot exceed overall length (data quality issue)"

**Fix**: Correct the length measurements (waterline length ≤ overall length).

---

## Integration with Data Processing

### Validation Workflow

1. **Ingest Raw Data**: Load data from sources (NVR, WDQS, CSV)
2. **Transform to RDF**: Convert to RDF using ontology
3. **SHACL Validation**: Run SHACL validator against shapes
4. **Report Generation**: Generate validation report
5. **Data Cleaning**: Fix violations based on report
6. **Re-validation**: Re-run validation until all violations resolved
7. **Ingest to Graph**: Load validated data into knowledge graph

### Tools

- **SHACL Validators**:
  - TopBraid SHACL API
  - Apache Jena SHACL
  - pySHACL (Python)
  - SHACL Playground (online)

- **Data Cleaning Tools**:
  - OpenRefine (https://openrefine.org/)
  - Data cleaning scripts
  - Manual review for complex cases

---

## References

- **SHACL Specification**: https://www.w3.org/TR/shacl/
- **OpenRefine**: https://openrefine.org/
- **SKOS Reference**: https://www.w3.org/TR/skos-reference/
- **OWL 2 Overview**: https://www.w3.org/TR/owl2-overview/
- **Navy Vessel Register**: https://www.nvr.navy.mil

---

## Change Log

- **2024-01-15**: Initial data quality rules document created
  - Defined severity levels
  - Documented core validation rules
  - Created identifier pattern rules
  - Established controlled vocabulary validation
  - Added logical constraints
  - Created validation report structure
