---
name: pr-auditor
description: Executa auditoria técnica completa ou follow-up de Pull Requests, validando código, contratos, testes, builds, CI, documentação e conformidade com a descrição oficial do PR (incluindo o template de Identificação, Entregas, Checklist de Padrões e Como Testar).
---

# Auditoria e Correção de Pull Requests

Sempre que o usuário solicitar a auditoria de um PR, análise de branch para merge, follow-up de correções de PR ou revisão antes de integração, execute rigorosamente o protocolo abaixo.

## 1. Coleta de Contexto e Identificação do Modo

Antes de escrever qualquer análise, inspecione o ambiente via terminal/ferramentas:
1. Identifique a branch base (ex: `dev` ou `main`), o commit `merge-base`, o `head` SHA atual, autor, escopo do diff (`git diff --stat base..head`) e o texto da descrição do PR (via `gh pr view`, MCP do GitHub ou fornecido no prompt pelo usuário).
2. Leia as regras locais do projeto (`.agents/AGENTS.md`, `.github/pull_request_template.md`, `README.md` e documentos em `Docs/` ou `docs/`) para conhecer os contratos globais, padrões de arquitetura, fuso horário de negócio, formatos de dados (ex: medidas em centímetros, cores `{ name, hex_code }`, IDs) e regras de documentação.
3. Determine o **Modo de Execução**:
   - **Modo Auditoria Completa (PR novo ou primeira análise):** Aplica-se na primeira revisão do PR.
   - **Modo Follow-up (Reauditoria de commits):** Aplica-se quando já houve uma auditoria anterior e novos commits foram enviados para corrigir pendências.

---

## 2. Protocolo de Verificação Técnica (Obrigatório)

Execute todas as validações aplicáveis ao projeto (frontend, backend, banco/ORM):
1. **Integridade do Git:** Execute `git diff --check base..head` para garantir ausência de marcadores de conflito ou problemas de whitespace.
2. **Checagem Estática e Build:** Execute `typecheck` (`npm run typecheck` ou `npx tsc --noEmit`), `lint` (`npm run lint`) e `build` (`npm run build` / `./mvnw test-compile`). Diferencie falhas introduzidas pelo PR de dívidas preexistentes na branch base.
3. **Suíte de Testes e Reprodução de Falhas:**
   - Rode os testes existentes (`npm test` / `./mvnw test`).
   - Se identificar suspeita de bug funcional, condição de corrida (race condition), quebra de autorização/entitlement, problema de fuso horário, falha de idempotência ou quebra de contrato: **escreva um teste automatizado direcionado** para provar o cenário, execute-o para confirmar a reprodução e **remova qualquer arquivo temporário** antes de encerrar, mantendo o worktree limpo (`git status` limpo).
4. **Cruzamento Descrição (Template do PR) vs. Código:**
   Valide cada seção do comentário geral do PR fornecido:
   - **📌 Identificação do Pull Request:** Verifique se o desenvolvedor está marcado (`[x] Atus` | `[x] Flavio` | `[x] Alef` | `[x] Rian`) e se o Card/Onda corresponde ao escopo do diff.
   - **🎯 O que foi entregue & 💡 Melhorias:** Confirme se cada item listado (ex: tabelas, migrations, seeds idempotentes, scripts, requisitos de runtime) existe de fato e funciona conforme descrito.
   - **🛡️ Checklist de Padrões do Projeto:**
     - *Prevenção de Conflitos Git:* Verifique se o PR invadiu arquivos/domínios de outros desenvolvedores sem justificativa.
     - *Qualidade Técnica e Contratos Globais:* Valide `typecheck`, `test`, `build`, padrão de medidas (ex: dimensões em centímetros), padrão de cores (ex: `{ name, hex_code }`), ausência de dados sensíveis/credenciais e separação de conexões de banco (ex: `DATABASE_URL` e `DIRECT_URL`) quando aplicável.
   - **🧪 Como Testar Rapidamente:** Verifique se os comandos documentados funcionam e batem com o `package.json` / `pom.xml`.
   - **Documentação:** Se o PR marcar que "documentação não se aplica", valide se algum documento oficial em `Docs/` ou `README.md` ficou obsoleto em decorrência das mudanças.
5. **CI Remoto:** Se `gh` ou MCP estiver disponível, consulte o status dos checks do `head` SHA. Caso não esteja acessível no ambiente, registre explicitamente a limitação sem inventar resultados.

---

## 3. Classificação de Severidade

Classifique cada achado em exatamente um nível:
- **[Alta] (Bloqueia o merge):** Bugs funcionais graves, perda/regressão de estado em cache ou banco, falhas de segurança/autorização/bypass de plano, exposição de credenciais ou quebra de contrato crítico.
- **[Média] (Deve ser corrigido no mesmo ciclo):** Falhas de cobertura obrigatória de testes, inconsistência de fuso horário/formatação, mensagens de erro contraditórias na UI, checklists/templates inválidos no CI ou documentação oficial desatualizada.
- **[Baixa] (Correção recomendada):** Compatibilidade com dados/modalidades legadas, edge cases de UX/sessão ou melhorias pontuais de resiliência.

---

## 4. Formato de Saída — Modo Auditoria Completa

Use esta estrutura exata quando for a auditoria principal do PR:

```markdown
# Auditoria — PR #<número> · <Título objetivo>

**Repositório:** `<org/repo>`  
**Issue/card:** <Card Referência da de descrição>  
**Branch:** `<head-branch>` → `<base-branch>`  
**Head SHA:** `<sha-curto>` (`<sha-completo>`)  
**Base auditada:** `<base-branch>` em `<base-sha-curto>`  
**Autor do commit:** `<autor>`  
**Escopo do diff:** <X> arquivos · +<adições> / -<remoções> linhas  
**CI remoto:** <Status caso checks dos inacessível justificativa ou>  
**Validação local:** <Resumo build dos e executados lint, testes, typecheck>  
**Checklist na descrição:** <Avaliação conformidade da declarações do e objetiva template>  
**Veredito:** **<Aprovar Solicitar alterações antes do merge |>.** <Resumo 1 de do frase motivo.>

---

## Resumo executivo

<2 parágrafos curtos: o que o PR entrega e quais são os principais riscos/achados encontrados.>

### Contagem dos achados

| Severidade | Quantidade | Situação |
|---|---:|---|
| Alta | X | Bloqueiam o merge |
| Média | X | Devem ser corrigidos no mesmo ciclo |
| Baixa | X | Correção recomendada |
| **Total** | **X** | — |

---

## Achados

### **[Alta | Média | Baixa] <Título claro do problema> — `<arquivo:linha>`**

**Cenário:** <Explicação código de do e fluxo relevante. trecho técnica>

**Evidência reproduzida:** <Como análise dirigido. foi na ou teste validado/reproduzido>

**Impacto:** <Consequência o operação ou para real segurança. usuário,>

**Correção esperada:** <Solução adicionados. devem e objetiva quais ser testes técnica>

---

## Itens verificados sem defeito

| Área | Resultado |
|---|---|
| <Item/Contrato verificado> | ✅ <Confirmado com curta evidência> |

---

## Validações executadas

| Verificação | Resultado |
|---|---|
| `<comando executado>` | <✅ ou ❌ ou ⚠️ com resumo objetivo> |

---

## Cobertura de testes do PR

<Análise ausentes breve cenários cobertos. de devem dos e existentes lista numerada que ser testes>

---

## Conformidade com a descrição do PR

| Declaração da descrição | Auditoria |
|---|---|
| <Item PR declarado no> | <✅ Confirmado | ⚠️ Parcial | ❌ Incorreto + motivo> |

---

## Registro

**PR #<número>** · head `<sha-curto>` · base `<base-branch>` em `<base-sha-curto>`  
**Commits auditados:** `<lista de commits>`  
**Arquivos modificados auditados:** <lista de arquivos>  
**Limitações:** <O ambiente neste não pôde que ser testado>  
**Alterações realizadas pela auditoria:** Nenhuma alteração persistente aplicada ao repositório; worktree limpo ao final.  
**Recomendação final:** **<Decisão destrava do e final lista merge. o objetiva que>**

```

---

## 5. Formato de Saída — Modo Follow-up (Reauditoria)

Use esta estrutura quando estiver auditando novos commits enviados após uma revisão anterior:

```markdown
## Follow-up da auditoria do PR #<número>

**Resultado: <X> Alta, <Y> Média, <Z> Baixa. Snapshot atual:** head `<sha-completo>`, base `<base-sha-completo>` (`<base-branch>`). **Conclusão: <Aprovado antes correção/decisão do merge para requer |>** (<resumo curto>).

### Pendências

**[Alta | Média | Baixa] <Título da pendência> — `<arquivo:linha>`**

<Explicação ainda ajustar, direta do e falta feito. o precisa projeto que referenciando regras ser>

### Estado das correções verificadas no snapshot atual

| Item verificado | Estado |
|---|---|
| <Descrição anteriormente correção da solicitada> | ✅ **Resolvido no commit `<sha-curto>`.** <Evidência técnica.> |

### Declarações do template

- <✅ | ❌ | ➖> **<Categoria checklist do>:** <Verificação CI. contra código, docs e objetiva>

### Verificações

- **Executado localmente no head `<sha-curto>`:**
  - `<comando>` — <resultado>
- **Lido no CI do head `<sha-completo>`:**
  - `<check>` — <status>

### Registro

- **PR:** `<org/repo>#<número>` (`<head-branch>` → `<base-branch>`).
- **Base (`origin/<base-branch>` e `merge-base`):** `<base-sha-completo>`.
- **Head auditado:** `<sha-completo>`.
- **Escopo coberto:** <resumo dos arquivos/linhas e delta de commits>.

### Conclusão

**<Decisão final>.** <Lista dos exatos liberar merge. numerada o para passos>

```

---

## 6. Regras Inegociáveis

1. **Nunca deixe sujeira no repositório:** Se criar testes temporários para provar um bug, apague-os antes de finalizar e confirme que o `git status` está idêntico ao início.
2. **Não invente evidências:** Nunca afirme que um check do CI passou se você não leu o log/status dele. Se não tiver acesso ao GitHub CLI ou API, informe na seção de limitações.
3. **Respeite o domínio e contratos do projeto:** Sempre valide regras específicas descritas no PR ou na base (ex: medidas em centímetros, cores `{ name, hex_code }`, idempotência de seed, fusos horários de negócio, isolamento entre usuários/lojas).
4. **Foco em correção acionável:** Ao apontar qualquer falha, indique sempre o arquivo, a linha exata, o porquê da falha e como corrigir.
