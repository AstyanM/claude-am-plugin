---
name: api-docs
description: "Use this skill whenever the user wants to generate, edit, or improve API documentation. Triggers include: any mention of 'API docs', 'API documentation', 'OpenAPI', 'Swagger', 'endpoint documentation', 'route docs', 'API reference', 'Postman collection', or requests to document REST/GraphQL/gRPC APIs. Also use when the user asks to generate OpenAPI specs from code, document request/response schemas, or create API usage examples. Do NOT use for general README documentation (use readme-editor), internal code documentation (docstrings), or database schema documentation (use dbml)."
tools: Read, Glob, Grep, Bash, Edit
---

# API Documentation Generator

## Overview

Generate accurate, comprehensive API documentation by scanning the actual codebase. Supports multiple output formats: OpenAPI 3.1 YAML, Markdown reference docs, and Postman collections.

**This skill reads code to produce documentation.** It never invents endpoints — every documented route must exist in the source code.

## Workflow

### Phase 1: API Discovery

Scan the codebase to identify the API framework and all endpoints.

**Framework Detection** — Check for these patterns:

| Framework | Detection Pattern |
|-----------|------------------|
| Express.js | `app.get/post/put/delete`, `router.get/post/put/delete` |
| Fastify | `fastify.get/post`, `fastify.register` |
| NestJS | `@Get()`, `@Post()`, `@Controller()` decorators |
| Next.js API Routes | `app/api/**/route.ts`, `pages/api/**/*.ts` |
| FastAPI | `@app.get`, `@router.post`, `APIRouter` |
| Django REST | `urlpatterns`, `@api_view`, `ViewSet` |
| Flask | `@app.route`, `Blueprint` |
| Spring Boot | `@GetMapping`, `@PostMapping`, `@RestController` |
| Go (net/http) | `http.HandleFunc`, `mux.HandleFunc` |
| Go (Gin) | `r.GET`, `r.POST`, `gin.Default()` |
| Hono | `app.get`, `app.post`, `Hono()` |

**Endpoint Extraction** — For each endpoint, extract:
- HTTP method and path (with path parameters)
- Request body schema (from types, validation, DTOs)
- Query parameters and headers
- Response schema (from return types, serializers)
- Authentication/authorization requirements
- Middleware applied (rate limiting, CORS, validation)
- Status codes returned (success and error)

### Phase 2: Schema Discovery

Extract data models used in requests and responses:

1. **Type definitions**: TypeScript interfaces/types, Python dataclasses/Pydantic models, Java DTOs, Go structs
2. **Validation schemas**: Zod, Joi, Yup, class-validator, Marshmallow, Cerberus
3. **Database models**: Prisma, SQLAlchemy, TypeORM, Mongoose (as reference for response shapes)
4. **Enums**: Status values, type discriminators, role names

For each field, capture:
- Name, type, required/optional
- Validation rules (min/max, pattern, enum values)
- Default values
- Description (from comments, docstrings, or decorators)

### Phase 3: Documentation Generation

Generate documentation in the requested format.

---

## Output Format: OpenAPI 3.1 (YAML)

```yaml
openapi: 3.1.0
info:
  title: Project Name API
  description: Brief description extracted from package.json or README
  version: 1.0.0  # from package.json

servers:
  - url: http://localhost:{port}
    description: Local development
    variables:
      port:
        default: "3000"  # from actual dev server config

paths:
  /api/resource:
    get:
      operationId: listResources
      summary: Short description
      description: Longer description if available from code comments
      tags:
        - Resources
      parameters:
        - name: page
          in: query
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        "200":
          description: Success
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ResourceList"
        "401":
          description: Unauthorized
        "500":
          description: Internal server error
      security:
        - bearerAuth: []

    post:
      operationId: createResource
      summary: Create a new resource
      tags:
        - Resources
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateResourceInput"
      responses:
        "201":
          description: Created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Resource"
        "400":
          description: Validation error
        "401":
          description: Unauthorized

components:
  schemas:
    Resource:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
          minLength: 1
          maxLength: 255
        createdAt:
          type: string
          format: date-time

    CreateResourceInput:
      type: object
      required:
        - name
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 255

    ResourceList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: "#/components/schemas/Resource"
        total:
          type: integer
        page:
          type: integer
        limit:
          type: integer

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### OpenAPI Rules

- Use `operationId` on every endpoint (camelCase, unique)
- Extract types from actual code — never guess field types
- Use `$ref` for all reusable schemas
- Include all status codes found in the code (not just 200)
- Tag endpoints by resource/module for grouping
- Add `format` hints: `date-time`, `email`, `uuid`, `uri` where appropriate
- Validation constraints (`minimum`, `maximum`, `minLength`, `maxLength`, `pattern`, `enum`) come from the actual validation logic

---

## Output Format: Markdown API Reference

```markdown
# API Reference

Base URL: `http://localhost:3000`

## Authentication

All endpoints require a Bearer token in the `Authorization` header unless marked as public.

---

## Resources

### List Resources

`GET /api/resources`

Returns a paginated list of resources.

**Query Parameters**

| Parameter | Type    | Required | Default | Description          |
|-----------|---------|----------|---------|----------------------|
| page      | integer | No       | 1       | Page number          |
| limit     | integer | No       | 20      | Items per page (max 100) |

**Response** `200 OK`

\`\`\`json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Example",
      "createdAt": "2025-01-15T10:30:00Z"
    }
  ],
  "total": 42,
  "page": 1,
  "limit": 20
}
\`\`\`

**Errors**

| Status | Description           |
|--------|-----------------------|
| 401    | Missing or invalid token |
| 500    | Internal server error |

---

### Create Resource

`POST /api/resources`

Creates a new resource.

**Request Body**

\`\`\`json
{
  "name": "New Resource"
}
\`\`\`

| Field | Type   | Required | Constraints      |
|-------|--------|----------|------------------|
| name  | string | Yes      | 1-255 characters |

**Response** `201 Created`

\`\`\`json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "New Resource",
  "createdAt": "2025-01-15T10:30:00Z"
}
\`\`\`
```

### Markdown Rules

- Group endpoints by resource/domain
- Always show a realistic example response (with plausible data, not "string" or "example")
- Include all query parameters, path parameters, headers, and body fields in tables
- Show error responses with their status codes
- Mark public endpoints explicitly

---

## Output Format: Postman Collection

Generate a Postman v2.1 collection JSON with:
- Folder structure matching API resource groups
- Pre-filled request bodies with example data
- Environment variables for `{{baseUrl}}` and `{{token}}`
- Test scripts for status code validation

---

## Multi-Framework Extraction Patterns

### Express.js / Fastify
```
Search patterns:
- router.(get|post|put|patch|delete)\s*\(
- app.(get|post|put|patch|delete)\s*\(
- Middleware: app.use, router.use
- Auth: passport, jwt, requireAuth, isAuthenticated
```

### FastAPI (Python)
```
Search patterns:
- @(app|router)\.(get|post|put|patch|delete)\s*\(
- Pydantic models: class.*\(BaseModel\)
- Depends() for dependency injection
- HTTPException for error responses
```

### Next.js App Router
```
Search patterns:
- export async function (GET|POST|PUT|PATCH|DELETE)
- Files: app/api/**/route.ts
- NextRequest, NextResponse
- Zod schemas in same directory
```

### Django REST Framework
```
Search patterns:
- class.*ViewSet, class.*APIView
- @api_view\[.*\]
- serializer_class, Serializer
- permission_classes
```

---

## Validation Checklist

Before finalizing the documentation:

- [ ] Every endpoint in the code is documented (search for all route definitions)
- [ ] Request/response schemas match actual types (no invented fields)
- [ ] All path parameters are documented with their types
- [ ] Authentication requirements are accurate per endpoint
- [ ] Status codes match what the code actually returns
- [ ] Example values are realistic (valid UUIDs, proper dates, plausible names)
- [ ] Validation constraints match the actual validation logic
- [ ] No placeholder text ("TODO", "description here", "example")
- [ ] OpenAPI spec is valid (if YAML format)
- [ ] Endpoints are correctly tagged/grouped

## File Output

- OpenAPI: Write as `openapi.yaml` (or `docs/openapi.yaml`)
- Markdown: Write as `API.md` (or `docs/API.md`)
- Postman: Write as `postman_collection.json` (or `docs/postman_collection.json`)

Ask the user which format they prefer if not specified. Default to OpenAPI YAML.