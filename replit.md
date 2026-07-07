# ClipZ — Short Video Social App

A TikTok / Instagram Reels-style mobile app built with Expo. Users can sign up, upload short videos, scroll an infinite vertical feed, like and comment, follow creators, and receive real-time notifications. Powered by Firebase Auth, Firestore, and Storage.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm --filter @workspace/mobile run dev` — run the Expo dev server
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Mobile: Expo SDK 54, Expo Router (file-based routing), React Native
- Backend/DB: Firebase Auth, Firestore, Firebase Storage
- State: React Query (@tanstack/react-query) + React Context
- Video: expo-video
- API: Express 5 (shared api-server)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/mobile/` — Expo mobile app
  - `app/(auth)/` — Login + Register screens
  - `app/(tabs)/` — Feed, Discover, Upload, Notifications, Profile tabs
  - `app/profile/[id].tsx` — Dynamic user profile
  - `app/setup.tsx` — Firebase setup guide (shown when env vars are missing)
  - `components/VideoCard.tsx` — Full-screen vertical video card with like/comment/share
  - `components/CommentModal.tsx` — Comments bottom sheet
  - `components/VideoGrid.tsx` — 3-column grid for profile views
  - `context/AuthContext.tsx` — Firebase auth state provider
  - `lib/firebase.ts` — Firebase initialization (conditional on env vars)
  - `lib/firestore.ts` — All Firestore CRUD helpers
  - `types/index.ts` — Shared TypeScript interfaces
  - `constants/colors.ts` — Dark theme design tokens (TikTok-style)

## Architecture decisions

- Firebase is initialized lazily — only when all 6 `EXPO_PUBLIC_FIREBASE_*` env vars are present. Missing vars show the setup screen instead of crashing.
- Auth state is managed via `AuthContext` using `onAuthStateChanged` with `AsyncStorage` persistence.
- The video feed uses a `FlatList` with `pagingEnabled` + `snapToInterval=SCREEN_HEIGHT` for TikTok-style vertical scrolling.
- `expo-video`'s `VideoView` handles playback; each card auto-plays when active and pauses when scrolled away.
- Notifications use a real-time Firestore `onSnapshot` listener scoped to the current user.

## Required Environment Variables (Replit Secrets)

Set these in the Replit Secrets panel (lock icon in sidebar):

- `EXPO_PUBLIC_FIREBASE_API_KEY`
- `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN`
- `EXPO_PUBLIC_FIREBASE_PROJECT_ID`
- `EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET`
- `EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
- `EXPO_PUBLIC_FIREBASE_APP_ID`

Get them from: Firebase Console → Project Settings → Your Apps → Web → Config

Also enable in Firebase Console:
- Authentication → Email/Password provider
- Firestore Database (start in test mode)
- Storage

## Product

- **Feed** — Infinite vertical video feed with auto-play; double-tap to like
- **Discover** — Search creators by username; trending hashtags
- **Upload** — Pick video from library → add caption + hashtags → upload to Firebase Storage → appears in feed
- **Notifications** — Real-time like / comment / follow notifications via Firestore listener
- **Profile** — Avatar (changeable), stats, video grid; own profile + other users' profiles
- **Follow system** — Follow/unfollow with real-time follower counts

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Firebase must be initialized before any Firestore/Auth/Storage call — all guarded by `isFirebaseConfigured`.
- expo-video must be pinned to `~3.0.16` for Expo SDK 54 compatibility.
- Do NOT use the `uuid` package — use `Date.now().toString() + Math.random().toString(36).slice(2)` instead.
- After adding Firebase env vars, restart the Expo workflow for them to take effect.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
