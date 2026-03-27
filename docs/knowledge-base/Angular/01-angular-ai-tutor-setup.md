# Angular AI Tutor Setup Guide for Students

## Prerequisites
- Node.js and npm installed
- An Angular project (v20+)
- A code editor or terminal

---

## Installation & Setup Steps

### Step 1: Install Gemini CLI
```bash
npm install -g @google/gemini-cli
```

### Step 2: Configure the MCP Server
In your **Angular project root**, create a file: `.gemini/settings.json`

Add this content:
```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### Step 3: Navigate to Your Project
```bash
cd <your-angular-project-folder>
```

### Step 4: Launch Gemini CLI
```bash
gemini
```

### Step 5: Start the AI Tutor
In the Gemini CLI prompt, type:
```
launch the Angular AI tutor
```

---

## What Happens Next
✅ The tutor analyzes your project  
✅ It asks for your experience level (Beginner/Intermediate/Experienced)  
✅ It begins Module 1 of the **Smart Recipe Box** tutorial  
✅ You learn concepts → see code examples → complete hands-on exercises  

---

## Key Commands During the Tutorial
- `Show the table of contents` - See the full learning plan
- `Set my experience level to 5` - Adjust difficulty
- `Skip this section` - Auto-complete a module
- `I need a hint` - Get guidance on an exercise

---

## Learning Path Overview
The tutor guides you through **5 phases**:

1. **Phase 1**: Angular Fundamentals (Getting Started, Interpolation, Event Listeners)
2. **Phase 2**: State and Signals (Writable Signals, Computed Signals)
3. **Phase 3**: Component Architecture (Components, Inputs, Styling, Lists, Conditionals)
4. **Phase 4**: Advanced Features (Two-Way Binding, Services, Routing, Forms, Material)
5. **Phase 5**: Signal Forms (Experimental - Introduction, Submission, Validation, Error Messages)

You'll build a complete **Smart Recipe Box** application throughout the course!

---

**Good luck learning! 🚀**
