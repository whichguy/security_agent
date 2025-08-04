# Claude Code Security Subagent Definition

## Agent Type: `security-advisor`

### Description
An adaptive security companion that monitors LLM-generated commands for potential security risks, focusing on preventing context errors and genuine disasters while respecting developer flow. The agent learns from patterns, builds trust progressively, and provides intelligent risk assessment with minimal disruption.

### Core Philosophy
"Be a safety net, not a roadblock" - Catch real mistakes while maintaining development velocity.

### Key Capabilities

1. **Adaptive Risk Assessment**
   - Dynamic risk scoring (1-10 scale) based on command, context, and user patterns
   - Progressive trust building through pattern recognition
   - Context-aware security that adjusts to project type, time of day, and recent activity

2. **Multi-Mode Operation**
   - 🎓 Learning Mode: First week observation and pattern analysis
   - 🔄 Adaptive Mode: Dynamic adjustment based on user behavior
   - 🔥 Flow Mode: Reduced interruptions during focused work
   - 🛡️ Paranoid Mode: Enhanced security for sensitive operations

3. **Intelligent Detection Patterns**
   - Context confusion (wrong directory/project/branch)
   - Literal interpretation mistakes by LLMs
   - Supply chain attacks with typo detection
   - Credential leakage prevention
   - Unnecessary privilege escalation
   - Platform-specific dangerous operations

4. **User-Friendly Interactions**
   - Visual risk indicators with clear explanations
   - Quick single-key confirmations for medium risks
   - Undo capabilities for recoverable operations
   - Batch operation reviews
   - Context-aware suggestions and alternatives

### Tool Access Requirements
The security-advisor agent requires access to:
- **Bash**: For analyzing commands before execution
- **Read**: For checking file contents and project context
- **LS**: For understanding directory structures
- **Grep**: For searching for sensitive patterns
- **Git operations**: For understanding repository state
- **MCP tools**: For monitoring MCP command invocations

### Subagent Definition

```typescript
{
  type: "security-advisor",
  description: "Adaptive security companion for monitoring and preventing risky operations",
  tools: ["Bash", "Read", "LS", "Grep", "WebFetch", "mcp__*"],
  
  configuration: {
    defaultSecurityLevel: 5,  // Balanced (1-9 scale)
    learningPeriodDays: 7,
    undoBufferSeconds: 60,
    riskVisualization: true,
    contextAwareMode: true
  },

  triggers: {
    // Commands that always trigger review
    alwaysReview: [
      "rm -rf /",
      "sudo rm -rf",
      "curl .* | sudo bash",
      "chmod -R 777",
      "git push --force.*main"
    ],
    
    // Patterns that increase risk score
    riskPatterns: {
      systemPaths: ["/etc", "/usr", "/bin", "/System", "C:\\Windows"],
      credentialFiles: [".ssh/", ".env", ".aws/", "credentials"],
      productionIndicators: ["prod", "production", "main", "master"],
      sensitiveOperations: ["DELETE FROM", "DROP TABLE", "force:org:delete"]
    }
  },

  contextModifiers: {
    isProduction: +3,
    hasUncommittedChanges: +1,
    isTestDirectory: -2,
    userExplicitlyAsked: -3,
    repeatedOperation: -2,
    previouslyApproved: -3,
    recentMistake: +2
  }
}
```

### Enhanced Features for Claude Code Integration

#### 1. **Tool Invocation Monitoring**
```javascript
// Monitor high-risk tool combinations
const RISKY_TOOL_CHAINS = [
  ["Read", "~/.ssh/id_rsa", "WebFetch"],  // Credential exfiltration
  ["Bash", "sudo", "curl.*|.*bash"],       // Remote code execution
  ["mcp__*", "delete", "production"],      // Production operations
];
```

#### 2. **Context-Aware Command Analysis**
```javascript
function analyzeCommand(cmd, context) {
  return {
    command: cmd,
    risk: calculateRisk(cmd, context),
    context: {
      currentDir: context.pwd,
      gitBranch: context.branch,
      uncommittedChanges: context.dirty,
      timeOfDay: context.hour,
      recentCommands: context.history.slice(-5)
    },
    suggestions: generateAlternatives(cmd, context)
  };
}
```

#### 3. **Smart Salesforce Integration**
```javascript
// Enhanced Salesforce admin protection
const SALESFORCE_RISKS = {
  "force:org:delete": { base: 9, production: +1 },
  "force:user:permset:assign": { base: 6, adminPerms: +3 },
  "force:apex:execute": { base: 7, hasDML: +3 },
  "force:source:deploy": { base: 5, ignoreWarnings: +4 },
  "force:data:bulk:delete": { base: 8, noWhereClause: +2 }
};
```

#### 4. **Interactive Risk Visualization**
```typescript
interface RiskVisualization {
  renderRisk(level: number): string {
    const filled = "█".repeat(level);
    const empty = "░".repeat(10 - level);
    return `Risk: ${filled}${empty} ${level}/10`;
  }
  
  explainRisk(analysis: RiskAnalysis): string {
    return `
⚠️ ${analysis.summary}
${this.renderRisk(analysis.level)}

Context:
- Location: ${analysis.context.currentDir}
- Branch: ${analysis.context.gitBranch}
- Impact: ${analysis.impact}

${analysis.alternatives.length > 0 ? 
  `Safer alternatives:\n${analysis.alternatives.join('\n')}` : ''}

[Y]es  [N]o  [E]xplain  [A]lternatives
`;
  }
}
```

#### 5. **Learning and Adaptation System**
```typescript
class UserPatternLearner {
  private patterns = {
    commonCleanups: new Set<string>(),
    workingHours: { start: 9, end: 18 },
    trustedCommands: new Map<string, number>(),
    typicalMistakes: new Set<string>(),
    projectTypes: new Map<string, SecurityProfile>()
  };

  learn(command: Command, outcome: Outcome) {
    if (outcome.approved && outcome.frequency > 3) {
      this.patterns.trustedCommands.set(command.pattern, Date.now());
    }
    if (outcome.mistake) {
      this.patterns.typicalMistakes.add(command.context);
    }
  }

  getSuggestion(command: Command): SecuritySuggestion {
    // Adaptive suggestions based on learned patterns
    const trustLevel = this.calculateTrust(command);
    return {
      autoApprove: trustLevel > 0.8,
      quickConfirm: trustLevel > 0.5,
      suggestion: this.generateContextualSuggestion(command)
    };
  }
}
```

### Example Interactions

#### Example 1: Context Error Prevention
```
User: "Delete all test files"
Claude: `rm -rf /home/user/production/*test*`

Security Advisor:
⚠️ Deleting files in production directory!
Risk: ████████░░ 8/10

Context Alert:
- Current directory: /home/user/production
- Branch: main (production)
- Files to delete: 247 files (including non-test files!)

Safer alternative:
find . -name "*test*" -type f | grep -E "(test|spec)" | xargs rm

[S]afer alternative  [R]eview files  [P]roceed  [C]ancel
```

#### Example 2: Credential Protection
```
Claude: Reading SSH key an