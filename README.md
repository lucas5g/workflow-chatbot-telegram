# Workflow Chatbot Telegram
Workflow do n8n que recebe mensagens de texto pelo Telegram, consulta a temperatura de cidades do Brasil com o OpenWeatherMap e responde em português. O agente usa um modelo conectado pelo OpenRouter e deve manter as respostas cordiais e limitadas ao clima de cidades brasileiras.

## Demonstração

Veja o teste do bot no Telegram: a mensagem com uma cidade brasileira, a consulta da temperatura e a resposta cordial do agente.

![Demonstração do teste do bot de clima no Telegram](demo.gif)

## Nós do workflow

O workflow tem **7 nós no total**: 5 no fluxo principal e 2 conectados ao agente como recursos de IA.

### Fluxo principal (5 nós)

1. **Telegram Trigger** recebe atualizações do tipo `message`.
2. **Capturar mensagem** guarda o texto recebido e o ID da conversa.
3. **Mensagem de texto?** encaminha mensagens de texto ao agente.
4. **Agente de clima** valida o pedido e gera a resposta.
5. **Enviar resposta** publica a saída do agente na mesma conversa do Telegram.

### Recursos de IA (2 nós)

- **OpenRouter principal** fornece o modelo de linguagem ao agente. O identificador do modelo deve ser conferido no campo `model` desse nó no n8n; essa configuração pode mudar independentemente deste README.
- **OpenWeatherMap** é a ferramenta meteorológica conectada ao agente e configurada para português brasileiro.

O agente deve pedir cidade e UF quando a localização não estiver clara. Para cidade/UF inválida ou não encontrada, a resposta configurada é:

> ❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).

Mensagens sobre outros assuntos devem receber uma explicação breve e cordial de que o bot atende somente perguntas sobre clima de cidades brasileiras.

## Requisitos

- Uma instância do **n8n** com os nós Telegram, OpenRouter, OpenWeatherMap e AI Agent disponíveis.
- Um bot criado no Telegram por meio do [@BotFather](https://t.me/BotFather).
- Uma chave de API do [OpenRouter](https://openrouter.ai/).
- Uma chave de API do [OpenWeatherMap](https://openweathermap.org/api).
- A instância n8n acessível pela internet para que o Telegram possa chamar o webhook do trigger.

## Configuração no n8n

1. Importe ou abra o workflow no n8n.
2. Crie uma credencial **Telegram API** com o token obtido no BotFather. Selecione essa credencial nos nós **Telegram Trigger** e **Enviar resposta**.
3. Crie uma credencial **OpenRouter API** com sua chave da API e selecione-a no nó **OpenRouter principal**. Confira o campo `model` do nó e escolha um identificador de modelo disponível na sua conta OpenRouter.
4. Crie uma credencial **OpenWeatherMap API** com sua chave e selecione-a no nó **OpenWeatherMap**. Confira se a conta e o plano permitem as chamadas necessárias.
5. Confirme as conexões: Telegram Trigger → Capturar mensagem → Mensagem de texto? → Agente de clima → Enviar resposta. O modelo e a ferramenta meteorológica devem estar conectados às entradas de IA do agente.
6. Revise as instruções do nó **Agente de clima** e mantenha o escopo restrito a clima de cidades brasileiras, a validação de localização e as respostas amigáveis para entradas inválidas.
7. Salve e ative/publique o workflow. O Telegram Trigger registra o webhook de produção do bot; a URL pública do n8n precisa estar acessível por HTTPS.
8. Envie uma mensagem ao bot para validar a integração.

### Teste sugerido

- `São Paulo,SP,BR` — deve consultar e responder a temperatura atual.
- `Curitiba, Paraná` — deve reconhecer cidade e estado, se a localização for identificada sem ambiguidade.
- Uma cidade inexistente ou estado incompatível — deve responder com a mensagem de cidade não encontrada mostrada acima.
- Uma pergunta sobre assunto não relacionado ao clima — deve explicar cordialmente o escopo do bot.

## Variáveis e segredos

As chaves de Telegram, OpenRouter e OpenWeatherMap devem ser armazenadas nas **Credentials** do n8n, não no prompt, em expressões, no README ou em arquivos versionados. Não reutilize tokens expostos em arquivos de configuração; se alguma chave real tiver sido compartilhada ou versionada, revogue-a e gere outra.

## Observações

- O trigger está configurado para atualizações de mensagens; o fluxo processa texto. Mensagens sem texto não são encaminhadas ao agente.
- Não há memória entre mensagens: cada solicitação deve informar a localização necessária.
- A resposta de clima depende da disponibilidade e dos limites de uso das APIs externas.
- Para que a validação de cidade/UF descrita nas instruções do agente funcione como previsto, confirme no editor do n8n que todas as ferramentas de geocodificação citadas no prompt estão conectadas ao agente. A consulta meteorológica usa o nó OpenWeatherMap.
