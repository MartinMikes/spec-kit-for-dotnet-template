# UI Guidelines - React/Vue.js with FluentUI

## UI Framework Selection

### React with FluentUI
**Recommended for**: Complex enterprise applications, strong TypeScript support

### Vue.js with FluentUI
**Recommended for**: Rapid development, simpler learning curve

Both frameworks can use Microsoft's FluentUI (Fluent UI React or Fluent UI Vue) for consistent, accessible UI components.

## Project Setup

### React + FluentUI + TypeScript
```bash
# Create React app with TypeScript
npx create-react-app my-app --template typescript

# Install FluentUI
npm install @fluentui/react @fluentui/react-icons

# Install additional dependencies
npm install react-router-dom @tanstack/react-query axios
npm install -D @types/react-router-dom
```

### Vue 3 + FluentUI + TypeScript
```bash
# Create Vue app
npm create vue@latest

# Install FluentUI Web Components
npm install @fluentui/web-components

# Install additional dependencies
npm install vue-router pinia axios
```

## Project Structure

### React Project Structure
```
src/
├── components/
│   ├── common/           # Reusable UI components
│   │   ├── Button/
│   │   ├── Card/
│   │   └── Input/
│   ├── layout/           # Layout components
│   │   ├── Header/
│   │   ├── Sidebar/
│   │   └── Footer/
│   └── features/         # Feature-specific components
│       ├── users/
│       └── orders/
│
├── pages/                # Page components
│   ├── Home/
│   ├── Users/
│   └── Dashboard/
│
├── hooks/                # Custom React hooks
│   ├── useAuth.ts
│   ├── useApi.ts
│   └── useDebounce.ts
│
├── services/             # API services
│   ├── api.ts
│   ├── authService.ts
│   └── userService.ts
│
├── store/                # State management
│   ├── slices/
│   └── index.ts
│
├── types/                # TypeScript types
│   ├── models/
│   └── api/
│
├── utils/                # Utility functions
│   ├── validation.ts
│   ├── formatting.ts
│   └── constants.ts
│
├── styles/               # Global styles
│   ├── theme.ts
│   └── global.css
│
├── App.tsx
└── main.tsx
```

### Vue Project Structure
```
src/
├── components/
│   ├── common/
│   ├── layout/
│   └── features/
│
├── views/                # Page views
│   ├── HomeView.vue
│   ├── UsersView.vue
│   └── DashboardView.vue
│
├── composables/          # Composition API composables
│   ├── useAuth.ts
│   ├── useApi.ts
│   └── useDebounce.ts
│
├── services/
│   ├── api.ts
│   └── userService.ts
│
├── stores/               # Pinia stores
│   ├── auth.ts
│   └── user.ts
│
├── types/
│   └── models.ts
│
├── router/
│   └── index.ts
│
├── App.vue
└── main.ts
```

## FluentUI Implementation

### React FluentUI Theme
```typescript
// src/styles/theme.ts
import { createTheme, Theme } from '@fluentui/react';

export const lightTheme: Theme = createTheme({
  palette: {
    themePrimary: '#0078d4',
    themeLighterAlt: '#eff6fc',
    themeLighter: '#deecf9',
    themeLight: '#c7e0f4',
    themeTertiary: '#71afe5',
    themeSecondary: '#2b88d8',
    themeDarkAlt: '#106ebe',
    themeDark: '#005a9e',
    themeDarker: '#004578',
    neutralLighterAlt: '#faf9f8',
    neutralLighter: '#f3f2f1',
    neutralLight: '#edebe9',
    neutralQuaternaryAlt: '#e1dfdd',
    neutralQuaternary: '#d0d0d0',
    neutralTertiaryAlt: '#c8c6c4',
    neutralTertiary: '#a19f9d',
    neutralSecondary: '#605e5c',
    neutralPrimaryAlt: '#3b3a39',
    neutralPrimary: '#323130',
    neutralDark: '#201f1e',
    black: '#000000',
    white: '#ffffff',
  },
});

export const darkTheme: Theme = createTheme({
  palette: {
    themePrimary: '#3aa0f3',
    themeLighterAlt: '#02060a',
    themeLighter: '#091823',
    themeLight: '#112d43',
    themeTertiary: '#235a85',
    themeSecondary: '#3385c3',
    themeDarkAlt: '#4ba6f4',
    themeDark: '#65b1f6',
    themeDarker: '#8ac4f8',
    neutralLighterAlt: '#1c1c1c',
    neutralLighter: '#252525',
    neutralLight: '#343434',
    neutralQuaternaryAlt: '#3d3d3d',
    neutralQuaternary: '#454545',
    neutralTertiaryAlt: '#656565',
    neutralTertiary: '#c8c8c8',
    neutralSecondary: '#d0d0d0',
    neutralPrimaryAlt: '#dadada',
    neutralPrimary: '#ffffff',
    neutralDark: '#f4f4f4',
    black: '#f8f8f8',
    white: '#121212',
  },
});
```

### React FluentUI App Setup
```typescript
// App.tsx
import { ThemeProvider } from '@fluentui/react';
import { lightTheme, darkTheme } from './styles/theme';
import { useState } from 'react';

function App() {
  const [isDarkMode, setIsDarkMode] = useState(false);

  return (
    <ThemeProvider theme={isDarkMode ? darkTheme : lightTheme}>
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/users" element={<Users />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Router>
    </ThemeProvider>
  );
}
```

### Common FluentUI Components (React)

#### Button Component
```typescript
// components/common/Button/Button.tsx
import { PrimaryButton, DefaultButton, IButtonProps } from '@fluentui/react/lib/Button';

interface CustomButtonProps extends IButtonProps {
  variant?: 'primary' | 'default';
}

export const Button: React.FC<CustomButtonProps> = ({ 
  variant = 'default', 
  children, 
  ...props 
}) => {
  return variant === 'primary' ? (
    <PrimaryButton {...props}>{children}</PrimaryButton>
  ) : (
    <DefaultButton {...props}>{children}</DefaultButton>
  );
};
```

#### Card Component
```typescript
// components/common/Card/Card.tsx
import { Card as FluentCard, ICardTokens } from '@fluentui/react-cards';
import { Stack } from '@fluentui/react';
import styles from './Card.module.css';

interface CardProps {
  title?: string;
  children: React.ReactNode;
  actions?: React.ReactNode;
}

export const Card: React.FC<CardProps> = ({ title, children, actions }) => {
  const cardTokens: ICardTokens = {
    childrenMargin: 12,
    padding: 20,
  };

  return (
    <FluentCard className={styles.card} tokens={cardTokens}>
      {title && <h3 className={styles.title}>{title}</h3>}
      <Stack className={styles.content}>{children}</Stack>
      {actions && <div className={styles.actions}>{actions}</div>}
    </FluentCard>
  );
};
```

#### Form Input Component
```typescript
// components/common/Input/Input.tsx
import { TextField, ITextFieldProps } from '@fluentui/react/lib/TextField';

interface InputProps extends ITextFieldProps {
  error?: string;
}

export const Input: React.FC<InputProps> = ({ error, ...props }) => {
  return (
    <TextField
      {...props}
      errorMessage={error}
      validateOnLoad={false}
    />
  );
};
```

### Vue FluentUI Setup
```typescript
// main.ts
import { createApp } from 'vue';
import { 
  provideFluentDesignSystem,
  fluentButton,
  fluentCard,
  fluentTextField,
} from '@fluentui/web-components';

provideFluentDesignSystem()
  .register(
    fluentButton(),
    fluentCard(),
    fluentTextField()
  );

const app = createApp(App);
app.mount('#app');
```

```vue
<!-- Example Vue Component -->
<template>
  <fluent-card>
    <h3>{{ title }}</h3>
    <fluent-text-field 
      v-model="name" 
      placeholder="Enter name"
    />
    <fluent-button 
      appearance="primary" 
      @click="handleSubmit"
    >
      Submit
    </fluent-button>
  </fluent-card>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const name = ref('');

const handleSubmit = () => {
  console.log('Submitted:', name.value);
};
</script>
```

## State Management

### React - Redux Toolkit
```typescript
// store/slices/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { userService } from '../../services/userService';
import { User } from '../../types/models';

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  loading: false,
  error: null,
};

export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async () => {
    const response = await userService.getAll();
    return response.data;
  }
);

const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action: PayloadAction<User[]>) => {
        state.loading = false;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch users';
      });
  },
});

export const { clearError } = userSlice.actions;
export default userSlice.reducer;
```

### Vue - Pinia
```typescript
// stores/user.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { userService } from '@/services/userService';
import type { User } from '@/types/models';

export const useUserStore = defineStore('user', () => {
  const users = ref<User[]>([]);
  const loading = ref(false);
  const error = ref<string | null>(null);

  const activeUsers = computed(() => 
    users.value.filter(u => u.isActive)
  );

  async function fetchUsers() {
    loading.value = true;
    error.value = null;
    try {
      const response = await userService.getAll();
      users.value = response.data;
    } catch (e) {
      error.value = 'Failed to fetch users';
    } finally {
      loading.value = false;
    }
  }

  async function createUser(user: Omit<User, 'id'>) {
    loading.value = true;
    try {
      const response = await userService.create(user);
      users.value.push(response.data);
    } catch (e) {
      error.value = 'Failed to create user';
      throw e;
    } finally {
      loading.value = false;
    }
  }

  return {
    users,
    loading,
    error,
    activeUsers,
    fetchUsers,
    createUser,
  };
});
```

## API Integration

### Axios Setup
```typescript
// services/api.ts
import axios, { AxiosError, AxiosInstance } from 'axios';

const api: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:5000/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
api.interceptors.response.use(
  (response) => response,
  (error: AxiosError) => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

### Service Layer
```typescript
// services/userService.ts
import api from './api';
import { User, CreateUserRequest, UpdateUserRequest } from '../types/models';

export const userService = {
  async getAll() {
    return api.get<User[]>('/users');
  },

  async getById(id: number) {
    return api.get<User>(`/users/${id}`);
  },

  async create(user: CreateUserRequest) {
    return api.post<User>('/users', user);
  },

  async update(id: number, user: UpdateUserRequest) {
    return api.put<User>(`/users/${id}`, user);
  },

  async delete(id: number) {
    return api.delete(`/users/${id}`);
  },
};
```

### React Query (React)
```typescript
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService } from '../services/userService';
import { CreateUserRequest } from '../types/models';

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => userService.getAll().then(res => res.data),
  });
};

export const useUser = (id: number) => {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => userService.getById(id).then(res => res.data),
  });
};

export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (user: CreateUserRequest) => userService.create(user),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

## Component Patterns

### Smart vs Dumb Components

#### Smart Component (Container)
```typescript
// pages/Users/UsersList.tsx
import { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUsers } from '../../store/slices/userSlice';
import { UserTable } from '../../components/features/users/UserTable';
import { Spinner } from '@fluentui/react';

export const UsersList: React.FC = () => {
  const dispatch = useDispatch();
  const { users, loading, error } = useSelector((state) => state.users);

  useEffect(() => {
    dispatch(fetchUsers());
  }, [dispatch]);

  if (loading) return <Spinner label="Loading users..." />;
  if (error) return <div>Error: {error}</div>;

  return <UserTable users={users} />;
};
```

#### Dumb Component (Presentational)
```typescript
// components/features/users/UserTable.tsx
import { DetailsList, IColumn } from '@fluentui/react/lib/DetailsList';
import { User } from '../../../types/models';

interface UserTableProps {
  users: User[];
  onUserClick?: (user: User) => void;
}

export const UserTable: React.FC<UserTableProps> = ({ users, onUserClick }) => {
  const columns: IColumn[] = [
    {
      key: 'name',
      name: 'Name',
      fieldName: 'name',
      minWidth: 100,
      maxWidth: 200,
    },
    {
      key: 'email',
      name: 'Email',
      fieldName: 'email',
      minWidth: 150,
      maxWidth: 250,
    },
    {
      key: 'status',
      name: 'Status',
      fieldName: 'isActive',
      minWidth: 100,
      onRender: (item: User) => (
        <span>{item.isActive ? 'Active' : 'Inactive'}</span>
      ),
    },
  ];

  return (
    <DetailsList
      items={users}
      columns={columns}
      onItemInvoked={onUserClick}
    />
  );
};
```

### Custom Hooks (React)

```typescript
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const SearchComponent = () => {
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, 500);

  useEffect(() => {
    // API call with debounced search
    if (debouncedSearch) {
      searchUsers(debouncedSearch);
    }
  }, [debouncedSearch]);

  return (
    <TextField
      value={search}
      onChange={(_, value) => setSearch(value || '')}
      placeholder="Search users..."
    />
  );
};
```

### Composables (Vue)

```typescript
// composables/useDebounce.ts
import { ref, watch } from 'vue';

export function useDebounce<T>(value: Ref<T>, delay: number) {
  const debouncedValue = ref(value.value) as Ref<T>;

  watch(value, () => {
    const handler = setTimeout(() => {
      debouncedValue.value = value.value;
    }, delay);

    return () => clearTimeout(handler);
  });

  return debouncedValue;
}
```

## Form Handling

### React Hook Form
```typescript
// components/features/users/CreateUserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { TextField } from '@fluentui/react/lib/TextField';
import { PrimaryButton } from '@fluentui/react/lib/Button';

const userSchema = z.object({
  firstName: z.string().min(1, 'First name is required'),
  lastName: z.string().min(1, 'Last name is required'),
  email: z.string().email('Invalid email address'),
});

type UserFormData = z.infer<typeof userSchema>;

export const CreateUserForm: React.FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit = (data: UserFormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <TextField
        label="First Name"
        {...register('firstName')}
        errorMessage={errors.firstName?.message}
      />
      <TextField
        label="Last Name"
        {...register('lastName')}
        errorMessage={errors.lastName?.message}
      />
      <TextField
        label="Email"
        {...register('email')}
        errorMessage={errors.email?.message}
      />
      <PrimaryButton type="submit">Create User</PrimaryButton>
    </form>
  );
};
```

## Responsive Design

### Responsive Layout
```typescript
// components/layout/ResponsiveLayout.tsx
import { Stack, StackItem, useTheme } from '@fluentui/react';
import { useMediaQuery } from '../../hooks/useMediaQuery';

export const ResponsiveLayout: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const theme = useTheme();
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(max-width: 1024px)');

  return (
    <Stack
      horizontal={!isMobile}
      tokens={{ childrenGap: isMobile ? 10 : 20 }}
      styles={{
        root: {
          padding: isMobile ? 10 : isTablet ? 20 : 40,
        },
      }}
    >
      {children}
    </Stack>
  );
};
```

## Accessibility

### Best Practices
```typescript
// Proper semantic HTML
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

// ARIA labels
<button 
  aria-label="Close dialog"
  onClick={onClose}
>
  <Icon iconName="Cancel" />
</button>

// Keyboard navigation
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyPress={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleClick();
    }
  }}
>
  Click me
</div>

// Skip links
<a href="#main-content" className="skip-link">
  Skip to main content
</a>
```

## UI Best Practices Checklist

### Design
- [ ] Use consistent spacing (8px grid system)
- [ ] Follow FluentUI design tokens
- [ ] Implement responsive breakpoints
- [ ] Support dark mode
- [ ] Use accessible color contrast (WCAG AA)

### Performance
- [ ] Code splitting for routes
- [ ] Lazy load components
- [ ] Optimize images
- [ ] Minimize bundle size
- [ ] Use production builds

### Accessibility
- [ ] Semantic HTML
- [ ] ARIA attributes where needed
- [ ] Keyboard navigation support
- [ ] Screen reader friendly
- [ ] Focus indicators visible

### Testing
- [ ] Unit tests for components
- [ ] Integration tests for flows
- [ ] E2E tests for critical paths
- [ ] Visual regression testing
- [ ] Accessibility testing
