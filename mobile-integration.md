# Salamly Referral System - Mobile Integration API Documentation

## 1. Overview

- **Base URL:** `https://dev.d1dxbobz3kirt3.amplifyapp.com/api`
- **Content Type:** `application/json`

This API documentation provides details on how to integrate mobile applications with the Salamly Referral System. The system allows users to register using referral codes and tracks user activities for referral rewards.

---

## 2. Endpoints

### GET `/referral/validate`

> Validate a referral code before user registration

**Request:**

- **Method:** `GET`
- **URL:** `/referral/validate`

**Response:**

```json
{
  "success": true,
  "message": "Referral code is valid",
  "inviterData": {
    "id": "inviter123",
    "username": "inviter_username"
  }
}
```

**Error Responses:**

```json
{
  "success": false,
  "message": "Referral code is required"
}
```

```json
{
  "success": false,
  "message": "Referral code not found"
}
```

```json
{
  "success": false,
  "message": "Referral code is inactive"
}
```

---

### POST `/referral/track-registration`

> Track a new user registration with a referral code

**Request:**

- **Method:** `POST`
- **URL:** `/referral/track-registration`
- **Headers:** `Content-Type: application/json`

**Request Body:**

```json
{
  "referralCode": "ABC123",
  "userData": {
    "uid": "user123",
    "username": "new_user",
    "platform": "android",
    "loginMethod": "email"
  }
}
```

**Response:**

```json
{
  "success": true,
  "message": "User registration tracked successfully",
  "data": {
    "trackingId": "tracking123",
    "inviterId": "inviter123",
    "userId": "user123",
    "username": "new_user",
    "platform": "android",
    "loginMethod": "email",
    "hasPosted": false,
    "earned": 0
  }
}
```

**Error Responses:**

```json
{
  "success": false,
  "message": "Referral code and user data are required"
}
```

```json
{
  "success": false,
  "message": "Invalid referral code"
}
```

---

### POST `/referral/track-post`

> Track when a user makes their first post

**Request:**

- **Method:** `POST`
- **URL:** `/referral/track-post`
- **Headers:** `Content-Type: application/json`

**Request Body:**

```json
{
  "userId": "user123"
}
```

**Response:**

```json
{
  "success": true,
  "message": "User post status updated successfully",
  "data": {
    "success": true,
    "tracked": true,
    "updated": true,
    "updatedInviters": [
      {
        "inviterId": "inviter123",
        "trackingId": "tracking123",
        "earned": 0.02
      }
    ],
    "message": "User post status updated successfully"
  }
}
```

**Alternative Responses:**

```json
{
  "success": true,
  "message": "User not tracked by any inviter",
  "data": {
    "success": true,
    "tracked": false,
    "message": "User not registered with any referral code, no action needed"
  }
}
```

```json
{
  "success": true,
  "message": "Post tracking processed successfully",
  "data": {
    "success": true,
    "tracked": true,
    "updated": false,
    "message": "User already marked as posted, no update needed"
  }
}
```

**Error Response:**

```json
{
  "success": false,
  "message": "User ID is required"
}
```

---

## 3. Integration Flow

### User Registration with Referral Code

1. **Validate Referral Code:**

   - Before completing user registration, validate the referral code using the `/referral/validate` endpoint
   - If the code is valid, proceed with registration
   - If invalid, inform the user and allow them to continue without a referral code

2. **Complete Registration:**

   - Register the user in your authentication system
   - Create a user document in Firestore with required user information

3. **Track Registration:**

   - After successful registration, call the `/referral/track-registration` endpoint
   - This will link the new user to the inviter and prepare for reward tracking

4. **Track First Post:**
   - When the user makes their first post in the app, call the `/referral/track-post` endpoint
   - This will update the inviter's earnings and complete the referral process

---

## 4. Testing

You can test these APIs using the following test pages:

- Validate Referral Code: `https://dev.d1dxbobz3kirt3.amplifyapp.com/test-register-api`
- Track Registration: `https://dev.d1dxbobz3kirt3.amplifyapp.com/test-register-api`
- Track Post: `https://dev.d1dxbobz3kirt3.amplifyapp.com/test-post-api`
