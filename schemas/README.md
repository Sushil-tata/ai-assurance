# Schemas

Machine-readable JSON Schema for agent and tool input/output, used for Copilot Studio action validation and automated testing (`testing/AGENT_TEST_CASES.md`).

## Coverage in this repository
Representative schemas are provided for the Investigation Agent (input/output) and the `get_monitoring_metric` tool (request/response) — these are the most structurally complex examples and are sufficient to establish the pattern for a builder to replicate across the remaining six agents and eleven tools.

**Full coverage of all 7 agents × 2 (input/output) and 12 tools × 2 (request/response) is an enterprise-build task**, tracked in `MVP_VS_ENTERPRISE.md`, generated mechanically from the field tables already fully specified in each `agents/*.md` "Input schema" / "Output schema" section and `tools/API_CONTRACTS.md` — the authoring work is documentation, not design, once those markdown tables exist.

## Folder structure
```
schemas/
├── agent_inputs/       one *.schema.json per agent, named <agent_id>.schema.json
├── agent_outputs/      one *.schema.json per agent, named <agent_id>.schema.json
├── tool_requests/       one *.schema.json per tool, named <tool_id>.schema.json
└── tool_responses/      one *.schema.json per tool, named <tool_id>.schema.json
```

## Validation usage
Every Copilot Studio action should be configured with request/response validation against the matching schema file where the platform supports it; where it doesn't (native low-code actions), the schema still serves as the contract for the automated test suite (`testing/AGENT_TEST_CASES.md`) to validate payloads captured in `fact_agent_execution_log` / `fact_tool_execution_log`.
