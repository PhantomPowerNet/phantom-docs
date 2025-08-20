# System Overview

## Project Vision

Phantom Power is an innovative platform that connects music collaborators through intelligent audio analysis and matching. The application enables musicians to discover compatible partners based not just on profiles and preferences, but on actual musical characteristics extracted from audio samples.

## Core Features

### 1. User Profiles & Authentication
- **JWT-based authentication** with Spring Security
- **Rich user profiles** with bio, links, featured audio, tags, and customizable themes
- **Audio-enhanced profiles** displaying analysis results from uploaded samples

### 2. Audio Analysis & Matching
- **AI-powered audio analysis** using Python microservices (librosa/madmom)
- **Automated extraction** of tempo, key, genre, and advanced metrics like "Dynamics Quotient"
- **User-contributed analysis** for subjective elements (mood, instruments, tags)
- **Intelligent matching** algorithm combining profile data with audio characteristics

### 3. Social Features
- **Real-time messaging** via WebSocket connections
- **Group chat rooms** for community building
- **Follow system** to track favorite collaborators
- **Comments and likes** on audio content and profiles

### 4. Project Management
- **Collaborative projects** with role-based participation
- **Booking system** for scheduling sessions and commitments
- **Project status tracking** (requested, confirmed, completed)

### 5. Content Management
- **Audio file uploads** with metadata storage
- **Waveform visualization** and embedded audio players
- **Tag-based categorization** for easy discovery
- **Playlist creation** and curation

## Architecture Philosophy

The system follows a **microservices architecture** with clear separation of concerns:

- **Frontend**: Next.js with TypeScript for robust, type-safe UI development
- **Backend API**: Spring Boot for enterprise-grade REST services and security
- **Audio Processing**: Python FastAPI microservice for specialized audio analysis
- **Database**: PostgreSQL for reliable, ACID-compliant data persistence
- **Real-time**: WebSocket integration for live chat and notifications

<!-- TODO: Add system architecture diagram -->
<!-- TODO: Detail data flow between services -->
<!-- TODO: Document security model -->
<!-- TODO: Explain scalability considerations -->