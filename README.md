# CommunityShield API Documentation

## Base URL
```
https://localhost:8000/api/v1
```

## Authentication
Most endpoints require authentication using a JWT token. Include the token in the Authorization header:
```
Authorization: Bearer <your_jwt_token>
```

## Error Responses
All endpoints may return these error status codes:
- 400: Bad Request - Invalid input data
- 401: Unauthorized - Missing or invalid authentication
- 403: Forbidden - Insufficient permissions
- 404: Not Found - Resource doesn't exist
- 429: Too Many Requests - Rate limit exceeded
- 500: Internal Server Error - Server-side error

## User Management

### Register New User
```http
POST /users/register
Content-Type: application/json

{
  "username": "string",
  "email": "string",
  "password": "string",
  "skills": [
    {
      "name": "string",
      "level": "BEGINNER" | "INTERMEDIATE" | "EXPERT"
    }
  ],
  "location": {
    "type": "Point",
    "coordinates": [longitude, latitude]
  }
}

Response 201:
{
  "success": true,
  "data": {
    "user": {
      "id": "string",
      "username": "string",
      "email": "string",
      "skills": [...],
      "location": {...}
    },
    "token": "string"
  }
}
```

### Authenticate User
```http
POST /users/login
Content-Type: application/json

{
  "email": "string",
  "password": "string"
}

Response 200:
{
  "success": true,
  "data": {
    "user": {...},
    "token": "string"
  }
}
```

### Update User Skills
```http
PUT /users/skills
Authorization: Bearer <token>
Content-Type: application/json

{
  "skills": [
    {
      "name": "string",
      "level": "BEGINNER" | "INTERMEDIATE" | "EXPERT"
    }
  ]
}

Response 200:
{
  "success": true,
  "data": {
    "user": {...}
  }
}
```

### Find Nearby Skills
```http
GET /users/skills/nearby?latitude=<number>&longitude=<number>&skill=<string>&maxDistance=<number>
Authorization: Bearer <token>

Response 200:
{
  "success": true,
  "data": [
    {
      "username": "string",
      "skills": [...],
      "location": {...},
      "distance": number
    }
  ]
}
```

## Resource Management

### Create Resource
```http
POST /resources
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "string",
  "type": "WATER" | "MEDICAL" | "SHELTER" | "FOOD" | "OTHER",
  "quantity": number,
  "location": {
    "type": "Point",
    "coordinates": [longitude, latitude]
  },
  "status": "AVAILABLE" | "LOW" | "DEPLETED",
  "description": "string",
  "expiryDate": "ISO8601 date string",
  "contactInfo": "string"
}

Response 201:
{
  "success": true,
  "data": {
    "resource": {...}
  }
}
```

### Find Nearby Resources
```http
GET /resources/nearby?latitude=<number>&longitude=<number>&type=<string>&maxDistance=<number>
Authorization: Bearer <token>

Response 200:
{
  "success": true,
  "data": [
    {
      "name": "string",
      "type": "string",
      "quantity": number,
      "location": {...},
      "status": "string",
      "distance": number
    }
  ]
}
```

### Update Resource Status
```http
PATCH /resources/:id/status
Authorization: Bearer <token>
Content-Type: application/json

{
  "status": "AVAILABLE" | "LOW" | "DEPLETED",
  "quantity": number
}

Response 200:
{
  "success": true,
  "data": {
    "resource": {...}
  }
}
```

## Emergency Alerts

### Create Alert
```http
POST /alerts
Authorization: Bearer <token>
Content-Type: application/json

{
  "type": "EMERGENCY" | "MEDICAL" | "EVACUATION" | "SUPPLY",
  "priority": "HIGH" | "MEDIUM" | "LOW",
  "location": {
    "type": "Point",
    "coordinates": [longitude, latitude]
  },
  "description": "string"
}

Response 201:
{
  "success": true,
  "data": {
    "alert": {...}
  }
}
```

### Respond to Alert
```http
POST /alerts/:alertId/respond
Authorization: Bearer <token>
Content-Type: application/json

{
  "message": "string"
}

Response 200:
{
  "success": true,
  "data": {
    "alert": {...}
  }
}
```

### Resolve Alert
```http
POST /alerts/:alertId/resolve
Authorization: Bearer <token>

Response 200:
{
  "success": true,
  "data": {
    "alert": {...}
  }
}
```

## Knowledge Base

### Create Article
```http
POST /knowledge
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "string",
  "category": "FIRST_AID" | "DISASTER_RESPONSE" | "SURVIVAL" | "TECHNICAL" | "GENERAL",
  "content": "string",
  "tags": ["string"],
  "language": "string",
  "region": ["string"],
  "offline": boolean,
  "priority": number
}

Response 201:
{
  "success": true,
  "data": {
    "article": {...}
  }
}
```

### Search Knowledge Base
```http
GET /knowledge/search?query=<string>&category=<string>&tags=<comma-separated-strings>&language=<string>&region=<string>&page=<number>&limit=<number>
Authorization: Bearer <token>

Response 200:
{
  "success": true,
  "data": {
    "items": [...],
    "total": number,
    "page": number,
    "totalPages": number
  }
}
```

### Get Offline Articles
```http
GET /knowledge/offline?language=<string>&region=<string>
Authorization: Bearer <token>

Response 200:
{
  "success": true,
  "data": [
    {
      "title": "string",
      "category": "string",
      "content": "string",
      "version": number,
      "lastUpdated": "ISO8601 date string"
    }
  ]
}
```

### Review Article
```http
POST /knowledge/:articleId/review
Authorization: Bearer <token>
Content-Type: application/json

{
  "status": "APPROVED" | "REJECTED",
  "comments": "string"
}

Response 200:
{
  "success": true,
  "data": {
    "article": {...}
  }
}
```

## WebSocket Events
The application also supports real-time communication through WebSocket connections:

### Connection
```javascript
// Connect with authentication
socket.connect({
  auth: {
    token: "your_jwt_token"
  }
});
```

### Events
- `resource:update`: Emitted when a resource is updated
- `emergency:alert`: Emitted when a new emergency alert is created
- `mesh:message`: Emitted for peer-to-peer communication
- `user:status`: Emitted when a user's status changes
- `knowledge:sync`: Emitted during knowledge base synchronization

## Rate Limiting
All endpoints are subject to rate limiting:
- 100 requests per 15-minute window per IP address
- WebSocket connections are limited to 10 concurrent connections per user

## Offline Support
The application supports offline functionality through:
- IndexedDB for local storage
- Service Workers for offline access
- Mesh networking for peer-to-peer communication
- Automatic synchronization when online connectivity is restored