---
name: deontic-video-creator
description: >
  Create a video of an existing Deontic driving scenario using the Deontic MCP server.
  Use this skill whenever a user asks to render, visualize, record, or create a video
  of a scenario that was previously generated. Triggers include: "create a video",
  "render the scenario", "show me a video", "visualize the simulation", "record this
  scenario", "make a video of it", or any phrasing that asks to turn an existing
  scenario into a video or animation. Always use this skill when a scenario_id is
  available in context and the user expresses any desire to see or export it as video.
compatibility: "Requires Deontic MCP server connection"
---

# Deontic Video Creator

Render an existing Deontic driving scenario as an MP4 video using the Deontic MCP server.

## Overview

This skill takes a `scenario_id` from a previously generated scenario and produces a
downloadable MP4 video of the simulation.

---

## Workflow

### Step 1 — Find the scenario_id

Look in the current conversation for a `scenario_id` returned by a prior step
(e.g. from `deontic-scenario-generator` or `deontic-scenario-editor`).

- If a `scenario_id` is present in context: proceed directly to Step 2.
- If no `scenario_id` is found: inform the user that a scenario must be generated
  first, and suggest using the scenario generator skill.

---

### Step 2 — Create the video

Call `Deontic:create_video` with:
- `scenario_id`: the scenario ID from context

Tell the user that rendering is in progress — it may take a moment.

---

### Step 3 — Present the result

The tool returns an MP4 video. Present it to the user with a download link or inline
preview so they can view the simulation.

If the tool returns a URL, display it clearly as a clickable download link.
If the tool returns raw video data or a file reference, surface it for download.

After presenting the video, suggest follow-up actions:

> **What would you like to do next?**
> - 💬 **Give feedback** to adjust the scenario and re-render
> - 🔄 **Generate a new scenario** with a different prompt

---

## Error Handling

- If `create_video` fails, report the error message to the user and suggest they try
  again or adjust the scenario first using the editor skill.
- If no scenario exists in context, do not attempt to call the tool — guide the user
  to generate one first.
