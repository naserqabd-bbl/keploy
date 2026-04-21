# MFE + BFF + Microservice + Keploy Starter

A ready-to-run starter project with:

- **Micro frontend (MFE)** using React + Vite Module Federation
- **BFF** using Express
- **Microservice** using Express
- **Keploy-ready testing flow** for record/replay and contract-style API validation
- **Docker Compose** for local orchestration

## Architecture

```text
Browser
  -> Shell App (port 5173)
    -> loads Catalog MFE (port 5174)
    -> calls BFF /api/products (port 8080)
      -> proxies to Product Service /products (port 8081)
```

## Project structure

```text
apps/
  shell/           # host application
  catalog-mfe/     # remote micro frontend
services/
  bff/             # backend for frontend
  product-service/ # downstream microservice
scripts/
  smoke-test.js    # simple runtime validation
```

## Quick start

### 1) Install dependencies

```bash
npm install
```

### 2) Run everything locally

```bash
npm run dev
```

Open:

- Shell: http://localhost:5173
- Remote MFE entry: http://localhost:5174/assets/remoteEntry.js
- BFF health: http://localhost:8080/health
- Product service health: http://localhost:8081/health

### 3) Run the smoke test

```bash
npm run test:smoke
```

## Run with Docker Compose

```bash
docker compose up --build
```

## Keploy workflow

Keploy’s current docs describe two main paths: **AI test generation from OpenAPI/Postman/cURL** and **record/replay traffic-based testing**. The docs also describe contract testing as capturing API interactions and validating them against recorded baselines or OpenAPI-style expectations. citeturn346780view0turn346780view1turn346780view2

### Install Keploy

```bash
curl --silent -O -L https://keploy.io/install.sh && source install.sh
```

### Record BFF interactions as tests

In one terminal:

```bash
keploy record -c "npm run dev -w services/bff"
```

In other terminals, run the product service and hit the BFF:

```bash
npm run dev -w services/product-service
curl http://localhost:8080/api/products
```

### Replay tests

```bash
keploy test -c "npm run dev -w services/bff" --delay 10
```

## Suggested contract-testing scenario

1. Record a passing interaction from BFF to product-service.
2. Change the product-service response shape, such as renaming `price` to `unitPrice`.
3. Re-run Keploy tests to detect schema drift or a breaking response change.

## Environment variables

### BFF

- `PORT` default `8080`
- `PRODUCT_SERVICE_URL` default `http://localhost:8081`

### Product service

- `PORT` default `8081`

## Notes

- This starter keeps the example intentionally small so you can extend it into auth, multiple MFEs, service discovery, Kafka, or database-backed services.
- The UI fetches data only through the BFF, not directly from the microservice.
