# Context7 Integration Guide for Cursor

## What is Context7?

Context7 is an MCP (Model Context Protocol) server that provides **up-to-date, version-specific documentation and code examples** for thousands of software libraries and frameworks. It solves the problem of LLMs relying on outdated training data by fetching current documentation directly from the source.

### Key Benefits:
- ✅ **Real-time documentation**: Always up-to-date with the latest versions
- ✅ **Version-specific examples**: Code that actually works with current versions
- ✅ **No hallucinated APIs**: All examples are verified against actual documentation
- ✅ **Comprehensive coverage**: Supports thousands of libraries across all major ecosystems

## Installation in Cursor

### Method 1: One-Click Installation (Recommended)

Click this button for instant installation:
[![Install MCP Server](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=context7&config=eyJjb21tYW5kIjoibnB4IC15IEB1cHN0YXNoL2NvbnRleHQ3LW1jcCJ9)

### Method 2: Manual Configuration

1. **Open Cursor Settings**: Go to `Settings` → `Cursor Settings` → `MCP` → `Add new global MCP server`

2. **Add Configuration**: Paste this into your `~/.cursor/mcp.json` file:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### Method 3: Project-Specific Installation

Create `.cursor/mcp.json` in your project folder for project-specific configuration.

## How to Use Context7

### Basic Usage

Simply add `use context7` to your prompts in Cursor:

```
Create a Next.js middleware that checks for a valid JWT in cookies and redirects unauthenticated users to `/login`. use context7
```

```
Configure a Cloudflare Worker script to cache JSON API responses for five minutes. use context7
```

```
Build a React component with TypeScript that fetches data from an API and displays it in a table. use context7
```

### Advanced Usage Patterns

#### 1. **Framework-Specific Requests**
```
Create a FastAPI endpoint with SQLAlchemy ORM that handles user authentication. use context7
```

#### 2. **Library Integration**
```
Show me how to integrate Stripe payments with a Next.js app using the latest Stripe SDK. use context7
```

#### 3. **Version-Specific Code**
```
Write a React component using React 18's new concurrent features and Suspense. use context7
```

#### 4. **Full-Stack Integration**
```
Create a full-stack app with Next.js 14, Prisma ORM, and PostgreSQL with authentication. use context7
```

## Supported Libraries and Frameworks

Context7 supports thousands of libraries across all major ecosystems:

### Frontend Frameworks
- **React** (including React 18+ features)
- **Vue.js** (Vue 3 Composition API)
- **Angular** (latest versions)
- **Svelte/SvelteKit**
- **Next.js** (App Router, Server Components)
- **Nuxt.js**
- **Remix**
- **Astro**

### Backend Frameworks
- **Node.js/Express**
- **FastAPI** (Python)
- **Django** (Python)
- **Flask** (Python)
- **Spring Boot** (Java)
- **ASP.NET Core** (C#)
- **Ruby on Rails**
- **Laravel** (PHP)

### Databases & ORMs
- **Prisma** (TypeScript/JavaScript)
- **SQLAlchemy** (Python)
- **Entity Framework** (C#)
- **Sequelize** (JavaScript)
- **Mongoose** (MongoDB)
- **Drizzle ORM**

### Cloud & Infrastructure
- **AWS SDK** (various languages)
- **Google Cloud Platform**
- **Azure SDK**
- **Vercel**
- **Netlify**
- **Cloudflare Workers**
- **Docker**
- **Kubernetes**

### Authentication & Security
- **NextAuth.js**
- **Auth.js**
- **Firebase Auth**
- **Supabase Auth**
- **Stripe**
- **PayPal**

### UI Libraries
- **Tailwind CSS**
- **Material-UI**
- **Chakra UI**
- **Ant Design**
- **Bootstrap**
- **Framer Motion**

## Integration Strategies for Software Stacks

### 1. **Modern Web Development Stack**

**Prompt:**
```
Create a modern web app with Next.js 14, TypeScript, Tailwind CSS, Prisma ORM, PostgreSQL, and NextAuth.js authentication. Include proper error handling and loading states. use context7
```

**Benefits:**
- Gets latest Next.js 14 App Router patterns
- TypeScript configuration best practices
- Prisma schema and migration examples
- NextAuth.js v5 configuration

### 2. **Full-Stack Python Stack**

**Prompt:**
```
Build a FastAPI backend with SQLAlchemy ORM, Alembic migrations, Pydantic models, and JWT authentication. Include Docker configuration and deployment setup. use context7
```

**Benefits:**
- Latest FastAPI patterns and best practices
- SQLAlchemy 2.0 async patterns
- Proper Pydantic v2 model definitions
- Docker multi-stage builds

### 3. **Mobile-First React Stack**

**Prompt:**
```
Create a React Native app with Expo, TypeScript, React Navigation, AsyncStorage, and integration with a REST API. Include proper state management with Zustand. use context7
```

**Benefits:**
- Latest Expo SDK features
- React Navigation v6 patterns
- Modern state management approaches
- TypeScript best practices for mobile

### 4. **Microservices Architecture**

**Prompt:**
```
Design a microservices architecture with Node.js/Express, Docker containers, Redis caching, message queues with RabbitMQ, and API gateway with Kong. use context7
```

**Benefits:**
- Service discovery patterns
- Inter-service communication
- Caching strategies
- API gateway configuration

## Best Practices for Using Context7

### 1. **Be Specific in Your Requests**
```
❌ "Create a React app"
✅ "Create a React 18 app with TypeScript, Vite, and Tailwind CSS for a todo list with local storage persistence"
```

### 2. **Request Complete Examples**
```
❌ "How to use Prisma?"
✅ "Show me a complete Prisma setup with PostgreSQL, including schema definition, migrations, and CRUD operations with proper error handling"
```

### 3. **Ask for Integration Patterns**
```
❌ "How to add authentication?"
✅ "Integrate NextAuth.js v5 with Next.js 14 App Router, including protected routes, middleware, and database session storage"
```

### 4. **Request Modern Patterns**
```
❌ "Create a form"
✅ "Create a React Hook Form with Zod validation, TypeScript, and proper error handling for a user registration form"
```

## Troubleshooting

### Common Issues:

1. **MCP Server Not Starting**
   - Ensure Node.js >= v18.0.0 is installed
   - Check your `~/.cursor/mcp.json` configuration
   - Try running `npx @upstash/context7-mcp` manually to test

2. **No Results Found**
   - Be more specific in your search terms
   - Try alternative library names
   - Check if the library is supported by searching on context7.com

3. **Outdated Information**
   - Context7 updates automatically, but you can request specific versions
   - Use version-specific queries when needed

### Alternative Installation Methods:

**Using Bun:**
```json
{
  "mcpServers": {
    "context7": {
      "command": "bunx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

**Using Deno:**
```json
{
  "mcpServers": {
    "context7": {
      "command": "deno",
      "args": ["run", "--allow-env=NO_DEPRECATION,TRACE_DEPRECATION", "--allow-net", "npm:@upstash/context7-mcp"]
    }
  }
}
```

## Advanced Integration Examples

### 1. **AI-Powered Development Workflow**

Combine Context7 with other AI tools:

```
Create a Next.js 14 app with the following features:
- App Router with TypeScript
- Tailwind CSS for styling
- Prisma ORM with PostgreSQL
- NextAuth.js for authentication
- API routes for CRUD operations
- Proper error boundaries and loading states
- Unit tests with Jest and React Testing Library
- E2E tests with Playwright
- Docker configuration for development and production
- GitHub Actions CI/CD pipeline

Include proper documentation and README. use context7
```

### 2. **Legacy System Modernization**

```
Help me modernize a legacy PHP application by:
- Converting to a modern PHP framework (Laravel 10)
- Implementing API-first architecture
- Adding TypeScript frontend with React
- Setting up Docker containerization
- Implementing proper testing strategies
- Adding monitoring and logging
- Creating migration documentation

use context7
```

### 3. **Cross-Platform Development**

```
Create a cross-platform mobile app with:
- React Native with Expo
- TypeScript throughout
- State management with Zustand
- Navigation with React Navigation v6
- Offline-first with AsyncStorage and sync
- Push notifications
- Deep linking
- App store deployment configuration

use context7
```

## Resources and Links

- **Official Website**: https://context7.com
- **GitHub Repository**: https://github.com/upstash/context7
- **MCP Documentation**: https://modelcontextprotocol.io
- **Cursor MCP Docs**: https://docs.cursor.com/context/model-context-protocol
- **Adding Projects Guide**: https://github.com/upstash/context7/blob/main/docs/adding-projects.md

## Conclusion

Context7 is a powerful tool that transforms how you integrate software stacks into your projects. By providing real-time, accurate documentation and code examples, it eliminates the guesswork and outdated information that often plagues AI-assisted development.

The key to success is being specific in your requests and leveraging Context7's comprehensive library coverage to get the most up-to-date and accurate information for your development needs.
