# Documentação — Envio de Mensagens - Movimentações Internas (Slack)

## 1. Resumo

**Nome da automação:** Envio de Mensagens - Movimentações Internas (Slack)  
**Status:** Ativa / em produção  
**Área responsável:** People  
**Time mantenedor:** People  
**Canal de suporte:** Time de Remuneração (Comp & People Analytics)  
**Tipo:** Automação simples  
**Forma de execução:** Agendada  
**Ambiente:** Produção  
**Versão atual:** 1.0  
**Data de criação:** 13/08/2026  
**Última revisão:** 13/08/2026  
**Próxima revisão:** 10/09/2026  
**Responsável de negócio:** Matheus Guesser  
**Responsável técnico:** Matheus Guesser  
**Criticidade:** Média  
**Usa IA?:** Não  
**Usa dados pessoais?:** Não  
**Usa dados sensíveis?:** Não  

Esta automação envia mensalmente uma mensagem no Slack para o canal `#ca-lideres`, lembrando a liderança sobre o prazo de solicitação de movimentações internas.

Antes do envio no canal, a automação envia uma mensagem de aprovação para Matheus Guesser. A mensagem final só é publicada se houver aprovação humana.

---

## 2. Problema que resolve

A automação reduz o trabalho manual e recorrente de lembrar a liderança, no Slack, sobre o prazo mensal de solicitações de movimentações internas.

### Antes da automação

- O lembrete precisava ser preparado manualmente.
- O envio dependia de uma ação manual recorrente.
- Havia risco de esquecimento, atraso ou envio incorreto.

### Depois da automação

- O lembrete é montado automaticamente.
- A mensagem passa por aprovação humana antes de ser publicada.
- O envio só ocorre após aprovação.
- O resultado do envio é registrado em planilha.

---

## 3. Quando a automação é executada

**Gatilho:** agendamento mensal no n8n  
**Frequência:** mensal  
**Canal de destino:** `#ca-lideres`  
**Aprovação:** antes da publicação, o fluxo envia uma mensagem de aprovação para Matheus Guesser no Slack.

---

## 4. Fluxo da automação

1. O workflow **Envio de Mensagens - Movimentações Internas (Slack)** inicia pelo nó **Gatilho Mensal**.
2. O fluxo chama o workflow auxiliar de dica mensal pelo nó **Google Sheets - Ler Dica**.
3. A dica do mês é retornada para compor o texto do lembrete.
4. O nó **Elaborar mensagem** monta a mensagem final que será enviada ao canal.
5. Antes do envio público, o nó **Mensagem de Aprovação** envia a prévia para aprovação.
6. O nó **Switch** avalia a decisão da aprovação.
7. Se aprovado, o nó **Envio Mensagem** publica a mensagem no canal `#ca-lideres`.
8. Após envio aprovado, o nó **Log de Envio** registra o envio na planilha.
9. Se recusado, o nó **Log - Recusado** registra a recusa.
10. Em caso de recusa, o nó **Aviso - Recusado** envia um aviso no Slack.

---

## 5. Ferramentas utilizadas

| Ferramenta | Uso |
|---|---|
| n8n | Orquestração do fluxo, gatilho mensal, aprovação, roteamento e logs |
| Slack | Envio da mensagem de aprovação e publicação do lembrete no canal `#ca-lideres` |
| Google Sheets | Fonte da dica mensal e registro dos logs de envio |

---

## 6. Entradas e resultados

| Informação | Origem | É dado sensível? |
|---|---|---|
| Dica mensal | Workflow/planilha de dicas de movimentações internas | Não |
| Mensagem de lembrete | Nó de montagem da mensagem no n8n | Não |
| Decisão de aprovação | Mensagem interativa no Slack enviada para Matheus | Não |
| Canal de destino | Configuração do workflow no n8n | Não |
| Log de envio | Planilha de log | Não |

**Resultado gerado:** mensagem publicada no canal `#ca-lideres` após aprovação humana, além de uma linha de log com o status do envio.

---

## 7. Regras principais

- A mensagem final só pode ser publicada no canal após aprovação humana.
- O aprovador oficial é Matheus Guesser.
- O canal de destino em produção é `#ca-lideres`.
- A dica muda mensalmente e vem de uma base/planilha de dicas.
- O fluxo deve registrar log para envio aprovado e envio recusado.
- A regra de reprovação no **Switch** deve usar comparação estrita com operador **is true**.
- O nó de aprovação precisa estar conectado ao **Switch**. Se essa conexão quebrar, o fluxo não continua.
- Em testes, é necessário diferenciar envio para usuário/DM e envio para canal, para evitar publicação no destino errado.

---

## 8. Mensagem ou conteúdo enviado

**Canal de destino:** `#ca-lideres`  
**Tipo de mensagem:** lembrete mensal sobre solicitação de movimentações internas.  

**Conteúdo esperado:** mensagem para líderes reforçando o prazo mensal de solicitação de movimentações internas e incluindo a dica do mês.

**Campo dinâmico principal:** dica mensal retornada pelo workflow/planilha de dicas.

---

## 9. Como testar

1. Executar o workflow manualmente no n8n.
2. Conferir se a dica mensal foi retornada corretamente.
3. Conferir se a mensagem de aprovação chegou para Matheus no Slack.
4. Aprovar o envio e validar se a mensagem foi publicada no canal correto.
5. Reexecutar o teste recusando o envio e validar se a mensagem não foi publicada no canal.
6. Validar se o log de envio aprovado foi registrado na planilha.
7. Validar se o log de envio recusado foi registrado na planilha.
8. Conferir se o canal de destino está como `#ca-lideres`, e não como DM de teste.
9. Conferir se o workflow está ativo em produção.

**Resultado esperado do teste:** mensagem publicada apenas quando aprovada, log correto em cada cenário e nenhum envio indevido em caso de recusa ou timeout.

---

## 10. Tratamento de erros

| Possível problema | O que fazer |
|---|---|
| Mensagem de aprovação não segue para o Switch | Conferir se o nó **Mensagem de Aprovação** está conectado ao nó **Switch** |
| Mensagem enviada para usuário em vez de canal | Conferir se o nó de envio final está configurado como canal e apontando para `#ca-lideres` |
| Mensagem não é enviada após aprovação | Validar saída “Aprovado” do Switch e conexão com o nó de envio no Slack |
| Log não é registrado | Conferir permissões da planilha e mapeamento de colunas no nó de Google Sheets |
| Dica não aparece na mensagem | Conferir o workflow auxiliar de dica mensal e o mapeamento do campo de dica no nó de mensagem |
| Envio recusado não avisa o responsável | Conferir os nós **Log - Recusado** e **Aviso - Recusado** |

---

## 11. Segurança e privacidade

- Não incluir tokens, chaves, segredos, credenciais ou URLs sensíveis de webhook no Notion ou no GitHub.
- As credenciais de Slack e Google Sheets devem ficar apenas nas conexões seguras do n8n.
- A mensagem não deve conter dados pessoais sensíveis.
- A publicação no canal depende de revisão humana, reduzindo risco de mensagem incorreta para a liderança.

**Dados sensíveis envolvidos:** não identificados. A automação usa conteúdo operacional e logs de envio.

---

## 12. Links importantes

- **Canal de destino:** `#ca-lideres`
- **Workflow n8n:** https://contaazul.app.n8n.cloud/workflow/URjQWIhF9WwXBVVG
- **Planilha de log:** https://docs.google.com/spreadsheets/d/1j0aY62reb3WJb1JwE1Xmh3fvLjcHz6m-ZuqWjKFb3l4/edit?usp=sharing
- **Repositório GitHub:** https://github.com/matheus-guesser-ca/remuneracao-lembrete-mov-internas-slack
- **Documentação técnica:** https://drive.google.com/drive/folders/1DHivWBZcQUYe0eXK7KdHF61piy89LIAF?usp=sharing

Observação: a documentação técnica no Google Drive contém arquivos JSON relacionados à automação.

---

## 13. Como pausar ou desativar

Para pausar a automação:

1. Acessar o n8n.
2. Abrir o workflow **Envio de Mensagens - Movimentações Internas (Slack)**.
3. Desativar o botão **Active/Ativo**.

Se for necessário impedir apenas o envio público, remover temporariamente o canal `#ca-lideres` do nó de envio final ou trocar por um destino de teste.

---

## 14. Manutenção

### Quem deve revisar

- Matheus Guesser
- Time de Remuneração / Comp & People Analytics

### Quando revisar

Revisar a automação:

- sempre que o processo de movimentações internas mudar;
- sempre que o texto da mensagem mudar;
- sempre que o canal de destino mudar;
- sempre que a lógica de aprovação mudar;
- sempre que a planilha de dicas/log mudar;
- a cada nova versão publicada.

---

## 15. Histórico de mudanças

| Data | Alteração | Responsável |
|---|---|---|
| 13/08/2026 | Criação da documentação inicial da automação de lembrete de movimentações internas no Slack, com aprovação humana antes do envio ao canal `#ca-lideres`. | Matheus Guesser |

---

## 16. Observações

Automação em produção para publicar lembrete mensal no canal `#ca-lideres` após aprovação humana de Matheus.

A documentação foi criada a partir do JSON do n8n, da planilha de log e da conversa com Claude.
