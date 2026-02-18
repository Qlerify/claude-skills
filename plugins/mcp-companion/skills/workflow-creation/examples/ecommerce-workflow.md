# Example: E-Commerce Order Workflow

A complete workflow creation sequence for a simple e-commerce order process.
All IDs below are placeholders — actual calls return real UUIDs.

---

## Phase 1: Foundation

### Step 1 — Create workflow

```
create_workflow(projectId: "proj-1", name: "E-Commerce Order Process")
→ { workflowId: "wf-1" }
```

### Step 2 — Create lanes

```
create_lane(workflowId: "wf-1", projectId: "proj-1", name: "Customer")
→ { laneId: "lane-customer" }

create_lane(workflowId: "wf-1", projectId: "proj-1", name: "Order Service")
→ { laneId: "lane-order" }

create_lane(workflowId: "wf-1", projectId: "proj-1", name: "Payment Gateway")
→ { laneId: "lane-payment" }
```

### Step 3 — Create groups

```
create_group(workflowId: "wf-1", projectId: "proj-1", name: "Browse & Cart")
→ { groupId: "grp-browse" }

create_group(workflowId: "wf-1", projectId: "proj-1", name: "Checkout")
→ { groupId: "grp-checkout" }

create_group(workflowId: "wf-1", projectId: "proj-1", name: "Fulfillment")
→ { groupId: "grp-fulfill" }
```

---

## Phase 2: Event Flow

### Step 4 — Create domain events

Build the chain left-to-right. Each event references the previous one as parent.

```
create_domain_event(
  workflowId: "wf-1", projectId: "proj-1",
  description: "Item Added to Cart",
  type: "bpmn:Task",
  laneId: "lane-customer",
  groupId: "grp-browse",
  parentEventId: "start",
  color: "blue"
)
→ { eventId: "evt-1" }

create_domain_event(
  workflowId: "wf-1", projectId: "proj-1",
  description: "Order Placed",
  type: "bpmn:Task",
  laneId: "lane-customer",
  groupId: "grp-checkout",
  parentEventId: "evt-1",
  color: "peach"
)
→ { eventId: "evt-2" }

create_domain_event(
  workflowId: "wf-1", projectId: "proj-1",
  description: "Payment Successful?",
  type: "bpmn:ExclusiveGateway",
  laneId: "lane-payment",
  groupId: "grp-checkout",
  parentEventId: "evt-2"
)
→ { eventId: "evt-3" }

create_domain_event(
  workflowId: "wf-1", projectId: "proj-1",
  description: "Payment Confirmed",
  type: "bpmn:Task",
  laneId: "lane-payment",
  groupId: "grp-checkout",
  parentEventId: "evt-3",
  color: "green"
)
→ { eventId: "evt-4" }

create_domain_event(
  workflowId: "wf-1", projectId: "proj-1",
  description: "Payment Failed",
  type: "bpmn:Task",
  laneId: "lane-payment",
  groupId: "grp-checkout",
  parentEventId: "evt-3",
  color: "pink"
)
→ { eventId: "evt-5" }

create_domain_event(
  workflowId: "wf-1", projectId: "proj-1",
  description: "Order Shipped",
  type: "bpmn:Task",
  laneId: "lane-order",
  groupId: "grp-fulfill",
  parentEventId: "evt-4",
  color: "peach"
)
→ { eventId: "evt-6" }
```

---

## Phase 3: Domain Model

### Step 5 — Create entities

```
create_entity(
  workflowId: "wf-1", projectId: "proj-1",
  name: "Order",
  fields: [
    { name: "id", dataType: "string", exampleData: ["ord-001", "ord-002", "ord-003"], isRequired: true, primaryKey: true },
    { name: "customerId", dataType: "string", exampleData: ["cust-10", "cust-22", "cust-07"], isRequired: true },
    { name: "status", dataType: "string", exampleData: ["pending", "confirmed", "shipped"], isRequired: true },
    { name: "totalAmount", dataType: "number", exampleData: ["59.99", "124.50", "9.99"], isRequired: true },
    { name: "createdAt", dataType: "string", exampleData: ["2026-01-15T10:00:00Z", "2026-01-16T14:30:00Z", "2026-01-17T09:15:00Z"], isRequired: true }
  ]
)
→ { entityId: "ent-order" }

create_entity(
  workflowId: "wf-1", projectId: "proj-1",
  name: "CartItem",
  fields: [
    { name: "id", dataType: "string", exampleData: ["ci-001", "ci-002", "ci-003"], isRequired: true, primaryKey: true },
    { name: "productName", dataType: "string", exampleData: ["Wireless Mouse", "USB-C Cable", "Laptop Stand"], isRequired: true },
    { name: "quantity", dataType: "number", exampleData: ["1", "3", "2"], isRequired: true },
    { name: "unitPrice", dataType: "number", exampleData: ["29.99", "8.50", "45.00"], isRequired: true },
    { name: "orderId", dataType: "string", exampleData: ["ord-001", "ord-001", "ord-002"], isRequired: true, relatedEntityId: "ent-order", cardinality: "one-to-many" }
  ]
)
→ { entityId: "ent-cart-item" }
```

### Step 6 — Create commands

```
create_command(
  workflowId: "wf-1", projectId: "proj-1",
  name: "AddItemToCart",
  fields: [
    { name: "productId", dataType: "string", exampleData: ["prod-101", "prod-202", "prod-303"], isRequired: true },
    { name: "quantity", dataType: "number", exampleData: ["1", "2", "5"], isRequired: true }
  ]
)
→ { commandId: "cmd-add" }

create_command(
  workflowId: "wf-1", projectId: "proj-1",
  name: "PlaceOrder",
  fields: [
    { name: "cartId", dataType: "string", exampleData: ["cart-01", "cart-02", "cart-03"], isRequired: true },
    { name: "shippingAddress", dataType: "string", exampleData: ["123 Main St", "456 Oak Ave", "789 Pine Rd"], isRequired: true },
    { name: "paymentMethod", dataType: "string", exampleData: ["credit_card", "paypal", "bank_transfer"], isRequired: true }
  ]
)
→ { commandId: "cmd-place" }

create_command(
  workflowId: "wf-1", projectId: "proj-1",
  name: "ShipOrder",
  fields: [
    { name: "orderId", dataType: "string", exampleData: ["ord-001", "ord-002", "ord-003"], isRequired: true },
    { name: "trackingNumber", dataType: "string", exampleData: ["TRK-1234", "TRK-5678", "TRK-9012"], isRequired: true },
    { name: "carrier", dataType: "string", exampleData: ["FedEx", "UPS", "DHL"], isRequired: true }
  ]
)
→ { commandId: "cmd-ship" }
```

### Step 7 — Create read models

```
create_read_model(
  workflowId: "wf-1", projectId: "proj-1",
  name: "GetOrderDetails",
  entityId: "ent-order",
  fields: [
    { name: "orderId", dataType: "string", exampleData: ["ord-001", "ord-002", "ord-003"], isRequired: true, isFilter: true }
  ]
)
→ { readModelId: "rm-details" }

create_read_model(
  workflowId: "wf-1", projectId: "proj-1",
  name: "ListCustomerOrders",
  entityId: "ent-order",
  fields: [
    { name: "customerId", dataType: "string", exampleData: ["cust-10", "cust-22", "cust-07"], isRequired: true, isFilter: true },
    { name: "status", dataType: "string", exampleData: ["pending", "confirmed", "shipped"], isFilter: true }
  ]
)
→ { readModelId: "rm-list" }
```

---

## Phase 4: Linking

### Step 8 — Get card types

```
list_card_types(workflowId: "wf-1", projectId: "proj-1")
→ {
    cardTypes: [
      { id: "ct-cmd", name: "Command" },
      { id: "ct-agg", name: "AggregateRoot" },
      { id: "ct-rm", name: "ReadModel" },
      { id: "ct-gwt", name: "GivenWhenThen" },
      { id: "ct-us", name: "UserStory" }
    ]
  }
```

### Step 9 — Create cards on events

Link domain model elements to their corresponding events:

```
# "Item Added to Cart" ← AddItemToCart command + CartItem aggregate
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-1", cardTypeId: "ct-cmd", schemaId: "cmd-add")
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-1", cardTypeId: "ct-agg", schemaId: "ent-cart-item")

# "Order Placed" ← PlaceOrder command + Order aggregate
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-2", cardTypeId: "ct-cmd", schemaId: "cmd-place")
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-2", cardTypeId: "ct-agg", schemaId: "ent-order")

# "Order Placed" ← GetOrderDetails read model (single order)
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-2", cardTypeId: "ct-rm", schemaId: "rm-details", cardinality: "one-to-one")

# "Order Shipped" ← ShipOrder command + Order aggregate
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-6", cardTypeId: "ct-cmd", schemaId: "cmd-ship")
create_card(workflowId: "wf-1", projectId: "proj-1", eventId: "evt-6", cardTypeId: "ct-agg", schemaId: "ent-order")
```

---

## Phase 5: Organization

### Step 10 — Create bounded context

```
create_bounded_context(workflowId: "wf-1", projectId: "proj-1", name: "Order Management")
→ { boundedContextId: "bc-orders" }

# Assign entities to the context
update_entity(workflowId: "wf-1", projectId: "proj-1", entityId: "ent-order", boundedContextId: "bc-orders")
update_entity(workflowId: "wf-1", projectId: "proj-1", entityId: "ent-cart-item", boundedContextId: "bc-orders")
```

---

## Result

The workflow now has:

- 3 lanes (Customer, Order Service, Payment Gateway)
- 3 groups (Browse & Cart, Checkout, Fulfillment)
- 6 domain events with a decision gateway for payment
- 2 entities (Order, CartItem) with typed fields and references
- 3 commands (AddItemToCart, PlaceOrder, ShipOrder)
- 2 read models (GetOrderDetails, ListCustomerOrders)
- 7 cards linking domain model to events
- 1 bounded context (Order Management)
