# eStore API — Test Suite

## Newman Tests

A professional REST API test suite for [FakeStore API](https://fakestoreapi.com) — an e-commerce backend simulation covering Products, Users, Authentication, and Carts. Built with **Postman** and **Newman**, with automated CI execution and live HTML reporting via GitHub Pages.

> 📄 **[Published API Documentation](https://documenter.getpostman.com/view/5814369/2sBXwqsrFL)** — Interactive request docs with examples  
> 📊 **[Live Test Report](https://hussainimd.github.io/eStore-API-Testing/)** — Latest Newman htmlextra report  
> ⚙️ **[CI Pipeline](https://github.com/HussainiMD/eStore-API-Testing/actions)** — GitHub Actions workflow

---

## What This Collection Tests

End-to-end API test coverage for [FakeStore API](https://fakestoreapi.com) — a REST API simulating a real-world e-commerce backend. The suite validates functional correctness, response contract integrity, and negative/edge case handling across all four resource domains.

| Domain | Requests | Happy Path | Negative Cases |
|--------|----------|------------|----------------|
| **Auth** | 3 | Login, password update | Invalid credentials |
| **Products** | 7 | CRUD + sorted/limited fetch | Non-existent ID, delete missing product |
| **Users** | 6 | CRUD | Non-existent ID, delete missing user |
| **Carts** | 10 | CRUD, quantity update, zero quantity | Invalid user, invalid product, missing cart |
| **Total** | **26** | | |

---

## Test Design Principles

| Principle | Detail |
|-----------|--------|
| **Schema-first validation** | Every success response is validated against a JSON Schema using AJV (v8, strict mode, `allErrors`). Type mismatches, missing fields, and unexpected properties all fail the test. |
| **Pre-request isolation** | Test data is set in Pre-request Scripts and cleaned up in Tests via `pm.variables.unset()`. No test pollutes the variable scope for the next. |
| **Negative case coverage** | Every resource (Products, Users, Carts) has corresponding negative tests: non-existent IDs, invalid payloads, and invalid credentials. |
| **Response time assertions** | All GET endpoints assert response time below 1500ms, establishing a performance baseline. |

---

## Collection Variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `baseUrl` | `https://fakestoreapi.com` | Base URL for all requests |
| `expectedProductCount` | `20` | Expected total products in catalogue |
| `expectedUsersCount` | `10` | Expected total registered users |
| `countOfAllCarts` | *(set at runtime)* | Captured from Get All Carts response |
| `Cart7Prdt18Qty` | *(set at runtime)* | Quantity of product 18 in cart 7, used for update verification |

---

## Test Coverage Detail

### 🔐 Auth
| Request | Assertions |
|---------|-----------|
| Login with valid credentials | Status 200, JWT token present in response |
| Update user password | Status 200, updated password reflected in response |
| Login with invalid credentials | Status 401, error response validated |

### 📦 Products
| Request | Assertions |
|---------|-----------|
| Get all products | Status 200, JSON content type, array schema, count = 20 |
| Get limited products in sorted order | Status 200, JSON content type, array schema, limit respected |
| Get product by ID | Status 200, AJV schema, ID match, title and category verified |
| Add new product | Status 200, AJV schema, backend-generated ID captured, title and category verified |
| Update existing product | Status 200, correct product ID in response, brand name verified |
| Delete product | Status 200, deleted product ID in response |
| Get non-existent product | Status 404 |
| Delete non-existent product | Status 404, error status in response object |

### 👤 Users
| Request | Assertions |
|---------|-----------|
| Get all users | Status 200, JSON content type, AJV schema, non-empty list, count = 10 |
| Get user by ID | Status 200, JSON content type, AJV schema, ID match, username verified |
| Add new user | Status 200, JSON content type, backend-generated ID captured |
| Delete user | Status 200, correct user ID confirmed deleted |
| Get non-existent user | Status 404, error status in response object |
| Delete non-existent user | Status 404, error status in response object |

### 🛒 Carts
| Request | Assertions |
|---------|-----------|
| Get all carts | Status 200, JSON content type, AJV schema, valid array |
| Get cart by ID | Status 200, JSON content type, AJV schema, ID match |
| Add cart to user | Status 200, JSON content type, user ID and product ID verified |
| Add product to existing cart | Status 200, cart ID verified, products present |
| Update product quantity | Status 200, cart ID verified, before/after quantity delta validated |
| Zero quantity update | Status 200, cart ID verified, product removed from cart |
| Delete cart | Status 200, JSON content type, deletion confirmed |
| Add cart to non-existent user | Status 404, error status in response object |
| Update cart with invalid product | Status 404, error status in response object |
| Delete non-existent cart | Status 404, error status in response object |

---

## Bugs Discovered

| # | Request | Issue | Severity |
|---|---------|-------|----------|
| 1 | Get Non-Existing Product | Returns `200 OK` instead of `404 Not Found` for a non-existent product ID | 🔴 Critical |
| 2 | Zero Quantity Update of Cart Item | Allows setting an item quantity to zero — at least one quantity item must be present on a cart | 🟠 High |
| 3 | Add Cart to Non-Existing User | Returns `201 Created` instead of `404 Not Found` for a non-existent user ID | 🟡 Low |
| 4 | Add Non-Existing Product to Cart | Returns `200 OK` instead of `404 Not Found` when adding a non-existent product to a cart | 🟡 Low |

---

## Project Structure

```
estore-api-testing/
├── .github/
│   └── workflows/
│       └── newman.yml                          # CI pipeline
├── ESTORE_API_Testing.postman_collection.json  # Main test collection
├── environment.json                            # Postman environment (baseUrl)
├── newman-report/                              # Generated on CI run (gitignored)
├── package.json
├── package-lock.json
└── README.md
```

---

## How to Run

### Prerequisites
```bash
node -v   # v18+
npm -v
```

### Install
```bash
npm install
```

### Run collection locally
```bash
npm test
```

This executes the full collection against `https://fakestoreapi.com` and generates an HTML report at `newman-report/index.html`.

### Open the report
```bash
open newman-report/index.html     # macOS
start newman-report/index.html    # Windows
```

---

## CI/CD Pipeline

**GitHub Actions** (`.github/workflows/newman.yml`)

| Trigger | Action |
|---------|--------|
| Push to `main` / `develop` | Full collection run + report deploy |
| Pull Request to `main` | Full collection run (no deploy) |
| Manual (`workflow_dispatch`) | Full collection run + report deploy |

> ⚠️ **Note on CI reliability**: FakeStore API is a free public mock with no uptime SLA. CI runs may occasionally return `403 Forbidden` due to IP-based rate limiting from GitHub Actions' cloud runners — this is an API-side constraint, not a test defect. Run locally for consistent results.

---

## Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| **Postman** | Latest | Collection authoring, schema validation, test scripting |
| **Newman** | ^6.2.2 | CLI runner for CI execution |
| **newman-reporter-htmlextra** | ^1.23.1 | Rich HTML test reports |
| **AJV** | v8 (bundled) | JSON Schema validation in test scripts |
| **GitHub Actions** | — | CI pipeline |
| **GitHub Pages** | — | Live report hosting |

---

## Part of a Broader QA Portfolio

This project is one layer of a multi-skill QA portfolio:

| Project | Stack | What It Demonstrates |
|---------|-------|----------------------|
| **FakeStore API** *(this project)* | Postman · Newman · GitHub Actions | REST API contract testing, schema validation, negative cases, CI reporting |
| **OrangeHRM Automation Suite** | Playwright · TypeScript · Allure | UI + API hybrid automation, POM architecture, RBAC testing, cross-browser CI |
| **OpenCart Manual Testing** | Google Sheets · GitHub | Test case design, exploratory testing, defect reporting, test summary |

Each layer demonstrates a distinct, stackable skill set.

---

## References

- [FakeStore API Documentation](https://fakestoreapi.com/docs)
- [Postman Learning Center](https://learning.postman.com)
- [Newman Documentation](https://learning.postman.com/docs/collections/using-newman-cli/command-line-integration-with-newman/)
- [newman-reporter-htmlextra](https://github.com/DannyDainton/newman-reporter-htmlextra)
- [AJV JSON Schema Validator](https://ajv.js.org)