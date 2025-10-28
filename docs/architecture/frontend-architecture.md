# Frontend Architecture

This section details the frontend application structure, component organization, state management patterns, and key implementation details for the React SPA.

## Project Structure

```
frontend/
├── src/
│   ├── main.tsx                 # App entry point
│   ├── App.tsx                  # Root component with providers
│   ├── components/              # Reusable UI components
│   │   ├── ui/                  # Untitled UI base components
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Input.tsx
│   │   │   └── ...
│   │   ├── layout/              # Layout components
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── Footer.tsx
│   │   └── shared/              # Shared business components
│   │       ├── LoadingSpinner.tsx
│   │       ├── ErrorBoundary.tsx
│   │       └── Toast.tsx
│   ├── pages/                   # Route-level page components
│   │   ├── auth/
│   │   │   ├── LoginPage.tsx
│   │   │   ├── RegisterPage.tsx
│   │   │   └── PendingApprovalPage.tsx
│   │   ├── map/
│   │   │   └── MapView.tsx
│   │   ├── colonies/
│   │   │   ├── ColonyDetailsPage.tsx
│   │   │   └── ColonyListPage.tsx
│   │   ├── cats/
│   │   │   ├── CatProfilePage.tsx
│   │   │   └── CatListPage.tsx
│   │   ├── admin/
│   │   │   └── AdminDashboard.tsx
│   │   └── NotFoundPage.tsx
│   ├── features/                # Feature-specific components
│   │   ├── auth/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── useAuth.ts
│   │   ├── colonies/
│   │   │   ├── ColonyHeader.tsx
│   │   │   ├── ColonyInfoCard.tsx
│   │   │   ├── AddColonyModal.tsx
│   │   │   └── useColonies.ts
│   │   ├── cats/
│   │   │   ├── CatCard.tsx
│   │   │   ├── CatPhotoGallery.tsx
│   │   │   ├── EditCatModal.tsx
│   │   │   └── useCats.ts
│   │   ├── activities/
│   │   │   ├── ActivityLogList.tsx
│   │   │   ├── ActivityLogItem.tsx
│   │   │   ├── LogActivityModal.tsx
│   │   │   └── useActivities.ts
│   │   └── map/
│   │       ├── MapControls.tsx
│   │       ├── ColonyMarker.tsx
│   │       ├── SearchBar.tsx
│   │       └── FilterPanel.tsx
│   ├── services/                # API client services
│   │   ├── api.ts               # Axios instance configuration
│   │   ├── authService.ts
│   │   ├── colonyService.ts
│   │   ├── catService.ts
│   │   ├── activityService.ts
│   │   └── adminService.ts
│   ├── hooks/                   # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useToast.ts
│   │   ├── useDebounce.ts
│   │   └── useLocalStorage.ts
│   ├── contexts/                # React contexts
│   │   ├── AuthContext.tsx
│   │   └── ToastContext.tsx
│   ├── utils/                   # Utility functions
│   │   ├── formatters.ts        # Date, distance formatters
│   │   ├── validators.ts        # Zod schemas
│   │   └── constants.ts         # App constants
│   ├── types/                   # TypeScript type definitions
│   │   └── index.ts             # Re-export from @paw-patrol/shared
│   └── styles/                  # Global styles
│       ├── globals.css          # Tailwind imports
│       └── tailwind.config.js
├── public/                      # Static assets
│   ├── favicon.ico
│   └── images/
├── index.html                   # HTML entry point
├── vite.config.ts               # Vite configuration
├── tsconfig.json                # TypeScript configuration
└── package.json
```

---

## State Management Strategy

**Three-Tier State Architecture:**

1. **Server State (TanStack Query)**
   - All data from backend API (colonies, cats, activities, users)
   - Cached with stale-while-revalidate strategy
   - Optimistic updates for mutations
   - Automatic refetching on window focus

2. **Authentication State (React Context)**
   - Current user object
   - JWT token (stored in localStorage)
   - Login/logout functions
   - Role-based access control helpers

3. **UI State (React Hooks)**
   - Local component state (form inputs, modals, dropdowns)
   - URL state (React Router params, query params)
   - Toast notifications (React Context)

**TanStack Query Configuration:**

```typescript
// src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      retry: 1,
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,
    },
    mutations: {
      retry: 0,
    },
  },
});
```

**Query Key Structure:**

```typescript
// Hierarchical query keys for efficient invalidation
const queryKeys = {
  colonies: ['colonies'] as const,
  colony: (id: string) => ['colonies', id] as const,
  colonyCats: (id: string) => ['colonies', id, 'cats'] as const,
  colonyActivities: (id: string) => ['colonies', id, 'activities'] as const,
  
  cats: ['cats'] as const,
  cat: (id: string) => ['cats', id] as const,
  catActivities: (id: string) => ['cats', id, 'activities'] as const,
  
  activities: ['activities'] as const,
  
  users: ['users'] as const,
  user: (id: string) => ['users', id] as const,
  pendingUsers: ['users', 'pending'] as const,
};
```

---

## Routing Configuration

**React Router Setup:**

```typescript
// src/App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/login" element={<LoginPage />} />
        <Route path="/register" element={<RegisterPage />} />
        <Route path="/pending" element={<PendingApprovalPage />} />

        {/* Protected routes */}
        <Route element={<ProtectedRoute />}>
          <Route path="/" element={<MapView />} />
          <Route path="/colonies/:colonyId" element={<ColonyDetailsPage />} />
          <Route path="/cats/:catId" element={<CatProfilePage />} />
        </Route>

        {/* Admin routes */}
        <Route element={<AdminRoute />}>
          <Route path="/admin" element={<AdminDashboard />} />
        </Route>

        {/* Fallback */}
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Route Guards:**

```typescript
// src/components/ProtectedRoute.tsx
import { Navigate, Outlet } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';

export function ProtectedRoute() {
  const { user, isLoading } = useAuth();

  if (isLoading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" replace />;
  if (user.status === 'PENDING_VERIFICATION') return <Navigate to="/pending" replace />;
  if (user.status === 'REJECTED' || user.status === 'INACTIVE') return <Navigate to="/login" replace />;

  return <Outlet />;
}

export function AdminRoute() {
  const { user, isLoading } = useAuth();

  if (isLoading) return <LoadingSpinner />;
  if (!user || (user.role !== 'ADMIN' && user.role !== 'SUPER_ADMIN')) {
    return <Navigate to="/" replace />;
  }

  return <Outlet />;
}
```

---

## API Client Configuration

**Axios Instance with Interceptors:**

```typescript
// src/services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:8080/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor: Add JWT token
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor: Handle 401 (token expired)
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // If 401 and not already retried
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Try to refresh token
        const refreshToken = localStorage.getItem('refreshToken');
        const response = await axios.post('/api/auth/refresh', { refreshToken });
        const { token } = response.data;

        localStorage.setItem('authToken', token);
        originalRequest.headers.Authorization = `Bearer ${token}`;

        return api(originalRequest);
      } catch (refreshError) {
        // Refresh failed, logout user
        localStorage.removeItem('authToken');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);

export default api;
```

---

## Error Handling Pattern

**Global Error Boundary:**

```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    // TODO: Send to error tracking service (e.g., Sentry)
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen flex items-center justify-center bg-gray-50">
          <div className="max-w-md p-8 bg-white rounded-lg shadow-lg">
            <h1 className="text-2xl font-bold text-red-600 mb-4">
              Something went wrong
            </h1>
            <p className="text-gray-700 mb-4">
              {this.state.error?.message || 'An unexpected error occurred'}
            </p>
            <button
              onClick={() => window.location.reload()}
              className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
            >
              Reload Page
            </button>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

---

## Performance Optimization

**Code Splitting with React.lazy:**

```typescript
// src/App.tsx
import { lazy, Suspense } from 'react';

const MapView = lazy(() => import('@/pages/map/MapView'));
const ColonyDetailsPage = lazy(() => import('@/pages/colonies/ColonyDetailsPage'));
const CatProfilePage = lazy(() => import('@/pages/cats/CatProfilePage'));
const AdminDashboard = lazy(() => import('@/pages/admin/AdminDashboard'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<MapView />} />
        <Route path="/colonies/:colonyId" element={<ColonyDetailsPage />} />
        <Route path="/cats/:catId" element={<CatProfilePage />} />
        <Route path="/admin" element={<AdminDashboard />} />
      </Routes>
    </Suspense>
  );
}
```

**Image Optimization:**

```typescript
// src/utils/imageOptimizer.ts
export async function optimizeImage(file: File, maxWidth: number = 1920): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = (event) => {
      const img = new Image();
      img.src = event.target?.result as string;
      img.onload = () => {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d')!;

        // Calculate new dimensions
        let width = img.width;
        let height = img.height;
        if (width > maxWidth) {
          height = (height * maxWidth) / width;
          width = maxWidth;
        }

        canvas.width = width;
        canvas.height = height;
        ctx.drawImage(img, 0, 0, width, height);

        // Convert to base64 with quality compression
        resolve(canvas.toDataURL('image/jpeg', 0.8));
      };
      img.onerror = reject;
    };
    reader.onerror = reject;
  });
}
```

---
