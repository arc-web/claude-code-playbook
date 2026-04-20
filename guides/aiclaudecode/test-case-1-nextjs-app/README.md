# Next.js 14 Full-Stack Application

A modern Next.js 14 application with App Router, TypeScript, Prisma ORM, PostgreSQL, and NextAuth.js v5 authentication.

## Features

- ✅ Next.js 14 with App Router
- ✅ TypeScript throughout
- ✅ Prisma ORM with PostgreSQL
- ✅ NextAuth.js v5 authentication
- ✅ API routes for user management
- ✅ Blog system with CRUD operations
- ✅ Protected routes and middleware
- ✅ Proper error handling and loading states

## Project Structure

```
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   └── profile/
│   ├── api/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── posts/
│   │   └── comments/
│   ├── blog/
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── components/
├── lib/
├── prisma/
├── types/
└── middleware.ts
```

## Setup Instructions

### 1. Install Dependencies

```bash
npm install next@14 react@18 react-dom@18 typescript @types/node @types/react @types/react-dom
npm install prisma @prisma/client
npm install next-auth@5 @auth/prisma-adapter
npm install bcryptjs @types/bcryptjs
npm install zod react-hook-form @hookform/resolvers
npm install @tailwindcss/typography tailwindcss postcss autoprefixer
```

### 2. Environment Variables

Create `.env.local`:

```env
# Database
DATABASE_URL="postgresql://username:password@localhost:5432/nextjs_app"

# NextAuth.js
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key"

# OAuth Providers (optional)
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
```

### 3. Database Setup

```bash
# Initialize Prisma
npx prisma init

# Generate Prisma client
npx prisma generate

# Run migrations
npx prisma migrate dev --name init

# Seed database (optional)
npx prisma db seed
```

### 4. Run Development Server

```bash
npm run dev
```

## Key Implementation Details

### Prisma Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  password      String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  accounts Account[]
  sessions Session[]
  posts    Post[]
  comments Comment[]
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

model Post {
  id        String    @id @default(cuid())
  title     String
  content   String
  published Boolean   @default(false)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  authorId  String
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments  Comment[]
}

model Comment {
  id        String   @id @default(cuid())
  content   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  authorId  String
  postId    String
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
}
```

### NextAuth.js Configuration

```typescript
// lib/auth.ts
import NextAuth from "next-auth"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { prisma } from "@/lib/prisma"
import GoogleProvider from "next-auth/providers/google"
import CredentialsProvider from "next-auth/providers/credentials"
import bcrypt from "bcryptjs"

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    CredentialsProvider({
      name: "credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null
        }

        const user = await prisma.user.findUnique({
          where: { email: credentials.email }
        })

        if (!user || !user.password) {
          return null
        }

        const isPasswordValid = await bcrypt.compare(
          credentials.password,
          user.password
        )

        if (!isPasswordValid) {
          return null
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          image: user.image,
        }
      }
    })
  ],
  session: {
    strategy: "jwt"
  },
  pages: {
    signIn: "/login",
  },
  callbacks: {
    async session({ session, token }) {
      if (token.sub && session.user) {
        session.user.id = token.sub
      }
      return session
    },
  }
})
```

### API Routes

```typescript
// app/api/posts/route.ts
import { NextRequest, NextResponse } from "next/server"
import { auth } from "@/lib/auth"
import { prisma } from "@/lib/prisma"
import { z } from "zod"

const postSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(1),
  published: z.boolean().default(false),
})

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url)
    const page = parseInt(searchParams.get("page") || "1")
    const limit = parseInt(searchParams.get("limit") || "10")
    const skip = (page - 1) * limit

    const posts = await prisma.post.findMany({
      where: { published: true },
      include: {
        author: {
          select: { name: true, image: true }
        },
        _count: {
          select: { comments: true }
        }
      },
      orderBy: { createdAt: "desc" },
      skip,
      take: limit,
    })

    const total = await prisma.post.count({
      where: { published: true }
    })

    return NextResponse.json({
      posts,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    })
  } catch (error) {
    return NextResponse.json(
      { error: "Failed to fetch posts" },
      { status: 500 }
    )
  }
}

export async function POST(request: NextRequest) {
  try {
    const session = await auth()
    if (!session?.user?.id) {
      return NextResponse.json(
        { error: "Unauthorized" },
        { status: 401 }
      )
    }

    const body = await request.json()
    const validatedData = postSchema.parse(body)

    const post = await prisma.post.create({
      data: {
        ...validatedData,
        authorId: session.user.id,
      },
      include: {
        author: {
          select: { name: true, image: true }
        }
      }
    })

    return NextResponse.json(post, { status: 201 })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "Invalid data", details: error.errors },
        { status: 400 }
      )
    }
    return NextResponse.json(
      { error: "Failed to create post" },
      { status: 500 }
    )
  }
}
```

### Middleware for Route Protection

```typescript
// middleware.ts
import { auth } from "@/lib/auth"
import { NextResponse } from "next/server"
import type { NextRequest } from "next/server"

export default auth((req) => {
  const isLoggedIn = !!req.auth
  const { pathname } = req.nextUrl

  // Protected routes
  if (pathname.startsWith("/dashboard") && !isLoggedIn) {
    return NextResponse.redirect(new URL("/login", req.url))
  }

  // Auth routes (redirect if already logged in)
  if (pathname.startsWith("/login") && isLoggedIn) {
    return NextResponse.redirect(new URL("/dashboard", req.url))
  }

  return NextResponse.next()
})

export const config = {
  matcher: ["/((?!api|_next/static|_next/image|favicon.ico).*)"]
}
```

### React Components

```typescript
// components/PostCard.tsx
"use client"

import { useState } from "react"
import Link from "next/link"
import { formatDistanceToNow } from "date-fns"

interface PostCardProps {
  post: {
    id: string
    title: string
    content: string
    createdAt: Date
    author: {
      name: string | null
      image: string | null
    }
    _count: {
      comments: number
    }
  }
}

export function PostCard({ post }: PostCardProps) {
  const [isLiked, setIsLiked] = useState(false)

  return (
    <article className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <div className="flex items-center space-x-3 mb-4">
        {post.author.image && (
          <img
            src={post.author.image}
            alt={post.author.name || "Author"}
            className="w-10 h-10 rounded-full"
          />
        )}
        <div>
          <p className="font-medium text-gray-900">
            {post.author.name || "Anonymous"}
          </p>
          <p className="text-sm text-gray-500">
            {formatDistanceToNow(new Date(post.createdAt), { addSuffix: true })}
          </p>
        </div>
      </div>

      <Link href={`/blog/${post.id}`}>
        <h2 className="text-xl font-bold text-gray-900 mb-2 hover:text-blue-600 transition-colors">
          {post.title}
        </h2>
      </Link>

      <p className="text-gray-600 mb-4 line-clamp-3">
        {post.content}
      </p>

      <div className="flex items-center justify-between">
        <div className="flex items-center space-x-4">
          <button
            onClick={() => setIsLiked(!isLiked)}
            className={`flex items-center space-x-1 text-sm ${
              isLiked ? "text-red-500" : "text-gray-500"
            } hover:text-red-500 transition-colors`}
          >
            <span>{isLiked ? "❤️" : "🤍"}</span>
            <span>Like</span>
          </button>
          <Link
            href={`/blog/${post.id}#comments`}
            className="flex items-center space-x-1 text-sm text-gray-500 hover:text-blue-500 transition-colors"
          >
            <span>💬</span>
            <span>{post._count.comments} comments</span>
          </Link>
        </div>
      </div>
    </article>
  )
}
```

## Deployment

### Vercel Deployment

1. Push your code to GitHub
2. Connect your repository to Vercel
3. Add environment variables in Vercel dashboard
4. Deploy automatically

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json package-lock.json* ./
RUN npm ci --only=production

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Generate Prisma client
RUN npx prisma generate

# Build the application
RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

## Testing

```typescript
// __tests__/api/posts.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { createMocks } from 'node-mocks-http'
import { GET, POST } from '@/app/api/posts/route'

describe('/api/posts', () => {
  it('GET returns posts with pagination', async () => {
    const { req } = createMocks({
      method: 'GET',
      url: '/api/posts?page=1&limit=5',
    })

    const response = await GET(req)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data).toHaveProperty('posts')
    expect(data).toHaveProperty('pagination')
    expect(Array.isArray(data.posts)).toBe(true)
  })

  it('POST creates a new post', async () => {
    const { req } = createMocks({
      method: 'POST',
      body: {
        title: 'Test Post',
        content: 'Test content',
        published: false,
      },
    })

    const response = await POST(req)
    const data = await response.json()

    expect(response.status).toBe(201)
    expect(data.title).toBe('Test Post')
    expect(data.content).toBe('Test content')
  })
})
```

This implementation demonstrates the latest Next.js 14 patterns, proper TypeScript usage, modern authentication with NextAuth.js v5, and comprehensive API design with Prisma ORM.
