# Memry.ai System Architecture

## 1. Overview

This document outlines the technical blueprint for Memry.ai, a mobile application that provides automated audio recording, transcription, and AI-powered summarization. The architecture is designed to be scalable, secure, and cost-effective, leveraging modern cloud-native technologies.

## 2. System Components & Technology Stack

### 2.1. Mobile Client (Frontend)
*   **Technology:** React Native
*   **Justification:** Enables cross-platform development for iOS and Android from a single codebase, accelerating time-to-market and reducing maintenance overhead.
*   **Key Responsibilities:**
    *   User Authentication UI
    *   Audio Recording Interface (start/stop)
    *   Displaying real-time transcription stream
    *   Rendering summaries and historical notes
    *   Managing local audio file uploads to cloud storage

### 2.2. Backend API Server
*   **Technology:** Python with Flask Framework
*   **Justification:** Python has best-in-class libraries for AI/ML and data processing. Flask is a lightweight, flexible framework ideal for building RESTful APIs quickly.
*   **Key Responsibilities:**
    *   User authentication and authorization (JWT tokens)
    *   Handling API requests from the mobile client
    *   Orchestrating the transcription and summarization pipeline
    *   Communicating with the database and cloud services

### 2.3. Database
*   **Technology:** PostgreSQL
*   **Justification:** A robust, open-source relational database. It is ideal for storing structured data like user profiles, session metadata, and generated summaries. It offers ACID compliance, which is critical for data integrity.
*   **Stored Data:**
    *   User accounts and credentials (hashed passwords)
    *   Recording sessions (metadata: timestamp, duration, user_id)
    *   AI-generated data (transcripts, summaries, action items)

### 2.4. Cloud Storage
*   **Technology:** AWS S3 (Simple Storage Service)
*   **Justification:** Highly scalable, durable, and secure object storage. Perfect for storing large, binary files like audio recordings.
*   **Usage:** Raw audio files from the mobile client are uploaded directly to a secure S3 bucket.

### 2.5. AI & Processing Services
*   **Transcription:** OpenAI Whisper API
*   **Summarization:** OpenAI GPT-4 API
*   **Justification:** Using established APIs for core AI tasks drastically reduces initial development complexity and cost. It allows us to leverage state-of-the-art models without the overhead of training and hosting them ourselves. We can reassess this cost-benefit analysis as scale increases.

## 3. Data Flow & Component Communication

The following sequence details the journey of a user's audio recording:

1.  **Record:** User initiates recording in the React Native app. The app captures audio and stores it temporarily locally.
2.  **Upload:** Upon stopping, the app securely uploads the audio file directly to an **AWS S3** bucket, alongside relevant metadata (user_id, session_id).
3.  **Trigger Processing:** The backend **Flask API** is notified of the new S3 upload (via an S3 event notification).
4.  **Transcribe:** The Flask server retrieves the audio file from S3 and sends it to the **OpenAI Whisper API** for transcription.
5.  **Summarize:** The Flask server takes the returned transcript and sends it to the **OpenAI GPT-4 API** with a carefully engineered prompt to generate a summary, list key topics, and extract action items.
6.  **Store Results:** The Flask server saves the transcript, summary, and other AI-generated data to the **PostgreSQL** database, linked to the original session record.
7.  **Client Update:** The mobile client can now fetch and display the processed results for the user via the Flask API.

```mermaid
graph TD
    A[React Native App<br>Record Audio] --> B[Upload to AWS S3];
    B --> C[S3 Event Trigger];
    C --> D[Flask API Server];
    D --> E[Call OpenAI Whisper API];
    E --> F[Receive Transcript];
    F --> G[Call OpenAI GPT-4 API];
    G --> H[Receive Summary & Insights];
    H --> I[Save to PostgreSQL];
    I --> J[App Fetches & Displays Data];
