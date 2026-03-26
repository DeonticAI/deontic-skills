---
name: deontic-odd-scenario-generator
description: >
  Generate MULTIPLE autonomous driving test scenarios using the Deontic MCP server. Only use this
  skill when BOTH of these conditions are met: (1) the user wants to generate more than one scenario
  (e.g. "generate 5 scenarios", "create a batch of test scenarios", "multiple scenario variants"),
  AND (2) the user provides or references an ODD (Operational Design Domain) description — either
  as a file or inline text describing the operational context. Do NOT use this skill for single
  scenario generation or when no ODD is provided. Triggers include phrases like "generate N
  scenarios for this ODD", "create multiple test scenarios based on my ODD", or "batch scenario
  generation with ODD coverage".
compatibility: "Requires Deontic MCP server (https://osvwppoj5xu2m5lw-mcp.azurewebsites.net/mcp)"
---

# Deontic ODD Scenario Generator

A skill for generating structured autonomous driving test scenarios using the Deontic MCP server.

## Overview

This skill orchestrates an interactive workflow to:
1. Suggest scenario attribute combinations for a given scenario prompt + ODD
2. Get user approval/modification of attributes
3. Generate OpenDRIVE + OpenSCENARIO files for each approved combination
4. Save all artifacts in a structured output directory

---

## Workflow

### Step 1 — Gather Inputs

Before starting, ensure you have:
- **Scenario prompt**: The specific driving scenario prompt to be tested (e.g., "The vehicle shall come to a complete stop at a red traffic light")
- **Number of scenarios**: How many scenario variants to generate
- **ODD description**: Either a file path or inline text describing the Operational Design Domain
- **Output directory**: Where to save the generated scenario files

If any of these are missing, ask the user to provide them before proceeding.

---

### Step 2 — Read the ODD

If the ODD is provided as a file, read it fully. Note any taxonomy files it references and read those too. This context is needed for `suggest-scenario-attributes`.

---

### Step 3 — Suggest Scenario Attributes

Call `suggest-scenario-attributes` from the Deontic MCP server:
- Pass the prompt as `scenario_prompt`
- Pass the ODD content as `odd`
- Request the desired number of scenario variants

**Present the suggested attribute combinations to the user in a clear table or list**, showing each combination with its labels. Explain what each attribute controls.

Ask the user:
> "Here are the suggested attribute combinations. Would you like to approve these, or would you like to modify, add, or remove any attributes before I generate the scenarios?"

Wait for explicit approval. Allow the user to:
- Approve all combinations as-is
- Remove specific combinations
- Modify attribute values
- Add new combinations manually

---

### Step 4 — Generate Scenarios

Generate the scenarios one by one (not in parallel). For each **approved** attribute combination:

**4.1 — Call `generate-scenario`**
```
scenario_prompt: <the scenario prompt>
scenario_constraints: <attributes joined by "; ">
odd: <ODD content>
```

**4.2 — Create output subdirectory**

Name the subdirectory using the scenario number and attribute labels:
```
{output_directory}/scenario_{N}_{attribute_label_slug}/
```
Where `N` is a zero-padded index (01, 02, ...) and the slug replaces spaces/special chars with underscores.

**4.3 — Save scenario.md**

Write a `scenario.md` file to the subdirectory containing:
```markdown
# Scenario {N}: {attribute labels}

## Road Network Description
{road_network_description from API response}

## Scene Description
{scene_description from API response}
```

**4.4 — Download OpenDRIVE file**

Download the file from the `open_drive` URL in the response. Keep the original filename from the URL. Save to the scenario subdirectory.

**4.5 — Download OpenSCENARIO file**

Download the file from the `open_scenario` URL in the response. Keep the original filename from the URL. Save to the scenario subdirectory.

---

### Step 5 — Summarize and Suggest Follow-ups

After all scenarios are generated, provide a summary:
- List all generated scenarios with their subdirectory paths
- Show which attribute combination each corresponds to

Then suggest follow-up tasks:
> **Next steps you might want to try:**
> - 🎬 **Create a video** for one or more scenarios to visualize the simulation
> - 💬 **Give feedback** on the generated scenarios to refine them
> - 🔄 **Regenerate** a specific scenario with adjusted constraints

---

## Error Handling

- If `suggest-scenario-attributes` returns no results, inform the user and ask them to provide attributes manually.
- If a scenario generation fails, log the error, skip that combination, and continue with the rest. Report failures at the end.
- If a file download fails, note it in the scenario.md and continue.

---

## Directory Structure Example

```
output_directory/
├── scenario_01_urban_intersection_rain/
│   ├── scenario.md
│   ├── 3f7a2c1e-84b0-4d9f-a123-56789abcdef0.xodr
│   └── 7b1d4e8f-23c0-4a6e-b456-89012cdef123.xosc
├── scenario_02_highway_merge_fog/
│   ├── scenario.md
│   ├── a2b3c4d5-e6f7-4890-b123-456789abcdef.xodr
│   └── f1e2d3c4-b5a6-4789-c012-345678901234.xosc
└── scenario_03_parking_lot_pedestrian/
    ├── scenario.md
    ├── 9c8b7a6f-5e4d-4321-d890-123456789abc.xodr
    └── 0d1e2f3a-4b5c-4678-e901-234567890bcd.xosc
```
