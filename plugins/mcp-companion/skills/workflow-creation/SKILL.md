---
name: workflow-creation
description: >
  This skill should be used when the user asks to "create a workflow",
  "build a domain model", "set up a new Qlerify workflow", "add events and lanes",
  "create BPMN diagram", "model a business process", "set up domain events",
  "add commands and read models to workflow", or any request involving building
  a Qlerify workflow from scratch or adding structural elements to an existing one.
  Provides the full creation sequence, tool ordering, and domain modeling guidance.
allowed-tools: Read, Glob, Grep
---

# Workflow Creation Guide

Build well-structured Qlerify workflows by following a specific tool sequence. Skipping steps
or calling tools out of order leads to broken references and incomplete diagrams.

## Core Concepts

A Qlerify workflow is a BPMN-style diagram combined with domain-driven design (DDD) elements:

- **Lanes** — Horizontal swim lanes representing actors or systems (e.g., "Customer", "Payment Service")
- **Groups** — Vertical columns representing phases or stages (e.g., "Order Placement", "Fulfillment")
- **Domain Events** — The core building blocks placed at lane/group intersections. Each event represents something that
  happens in the business process (e.g., "Order Created", "Payment Received")
- **Cards** — Requirements attached to events. Each event can have cards for Command, Aggregate Root, Read Model,
  Given-When-Then, and User Story
- **Entities** — Persistent domain objects (Aggregate Roots) with typed fields
- **Commands** — State-changing operations with input fields
- **Read Models** — Data queries/views with filter fields
- **Bounded Contexts** — Logical boundaries grouping related entities

## Creation Sequence

Follow these steps in order. Each step depends on the previous one.

### Phase 1: Foundation

**Step 1 — Identify or create the workflow**

For an existing workflow, call `list_workflows` to find it, then `get_workflow_overview` to understand
its current state. For a new workflow, call `create_workflow` with a descriptive name.

**Step 2 — Create lanes**

Every event must belong to a lane. Call `create_lane` for each actor or system involved in the process.
Aim for 2-5 lanes. Common patterns:
- User-facing: "Customer", "Admin", "Manager"
- System: "Order Service", "Payment Gateway", "Notification System"
- External: "Third-party API", "Bank"

**Step 3 — Create groups**

Groups organize events into phases. Call `create_group` for each phase in chronological order — the `startSlotX` value
controls left-to-right positioning. Common patterns:
- E-commerce: "Browse", "Cart", "Checkout", "Fulfillment", "Post-delivery"
- Onboarding: "Registration", "Verification", "Setup", "Activation"

### Phase 2: Event Flow

**Step 4 — Create domain events**

Build the event flow by chaining calls to `create_domain_event`. Each event needs a `laneId` and optionally a `groupId`.

- Use `parentEventId: "start"` for flow starting points
- Chain subsequent events by referencing the previous event's ID as `parentEventId`
- Use type `bpmn:Task` for regular events, `bpmn:ExclusiveGateway` for decision points
- Assign meaningful colors to distinguish event categories (peach, yellow, green, teal, blue, lavender, pink, gray)

Build the flow left-to-right, top-to-bottom, creating events in the order they occur in the
business process.

### Phase 3: Domain Model

**Step 5 — Create entities**

Call `create_entity` for each core domain object. Each entity represents a persistent data structure:
- Include fields with `name`, `dataType`, `exampleData` (3 realistic values), `isRequired`
- Use `relatedEntityId` and `cardinality` to express entity relationships from the owning entity's perspective
- Field types: `string`, `number`, `boolean`, `object`

**Step 6 — Create commands**

Call `create_command` for each state-changing operation. Name with action verbs (Create, Update, Delete, Submit,
Cancel):

- Command fields should correspond to fields on the aggregate root entity they modify
- Mark auto-generated fields (IDs, timestamps) with `hideInForm: true`
- Use `relatedEntityId` and `cardinality` to reference related entities, with nested `fields` containing field names
  from that entity

**Step 7 — Create read models**

Call `create_read_model` for each data retrieval view. Name with Get/List/Search prefixes:
- Link to the source entity via `entityId`
- Fields represent the full query contract: both inputs and outputs
- Set `isFilter: true` for query parameters (what you search by), omit for returned data fields
- Use `relatedEntityId` on return fields that reference other entities, with nested `fields` for their field names

### Phase 4: Linking

**Step 8 — Get card types**

Call `list_card_types` to get available card type definitions for the workflow. Note the IDs
for Command, AggregateRoot, ReadModel, and other card types needed.

**Step 9 — Create cards on events**

Attach domain model elements to events by calling `create_card`:
- **Command card** — Links a command to the event. One per event maximum.
- **Aggregate Root card** — Links an entity to the event. One per event maximum.
- **Read Model card** — Links a read model to the event. Requires `cardinality` ("one-to-one" or "one-to-many").

Each card needs `eventId`, `cardTypeId`, and `schemaId` (the ID returned by `create_entity`, `create_command`, or
`create_read_model` in earlier steps).

### Phase 5: Organization

**Step 10 — Create bounded contexts**

Group related entities into bounded contexts for logical separation. Call `create_bounded_context`
and then update entities to assign them to contexts. Required before generating OpenAPI specs.

## Constraints and Rules

- **One Aggregate Root card per event** — An event can only have one entity linked as aggregate root
- **One Command card per event** — An event can only have one command
- **Read Model cards require cardinality** — Always specify "one-to-one" or "one-to-many"
- **Lanes and groups cannot be deleted if they contain events** — Move or delete events first
- **Domain events require a lane** — Every event must be assigned to a lane
- **Chain events via parentEventId** — Use "start" for root events, otherwise reference the parent

## Common Workflow Patterns

### CRUD Service
Lanes: User, Service | Groups: Create, Read, Update, Delete
Flow: User action event → Service processing event per operation

### Approval Pipeline
Lanes: Requester, Approver, System | Groups: Submit, Review, Execute
Flow: Request submitted → Review pending → Approved/Rejected gateway → Executed

### Event-Driven Saga
Lanes: Service A, Service B, Orchestrator | Groups per saga step
Flow: Each service emits events, orchestrator coordinates via decision gateways

## Tips for Well-Structured Workflows

- **Start with events, not entities.** Map the business process flow first, then identify what data each step needs.
- **Use decision gateways sparingly.** Only for genuine branching logic, not optional steps.
- **Color-code by domain.** Use consistent colors for events in the same bounded context.
- **3-5 lanes is ideal.** More than 5 makes the diagram hard to read.
- **Name events as past-tense occurrences.** "Order Created", not "Create Order" — commands go on cards.
- **Include realistic example data.** 3 values per field helps stakeholders understand the model.

## Additional Resources

### Reference Files

For detailed natural-language descriptions of every MCP tool, parameters, and usage tips:
- **`references/tools.md`** — Complete tool reference with natural-language descriptions, parameters, and usage tips

### Example Files

For a complete worked example showing all 10 steps with realistic tool calls and data:
- **`examples/ecommerce-workflow.md`** — End-to-end e-commerce order workflow creation with 3 lanes, 6 events, 2
  entities, 3 commands, 2 read models, and a decision gateway
