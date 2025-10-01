# NextChat Codebase Documentation

## Overview
NextChat is a modern, full-stack AI chat application built with Next.js that supports multiple AI providers and offers a comprehensive chat interface with advanced features like real-time chat, plugins, artifacts, and MCP (Model Context Protocol) support.

## Architecture & Technology Stack

### Core Technologies
- **Framework**: Next.js 14+ with App Router
- **Language**: TypeScript
- **Styling**: SCSS Modules
- **State Management**: Zustand
- **UI Components**: Custom component library with modular SCSS
- **Icons**: SVG icons with custom React components
- **Internationalization**: Multi-language support (English, Chinese, Japanese, Korean, etc.)
- **Desktop Support**: Tauri integration for cross-platform desktop app

### Key Dependencies
- React 18+ with hooks
- Multiple AI provider SDK integrations
- Real-time audio support (Azure SDK)
- Markdown rendering with syntax highlighting
- PWA capabilities with service workers
- Docker containerization support

## Project Structure

### Root Level (`/`)
```
├── app/                    # Next.js App Router directory
├── docs/                   # Documentation files
├── public/                 # Static assets
├── scripts/                # Build and utility scripts
├── src-tauri/              # Desktop app configuration
├── test/                   # Test files
└── Configuration files     # package.json, tsconfig.json, etc.
```

## App Directory Structure (`/app`)

### Core Application (`/app/`)
```
app/
├── api/                    # API route handlers for AI providers
├── client/                 # Client-side API integration
├── components/             # React components
├── config/                 # Configuration management
├── icons/                  # SVG icon components
├── locales/                # Internationalization files
├── masks/                  # Prompt template system
├── mcp/                    # Model Context Protocol implementation
├── store/                  # Zustand state management
├── styles/                 # Global and component styles
└── utils/                  # Utility functions
```

### API Routes (`/app/api`)
Handles communication with various AI providers:

**Provider-Specific Handlers:**
- `openai.ts` - OpenAI GPT models
- `anthropic.ts` - Claude models
- `google.ts` - Gemini models
- `azure.ts` - Azure OpenAI
- `baidu.ts` - Ernie models
- `bytedance.ts` - Doubao models
- `alibaba.ts` - Qwen models
- `tencent.ts` - Hunyuan models
- `moonshot.ts` - Moonshot models
- `iflytek.ts` - iFlytek models
- `deepseek.ts` - DeepSeek models
- `xai.ts` - Grok models
- `chatglm.ts` - ChatGLM models
- `siliconflow.ts` - SiliconFlow models
- `302ai.ts` - 302.AI models

**Utility & Feature APIs:**
- `auth.ts` - Authentication middleware
- `common.ts` - Shared API utilities
- `proxy.ts` - Proxy configuration
- `stability.ts` - Stability AI image generation
- `artifacts/` - Artifact management
- `config/` - Configuration endpoints
- `upstash/` - Upstash database integration
- `webdav/` - WebDAV synchronization

### Client Integration (`/app/client`)
- `api.ts` - Main API client interface
- `controller.ts` - Request/response handling
- `platforms/` - Platform-specific implementations

### Components (`/app/components`)
**Core UI Components:**
- `home.tsx` - Main application layout and routing
- `chat.tsx` - Chat interface and message handling
- `sidebar.tsx` - Navigation sidebar
- `settings.tsx` - Settings panel
- `auth.tsx` - Authentication interface

**Feature Components:**
- `artifacts.tsx` - Generated content display
- `mask.tsx` - Prompt template management
- `plugin.tsx` - Plugin system interface
- `mcp-market.tsx` - MCP marketplace
- `search-chat.tsx` - Chat history search
- `new-chat.tsx` - New conversation creation
- `realtime-chat/` - Real-time audio chat
- `sd/` - Stable Diffusion integration
- `voice-print/` - Voice visualization

**Utility Components:**
- `ui-lib.tsx` - Base UI component library
- `button.tsx` - Button components
- `input-range.tsx` - Range input controls
- `emoji.tsx` - Emoji picker
- `error.tsx` - Error boundary and display
- `markdown.tsx` - Markdown rendering
- `exporter.tsx` - Chat export functionality
- `message-selector.tsx` - Message selection tools

### State Management (`/app/store`)
Zustand stores for application state:

- `config.ts` - Application configuration
- `chat.ts` - Chat sessions and messages
- `access.ts` - API access control and validation
- `mask.ts` - Prompt template management
- `prompt.ts` - Custom prompt storage
- `plugin.ts` - Plugin management
- `sync.ts` - Cloud synchronization
- `update.ts` - Version and update management
- `sd.ts` - Stable Diffusion state

### Configuration (`/app/config`)
- `client.ts` - Client-side configuration
- `server.ts` - Server-side configuration
- `build.ts` - Build-time configuration

### Utilities (`/app/utils`)
- `utils.ts` - General utility functions
- `command.ts` - Command processing
- `polyfill.ts` - Browser compatibility
- `typing.ts` - TypeScript type definitions

## Key Features & Functionality

### 1. Multi-Provider AI Support
**Supported Providers:**
- OpenAI (GPT-4, GPT-3.5, etc.)
- Anthropic (Claude 3, etc.)
- Google (Gemini Pro, etc.)
- Azure OpenAI
- Baidu (Ernie)
- ByteDance (Doubao)
- Alibaba (Qwen)
- Tencent (Hunyuan)
- Moonshot (kimi)
- iFlytek (Spark)
- DeepSeek
- xAI (Grok)
- ChatGLM
- SiliconFlow
- 302.AI

### 2. Advanced Chat Features
- **Real-time streaming** responses
- **Message history** with local storage
- **Chat export** functionality
- **Search** through chat history
- **Message selection** and management
- **Auto-summarization** for long conversations

### 3. Prompt Management System
- **Mask system** for prompt templates
- **Custom prompts** creation and storage
- **Built-in prompt library**
- **Multi-language prompt support**

### 4. Plugin System
- **Extensible plugin architecture**
- **Network search integration**
- **Calculator functionality**
- **Custom API integrations**

### 5. MCP (Model Context Protocol)
- **Tool integration** system
- **External service connections**
- **Custom tool development**

### 6. Artifacts System
- **Generated content display**
- **Code execution environment**
- **Interactive content previews**

### 7. Image Generation
- **Stable Diffusion integration**
- **Multiple AI image providers**
- **Image variation and editing**

### 8. Real-time Communication
- **Voice chat capabilities**
- **Audio processing**
- **Real-time transcription**

### 9. Desktop Application
- **Tauri-based cross-platform app**
- **Native OS integration**
- **Offline functionality**

## Configuration & Environment

### Environment Variables
```bash
# Required
OPENAI_API_KEY=your_key

# Optional
BASE_URL=https://api.openai.com
AZURE_URL=https://your-resource.openai.azure.com
GOOGLE_API_KEY=your_key
ANTHROPIC_API_KEY=your_key
# ... (many more provider keys)

# Feature toggles
ENABLE_MCP=true
HIDE_USER_API_KEY=1
DISABLE_GPT4=1
```

### Build & Deployment
```bash
# Development
yarn dev

# Production build
yarn build
yarn start

# Desktop app
yarn app:dev
yarn app:build
```

## Data Flow & Architecture Patterns

### State Management Flow
1. **Configuration** → `config.ts` store
2. **Chat Sessions** → `chat.ts` store
3. **API Access** → `access.ts` store
4. **Templates** → `mask.ts` store
5. **Plugins** → `plugin.ts` store

### API Communication Flow
1. **User Input** → Component
2. **State Update** → Zustand Store
3. **API Request** → Client API (`/client`)
4. **Provider Handler** → API Routes (`/api`)
5. **External API** → AI Provider
6. **Response Streaming** → Component Update

### Component Architecture
- **Container Components**: `home.tsx`, `chat.tsx`
- **Feature Components**: `artifacts.tsx`, `mask.tsx`, `plugin.tsx`
- **UI Components**: `ui-lib.tsx`, `button.tsx`
- **Layout Components**: `sidebar.tsx`, `settings.tsx`

## Key Constants & Configuration

### Provider Endpoints (`constant.ts`)
```typescript
export const OPENAI_BASE_URL = "https://api.openai.com"
export const ANTHROPIC_BASE_URL = "https://api.anthropic.com"
export const GEMINI_BASE_URL = "https://generativelanguage.googleapis.com/"
// ... (15+ providers)
```

### Model Definitions
- **1000+ pre-configured models** across all providers
- **Vision model detection** patterns
- **Provider-specific model mappings**
- **Custom model support**

### Path Routing (`constant.ts`)
```typescript
export enum Path {
  Home = "/",
  Chat = "/chat",
  Settings = "/settings",
  NewChat = "/new-chat",
  Masks = "/masks",
  Plugins = "/plugins",
  // ... (10+ routes)
}
```

## Development Guidelines

### Code Organization
- **Feature-based structure** in components
- **Provider-based structure** in API routes
- **Store-based state management**
- **Modular SCSS styling**

### Key Development Patterns
1. **API Handler Pattern**: Each provider has dedicated handler
2. **Store Pattern**: Zustand stores for state management
3. **Component Pattern**: Modular, reusable components
4. **Configuration Pattern**: Environment-based configuration

### Testing Structure
- **Unit tests** for utilities and stores
- **Integration tests** for API handlers
- **Component tests** for React components

## Deployment & DevOps

### Supported Platforms
- **Vercel** (primary)
- **Docker** containers
- **Desktop** (Windows/Mac/Linux)
- **Mobile PWA**

### Build Process
1. **Mask Generation** → `app/masks/build.ts`
2. **Next.js Build** → Standard Next.js process
3. **Desktop Packaging** → Tauri build process

## Security & Privacy

### Data Storage
- **Local-first approach** - all data stored in browser
- **Optional cloud sync** via Upstash
- **WebDAV synchronization** support

### Authentication
- **API key management** per provider
- **Access code protection** for instances
- **Environment-based configuration**

## Performance Optimizations

### Bundle Optimization
- **Dynamic imports** for heavy components
- **Code splitting** by routes and features
- **Tree shaking** for unused code

### Runtime Performance
- **Message pagination** (15 messages per page)
- **Auto-summarization** for long conversations
- **Efficient re-renders** with proper state management

## Extension Points

### Plugin System
- **Custom API integrations**
- **Tool development**
- **UI extensions**

### MCP Integration
- **External tool connections**
- **Service integrations**
- **Custom protocol handlers**

### Custom Providers
- **Easy provider addition** via API route pattern
- **Configuration-driven model support**
- **Standardized request/response handling**

## Troubleshooting & Maintenance

### Common Patterns
- **Provider API changes** → Update respective handler
- **Model additions** → Add to `constant.ts` model lists
- **Feature additions** → Follow component/store patterns
- **Configuration changes** → Update environment variables

### Development Workflow
1. **Feature Development** → Component + Store + API
2. **Testing** → Unit + Integration tests
3. **Documentation** → Update relevant docs
4. **Deployment** → Build and release process

This documentation provides a comprehensive overview of the NextChat codebase architecture, helping developers understand the system topology and navigate the various features and components effectively.
