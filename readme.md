# V1 System Architecture Design Document

**Author:** @YOKESH T
**Phase / Milestone:** Day 3 — System Architecture Design
**Project:** Competitive Coding Platform (Java V1 MVP)

---

## 1. Executive Summary
This document defines the V1 system architecture for our competitive coding platform. The system is designed to handle user registration, problem viewing, contest participation, and safe execution of Java submissions using containerized isolation.

---

## 2. Selected Technology Stack

* **Frontend:** **React.js + TypeScript**
  * *Role:* Student and Admin web portals. Handles code editor, problem viewing, submission status updates, and contest rankings.
* **Backend API:** **Node.js (Express) / Java (Spring Boot)**
  * *Role:* Core RESTful service for user authentication, CRUD operations on contests/problems, and dispatching submission jobs.
* **Primary Database:** **PostgreSQL**
  * *Role:* Relational data storage for users, contest configurations, problem metadata, and submission historical records with ACID compliance.
* **Queue System:** **Redis + BullMQ**
  * *Role:* Asynchronous job queue for code execution requests to decouple the API backend from judging workers during high load.
* **Judge Worker:** **Node.js / Go Service**
  * *Role:* Consumes queue jobs, retrieves hidden test case files, manages Docker container creation, evaluates execution verdicts (`AC`, `WA`, `TLE`, `CE`), and writes results back to PostgreSQL.
* **Sandbox Runtime:** **Docker Engine API**
  * *Role:* Ephemeral, restricted containers enforcing strictly allocated limits (e.g., 2 CPU cores, 256MB RAM, disabled network access) to safely run compiled Java binaries.
* **Test Case Storage:** **AWS S3 / MinIO Object Storage**
  * *Role:* Efficiently stores large `.in` and `.out` input/output test case files outside PostgreSQL.

---

## 3. End-to-End Submission Execution Flow

1. **Submission Request:** A student submits a Java solution from the React frontend via a REST API endpoint to the Backend API.
2. **Persistence & Queueing:** The Backend API logs the submission as `Pending` in PostgreSQL and pushes a job containing `submission_id` and `problem_id` to the Redis Queue.
3. **Queue Processing:** A Judge Worker pops the job from the Redis Queue.
4. **Fetching Assets:** The Judge Worker downloads the corresponding test case input and output files from Object Storage (MinIO/S3).
5. **Isolated Execution:** The Judge Worker writes the user's code to a temporary directory, invokes a Docker container with strict memory/CPU limits, compiles `Main.java`, and executes it against the input test cases.
6. **Verdict & Storage:** The Judge Worker compares container standard output against expected output, determines the final verdict (`Accepted`, `Wrong Answer`, `Time Limit Exceeded`, `Compile Error`), and updates the PostgreSQL database record.
7. **Result Notification:** The Frontend receives the updated verdict via REST polling or WebSockets.
