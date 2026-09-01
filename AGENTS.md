# extension-docs agent instructions

## Required startup

Before investigating, planning, or changing anything:

1. Read `../AGENTS.md`. Its engineering, documentation, testing, security, Git, and deployment rules are mandatory here.
2. Read `../README.md` for platform-wide context and documentation routing.
3. Read this repository's `readme.md`.
4. Read `../test-and-build.md` before using or adding validation tooling.

Only this repository's root README is mandatory. Nested READMEs are contextual
and should only be read when the task directly concerns their directory.

Before concluding that server, database, Help Scout, logging, or other access
information is unavailable, follow the documentation routing in `../AGENTS.md`
and `../README.md`.

## Repository scope and related repositories

- This repository owns the public extension language guide, examples, public extension JSON definitions, and their developer and configuration guides.
- `ITM.Connector` owns the extension parser and execution model. `ITM.Scheduler` owns scheduled execution, and `ITM.Framework` owns shared action and event contracts. Treat implementation changes in those repositories as separate work in their own Git roots.
- Keep examples and public extension documentation aligned with the behavior that is actually supported by those runtimes. Label proposals or unreleased behavior explicitly.

## Documentation and examples

- Update the canonical root `readme.md`, the relevant example README, and any public extension guide affected by a contract change. Do not manually maintain deprecated or generated HTML unless the task specifically requires it.
- Examples must be minimal, internally consistent, and safe to copy. Use placeholders for tokens, account names, email addresses, endpoints, and other environment-specific values.
- Preserve the documented extension JSON schema and naming conventions. Parse every changed JSON file and validate cross-references, action names, triggers, configuration keys, and example paths.
- Never activate or run an example or public extension against a real tenant merely to validate documentation. Live execution can write ITM Platform or third-party data, call external services, or send email and requires explicit authorization.

## Verification

- This repository has no general automated test suite. For documentation-only changes, run JSON parsing for every changed JSON file, check links and referenced paths, and inspect the rendered Markdown where formatting changed.
- For behavioral examples, also verify the corresponding runtime contract in the owning sibling repository and record what was checked. A successful JSON parse alone does not prove that an extension is executable.
