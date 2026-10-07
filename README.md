# MyGunj

A modern, open-source messaging application for Android and Web.

## Overview

MyGunj is a lightweight messaging platform built with React Native (Expo) for mobile, React for web, and Express.js for the backend. It focuses on reliability, security, and a clean user experience.

## Project Structure

```
MyGunj/
├── backend/              # Express.js server
│   ├── src/
│   │   ├── server.ts
│   │   ├── auth/         # Authentication
│   │   ├── users/        # User management
│   │   ├── conversations/# Chat functionality
│   │   ├── messages/     # Message handling
│   │   ├── ws/           # WebSocket logic
│   │   ├── middleware/   # Express middleware
│   │   ├── db/           # Database layer
│   │   └── utils/        # Utilities
│   ├── package.json
│   ├── tsconfig.json
│   └── .env.example
├── web/                  # React web app
│   ├── src/
│   │   ├── App.tsx
│   │   ├── pages/        # Page components
│   │   ├── components/   # Reusable UI components
│   │   ├── hooks/        # Custom hooks
│   │   ├── services/     # API client
│   │   ├── store/        # State management
│   │   └── styles/       # Styling
│   ├── public/
│   ├── package.json
│   └── tsconfig.json
├── mobile/               # React Native (Expo) app
│   ├── app/              # Expo Router
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── screens/
│   │   ├── services/
│   │   └── store/
│   ├── app.json          # Expo config
│   ├── package.json
│   └── tsconfig.json
└── docs/                 # Documentation
    ├── architecture.md
    ├── api.md
    └── setup.md
```

## Core Features

### Authentication
- Phone number based registration/login
- OTP verification (architecture-ready)
- Session management
- Protected routes/screens

### Messaging
- One-to-one text messaging
- Real-time WebSocket updates
- Message delivery states (sending, sent, delivered, read, failed)
- Message history and persistence
- Typing indicators

### User Management
- User profiles with display names and photos
- User search and discovery
- Contact list
- Block/unblock users

### UI/UX
- Clean, modern interface
- Responsive web layout
- Android-optimized mobile layout
- Accessibility considerations
- Connection status indicators

## Quick Start

### Prerequisites
- Node.js 18+
- npm or yarn
- For Android: Android SDK, Android Studio, or Expo Go app

### Backend Setup

```bash
cd backend
npm install
cp .env.example .env
# Edit .env with your configuration
npm run dev
```

### Web Development

```bash
cd web
npm install
cp .env.example .env
npm start
```

### Mobile Development (Expo)

```bash
cd mobile
npm install
cp .env.example .env
npm start
```

Then:
- Press `a` for Android (requires Android emulator or device)
- Press `i` for iOS (macOS only)
- Press `w` for Web
- Or scan QR code with Expo Go app

## Architecture

### Backend
- **Framework**: Express.js
- **Language**: TypeScript
- **Database**: SQLite (development), upgradeable to PostgreSQL
- **Real-time**: WebSocket (ws library)
- **Authentication**: JWT + Session tokens

### Frontend
- **Web**: React 18 + TypeScript + Vite
- **Mobile**: React Native + Expo + TypeScript
- **State**: Context API + hooks
- **Styling**: CSS/CSS-in-JS

## Development

### Running Tests

```bash
# Backend tests
cd backend
npm test

# Web tests
cd web
npm test

# Mobile tests
cd mobile
npm test
```

### Build for Production

```bash
# Web
cd web
npm run build

# Mobile (Android APK)
cd mobile
eas build --platform android
```

## Environment Variables

See `.env.example` files in each directory for required configuration.

Key variables:
- `SERVER_URL`: Backend API URL
- `WS_URL`: WebSocket URL
- `API_PORT`: Backend port (default: 3000)
- `DB_PATH`: Database file path (SQLite)

## Security

- All credentials are environment-based, never hardcoded
- Authentication tokens are short-lived (15 minutes)
- Refresh tokens for session persistence
- User data is validated server-side
- Authorization checks on all protected endpoints
- WebSocket messages authenticated
- No sensitive data in logs

## Contributing

1. Create a feature branch
2. Make changes with clear commits
3. Test thoroughly
4. Submit for review

## License

MIT License - See LICENSE file for details

## Status

🚧 **In Development** - Core messaging functionality being implemented

## Roadmap

- [x] Project structure
- [ ] Backend API foundation
- [ ] Authentication system
- [ ] User profiles
- [ ] One-to-one messaging
- [ ] Web UI
- [ ] Mobile UI
- [ ] Real-time WebSocket
- [ ] Message delivery states
- [ ] Offline handling
- [ ] Search functionality
- [ ] Settings/Privacy
- [ ] Testing suite
- [ ] Documentation
- [ ] Production deployment
