# Aspose.Slides Cloud MCP Server - Agent Guide

Guidance for AI agents connected to this MCP server.

## Core rules

1. **Credentials are never a tool parameter.** They're resolved once at server launch from the
   `ASPOSE_CLIENT_ID`/`ASPOSE_CLIENT_SECRET` environment variables. Never ask the user to pass a
   credential into a tool call, and never accept one if offered.
2. **This server bundles Aspose Storage Cloud's core-4 tools** (`storage_upload_file`,
   `storage_download_file`, `storage_list_files`, `storage_delete_file`) so you can complete a full
   workflow without a second server connection: upload a file, call a `slides_*` tool against it
   by filename, then download the result.
3. **One product per server.** This server only understands PPTX, PPT, ODP. Route a request for
   a different file format to that format's own Aspose Cloud MCP server
   (`aspose-<product>-cloud-python-mcp` under [github.com/aspose-cloud](https://github.com/aspose-cloud))
   rather than attempting it here.
4. **Every mutating tool supports a dry-run.** Pass `dry_run=true` to preview a destructive or
   costly operation before committing to it - use this when the user's intent is ambiguous.
5. **Tool errors are structured**, not raw exceptions (bad input / auth failure / server error /
   rate limited). Surface the real reason to the user rather than retrying blindly.

## Tools at a glance

- `slides_create_presentation` - Create a new presentation in Aspose Cloud Storage
- `slides_download_presentation` - Download an existing Cloud-stored presentation in a given format
- `slides_split_presentation` - Split a presentation into separate slide files
- `slides_list_images` - List a presentation's images
- `slides_get_presentation_properties` - Get a presentation's document properties
- `slides_list_slides` - List a presentation's slides
- `slides_get_text_items` - Get all text items in a presentation
- `slides_replace_text` - Replace text throughout a presentation
