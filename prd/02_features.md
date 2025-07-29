# 2. Features

This section details the core features of the Impala application, categorized by user workflow. Each feature includes a description of its purpose and the criteria by which its successful implementation will be measured.

## 2.1. Core Platform & User Management

These features form the foundation of the application, ensuring a secure and personalized user experience.

### **2.1.1. Secure User Authentication**

- **Description:** Provides a complete and secure system for users to register, log in, and manage their accounts. This is the entry point to the entire application, protecting all user data and projects.
- **Acceptance Criteria:**
  - Users can register for a new account with a unique username and a securely hashed password.
  - Registered users can log in to receive a JSON Web Token (JWT) for session management.
  - All API endpoints related to user-specific data (projects, canvases) are protected and require a valid JWT.
  - Users can navigate to an "Account" page to update their first and last name.
  - Users can log out, which invalidates their session token.

### **2.1.2. Project-Based Workspace**

- **Description:** Enables users to organize their work into distinct projects. Each project acts as a container for related data sources and analysis canvases, creating a structured and manageable workspace.
- **Acceptance Criteria:**
  - Authenticated users can create a new project by providing a name and an optional description.
  - A user's projects are listed in a persistent sidebar for easy navigation.
  - Each project is owned by the user who created it, and cannot be accessed by other users unless explicitly shared.
  - Clicking a project opens a dedicated detail view for that project.

## 2.2. Data Management & Preparation (Data Studio)

This group of features focuses on getting data into Impala and preparing it for analysis. The goal is to automate the mundane and time-consuming tasks of data cleaning and structuring.

### **2.2.1. CSV Data Upload**

- **Description:** Allows users to upload CSV files directly into a project. This is the primary method for adding new data sources to the application.
- **Acceptance Criteria:**
  - Within a project's "Data" tab, a user can select and upload a CSV file.
  - The uploaded file is associated with the current project.
  - The system can handle files up to 1GB and shows a progress indicator during upload.
  - Uploaded files are listed in the "Data Sources" view within their project.

### **2.2.2. AI-Powered Data Profiling & Cleaning**

- **Description:** Automates the initial analysis of uploaded data. Impala will intelligently inspect the data to suggest data types and identify common issues, drastically reducing the manual effort required for data preparation.
- **Acceptance Criteria:**
  - Upon upload, the system automatically detects column data types (e.g., string, number, date) with at least 90% accuracy.
  - A preview of the data (first 100 rows) is displayed to the user.
  - The UI allows the user to override or correct the AI-suggested data types for any column.
  - The system heuristically detects wide-format data (e.g., months as columns) and suggests a one-click "unpivot" transformation.

### **2.2.3. Automated ETL and Data Refresh**

- **Description:** Turns the user-defined mappings and cleaning steps into a repeatable pipeline. This allows for scheduled data refreshes, ensuring that analyses are always based on the latest information without manual intervention.
- **Acceptance Criteria:**
  - A user-defined mapping can be saved as a repeatable ETL pipeline.
  - Pipelines can be run manually or scheduled to run on an hourly, daily, or weekly basis.
  - The system provides a monitoring dashboard showing the run history, status, and any errors for each pipeline.
  - If a scheduled run fails, an in-app notification is sent to the data source owner.

## 2.3. Analysis & Visualization (Analysis Composer)

These features provide the interactive canvas where users explore their data, build visualizations, and discover insights. The experience is designed to be beautiful, intuitive, and incredibly fast.

### **2.3.1. Interactive Analysis Canvas**

- **Description:** A flexible, grid-based canvas where users can create and arrange various analytical components. Canvases are stored in MongoDB to allow for a flexible, JSON-based structure. This is the core workspace for building an analysis.
- **Acceptance Criteria:**
  - Users can create a new, named canvas within a project.
  - All canvases for a project are listed in the "Canvases" tab.
  - The full CRUD (Create, Read, Update, Delete) functionality for canvases is implemented.
  - The canvas supports adding, resizing, and repositioning content blocks.

### **2.3.2. AI-Assisted Chart Generation**

- **Description:** Enables users to create rich visualizations using natural language prompts. The AI interprets the user's request, selects an appropriate chart type, and renders it on the canvas, applying the correct business logic from the metadata graph.
- **Acceptance Criteria:**
  - A prompt bar allows users to type queries like "bar chart of sales by month."
  - The LLM translates the query into a valid chart specification (e.g., chart type, x-axis, y-axis).
  - The resulting chart is rendered on the canvas instantly.
  - The system correctly applies predefined aggregation rules (e.g., SUM, AVERAGE, RECOMPUTE) when generating charts.

### **2.3.3. AI-Assisted Table Generation**

- **Description:** Allows for the quick creation of tabular data summaries through natural language. Users can request specific cuts of data without needing to manually configure pivot tables.
- **Acceptance Criteria:**
  - Users can prompt for tabular data, such as "show me revenue and profit by country."
  - The system generates a table with the requested dimensions and metrics.
  - Tables correctly reflect the aggregation logic and formatting defined in the metadata.

### **2.3.4. AI-Generated Narrative & Insights**

- **Description:** Automatically generates plain-language summaries and insights for any visualization. Every number in the narrative is linked back to the data, making the analysis transparent and trustworthy.
- **Acceptance Criteria:**
  - When a chart or table is created, a concise text summary is automatically generated below it.
  - Numeric values within the narrative are linked to the corresponding data points in the visual.
  - Hovering over a linked number in the text highlights the relevant part of the chart or table.
  - The AI can be prompted to "Explain" a specific data point or trend, generating a more detailed narrative.

## 2.4. Collaboration & Sharing

These features transform the canvas from a personal workspace into a collaborative hub for teams to discuss and iterate on analysis.

### **2.4.1. Canvas Sharing & Permissions**

- **Description:** Enables the owner of a canvas to share it with other users and assign specific roles, ensuring that collaborators have the appropriate level of access.
- **Acceptance Criteria:**
  - A canvas owner can share a canvas with another registered user via a unique link or invitation.
  - The following roles can be assigned:
    - **Commenter:** Can view the canvas and add comments, but cannot make edits.
    - **Editor:** Can view, comment on, and edit the canvas content (charts, text). Cannot access or modify underlying data sources.
    - **Full Access:** Has full editor rights and can also access the underlying project to view and manipulate data sources.
  - The sharing interface is intuitive and allows owners to easily manage and revoke access.

### **2.4.2. Canvas Comments**

- **Description:** Allows users with the appropriate permissions to leave comments directly on the canvas or on specific blocks within it, facilitating discussion and feedback.
- **Acceptance Criteria:**
  - Users with "Commenter" roles or higher can add comments to any block on the canvas.
  - Comments are displayed in a threaded view.
  - Users can be @mentioned in comments to trigger in-app notifications.

### **2.4.3. Version Control & History**

- **Description:** Provides a change log for each canvas, allowing users to track edits over time and revert to previous versions. This ensures that no work is accidentally lost and provides a clear audit trail.
- **Acceptance Criteria:**
  - **Phase 1 (MVP):** "Undo" and "Redo" buttons are available on the canvas, allowing users to step backward and forward through their recent changes within a session.
  - **Phase 2 (Long-term):** A full version history is available for each canvas, showing a log of saved changes with timestamps and authors.
  - Users can view and restore any previous version of the canvas.
