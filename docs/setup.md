# MyGunj Setup Guide

## Prerequisites

- Node.js 18.0.0 or higher
- npm 9.0.0 or higher (or yarn)
- Git

### Optional (for mobile development)
- Android SDK (API level 21+)
- Android Studio or Android emulator
- Expo CLI: `npm install -g expo-cli`
- Expo Go app (on physical device)

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/himanshupandey91/Mygunj.git
cd Mygunj
```

### 2. Install Backend Dependencies

```bash
cd backend
npm install
cp .env.example .env
```

Edit `.env` with your configuration:
```bash
API_PORT=3000
NODE_ENV=development
JWT_SECRET=your-development-secret-key
DB_PATH=./data/mygunj.db
```

### 3. Install Web Dependencies

```bash
cd ../web
npm install
cp .env.example .env
```

Edit `.env`:
```bash
REACT_APP_API_URL=http://localhost:3000
REACT_APP_WS_URL=ws://localhost:3000
```

### 4. Install Mobile Dependencies

```bash
cd ../mobile
npm install
cp .env.example .env
```

Edit `.env`:
```bash
EXPO_PUBLIC_API_URL=http://localhost:3000
EXPO_PUBLIC_WS_URL=ws://localhost:3000
```

## Running Locally

### Start Backend

```bash
cd backend
npm run dev
```

Server will be available at `http://localhost:3000`

### Start Web Client

In a new terminal:
```bash
cd web
npm start
```

Web app will open at `http://localhost:3001` (or next available port)

### Start Mobile Client (Expo)

In a new terminal:
```bash
cd mobile
npm start
```

Then:
- Press `a` to run on Android emulator
- Press `i` to run on iOS simulator (macOS only)
- Press `w` to run on web
- Scan QR code with Expo Go app on physical device

## Building for Production

### Web Build

```bash
cd web
npm run build
```

Output in `web/dist/` - deploy to any static host.

### Android APK (via Expo)

```bash
cd mobile
npx eas-cli build --platform android --local
```

Requires EAS account and local Android build setup.

### Backend Production

```bash
cd backend
npm run build
NODE_ENV=production node dist/server.js
```

## Testing

### Run Backend Tests

```bash
cd backend
npm test
```

### Run Web Tests

```bash
cd web
npm test
```

### Run Mobile Tests

```bash
cd mobile
npm test
```

## Troubleshooting

### Port Already in Use

If port 3000 is in use:
```bash
# Change in backend/.env
API_PORT=3001
```

### Database Lock (SQLite)

Delete the database and restart:
```bash
rm backend/data/mygunj.db
cd backend && npm run dev
```

### WebSocket Connection Failed

Ensure backend is running and check `WS_URL` in frontend `.env`

### Expo Android Issues

```bash
# Clear cache and reinstall
cd mobile
rm -rf node_modules
npm install
npm start -- --clear
```

## Environment Variables

See `.env.example` files in each directory.

### Backend (.env)
- `API_PORT`: Server port (default: 3000)
- `NODE_ENV`: Environment (development/production)
- `JWT_SECRET`: Secret key for signing tokens
- `DB_PATH`: SQLite database file path
- `CORS_ORIGIN`: Allowed origins for CORS

### Web (.env)
- `REACT_APP_API_URL`: Backend API base URL
- `REACT_APP_WS_URL`: WebSocket URL

### Mobile (.env)
- `EXPO_PUBLIC_API_URL`: Backend API base URL
- `EXPO_PUBLIC_WS_URL`: WebSocket URL

## Development Workflow

1. Create a feature branch: `git checkout -b feature/my-feature`
2. Make changes
3. Test locally (all three apps)
4. Commit with clear message: `git commit -m "Add feature: description"`
5. Push to GitHub: `git push origin feature/my-feature`
6. Open a Pull Request

## Code Style

- Use TypeScript for type safety
- Follow existing patterns in codebase
- Format with Prettier (configured in each app)
- Use ESLint rules (configured)

## Documentation

- API docs: see `docs/api.md`
- Architecture: see `docs/architecture.md`
- This file: general setup and running instructions
