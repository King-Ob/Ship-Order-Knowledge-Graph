# Demo Script - Ship Order Knowledge Graph

## Overview
This document provides a presentation script for demonstrating the Ship Order Knowledge Graph using SPARQL queries. The queries are designed to be "presentation-ready" and showcase key capabilities of the knowledge graph.

**Query File**: `queries/demo_queries.sparql` (60 queries)  
**Reference**: [Wikidata Query Service User Manual](https://www.mediawiki.org/wiki/Wikidata_Query_Service/User_Manual)

**Total Queries**: 60 presentation-ready SPARQL queries covering:
- Active ships (by navy, type, homeport, propulsion)
- Class comparisons (length, displacement, crew, beam, draft, speed)
- Conflict reports (displacement, dates, length, crew, name, class, status, homeport)
- Lifecycle timelines (single ship, class, events, refits, renames, reclassifications)
- Provenance queries (claims, confidence, sources, activities)
- Data quality (completeness, missing fields)
- Temporal analysis (commission/decommission trends, service duration)
- Aggregations (fleet totals, rankings)

---

## Prerequisites

> **⚠️ New to this project?** Start with the [Setup Guide](setup_guide.md) for complete step-by-step installation instructions.

### SPARQL Endpoint
- Local SPARQL endpoint (e.g., Apache Jena Fuseki, Blazegraph)
- Or remote endpoint with knowledge graph data loaded
- **Setup instructions**: See [Setup Guide - Step 3](setup_guide.md#step-3-set-up-fuseki-server)

### Data Loaded
- Core ontology (`ontology/naval_core.owl.ttl`)
- Events ontology (`ontology/naval_events.owl.ttl`)
- Provenance ontology (`ontology/naval_prov.owl.ttl`)
- Ship data (from RML mappings)
- Event data
- Provenance/claim data
- **Loading instructions**: See [Setup Guide - Step 4](setup_guide.md#step-4-load-the-data)

### Tools
- SPARQL query interface (web UI or command-line)
- Results export capability (CSV, JSON, HTML)
- **Access**: http://localhost:3030 (after starting Fuseki)

---

## Presentation Flow

### 1. Introduction (2 minutes)

**Slide**: "Ship Order Knowledge Graph - Demo"

**Talking Points**:
- "Today I'll demonstrate our knowledge graph for naval vessels"
- "We've integrated data from multiple sources: NVR (Navy Vessel Register), Wikidata, and messy CSV files"
- "Every fact is tracked with full provenance - we know where it came from, when, and how confident we are"
- "Let's start with some basic queries"

---

### 2. Active Ships Overview (3 minutes)

**Query**: Query 1 - Active Ships by Navy

**Demonstration**:
1. Run query showing active ships grouped by operator
2. Point out: "We can see ships from US Navy, Royal Navy, etc."
3. Show: "Each ship has its class, homeport, and hull symbol"

**Talking Points**:
- "Our first query shows all active ships organized by navy"
- "Notice we have data from multiple navies - US Navy, Royal Navy"
- "Each ship includes its class, homeport assignment, and official hull symbol"

**Query**: Query 2 - Active Ships by Type

**Demonstration**:
1. Run query showing distribution of ship types
2. Point out: "Aircraft carriers, destroyers, cruisers, etc."
3. Show counts for each type

**Talking Points**:
- "Here we see the distribution of ship types in our active fleet"
- "Aircraft carriers, destroyers, cruisers, and submarines"
- "This gives us a quick overview of fleet composition"

**Query**: Query 3 - Active Ships by Homeport

**Demonstration**:
1. Run query showing geographic distribution
2. Point out: "San Diego, Norfolk, Portsmouth, etc."
3. Show ship counts per homeport

**Talking Points**:
- "This query shows where our ships are based"
- "We can see the geographic distribution of the fleet"
- "Useful for operational planning and logistics"

---

### 3. Class Comparisons (4 minutes)

**Query**: Query 4 - Class Comparison: Length Ranges

**Demonstration**:
1. Run query showing length statistics per class
2. Point out: "Nimitz-class carriers are the longest"
3. Show min, max, and average lengths

**Talking Points**:
- "Now let's compare ship classes by their dimensions"
- "Here we see length ranges - Nimitz-class carriers are over 330 meters"
- "Destroyers are around 150 meters, submarines around 110 meters"

**Query**: Query 5 - Class Comparison: Displacement Ranges

**Demonstration**:
1. Run query showing displacement statistics
2. Point out: "Aircraft carriers have the highest displacement"
3. Compare different classes

**Talking Points**:
- "Displacement tells us about the size and capacity of ships"
- "Aircraft carriers displace over 100,000 tons"
- "Destroyers are around 9,000 tons"

**Query**: Query 6 - Class Comparison: Crew Complement

**Demonstration**:
1. Run query showing crew size ranges
2. Point out: "Carriers need large crews, submarines are smaller"
3. Show crew requirements

**Talking Points**:
- "Crew complement varies significantly by ship type"
- "Aircraft carriers need over 3,000 crew members"
- "Submarines operate with much smaller crews"

---

### 4. Data Quality and Conflicts (5 minutes)

**Query**: Query 7 - Conflict Report: Displacement Conflicts

**Demonstration**:
1. Run query showing conflicting displacement values
2. Point out: "Same ship, different values from different sources"
3. Show confidence scores

**Talking Points**:
- "One of our key features is handling conflicting data"
- "Here we see where different sources disagree on displacement"
- "Notice we keep BOTH values - we don't overwrite"
- "Each has a confidence score - NVR is authoritative for US Navy ships"

**Query**: Query 8 - Conflict Report: Date Conflicts

**Demonstration**:
1. Run query showing conflicting commission dates
2. Point out: "Temporal data conflicts"
3. Show source attribution

**Talking Points**:
- "We also track conflicts in dates"
- "Sometimes sources have slightly different dates"
- "We preserve both and mark which is preferred based on precedence rules"

**Query**: Query 9 - Conflict Report: All Conflicts Summary

**Demonstration**:
1. Run query showing summary of all conflicts
2. Point out: "Overview of data quality issues"
3. Show which properties have most conflicts

**Talking Points**:
- "This gives us an overview of all data quality issues"
- "We can see which properties have the most conflicts"
- "This helps us prioritize data cleaning efforts"

---

### 5. Lifecycle Timelines (4 minutes)

**Query**: Query 10 - Lifecycle Timeline: Single Ship

**Demonstration**:
1. Run query for USS Nimitz (CVN-68)
2. Show chronological events: Launch → Commission → Refits
3. Point out event types and dates

**Talking Points**:
- "Let's look at the complete lifecycle of a ship"
- "Here's USS Nimitz - launched in 1972, commissioned in 1975"
- "We can see refits, deployments, and other events"
- "This gives us a complete history"

**Query**: Query 11 - Lifecycle Timeline: Ship Class

**Demonstration**:
1. Run query for Nimitz-class carriers
2. Show all ships in the class with their events
3. Compare timelines

**Talking Points**:
- "We can also compare lifecycles across a ship class"
- "Here are all Nimitz-class carriers"
- "We can see when each was launched, commissioned, and any refits"
- "This helps us understand class-wide patterns"

---

### 6. Provenance and Trust (4 minutes)

**Query**: Query 15 - Provenance Chain: Why We Believe a Value

**Demonstration**:
1. Run query for displacement of CVN-68
2. Show full provenance: source, activity, timestamp, confidence
3. Point out: "We can answer 'Why do we believe this?'"

**Talking Points**:
- "One of our most powerful features is full provenance tracking"
- "For any fact, we can show exactly where it came from"
- "Here's why we believe the displacement is 104,600 tons"
- "It came from NVR, extracted via RML mapping on January 15th"
- "Confidence score is 0.95 - very high because NVR is authoritative"

**Query**: Query 13 - Source Coverage Report

**Demonstration**:
1. Run query showing data coverage by source
2. Show: "How many ships have data from each source"
3. Show average confidence scores

**Talking Points**:
- "This shows our data coverage"
- "How many ships have data from NVR, from Wikidata"
- "Average confidence scores tell us about overall data quality"

**Query**: Query 14 - Ships with Multiple Sources

**Demonstration**:
1. Run query showing ships with data from multiple sources
2. Point out: "Data integration success"
3. Show source combinations

**Talking Points**:
- "These are ships where we successfully integrated multiple sources"
- "We have NVR data AND Wikidata data"
- "This gives us richer, more complete information"

---

### 7. Advanced Analytics (5 minutes)

**Query**: Query 17 - Ships Commissioned by Decade

**Demonstration**:
1. Run query showing fleet growth over time
2. Show: "Historical trends"
3. Point out peak decades

**Talking Points**:
- "We can analyze historical trends"
- "Here's when ships were commissioned by decade"
- "We can see fleet growth patterns over time"

**Query**: Query 18 - Largest Ships by Displacement

**Demonstration**:
1. Run query showing top 10 largest ships
2. Show: "Ranking and comparison"
3. Point out size differences

**Talking Points**:
- "We can rank and compare ships"
- "Here are the 10 largest ships by displacement"
- "Aircraft carriers dominate the list"

**Query**: Query 20 - Confidence Score Distribution

**Demonstration**:
1. Run query showing confidence score statistics
2. Show: "Data quality metrics"
3. Point out: "Most claims have high confidence"

**Talking Points**:
- "Finally, let's look at overall data quality"
- "Most of our claims have high confidence scores"
- "This shows we're maintaining good data quality"

**Additional Queries Available**:
- Queries 21-60 cover additional scenarios:
  - Active ships by propulsion type
  - Class comparisons (beam, draft, speed)
  - Additional conflict reports (length, crew, name, class, status, homeport)
  - Detailed lifecycle timelines (commission, decommission, launch, refits, renames, reclassifications)
  - Provenance analysis (all claims, confidence levels, source authority, extraction activities)
  - Data quality checks (missing fields, incomplete specs)
  - Temporal analysis (service duration, active years)
  - Cross-source comparisons (NVR vs WDQS)
  - Aggregations (total fleet displacement, crew capacity)
  - Rankings (fastest, longest, largest crew)

---

## Key Messages

### 1. Multi-Source Integration
- "We successfully integrate data from multiple sources"
- "NVR for authoritative US Navy data"
- "Wikidata for international coverage and technical specs"
- "All sources are preserved with full provenance"

### 2. Conflict Handling
- "We don't overwrite conflicting data"
- "Multiple values are preserved"
- "Precedence rules determine preferred values"
- "But all claims remain accessible"

### 3. Full Provenance
- "Every fact has full provenance"
- "We can answer 'Why do we believe this?'"
- "Source, activity, timestamp, and confidence for every claim"
- "Full traceability and accountability"

### 4. Rich Querying
- "Complex queries across multiple dimensions"
- "By navy, type, homeport, class, time"
- "Lifecycle timelines, comparisons, conflicts"
- "All queryable with standard SPARQL"

---

## Presentation Tips

### Before the Demo

1. **Test All Queries**: Run each query beforehand to ensure they work
2. **Prepare Results**: Have example results ready in case of issues
3. **Check Data**: Verify data is loaded and accessible
4. **Backup Plan**: Have screenshots or pre-generated results as backup

### During the Demo

1. **Start Simple**: Begin with basic queries, build to complex ones
2. **Explain Results**: Don't just show results, explain what they mean
3. **Highlight Features**: Point out provenance, conflicts, timelines
4. **Engage Audience**: Ask questions, invite discussion

### After the Demo

1. **Q&A**: Be prepared for questions about:
   - Data sources and authority
   - Conflict resolution strategies
   - Confidence score calculation
   - Query performance
   - Data quality metrics

---

## Query Execution Guide

### Using SPARQL Endpoint (Web UI)

1. Open SPARQL endpoint in browser
2. Copy query from `queries/demo_queries.sparql`
3. Paste into query editor
4. Click "Run Query"
5. View results in table format
6. Export as CSV/JSON if needed

### Using Command Line

```bash
# Using curl
curl -X POST \
  -H "Content-Type: application/sparql-query" \
  -H "Accept: application/sparql-results+json" \
  --data-binary @queries/demo_queries.sparql \
  http://localhost:3030/ds/query

# Using rapper (for validation)
rapper -i turtle -o turtle output.ttl
```

### Using Python (rdflib)

```python
from rdflib import Graph, Namespace
from rdflib.plugins.sparql import prepareQuery

# Load graph
g = Graph()
g.parse("output/merged_output.ttl", format="turtle")

# Prepare query
query = prepareQuery("""
    SELECT ?shipName ?hullSymbol
    WHERE {
        ?ship a ship:Ship ;
            ship:shipName ?shipName ;
            ship:hullSymbol ?hullSymbol .
    }
""")

# Execute
results = g.query(query)
for row in results:
    print(row.shipName, row.hullSymbol)
```

---

## Expected Results Summary

### Query 1: Active Ships by Navy
- **Expected**: 10-15 ships grouped by operator
- **Key Insight**: Multi-navy coverage

### Query 2: Active Ships by Type
- **Expected**: 4-6 ship types with counts
- **Key Insight**: Fleet composition

### Query 4-6: Class Comparisons
- **Expected**: Statistics for 5-8 ship classes
- **Key Insight**: Size and capacity differences

### Query 7-9: Conflict Reports
- **Expected**: 2-5 conflicts identified
- **Key Insight**: Data quality transparency

### Query 10-11: Lifecycle Timelines
- **Expected**: 5-10 events per ship/class
- **Key Insight**: Complete historical records

### Query 13-15: Provenance Queries
- **Expected**: Full provenance chains
- **Key Insight**: Traceability and trust

---

## Troubleshooting

### Query Returns No Results

1. **Check Data Loading**: Verify data is loaded into endpoint
2. **Check Prefixes**: Ensure all prefixes are defined
3. **Check URIs**: Verify entity URIs match your data
4. **Check Syntax**: Validate SPARQL syntax

### Query Returns Partial Results

1. **Check Optional Clauses**: Some data may be missing
2. **Check Filters**: Filters may be too restrictive
3. **Check Data Quality**: Some properties may not be populated

### Performance Issues

1. **Add Limits**: Use `LIMIT` clause for large result sets
2. **Optimize Queries**: Add indexes, simplify patterns
3. **Use Subqueries**: Break complex queries into parts

---

## References

- **Wikidata Query Service User Manual**: https://www.mediawiki.org/wiki/Wikidata_Query_Service/User_Manual
- **SPARQL 1.1 Query Language**: https://www.w3.org/TR/sparql11-query/
- **SPARQL Tutorial**: https://www.w3.org/TR/sparql11-query/#tutorial

---

## Change Log

- **2024-01-15**: Initial demo script created
  - Created 20 presentation-ready queries
  - Documented presentation flow
  - Added execution guide
  - Created troubleshooting section
