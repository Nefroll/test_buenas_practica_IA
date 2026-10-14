---
name: Agente maestro
description: llama agentes esclavos
tools: ['read', 'search', 'edit', 'agent']
agents: ['agente-qa-esclavo']
user-invocable: true
disable-model-invocation: false
---

# Instructions
Cuando se te llame quiero que invoque al agente "agente-qa-esclavo" y al agente "agente-qa-esclavo-2"
