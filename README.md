# KindCampus

A modern web application designed to make campus life easier and more fun through peer skill-swapping and online practical assignments.

## Features

### 🎯 Peer Skill-Swapping & Team-Up (KindCollab)
- **Post Skills**: Share what you can teach or what you need help with
- **Browse & Connect**: Find study buddies and form teams for projects
- **Instant Meetings**: Get Google Meet links automatically generated
- **AI Icebreakers**: AI-generated conversation starters for better connections
- **Smart Matching**: Find perfect matches based on skills and availability

### 📚 Online Practical Assignments (KindTasks)
- **Code Challenges**: Write and test code with instant feedback
- **Design Tasks**: Submit creative work with detailed specifications
- **Interactive Quizzes**: Multiple-choice questions with immediate grading
- **AI-Powered Hints**: Get helpful suggestions when you're stuck
- **Live Dashboard**: Instructors can monitor progress in real-time

## Tech Stack

- **Frontend**: Next.js 15, React 18, TypeScript
- **Styling**: Tailwind CSS, shadcn/ui components
- **Backend**: Firebase (Firestore, Authentication, Cloud Functions)
- **AI**: Google Gemini API for intelligent interactions
- **Animations**: Framer Motion
- **Icons**: Lucide React

## Getting Started

### Prerequisites

- Node.js 18+ 
- npm or yarn
- Firebase project
- Google Gemini API key

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd kindcampus
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   Create a `.env.local` file in the root directory:
   ```env
   # Firebase Configuration
   NEXT_PUBLIC_FIREBASE_API_KEY=your_firebase_api_key
   NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
   NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
   NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
   NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123456789
   NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id

   # Gemini API
   NEXT_PUBLIC_GEMINI_API_KEY=AIzaSyCL-wB_Ym_40vV17e1gFhyyL-o2864KQN8
   ```

4. **Set up Firebase**
   - Create a new Firebase project
   - Enable Authentication, Firestore, and Cloud Functions
   - Set up Firestore security rules
   - Configure authentication providers (Google, Email)

5. **Run the development server**
```bash
npm run dev
   ```

6. **Open your browser**
   Navigate to [http://localhost:3000](http://localhost:3000)

## Project Structure

```
kindcampus/
├── src/
│   ├── app/                    # Next.js app router
│   │   ├── page.tsx           # Home page
│   │   ├── kindcollab/        # Peer collaboration module
│   │   └── kindtasks/         # Online assignments module
│   ├── components/
│   │   ├── ui/                # shadcn/ui components
│   │   │   ├── shuffle-grid.tsx
│   │   │   └── ...
│   │   └── layout/
│   │       └── navbar.tsx
│   ├── lib/
│   │   ├── firebase.ts        # Firebase configuration
│   │   ├── gemini.ts          # Gemini API utilities
│   │   └── utils.ts           # Utility functions
│   └── types/
│       └── index.ts           # TypeScript interfaces
├── public/                     # Static assets
└── package.json
```

## Firebase Setup

### Firestore Collections

1. **users** - User profiles and preferences
2. **cards** - Skill-swapping posts and requests
3. **matches** - Connection requests and meeting details
4. **tasks** - Instructor-created assignments
5. **submissions** - Student task submissions

### Security Rules

```javascript
// Firestore security rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can read/write their own profile
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Anyone can read cards, authenticated users can create
    match /cards/{cardId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null && 
        request.auth.uid == resource.data.posterId;
    }
    
    // Matches require authentication
    match /matches/{matchId} {
      allow read, write: if request.auth != null;
    }
    
    // Tasks and submissions
    match /tasks/{taskId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'instructor';
    }
    
    match /submissions/{submissionId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

## Cloud Functions

### Match Creation
```javascript
exports.onMatchCreated = functions.firestore
  .document('matches/{matchId}')
  .onCreate(async (snap, context) => {
    const match = snap.data();
    
    // Generate Google Meet link
    const meetLink = await generateMeetLink();
    
    // Send notifications
    await sendNotifications(match, meetLink);
    
    // Generate icebreaker questions
    const questions = await generateIcebreakerQuestions(match);
    
    // Update match with meet link and questions
    await snap.ref.update({
      meetLink,
      icebreakerQuestions: questions
    });
  });
```

### Task Grading
```javascript
exports.gradeSubmission = functions.firestore
  .document('submissions/{submissionId}')
  .onCreate(async (snap, context) => {
    const submission = snap.data();
    const task = await getTask(submission.taskId);
    
    let passed = false;
    let hint = null;
    
    if (task.type === 'code') {
      const result = await runCodeTests(submission.code, task.testCases);
      passed = result.passed;
      if (!passed) {
        hint = await generateHint(task.type, task.description, submission.code);
      }
    } else if (task.type === 'quiz') {
      passed = gradeQuiz(submission.quizAnswers, task.quizQuestions);
    }
    
    await snap.ref.update({
      status: passed ? 'passed' : 'failed',
      hint,
      gradedAt: new Date()
    });
  });
```

## Deployment

### Vercel (Recommended)

1. **Connect your repository**
   - Push your code to GitHub
   - Connect your repository to Vercel

2. **Configure environment variables**
   - Add all environment variables in Vercel dashboard

3. **Deploy**
   - Vercel will automatically deploy on push to main branch

### Firebase Hosting

1. **Build the project**
   ```bash
   npm run build
   ```

2. **Deploy to Firebase**
   ```bash
   firebase deploy
   ```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, email support@kindcampus.com or create an issue in this repository.

---

Built with ❤️ for better campus experiences
