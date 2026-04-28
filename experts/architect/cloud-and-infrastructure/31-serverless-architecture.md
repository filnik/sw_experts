# Serverless Architecture

## Profile

### What It Is
Serverless Architecture is a cloud computing execution model where the cloud provider dynamically manages the allocation of machine resources. Applications are composed of functions (FaaS) and managed services (BaaS) that scale automatically, charge per-use, and require no server management from the developer.

### Origin and Evolution
- AWS Lambda launched 2014, pioneering Function-as-a-Service (FaaS)
- Azure Functions (2016), Google Cloud Functions (2016) followed
- Backend-as-a-Service (BaaS): Firebase, Auth0, Stripe extend serverless beyond FaaS
- Serverless Framework, SAM, SST simplify deployment
- Current: edge functions (Cloudflare Workers, Vercel), serverless containers (AWS Fargate)

### Key Authors and References
- **Mike Roberts** — "Serverless Architectures" (Martin Fowler's site)
- **Jeremy Daly** — Serverless patterns and best practices
- **Yan Cui** — "Production-Ready Serverless" course
- **AWS** — Well-Architected Serverless Lens

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| FaaS | Functions triggered by events, scaled by provider (Lambda, Cloud Functions) |
| BaaS | Managed backend services (auth, database, storage, messaging) |
| Cold Start | Latency on first invocation when no warm instance exists |
| Event Source | Trigger for function execution (HTTP, queue, schedule, stream) |
| Pay-per-use | Billed by invocation count and execution duration |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **No server management** — Focus on business logic, not infrastructure. The provider handles scaling, patching, and availability.
2. **Event-driven by nature** — Functions are triggered by events (HTTP requests, queue messages, schedules). Design around event flows.
3. **Pay for value** — Zero cost at zero traffic. Cost scales linearly with usage. No paying for idle servers.
4. **Micro-architecture** — Each function handles one task. Small, focused, independently deployable units.
5. **Managed services first** — Use managed databases, queues, auth, and storage. Build only what differentiates your application.

### When to Use vs. When to Avoid

**Use when:**
- Variable/unpredictable traffic (scales to zero and up)
- Event-driven workloads (webhooks, queue processing, scheduled tasks)
- Rapid development with minimal infrastructure management
- Cost optimization for low-traffic applications
- Glue logic between managed services

**Avoid when:**
- Consistent high-traffic requiring sustained compute
- Long-running processes (>15 min) or stateful operations
- Cold start latency is unacceptable
- Complex local development/debugging requirements
- Vendor lock-in is a concern

---

## Frameworks and Methodologies

### 1. Serverless Design — step-by-step
1. Decompose application into event-driven functions
2. Identify event sources (API Gateway, SQS, S3, CloudWatch Events)
3. Choose managed services for state (DynamoDB, RDS Proxy, S3)
4. Design function interfaces (event in, response out)
5. Handle cold starts (keep functions small, use provisioned concurrency if needed)
6. Implement structured logging and tracing (X-Ray, CloudWatch)

### 2. Serverless Migration — step-by-step
1. Identify a standalone feature/endpoint to migrate first
2. Implement as a Lambda function behind API Gateway
3. Connect to existing database via VPC/proxy
4. Redirect traffic for that endpoint to the serverless version
5. Monitor cost, latency, and errors
6. Migrate additional features incrementally

---

## How to Apply

### Decision Checklist
- [ ] Is the workload event-driven or request-driven with variable traffic?
- [ ] Can cold start latency be tolerated (or mitigated)?
- [ ] Is the execution duration within limits (15 min for Lambda)?
- [ ] Are managed services used for state and infrastructure?
- [ ] Is the cost model favorable compared to always-on compute?

### Implementation Patterns

**API + Lambda + DynamoDB:**
```python
# handler.py — AWS Lambda function
import json
import boto3

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("orders")

def create_order(event, context):
    body = json.loads(event["body"])
    order = {
        "order_id": generate_id(),
        "customer_id": body["customer_id"],
        "items": body["items"],
        "status": "placed",
    }
    table.put_item(Item=order)
    return {
        "statusCode": 201,
        "body": json.dumps({"order_id": order["order_id"]}),
    }

def get_order(event, context):
    order_id = event["pathParameters"]["id"]
    result = table.get_item(Key={"order_id": order_id})
    if "Item" not in result:
        return {"statusCode": 404, "body": json.dumps({"error": "Not found"})}
    return {"statusCode": 200, "body": json.dumps(result["Item"])}
```

**Event-Driven Processing:**
```python
# Process SQS messages
def process_order_events(event, context):
    for record in event["Records"]:
        message = json.loads(record["body"])
        match message["event_type"]:
            case "OrderPlaced":
                send_confirmation_email(message)
            case "OrderShipped":
                send_shipping_notification(message)
    # Return success (SQS deletes processed messages)

# Scheduled function (CloudWatch Events)
def daily_report(event, context):
    """Runs every day at 8 AM UTC."""
    report = generate_daily_sales_report()
    send_to_slack(report)
```

**Infrastructure (SAM/CloudFormation):**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Runtime: python3.12
    Timeout: 30
    MemorySize: 256

Resources:
  CreateOrderFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handler.create_order
      Events:
        Api:
          Type: Api
          Properties:
            Path: /orders
            Method: post
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref OrdersTable

  OrdersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: orders
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: order_id
          AttributeType: S
      KeySchema:
        - AttributeName: order_id
          KeyType: HASH
```

### Common Mistakes
1. **Monolith in a Lambda** — Putting the entire application in one function defeats the purpose
2. **Ignoring cold starts** — Cold starts add 100ms-5s depending on runtime and package size
3. **VPC for everything** — Putting Lambda in a VPC when not needed adds cold start latency
4. **No timeout/retry strategy** — Lambda retries on failure; without idempotency, this causes duplicates
5. **Over-engineering** — Using Lambda for a service that runs 24/7 at high load (EC2/ECS is cheaper)

### Integration with Other Topics
- **Cloud-Native Architecture** — Serverless is cloud-native taken to the extreme
- **Event-Driven Architecture** — Serverless functions are event-driven by nature
- **API Design** — API Gateway + Lambda for REST/GraphQL APIs
- **Microservices** — Functions as micro-microservices
- **Infrastructure as Code** — SAM, CDK, Serverless Framework for deployment
- **Observability** — X-Ray, CloudWatch for function monitoring

---

## Resources

### Essential Reading
- *Serverless Architectures on AWS* — Peter Sbarski
- AWS Serverless Lens (Well-Architected Framework)
- "Serverless Architectures" — Mike Roberts (martinfowler.com)

### Online Resources
- Yan Cui's "Production-Ready Serverless" course
- Jeremy Daly's serverlessland.com
- AWS Lambda documentation

### Practice Exercises
1. Build a REST API with Lambda + API Gateway + DynamoDB
2. Process SQS messages with a Lambda consumer
3. Implement a scheduled Lambda for daily data processing
4. Measure and optimize cold start latency
5. Compare cost of Lambda vs. ECS for different traffic patterns
