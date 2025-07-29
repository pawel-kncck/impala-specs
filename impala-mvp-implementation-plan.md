# Impala MVP Implementation Plan

## Phase 1: Foundation (MVP) - Granular Step-by-Step Implementation

This implementation plan provides an extremely granular, step-by-step approach to building the Impala MVP. Each step represents a single, atomic unit of work that results in a runnable and testable application state.

---

### Step 1: Initial Project Scaffolding

- **Goal:** Set up the basic monorepo structure and initialize the frontend and backend applications with necessary configurations.
- **Technical Tasks:**
  - Create the root directory `impala/`
  - Initialize a Git repository in the root with `git init`
  - Create a `frontend/` directory
  - Create a `backend/` directory
  - Create a `shared/` directory for shared types and utilities
  - Add a root `.gitignore` file with common ignores for Python, Node.js, and OS files
  - Add a root `README.md` with the project title and basic description
  - Create a root `package.json` with workspace configuration for the monorepo
- **Commit Message:** `chore: initial project scaffolding and monorepo setup`

### Step 2: Frontend React Application Setup

- **Goal:** Initialize a React application with TypeScript and essential configurations.
- **Technical Tasks:**
  - Navigate to `frontend/` directory
  - Initialize a new Vite React TypeScript project with `npm create vite@latest . -- --template react-ts`
  - Install dependencies with `npm install`
  - Add a basic `App.tsx` that displays "Impala - Data Analytics Platform"
  - Update `index.html` title to "Impala"
  - Configure TypeScript with strict mode in `tsconfig.json`
  - Verify the app runs with `npm run dev`
- **Commit Message:** `feat: initialize React frontend with TypeScript and Vite`

### Step 3: Backend FastAPI Application Setup

- **Goal:** Set up a basic Python FastAPI backend with virtual environment and project structure.
- **Technical Tasks:**
  - Navigate to `backend/` directory
  - Create a Python virtual environment with `python -m venv venv`
  - Create a `requirements.txt` file with initial dependencies: `fastapi==0.104.1`, `uvicorn[standard]==0.24.0`, `python-dotenv==1.0.0`
  - Create a `src/` directory for source code
  - Create `src/main.py` with a basic FastAPI app and health check endpoint
  - Create `.env.example` file with placeholder environment variables
  - Add Python-specific entries to `.gitignore`
  - Verify the backend runs with `uvicorn src.main:app --reload`
- **Commit Message:** `feat: initialize FastAPI backend with basic structure`

### Step 4: Database Setup with PostgreSQL Configuration

- **Goal:** Configure PostgreSQL database connection and create initial database schema setup.
- **Technical Tasks:**
  - Add database dependencies to `requirements.txt`: `sqlalchemy==2.0.23`, `psycopg2-binary==2.9.9`, `alembic==1.12.1`
  - Create `backend/src/database/` directory
  - Create `backend/src/database/connection.py` with SQLAlchemy database configuration
  - Create `backend/src/config.py` for environment variable management
  - Create `backend/alembic.ini` for database migration configuration
  - Initialize Alembic with `alembic init alembic`
  - Update `.env.example` with DATABASE_URL placeholder
  - Create a health check endpoint that verifies database connectivity
- **Commit Message:** `feat: add PostgreSQL database configuration and connection`

### Step 5: User Model and Database Schema

- **Goal:** Create the user model and generate the first database migration.
- **Technical Tasks:**
  - Create `backend/src/models/` directory
  - Create `backend/src/models/user.py` with User SQLAlchemy model (id, username, email, password_hash, created_at, updated_at)
  - Create `backend/src/models/__init__.py` to export models
  - Update Alembic configuration to import models
  - Generate first migration with `alembic revision --autogenerate -m "create user table"`
  - Create a Docker Compose file for local PostgreSQL development
  - Run migration with `alembic upgrade head`
- **Commit Message:** `feat: add User model and initial database migration`

### Step 6: JWT Authentication Setup

- **Goal:** Implement JWT token generation and validation for secure authentication.
- **Technical Tasks:**
  - Add authentication dependencies to `requirements.txt`: `python-jose[cryptography]==3.3.0`, `passlib[bcrypt]==1.7.4`, `python-multipart==0.0.6`
  - Create `backend/src/auth/` directory
  - Create `backend/src/auth/utils.py` with password hashing and JWT token functions
  - Create `backend/src/auth/dependencies.py` with FastAPI dependency for getting current user
  - Add JWT_SECRET_KEY and JWT_ALGORITHM to `.env.example`
  - Create unit tests for authentication utilities in `backend/tests/test_auth.py`
- **Commit Message:** `feat: implement JWT authentication utilities`

### Step 7: User Registration API Endpoint

- **Goal:** Create the user registration endpoint with proper validation and error handling.
- **Technical Tasks:**
  - Create `backend/src/schemas/` directory
  - Create `backend/src/schemas/user.py` with Pydantic schemas for user registration
  - Create `backend/src/api/` directory structure
  - Create `backend/src/api/v1/endpoints/users.py` with registration endpoint
  - Implement unique username/email validation
  - Add proper error responses for duplicate users
  - Create integration test for registration endpoint
  - Verify registration works via API testing tool
- **Commit Message:** `feat: add user registration endpoint with validation`

### Step 8: User Login API Endpoint

- **Goal:** Implement user login endpoint that returns JWT tokens.
- **Technical Tasks:**
  - Add login request/response schemas to `backend/src/schemas/user.py`
  - Create login endpoint in `backend/src/api/v1/endpoints/users.py`
  - Implement username/password validation
  - Return JWT access token on successful login
  - Add token expiration configuration
  - Create integration test for login endpoint
  - Test login flow with previously registered user
- **Commit Message:** `feat: add user login endpoint with JWT token generation`

### Step 9: Frontend Routing and Navigation Setup

- **Goal:** Set up React Router and basic navigation structure.
- **Technical Tasks:**
  - Install React Router with `npm install react-router-dom @types/react-router-dom`
  - Create `frontend/src/pages/` directory
  - Create `frontend/src/pages/Home.tsx` component
  - Create `frontend/src/pages/Login.tsx` component
  - Create `frontend/src/pages/Register.tsx` component
  - Set up routing in `App.tsx` with BrowserRouter
  - Add basic navigation links
  - Verify all routes render correctly
- **Commit Message:** `feat: add React Router with basic page structure`

### Step 10: Frontend API Client Setup

- **Goal:** Create a reusable API client for frontend-backend communication.
- **Technical Tasks:**
  - Install axios with `npm install axios`
  - Create `frontend/src/services/` directory
  - Create `frontend/src/services/api.ts` with axios instance configuration
  - Configure base URL from environment variables
  - Add request/response interceptors for error handling
  - Create `frontend/src/services/auth.ts` with auth-specific API calls
  - Add VITE_API_URL to `.env.example`
- **Commit Message:** `feat: add axios API client with service layer`

### Step 11: Registration Page UI Implementation

- **Goal:** Build a functional registration form with validation.
- **Technical Tasks:**
  - Install form library with `npm install react-hook-form @hookform/resolvers yup`
  - Create registration form component in `frontend/src/pages/Register.tsx`
  - Add form fields for username, email, password, confirm password
  - Implement client-side validation with Yup schema
  - Style form with basic CSS for clean appearance
  - Add loading state during submission
  - Display success/error messages
  - Connect form to registration API endpoint
- **Commit Message:** `feat: implement registration page with form validation`

### Step 12: Login Page UI Implementation

- **Goal:** Build a functional login form that stores JWT tokens.
- **Technical Tasks:**
  - Create login form component in `frontend/src/pages/Login.tsx`
  - Add form fields for username and password
  - Implement form validation
  - Add "Remember Me" checkbox
  - Style form consistently with registration page
  - Store JWT token in localStorage on successful login
  - Redirect to home page after login
  - Add link to registration page
- **Commit Message:** `feat: implement login page with JWT token storage`

### Step 13: Authentication State Management

- **Goal:** Implement global authentication state using React Context.
- **Technical Tasks:**
  - Create `frontend/src/contexts/` directory
  - Create `frontend/src/contexts/AuthContext.tsx` with auth state and methods
  - Implement login, logout, and user state management
  - Add axios interceptor to include JWT in requests
  - Create `useAuth` custom hook
  - Wrap App component with AuthProvider
  - Update navigation to show user state
  - Add logout functionality
- **Commit Message:** `feat: add authentication context and state management`

### Step 14: Protected Route Implementation

- **Goal:** Create protected routes that require authentication.
- **Technical Tasks:**
  - Create `frontend/src/components/ProtectedRoute.tsx` component
  - Implement authentication check and redirect logic
  - Create `frontend/src/pages/Dashboard.tsx` as protected page
  - Update routing to use ProtectedRoute wrapper
  - Add automatic redirect to login for unauthenticated users
  - Preserve intended destination after login
  - Test protected route behavior
- **Commit Message:** `feat: implement protected routes with auth guards`

### Step 15: Project Model and API Setup

- **Goal:** Create the project model and basic CRUD endpoints.
- **Technical Tasks:**
  - Create `backend/src/models/project.py` with Project model (id, name, description, owner_id, created_at, updated_at)
  - Add foreign key relationship to User model
  - Create migration with `alembic revision --autogenerate -m "create project table"`
  - Create `backend/src/schemas/project.py` with project schemas
  - Create `backend/src/api/v1/endpoints/projects.py` with CRUD endpoints
  - Implement authorization checks for project access
  - Add project routes to API router
- **Commit Message:** `feat: add Project model with CRUD endpoints`

### Step 16: Projects Dashboard UI

- **Goal:** Build a dashboard to display and manage user projects.
- **Technical Tasks:**
  - Create `frontend/src/pages/Projects.tsx` component
  - Create `frontend/src/services/projects.ts` with project API calls
  - Build project list view with grid layout
  - Add "Create New Project" button and modal
  - Implement project creation form
  - Add project cards with name and description
  - Include creation date and last modified info
  - Add loading states and empty state
- **Commit Message:** `feat: implement projects dashboard with creation flow`

### Step 17: File Upload Backend Infrastructure

- **Goal:** Set up file storage and upload handling infrastructure.
- **Technical Tasks:**
  - Add file handling dependencies: `python-multipart==0.0.6`, `aiofiles==23.2.1`
  - Create `backend/src/storage/` directory
  - Create `backend/src/storage/local.py` with local file storage implementation
  - Create `backend/src/models/file.py` with File model (id, filename, path, size, mime_type, project_id, uploaded_by, created_at)
  - Generate migration for files table
  - Create file upload endpoint in `backend/src/api/v1/endpoints/files.py`
  - Implement file size and type validation
  - Configure upload directory in environment variables
- **Commit Message:** `feat: add file upload infrastructure with local storage`

### Step 18: CSV File Upload UI

- **Goal:** Implement CSV file upload interface in the frontend.
- **Technical Tasks:**
  - Create `frontend/src/components/FileUpload.tsx` component
  - Implement drag-and-drop file selection
  - Add file type validation (CSV only)
  - Display upload progress bar
  - Create `frontend/src/services/files.ts` with upload API call
  - Add file upload to project detail page
  - Show list of uploaded files
  - Implement file size formatting utility
- **Commit Message:** `feat: add CSV file upload UI with drag-and-drop`

### Step 19: CSV Parsing and Data Preview

- **Goal:** Parse uploaded CSV files and provide data preview functionality.
- **Technical Tasks:**
  - Add pandas dependency: `pandas==2.1.3`
  - Create `backend/src/services/data_processing.py` with CSV parsing logic
  - Implement column type detection (string, number, date)
  - Create data preview endpoint that returns first 100 rows
  - Handle various CSV formats and encodings
  - Add error handling for malformed files
  - Return parsed data with column metadata
  - Create frontend component to display data preview table
- **Commit Message:** `feat: implement CSV parsing with data preview`

### Step 20: Canvas Model and Basic Structure

- **Goal:** Create the canvas model for storing analysis workspaces.
- **Technical Tasks:**
  - Create `backend/src/models/canvas.py` with Canvas model (id, name, project_id, content_json, created_by, created_at, updated_at)
  - Add relationship to Project model
  - Generate migration for canvas table
  - Create `backend/src/schemas/canvas.py` with canvas schemas
  - Create basic CRUD endpoints in `backend/src/api/v1/endpoints/canvases.py`
  - Implement canvas listing by project
  - Add authorization checks for canvas access
- **Commit Message:** `feat: add Canvas model with basic CRUD operations`

### Step 21: Canvas Editor UI Foundation

- **Goal:** Build the basic canvas editor interface structure.
- **Technical Tasks:**
  - Create `frontend/src/pages/CanvasEditor.tsx` component
  - Set up three-panel layout (sidebar, canvas, properties)
  - Create `frontend/src/components/Canvas/` directory
  - Implement `CanvasArea.tsx` with grid background
  - Add `BlockLibrary.tsx` sidebar with draggable blocks
  - Create basic block types (text, chart placeholder)
  - Style canvas with proper spacing and borders
  - Add canvas name display and edit functionality
- **Commit Message:** `feat: create canvas editor UI with basic layout`

### Step 22: Drag and Drop Implementation

- **Goal:** Implement drag-and-drop functionality for canvas blocks.
- **Technical Tasks:**
  - Install DnD library: `npm install @dnd-kit/sortable @dnd-kit/core`
  - Create `frontend/src/components/Canvas/DraggableBlock.tsx`
  - Implement drag handlers for block library items
  - Add drop zones on the canvas
  - Create block positioning logic
  - Implement visual feedback during drag
  - Store block positions in component state
  - Add snap-to-grid functionality
- **Commit Message:** `feat: implement drag-and-drop for canvas blocks`

### Step 23: Text Block Implementation

- **Goal:** Create a functional text block with editing capabilities.
- **Technical Tasks:**
  - Create `frontend/src/components/Blocks/TextBlock.tsx`
  - Implement inline text editing on double-click
  - Add rich text editor with basic formatting
  - Create block selection and focus states
  - Implement block deletion on keyboard delete
  - Add text content to canvas state model
  - Style text block with proper typography
  - Save text content on blur
- **Commit Message:** `feat: add editable text block with rich text support`

### Step 24: Canvas State Persistence

- **Goal:** Save and load canvas state from the backend.
- **Technical Tasks:**
  - Create canvas update endpoint in backend
  - Implement canvas state serialization format
  - Add auto-save functionality with debouncing
  - Create `frontend/src/services/canvas.ts` with API calls
  - Implement save indicator UI
  - Add error handling for failed saves
  - Load canvas state on editor mount
  - Show last saved timestamp
- **Commit Message:** `feat: implement canvas state persistence with auto-save`

### Step 25: Basic AI Service Integration

- **Goal:** Set up OpenAI integration for natural language processing.
- **Technical Tasks:**
  - Add OpenAI dependency: `openai==1.3.5`
  - Create `backend/src/services/ai/` directory
  - Create `backend/src/services/ai/openai_client.py` with client setup
  - Add OPENAI_API_KEY to environment variables
  - Create basic prompt templates for chart generation
  - Implement token counting and cost estimation
  - Add request caching to reduce API calls
  - Create health check for AI service availability
- **Commit Message:** `feat: integrate OpenAI API for AI-powered features`

### Step 26: Natural Language Chart Request UI

- **Goal:** Build UI for users to request charts using natural language.
- **Technical Tasks:**
  - Create `frontend/src/components/AI/ChartPrompt.tsx` component
  - Add floating action button to open AI prompt
  - Build modal with text input for chart request
  - Add example prompts as suggestions
  - Show available data context (uploaded files)
  - Implement loading state during AI processing
  - Add error handling for failed requests
  - Style modal with smooth animations
- **Commit Message:** `feat: add natural language chart request interface`

### Step 27: AI Chart Generation Endpoint

- **Goal:** Create backend endpoint for AI-powered chart generation.
- **Technical Tasks:**
  - Create `backend/src/api/v1/endpoints/ai.py` with chart generation endpoint
  - Parse natural language request with OpenAI
  - Extract chart type, data requirements, and configuration
  - Validate requested data against available datasets
  - Generate chart specification JSON
  - Return structured response for frontend rendering
  - Add usage tracking for AI requests
  - Implement rate limiting for AI endpoints
- **Commit Message:** `feat: implement AI chart generation endpoint`

### Step 28: Chart Rendering Library Setup

- **Goal:** Integrate a chart rendering library for data visualization.
- **Technical Tasks:**
  - Install chart library: `npm install recharts`
  - Create `frontend/src/components/Charts/` directory
  - Create base `ChartContainer.tsx` component
  - Implement `BarChart.tsx` component
  - Implement `LineChart.tsx` component
  - Implement `PieChart.tsx` component
  - Add responsive sizing and tooltips
  - Configure default color schemes
- **Commit Message:** `feat: add Recharts library with basic chart components`

### Step 29: Dynamic Chart Block Implementation

- **Goal:** Create chart blocks that render based on AI-generated specifications.
- **Technical Tasks:**
  - Create `frontend/src/components/Blocks/ChartBlock.tsx`
  - Parse chart specification from AI response
  - Dynamically render appropriate chart type
  - Connect chart to project data sources
  - Add chart title and axis labels
  - Implement chart refresh on data change
  - Add loading state while data processes
  - Include error boundary for render failures
- **Commit Message:** `feat: implement dynamic chart block with AI specs`

### Step 30: Data Transformation Service

- **Goal:** Build service to transform raw data for chart consumption.
- **Technical Tasks:**
  - Create `backend/src/services/data_transformation.py`
  - Implement data aggregation functions (sum, average, count)
  - Add data filtering based on conditions
  - Create pivot/unpivot transformations
  - Handle date parsing and formatting
  - Implement grouping operations
  - Add data sorting capabilities
  - Create unit tests for transformations
- **Commit Message:** `feat: add data transformation service for charts`

### Step 31: Share Canvas Functionality

- **Goal:** Implement basic canvas sharing via public links.
- **Technical Tasks:**
  - Add share_token field to Canvas model
  - Create share token generation endpoint
  - Implement public canvas view endpoint
  - Create `frontend/src/pages/SharedCanvas.tsx` component
  - Add share button to canvas editor
  - Generate and display shareable link
  - Implement read-only canvas viewer
  - Add link copy-to-clipboard functionality
- **Commit Message:** `feat: implement canvas sharing with public links`

### Step 32: Basic Comment System

- **Goal:** Add commenting functionality to canvas blocks.
- **Technical Tasks:**
  - Create `backend/src/models/comment.py` with Comment model
  - Generate migration for comments table
  - Create comment CRUD endpoints
  - Add comment thread component in frontend
  - Implement comment creation UI
  - Display comments with author and timestamp
  - Add comment deletion for authors
  - Style comment threads with indentation
- **Commit Message:** `feat: add basic commenting system for canvas blocks`

### Step 33: User Profile Management

- **Goal:** Allow users to update their profile information.
- **Technical Tasks:**
  - Create profile update endpoint in backend
  - Create `frontend/src/pages/Profile.tsx` component
  - Add form for updating first name, last name
  - Implement email update with verification
  - Add password change functionality
  - Create profile picture upload
  - Show account creation date
  - Add profile update success notifications
- **Commit Message:** `feat: implement user profile management`

### Step 34: Sample Data and Templates

- **Goal:** Provide sample data and templates for new users.
- **Technical Tasks:**
  - Create `backend/src/data/samples/` directory
  - Add sample CSV files (sales, expenses, demographics)
  - Create sample data seeding script
  - Add template canvases for common use cases
  - Create onboarding flow with sample selection
  - Implement template duplication functionality
  - Add template preview images
  - Include template descriptions
- **Commit Message:** `feat: add sample data and canvas templates`

### Step 35: Error Handling and Notifications

- **Goal:** Implement comprehensive error handling and user notifications.
- **Technical Tasks:**
  - Create `frontend/src/components/Notifications/` directory
  - Implement toast notification system
  - Add global error boundary component
  - Create error logging service
  - Implement retry logic for failed requests
  - Add offline detection and messaging
  - Style notifications with animations
  - Include notification persistence options
- **Commit Message:** `feat: implement error handling and notification system`

### Step 36: Performance Monitoring Setup

- **Goal:** Add basic performance monitoring and metrics.
- **Technical Tasks:**
  - Add APM dependencies: `prometheus-client==0.19.0`
  - Create metrics endpoint in backend
  - Track API response times
  - Monitor database query performance
  - Add frontend performance tracking
  - Create simple metrics dashboard
  - Set up error rate monitoring
  - Configure performance thresholds
- **Commit Message:** `feat: add performance monitoring and metrics`

### Step 37: Basic Search Functionality

- **Goal:** Implement search across projects and canvases.
- **Technical Tasks:**
  - Create search endpoint in backend
  - Implement full-text search for projects
  - Add canvas content searching
  - Create `frontend/src/components/Search/` directory
  - Build search bar component
  - Display search results with highlighting
  - Add search filters (type, date, owner)
  - Implement keyboard navigation
- **Commit Message:** `feat: implement basic search functionality`

### Step 38: Mobile Responsive Design

- **Goal:** Ensure the application works well on mobile devices.
- **Technical Tasks:**
  - Add viewport meta tag configuration
  - Create responsive navigation menu
  - Implement mobile-friendly canvas viewer
  - Adjust forms for mobile screens
  - Create touch-friendly interactions
  - Test on various device sizes
  - Fix overflow and scrolling issues
  - Optimize button sizes for touch
- **Commit Message:** `feat: implement mobile responsive design`

### Step 39: Docker Configuration

- **Goal:** Create Docker configuration for easy deployment.
- **Technical Tasks:**
  - Create `Dockerfile` for frontend application
  - Create `Dockerfile` for backend application
  - Create `docker-compose.yml` for full stack
  - Add environment variable configuration
  - Configure nginx for frontend serving
  - Set up container networking
  - Add health checks to services
  - Create build and run scripts
- **Commit Message:** `feat: add Docker configuration for deployment`

### Step 40: Production Build and Optimization

- **Goal:** Optimize the application for production deployment.
- **Technical Tasks:**
  - Configure production build settings
  - Enable frontend code splitting
  - Implement lazy loading for routes
  - Minify and compress assets
  - Configure CDN for static assets
  - Set up CORS properly
  - Add security headers
  - Create production environment files
- **Commit Message:** `feat: configure production build and optimizations`

### Step 41: Basic CI/CD Pipeline

- **Goal:** Set up continuous integration and deployment pipeline.
- **Technical Tasks:**
  - Create `.github/workflows/` directory
  - Add `ci.yml` for continuous integration
  - Configure frontend build and test jobs
  - Configure backend test job
  - Add linting and type checking
  - Set up database for integration tests
  - Configure deployment workflow
  - Add build status badges to README
- **Commit Message:** `feat: implement CI/CD pipeline with GitHub Actions`

### Step 42: Documentation and API Docs

- **Goal:** Create comprehensive documentation for the project.
- **Technical Tasks:**
  - Add API documentation with OpenAPI/Swagger
  - Create developer setup guide
  - Document environment variables
  - Add architecture diagrams
  - Create user guide with screenshots
  - Document deployment process
  - Add contribution guidelines
  - Generate API client documentation
- **Commit Message:** `docs: add comprehensive project documentation`

### Step 43: Security Hardening

- **Goal:** Implement security best practices and hardening.
- **Technical Tasks:**
  - Add rate limiting to all endpoints
  - Implement CSRF protection
  - Configure secure session management
  - Add input sanitization
  - Implement SQL injection prevention
  - Configure secure headers
  - Add API request validation
  - Create security audit checklist
- **Commit Message:** `feat: implement security hardening measures`

### Step 44: Onboarding Flow Enhancement

- **Goal:** Create the 60-second onboarding experience.
- **Technical Tasks:**
  - Create `frontend/src/components/Onboarding/` directory
  - Build step-by-step onboarding wizard
  - Add progress indicators
  - Implement quick data upload option
  - Create first chart generation flow
  - Add celebration on first success
  - Track onboarding completion metrics
  - Allow skipping for experienced users
- **Commit Message:** `feat: implement 60-second onboarding flow`

### Step 45: Final Testing and Bug Fixes

- **Goal:** Comprehensive testing and bug fixing before launch.
- **Technical Tasks:**
  - Run full end-to-end test suite
  - Perform load testing with multiple users
  - Test all critical user journeys
  - Fix identified bugs and issues
  - Verify mobile functionality
  - Test sharing and collaboration features
  - Validate AI chart generation accuracy
  - Ensure proper error handling throughout
- **Commit Message:** `fix: final bug fixes and testing improvements`

### Step 46: Production Deployment

- **Goal:** Deploy the MVP to production environment.
- **Technical Tasks:**
  - Set up production database
  - Configure production environment variables
  - Deploy backend services
  - Deploy frontend application
  - Configure domain and SSL certificates
  - Set up monitoring and alerting
  - Verify all features in production
  - Create initial admin user
- **Commit Message:** `feat: deploy MVP to production environment`

### Step 47: Launch Preparation

- **Goal:** Prepare for public launch of the MVP.
- **Technical Tasks:**
  - Create landing page with feature highlights
  - Set up analytics tracking
  - Configure error tracking service
  - Prepare support documentation
  - Create feedback collection mechanism
  - Set up user communication channels
  - Configure backup procedures
  - Plan launch announcement
- **Commit Message:** `feat: complete launch preparation for MVP`

---

## Summary

This implementation plan breaks down the Impala MVP into 47 atomic steps, each resulting in a functional, testable state of the application. The plan ensures:

1. **Constant Testability**: After each step, the application remains runnable and testable
2. **Atomic Commits**: Each step corresponds to a single, focused git commit
3. **Progressive Enhancement**: Features build upon each other logically
4. **MVP Completeness**: All Phase 1 requirements from the PRD are covered
5. **60-Second Onboarding**: The final product achieves the goal of first insight within 60 seconds

The plan can be executed sequentially by a developer or AI coding agent, with clear technical tasks and expected outcomes for each step.