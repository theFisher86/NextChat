# NextChat SillyTavern Integration API Documentation

## Overview

This document provides comprehensive documentation for all APIs related to SillyTavern functionality in NextChat. It covers internal REST APIs for character management, external API integration with SillyTavern services, WebSocket interfaces for real-time communication, and data structures used throughout the integration.

## Core Interfaces

### Character Management API

#### REST Endpoints

##### Character CRUD Operations

**GET /api/characters**
- **Description**: Retrieve user's character collection with optional filtering
- **Authentication**: Required (JWT token)
- **Query Parameters**:
  - `limit` (optional): Maximum number of characters to return (default: 50)
  - `offset` (optional): Number of characters to skip (default: 0)
  - `search` (optional): Search query for character name/description
  - `tags` (optional): Comma-separated list of tags to filter by
  - `sort` (optional): Sort field (name, created, modified, usage)
  - `order` (optional): Sort order (asc, desc)

**Response**:
```typescript
interface CharacterListResponse {
  characters: Character[];
  total: number;
  hasMore: boolean;
  filters: AppliedFilters;
}
```

**POST /api/characters**
- **Description**: Create a new character
- **Authentication**: Required (JWT token)
- **Content-Type**: `application/json`

**Request Body**:
```typescript
interface CreateCharacterRequest {
  name: string;
  description: string;
  personality: PersonalityProfile;
  avatar?: File | string;
  voice?: VoiceSettings;
  isPublic?: boolean;
  tags?: string[];
}
```

**Response**:
```typescript
interface CharacterResponse extends Character {
  created: true;
  uploadUrls?: AssetUploadUrls;
}
```

**GET /api/characters/{characterId}**
- **Description**: Retrieve detailed character information
- **Authentication**: Required (JWT token or public character access)
- **Path Parameters**:
  - `characterId`: UUID of the character

**Response**:
```typescript
interface CharacterDetailResponse extends Character {
  statistics: CharacterStatistics;
  permissions: CharacterPermissions;
  relatedCharacters: Character[];
}
```

**PUT /api/characters/{characterId}**
- **Description**: Update character information
- **Authentication**: Required (character owner or admin)
- **Content-Type**: `application/json`

**Request Body**:
```typescript
interface UpdateCharacterRequest extends Partial<CreateCharacterRequest> {
  version: number; // For optimistic locking
}
```

**DELETE /api/characters/{characterId}**
- **Description**: Delete a character and all associated data
- **Authentication**: Required (character owner or admin)

##### Character Interaction Endpoints

**POST /api/characters/{characterId}/interact**
- **Description**: Send a message to a character and receive response
- **Authentication**: Required (JWT token)
- **Content-Type**: `application/json`

**Request Body**:
```typescript
interface CharacterInteractionRequest {
  message: string;
  conversationId?: string;
  context?: ConversationContext;
  parameters?: InteractionParameters;
}
```

**Response**:
```typescript
interface CharacterInteractionResponse {
  response: CharacterMessage;
  conversationId: string;
  context: UpdatedConversationContext;
  usage: TokenUsage;
}
```

**GET /api/characters/{characterId}/conversations**
- **Description**: Retrieve conversation history for a character
- **Authentication**: Required (JWT token)
- **Query Parameters**:
  - `limit` (optional): Maximum conversations to return (default: 20)
  - `before` (optional): Cursor for pagination
  - `context` (optional): Include full context data

### Memory Management API

**GET /api/characters/{characterId}/memory**
- **Description**: Retrieve character's memory and context data
- **Authentication**: Required (character owner)

**Response**:
```typescript
interface MemoryResponse {
  memory: ConversationMemory[];
  summary: MemorySummary;
  statistics: MemoryStatistics;
}
```

**POST /api/characters/{characterId}/memory**
- **Description**: Add new memory entry to character
- **Authentication**: Required (character owner)
- **Content-Type**: `application/json`

**Request Body**:
```typescript
interface MemoryUpdateRequest {
  memory: ConversationMemory;
  operation: 'add' | 'update' | 'remove';
}
```

## Data Structures

### Core Data Types

#### Character Entity
```typescript
interface Character {
  // Basic Information
  id: string;
  name: string;
  description: string;
  createdAt: string;
  updatedAt: string;

  // Core Data
  personality: PersonalityProfile;
  scenario?: Scenario;
  firstMes?: string;
  mesExample?: string;

  // Assets
  avatar?: string;
  voice?: VoiceSettings;

  // Metadata
  tags?: string[];
  isPublic: boolean;
  isActive: boolean;
  usageCount: number;

  // Relationships
  creatorId: string;
  groupId?: string;
}
```

#### Personality Profile
```typescript
interface PersonalityProfile {
  // Basic traits
  name: string;
  description: string;
  personality: string;
  scenario: string;

  // Behavioral patterns
  firstMes: string;
  mesExample: string;

  // Advanced configuration
  creatorcomment?: string;
  avatar?: string;
  chat?: string;
  viewScreen?: string;

  // System prompts
  systemPrompt?: string;
  postHistoryInstructions?: string;

  // Alternate greetings
  alternateGreetings?: string[];

  // Character notes
  characterBook?: CharacterBook;

  // Group settings
  groupOnly?: boolean;
  activationGroups?: string[];
}
```

#### Conversation Context
```typescript
interface ConversationContext {
  // Current conversation state
  conversationId: string;
  characterId: string;
  userId: string;

  // Message history
  messages: ConversationMessage[];
  currentMes: number;

  // Memory and context
  memory: ConversationMemory[];
  context?: AIPersonalityContext;

  // Session data
  sessionId: string;
  startedAt: string;
  lastActivity: string;

  // Settings
  settings: ConversationSettings;
}
```

### Message Types

#### User Message
```typescript
interface UserMessage {
  // Core message data
  id: string;
  role: 'user';
  content: string;
  timestamp: string;

  // Context
  conversationId: string;
  characterId?: string;

  // Metadata
  metadata?: MessageMetadata;
}
```

#### Character Message
```typescript
interface CharacterMessage {
  // Core message data
  id: string;
  role: 'character';
  content: string;
  timestamp: string;

  // Character context
  characterId: string;
  characterName: string;

  // Generation info
  generated: boolean;
  model?: string;
  usage?: TokenUsage;

  // Metadata
  metadata?: MessageMetadata;
}
```

#### System Message
```typescript
interface SystemMessage {
  // Core message data
  id: string;
  role: 'system';
  content: string;
  type: SystemMessageType;
  timestamp: string;

  // Context
  conversationId: string;
}
```

## Integration APIs

### SillyTavern API Integration

#### HTTP API Client

**Base Configuration**:
```typescript
interface STAPIConfig {
  baseURL: string;
  apiKey: string;
  timeout: number;
  retries: number;
  retryDelay: number;
}
```

**Core API Methods**:

**POST /api/characters/{characterId}/generate**
- **Description**: Generate character response using SillyTavern API
- **Headers**:
  - `Authorization: Bearer {apiKey}`
  - `Content-Type: application/json`

**Request Body**:
```typescript
interface STGenerateRequest {
  // Character context
  characterId: string;
  userInput: string;

  // Conversation history
  history?: STMessage[];

  // Generation parameters
  genParams?: GenerationParameters;

  // Context settings
  contextSettings?: ContextSettings;

  // Advanced options
  overrideSettings?: OverrideSettings;
}
```

**Response**:
```typescript
interface STGenerateResponse {
  // Generated content
  content: string;
  model: string;

  // Token usage
  usage: TokenUsage;

  // Generation metadata
  generationTime: number;
  finishReason: string;

  // Character state updates
  characterState?: CharacterStateUpdate;
}
```

#### WebSocket Integration

**Connection Management**:
```typescript
interface STWebSocketConfig {
  url: string;
  protocols?: string[];
  reconnectInterval: number;
  maxReconnectAttempts: number;
}
```

**Message Types**:
```typescript
interface STWebSocketMessage {
  type: 'generate' | 'stream' | 'control' | 'heartbeat';
  payload: any;
  id?: string;
  timestamp: string;
}
```

**Real-time Generation**:
```typescript
interface STStreamRequest {
  type: 'stream';
  characterId: string;
  userInput: string;
  stream: true;
  onChunk: (chunk: string) => void;
  onComplete: (result: STGenerateResponse) => void;
  onError: (error: Error) => void;
}
```

## Error Handling

### Error Response Format
```typescript
interface APIErrorResponse {
  error: {
    code: string;
    message: string;
    details?: any;
    timestamp: string;
    requestId: string;
  };
}
```

### Common Error Codes

#### Character Management Errors
- `CHARACTER_NOT_FOUND`: Specified character does not exist
- `CHARACTER_ACCESS_DENIED`: Insufficient permissions for character operation
- `CHARACTER_INVALID_DATA`: Invalid or malformed character data
- `CHARACTER_QUOTA_EXCEEDED`: User has exceeded character creation limit

#### Integration Errors
- `ST_API_UNAVAILABLE`: SillyTavern API is not accessible
- `ST_API_RATE_LIMITED`: API rate limit exceeded
- `ST_API_AUTH_FAILED`: Authentication with SillyTavern API failed
- `ST_GENERATION_FAILED`: Character response generation failed

#### System Errors
- `INTERNAL_SERVER_ERROR`: Unexpected server error
- `SERVICE_UNAVAILABLE`: Service temporarily unavailable
- `TIMEOUT`: Request timeout
- `VALIDATION_ERROR`: Request validation failed

### Error Recovery Strategies

#### Retry Logic
```typescript
interface RetryConfig {
  maxAttempts: number;
  baseDelay: number;
  maxDelay: number;
  backoffFactor: number;
  retryableErrors: string[];
}
```

#### Circuit Breaker Pattern
```typescript
interface CircuitBreakerConfig {
  failureThreshold: number;
  recoveryTimeout: number;
  monitoringWindow: number;
}
```

## Version History

### API Version 1.0 (Current)
**Release Date**: October 2025
**Breaking Changes**:
- Initial release of SillyTavern integration API
- Introduction of character management endpoints
- Real-time WebSocket support for character interactions

**New Features**:
- Complete character CRUD operations
- Memory management and context handling
- Real-time streaming for character responses
- Comprehensive error handling and recovery

### Migration Guides

#### From NextChat Native Chat
1. **Character Import**: Use `/api/characters/import` endpoint
2. **Memory Migration**: Automatic migration of conversation history
3. **Settings Transfer**: Preserve user preferences and settings

## Future Considerations

### Upcoming API Versions

#### Version 1.1 (Q1 2026)
**Planned Enhancements**:
- **Voice Integration**: API endpoints for TTS and voice synthesis
- **Multi-character Conversations**: Support for group character interactions
- **Advanced Memory**: Enhanced memory management with semantic search
- **Analytics API**: Character usage and performance metrics

#### Version 2.0 (Q3 2026)
**Major Updates**:
- **Plugin System**: Extensible API for third-party character integrations
- **Federated Characters**: Cross-platform character sharing and discovery
- **Advanced AI Features**: Integration with next-generation language models
- **Real-time Collaboration**: Multi-user character interaction sessions

### API Evolution Strategy

#### Deprecation Policy
- APIs will be marked as deprecated 2 versions before removal
- Migration guides provided for deprecated endpoints
- Breaking changes communicated 3 months in advance

#### Extension Points
- **Webhooks**: Real-time notifications for character events
- **Plugin API**: Extensible interface for custom character behaviors
- **Custom Models**: Support for user-trained AI models
- **Federation**: Cross-service character data sharing

### Performance Optimizations

#### Planned Improvements
- **Batch Operations**: Bulk character and memory operations
- **Caching Strategy**: Advanced caching for frequently accessed data
- **Compression**: Automatic compression for large payloads
- **Connection Pooling**: Optimized connection management for external APIs

---

*This API documentation serves as the definitive reference for all SillyTavern integration APIs in NextChat. All implementations should conform to these specifications.*