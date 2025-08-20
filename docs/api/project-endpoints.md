# Project & Collaboration API

## Overview

The Project API manages collaborative creative endeavors, gig postings, booking systems, and match-based recommendations. This comprehensive API ensures secure project management with role-based access control and real-time collaboration features.

## Base URL

```
Development: http://localhost:8080/api/projects
Production: https://api.phantompower.app/api/projects
```

## Core Data Models

### Project
Collaborative creative endeavors or gig postings with:
- Title, description, and requirements
- Owner and participant management
- Status tracking (open, in-progress, completed)
- Location and time requirements
- Budget and compensation details
- Required skills and instruments
- Application and approval workflow

### Booking
Scheduling agreements and confirmed collaborations:
- Time slots and duration
- Location (in-person/remote)
- Role-specific requirements
- Status management (requested, confirmed, cancelled, completed)
- Payment and compensation tracking
- Calendar integration

### Match
AI-based user matching results:
- Compatibility scores based on audio analysis
- Profile similarity metrics
- Mutual interests and tags
- Geographic proximity
- Availability alignment
- Recommendation reasoning

## Project Management

### Create Project

**Endpoint:** `POST /projects`

**Request Body:**
```json
{
  "title": "Ambient Album Collaboration",
  "description": "Looking for modular synth & field recording artists for a collaborative ambient album. Remote collaboration welcome.",
  "type": "album", // "album", "single", "gig", "session", "competition"
  "genre": ["ambient", "experimental", "electronic"],
  "requiredSkills": ["modular-synth", "field-recording", "audio-editing"],
  "instruments": ["synthesizer", "sampler", "effects-processing"],
  "location": {
    "type": "remote", // "remote", "in-person", "hybrid"
    "city": "Brooklyn",
    "state": "NY",
    "country": "US",
    "timezone": "America/New_York"
  },
  "timeline": {
    "startDate": "2025-08-01T00:00:00Z",
    "endDate": "2025-12-01T00:00:00Z",
    "estimatedHours": 40
  },
  "compensation": {
    "type": "revenue-split", // "paid", "revenue-split", "credit-only", "negotiable"
    "amount": null,
    "currency": "USD",
    "details": "Equal revenue split among all contributors"
  },
  "requirements": {
    "experienceLevel": "intermediate", // "beginner", "intermediate", "advanced", "professional"
    "minimumAge": 18,
    "equipmentRequired": ["DAW", "Audio Interface", "Studio Monitors"],
    "portfolioRequired": true,
    "references": false
  },
  "applicationProcess": {
    "requiresApplication": true,
    "autoAccept": false,
    "maxParticipants": 5,
    "applicationDeadline": "2025-07-25T23:59:59Z"
  },
  "visibility": "public", // "public", "private", "invite-only"
  "isOpen": true
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "project": {
      "id": "01HYNPRJ01ABCDE1234567890",
      "title": "Ambient Album Collaboration",
      "slug": "ambient-album-collaboration-01hynprj01",
      "ownerId": "01HYN8XYZ9QRS1234567890",
      "status": "open",
      "participantCount": 1,
      "applicationCount": 0,
      "createdAt": "2025-07-19T16:20:00Z",
      "updatedAt": "2025-07-19T16:20:00Z"
    }
  }
}
```

### Get Project Details

**Endpoint:** `GET /projects/{projectId}`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "project": {
      "id": "01HYNPRJ01ABCDE1234567890",
      "title": "Ambient Album Collaboration",
      "description": "Looking for modular synth & field recording artists...",
      "owner": {
        "id": "01HYN8XYZ9QRS1234567890",
        "profile": {
          "displayName": "Alex Chen",
          "avatarUrl": "https://cdn.phantompower.app/avatars/alexchen.jpg",
          "location": "Brooklyn, NY",
          "verificationStatus": "verified"
        },
        "stats": {
          "completedProjects": 12,
          "rating": 4.8,
          "responseRate": 0.95
        }
      },
      "participants": [
        {
          "userId": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
          "role": "producer",
          "status": "confirmed",
          "joinedAt": "2025-07-19T17:30:00Z",
          "profile": {
            "displayName": "Jane D",
            "avatarUrl": "https://cdn.phantompower.app/avatars/janedoe.jpg"
          }
        }
      ],
      "stats": {
        "views": 247,
        "applications": 8,
        "bookmarks": 15,
        "shares": 3
      },
      "matchScore": 0.87, // For authenticated user
      "hasApplied": false,
      "isBookmarked": false,
      "canApply": true
    }
  }
}
```

### Search Projects

**Endpoint:** `GET /projects/search`

**Query Parameters:**
- `q` (string) - Search query
- `genre` (string[]) - Genre filters
- `location` (string) - Location filter
- `type` (string) - Project type
- `compensation` (string) - Compensation type
- `experienceLevel` (string) - Required experience level
- `remote` (boolean) - Remote projects only
- `sort` (string) - "recent", "popular", "match", "deadline"
- `page`, `limit` - Pagination

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "projects": [
      {
        "id": "01HYNPRJ01ABCDE1234567890",
        "title": "Ambient Album Collaboration",
        "summary": "Remote collaboration for ambient album with modular synth focus",
        "genre": ["ambient", "experimental"],
        "compensation": {
          "type": "revenue-split",
          "summary": "Equal revenue split"
        },
        "location": {
          "type": "remote",
          "city": "Brooklyn, NY"
        },
        "timeline": {
          "startDate": "2025-08-01T00:00:00Z",
          "duration": "4 months"
        },
        "owner": {
          "displayName": "Alex Chen",
          "avatarUrl": "https://cdn.phantompower.app/avatars/alexchen.jpg",
          "rating": 4.8
        },
        "stats": {
          "applications": 8,
          "spots": 5,
          "spotsRemaining": 3
        },
        "matchScore": 0.87,
        "tags": ["remote", "revenue-split", "album", "experimental"],
        "applicationDeadline": "2025-07-25T23:59:59Z",
        "createdAt": "2025-07-19T16:20:00Z"
      }
    ],
    "filters": {
      "appliedFilters": {
        "genre": ["ambient"],
        "remote": true
      },
      "availableFilters": {
        "genres": ["ambient", "rock", "jazz", "electronic"],
        "locations": ["New York", "Los Angeles", "Nashville"],
        "types": ["album", "single", "gig", "session"]
      }
    },
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "totalPages": 8
    }
  }
}
```

### Apply to Project

**Endpoint:** `POST /projects/{projectId}/apply`

**Request Body:**
```json
{
  "message": "Hi Alex! I'm a field recording artist with 5+ years experience in ambient music. I'd love to contribute to this project with my collection of nature recordings and modular processing techniques.",
  "role": "sound-designer",
  "portfolioItems": [
    "01HYN91QZAYVNNRH53SJ1KMBCY", // AudioFile IDs
    "01HYN92QZAYVNNRH53SJ1KMBCZ"
  ],
  "availability": {
    "hoursPerWeek": 8,
    "flexibleSchedule": true,
    "timezone": "America/New_York",
    "preferredDays": ["monday", "tuesday", "wednesday", "thursday"]
  },
  "equipment": [
    "Field Recorder (Zoom H6)",
    "Modular Synth Setup",
    "Pro Tools Studio"
  ],
  "references": [
    {
      "name": "Sarah Johnson",
      "relationship": "Previous collaborator",
      "contact": "sarah@email.com",
      "projectName": "Urban Soundscapes EP"
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Application submitted successfully",
  "data": {
    "application": {
      "id": "01HYNAPP01ABCDE1234567890",
      "status": "pending",
      "submittedAt": "2025-07-19T18:00:00Z",
      "reviewBy": "2025-07-26T18:00:00Z"
    }
  }
}
```

### Manage Project Applications

**Endpoint:** `GET /projects/{projectId}/applications` (Project Owner Only)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "applications": [
      {
        "id": "01HYNAPP01ABCDE1234567890",
        "applicant": {
          "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
          "profile": {
            "displayName": "Jane D",
            "avatarUrl": "https://cdn.phantompower.app/avatars/janedoe.jpg",
            "location": "Brooklyn, NY"
          },
          "stats": {
            "completedProjects": 8,
            "rating": 4.6,
            "responseRate": 0.92
          },
          "verificationStatus": "verified"
        },
        "message": "Hi Alex! I'm a field recording artist...",
        "role": "sound-designer",
        "matchScore": 0.91,
        "portfolioItems": [
          {
            "id": "01HYN91QZAYVNNRH53SJ1KMBCY", 
            "title": "Forest Ambience Collection",
            "plays": 1456,
            "likes": 89
          }
        ],
        "status": "pending",
        "submittedAt": "2025-07-19T18:00:00Z"
      }
    ],
    "summary": {
      "total": 8,
      "pending": 5,
      "accepted": 2,
      "rejected": 1,
      "averageMatchScore": 0.74
    }
  }
}
```

### Respond to Application

**Endpoint:** `PUT /projects/{projectId}/applications/{applicationId}` (Project Owner Only)

**Request Body:**
```json
{
  "status": "accepted", // "accepted", "rejected", "pending"
  "message": "Love your field recording work! Welcome to the project. Let's schedule a kick-off call.",
  "role": "sound-designer",
  "terms": {
    "compensation": "20% revenue share",
    "creditAs": "Sound Design & Field Recordings",
    "deadlines": {
      "firstDraft": "2025-09-01T00:00:00Z",
      "finalDelivery": "2025-11-15T00:00:00Z"
    }
  }
}
```

## Booking System

### Create Booking Request

**Endpoint:** `POST /projects/{projectId}/bookings`

**Request Body:**
```json
{
  "participantId": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
  "type": "recording-session", // "recording-session", "rehearsal", "writing-session", "mixing", "performance"
  "title": "Initial Recording Session",
  "description": "Record foundation tracks for ambient album",
  "schedule": {
    "startTime": "2025-08-15T14:00:00Z",
    "endTime": "2025-08-15T18:00:00Z",
    "timezone": "America/New_York",
    "recurring": false
  },
  "location": {
    "type": "in-person", // "in-person", "remote", "hybrid"
    "venue": "Brooklyn Recording Studio",
    "address": "123 Music St, Brooklyn, NY 11201",
    "equipment": ["Pro Tools Studio", "Neve Console", "Vintage Mics"]
  },
  "requirements": {
    "preparation": "Bring modular synth setup and 3-4 ambient sketches",
    "equipment": ["Modular Synth", "Cables", "Laptop with DAW"],
    "materials": ["Sheet music", "Reference tracks"]
  },
  "compensation": {
    "sessionFee": 200,
    "currency": "USD",
    "paymentTerms": "Net 30",
    "expenses": "Travel reimbursed"
  }
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "booking": {
      "id": "01HYBOOK01ABCDE1234567890",
      "status": "requested",
      "calendarInviteSent": true,
      "createdAt": "2025-07-19T17:20:00Z"
    }
  }
}
```

### Respond to Booking Request

**Endpoint:** `PUT /bookings/{bookingId}/respond`

**Request Body:**
```json
{
  "status": "confirmed", // "confirmed", "declined", "counter-offer"
  "message": "Confirmed! Looking forward to the session.",
  "counterOffer": null, // Only if status is "counter-offer"
  "calendarSync": true
}
```

### Get Project Bookings

**Endpoint:** `GET /projects/{projectId}/bookings`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "bookings": [
      {
        "id": "01HYBOOK01ABCDE1234567890",
        "type": "recording-session",
        "title": "Initial Recording Session",
        "status": "confirmed",
        "participants": [
          {
            "userId": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
            "role": "producer",
            "status": "confirmed",
            "profile": {
              "displayName": "Jane D",
              "avatarUrl": "https://cdn.phantompower.app/avatars/janedoe.jpg"
            }
          }
        ],
        "schedule": {
          "startTime": "2025-08-15T14:00:00Z",
          "endTime": "2025-08-15T18:00:00Z",
          "duration": "4 hours"
        },
        "location": {
          "type": "in-person",
          "venue": "Brooklyn Recording Studio",
          "address": "123 Music St, Brooklyn, NY 11201"
        },
        "createdAt": "2025-07-19T17:20:00Z",
        "updatedAt": "2025-07-19T17:25:00Z"
      }
    ],
    "calendar": {
      "icsUrl": "https://api.phantompower.app/projects/01HYNPRJ01/calendar.ics",
      "googleCalendarUrl": "https://calendar.google.com/calendar/embed?src=..."
    }
  }
}
```

## Matching System

### Get Project Matches

**Endpoint:** `GET /matches/projects`

**Query Parameters:**
- `genre` (string[]) - Genre preferences
- `location` (string) - Location filter
- `remote` (boolean) - Include remote projects
- `experience` (string) - Experience level
- `limit` (number) - Number of matches

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "matches": [
      {
        "project": {
          "id": "01HYNPRJ01ABCDE1234567890",
          "title": "Ambient Album Collaboration",
          "summary": "Remote collaboration for ambient album",
          "owner": {
            "displayName": "Alex Chen",
            "rating": 4.8
          }
        },
        "matchScore": 0.91,
        "reasons": [
          "Genre match: ambient, experimental",
          "Similar audio style (87% compatibility)",
          "Remote collaboration preferred",
          "Compatible equipment setup"
        ],
        "compatibility": {
          "audioStyle": 0.87,
          "genre": 0.95,
          "location": 1.0,
          "experience": 0.82,
          "schedule": 0.78
        },
        "matchedAt": "2025-07-19T19:00:00Z"
      }
    ],
    "algorithm": {
      "version": "v2.1",
      "factors": [
        "Audio analysis compatibility",
        "Genre preferences",
        "Location and remote preferences",
        "Experience level alignment",
        "Schedule compatibility",
        "Equipment compatibility"
      ]
    }
  }
}
```

### Get User Matches

**Endpoint:** `GET /matches/users`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "matches": [
      {
        "user": {
          "id": "01HYN8DEF456GHI789012345",
          "profile": {
            "displayName": "Beat Maker",
            "avatarUrl": "https://cdn.phantompower.app/avatars/beatmaker.jpg",
            "location": "Brooklyn, NY",
            "bio": "Electronic producer specializing in ambient textures"
          },
          "stats": {
            "completedProjects": 15,
            "rating": 4.7
          }
        },
        "matchScore": 0.84,
        "reasons": [
          "Complementary audio styles",
          "Both interested in ambient/electronic fusion",
          "Similar location (Brooklyn)",
          "Compatible schedules and availability"
        ],
        "compatibility": {
          "audioAnalysis": 0.78,
          "genrePreferences": 0.91,
          "location": 0.95,
          "collaborationStyle": 0.82
        },
        "commonConnections": 3,
        "mutualProjects": 1,
        "matchedAt": "2025-07-19T19:15:00Z"
      }
    ]
  }
}
```

## Project Analytics & Insights

### Get Project Analytics

**Endpoint:** `GET /projects/{projectId}/analytics` (Project Owner Only)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "analytics": {
      "views": {
        "total": 247,
        "unique": 198,
        "daily": [12, 18, 25, 31, 22, 19, 28],
        "sources": {
          "search": 0.45,
          "recommendations": 0.32,
          "profile": 0.18,
          "direct": 0.05
        }
      },
      "applications": {
        "total": 8,
        "accepted": 2,
        "pending": 5,
        "rejected": 1,
        "conversionRate": 0.032,
        "averageMatchScore": 0.74,
        "timeToApply": {
          "average": "2.3 days",
          "median": "1.8 days"
        }
      },
      "engagement": {
        "bookmarks": 15,
        "shares": 3,
        "profileViews": 42,
        "portfolioPlays": 156
      },
      "demographics": {
        "experience": {
          "beginner": 0.125,
          "intermediate": 0.5,
          "advanced": 0.25,
          "professional": 0.125
        },
        "locations": {
          "New York": 0.375,
          "Remote": 0.25,
          "Los Angeles": 0.125,
          "Other": 0.25
        },
        "genres": {
          "ambient": 0.75,
          "electronic": 0.5,
          "experimental": 0.375,
          "drone": 0.25
        }
      },
      "recommendations": [
        "Consider expanding location to include remote collaborators",
        "Your project attracts high-quality intermediate to advanced musicians",
        "Applications typically arrive within 2-3 days of viewing"
      ]
    }
  }
}
```

## Advanced Features

### Project Templates

**Endpoint:** `GET /projects/templates`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "id": "album-collaboration",
        "name": "Album Collaboration",
        "description": "Multi-track album with revenue sharing",
        "category": "recording",
        "popularity": 0.85,
        "template": {
          "type": "album",
          "timeline": {
            "estimatedHours": 40,
            "duration": "3-6 months"
          },
          "compensation": {
            "type": "revenue-split"
          },
          "roles": ["producer", "songwriter", "vocalist", "mixer"],
          "milestones": [
            "Pre-production planning",
            "Recording sessions",
            "Mixing and mastering",
            "Release preparation"
          ]
        }
      },
      {
        "id": "live-performance",
        "name": "Live Performance",
        "description": "One-time or recurring live performances",
        "category": "performance",
        "template": {
          "type": "gig",
          "location": { "type": "in-person" },
          "compensation": { "type": "paid" },
          "requirements": {
            "equipmentRequired": ["Instrument", "Amplifier"],
            "rehearsals": true
          }
        }
      }
    ]
  }
}
```

### Dispute Resolution

**Endpoint:** `POST /projects/{projectId}/disputes`

**Request Body:**
```json
{
  "type": "payment", // "payment", "creative", "timeline", "scope", "communication"
  "description": "Payment terms not being met as agreed",
  "involvedParties": ["01HYN8FXZ2W6X8K6YTNPEKXYM7"],
  "evidence": [
    {
      "type": "message",
      "id": "01HYNMSG1ABCDE1234567890",
      "description": "Agreement discussion"
    },
    {
      "type": "file",
      "url": "https://cdn.phantompower.app/disputes/contract.pdf",
      "description": "Signed agreement"
    }
  ],
  "requestedResolution": "Mediation with platform moderator",
  "priority": "medium" // "low", "medium", "high", "urgent"
}
```

### Project Reviews & Ratings

**Endpoint:** `POST /projects/{projectId}/reviews`

**Request Body:**
```json
{
  "rating": 5, // 1-5 stars
  "review": "Amazing collaboration experience! Alex was professional, creative, and delivered high-quality work on time. Would definitely work together again.",
  "categories": {
    "communication": 5,
    "creativity": 5,
    "reliability": 5,
    "technical": 4,
    "collaboration": 5
  },
  "wouldCollaborateAgain": true,
  "tags": ["professional", "creative", "reliable", "great-communicator"],
  "private": false // If true, review only visible to reviewed user
}
```

## Security & Permissions

### Role-Based Access Control

#### Project Owner Permissions
- Create, edit, delete projects
- View all applications and analytics
- Accept/reject applications
- Create and manage bookings
- Access dispute resolution
- Export project data

#### Project Participant Permissions
- View project details and updates
- Create booking requests
- Access project communication channels
- Submit deliverables
- Rate and review collaboration

#### Public Permissions
- View public project listings
- Apply to open projects
- Bookmark projects
- Share project links

### Data Protection

#### Sensitive Data Handling
- Payment information encrypted
- Personal contact details protected
- Application data only visible to project owners
- Analytics anonymized for privacy
- GDPR compliance for EU users

#### Audit Logging
- All project modifications logged
- Application status changes tracked
- Payment and booking events recorded
- User access patterns monitored
- Dispute activities documented

## Error Handling

### Common Error Codes
- `PROJECT_NOT_FOUND` - Project does not exist
- `APPLICATION_CLOSED` - Project no longer accepting applications
- `INSUFFICIENT_PERMISSIONS` - User lacks required permissions
- `APPLICATION_DUPLICATE` - User already applied to project
- `BOOKING_CONFLICT` - Scheduling conflict detected
- `PAYMENT_REQUIRED` - Premium feature requires payment
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `VALIDATION_ERROR` - Invalid input data
- `CAPACITY_EXCEEDED` - Project at maximum participants

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "APPLICATION_CLOSED",
    "message": "This project is no longer accepting applications",
    "details": {
      "projectId": "01HYNPRJ01ABCDE1234567890",
      "applicationDeadline": "2025-07-25T23:59:59Z",
      "currentStatus": "in-progress"
    }
  },
  "timestamp": "2025-07-19T20:00:00Z",
  "path": "/api/projects/01HYNPRJ01ABCDE1234567890/apply"
}
```

## Integration Notes

### Calendar Integration
- iCal feed export for bookings
- Google Calendar sync
- Outlook integration
- Timezone handling
- Recurring event support

### Payment Integration
- Stripe for payment processing
- Escrow services for project payments
- Revenue split calculations
- Tax reporting compliance
- Multi-currency support

### Communication Integration
- Slack workspace creation
- Discord server setup
- Zoom meeting scheduling
- File sharing integration
- Real-time collaboration tools

<!-- TODO: Implement contract generation system -->
<!-- TODO: Add project milestone tracking -->
<!-- TODO: Create project portfolio generation -->
<!-- TODO: Implement reputation scoring algorithm -->
<!-- TODO: Add project recommendation engine -->
<!-- TODO: Create advanced matching algorithms -->
<!-- TODO: Implement dispute mediation system -->
<!-- TODO: Add project analytics dashboard -->
<!-- TODO: Create payment processing workflows -->
<!-- TODO: Implement project template system -->