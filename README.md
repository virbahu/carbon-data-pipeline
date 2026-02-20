# 🔍 carbon-data-pipeline

[![Apache Kafka](https://img.shields.io/badge/Apache-Kafka-231F20?logo=apachekafka)](https://kafka.apache.org/)
[![Apache Spark](https://img.shields.io/badge/Apache-Spark-E25A1C?logo=apachespark)](https://spark.apache.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791?logo=postgresql)](https://postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker)](https://docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Google Scholar](https://img.shields.io/badge/Google%20Scholar-Virbahu%20Jain-4285F4?logo=googlescholar&logoColor=white)](https://scholar.google.com/citations?user=4SN8o-QAAAAJ&hl=en)

> **Real-time IoT data ingestion and streaming pipeline for continuous Scope 3 carbon accounting — replacing 12-month survey cycles with live emission telemetry.**
>
> ---
>
> ## 📋 Overview
>
> **carbon-data-pipeline** is a production-grade event-streaming infrastructure that ingests IoT sensor data, ERP transactions, logistics telemetry, and supplier API feeds in real-time to maintain a continuously updated Scope 3 carbon accounting ledger.
>
> The fundamental problem with traditional Scope 3 accounting is temporal: by the time emission data is collected, validated, and reported, it is 12–18 months stale. Decarbonization interventions based on last year's data are flying blind. This pipeline solves the staleness problem by treating carbon data as a first-class event stream.
>
> Core capabilities:
>
> - **Real-time IoT ingestion** from factory sensors, smart meters, vehicle telematics, and logistics platforms
> - - **Event-driven carbon accounting** with sub-minute latency from emission event to ledger update
>   - - **Multi-source integration** across ERP systems (SAP, Oracle), logistics APIs (DHL, FedEx), and supplier portals
>     - - **Streaming Scope 3 computation** using configurable emission factor lookup at the event level
>       - - **Immutable audit ledger** built on Apache Kafka for regulatory-grade data lineage
>         - - **Scalable to billions of events** with horizontal scaling via Kubernetes
>          
>           - ---
>
> ## 🖼️ Data Flow Overview
>
> ![Streaming Architecture](https://img.shields.io/badge/IoT%20→%20Kafka%20→%20Spark%20→%20PostgreSQL-Real--time%20Carbon%20Ledger-orange?style=for-the-badge&logo=apachekafka)
>
> ```
>  IoT Sensors (MQTT)  ─┐
>  ERP Events (CDC)    ──┼──► Kafka Topics ──► Spark Streaming ──► Carbon Ledger
>  Logistics APIs      ──┤                            │
>  Supplier Portals    ─┘                             ▼
>                                            Emission Factor Lookup
>                                            (broadcast join)
>                                                      │
>                                   ┌──────────────────┼──────────────┐
>                                   ▼                  ▼              ▼
>                              PostgreSQL         TimescaleDB    Data Warehouse
>                              (Audit Log)        (Dashboard)    (Annual Report)
> ```
>
> ---
>
> ## 🏗️ Architecture Diagram
>
> ```
> ╔═══════════════════════════════════════════════════════════════════╗
> ║         CARBON DATA PIPELINE — STREAMING ARCHITECTURE             ║
> ╠═══════════════════════════════════════════════════════════════════╣
> ║                                                                   ║
> ║  DATA SOURCES (Real-time)                                         ║
> ║  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────────┐ ║
> ║  │  Factory   │ │  Vehicle   │ │  ERP/SAP   │ │   Supplier     │ ║
> ║  │  IoT Snrs  │ │  Telemtcs  │ │  PO Events │ │   API Feeds    │ ║
> ║  │  (MQTT)    │ │  (REST)    │ │  (Webhooks)│ │  (REST/SFTP)   │ ║
> ║  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └──────┬─────────┘ ║
> ║        └──────────────┴──────────────┴────────────────┘            ║
> ║                                      │                             ║
> ║                     ┌────────────────▼──────────────┐              ║
> ║                     │    INGESTION LAYER             │              ║
> ║                     │    Kafka Connect + Producers   │              ║
> ║                     │    • Schema Registry (Avro)    │              ║
> ║                     │    • Dead Letter Queue         │              ║
> ║                     └────────────────┬──────────────┘              ║
> ║                                      │                             ║
> ║  KAFKA TOPICS:                       ▼                             ║
> ║  ┌─────────────────────────────────────────────────────────────┐  ║
> ║  │  raw.iot.energy  │  raw.logistics  │  raw.procurement       │  ║
> ║  └─────────────────────────────┬───────────────────────────────┘  ║
> ║                                 │                                   ║
> ║  STREAM PROCESSING              ▼                                   ║
> ║  ┌─────────────────────────────────────────────────────────────┐  ║
> ║  │  Apache Spark Structured Streaming                          │  ║
> ║  │  • Emission Factor Lookup (broadcast join)                  │  ║
> ║  │  • GHG Protocol category assignment                         │  ║
> ║  │  • kgCO2e computation per event                             │  ║
> ║  │  • 5-min / 1-hour / 24-hour aggregation windows             │  ║
> ║  └─────────────────────────────┬───────────────────────────────┘  ║
> ║                                 │                                   ║
> ║  ┌────────────────┐   ┌──────────────────┐   ┌──────────────────┐ ║
> ║  │  PostgreSQL    │   │   TimescaleDB     │   │  Data Warehouse  │ ║
> ║  │  (Audit Ledgr) │   │   (Dashboards)    │   │  (Annual rpt)   │ ║
> ║  └────────────────┘   └──────────────────┘   └──────────────────┘ ║
> ╚═══════════════════════════════════════════════════════════════════╝
> ```
>
> ---
>
> ## ❗ Problem Statement
>
> ### The Carbon Data Latency Crisis
>
> Enterprise Scope 3 accounting operates on a 12–18 month reporting cycle. Companies set decarbonization targets against data that is already stale before intervention can begin.
>
> | Dimension | Batch Approach | Streaming Approach |
> |---|---|---|
> | **Data Freshness** | 12–18 months stale | Sub-minute latency |
> | **Anomaly Detection** | Post-hoc, annual | Real-time threshold alerts |
> | **Intervention Speed** | Next fiscal year | Same operational day |
> | **Data Sources** | Surveys + invoices | IoT + ERP + logistics live |
> | **Audit Trail** | Manual spreadsheets | Immutable Kafka log |
> | **Scalability** | Excel/VLOOKUP | Billions of events/day |
>
> > *"You cannot decarbonize a supply chain on a 12-month feedback loop. Real-time emission telemetry is the foundation of science-based action."*
> >
> > ---
> >
> > ## ✅ Solution Overview
> >
> > ### Event-Driven Carbon Accounting Architecture
> >
> > The pipeline treats every energy consumption reading, every purchase order creation, every logistics leg departure, and every supplier production event as an emission-relevant event that must be immediately classified, quantified, and recorded.
> >
> > **Ingestion Layer**
> > Apache Kafka Connect with pre-built connectors ingests data from MQTT brokers (factory IoT), REST APIs (logistics, supplier portals), SAP/Oracle CDC streams (ERP procurement events), and SFTP file drops. All events are schema-validated with Apache Avro and registered in the Schema Registry.
> >
> > **Stream Processing Layer**
> > Spark Structured Streaming jobs run continuously with micro-batch intervals of 30 seconds to 5 minutes. Each event undergoes emission factor lookup via a broadcast-joined reference table, Scope 3 category assignment, and kgCO2e computation. Windowed aggregations produce rolling inventory totals.
> >
> > **Storage and Serving Layer**
> > Processed emission records land in three stores: PostgreSQL (audit ledger), TimescaleDB (time-series for dashboards), and a data warehouse (historical analytics). A FastAPI service layer exposes inventory data to downstream applications.
> >
> > ---
> >
> > ## 💻 Code, Installation & Analysis
> >
> > ### Quick Start with Docker
> >
> > ```bash
> > git clone https://github.com/virbahu/carbon-data-pipeline.git
> > cd carbon-data-pipeline
> >
> > # Start the full stack
> > docker-compose up -d
> >
> > # Services: Kafka, Schema Registry, Kafka Connect,
> > # Spark (1 master + 2 workers), PostgreSQL, TimescaleDB, Grafana, FastAPI
> >
> > # Load demo data
> > python scripts/load_demo_data.py --events 10000 --duration 60
> > ```
> >
> > ### Producing Carbon Events
> >
> > ```python
> > from pipeline.producers import CarbonEventProducer, IoTEnergyEvent
> > from datetime import datetime
> >
> > producer = CarbonEventProducer(bootstrap_servers="localhost:9092")
> >
> > event = IoTEnergyEvent(
> >     sensor_id="SM-PLANT-DE-042",
> >     facility_id="FACILITY_MUENCHEN_01",
> >     country_iso2="DE",
> >     energy_kwh=1247.3,
> >     energy_source="grid",
> >     grid_carbon_intensity_gco2_kwh=385.2,
> >     timestamp=datetime.utcnow()
> > )
> >
> > producer.send("raw.iot.energy", key=event.sensor_id, value=event)
> > ```
> >
> > ### Querying the Carbon Ledger
> >
> > ```python
> > from api.client import CarbonLedgerClient
> >
> > client = CarbonLedgerClient(base_url="http://localhost:8000")
> >
> > inventory = client.get_supplier_inventory(
> >     supplier_id="SUP_042_DE",
> >     scope=3,
> >     start_date="2025-01-01",
> >     end_date="2025-12-31"
> > )
> >
> > print(f"YTD Scope 3: {inventory.total_tco2e:,.1f} tCO2e")
> > print(f"Last updated: {inventory.last_event_timestamp}")
> > # >> YTD Scope 3: 4,832.7 tCO2e
> > # >> Last updated: 2025-12-20T14:32:07Z  (< 1 minute ago)
> > ```
> >
> > ---
> >
> > ## 📦 Dependencies
> >
> > ```yaml
> > # docker-compose.yml
> > services:
> >   kafka:         confluentinc/cp-kafka:7.6.0
> >   schema-reg:    confluentinc/cp-schema-registry:7.6.0
> >   kafka-connect: confluentinc/cp-kafka-connect:7.6.0
> >   spark-master:  bitnami/spark:3.5
> >   spark-worker:  bitnami/spark:3.5
> >   postgres:      postgres:15
> >   timescaledb:   timescale/timescaledb:latest-pg15
> >   grafana:       grafana/grafana:10.3.0
> > ```
> >
> > ```toml
> > [tool.poetry.dependencies]
> > python = "^3.10"
> > confluent-kafka = "^2.3"
> > pyspark = "^3.5"
> > fastavro = "^1.9"
> > psycopg2-binary = "^2.9"
> > sqlalchemy = "^2.0"
> > fastapi = "^0.110"
> > pandas = "^2.0"
> > pydantic = "^2.0"
> > ```
> >
> > ---
> >
> > ## 👤 Author
> >
> > <img src="https://avatars.githubusercontent.com/u/virbahu" width="80" align="left" style="margin-right:15px; border-radius:50%"/>

**Virbahu Jain** — Founder & CEO, [Quantisage](https://quantisage.com)

> *Building the AI Operating System for Scope 3 emissions management and supply chain decarbonization.*
>
> <br clear="left"/>

| | |
|---|---|
| 🎓 **Education** | MBA, Kellogg School of Management, Northwestern University |
| 🏭 **Experience** | 20+ years across manufacturing, life sciences, energy & public sector |
| 🌍 **Scope** | Supply chain operations on five continents |
| 📝 **Research** | Peer-reviewed publications on AI in sustainable supply chains |
| 🔬 **Patents** | IoT and AI solutions for manufacturing and logistics |

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?logo=linkedin)](https://linkedin.com/in/virbahu)
[![GitHub](https://img.shields.io/badge/GitHub-virbahu-181717?logo=github)](https://github.com/virbahu)
[![Google Scholar](https://img.shields.io/badge/Google%20Scholar-Publications-4285F4?logo=googlescholar&logoColor=white)](https://scholar.google.com/citations?user=4SN8o-QAAAAJ&hl=en)
[![Quantisage](https://img.shields.io/badge/Company-Quantisage-00C853)](https://quantisage.com)

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

![Quantisage](https://img.shields.io/badge/Quantisage-Open%20Source%20Initiative-00C853?style=for-the-badge)
![Supply Chain](https://img.shields.io/badge/AI-Supply%20Chain-blue?style=for-the-badge)
![Climate](https://img.shields.io/badge/Climate-Tech-green?style=for-the-badge)

<sub>Part of the <strong>Quantisage Open Source Initiative</strong> | AI × Supply Chain × Climate</sub>
</div>
