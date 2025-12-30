# Smart Issue Board

A production-ready web application for managing issues with smart similarity detection, built with React, TypeScript, Firebase Authentication, and Firestore.

## Features

- **User Authentication**: Email/password sign up and login with Firebase Auth
- **Smart Issue Detection**: Automatically detects similar issues based on title and description keywords before creation
- **Issue Management**: Create issues with title, description, priority (Low/Medium/High), and status (Open, In Progress, Done)
- **Status Validation**: Enforces workflow rule - issues cannot move directly from Open to Done
- **Filtering**: Filter issues by status and priority
- **Real-time Updates**: Uses Firestore real-time listeners for instant updates
- **User-specific Data**: Each user only sees their own issues

## Tech Stack

- React 18
- TypeScript
- Firebase (Auth + Firestore)
- Tailwind CSS
- Vite
- Lucide React (icons)

## Firestore Schema

### Collection: `issues`

```
{
  id: string (auto-generated document ID)
  title: string
  description: string
  priority: 'Low' | 'Medium' | 'High'
  status: 'Open' | 'In Progress' | 'Done'
  userId: string (Firebase Auth UID)
  createdAt: timestamp
}
```

### Security Rules

The Firestore security rules ensure users can only access their own issues:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /issues/{issueId} {
      allow read, write: if request.auth != null && request.auth.uid == resource.data.userId;
      allow create: if request.auth != null && request.auth.uid == request.resource.data.userId;
    }
  }
}
```

### Required Index

Create a composite index in Firestore with these fields:
- Collection: `issues`
- Fields:
  - `userId` (Ascending)
  - `createdAt` (Descending)
  - `__name__` (Descending)

## Environment Variables

Create a `.env` file in the root directory with your Firebase configuration:

```env
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
```

## Local Development

1. Install dependencies:
```bash
npm install
```

2. Create `.env` file with Firebase credentials

3. Start development server:
```bash
npm run dev
```

4. Build for production:
```bash
npm run build
```

## Deployment to Vercel

### Step 1: Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin your-github-repo-url
git push -u origin main
```

### Step 2: Deploy on Vercel

1. Go to [vercel.com](https://vercel.com) and sign in
2. Click "Add New Project"
3. Import your GitHub repository
4. Configure environment variables in Vercel dashboard:
   - Add all `VITE_FIREBASE_*` variables from your `.env` file
5. Click "Deploy"

### Step 3: Configure Firebase

In your Firebase console:
1. Go to Authentication > Settings > Authorized domains
2. Add your Vercel deployment domain (e.g., `your-app.vercel.app`)

## Key Features Explained

### Smart Similarity Detection

The app analyzes issue titles and descriptions to find similar existing issues:
- Extracts keywords (words longer than 3 characters)
- Compares with existing issues using keyword matching
- Shows warning with similar issues if 30%+ similarity is detected
- Allows user to proceed anyway or cancel

### Status Transition Validation

Enforces a proper workflow:
- Issues start as "Open" by default
- Cannot skip from "Open" directly to "Done"
- Must go: Open → In Progress → Done
- Shows friendly error message if validation fails

### Real-time Updates

Uses Firestore's `onSnapshot` to listen for changes:
- Instant updates when issues are created or modified
- No page refresh needed
- Efficient data synchronization

## Project Structure

```
src/
├── components/
│   ├── Auth.tsx              # Login/signup component
│   ├── CreateIssue.tsx       # Issue creation with similarity check
│   └── IssueList.tsx         # Issue list with filters
├── contexts/
│   └── AuthContext.tsx       # Authentication state management
├── lib/
│   └── firebase.ts           # Firebase configuration
├── types/
│   └── issue.ts              # TypeScript types
├── utils/
│   └── similarity.ts         # Similarity detection logic
├── App.tsx                   # Main application component
└── main.tsx                  # Application entry point
```

## License

MIT
