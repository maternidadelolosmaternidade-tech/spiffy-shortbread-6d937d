---
name: "ata-reuniao-compliance"
description: "Use quando Igor enviar ou indicar uma gravação, transcrição ou anotações de reunião (Notion Meeting Notes, Teams, Google Meet, áudio, texto colado) para gerar ata, decisões e plano de ação (o quê / quem / até quando), registrar no Notion, criar lembretes no Calendar, deixar rascunho de e-mail de follow-up e acionar a auditoria de compliance. Também usar na varredura diária de reuniões não processadas. Dispara com: \"processa a reunião\", \"faz a ata\", \"o que ficou de ação\", \"/ata-reuniao-compliance\"."
---

# Reunião → Ata + Plano de Ação + Auditoria de Compliance

## Destinos fixos (Notion)
- Base **🎙️ Reuniões** — data source `collection://f1b0366e-0109-41a6-a21b-49c42176cb00`
  (https://app.notion.com/p/e1f56f2e26fd4d80a9d4429d48bb1a72)
- Base **✅ Ações de Reunião** — data source `collection://c7af2477-9cc6-48cb-ba23-7629a994cfc0`
  (https://app.notion.com/p/ebad3ecd74b4422ea0db5a9dfe512155) — relação `Reunião` ↔ `Ações`.

Não criar bases novas. Não alterar a base "Brief diário — Assistente" nem as de PSCPP.

## 1. Entrada (localizar a transcrição)
| Origem | Como ler |
|---|---|
| Notion Meeting Notes (padrão no celular) | `notion-query-meeting-notes` (filtro de data) → `notion-fetch` da página |
| Teams | `sharepoint_search` pela transcrição (.vtt/.docx) em OneDrive/Recordings → `read_resource` |
| Google Meet | Gmail `search_threads` "Anotações do Gemini" / "Notes by Gemini" → Doc |
| Texto colado / arquivo | usar direto |
| Áudio avulso sem transcrição | GAP: pedir para abrir no Notion/Teams para transcrever. Não inventar conteúdo. |

Antes de processar, consultar a base Reuniões pelo link/título: se já existir com `Processado = true`, não duplicar — apenas atualizar se Igor pedir.

## 2. Ata (corpo da página na base Reuniões)
1. **Cabeçalho**: data, tipo (Bordo/Capital/Petrobras/DNV/Outros), participantes, duração, origem, link da gravação.
2. **Resumo executivo**: 5–8 linhas.
3. **Decisões tomadas**: lista objetiva.
4. **Plano de ação**: tabela `O quê | Quem | Até quando | Trecho-fonte`.
   - Prazo não dito → `GAP / prazo não definido` (`Prazo definido? = falso`). Nunca estimar prazo.
   - Responsável não dito → `GAP / responsável não definido`.
   - Datas relativas ("sexta", "semana que vem") → converter para data absoluta com base na data da reunião e marcar `(convertido)`.
5. **Auditoria de compliance** — ver §4.
6. **Pendências e inaudíveis**: trechos `[inaudível]`, pontos a confirmar, correções incertas do reconhecimento de voz.

Regras: não inventar o que não está na transcrição; tom formal, técnico, direto; corrigir erros evidentes de transcrição sinalizando quando incertos.

## 3. Distribuição
1. Criar/atualizar a página em **Reuniões** (propriedades: Data, Tipo, Participantes, Origem, Gravação / Transcrição, Resumo ≤ 2 frases, Auditoria, Processado = true).
2. Criar um item em **Ações de Reunião** por ação (Responsável, Prazo, Prioridade, Trecho-fonte, relação Reunião, Status = A fazer).
3. Ações com prazo definido e responsável = Igor (ou a bordo do ACE DEFENDER) → evento de dia inteiro no Google Calendar (`create_event`), título `[Ação REU-n] <ação>`, descrição com link da página; marcar `No Calendar = true`. Não convidar terceiros.
4. E-mail de follow-up com a ata → **somente rascunho** (Gmail `create_draft`; Outlook `outlook_create_draft` se a reunião for de trabalho no M365). **Nunca enviar.** Destinatários = participantes com e-mail conhecido; na dúvida, deixar em branco.
5. Responder a Igor em até 6 linhas: link da página, nº de ações, prazos mais próximos, status da auditoria, GAPs.

## 4. Auditoria de compliance (módulo externo — não implementar aqui)
A auditoria é feita pelo **módulo de compliance que Igor está construindo em outra conversa**.
- Se o módulo estiver disponível (skill instalada — nome a confirmar; procurar na lista de skills por "compliance"/"auditoria"): invocá-lo com **entrada** = transcrição + resumo + decisões + plano de ação + tipo da reunião; **saída esperada** = achados `Tema | Norma + item exato | Status (Conforme / Atenção / Não conforme / GAP) | Recomendação`. Inserir os achados na seção 5 da ata e gravar o pior status no campo `Auditoria`.
- Se o módulo **não** estiver disponível: seção 5 = `Auditoria pendente — módulo de compliance`, lista dos temas normativos identificados (sem citar itens de memória), `Auditoria = Pendente — módulo de compliance`. A varredura diária reprocessa essas reuniões quando o módulo existir.
- Em qualquer caso, nunca atribuir à norma o que ela não diz; item não localizado → `GAP / base não localizada` (padrão de `analise-normativa-fundamentada`).

## 5. Modo varredura diária (tarefa agendada)
1. `notion-query-meeting-notes` com `created_time` nas últimas 24 h + Gmail (Gemini) + Teams (últimas 24 h).
2. Ignorar o que já está em Reuniões com `Processado = true`.
3. Processar cada uma (§1–§4).
4. Reprocessar só a auditoria das reuniões com `Auditoria = Pendente — módulo de compliance`, se o módulo já existir.
5. Fechar com resumo curto: reuniões processadas, ações novas, ações vencendo em 7 dias (consultar Ações de Reunião, Status ≠ Concluída), GAPs.

## Privacidade
Gravação envolve dados pessoais (LGPD — Lei 13.709/2018; item específico: GAP até verificação na fonte). Lembrar Igor, uma vez por reunião nova, de avisar os participantes no início da gravação. Conteúdo Petrobras/Capital fica apenas no Notion/e-mail de Igor; não publicar artifacts públicos.
