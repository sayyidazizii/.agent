# STRICT SECURITY, PERFORMANCE & REGRESSION GUARDRAILS (ZERO-TRUST)

You must strictly adhere to the following negative constraints and prohibitions during any code generation, refactoring, or modification. Any violation will result in task rejection.

## 1. SECRETS & CREDENTIALS
- STRICTLY PROHIBITED: Hardcoding credentials, API keys, private tokens, passwords, JWT secrets, encryption keys, or connection strings into source code.
- Always use environment variables, secret managers, or designated configuration stores.

## 2. FILE HANDLING & SANITIZATION (STRICT ZERO-TRUST)
- STRICTLY PROHIBITED: Using user-supplied filenames directly in filesystem operations. Filenames must be sanitized or replaced with cryptographically secure identifiers (e.g., UUIDv4, ULID, or nanoid).
- STRICTLY PROHIBITED: Relying solely on client-supplied MIME types (`Content-Type` header) or file extensions. File uploads MUST be validated using magic bytes/file signatures.
- STRICTLY PROHIBITED: Storing uploaded files inside public or web-executable directories. Files must be stored outside web root or in object storage (S3/GCS) with execution permissions explicitly stripped (`chmod 0644` or non-executable flags).
- STRICTLY PROHIBITED: Unsafe archive extraction (Zip Slip vulnerability). Path resolutions during decompression must be validated to never escape the designated target directory.
- STRICTLY PROHIBITED: Processing files without size limits, memory thresholds, and stream controls (prevent DoS and Decompression Bombs).

## 3. STRICT INPUT VALIDATION & WHITELISTING
- STRICTLY PROHIBITED: Using blacklist-based filtering. All inputs must adhere to an EXPLICIT ALLOWLIST (positive security model) via strict schemas (e.g., Zod, Joi, Pydantic, typed DTOs).
- STRICTLY PROHIBITED: Loose type conversions or unconstrained wildcards in schemas (e.g., untyped `any`, unvalidated JSON blobs, loose dicts).
- STRICTLY PROHIBITED: Bypassing boundary sanitization. All incoming data (URL parameters, headers, query strings, cookies, request body) must be strictly validated before touching application logic.

## 4. ARBITRARY BEHAVIOR PREVENTION (RCE, ARBITRARY READ/WRITE, SSRF)
- STRICTLY PROHIBITED: Arbitrary File Read/Write/Delete. User input must never determine base paths. Always canonicalize paths and assert that `path.resolve(baseDir, inputPath).startsWith(baseDir)`.
- STRICTLY PROHIBITED: Dynamic code execution (e.g., `eval()`, `exec()`, `child_process.exec()` with unsanitized arguments, or `dangerouslySetInnerHTML`).
- STRICTLY PROHIBITED: Dynamic imports or module loading based on user input (e.g., dynamic `require(userInput)`, `import(userInput)`).
- STRICTLY PROHIBITED: Insecure deserialization of untrusted payloads (e.g., unsafe YAML loaders like `yaml.load()`, Python `pickle`, Java/PHP native serialization, or `node-serialize`).
- STRICTLY PROHIBITED: Arbitrary Outbound Requests (SSRF). Outbound HTTP requests using user-supplied URLs must strictly block private IP ranges (RFC 1918, `127.0.0.1`, `::1`, and cloud metadata endpoints such as `169.254.169.254`) and enforce protocol allowlisting (`https:` only).

## 5. DATABASE & INJECTION DEFENSES
- STRICTLY PROHIBITED: Unsanitized raw database queries or string concatenation in queries (prevent SQL/NoSQL/ORM Injection). Always use parameterized bindings or safe ORM methods.
- STRICTLY PROHIBITED: Leaking stack traces, file system paths, or raw database errors in client-facing HTTP/API responses.

## 6. EFFICIENCY & PERFORMANCE
- STRICTLY PROHIBITED: Suboptimal algorithms (e.g., nested O(N^2) loops where O(N) or O(log N) hash maps/indexing can be used).
- STRICTLY PROHIBITED: Resource and memory leaks (unclosed streams, unreleased locks, hanging event listeners, unclosed database sessions).
- STRICTLY PROHIBITED: Blocking synchronous I/O operations inside asynchronous or high-throughput request cycles.
- STRICTLY PROHIBITED: Adding bloated third-party dependencies when native language features or existing project utilities suffice.

## 7. CODE INTEGRITY & ZERO REGRESSION
- STRICTLY PROHIBITED: Modifying or deleting unrelated files, interfaces, or working features outside the explicit target scope.
- STRICTLY PROHIBITED: Using hallucinated, unverified, or deprecated third-party packages.
- Always maintain backwards compatibility of existing APIs and public contracts.

## 8. VERSION CONTROL & GIT OPERATIONS (STRICT BOUNDARY)
- STRICTLY PROHIBITED: Running `git commit`, `git push`, or modifying remote branches under any circumstances without explicit, written confirmation from the user.
- STRICTLY PROHIBITED: Running destructive Git commands that can erase uncommitted work (e.g., `git reset --hard`, `git clean -fd`, `git checkout -- .`, `git restore .`).
- Permitted Git actions are READ-ONLY: You are ONLY allowed to inspect Git state using `git status`, `git diff`, and `git log` to review your own changes.
- Final commit and push actions MUST ALWAYS be left to the human developer.
