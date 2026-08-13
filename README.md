# Envio de Mensagens - Movimentações Internas (Slack)

Automação em n8n para enviar mensalmente uma mensagem no Slack ao canal `#ca-lideres`, lembrando a liderança sobre o prazo de solicitação de movimentações internas.

Antes de publicar no canal, o fluxo envia uma prévia para aprovação de Matheus Guesser no Slack. A mensagem final só é publicada se houver aprovação humana.

## Status

- **Versão:** 1.0
- **Status:** Ativa
- **Ambiente:** Produção
- **Canal de destino:** `#ca-lideres`
- **Aprovador oficial:** Matheus Guesser
- **Última revisão:** 13/08/2026
- **Ritmo de atualização:** A cada nova versão

## Visão geral

A automação:

- roda mensalmente por agendamento no n8n;
- busca a dica mensal em uma planilha/workflow auxiliar;
- monta a mensagem de lembrete sobre movimentações internas;
- envia uma prévia para aprovação no Slack;
- publica no canal `#ca-lideres` apenas após aprovação;
- registra logs de envio aprovado ou recusado em planilha;
- evita que timeout seja tratado como reprovação automática.

## Workflow n8n

Workflow principal:

```text
Envio de Mensagens - Movimentações Internas (Slack)
```

Nós identificados no JSON:

1. `Gatilho Mensal`
2. `HTTP Request1`
3. `Elaborar mensagem1`
4. `Mensagem de Aprovação1`
5. `Switch1`
6. `Envio Mensagem1`
7. `Log de Envio1`
8. `Log - Recusado1`
9. `Aviso - Recusado1`
10. `Google Sheets - Ler Dica1`

## Como funciona

1. O workflow inicia pelo `Gatilho Mensal`.
2. O fluxo busca a dica mensal pelo nó `Google Sheets - Ler Dica1`.
3. O nó `Elaborar mensagem1` monta a mensagem final.
4. O nó `Mensagem de Aprovação1` envia uma prévia para aprovação de Matheus Guesser.
5. O nó `Switch1` avalia a resposta de aprovação.
6. Se aprovado, o nó `Envio Mensagem1` publica a mensagem no canal `#ca-lideres`.
7. O nó `Log de Envio1` registra o envio aprovado.
8. Se recusado, o nó `Log - Recusado1` registra a recusa.
9. O nó `Aviso - Recusado1` envia um aviso de recusa no Slack.
10. O fluxo deve tratar timeout separadamente, sem interpretar ausência de resposta como reprovação.

## Regras principais

- A mensagem final só pode ser publicada no canal após aprovação humana.
- O aprovador oficial é Matheus Guesser.
- O canal de destino em produção é `#ca-lideres`.
- A dica muda mensalmente e vem de uma base/planilha de dicas.
- O fluxo deve registrar log para envio aprovado e envio recusado.
- Timeout não deve ser tratado como reprovação automática.
- A regra de reprovação no `Switch` deve usar comparação estrita:

```js
{{ $json.data?.approved === false }}
```

com operador `is true`.

- A regra de aprovação pode usar comparação estrita:

```js
{{ $json.data?.approved === true }}
```

com operador `is true`.

## Planilha de log

Arquivo analisado:

```text
Automação - Envio Slack liderança.xlsx
```

A planilha contém a aba `Log`, com registros como:

- data/hora do disparo;
- mês de referência;
- dica utilizada;
- status do envio.

Status identificados nos registros de teste:

- `Envio Recusado`
- `Enviado com sucesso`

## Como testar

1. Executar o workflow manualmente no n8n.
2. Conferir se a dica mensal foi retornada corretamente.
3. Conferir se a mensagem de aprovação chegou para Matheus no Slack.
4. Aprovar o envio e validar se a mensagem foi publicada no canal correto.
5. Reexecutar recusando o envio e validar se a mensagem não foi publicada no canal.
6. Validar se o log de envio aprovado foi registrado na planilha.
7. Validar se o log de envio recusado foi registrado na planilha.
8. Testar cenário de timeout e confirmar que ele não cai no ramo `Reprovado`.
9. Conferir se o canal de destino está como `#ca-lideres`, e não como DM de teste.
10. Conferir se o workflow está ativo em produção.

## Como pausar

No n8n, abrir o workflow:

```text
Envio de Mensagens - Movimentações Internas (Slack)
```

E desativar o botão `Active/Ativo`.

Se for necessário impedir apenas o envio público, trocar temporariamente o canal `#ca-lideres` por um destino de teste.

## Segurança

Não versionar no GitHub:

- tokens;
- chaves;
- segredos;
- credenciais;
- URLs sensíveis de webhook;
- IDs internos que não devam ser expostos;
- dados pessoais desnecessários.

As credenciais de Slack e Google Sheets devem ficar apenas nas conexões seguras do n8n.

Antes de publicar o JSON no GitHub, revisar e sanitizar o arquivo.

## Manutenção

Responsável técnico e de negócio:

- Matheus Guesser

Área responsável:

- People — Remuneração / Comp & People Analytics

Revisar sempre que:

- o processo de movimentações internas mudar;
- o texto da mensagem mudar;
- o canal de destino mudar;
- a lógica de aprovação mudar;
- a planilha de dicas/log mudar;
- houver nova versão publicada.


## Documentação no Notion

Página criada na base `Automações e IA — People`:

```text
Envio de Mensagens - Movimentações Internas (Slack)
```

## Histórico de versões

| Versão | Data | Alteração |
|---|---|---|
| 1.0 | 13/08/2026 | Criação da documentação inicial da automação de lembrete no Slack, com aprovação humana antes do envio ao canal `#ca-lideres` |
