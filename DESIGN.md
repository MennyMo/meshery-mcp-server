# Meshery MCP Server – Design

## Goals

The Meshery MCP Server extension is intended to:

- Provide an AI-native interface to Meshery through the Model Context Protocol (MCP).
- Enable AI agents and other MCP clients to create, inspect, export, and analyze Meshery designs and related artifacts.
- Establish clear, stable tool contracts that can grow with Meshery capabilities without duplicating server or client infrastructure.

## Architecture Overview

At a high level, the MCP Server consists of:

- **MCP Server**: The server process that implements MCP and exposes tools to MCP clients.
- **Shared Meshery client**: The common client layer used by MCP tools to communicate with Meshery Server.
- **MCP tools**: Focused tool implementations for Meshery designs, deployment workflows, and test results.

The MCP Server is a thin layer that translates MCP tool calls into Meshery client operations and returns structured, predictable results suitable for MCP clients.

The implementation builds on the repository foundation established by the MCP Server scaffold. This includes the selected Go MCP SDK, an initial stdio transport, the shared server and tool-registration structure, and common configuration support. Tool implementations should extend this foundation instead of introducing parallel server or registration patterns.

## Transport Considerations

The initial MCP Server transport is stdio. This aligns with the repository foundation and provides a focused starting point for local MCP client integrations.

The MCP Server core should remain transport-agnostic so additional transports can be considered later when they are supported by the project foundation and maintainer priorities. The existing proof of concept provides useful implementation input for future transport work, but it does not determine the main project transport strategy.

## Meshery REST Integration

The initial MCP tools will use Meshery's existing REST APIs through the shared Meshery client.

REST is the practical initial integration path because the required Meshery Server APIs, external-client workflows, and JSON response shapes already exist. Introducing a separate gRPC integration would require new protobuf contracts and corresponding server-side services for the required resources.

Streaming-oriented capabilities, including MeshSync or other live-state workflows, are outside the initial scope. They can be evaluated later without changing the core MCP tool architecture.

### Shared client responsibilities

The shared Meshery client is the single integration layer between MCP tools and Meshery Server. It is responsible for:

- Meshery Server base-URL configuration.
- Authentication configuration and request handling.
- REST request execution and error handling.
- Explicit mapping of Meshery API response fields.

MCP tools must use the shared client rather than directly construct REST requests, authorization headers, or authentication cookies. This keeps individual tools independent of the final authentication implementation and consistent with the repository configuration contract.

### Data Shape and Pagination

For the list-designs API, the known REST response fields include `page`, `pageSize`, `totalCount`, and `patterns`.

The shared Meshery client should use explicit JSON tags or equivalent field mapping to correctly parse Meshery's camelCase response fields. The `list_designs` MCP tool should expose a documented, stable response contract that identifies returned design data and pagination metadata.

Where a tool transforms a REST response, the transformation should be explicit and documented so MCP clients and AI agents receive predictable, stable results.

## Initial MCP Tools

The initial scope of MCP tools is expected to cover:

- **List Meshery designs**: Retrieve designs available in Meshery, with documented pagination metadata.
- **Export a design**: Export a selected design in a requested format, such as YAML or JSON.
- **Snapshot a design**: Create a snapshot of a design at a point in time.
- **Retrieve deployment dry-run results**: Access dry-run outputs associated with a design.
- **Retrieve performance test results**: Access performance-test results associated with a design.

These tools will build on the shared Meshery client and return structured JSON responses and human-readable error messages suitable for MCP clients and AI agents.

## Future Tool Candidates

The Meshery MCP proof of concept also demonstrates read-only access to MeshSync-discovered Kubernetes resources and Kubernetes cluster connections. These are promising future tool candidates.

Before they are added to the main MCP Server scope, they should be proposed as separate issues and aligned with maintainer priorities, the shared Meshery client, and the project transport strategy.
