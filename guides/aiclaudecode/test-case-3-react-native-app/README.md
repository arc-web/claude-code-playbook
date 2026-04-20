# React Native App with Expo SDK 50

A modern React Native application with Expo SDK 50, TypeScript, React Navigation v6, Zustand state management, and comprehensive mobile development patterns.

## Features

- ✅ Expo SDK 50 with latest React Native
- ✅ TypeScript throughout the application
- ✅ React Navigation v6 with stack and tab navigation
- ✅ Zustand for state management
- ✅ AsyncStorage for offline data persistence
- ✅ React Query for API integration
- ✅ Push notifications setup
- ✅ Deep linking configuration
- ✅ Image picker and camera integration
- ✅ Social authentication
- ✅ Offline-first architecture

## Project Structure

```
├── app/
│   ├── (auth)/
│   │   ├── login.tsx
│   │   ├── register.tsx
│   │   └── forgot-password.tsx
│   ├── (tabs)/
│   │   ├── home/
│   │   │   ├── index.tsx
│   │   │   └── [id].tsx
│   │   ├── profile/
│   │   │   ├── index.tsx
│   │   │   └── edit.tsx
│   │   ├── search/
│   │   │   └── index.tsx
│   │   └── settings/
│   │       └── index.tsx
│   ├── _layout.tsx
│   ├── index.tsx
│   └── +not-found.tsx
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   └── Modal.tsx
│   ├── forms/
│   │   ├── LoginForm.tsx
│   │   └── RegisterForm.tsx
│   └── shared/
│       ├── Header.tsx
│       ├── LoadingSpinner.tsx
│       └── ErrorBoundary.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useApi.ts
│   └── useStorage.ts
├── services/
│   ├── api.ts
│   ├── auth.ts
│   ├── storage.ts
│   └── notifications.ts
├── stores/
│   ├── authStore.ts
│   ├── userStore.ts
│   └── appStore.ts
├── types/
│   ├── api.ts
│   ├── auth.ts
│   └── navigation.ts
├── utils/
│   ├── constants.ts
│   ├── helpers.ts
│   └── validation.ts
├── assets/
├── app.json
├── package.json
└── tsconfig.json
```

## Setup Instructions

### 1. Install Dependencies

```bash
# Create new Expo project
npx create-expo-app@latest MyApp --template blank-typescript

# Navigate to project
cd MyApp

# Install core dependencies
npm install expo-router@3.4.6
npm install @react-navigation/native @react-navigation/stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context
npm install zustand
npm install @tanstack/react-query
npm install @react-native-async-storage/async-storage
npm install expo-notifications expo-device expo-constants
npm install expo-image-picker expo-camera expo-media-library
npm install expo-auth-session expo-crypto expo-web-browser
npm install expo-linking expo-splash-screen expo-status-bar
npm install react-hook-form @hookform/resolvers zod
npm install react-native-reanimated react-native-gesture-handler
npm install expo-font expo-asset
```

### 2. Environment Configuration

Create `app.config.ts`:

```typescript
import { ExpoConfig, ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: 'MyApp',
  slug: 'my-app',
  version: '1.0.0',
  orientation: 'portrait',
  icon: './assets/icon.png',
  userInterfaceStyle: 'light',
  splash: {
    image: './assets/splash.png',
    resizeMode: 'contain',
    backgroundColor: '#ffffff'
  },
  assetBundlePatterns: [
    '**/*'
  ],
  ios: {
    supportsTablet: true,
    bundleIdentifier: 'com.yourcompany.myapp'
  },
  android: {
    adaptiveIcon: {
      foregroundImage: './assets/adaptive-icon.png',
      backgroundColor: '#FFFFFF'
    },
    package: 'com.yourcompany.myapp'
  },
  web: {
    favicon: './assets/favicon.png'
  },
  plugins: [
    'expo-router',
    [
      'expo-notifications',
      {
        icon: './assets/notification-icon.png',
        color: '#ffffff'
      }
    ],
    [
      'expo-image-picker',
      {
        photosPermission: 'The app accesses your photos to let you share them with your friends.'
      }
    ]
  ],
  scheme: 'myapp',
  extra: {
    eas: {
      projectId: 'your-project-id'
    }
  }
});
```

### 3. App Entry Point

```typescript
// app/_layout.tsx
import { Stack } from 'expo-router';
import { useEffect } from 'react';
import { useFonts } from 'expo-font';
import * as SplashScreen from 'expo-splash-screen';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { StatusBar } from 'expo-status-bar';

// Keep the splash screen visible while we fetch resources
SplashScreen.preventAutoHideAsync();

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 10, // 10 minutes
      retry: 2,
    },
  },
});

export default function RootLayout() {
  const [loaded, error] = useFonts({
    'Inter-Regular': require('../assets/fonts/Inter-Regular.ttf'),
    'Inter-Medium': require('../assets/fonts/Inter-Medium.ttf'),
    'Inter-Bold': require('../assets/fonts/Inter-Bold.ttf'),
  });

  useEffect(() => {
    if (error) throw error;
  }, [error]);

  useEffect(() => {
    if (loaded) {
      SplashScreen.hideAsync();
    }
  }, [loaded]);

  if (!loaded) {
    return null;
  }

  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <QueryClientProvider client={queryClient}>
        <Stack>
          <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          <Stack.Screen name="(auth)" options={{ headerShown: false }} />
        </Stack>
        <StatusBar style="auto" />
      </QueryClientProvider>
    </GestureHandlerRootView>
  );
}
```

### 4. Tab Navigation

```typescript
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
        tabBarStyle: {
          backgroundColor: '#FFFFFF',
          borderTopWidth: 1,
          borderTopColor: '#E5E5EA',
        },
        headerStyle: {
          backgroundColor: '#FFFFFF',
        },
        headerTitleStyle: {
          fontWeight: '600',
        },
      }}>
      <Tabs.Screen
        name="home"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, focused }) => (
            <Ionicons
              name={focused ? 'home' : 'home-outline'}
              size={24}
              color={color}
            />
          ),
        }}
      />
      <Tabs.Screen
        name="search"
        options={{
          title: 'Search',
          tabBarIcon: ({ color, focused }) => (
            <Ionicons
              name={focused ? 'search' : 'search-outline'}
              size={24}
              color={color}
            />
          ),
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, focused }) => (
            <Ionicons
              name={focused ? 'person' : 'person-outline'}
              size={24}
              color={color}
            />
          ),
        }}
      />
      <Tabs.Screen
        name="settings"
        options={{
          title: 'Settings',
          tabBarIcon: ({ color, focused }) => (
            <Ionicons
              name={focused ? 'settings' : 'settings-outline'}
              size={24}
              color={color}
            />
          ),
        }}
      />
    </Tabs>
  );
}
```

### 5. State Management with Zustand

```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { User, AuthState } from '../types/auth';

interface AuthStore extends AuthState {
  setUser: (user: User | null) => void;
  setToken: (token: string | null) => void;
  setLoading: (loading: boolean) => void;
  logout: () => void;
  login: (user: User, token: string) => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isLoading: false,
      isAuthenticated: false,
      
      setUser: (user) =>
        set((state) => ({
          user,
          isAuthenticated: !!user,
        })),
      
      setToken: (token) =>
        set(() => ({
          token,
        })),
      
      setLoading: (isLoading) =>
        set(() => ({
          isLoading,
        })),
      
      logout: () =>
        set(() => ({
          user: null,
          token: null,
          isAuthenticated: false,
        })),
      
      login: (user, token) =>
        set(() => ({
          user,
          token,
          isAuthenticated: true,
        })),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({
        user: state.user,
        token: state.token,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);
```

### 6. API Service

```typescript
// services/api.ts
import { QueryClient } from '@tanstack/react-query';
import { useAuthStore } from '../stores/authStore';

const API_BASE_URL = 'https://api.yourapp.com';

class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public data?: any
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

class ApiService {
  private baseURL: string;
  private queryClient: QueryClient;

  constructor(baseURL: string, queryClient: QueryClient) {
    this.baseURL = baseURL;
    this.queryClient = queryClient;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const token = useAuthStore.getState().token;
    
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...(token && { Authorization: `Bearer ${token}` }),
        ...options.headers,
      },
      ...options,
    };

    const response = await fetch(`${this.baseURL}${endpoint}`, config);
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new ApiError(
        errorData.message || 'An error occurred',
        response.status,
        errorData
      );
    }

    return response.json();
  }

  // Auth endpoints
  async login(email: string, password: string) {
    return this.request<{ user: User; token: string }>('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
  }

  async register(userData: RegisterData) {
    return this.request<{ user: User; token: string }>('/auth/register', {
      method: 'POST',
      body: JSON.stringify(userData),
    });
  }

  async refreshToken() {
    return this.request<{ token: string }>('/auth/refresh', {
      method: 'POST',
    });
  }

  // User endpoints
  async getProfile() {
    return this.request<User>('/user/profile');
  }

  async updateProfile(userData: Partial<User>) {
    return this.request<User>('/user/profile', {
      method: 'PUT',
      body: JSON.stringify(userData),
    });
  }

  // Posts endpoints
  async getPosts(page = 1, limit = 10) {
    return this.request<{ posts: Post[]; total: number }>(
      `/posts?page=${page}&limit=${limit}`
    );
  }

  async getPost(id: string) {
    return this.request<Post>(`/posts/${id}`);
  }

  async createPost(postData: CreatePostData) {
    return this.request<Post>('/posts', {
      method: 'POST',
      body: JSON.stringify(postData),
    });
  }

  async updatePost(id: string, postData: Partial<Post>) {
    return this.request<Post>(`/posts/${id}`, {
      method: 'PUT',
      body: JSON.stringify(postData),
    });
  }

  async deletePost(id: string) {
    return this.request<void>(`/posts/${id}`, {
      method: 'DELETE',
    });
  }

  // Invalidate queries
  invalidatePosts() {
    this.queryClient.invalidateQueries({ queryKey: ['posts'] });
  }

  invalidateUser() {
    this.queryClient.invalidateQueries({ queryKey: ['user'] });
  }
}

export const createApiService = (queryClient: QueryClient) =>
  new ApiService(API_BASE_URL, queryClient);
```

### 7. React Query Hooks

```typescript
// hooks/useApi.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { createApiService } from '../services/api';
import { useAuthStore } from '../stores/authStore';

const api = createApiService(new QueryClient());

export const usePosts = (page = 1, limit = 10) => {
  return useQuery({
    queryKey: ['posts', page, limit],
    queryFn: () => api.getPosts(page, limit),
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
};

export const usePost = (id: string) => {
  return useQuery({
    queryKey: ['post', id],
    queryFn: () => api.getPost(id),
    enabled: !!id,
  });
};

export const useProfile = () => {
  const { isAuthenticated } = useAuthStore();
  
  return useQuery({
    queryKey: ['user', 'profile'],
    queryFn: () => api.getProfile(),
    enabled: isAuthenticated,
  });
};

export const useCreatePost = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (postData: CreatePostData) => api.createPost(postData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
};

export const useUpdatePost = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<Post> }) =>
      api.updatePost(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      queryClient.invalidateQueries({ queryKey: ['post', id] });
    },
  });
};

export const useDeletePost = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (id: string) => api.deletePost(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
};
```

### 8. Authentication Hook

```typescript
// hooks/useAuth.ts
import { useMutation } from '@tanstack/react-query';
import { useAuthStore } from '../stores/authStore';
import { createApiService } from '../services/api';
import { router } from 'expo-router';

const api = createApiService(new QueryClient());

export const useAuth = () => {
  const { user, token, isAuthenticated, setLoading, login, logout } = useAuthStore();

  const loginMutation = useMutation({
    mutationFn: ({ email, password }: { email: string; password: string }) =>
      api.login(email, password),
    onSuccess: ({ user, token }) => {
      login(user, token);
      router.replace('/(tabs)');
    },
    onError: (error) => {
      console.error('Login error:', error);
    },
  });

  const registerMutation = useMutation({
    mutationFn: (userData: RegisterData) => api.register(userData),
    onSuccess: ({ user, token }) => {
      login(user, token);
      router.replace('/(tabs)');
    },
    onError: (error) => {
      console.error('Register error:', error);
    },
  });

  const logoutUser = () => {
    logout();
    router.replace('/(auth)/login');
  };

  return {
    user,
    token,
    isAuthenticated,
    isLoading: loginMutation.isPending || registerMutation.isPending,
    login: loginMutation.mutate,
    register: registerMutation.mutate,
    logout: logoutUser,
    loginError: loginMutation.error,
    registerError: registerMutation.error,
  };
};
```

### 9. UI Components

```typescript
// components/ui/Button.tsx
import React from 'react';
import {
  TouchableOpacity,
  Text,
  StyleSheet,
  ActivityIndicator,
  ViewStyle,
  TextStyle,
} from 'react-native';

interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
}

export const Button: React.FC<ButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  style,
  textStyle,
}) => {
  const buttonStyle = [
    styles.button,
    styles[variant],
    styles[size],
    disabled && styles.disabled,
    style,
  ];

  const textStyles = [
    styles.text,
    styles[`${variant}Text`],
    styles[`${size}Text`],
    disabled && styles.disabledText,
    textStyle,
  ];

  return (
    <TouchableOpacity
      style={buttonStyle}
      onPress={onPress}
      disabled={disabled || loading}
      activeOpacity={0.8}
    >
      {loading ? (
        <ActivityIndicator
          color={variant === 'primary' ? '#FFFFFF' : '#007AFF'}
          size="small"
        />
      ) : (
        <Text style={textStyles}>{title}</Text>
      )}
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  primary: {
    backgroundColor: '#007AFF',
  },
  secondary: {
    backgroundColor: '#F2F2F7',
  },
  outline: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#007AFF',
  },
  small: {
    paddingVertical: 8,
    paddingHorizontal: 16,
  },
  medium: {
    paddingVertical: 12,
    paddingHorizontal: 24,
  },
  large: {
    paddingVertical: 16,
    paddingHorizontal: 32,
  },
  disabled: {
    opacity: 0.5,
  },
  text: {
    fontWeight: '600',
  },
  primaryText: {
    color: '#FFFFFF',
  },
  secondaryText: {
    color: '#000000',
  },
  outlineText: {
    color: '#007AFF',
  },
  smallText: {
    fontSize: 14,
  },
  mediumText: {
    fontSize: 16,
  },
  largeText: {
    fontSize: 18,
  },
  disabledText: {
    opacity: 0.5,
  },
});
```

### 10. Push Notifications

```typescript
// services/notifications.ts
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export class NotificationService {
  static async registerForPushNotificationsAsync() {
    let token;

    if (Platform.OS === 'android') {
      await Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: '#FF231F7C',
      });
    }

    if (Device.isDevice) {
      const { status: existingStatus } = await Notifications.getPermissionsAsync();
      let finalStatus = existingStatus;
      
      if (existingStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }
      
      if (finalStatus !== 'granted') {
        alert('Failed to get push token for push notification!');
        return;
      }
      
      token = (await Notifications.getExpoPushTokenAsync({
        projectId: 'your-project-id',
      })).data;
    } else {
      alert('Must use physical device for Push Notifications');
    }

    return token;
  }

  static async scheduleLocalNotification(title: string, body: string, data?: any) {
    await Notifications.scheduleNotificationAsync({
      content: {
        title,
        body,
        data,
      },
      trigger: null, // Send immediately
    });
  }

  static async scheduleReminderNotification(title: string, body: string, trigger: any) {
    await Notifications.scheduleNotificationAsync({
      content: {
        title,
        body,
      },
      trigger,
    });
  }

  static addNotificationReceivedListener(listener: (notification: Notifications.Notification) => void) {
    return Notifications.addNotificationReceivedListener(listener);
  }

  static addNotificationResponseReceivedListener(listener: (response: Notifications.NotificationResponse) => void) {
    return Notifications.addNotificationResponseReceivedListener(listener);
  }
}
```

### 11. Deep Linking

```typescript
// app/_layout.tsx (add to existing)
import * as Linking from 'expo-linking';

// Add this to your RootLayout component
useEffect(() => {
  const subscription = Linking.addEventListener('url', (event) => {
    // Handle deep link
    const { url } = event;
    console.log('Deep link received:', url);
    
    // Parse the URL and navigate accordingly
    const parsedUrl = Linking.parse(url);
    
    if (parsedUrl.path) {
      router.push(parsedUrl.path as any);
    }
  });

  return () => subscription?.remove();
}, []);
```

### 12. Image Picker Integration

```typescript
// hooks/useImagePicker.ts
import { useState } from 'react';
import * as ImagePicker from 'expo-image-picker';
import { Alert } from 'react-native';

export const useImagePicker = () => {
  const [image, setImage] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const requestPermissions = async () => {
    const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert(
        'Permission Required',
        'Sorry, we need camera roll permissions to make this work!'
      );
      return false;
    }
    return true;
  };

  const pickImage = async () => {
    const hasPermission = await requestPermissions();
    if (!hasPermission) return;

    setLoading(true);
    try {
      const result = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: ImagePicker.MediaTypeOptions.Images,
        allowsEditing: true,
        aspect: [1, 1],
        quality: 0.8,
      });

      if (!result.canceled && result.assets[0]) {
        setImage(result.assets[0].uri);
        return result.assets[0];
      }
    } catch (error) {
      console.error('Error picking image:', error);
      Alert.alert('Error', 'Failed to pick image');
    } finally {
      setLoading(false);
    }
  };

  const takePhoto = async () => {
    const { status } = await ImagePicker.requestCameraPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert(
        'Permission Required',
        'Sorry, we need camera permissions to make this work!'
      );
      return;
    }

    setLoading(true);
    try {
      const result = await ImagePicker.launchCameraAsync({
        allowsEditing: true,
        aspect: [1, 1],
        quality: 0.8,
      });

      if (!result.canceled && result.assets[0]) {
        setImage(result.assets[0].uri);
        return result.assets[0];
      }
    } catch (error) {
      console.error('Error taking photo:', error);
      Alert.alert('Error', 'Failed to take photo');
    } finally {
      setLoading(false);
    }
  };

  return {
    image,
    loading,
    pickImage,
    takePhoto,
    clearImage: () => setImage(null),
  };
};
```

### 13. App Configuration

```json
// app.json
{
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.yourcompany.myapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-router",
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ],
      [
        "expo-image-picker",
        {
          "photosPermission": "The app accesses your photos to let you share them with your friends."
        }
      ]
    ],
    "scheme": "myapp",
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

This React Native implementation demonstrates modern Expo SDK 50 patterns, comprehensive state management with Zustand, efficient data fetching with React Query, and production-ready mobile development practices.
