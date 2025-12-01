# Multi-Authentication Login System

A Spring Boot backend API supporting multiple authentication methods: Email, Mobile, and Google OAuth2. Includes JWT-based authentication and protected CRUD endpoints.

## Features

- Register and login with email, mobile, or both
- Google OAuth2 integration
- JWT token-based authentication
- Protected username CRUD operations
- MongoDB Atlas cloud database
- Spring Security with industry-standard practices

## Tech Stack

- **Spring Boot 3.5.7** | **Java 25** | **MongoDB Atlas** | **JWT** | **Spring Security** | **Maven**

***

## Quick Setup

### 1. Prerequisites

- Java 17+ installed
- Maven installed
- MongoDB Atlas account ([sign up free](https://www.mongodb.com/cloud/atlas))
- Google Cloud Console account ([get started](https://console.cloud.google.com/))

### 2. MongoDB Atlas Setup

- Create a free cluster
- Go to **Database Access** → Create user with password
- Go to **Network Access** → Add IP `0.0.0.0/0` (allow from anywhere)
- Click **Connect** → Copy your connection string

### 3. Google OAuth2 Setup

- Create a project in Google Cloud Console
- Go to **APIs & Services** → **Credentials**
- Create **OAuth 2.0 Client ID** (Web application)
- Add redirect URI: `http://localhost:8080/login/oauth2/code/google`
- Save your **Client ID** and **Client Secret**

### 4. Configure Application

Edit `src/main/resources/application.properties`:

```properties
spring.application.name=demo

# MongoDB Atlas
spring.data.mongodb.uri=mongodb+srv://USERNAME:PASSWORD@CLUSTER_URL/DATABASE_NAME?retryWrites=true&w=majority

# Google OAuth2
spring.security.oauth2.client.registration.google.client-id=YOUR_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_CLIENT_SECRET
spring.security.oauth2.client.registration.google.scope=email,profile
spring.security.oauth2.client.registration.google.redirect-uri={baseUrl}/login/oauth2/code/{registrationId}

spring.security.oauth2.client.provider.google.authorization-uri=https://accounts.google.com/o/oauth2/auth
spring.security.oauth2.client.provider.google.token-uri=https://oauth2.googleapis.com/token
spring.security.oauth2.client.provider.google.user-info-uri=https://www.googleapis.com/oauth2/v3/userinfo
spring.security.oauth2.client.provider.google.user-name-attribute=sub

# JWT
jwt.secret=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970
jwt.expiration=86400000

# Server
server.port=8080
```

### 5. Run Application

```bash
mvn clean install
mvn spring-boot:run
```

Application starts at: `http://localhost:8080`

***

## API Testing with Postman

### Setup Postman Environment

1. Create environment: `Multi-Auth Dev`
2. Add variable: `jwt_token` (leave empty)
3. Select this environment

### Test Flow

#### 1. Register User

```http
POST http://localhost:8080/api/auth/register
Content-Type: application/json

{
  "email": "alice@example.com",
  "mobile": "+919876543210",
  "password": "Alice@123"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "userId": "674b3c8f9e1234567890abcd",
  "message": "Registration successful"
}
```

**Save token:** Go to **Tests** tab, add:
```javascript
if (pm.response.code === 200) {
    pm.environment.set("jwt_token", pm.response.json().token);
}
```

***

#### 2. Login with Email

```http
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "email": "alice@example.com",
  "password": "Alice@123"
}
```

***

#### 3. Login with Mobile

```http
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "mobile": "+919876543210",
  "password": "Alice@123"
}
```

***

#### 4. Login with Google

First, get Google ID token from [OAuth Playground](https://developers.google.com/oauthplayground/):
- Configure with your Client ID and Secret
- Select scopes: `userinfo.email` and `userinfo.profile`
- Authorize and get `id_token`

```http
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "googleToken": "YOUR_GOOGLE_ID_TOKEN"
}
```

***

#### 5. Create Username (Protected)

```http
POST http://localhost:8080/api/username
Authorization: Bearer {{jwt_token}}
Content-Type: application/json

{
  "username": "alice_wonder"
}
```

**Response:**
```json
{
  "userId": "674b3c8f9e1234567890abcd",
  "username": "alice_wonder"
}
```

***

#### 6. Get Username

```http
GET http://localhost:8080/api/username
Authorization: Bearer {{jwt_token}}
```

***

#### 7. Update Username

```http
PUT http://localhost:8080/api/username
Authorization: Bearer {{jwt_token}}
Content-Type: application/json

{
  "username": "alice_updated"
}
```

***

#### 8. Delete Username

```http
DELETE http://localhost:8080/api/username
Authorization: Bearer {{jwt_token}}
```

**Response:** `204 No Content`

***

## Project Structure

```
src/main/java/com/example/multiauth/
├── controller/      # REST endpoints
├── dto/            # Request/Response objects
├── model/          # User entity
├── repository/     # MongoDB repository
├── service/        # Business logic
├── security/       # JWT & Security config
└── exception/      # Error handling
```

***

## Common Issues

**MongoDB connects to localhost instead of Atlas:**
- Verify `application.properties` is in `src/main/resources/`
- Run `mvn clean install` and restart

**JWT classes not found:**
- Maven → Reload Project
- Build → Rebuild Project

**401 Unauthorized on protected endpoints:**
- Check Authorization header: `Bearer YOUR_TOKEN`
- Token expires after 24 hours - login again

***

## Security Notes

For production:
- Change `jwt.secret` to a strong random value
- Use environment variables for secrets
- Restrict MongoDB network access to specific IPs
- Enable HTTPS

***