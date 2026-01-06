# Running RML Mappings - ETL Documentation

## Overview
This document describes how to run RML (R2RML Mapping Language) mappings to transform raw data sources into RDF triples for the Ship Order Knowledge Graph.

**Mapping Files**:
- `mappings/nvr.rml.ttl` - Navy Vessel Register (NVR) JSON mapping
- `mappings/wikidata_extract.rml.ttl` - Wikidata Query Service (WDQS) JSON mapping

**Reference**: RML Specification (https://rml.io/specs/rml/)

---

## Prerequisites

### Required Tools

1. **RML Processor** (choose one):
   - **RMLMapper** (Java): https://github.com/RMLio/rmlmapper-java
   - **RMLStreamer** (Python): https://github.com/RMLio/rmlstreamer
   - **SDM-RDFizer** (Python): https://github.com/SDM-TIB/SDM-RDFizer
   - **Morph-KGC** (Python): https://github.com/oeg-upm/morph-kgc

2. **Java Runtime** (for RMLMapper):
   - Java 8 or higher
   - Download: https://adoptium.net/

3. **Python** (for RMLStreamer/Morph-KGC):
   - Python 3.7 or higher
   - pip package manager

### Data Sources

Ensure the following files exist:
- `data/raw/nvr_export.json`
- `data/raw/wdqs_export.json`

### Ontology Files

Ensure ontology files are available:
- `ontology/naval_core.owl.ttl`
- `ontology/naval_events.owl.ttl` (for events)

---

## Running Mappings

### Option 1: Using RMLMapper (Java)

#### Installation

```bash
# Download RMLMapper
wget https://github.com/RMLio/rmlmapper-java/releases/download/v6.0.0/rmlmapper-6.0.0-r367-all.jar

# Or use Maven
mvn install -DskipTests
```

#### Run NVR Mapping

```bash
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/nvr.rml.ttl \
    -o output/nvr_output.ttl \
    -s turtle
```

#### Run WDQS Mapping

```bash
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/wikidata_extract.rml.ttl \
    -o output/wdqs_output.ttl \
    -s turtle
```

#### Run Both Mappings

```bash
# Create output directory
mkdir -p output

# Run NVR mapping
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/nvr.rml.ttl \
    -o output/nvr_output.ttl \
    -s turtle

# Run WDQS mapping
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/wikidata_extract.rml.ttl \
    -o output/wdqs_output.ttl \
    -s turtle

# Merge outputs (optional)
cat output/nvr_output.ttl output/wdqs_output.ttl > output/merged_output.ttl
```

### Option 2: Using Morph-KGC (Python)

#### Installation

```bash
pip install morph-kgc
```

#### Run NVR Mapping

```python
from morph_kkg import Materialize

# Materialize RDF from NVR mapping
Materialize(
    mapping_path="mappings/nvr.rml.ttl",
    output_path="output/nvr_output.ttl",
    output_format="ttl"
)
```

#### Run WDQS Mapping

```python
from morph_kkg import Materialize

# Materialize RDF from WDQS mapping
Materialize(
    mapping_path="mappings/wikidata_extract.rml.ttl",
    output_path="output/wdqs_output.ttl",
    output_format="ttl"
)
```

#### Command Line Usage

```bash
# Run NVR mapping
morph-kgc mappings/nvr.rml.ttl -o output/nvr_output.ttl

# Run WDQS mapping
morph-kgc mappings/wikidata_extract.rml.ttl -o output/wdqs_output.ttl
```

### Option 3: Using RMLStreamer (Python)

#### Installation

```bash
pip install rmlstreamer
```

#### Run Mappings

```bash
# Run NVR mapping
rmlstreamer mappings/nvr.rml.ttl -o output/nvr_output.ttl

# Run WDQS mapping
rmlstreamer mappings/wikidata_extract.rml.ttl -o output/wdqs_output.ttl
```

---

## Deterministic Execution

### Ensuring Reproducibility

To ensure mappings produce the same triples on each run:

1. **Fixed Source Files**: Use versioned source files
   ```bash
   # Use specific versions
   data/raw/nvr_export_2024-01-15.json
   data/raw/wdqs_export_2024-01-14.json
   ```

2. **Fixed Mapping Files**: Use versioned mapping files
   ```bash
   mappings/nvr.rml.v1.0.ttl
   mappings/wikidata_extract.rml.v1.0.ttl
   ```

3. **Fixed Processor Version**: Use specific RML processor version
   ```bash
   rmlmapper-6.0.0-r367-all.jar  # Specific version
   ```

4. **Deterministic Sorting**: Sort output for comparison
   ```bash
   # Sort triples for deterministic output
   sort output/nvr_output.ttl > output/nvr_output_sorted.ttl
   ```

### Verification

```bash
# Generate checksum for output
md5sum output/nvr_output.ttl > output/nvr_output.md5

# Verify on subsequent runs
md5sum -c output/nvr_output.md5
```

---

## Provenance Tracking

### Source Attribution

Each triple includes provenance information:

1. **Source URI**: `dct:source` property
   - NVR: `https://www.navsea.navy.mil`
   - WDQS: `https://query.wikidata.org/sparql`

2. **Extraction Date**: `prov:generatedAtTime` property
   - NVR: `2024-01-15T10:30:00Z`
   - WDQS: `2024-01-14T08:15:00Z`

3. **Source Row Reference**: Tracked via mapping context

### Querying Provenance

```sparql
# Find all triples from NVR
SELECT ?s ?p ?o ?source ?date
WHERE {
    ?s ?p ?o .
    ?s dct:source ?source .
    ?s prov:generatedAtTime ?date .
    FILTER (?source = <https://www.navsea.navy.mil>)
}

# Find all triples from WDQS
SELECT ?s ?p ?o ?source ?date
WHERE {
    ?s ?p ?o .
    ?s dct:source ?source .
    ?s prov:generatedAtTime ?date .
    FILTER (?source = <https://query.wikidata.org/sparql>)
}
```

### Source Row Mapping

To track which source row produced which triple:

1. **Add Row Identifier**: Include row index or ID in mapping
2. **Store Mapping Log**: Log mapping execution with row-to-triple mapping
3. **Query Mapping**: Use SPARQL to trace triples back to source rows

Example mapping log format:
```json
{
  "mapping": "nvr.rml.ttl",
  "source_file": "data/raw/nvr_export.json",
  "source_row": 0,
  "triples": [
    "ship-entity:ship/CVN-68 ship:shipName \"USS Nimitz\"",
    "ship-entity:ship/CVN-68 ship:hullSymbol \"CVN-68\""
  ]
}
```

---

## Output Structure

### Expected Output Files

```
output/
├── nvr_output.ttl          # NVR RDF triples
├── wdqs_output.ttl        # WDQS RDF triples
├── merged_output.ttl      # Combined triples
└── mapping_log.json       # Mapping execution log
```

### Output Format

Triples are generated in Turtle (TTL) format:

```turtle
ship-entity:ship/CVN-68 a ship:Ship ;
    ship:shipName "USS Nimitz" ;
    ship:hullSymbol "CVN-68" ;
    ship:status "Active" ;
    ship:commissionDate "1975-05-03"^^xsd:date ;
    ship:hasClass ship-entity:class/nimitz ;
    ship:operatedBy ship-entity:operator/USN ;
    ship:homeportedAt ship-entity:homeport/bremerton-wa-us ;
    dct:source <https://www.navsea.navy.mil> ;
    prov:generatedAtTime "2024-01-15T10:30:00Z"^^xsd:dateTime .
```

---

## Troubleshooting

### Common Issues

1. **Missing Source Files**
   ```
   Error: Cannot find source file data/raw/nvr_export.json
   ```
   **Fix**: Ensure source files exist in `data/raw/` directory

2. **Invalid JSON Path**
   ```
   Error: Invalid JSONPath expression
   ```
   **Fix**: Verify JSON structure matches mapping expectations

3. **URI Template Errors**
   ```
   Error: Invalid URI template
   ```
   **Fix**: Check URI templates in mapping file, ensure proper escaping

4. **Missing Normalization Functions**
   ```
   Warning: Normalization function not found
   ```
   **Fix**: Implement normalization functions or preprocess data

### Debug Mode

Enable verbose logging:

```bash
# RMLMapper with debug
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/nvr.rml.ttl \
    -o output/nvr_output.ttl \
    -s turtle \
    -v

# Morph-KGC with debug
morph-kgc mappings/nvr.rml.ttl -o output/nvr_output.ttl --verbose
```

---

## Normalization Functions

### Required Functions

The mappings reference normalization functions that need to be implemented:

1. **`<#normalizeClassName>`**: Normalizes class names
   - Input: `"Nimitz-class aircraft carrier"`
   - Output: `"nimitz"`

2. **`<#normalizeOperator>`**: Normalizes operator names
   - Input: `"United States Navy"`
   - Output: `"USN"`

3. **`<#normalizeHomeport>`**: Normalizes homeport names
   - Input: `"Naval Base San Diego, CA"`
   - Output: `"san-diego-ca-us"`

### Implementation Options

1. **Preprocessing**: Normalize data before mapping
2. **RML Functions**: Implement as RML functions in processor
3. **Post-processing**: Normalize URIs after mapping

### Preprocessing Script Example

```python
import json
import re

def normalize_class_name(class_name):
    # Remove "-class" suffix and extra words
    normalized = class_name.lower()
    normalized = re.sub(r'-class.*$', '', normalized)
    normalized = re.sub(r'\s+', '-', normalized)
    return normalized

def normalize_operator(operator):
    operator_map = {
        "United States Navy": "USN",
        "Royal Navy": "RN",
        # ... more mappings
    }
    return operator_map.get(operator, operator.lower().replace(' ', '-'))

# Preprocess NVR data
with open('data/raw/nvr_export.json', 'r') as f:
    data = json.load(f)

for vessel in data['vessels']:
    vessel['class_normalized'] = normalize_class_name(vessel['class'])
    vessel['operator_normalized'] = normalize_operator(vessel['operator'])

with open('data/raw/nvr_export_normalized.json', 'w') as f:
    json.dump(data, f, indent=2)
```

---

## Validation After Mapping

### Validate Output

1. **Syntax Validation**: Check RDF syntax
   ```bash
   # Using rapper (from Redland)
   rapper -i turtle -o turtle output/nvr_output.ttl
   ```

2. **SHACL Validation**: Validate against SHACL shapes
   ```bash
   # Using pySHACL
   pyshacl -s shapes/ship_core.shacl.ttl output/nvr_output.ttl
   ```

3. **SPARQL Validation**: Query for data quality issues
   ```sparql
   # Find ships without required fields
   SELECT ?ship ?name
   WHERE {
       ?ship a ship:Ship .
       OPTIONAL { ?ship ship:shipName ?name . }
       FILTER (!BOUND(?name))
   }
   ```

---

## Automation

### Batch Processing Script

```bash
#!/bin/bash
# run_all_mappings.sh

# Create output directory
mkdir -p output

# Run NVR mapping
echo "Running NVR mapping..."
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/nvr.rml.ttl \
    -o output/nvr_output.ttl \
    -s turtle

# Run WDQS mapping
echo "Running WDQS mapping..."
java -jar rmlmapper-6.0.0-r367-all.jar \
    -m mappings/wikidata_extract.rml.ttl \
    -o output/wdqs_output.ttl \
    -s turtle

# Merge outputs
echo "Merging outputs..."
cat output/nvr_output.ttl output/wdqs_output.ttl > output/merged_output.ttl

# Validate
echo "Validating output..."
pyshacl -s shapes/ship_core.shacl.ttl output/merged_output.ttl

echo "Done!"
```

### Makefile

```makefile
# Makefile for running mappings

RMLMAPPER = rmlmapper-6.0.0-r367-all.jar
OUTPUT_DIR = output
MAPPINGS_DIR = mappings

.PHONY: all nvr wdqs merge validate clean

all: nvr wdqs merge validate

nvr:
	@echo "Running NVR mapping..."
	@mkdir -p $(OUTPUT_DIR)
	java -jar $(RMLMAPPER) -m $(MAPPINGS_DIR)/nvr.rml.ttl -o $(OUTPUT_DIR)/nvr_output.ttl -s turtle

wdqs:
	@echo "Running WDQS mapping..."
	@mkdir -p $(OUTPUT_DIR)
	java -jar $(RMLMAPPER) -m $(MAPPINGS_DIR)/wikidata_extract.rml.ttl -o $(OUTPUT_DIR)/wdqs_output.ttl -s turtle

merge: nvr wdqs
	@echo "Merging outputs..."
	cat $(OUTPUT_DIR)/nvr_output.ttl $(OUTPUT_DIR)/wdqs_output.ttl > $(OUTPUT_DIR)/merged_output.ttl

validate: merge
	@echo "Validating output..."
	pyshacl -s shapes/ship_core.shacl.ttl $(OUTPUT_DIR)/merged_output.ttl

clean:
	rm -rf $(OUTPUT_DIR)

# Usage: make all
```

---

## References

- **RML Specification**: https://rml.io/specs/rml/
- **RMLMapper**: https://github.com/RMLio/rmlmapper-java
- **Morph-KGC**: https://github.com/oeg-upm/morph-kgc
- **RMLStreamer**: https://github.com/RMLio/rmlstreamer
- **WDQS Manual**: https://www.mediawiki.org/wiki/Wikidata_Query_Service
- **R2RML Specification**: https://www.w3.org/TR/r2rml/

---

## Change Log

- **2024-01-15**: Initial ETL documentation created
  - Documented RML mapping execution
  - Added provenance tracking information
  - Created troubleshooting guide
  - Added automation scripts
