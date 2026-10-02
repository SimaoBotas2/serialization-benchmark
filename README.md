# Serialization Benchmark: VMs vs Containers

Benchmark of XML and JSON serialization and deserialization in Java, measured in two deployment models: virtual machines and Docker containers. The goal is to answer: **is serialization/deserialization faster in container-based systems than in VMs?**

University project for the *Integração de Sistemas* (Systems Integration) course, Computer Engineering (LEI), University of Coimbra.

## Scenario

A legacy application running on a VM is to be migrated to the cloud. A sender and a receiver exchange a list of `Student` records (grouped in a `ClassPackage`) over the network: first on one VM per machine, then in one container per machine with the same code. Compared aspects: resource overhead, speed, setup complexity and scalability.

## Features

- XML serialization with JAXB, and JSON serialization with Jackson (see each `pom.xml`)
- Sender and receiver communicating over TCP sockets
- Benchmarks with 10,000, 100,000 and 250,000 records
- Metrics: payload size, time, throughput and memory used
- Docker Compose setup for the containerized version, with a WireGuard recipe to connect containers across two hosts
- Script that generates the comparison charts

## Project Structure

| Path | Contents |
|---|---|
| `VMs/` | Sender/Receiver (XML, JAXB) for the VM scenario, plus `xml-benchmark/` and `json-benchmark/` Maven projects and run instructions |
| `Containers/` | Same code in Docker: `setup/` (Dockerfiles and `docker-compose.yml`), `json/` and the XML sources, with their own readme and quickstart |
| `Results - graphs/` | Result tables and `graphs.py`, which generates the bar plots and comparisons (PNG) |
| `Problem Statement.md` | Scenario and research question |

## Tech Stack

Java · Maven · JAXB · JSON · Docker / Docker Compose · WireGuard · Python (pandas, matplotlib) for the charts

## Getting Started

### VMs

See [`VMs/readme_VMs.md`](VMs/readme_VMs.md), [`VMs/xml-benchmark/HowToRunXML.md`](VMs/xml-benchmark/HowToRunXML.md) and [`VMs/json-benchmark/HowToRunJSON.md`](VMs/json-benchmark/HowToRunJSON.md). In short, with Java 17 and JAXB installed:

```bash
javac *.java
java Receiver     # on machine 2
java Sender       # on machine 1
```

### Containers

From `Containers/setup`:

```bash
docker compose up --build receiver     # terminal 1
docker compose run --rm sender         # terminal 2
```

See [`Containers/readme_Containers.md`](Containers/readme_Containers.md) and [`Containers/QUICKSTART.md`](Containers/QUICKSTART.md) for the multi-host setup.

### Charts

```bash
pip install pandas matplotlib
python "Results - graphs/graphs.py"
```

## Results

Complete tables are in [`Results - graphs/Results_table.md`](<Results - graphs/Results_table.md>). Summary of the runs with 10,000, 100,000 and 250,000 records:

| | VM | Container |
|---|---|---|
| JSON payload vs XML | about 34% smaller | about 47% smaller |
| JSON deserialization vs XML | about 3x to 4x faster | about 3x to 4x faster |
| JSON serialization vs XML | faster at 100,000 and 250,000 records | faster at all sizes |

Example, 250,000 records:

| Environment | Format | Serialization (s) | Deserialization (s) | Memory (MB) |
|---|---|---|---|---|
| VM | XML | 0.529 | 1.871 | 65 |
| VM | JSON | 0.312 | 0.433 | 37 |
| Container | XML | 0.131 | 1.236 | 171 |
| Container | JSON | 0.050 | 0.285 | 81 |

Conclusions:

- **XML vs JSON:** JSON produced smaller payloads, deserialized 3x to 4x faster and used less memory in both environments.
- **VMs vs containers:** serialization was several times faster in the container runs, and deserialization was similar or up to 1.5x faster at 250,000 records. Memory use was higher in containers in these runs.
- **Caveat:** the payload sizes differ between the VM and the container runs (for example 648,103 and 1,037,898 bytes for 10,000 XML records), so the comparison is not strictly like for like. Each configuration was measured once, so the results are indicative.

## Authors

Simão Carvalho, André Rodrigues · University of Coimbra · Computer Engineering · 2026
