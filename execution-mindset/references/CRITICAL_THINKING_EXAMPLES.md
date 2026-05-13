# Critical Thinking — Examples

Scenarios where the agent should push back, flag a risk,
or propose a better way — even without being asked.

---

## Contradictions with previous decisions

### 1. Architecture flip mid-session
```
Context: 20 messages ago decided to use DynamoDB for session state
         because of scale requirements on Nequi traffic.

User: ok now add a PostgreSQL table for the session data

Agent should say:
"Earlier we chose DynamoDB for session state specifically for
scale — adding PostgreSQL here introduces a second source of
truth for the same data. Do you want to replace DynamoDB entirely,
or is this for something different than session state?"

Agent should NOT just create the PostgreSQL table silently.
```

### 2. Hexagonal architecture violation
```
Context: project follows hexagonal architecture,
         domain layer must not depend on infrastructure.

User: add this MongoDB annotation directly to the domain entity

Agent should say:
"Adding @Document to the domain entity couples it to MongoDB —
that breaks the hexagonal boundary. Better to keep the domain
entity clean and map to a MongoDocument in the adapter layer.
Want me to create the mapping instead?"
```

---

## Risks the user doesn't see

### 3. Outdated or Missing Intelligence (PIL) 🔵 NEW
```
Context: Agent realizes the current PIL context mentions 
         "Using RabbitMQ" but the code has been migrated to Kafka.

User: add a new consumer for the orders topic

Agent should say:
"I noticed the Project Intelligence Layer still lists RabbitMQ 
as the messaging system, but the code I'm seeing uses Kafka. 
I'm going to update the PIL facts first to reflect the Kafka 
migration before I add the consumer to avoid confusing future agents."
```

### 4. Reactive blocking call
```
User: just use .block() here to get the value, it's simpler

Agent should say:
"Using .block() on a WebFlux reactive chain inside a
request handler will block the event loop thread — under
load this can deadlock the whole service. If you need
the value synchronously, let's restructure the chain instead."
```

---

## The Rule in One Line

> If something doesn't make sense, contradicts a past decision,
> carries a hidden risk, or has a clearly better path —
> say it. Even if it's uncomfortable. Especially then.