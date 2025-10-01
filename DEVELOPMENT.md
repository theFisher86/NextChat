# NextChat SillyTavern Integration Development Guide

## Overview

This document provides comprehensive development standards, processes, and guidelines for the NextChat SillyTavern integration project. It covers development environment setup, coding standards, AI collaboration practices, testing strategies, and deployment processes to ensure consistent, high-quality development across the team.

## Development Environment

### System Requirements

#### Minimum Specifications
- **Operating System**: Windows 10+, macOS 12+, or Linux (Ubuntu 20.04+)
- **RAM**: 16GB minimum, 32GB recommended
- **Storage**: 50GB free space for development tools and dependencies
- **Network**: Stable internet connection for API testing and collaboration

#### Development Tools
- **Node.js**: Version 20.0.0+ (managed via nvm, fnm, or volta)
- **Yarn**: Version 4.0.0+ for package management
- **Git**: Version 2.40.0+ for version control
- **Docker**: Version 24.0.0+ for containerized development
- **Visual Studio Code**: Latest version with recommended extensions

### Environment Setup

#### 1. Repository Setup
```bash
# Clone the repository
git clone https://github.com/your-org/nextchat-sillytavern.git
cd nextchat-sillytavern

# Install dependencies
yarn install

# Set up environment variables
cp .env.template .env.local
# Edit .env.local with your configuration

# Initialize development database
yarn db:setup

# Start development server
yarn dev
```

#### 2. Development Tools Installation
```bash
# Install Node Version Manager (nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install and use Node.js 20
nvm install 20
nvm use 20

# Install Yarn globally
npm install -g yarn

# Install VS Code extensions
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension bradlc.vscode-tailwindcss
```

#### 3. API Keys and Configuration
```bash
# Required API keys for development
# Edit .env.local with the following keys:

# SillyTavern API Configuration
ST_API_BASE_URL=https://api.sillytavern.app/v1
ST_API_KEY=your_sillytavern_api_key

# Database Configuration
DATABASE_URL=postgresql://username:password@localhost:5432/nextchat_st_dev

# Redis Configuration (for caching)
REDIS_URL=redis://localhost:6379

# Storage Configuration (for character assets)
S3_BUCKET=nextchat-st-assets-dev
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Coding Standards

### TypeScript Guidelines

#### Type Definitions
```typescript
// ✅ Good: Comprehensive interface definition
interface Character {
  readonly id: string;
  name: string;
  description: string;
  personality: PersonalityProfile;
  avatar?: string;
  voice?: VoiceSettings;
  readonly createdAt: Date;
  updatedAt: Date;
  isActive: boolean;
  tags: readonly string[];
}

// ❌ Avoid: Weak typing
interface BadCharacter {
  id: any;
  data: object;
}
```

#### Function Signatures
```typescript
// ✅ Good: Proper typing and documentation
interface CreateCharacterParams {
  name: string;
  description: string;
  personality: PersonalityProfile;
  avatar?: File | string;
}

interface CreateCharacterResult {
  success: true;
  character: Character;
  uploadUrls?: AssetUploadUrls;
}

/**
 * Creates a new character with full validation and processing
 * @param params Character creation parameters
 * @returns Promise resolving to creation result
 */
async function createCharacter(
  params: CreateCharacterParams
): Promise<CreateCharacterResult> {
  // Implementation
}

// ❌ Avoid: Poor typing and missing documentation
function makeChar(data) {
  // Implementation
}
```

### Code Style Guidelines

#### Naming Conventions
- **Files**: `kebab-case` for file names (e.g., `character-manager.ts`)
- **Classes**: `PascalCase` (e.g., `CharacterManager`)
- **Functions**: `camelCase` (e.g., `createCharacter`)
- **Variables**: `camelCase` (e.g., `characterData`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `MAX_CHARACTERS`)
- **Types/Interfaces**: `PascalCase` (e.g., `CharacterProfile`)

#### Code Organization
```typescript
// File structure for character-related modules
src/
├── characters/
│   ├── components/          # React components
│   │   ├── CharacterEditor.tsx
│   │   ├── CharacterSelector.tsx
│   │   └── CharacterChat.tsx
│   ├── hooks/              # Custom React hooks
│   │   ├── useCharacter.ts
│   │   ├── useCharacterMemory.ts
│   │   └── useCharacterInteraction.ts
│   ├── services/           # Business logic services
│   │   ├── characterService.ts
│   │   ├── memoryService.ts
│   │   └── interactionService.ts
│   ├── types/              # TypeScript type definitions
│   │   ├── character.ts
│   │   ├── memory.ts
│   │   └── interaction.ts
│   ├── utils/              # Utility functions
│   │   ├── characterHelpers.ts
│   │   ├── validation.ts
│   │   └── formatting.ts
│   └── index.ts            # Public API exports
```

### Component Development Standards

#### React Component Structure
```typescript
import React, { useState, useCallback, useMemo } from 'react';
import { useCharacterStore } from '@/stores/characterStore';
import { Character } from '@/types/character';

interface CharacterEditorProps {
  characterId: string;
  onSave: (character: Character) => void;
  onCancel: () => void;
  className?: string;
}

/**
 * CharacterEditor component for creating and editing characters
 */
export const CharacterEditor: React.FC<CharacterEditorProps> = ({
  characterId,
  onSave,
  onCancel,
  className,
}) => {
  // Custom hooks for state management
  const { character, loading, error, updateCharacter } = useCharacterStore(
    (state) => ({
      character: state.characters.find((c) => c.id === characterId),
      loading: state.loading,
      error: state.error,
      updateCharacter: state.updateCharacter,
    })
  );

  // Local state
  const [formData, setFormData] = useState<CharacterFormData>(() =>
    character ? mapCharacterToForm(character) : createEmptyForm()
  );

  // Event handlers
  const handleSubmit = useCallback(async (event: React.FormEvent) => {
    event.preventDefault();

    try {
      const validatedData = validateCharacterForm(formData);
      const updatedCharacter = await updateCharacter(characterId, validatedData);
      onSave(updatedCharacter);
    } catch (error) {
      // Error handling
      console.error('Failed to save character:', error);
    }
  }, [formData, characterId, updateCharacter, onSave]);

  // Memoized values
  const isDirty = useMemo(() =>
    hasFormChanged(formData, character), [formData, character]
  );

  if (loading) return <CharacterEditorSkeleton />;
  if (error) return <ErrorMessage error={error} onRetry={() => window.location.reload()} />;

  return (
    <form onSubmit={handleSubmit} className={cn('character-editor', className)}>
      {/* Component JSX */}
    </form>
  );
};
```

## AI Collaboration Guidelines

### Working with AI Assistants

#### Code Generation Best Practices
1. **Provide Context**: Always include relevant files, interfaces, and project structure
2. **Specify Requirements**: Clearly state functional and non-functional requirements
3. **Request Examples**: Ask for code examples when implementing new patterns
4. **Iterative Development**: Build complex features incrementally

#### Example AI Collaboration Session
```typescript
// When requesting character management functionality:

// 1. First, provide context about existing architecture
// "Looking at our ARCHITECTURE.md and existing store structure..."

// 2. Specify the exact functionality needed
// "I need to implement a character memory management system that..."

// 3. Reference existing patterns
// "Following the pattern used in our existing chat store..."

// 4. Request specific implementation
// "Please implement the memory service with the following interface..."
```

#### Code Review Guidelines for AI-Generated Code
- **Type Safety**: Verify all types are properly defined and used
- **Error Handling**: Ensure proper error boundaries and fallback mechanisms
- **Performance**: Check for unnecessary re-renders and optimize as needed
- **Accessibility**: Verify ARIA labels and keyboard navigation
- **Security**: Review for potential security vulnerabilities

### Testing Strategy

#### Unit Testing Standards

##### Test File Organization
```typescript
// src/characters/services/__tests__/characterService.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { characterService } from '../characterService';
import { createMockCharacter } from '@/test-utils/mocks';

describe('CharacterService', () => {
  describe('createCharacter', () => {
    it('should create a valid character successfully', async () => {
      // Arrange
      const characterData = createMockCharacter();

      // Act
      const result = await characterService.createCharacter(characterData);

      // Assert
      expect(result.success).toBe(true);
      expect(result.character).toMatchObject(characterData);
    });

    it('should throw validation error for invalid character data', async () => {
      // Arrange
      const invalidData = { name: '' }; // Invalid data

      // Act & Assert
      await expect(characterService.createCharacter(invalidData))
        .rejects.toThrow(ValidationError);
    });
  });
});
```

##### Test Coverage Requirements
- **Functions**: > 90% coverage for public functions
- **Components**: > 85% coverage for React components
- **Services**: > 95% coverage for business logic services
- **APIs**: > 90% coverage for API handlers

#### Integration Testing

##### API Integration Tests
```typescript
// tests/integration/character-api.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';
import { app } from '@/app';
import { createTestDatabase, cleanupTestDatabase } from '@/test-utils/database';

describe('/api/characters', () => {
  beforeAll(async () => {
    await createTestDatabase();
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  describe('POST /api/characters', () => {
    it('should create character and return 201 status', async () => {
      const characterData = {
        name: 'Test Character',
        description: 'A test character',
        personality: { /* ... */ }
      };

      const response = await request(app)
        .post('/api/characters')
        .send(characterData)
        .set('Authorization', `Bearer ${testToken}`)
        .expect(201);

      expect(response.body).toHaveProperty('character');
      expect(response.body).toHaveProperty('uploadUrls');
    });
  });
});
```

#### End-to-End Testing

##### Critical Path Testing
```typescript
// tests/e2e/character-interaction.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Character Interaction', () => {
  test('should complete full character creation and interaction flow', async ({
    page
  }) => {
    // Navigate to character creation
    await page.goto('/characters/create');

    // Fill out character form
    await page.fill('[data-testid="character-name"]', 'Test Character');
    await page.fill('[data-testid="character-description"]', 'A test character for e2e testing');

    // Submit form
    await page.click('[data-testid="create-character"]');
    await expect(page.locator('[data-testid="character-created"]')).toBeVisible();

    // Start conversation
    await page.click('[data-testid="start-conversation"]');
    await expect(page.locator('[data-testid="chat-interface"]')).toBeVisible();

    // Send message and verify response
    await page.fill('[data-testid="message-input"]', 'Hello, character!');
    await page.click('[data-testid="send-message"]');

    await expect(page.locator('[data-testid="character-response"]')).toBeVisible();
  });
});
```

## Deployment Process

### Development Workflow

#### Branch Strategy
```mermaid
graph TD
    A[main] --> B[develop]
    B --> C[feature/character-memory]
    B --> D[feature/voice-integration]
    C --> E[pull-request]
    D --> F[pull-request]
    E --> G[code-review]
    F --> H[code-review]
    G --> I[merge-to-develop]
    H --> J[merge-to-develop]
    I --> K[staging-deployment]
    J --> L[staging-deployment]
    K --> M[testing]
    L --> N[testing]
    M --> O[merge-to-main]
    N --> P[merge-to-main]
    O --> Q[production-deployment]
    P --> R[production-deployment]
```

#### Git Workflow
```bash
# Create feature branch
git checkout -b feature/character-memory-management

# Make changes and commit
git add .
git commit -m "feat: implement character memory management system

- Add memory service for conversation context
- Implement memory storage and retrieval
- Add memory optimization algorithms
- Update API endpoints for memory operations

Closes #123"

# Push and create pull request
git push origin feature/character-memory-management
```

### Deployment Pipeline

#### Continuous Integration (CI)

**.github/workflows/ci.yml**:
```yaml
name: Continuous Integration

on:
  push:
    branches: [ develop, main ]
  pull_request:
    branches: [ develop, main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run type checking
        run: yarn type-check

      - name: Run linting
        run: yarn lint

      - name: Run tests
        run: yarn test --coverage

      - name: Build application
        run: yarn build

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
```

#### Continuous Deployment (CD)

**.github/workflows/cd.yml**:
```yaml
name: Continuous Deployment

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Build application
        run: yarn build

      - name: Run database migrations
        run: yarn db:migrate
        env:
          DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}

      - name: Deploy to production
        run: yarn deploy
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

### Quality Assurance

#### Code Review Checklist
- [ ] **Functionality**: Does the code implement the required functionality?
- [ ] **Type Safety**: Are all types properly defined and used?
- [ ] **Error Handling**: Are errors properly caught and handled?
- [ ] **Performance**: Is the code optimized for performance?
- [ ] **Security**: Are there any security vulnerabilities?
- [ ] **Accessibility**: Does the code meet accessibility standards?
- [ ] **Testing**: Are there appropriate tests for the functionality?
- [ ] **Documentation**: Is the code properly documented?

#### Pre-deployment Checklist
- [ ] All tests passing in CI/CD pipeline
- [ ] Code coverage meets minimum requirements
- [ ] No high-severity security vulnerabilities
- [ ] Database migrations tested in staging environment
- [ ] Performance benchmarks meet requirements
- [ ] Monitoring and alerting configured
- [ ] Rollback plan documented and tested

## Future Considerations

### Development Process Evolution

#### Automated Code Quality
- **AI-Powered Code Review**: Automated analysis of code quality and best practices
- **Smart Testing**: AI-generated test cases for better coverage
- **Performance Monitoring**: Real-time performance analysis and optimization suggestions

#### Advanced Development Practices
- **Feature Flags**: Enable/disable features without deployments
- **Gradual Rollouts**: Canary deployments for new character features
- **A/B Testing**: Test different character interaction approaches
- **Real User Monitoring**: Track user behavior and performance in production

### Team Growth and Scaling
- **Documentation as Code**: Keep documentation in sync with code changes
- **Knowledge Sharing**: Regular tech talks and brown bag sessions
- **Mentorship Program**: Pair programming and code review training
- **Contributing Guidelines**: Clear process for external contributions

---

*This development guide ensures consistent, high-quality development practices across the NextChat SillyTavern integration team.*