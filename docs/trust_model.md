# Trust Model and Provenance Framework

## Overview
This document describes the trust model and provenance framework for the Ship Order Knowledge Graph. Every fact is modeled as a **claim** with full provenance information, allowing us to track why we believe something and handle conflicting values from different sources.

**Reference**: [PROV-O: The PROV Ontology](https://www.w3.org/TR/prov-o/) (W3C Recommendation)

---

## Core Principles

### 1. Claim-Level Provenance
Every fact about a ship is modeled as a **claim** with:
- **Source Dataset**: Where the data came from (NVR, WDQS, CSV)
- **Extraction Activity**: How it was extracted (RML mapping, manual entry, API query)
- **Timestamp**: When it was extracted/created
- **Confidence Score**: How confident we are in the claim (0.0 to 1.0)

### 2. Multiple Claims, No Overwriting
- **Conflicting values are preserved**: Two sources can assert different values for the same property
- **Both claims stored**: Each claim maintains its own provenance
- **Precedence rules**: Preferred claims are marked, but all claims remain accessible

### 3. Full Traceability
- **Answer "Why?":** Can trace any fact back to its source
- **Answer "When?":** Timestamps for extraction and source data
- **Answer "How confident?":** Confidence scores for all claims

---

## Claim Model

### Claim Structure

A **claim** is a statement about a ship with full provenance:

```turtle
ship-entity:claim/displacement-CVN-68-NVR a ship:Claim ;
    ship:aboutShip ship-entity:ship/CVN-68 ;
    ship:assertsProperty ship:displacementFullLoad ;
    ship:claimValue "104600"^^xsd:decimal ;
    ship:hasSource ship-entity:source/NVR ;
    ship:extractedBy ship-entity:activity/RML_NVR ;
    ship:extractedAt "2024-01-15T10:30:00Z"^^xsd:dateTime ;
    ship:confidenceScore 0.95^^xsd:double ;
    prov:wasAttributedTo ship-entity:source/NVR ;
    prov:wasGeneratedBy ship-entity:activity/RML_NVR ;
    prov:generatedAtTime "2024-01-15T10:30:00Z"^^xsd:dateTime .
```

### Components

1. **Claim Entity** (`ship:Claim`)
   - Subclass of `prov:Entity`
   - Represents a single statement about a ship

2. **About Ship** (`ship:aboutShip`)
   - Links claim to the ship it describes
   - Example: `ship-entity:ship/CVN-68`

3. **Asserts Property** (`ship:assertsProperty`)
   - The property being asserted
   - Example: `ship:displacementFullLoad`

4. **Claim Value** (`ship:claimValue`)
   - The value being asserted
   - Can be literal (string, number, date) or IRI

5. **Source** (`ship:hasSource`)
   - The data source (NVR, WDQS, CSV)
   - Links to `ship:DataSource` (subclass of `prov:Agent`)

6. **Extraction Activity** (`ship:extractedBy`)
   - The activity that created the claim
   - Links to `ship:ExtractionActivity` (subclass of `prov:Activity`)

7. **Timestamp** (`ship:extractedAt`)
   - When the claim was extracted/created
   - Uses `xsd:dateTime`

8. **Confidence Score** (`ship:confidenceScore`)
   - Confidence in the claim (0.0 to 1.0)
   - Based on source authority, data quality, validation results

---

## Data Sources

### Source Properties

Each data source is modeled as a `ship:DataSource` (subclass of `prov:Agent`):

```turtle
ship-entity:source/NVR a ship:DataSource ;
    rdfs:label "Navy Vessel Register (NVR)"@en ;
    ship:sourceURL <https://www.navsea.navy.mil> ;
    ship:authorityLevel "authoritative" ;
    ship:sourceExtractedAt "2024-01-15T10:30:00Z"^^xsd:dateTime .
```

**Properties**:
- `ship:sourceURL`: URL or identifier of the source
- `ship:authorityLevel`: Authority level ("authoritative", "verified", "community", "unverified")
- `ship:sourceExtractedAt`: When data was extracted from source

### Authority Levels

1. **Authoritative** (NVR for US Navy)
   - Official, government-maintained sources
   - Highest confidence for their domain
   - Example: NVR for US Navy status, dates, hull symbols

2. **Verified** (WDQS)
   - Community-maintained but verified sources
   - Good confidence, may have inconsistencies
   - Example: WDQS for international vessels, technical specs

3. **Community** (User contributions)
   - Community-maintained sources
   - Moderate confidence, requires validation
   - Example: User-submitted data

4. **Unverified** (Messy CSV)
   - Unverified or low-quality sources
   - Low confidence, requires validation
   - Example: Messy CSV with data quality issues

---

## Extraction Activities

### Activity Properties

Each extraction activity is modeled as a `ship:ExtractionActivity` (subclass of `prov:Activity`):

```turtle
ship-entity:activity/RML_NVR a ship:ExtractionActivity ;
    rdfs:label "RML Mapping: NVR"@en ;
    ship:activityType "RML_Mapping" ;
    ship:activityConfig "mappings/nvr.rml.ttl" ;
    prov:wasAssociatedWith ship-entity:source/NVR ;
    prov:startedAtTime "2024-01-15T10:30:00Z"^^xsd:dateTime .
```

**Properties**:
- `ship:activityType`: Type of activity ("RML_Mapping", "Manual_Entry", "API_Query")
- `ship:activityConfig`: Configuration or parameters (mapping file, query string)
- `prov:wasAssociatedWith`: Associated data source
- `prov:startedAtTime`: When activity started

### Activity Types

1. **RML_Mapping**: RML/R2RML mapping transformation
2. **Manual_Entry**: Human-entered data
3. **API_Query**: API query result
4. **Entity_Resolution**: Entity resolution/linking activity
5. **Data_Cleaning**: Data cleaning/transformation activity

---

## Confidence Scoring

### Confidence Score Calculation

Confidence scores (0.0 to 1.0) are calculated based on:

1. **Source Authority** (40% weight)
   - Authoritative: 1.0
   - Verified: 0.8
   - Community: 0.6
   - Unverified: 0.4

2. **Data Quality** (30% weight)
   - Passes SHACL validation: 1.0
   - Warnings only: 0.8
   - Failures: 0.5

3. **Source Completeness** (20% weight)
   - Complete data: 1.0
   - Partial data: 0.7
   - Minimal data: 0.4

4. **Temporal Freshness** (10% weight)
   - Recent (< 1 month): 1.0
   - Recent (< 1 year): 0.8
   - Older (> 1 year): 0.6

### Example Calculation

```python
def calculate_confidence(source, data_quality, completeness, freshness):
    """Calculate confidence score for a claim."""
    authority_scores = {
        'authoritative': 1.0,
        'verified': 0.8,
        'community': 0.6,
        'unverified': 0.4
    }
    
    authority = authority_scores.get(source.authority_level, 0.5)
    quality = 1.0 if data_quality == 'pass' else 0.8 if data_quality == 'warn' else 0.5
    complete = 1.0 if completeness == 'complete' else 0.7 if completeness == 'partial' else 0.4
    fresh = 1.0 if freshness < 30 else 0.8 if freshness < 365 else 0.6
    
    confidence = (
        authority * 0.4 +
        quality * 0.3 +
        complete * 0.2 +
        fresh * 0.1
    )
    
    return confidence
```

---

## Conflict Resolution

### Handling Conflicting Claims

When multiple sources provide different values for the same property:

1. **Store All Claims**: All claims are preserved with full provenance
2. **Mark Conflicts**: Use `ship:conflictsWith` to link conflicting claims
3. **Mark Preferred**: Use `ship:isPreferred` to mark preferred claim based on precedence rules
4. **Query Options**: Queries can return all claims or just preferred ones

### Example: Conflicting Displacement Values

```turtle
# Claim 1: From NVR
ship-entity:claim/displacement-CVN-68-NVR a ship:Claim ;
    ship:aboutShip ship-entity:ship/CVN-68 ;
    ship:assertsProperty ship:displacementFullLoad ;
    ship:claimValue "104600"^^xsd:decimal ;
    ship:hasSource ship-entity:source/NVR ;
    ship:confidenceScore 0.95^^xsd:double ;
    ship:isPreferred true^^xsd:boolean .

# Claim 2: From WDQS (conflicting value)
ship-entity:claim/displacement-CVN-68-WDQS a ship:Claim ;
    ship:aboutShip ship-entity:ship/CVN-68 ;
    ship:assertsProperty ship:displacementFullLoad ;
    ship:claimValue "105000"^^xsd:decimal ;
    ship:hasSource ship-entity:source/WDQS ;
    ship:confidenceScore 0.80^^xsd:double ;
    ship:conflictsWith ship-entity:claim/displacement-CVN-68-NVR ;
    ship:isPreferred false^^xsd:boolean .
```

### Precedence Rules

1. **NVR is Preferred For**:
   - US Navy ship status
   - Commission/decommission dates
   - Hull symbols
   - Homeports
   - Official names

2. **WDQS is Preferred For**:
   - International vessels (non-US Navy)
   - MMSI/IMO numbers
   - Technical specifications (when NVR doesn't have them)

3. **Higher Confidence Wins**:
   - When sources have equal precedence, prefer higher confidence score

---

## Querying Provenance

### Answer: "Why do we believe this displacement/date?"

```sparql
# Query: Why do we believe the displacement of CVN-68 is 104600?

SELECT ?claim ?value ?source ?sourceURL ?authority ?confidence ?extractedAt ?activity
WHERE {
    ?claim a ship:Claim ;
        ship:aboutShip ship-entity:ship/CVN-68 ;
        ship:assertsProperty ship:displacementFullLoad ;
        ship:claimValue ?value ;
        ship:hasSource ?source ;
        ship:confidenceScore ?confidence ;
        ship:extractedAt ?extractedAt ;
        ship:extractedBy ?activity .
    
    ?source ship:sourceURL ?sourceURL ;
        ship:authorityLevel ?authority .
}
ORDER BY DESC(?confidence)
```

**Result**:
- Value: 104600
- Source: NVR (https://www.navsea.navy.mil)
- Authority: authoritative
- Confidence: 0.95
- Extracted: 2024-01-15T10:30:00Z
- Activity: RML Mapping: NVR

### Query: All Claims for a Property

```sparql
# Query: All displacement claims for CVN-68 (including conflicts)

SELECT ?claim ?value ?source ?confidence ?isPreferred ?conflictsWith
WHERE {
    ?claim a ship:Claim ;
        ship:aboutShip ship-entity:ship/CVN-68 ;
        ship:assertsProperty ship:displacementFullLoad ;
        ship:claimValue ?value ;
        ship:hasSource ?source ;
        ship:confidenceScore ?confidence .
    
    OPTIONAL { ?claim ship:isPreferred ?isPreferred . }
    OPTIONAL { ?claim ship:conflictsWith ?conflictsWith . }
}
ORDER BY DESC(?isPreferred) DESC(?confidence)
```

### Query: Conflicting Claims

```sparql
# Query: Find all conflicting claims

SELECT ?ship ?property ?claim1 ?value1 ?source1 ?claim2 ?value2 ?source2
WHERE {
    ?claim1 a ship:Claim ;
        ship:aboutShip ?ship ;
        ship:assertsProperty ?property ;
        ship:claimValue ?value1 ;
        ship:hasSource ?source1 ;
        ship:conflictsWith ?claim2 .
    
    ?claim2 ship:claimValue ?value2 ;
        ship:hasSource ?source2 .
    
    FILTER (?value1 != ?value2)
}
```

---

## Example: Complete Claim with Provenance

### Displacement Claim from NVR

```turtle
# Ship
ship-entity:ship/CVN-68 a ship:Ship ;
    ship:shipName "USS Nimitz" ;
    ship:hullSymbol "CVN-68" .

# Data Source
ship-entity:source/NVR a ship:DataSource ;
    rdfs:label "Navy Vessel Register (NVR)"@en ;
    ship:sourceURL <https://www.navsea.navy.mil> ;
    ship:authorityLevel "authoritative" ;
    ship:sourceExtractedAt "2024-01-15T10:30:00Z"^^xsd:dateTime .

# Extraction Activity
ship-entity:activity/RML_NVR a ship:ExtractionActivity ;
    rdfs:label "RML Mapping: NVR"@en ;
    ship:activityType "RML_Mapping" ;
    ship:activityConfig "mappings/nvr.rml.ttl" ;
    prov:wasAssociatedWith ship-entity:source/NVR ;
    prov:startedAtTime "2024-01-15T10:30:00Z"^^xsd:dateTime .

# Claim
ship-entity:claim/displacement-CVN-68-NVR a ship:Claim ;
    ship:aboutShip ship-entity:ship/CVN-68 ;
    ship:assertsProperty ship:displacementFullLoad ;
    ship:claimValue "104600"^^xsd:decimal ;
    ship:hasSource ship-entity:source/NVR ;
    ship:extractedBy ship-entity:activity/RML_NVR ;
    ship:extractedAt "2024-01-15T10:30:00Z"^^xsd:dateTime ;
    ship:confidenceScore 0.95^^xsd:double ;
    ship:isPreferred true^^xsd:boolean ;
    prov:wasAttributedTo ship-entity:source/NVR ;
    prov:wasGeneratedBy ship-entity:activity/RML_NVR ;
    prov:generatedAtTime "2024-01-15T10:30:00Z"^^xsd:dateTime .
```

### Commission Date Claim from NVR

```turtle
ship-entity:claim/commission-CVN-68-NVR a ship:Claim ;
    ship:aboutShip ship-entity:ship/CVN-68 ;
    ship:assertsProperty ship:commissionDate ;
    ship:claimValue "1975-05-03"^^xsd:date ;
    ship:hasSource ship-entity:source/NVR ;
    ship:extractedBy ship-entity:activity/RML_NVR ;
    ship:extractedAt "2024-01-15T10:30:00Z"^^xsd:dateTime ;
    ship:confidenceScore 0.98^^xsd:double ;
    ship:isPreferred true^^xsd:boolean .
```

---

## Trust Model Summary

### Key Features

1. **Claim-Level Provenance**: Every fact is a claim with full provenance
2. **Multiple Claims**: Conflicting values are preserved, not overwritten
3. **Confidence Scores**: Quantitative measure of trust (0.0 to 1.0)
4. **Source Authority**: Hierarchical authority levels
5. **Full Traceability**: Can answer "why", "when", "how confident"
6. **Conflict Resolution**: Precedence rules with all claims preserved

### Benefits

- **Transparency**: Full visibility into data sources and extraction methods
- **Accountability**: Can trace any fact back to its origin
- **Flexibility**: Can query preferred claims or all claims
- **Quality Assessment**: Confidence scores enable quality-based filtering
- **Conflict Handling**: Multiple values preserved without data loss

---

## References

- **PROV-O: The PROV Ontology**: https://www.w3.org/TR/prov-o/
- **PROV Data Model**: https://www.w3.org/TR/prov-dm/
- **PROV Primer**: https://www.w3.org/TR/prov-primer/

---

## Change Log

- **2024-01-15**: Initial trust model documentation created
  - Defined claim-level provenance model
  - Created confidence scoring framework
  - Documented conflict resolution strategy
  - Added provenance query examples
