# Documentation Page Template

> **How to use this template:** Copy the structure below, fill in the placeholders, and delete any sections that are not applicable to the feature or service you are documenting.

---

# <Feature or Service Name>

**Owner:** <team name>  
**Last updated:** YYYY-MM-DD  
**Status:** `Stable` / `Beta` / `Deprecated`

---

## Overview

> 1–2 sentences describing what this feature/service does and who it is for.

---

## Quick start

> Minimal steps to get a user up and running. Use a numbered list.

1. Step one
2. Step two
3. Step three

---

## Prerequisites

- <prerequisite 1 — e.g. "Node.js >= 18">
- <prerequisite 2 — e.g. "Access to the `prod` AWS account">

---

## Configuration

| Variable / Option | Type | Default | Required | Description |
|-------------------|------|---------|----------|-------------|
| `<VARIABLE_NAME>` | string | — | Yes | <what it controls> |
| `<VARIABLE_NAME>` | boolean | `false` | No | <what it controls> |

---

## Usage

### Common use case 1 — <name>

> Describe the use case briefly.

```<language>
# Example code or command
```

### Common use case 2 — <name>

```<language>
# Example code or command
```

---

## API reference

> Include this section for services that expose an API.

### `<METHOD> /path/to/endpoint`

**Description:** <what this endpoint does>

**Request**

| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|
| `<param>` | body / query / path | string | Yes | <description> |

**Example request**

```http
POST /api/v1/example
Content-Type: application/json

{
  "field": "value"
}
```

**Response**

```json
{
  "id": "123",
  "status": "ok"
}
```

**Error codes**

| Code | Meaning |
|------|---------|
| 400 | Validation error — check request body |
| 401 | Unauthenticated |
| 403 | Insufficient permissions |
| 404 | Resource not found |
| 500 | Internal server error |

---

## Architecture

> Include a diagram or brief description of how this feature fits into the larger system.

<diagram or description>

---

## Runbook

> Link to the operational runbook for this service (see `skills/incident-response/runbook-template.md`).

Link: <link>

---

## Known limitations

- <limitation 1>
- <limitation 2>

---

## FAQ

**Q: <common question>**  
A: <answer>

**Q: <common question>**  
A: <answer>

---

## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | YYYY-MM-DD | Initial release |

---

## Related

- <link to related feature doc>
- <link to ADR>
- <link to RFC>
