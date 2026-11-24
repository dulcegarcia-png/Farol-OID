# Dependências e itens fixos

## 1. Resumo

| Tipo | Situação |
|---|---|
| Bibliotecas de código (npm, CDN de scripts, frameworks) | **Nenhuma.** O JavaScript é puro, sem `import`, `require` nem `<script src>`. |
| Chamadas de rede feitas pelo código | **Nenhuma direta.** Não há `fetch`, `XMLHttpRequest` nem `WebSocket`. Tudo passa pelo `window.claude` do claude.ai. |
| Recurso externo carregado pelo navegador | **Google Fonts** (fontes). Há fontes de sistema como reserva. |
| Ambiente de execução | **claude.ai** (artifact publicado), com quatro capabilities e dois conectores. |
| Ferramentas de desenvolvimento | **Node.js 18 ou mais novo**, só para `build.js` e o teste. Nenhum pacote para instalar. |

## 2. Para rodar (usuário final)

### Navegador
Qualquer navegador atual (Chrome, Edge, Firefox, Safari de 2020 em diante). O código usa `async/await`, `Promise.all`, `matchMedia`, `insertAdjacentHTML` e `toLocaleString("pt-BR")`. No CSS usa `env(safe-area-inset-*)`, `min()`, `calc()`, `color-scheme`, `prefers-color-scheme`, `prefers-reduced-motion` e `:focus-visible`.

### Fontes (Google Fonts)
| Família | Pesos | Uso |
|---|---|---|
| Public Sans | 400, 500, 600, 700 | Interface |
| Newsreader | 400, 500 e 400 itálico | Texto das respostas |

Carregadas por `fonts.googleapis.com` e `fonts.gstatic.com`. Sem acesso a esses endereços, a página usa as fontes de sistema e continua funcionando. As duas famílias são distribuídas sob a licença SIL Open Font License (confira os termos no Google Fonts).

### Ambiente: claude.ai
A versão atual roda como artifact publicado, com o contrato de runtime `0.2.99` no momento da entrega.

| Capability | Obrigatória para | Sem ela |
|---|---|---|
| `db` | Dados compartilhados, Atualizar, planos | Mostra só os 10 casos embutidos. |
| `mcp` | Botão Atualizar (ler Slack e Jira) | Atualizar fica desativado. |
| `sample` | Resumo dos casos, planos e assistente | Assistente escondido; Atualizar e geração de planos desativados. |
| `user` | Saber se a pessoa pode gravar | A página tenta gravar e avisa se o acesso for só de leitura. |

### Conectores do claude.ai (na conta de quem atualiza)
| Nome de exibição exato | Ferramentas usadas |
|---|---|
| `Slack` | `slack_read_channel`, `slack_read_thread` |
| `Atlassian` | `getJiraIssue` |

Se a organização tiver dois conectores com o mesmo nome, o claude.ai pede para escolher qual usar. Se o nome for diferente, ajuste `SLACK_SRV` e `JIRA_SRV` em `src/app.js` e o `manifest/capabilities.json`.

### Acessos
- Canal privado **#operational-intelligence-desk**: quem atualiza precisa ser membro.
- Jira **nubank.atlassian.net**: quem atualiza precisa de acesso aos tickets JOI.
- Compartilhamento do artifact: **Colaborador** para atualizar; qualquer nível para consultar.

## 3. Para desenvolver

| Item | Versão | Uso |
|---|---|---|
| Node.js | 18 ou mais novo | `node build.js` (monta a página única) e `node tests/smoke-test.js` |
| npm | Não é necessário | Não há pacotes; o `package.json` só guarda os atalhos `build`, `check` e `test`. |

Comandos:

```bash
node build.js           # gera farol-oid.html a partir de src/
node build.js --check   # confere se farol-oid.html está em dia com src/
node tests/smoke-test.js
```

## 4. Valores fixos no código

Ficam no início do bloco "Atualizar" em `src/app.js` (e dois no começo do arquivo). Troque ao levar para outro canal, outro Jira ou outro Slack.

| Constante | Valor atual | O que controla |
|---|---|---|
| `CHANNEL` | `C0A0EC24DJ6` | Canal lido (#operational-intelligence-desk). |
| `CLOUD` | `nubank.atlassian.net` | Site do Jira usado nas consultas. |
| `JIRA` | `https://nubank.atlassian.net/browse/` | Base dos links dos tickets. |
| `SLACK` | `https://nubank.slack.com/archives/C0A0EC24DJ6/p` | Base dos links das threads. |
| `SLACK_SRV` e `JIRA_SRV` | `Slack` e `Atlassian` | Nomes dos conectores. |
| `MAX_PER_CLICK` | `40` | Casos lidos por clique em Atualizar. |
| `TEAM_LIST` | `OOS`, `Simba`, `LW Suits` | Times aceitos. |
| `ASK_IDEAS` | 7 perguntas | Perguntas sugeridas do assistente. |
| tempo da trava | `300000` ms (5 min) | Validade de `meta/lock`, renovada a cada caso. |
| limite de mensagens por página | `100`, até `40` páginas | Leitura do canal. |
| mensagens por thread | `200` | Leitura de cada thread. |
| `BASE_CASES` e `BASE_PLANS` | 10 casos e planos | Dados embutidos. Podem ser removidos (veja `INTEGRACAO-HUB.md`). |

## 5. Dependências de formato (o que quebra se mudar)

- **Texto do conector Slack.** `parseChannel()` lê blocos `=== Message from`, `Message TS:`, o link `browse/JOI-nnn|título`, `Prazo: aaaa-mm-dd` e `Thread: n replies`, e a paginação por `cursor: \`...\`` em `pagination_info`.
- **Mensagens do bot do Jira no canal.** O tipo do caso vem dos emojis e grupos da mensagem: `:reiteracao-oid:`, `:important_flag:`, `:suporte-judefense-legal-oid:`, o grupo `@lawsuits-oid`, a frase "fraude no SISBAJUD" e `:agora-alinhamentos:` (ignorado).
- **Mensagem de encerramento.** O prompt procura a mensagem "OID Finalizado" na thread para montar a aba Resposta enviada.
- **Resposta do Jira.** `jiraBrief()` espera o JSON do `getJiraIssue` com `fields` e `comment.comments`.
- **Saída do Claude.** Os prompts pedem JSON. `cleanCase`, `cleanPlan` e `cleanPlanDoc` descartam o que não se encaixa.

Se o time mudar o texto do bot, os emojis ou o formato do encerramento, ajuste `parseChannel()` e os prompts.

## 6. Inventário de arquivos de código

| Arquivo | Linhas aprox. | Conteúdo |
|---|---|---|
| `src/index.html` | 190 | Estrutura da página (topo, abas, painel, assistente, mascote em SVG). |
| `src/styles.css` | 330 | Estilos e variáveis de cor (claro e escuro). |
| `src/app.js` | 1.500 | Toda a lógica. |
| `build.js` | 40 | Junta os três em `farol-oid.html`. |
| `farol-oid.html` | 1.900 | Resultado do build. É o arquivo publicado e hospedável. |
| `tests/smoke-test.js` | 90 | Teste rápido, sem rede. |
