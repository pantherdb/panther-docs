# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PantherDB is a bioinformatics web application for protein classification, gene ontology, and pathway analysis. It serves the PANTHER (Protein ANalysis THrough Evolutionary Relationships) classification system. The main application code lives under `pantherSite/`.

## Build Commands

All commands run from `pantherSite/` directory using Apache Ant:

```bash
# Full build (compiles all Java source)
ant -f pantherBuild.xml all

# Clean compiled classes
ant -f pantherBuild.xml clean

# Run JUnit tests
ant -f pantherBuild.xml run-tests

# Publish for development (copies dev config to WEB-INF)
ant -f pantherBuild.xml pub

# Publish for production
ant -f pantherBuild.xml pub_prd

# Publish for ISP
ant -f pantherBuild.xml pub_isp

# Compile JSPs
ant -f pantherBuild.xml jsp

# Static analysis with FindBugs
ant -f pantherBuild.xml findbugs

# Create release tar.gz
ant -f pantherBuild.xml release
```

The `all` target builds in order: `ab_util` → `pub_panther_treeviewer` → `sci_site_all`. Java source/target level is 11.

## Architecture

### Three-Tier Design

**Web Layer (Struts 1.x MVC)**
- Struts actions/forms configured in `config/struts-config.xml`
- 394 JSP pages in `wwwroot/`
- Servlet definitions in `config/web.xml`
- Filters: `IPAddrFilter` (access control), `CorsFilter` (for `/services/oai/pantherdb/*`)
- Listeners: `SessionListener`, `StartupShutdownListener`

**Business Logic / REST API**
- Legacy servlets: `com.ab.scienceSite.servlet.*`
- Modern REST API services: `edu.usc.ksom.pm.panther.pantherdb.servlet.services.*` (17+ endpoints)
- API configuration and Swagger/OpenAPI support via `edu.usc.ksom.pm.panther.pantherdb.servlet.api.*`
- Data managers: `edu.usc.ksom.pm.panther.pantherdb.data.manager.*`

**Data Access Layer**
- iBatis 2.x SQL mapping framework, configured in `config/sql-map-config.xml`
- SQL map XML files: `src/com/ab/scienceSite/dao/ibatis/maps/*.xml`
- JDBC DAOs: `src/com/ab/scienceSite/dao/jdbc/`
- PostgreSQL database with Apache DBCP2 connection pooling (min 2, max 20 connections)

### Key Package Structure

| Package | Purpose |
|---------|---------|
| `com.ab.scienceSite.dao` | Data access (iBatis + JDBC implementations) |
| `com.ab.scienceSite.servlet` | Legacy request handlers |
| `com.ab.scienceSite.webapp` | Struts action forms |
| `com.ab.scienceSite.domain` | Domain objects |
| `com.ab.scienceSite.logic` | Business logic |
| `com.ab.scienceSite.biodata` | Biological data handling |
| `com.ab.scienceSite.statistics` | Statistical analysis |
| `com.celera.pubPanther.treeViewer` | Phylogenetic tree viewer (applet/servlet/GUI) |
| `edu.usc.ksom.pm.panther.pantherdb.servlet.services` | REST API endpoints |
| `edu.usc.ksom.pm.panther.pantherdb.biodata` | Advanced biological data models |
| `com.ab.util` | Shared utilities (logging, caching, config, email) |

### Configuration

Environment-specific configs in `config/` are copied to `wwwroot/WEB-INF/classes/` by the `pub`/`pub_prd`/`pub_isp` targets:
- `database.properties.{development,isp,production}` → `database.properties`
- `extra.properties.{development,isp,production}` → `extra.properties`
- `pubPanther.properties` → application properties

Logging uses Log4j 2.25.1 configured in `config/log4j2.xml`. Log output directory is set via the `PANTHER_LOGS` environment variable.

### Scoring Pipeline

`scoring/` contains Perl-based PANTHER HMM scoring tools and SNP scoring scripts (CGI). These require external dependencies: NCBI BLAST, HMMER, and the PANTHER HMM library.

## Testing

Tests use JUnit 3.x (extends `TestCase`). Test classes are in:
- `src/com/ab/scienceSite/dao/ibatis/test/` — iBatis DAO tests
- `src/com/ab/scienceSite/dao/jdbc/test/` — JDBC DAO tests

Tests require a running PostgreSQL database connection.

## Version Control

This repository uses Subversion (SVN), not Git.
