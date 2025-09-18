# 🖥️ Frontend Applications Guide

Complete guide to ShareUpTime's web and mobile frontend applications

## 📋 Table of Contents

1. [Frontend Architecture Overview](#frontend-architecture-overview)
2. [Next.js Web Application](#nextjs-web-application)
3. [React Native Mobile Application](#react-native-mobile-application)
4. [Component Libraries](#component-libraries)
5. [State Management](#state-management)
6. [API Integration](#api-integration)
7. [Development Workflow](#development-workflow)

---

## 🏗️ Frontend Architecture Overview

ShareUpTime uses a **multi-platform frontend architecture** with shared components and services:

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interfaces                         │
├───────────────────────────┬─────────────────────────────────┤
│     Web Application       │        Mobile Application       │
│    (Next.js 15 + TS)      │     (React Native + TS)        │
│                           │                                 │
│  ┌─────────────────────┐  │  ┌─────────────────────────────┐ │
│  │   App Directory     │  │  │      Screen Components      │ │
│  │   ├── page.tsx     │  │  │      ├── Home               │ │
│  │   ├── layout.tsx   │  │  │      ├── Profile            │ │
│  │   └── components/  │  │  │      ├── Feed               │ │
│  └─────────────────────┘  │  │      └── Messages           │ │
│                           │  └─────────────────────────────┘ │
├───────────────────────────┴─────────────────────────────────┤
│                    Shared Layer                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   API Services  │  │   Utils & Hooks │  │   Types      │ │
│  │   ├── auth      │  │   ├── useShare  │  │   ├── User   │ │
│  │   ├── posts     │  │   ├── useSwap   │  │   ├── Post   │ │
│  │   └── users     │  │   └── useAuth   │  │   └── Feed   │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Backend API Layer                        │
│           API Gateway (localhost:3000)                      │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack

**Web Frontend (Next.js)**
- **Framework**: Next.js 15 with App Directory
- **Language**: TypeScript
- **Styling**: Tailwind CSS + CSS Modules
- **State**: React Context + Zustand
- **Testing**: Jest + React Testing Library
- **Build**: Webpack 5 + SWC

**Mobile Frontend (React Native)**
- **Framework**: React Native 0.72+
- **Language**: TypeScript
- **Navigation**: React Navigation 6
- **State**: Redux Toolkit + RTK Query
- **Testing**: Jest + Detox
- **Build**: Metro Bundler

---

## 💻 Next.js Web Application

### Project Structure

```
shareuptime-frontend/
├── src/
│   ├── app/                      # Next.js 15 App Directory
│   │   ├── layout.tsx           # Root layout
│   │   ├── page.tsx             # Home page
│   │   ├── login/page.tsx       # Login page
│   │   ├── register/page.tsx    # Register page
│   │   ├── dashboard/page.tsx   # Dashboard
│   │   ├── profile/page.tsx     # User profile
│   │   ├── feed/page.tsx        # Social feed
│   │   ├── messages/page.tsx    # Messaging
│   │   ├── groups/page.tsx      # Groups
│   │   ├── stories/page.tsx     # Stories
│   │   ├── reels/page.tsx       # Reels
│   │   └── swap/page.tsx        # Swap feature
│   ├── components/              # Reusable components
│   │   ├── ui/                  # Base UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   └── Card.tsx
│   │   ├── layout/              # Layout components
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── Navigation.tsx
│   │   ├── auth/                # Authentication
│   │   │   ├── LoginForm.tsx
│   │   │   └── RegisterForm.tsx
│   │   ├── feed/                # Feed components
│   │   │   ├── PostCard.tsx
│   │   │   ├── PostCreator.tsx
│   │   │   └── FeedContainer.tsx
│   │   ├── profile/             # Profile components
│   │   ├── chat/                # Chat components
│   │   └── media/               # Media components
│   ├── services/                # API service layer
│   │   ├── api.ts               # Base API configuration
│   │   ├── auth.service.ts      # Authentication API
│   │   ├── user.service.ts      # User management API
│   │   ├── post.service.ts      # Posts API
│   │   └── media.service.ts     # Media upload API
│   ├── hooks/                   # Custom React hooks
│   │   ├── useAuth.ts           # Authentication hook
│   │   ├── useShare.ts          # Share functionality
│   │   └── useSwap.ts           # Swap functionality
│   ├── store/                   # State management
│   │   ├── authSlice.ts         # Auth state
│   │   └── index.ts             # Store configuration
│   ├── lib/                     # Utility libraries
│   │   ├── utils.ts             # General utilities
│   │   ├── auth.ts              # Auth utilities
│   │   └── api.ts               # API utilities
│   └── styles/                  # Global styles
│       ├── globals.css          # Global CSS
│       └── components.css       # Component styles
├── public/                      # Static assets
├── next.config.js               # Next.js configuration
├── tailwind.config.js           # Tailwind CSS config
├── tsconfig.json                # TypeScript config
└── package.json                 # Dependencies
```

### Key Features Implementation

#### 1. Authentication System

**Login Component**:
```typescript
// src/components/auth/ShareupLoginForm.tsx
import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { authService } from '@/services/auth.service';

export default function ShareupLoginForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      const response = await authService.login(formData);
      localStorage.setItem('token', response.token);
      router.push('/dashboard');
    } catch (error) {
      console.error('Login failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
        className="w-full p-3 border rounded-lg"
        required
      />
      <input
        type="password"
        placeholder="Password"
        value={formData.password}
        onChange={(e) => setFormData({...formData, password: e.target.value})}
        className="w-full p-3 border rounded-lg"
        required
      />
      <button 
        type="submit" 
        disabled={loading}
        className="w-full bg-blue-500 text-white p-3 rounded-lg"
      >
        {loading ? 'Signing In...' : 'Sign In'}
      </button>
    </form>
  );
}
```

#### 2. Feed System

**Feed Component**:
```typescript
// src/components/feed/ShareupFeed.tsx
import { useEffect, useState } from 'react';
import { postService } from '@/services/post.service';
import PostCard from './PostCard';

interface Post {
  id: string;
  content: string;
  author: {
    id: string;
    name: string;
    avatar: string;
  };
  createdAt: string;
  likes: number;
  comments: number;
}

export default function ShareupFeed() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);
  const [page, setPage] = useState(1);

  useEffect(() => {
    loadFeed();
  }, [page]);

  const loadFeed = async () => {
    try {
      const response = await postService.getFeed(page);
      setPosts(prev => page === 1 ? response.posts : [...prev, ...response.posts]);
    } catch (error) {
      console.error('Failed to load feed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-2xl mx-auto space-y-6">
      {posts.map(post => (
        <PostCard key={post.id} post={post} />
      ))}
      
      {loading && (
        <div className="flex justify-center p-4">
          <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500" />
        </div>
      )}
      
      <button 
        onClick={() => setPage(p => p + 1)}
        className="w-full p-3 text-blue-500 border border-blue-500 rounded-lg hover:bg-blue-50"
      >
        Load More Posts
      </button>
    </div>
  );
}
```

#### 3. Real-time Features

**WebSocket Integration**:
```typescript
// src/hooks/useWebSocket.ts
import { useEffect, useRef } from 'react';

export function useWebSocket(url: string, onMessage: (data: any) => void) {
  const ws = useRef<WebSocket | null>(null);

  useEffect(() => {
    ws.current = new WebSocket(url);
    
    ws.current.onmessage = (event) => {
      const data = JSON.parse(event.data);
      onMessage(data);
    };

    ws.current.onclose = () => {
      // Reconnect logic
      setTimeout(() => {
        ws.current = new WebSocket(url);
      }, 3000);
    };

    return () => {
      ws.current?.close();
    };
  }, [url, onMessage]);

  const sendMessage = (message: any) => {
    if (ws.current?.readyState === WebSocket.OPEN) {
      ws.current.send(JSON.stringify(message));
    }
  };

  return { sendMessage };
}
```

### Running the Web Application

#### Development Setup

1. **Install Dependencies**:
```bash
cd shareuptime-frontend
npm install
```

2. **Environment Configuration**:
```env
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_WS_URL=ws://localhost:3007
NEXT_PUBLIC_APP_NAME=ShareUpTime
NEXT_PUBLIC_VERSION=1.0.0
```

3. **Start Development Server**:
```bash
npm run dev
```

4. **Access the Application**:
- Web App: `http://localhost:3007`
- Admin Panel: `http://localhost:3007/admin`

#### Build for Production

```bash
# Build the application
npm run build

# Start production server
npm start

# Or export static files
npm run export
```

---

## 📱 React Native Mobile Application

### Project Structure

```
ShareUpTimeMobile/
├── src/
│   ├── screens/                 # Screen components
│   │   ├── auth/
│   │   │   ├── LoginScreen.tsx
│   │   │   └── RegisterScreen.tsx
│   │   ├── main/
│   │   │   ├── HomeScreen.tsx
│   │   │   ├── FeedScreen.tsx
│   │   │   ├── ProfileScreen.tsx
│   │   │   ├── MessagesScreen.tsx
│   │   │   └── SettingsScreen.tsx
│   │   ├── social/
│   │   │   ├── StoriesScreen.tsx
│   │   │   ├── ReelsScreen.tsx
│   │   │   └── GroupsScreen.tsx
│   │   └── swap/
│   │       └── SwapScreen.tsx
│   ├── components/              # Reusable components
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Card.tsx
│   │   │   └── Modal.tsx
│   │   ├── navigation/
│   │   │   ├── TabNavigator.tsx
│   │   │   └── StackNavigator.tsx
│   │   ├── feed/
│   │   │   ├── PostCard.tsx
│   │   │   ├── PostActions.tsx
│   │   │   └── CommentsList.tsx
│   │   ├── camera/
│   │   │   ├── CameraView.tsx
│   │   │   └── MediaPicker.tsx
│   │   └── chat/
│   │       ├── MessageBubble.tsx
│   │       └── ChatInput.tsx
│   ├── services/                # API services
│   │   ├── api.ts
│   │   ├── authService.ts
│   │   ├── postService.ts
│   │   └── mediaService.ts
│   ├── navigation/              # Navigation setup
│   │   ├── AppNavigator.tsx
│   │   └── AuthNavigator.tsx
│   ├── store/                   # Redux store
│   │   ├── slices/
│   │   │   ├── authSlice.ts
│   │   │   ├── feedSlice.ts
│   │   │   └── userSlice.ts
│   │   └── store.ts
│   ├── utils/                   # Utility functions
│   │   ├── helpers.ts
│   │   ├── constants.ts
│   │   └── permissions.ts
│   └── assets/                  # Static assets
│       ├── images/
│       ├── icons/
│       └── fonts/
├── android/                     # Android native code
├── ios/                         # iOS native code
├── app.json                     # Expo configuration
├── metro.config.js              # Metro bundler config
├── package.json                 # Dependencies
└── tsconfig.json                # TypeScript config
```

### Key Mobile Features

#### 1. Navigation System

```typescript
// src/navigation/AppNavigator.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createStackNavigator } from '@react-navigation/stack';

const Tab = createBottomTabNavigator();
const Stack = createStackNavigator();

function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarStyle: { backgroundColor: '#1a1a1a' },
        tabBarActiveTintColor: '#007AFF',
        headerShown: false
      }}
    >
      <Tab.Screen name="Feed" component={FeedScreen} />
      <Tab.Screen name="Stories" component={StoriesScreen} />
      <Tab.Screen name="Camera" component={CameraScreen} />
      <Tab.Screen name="Messages" component={MessagesScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}

export default function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        <Stack.Screen name="MainTabs" component={MainTabs} />
        <Stack.Screen name="PostDetails" component={PostDetailsScreen} />
        <Stack.Screen name="UserProfile" component={UserProfileScreen} />
        <Stack.Screen name="EditProfile" component={EditProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

#### 2. Camera Integration

```typescript
// src/components/camera/ShareupCamera.tsx
import { Camera, CameraType } from 'expo-camera';
import { useState, useRef } from 'react';
import { View, TouchableOpacity, Image } from 'react-native';

export default function ShareupCamera() {
  const [type, setType] = useState(CameraType.back);
  const [permission, requestPermission] = Camera.useCameraPermissions();
  const cameraRef = useRef<Camera>(null);

  const takePicture = async () => {
    if (cameraRef.current) {
      const photo = await cameraRef.current.takePictureAsync({
        quality: 0.8,
        base64: true
      });
      
      // Upload to media service
      await mediaService.uploadPhoto(photo);
    }
  };

  const toggleCameraType = () => {
    setType(current => (
      current === CameraType.back ? CameraType.front : CameraType.back
    ));
  };

  return (
    <View style={{ flex: 1 }}>
      <Camera
        style={{ flex: 1 }}
        type={type}
        ref={cameraRef}
      >
        <View style={{ flex: 1, justifyContent: 'flex-end', alignItems: 'center', padding: 20 }}>
          <TouchableOpacity onPress={takePicture} style={styles.captureButton}>
            <View style={styles.captureInner} />
          </TouchableOpacity>
          
          <TouchableOpacity onPress={toggleCameraType} style={styles.flipButton}>
            <Image source={require('../../assets/flip-icon.png')} />
          </TouchableOpacity>
        </View>
      </Camera>
    </View>
  );
}
```

#### 3. Push Notifications

```typescript
// src/services/notificationService.ts
import * as Notifications from 'expo-notifications';
import { Platform } from 'react-native';

class NotificationService {
  async setupNotifications() {
    const { status } = await Notifications.requestPermissionsAsync();
    
    if (status === 'granted') {
      const token = await Notifications.getExpoPushTokenAsync();
      await this.registerToken(token.data);
    }

    // Configure notification behavior
    Notifications.setNotificationHandler({
      handleNotification: async () => ({
        shouldShowAlert: true,
        shouldPlaySound: true,
        shouldSetBadge: true,
      }),
    });
  }

  async registerToken(token: string) {
    // Send token to backend
    await fetch(`${API_BASE_URL}/notifications/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ token })
    });
  }

  async scheduleLocalNotification(title: string, body: string) {
    await Notifications.scheduleNotificationAsync({
      content: { title, body },
      trigger: { seconds: 1 }
    });
  }
}

export default new NotificationService();
```

### Running the Mobile Application

#### Development Setup

1. **Install Dependencies**:
```bash
cd ShareUpTimeMobile
npm install
```

2. **Configure Environment**:
```typescript
// src/config/settings.js
export const API_CONFIG = {
  BASE_URL: __DEV__ 
    ? 'http://localhost:3000' 
    : 'https://api.shareuptime.com',
  WS_URL: __DEV__ 
    ? 'ws://localhost:3007' 
    : 'wss://ws.shareuptime.com'
};
```

3. **Start Development**:
```bash
# Start Metro bundler
npx react-native start

# Run on Android
npx react-native run-android

# Run on iOS (macOS only)
npx react-native run-ios
```

#### Build for Production

**Android APK**:
```bash
cd android
./gradlew assembleRelease
```

**iOS Build (macOS only)**:
```bash
cd ios
xcodebuild -workspace ShareUpTimeMobile.xcworkspace -scheme ShareUpTimeMobile archive
```

---

## 🧩 Component Libraries

### Shared UI Components

Both web and mobile applications share design principles and component patterns:

#### Design System

```typescript
// src/styles/shareup-colors.ts
export const ShareUpColors = {
  primary: {
    50: '#eff6ff',
    500: '#3b82f6',
    600: '#2563eb',
    900: '#1e3a8a'
  },
  secondary: {
    50: '#f8fafc',
    500: '#64748b',
    900: '#0f172a'
  },
  success: '#10b981',
  warning: '#f59e0b',
  error: '#ef4444'
};

export const ShareUpSpacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32
};
```

#### Button Component (Web)

```typescript
// src/components/ui/ShareupButton.tsx
import { ButtonHTMLAttributes, ReactNode } from 'react';
import { ShareUpColors } from '@/styles/shareup-colors';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  children: ReactNode;
}

export default function ShareupButton({ 
  variant = 'primary',
  size = 'md',
  loading = false,
  children,
  className = '',
  ...props 
}: ButtonProps) {
  const baseClasses = 'font-medium rounded-lg transition-colors';
  
  const variantClasses = {
    primary: 'bg-blue-600 hover:bg-blue-700 text-white',
    secondary: 'bg-gray-600 hover:bg-gray-700 text-white',
    outline: 'border-2 border-blue-600 text-blue-600 hover:bg-blue-50'
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };

  return (
    <button
      className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${className}`}
      disabled={loading}
      {...props}
    >
      {loading ? (
        <span className="flex items-center">
          <svg className="animate-spin -ml-1 mr-2 h-4 w-4" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
          </svg>
          Loading...
        </span>
      ) : children}
    </button>
  );
}
```

#### Button Component (Mobile)

```typescript
// ShareUpTimeMobile/src/components/common/Button.tsx
import React from 'react';
import { TouchableOpacity, Text, StyleSheet, ActivityIndicator } from 'react-native';

interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
}

export default function Button({ 
  title, 
  onPress, 
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false 
}: ButtonProps) {
  const buttonStyle = [
    styles.button,
    styles[variant],
    styles[size],
    (disabled || loading) && styles.disabled
  ];

  return (
    <TouchableOpacity 
      style={buttonStyle}
      onPress={onPress}
      disabled={disabled || loading}
    >
      {loading ? (
        <ActivityIndicator color={variant === 'outline' ? '#3b82f6' : '#ffffff'} />
      ) : (
        <Text style={[styles.text, styles[`${variant}Text`]]}>{title}</Text>
      )}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  button: {
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center'
  },
  primary: {
    backgroundColor: '#3b82f6'
  },
  secondary: {
    backgroundColor: '#6b7280'
  },
  outline: {
    borderWidth: 2,
    borderColor: '#3b82f6',
    backgroundColor: 'transparent'
  },
  sm: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    minHeight: 32
  },
  md: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    minHeight: 40
  },
  lg: {
    paddingHorizontal: 24,
    paddingVertical: 12,
    minHeight: 48
  },
  disabled: {
    opacity: 0.5
  },
  text: {
    fontWeight: '600'
  },
  primaryText: {
    color: '#ffffff'
  },
  secondaryText: {
    color: '#ffffff'
  },
  outlineText: {
    color: '#3b82f6'
  }
});
```

---

## 🔄 State Management

### Web Application (Zustand)

```typescript
// src/store/authSlice.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (user: User, token: string) => void;
  logout: () => void;
  updateUser: (userData: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      
      login: (user, token) => set({ 
        user, 
        token, 
        isAuthenticated: true 
      }),
      
      logout: () => set({ 
        user: null, 
        token: null, 
        isAuthenticated: false 
      }),
      
      updateUser: (userData) => set((state) => ({
        user: state.user ? { ...state.user, ...userData } : null
      }))
    }),
    {
      name: 'auth-storage'
    }
  )
);
```

### Mobile Application (Redux Toolkit)

```typescript
// ShareUpTimeMobile/src/store/slices/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { authService } from '../../services/authService';

interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  token: null,
  isLoading: false,
  error: null
};

export const loginUser = createAsyncThunk(
  'auth/login',
  async (credentials: { email: string; password: string }) => {
    const response = await authService.login(credentials);
    return response;
  }
);

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    logout: (state) => {
      state.user = null;
      state.token = null;
    },
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.error.message || 'Login failed';
      });
  }
});

export const { logout, clearError } = authSlice.actions;
export default authSlice.reducer;
```

---

## 🌐 API Integration

### Shared API Service

```typescript
// src/services/api.ts (shared between web and mobile)
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000';

class ApiService {
  private baseURL: string;
  private token: string | null = null;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  setToken(token: string) {
    this.token = token;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers
      },
      ...options
    };

    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`API Error: ${response.status} ${response.statusText}`);
    }

    return response.json();
  }

  // Auth endpoints
  async login(credentials: { email: string; password: string }) {
    return this.request('/api/auth/login', {
      method: 'POST',
      body: JSON.stringify(credentials)
    });
  }

  async register(userData: { email: string; password: string; name: string }) {
    return this.request('/api/auth/register', {
      method: 'POST',
      body: JSON.stringify(userData)
    });
  }

  // Posts endpoints
  async getFeed(page = 1, limit = 20) {
    return this.request(`/api/posts/feed?page=${page}&limit=${limit}`);
  }

  async createPost(postData: { content: string; mediaUrls?: string[] }) {
    return this.request('/api/posts', {
      method: 'POST',
      body: JSON.stringify(postData)
    });
  }

  async likePost(postId: string) {
    return this.request(`/api/posts/${postId}/like`, {
      method: 'POST'
    });
  }

  // Users endpoints
  async getUserProfile(userId: string) {
    return this.request(`/api/users/${userId}`);
  }

  async updateProfile(profileData: any) {
    return this.request('/api/users/profile', {
      method: 'PUT',
      body: JSON.stringify(profileData)
    });
  }

  // Media endpoints
  async uploadMedia(file: File | FormData) {
    return this.request('/api/media/upload', {
      method: 'POST',
      body: file,
      headers: {} // Let browser set Content-Type for FormData
    });
  }
}

export const apiService = new ApiService(API_BASE_URL);
```

---

## 🚀 Development Workflow

### Development Environment Setup

1. **Start Backend Services**:
```bash
# Start all 7 microservices
npm run dev:backend
```

2. **Start Web Frontend**:
```bash
cd shareuptime-frontend
npm run dev
```

3. **Start Mobile App**:
```bash
cd ShareUpTimeMobile
npx react-native start
```

### Testing Strategy

**Web Testing**:
```bash
# Unit tests
npm run test

# E2E tests
npm run test:e2e

# Coverage report
npm run test:coverage
```

**Mobile Testing**:
```bash
# Unit tests
npm run test

# Integration tests
detox test

# Performance testing
npm run test:performance
```

### Build and Deployment

**Web Deployment**:
```bash
# Build for production
npm run build

# Deploy to Vercel
vercel deploy

# Deploy to Netlify
netlify deploy --prod
```

**Mobile Deployment**:
```bash
# Android
cd android && ./gradlew bundleRelease

# iOS (requires macOS and Xcode)
cd ios && xcodebuild archive
```

This comprehensive frontend guide provides everything needed to develop, test, and deploy both web and mobile applications for the ShareUpTime platform.