# Clearword - Bible Study & Learning App

A comprehensive mobile application for Bible study, scripture memorization, and faith-based learning through interactive quizzes, flashcards, and community engagement.

## 🌟 Features

### 📚 Core Study Features

**Flashcard System**
- 800+ Bible flashcards across multiple categories (Bible Characters, Commandments, Parables, Miracles, etc.)
- Spaced repetition learning system
- Card mastery tracking with visual progress indicators
- Multiple study modes:
  - Random Mix (excludes mastered cards)
  - Review Queue (cards marked for review)
  - Topic-based learning
  - Favorites collection
- Interactive card flipping with animations
- Scripture lookup integration with Bible API

**Quiz System**
- Dynamic quiz generation from flashcard database
- Daily rotating quiz cards (5 per day)
- Perfect score tracking with XP rewards
- Difficulty levels: Newcomer, Scholar, Expert
- Category-based quizzes (automatically counted from database)
- Real-time scoring and performance feedback
- Milestone celebrations with confetti animations

**Bible Study Tools**
- Daily Prayer modal with rotating prayers
- Scripture references with tap-to-read functionality
- Prayer history tracking
- Hourly rotating Bible quotes on home screen
- Integration with Bible API for full verse text

### 🎯 Gamification & Progress

**XP & Leveling System**
- Earn XP through:
  - Quiz completions (10-50 XP based on performance)
  - Perfect quiz scores (100 XP bonus)
  - Daily study streaks
  - Friend referrals (100 XP)
  - Milestone achievements (50 XP)
- Rank progression: Newcomer → Scholar → Sage → Prophet → Apostle
- Visual rank badges with colors

**Streak Tracking**
- Daily study streak counter
- Weekly activity calendar with visual indicators
- Activity rings showing:
  - Cards studied
  - Quizzes completed
  - Minutes spent learning
- Streak reminder notifications

**Leaderboard & Competition**
- Friends-based leaderboard with XP rankings
- Top 50 global leaderboard
- Battle system: Challenge friends to head-to-head quizzes
- Real-time XP updates
- Podium view with confetti for top 3

### 👥 Social Features

**Friends System**
- Add friends by username search
- Friend request management
- Friends-only leaderboard
- Challenge friends to quiz battles
- See friend progress and achievements

**Referral System**
- Unique 8-character referral codes per user
- One-tap code sharing via native share sheet
- Automatic rewards:
  - Both users get 100 XP on signup
  - Inviter gets 50 XP when friend completes first quiz
- Referral tracking dashboard
- List of invited friends with reward status

### 📊 Analytics & Insights

**Comprehensive Stats Dashboard**
- Weekly activity overview
- Study time tracking (7-day average)
- Quiz completion metrics
- Mastery rate (cards mastered / total cards)
- Current streak & best streak
- Total XP and rank display
- Interactive charts:
  - Weekly XP trend line chart
  - Topic mastery radar chart (6 categories)
  - Activity rings

**Achievements System**
- Rank-based badges (Bronze, Silver, Gold, Diamond)
- Visual achievement cards with blur effects for locked achievements
- Milestone tracking
- Badge collection display

### 💬 AI Chat Assistant

**Biblical Q&A**
- AI-powered chat for Bible questions
- Context-aware responses
- Conversation history
- Copy responses to clipboard
- Conversation embeddings for semantic search
- Supabase vector database integration

### ⚙️ User Profile & Settings

**Profile Management**
- Avatar upload and display
- Display name customization
- Email management
- Profile statistics
- Account creation date

**Notification Preferences**
- General notifications toggle
- Daily learning reminders
- Progress updates
- Challenge alerts
- Achievement notifications
- Leaderboard updates
- Security alerts
- Customizable notification times

**App Preferences**
- Sound effects toggle
- Daily reminder scheduling
- Reminder time picker
- First day of week customization
- Time format (12/24-hour or system default)
- Day reset time configuration

**Account & Security**
- Change password (with validation)
- Email verification with resend option
- Privacy controls:
  - Share progress with friends
  - Show in leaderboard
  - Allow friend requests
- Delete account option (with double confirmation)

**Feedback & Support**
- In-app feedback form
- Star rating (1-5 stars)
- Detailed feedback text
- Device info collection (for bug reports)
- Feedback history viewing

### 🎨 UI/UX Features

**Beautiful Design**
- Consistent blue gradient backgrounds across screens
- Glassmorphism effects on cards
- Smooth animations and transitions
- Haptic feedback
- Dark mode support (system-based)
- Custom navigation with animated icons
- Bouncy button interactions
- Apple-inspired design language

**Navigation**
- Bottom tab navigation with custom icons
- Screen transitions with animations
- Deep linking support
- Gesture-based interactions (swipe quiz cards)

## 🛠️ Tech Stack

**Frontend**
- React Native (Expo)
- TypeScript
- React Native Reanimated (animations)
- React Native Gesture Handler
- Expo Linear Gradient
- Phosphor Icons
- Victory Charts (analytics visualization)

**Backend & Database**
- Supabase (PostgreSQL)
- Row Level Security (RLS) policies
- Real-time subscriptions
- Edge Functions for serverless logic
- Vector database for AI embeddings

**External APIs**
- Bible API (bible-api.com) for scripture lookups
- OpenAI GPT-4 for chat assistant

**Storage & Media**
- Supabase Storage for avatars
- AsyncStorage for local preferences
- Expo Audio for sound effects

## 📁 Project Structure

```
Clearword.app/
├── App.tsx                      # Main app entry & navigation
├── src/
│   ├── components/
│   │   ├── home/
│   │   │   ├── StackedQuizCards.tsx
│   │   │   ├── WeeklyStreakCalendar.tsx
│   │   │   └── TodaysPrayerModal.tsx
│   │   └── ui/
│   │       ├── BouncyButton.tsx
│   │       └── CustomIcons.tsx
│   ├── screens/
│   │   ├── NewHomeScreen.tsx
│   │   ├── StudyScreen.tsx
│   │   ├── FlashcardScreen.tsx
│   │   ├── QuizScreen.tsx
│   │   ├── FriendsScreen.tsx
│   │   ├── InsightScreen.tsx
│   │   ├── ProfileScreen.tsx
│   │   ├── ChatScreen.tsx
│   │   ├── NotificationScreen.tsx
│   │   ├── PreferencesScreen.tsx
│   │   ├── AccountSecurityScreen.tsx
│   │   ├── InviteFriendsScreen.tsx
│   │   ├── RateUsScreen.tsx
│   │   └── ...
│   ├── services/
│   │   ├── auth.ts
│   │   ├── flashcards.ts
│   │   ├── cardTracking.ts
│   │   ├── quizService.ts
│   │   ├── userProgress.ts
│   │   ├── friendsService.ts
│   │   ├── profileService.ts
│   │   ├── referralService.ts
│   │   ├── feedbackService.ts
│   │   ├── chatService.ts
│   │   ├── prayerService.ts
│   │   └── soundEffects.ts
│   ├── config/
│   │   └── supabase.ts
│   └── types/
│       └── database.ts
├── database/
│   ├── flashcards.sql
│   ├── user_progress.sql
│   ├── referrals.sql
│   ├── app_feedback.sql
│   └── ...
└── assets/
    ├── sounds/
    └── images/

```

## 🗄️ Database Schema

**Core Tables**
- `flashcards` - Bible study flashcards with categories
- `card_progress` - User progress tracking per card
- `user_progress` - Overall user statistics
- `user_daily_activity` - Daily activity tracking
- `quiz_attempts` - Quiz completion records
- `profiles` - User profile information
- `friendships` - Friend relationships
- `friend_requests` - Pending friend requests
- `challenges` - Quiz battle challenges
- `challenge_results` - Battle outcomes
- `referrals` - Referral tracking
- `app_feedback` - User feedback submissions
- `chat_conversations` - AI chat history
- `conversation_embeddings` - Vector embeddings for chat
- `saved_prayers` - User's saved prayers

**Security**
- Row Level Security (RLS) enabled on all tables
- Secure functions with `SET search_path = public`
- Optimized policies using subqueries

## 🚀 Setup Instructions

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Clearword.app
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure Supabase**
   - Create a Supabase project
   - Copy `.env.example` to `.env`
   - Add your Supabase URL and anon key
   - Run database migrations from `/database` folder

4. **Run the app**
   ```bash
   # iOS
   npx expo run:ios
   
   # Android
   npx expo run:android
   
   # Web
   npm start
   ```

## 📊 Database Setup

Run the SQL files in this order:
1. `database/flashcards.sql` - Core flashcard data
2. `database/user_progress.sql` - User tracking tables
3. `database/referrals.sql` - Referral system
4. `database/app_feedback.sql` - Feedback management

## 🎯 Recent Updates

**Referral System** (Latest)
- Unique code generation
- Native share integration
- Automatic XP rewards
- Referral tracking dashboard

**Account & Security**
- Password management
- Email verification
- Privacy controls
- Account deletion

**Preferences Management**
- Sound effects toggle
- Notification preferences
- Time format customization
- Day/week settings

**Prayer System**
- Daily rotating prayers
- Prayer history tracking
- Save favorite prayers
- Scripture integration

**Feedback System**
- Star ratings
- Detailed feedback forms
- Device info collection
- Feedback management

## 📝 License

Copyright © 2024 Clearword. All rights reserved.

## 🤝 Contributing

This is a private project. Contact the owner for contribution guidelines.

## 📧 Support

For support, email support@clearword.app or use the in-app feedback form.

---

**Built with ❤️ for Bible study and faith-based learning**
