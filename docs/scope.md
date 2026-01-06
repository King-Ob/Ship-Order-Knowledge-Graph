# Ship Order Knowledge Graph - Project Scope

## Overview
This project aims to build a comprehensive knowledge graph for naval vessels, integrating data from multiple authoritative and public sources to enable complex queries about ship classifications, identities, lifecycles, technical specifications, and cross-references.

## Data Sources

### 1. Navy Vessel Register (NVR) - Authoritative Source
**Source**: navsea.navy.mil (Naval Sea Systems Command)

**Why NVR is Authoritative for US Inventory/Lifecycle**:
- **Official Government Record**: The NVR is maintained by the Naval Sea Systems Command (NAVSEA), the official authority responsible for the technical and engineering oversight of all US Navy ships and systems.
- **Legal and Administrative Authority**: NAVSEA is the designated authority for maintaining the official inventory of all US Navy vessels, including their hull classification symbols, names, and status.
- **Lifecycle Management**: NVR provides authoritative information on:
  - Commissioning dates (official entry into service)
  - Decommissioning dates (official removal from active service)
  - Status changes (Active, Reserve, Museum, etc.)
  - Reclassification events (official changes to ship type/class)
  - Homeport assignments (official operational assignments)
- **Real-time Updates**: As the official administrative system, NVR is updated in real-time as ships are commissioned, decommissioned, or change status, making it the most current source for US Navy vessel lifecycle information.
- **Standardized Classification**: NVR uses the official US Navy Hull Classification Symbol system, which is the standard reference for all US naval vessels.

**Use Cases**:
- Official ship status (Active, Reserve, Decommissioned, Museum)
- Commissioning/decommissioning dates
- Hull classification symbols
- Homeport assignments
- Official ship names and name changes

### 2. Wikidata Query Service (WDQS) - Public Knowledge Base
**Source**: wikidata.org (via SPARQL endpoint)

**Purpose**: Provides international coverage, cross-references, and additional technical specifications from a community-maintained knowledge base.

**Strengths**:
- International vessel coverage (beyond US Navy)
- MMSI and IMO numbers
- Cross-references to other databases
- Technical specifications from various sources
- Historical information

**Limitations**:
- Community-maintained (may have inconsistencies)
- May not reflect real-time status changes
- Less authoritative for official US Navy status

**Use Cases**:
- International vessel identification
- MMSI/IMO cross-referencing
- Technical specifications comparison
- Historical data enrichment

### 3. Messy Table/CSV - Real-world Data Challenge
**Purpose**: Represents real-world data integration challenges with inconsistent formatting, missing fields, and data quality issues.

**Characteristics**:
- Inconsistent date formats
- Missing or incomplete fields
- Mixed naming conventions
- Duplicate entries
- Data quality issues requiring cleaning and normalization

**Use Cases**:
- Data quality assessment
- Data cleaning and normalization workflows
- Conflict resolution strategies
- Provenance tracking for uncertain data

## Knowledge Graph Objectives

### Structural/Classification
- Establish SKOS taxonomy for ship types and classifications
- Link OWL classes correctly (e.g., "surface combatant" → specific ship types)
- Maintain hierarchical relationships (classes, types, categories)

### Identity and Cross-Reference
- Link ships across different data sources using multiple identifiers:
  - Hull Classification Symbols (NVR)
  - Pennant Numbers
  - MMSI (Maritime Mobile Service Identity)
  - IMO (International Maritime Organization) numbers
- Resolve identity conflicts and duplicates

### Temporal and Lifecycle
- Track ship lifecycle events:
  - Launch → Commission → Active Service → Refit → Decommission → Museum/Scrapping
- Handle name changes and reclassifications
- Maintain temporal relationships and timelines

### Technical Specifications
- Store and compare technical specifications:
  - Displacement (light, standard, full load)
  - Dimensions (length, beam, draft)
  - Propulsion type
  - Crew complement
  - Speed, range, etc.
- Identify and flag data quality issues (e.g., waterline length > overall length)

### Provenance and Conflict Resolution
- Track data provenance (source URL, extraction date, confidence scores)
- Identify conflicts between sources
- Enable queries about data quality and source reliability
- Support confidence scoring for uncertain data

## Query Capabilities

The knowledge graph will support queries across:
1. **Structural queries**: Ship types, classifications, fleet assignments
2. **Identity queries**: Cross-referencing ships across sources
3. **Temporal queries**: Lifecycle events, status changes over time
4. **Technical queries**: Specifications, comparisons, aggregations
5. **Provenance queries**: Source tracking, conflict identification, data quality

## Success Criteria

- All 20 competency questions can be answered with traceable queries
- Data from all three sources is integrated and cross-referenced
- Provenance is maintained for all facts
- Conflicts between sources are identified and documented
- The knowledge graph supports complex multi-hop queries
