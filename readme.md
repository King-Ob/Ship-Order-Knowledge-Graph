# Ship Order Knowledge Graph

A comprehensive knowledge graph for naval vessels, integrating data from multiple sources (NVR, Wikidata, CSV) with full provenance tracking, entity resolution, and data quality validation.

## Quick Start

**New to this project?** Start here: [Setup Guide](docs/setup_guide.md) - Complete step-by-step instructions for beginners.

## Project Structure

```
├── data/
│   ├── raw/          # Source data files (NVR, Wikidata, CSV)
│   ├── golden/       # Resolved/merged ship records
│   └── rdf/          # Generated RDF data
├── docs/             # Documentation
│   ├── setup_guide.md        # ⭐ START HERE - Setup instructions
│   ├── demo_script.md        # Presentation guide
│   ├── competency_questions.md
│   └── ...
├── ontology/         # OWL ontologies
│   ├── naval_core.owl.ttl
│   ├── naval_events.owl.ttl
│   └── naval_prov.owl.ttl
├── taxonomy/        # SKOS concept schemes
│   ├── ship_type.skos.ttl
│   ├── mission.skos.ttl
│   └── status.skos.ttl
├── queries/         # SPARQL queries
│   └── demo_queries.sparql   # 60+ demo queries
├── mappings/        # RML/R2RML mappings
│   ├── nvr.rml.ttl
│   └── wikidata_extract.rml.ttl
└── shapes/          # SHACL validation shapes
    └── ship_core.shacl.ttl
```

## Getting Started

1. **Read the Setup Guide**: [docs/setup_guide.md](docs/setup_guide.md)
2. **Install Prerequisites**: Java, Apache Jena Fuseki
3. **Load the Data**: Follow Step 4 in the setup guide
4. **Run Queries**: Use the 60+ demo queries in `queries/demo_queries.sparql`

## Documentation

- **[Setup Guide](docs/setup_guide.md)** - Complete installation and setup instructions
- **[Demo Script](docs/demo_script.md)** - Presentation guide with 60+ queries
- **[Scope](docs/scope.md)** - Project objectives and data sources
- **[Competency Questions](docs/competency_questions.md)** - 20 functional requirements

## Features

- ✅ Multi-source data integration (NVR, Wikidata, CSV)
- ✅ Full provenance tracking (PROV-O)
- ✅ Entity resolution and conflict handling
- ✅ Data quality validation (SHACL)
- ✅ Temporal event modeling
- ✅ 60+ presentation-ready SPARQL queries

## Technologies

- **OWL 2** - Core ontology
- **SKOS** - Taxonomies and controlled vocabularies
- **SHACL** - Data validation
- **RML** - Data mapping
- **SPARQL** - Query language
- **PROV-O** - Provenance tracking

## License

This project is for educational/demonstration purposes.
