# OpenFGA AuthZEN Interop

This project contains the OpenFGA models and data sets for the AuthZen Interop scenarios.


The [AuthZEN Interop](https://authzen-interop.net/) is an official [OpenID Foundation](https://openid.net/) project for interoperability testing between different Policy Decision Point (PDP) implementations that follow the [AuthZEN Authorization API specification](https://openid.net/wg/authzen/).

## Overview

The AuthZEN working group has created standardized test scenarios that allow PDP vendors to demonstrate their implementations work correctly and can interoperate with a common authorization API. This enables Policy Enforcement Points (PEPs) to integrate with any AuthZEN-compliant PDP without vendor-specific code.

## Pre-requisites

- Install the [OpenFGA CLI](https://github.com/openfga/cli?tab=readme-ov-file#installation)
- Clone the AuthZen interop repository.

```bash
git clone https://github.com/openid/authzen.git
```

- Run OpenFGA with the AuthZen experimental flag turned on:

```bash
openfga run --experimentals=authzen
```

### 1. Todo Application 

The [Todo App scenario](https://authzen-interop.net/docs/scenarios/todo-1.1/) tests a task management application with role-based access control. The test harness is defined [here](https://github.com/openid/authzen/tree/main/interop/authzen-todo-backend).

To test the scenario:

- Import the model and tuples in OpenFGA:
```
fga store import --file todo/authzen-todo.fga.yaml
```

After importing it, it will print the store id and model id and additional details. Copy the Store ID.

```json
{                                                                                               
  "store": {
    "created_at":"2026-01-17T01:25:28.886128Z",
    "id":"01KF4RXANNH2CVJ2W0TMDV3NEY",
    "name":"AuthZen Interop Todo",
    "updated_at":"2026-01-17T01:25:28.886128Z"
  },
  "model": {
    "authorization_model_id":"01KF4RXANZ194VH2HX6614T6FM"
  }
}
```

- In the [AuthZen repository](https://github.com/openid/authzen/), run:

```bash
cd interop/authzen-todo-backend/test
yarn install
yarn build
yarn test http://localhost:8080/stores/<store_id>
```

You can also install [Rest Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) in Visual Studio Code, open `authzen-todo.http`, set the `@fga_store_id` field, and try running each API call.

### 2. API Gateway

The [API Gateway scenario](https://authzen-interop.net/docs/category/api-gateway-10-draft-02) tests HTTP method-based access control for API routes. The test harness is defined [here](https://github.com/openid/authzen/tree/main/interop/authzen-gateway).

To test the scenario:

- Import the model and tuples in OpenFGA:
```
fga store import --file gateway/authzen-gateway.fga.yaml
```

After importing it, it will print the store id and model id and additional details. Copy the Store ID.

```json
{                                                                                               
  "store": {
    "created_at":"2026-01-17T01:25:28.886128Z",
    "id":"01KF4RXANNH2CVJ2W0TMDV3NEY",
    "name":"AuthZen Gateway Interop",
    "updated_at":"2026-01-17T01:25:28.886128Z"
  },
  "model": {
    "authorization_model_id":"01KF4RXANZ194VH2HX6614T6FM"
  }
}
```

- In the [AuthZen repository](https://github.com/openid/authzen/), run:

```bash
cd interop/authzen-api-gateways/test-harness
yarn install
yarn build
yarn test http://localhost:8080/stores/<store_id>
```

You can also install [Rest Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) in Visual Studio Code, open `authzen-gateway.http`, set the `@fga_store_id` field, and try running each API call.

## 3. Search

The [Search scenario](https://authzen-interop.net/docs/category/results-4) allows listing resources, actions and subjects.
)
### OpenFGA Compatibility 

>Note: OpenFGA will not pass the search interop tests. The endpoints are implemented properly, but the way OpenFGA works make its impossible to make the 'search/action' endpint work.

The [OpenFGA model](search/authzen-search.fga) has this `record` type:

```python
type record
  relations
    # The department this record belongs to
    define department: [department]
    # The owner of this record
    define owner: [user]
    # Viewers include managers group members (for "any manager can view any record")
    define viewer: [user, group#member]

    # Helper relations for computed permissions
    define department_member: member from department
    define department_manager: manager from department

    # view: owner OR department member OR viewer (managers)
    define view: owner or department_member or viewer

    # edit: owner OR department manager
    define edit: owner or department_manager

    # delete: owner only
    define delete: owner

```
OpenFGA does not differentiate 'actions' from relations. When 'search/actions' is called, all relations are returned. The AuthZen interop scenario requires to return `view`, `edit` and `delete` only. OpenFGA returns `delete` `edit`, `view` , `department_management','department_member` `viewer`, `owner`.

### Running the tests

To test the scenario:

- Import the model and tuples in OpenFGA:
```bash
fga store import --file search/authzen-search.fga.yaml
```

After importing it, it will print the store id and model id and additional details. Copy the Store ID.

```json
{                                                                                               
  "store": {
    "created_at":"2026-01-17T01:25:28.886128Z",
    "id":"01KF4RXANNH2CVJ2W0TMDV3NEY",
    "name":"AuthZen Search Interop",
    "updated_at":"2026-01-17T01:25:28.886128Z"
  },
  "model": {
    "authorization_model_id":"01KF4RXANZ194VH2HX6614T6FM"
  }
}
```

- In the [AuthZen repository](https://github.com/openid/authzen/), run:

```bash
cd interop/authzen-search-demo/test-harness
yarn install
yarn build
yarn test http://localhost:8080/stores/<store_id>
```

As mentioned before, the tests will fail for the action search use cases.

You can also install [Rest Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) in Visual Studio Code, open `authzen-gateway.http`, set the `@fga_store_id` field, and try running each API call.