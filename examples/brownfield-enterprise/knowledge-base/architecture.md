# Architecture — Enterprise Java Platform

*KB Document — produced by aid-discover, updated by aid-interview*

## Overview

OSGi-based modular platform (~200 bundles) built on Eclipse Tycho/Maven. The platform is a monorepo containing backend services, REST APIs, Aura (React) frontends, and API/Chaos test suites.

## Architecture Style

- **Runtime:** OSGi container (Eclipse Equinox)
- **Build:** Eclipse Tycho + Maven reactor
- **Module coupling:** OSGi Import-Package / Fragment-Host (loose coupling by design)
- **Persistence:** JPA/Hibernate with context-specific session factories
- **Search:** 4-layer stack (Framework → Indexing → ES Client → Advanced Search)
- **Frontend:** React (Aura) packages built via Node/pnpm, deployed as OSGi bundles

## Key Subsystems

| Subsystem | Bundle Pattern | Responsibility |
|-----------|---------------|----------------|
| Core Platform | `com.example.common.platform.*` | Base services, models, security |
| Search | `com.example.common.platform.search*` | Search framework + indexing |
| Metadata | `com.example.common.platform.metadata` | Advanced search, custom attributes |
| Hibernate | `com.example.common.hibernate.*` | JPA, search bridges, ES integration |
| Elasticsearch | `com.example.common.elasticsearch.*` | ES client, mapping, configuration |
| Aura Frontend | `packages/platform-*` | React UI components |

## Deployment Model

- Monorepo → single build → deployable product assembly
- Eclipse features group bundles for installation
- Version: `29.x.y-SNAPSHOT` (continuous development)

## Constraints

- Java 17 (Zulu distribution)
- Must maintain backward compatibility with existing JPA schemas
- OSGi classloader isolation affects dependency management
- No containerization — traditional installation model
