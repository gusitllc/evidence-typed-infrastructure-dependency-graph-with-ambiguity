# Evidence-typed infrastructure dependency graph with ambiguity-refusing host resolution and cycle-reporting cutover wave sequencing

**Defensive Publication** - published to establish prior art.
**Publisher:** Gus IT LLC.

## Abstract

A method plans migration cutover waves from discovered infrastructure using only observed evidence. Every edge kind has a fixed type, confidence and inferred flag. Direct configuration references found in web-server site configuration (network share paths, database connection targets, site binding host names) yield non-inferred dependency edges; shared virtual switches, virtual networks, address subnets, hybrid location identifiers and hypervisor hosts yield inferred co-location edges. Edges are created only between discovered workloads. A host token claimed by more than one workload is recorded as an ambiguous reference with all candidates and creates no edge. A single configured confidence floor decides both which dependency edges are promoted into an evidence-free list of dependency identifiers and which edges order waves. Waves are longest-path depths from a topological sort; cycles are reported as whole strongly connected components with their evidence and are never broken automatically.

## Technical Field

Infrastructure migration planning, specifically dependency discovery and ordering of workload cutover from inventory data gathered across hypervisors, clouds and edge clusters.

## Background

Migration programs move many servers and applications in waves. A wave plan that moves an application before the database it uses causes an outage. Discovery tools therefore infer dependencies from configuration and network observation. Two common practices cause wrong plans. First, host names and addresses are resolved to workloads by first match; in multi-tenant or multi-subscription estates, default first addresses of virtual networks and generic host names recur, so a reference may be bound to the wrong workload with high apparent confidence. Second, weak signals such as sharing a host or subnet are mixed with direct configuration references, and tools break dependency cycles automatically by dropping an edge chosen by heuristic, hiding a decision that should be made by a person. A further problem is that a dependency list stored without its evidence or confidence cannot later be distinguished from a certain one, so the threshold used to store dependencies and the threshold used to order cutover can drift apart.

## Summary of the Disclosure

Disclosed is a two-stage pure computation. The graph stage takes the discovered workloads with their normalized facts and produces nodes, edges and statistics. Each edge carries its origin, target, type, an evidence record naming the observation kind, the observed token and the vantage from which it was collected, a confidence and an inferred flag, all fixed per evidence kind. Direct configuration references create directed dependency edges; co-location signals create undirected co-location edges. A host index maps each token to the single workload that answers to it; tokens claimed by more than one workload are moved to an ambiguity map, and any reference to them is recorded with its candidate list instead of creating an edge. A projection promotes only dependency edges at or above a configured floor into a per-workload list of target row identifiers. The sequencing stage reads the same floor, keeps only dependency edges between known nodes at or above it, and computes waves by longest-path topological sorting. Unsequenced nodes trigger strongly-connected-component detection; cycles are returned with their edges and evidence. Blocked workloads are annotated in place.

## Detailed Description

Inputs. Each discovered workload has a stable source reference, a name, a kind and a facts record collected from a vantage such as a hypervisor host, a public cloud or an edge cluster. Facts may include network interfaces with addresses, switch names and virtual network references, a hostname and fully qualified name, a hybrid location identifier, and web-server site configuration with physical paths, connection strings or pre-extracted connection targets, and bindings. Workloads are deduplicated by reference and processed in reference order.

Evidence kinds. A frozen table defines every kind that may create an edge. Three kinds are direct configuration references and produce a dependency type with no inferred flag: a network share path appearing in a site or virtual-directory physical path, a database connection target, and a site binding host name. Five kinds are co-location signals and produce a co-location type with the inferred flag: a shared virtual switch on one hypervisor host, a shared virtual network reference, a shared address subnet, a shared hybrid location identifier, and a shared hypervisor host. Each kind carries its own fixed confidence, with direct references above co-location signals.

Host index. For each workload, the method derives host tokens: its name, hostname and fully qualified name in lower case, the short form before the first dot of each, and every interface address. It builds a map from token to workload. When a second, different workload claims a token, the token is removed from the map and placed in an ambiguity map with both candidates; further claimants are appended. Candidate lists are sorted.

Direct edges. For each workload with web-server facts, the method extracts the host of every share path (the first path segment, recording only the host and share prefix as observed text), the host of every connection target or of every raw connection string after scrubbing it to its host token so no credential is retained, and every binding host name. For each token, if it is in the ambiguity map, an ambiguous-reference record with the source, the kind, the token, the observed text and all candidates is appended and nothing else happens. If the token maps to a single workload, a dependency edge is added from the source to that workload. A token that maps to nothing creates nothing.

Co-location edges. For each co-location kind, the method groups workloads by the projected key. Groups with one member are ignored. Groups larger than a configured maximum are not expanded; a truncated-group record with kind, observed key, size and limit is reported instead. Otherwise every unordered pair in the group receives one co-location edge.

Edge admission. An edge is refused if its kind is unknown, either endpoint is missing or not a discovered workload, the endpoints are equal, or the same source, target and kind already exists. Edges are sorted by source, target and kind; statistics report counts by kind, inferred count, the floor in force, the number of ambiguous tokens, and the ambiguous references and truncated groups when present.

Promotion. A projection reads the configured confidence floor and, for each dependency edge at or above it, looks up the persisted row identifier of the target. If none exists the edge is counted as skipped; otherwise the identifier is added once to the source's list. Lists are sorted. Co-location edges and dependency edges below the floor are never promoted; they remain in the graph for display.

Sequencing. The sequencer resolves the floor from the same configuration key, clamped to the unit interval, with an optional explicit override for callers. It keeps only dependency edges with known endpoints and a finite confidence at or above the floor, counting each rejection by reason (not a dependency, below floor, unknown endpoint, duplicate). Edges are reversed so a dependency precedes its dependant. Starting from nodes with no incoming edges, processed in identifier order, it assigns each successor a depth equal to the maximum of its predecessors' depths plus one and releases it when its in-degree reaches zero. Waves group nodes by depth with identifiers sorted; nodes listed as blocked carry a reason and detail annotation but keep their wave.

Cycles. If not every node was placed, the result is marked failed with a cycle error but still returns the waves that could be placed and the list of unplaced nodes, which includes cycle members and nodes downstream of them. An iterative strongly-connected-component search over the kept edges, visiting nodes and neighbours in identifier order, returns each component with more than one member or with a self-edge, together with the edges inside it and their evidence, confidence and inferred flag. No edge is dropped automatically; a person decides which inferred or weak edge to remove.

## Disclosed Features (each disclosed alone and in combination)

1. A method assigns every dependency-evidence kind a fixed edge type, confidence and inferred flag, and refuses any edge whose kind is not in that table.
2. The method creates non-inferred dependency edges only from direct configuration references found in web-server site configuration, namely network share paths, database connection targets and site binding host names.
3. The method creates inferred co-location edges from shared virtual switches, virtual networks, address subnets, hybrid location identifiers and hypervisor hosts, and reports rather than expands co-location groups above a configured size.
4. A system creates an edge only when both endpoints are discovered workloads, and attaches to every edge an evidence record naming the observation kind, the observed token and the collection vantage.
5. The system records a host token claimed by more than one workload as ambiguous, and records each reference to it with all candidates instead of creating an edge.
6. The method promotes only dependency edges at or above a configured confidence floor into an evidence-free list of dependency identifiers, skipping and counting targets without a persisted identifier.
7. The method uses the same configured floor to select the edges that order cutover waves, and counts every rejected edge by reason.
8. The method assigns each workload to a wave equal to its longest-path depth in a topological sort over dependency edges, processing ties in identifier order.
9. The system reports cycles as complete strongly connected components with the evidence of each internal edge and never breaks a cycle automatically.
10. The system returns partial waves and the list of unsequenced workloads when a cycle exists, and annotates blocked workloads without removing them from their wave.

## Variations

- Direct references may also be harvested from application configuration files, service definitions or observed network flows, each with its own fixed kind and confidence.
- Ambiguity may be resolved by an additional scoping key, such as tenant or network segment, before a token is declared ambiguous.
- The promoted dependency list may store the evidence kind and confidence alongside each identifier, removing the need for a promotion floor.
- Waves may be computed by breadth-first layering on the reversed graph or by a constraint solver that also balances wave size.
- Cycle reports may rank internal edges by confidence to suggest, but not apply, a removal.
- The subnet key may use the actual prefix length of each interface instead of a fixed prefix.

## Note on omitted parameters

Per-kind confidence values, the promotion and sequencing floor, the maximum co-location group size and the subnet prefix length are implementation choices.
