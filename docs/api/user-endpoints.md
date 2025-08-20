# User & Profile API

## Overview

The User API manages user profiles, discovery, following relationships, and social interactions. All endpoints require authentication unless otherwise specified.

## Base URL

```
Development: http://localhost:8080/api/users
Production: https://api.phantompower.app/api/users
```

## User Profile Management

### Get User Profile

Retrieve detailed user profile information.

**Endpoint:** `GET /profile/{userId}`

**Parameters:**
- `userId` (path) - User ID or "me" for current user

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
      "email": "jane.doe@example.com"
    },
    "profile": {
      "id": "01HYN8ZMEW2DTPP7YYPXFPZ27R",
      "userId": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
      "displayName": "Jane D",
      "bio": "Producer. Vocalist. Gearhead. Let's build something weird.",
      "location": "Brooklyn, NY",
      "avatarUrl": "https://cdn.phantompower.app/avatars/janedoe.jpg",
      "bannerUrl": "https://cdn.phantompower.app/banners/janedoe-banner.jpg",
      "theme": "retro-dark",
      "socialLinks": {
        "soundcloud": "https://soundcloud.com/janedoe",
        "instagram": "https://instagram.com/janedoe"
      },
      "featuredAudioId": "01HYN91QZAYVNNRH53SJ1KMBCY",
      "tags": ["ambient", "experimental", "modular"],
      "stats": {
        "followers": 1247,
        "following": 89,
        "audioFiles": 23,
        "projects": 5
      },
      "createdAt": "2025-07-19T12:10:00Z",
      "updatedAt": "2025-07-19T18:30:00Z"
    }
  }
}
```

**Error Responses:**
- `404 Not Found` - User not found
- `403 Forbidden` - Private profile, access denied

### Update User Profile

Update current user's profile information.

**Endpoint:** `PUT /profile/me`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "displayName": "Jane Doe",
  "bio": "Electronic music producer and sound designer",
  "location": "Brooklyn, NY",
  "theme": "dark-mode",
  "socialLinks": {
    "soundcloud": "https://soundcloud.com/janedoe",
    "instagram": "https://instagram.com/janedoe",
    "twitter": "https://twitter.com/janedoe"
  },
  "tags": ["ambient", "techno", "experimental"]
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "data": {
    "profile": {
      "id": "01HYN8ZMEW2DTPP7YYPXFPZ27R",
      "displayName": "Jane Doe",
      "bio": "Electronic music producer and sound designer",
      "updatedAt": "2025-07-19T19:00:00Z"
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input data
- `401 Unauthorized` - Authentication required
- `422 Unprocessable Entity` - Validation errors

### Upload Profile Images

Upload avatar or banner images for user profile.

**Endpoint:** `POST /profile/me/images`

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: multipart/form-data
```

**Request Body (multipart/form-data):**
- `avatar` (file) - Profile avatar image (optional)
- `banner` (file) - Profile banner image (optional)
- `type` (string) - "avatar" or "banner"

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Image uploaded successfully",
  "data": {
    "imageUrl": "https://cdn.phantompower.app/avatars/01HYN8FXZ2W6X8K6YTNPEKXYM7.jpg",
    "type": "avatar"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid file format or size
- `401 Unauthorized` - Authentication required
- `413 Payload Too Large` - File size exceeds limit (5MB)

## User Discovery

### Search Users

Search for users by name, location, tags, or other criteria.

**Endpoint:** `GET /search`

**Query Parameters:**
- `q` (string) - Search query (name, bio, location)
- `tags` (string) - Comma-separated tags
- `location` (string) - Location filter
- `page` (number) - Page number (default: 1)
- `limit` (number) - Results per page (default: 20, max: 100)
- `sort` (string) - Sort by: "relevance", "recent", "popular"

**Example Request:**
```
GET /api/users/search?q=ambient&tags=experimental,modular&limit=10
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
        "profile": {
          "displayName": "Jane D",
          "bio": "Producer. Vocalist. Gearhead.",
          "location": "Brooklyn, NY",
          "avatarUrl": "https://cdn.phantompower.app/avatars/janedoe.jpg",
          "tags": ["ambient", "experimental", "modular"],
          "stats": {
            "followers": 1247,
            "audioFiles": 23
          }
        },
        "isFollowing": false,
        "mutualFollowers": 3
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 87,
      "totalPages": 9,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

### Get Recommended Users

Get personalized user recommendations based on music taste and activity.

**Endpoint:** `GET /recommendations`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `limit` (number) - Number of recommendations (default: 10, max: 50)
- `reason` (string) - Filter by recommendation reason: "similar_taste", "mutual_followers", "location"

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "user": {
          "id": "01HYN8XYZ9QRS1234567890",
          "profile": {
            "displayName": "Alex Chen",
            "bio": "Ambient composer and field recording artist",
            "avatarUrl": "https://cdn.phantompower.app/avatars/alexchen.jpg",
            "tags": ["ambient", "field-recording", "drone"]
          }
        },
        "reason": "similar_taste",
        "matchScore": 0.87,
        "commonTags": ["ambient", "experimental"],
        "mutualFollowers": 5
      }
    ]
  }
}
```

## Following System

### Follow User

Follow another user to see their updates in your feed.

**Endpoint:** `POST /follow/{userId}`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Parameters:**
- `userId` (path) - ID of user to follow

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Successfully followed user",
  "data": {
    "isFollowing": true,
    "followedAt": "2025-07-19T20:00:00Z"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Cannot follow yourself
- `404 Not Found` - User not found
- `409 Conflict` - Already following user

### Unfollow User

Unfollow a user to stop seeing their updates.

**Endpoint:** `DELETE /follow/{userId}`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Successfully unfollowed user",
  "data": {
    "isFollowing": false
  }
}
```

### Get Followers

Get list of users following the specified user.

**Endpoint:** `GET /{userId}/followers`

**Query Parameters:**
- `page` (number) - Page number (default: 1)
- `limit` (number) - Results per page (default: 20, max: 100)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "followers": [
      {
        "id": "01HYN8ABC123DEF456789",
        "profile": {
          "displayName": "Music Lover",
          "avatarUrl": "https://cdn.phantompower.app/avatars/musiclover.jpg"
        },
        "followedAt": "2025-07-19T18:00:00Z",
        "isFollowingBack": true
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 1247,
      "totalPages": 63
    }
  }
}
```

### Get Following

Get list of users that the specified user is following.

**Endpoint:** `GET /{userId}/following`

**Response:** Same format as Get Followers

## Audio Files

### Get User Audio Files

Retrieve audio files uploaded by a specific user.

**Endpoint:** `GET /{userId}/audio`

**Query Parameters:**
- `page` (number) - Page number (default: 1)
- `limit` (number) - Results per page (default: 20, max: 50)
- `sort` (string) - Sort by: "recent", "popular", "title"
- `visibility` (string) - Filter by visibility: "public", "unlisted" (only for own files)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "audioFiles": [
      {
        "id": "01HYN91QZAYVNNRH53SJ1KMBCY",
        "title": "Cosmic Jam #3",
        "description": "Weird ambient mod synth groove",
        "fileUrl": "https://cdn.phantompower.app/audio/cosmic-jam-3.mp3",
        "waveformDataUrl": "https://cdn.phantompower.app/waveforms/cosmic-jam-3.json",
        "duration": 212.4,
        "visibility": "public",
        "tags": ["ambient", "modular", "experimental"],
        "stats": {
          "plays": 1456,
          "likes": 89,
          "comments": 12
        },
        "analysis": {
          "tempo": 120.5,
          "key": "C#m",
          "genre": "ambient",
          "mood": "contemplative"
        },
        "createdAt": "2025-07-19T13:00:00Z",
        "updatedAt": "2025-07-19T13:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 23,
      "totalPages": 2
    }
  }
}
```

### Upload Audio File

Upload a new audio file for the current user.

**Endpoint:** `POST /me/audio`

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: multipart/form-data
```

**Request Body (multipart/form-data):**
- `audio` (file) - Audio file (MP3, WAV, FLAC, M4A)
- `title` (string) - Audio title
- `description` (string, optional) - Audio description
- `tags` (string, optional) - Comma-separated tags
- `visibility` (string, optional) - "public", "private", "unlisted" (default: "public")

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Audio file uploaded successfully",
  "data": {
    "audioFile": {
      "id": "01HYN91QZAYVNNRH53SJ1KMBCY",
      "title": "New Track",
      "fileUrl": "https://cdn.phantompower.app/audio/new-track.mp3",
      "status": "processing",
      "createdAt": "2025-07-19T20:30:00Z"
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid file format or missing required fields
- `413 Payload Too Large` - File size exceeds limit (50MB)
- `429 Too Many Requests` - Upload rate limit exceeded

## User Activity

### Get User Feed

Get activity feed for the current user (posts from followed users).

**Endpoint:** `GET /me/feed`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `page` (number) - Page number (default: 1)
- `limit` (number) - Results per page (default: 20, max: 50)
- `type` (string) - Filter by activity type: "audio", "comment", "like", "follow"

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "activities": [
      {
        "id": "01HYNACT1ABCDE1234567890",
        "type": "audio_upload",
        "user": {
          "id": "01HYN8XYZ9QRS1234567890",
          "profile": {
            "displayName": "Alex Chen",
            "avatarUrl": "https://cdn.phantompower.app/avatars/alexchen.jpg"
          }
        },
        "content": {
          "audioFile": {
            "id": "01HYN91QZAYVNNRH53SJ1KMBCY",
            "title": "Morning Drone",
            "waveformDataUrl": "https://cdn.phantompower.app/waveforms/morning-drone.json"
          }
        },
        "createdAt": "2025-07-19T19:45:00Z"
      },
      {
        "id": "01HYNACT2ABCDE1234567891",
        "type": "follow",
        "user": {
          "id": "01HYN8DEF456GHI789012345",
          "profile": {
            "displayName": "Beat Maker",
            "avatarUrl": "https://cdn.phantompower.app/avatars/beatmaker.jpg"
          }
        },
        "content": {
          "followedUser": {
            "id": "01HYN8XYZ9QRS1234567890",
            "profile": {
              "displayName": "Alex Chen"
            }
          }
        },
        "createdAt": "2025-07-19T19:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "totalPages": 8
    }
  }
}
```

### Get User Notifications

Get notifications for the current user.

**Endpoint:** `GET /me/notifications`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
- `page` (number) - Page number (default: 1)
- `limit` (number) - Results per page (default: 20, max: 50)
- `unread` (boolean) - Filter unread notifications only
- `type` (string) - Filter by type: "like", "comment", "follow", "match"

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "01HYNNOTF1ABCDE1234567890",
        "type": "new_like",
        "message": "Alex Chen liked your track 'Cosmic Jam #3'",
        "isRead": false,
        "user": {
          "id": "01HYN8XYZ9QRS1234567890",
          "profile": {
            "displayName": "Alex Chen",
            "avatarUrl": "https://cdn.phantompower.app/avatars/alexchen.jpg"
          }
        },
        "content": {
          "audioFile": {
            "id": "01HYN91QZAYVNNRH53SJ1KMBCY",
            "title": "Cosmic Jam #3"
          }
        },
        "createdAt": "2025-07-19T19:25:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "unreadCount": 12
    }
  }
}
```

### Mark Notifications as Read

Mark one or more notifications as read.

**Endpoint:** `PUT /me/notifications/read`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "notificationIds": ["01HYNNOTF1ABCDE1234567890", "01HYNNOTF2ABCDE1234567891"],
  "markAll": false
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Notifications marked as read",
  "data": {
    "markedCount": 2
  }
}
```

## User Statistics

### Get User Stats

Get detailed statistics for a user profile.

**Endpoint:** `GET /{userId}/stats`

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "stats": {
      "profile": {
        "followers": 1247,
        "following": 89,
        "profileViews": 5632
      },
      "content": {
        "audioFiles": 23,
        "totalPlays": 45678,
        "totalLikes": 2341,
        "totalComments": 456
      },
      "social": {
        "projects": 5,
        "collaborations": 12,
        "matches": 34
      },
      "activity": {
        "lastActive": "2025-07-19T20:00:00Z",
        "joinDate": "2025-01-15T10:00:00Z",
        "streakDays": 15
      }
    }
  }
}
```

## Privacy & Settings

### Update Privacy Settings

Update user privacy and visibility settings.

**Endpoint:** `PUT /me/privacy`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "profileVisibility": "public",
  "showEmail": false,
  "showLocation": true,
  "allowMessages": "followers",
  "showActivity": true,
  "searchable": true
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Privacy settings updated successfully"
}
```

## Error Handling

All endpoints return standardized error responses:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "The requested user could not be found",
    "details": {
      "userId": "01INVALIDUSER123456789"
    }
  },
  "timestamp": "2025-07-19T20:00:00Z",
  "path": "/api/users/profile/01INVALIDUSER123456789"
}
```

### Common Error Codes
- `USER_NOT_FOUND` - User does not exist
- `PROFILE_PRIVATE` - Profile is private and not accessible
- `FOLLOW_SELF_ERROR` - Cannot follow yourself
- `ALREADY_FOLLOWING` - Already following this user
- `FILE_TOO_LARGE` - Uploaded file exceeds size limit
- `INVALID_FILE_TYPE` - Unsupported file format
- `RATE_LIMIT_EXCEEDED` - Too many requests

<!-- TODO: Implement user blocking functionality -->
<!-- TODO: Add user verification system -->
<!-- TODO: Implement advanced search filters -->
<!-- TODO: Add user analytics and insights -->