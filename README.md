# pr-auditor

Skill universal para **Antigravity**, **Claude Code** e **Codex** que executa auditoria técnica completa ou follow-up (reauditoria) de Pull Requests, validando código, contratos globais, testes automatizados, builds, CI, documentação e conformidade com a descrição oficial do PR.

## O que ela faz

- **Modo Auditoria Completa:** Analisa PRs novos validando integridade do Git (`git diff --check`), checagem estática (`typecheck`, `lint`, `build`), execução da suíte de testes e criação de testes direcionados para reproduzir bugs sem sujar o repositório ao final.
- **Modo Follow-up (Reauditoria):** Revalida novos commits enviados após uma revisão anterior, atualizando o estado de cada pendência com evidência técnica por commit.
- **Cruzamento Template vs. Código:** Verifica todas as seções do template do PR (Identificação do PR, O que foi entregue & Melhorias, Checklist de Padrões do Projeto, Como Testar Rapidamente e Documentação).
- **Classificação Estrita de Severidade:** Categoriza achados em `[Alta]` (bloqueia o merge), `[Média]` (deve ser corrigido no mesmo ciclo) e `[Baixa]` (correção recomendada).

---

## Instalação

### 1. Via `npx skills` (Universal — Antigravity, Claude Code, Codex, Cursor)
```bash
npx skills add FlavioJr0010/pr-auditor
```

### 2. Instalação Global Manual (para funcionar em qualquer projeto da máquina)
```bash
# Antigravity
mkdir -p ~/.agent/skills/pr-auditor ~/.agents/skills/pr-auditor
cp pr-auditor/SKILL.md ~/.agent/skills/pr-auditor/SKILL.md
cp pr-auditor/SKILL.md ~/.agents/skills/pr-auditor/SKILL.md

# Claude Code
mkdir -p ~/.claude/skills/pr-auditor
cp pr-auditor/SKILL.md ~/.claude/skills/pr-auditor/SKILL.md

# OpenAI Codex
mkdir -p ~/.codex/skills/pr-auditor
cp pr-auditor/SKILL.md ~/.codex/skills/pr-auditor/SKILL.md
```

### 3. Instalação por Projeto (Local no Repositório)
- **Antigravity:** `.agent/skills/pr-auditor/SKILL.md` ou `.agents/skills/pr-auditor/SKILL.md`
- **Claude Code:** `.claude/skills/pr-auditor/SKILL.md`
- **Codex:** `.codex/skills/pr-auditor/SKILL.md`

---

## Como usar (Antigravity, Claude Code e Codex)

### 1. Auditoria Completa de um PR
No chat do agente (**Antigravity**, **Claude Code** via `/pr-auditor` ou prompt direto, ou **Codex** via `$pr-auditor` ou prompt direto):

```text
Use a skill pr-auditor para auditar este PR contra a branch dev.
Descrição do PR:
[cole o comentário geral do PR ou informe o número do PR]
```

### 2. Follow-up (Reauditoria de novos commits)
Quando novos commits forem enviados para corrigir apontamentos:

```text
Use a skill pr-auditor em modo follow-up para verificar os novos commits deste PR.
```

---

by [@FlavioJr0010](https://github.com/FlavioJr0010)
