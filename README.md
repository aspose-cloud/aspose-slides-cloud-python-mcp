# Aspose.Slides Cloud MCP Server

Presentation processing — convert, create, split, replace text, and read slides/images/properties.

An [MCP](https://modelcontextprotocol.io) server exposing Aspose.Slides Cloud's REST API as typed,
agent-callable tools. Also bundles Aspose Storage Cloud's core file operations
(`storage_upload_file`/`storage_download_file`/`storage_list_files`/`storage_delete_file`), so a
client connected to only this server can complete a full upload -> process -> download workflow with
no second server connection.

Handles: PPTX, PPT, ODP.

---

## Requirements

- Python 3.11 or later
- An [Aspose Cloud](https://dashboard.aspose.cloud/) account (free evaluation tier available) - you'll
  need a **Client ID** and **Client Secret** from your dashboard's Applications page
- An MCP-compatible AI client (Claude Desktop, Claude Code, VS Code, Cursor, Cline, Windsurf, etc.)

---

## Setup

### 1. Create a virtual environment and install

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS / Linux:
source .venv/bin/activate

pip install git+https://github.com/aspose-cloud/aspose-storage-cloud-python-mcp.git@v1.0.5 git+https://github.com/aspose-cloud/aspose-slides-cloud-python-mcp.git@v1.0.3
```

This installs the `aspose-slides-mcp` command into your virtual environment. The first package is the shared file-tool library (upload, download, list, delete) that every Aspose Cloud MCP server depends on; it is not on PyPI, so it is installed from GitHub in the same command.

### 2. Configure your AI client

Three environment variables are required. The first two come from your Aspose Cloud dashboard's Applications page; the third is a folder you create on your own computer (see [Your files folder](#your-files-folder-aspose_files_path) below):

| Variable | Value |
|---|---|
| `ASPOSE_CLIENT_ID` | Your application's Client ID |
| `ASPOSE_CLIENT_SECRET` | Your application's Client Secret |
| `ASPOSE_FILES_PATH` | Full path of the folder on your computer where your input files are placed and results are saved |

Credentials are never passed as a tool parameter - the server resolves them once at launch from
these environment variables, exchanges them for a short-lived OAuth2 token, and caches/refreshes it
transparently.

#### Your files folder (`ASPOSE_FILES_PATH`)

Create a folder on your computer (for example `C:\Users\you\Documents\aspose-files` on Windows, or
`/home/you/aspose-files` on macOS/Linux) and set `ASPOSE_FILES_PATH` to its full path. It is the one
place this server reads your files from and writes results to:

- **Input:** put the document you want processed in that folder, then ask your assistant to upload it with
  `storage_upload_file`, giving only the bare filename (for example `report.docx`). The upload sends it to
  your Aspose Cloud Storage, where the processing tools read it. Subfolders and full paths are rejected.
- **Output:** `storage_download_file` saves a result from Cloud Storage back into the same folder, and its
  reply contains the full local path of the saved file.
- The file and folder arguments of the other tools name locations in your Aspose Cloud Storage, not your
  computer.

#### Claude Desktop

Config file location:

| Platform | Path |
|---|---|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |

```json
{
  "mcpServers": {
    "slides": {
      "command": "C:\\path\\to\\.venv\\Scripts\\aspose-slides-mcp.exe",
      "env": {
        "ASPOSE_CLIENT_ID": "your-client-id",
        "ASPOSE_CLIENT_SECRET": "your-client-secret",
        "ASPOSE_FILES_PATH": "C:\\Users\\you\\Documents\\aspose-files"
      }
    }
  }
}
```

On macOS/Linux, use `/path/to/.venv/bin/aspose-slides-mcp` instead. Fully quit and restart Claude Desktop after
editing.

#### VS Code (`.vscode/mcp.json`), Cursor (`~/.cursor/mcp.json`), Cline, Windsurf

Same shape, under a `"servers"` key instead of `"mcpServers"` for VS Code:

```json
{
  "servers": {
    "slides": {
      "type": "stdio",
      "command": "/path/to/.venv/bin/aspose-slides-mcp",
      "env": {
        "ASPOSE_CLIENT_ID": "your-client-id",
        "ASPOSE_CLIENT_SECRET": "your-client-secret",
        "ASPOSE_FILES_PATH": "/home/you/aspose-files"
      }
    }
  }
}
```

---

## Available tools

| Tool | What it does | Read-only | Dry-run |
|---|---|---|---|
| `slides_create_presentation` | Creates a new presentation in Aspose Slides, optionally using an uploaded source file as a template and optionally applying a password to the new file. | no | yes |
| `slides_download_presentation` | The tool retrieves a presentation from Aspose Slides Cloud, converts it to a specified export format, and returns the resulting file as binary data. | yes | - |
| `slides_split_presentation` | This tool splits a PowerPoint presentation into separate image files, creating one image for each selected slide and uploading the images to cloud storage. | no | yes |
| `slides_list_images` | Retrieves metadata for every image that is embedded in a specified PowerPoint presentation stored in Aspose Cloud. | yes | - |
| `slides_get_presentation_properties` | Retrieves the metadata properties of a PowerPoint presentation file stored in Aspose Slides Cloud. | yes | - |
| `slides_list_slides` | Retrieves information about all slides in a specified PowerPoint presentation stored in Aspose Cloud. | yes | - |
| `slides_get_text_items` | Extracts all text items from a specified PowerPoint presentation file. | yes | - |
| `slides_replace_text` | Replaces every occurrence of a specified text string in a Slides presentation with a new string. | no | yes |

Every mutating tool marked "yes" under **Dry-run** accepts a `dry_run=true` parameter to preview
the change without applying it.

Tool errors use a fixed taxonomy (bad input / auth failure / server error / rate limited), returned
as structured MCP tool errors - never a silent failure or a raw exception message.

---

## Part of the Aspose Cloud MCP family

One MCP server per Aspose Cloud product, published under [github.com/aspose-cloud](https://github.com/aspose-cloud).
This server depends on [`aspose-storage-core-mcp`](https://github.com/aspose-cloud/aspose-storage-cloud-python-mcp)
for its bundled storage tools — that repo is a shared library, not a standalone server (there's no
real Aspose Cloud API route for storage on its own; every real storage call goes through some
product's own gateway, this one included).

---

## License

MIT (see [`LICENSE`](LICENSE)) — covers only this repository's own MCP wrapper/integration code.
It does **not** cover, and grants no rights to, the Aspose Cloud product or API themselves, which
remain governed entirely by [Aspose's own product and usage terms](https://purchase.aspose.cloud/policies).
A valid Aspose Cloud account and subscription/credentials are required to actually call the API,
regardless of this code's license.
