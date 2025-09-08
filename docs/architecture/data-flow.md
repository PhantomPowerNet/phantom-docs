# Data Flow Architecture

## Overview

Phantom Power employs a microservices architecture with distinct services handling different aspects of the application. Data flows through multiple layers with clear separation of concerns.

## Service Architecture

### Core Services

```
    NextJS["Next.js Frontend"] -->|API Calls| SpringBoot["Spring Boot API Server"]
    SpringBoot -->|Invokes| Python["Python Audio Service"]
    NextJS -- WebSocket --> WS["WebSocket (Real-time Communication)"]

    SpringBoot -->|DB Access| PG["PostgreSQL Database"]
    Python -->|Stores Files| S3["S3/CDN File Storage"]
```

## Data Flow Patterns

### 1. User Authentication Flow

```
    FE["Frontend"] --> SB["Spring Boot API"]
    SB --> DB["Database"]
    SB -->|JWT Generation| JWT
    JWT --> FEJWT["JWT Storage"]
    FEJWT -->|Authenticated Request| SB
    SB -->|Creates| UR["User Record"]
```

**Steps:**
1. User submits credentials via Next.js form
2. Spring Boot validates against PostgreSQL user table
3. JWT token generated and returned
4. Frontend stores JWT for subsequent requests
5. All API calls include JWT for authorization

### 2. Audio Analysis Flow

```
    FE["Frontend"] --> SB["Spring Boot"]
    SB --> PY["Python Service"]
    PY --> DB["Database"]

    SB -->|Audio Metadata Stored| DB
    PY -->|AI Analysis Generated| DB
    FE -->|User sees results| UIRes["Analysis Results Displayed"]

    DB -->|Metrics Stored| Metrics["Metrics Stored"]
    DB -->|AudioFileReport Created| AFR["AudioFileReport"]
```

**Steps:**
1. User uploads audio file through Next.js interface
2. Spring Boot stores metadata and file reference
3. Python service analyzes audio using librosa/madmom
4. Analysis results stored in AudioFileReport
5. Frontend displays waveform and analysis data

### 3. Real-time Messaging Flow

```
    UA["User A"] -- Connects --> WS1["WebSocket"]
    WS1 --> SB["Spring Boot"]
    SB --> DB["Database"]
    SB --> WS2["WebSocket"]
    WS2 --> UB["User B"]
    SB -->|Creates| MSG["ChatMessage"]

    UA -- Sends ->|Message| WS1
    SB -- Broadcasts -> WS2
    UB -- Receives ->|Message Displayed| WS2
```

**Steps:**
1. Users establish WebSocket connections
2. Message sent through WebSocket to Spring Boot
3. Message validated and stored in database
4. Message broadcast to all room participants
5. Recipients receive real-time updates

### 4. Matching Algorithm Flow

```
    U["User Profile"] --> SB["Spring Boot"]
    SB --> PY["Python Service"]
    PY --> DB["Database"]

    U -->|Audio Files| PY
    PY -->|Analysis Data| DB
    DB -->|Match Scoring| MR["Match Records"]
    MR -->|Recommendations| Rec["Recommendations"]
    Rec -->|Notifications| N["Notifications"]

    U -- Tag Data --> PT["Profile Tag"]
    PY -- Combined Profile --> CP["Combined Profile"]
```

**Steps:**
1. User profiles and audio analyzed continuously
2. Python service calculates compatibility scores
3. Match records created for high-scoring pairs
4. Users receive match notifications
5. Recommendations displayed in discovery feed

## Database Relationships

### Core Entity Relationships

```
    User -- has --> Profile
    User -- uploads --> AudioFile
    AudioFile -- has --> AudioFileReport
    AudioFileReport -- has --> AIAudioFileAnalysisResult
    User -- sends --> Message
    User -- participates --> Project
    User -- can book --> Booking
    User -- has --> Match
    AudioFile -- tagged --> Tag
    Profile -- has --> Tag
    User -- follows --> UserFollow
    ChatRoom -- contains --> ChatMessage
    ChatMessage -- sent by --> User
```

### Data Consistency Patterns

- **ACID Transactions**: Critical operations use database transactions
- **Event Sourcing**: Audio analysis results stored as immutable events
- **Eventual Consistency**: Real-time features may have slight delays
- **Optimistic Locking**: Concurrent updates handled gracefully

## API Communication Patterns

### REST Endpoints

- **Authentication**: `/api/auth/*` - User registration, login, profile
- **Users**: `/api/users/*` - Profile management, discovery
- **Projects**: `/api/projects/*` - Collaboration management
- **Audio**: `/api/audio/*` - File upload, analysis results
- **Messages**: `/api/messages/*` - Chat history, room management

### WebSocket Events

- **Connection**: User joins/leaves rooms
- **Messages**: Real-time chat communication
- **Notifications**: Match alerts, booking updates
- **Presence**: Online status updates

### Microservice Integration

- **Synchronous**: REST calls for immediate responses
- **Asynchronous**: Message queues for long-running analysis <!-- TODO: Implement message queue -->
- **Circuit Breaker**: Fallback when services unavailable <!-- TODO: Add resilience patterns -->

## Security Considerations

### Data Protection

- **Encryption**: All data encrypted in transit (HTTPS/WSS)
- **Authorization**: JWT-based access control
- **Data Validation**: Input sanitization at all layers
- **Rate Limiting**: API abuse prevention <!-- TODO: Implement rate limiting -->

### Privacy

- **User Consent**: Explicit permission for audio analysis
- **Data Retention**: Configurable deletion policies <!-- TODO: Define retention policies -->
- **GDPR Compliance**: User data export/deletion capabilities <!-- TODO: Implement GDPR features -->

## Performance Optimizations

### Caching Strategy

- **Redis**: Session data and frequently accessed content <!-- TODO: Implement Redis caching -->
- **CDN**: Static assets and audio files
- **Database**: Query result caching for expensive operations

### Scaling Considerations

- **Horizontal Scaling**: Stateless services can be replicated
- **Database Sharding**: User data partitioned by region <!-- TODO: Plan sharding strategy -->
- **Load Balancing**: Traffic distributed across service instances

<!-- TODO: Add monitoring and observability patterns -->
<!-- TODO: Document backup and disaster recovery -->
<!-- TODO: Detail error handling and logging flows -->
