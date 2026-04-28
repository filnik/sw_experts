# Edge Computing

## Profile

### What It Is
Edge Computing is an architecture that brings computation and data storage closer to the sources of data and end users, reducing latency and bandwidth usage. Instead of sending all data to centralized cloud data centers, processing happens at the network edge — in CDN nodes, IoT gateways, or regional edge servers.

### Origin and Evolution
- CDNs (Akamai, 1998) were the first edge computing — caching content at the edge
- IoT growth drove edge processing needs (2010s) — sensor data too voluminous for cloud
- Edge functions: Cloudflare Workers (2017), AWS Lambda@Edge, Vercel Edge Functions
- 5G and MEC (Multi-access Edge Computing) enable compute at cellular tower level
- Current: AI inference at the edge, edge-native applications, hybrid cloud-edge architectures

### Key Authors and References
- **Mahadev Satyanarayanan** — Edge computing pioneer (Carnegie Mellon)
- **Cloudflare** — Workers and edge networking architecture
- **AWS** — Wavelength, Outposts, Lambda@Edge
- **Linux Foundation** — LF Edge projects

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Edge Function | Lightweight serverless function running at CDN edge locations |
| Edge Cache | Content cached at points of presence (PoPs) closest to users |
| IoT Edge Gateway | Device that processes sensor data before sending to cloud |
| Fog Computing | Computing layer between edge devices and cloud |
| Edge AI | ML inference running on edge devices or servers |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Latency is physics** — The speed of light limits round-trip times. The only way to reduce latency below ~30ms is to move compute closer to the user.
2. **Process where the data is** — Don't move terabytes of data to the cloud when you can process it at the source and send only the results.
3. **Graceful degradation** — Edge nodes may lose connectivity. Design for offline-first or eventually consistent operation.
4. **Minimize edge complexity** — Edge compute should be lightweight. Keep complex business logic in the cloud; push only latency-sensitive operations to the edge.
5. **Hybrid by default** — Most architectures are cloud + edge. The edge handles latency-sensitive tasks; the cloud handles heavy compute and storage.

### When to Use vs. When to Avoid

**Use when:**
- Sub-10ms latency is required (gaming, AR/VR, real-time trading)
- Bandwidth costs for shipping raw data to cloud are prohibitive (IoT, video)
- Data privacy requires local processing (not sending data to cloud)
- Global user base needs low-latency responses everywhere
- Offline/intermittent connectivity must be handled

**Avoid when:**
- Latency requirements are easily met by a single cloud region
- Workload requires heavy computation best done centrally
- Data volumes are small enough to send to cloud cheaply
- Operational complexity of edge infrastructure isn't justified

---

## Frameworks and Methodologies

### 1. Edge Function Design — step-by-step
1. Identify latency-sensitive operations (auth, redirects, personalization, A/B testing)
2. Determine what can run in constrained edge runtime (limited CPU, memory, execution time)
3. Design stateless functions that read from edge cache or KV store
4. Implement fallback to origin for complex operations
5. Deploy globally to all edge locations
6. Monitor per-region latency and error rates

### 2. IoT Edge Architecture — step-by-step
1. Define data flow: sensors → edge gateway → cloud
2. Determine which processing happens at the edge (filtering, aggregation, anomaly detection)
3. Choose edge runtime (AWS Greengrass, Azure IoT Edge, custom)
4. Design local storage for connectivity gaps
5. Implement sync protocol for edge-to-cloud data transfer
6. Plan edge device updates (OTA firmware/software updates)

---

## How to Apply

### Decision Checklist
- [ ] Is the latency requirement below what cloud-only can achieve?
- [ ] Is the edge compute lightweight enough for edge constraints?
- [ ] Is there a fallback when edge compute fails?
- [ ] Is the deployment pipeline covering all edge locations?
- [ ] Is monitoring in place for edge nodes?

### Implementation Patterns

**Edge Function (Cloudflare Workers):**
```typescript
// Runs at 300+ edge locations worldwide
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    // A/B testing at the edge
    const bucket = hashUser(request) % 100;
    if (bucket < 10) {
      url.pathname = `/experiment${url.pathname}`;
    }

    // Geo-based routing
    const country = request.headers.get("CF-IPCountry");
    if (country === "DE" || country === "FR") {
      return Response.redirect(`https://eu.example.com${url.pathname}`);
    }

    // Auth token validation at the edge
    const token = request.headers.get("Authorization");
    if (!await validateJWT(token, env.JWT_SECRET)) {
      return new Response("Unauthorized", { status: 401 });
    }

    // Forward to origin
    return fetch(request);
  }
};
```

**Edge KV Store for Configuration:**
```typescript
// Read from edge KV (Cloudflare KV, Vercel Edge Config)
async function getFeatureFlags(env: Env): Promise<FeatureFlags> {
  const cached = await env.KV.get("feature-flags", "json");
  if (cached) return cached;

  // Fallback to origin if KV miss
  const response = await fetch("https://api.example.com/feature-flags");
  const flags = await response.json();
  await env.KV.put("feature-flags", JSON.stringify(flags), { expirationTtl: 300 });
  return flags;
}
```

**IoT Edge Processing:**
```python
# Edge gateway — filter and aggregate sensor data
class EdgeProcessor:
    def __init__(self, cloud_client: CloudClient):
        self.cloud_client = cloud_client
        self.buffer: list[SensorReading] = []

    def process_reading(self, reading: SensorReading) -> None:
        # Local anomaly detection (no cloud round-trip)
        if reading.value > THRESHOLD:
            self.alert_local(reading)

        self.buffer.append(reading)

        # Batch send aggregated data to cloud every 60 seconds
        if len(self.buffer) >= 60:
            aggregated = self.aggregate(self.buffer)
            try:
                self.cloud_client.send(aggregated)
                self.buffer.clear()
            except ConnectionError:
                # Store locally, sync when connectivity returns
                self.store_local(self.buffer)
```

### Common Mistakes
1. **Moving too much logic to the edge** — Edge runtimes have constraints; complex business logic belongs in the cloud
2. **No fallback to origin** — Edge function fails and there's no way to serve the request
3. **Stale edge data** — KV stores and caches at the edge can serve outdated data; plan invalidation
4. **Ignoring cold starts at edge** — Some edge platforms have cold start delays
5. **Not testing globally** — Edge functions behave differently at different PoPs; test from multiple regions

### Integration with Other Topics
- **CDN / Caching Strategies** — CDNs are the original edge computing
- **Serverless Architecture** — Edge functions are serverless at the edge
- **API Gateway** — Edge can handle gateway functions (auth, rate limiting)
- **Microservices** — Some microservices can run at the edge
- **Resilience Patterns** — Edge provides resilience through geographic distribution
- **AI/ML Architecture** — ML inference at the edge for real-time predictions

---

## Resources

### Essential Reading
- Cloudflare Workers documentation and patterns
- AWS Wavelength and Lambda@Edge documentation
- *Edge AI* — various O'Reilly resources

### Online Resources
- Cloudflare blog on edge computing patterns
- Vercel Edge Functions documentation
- Deno Deploy edge runtime documentation

### Practice Exercises
1. Deploy an edge function that does A/B testing based on user hash
2. Implement JWT validation at the edge to protect an API
3. Build an edge KV cache with automatic invalidation
4. Design an IoT edge gateway that processes sensor data locally
5. Compare response latency between cloud function and edge function
