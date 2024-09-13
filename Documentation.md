# Flipas Project Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Tech Stack](#tech-stack)
3. [Best Practices](#best-practices)
4. [Project Structure](#project-structure)
5. [Key Components](#key-components)
   - [Profiles](#profiles)
   - [Connect](#connect)
   - [Messaging](#messaging)
   - [Matches](#matches)
6. [Getting Started](#getting-started)
7. [Common Tasks](#common-tasks)
8. [Troubleshooting](#troubleshooting)
9. [Learning Resources](#learning-resources)

## Introduction

Flipas is a modern web application designed to revolutionize professional networking and job searching, with a focus on Gen Z users. It provides unique user profiles, networking features, and job-matching capabilities.

## Tech Stack

- **Frontend**: Next.js 14 (React 18) - [Next.js Documentation](https://nextjs.org/docs)

  - Next.js is a React framework that enables features such as server-side rendering and generating static websites.
  - Example of a basic Next.js page:

    ```tsx
    // src/app/[locale]/page.tsx
    import { Metadata } from 'next';
    import { getDictionary } from '@/get-dictionary';
    import { Locale } from '@/i18n-config';

    export const metadata: Metadata = {
      title: 'Flipas',
      description: 'The People's Accelerator',
    };

    export default async function Home({
      params: { locale },
    }: {
      params: { locale: Locale };
    }) {
      const dictionary = await getDictionary(locale);

      return (
        <main>
          <h1>{dictionary.home.title}</h1>
        </main>
      );
    }
    ```

- **Backend**: Node.js (via Next.js API routes) - [Node.js Documentation](https://nodejs.org/en/docs/)

  - We use Next.js API routes to handle server-side logic. Here's an example:

    ```typescript
    // src/app/api/v1/cv/builder/update-work-item/route.ts
    import { NextRequest, NextResponse } from 'next/server';
    import { z } from 'zod';
    import { prisma } from '@/core/database/prisma';
    import { getServerSession } from '@/features/Auth/auth';

    export async function POST(req: NextRequest) {
      const session = await getServerSession();
      if (!session) {
        return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
      }

      const body = await req.json();
      // ... (rest of the function)
    }
    ```

- **Database**: PostgreSQL with Prisma ORM - [Prisma Documentation](https://www.prisma.io/docs/)

  - Prisma is an ORM that makes it easy to interact with our PostgreSQL database.
  - Example of a Prisma query:
    ```typescript
    // src/app/api/v1/login/route.ts
    const user = await prisma.core__user.findUnique({
      where: { email },
      include: {
        core__user_profile: true,
      },
    });
    ```

- **State Management**: React Query/Tanstack Query - [React Query Documentation](https://tanstack.com/query/latest/docs/react/overview)

  - React Query is used for managing server state in our React components.
  - Basic usage example:
    ```typescript
    // src/hooks/useUser.ts
    export const useUser = (userId: string) => {
      return useQuery(['user', userId], () => fetchUser(userId), {
        staleTime: 5 * 60 * 1000, // Data considered fresh for 5 minutes
        cacheTime: 30 * 60 * 1000, // Cache data for 30 minutes
      });
    };
    ```

- **Styling**: Material-UI (MUI) & Emotion - [MUI Documentation](https://mui.com/material-ui/getting-started/overview/)

  - We use MUI components for consistent styling. Example:

    ```tsx
    // src/app/AppWrapper.tsx
    import { ThemeProvider, createTheme } from '@mui/material/styles';

    const theme = createTheme({
      palette: {
        primary: {
          main: '#556cd6',
        },
        secondary: {
          main: '#19857b',
        },
        error: {
          main: '#ff1744',
        },
        background: {
          default: '#fff',
        },
      },
    });

    export default function AppWrapper({ children }: { children: React.ReactNode }) {
      return <ThemeProvider theme={theme}>{children}</ThemeProvider>;
    }
    ```

- **Authentication**: Custom implementation

  - We have a custom authentication system. You'll find related code in `src/features/Auth/`.

- **Internationalization**: middleware.ts & i18nConfig.ts

  - We use a custom internationalization setup. Check these files for more details.

- **Testing**: Jest & React Testing Library - [Jest Documentation](https://jestjs.io/docs/getting-started), [React Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)

  - We write tests using Jest and React Testing Library. Example test:

    ```typescript
    // src/core/filesystem/normalizeFileName.test.ts
    import { normalizeFileName } from './normalizeFileName';

    describe('normalizeFileName', () => {
      it('should remove special characters and replace spaces with underscores', () => {
        const input = 'Hello World! @#$%^&*()';
        const expected = 'Hello_World';
        expect(normalizeFileName(input)).toBe(expected);
      });

      it('should convert to lowercase', () => {
        const input = 'UPPERCASE';
        const expected = 'uppercase';
        expect(normalizeFileName(input)).toBe(expected);
      });

      // ... more tests
    });
    ```

- **Deployment**: Vercel - [Vercel Documentation](https://vercel.com/docs)
  - We deploy our app on Vercel. It's automatically set up to deploy when we push to the main branch.

## Best Practices

1. **TypeScript**: The project uses TypeScript for type safety and better developer experience. [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

   - Always define types for your props and state. Example:

     ```typescript
     // src/components/Profile/ProfileImage.tsx
     type ProfileImageProps = {
       src: string;
       alt: string;
     };

     const ProfileImage: React.FC<ProfileImageProps> = ({ src, alt }) => {
       return <Image src={src} alt={alt} width={200} height={200} />;
     };
     ```

2. **API Routes**: Server-side logic is implemented using Next.js API routes. [Next.js API Routes](https://nextjs.org/docs/api-routes/introduction)

   - API routes go in the `src/app/api/` directory. Each file becomes an API endpoint.

3. **React Query**: Used for efficient data fetching, caching, and state management. [React Query Overview](https://tanstack.com/query/latest/docs/react/overview)

   - Wrap your app with QueryClientProvider and use hooks like useQuery and useMutation.

4. **Component-Based Architecture**: UI is built using reusable React components. [React Components](https://reactjs.org/docs/components-and-props.html)

   - Break your UI into small, reusable components. Example:

     ```tsx
     // src/components/UserList/UserListItem.tsx
     const UserListItem = React.memo(({ user }) => <li>{user.name}</li>);

     // src/components/UserList/UserList.tsx
     const UserList: React.FC<{ users: User[] }> = ({ users }) => (
       <ul>
         {users.map((user) => (
           <UserListItem key={user.id} user={user} />
         ))}
       </ul>
     );
     ```

5. **Custom Hooks**: Complex logic is encapsulated in custom React hooks. [React Hooks](https://reactjs.org/docs/hooks-intro.html)

   - Create hooks for reusable logic. Example:
     ```typescript
     // src/hooks/useUser.ts
     export const useUser = (userId: string) => {
       return useQuery(['user', userId], () => fetchUser(userId), {
         staleTime: 5 * 60 * 1000, // Data considered fresh for 5 minutes
         cacheTime: 30 * 60 * 1000, // Cache data for 30 minutes
       });
     };
     ```

6. **Internationalization**: The app supports multiple languages using a custom dictionary system.

   - Use the provided translation functions in your components.

7. **Environment Variables**: Sensitive data and configuration are managed using environment variables. [Next.js Environment Variables](https://nextjs.org/docs/basic-features/environment-variables)

   - Use `process.env.VARIABLE_NAME` to access environment variables.

8. **Prisma ORM**: Database operations are handled using Prisma for type-safe database access. [Prisma Getting Started](https://www.prisma.io/docs/getting-started)

## Project Structure

├── src/
│ ├── app/ # Next.js 13+ app directory
│ │ ├── [locale]/ # Localized routes
│ │ ├── api/ # API routes
│ │ └── layout.tsx # Root layout component
│ ├── components/ # Reusable React components
│ │ ├── Authenticator/ # Authentication-related components
│ │ ├── Profile/ # Profile-related components
│ │ └── Uploader/ # File upload components
│ ├── features/ # Feature-specific components and logic
│ │ ├── Auth/ # Authentication logic
│ │ ├── Connect/ # Connection feature components
│ │ ├── Conversation/ # Messaging components
│ │ ├── cvBuilder/ # CV building components
│ │ ├── Dictionaries/ # Internationalization dictionaries
│ │ ├── Matches/ # Matching feature components
│ │ ├── OnBoarding/ # User onboarding components
│ │ └── Profile/ # Profile feature components
│ ├── core/ # Core utilities and helpers
│ │ ├── calendar/ # Date and time utilities
│ │ ├── database/ # Database-related utilities
│ │ ├── filesystem/ # File system utilities
│ │ ├── routing/ # Routing utilities
│ │ └── string/ # String manipulation utilities
│ ├── hooks/ # Custom React hooks
│ ├── query/ # React Query related files
│ ├── types/ # TypeScript type definitions
│ └── ui/ # UI-specific components
├── public/ # Static assets
├── prisma/ # Prisma schema and migrations
│ ├── schema.prisma # Database schema
│ └── migrations/ # Database migrations
├── package.json
├── next.config.js # Next.js configuration
├── tsconfig.json # TypeScript configuration
└── .env # Environment variables (not in version control)

## Key Concepts

1. **TypeScript**: We use TypeScript for type safety. Here's a basic example:

   ```typescript
   // src/types/cv.ts
   export type CvBuilderItem = {
     code: string;
     name: string;
     selected: boolean;
     level: number;
   };

   export type CvBuilderLanguageItem = CvBuilderItem & {
     level?: number;
   };
   ```

2. **React Components**: We use functional components with hooks. Example:

   ```tsx
   // src/features/Profile/ProfileIntro.tsx
   import { FC } from 'react';
   import { Card, CardContent, Typography, Box, IconButton } from '@mui/material';
   import { Edit as EditIcon } from '@mui/icons-material';
   import { Profile } from '@/types/profile';

   type Props = {
     userProfile: Profile;
     onEdit?: () => void;
   };

   export const ProfileIntro: FC<Props> = ({ userProfile, onEdit }) => {
     const introText = userProfile.intro || 'No introduction provided.';

     return (
       <Card sx={{ mt: 2 }}>
         <CardContent>
           <Box display="flex" justifyContent="space-between" alignItems="center">
             <Typography variant="h6" component="h2">
               Intro
             </Typography>
             {onEdit && (
               <IconButton onClick={onEdit} size="small">
                 <EditIcon />
               </IconButton>
             )}
           </Box>
           <Box mt={1}>
             <Typography variant="body1" color="text.secondary">
               {introText}
             </Typography>
           </Box>
         </CardContent>
       </Card>
     );
   };
   ```

3. **API Routes**: Server-side logic is implemented in the `src/app/api` directory. Example:

   ```typescript
   // src/app/api/v1/cv/builder/route.ts
   import { NextRequest, NextResponse } from 'next/server';
   import { getMyProfile } from '@/features/user/getMyProfile';
   import { prisma } from '../../../../../../prisma/prisma';

   export const GET = async (request: NextRequest): Promise<NextResponse<CvBuilderStatusResponse>> => {
     const profile = await getMyProfile();

     const relations = await prisma.core__profile_rel_label.findMany({
       select: { label_id: true, level: true },
       where: { profile_id: profile.id },
     });

     // ... (rest of the implementation)

     return NextResponse.json({
       language: processItem(1),
       'soft-skill': processItem(2),
       'hard-skill': processItem(3),
       // ... (other fields)
     });
   };
   ```

4. **React Query**: We use React Query for data fetching and caching. Example:

   ```tsx
   // src/query/useCvBuilderQuery.ts
   import { CvBuilderStatusResponse } from '@/types/cv';
   import { fetchData } from '@/features/api/fetchData';
   import { useQuery } from '@tanstack/react-query';

   export const useCvBuilderQuery = () =>
     useQuery({
       queryKey: ['cvBuilder'],
       staleTime: Infinity,
       refetchInterval: 60_000,
       queryFn: async (): Promise<CvBuilderStatusResponse> => {
         const data = await fetchData<CvBuilderStatusResponse>(`/api/v1/cv/builder`);

         return {
           ...data,
           educations: data.educations.map((i) => ({
             ...i,
             startDate: i.startDate ? new Date(i.startDate) : undefined,
             endDate: i.endDate ? new Date(i.endDate) : undefined,
           })),
           work_experiences: data.work_experiences.map((i) => ({
             ...i,
             startDate: i.startDate ? new Date(i.startDate) : undefined,
             endDate: i.endDate ? new Date(i.endDate) : undefined,
           })),
         };
       },
       enabled: true,
     });
   ```

5. **Internationalization**: We use a custom dictionary system. Example:

   ```typescript
   // src/features/Dictionaries/dictionaryEn.ts
   export const DICTIONARY_EN = {
     common: {
       userEmail: 'Username or E-mail',
       usernm: 'Username',
       pw: 'Password',
       email: 'Email',
       haveAcc: 'Already have an Account?',
       login: 'Login',
       signIn: 'Sign In',
       signup: 'Sign Up',
       // ... other translations
     },
     // ... other categories
   };
   ```

   usage in components:

   ```tsx
   // src/features/Login/LoginForm.tsx
   import { useDictionary } from '@/features/Dictionaries/DictionaryProvider';

   export const LoginForm = () => {
     const d = useDictionary();

     return (
       <form onSubmit={handleSubmit(onSubmit)}>
         <TextField
           variant="standard"
           autoComplete="off"
           label={d.common.usernm}
           // ... other props
         />
         {/* ... other form fields */}
       </form>
     );
   };
   ```

6. API Routes: Our API routes are implemented using Next.js API routes and follow a RESTful structure. Here's a more complex example:

```typescript
// src/app/api/v1/connect/profile-list/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { selectPossibleMatchProfileIdsToConnect } from '@/features/user/selectPossibleMatchProfileIdsToConnect';
import { selectLabelByProfileIds } from '@/features/user/selectLabelByProfileIds';
import { selectSimpleProfileList } from '@/features/user/selectSimpleProfileList';
import { resolveBlobUrlByPath } from '@/features/blob/resolveBlobUrlByPath';
import { getMyUserId } from '@/features/user/getMyUserId';
import { randomInt } from 'crypto';

export const GET = async (request: NextRequest): Promise<NextResponse<ConnectProfileListResponse>> => {
  const userId = await getMyUserId();

  const profileList = await selectSimpleProfileList(await selectPossibleMatchProfileIdsToConnect(userId));
  const labelList = await selectLabelByProfileIds(profileList.map((p) => p.profile_id));

  return NextResponse.json({
    items: await Promise.all(
      profileList.map(
        async (profile): Promise<ConnectProfileItem> => ({
          username: profile.username,
          first_name: profile.first_name || '',
          last_name: profile.last_name || '',
          intro: profile.intro || '',
          avatarUrl: resolveBlobUrlByPath(profile.blob_path || '') || undefined,
          compatibilityPercentage: randomInt(0, 100), // TODO: Implement actual compatibility calculation
          labelList: labelList.filter((l) => l.profileId === profile.profile_id),
        })
      )
    ),
  });
};
```

This API route demonstrates how we handle complex data fetching and processing for the Connect feature.

7. Custom Hooks: We extensively use custom hooks to encapsulate complex logic. Here's an example of a custom hook for user authentication:

```typescript
// src/hooks/useUserStatusQuery.ts
import { UserStatusResponse } from '@/types/user';
import { useRouter, usePathname } from 'next/navigation';
import { fetchData } from '@/features/api/fetchData';
import { useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';

export const useUserStatusQuery = () => {
  const router = useRouter();
  const pathname = usePathname();
  const query = useQuery({
    queryKey: ['userStatus'],
    staleTime: Infinity,
    refetchInterval: 10000,
    queryFn: async () => fetchData<UserStatusResponse>(`/api/v1/user/status`),
    enabled: true,
  });

  const data = query.data;
  const isLoggedIn = data?.isLoggedIn;
  const isProfileComplete = data?.isLoggedIn ? data.isProfileComplete : false;

  useEffect(() => {
    if (!isLoggedIn || isProfileComplete) return;
    const isOnboardingPage = pathname === '/onboarding';
    if (isOnboardingPage) return;
    router.push('/onboarding');
  }, [pathname, router, isLoggedIn, isProfileComplete]);

  return query;
};
```

This hook manages user authentication state and handles redirects for incomplete profiles.

## Key Components

### Profiles

The profile system is built around the Profile type:

```typescript
// src/types/profile.ts
export type Profile = {
  id: number;
  isMyProfile: boolean;
  username: string;
  firstName?: string;
  lastName?: string;
  name?: string;
  avatarUrl?: string;
  facebookUrl?: string;
  instagramUrl?: string;
  twitterUrl?: string;
  intro?: string;
  education?: ProfileEducationRecord[];
  workExperience?: ProfileWorkExperienceRecord[];
  quote?: string;
  labelList: ProfileLabelItem[];
};
```

- Located in `src/features/Profile/`
- Key files:
  - `ProfileContainer.tsx`: Main profile component
  - `EditProfile.tsx`: Profile editing functionality
  - `ProfileLabelList.tsx`: Displays user skills and attributes

Profiles are central to the Flipas experience. They showcase user information, skills, and professional experiences. The profile system is designed to be more engaging and informative than traditional CVs.

The EditProfile component allows users to modify their profile:

```typescript
// src/features/Profile/EditProfile.tsx
export const EditProfile: FC<Props> = ({ profile }) => {
  const router = useRouter();
  return (
    <RequireUserLoggedInWrapper>
      <ProfileOverview userProfile={profile} />
      <ProfileIntro onEdit={() => router.push('/cv/intro?edit=1')} userProfile={profile} />
      <ProfileLabelList
        labelList={profile.labelList}
        onEdit={(categoryCode: LabelCategoryType) => {
          router.push(`/cv/${categoryCode}?edit=1`);
        }}
      />
      <ProfileEducation onEdit={() => router.push('/cv/education?edit=1')} userProfile={profile} />
      <ProfileWorkExperience onEdit={() => router.push('/cv/work-experience?edit=1')} userProfile={profile} />
      <ProfileLinks onEdit={() => router.push('/cv/links?edit=1')} userProfile={profile} />
      <ProfileQuote userProfile={profile} onEdit={() => router.push('/cv/quote?edit=1')} />
    </RequireUserLoggedInWrapper>
  );
};
```

### Connect

The Connect feature uses a stack-based UI for browsing potential connections:

```typescript
// src/features/Connect/ConnectStack.tsx
export const ConnectStack = () => {
  const d = useDictionary();
  const { loadedProfiles, isLoaded, loadNextProfile } = useConnectProfileList();

  return loadedProfiles.length > 0 ? (
    <Box>
      <ConnectProfileDetail profile={loadedProfiles[0]} loadNextProfile={loadNextProfile} />
    </Box>
  ) : isLoaded ? (
    <Box sx={{ my: 5 }}>{d.connect.noProfiles}</Box>
  ) : (
    <Box sx={{ py: 2, textAlign: 'center' }}>
      <CircularProgress />
    </Box>
  );
};
```

The Connect feature allows users to discover and connect with other professionals. It implements a card-stack interface similar to
popular dating apps but focused on professional networking.

- Located in `src/features/Connect/`
- Key files:
  - `ConnectStack.tsx`: Main component for browsing potential connections
  - `ConnectProfileDetail.tsx`: Detailed view of a potential connection
  - `ConnectProfileCard.tsx`: Card view of a potential connection

The Connect feature allows users to discover and connect with other professionals. It implements a card-stack interface similar to popular dating apps but focused on professional networking.

### Messaging

```typescript
// src/features/Conversation/ConversationWriteMessage.tsx
export const ConversationWriteMessage: FC<Props> = ({ roomCode, refetch }) => {
  const [message, setMessage] = useState('');

  const handleSend = (event: React.FormEvent) => {
    event.preventDefault();
    setMessage('');
    fetchData(`/api/v1/message/send-message`, { postData: { roomCode, message } }).then(() => {
      document.getElementById(MESSAGE_BOX_ID)?.focus();
      refetch();
    });
  };

  return (
    <form onSubmit={handleSend}>
      <Box sx={{ display: 'flex' }}>
        <Box sx={{ width: '100%' }}>
          <TextField
            id={MESSAGE_BOX_ID}
            multiline
            minRows={1}
            maxRows={Infinity}
            value={message}
            onChange={(e) => setMessage(e.target.value)}
            onKeyDown={handleKeyDown}
            fullWidth
          />
        </Box>
        <Box>
          <ButtonBase type="submit" sx={{ p: 1 }}>
            <SendIcon />
          </ButtonBase>
        </Box>
      </Box>
    </form>
  );
};
```

- Located in `src/features/Conversation/`
- Key files:
  - `Conversation.tsx`: Main conversation component
  - `ConversationMessage.tsx`: Individual message component
  - `ConversationWriteMessage.tsx`: Message input component

The messaging system allows users to communicate with their connections. It supports real-time updates and provides a smooth user experience.

### Matches

- Located in `src/features/Matches/`
- Key files:
  - `MatchesList.tsx`: Displays a list of user matches
  - `MatchesWrapper.tsx`: Wrapper component for matches functionality

The Matches feature shows users who have mutually connected with each other, enabling further interaction and networking.

## Getting Started

1. Clone the repository
2. Install yarn: `npm install -g yarn`
3. Install dependencies: `yarn install`
4. Build the project: `yarn build`
5. Run the development server: `yarn dev`
6. Open `http://localhost:3000` in your browser

## Database Schema

The database schema is defined in `prisma/schema.prisma`. It includes tables for users, profiles, messages, and other relevant entities.

```typescript
// prisma/schema.prisma
model core__user {
  id                 Int      @id @default(autoincrement())
  username           String   @unique
  email              String   @unique
  password           String
  name               String
  email_verified_date DateTime?
  // ... other fields and relations
}

model core__profile {
  id            Int      @id @default(autoincrement())
  user_id       Int
  intro         String?
  quote         String?
  // ... other fields and relations
}

model core__message {
  id            BigInt   @id @default(autoincrement())
  room_id       BigInt
  user_id       Int
  message       String
  inserted_date DateTime
  // ... relations
}

// ... other models
```

To update the schema:
Modify prisma/schema.prisma
Run npx prisma migrate dev --name your_migration_name
Apply the migration to your development database

##Authentication flow
The authentication flow in Flipas involves several steps:

1. **Sign Up**:

   - User fills out the sign-up form (`SignUpForm.tsx`)
   - Form data is sent to `/api/v1/signup` (`src/app/api/v1/signup/route.ts`)
   - Server creates a new user record with a verification token
   - Verification email is sent to the user

   Example from `SignUpForm.tsx`:

   ```typescript
   const onSubmit = (data: SignUpFormSchema) => {
     fetchData<SignUpResponse>(`/api/v1/signup`, { postData: data }).then(({ success }) => {
       if (success) alert('Signup success!');
       router.push('/signup/check-email');
     });
   };
   ```

2. **Email Verification**:

   - User clicks the verification link in the email
   - Request is sent to `/api/v1/signup/verify` (`src/app/api/v1/signup/verify/route.ts`)
   - Server verifies the token and marks the user's email as verified
   - User is redirected to their profile page

   Example from `verify/route.ts`:

   ```typescript
   await prisma.core__user.update({
     where: { id: user.id },
     data: {
       email_verified_date: new Date(),
       verification_token: null,
     },
   });
   ```

3. **Login**:

   - User enters credentials on the login page
   - Credentials are sent to `/api/v1/login` (`src/app/api/v1/login/route.ts`)
   - Server verifies the credentials and creates a new session
   - User is redirected to their profile or onboarding page

   Example from `login/route.ts`:

   ```typescript
   const token = randomStringGenerator(32, { case: 'random' });
   await prisma.core__user_session.create({
     data: {
       user_id: user.id,
       token,
       inserted_date: new Date(),
     },
   });
   ```

4. **Session Management**:

   - After successful login or signup, a session token is set in an HTTP-only cookie
   - This token is used for subsequent authenticated requests

   Example of setting the cookie:

   ```typescript
   cookies().set({
     name: AUTH_COOKIES_NAME,
     value: token,
     secure: true,
     httpOnly: true,
     path: '/',
     expires: new Date(new Date().setMonth(new Date().getMonth() + 3)),
   });
   ```

5. **Protected Routes**:

   - Certain pages (e.g., profile page) are protected and require authentication
   - The `RequireUserLoggedInWrapper` component is used to wrap these pages
   - If the user is not authenticated, they are redirected to the login page

   Example from `signup/page.tsx`:

   ```typescript
   <RequireUserLoggedInWrapper login redirectTo="/my-profile">
     {/* Protected content */}
   </RequireUserLoggedInWrapper>
   ```

This flow ensures secure user authentication and authorization throughout the application.

## Common Tasks

1. **Adding a new component**:

   - Create a new file in the appropriate directory under `src/components/` or `src/features/`
   - Use TypeScript and functional components with hooks
   - Export the component as default
   - Import and use the component where needed

2. **Creating a new API route**:

   - Add a new file in `src/app/api/`
   - Implement the necessary HTTP methods (GET, POST, etc.)
   - Use Prisma for database operations if required

3. **Adding a new page**:

   - Create a new file in `src/app/`
   - Use the appropriate Next.js conventions for routing

4. **Updating the database schema**:

   - Modify the schema in `prisma/schema.prisma`
   - Run `npx prisma migrate dev` to create a new migration
   - Apply the migration to your development database

5. **Adding a new translation**:
   - Update the dictionary files in `src/features/Dictionaries/`
   - Use the new translation key in your components

##Performance Optimization

1. Code Splitting: Next.js automatically code-splits your application. Ensure you're using dynamic imports for large components or libraries:

   Example from `src/app/[locale]/page.tsx`:

   ```typescript
   const DynamicComponent = dynamic(() => import('@/components/HeavyComponent'), {
     loading: () => <p>Loading...</p>,
   });
   ```

2. Image Optimization: Use Next.js Image component for automatic image optimization:

   Example from `src/components/Profile/ProfileImage.tsx`:

   ```tsx
   import Image from 'next/image';

   const ProfileImage = ({ src, alt }) => <Image src={src} alt={alt} width={200} height={200} />;
   ```

3. Memoization: Use React.memo for components that render often with the same props:

   Example from `src/components/UserList/UserListItem.tsx`:

   ```tsx
   const UserListItem = React.memo(({ user }) => <li>{user.name}</li>);
   ```

4. Lazy Loading: Implement lazy loading for images and components that are not immediately visible:

   Example using Intersection Observer API:

   ```tsx
   const LazyLoadedImage = () => {
     const [isVisible, setIsVisible] = useState(false);
     const imgRef = useRef();

     useEffect(() => {
       const observer = new IntersectionObserver((entries) => {
         if (entries[0].isIntersecting) {
           setIsVisible(true);
           observer.disconnect();
         }
       });

       if (imgRef.current) {
         observer.observe(imgRef.current);
       }

       return () => {
         if (imgRef.current) {
           observer.unobserve(imgRef.current);
         }
       };
     }, []);

     return <div ref={imgRef}>{isVisible && <Image src="/large-image.jpg" alt="Large Image" />}</div>;
   };
   ```

5. Caching: Utilize React Query's caching capabilities to reduce unnecessary network requests:

   Example from `src/hooks/useUser.ts`:

   ```typescript
   export const useUser = (userId: string) => {
     return useQuery(['user', userId], () => fetchUser(userId), {
       staleTime: 5 * 60 * 1000, // Data considered fresh for 5 minutes
       cacheTime: 30 * 60 * 1000, // Cache data for 30 minutes
     });
   };
   ```

These optimizations should significantly improve the performance of the Flipas project, providing a smoother user experience and faster load times.

## Troubleshooting

1. **Build errors**:

   - Ensure all dependencies are installed: `yarn install`
   - Clear the Next.js cache: `rm -rf .next`
   - Rebuild the project: `yarn build`

2. **Database connection issues**:

   - Check your database URL in the `.env` file
   - Ensure your database is running and accessible
   - Verify your Prisma schema matches your database schema

3. **API route errors**:

   - Check the API route implementation
   - Verify the correct HTTP method is being used
   - Check for any middleware that might be affecting the route

4. **React Query issues**:

   - Ensure the QueryClient is properly set up in your app
   - Check that query keys are unique and consistent
   - Verify that the query function is returning the expected data

5. **Styling problems**:
   - Check that MUI components are being used correctly
   - Verify that custom styles are being applied properly
   - Ensure theme variables are being used consistently

## Learning Resources

1. **Next.js**:

   - [Next.js Documentation](https://nextjs.org/docs)
   - [Learn Next.js](https://nextjs.org/learn)

2. **React**:

   - [React Documentation](https://reactjs.org/docs/getting-started.html)
   - [React Hooks](https://reactjs.org/docs/hooks-intro.html)

3. **TypeScript**:

   - [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
   - [TypeScript with React](https://www.typescriptlang.org/docs/handbook/react.html)

4. **Prisma**:

   - [Prisma Documentation](https://www.prisma.io/docs/)
   - [Prisma with Next.js](https://www.prisma.io/nextjs)

5. **React Query**:

   - [React Query Documentation](https://tanstack.com/query/latest/docs/react/overview)
   - [React Query with TypeScript](https://tanstack.com/query/latest/docs/react/typescript)

6. **Material-UI**:

   - [MUI Documentation](https://mui.com/material-ui/getting-started/overview/)
   - [MUI with Next.js](https://mui.com/material-ui/guides/next-js/)

7. **Testing**:
   - [Jest Documentation](https://jestjs.io/docs/getting-started)
   - [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)

Remember to consult these resources when you encounter challenges or need to implement new features. They provide valuable insights and best practices for working with our tech stack.
