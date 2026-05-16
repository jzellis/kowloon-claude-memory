---
name: joplin-integration
description: "Joplin Web Clipper API on localhost — token, folder ID, and key note IDs for Kowloon design docs"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 441e186f-92df-4648-b5ed-e6a45209573c
---

Joplin Web Clipper exposes a local HTTP API for reading project design notes.

- **API**: `http://localhost:41184`
- **Token (this machine)**: `888b63fdce9ff115941f596b0867141b6a43b35a280ae2995cd83ba9a52e4b05d7598e012afde02f0e7b6595c7804ff5f0ed5e9f640c44abc640f2011ddff53c`
- **Also stored** in `server/.env` as `JOPLIN_TOKEN`
- **Kowloon folder ID**: `112f3b6f046a4664ad4733477953ceb4`

### Known note IDs
- Consolidated Client API spec: `1cfd6eaee9b64494a577617d4f9e5847`
- Federation: Multi-Server Notification Cases: `97449ef9bedb4d2787f1117af9703bc4` (added 2026-05-14)
- Activity Lifecycle in Kowloon: `a6cd26398b91444692a1330f65b8e223` (added 2026-05-14)

### Usage
```bash
# Read a note
curl "http://localhost:41184/notes/<id>?token=$JOPLIN_TOKEN&fields=body"

# List notes in a folder
curl "http://localhost:41184/folders/<folderId>/notes?token=$JOPLIN_TOKEN&fields=id,title"
```
