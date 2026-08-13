# Envio de Mensagens - Movimentações Internas (Slack)

## 1. Resumo

**O que esta automação faz:** envia mensalmente uma mensagem no Slack para o canal `#ca-lideres`, lembrando a liderança sobre o prazo de solicitação de movimentações internas. Antes do envio no canal, a automação envia uma mensagem de aprovação para Matheus Guesser; a mensagem final só é publicada se houver aprovação.

**Área responsável:** People — Remuneração / Comp & People Analytics

**Responsável de negócio:** Matheus Guesser

**Responsável técnico:** Matheus Guesser

**Status:** Em produção

**Data de criação:** 13/08/2026 — **Última revisão:** 13/08/2026

## 2. Problema que resolve

Reduz o trabalho manual e recorrente de lembrar a liderança, no Slack, sobre o prazo mensal de solicitações de movimentações internas.

- **Antes da automação:** o lembrete precisava ser preparado e enviado manualmente no canal, com risco de esquecimento, atraso ou envio sem revisão.
- **Depois da automação:** o lembrete é montado automaticamente, passa por aprovação humana e só então é publicado no canal `#ca-lideres`. O resultado do envio é registrado em planilha.

## 3. Quando a automação é executada

**Gatilho:** agendamento mensal no n8n.

**Frequência:** mensal.

**Canal de destino:** `#ca-lideres`.

**Aprovação:** antes de publicar no canal, o fluxo envia uma mensagem de aprovação para Matheus Guesser no Slack.

## 4. Fluxo da automação

1. O workflow **Envio de Mensagens - Movimentações Internas (Slack)** inicia pelo nó **Gatilho Mensal**.
2. O fluxo chama o workflow auxiliar de dica mensal pelo nó **Google Sheets - Ler Dica1**.
3. A dica do mês é retornada para compor o texto do lembrete.
4. O nó **Elaborar mensagem1** monta a mensagem final que será enviada ao canal.
5. Antes do envio público, o nó **Mensagem de Aprovação1** envia a prévia para aprovação.
6. O nó **Switch1** avalia a decisão da aprovação.
7. Se aprovado, o nó **Envio Mensagem1** publica a mensagem no canal `#ca-lideres`.
8. Após envio aprovado, o nó **Log de Envio1** registra o envio na planilha.
9. Se recusado, o nó **Log - Recusado1** registra a recusa.
10. Em caso de recusa, o nó **Aviso - Recusado1** envia um aviso no Slack.
11. O fluxo deve manter tratamento correto para timeout, evitando que ausência de resposta seja interpretada como reprovação.

## 5. Ferramentas utilizadas

| Ferramenta | Uso |
|---|---|
| n8n | Orquestração do fluxo, gatilho mensal, aprovação, roteamento e logs |
| Slack | Envio da mensagem de aprovação e publicação do lembrete no canal `#ca-lideres` |
| Google Sheets | Fonte da dica mensal e registro dos logs de envio |

## 6. Entradas e resultados

| Informação | Origem | É dado sensível? |
|---|---|---|
| Dica mensal | Workflow/planilha de dicas de movimentações internas | Não |
| Mensagem de lembrete | Nó de montagem da mensagem no n8n | Não |
| Decisão de aprovação | Mensagem interativa no Slack enviada para Matheus | Não |
| Canal de destino | Configuração do workflow no n8n | Não |
| Log de envio | Planilha de log | Não |

**Resultado gerado:** mensagem publicada no canal `#ca-lideres` após aprovação humana + linha de log com o status do envio.

## 7. Regras principais

- A mensagem final só pode ser publicada no canal após aprovação humana.
- O aprovador oficial é Matheus Guesser.
- O canal de destino em produção é `#ca-lideres`.
- A dica muda mensalmente e vem de uma base/planilha de dicas.
- O fluxo deve registrar log para envio aprovado e envio recusado.
- Timeout não deve ser tratado como reprovação automática.
- A regra de reprovação no **Switch** deve usar comparação estrita: `{{ $json.data?.approved === false }}` com operador **is true**.
- A regra de aprovação pode usar comparação estrita: `{{ $json.data?.approved === true }}` com operador **is true**.
- O nó de aprovação precisa estar conectado ao **Switch**; se essa conexão quebrar, o fluxo não continua.
- Em testes, diferenciar envio para usuário/DM e envio para canal para evitar publicar no destino errado.

## 8. Mensagem ou conteúdo enviado

**Canal de destino:** `#ca-lideres`

**Tipo de mensagem:** lembrete mensal sobre solicitação de movimentações internas.

**Conteúdo esperado:** mensagem para líderes reforçando o prazo mensal de solicitação de movimentações internas e incluindo a dica do mês.

**Campo dinâmico principal:** dica mensal retornada pelo workflow/planilha de dicas.

## 9. Como testar

1. Executar o workflow manualmente no n8n.
2. Conferir se a dica mensal foi retornada corretamente.
3. Conferir se a mensagem de aprovação chegou para Matheus no Slack.
4. Aprovar o envio e validar se a mensagem foi publicada no canal correto.
5. Reexecutar o teste recusando o envio e validar se a mensagem não foi publicada no canal.
6. Validar se o log de envio aprovado foi registrado na planilha.
7. Validar se o log de envio recusado foi registrado na planilha.
8. Testar cenário de timeout e confirmar que ele não cai no ramo “Reprovado”.
9. Conferir se o canal de destino está como `#ca-lideres`, e não como DM de teste.
10. Conferir se o workflow está ativo em produção.

**Resultado esperado do teste:** mensagem publicada apenas quando aprovada, log correto em cada cenário e nenhum envio indevido em caso de recusa ou timeout.

## 10. Tratamento de erros

| Possível problema | O que fazer |
|---|---|
| Mensagem de aprovação não segue para o Switch | Conferir se o nó **Mensagem de Aprovação1** está conectado ao nó **Switch1** |
| Timeout cai como reprovado | Usar comparação estrita no Switch: `{{ $json.data?.approved === false }}` com operador **is true** |
| Mensagem enviada para usuário em vez de canal | Conferir se o nó de envio final está configurado como canal e apontando para `#ca-lideres` |
| Mensagem não é enviada após aprovação | Validar saída “Aprovado” do Switch e conexão com o nó de envio no Slack |
| Log não é registrado | Conferir permissões da planilha e mapeamento de colunas no nó de Google Sheets |
| Dica não aparece na mensagem | Conferir o workflow auxiliar de dica mensal e o mapeamento do campo de dica no nó de mensagem |
| Envio recusado não avisa o responsável | Conferir os nós **Log - Recusado1** e **Aviso - Recusado1** |

## 11. Segurança e privacidade

- Não incluir tokens, chaves, segredos, credenciais ou URLs sensíveis de webhook no Notion ou no GitHub.
- As credenciais de Slack e Google Sheets devem ficar apenas nas conexões seguras do n8n.
- A mensagem não deve conter dados pessoais sensíveis.
- Antes de subir o JSON no GitHub, revisar e remover credenciais, IDs sensíveis, URLs internas ou metadados que não devam ser expostos.
- A publicação no canal depende de revisão humana, reduzindo risco de mensagem incorreta para a liderança.

**Dados sensíveis envolvidos:** não identificados. A automação usa conteúdo operacional e logs de envio.

## 12. Links importantes

- **Canal de destino:** `#ca-lideres`
- **Workflow n8n:** [PREENCHER]
- **Planilha de log:** [PREENCHER]
- **Repositório GitHub:** [PREENCHER]
- **Documentação técnica:** [PREENCHER]

## 13. Como pausar ou desativar

No n8n, abrir o workflow **Envio de Mensagens - Movimentações Internas (Slack)** e desativar o botão **Active/Ativo**.

Se for necessário impedir apenas o envio público, remover temporariamente o canal `#ca-lideres` do nó de envio final ou trocar por um destino de teste.

## 14. Manutenção

**Quem deve revisar:** Matheus Guesser e time de Remuneração / Comp & People Analytics.

**Quando revisar:**

- sempre que o processo de movimentações internas mudar;
- sempre que o texto da mensagem mudar;
- sempre que o canal de destino mudar;
- sempre que a lógica de aprovação mudar;
- sempre que a planilha de dicas/log mudar;
- a cada nova versão publicada.

## 15. Histórico de mudanças

| Data | Alteração | Responsável |
|---|---|---|
| 13/08/2026 | Criação da documentação inicial da automação de lembrete de movimentações internas no Slack, com aprovação humana antes do envio ao canal `#ca-lideres`. | Matheus Guesser |

## 16. Código / prompt / fluxo na íntegra

Os arquivos técnicos originais usados para documentar esta automação foram:

- `Envio_de_Mensagens_-_Movimentacoes_Internas_Slack.json`
- `Automação - Envio Slack liderança.xlsx`

Para versionamento no GitHub, subir os arquivos sanitizados junto com:

- `README.md`
- `documentacao.md`

> Antes de publicar, remover credenciais, tokens, URLs sensíveis de webhook, IDs internos que não devam ser expostos e qualquer dado pessoal desnecessário.

## Checklist para ativação

- [x] A automação está em produção.
- [x] O canal de destino foi definido como `#ca-lideres`.
- [x] O aprovador oficial foi definido.
- [ ] O link do workflow n8n foi registrado.
- [ ] O link da planilha de log foi registrado.
- [ ] O link do repositório GitHub foi registrado.
- [x] Existe uma forma simples de pausar a automação.
- [x] A mensagem depende de aprovação humana antes de ser publicada.
- [x] A automação registra log de envio.
- [ ] O fluxo de timeout foi validado em produção.
