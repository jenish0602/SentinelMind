# SentinelMind

**Intelligent Cyber Security Incident Detection and Threat Management System**

SentinelMind is a Java 21 Maven WAR web application for handling security incidents, tracking threats, logging user activity, and managing role-based workflows. It is an academic/portfolio project and is **not a production security monitoring platform**.

## Tech stack
- Java 21, Jakarta Servlets and JSP, JDBC, Hibernate, Spring, Maven
- Tomcat 10.1, MySQL-compatible database
- BCrypt for password hashing, session-based authentication
- ADMIN, ANALYST, USER role-based authorization

## Features
- Registration through role-specific access codes
- Login and logout with HTTP session management
- Dashboard, incidents, threat records, assignments, evidence, tasks and notifications
- Monitoring activity, heartbeat records, reports and audit history
- Queued scan requests; the application **does not actually execute network scans**

## Getting started
1. Install JDK 21, Maven, Tomcat 10.1, and MySQL.
2. Import the reviewed SQL schema into a local `sentinelmind_db` database. Do not import real user/credential data into public repositories.
3. Set `SM_DB_URL`, `SM_DB_USER`, and `SM_DB_PASSWORD` for the Java JDBC connection. Hibernate's XML connection configuration must also be reviewed and adjusted for your environment.
4. From the project root, run `mvn clean package`.
5. Deploy `target/SentinelMind.war` to Tomcat; open `http://localhost:8081/SentinelMind/` if your Tomcat uses port 8081.

The source archive contains setup SQL and optional migrations; read their instructions before deciding which SQL to run.

## Authentication
See [Authentication architecture](docs/AUTHENTICATION.md) for components, request flow, password storage, registration-code handling, and sessions.

## Repository hygiene
Exclude `target/`, IDE metadata, uploaded user files, environment/secrets files, and database dumps containing real data. Do not publish usable administrative registration codes or live credentials.

## Current repository status
Initial documentation is published here. Application source files are to be added from the sanitized project bundle.
