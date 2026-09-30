# Day 3 — Application Analysis Document

## Project Name: CloudDeploy
**Company:** NexaCore Technologies  
**Target Application:** ProShop v2  
**Analyzed By:** DevOps Engineering Team  

---

### 1. Technical Stack & Runtime Specifications

* **Programming Language:** JavaScript (Node.js runtime, ES6+ modules via )
* **Backend Framework:** Express.js (v4.x)
* **Frontend Framework:** React.js (v18.x) with Redux Toolkit and React-Bootstrap
* **Package Manager:** npm (Node Package Manager)
* **Application Entry Points:**
  * Backend Entry Point: 
  * Frontend Entry Point: 
* **Application Ports:**
  * Backend API Server: Port `5000`
  * Frontend Development Server: Port `3000`
  * Database Engine: Port `27017`
* **Key Dependencies:**
  * Backend: `express`, `mongoose`, `dotenv`, `bcryptjs`, `jsonwebtoken`, `cookie-parser`
  * Frontend: `react`, `react-dom`, `react-router-dom`, `@reduxjs/toolkit`, `react-redux`, `react-bootstrap`
  * Concurrency & Tooling: `concurrently`, `nodemon`
* **Build Command:**
  * Frontend Build: `npm run build` (runs `npm run build --prefix frontend`, compiling the React client into static production assets inside `frontend/build`)
* **Start Commands:**
  * Production Start: `npm start` (launches Node runtime: `node backend/server.js`)
  * Development Start: `npm run dev` (runs both backend nodemon and frontend React dev server concurrently)
* **Environment Variables:**
  * `NODE_ENV`: Runtime environment state (`development` vs. `production`)
  * `PORT`: Port on which the Express server listens (default `5000`)
  * `MONGO_URI`: Connection connection string pointing to the MongoDB database instance
  * `JWT_SECRET`: Secret key used for signing and verifying JSON Web Tokens
  * `PAYPAL_CLIENT_ID`: Client ID credential used for PayPal checkout gateway integration
* **External Services:**
  * PayPal Sandbox API (for client-side payment processing and transaction verification)
* **Database System:**
  * MongoDB (NoSQL Document Store) interfaced using Mongoose Object Data Modeling (ODM)

---

### 2. Core Architectural & Operational Questions

#### Q1: How does the application start?
* **Development Mode:** Running `npm run dev` uses the `concurrently` package to trigger two simultaneous processes: `nodemon backend/server.js` (which connects to MongoDB and binds to port 5000) and `npm start --prefix frontend` (which launches the Webpack development server hosting the React SPA on port 3000).
* **Production Mode:** The frontend is pre-compiled into static HTML, CSS, and JS bundles via `npm run build`. The command `npm start` then runs `node backend/server.js`. Express serves both the REST API endpoints under `/api/*` and static frontend files under `frontend/build`.

#### Q2: Which port does it use?
* **Port 5000:** Express API server.
* **Port 3000:** React development server (in development mode).
* **Port 27017:** MongoDB local database listening port.

#### Q3: What happens when a user opens the application?
1. The user navigates to `http://localhost:3000` (or domain in production).
2. The browser downloads the React Single Page Application (HTML, JS, CSS bundles).
3. The React client mounts and Redux triggers asynchronous API requests (e.g., `GET /api/products`) to port 5000.
4. The Express server intercepts the API request, executes authentication middleware if required, and queries MongoDB using Mongoose.
5. MongoDB returns document data back to Express.
6. Express serializes the data into JSON and returns it to the client.
7. Redux stores the JSON response, and React renders the product catalog and UI components dynamically.

#### Q4: What dependencies are required?
* System runtime requires **Node.js (v18 or v20)** and **npm**.
* Storage layer requires a running instance of **MongoDB (v6.0 or v7.0+)**.

#### Q5: Which configuration should be externalized?
All environment-specific parameters must remain outside the codebase via a `.env` file or container environment variables:
* Database connection string (`MONGO_URI`)
* Application port (`PORT`)
* Environment mode (`NODE_ENV`)
* Security credentials and encryption keys (`JWT_SECRET`)
* External payment gateway keys (`PAYPAL_CLIENT_ID`)

#### Q6: Does the application require a database?
Yes. ProShop v2 requires MongoDB to store collections of:
* **Users:** Names, hashed passwords (bcrypt), admin privileges.
* **Products:** Names, prices, descriptions, stock inventory, ratings, reviews.
* **Orders:** Purchased line items, shipping addresses, payment status, timestamps.

---

### 3. Application Architecture Diagram

```text
+------------------+
|   Client User    |
| (Web Browser UI) |
+--------+---------+
         |
         | HTTP Requests (Port 3000 Dev / Port 80 Prod)
         v
+------------------+
|  Frontend Layer  |
|   (React SPA)    |
+--------+---------+
         |
         | REST API Calls: /api/products, /api/users (Port 5000)
         v
+------------------+
|  Backend Layer   |
| (Node / Express) |
+--------+---------+
         |
         | Mongoose ODM Queries (Port 27017)
         v
+------------------+          +-------------------------+
|     Database     |          |    External Gateway     |
|    (MongoDB)     |          | (PayPal Sandbox Server) |
+------------------+          +-------------------------+
         ^                                 ^
         | (Persistent Storage)            | (Payment Verification)
         +---------------------------------+
```
