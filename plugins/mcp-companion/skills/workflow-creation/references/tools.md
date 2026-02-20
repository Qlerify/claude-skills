# Qlerify MCP Tool Reference

Natural-language descriptions of every Qlerify MCP tool, organized by category.
Each entry describes what the tool does, when to use it, and key parameters.

---

## Workflow Tools

### list_workflows
Browse all workflows accessible to the current API key. Returns workflow names, IDs, and
project IDs. Supports pagination for accounts with many workflows. Use this as the starting
point to find the right workflow before doing anything else.

### create_workflow
Create a brand new empty workflow inside a project. Provide a descriptive name — this appears
in the UI and helps users find the workflow later. Returns the new workflow ID.

- `projectId` — The project to create the workflow in
- `name` — A descriptive name for the workflow

### get_workflow
Retrieve the full workflow with all its data — events, lanes, groups, cards, schemas, everything.
This is a large response. For a quick overview, use `get_workflow_overview` instead. For saving
to a file, use the download skill with curl + jq.

- `workflowId`, `projectId` — Identifies the workflow

### get_workflow_overview
Get a lightweight summary of the workflow: lane count, group count, event count, bounded contexts,
and element statistics. Use this to understand what already exists before making changes. Much
faster and smaller than `get_workflow`.

- `workflowId`, `projectId` — Identifies the workflow

### generate_openapi_spec
Generate an OpenAPI/Swagger YAML specification from the workflow's domain model. Requires at least
one bounded context with entities. Useful for bootstrapping API implementations from the domain model.

- `workflowId`, `projectId` — Identifies the workflow
- `boundedContextId` — Which bounded context to generate the spec for

---

## Lane Tools

Lanes are the horizontal swim lanes in the BPMN diagram. Each lane represents an actor (person,
role) or system (service, API) involved in the process.

### list_lanes
Get all lanes in a workflow with their names and IDs. Use this to find lane IDs before creating
events, since every event must be assigned to a lane.

### create_lane
Add a new swim lane to the workflow. Name it after the actor or system it represents.

- `workflowId`, `projectId` — Identifies the workflow
- `name` — The lane name (e.g., "Customer", "Order Service")

### update_lane
Rename an existing lane. Does not affect events already in the lane.

- `workflowId`, `projectId`, `laneId` — Identifies the lane
- `name` — The new name

### delete_lane
Remove a lane from the workflow. The lane must be empty — move or delete all events in it first.

- `workflowId`, `projectId`, `laneId` — Identifies the lane

---

## Group Tools

Groups are the vertical columns in the BPMN diagram. Each group represents a phase, stage,
or logical section of the business process.

### list_groups
Get all groups in a workflow with their names, IDs, and positions. Use this to find group IDs
before creating events that should belong to a specific phase.

### create_group
Add a new phase/stage column to the workflow. The `startSlotX` value controls where the group
appears left-to-right — use incrementing values to order groups chronologically.

- `workflowId`, `projectId` — Identifies the workflow
- `name` — The group name (e.g., "Checkout", "Fulfillment")
- `startSlotX` — Horizontal position (higher = further right)

### update_group
Rename an existing group.

- `workflowId`, `projectId`, `groupId` — Identifies the group
- `name` — The new name

### delete_group
Remove a group. The group must be empty — move or delete all events in it first.

- `workflowId`, `projectId`, `groupId` — Identifies the group

---

## Domain Event Tools

Domain events are the core elements of the BPMN diagram — they represent things that happen
in the business process. Events are placed at the intersection of a lane and optionally a group.

### list_domain_events_overview
Quick summary listing of all events with names, types, and positions. Use this for a fast
inventory of what events exist without loading full details.

### list_domain_events
Full details of all domain events including their linked cards, schemas, and relationships.
Supports pagination for workflows with many events. Use for detailed analysis; for a quick
check, prefer the overview variant.

### create_domain_event
Add a new event to the workflow. This is the most important creation tool — events are what
make up the process flow.

- `workflowId`, `projectId` — Identifies the workflow
- `description` — The event name (use past-tense: "Order Created", not "Create Order")
- `type` — Either `bpmn:Task` (regular event) or `bpmn:ExclusiveGateway` (decision diamond)
- `laneId` — Which lane/actor this event belongs to (required)
- `groupId` — Which phase/group this event belongs to (optional)
- `parentEventId` — The preceding event in the flow. Use `"start"` for flow entry points.
  The new event is inserted after the parent in the flow sequence.
- `color` — Visual color: peach, yellow, green, teal, blue, lavender, pink, gray

### update_domain_event
Modify an existing event — change its name, type, lane, group, or color. Use this to reorganize
events or fix naming.

- `workflowId`, `projectId`, `eventId` — Identifies the event
- `description`, `type`, `laneId`, `groupId`, `color` — Fields to update (all optional)

### delete_domain_event
Remove an event from the workflow. Connected child events are automatically relinked to the
deleted event's parent, preserving the flow.

- `workflowId`, `projectId`, `eventId` — Identifies the event

---

## Entity Tools

Entities represent persistent domain objects — the data structures at the heart of the domain
model.

### list_entities
Get all entities in the workflow with their full field definitions. Use this to understand the
current data model before making changes.

### create_entity
Define a new domain entity with typed fields. Each field needs a name, data type, and ideally
example data to make the model concrete.

- `workflowId`, `projectId` — Identifies the workflow
- `name` — Entity name in PascalCase (e.g., "ShoppingCart", "CustomerProfile")
- `fields` — Array of field definitions:
  - `name` — Field name in camelCase
  - `dataType` — One of: `string`, `number`, `boolean`, `object`
  - `exampleData` — Array of 3 realistic example values
  - `isRequired` — Whether the field is mandatory (true/false)
  - `relatedEntityId` — ID of another entity to express a relationship
  - `cardinality` — `"one-to-one"` or `"one-to-many"` for reference fields with relatedEntityId

### update_entity
Modify an entity — rename it, add/update/remove fields, or change its bounded context assignment.

- `workflowId`, `projectId`, `entityId` — Identifies the entity
- `name` — New name (optional)
- `addFields`, `updateFields`, `removeFields` — Field modification arrays
- `boundedContextId` — Assign to a bounded context

### delete_entity
Remove an entity from the domain model. Also removes any card references to this entity on events.

- `workflowId`, `projectId`, `entityId` — Identifies the entity

---

## Command Tools

Commands represent state-changing operations — actions that modify data. They correspond to
POST/PUT/DELETE API endpoints or write operations.

### list_commands
Get all commands in the workflow with their field definitions.

### create_command
Define a new command with input fields. Name with action verbs.

- `workflowId`, `projectId` — Identifies the workflow
- `name` — Command name with verb prefix (e.g., "CreateOrder", "CancelSubscription")
- `fields` — Array of field definitions:
  - `name` — Field name in camelCase
  - `isRequired` — Whether the field is required/mandatory
  - `hideInForm` — Set true for auto-generated fields like IDs and timestamps
  - `relatedEntityId` — ID of another entity to express a relationship
  - `cardinality` — `"one-to-one"` or `"one-to-many"` for reference fields
  - `fields` — Nested field names from the related entity (for reference fields with relatedEntityId)

### update_command
Modify a command — rename or change fields.

- `workflowId`, `projectId`, `commandId` — Identifies the command
- `name` — New name (optional)
- `addFields`, `updateFields`, `removeFields` — Field modifications

### delete_comman
Remove a command. Also removes card references on events.

- `workflowId`, `projectId`, `commandId` — Identifies the command

---

## Read Model Tools

Read models represent data queries and views — how data is retrieved and displayed. They
correspond to GET API endpoints or query operations.

### list_read_model
Get all read models in the workflow with their field definitions.

### create_read_mode
Define a new read model/query. Name with retrieval prefixes.

- `workflowId`, `projectId` — Identifies the workflow
- `name` — Read model name (e.g., "GetOrderDetails", "ListActiveProducts", "SearchCustomers")
- `entityId` — The source entity this query reads from (optional but recommended)
- `fields` — Array of field definitions representing the full query contract (both inputs and outputs):
  - `name` — Field name in camelCase
  - `isFilter` — Set true for query parameters/filters, omit for returned data fields
  - `relatedEntityId` — ID of another entity to express a relationship
  - `fields` — Nested field names from the related entity (for reference fields with relatedEntityId)

### update_read_mode
Modify a read model — rename, change source entity, or modify fields.

- `workflowId`, `projectId`, `readModelId` — Identifies the read model
- `name`, `entityId` — Updated values (optional)
- `addFields`, `updateFields`, `removeFields` — Field modifications

### delete_read_mode
Remove a read model. Also removes card references on events.

- `workflowId`, `projectId`, `readModelId` — Identifies the read model

---

## Card Tools

Cards attach domain model elements and requirements to events. They are the bridge between
the process flow (events) and the domain model (entities, commands, read models).

### list_card_type
Get the available card type definitions for a workflow. Each card type has an ID, name, and
role. Call this before creating cards to get the correct `cardTypeId` values.

Common card types
- **Command** — Links a command to an event
- **AggregateRoot** — Links an entity to an event
- **ReadModel** — Links a read model to an event (requires cardinality)
- **GivenWhenThen** — BDD-style acceptance criteria
- **UserStory** — User story description

### list_cards
Get all cards attached to a specific event or all events in the workflow.

### create_card
Attach a card to an event. This links domain model elements to the process flow.

- `workflowId`, `projectId`, `eventId` — Which event to attach the card to
- `cardTypeId` — The card type (from `list_card_types`)
- `description` — Card content/description
- `schemaId` — ID of the linked entity, command, or read model (for domain model cards)
- `cardinality` — Required for ReadModel cards: `"one-to-one"` or `"one-to-many"`

**Constraints:**
- Maximum one Command card per event
- Maximum one AggregateRoot card per event
- ReadModel cards must specify cardinality

### update_card
Modify a card's content or linked schema.

- `workflowId`, `projectId`, `cardId` — Identifies the card
- `description`, `schemaId`, `cardinality` — Fields to update

### delete_card
Remove a card from an event.

- `workflowId`, `projectId`, `cardId` — Identifies the card

---

## Bounded Context Tools

Bounded contexts are logical boundaries that group related entities together. They represent
separate areas of the domain that could potentially be separate microservices.

### create_bounded_context
Create a new bounded context. After creation, assign entities to it using `update_entity`.

- `workflowId`, `projectId` — Identifies the workflow
- `name` — Context name (e.g., "Order Management", "Customer Identity")

### update_bounded_context
Rename a bounded context.

- `workflowId`, `projectId`, `boundedContextId` — Identifies the context
- `name` — New name

### delete_bounded_context
Remove a bounded context. Entities in it become unassigned.

- `workflowId`, `projectId`, `boundedContextId` — Identifies the context
