# Security Agent Prompts

This repository contains prompts for building security-focused AI agents with automated evaluation powered by the **Security Command Analysis Expert**.

## 🛡️ Automated Security Evaluation

All prompt changes are automatically evaluated using the [Prompt Expert Bank](https://github.com/whichguy/prompt-expert-bank) security expert module, which provides:

- **8 Security Test Scenarios**: From malicious commands to edge cases
- **6 Performance Metrics**: Detection rate, false positives, context awareness, etc.
- **Binary Recommendations**: Clear APPROVE or REQUEST_CHANGES decisions
- **A/B Testing**: Compare prompt versions with isolated LLM contexts

## 📊 Evaluation Metrics

| Metric | Weight | Description |
|--------|--------|-------------|
| Detection Rate | 30% | Identifies dangerous commands |
| False Positive Rate | 25% | Avoids blocking safe commands |
| Context Awareness | 20% | Understands environment context |
| Response Clarity | 10% | Clear explanations |
| Alternative Suggestions | 10% | Provides safer alternatives |
| Risk Assessment Quality | 5% | Structured risk analysis |

## 🧪 Test Scenarios

The security expert tests prompts against:
1. System destruction commands (rm -rf /)
2. Credential access attempts
3. Sudo/privilege escalation
4. Safe operations (npm install, git)
5. Context-dependent commands
6. Ambiguous requests

## 🚀 How It Works

1. **Create/Edit** a security prompt (`.md` file)
2. **Submit PR** with your changes
3. **Automatic Evaluation** runs via GitHub Actions
4. **Get Feedback** as a PR comment with metrics and recommendation

## 📈 Success Criteria

- **65%+ Overall Score**: Prompt approved for production
- **5%+ Improvement**: Changes approved
- **Clear Binary Decision**: No neutral recommendations

## 🔧 Setup

Add to repository secrets:
- `ANTHROPIC_API_KEY`: For Claude-based evaluations

## 📚 Resources

- [Example Security Prompt](sec_agent_prompt.md)
- [Prompt Expert Bank](https://github.com/whichguy/prompt-expert-bank)
- [Security Test Scenarios](https://github.com/whichguy/prompt-expert-bank/blob/main/test-scenarios/security-tests.js)