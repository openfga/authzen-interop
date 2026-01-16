# OpenFGA AuthZEN Interop

This project contains the OpenFGA models and data sets for the AuthZen Interop scenarios.


The [AuthZEN Interop](https://authzen-interop.net/) is an official [OpenID Foundation](https://openid.net/) project for interoperability testing between different Policy Decision Point (PDP) implementations that follow the [AuthZEN Authorization API specification](https://openid.net/wg/authzen/).

## Overview

The AuthZEN working group has created standardized test scenarios that allow PDP vendors to demonstrate their implementations work correctly and can interoperate with a common authorization API. This enables Policy Enforcement Points (PEPs) to integrate with any AuthZEN-compliant PDP without vendor-specific code.

## Test Scenarios

### 1. Todo Application (1.1)

The [Todo App scenario](https://authzen-interop.net/docs/scenarios/todo-1.1/) tests a task management application with role-based access control:

**Roles:**
- **Admin** - Full access to all operations
- **Editor** - Can create and manage todo items
- **Viewer** - Read-only access
- **Evil Genius** - Can manage todos and update any todo item

**Key Authorization Rules:**
- Admins can perform any operation
- Editors can create, read, update, and delete their own todos
- Viewers can only read todos
- Evil geniuses can update any todo (but can only delete their own)

**Test Users (Rick and Morty theme):**
| User | Role | Capabilities |
|------|------|--------------|
| Rick Sanchez | Admin | Everything |
| Morty Smith | Editor | Create, read, manage own todos |
| Summer Smith | Viewer | Read only |
| Beth Smith | Evil Genius | Update any todo, manage own |

### 2. API Gateway (1.0-draft-02)

The [API Gateway scenario](https://authzen-interop.net/docs/category/api-gateway-10-draft-02) tests HTTP method-based access control for API routes:

**Roles:**
- **Admin** - Full access (GET, POST, PUT, DELETE)
- **Developer** - Read/write access (GET, POST, PUT)
- **Viewer** - Read-only access (GET)

**Key Authorization Rules:**
- Some routes are public (GET available to all authenticated users)
- Write operations (POST, PUT) require specific roles
- Delete operations require admin role
- Role assignments are identity-based

## OpenFGA Implementation

OpenFGA's AuthZEN implementation supports both interop scenarios:

### Todo App Model

```fga
model
  schema 1.1

type user
  relations
    define can_read_user: [user:*]

type todo
  relations
    define admin: [user]
    define editor: [user]
    define evil_genius: [user]
    define viewer: [user]

    define can_manage_todo_items: editor or admin or evil_genius
    define can_create_todo: can_manage_todo_items
    define can_read_todos: viewer or can_manage_todo_items

    define parent: [todo]
    define owner: [user]

    define can_delete_todo: (can_manage_todo_items from parent and owner) or admin from parent
    define can_update_todo: (can_manage_todo_items from parent and owner) or evil_genius from parent
```

### API Gateway Model

```fga
model
  schema 1.1

type identity

type role
  relations
    define assignee: [identity]

type route
  relations
    define GET: [identity:*, role#assignee]
    define POST: [role#assignee]
    define PUT: [role#assignee]
    define DELETE: [role#assignee]
```

## Running the Interop Tests

### Prerequisites

1. Clone the AuthZEN repository:
   ```bash
   git clone https://github.com/openid/authzen
   cd authzen
   ```

2. Set the shared API key (available in OpenFGA vault as "OpenFGA AuthZeN shared key"):
   ```bash
   export AUTHZEN_PDP_API_KEY="<shared-key>"
   ```

### Todo Backend Tests

```bash
cd interop/authzen-todo-backend
yarn install
yarn test https://authzen-interop.openfga.dev/stores/<store_id>
```

### API Gateway Tests

```bash
cd interop/authzen-api-gateway
yarn install
yarn test https://authzen-interop.openfga.dev/stores/<store_id>
```

### Running Against Local OpenFGA

To run tests against a local OpenFGA instance:

1. Start OpenFGA with AuthZEN enabled:
   ```bash
   openfga run --experimentals enable_authzen
   ```

2. Create a store and load the appropriate model

3. Run tests pointing to localhost:
   ```bash
   yarn test http://localhost:8080/stores/<store_id>
   ```

**Note:** If running the Todo application locally, use a different port for OpenFGA to avoid conflicts:
```bash
openfga run --http-addr 0.0.0.0:4000 --experimentals enable_authzen
```

## Live Demo

Try the AuthZEN Todo application live at [todo.authzen-interop.net](https://todo.authzen-interop.net/).

**Test Credentials:**
See the [official identity list](https://github.com/openid/authzen/blob/main/interop/authzen-todo-application/README.md#identities) for user credentials.

## Resources

- [AuthZEN Interop Portal](https://authzen-interop.net/)
- [AuthZEN Working Group](https://openid.net/wg/authzen/)
- [Authorization API 1.0 Specification](https://openid.github.io/authzen/)
- [OpenID AuthZEN GitHub](https://github.com/openid/authzen)
- [OpenFGA AuthZEN Implementation](./README.md)
- [OIDF Slack](https://openid.net/slack) - Join the #authzen channel for community discussions

## OpenFGA Test Files

OpenFGA includes local test files for validating the interop scenarios:

| File | Description |
|------|-------------|
| [`authzen-todo.fga.yaml`](./authzen-todo.fga.yaml) | Todo App model with inline tests |
| [`authzen-gateway.fga.yaml`](./authzen-gateway.fga.yaml) | API Gateway model with inline tests |
| [`authzen-todo.http`](./authzen-todo.http) | HTTP requests for Todo scenario |
| [`authzen-gateway.http`](./authzen-gateway.http) | HTTP requests for Gateway scenario |

Run local model tests using the [FGA CLI](https://github.com/openfga/cli):

```bash
fga model test --test authzen-todo.fga.yaml
fga model test --test authzen-gateway.fga.yaml
```
