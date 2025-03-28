# Salamly Referral System

## System Overview

The Salamly Referral System is designed to track and reward inviters who successfully bring new users to the Salamly application. This document provides a comprehensive explanation of the system's workflow, from inviter creation to payment
processing.

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Inviter Management](#inviter-management)
3. [Referral Link Generation and Distribution](#referral-link-generation-and-distribution)
4. [User Registration with Referral Code](#user-registration-with-referral-code)
5. [Post Tracking and Earnings Calculation](#post-tracking-and-earnings-calculation)
6. [Dashboard and Reporting](#dashboard-and-reporting)
7. [Administration and Payment](#administration-and-payment)
8. [API Integration for Mobile Applications](#api-integration-for-mobile-applications)

## System Architecture

### Database Structure

The system uses Firebase Firestore with the following collections:

1. **`inviters` Collection**: Stores inviter information

   - `username`: Unique username for login
   - `password`: Encrypted password (using bcrypt)
   - `fullName`: Inviter's full name
   - `email`: Inviter's email address
   - `referralCode`: Unique referral code
   - `isActive`: Status of the inviter account
   - `createdAt`: Registration timestamp
   - `updatedAt`: Last update timestamp

2. **`userTracking` Subcollection**: Tracks users registered with the inviter's referral code

   - `userId`: User's UID in Firestore
   - `username`: User's username
   - `platform`: Registration platform (iOS/Android)
   - `loginMethod`: Authentication method used
   - `hasPosted`: Whether the user has made a post
   - `earned`: Amount earned from this user (0 or 0.02)
   - `registeredAt`: Registration timestamp

3. **`earnings` Subcollection**: Stores monthly earnings data
   - `monthYear`: Month and year (format: "MM-YYYY")
   - `userCount`: Number of users who made posts this month
   - `totalEarnings`: Total earnings for the month
   - `isPaid`: Payment status
   - `paidAt`: Payment timestamp (if paid)

## Inviter Management

### Creating a New Inviter

1. Admin or authorized user accesses the inviter creation page
2. Admin fills the form with inviter data:
   - Username
   - Password
   - Full name
   - Email
3. System validates the data:
   - Ensures username is unique
   - Verifies email format
   - Confirms password meets security criteria
4. System encrypts the password using bcrypt
5. System generates a unique referral code for the inviter:
   - Code is based on username with modifications to ensure uniqueness
   - Format: combination of letters and numbers (e.g., "ABC123")
6. System saves inviter data to the `inviters` collection in Firestore
7. System returns confirmation that the inviter was successfully created

### Inviter Authentication

1. Inviter opens the dashboard login page
2. Inviter enters username and password
3. System validates the credentials:
   - Finds inviter with the provided username
   - Compares entered password with stored hash
4. If valid, system creates a JWT token containing:
   - Inviter ID
   - Username
5. System returns the JWT token to the client
6. Client stores the token in localStorage for future authentication

## Referral Link Generation and Distribution

### Generating Referral Links

1. Inviter logs into the dashboard
2. System retrieves the inviter's referral code from the database
3. System creates referral links in the following formats:
   - App Store: `https://apps.apple.com/us/app/salamly-ramadan-discover/id1612064624?referral=REFERRAL_CODE`
   - Play Store: `https://play.google.com/store/apps/details?id=com.salamgram.salamgram&referral=REFERRAL_CODE`
4. System displays the referral links on the inviter's dashboard

### Distributing Referral Links

1. Inviter can copy the referral link from the dashboard
2. Inviter can share the link through:
   - "Copy Link" button
3. System creates links with appropriate parameters for each platform

## User Registration with Referral Code

### User Clicks Referral Link

1. User receives a referral link from an inviter
2. User clicks the link
3. Link opens App Store (iOS) or Play Store (Android) with the `referral` parameter in the URL
4. User downloads and installs the Salamly application

### Mobile App Captures Referral Code

1. When the app is first opened, the mobile developer captures the `referral` parameter from the URL
2. App stores this referral code locally for use during registration

### User Registers in the App

1. User fills out the registration form in the mobile app
2. Mobile developer creates a user document in the `users` collection in Firestore
3. If the user registers with a referral code, the mobile app calls the `/api/referral/track-registration` API

### Referral Code Validation and Tracking

1. API receives a request with `referralCode` and `userData` (containing at minimum `uid` and `username`)
2. API calls `ReferralService.trackUserRegistration()`
3. System validates the referral code with `InviterModel.getByReferralCode()`
4. If the code is valid and the inviter is active, system creates tracking data:
   ```javascript
   const trackingData = {
     userId: userData.uid,
     username: userData.username,
     platform: userData.platform || '',
     loginMethod: userData.loginMethod || '',
     hasPosted: false,
     earned: 0,
   };
   ```
5. System saves tracking data in the inviter's `userTracking` subcollection using `InviterModel.addUserTracking()`
6. API returns a response with the tracking data

## Post Tracking and Earnings Calculation

### User Creates a Post

1. User creates a post in the Salamly application
2. Mobile developer saves the post in the database
3. After the post is successfully saved, the mobile app calls the `/api/referral/track-post` API with the `userId`

### Post Status Validation and Update

1. API receives a request with `userId`
2. API calls `ReferralService.checkAndUpdateUserPostStatus()`
3. System finds all inviters tracking this user with `InviterModel.getAllInvitersTrackingData()`
4. System checks if the user exists in any inviter's tracking data:
   - If not found, API returns: `{ success: true, tracked: false, message: 'User not registered with any referral code, no action needed' }`
   - If found but already updated, API returns: `{ success: true, tracked: true, updated: false, message: 'User already marked as posted, no update needed' }`

### Status Update and Earnings Calculation

1. If the user is found and has not been updated before, the system:
   1. Updates the `hasPosted` status to `true` using `InviterModel.updateUserPosted()`
   2. Calculates the current month and year: `monthYear = "MM-YYYY"`
   3. Retrieves the inviter's earnings data with `InviterModel.getEarningsData()`
   4. Adds 1 to `userCount` and 0.02 to `totalEarnings` for the current month
   5. Updates the monthly earnings with `InviterModel.updateMonthlyEarnings()`
   6. Returns details of the updated inviters

## Dashboard and Reporting

### Dashboard Statistics

1. Inviter logs into the dashboard
2. System calls `DashboardService.getDashboardStats()` to retrieve statistics:
   - New users today and this month
   - Earnings today and this month
   - Total registered users
   - Total overall earnings
3. System displays statistics on the dashboard

### User Tracking List

1. Inviter can view a complete list of users who registered with their referral code
2. System displays:
   - User's username
   - Registration date
   - Platform
   - Post status (completed/pending)
   - Earnings (0 or 0.02)

### Earnings Reports

1. Inviter can view monthly earnings reports
2. System displays:
   - Month and year
   - Number of users who made posts
   - Total earnings
   - Payment status (paid/unpaid)
   - Payment date (if paid)

## Administration and Payment

### Admin Dashboard

1. Admin logs into the admin dashboard
2. Admin can view:
   - List of all inviters
   - Global statistics (total users, total earnings, etc.)
   - Unpaid earnings reports

### Inviter Management

1. Admin can view, add, edit, and deactivate inviters
2. Admin can view inviter details:
   - Profile data
   - Tracking statistics
   - Earnings reports

### Payment Processing

1. Admin views the list of unpaid earnings
2. Admin processes payments to inviters
3. Admin updates payment status through the dashboard
4. System updates the `isPaid` status and `paidAt` timestamp

## API Integration for Mobile Applications

### Key Integration Points

1. **Referral Code Validation**

   - Endpoint: `GET /api/referral/validate`
   - Used to validate referral codes before user registration

2. **Registration Tracking**

   - Endpoint: `POST /api/referral/track-registration`
   - Called after successful user registration with a referral code

3. **Post Tracking**
   - Endpoint: `POST /api/referral/track-post`
   - Called when a user makes their first post

### Integration Flow

1. **Validate Referral Code:**

   - Before completing user registration, validate the referral code
   - If valid, proceed with registration
   - If invalid, inform the user and allow them to continue without a referral code

2. **Complete Registration:**

   - Register the user in authentication system
   - Create a user document in Firestore

3. **Track Registration:**

   - After successful registration, track the registration with the referral code
   - This links the new user to the inviter

4. **Track First Post:**
   - When the user makes their first post, update the tracking status
   - This updates the inviter's earnings

---

## Security Considerations

- All passwords are encrypted using bcrypt
- JWT tokens are used for authentication

---

## Testing

The system includes test pages for each API endpoint:

- Create Inviter: `/test-inviter-api`
- User Registration: `/test-register-api`
- Post Tracking: `/test-post-api`
- Payment Status: `/test-payment-api`

These test pages can be used to verify the functionality of each component of the referral system.
