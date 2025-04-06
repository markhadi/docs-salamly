# Mobile App Integration

This documentation explains how to integrate the referral system with mobile applications.

### 1. APIs for Mobile Applications

Mobile applications integrate with the referral system through two main APIs:

#### 1.1 Track Registration API

**Endpoint:** `/api/referral/track-registration`

**Method:** POST

**Request Body:**
```json
{
  "referralCode": "string", // Referral code (required)
  "userData": {
    "uid": "string", // Unique user ID (required)
    "username": "string", // Username
    "email": "string", // User email (optional)
    "platform": "string", // User platform (android/ios)
    "loginMethod": "string" // Login method (google/apple/email)
  }
}
```

**Responses:**

- **200 OK**
```json
{
  "success": true,
  "statusCode": 200,
  "message": "User registration with referral code tracked successfully",
  "data": {
    "userId": "string",
    "username": "string",
    "inviterId": "string",
    "inviterUsername": "string",
    "referralCode": "string",
    "platform": "string",
    "loginMethod": "string",
    "hasPosted": false,
    "earned": 0,
    "createdAt": "string"
  },
  "timestamp": "string"
}
```

- **400 Bad Request** - Invalid referral code
```json
{
  "success": false,
  "statusCode": 400,
  "message": "Failed to track user registration",
  "errors": {
    "code": "REFERRAL_CODE_INVALID",
    "details": {
      "referralCode": "string",
      "message": "The referral code does not exist"
    }
  },
  "timestamp": "string"
}
```

- **409 Conflict** - User already tracked with a referral code
```json
{
  "success": false,
  "statusCode": 409,
  "message": "Failed to track user registration",
  "errors": {
    "code": "REFERRAL_USER_ALREADY_TRACKED",
    "details": {
      "userId": "string",
      "inviterId": "string",
      "message": "This user has already been tracked with a referral code"
    }
  },
  "timestamp": "string"
}
```

#### 1.2 Track Post API

**Endpoint:** `/api/referral/track-post`

**Method:** POST

**Request Body:**
```json
{
  "userId": "string" // Unique user ID (required)
}
```

**Responses:**

- **200 OK** - Successfully tracked post
```json
{
  "success": true,
  "statusCode": 200,
  "message": "User post tracked successfully",
  "data": {
    "userId": "string",
    "tracked": true,
    "updated": true,
    "hasPosted": true,
    "postedAt": "string"
  },
  "timestamp": "string"
}
```

- **200 OK** - User has already posted
```json
{
  "success": true,
  "statusCode": 200,
  "message": "User already marked as posted, no update needed",
  "data": {
    "userId": "string",
    "tracked": true,
    "updated": false,
    "hasPosted": true,
    "inviterId": "string",
    "referralCode": "string",
    "postedAt": "string"
  },
  "timestamp": "string"
}
```

- **200 OK** - User not registered with any referral code
```json
{
  "success": true,
  "statusCode": 200,
  "message": "User not registered with any referral code, no action needed",
  "data": {
    "userId": "string",
    "tracked": false,
    "updated": false
  },
  "timestamp": "string"
}
```

- **404 Not Found** - Tracking record not found
```json
{
  "success": false,
  "statusCode": 404,
  "message": "No tracking record found to update",
  "errors": {
    "code": "REFERRAL_TRACKING_NOT_FOUND",
    "details": {
      "userId": "string",
      "message": "No tracking record found for this user"
    }
  },
  "timestamp": "string"
}
```

### 2. Mobile Integration Flow

#### 2.1 Capturing Referral Code

1. Implement deep linking in your mobile app to capture the `referral` parameter from the URL
   ```
   example: https://play.google.com/store/apps/details?id=com.salamgram.salamgram&referral=testuser5
   example: https://apps.apple.com/us/app/salamly-ramadan-discover/id1612064624?referral=testuser5
   ```

2. Store the referral code in local storage

#### 2.2 Tracking User Registration

1. When a user successfully registers or logs in, retrieve the referral code from local storage
2. Send a POST request to the track-registration API with the following data:
   - Referral code retrieved from storage
   - User data including unique ID, username, platform (iOS/Android), and login method
3. Process the API response:
   - If successful (200 OK), store relevant information about the referral
   - If error occurs (400, 409), handle the error appropriately


#### 2.3 Tracking User Posts

1. When a user creates a post for the first time, send a POST request to the track-post API with the user's unique ID
2. Process the API response:
   - If successful with updated=true, the user's post has been tracked and the referrer will receive credit
   - If successful with updated=false, the user has already been marked as having posted
   - If the user was not registered with a referral code, no action is needed
