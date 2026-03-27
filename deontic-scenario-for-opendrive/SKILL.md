---
name: deontic-scenario-for-opendrive
description: >
  Generate an OpenSCENARIO for an existing OpenDRIVE (.xodr) road network file using the Deontic
  MCP server. Use this skill whenever the user provides or references an existing OpenDRIVE file
  and wants to generate a driving scenario for it — for example: "generate a scenario for this
  .xodr file", "use my road network to create a scenario", "I have an OpenDRIVE file, can you
  make a scenario?", "create a scenario using my existing road", or any request where the user
  already has a .xodr file and wants scenario files generated on top of it. Also triggers when the
  user uploads or mentions a .xodr file alongside a driving situation description. Do NOT use this
  skill when no OpenDRIVE file is provided — use deontic-scenario-generator instead (which creates
  both the road network and scenario from scratch).
compatibility: "Requires Deontic MCP server connection"
---

# Deontic Scenario Generation for Existing OpenDRIVE

Generate an OpenSCENARIO file for an existing OpenDRIVE (.xodr) road network using the Deontic MCP server.

## Overview

This skill is for when the user already has an OpenDRIVE road network and wants to generate driving scenario actors and events on top of it. It produces:
- A scene description
- A saved OpenSCENARIO (.xosc) file in the output directory

---

## Workflow

### Step 1 — Gather Inputs

You need three things:

- **OpenDRIVE file path** (required): The absolute path to the user's `.xodr` file on disk. The file may have been uploaded (check `/sessions/.../mnt/uploads/`) or be in the user's selected folder. If the path is unclear or the file hasn't been located yet, ask the user to confirm where it is.
- **Scenario prompt** (required): A natural language description of the driving situation to simulate on this road network (e.g., "A vehicle cuts in front of the ego car" or "A pedestrian crosses at a junction while ego approaches at 30 km/h").
- **Output directory** (required): Where to save the generated `.xosc` file. Use the user's selected workspace folder as the default (e.g., `/sessions/.../mnt/scenarios`). Create the directory if it does not exist.

If the file path or scenario prompt is missing, ask for it before proceeding.

---

### Step 2 — Get a Pre-signed Upload URL

Call the MCP tool `get_open_drive_upload_url` (no parameters required).

This returns:
- `upload_url`: a pre-signed URL to upload the file to
- `blob_name`: the identifier to reference this file in subsequent calls

---

### Step 3 — Upload the OpenDRIVE File

Use `curl` via Bash to upload the `.xodr` file directly to the pre-signed URL:

```bash
curl -X PUT \
  -H "x-ms-blob-type: BlockBlob" \
  --upload-file "<path-to-file>" \
  "<upload_url>"
```

Replace `<path-to-file>` with the absolute path from Step 1 and `<upload_url>` with the URL from Step 2. A successful upload returns an empty 201 response — no output is expected.

---

### Step 4 — Generate the Scenario

Call the MCP tool `generate_scenario_for_opendrive` with:
- `scenario_prompt`: the scenario description from Step 1
- `open_drive_file_path`: the `blob_name` returned in Step 2 (not the local file path)

Wait for the response, which includes:
- `scenario_id`: identifier for the generated scenario
- `scene_description`: textual description of the generated scene
- `open_scenario_download_url`: URL to download the `.xosc` file

---

### Step 5 — Save the OpenSCENARIO File

1. Create the output directory if it does not exist:
   ```bash
   mkdir -p "<output_directory>"
   ```
2. Extract the filename from the `open_scenario_download_url` (use the last path segment before any query string).
3. Download the `.xosc` file to the output directory, preserving that filename:
   ```bash
   curl -L -o "<output_directory>/<filename>.xosc" "<open_scenario_download_url>"
   ```

---

### Step 6 — Display Results

Present the response clearly:

#### 🗺️ Scene Description
Show the `scene_description` field from the response.

#### 📁 Saved File
Provide a `computer://` link to the saved `.xosc` file so the user can open it directly.

---

### Step 7 — Suggest Follow-up Actions

After presenting the results, suggest next steps:

> **What would you like to do next?**
> - 🎬 **Create a video** to visualize this scenario
> - 💬 **Refine the scenario** by describing changes (e.g., "add fog", "make the ego vehicle faster")
> - 🔄 **Generate another scenario** on the same road network with a different prompt

---

## Error Handling

- If the `.xodr` file path is wrong or the file doesn't exist, let the user know and ask them to confirm the correct path.
- If the `curl` upload returns a non-2xx status code, report the error and do not proceed to generation.
- If `generate_scenario_for_opendrive` fails, inform the user and suggest rephrasing the scenario prompt or checking that the OpenDRIVE file is valid.
- If the download URL is missing from the response, still display the scene description and note the issue.

---

## Example Interaction

**User**: I have this road network file at `/Users/yves/roads/highway.xodr`. Can you generate a scenario where a truck merges into the ego lane?

**Assistant**:
1. Confirms the file path and scenario prompt
2. Calls `get_open_drive_upload_url` → receives `upload_url` and `blob_name`
3. Runs `curl` to upload `/Users/yves/roads/highway.xodr` to the pre-signed URL
4. Calls `generate_scenario_for_opendrive` with:
   - `open_drive_file_path`: the `blob_name` from step 2
   - `scenario_prompt`: "A truck merges into the ego lane from the right"
5. Creates the output directory if needed, downloads the `.xosc` file to it
6. Displays the scene description and a link to the saved file
7. Suggests follow-up actions (video, edits, new scenario)
