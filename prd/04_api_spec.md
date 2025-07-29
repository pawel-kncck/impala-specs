# **API Specifications**

This section details the API endpoints that power the Impala application. All endpoints are prefixed with `/api`.

## **Authentication**

- **`POST /register`**

  - **Description:** Registers a new user.
  - **Request Body:**
    ```json
    {
      "username": "newuser",
      "password": "securepassword"
    }
    ```
  - **Response:**
    ```json
    {
      "status": "success",
      "message": "User 'newuser' registered."
    }
    ```

- **`POST /login`**

  - **Description:** Authenticates a user and returns a JWT.
  - **Request Body:**
    ```json
    {
      "username": "testuser",
      "password": "password"
    }
    ```
  - **Response:**
    ```json
    {
      "access_token": "your.jwt.token",
      "token_type": "bearer"
    }
    ```

## **User Management**

- **`GET /me`**

  - **Description:** Retrieves the current user's profile information.
  - **Authentication:** JWT required.
  - **Response:**
    ```json
    {
      "id": 1,
      "username": "testuser",
      "first_name": "John",
      "last_name": "Doe"
    }
    ```

- **`PUT /me/update`**

  - **Description:** Updates the current user's profile.
  - **Authentication:** JWT required.
  - **Request Body:**
    ```json
    {
      "first_name": "John",
      "last_name": "Doe"
    }
    ```
  - **Response:** The updated user object.

## **Projects**

- **`POST /projects`**

  - **Description:** Creates a new project.
  - **Authentication:** JWT required.
  - **Request Body:**
    ```json
    {
      "name": "New Marketing Campaign",
      "description": "Analysis of Q3 campaign data."
    }
    ```
  - **Response:** The newly created project object.

- **`GET /projects`**

  - **Description:** Retrieves all projects for the current user.
  - **Authentication:** JWT required.
  - **Response:** A list of project objects.

## **Data Sources**

- **`POST /projects/{project_id}/upload-csv`**

  - **Description:** Uploads a CSV file to a project.
  - **Authentication:** JWT required.
  - **Request Body:** `multipart/form-data` containing the file.
  - **Response:** The newly created data source object.

- **`GET /projects/{project_id}/data-sources`**

  - **Description:** Retrieves all data sources for a specific project.
  - **Authentication:** JWT required.
  - **Response:** A list of data source objects.

## **Canvases**

- **`POST /projects/{project_id}/canvases`**

  - **Description:** Creates a new analysis canvas within a project.
  - **Authentication:** JWT required.
  - **Request Body:**
    ```json
    {
      "name": "Q3 Sales Analysis"
    }
    ```
  - **Response:** The newly created canvas object.

- **`GET /projects/{project_id}/canvases`**

  - **Description:** Retrieves all canvases for a specific project.
  - **Authentication:** JWT required.
  - **Response:** A list of canvas objects.

- **`PUT /canvases/{canvas_id}`**

  - **Description:** Updates a specific canvas.
  - **Authentication:** JWT required.
  - **Request Body:**
    ```json
    {
      "name": "Updated Canvas Name",
      "content": { ... }
    }
    ```
  - **Response:** The updated canvas object.

- **`DELETE /canvases/{canvas_id}`**

  - **Description:** Deletes a specific canvas.
  - **Authentication:** JWT required.
  - **Response:** `204 No Content`.
