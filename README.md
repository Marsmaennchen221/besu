# Peer Performance Metrics — Issue #9721

> **Upstream Pull Request:** https://github.com/besu-eth/besu/pull/10506  
> **Branch (Pull Request):** `9721-add-peer-performance-metrics`  
> **Branch (including README):** `main`  
> **Author:** Marsmaennchen221

## What This Contribution Does

This branch implements per-peer performance metrics for Hyperledger Besu, addressing issue [#9721](https://github.com/besu-eth/besu/issues/9721). Prior to this change, Besu had no built-in way to observe how individual peers were performing in terms of response latency, data throughput, or response quality. This contribution adds that observability layer directly into Besu's peer management subsystem and exposes fleet-wide aggregates via Besu's existing Prometheus metrics endpoint.

The following metrics are now tracked per peer and exposed as Prometheus gauges:

| Metric | Description |
|--------|-------------|
| P50 / P95 / P99 latency | Response latency percentiles over a sliding window of up to 1,000 samples |
| Success rate | Ratio of useful to total responses (defaults to 1.0 if no responses yet) |
| Total bytes transferred | Cumulative bytes received from this peer |
| Average bytes per response | Total bytes divided by total responses |
| Rolling average bytes/sec | Throughput over a 30-second sliding window |

Fleet-wide average, minimum, and maximum values for each metric are registered at startup and available on the `/metrics` endpoint.

## Files Changed

| File | Change |
|------|--------|
| `ethereum/eth/src/main/java/.../EthPeer.java` | Core metric fields, recording methods (`recordLatency`, `recordBytesReceived`), and all getter methods |
| `ethereum/eth/src/main/java/.../EthPeers.java` | 16 new Prometheus gauges (avg/min/max for latency, success rate, throughput) |
| `ethereum/eth/src/main/java/.../RequestManager.java` | Hook to automatically record latency and byte counts on every response |
| `ethereum/eth/src/main/java/.../EthPeerImmutableAttributes.java` | 7 new record components exposing all metrics in peer snapshots |
| `ethereum/eth/src/test/java/.../EthPeerTest.java` | Unit tests for percentile computation, byte recording, and success rate edge cases |
| `CHANGELOG.md` | Entry added under the current development version |

## Prerequisites

- Java 21
- Gradle (wrapper included, no separate install needed)

## Build

```bash
git clone https://github.com/Marsmaennchen221/besu.git
cd besu
./gradlew build
```

## Run the Tests

To run only the tests for the affected module (faster than the full build):

```bash
./gradlew :ethereum:eth:test
```

To run the full test suite:

```bash
./gradlew build
```

To apply code style formatting (required before any PR):

```bash
./gradlew spotlessApply
```

## Verifying the Metrics

Start a Besu node with metrics enabled:

```bash
./gradlew installDist
./build/install/besu/bin/besu \
  --network=sepolia \
  --metrics-enabled \
  --metrics-port=9545
```

Once peers connect, the new gauges will appear at:  
http://localhost:9545/metrics  
Look for metrics prefixed with `besu_peers_peer_avg_`, `besu_peers_peer_min_`, `besu_peers_peer_max_`, and `besu_peers_peer_total_`.

---

*The upstream Besu project README continues below.*

---

# Besu Ethereum Client
[![CircleCI](https://circleci.com/gh/besu-eth/besu/tree/main.svg?style=svg)](https://circleci.com/gh/besu-eth/besu/tree/main)
[![CodeQL](https://github.com/besu-eth/besu/actions/workflows/codeql.yml/badge.svg)](https://github.com/besu-eth/besu/actions/workflows/codeql.yml)
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/3174/badge)](https://bestpractices.coreinfrastructure.org/projects/3174)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/besu-eth/besu/blob/main/LICENSE)
[![Discord](https://img.shields.io/discord/905194001349627914?logo=Hyperledger&style=plastic)](https://discord.com/invite/hyperledger)
[![Twitter Follow](https://img.shields.io/twitter/follow/HyperledgerBesu)](https://twitter.com/HyperledgerBesu)

[Download](https://github.com/besu-eth/besu/releases)

Besu is an Apache 2.0 licensed, MainNet compatible, Ethereum client written in Java.

## Useful Links

* [Besu User Documentation]
* [Besu Issues]
* [Besu Wiki](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/)
* [How to Contribute to Besu](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22156850/How+to+Contribute)
* [Besu Roadmap & Planning](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154278/Besu+Roadmap+Planning)


## Issues

Besu issues are tracked [in the github issues tab][Besu Issues].
See our [guidelines](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154243/Issues) for more details on searching and creating issues.

If you have any questions, queries or comments, [Besu channel on Discord] is the place to find us.


## Besu Users

To install the Besu binary, follow [these instructions](https://besu.hyperledger.org/public-networks/get-started/install/binary-distribution).

## Besu Developers

* [Contributing Guidelines]
* [Coding Conventions](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154259/Coding+Conventions)
* [Command Line Interface (CLI) Style Guide](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154260/Besu+CLI+Style+Guide)
* [Besu User Documentation] for running and using Besu


### Development

Instructions for how to get started with developing on the Besu codebase. Please also read the
[wiki](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154251/Pull+Requests) for more details on how to submit a pull request (PR).

* [Checking Out and Building](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154264/Building+from+source)
* [Code Coverage](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154288/Code+coverage)
* [Logging](https://lf-hyperledger.atlassian.net/wiki/spaces/BESU/pages/22154291/Logging) or the [Documentation's Logging section](https://besu.hyperledger.org/public-networks/how-to/monitor/logging)

#### Dependency Verification

This project uses [Gradle dependency verification](https://docs.gradle.org/current/userguide/dependency_verification.html). When adding or updating dependencies, regenerate `gradle/verification-metadata.xml` with:

```shell
./gradlew --write-verification-metadata sha256 resolveSourceArtifacts
```

The `resolveSourceArtifacts` task ensures source JARs are included in the metadata, which is required for IDE sync (e.g. IntelliJ automatically downloads sources).

### Profiling Besu

Besu supports performance profiling using [Async Profiler](https://github.com/async-profiler/async-profiler), a low-overhead sampling profiler.  
You can find setup and usage instructions in the [Profiling Guide](docs/PROFILING.md).

Profiling can help identify performance bottlenecks in block processing, transaction validation, and EVM execution.  
Please ensure the profiler is run as the same user that started the Besu process.

## Release Notes

[Release Notes](CHANGELOG.md)

## Reference Tests and JSON Tracing

Besu includes support for running Ethereum reference tests and generating detailed EVM execution traces.

To learn how to run the tests and enable opcode-level JSON tracing for debugging and correctness verification, see the [Reference Test Execution and Tracing Guide](REFERENCE_TESTS.md).

[Besu Issues]: https://github.com/besu-eth/besu/issues
[Besu User Documentation]: https://besu.hyperledger.org
[Besu channel on Discord]: https://discord.com/invite/hyperledger
[Contributing Guidelines]: CONTRIBUTING.md
