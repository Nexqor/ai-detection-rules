<p align="center">
  <img src="https://raw.githubusercontent.com/Nexqor/Nexqor/main/banner.png" alt="Nexqor" width="100%">
</p>

<h1 align="center">ai-detection-rules 🛰️</h1>

<p align="center">
  <b>Sigma detection rules for attacks on AI infrastructure and AI agents.</b><br/>
  LLMjacking, LLM gateway abuse, agent-driven credential theft, prompt-injection exfiltration,<br/>
  MCP config tampering and exposed model servers. Convert to Splunk, Sentinel, Elastic and more.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/format-Sigma-c41237?style=for-the-badge" alt="sigma"/>
  <img src="https://img.shields.io/badge/rules-7-c41237?style=for-the-badge" alt="rules"/>
  <img src="https://img.shields.io/badge/status-experimental-000000?style=for-the-badge" alt="status"/>
</p>

---

## Rules

| Rule file | Detects | Log source | Level | ATT&CK |
|-----------|---------|------------|-------|--------|
| [`aws_bedrock_llmjacking_recon.yml`](aws_bedrock_llmjacking_recon.yml) | Bedrock logging checks, model-access enablement after credential theft | AWS CloudTrail | medium | T1496, T1562.008 |
| ↳ (second rule in file) | Bedrock invocation logging deleted | AWS CloudTrail | high | T1562.008 |
| [`web_llm_gateway_admin_key_endpoints.yml`](web_llm_gateway_admin_key_endpoints.yml) | LiteLLM-style key/user endpoints hit from outside admin network | Web server | high | T1190 |
| [`proc_ai_agent_reads_secrets.yml`](proc_ai_agent_reads_secrets.yml) | AI coding agent / MCP server spawns process touching SSH keys, cloud creds, `.env` | Process creation | high | T1552.001, T1552.004 |
| [`net_ai_agent_exfil_destinations.yml`](net_ai_agent_exfil_destinations.yml) | Agent runtimes connecting to webhook / tunnel / paste services | Network connection | medium | T1567 |
| [`file_mcp_config_modified.yml`](file_mcp_config_modified.yml) | MCP client config created or changed (new tools for your agent) | File event | low | T1546 |
| [`web_exposed_ollama_model_tampering.yml`](web_exposed_ollama_model_tampering.yml) | Ollama pull/create/push/delete from external IPs | Proxy | high | T1190 |

Each rule links to the matching play in [ai-attack-playbook](https://github.com/Nexqor/ai-attack-playbook).

## Convert to your SIEM

```bash
pip install sigma-cli
sigma plugin install splunk          # or: microsoft365defender, elasticsearch, ...

sigma convert -t splunk -p sysmon proc_ai_agent_reads_secrets.yml
sigma convert -t lucene -p ecs_windows net_ai_agent_exfil_destinations.yml
```

> Multi-document files (`---`) contain related rules; most converters handle them directly.

## Tuning notes

- **`proc_ai_agent_reads_secrets`**: matching on `ParentCommandLine` keeps it generic across Claude Code, Cursor, Copilot, Gemini CLI and MCP servers. If your EDR records the full process tree, anchor on the agent's root process instead.
- **`web_llm_gateway_admin_key_endpoints`**: replace the private CIDR filter with your actual admin jump-host ranges.
- **`net_ai_agent_exfil_destinations`**: expect noise on developer machines that test webhooks; suppress per user, not globally.
- **`file_mcp_config_modified`**: low on its own; raise severity when the writing process is not the IDE/client itself, or chain it with [mcp-audit](https://github.com/Nexqor/mcp-audit) to inspect the new config automatically.

## Detection philosophy

Prompt injection itself is hard to detect reliably. Its **effects** are not: an agent reading `id_rsa`, a gateway minting keys for an unknown IP, logging switched off right before a spike in model usage. These rules watch for the effects.

## Status

All rules are `experimental`: written from public incident reporting and attack research, and meant to be tuned to your environment. PRs with field-tested tuning or new rules are welcome.

## Related

- [ai-attack-playbook](https://github.com/Nexqor/ai-attack-playbook): the attacks behind each rule
- [mcp-audit](https://github.com/Nexqor/mcp-audit): static audit for MCP configs
- [rag-canary](https://github.com/Nexqor/rag-canary): canary tokens for RAG leaks
- [AI Agent Security Checklist](https://nexqor.github.io/checklist.html)

---

MIT License · [Nexqor](https://github.com/Nexqor) · [@whyNexqor](https://x.com/whyNexqor)
