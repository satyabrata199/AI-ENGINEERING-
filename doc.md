# LiveKit → FastAPI → vLLM: Production LLM Inference Request Lifecycle

> **Purpose:** This document explains the complete lifecycle of an LLM
> request in our architecture: an external LiveKit Agent connects to our
> OpenAI-compatible FastAPI gateway, which forwards requests to vLLM
> serving open-source or fine-tuned models.
>
> **Scope:** Architecture and production engineering concepts. This is a
> learning/reference document, not an instruction to modify the existing
> project.
>
> **Important distinction:** LiveKit owns the real-time agent/session
> workflow. Our FastAPI service is an LLM inference API gateway. vLLM
> performs model inference. This gateway is not, by itself, the complete
> conversational AI application.

------------------------------------------------------------------------

## 1. Architecture at a glance

``` text
Participant
    |
    | Audio / real-time interaction
    v
LiveKit Room + LiveKit Agent
    |
    | STT result / agent conversation context
    | OpenAI Chat Completions request
    v
Nginx / Load Balancer
    |
    | HTTPS termination, routing, body limits,
    | connection handling, network protection
    v
FastAPI LLM Gateway
    |
    | Request ID / authentication / authorization
    | validation / quotas / model routing
    | upstream connection management
    v
vLLM OpenAI-compatible server
    |
    | Tokenization / scheduling / prefix-cache lookup
    | prefill / KV cache / decode
    v
Open-source or fine-tuned model on GPU
    |
    | OpenAI-compatible streamed chunks
    v
vLLM → FastAPI → Nginx → LiveKit Agent
    |
    | Agent consumes generated text
    v
TTS / speech output → LiveKit Room → Participant
```

The actual STT and TTS providers may be separate services. LiveKit
Agents orchestrates the agent pipeline; the exact pipeline depends on
the application configuration.

### Responsibility boundaries

  -----------------------------------------------------------------------
  Component               Main responsibility     What it should not be
                                                  confused with
  ----------------------- ----------------------- -----------------------
  LiveKit Agent           Real-time session,      The model inference
                          conversation flow, turn server
                          handling, STT/LLM/TTS   
                          orchestration           

  FastAPI gateway         Client-facing API,      The GPU model itself
                          authentication,         
                          validation, quotas,     
                          routing, streaming      
                          proxy                   

  vLLM                    Model serving, request  User/session management
                          scheduling, batching,   
                          prefill and token       
                          generation              

  GPU                     Tensor computation for  API request handling
                          inference               

  Redis                   Shared counters, rate   The authoritative store
                          limits, short-lived     for every business
                          coordination/cache      record

  PostgreSQL (if used)    Durable users, API-key  A requirement for every
                          metadata, usage and     inference request
                          operational records     

  Nginx                   Reverse proxy, TLS,     Application
                          network-level limits    authorization
                          and connection handling 

  Prometheus / Grafana /  Metrics, dashboards,    Request execution
  tracing                 alerting and            
                          request-path diagnosis  
  -----------------------------------------------------------------------

The gateway should keep the vLLM endpoint on a private network. External
LiveKit agents should call the gateway, not bypass it and call the GPU
server directly.

------------------------------------------------------------------------

## 2. Phase 0 --- LiveKit decides that an LLM response is needed

A participant speaks or sends input in a LiveKit session. A typical
voice-agent pipeline is:

1.  Audio arrives in the LiveKit room.
2.  The agent's VAD/turn-detection logic determines when the participant
    has finished or paused.
3.  An STT component produces text, or another input source supplies
    text directly.
4.  The agent builds the LLM context from its instructions and
    conversation history.
5.  The agent invokes its configured LLM provider.

The important point is that **the gateway does not normally receive the
raw LiveKit audio in this architecture**. It receives an LLM request
containing messages and generation parameters. Audio processing is
handled elsewhere unless a separate multimodal endpoint has explicitly
been designed.

### Who owns conversation history?

In the normal LiveKit Agent pattern, the agent maintains its chat
context and sends the relevant messages on each LLM call. The gateway
should not assume that a request contains only the latest user sentence.

For example, the request can include:

-   A system/developer instruction.
-   Earlier user and assistant turns.
-   The latest user message.
-   Tool definitions or tool-call history, if the agent uses tools.

The model is stateless between independent requests unless the
serving/application layer explicitly implements state. The model can
only use the context provided to it for that inference.

------------------------------------------------------------------------

## 3. Phase 1 --- LiveKit constructs an OpenAI-compatible request

LiveKit's OpenAI plugin can connect to a custom OpenAI-compatible
endpoint using a configured `base_url`, model identifier and API key.
For Chat Completions-compatible endpoints, the request path is normally:

``` http
POST /v1/chat/completions
```

A simplified request looks like this:

``` http
POST /v1/chat/completions HTTP/1.1
Host: llm-gateway.example.internal
Authorization: Bearer <gateway-api-key>
Content-Type: application/json
Accept: text/event-stream
```

``` json
{
  "model": "qwen2.5-7b",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful voice assistant. Follow the agent instructions..."
    },
    {
      "role": "user",
      "content": "Explain the difference between a CPU and a GPU."
    }
  ],
  "temperature": 0.7,
  "max_tokens": 256,
  "stream": true
}
```

This is illustrative. The exact request fields depend on the LiveKit
plugin version, model, tools and configuration.

### Important API compatibility points

OpenAI-compatible means the gateway accepts the relevant Chat
Completions request and response conventions. It does **not** mean every
OpenAI feature is automatically supported.

The gateway and vLLM model configuration must agree on:

-   Message roles and content formats.
-   Supported sampling parameters.
-   Model identifiers.
-   Tool/function calling, if used.
-   Streaming chunk structure.
-   Finish reasons and usage fields.
-   Error response format.
-   Any provider-specific extra fields.

Do not assume that a parameter accepted by one provider is supported by
every vLLM version or model.

### Why `stream: true` matters

For a real-time voice agent, waiting for the entire response before
delivering any text can increase perceived latency. Streaming allows the
client to receive generated text incrementally.

The gateway must preserve the upstream stream rather than collecting the
complete response and returning it only at the end.

------------------------------------------------------------------------

## 4. Phase 2 --- Network ingress: DNS, TLS, Nginx and Uvicorn

The request first travels through the network.

### 4.1 DNS and connection

The LiveKit Agent resolves the gateway hostname and opens a TCP
connection. If the public or private endpoint uses HTTPS, TLS protects
the request in transit.

Production considerations:

-   Use HTTPS/TLS for traffic crossing an untrusted network.
-   Keep the vLLM service private.
-   Restrict inbound ports with firewall/network policy.
-   Use trusted certificates and renew them before expiry.
-   Avoid exposing Redis, PostgreSQL or internal inference ports
    publicly.

### 4.2 Nginx / reverse proxy

Nginx may perform:

-   TLS termination.
-   Reverse proxy routing to Uvicorn.
-   Maximum request-body enforcement.
-   Connection and request timeouts.
-   IP/network-level restrictions.
-   Access logging.
-   Basic edge-level rate limiting.
-   Streaming proxy configuration.

For SSE/streaming, proxy buffering must be configured appropriately. If
the proxy buffers the response, tokens may arrive at LiveKit in batches
or only after the response is complete, defeating low-latency streaming.

Nginx timeout values must accommodate legitimate long-running
generations without allowing connections to remain stuck indefinitely.

### 4.3 Uvicorn and ASGI

Uvicorn is an ASGI server. It accepts network connections and invokes
the FastAPI application through the ASGI interface.

ASGI is an asynchronous interface between the server and Python web
application. It supports HTTP and WebSocket connection lifecycles.

Important distinction:

-   Uvicorn handles network connections and ASGI communication.
-   FastAPI handles application routing, validation and dependencies.
-   Python `async` code can yield while waiting for I/O.
-   The model's GPU computation is performed by vLLM, not by the FastAPI
    event loop.

A slow upstream inference request should not block the event loop if the
gateway uses a proper asynchronous HTTP client and avoids blocking work
in async routes.

------------------------------------------------------------------------

## 5. Phase 3 --- Request context, authentication and authorization

Before expensive inference work is admitted, the gateway must establish
who is calling and what that caller is permitted to do.

### 5.1 Request ID / correlation ID

Assign or validate a request correlation ID early in the request
lifecycle.

Example:

``` text
request_id = req_01J...
```

Propagate it through:

``` text
Nginx → FastAPI → vLLM request metadata/logs → response logs
```

The ID helps connect gateway logs, upstream timings and error events. Do
not trust an arbitrary client-supplied ID as a security identity;
validate its length/format or generate a new internal ID.

Useful context fields include:

-   request_id
-   authenticated principal / API-key ID
-   tenant or organization ID
-   requested model
-   start timestamp
-   client/session correlation metadata, if permitted

Avoid logging raw API keys or full private prompts by default.

### 5.2 Authentication: who is calling?

A common gateway approach is a bearer API key:

``` http
Authorization: Bearer <secret-key>
```

A secure API-key design normally has:

1.  A key identifier or lookup prefix.
2.  A cryptographically random secret portion.
3.  A hash of the secret stored server-side, not plaintext.
4.  A status such as active, revoked or expired.
5.  A tenant/user owner.
6.  Scopes/permissions.
7.  Created, last-used and expiry timestamps.
8.  Rotation and revocation procedures.

The gateway extracts the presented key, identifies the candidate record,
hashes/verifies the secret using a suitable constant-time verification
approach, and checks that the key is active.

Do not put API keys in URLs or query strings. URLs are commonly captured
in access logs, browser histories and monitoring systems.

### 5.3 Authorization: what may this caller do?

Authentication and authorization are different:

-   Authentication: "Which client is this?"
-   Authorization: "What is this client allowed to do?"

Authorization can check:

-   Which model IDs the key may access.
-   Which maximum output-token limit applies.
-   Whether streaming is allowed.
-   Whether tool calling is allowed.
-   Which tenant's resources can be accessed.
-   Whether the key is permitted to use a premium model.
-   Whether the tenant has remaining quota.

Never let a caller choose an arbitrary internal upstream URL. Model
routing must use a server-side allowlist/configuration to prevent
server-side request forgery and unauthorized model access.

### 5.4 API key for LiveKit versus user identity

The API key configured in the LiveKit LLM plugin authenticates the
LiveKit Agent to our gateway. It does not automatically authenticate the
human participant in the LiveKit room.

If per-participant authorization or billing is required, the
agent/gateway design must carry a trusted participant/tenant identity
through a controlled mechanism. Do not trust a user ID supplied in an
arbitrary JSON field without verifying it.

### 5.5 Multi-tenant isolation

If multiple organizations use the same gateway:

-   Associate each API key with a tenant.
-   Enforce tenant model permissions and quotas.
-   Keep conversation/session data tenant-scoped if it is stored.
-   Avoid leaking tenant IDs or secrets in logs.
-   Consider separate inference deployments for tenants requiring strong
    isolation.

vLLM's shared server process is not a complete tenant-isolation
boundary. Gateway authorization and infrastructure separation are still
necessary.

------------------------------------------------------------------------

## 6. Phase 4 --- Request validation and admission control

Authentication answers who the caller is. Validation answers whether the
request is structurally and operationally acceptable.

### 6.1 Schema validation

Validate at least:

  -----------------------------------------------------------------------
  Field / property                    Validation
  ----------------------------------- -----------------------------------
  `model`                             Must map to an enabled model

  `messages`                          Must be a valid, non-empty list
                                      within configured limits

  `role`                              Must be an allowed role

  `content`                           Must have a supported content type
                                      and size

  `temperature`                       Must be within the selected model's
                                      supported range

  `top_p`                             Must be in the supported range

  `max_tokens` / output limit         Must be positive and within
                                      policy/model limits

  `stream`                            Must be a boolean

  tool definitions                    Must conform to supported schema
                                      and policy

  extra parameters                    Reject or explicitly allow known
                                      supported fields
  -----------------------------------------------------------------------

Also enforce:

-   Maximum HTTP body size at the proxy and application layers.
-   Maximum message count.
-   Maximum individual message size.
-   Maximum total input tokens.
-   Maximum expected output tokens.
-   Maximum context length for the selected model.
-   Maximum number of concurrent requests per caller/tenant.
-   Supported content types.

Do not rely only on character count for token limits. Token counts
depend on the tokenizer and content. Use the tokenizer corresponding to
the served model where practical, or use a conservative estimate before
inference and reconcile with actual usage where available.

### 6.2 Context-window arithmetic

The model has a maximum context length. A request must fit within it.

``` text
system/developer prompt tokens
+ conversation history tokens
+ latest user message tokens
+ tool/schema tokens
+ other template/special tokens
+ reserved output tokens
<= model context limit
```

For example, if the configured context limit is 32,768 tokens and the
request needs 2,000 tokens reserved for output, the input and formatting
overhead must fit in the remaining budget.

Do not assume that the JSON character length equals the model token
count.

### 6.3 Admission control

A request can be valid but still not be safe to admit immediately.

Admission control decides whether the gateway should:

-   Forward the request now.
-   Wait briefly in a bounded queue.
-   Reject it with a retryable overload response.
-   Route it to another healthy model server.

A bounded queue is important. An unlimited queue converts overload into
growing latency and memory consumption.

For real-time voice traffic, a request that waits too long may no longer
be useful. The gateway should have a queue-wait budget and return a
clear overload response when it cannot serve the request in time.

------------------------------------------------------------------------

## 7. Phase 5 --- Rate limiting and quota enforcement

Rate limiting controls how quickly a caller can submit requests. Quotas
control how much total resource a caller can consume.

These are related but not identical.

### 7.1 Useful rate-limit dimensions

  Limit                       Example purpose
  --------------------------- ----------------------------------------------
  Requests per minute (RPM)   Prevent request floods
  Tokens per minute (TPM)     Control variable-size inference cost
  Concurrent requests         Prevent one tenant occupying all slots
  Input-token maximum         Protect prefill/context capacity
  Output-token maximum        Bound generation work
  Daily/monthly usage         Billing or subscription quota
  Per-model limit             Protect expensive or scarce models
  Per-IP edge limit           Basic abuse protection before API-key lookup

The actual numbers must come from load testing and product policy; there
is no universal correct RPM/TPM value.

### 7.2 Why request count alone is insufficient

A request with a 100-token prompt and 100-token answer is not equivalent
to a request with a 20,000-token prompt and 2,000-token answer.

A practical token quota accounts for input and output usage:

``` text
estimated reservation = estimated input tokens + maximum output tokens
actual usage           = actual input tokens + actual output tokens
```

A robust accounting pattern is:

1.  Estimate/reserve capacity before inference.
2.  Admit only if the tenant has sufficient quota.
3.  Run the request.
4.  Read actual token usage when available.
5.  Reconcile the reservation against actual usage.
6.  Release unused reservation on success, failure or cancellation.

Be careful with streaming: the final usage information may arrive only
at the end, and some servers/clients require explicit usage-in-stream
configuration. If usage is unavailable, record that fact rather than
inventing an exact value.

### 7.3 Redis and distributed rate limiting

A process-local Python dictionary or semaphore is not sufficient when
multiple FastAPI workers/replicas exist. Each process would have a
separate counter.

Redis can provide shared counters and atomic operations for distributed
rate limiting.

Common algorithms:

-   Fixed window: simple counter per time window; can allow boundary
    bursts.
-   Sliding window: smoother accounting, more precise but more involved.
-   Token bucket: allows controlled bursts while enforcing a long-term
    rate.
-   Leaky bucket: smooths traffic at a configured drain rate.

Use atomic Redis operations or scripts for multi-step updates. Define
what happens if Redis is unavailable: fail closed for
sensitive/high-cost workloads, or use a deliberately bounded degraded
mode if product requirements permit. Do not silently bypass all limits.

### 7.4 Rate-limit response

A rejected request should return a clear HTTP 429 response and, where
appropriate, a `Retry-After` header. The client should not retry in a
tight loop.

------------------------------------------------------------------------

## 8. Phase 6 --- Large system prompts and prompt design

This is especially important in our use case because LiveKit agents may
send a large system instruction on every LLM request.

A system prompt can contain:

-   Agent identity and role.
-   Business rules.
-   Conversation style.
-   Safety constraints.
-   Domain knowledge.
-   Tool usage instructions.
-   Output formatting requirements.
-   Escalation rules.
-   Organization-specific instructions.

A large system prompt is not automatically a problem, but repeatedly
processing it can increase prefill latency and consume KV-cache
capacity.

### 8.1 Separate stable instructions from dynamic information

A useful conceptual structure is:

``` text
[STABLE PREFIX]
1. Stable system/agent identity
2. Stable behavior and response rules
3. Stable tool definitions and schemas
4. Stable business/domain instructions

[DYNAMIC SUFFIX]
5. Tenant-specific or session-specific data
6. Relevant conversation history
7. Current user message
```

The goal is to keep the common, stable prefix
byte-for-byte/token-for-token consistent after chat-template processing,
while placing changing content later.

This is a prompt organization strategy, not a guarantee of a cache hit.

### 8.2 What breaks prefix reuse?

Even small changes near the beginning of the prompt can change the token
prefix and reduce reuse.

Examples:

-   Inserting a changing timestamp at the top.
-   Adding a random request ID inside the system prompt.
-   Reordering instruction sections.
-   Changing whitespace or formatting.
-   Inserting different tenant-specific instructions before shared
    instructions.
-   Changing tool schemas between requests.
-   Changing the model or tokenizer/chat template.
-   Changing earlier conversation history.

A better ordering is:

``` text
Stable shared system instructions
Stable shared tool definitions
Tenant-specific instructions
Session-specific context
Conversation history
Latest user message
```

The exact order must still preserve instruction precedence and
application correctness. Do not move security-critical or
higher-priority instructions into a lower-priority role merely to chase
cache hits.

### 8.3 Do not put secrets in system prompts

System prompts are model input, not a secure secret store.

Do not include:

-   API keys.
-   Database passwords.
-   Private signing keys.
-   Internal credentials.
-   Secrets that the model does not need.

A prompt instruction such as "never reveal this secret" is not a
security boundary. Enforce permissions in backend code.

### 8.4 Should the gateway store and inject the system prompt?

There are two broad patterns:

**Pattern A --- LiveKit sends the full context.**

The gateway validates and forwards the messages. This keeps the gateway
relatively stateless and preserves LiveKit's ownership of agent context.

**Pattern B --- Gateway manages prompt templates.**

The gateway selects a server-controlled template by model/tenant/version
and composes the request. This can centralize policy and make prompt
versions consistent, but it changes the gateway's responsibility and
must be designed carefully.

If the gateway injects a stable system prompt, it must avoid duplicate
system instructions and define precedence clearly. It must also ensure
callers cannot override protected instructions through a conflicting
message.

Do not silently rewrite arbitrary messages. Make prompt composition an
explicit, versioned policy.

------------------------------------------------------------------------

## 9. Phase 7 --- vLLM Automatic Prefix Caching (APC)

### 9.1 First understand the KV cache

During transformer inference, the model computes attention Key and Value
tensors for tokens. These tensors are stored in the KV cache so the
model does not have to recompute the entire previous sequence for every
generated token.

Without KV caching, autoregressive generation would repeatedly recompute
prior token representations, which is inefficient.

KV cache consumes GPU memory. Longer contexts and more concurrent
sequences generally require more KV-cache capacity.

### 9.2 What Automatic Prefix Caching does

vLLM Automatic Prefix Caching (APC) reuses KV-cache blocks from previous
requests when a new request has a matching prefix.

It does **not** cache the final text answer. It caches reusable
inference state for matching prompt tokens.

Conceptual example:

Request A:

``` text
[Shared system prompt]
[Shared tool definitions]
[Question A]
```

Request B:

``` text
[Shared system prompt]
[Shared tool definitions]
[Question B]
```

If the tokenized prefix matches and the relevant cache blocks are still
available, request B can reuse the cached prefix computation and prefill
only the uncached suffix.

### 9.3 How matching works conceptually

vLLM divides KV cache into blocks. Its prefix-caching implementation
uses hashes based on block token content and preceding-prefix
information to identify reusable blocks.

Conceptually:

``` text
Request A:
| Block 1: shared system | Block 2: shared rules |
| Block 3: question A    |

Request B:
| Block 1: shared system | Block 2: shared rules |
| Block 3: question B    |

Request B may reuse:
| Block 1 | Block 2 |

Request B must process:
| Block 3: question B |
```

This is a conceptual diagram. Actual block sizes, matching boundaries
and implementation details depend on the vLLM version and model/cache
configuration.

### 9.4 What APC improves

APC reduces repeated prefill computation for matching prompt prefixes.
It can improve:

-   Time to first token (TTFT).
-   Prefill workload.
-   Throughput under repeated-prefix workloads.
-   Efficiency for repeated large system prompts.
-   Efficiency for repeated long conversation prefixes or documents.

### 9.5 What APC does not improve

APC does not eliminate:

-   The HTTP request body transfer.
-   Request authentication or validation.
-   Prompt tokenization/processing required to identify the input.
-   Processing of a new, unmatched suffix.
-   Autoregressive generation of new output tokens.
-   The need to fit the prompt and output within the context limit.

APC mainly helps the prefill portion. If generation of a very long
answer dominates latency, APC may have a smaller effect on total
response time.

A cache hit is also not guaranteed. Cache blocks may have been evicted,
the prefix may differ, or the matching portion may not reach a reusable
block boundary.

### 9.6 Large system prompt: practical APC strategy

For our LiveKit-connected gateway:

1.  Keep stable system instructions consistent across calls.
2.  Keep shared tool schemas stable when possible.
3.  Put changing timestamps, request IDs and user-specific data after
    the stable prefix.
4.  Avoid unnecessary prompt formatting changes.
5.  Keep model/tokenizer/chat-template configuration consistent.
6.  Enable APC in the vLLM serving configuration for versions that
    support it.
7.  Measure cache hit behavior and TTFT under realistic concurrent
    traffic.
8.  Do not assume that every request will hit the cache.

vLLM's serving option is commonly exposed as `--enable-prefix-caching`
(and the Python engine configuration has used
`enable_prefix_caching=True`). Verify the option against the exact
installed vLLM version rather than copying a flag from a different
release.

### 9.7 Cache memory and eviction

The prefix cache uses KV-cache blocks, which are a finite GPU memory
resource.

If the cache is under pressure, reusable blocks can be evicted. A future
request may then need to recompute that prefix.

Therefore:

-   APC does not create unlimited free memory.
-   More cache retention competes with active inference capacity.
-   High concurrency and long contexts can pressure KV-cache capacity.
-   A cache hit does not mean the request has no GPU cost.
-   Benchmark both latency and throughput; do not judge only by
    cache-hit rate.

### 9.8 Multi-tenant security and cache salting

Prefix-cache reuse can create a timing side channel in shared
deployments: a caller may observe a faster TTFT when a guessed prefix is
already cached.

Current vLLM versions document an optional `cache_salt` field for
prefix-cache isolation. The salt is mixed into the first block's hash so
that requests with different salts do not share prefix-cache blocks.

A gateway serving multiple tenants can derive a stable, secret salt per
tenant and pass it to vLLM when the installed version and endpoint
support it.

Important rules:

-   Do not accept an arbitrary tenant salt directly from an untrusted
    caller.
-   Derive or look up the salt server-side.
-   Use the same tenant salt when that tenant is intentionally allowed
    to share prefix-cache blocks.
-   Use different salts across trust boundaries.
-   Treat salt as defense in depth, not complete tenant isolation.
-   vLLM's shared process still does not provide full tenant isolation.

Cache salting reduces cross-tenant prefix reuse and its timing signal,
but it does not replace authorization, network isolation or separate
deployments where strong isolation is required.

### 9.9 APC versus Redis response caching

These are different technologies:

  -----------------------------------------------------------------------
  vLLM APC                            Redis response cache
  ----------------------------------- -----------------------------------
  Stores/reuses KV-cache blocks       Stores completed application
                                      responses

  Reuses matching prompt computation  May return a previously generated
                                      answer

  Helps prefill latency               Can avoid inference entirely for a
                                      valid cache hit

  Managed by inference engine         Managed by gateway/application

  Depends on token-prefix matching    Depends on cache key, TTL and
  and cache availability              correctness rules
  -----------------------------------------------------------------------

A response cache for conversational AI is risky unless the key includes
all relevant context, model version, prompt version, tenant, parameters
and authorization scope. For many voice-agent workloads, response
caching is not appropriate because each turn is context-dependent.

------------------------------------------------------------------------

## 10. Phase 8 --- FastAPI forwards the request to vLLM

After authentication, validation, quota checks and admission control,
the gateway selects an allowed inference backend.

Example topology:

``` text
LiveKit
   |
   v
FastAPI Gateway
   |
   v
Model Router
   |
   +---- qwen2.5-7b ------> vLLM server A
   |
   +---- fine-tuned-qwen -> vLLM server B
   |
   +---- llama-model -----> vLLM server C
```

A model name supplied by the client should map to a known server-side
configuration. It should never become an arbitrary URL.

### 10.1 Async HTTP client and connection pooling

The gateway usually communicates with vLLM over internal HTTP.

A production gateway should use a long-lived asynchronous HTTP client
with connection pooling rather than creating a new client for every
request.

Connection pooling can reuse TCP connections and reduce connection setup
overhead.

The client configuration should consider:

-   Connect timeout.
-   Pool-acquisition timeout.
-   Write timeout.
-   Read/stream idle timeout.
-   Maximum connections.
-   Maximum keep-alive connections.
-   Upstream health and error handling.

Timeouts must be chosen for the workload. A short read timeout can
incorrectly terminate a legitimate long generation; an unlimited timeout
can leave stuck requests consuming resources.

### 10.2 Avoid blocking the FastAPI event loop

The gateway spends much of its time waiting for network I/O from vLLM.
Async HTTP operations allow the event loop to serve other requests while
one request waits.

Do not run long blocking CPU work inside an async endpoint. Do not load
a second copy of the LLM model into every FastAPI worker. The model
should be owned by the inference service.

------------------------------------------------------------------------

## 11. Phase 9 --- vLLM receives and schedules the request

The vLLM server receives the OpenAI-compatible request and performs its
inference-serving workflow.

A simplified lifecycle:

``` text
HTTP request
    |
    v
Request parsing and validation
    |
    v
Model/tokenizer selection
    |
    v
Prompt construction / chat template
    |
    v
Tokenization
    |
    v
Prefix-cache lookup
    |
    v
Scheduler / waiting queue
    |
    v
Prefill computation for uncached tokens
    |
    v
Autoregressive decode
    |
    v
Stream generated token chunks
```

Exact internal components vary by vLLM version and configuration.

### 11.1 Tokenization and chat template

The messages are converted into the model's expected token sequence. The
chat template adds model-specific role markers, separators and control
tokens.

Two requests that look similar as JSON do not necessarily have identical
token prefixes if their templates or formatting differ.

This is another reason to keep the model and prompt format stable.

### 11.2 Prefill

Prefill processes the input prompt and computes the model states/KV
cache needed for generation.

Large prompts create more prefill work. APC can reduce the repeated
prefill work for matching cached prefixes.

### 11.3 Decode

During decode, the model generates new tokens autoregressively. Each new
token depends on the prior context and generated tokens.

Decode performance is affected by model size, hardware, batch
composition, output length and memory/bandwidth behavior.

### 11.4 Continuous batching

vLLM can schedule multiple requests together using continuous batching.
Requests can enter and leave the active batch as their generation
progresses rather than requiring one fixed batch to finish before
another starts.

This improves GPU utilization and serving throughput.

The gateway should control admission and overload. It should not try to
replace vLLM's internal GPU scheduler with a simplistic
application-level queue.

------------------------------------------------------------------------

## 12. Phase 10 --- Streaming response: vLLM → FastAPI → LiveKit

With `stream: true`, vLLM returns an SSE stream.

A simplified stream can look like:

``` text
data: {"choices":[{"delta":{"content":"A CPU"}}]}

data: {"choices":[{"delta":{"content":" handles"}}]}

data: {"choices":[{"delta":{"content":" general"}}]}

data: {"choices":[{"delta":{"content":" processing."}}]}

data: [DONE]
```

Real responses can contain additional fields, IDs, finish reasons and
optional usage information.

### 12.1 The gateway's streaming responsibility

The gateway should:

1.  Receive the upstream stream.
2.  Preserve valid SSE framing and chunk boundaries as appropriate.
3.  Forward chunks as soon as they are available.
4.  Avoid buffering the entire response.
5.  Preserve the final completion marker.
6.  Handle upstream errors.
7.  Detect client disconnects.
8.  Cancel/close the upstream request when the client no longer needs
    it.
9.  Release concurrency and quota reservations correctly.

Do not assume that the upstream and downstream chunk boundaries will
always be identical. HTTP libraries can split or combine network reads.
The gateway must handle SSE framing correctly.

### 12.2 What happens to the response?

``` text
vLLM token generation
       |
       v
vLLM SSE chunk
       |
       v
FastAPI streaming response
       |
       v
Nginx (streaming, no unwanted buffering)
       |
       v
LiveKit OpenAI-compatible client
       |
       v
LiveKit agent consumes text
       |
       v
TTS produces audio
       |
       v
Participant hears the answer
```

### 12.3 Finish reasons

The final response may indicate:

-   `stop`: normal stopping condition.
-   `length`: output limit reached.
-   `tool_calls`: the model returned a tool call, where supported.

The gateway should preserve compatible finish information. A tool call
is not ordinary assistant prose; the LiveKit agent/application must
understand and execute the tool according to its own policy.

### 12.4 Usage accounting

If the server provides input/output token usage, the gateway can record
it. Streaming usage may require a specific server/client option and may
be delivered in a final chunk.

Do not calculate exact billing from character counts. Use actual
tokenizer/server usage where available.

------------------------------------------------------------------------

## 13. Phase 11 --- Cancellation and interruption

This is critical for voice conversations.

Example:

1.  User asks a question.
2.  LiveKit sends an LLM request.
3.  vLLM begins generation.
4.  LiveKit receives and speaks the response.
5.  The user interrupts.
6.  LiveKit cancels the old generation task.

Cancellation must propagate through the whole chain:

``` text
Participant interruption
        |
        v
LiveKit cancels LLM task
        |
        v
Client closes/cancels HTTP stream
        |
        v
FastAPI detects disconnect/cancellation
        |
        v
FastAPI closes upstream vLLM stream
        |
        v
vLLM aborts the corresponding request
        |
        v
Resources become available for other work
```

Important: a downstream disconnect does not automatically prove that the
upstream model request has been aborted. The gateway must explicitly
handle cancellation and close/cancel the upstream request using the
supported HTTP client and vLLM behavior.

Also distinguish:

-   **Client cancellation:** The caller no longer wants the response.
-   **Timeout:** The request exceeded a configured time budget.
-   **Server failure:** The inference service failed.
-   **Normal completion:** The model finished successfully.

These events should have different metrics and logs.

------------------------------------------------------------------------

## 14. Phase 12 --- Errors, retries and overload behavior

Production systems must assume that dependencies can fail.

  -----------------------------------------------------------------------
  Failure                             Expected gateway behavior
  ----------------------------------- -----------------------------------
  Missing/invalid API key             Reject with 401

  Authenticated but not permitted     Reject with 403

  Invalid request schema              Reject with 400/422 according to
                                      API contract

  Request too large                   Reject with 413

  Rate limit exceeded                 Reject with 429 and appropriate
                                      retry information

  Queue/admission limit exceeded      Return a documented overload
                                      response, commonly 429 or 503

  vLLM unavailable                    Return a controlled 503-style
                                      upstream error

  Upstream timeout                    Return a controlled timeout/gateway
                                      error

  Model/context limit exceeded        Return a clear validation or
                                      upstream error

  Client disconnect                   Cancel upstream work where
                                      possible; do not treat as a normal
                                      successful completion
  -----------------------------------------------------------------------

### Retry caution

Blindly retrying an inference request can double GPU work.

For example, the model may have started generating but the gateway may
have lost the connection before receiving the response. Retrying can
create a second generation.

Retry only where the failure is known to be transient and the operation
is safe to repeat. Use bounded retries, exponential backoff with jitter
and an overall deadline. Do not retry invalid requests, authentication
failures or rate-limit rejections automatically.

For streaming, once response headers/body have been sent, the gateway
cannot replace the stream with a new HTTP status code. It must use a
documented in-stream error convention or terminate the stream safely.

------------------------------------------------------------------------

## 15. Phase 13 --- Observability and performance measurement

A production LLM gateway needs application-level and inference-level
visibility.

### 15.1 Important latency metrics

  -----------------------------------------------------------------------
  Metric                              Meaning
  ----------------------------------- -----------------------------------
  Ingress latency                     Time spent reaching/entering the
                                      gateway

  Authentication/validation latency   Gateway policy and schema
                                      processing

  Queue wait                          Time waiting for
                                      admission/inference scheduling

  TTFT                                Time to first generated token

  Inter-token latency / TPOT          Time between generated output
                                      tokens, measured consistently

  Total generation time               Time until generation completes

  End-to-end latency                  Full request duration as observed
                                      by the client

  Input tokens                        Prompt token count

  Output tokens                       Generated token count

  Tokens/sec                          Generation throughput using a
                                      clearly defined numerator and
                                      interval
  -----------------------------------------------------------------------

For voice applications, TTFT and inter-token timing matter because they
influence how quickly the agent can begin speaking and how smoothly text
can be supplied to TTS.

### 15.2 Metrics to monitor

**Gateway metrics:**

-   Requests by status code.
-   Requests by model/tenant.
-   Authentication failures.
-   Rate-limit rejections.
-   Active streaming requests.
-   Upstream connection errors.
-   Timeouts.
-   Client cancellations.
-   Queue length and wait time.
-   Input/output token usage.

**vLLM metrics:**

-   Running requests.
-   Waiting requests.
-   Prompt-token throughput.
-   Generation-token throughput.
-   Time to first token.
-   Time per output token.
-   KV-cache usage.
-   Prefix-cache hit metrics, where exposed by the installed version.
-   GPU memory and execution health.

**Infrastructure metrics:**

-   GPU utilization and VRAM.
-   CPU and RAM.
-   Network throughput.
-   Redis latency and errors.
-   PostgreSQL pool usage, if used.
-   Nginx active connections and upstream errors.

### 15.3 Logging and privacy

Use structured logs with request IDs. Record enough metadata to diagnose
issues, but do not log secrets or full private prompts by default.

A useful event might contain:

``` json
{
  "request_id": "req_example",
  "tenant_id": "tenant_example",
  "model": "qwen2.5-7b",
  "stream": true,
  "input_tokens": 1800,
  "output_tokens": 120,
  "ttft_ms": 240,
  "total_duration_ms": 2100,
  "status": "completed"
}
```

This is an illustrative record, not a claim about our current logging
implementation.

Use Prometheus for metrics, Grafana for dashboards and OpenTelemetry for
tracing across Nginx, FastAPI and upstream services where
instrumentation is available.

------------------------------------------------------------------------

## 16. Phase 14 --- Production deployment and scaling

### 16.1 Separate API scaling from inference scaling

FastAPI is primarily handling network I/O and application logic. vLLM is
managing GPU inference.

They have different resource profiles and should be deployable/scalable
independently.

``` text
                  Load Balancer
                       |
             +---------+---------+
             |                   |
          FastAPI 1            FastAPI 2
             |                   |
             +---------+---------+
                       |
                  Model Router
                       |
             +---------+---------+
             |                   |
          vLLM GPU 1          vLLM GPU 2
             |                   |
          Model A              Model B
```

This is a conceptual multi-instance deployment. A single RTX 3060 12 GB
machine has much more limited capacity and cannot be treated as if it
already has this topology.

### 16.2 Health checks

Distinguish:

-   **Liveness:** Is the API process alive?
-   **Readiness:** Can the API safely accept requests?
-   **Inference health:** Is the selected vLLM backend reachable and
    serving?
-   **Dependency health:** Are required services such as Redis/database
    available?

Do not make every liveness check depend on GPU inference. A temporary
model-server issue should not necessarily cause the API process to
restart continuously.

### 16.3 Graceful shutdown

During deployment or restart:

1.  Stop accepting new work.
2.  Allow a bounded period for active requests.
3.  Cancel or safely terminate remaining streams.
4.  Close HTTP clients and Redis/database pools.
5.  Exit cleanly.

Long-lived streaming connections need explicit shutdown behavior.

### 16.4 Capacity planning

Capacity depends on more than model parameter count.

Measure:

-   Model weights and quantization.
-   GPU memory available for KV cache.
-   Context length.
-   Concurrent sequences.
-   Input prompt length.
-   Output length.
-   Prefix-cache hit rate.
-   Batching behavior.
-   TTFT and token generation rate.

A 12 GB GPU should be benchmarked with realistic prompt sizes and
concurrent LiveKit sessions. Do not assume that a model fitting in VRAM
means the desired concurrency will also fit.

------------------------------------------------------------------------

## 17. End-to-end request timeline

This is the complete conceptual lifecycle:

``` text
LIVEKIT
  |
  | 1. User turn is ready
  | 2. Agent assembles chat context
  | 3. Agent sends Chat Completions request
  v
NETWORK / NGINX
  |
  | 4. DNS, TCP and TLS
  | 5. Reverse proxy and body/connection limits
  v
UVICORN / FASTAPI
  |
  | 6. ASGI request accepted
  | 7. Request ID and context created
  | 8. API key authenticated
  | 9. Tenant/model permission checked
  | 10. Schema and context limits validated
  | 11. Rate limits and quota checked
  | 12. Admission/concurrency checked
  v
INFERENCE CLIENT
  |
  | 13. Select approved vLLM backend
  | 14. Reuse pooled HTTP connection
  | 15. Forward request
  v
vLLM
  |
  | 16. Parse request and apply chat template
  | 17. Tokenize prompt
  | 18. Check Automatic Prefix Cache
  | 19. Schedule request
  | 20. Prefill uncached prompt tokens
  | 21. Decode output tokens
  v
STREAMING RESPONSE
  |
  | 22. vLLM emits SSE chunks
  | 23. FastAPI forwards chunks
  | 24. Nginx preserves streaming
  | 25. LiveKit consumes generated text
  v
LIVEKIT AGENT
  |
  | 26. Agent forwards text to TTS
  | 27. Audio is delivered to participant
  |
  | 28. Usage and latency are recorded
  | 29. Reservations/concurrency slots released
  v
REQUEST COMPLETE
```

If the participant interrupts at step 22--25, the cancellation path
should travel upstream and abort the old generation where supported.

------------------------------------------------------------------------

## 18. Production checklist

Use this as a conceptual review checklist. It does not indicate which
features are already present in our existing codebase.

### API and protocol

-   [ ] OpenAI Chat Completions request compatibility
-   [ ] Correct model-name mapping
-   [ ] Correct SSE framing and `[DONE]` behavior
-   [ ] Correct finish reasons and supported usage fields
-   [ ] Consistent error responses
-   [ ] Request/correlation IDs

### Security

-   [ ] API keys stored as hashes
-   [ ] Key rotation and revocation
-   [ ] Authentication and authorization separated
-   [ ] Tenant/model permissions
-   [ ] TLS and private inference network
-   [ ] Request body limits
-   [ ] Secrets and prompts protected in logs
-   [ ] No arbitrary upstream URL routing

### Validation and traffic control

-   [ ] Message/role/schema validation
-   [ ] Input-token and context-window limits
-   [ ] Output-token limits
-   [ ] RPM and TPM controls
-   [ ] Concurrent request limits
-   [ ] Bounded queue/admission control
-   [ ] Distributed counters if multiple API replicas exist
-   [ ] Correct quota reservation and reconciliation

### Prompt and inference efficiency

-   [ ] Stable system prompt prefix
-   [ ] Dynamic content placed after stable content where semantically
    appropriate
-   [ ] No secrets in system prompts
-   [ ] APC enabled and verified for installed vLLM version
-   [ ] Prefix-cache metrics monitored
-   [ ] Cache salting considered for multi-tenant deployments
-   [ ] Context length and KV-cache capacity benchmarked
-   [ ] Continuous batching and concurrency benchmarked

### Streaming and reliability

-   [ ] No unwanted Nginx buffering
-   [ ] Async upstream HTTP client with connection pooling
-   [ ] Connect/read/write/pool timeouts
-   [ ] Client disconnect detection
-   [ ] Upstream cancellation propagation
-   [ ] Bounded retries and deadlines
-   [ ] Graceful shutdown
-   [ ] Overload responses and recovery behavior

### Observability and operations

-   [ ] Structured logs
-   [ ] Prometheus metrics
-   [ ] Grafana dashboards
-   [ ] OpenTelemetry tracing
-   [ ] TTFT and inter-token latency
-   [ ] vLLM queue and KV-cache metrics
-   [ ] GPU/CPU/RAM monitoring
-   [ ] Load tests using realistic prompt and output lengths
-   [ ] Alerts, backups and deployment rollback

------------------------------------------------------------------------

## 19. Key concepts to remember

  -----------------------------------------------------------------------
  Concept                             Remember it as
  ----------------------------------- -----------------------------------
  FastAPI                             Application-facing inference
                                      gateway

  LiveKit                             Real-time session and agent
                                      orchestration

  vLLM                                LLM inference and scheduling engine

  OpenAI compatibility                Shared API contract, not shared
                                      model implementation

  Authentication                      Identifies the calling client

  Authorization                       Determines what that client may use

  Validation                          Determines whether a request is
                                      acceptable

  Rate limiting                       Controls request/token arrival rate

  Admission control                   Decides whether work can be
                                      accepted now

  Prefill                             Processes input prompt tokens

  Decode                              Generates output tokens

  KV cache                            Stores attention Key/Value states
                                      for sequence processing

  Prefix caching                      Reuses KV-cache blocks for matching
                                      prompt prefixes

  SSE                                 Transports generated output
                                      incrementally

  Cancellation                        Stops work that is no longer needed

  TTFT                                Time until the first output token

  Backpressure                        Prevents overload from propagating
                                      uncontrollably
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 20. Official technical references

Use the documentation for the exact versions deployed in the
environment. vLLM flags, metrics, API fields and internal implementation
can change between releases.

1.  [LiveKit --- OpenAI-compatible
    LLMs](https://docs.livekit.io/agents/models/llm/openai-compatible-llms/)
    --- Custom `base_url`, model and API-key integration.
2.  [vLLM --- OpenAI-compatible
    server](https://docs.vllm.ai/en/latest/serving/openai_compatible_server/)
    --- Chat Completions API and serving behavior.
3.  [vLLM --- Automatic Prefix
    Caching](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/)
    --- Prefix reuse and APC behavior.
4.  [vLLM --- Prefix caching
    design](https://docs.vllm.ai/en/latest/design/prefix_caching/) ---
    Block-based cache implementation.
5.  [vLLM --- Security](https://docs.vllm.ai/en/latest/usage/security/)
    --- Cache salting and multi-tenant security considerations.
6.  [FastAPI ---
    Middleware](https://fastapi.tiangolo.com/tutorial/middleware/) ---
    Request/response middleware.
7.  [FastAPI ---
    Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)
    --- Dependency injection and shared request logic.
8.  [Uvicorn --- Deployment](https://www.uvicorn.org/deployment/) ---
    ASGI server deployment.
9.  [OpenTelemetry ---
    Python](https://opentelemetry.io/docs/languages/python/) --- Tracing
    and instrumentation.
10. [Prometheus ---
    Instrumentation](https://prometheus.io/docs/practices/instrumentation/)
    --- Metrics design.

------------------------------------------------------------------------

## Final architecture principle

Our system is an inference gateway serving real-time AI agents.

The most important engineering objective is not simply maximizing tokens
per second. It is balancing:

-   Low and predictable time to first token.
-   Consistent streaming behavior.
-   Safe concurrency and GPU utilization.
-   Correct prompt/context handling.
-   Strong authentication and tenant controls.
-   Reliable cancellation during interruptions.
-   Accurate usage accounting.
-   Observability and predictable failure handling.

A gateway is production-ready only when it can maintain these properties
under realistic concurrent load, including failures and overload---not
merely when one request successfully generates a response.
