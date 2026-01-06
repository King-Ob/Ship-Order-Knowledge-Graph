# Ship Order Knowledge Graph - Competency Questions

**Purpose**: These questions define the functional requirements for our knowledge graph build. Each question is traceable to specific data sources and queries.

---

## Structural/Classification Questions
*These ensure our SKOS taxonomy and OWL classes are correctly linked.*

1. **What are all the specific ship types that fall under the broader category of "surface combatant"?**
   - **Data Sources**: NVR (hull classification), WDQS (ship type taxonomy)
   - **Query Type**: Hierarchical classification query
   - **Expected Result**: List of ship types (e.g., Destroyer, Cruiser, Frigate) that are subclasses of "surface combatant"

2. **List all ships currently assigned to the 'U.S. 7th Fleet'**
   - **Data Sources**: NVR (homeport/fleet assignment)
   - **Query Type**: Filter by fleet assignment
   - **Expected Result**: List of ships with homeport or operational assignment to 7th Fleet

3. **Which ships are classified as Nuclear powered (propulsion type) and 'Aircraft Carrier' (ship type)?**
   - **Data Sources**: NVR (hull classification CVN), WDQS (propulsion type)
   - **Query Type**: Multi-attribute filter (propulsion + ship type)
   - **Expected Result**: All nuclear-powered aircraft carriers (CVN class)

4. **Show all ships that are currently in 'Reserve' or 'Museum' status**
   - **Data Sources**: NVR (status field - authoritative)
   - **Query Type**: Status filter
   - **Expected Result**: Ships with status = "Reserve" OR status = "Museum"

---

## Identity and Cross-Reference Questions
*These test our ability to link ships across different data sources using multiple identifiers.*

5. **What is the MMSI and IMO number for the ship with hull symbol CVN-71?**
   - **Data Sources**: NVR (hull symbol) → WDQS (MMSI, IMO lookup)
   - **Query Type**: Cross-reference identity resolution
   - **Expected Result**: MMSI and IMO numbers for USS Theodore Roosevelt (CVN-71)

6. **Find a ship in the Wikidata source that matches the Pennant Number of a ship in the NVR source**
   - **Data Sources**: NVR (hull symbol/pennant), WDQS (pennant number)
   - **Query Type**: Identity matching across sources
   - **Expected Result**: Ships that appear in both sources with matching pennant numbers

7. **List all the ships that have more than one active Hull Classification Symbol across different databases**
   - **Data Sources**: NVR, WDQS, Messy CSV (hull symbols)
   - **Query Type**: Duplicate/conflict detection
   - **Expected Result**: Ships with conflicting or multiple hull symbols

8. **Which ships in our graph are missing an ITU-standard 9-digit MMSI?**
   - **Data Sources**: WDQS (MMSI field), NVR (ship list)
   - **Query Type**: Data completeness check
   - **Expected Result**: Ships without MMSI or with invalid MMSI format

---

## Temporal and Lifecycle Questions
*These validate our ability to track ship lifecycle events and temporal relationships.*

9. **What was the original name of the ship currently known as USS Theodore Roosevelt?**
   - **Data Sources**: NVR (name history - authoritative for US ships)
   - **Query Type**: Temporal name change query
   - **Expected Result**: Original name before renaming (if applicable)

10. **Which ships have been reclassified (e.g., from Destroyer to Guided Missile Destroyer) in the last 10 years?**
    - **Data Sources**: NVR (reclassification events with dates)
    - **Query Type**: Temporal filter on classification changes
    - **Expected Result**: Ships with reclassification events in the last 10 years

11. **Provide a timeline of events (Launch → Commission → Refit) for the Nimitz-class carriers**
    - **Data Sources**: NVR (commission dates), WDQS (launch dates, refit events)
    - **Query Type**: Temporal sequence query
    - **Expected Result**: Chronological timeline of lifecycle events for all Nimitz-class carriers

12. **Which ships were active in the year 1995 but are now 'Decommissioned'?**
    - **Data Sources**: NVR (commission date, decommission date, status)
    - **Query Type**: Temporal status query
    - **Expected Result**: Ships that were commissioned before 1995 and decommissioned after 1995

---

## Technical Specification and Comparison Questions
*These test our ability to store, query, and compare technical specifications.*

13. **What is the average Full Load Displacement of all ships in the Arleigh Burke class?**
    - **Data Sources**: WDQS (displacement specifications), NVR (ship class)
    - **Query Type**: Aggregation query
    - **Expected Result**: Average displacement value for DDG-51 class ships

14. **List all ships with a Length greater than 300 meters, ordered by their commissioning date**
    - **Data Sources**: WDQS (length specifications), NVR (commission dates)
    - **Query Type**: Filter + sort query
    - **Expected Result**: Ships > 300m length, sorted chronologically by commission date

15. **Which ship class has the largest complement (crew size) in the Royal Navy?**
    - **Data Sources**: WDQS (crew complement, operator = Royal Navy)
    - **Query Type**: Aggregation + filter by operator
    - **Expected Result**: Ship class with maximum crew size for Royal Navy vessels

16. **Find ships where the 'waterline length' is recorded as greater than the 'overall length'**
    - **Data Sources**: WDQS, Messy CSV (length measurements)
    - **Query Type**: Data quality validation query
    - **Expected Result**: Ships with logically inconsistent length measurements (data quality issue)

---

## Provenance and Conflict Questions
*These validate our provenance tracking and conflict resolution capabilities.*

17. **Which source provides a value for displacement of HMS Prince of Wales, and do they disagree?**
    - **Data Sources**: NVR (if applicable), WDQS, Messy CSV
    - **Query Type**: Provenance + conflict detection
    - **Expected Result**: All sources with displacement values for HMS Prince of Wales, with comparison showing conflicts

18. **Show the provenance (source URL + extraction date) for the 'Homeport' of hull LHA-6**
    - **Data Sources**: NVR (homeport - authoritative), provenance metadata
    - **Query Type**: Provenance trace query
    - **Expected Result**: Source URL, extraction date, and confidence score for LHA-6 homeport data

19. **Which dataset (NVR vs Wikidata) has the most recent update for Ship status?**
    - **Data Sources**: NVR (extraction date), WDQS (last modified date)
    - **Query Type**: Provenance comparison query
    - **Expected Result**: Comparison of last update dates for status information across sources

20. **List all facts in the graph that have a confidence score of less than 80%**
    - **Data Sources**: All sources (confidence scores in provenance metadata)
    - **Query Type**: Data quality filter
    - **Expected Result**: All triples/facts with confidence < 0.8, indicating uncertain or conflicting data

---

## Query Traceability

Each competency question maps to:
- **Specific data sources**: NVR, WDQS, or Messy CSV
- **Query patterns**: SPARQL/Cypher/Gremlin queries (depending on graph database)
- **Expected results**: Validated against source data
- **Provenance**: Source tracking for all returned facts

## Data Source Priority

For conflicts or missing data:
1. **NVR is authoritative** for: US Navy ship status, lifecycle events, hull symbols, homeports, official names
2. **WDQS is authoritative** for: International vessels, MMSI/IMO numbers, cross-references
3. **Messy CSV**: Requires data quality assessment and conflict resolution
