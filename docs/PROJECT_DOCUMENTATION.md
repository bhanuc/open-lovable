# Open Lovable - Project Documentation

## Overview

Open Lovable is a sophisticated AI-powered React application builder that enables users to create and modify React applications through natural language interactions. Built with Next.js, TypeScript, and Tailwind CSS, it leverages E2B sandboxes for secure code execution and multiple AI models for intelligent code generation.

## Architecture Overview

### Core Technologies

- **Frontend Framework**: Next.js 15.4.3 with React 19.1.0
- **Language**: TypeScript
- **Styling**: Tailwind CSS 4.x
- **Sandbox Environment**: E2B Code Interpreter
- **AI Integration**: Multiple providers (Anthropic, OpenAI, Groq)
- **UI Components**: Radix UI primitives
- **Animations**: Framer Motion

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Open Lovable Frontend                     │
│  (Next.js + React + TypeScript + Tailwind)                  │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│                   API Routes Layer                          │
│  ┌───────────────┬───────────────┬─────────────────────────┐ │
│  │ Code Gen APIs │ Sandbox APIs  │ Analysis APIs           │ │
│  │ • generate-   │ • create-     │ • analyze-edit-intent   │ │
│  │   ai-code-    │   sandbox     │ • detect-packages       │ │
│  │   stream      │ • apply-code  │ • file-search          │ │
│  │ • apply-ai-   │ • get-files   │                        │ │
│  │   code        │ • run-cmd     │                        │ │
│  └───────────────┴───────────────┴─────────────────────────┘ │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│              External Services Integration                   │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐  │
│  │ AI Providers│ E2B Sandbox │ Firecrawl   │ Git/GitHub  │  │
│  │ • Anthropic │ • Code Exec │ • Web Scrape│ • PR Creation│ │
│  │ • OpenAI    │ • File Mgmt │ • Screenshot│ • Commit    │  │
│  │ • Groq      │ • Vite Dev  │             │             │  │
│  └─────────────┴─────────────┴─────────────┴─────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Frontend Application (`app/page.tsx`)

The main application interface that provides:
- **Chat Interface**: Natural language interaction with AI
- **Sandbox Management**: Create and manage E2B sandboxes
- **File Explorer**: Browse and view generated files
- **Preview Window**: Live preview of applications
- **URL Scraping**: Extract content from websites for cloning

### 2. API Routes Layer

#### Code Generation APIs

**`generate-ai-code-stream`** (`app/api/generate-ai-code-stream/route.ts`)
- **Purpose**: Stream AI-generated code responses in real-time
- **Key Features**:
  - Supports multiple AI providers (Anthropic Claude, OpenAI GPT-5, Groq)
  - Intelligent context selection for edits vs new generation
  - Real-time package detection and installation
  - Conversation history tracking
  - Surgical edit mode for targeted changes

**`apply-ai-code`** (`app/api/apply-ai-code/route.ts`)
- **Purpose**: Parse and apply AI-generated code to E2B sandbox
- **Key Features**:
  - XML parsing of code files
  - Package detection and auto-installation
  - File conflict resolution
  - Vite dev server management

**`analyze-edit-intent`** (`app/api/analyze-edit-intent/route.ts`)
- **Purpose**: Analyze user requests to determine optimal edit strategy
- **Key Features**:
  - AI-powered intent analysis
  - Search plan generation for finding relevant code
  - Edit type classification (UPDATE_COMPONENT, ADD_FEATURE, etc.)

#### Sandbox Management APIs

**`create-ai-sandbox`** (`app/api/create-ai-sandbox/route.ts`)
- Creates E2B sandboxes with Vite template
- Configures development environment
- Initializes file watching and error monitoring

**`get-sandbox-files`** (`app/api/get-sandbox-files/route.ts`)
- Retrieves current file state from sandbox
- Generates file manifest for context
- Provides component relationship mapping

**`run-command`** (`app/api/run-command/route.ts`)
- Executes shell commands in sandbox
- Package installation and management
- Build process execution

### 3. Library Components

#### Context Selection (`lib/context-selector.ts`)
- **File Selection**: Intelligent file selection for edits
- **Context Building**: Creates enhanced prompts with file structure
- **Edit Intent Processing**: Processes edit intentions for targeted changes

#### File Search Executor (`lib/file-search-executor.ts`)
- **Search Plan Execution**: Executes AI-generated search plans
- **Code Location**: Finds exact code locations for modifications
- **Result Ranking**: Prioritizes search results by relevance

#### Edit Intent Analyzer (`lib/edit-intent-analyzer.ts`)
- **Request Analysis**: Analyzes user requests for edit patterns
- **Component Detection**: Identifies target components for modifications
- **Confidence Scoring**: Provides confidence levels for edit strategies

### 4. Configuration Management

#### App Configuration (`config/app.config.ts`)
- **E2B Settings**: Sandbox timeout, Vite port configuration
- **AI Model Settings**: Available models, temperature, token limits
- **UI Configuration**: Animation durations, toast settings
- **Package Management**: Installation timeout, auto-restart options

### 5. Type System

#### Core Types
- **`SandboxState`**: Global sandbox state management
- **`ConversationState`**: Chat history and context tracking
- **`FileManifest`**: File structure and component relationships
- **`EditIntent`**: Edit classification and targeting

## AI Integration Architecture

### Multi-Provider Support

The system supports multiple AI providers through a unified interface:

```typescript
// Provider Configuration
const providers = {
  anthropic: createAnthropic({ apiKey: process.env.ANTHROPIC_API_KEY }),
  openai: createOpenAI({ apiKey: process.env.OPENAI_API_KEY }),
  groq: createGroq({ apiKey: process.env.GROQ_API_KEY })
};

// Model Selection Logic
const getProvider = (model: string) => {
  if (model.startsWith('anthropic/')) return providers.anthropic;
  if (model.startsWith('openai/')) return providers.openai;
  return providers.groq; // Default fallback
};
```

### Current AI Providers Integration

#### 1. Anthropic Claude
- **Model**: `claude-sonnet-4-20250514`
- **Usage**: Primary for complex reasoning and code generation
- **Features**: Advanced context understanding, surgical edits

#### 2. OpenAI GPT-5
- **Model**: `gpt-5`
- **Usage**: Advanced reasoning with high token limits
- **Features**: Reasoning effort metadata, complex problem solving

#### 3. Groq (Fast Inference)
- **Model**: `moonshotai/kimi-k2-instruct`
- **Usage**: Fast responses for simple operations
- **Features**: Low latency, cost-effective for basic tasks

### Gemini-CLI Integration Plan

To migrate API calls to use `gemini-cli`, the following changes would be needed:

#### 1. Provider Configuration Update

```typescript
// Current implementation
const anthropic = createAnthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  baseURL: process.env.ANTHROPIC_BASE_URL || 'https://api.anthropic.com/v1',
});

// Gemini-CLI implementation
const gemini = {
  generateText: async (prompt: string, options?: any) => {
    const { exec } = require('child_process');
    const command = `gemini-cli generate --prompt "${prompt}" --model="${options?.model || 'gemini-pro'}"`;
    return new Promise((resolve, reject) => {
      exec(command, (error, stdout, stderr) => {
        if (error) reject(error);
        else resolve({ text: stdout });
      });
    });
  }
};
```

#### 2. API Route Updates

The following files would need updates for gemini-cli integration:

1. **`generate-ai-code-stream/route.ts`**: Replace AI SDK calls with gemini-cli
2. **`analyze-edit-intent/route.ts`**: Update structured generation to use gemini-cli
3. **`config/app.config.ts`**: Add Gemini model configurations

#### 3. Key Integration Points

**File**: `app/api/generate-ai-code-stream/route.ts`
- **Lines 13-24**: Provider initialization
- **Lines 1150-1240**: Model selection and streaming
- **Lines 1156-1220**: Stream processing logic

**File**: `app/api/analyze-edit-intent/route.ts`  
- **Lines 9-21**: Provider setup
- **Lines 96-108**: Model selection
- **Lines 113-155**: AI generation call

**File**: `config/app.config.ts`
- **Lines 34-45**: Available models configuration
- **Lines 41-45**: Model display names

#### 4. CLI Command Integration

Replace direct API calls with CLI commands:

```typescript
// Instead of:
const result = await streamText({
  model: modelProvider(actualModel),
  messages: [...],
  maxTokens: 8192
});

// Use:
const result = await executeGeminiCLI({
  command: 'generate',
  model: actualModel,
  messages: messages,
  stream: true,
  maxTokens: 8192
});
```

#### 5. Environment Variables

Update environment configuration for Gemini:

```env
# Remove existing provider keys
# ANTHROPIC_API_KEY=your_anthropic_api_key
# OPENAI_API_KEY=your_openai_api_key
# GROQ_API_KEY=your_groq_api_key

# Add Gemini configuration
GOOGLE_API_KEY=your_google_api_key
GEMINI_CLI_PATH=/usr/local/bin/gemini-cli
GEMINI_DEFAULT_MODEL=gemini-1.5-pro
```

## File Structure

```
open-lovable/
├── app/                          # Next.js app directory
│   ├── api/                      # API routes
│   │   ├── analyze-edit-intent/  # AI edit analysis
│   │   ├── apply-ai-code/        # Code application
│   │   ├── create-ai-sandbox/    # Sandbox creation
│   │   ├── generate-ai-code-stream/ # AI code generation
│   │   ├── get-sandbox-files/    # File retrieval
│   │   └── [other APIs]/         # Additional endpoints
│   ├── components/               # React components
│   ├── globals.css              # Global styles
│   ├── layout.tsx               # Root layout
│   └── page.tsx                 # Main application page
├── components/                   # Shared components
│   ├── ui/                      # UI primitives
│   ├── CodeApplicationProgress.tsx
│   ├── HMRErrorDetector.tsx
│   └── SandboxPreview.tsx
├── config/
│   └── app.config.ts            # Application configuration
├── lib/                         # Utility libraries
│   ├── context-selector.ts      # File selection logic
│   ├── edit-intent-analyzer.ts  # Edit analysis
│   ├── file-search-executor.ts  # Code search
│   └── utils.ts                 # General utilities
├── types/                       # TypeScript definitions
│   ├── conversation.ts          # Chat types
│   ├── file-manifest.ts         # File structure types
│   └── sandbox.ts               # Sandbox types
├── docs/                        # Documentation
├── public/                      # Static assets
├── package.json                 # Dependencies
├── next.config.ts              # Next.js configuration
├── tailwind.config.ts          # Tailwind configuration
└── tsconfig.json               # TypeScript configuration
```

## Key Features

### 1. Intelligent Code Generation
- **Context-Aware**: Uses file structure and conversation history
- **Multi-Provider**: Supports multiple AI models for different tasks
- **Streaming Responses**: Real-time code generation with progress tracking
- **Error Recovery**: Automatic truncation detection and recovery

### 2. Surgical Code Editing
- **Intent Analysis**: AI analyzes user requests to determine edit type
- **File Search**: Finds exact code locations for modifications
- **Minimal Changes**: Preserves existing code structure and functionality
- **Conflict Resolution**: Handles file updates and dependency management

### 3. Sandbox Management
- **Isolated Environment**: E2B sandboxes for secure code execution
- **Live Preview**: Real-time application preview with hot reloading
- **Package Management**: Automatic dependency detection and installation
- **Error Monitoring**: Vite error detection and reporting

### 4. Conversation Context
- **History Tracking**: Maintains conversation state across sessions
- **Project Evolution**: Tracks major changes and file modifications
- **User Preferences**: Learns user editing patterns and preferences
- **Context Optimization**: Intelligent context window management

### 5. Web Scraping Integration
- **URL Analysis**: Extract content from websites for application cloning
- **Screenshot Capture**: Visual website analysis for design recreation
- **Content Processing**: Clean and structure scraped content for AI processing

## Configuration

### Environment Variables

```env
# Required - E2B Sandbox
E2B_API_KEY=your_e2b_api_key

# Required - Web Scraping
FIRECRAWL_API_KEY=your_firecrawl_api_key

# AI Providers (at least one required)
ANTHROPIC_API_KEY=your_anthropic_api_key
OPENAI_API_KEY=your_openai_api_key
GROQ_API_KEY=your_groq_api_key

# Optional - Custom endpoints
ANTHROPIC_BASE_URL=https://api.anthropic.com/v1
OPENAI_BASE_URL=https://api.openai.com/v1
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Model Configuration

Models are configured in `config/app.config.ts`:

```typescript
ai: {
  defaultModel: 'moonshotai/kimi-k2-instruct',
  availableModels: [
    'openai/gpt-5',
    'moonshotai/kimi-k2-instruct', 
    'anthropic/claude-sonnet-4-20250514'
  ],
  modelDisplayNames: {
    'openai/gpt-5': 'GPT-5',
    'moonshotai/kimi-k2-instruct': 'Kimi K2 Instruct',
    'anthropic/claude-sonnet-4-20250514': 'Sonnet 4'
  }
}
```

## Development Workflow

### 1. Local Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Run tests
npm run test:all
```

### 2. Adding New AI Providers

1. **Install SDK**: Add provider SDK to `package.json`
2. **Configure Provider**: Add to provider initialization in API routes
3. **Update Config**: Add model options to `app.config.ts`
4. **Model Selection**: Update model selection logic in relevant APIs

### 3. Extending API Functionality

1. **Create Route**: Add new route in `app/api/`
2. **Type Definitions**: Add types in `types/`
3. **Client Integration**: Update frontend to use new API
4. **Documentation**: Update this documentation

## Security Considerations

### 1. API Key Management
- All API keys stored as environment variables
- No hardcoded credentials in source code
- Separate development and production configurations

### 2. Sandbox Isolation
- E2B provides secure, isolated execution environments
- No direct file system access on host machine
- Automatic cleanup of sandbox resources

### 3. Input Validation
- Request validation on all API endpoints
- Sanitization of user inputs before AI processing
- Protection against code injection attacks

### 4. Rate Limiting
- Built-in protection against API abuse
- Configurable timeouts and retry logic
- Resource usage monitoring

## Performance Optimization

### 1. Context Management
- Intelligent context window optimization
- Conversation history trimming
- File content caching and compression

### 2. Streaming Responses
- Real-time code generation feedback
- Progressive loading of large responses
- Efficient memory usage during streaming

### 3. Package Management
- Automatic package detection from imports
- Cached package installation results
- Minimal Vite restarts for better performance

### 4. File Operations
- Batched file operations
- Differential updates for changed files only
- Optimized file structure analysis

## Troubleshooting

### Common Issues

1. **Sandbox Creation Failures**
   - Check E2B API key configuration
   - Verify network connectivity
   - Review E2B account limits

2. **AI Generation Errors**
   - Validate API keys for AI providers
   - Check model availability
   - Review token limits and context size

3. **Package Installation Failures**
   - Verify npm registry access
   - Check for package compatibility
   - Review Node.js version requirements

4. **Preview Not Loading**
   - Check Vite development server status
   - Verify port configuration (default 5173)
   - Review browser console for errors

### Debug Mode

Enable debug logging in `config/app.config.ts`:

```typescript
dev: {
  enableDebugLogging: true,
  logApiResponses: true,
  enablePerformanceMonitoring: true
}
```

## Future Enhancements

### Planned Features

1. **Enhanced AI Integration**
   - Support for additional AI providers
   - Model performance comparison
   - Automatic model selection based on task type

2. **Advanced Code Analysis**
   - Static code analysis integration
   - Performance optimization suggestions
   - Security vulnerability detection

3. **Collaboration Features**
   - Multi-user sandbox sharing
   - Version control integration
   - Team workspace management

4. **Template System**
   - Pre-built application templates
   - Custom template creation
   - Template marketplace integration

### Gemini-CLI Migration Roadmap

1. **Phase 1**: Replace direct API calls with CLI commands
2. **Phase 2**: Implement streaming support through CLI
3. **Phase 3**: Add Gemini-specific optimizations
4. **Phase 4**: Performance benchmarking and optimization
5. **Phase 5**: Migration of all existing functionality

## Contributing

### Development Setup

1. Fork and clone the repository
2. Install dependencies: `npm install`
3. Configure environment variables
4. Start development server: `npm run dev`
5. Run tests: `npm run test:all`

### Code Style

- TypeScript for type safety
- ESLint for code quality
- Prettier for formatting
- Conventional commits for git history

### Testing

- Unit tests for utility functions
- Integration tests for API endpoints
- End-to-end tests for user workflows

## License

MIT License - see LICENSE file for details.

---

This documentation serves as a comprehensive guide for understanding, developing, and extending the Open Lovable platform. For specific implementation details, refer to the individual source files and their inline documentation.