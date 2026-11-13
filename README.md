# WEB-DEV-3-LAB-4 - Secure Connect Platform (Lab 4)

> **Student Submission Details:**  
> - **Student Name:** Anubhav  
> - **Section:** F  
> - **Roll Number:** 2501730406  
> - **Project:** Lab 4 - Secure Connect Platform (REST API & Real-Time Communication)

---

## 📌 Project Overview

**Secure Connect Platform** is a full-featured backend application built with **Node.js**, **Express.js**, **MongoDB (Mongoose)**, and **Socket.IO**. It demonstrates enterprise-grade authentication, authorization, role-based access control (RBAC), user-scoped data persistence, decoupled event-driven workflows, and authenticated real-time WebSocket communication.

---

## ✨ Key Features

1. **Robust Authentication & Security**
   - Password encryption using **Bcrypt** (10 salt rounds).
   - Stateless authentication using **JSON Web Tokens (JWT)** with a 24-hour validity window.
   - Protected API route middleware verifying Bearer tokens in headers.

2. **Role-Based Access Control (RBAC)**
   - Multi-tier role management (`user` vs. `admin`).
   - Dedicated `adminMiddleware` enforcing privilege checks for elevated endpoints.

3. **Secure Note Management (CRUD with Ownership Scoping)**
   - Create, Read, Update, and Delete notes.
   - Strict resource isolation: Regular users can only access and manipulate their own notes.
   - Admin-exclusive endpoint to inspect all platform notes populated with author details.

4. **Event-Driven Architecture (Node.js EventEmitter)**
   - Internal pub/sub event bus decoupled from the request-response cycle.
   - Lifecycle triggers: `noteCreated`, `noteUpdated`, `noteDeleted` for audit logging and side effects.

5. **Real-Time Communication (Socket.IO)**
   - Secure WebSocket connection authenticated via JWT in the handshake middleware.
   - Room-based real-time chat broadcasting (`joinRoom`, `chatMessage`).
   - Automatic timestamping and user attribution on socket messages.

---

## 🛠️ Technology Stack

| Technology | Purpose |
| :--- | :--- |
| **Node.js** | Server-side JavaScript runtime environment |
| **Express.js** (v5) | High-performance web application framework |
| **MongoDB & Mongoose** | NoSQL database and schema-based object data modeling (ODM) |
| **JSON Web Token (JWT)** | Secure token-based user authentication and claims |
| **Bcrypt** | Industry-standard password hashing algorithm |
| **Socket.IO** (v4) | Bidirectional low-latency event-based real-time communication |
| **EventEmitter** | Native asynchronous event dispatching mechanism |
| **Dotenv** | Zero-dependency environment variable configuration |
| **Thunder Client** | REST API testing directly within VS Code |

---

## 📂 Project Structure

```text
├── middleware/
│   └── authMiddleware.js        # JWT verification & admin RBAC middleware
├── models/
│   ├── Note.js                  # Mongoose schema for Notes with user reference
│   └── User.js                  # Mongoose schema for Users (roles: user, admin)
├── routes/
│   ├── authRoutes.js            # User signup and login route handlers
│   └── noteRoutes.js            # Protected CRUD routes for notes + admin review
├── test on thunderclient/       # Visual API test evidence and screenshots
│   ├── Screenshot (355).png     # Test: POST /api/auth/signup (201 Created)
│   ├── Screenshot (356).png     # Test: POST /api/auth/login (200 OK + JWT)
│   ├── Screenshot (357).png     # Test: GET /api/notes without token (401 Unauthorized)
│   ├── Screenshot (358).png     # Test: Header inspection without token
│   ├── Screenshot (361).png     # Test: GET /api/notes with Bearer token (200 OK [])
│   ├── Screenshot (362).png     # Test: POST /api/notes Create note (201 Created)
│   ├── Screenshot (363).png     # Test: GET /api/notes Retrieve notes (200 OK)
│   ├── Screenshot (364).png     # Test: GET /api/notes/:id Retrieve single note (200 OK)
│   ├── Screenshot (365).png     # Test: PUT /api/notes/:id Update note (200 OK)
│   ├── Screenshot (366).png     # Test: DELETE /api/notes/:id Delete note (200 OK)
│   └── Screenshot (367).png     # Test: GET /api/notes/admin/all non-admin check (403 Forbidden)
├── .env.example                 # Sample environment variables template
├── .env                         # Environment configuration file
├── .gitignore                   # Files and directories ignored by Git
├── package.json                 # Project manifest, metadata, scripts & dependencies
├── package-lock.json            # Exact dependency lockfile
├── server.js                    # Entry point: Express server, Socket.IO & MongoDB init
└── README.md                    # Project documentation & execution guide
```

---

## ⚙️ Installation & Setup

### 1. Prerequisites
- **Node.js** (v18.0.0 or higher recommended)
- **npm** (v9.0.0 or higher)
- **MongoDB** running locally (`mongodb://localhost:27017`) or a **MongoDB Atlas** connection string

### 2. Clone / Open Project
Navigate to the root directory of the project:
```bash
cd -Lab4-Secure-Connect-Platform-main
```

### 3. Install Dependencies
```bash
npm install
```

### 4. Configure Environment Variables
Create or verify your `.env` file in the root folder with the following variables:
```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/secure_connect_lab4
JWT_SECRET=supersecretjwtkey_anubhav_lab4
```

### 5. Run the Server
- **Production mode:**
  ```bash
  npm start
  ```
- **Development mode (with auto-reload via nodemon):**
  ```bash
  npm run dev
  ```

Once started, the terminal will display:
```text
MongoDB Connected Successfully
Server running on port 5000
```

---

## 📡 API Reference & Documentation

### Base URL: `http://localhost:5000`

---

### 1. Authentication Endpoints (`/api/auth`)

#### `POST /api/auth/signup`
Registers a new user on the platform. Passwords must be at least 6 characters.

- **Access:** Public
- **Request Body:**
  ```json
  {
    "username": "anubhav",
    "email": "anubhav@example.com",
    "password": "password123"
  }
  ```
- **Success Response (`201 Created`):**
  ```json
  {
    "message": "User registered",
    "id": "6734c5e317d2d451279f9e72",
    "username": "anubhav",
    "email": "anubhav@example.com"
  }
  ```

#### `POST /api/auth/login`
Authenticates existing credentials and returns a signed JWT.

- **Access:** Public
- **Request Body:**
  ```json
  {
    "email": "anubhav@example.com",
    "password": "password123"
  }
  ```
- **Success Response (`200 OK`):**
  ```json
  {
    "message": "Login successful",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
  ```

---

### 2. Notes Endpoints (`/api/notes`)
All notes routes require the authorization header:
`Authorization: Bearer <your_jwt_token>`

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/notes` | Authenticated User | Create a new note owned by current user |
| `GET` | `/api/notes` | Authenticated User | Get all notes created by current user |
| `GET` | `/api/notes/:id` | Authenticated User | Retrieve a specific note by ID |
| `PUT` | `/api/notes/:id` | Authenticated User | Update note title or content by ID |
| `DELETE` | `/api/notes/:id` | Authenticated User | Delete a note by ID |
| `GET` | `/api/notes/admin/all` | **Admin Only** | Fetch all notes across users (with user details) |

#### Example: Create Note (`POST /api/notes`)
- **Headers:** `Authorization: Bearer <TOKEN>`
- **Request Body:**
  ```json
  {
    "title": "Web Dev Lab 4",
    "content": "Completed secure connect platform with Socket.IO and JWT"
  }
  ```
- **Success Response (`201 Created`):**
  ```json
  {
    "_id": "6734c7b517d2d451279f9e75",
    "title": "Web Dev Lab 4",
    "content": "Completed secure connect platform with Socket.IO and JWT",
    "owner": "6734c5e317d2d451279f9e72",
    "createdAt": "2026-11-13T14:55:00.000Z",
    "updatedAt": "2026-11-13T14:55:00.000Z"
  }
  ```

#### Example: Admin Only Check (`GET /api/notes/admin/all`)
- If accessed by a non-admin token:
  - **Status:** `403 Forbidden`
  - **Response:**
    ```json
    {
      "message": "Access denied. Admin only."
    }
    ```

---

## ⚡ Real-Time Socket.IO Communication

Socket.IO handles real-time rooms and chat messaging with mandatory token authentication during handshake.

### 1. Connecting to Socket
```javascript
const socket = io("http://localhost:5000", {
  auth: {
    token: "<YOUR_JWT_TOKEN>"
  }
});
```

### 2. Joining a Room
```javascript
socket.emit("joinRoom", "project-room-f");
```

### 3. Sending a Message
```javascript
socket.emit("chatMessage", "Hello from Anubhav!");
```

### 4. Listening for Messages
```javascript
socket.on("chatMessage", (data) => {
  console.log(`[${data.room}] ${data.username}: ${data.message} (${data.time})`);
});
```

---

## 📢 Event-Driven Lifecycle (EventEmitter)

Whenever a note mutation occurs, the server emits an internal event via `app.get("eventEmitter")`:
- `noteCreated` ➔ Emitted on `POST /api/notes`
- `noteUpdated` ➔ Emitted on `PUT /api/notes/:id`
- `noteDeleted` ➔ Emitted on `DELETE /api/notes/:id`

Console Output Example:
```text
New note created: Web Dev Lab 4
Note updated: Web Dev Lab 4
Note deleted: Web Dev Lab 4
```

---

## 🧪 Testing & Verification Evidence

All endpoints have been tested using **Thunder Client**. Complete test output screenshots are provided in the [`test on thunderclient/`](./test%20on%20thunderclient/) folder:

| Test Case | Method & Endpoint | Status | Screenshot File |
| :--- | :--- | :--- | :--- |
| User Registration | `POST /api/auth/signup` | `201 Created` | `Screenshot (355).png` |
| User Login & Token Issuance | `POST /api/auth/login` | `200 OK` | `Screenshot (356).png` |
| Unauthorized Request Check | `GET /api/notes` (No Token) | `401 Unauthorized` | `Screenshot (357).png` |
| Authorization Header Inspector | `GET /api/notes` | `401 Unauthorized` | `Screenshot (358).png` |
| Empty Notes Collection Query | `GET /api/notes` (With Token) | `200 OK` | `Screenshot (361).png` |
| Create First Note | `POST /api/notes` | `201 Created` | `Screenshot (362).png` |
| Fetch User Notes | `GET /api/notes` | `200 OK` | `Screenshot (363).png` |
| Fetch Specific Note by ID | `GET /api/notes/:id` | `200 OK` | `Screenshot (364).png` |
| Update Note by ID | `PUT /api/notes/:id` | `200 OK` | `Screenshot (365).png` |
| Delete Note by ID | `DELETE /api/notes/:id` | `200 OK` | `Screenshot (366).png` |
| RBAC Admin Access Denial | `GET /api/notes/admin/all` | `403 Forbidden` | `Screenshot (367).png` |

---

## 👨‍💻 Author

- **Name:** Anubhav
- **Section:** F
- **Roll Number:** 2501730406
- **License:** ISC

