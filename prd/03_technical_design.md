# 3. Technical Design

This section outlines the technical architecture and key implementation details for the Impala application. The design prioritizes a modern, scalable, and maintainable stack, enabling the rapid development of features while ensuring a robust and secure user experience.

## 3.1. System Architecture

The Impala application follows a classic **client-server architecture**, composed of three main tiers: a **frontend web application**, a **backend API**, and a **data persistence layer**. To facilitate development, deployment, and scalability, the entire system is containerized using **Docker** and managed with **Docker Compose**.

- **Frontend**: A responsive single-page application (SPA) built with **React** and **Vite**. This choice provides a fast, modern development experience and a highly interactive user interface. The frontend is served by a lightweight **Nginx** web server in production.

- **Backend**: A **Python-based API** built with the **FastAPI** framework. FastAPI's high performance, asynchronous support, and automatic data validation make it an ideal choice for building a robust and scalable API. The backend is served by a **Gunicorn** WSGI server with **Uvicorn** workers.

- **Persistence Layer**: The application utilizes a hybrid data persistence model to best suit the needs of different data types:
  - **PostgreSQL**: A powerful open-source relational database used for structured data, including user accounts, project information, and data source metadata.
  - **MongoDB**: A NoSQL document database used for storing unstructured and semi-structured data, such as the JSON-based content of analysis canvases.
  - **File System**: A simple `uploads/` directory on the backend server is used for storing uploaded CSV files.

## 3.2. Technology Stack

- **Frontend**:

  - Framework: **React 19** with **Vite** for building and development
  - Routing: **React Router DOM**
  - HTTP Client: **Axios**
  - Styling: **CSS** (with separate files for each component)
  - Package Manager: **npm**

- **Backend**:

  - Framework: **FastAPI**
  - Server: **Gunicorn** with **Uvicorn** workers
  - Databases:
    - **PostgreSQL** (with `psycopg2-binary` driver) for structured data
    - **MongoDB** (with `pymongo` driver) for canvas data
  - Authentication: **JWT (JSON Web Tokens)** with the `python-jose` library
  - Password Hashing: **Passlib** with **bcrypt**
  - Data Handling: **Pandas** for CSV processing
  - Package Manager: **pip** with `requirements.txt`

- **DevOps**:
  - Containerization: **Docker**
  - Orchestration: **Docker Compose**

## 3.3. Authentication and Authorization

Authentication is handled via **JSON Web Tokens (JWT)**.

1.  A user registers or logs in with their username and password.
2.  The backend verifies the credentials against the `users` table in the PostgreSQL database.
3.  Upon successful authentication, the backend generates a short-lived JWT containing the user's username.
4.  The frontend stores this token in `localStorage` and includes it in the `Authorization` header for all subsequent API requests.
5.  Protected API endpoints verify the JWT signature and expiration to authorize the request.

This token-based approach ensures that the backend remains stateless and that all user-specific data is protected.

## 3.4. Data Flow and Management

1.  **User and Project Data**: All user and project metadata is stored in the **PostgreSQL** database. The backend API provides standard CRUD (Create, Read, Update, Delete) endpoints for managing these resources.

2.  **Data Upload**: Users can upload CSV files through the frontend. The files are sent to a dedicated backend endpoint, which saves them to a local `uploads/` directory and creates a corresponding entry in the `data_sources` table in PostgreSQL.

3.  **Analysis Canvas Data**: The flexible, grid-based structure of the analysis canvases is well-suited for a document-based database. Canvases are stored as JSON documents in a **MongoDB** collection. This allows for easy addition, resizing, and repositioning of various content blocks (charts, tables, text) without the constraints of a rigid schema.

## 3.5. Development and Deployment

- **Development**: The `docker-compose.override.yml` file configures a hot-reloading development environment. Changes to the frontend or backend code will automatically trigger a rebuild and restart of the relevant service, enabling a fast and efficient development workflow. The frontend is served by Vite's development server on port `5173`, and the backend API is available on port `5001`.

- **Production**: For production deployment, the `docker-compose.yml` and the frontend `Dockerfile` are used. The React application is built into static assets, which are then served by a production-ready **Nginx** container. The backend runs in a separate container, and all services are connected via a dedicated Docker network. This setup ensures a secure, performant, and easily deployable application.
