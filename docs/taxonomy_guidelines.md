# Taxonomy Guidelines - Ship Order Knowledge Graph

## Overview
This document defines the naming conventions, synonym policies, and deprecation rules for the SKOS concept schemes used in the Ship Order Knowledge Graph project.

**Reference**: SKOS Reference (https://www.w3.org/TR/skos-reference/)  
**Primary Authority**: Navy Vessel Register (https://www.nvr.navy.mil)

---

## Naming Rules

### 1. Preferred Labels (prefLabel)

#### 1.1 Format
- **Capitalization**: Use Title Case for all preferred labels
  - ✅ Correct: "Aircraft Carrier", "Guided Missile Destroyer"
  - ❌ Incorrect: "aircraft carrier", "GUIDED MISSILE DESTROYER"

#### 1.2 Terminology
- **US Navy Standard**: Use official US Navy terminology as the primary source
  - Reference: Navy Vessel Register (NVR) ship type classifications
  - Example: "Aircraft Carrier" (not "Carrier Ship" or "Aircraft-Carrying Vessel")

#### 1.3 Specificity
- **Be Specific**: Use the most specific term that accurately describes the concept
  - ✅ Correct: "Guided Missile Destroyer" (specific)
  - ❌ Incorrect: "Destroyer" (too general when DDG is meant)

#### 1.4 Consistency
- **Consistent Terminology**: Use consistent terminology across all concept schemes
  - Status: "Active", "Decommissioned", "Museum" (not "Active Service", "Decom", "Museum Ship" as prefLabel)
  - Ship Type: "Aircraft Carrier" (not "Carrier" as prefLabel, though acceptable as altLabel)

#### 1.5 Abbreviations
- **Avoid Abbreviations in prefLabel**: Use full terms, not abbreviations
  - ✅ Correct: "Guided Missile Destroyer"
  - ❌ Incorrect: "DDG" (use as altLabel only)
  - Exception: Well-established acronyms may be used if they are the standard term (e.g., "ISR" for Intelligence, Surveillance, and Reconnaissance)

### 2. Alternative Labels (altLabel)

#### 2.1 Purpose
Alternative labels serve as:
- **Synonyms**: Different terms for the same concept
- **Abbreviations**: Common abbreviations and acronyms
- **Variations**: Spelling variations, common misspellings, or alternative phrasings
- **Legacy Terms**: Historical or deprecated terms still in use

#### 2.2 Abbreviations and Acronyms
- **Hull Classification Symbols**: Always include as altLabel
  - Example: "DDG" for "Guided Missile Destroyer"
  - Example: "CVN" for "Nuclear-Powered Aircraft Carrier"
  - Example: "SSN" for "Nuclear-Powered Attack Submarine"

#### 2.3 Common Variations
- **Capitalization Variations**: Include common capitalization variants
  - Example: "US Navy" and "U.S. Navy" for "United States Navy"
  
- **Spelling Variations**: Include British/American spelling differences
  - Example: "Defence" (British) as altLabel for "Defense" (American)

#### 2.4 Short Forms
- **Common Short Forms**: Include commonly used short forms
  - Example: "Carrier" for "Aircraft Carrier"
  - Example: "Destroyer" for "Guided Missile Destroyer" (when context is clear)
  - Example: "Decom" for "Decommissioned" (informal but common)

#### 2.5 Data Source Variations
- **Source-Specific Terms**: Include terms used in different data sources
  - Example: "Carrier" (from messy CSV) as altLabel for "Aircraft Carrier"
  - Example: "Decom" (from messy CSV) as altLabel for "Decommissioned"

#### 2.6 Limits
- **Relevance**: Only include altLabels that are:
  - Actually used in data sources
  - Commonly understood in naval/maritime context
  - Not misleading or ambiguous

### 3. Definitions (skos:definition)

#### 3.1 Format
- **Complete Sentences**: Write definitions as complete, grammatically correct sentences
- **Clarity**: Use clear, precise language accessible to domain experts
- **Completeness**: Include essential characteristics that distinguish the concept

#### 3.2 Content
- **Essential Characteristics**: Include what makes the concept unique
- **Operational Context**: Include operational or functional context when relevant
- **Distinguishing Features**: Highlight what distinguishes this concept from related concepts

#### 3.3 Examples
- **Include Examples**: When helpful, include examples in the definition or note
  - Example: "Examples: CVN-68 (Nimitz), CVN-71 (Theodore Roosevelt)"

### 4. Notes (skos:note)

#### 4.1 Purpose
Notes provide:
- **Additional Context**: Supplementary information not in the definition
- **Examples**: Specific examples of the concept
- **Usage Guidance**: How the concept is used in practice
- **Data Quality Information**: Notes about data source reliability or usage

#### 4.2 Scope Notes (skos:scopeNote)
- **Clarification**: Use scopeNote to clarify ambiguous usage or distinguish from similar concepts
  - Example: Distinguishing "Active" (status) from "Commissioned" (event)

---

## Synonym Policy

### 1. Synonym Inclusion Criteria

A term qualifies as a synonym (altLabel) if it meets **at least one** of the following criteria:

1. **Official Use**: Term is used in authoritative sources (NVR, official naval publications)
2. **Data Source Usage**: Term appears in our data sources (NVR, WDQS, messy CSV)
3. **Common Usage**: Term is commonly used in naval/maritime literature or discourse
4. **Abbreviation**: Term is a standard abbreviation or acronym (e.g., hull classification symbols)
5. **Historical Usage**: Term is a historical or legacy term still referenced
6. **International Variation**: Term is used in international navies (e.g., British vs. American terminology)

### 2. Synonym Exclusion Criteria

**Do NOT** include as altLabel if:
- Term is ambiguous or could refer to multiple concepts
- Term is a misspelling (unless commonly used misspelling)
- Term is overly technical or jargon not used in practice
- Term is deprecated and no longer in use (use deprecation process instead)

### 3. Synonym Maintenance

- **Regular Review**: Review synonyms when new data sources are added
- **Source Tracking**: Note which data sources use which synonyms
- **Conflict Resolution**: When synonyms conflict, prefer authoritative sources (NVR > WDQS > messy CSV)

---

## Deprecation Rules

### 1. When to Deprecate

Deprecate a concept when:
- **Obsolete Terminology**: The term is no longer used in official sources
- **Concept Merged**: The concept has been merged into another concept
- **Concept Split**: The concept has been split into more specific concepts
- **Replaced by Standard**: A non-standard term is replaced by an official standard term

### 2. Deprecation Process

#### 2.1 Mark as Deprecated
```turtle
ship:oldConcept a skos:Concept ;
    skos:prefLabel "Old Term"@en ;
    skos:note "DEPRECATED: Use ship:newConcept instead. Deprecated on 2024-01-15."@en .
```

#### 2.2 Maintain for Historical Data
- **Keep Deprecated Concepts**: Do not delete deprecated concepts
- **Historical Reference**: Maintain for querying historical data
- **Mapping**: Provide mapping to replacement concept using `skos:related` or `skos:broader`

#### 2.3 Documentation
- **Deprecation Date**: Record when the concept was deprecated
- **Replacement Concept**: Document the replacement concept
- **Reason**: Document why the concept was deprecated

### 3. Deprecation Examples

#### Example 1: Terminology Update
- **Old**: "Destroyer Escort" (deprecated term)
- **New**: "Frigate" or "Destroyer" (depending on context)
- **Reason**: US Navy retired "Destroyer Escort" designation

#### Example 2: Concept Refinement
- **Old**: Generic "Destroyer" when DDG is meant
- **New**: "Guided Missile Destroyer" (more specific)
- **Reason**: Need to distinguish DD from DDG

### 4. Handling Deprecated Concepts in Queries

- **Backward Compatibility**: Queries should handle both deprecated and current concepts
- **Migration Path**: Provide clear migration path from deprecated to current concepts
- **Warning**: Log warnings when deprecated concepts are used

---

## Concept Scheme Organization

### 1. Hierarchical Structure

#### 1.1 Use skos:broader and skos:narrower
- **Top-Level**: Broad categories (e.g., "Surface Combatant")
- **Mid-Level**: Specific types (e.g., "Destroyer")
- **Leaf-Level**: Most specific concepts (e.g., "Guided Missile Destroyer")

#### 1.2 Avoid Over-Nesting
- **Maximum Depth**: Generally 3-4 levels maximum
- **Clarity**: Ensure hierarchy adds clarity, not confusion

### 2. Concept Scheme Separation

- **Separate Schemes**: Use separate concept schemes for distinct domains
  - Ship Type: Physical/structural classification
  - Mission: Functional/operational classification
  - Status: Lifecycle classification
  - Navy/Branch: Organizational classification
  - Theater/Fleet: Operational assignment classification

### 3. Cross-References

- **Related Concepts**: Use `skos:related` for concepts that are related but not hierarchical
- **Mapping**: Use `skos:mappingRelation` for mapping between different concept schemes when appropriate

---

## Data Source Priority

When conflicts arise in terminology or classification:

1. **NVR (Navy Vessel Register)**: Highest priority for US Navy vessels
   - Ship types
   - Status
   - Official names

2. **WDQS (Wikidata)**: Secondary priority
   - International vessels
   - Technical specifications
   - Cross-references

3. **Messy CSV**: Lowest priority
   - Used for synonym discovery
   - Requires validation against authoritative sources

---

## Maintenance and Updates

### 1. Version Control
- **Date Stamps**: Include `dct:modified` dates on concept schemes
- **Change Log**: Document significant changes in taxonomy_guidelines.md

### 2. Regular Review
- **Quarterly Review**: Review taxonomies quarterly for:
  - New ship types or classifications
  - Terminology updates
  - Deprecation needs
  - Synonym additions

### 3. Stakeholder Input
- **Domain Experts**: Consult naval/maritime domain experts for terminology validation
- **Data Users**: Gather feedback from knowledge graph users on terminology clarity

---

## Examples

### Example 1: Ship Type Concept
```turtle
ship:guidedMissileDestroyer a skos:Concept ;
    skos:inScheme ship:shipTypeScheme ;
    skos:broader ship:destroyer ;
    skos:prefLabel "Guided Missile Destroyer"@en ;
    skos:altLabel "DDG"@en ;
    skos:altLabel "Missile Destroyer"@en ;
    skos:definition "A destroyer equipped with guided missile systems for air defense, anti-submarine warfare, and surface warfare."@en ;
    skos:note "Hull classification symbol: DDG. Examples: DDG-51 (Arleigh Burke class). Ships may be reclassified from DD to DDG."@en .
```

### Example 2: Status Concept
```turtle
ship:active a skos:Concept ;
    skos:inScheme ship:statusScheme ;
    skos:prefLabel "Active"@en ;
    skos:altLabel "In Service"@en ;
    skos:altLabel "Commissioned"@en ;
    skos:altLabel "Operational"@en ;
    skos:definition "Vessel is currently in active service, fully crewed, and capable of performing its assigned missions."@en ;
    skos:scopeNote "Distinct from 'Commissioned' which refers to the commissioning event."@en .
```

---

## References

- **SKOS Reference**: https://www.w3.org/TR/skos-reference/
- **Navy Vessel Register**: https://www.nvr.navy.mil
- **US Navy Hull Classification Symbols**: https://www.nvr.navy.mil/SHIPDETAILS/SHIPSDETAIL_CV_CVN.HTML
- **Dublin Core Terms**: http://purl.org/dc/terms/

---

## Change Log

- **2024-01-15**: Initial taxonomy guidelines document created
  - Established naming rules
  - Defined synonym policy
  - Documented deprecation rules
  - Created examples and references
