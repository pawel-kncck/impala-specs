# Impala

## Description

Impala is a web-based application designed to be like "Google Docs for Data". The main goal of Impala is to allow users, especially those without a technical background, to easily work with their data. The application provides an interactive platform for uploading data (starting with CSV files), analyzing it, and creating visualizations. The overall aim is to make data analysis accessible and user-friendly by removing the need for complex software and technical skills.

## Key Features

- **User Authentication**: Secure user registration and login functionality.
- **Project Management**: Users can create, view, and manage their own data projects.
- **CSV Data Upload**: Support for uploading CSV files within a project to be used as data sources.
- **Interactive Data Analysis**: An interactive canvas for data analysis and visualization.
- **AI-Powered Assistance**:
  - AI-powered chat and chart suggestions to guide users through the analysis process.
  - The ability to translate natural language questions into data queries.
- **NoSQL and SQL Databases**:
  - MongoDB is used for flexible, JSON-based canvas structures.
  - PostgreSQL for structured data like user and project information.

## Technology Stack

- **Frontend**: React
- **Backend**: Python with FastAPI
- **Databases**:
  - PostgreSQL
  - MongoDB
- **Containerization**: Docker and Docker Compose

## High-Level Setup Instructions

1.  **Prerequisites**: Ensure you have Docker and Docker Compose installed on your system.
2.  **Environment Variables**: Create a `.env` file for environment variables such as `POSTGRES_USER` and `POSTGRES_PASSWORD`.
3.  **Build and Run**: Use Docker Compose to build and run the application services.

## How to Run

To get the application running locally, execute the following command in your terminal at the root of the project:

```bash
docker-compose up --build
```

This will start the following services:

- **Frontend**: Accessible on port 5173 for development or port 3000 for production.
- **Backend**: The FastAPI backend will be running on port 5001.
- **Databases**: PostgreSQL and MongoDB will also be started as services.
