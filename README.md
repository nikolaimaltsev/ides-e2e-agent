## ides-e2e-agent
Agent to help develop e2e tests within the existing IDE Services project codebase


Uses 5-pass flow: 

1. Planning and task justification (ful Youtrack task and/or scenario as an input) 
2. Draft implementation and UI locators 
3. Browser verification via playwright MCP and adjust according to actual behavior seen in browser 
4. Cleanup and hardening 
5. Code Review: alignment to existing principles, reused patterns 

--

* Outputs md writeups after each pass (allows for review manually in between passes or with other llm) 
* Uses principles.md (taken from PRs and common sense) 
* Uses IDEA MCP to check inline errors (requires configuration)

## Usage

1. Copy CLAUDE.md into the project root
2. copy files in .agent folder to the same folder in the project

