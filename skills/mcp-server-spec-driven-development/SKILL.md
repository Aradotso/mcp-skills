---
name: mcp-server-spec-driven-development
description: MCP server for systematic spec-driven development workflow from requirements to design to code using EARS format
triggers:
  - "help me create requirements using EARS format"
  - "generate a design document from requirements"
  - "implement code from design specifications"
  - "set up spec-driven development workflow"
  - "create structured requirements document"
  - "follow spec-driven development process"
  - "generate code from design document"
  - "use EARS format for requirements"
---

# MCP Server: Spec-Driven Development

> Skill by [ara.so](https://ara.so) — MCP Skills collection

## Overview

The Spec-Driven Development MCP server provides a structured workflow for systematic software development by guiding you through three stages: requirements → design → code. It uses the EARS (Easy Approach to Requirements Syntax) format for requirements documentation and ensures traceability from initial requirements to final implementation.

**Key Benefits:**
- Structured development workflow prevents "vibe coding"
- EARS format ensures clear, testable requirements
- Traceable path from requirements through design to code
- Systematic approach scales with project complexity

## Installation

### VS Code / VS Code Insiders

Add to your `mcp.json` configuration file:

```json
{
  "servers": {
    "spec-driven": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-server-spec-driven-development@latest"
      ]
    }
  }
}
```

Or use the one-click install buttons from the project README.

### Cursor / Claude Desktop

Add to your `mcp.json` configuration file:

```json
{
  "mcpServers": {
    "spec-driven": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-server-spec-driven-development@latest"
      ]
    }
  }
}
```

### Verification

After installation, verify the server is loaded by checking available MCP prompts. You should see three prompts:
- `generate-requirements`
- `generate-design-from-requirements`
- `generate-code-from-design`

## Workflow & Usage

### Stage 1: Generate Requirements Document

**Prompt Name:** `generate-requirements`

Use this prompt to create a structured requirements document in EARS format.

**Input Example:**
```
A Vue.js todo application with task creation, completion tracking, and local storage persistence
```

**What happens:**
- Creates `specs/requirements.md` in your project
- Uses EARS format (Ubiquitous, Event-driven, Unwanted behavior, State-driven, Optional)
- Structures requirements into functional and non-functional categories

**Example EARS Requirements Output:**
```markdown
# Requirements

## Functional Requirements

### Task Management
- The system SHALL allow users to create new tasks with a title and description
- WHEN a user clicks the "Add Task" button, the system SHALL display a task creation form
- IF a task has no title, the system SHALL prevent task creation
- WHILE the application is running, the system SHALL persist all tasks to local storage
- The system MAY allow users to add tags to tasks

### Task Completion
- The system SHALL allow users to mark tasks as complete
- WHEN a task is marked complete, the system SHALL update the UI to show completion status
- The system SHALL allow users to toggle task completion status

## Non-Functional Requirements

### Performance
- The system SHALL load all tasks within 500ms
- The system SHALL respond to user interactions within 100ms

### Usability
- The system SHALL provide a clean, intuitive interface
- The system SHALL work on modern browsers (Chrome, Firefox, Safari, Edge)
```

### Stage 2: Generate Design from Requirements

**Prompt Name:** `generate-design-from-requirements`

Use this prompt to create a design document based on your requirements.

**Prerequisites:**
- `specs/requirements.md` must exist (created in Stage 1)

**What happens:**
- Reads `specs/requirements.md`
- Creates `specs/design.md` with architecture, components, data models, and implementation details
- Maps requirements to design decisions

**Example Design Output Structure:**
```markdown
# Design Document

## Architecture Overview
- Component-based Vue.js application
- Local Storage for persistence
- Reactive state management

## Component Structure

### TaskList Component
**Purpose:** Display and manage the list of tasks
**Props:**
- tasks: Array<Task>
- onTaskToggle: Function
- onTaskDelete: Function

**State:**
- filter: 'all' | 'active' | 'completed'

### TaskForm Component
**Purpose:** Handle task creation
**Events:**
- onTaskCreate(task: Task)

## Data Models

### Task Interface
```typescript
interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
  createdAt: Date;
  tags?: string[];
}
```

## State Management
- Use Vue 3 Composition API
- Centralized task state in composable
- Local storage sync on state changes

## API Design
- localStorage.getItem('tasks')
- localStorage.setItem('tasks', JSON.stringify(tasks))
```

### Stage 3: Generate Code from Design

**Prompt Name:** `generate-code-from-design`

Use this prompt to generate implementation code from your design document.

**Prerequisites:**
- `specs/design.md` must exist (created in Stage 2)

**What happens:**
- Reads `specs/design.md`
- Generates implementation files in your project root
- Creates all components, utilities, and configuration files

**Example Generated Code:**

**src/composables/useTasks.ts:**
```typescript
import { ref, computed, watch } from 'vue';

export interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
  createdAt: Date;
  tags?: string[];
}

const STORAGE_KEY = 'tasks';

export function useTasks() {
  const tasks = ref<Task[]>([]);

  // Load tasks from localStorage
  const loadTasks = () => {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
      tasks.value = JSON.parse(stored);
    }
  };

  // Save tasks to localStorage
  const saveTasks = () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks.value));
  };

  // Watch for changes and persist
  watch(tasks, saveTasks, { deep: true });

  // Create new task
  const createTask = (title: string, description: string, tags?: string[]) => {
    const newTask: Task = {
      id: crypto.randomUUID(),
      title,
      description,
      completed: false,
      createdAt: new Date(),
      tags
    };
    tasks.value.push(newTask);
  };

  // Toggle task completion
  const toggleTask = (id: string) => {
    const task = tasks.value.find(t => t.id === id);
    if (task) {
      task.completed = !task.completed;
    }
  };

  // Delete task
  const deleteTask = (id: string) => {
    tasks.value = tasks.value.filter(t => t.id !== id);
  };

  // Computed properties
  const activeTasks = computed(() => tasks.value.filter(t => !t.completed));
  const completedTasks = computed(() => tasks.value.filter(t => t.completed));

  // Initialize
  loadTasks();

  return {
    tasks,
    createTask,
    toggleTask,
    deleteTask,
    activeTasks,
    completedTasks
  };
}
```

**src/components/TaskForm.vue:**
```vue
<template>
  <form @submit.prevent="handleSubmit" class="task-form">
    <input
      v-model="title"
      type="text"
      placeholder="Task title"
      required
      class="task-input"
    />
    <textarea
      v-model="description"
      placeholder="Task description"
      class="task-textarea"
    />
    <button type="submit" class="submit-btn">Add Task</button>
  </form>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const emit = defineEmits<{
  taskCreate: [title: string, description: string]
}>();

const title = ref('');
const description = ref('');

const handleSubmit = () => {
  if (title.value.trim()) {
    emit('taskCreate', title.value, description.value);
    title.value = '';
    description.value = '';
  }
};
</script>

<style scoped>
.task-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 2rem;
}

.task-input, .task-textarea {
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.submit-btn {
  padding: 0.5rem 1rem;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
```

## Best Practices

### 1. Always Follow the Three-Stage Process

Don't skip stages. Each builds on the previous:

```
Requirements (what) → Design (how) → Code (implementation)
```

### 2. Iterate on Requirements First

Before generating design, ensure your requirements are:
- Complete
- Testable
- Unambiguous
- In proper EARS format

### 3. Review Generated Documents

After each stage, review and refine:
- Requirements: Are all features covered?
- Design: Does the architecture make sense?
- Code: Does it match the design?

### 4. Use Descriptive Initial Requirements

**Good:**
```
A real-time chat application with user authentication, message history, 
typing indicators, read receipts, file sharing up to 10MB, and emoji support. 
Must work offline with message queuing.
```

**Too Vague:**
```
Make a chat app
```

### 5. Maintain Spec Files

Keep `specs/requirements.md` and `specs/design.md` updated as your project evolves. These are living documents.

## Common Patterns

### Full Project Initialization

```typescript
// Step 1: Generate requirements
// Use prompt: generate-requirements
// Input: "A REST API for blog management with posts, comments, user auth, and markdown support"

// Step 2: Generate design
// Use prompt: generate-design-from-requirements
// Reviews specs/requirements.md automatically

// Step 3: Generate code
// Use prompt: generate-code-from-design
// Reviews specs/design.md automatically
```

### Modifying Existing Features

If you need to change a feature:

1. Update `specs/requirements.md` manually
2. Re-run `generate-design-from-requirements`
3. Re-run `generate-code-from-design`

### Adding New Features

1. Add new requirements to `specs/requirements.md` in EARS format
2. Re-run `generate-design-from-requirements` 
3. Review design changes
4. Re-run `generate-code-from-design`

## File Structure

After running all three stages, your project will have:

```
project/
├── specs/
│   ├── requirements.md    # EARS format requirements
│   └── design.md           # Architecture and design decisions
├── src/                    # Generated source code
│   ├── components/
│   ├── composables/
│   └── ...
└── ...                     # Other generated files
```

## EARS Format Reference

The EARS format structures requirements into five types:

### Ubiquitous (always active)
```
The system SHALL [action]
```

### Event-driven (triggered by events)
```
WHEN [trigger], the system SHALL [action]
```

### Unwanted behavior (error handling)
```
IF [unwanted condition], the system SHALL [action]
```

### State-driven (mode-based)
```
WHILE [in state], the system SHALL [action]
```

### Optional (nice-to-have)
```
The system MAY [action]
```

## Troubleshooting

### Prompt Not Available

**Problem:** MCP server prompts don't appear in your IDE.

**Solution:**
1. Verify `mcp.json` configuration is correct
2. Restart your IDE
3. Check MCP server is running: look for "spec-driven" in MCP server list
4. Ensure Node.js 20+ is installed

### Requirements File Not Found

**Problem:** `generate-design-from-requirements` fails with file not found.

**Solution:**
1. Ensure `specs/requirements.md` exists
2. Run `generate-requirements` first
3. Check file is in the correct location relative to project root

### Design File Not Found

**Problem:** `generate-code-from-design` fails with file not found.

**Solution:**
1. Ensure `specs/design.md` exists
2. Run `generate-design-from-requirements` first
3. Verify file path is `specs/design.md`

### Generated Code Doesn't Match Requirements

**Problem:** Implementation differs from what you specified.

**Solution:**
1. Review `specs/requirements.md` for clarity
2. Make requirements more specific using EARS format
3. Review `specs/design.md` to ensure it captured requirements correctly
4. Regenerate design and code after fixing requirements

### Incomplete Code Generation

**Problem:** Not all components/files were generated.

**Solution:**
1. Check `specs/design.md` includes all necessary components
2. Add missing components to design document
3. Re-run `generate-code-from-design`

## Advanced Usage

### Multi-Module Projects

For larger projects, create separate requirement documents:

```
specs/
├── requirements-auth.md
├── requirements-api.md
├── requirements-ui.md
├── design-auth.md
├── design-api.md
└── design-ui.md
```

Then run the prompts for each module separately.

### Integration with CI/CD

Keep spec files in version control to:
- Track requirement changes over time
- Review design decisions during code review
- Ensure implementation matches approved designs

### Documentation Generation

The spec files serve as excellent documentation:
- `requirements.md` → User stories / Product requirements
- `design.md` → Technical documentation / Architecture decision records

## Examples by Project Type

### REST API Project

**Requirements Input:**
```
A RESTful API for task management with CRUD operations, user authentication 
via JWT, pagination, filtering, sorting, rate limiting, and OpenAPI documentation
```

### Frontend Application

**Requirements Input:**
```
A React dashboard with data visualization charts, real-time updates via WebSocket,
dark mode support, responsive design, infinite scroll, and CSV export functionality
```

### CLI Tool

**Requirements Input:**
```
A command-line tool for file conversion supporting JSON to YAML, XML to JSON,
CSV to JSON, with validation, progress indicators, and batch processing
```

### Full-Stack Application

**Requirements Input:**
```
A full-stack e-commerce platform with product catalog, shopping cart, 
checkout process, payment integration, order tracking, admin panel, 
and email notifications
```
