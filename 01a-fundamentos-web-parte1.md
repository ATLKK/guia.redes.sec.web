> *(Início)* | [🏠 **Voltar ao Sumário Principal**](../README.md) | [1B. Fundamentos Web (1.7 a 1.13) ➡️](./01b-fundamentos-web-parte2.md)

---

<a id="s1"></a>
## 1. Fundamentos de Desenvolvimento Web

Esta seção constrói a base conceitual de todo o restante da apostila. Antes de auditar uma aplicação com o Chrome DevTools ([seção 2](./02a-devtools-network.md#s2)), explorar vulnerabilidades ([seção 3](./03a-vulnerabilidades-parte1.md#s3)) ou configurar headers de segurança ([seção 5](./05-headers-de-seguranca.md#s5)), é preciso entender **como a Web funciona por baixo dos panos**: quem conversa com quem, em que ordem, usando quais protocolos e com quais mensagens.

Para cada conceito, a apostila segue um roteiro fixo, pensado para a prova e para a prática profissional:

| Bloco | Pergunta que responde |
|---|---|
| **O que é** | Definição técnica precisa |
| **Para que serve** | Problema que o conceito resolve |
| **Como funciona** | Mecanismo interno, passo a passo |
| **Exemplo prático** | Mensagens, comandos, código ou cenário real |
| **Possíveis problemas de segurança** | O que dá errado quando mal usado ou mal configurado |
| **Configuração segura** | O que fazer, classificado em **[REQUISITO]**, **[RECOMENDAÇÃO]** ou **[BOA PRÁTICA]** |

> [!NOTE]
> Esta parte cobre os itens 1.1 a 1.6 (arquitetura, navegação, protocolos, métodos, códigos de status e headers). A visão de redes aprofundada (camadas OSI/TCP-IP, roteamento, portas, ferramentas) está na [seção 4 — Redes](./04-redes-de-computadores.md#s4), e a análise prática dessas mensagens no navegador está na [aba Network do DevTools](./02a-devtools-network.md#s2-1).

<a id="s1-1"></a>
### 1.1 Arquitetura cliente-servidor

<a id="s1-1-1"></a>
#### 1.1.1 Conceito e papéis

**O que é:** a arquitetura **cliente-servidor** é um modelo de comunicação em que um processo (o **cliente**) toma a iniciativa de enviar **requisições** e outro processo (o **servidor**) fica aguardando conexões, processa as requisições e devolve **respostas**. Na Web, o cliente típico é o **navegador** (Chrome, Firefox, Edge, Safari), mas também podem ser aplicativos móveis, scripts (`curl`, `python requests`), outros servidores (comunicação servidor-a-servidor) ou robôs de busca.

**Para que serve:** centralizar dados e regras de negócio em um ponto controlado (o servidor), permitindo que muitos clientes heterogêneos acessem o mesmo serviço. Isso facilita atualização (muda-se o servidor, não milhões de clientes), controle de acesso e consistência de dados.

**Como funciona:**

1. O servidor fica **escutando** (*listening*) em um endereço IP e uma porta (ex.: `0.0.0.0:443`).
2. O cliente descobre o endereço do servidor (normalmente via DNS — veja [1.3.1](#s1-3-1)) e abre uma conexão (TCP ou QUIC).
3. O cliente envia uma requisição em um protocolo de aplicação (HTTP).
4. O servidor processa (lê arquivos, consulta banco de dados, chama outros serviços) e devolve uma resposta.
5. A conexão pode ser reutilizada para novas requisições (*keep-alive*, multiplexação HTTP/2 e HTTP/3) ou encerrada.

| Papel | Responsabilidades | Exemplos |
|---|---|---|
| **Cliente** | Iniciar requisições, renderizar a interface, coletar entradas do usuário, armazenar estado local (cookies, `localStorage`) | Navegador, app móvel, `curl`, Postman |
| **Servidor** | Aguardar conexões, autenticar, autorizar, aplicar regras de negócio, persistir dados, gerar respostas | Nginx, Apache, Node.js/Express, Django, Spring Boot, IIS |

> [!IMPORTANT]
> Princípio fundamental de segurança: **o cliente está sob controle total do usuário (e, portanto, de um atacante)**. Tudo o que roda no navegador — HTML, JavaScript, validações de formulário, campos ocultos, cookies — pode ser lido e alterado com o próprio DevTools. Por isso, **toda validação e toda decisão de autorização precisa ser feita no servidor**.

<a id="s1-1-2"></a>
#### 1.1.2 Front-end e back-end

**O que é:**

- **Front-end** é a parte da aplicação executada no cliente: **HTML** (HyperText Markup Language — estrutura), **CSS** (Cascading Style Sheets — apresentação) e **JavaScript** (comportamento). Frameworks comuns: React, Angular, Vue, Svelte.
- **Back-end** é a parte executada no servidor: lógica de negócio, acesso a banco de dados, autenticação, integrações. Linguagens comuns: JavaScript/TypeScript (Node.js), Python, Java, C#, PHP, Go, Ruby.

**Para que serve:** separar responsabilidades — interface e experiência do usuário de um lado; dados, regras e segurança do outro.

**Como funciona:** o front-end se comunica com o back-end por requisições HTTP. Em aplicações tradicionais (**MPA** — Multi-Page Application), o servidor devolve páginas HTML completas a cada navegação. Em **SPA** (Single-Page Application), o servidor entrega uma "casca" HTML e um pacote JavaScript; depois, o JS chama uma **API** (Application Programming Interface) — geralmente REST (Representational State Transfer) ou GraphQL — que devolve dados em **JSON** (JavaScript Object Notation).

**Exemplo prático (SPA consumindo API):**

```javascript
// Front-end (executa no navegador do usuário)
const resp = await fetch("https://api.exemplo.test/v1/pedidos?status=aberto", {
  credentials: "include",           // envia cookies de sessão
  headers: { "Accept": "application/json" }
});
const pedidos = await resp.json();
```

```python
# Back-end (executa no servidor) — exemplo com Flask
@app.get("/v1/pedidos")
@login_required
def listar_pedidos():
    status = request.args.get("status", "aberto")
    # Autorização no servidor: só os pedidos do usuário autenticado
    pedidos = Pedido.query.filter_by(usuario_id=current_user.id, status=status).all()
    return jsonify([p.to_dict() for p in pedidos])
```

**Possíveis problemas de segurança:**

- **Validação só no front-end** (ex.: `maxlength`, `required`, regex em JS): o atacante a ignora enviando a requisição diretamente.
- **Segredos embutidos no front-end** (chaves de API, tokens de serviço em arquivos `.js` ou *source maps*): qualquer um vê no painel *Sources* do DevTools.
- **Controle de acesso por ocultação de botões**: esconder o botão "Excluir" não impede que a requisição `DELETE` seja enviada manualmente (falha de *Broken Access Control*, veja [seção 3](./03a-vulnerabilidades-parte1.md#s3)).
- **Excesso de dados na API** (*excessive data exposure*): a API devolve o objeto inteiro (com hash de senha, CPF etc.) e o front-end "só mostra" alguns campos — mas a resposta completa aparece na aba *Network*.

**Configuração segura:**

- **[REQUISITO]** Validar e autorizar toda entrada no back-end, independentemente das validações do front-end.
- **[REQUISITO]** Nunca colocar segredos de servidor (chaves privadas, senhas de banco, tokens administrativos) em código entregue ao cliente.
- **[RECOMENDAÇÃO]** Retornar apenas os campos necessários em cada endpoint (DTOs — Data Transfer Objects — ou *serializers* explícitos).
- **[BOA PRÁTICA]** Não publicar *source maps* em produção, ou restringir o acesso a eles.

<a id="s1-1-3"></a>
#### 1.1.3 Camadas e modelos de 2 e 3 camadas

**O que é:** uma **camada** (*tier*, quando se fala de separação física; *layer*, quando lógica) é um agrupamento de responsabilidades. Os modelos clássicos são:

| Modelo | Componentes | Características | Exemplo |
|---|---|---|---|
| **2 camadas** (*two-tier*) | Cliente ↔ Servidor de banco de dados | O cliente contém a interface **e** a lógica de negócio; conversa direto com o banco | Aplicação desktop antiga que conecta direto ao SQL Server |
| **3 camadas** (*three-tier*) | Apresentação ↔ Aplicação (lógica) ↔ Dados | Lógica centralizada no servidor de aplicação; banco não é exposto ao cliente | Navegador ↔ API Node.js ↔ PostgreSQL |
| **N camadas** / microsserviços | Apresentação ↔ *gateway* ↔ vários serviços ↔ vários bancos, filas, caches | Escalabilidade e implantação independentes, maior complexidade | Loja com serviços de catálogo, carrinho, pagamento |

**Para que serve:** isolar responsabilidades, facilitar manutenção e escalabilidade e — do ponto de vista de segurança — **reduzir a superfície de ataque**, já que o banco de dados não precisa ficar acessível pela Internet.

**Como funciona (3 camadas):**

1. **Camada de apresentação:** navegador renderiza a interface.
2. **Camada de aplicação:** servidor recebe HTTP, valida, autentica, autoriza e aplica regras.
3. **Camada de dados:** SGBD (Sistema Gerenciador de Banco de Dados), acessível apenas pela camada de aplicação, em rede interna.

**Possíveis problemas de segurança:**

- No modelo de 2 camadas, as **credenciais do banco ficam no cliente** — qualquer usuário pode extraí-las e acessar o banco diretamente.
- Banco de dados exposto à Internet (porta 3306/MySQL, 5432/PostgreSQL, 27017/MongoDB) é causa recorrente de vazamentos massivos.
- Comunicação interna sem TLS (Transport Layer Security) e sem autenticação entre serviços ("a rede interna é confiável") facilita movimentação lateral após uma invasão.

**Configuração segura:**

- **[REQUISITO]** Banco de dados acessível apenas pela rede interna/segmentada, nunca diretamente pela Internet.
- **[RECOMENDAÇÃO]** Usuário de banco da aplicação com **privilégio mínimo** (sem `DROP`, sem acesso a outros schemas).
- **[RECOMENDAÇÃO]** TLS e autenticação também entre serviços internos (modelo *Zero Trust*, mTLS — mutual TLS — quando aplicável).
- **[BOA PRÁTICA]** Segmentar a rede em zonas (DMZ — zona desmilitarizada —, aplicação, dados) com regras de firewall explícitas.

<a id="s1-1-4"></a>
#### 1.1.4 HTTP é stateless: como se mantém estado

**O que é:** o HTTP é um protocolo **sem estado** (*stateless*): cada requisição é independente e, por definição do protocolo, o servidor não precisa lembrar nada da requisição anterior (RFC 9110, *HTTP Semantics*).

**Para que serve:** simplicidade e escalabilidade — qualquer servidor de um *cluster* pode atender qualquer requisição, pois ela carrega tudo o que é necessário.

**Como funciona:** como aplicações reais precisam de estado (usuário logado, carrinho), o estado é **transportado** em cada requisição por mecanismos como:

| Mecanismo | Onde fica o estado | Exemplo |
|---|---|---|
| **Cookie de sessão** | ID aleatório no cliente; dados da sessão no servidor | `Cookie: SESSIONID=8f3a...` |
| **Token autocontido** (ex.: JWT — JSON Web Token, RFC 7519) | Dados assinados no próprio token, no cliente | `Authorization: Bearer eyJhbGciOi...` |
| **Parâmetros na URL** | Na própria URL | `?sid=abc123` (**inseguro**: vaza em logs, histórico, `Referer`) |
| **Armazenamento no cliente** | `localStorage`, `sessionStorage`, IndexedDB | Preferências de interface |

**Exemplo prático:**

```http
POST /login HTTP/1.1
Host: loja-ficticia.test
Content-Type: application/x-www-form-urlencoded

usuario=ana&senha=S3nh%40Forte
```

```http
HTTP/1.1 302 Found
Location: /minha-conta
Set-Cookie: __Host-sessao=Zx9...k2; Path=/; Secure; HttpOnly; SameSite=Lax
```

A partir daí, o navegador anexa automaticamente `Cookie: __Host-sessao=Zx9...k2` em cada requisição ao mesmo site.

**Possíveis problemas de segurança:** sequestro de sessão (*session hijacking*) por roubo do cookie/token (via XSS — Cross-Site Scripting —, rede sem TLS, logs); fixação de sessão (*session fixation*); CSRF (Cross-Site Request Forgery), porque o navegador envia cookies automaticamente; IDs de sessão previsíveis.

**Configuração segura:**

- **[REQUISITO]** IDs de sessão gerados por gerador criptograficamente seguro, com entropia adequada, e transmitidos apenas via HTTPS.
- **[REQUISITO]** Regenerar o ID de sessão após o login.
- **[RECOMENDAÇÃO]** Cookies de sessão com `Secure`, `HttpOnly` e `SameSite` (detalhes em [Application](./02b-devtools-application.md#s2-2) e [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **[BOA PRÁTICA]** Nunca transportar identificadores de sessão na URL.

<a id="s1-1-5"></a>
#### 1.1.5 Intermediários: proxy reverso, CDN e balanceador de carga

**O que é:** entre o cliente e o servidor de aplicação (a **origem**) costuma haver intermediários:

| Componente | O que é | Funções típicas | Exemplos |
|---|---|---|---|
| **Proxy (direto / *forward proxy*)** | Intermediário do lado do **cliente** | Controle de acesso de saída, cache corporativo, inspeção | Squid, proxy corporativo |
| **Proxy reverso** (*reverse proxy*) | Intermediário do lado do **servidor**; o cliente acha que fala com a origem | Terminação TLS, roteamento por caminho/host, compressão, cache, WAF | Nginx, HAProxy, Traefik, Envoy |
| **CDN** (Content Delivery Network — rede de distribuição de conteúdo) | Rede global de servidores de borda (*edge*) que armazenam cópias do conteúdo | Reduzir latência, absorver tráfego/DDoS, cache de estáticos | Cloudflare, Akamai, Fastly, CloudFront |
| **Balanceador de carga** (*load balancer*) | Distribui requisições entre várias instâncias | Alta disponibilidade, *health checks*, escalabilidade horizontal | AWS ELB/ALB, HAProxy, F5 |
| **WAF** (Web Application Firewall) | Filtro de camada 7 que inspeciona HTTP | Bloquear padrões de ataque (SQLi, XSS), *rate limiting* | ModSecurity, AWS WAF, Cloudflare WAF |

Balanceadores podem operar em **camada 4** (L4 — transporte: decide por IP/porta, não vê o HTTP) ou **camada 7** (L7 — aplicação: vê URL, headers e cookies, podendo rotear por caminho e manter "sessão grudada" — *sticky sessions*).

**Para que serve:** desempenho, escalabilidade, disponibilidade e uma camada adicional de defesa.

**Como funciona — diagrama ASCII de uma arquitetura típica:**

```text
                              INTERNET
                                 │
   ┌────────────┐   HTTPS/443    │
   │ Navegador  │ ───────────────┤
   │ (cliente)  │                ▼
   └────────────┘      ┌───────────────────┐
                       │  CDN / WAF (borda)│  cache de estáticos, filtro L7
                       └─────────┬─────────┘
                                 │ HTTPS
                       ┌─────────▼─────────┐
                       │ Balanceador (L7)  │  terminação TLS, health checks
                       └───┬───────────┬───┘
                 HTTP/HTTPS│           │
              ┌────────────▼──┐     ┌──▼────────────┐
  Rede de     │ App server #1 │     │ App server #2 │  lógica de negócio (stateless)
  aplicação   └──────┬────────┘     └───────┬───────┘
                     │   ┌──────────────┐   │
                     ├──►│ Cache (Redis)│◄──┤        sessões, cache de dados
                     │   └──────────────┘   │
  Rede de      ┌─────▼──────────────────────▼─────┐
  dados        │  Banco de dados (PostgreSQL)     │  sem acesso pela Internet
               └──────────────────────────────────┘
```

**Exemplo de fluxo:** o usuário acessa `https://loja-ficticia.test/produtos/42`.

1. O DNS resolve `loja-ficticia.test` para um IP da **CDN** (não da origem).
2. A CDN termina o TLS; se `/produtos/42` estiver em cache e ainda "fresco", responde direto (header como `Age: 120` e, em muitas CDNs, um header próprio de *cache hit*).
3. Se não estiver em cache, a CDN encaminha ao **balanceador**, que escolhe uma instância saudável.
4. A instância consulta Redis/PostgreSQL e devolve a resposta, que volta pela mesma cadeia e pode ser armazenada na CDN conforme `Cache-Control`.
5. Os intermediários costumam acrescentar headers como `Via`, `X-Forwarded-For`, `X-Forwarded-Proto` e `Forwarded` (RFC 7239).

**Possíveis problemas de segurança:**

- **Confiança indevida em `X-Forwarded-For`**: a aplicação usa o IP desse header para *rate limiting* ou listas de permissão, mas o cliente pode forjá-lo (veja [1.6.2](#s1-6-2)).
- **Bypass da CDN/WAF**: se o IP da origem for descoberto (registros DNS antigos, cabeçalhos, certificados), o atacante fala direto com a origem, sem WAF.
- **Cache poisoning / cache deception**: a CDN armazena uma resposta manipulada ou com dados privados e a serve para outros usuários.
- **HTTP Request Smuggling**: front-end e back-end interpretam de modo diferente o tamanho da mensagem (`Content-Length` vs `Transfer-Encoding`) — veja [1.6.3](#s1-6-3).
- **Tráfego interno em texto claro** após a terminação TLS.

**Configuração segura:**

- **[REQUISITO]** Se a aplicação usa IP de cliente, confiar em `X-Forwarded-For`/`Forwarded` **apenas** quando vier de proxies conhecidos, configurando explicitamente a lista de proxies confiáveis.
- **[RECOMENDAÇÃO]** Restringir a origem para aceitar conexões apenas dos IPs da CDN/balanceador (ou por túnel/mTLS).
- **[RECOMENDAÇÃO]** Definir `Cache-Control: private` ou `no-store` em respostas personalizadas/sensíveis.
- **[BOA PRÁTICA]** TLS de ponta a ponta (cliente→borda e borda→origem) quando houver dados sensíveis.

Exemplo de Nginx como proxy reverso com encaminhamento correto dos headers:

```nginx
server {
    listen 443 ssl;
    http2 on;                         # sintaxe de versões recentes do Nginx; versões antigas usam "listen 443 ssl http2;"
    server_name loja-ficticia.test;

    ssl_certificate     /etc/ssl/loja/fullchain.pem;
    ssl_certificate_key /etc/ssl/loja/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;

    server_tokens off;                # não divulgar versão do Nginx

    location / {
        proxy_pass http://app_backend;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        # Substitui (não concatena) o valor enviado pelo cliente:
        proxy_set_header X-Forwarded-For   $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> [!TIP]
> No DevTools, a aba *Network* → painel *Headers* → seção *General* mostra o **Remote Address** (IP:porta com que o navegador realmente se conectou). Se aparecer um IP de CDN, você está falando com a borda, não com a origem.

<a id="s1-2"></a>
### 1.2 O que acontece quando acessamos uma URL

Esta é uma das perguntas mais clássicas de prova e de entrevistas técnicas. A resposta completa envolve praticamente todas as camadas da Web.

<a id="s1-2-1"></a>
#### 1.2.1 Anatomia da URL

**O que é:** uma **URL** (Uniform Resource Locator) identifica um recurso e indica como acessá-lo. É um tipo de **URI** (Uniform Resource Identifier), definido na RFC 3986; os navegadores seguem o **URL Standard** do WHATWG (Web Hypertext Application Technology Working Group), que define o algoritmo de *parsing* realmente usado.

**Para que serve:** permitir que o navegador saiba **qual protocolo usar**, **com quem se conectar**, **em qual porta**, **qual recurso pedir** e **qual parte da página exibir**.

**Como funciona — decomposição:**

```text
  https://ana:segredo@www.loja-ficticia.test:8443/produtos/busca?q=tenis&page=2#avaliacoes
  └─┬──┘   └────┬────┘ └─────────┬─────────┘ └┬─┘└──────┬──────┘└───────┬──────┘└────┬────┘
 esquema  credenciais           host        porta   caminho          query      fragmento
          (userinfo)                                 (path)        (consulta)  (fragment)
          └─────────────────── autoridade (authority) ──────┘
```

| Parte | Descrição | Vai ao servidor? | Observações de segurança |
|---|---|---|---|
| **Esquema** (*scheme*) | Protocolo: `https`, `http`, `ws`, `wss`, `ftp`, `file`, `data`, `javascript`, `mailto`... | Determina o protocolo | Esquemas como `javascript:` e `data:` são vetores de XSS quando URLs vindas do usuário são aceitas em `href`/`src` sem validação |
| **Credenciais** (*userinfo*: `usuario:senha@`) | Credenciais embutidas | Em HTTP, historicamente convertidas em autenticação Basic | **[Obsoleto]** A RFC 3986 declara obsoleto o formato `user:password`. Navegadores modernos bloqueiam ou restringem seu uso (ex.: bloqueio em sub-recursos) **[Varia por versão]**. Usado em *phishing*: `https://banco.example.com@203.0.113.66/` leva ao IP `203.0.113.66`, não ao banco |
| **Host** | Nome de domínio ou IP (`[2001:db8::1]` para IPv6) | Sim, no header `Host` (HTTP/1.1) ou `:authority` (HTTP/2 e 3), e no **SNI** do TLS | Ataques de homógrafo/IDN (Internationalized Domain Names): `exаmple.com` com "а" cirílico; navegadores exibem Punycode (`xn--...`) em casos suspeitos |
| **Porta** | Número da porta TCP/UDP. Padrão: 80 (`http`), 443 (`https`) | Implícita na conexão | Portas fazem parte da **origem** (esquema + host + porta) |
| **Caminho** (*path*) | Localização do recurso no servidor | Sim | *Path traversal* (`../../etc/passwd`), normalização inconsistente entre proxy e aplicação |
| **Query** (consulta) | Parâmetros `chave=valor` separados por `&`, após `?` | Sim | Fica em logs de servidor e proxy, histórico e pode vazar via `Referer`: **não colocar senhas, tokens ou dados pessoais** |
| **Fragmento** | Após `#`; aponta para parte do documento ou estado de SPA | **Não** — o navegador **não envia** o fragmento ao servidor | É lido por JavaScript (`location.hash`); fonte de **DOM-based XSS** quando inserido no DOM sem tratamento |

**Exemplo prático — inspecionando com a API `URL` no Console:**

```javascript
const u = new URL("https://www.loja-ficticia.test:8443/produtos/busca?q=t%C3%AAnis&page=2#avaliacoes");
u.protocol;                 // "https:"
u.hostname;                 // "www.loja-ficticia.test"
u.port;                     // "8443"
u.pathname;                 // "/produtos/busca"
u.searchParams.get("q");    // "tênis"  (já decodificado)
u.hash;                     // "#avaliacoes"
u.origin;                   // "https://www.loja-ficticia.test:8443"
```

**Codificação percentual (*percent-encoding*):** caracteres reservados ou não ASCII são codificados como `%XX` (bytes UTF-8 em hexadecimal). Ex.: espaço → `%20` (ou `+` em `application/x-www-form-urlencoded`), `ê` → `%C3%AA`, `/` → `%2F`. Use `encodeURIComponent()` para valores de parâmetros.

**Possíveis problemas de segurança:** *open redirect* (`/login?next=https://malicioso.example`), dados sensíveis em query, XSS via `javascript:` em links, *phishing* com `@` e homógrafos, dupla codificação (`%252e%252e%252f`) para burlar filtros de *path traversal*.

**Configuração segura:**

- **[REQUISITO]** Nunca transmitir senhas, tokens de sessão ou dados pessoais sensíveis na query string.
- **[REQUISITO]** Validar URLs fornecidas pelo usuário com uma lista de permissão de esquemas (`https:`, eventualmente `http:`) e, para redirecionamentos, de destinos.
- **[BOA PRÁTICA]** Usar parsers padronizados (`new URL()`, `urllib.parse`) em vez de regex caseiras, e normalizar antes de validar.

<a id="s1-2-2"></a>
#### 1.2.2 Passo a passo da navegação

Visão geral do fluxo quando o usuário digita `loja-ficticia.test/produtos` e pressiona Enter:

```mermaid
flowchart TD
    A["1. Parsing da entrada: URL ou busca?"] --> B["2. Verifica HSTS / lista HSTS preload"]
    B --> C["3. Verifica cache HTTP e Service Worker"]
    C --> D["4. Resolução DNS em cascata"]
    D --> E["5. Conexão TCP (ou QUIC/UDP)"]
    E --> F["6. Handshake TLS"]
    F --> G["7. Envio da requisição HTTP"]
    G --> H["8. Processamento no servidor e resposta"]
    H --> I["9. Parsing do HTML: DOM"]
    I --> J["10. CSS: CSSOM"]
    J --> K["11. Render tree, layout, paint, composite"]
    I --> L["12. JavaScript e sub-recursos"]
    L --> K
```

##### Passo 1 — Parsing da entrada

O navegador decide se o texto digitado é uma URL ou um termo de busca. `loja-ficticia.test/produtos` parece um host com caminho, então é tratado como URL. Sem esquema explícito, navegadores modernos tendem a tentar **HTTPS primeiro** e recorrer a HTTP se falhar **[Varia por versão]** (o Chrome passou a priorizar HTTPS na barra de endereços e oferece o modo "Sempre usar conexões seguras", que avisa antes de carregar sites em HTTP). Em seguida, a URL é normalizada: host convertido para minúsculas, IDN convertido para Punycode, caminho com `.`/`..` resolvido, porta padrão omitida.

##### Passo 2 — HSTS e HSTS preload

**HSTS** (HTTP Strict Transport Security, RFC 6797) é uma política enviada pelo servidor (`Strict-Transport-Security: max-age=31536000; includeSubDomains`) que instrui o navegador a **acessar aquele domínio somente via HTTPS** durante `max-age` segundos. Se o domínio já tem HSTS registrado — ou está na **lista HSTS preload**, embutida no código do navegador (inscrição via hstspreload.org) — o navegador converte `http://` em `https://` **internamente, antes de qualquer tráfego de rede**. No DevTools, isso aparece como um redirecionamento interno **307 Internal Redirect** com o header `Non-Authoritative-Reason: HSTS`.

> [!NOTE]
> A lista *preload* resolve o problema do "primeiro acesso" (*trust on first use*): sem ela, a primeira requisição HTTP de um usuário novo ainda poderia ser interceptada por um ataque de *SSL stripping* antes que o header HSTS fosse recebido.

##### Passo 3 — Caches e Service Worker

Antes de ir à rede, o navegador verifica:

- Se há um **Service Worker** registrado para o escopo, que pode interceptar a requisição e responder do Cache Storage.
- O **cache HTTP** (memória e disco): se a resposta estiver fresca (`Cache-Control: max-age`), é usada sem rede — no DevTools, coluna *Size* mostra `(memory cache)` ou `(disk cache)`. Se estiver vencida mas tiver validadores (`ETag`, `Last-Modified`), o navegador faz uma **requisição condicional** e pode receber `304 Not Modified`.

##### Passo 4 — Resolução DNS em cascata

O navegador precisa transformar `loja-ficticia.test` em um endereço IP. A consulta percorre uma cascata de caches, parando no primeiro que tiver a resposta válida (dentro do TTL — Time To Live):

```text
 Navegador ──► 1. Cache DNS do navegador        (Chrome: chrome://net-internals/#dns)
     │
     ▼
            2. Cache/resolvedor do sistema operacional + arquivo hosts
               (Windows: ipconfig /displaydns ; C:\Windows\System32\drivers\etc\hosts
                Linux: /etc/hosts, systemd-resolved ; macOS: mDNSResponder)
     │
     ▼
            3. Resolvedor recursivo (do provedor, da empresa ou público, ex.: 8.8.8.8, 1.1.1.1)
               └─ tem cache próprio; se não souber, faz a resolução iterativa:
     │
     ├──► 4. Servidor RAIZ (.)        → "não sei, mas o .test/.com é com estes NS"
     ├──► 5. Servidor do TLD (.com)   → "o domínio exemplo.com é com ns1.exemplo.com"
     └──► 6. Servidor AUTORITATIVO    → "www.exemplo.com A 203.0.113.10, TTL 300"
     │
     ▼
 Resposta volta ao SO e ao navegador, que armazenam em cache conforme o TTL
```

O cliente faz uma consulta **recursiva** ao resolvedor ("me dê a resposta final"); o resolvedor faz consultas **iterativas** à hierarquia ("me diga a quem perguntar"). O navegador costuma pedir registros **A** (IPv4) e **AAAA** (IPv6) e, em versões recentes, também **HTTPS/SVCB** (que podem anunciar suporte a HTTP/3 e ECH) **[Varia por versão]**. Detalhes em [1.3.1](#s1-3-1).

> [!NOTE]
> `.test` é um TLD reservado para testes (RFC 2606/RFC 6761) e não existe na raiz pública; o exemplo acima usa `exemplo.com` apenas para ilustrar a hierarquia real.

##### Passo 5 — Conexão de transporte (TCP ou QUIC)

- **HTTP/1.1 e HTTP/2** rodam sobre **TCP** (Transmission Control Protocol): *three-way handshake* `SYN` → `SYN-ACK` → `ACK` (1 RTT — Round-Trip Time, tempo de ida e volta).
- **HTTP/3** roda sobre **QUIC** (RFC 9000), que usa **UDP** (User Datagram Protocol) e integra o handshake de transporte com o TLS 1.3, normalmente em 1 RTT (ou 0-RTT na retomada). O navegador descobre o suporte a HTTP/3 pelo header `Alt-Svc` de uma resposta anterior ou pelo registro DNS HTTPS.
- Se já houver conexão aberta reutilizável para a mesma origem, esta etapa (e a próxima) é pulada.

##### Passo 6 — Handshake TLS

Com HTTPS, cliente e servidor negociam versão do TLS, suíte criptográfica e chaves de sessão; o servidor apresenta seu **certificado**, que o navegador valida (cadeia até uma AC — Autoridade Certificadora — raiz confiável, nome do host, validade, revogação). No TLS 1.3 (RFC 8446) isso custa 1 RTT. O **ALPN** (Application-Layer Protocol Negotiation) decide, dentro do handshake, se a conexão usará `h2` (HTTP/2) ou `http/1.1`. Detalhes em [1.3.6](#s1-3-6).

##### Passo 7 — Requisição HTTP

O navegador monta e envia a requisição, com método, caminho, headers (`Host`, `User-Agent`, `Accept`, `Accept-Language`, `Accept-Encoding`, `Cookie`, `Sec-Fetch-*`...) e eventual corpo:

```http
GET /produtos HTTP/1.1
Host: loja-ficticia.test
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/... Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: pt-BR,pt;q=0.9,en;q=0.8
Accept-Encoding: gzip, deflate, br, zstd
Cookie: __Host-sessao=Zx9...k2
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-Dest: document
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
```

##### Passo 8 — Processamento e resposta

O servidor (possivelmente atrás de CDN e balanceador) roteia, autentica, executa a lógica e devolve:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Encoding: br
Cache-Control: private, no-cache
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
Set-Cookie: __Host-pref=tema-escuro; Path=/; Secure; SameSite=Lax

<!DOCTYPE html>
<html lang="pt-BR"> ...
```

O tempo entre o envio da requisição e a chegada do primeiro byte da resposta é o **TTFB** (Time To First Byte — tempo até o primeiro byte), dominado pelo processamento no servidor e pela latência.

##### Passo 9 — Parsing do HTML e construção do DOM

O navegador decodifica os bytes (usando o `charset`), *tokeniza* o HTML e constrói o **DOM** (Document Object Model), uma árvore de nós que o JavaScript pode manipular. O parsing é **incremental**: começa antes de o documento terminar de chegar. Um **preload scanner** paralelo varre o HTML à frente para descobrir cedo `<link rel="stylesheet">`, `<script src>`, `<img>` e iniciar seus downloads.

##### Passo 10 — CSS e CSSOM

Folhas de estilo são baixadas e transformadas no **CSSOM** (CSS Object Model). O CSS é **bloqueante de renderização** (*render-blocking*): o navegador não pinta a página até ter o CSSOM, para evitar o "piscar" de conteúdo sem estilo.

##### Passo 11 — Render tree, layout, paint e composite

1. **Render tree (árvore de renderização):** combinação de DOM + CSSOM contendo apenas o que será exibido (elementos com `display: none` ficam de fora).
2. **Layout** (ou *reflow*): cálculo de posição e tamanho exatos de cada caixa na *viewport*.
3. **Paint:** preenchimento de pixels — textos, cores, bordas, sombras, imagens — em uma ou mais camadas.
4. **Composite:** as camadas são combinadas (geralmente com aceleração por GPU — Graphics Processing Unit) na ordem correta para formar a imagem final. Animações de `transform` e `opacity` podem ser feitas só nesta etapa, por isso são mais baratas.

##### Passo 12 — JavaScript e sub-recursos

- Um `<script>` clássico (sem `async`/`defer`) **bloqueia o parser**: o HTML para enquanto o script é baixado e executado (o script pode alterar o DOM com `document.write`).
- `defer`: baixa em paralelo e executa **na ordem**, após o parsing do HTML, antes do evento `DOMContentLoaded`.
- `async`: baixa em paralelo e executa **assim que chega**, sem ordem garantida.
- `type="module"`: comportamento *defer* por padrão.
- Sub-recursos (imagens, fontes, *iframes*, chamadas `fetch`) disparam novas requisições — cada uma repete, conforme necessário, os passos 3 a 8 (com reuso de conexões e de cache DNS).
- Eventos marcantes: **`DOMContentLoaded`** (DOM pronto, scripts *defer* executados) e **`load`** (todos os sub-recursos, como imagens, carregados). O DevTools exibe ambos na barra de status da aba *Network* e como linhas verticais (azul e vermelha, respectivamente) no *Waterfall*.

**Possíveis problemas de segurança ao longo do fluxo:**

| Etapa | Ameaça | Mitigação |
|---|---|---|
| Parsing / primeira requisição em HTTP | *SSL stripping* (rebaixar para HTTP) | **[REQUISITO]** Redirecionar HTTP→HTTPS e enviar HSTS; **[RECOMENDAÇÃO]** inscrever no *preload* |
| DNS | Envenenamento de cache, sequestro de resolvedor, *spoofing* em Wi-Fi público | **[RECOMENDAÇÃO]** DNSSEC no domínio; DoH/DoT no cliente |
| TCP | SYN flood (negação de serviço) | SYN cookies, proteção anti-DDoS na borda |
| TLS | Certificado inválido, protocolos antigos, MITM (Man-in-the-Middle) | **[REQUISITO]** TLS 1.2+ e certificado válido; nunca instruir usuários a ignorar avisos |
| HTTP | Vazamento de dados em URL, cookies sem `Secure` | Ver [1.6](#s1-6) e [seção 5](./05-headers-de-seguranca.md#s5) |
| Parsing HTML/JS | XSS, scripts de terceiros comprometidos | CSP (Content Security Policy), SRI (Subresource Integrity), codificação de saída |

<a id="s1-2-3"></a>
#### 1.2.3 Relação com a aba Network e o painel Timing

Cada etapa acima pode ser observada no DevTools ([aba Network](./02a-devtools-network.md#s2-1)). Ao clicar em uma requisição e abrir a aba **Timing**, o Chrome detalha as fases (os rótulos exatos podem mudar **[Varia por versão]**):

| Fase no Timing | Etapa correspondente | O que indica quando está alta |
|---|---|---|
| **Queueing** | Requisição aguardando na fila do navegador | Prioridade baixa, limite de ~6 conexões por origem em HTTP/1.1 |
| **Stalled** | Requisição pronta, mas parada antes do envio | Mesmos motivos da fila, espera por conexão |
| **Proxy negotiation** | Negociação com proxy | Proxy lento/configuração PAC |
| **DNS Lookup** | Passo 4 | Resolvedor lento; zero quando há cache ou reuso de conexão |
| **Initial connection** | Passo 5 (TCP/QUIC, inclui TLS) | Latência alta / servidor sobrecarregado |
| **SSL** | Passo 6 (parte da *Initial connection*) | Handshake TLS lento, cadeia de certificados grande, OCSP |
| **Request sent** | Passo 7 | Normalmente desprezível; alto com uploads grandes |
| **Waiting for server response** (antes chamado **Waiting (TTFB)**) | Passo 8 | **Back-end lento** (consultas ao banco, lógica pesada) |
| **Content Download** | Recebimento do corpo | Resposta grande, banda limitada, falta de compressão |

Há também fases ligadas a Service Workers (como *ServiceWorker Preparation* e *Request to ServiceWorker*) quando aplicável.

> [!TIP]
> Para observar o fluxo completo "a frio": abra o DevTools, marque **Disable cache** na aba *Network*, ative a coluna **Protocol** (clique com o botão direito no cabeçalho da tabela) e recarregue. Você verá o documento HTML primeiro, depois CSS/JS/imagens descobertos pelo parser, o protocolo usado (`http/1.1`, `h2`, `h3`) e, na primeira requisição, DNS + conexão + SSL. Repetindo sem *Disable cache*, as conexões e o cache DNS são reaproveitados e essas fases tendem a sumir.

<a id="s1-3"></a>
### 1.3 DNS, IP, TCP, UDP, HTTP e HTTPS

A comunicação na Web apoia-se em uma pilha de protocolos cooperativos. Cada camada resolve um problema específico e entrega um serviço bem definido à camada imediatamente superior:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ Camada de Aplicação  │ HTTP/1.1, HTTP/2, HTTP/3, DNS (resolução nomes) │
├──────────────────────┼──────────────────────────────────────────────────┤
│ Camada de Segurança  │ TLS 1.2 / TLS 1.3 (embutido no QUIC no HTTP/3)   │
├──────────────────────┼──────────────────────────────────────────────────┤
│ Camada de Transporte │ TCP (HTTP/1.1 e HTTP/2)  │  UDP + QUIC (HTTP/3)  │
├──────────────────────┼──────────────────────────────────────────────────┤
│ Camada de Rede       │ IP (IPv4 / IPv6) — endereçamento e roteamento    │
└─────────────────────────────────────────────────────────────────────────┘
```

A seguir, estudamos cada um desses pilares com foco em funcionamento na Web, exemplos práticos e segurança. O detalhamento de infraestrutura de redes (sub-redes, CIDR, NAT, ARP, DHCP, ICMP e portas) encontra-se na [seção 4 — Redes de Computadores](./04-redes-de-computadores.md#s4).

<a id="s1-3-1"></a>
#### 1.3.1 DNS (Domain Name System)

**O que é:** o **DNS** (*Domain Name System* — Sistema de Nomes de Domínio, especificado originalmente nas RFC 1034 e RFC 1035) é um banco de dados distribuído, hierárquico e global que traduz nomes de domínio legíveis por humanos (como `www.loja-ficticia.test`) em endereços IP numéricos (como `203.0.113.10` ou `2001:db8::10`) e outros metadados de serviço.

**Para que serve:** desacoplar a identidade lógica de um serviço (o nome de domínio) de sua localização física/topológica na rede (o endereço IP). Graças ao DNS, você pode trocar de provedor de nuvem, colocar uma CDN na frente do site ou distribuir carga entre dezenas de endereços IP sem que o usuário precise decorar números ou alterar seus favoritos.

**Como funciona:**

A árvore do DNS é organizada da direita para a esquerda a partir da **raiz** (`.`), seguida pelos **TLDs** (*Top-Level Domains* — domínios de topo, como `.com`, `.org`, `.br`, `.test`), domínios de segundo nível (`loja-ficticia.test`) e subdomínios (`api.loja-ficticia.test`). O nome completo terminado pelo ponto da raiz (`api.loja-ficticia.test.`) é chamado de **FQDN** (*Fully Qualified Domain Name* — nome de domínio totalmente qualificado).

Os servidores DNS autoritativos armazenam **registros de recurso** (*Resource Records* — RR) em arquivos de zona. Os principais tipos cobrados em provas e usados em auditorias de segurança são:

| Tipo | Nome / Significado | Finalidade principal | Exemplo de valor |
|---|---|---|---|
| **A** | *Address* (IPv4) | Mapeia um nome para um endereço **IPv4** de 32 bits | `203.0.113.10` |
| **AAAA** | *Quad-A* (IPv6) | Mapeia um nome para um endereço **IPv6** de 128 bits (RFC 3596) | `2001:db8:85a3::8a2e:370:7334` |
| **CNAME** | *Canonical Name* | Cria um **apelido** (*alias*) apontando um nome para outro nome canônico (nunca para um IP diretamente). Não deve coexistir com outros registros no mesmo nó (como no ápice da zona `loja-ficticia.test`) | `www.loja-ficticia.test. -> loja-ficticia.cdn.example.net.` |
| **MX** | *Mail Exchange* | Indica os servidores responsáveis por receber e-mails para o domínio, com número de **prioridade** (menor valor = maior preferência) | `10 mail1.loja-ficticia.test.` |
| **TXT** | *Text* | Armazena texto arbitrário; amplamente usado para verificação de posse de domínio e políticas de segurança de e-mail (**SPF** — *Sender Policy Framework*, **DKIM** — *DomainKeys Identified Mail*, **DMARC** — *Domain-based Message Authentication, Reporting, and Conformance*) | `"v=spf1 include:_spf.exemplo.test -all"` |
| **NS** | *Name Server* | Delega autoridade da zona ou subzona para servidores DNS específicos | `ns1.loja-ficticia.test.` |
| **SOA** | *Start of Authority* | Define parâmetros globais da zona: servidor primário, e-mail do administrador, número de série (*serial*) e temporizadores de sincronização (*refresh*, *retry*, *expire*, *negative caching TTL*) | `ns1.loja-ficticia.test. hostmaster.loja-ficticia.test. 2026103001 7200 3600 1209600 300` |
| **PTR** | *Pointer* | **DNS reverso**: traduz um endereço IP de volta para um nome de domínio (usando as zonas especiais `in-addr.arpa` para IPv4 e `ip6.arpa` para IPv6) | `10.113.0.203.in-addr.arpa. -> mail1.loja-ficticia.test.` |
| **CAA** | *Certification Authority Authorization* | Especifica (RFC 8659) quais Autoridades Certificadoras (CAs) têm permissão para emitir certificados TLS para o domínio (`issue`, `issuewild`, `iodef`). As CAs são obrigadas a checar este registro antes da emissão | `0 issue "letsencrypt.org"` |

Além desses, ambientes modernos utilizam registros **SRV** (localização de serviços por porta/peso) e **SVCB/HTTPS** (RFC 9460, que permitem anunciar suporte a HTTP/3/QUIC e chaves ECH — *Encrypted Client Hello* — já durante a consulta DNS).

- **TTL (*Time To Live* — tempo de vida):** valor em segundos presente em cada registro DNS que determina por quanto tempo resolvedores recursivos e sistemas operacionais podem manter a resposta em cache antes de consultar novamente o servidor autoritativo.
  - **TTL alto** (ex.: `86400` = 24 h): melhora o desempenho (menos consultas na rede) e reduz carga nos servidores autoritativos, mas atrasa a propagação de mudanças.
  - **TTL baixo** (ex.: `60` a `300` = 1 a 5 min): permite troca rápida de IP em migrações ou *failover* de contingência, ao custo de mais consultas DNS.
- **DoT (*DNS over TLS*, RFC 7858) e DoH (*DNS over HTTPS*, RFC 8484):** tradicionalmente, consultas DNS trafegam em texto claro via UDP/53 (ou TCP/53), permitindo que o provedor de acesso ou alguém na mesma rede Wi-Fi veja todos os domínios visitados e até falsifique respostas.
  - **DoT** encapsula o protocolo DNS diretamente sobre uma conexão TLS na porta dedicada **TCP/853**. É fácil de identificar e filtrar em firewalls corporativos, pois usa porta própria.
  - **DoH** encapsula consultas DNS dentro de requisições HTTPS padrão na porta **TCP/443** (usando `GET` ou `POST` com tipo `application/dns-message`). Do ponto de vista da rede externa, o tráfego mistura-se ao tráfego HTTPS comum.
- **DNSSEC (*Domain Name System Security Extensions*, RFC 4033–4035):** conjunto de extensões que adiciona **autenticidade de origem e integridade** aos registros DNS por meio de assinaturas criptográficas de chave pública (registros `RRSIG`, `DNSKEY`, `DS`, `NSEC`/`NSEC3`). **Atenção para a prova:** o DNSSEC **não criptografa** as consultas nem garante confidencialidade (quem protege a confidencialidade do transporte é o DoH/DoT); o DNSSEC garante que a resposta recebida realmente veio do dono da zona e não foi adulterada no caminho.

**Exemplo prático (`dig` e `nslookup`):**

Consultando registros `A`, `TXT` e `CAA` com a ferramenta `dig` (Linux/macOS/BIND tools) e `nslookup` (Windows/Linux):

```bash
# Consulta detalhada de registro A com dig
$ dig www.loja-ficticia.test A +noall +answer

; <<>> DiG 9.18 <<>> www.loja-ficticia.test A +noall +answer
www.loja-ficticia.test.   300   IN   CNAME   loja-ficticia.cdn.example.net.
loja-ficticia.cdn.example.net. 60 IN A       203.0.113.10

# Consulta de políticas CAA e TXT (SPF)
$ dig loja-ficticia.test CAA +short
0 issue "letsencrypt.org"
0 iodef "mailto:security@loja-ficticia.test"

$ dig loja-ficticia.test TXT +short
"v=spf1 ip4:203.0.113.0/24 -all"

# Consulta equivalente no Windows/Linux usando nslookup
$ nslookup -type=MX loja-ficticia.test 1.1.1.1
Server:  one.one.one.one
Address: 1.1.1.1

Non-authoritative answer:
loja-ficticia.test      MX preference = 10, mail exchanger = mail1.loja-ficticia.test
```

**Possíveis problemas de segurança:**

1. **Envenenamento de cache DNS (*DNS Cache Poisoning* / *DNS Spoofing*):** um atacante na rede local (via ARP spoofing) ou que explore falhas em resolvedores recursivos injeta respostas DNS falsas antes da resposta legítima chegar. Como consequência, `loja-ficticia.test` passa a resolver para o IP do atacante.
2. **Sequestro de DNS (*DNS Hijacking*):** comprometimento da conta no registrador de domínios (*registrar*) ou no provedor de DNS autoritativo, alterando os registros `NS`, `A` ou `MX` para redirecionar tráfego e interceptar e-mails corporativos.
3. **Subdomain Takeover (Tomada de subdomínio):** ocorre quando um subdomínio (ex.: `campanha.loja-ficticia.test`) possui um registro `CNAME` ou `A` apontando para um serviço externo em nuvem (GitHub Pages, AWS S3, Azure App Service, Heroku) que foi **desativado ou excluído** pela empresa, mas o registro DNS foi esquecido. Qualquer atacante pode cadastrar aquele nome de recurso vago no provedor de nuvem e passar a responder em nome de `campanha.loja-ficticia.test` — podendo roubar cookies de escopo amplo (`Domain=.loja-ficticia.test`), burlar políticas de CORS/CSP que confiem em `*.loja-ficticia.test` e aplicar golpes de *phishing* em um domínio legítimo.
4. **Transferência de zona aberta (`AXFR`):** servidores autoritativos mal configurados que respondem a pedidos `AXFR` (TCP/53) para qualquer IP revelam a lista completa de subdomínios internos e servidores de homologação de uma só vez.

**Configuração segura:**

- **[REQUISITO]** Auditar periodicamente a zona DNS e remover imediatamente registros `CNAME`, `A` ou `NS` órfãos (*dangling DNS records*) antes ou no exato momento de descomissionar serviços na nuvem (prevenção de *Subdomain Takeover*).
- **[REQUISITO]** Restringir transferências de zona (`AXFR`/`IXFR`) exclusivamente aos endereços IP dos servidores DNS secundários autorizados (e autenticá-las com TSIG — *Transaction Signature*).
- **[RECOMENDAÇÃO]** Habilitar **DNSSEC** na zona do domínio e proteger a conta do painel do registrador com autenticação multifator (**MFA**) forte e bloqueio de transferência (*Registry Lock*).
- **[RECOMENDAÇÃO]** Publicar registros **CAA** restringindo quais Autoridades Certificadoras podem emitir certificados para o domínio e subdomínios, além de configurar **SPF**, **DKIM** e **DMARC** em registros `TXT` para evitar falsificação de e-mails.
- **[BOA PRÁTICA]** Utilizar resolvedores com suporte a validação DNSSEC e transporte criptografado (**DoH** ou **DoT**) nos clientes ou no resolvedor central da rede corporativa.

<a id="s1-3-2"></a>
#### 1.3.2 IP (Internet Protocol)

**O que é:** o **IP** (*Internet Protocol* — Protocolo de Internet) é o protocolo principal da camada de Rede (camada 3 do modelo OSI / camada Internet do modelo TCP/IP), responsável pelo **endereçamento lógico** de hosts e pelo **roteamento de pacotes** (datagramas IP) entre redes distintas através de roteadores. Possui duas versões em operação na Web: **IPv4** (RFC 791) e **IPv6** (RFC 8200).

**Para que serve:** identificar unicamente interfaces de rede em escala global ou privada e permitir que um pacote saia da máquina do usuário, atravesse dezenas de roteadores independentes (*hops* — saltos) na Internet e chegue ao servidor de destino.

**Como funciona:**

O IP é um protocolo de entrega por **melhor esforço** (*best-effort*), **sem conexão** (*connectionless*) e **não confiável**: ele não garante que os pacotes chegarão, nem que chegarão na ordem enviada, nem que não serão duplicados. Toda garantia de entrega e ordenação fica a cargo da camada superior (TCP ou QUIC). Cada pacote IP carrega no cabeçalho o **IP de origem**, o **IP de destino** e um contador de vida (**TTL** no IPv4 ou **Hop Limit** no IPv6), que é decrementado em `1` a cada roteador; se chegar a `0`, o pacote é descartado (evitando loops infinitos de roteamento).

Na Web atual, **IPv4 e IPv6** coexistem em estratégia **Dual-Stack** (pilha dupla):

| Característica | IPv4 (RFC 791) | IPv6 (RFC 8200) | Impacto direto no Desenvolvimento Web |
|---|---|---|---|
| **Tamanho do endereço** | 32 bits (4 octetos) — ~4,29 bilhões de endereços | 128 bits (16 octetos) — $\approx 3{,}4 \times 10^{38}$ endereços | Colunas de banco de dados para salvar IP de auditoria precisam suportar até 45 caracteres texto (ou 16 bytes binários), e não apenas 15 caracteres |
| **Notação textual** | Decimal pontuado: `203.0.113.10` | Hexadecimal em 8 grupos de 16 bits: `2001:db8::10` | Em URLs literais, o IPv6 exige colchetes para não confundir com a porta: `https://[2001:db8::10]:8443/` |
| **NAT (*Network Address Translation*)** | Uso massivo devido à exaustão de endereços públicos | Desenhado para conectividade fim a fim sem NAT | Em IPv4 (e CGNAT de operadoras móveis), milhares de usuários diferentes chegam ao servidor Web com o **mesmo IP público**; no IPv6, um usuário costuma ter um prefixo `/64` inteiro |
| **Escolha pelo navegador** | Registro DNS `A` | Registro DNS `AAAA` | Navegadores usam o algoritmo **Happy Eyeballs** (RFC 8305): tentam IPv6 com ligeira preferência e, se demorar (ex.: ~250 ms), disparam IPv4 em paralelo, usando o que conectar primeiro |

**Exemplo prático:**

No servidor Web, o endereço IP da conexão TCP aparece como o endereço remoto (`remote_addr`). Veja como uma aplicação precisa lidar corretamente com IPv4, IPv6 e representação em URLs:

```http
GET /status HTTP/1.1
Host: [2001:db8:85a3::8a2e:370:7334]:8080
User-Agent: curl/8.5.0
Accept: application/json
```

**Possíveis problemas de segurança:**

1. **IP Spoofing (Falsificação de IP de origem):** um atacante monta pacotes IP brutos colocando no campo "IP de origem" o endereço de outra máquina.
   - *Nota importante para a prova:* em conexões **TCP** completas na Internet aberta, o *IP spoofing* cego **não permite** concluir o handshake de 3 vias (porque o pacote `SYN-ACK` do servidor vai para o IP verdadeiro falsificado, e não para o atacante). Porém, o *IP spoofing* é amplamente explorado em pacotes **UDP** (ataques de amplificação/reflexão DDoS) e em ataques de inundação `SYN Flood` (onde o atacante não se importa em receber a resposta).
2. **Exposição do IP real do servidor de origem por trás de CDN/WAF:** muitas empresas contratam um WAF ou CDN (como Cloudflare ou Akamai) para proteger a aplicação, mas deixam o servidor de origem aceitando conexões na porta 80/443 vindas de qualquer IP da Internet. O atacante descobre o IP real da origem por meio de histórico de DNS, registros `MX`/`TXT` (SPF), subdomínios esquecidos (`dev.`, `ftp.`) ou busca em motores de certificados (Censys, Shodan) e envia ataques diretamente ao IP da origem, contornando 100% do WAF e da proteção DDoS.
3. **SSRF (*Server-Side Request Forgery*) para IPs internos e metadados de nuvem:** quando uma aplicação back-end aceita uma URL fornecida pelo usuário (ex.: para buscar um avatar ou gerar pré-visualização de link) e faz a requisição HTTP a partir do servidor, o atacante pode informar endereços IP privados (`127.0.0.1`, `10.0.0.1`, `192.168.1.1`), representações alternativas/IPv6 (`[::1]`, `[::ffff:127.0.0.1]`) ou o IP mágico de metadados de nuvem **`169.254.169.254`** (link-local usado por AWS, GCP, Azure) para roubar credenciais administrativas da instância (veja [seção 3.4 — SSRF](./03a-vulnerabilidades-parte1.md#s3)).
4. **Bloqueio abusivo por IP único em CGNAT / IPv6:** bloquear um único IPv4 por muitas tentativas de login pode derrubar centenas de clientes legítimos atrás do mesmo CGNAT de telefonia móvel; inversamente, bloquear um único endereço IPv6 de `/128` é ineficaz, pois o atacante gira facilmente entre os $2^{64}$ endereços do seu prefixo `/64`.

**Configuração segura:**

- **[REQUISITO]** Quando usar CDN/WAF na borda, configurar o firewall de rede (*Security Group* / `iptables` / `nftables`) do servidor de origem para aceitar tráfego HTTP/HTTPS **exclusivamente** das faixas de IP oficiais da CDN/balanceador (ou usar túneis privados e mTLS).
- **[REQUISITO]** Em qualquer rotina do servidor que busque URLs externas informadas pelo usuário, resolver o DNS e **bloquear conexões para faixas de IP não públicas** (loopback `127.0.0.0/8` e `::1/128`, privadas RFC 1918 `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, link-local/metadados `169.254.0.0/16` e `fe80::/10`, ULA `fc00::/7`), revalidando após o DNS e desativando redirecionamentos automáticos para evitar *DNS rebinding*.
- **[RECOMENDAÇÃO]** Em regras de *rate limiting* e reputação para IPv6, agrupar endereços pelo prefixo de rede do assinante (tipicamente `/64` ou `/56`), e não apenas pelo endereço individual `/128`.
- **[BOA PRÁTICA]** No nível de operadora/infraestrutura de rede, aplicar filtragem de egresso/ingresso **BCP 38** (RFC 2827) para impedir a saída de pacotes com IP de origem falsificado.

<a id="s1-3-3"></a>
#### 1.3.3 TCP (Transmission Control Protocol)

**O que é:** o **TCP** (*Transmission Control Protocol* — Protocolo de Controle de Transmissão, consolidado na RFC 9293, que atualizou a clássica RFC 793) é um protocolo da camada de Transporte (camada 4) **orientado a conexão**, que fornece um fluxo de bytes (*byte stream*) **confiável**, **ordenado** e **bidirecional** (*full-duplex*) sobre a rede IP não confiável. É a base de transporte do **HTTP/1.1** e do **HTTP/2**.

**Para que serve:** garantir que uma página HTML, um arquivo JavaScript ou um payload JSON enviado pela aplicação chegue ao outro lado **completo, sem corrupção e exatamente na mesma ordem em que foi transmitido**, adaptando a velocidade de envio à capacidade do receptor e ao congestionamento da rede.

**Como funciona:**

1. **Three-Way Handshake (aperto de mão de três vias):** antes de qualquer byte de HTTP trafegar, cliente e servidor estabelecem uma conexão lógica sincronizando números de sequência iniciais (**ISN** — *Initial Sequence Number*, gerados de forma imprevisível):
   - **Passo 1 (`SYN`):** o cliente envia um segmento com a flag `SYN=1` e seu número de sequência inicial `Seq = x`.
   - **Passo 2 (`SYN-ACK`):** o servidor responde com `SYN=1, ACK=1`, confirmando o recebimento com `Ack = x + 1` e informando seu próprio número inicial `Seq = y`.
   - **Passo 3 (`ACK`):** o cliente envia `ACK=1` com `Seq = x + 1` e `Ack = y + 1`. A conexão entra no estado `ESTABLISHED` e a requisição HTTP (ou o handshake TLS) pode começar.
2. **Confiabilidade e ordenação:** cada byte transmitido é numerado pelo campo *Sequence Number*. O receptor confirma os bytes recebidos com *Acknowledgments* (`ACK`). Se um segmento se perde no caminho, o emissor detecta a falta de confirmação (por *timeout* ou por `ACKs` duplicados — *Fast Retransmit*) e **retransmite** o segmento perdido. Se pacotes IP chegarem fora de ordem por rotas diferentes, o sistema operacional do receptor usa os números de sequência para reordená-los no buffer antes de entregá-los ao navegador ou ao servidor Web.
3. **Controle de fluxo vs. Controle de congestionamento (distinção clássica de prova):**
   - **Controle de fluxo (*Flow Control*):** protege o **receptor** para que ele não tenha seu buffer de memória inundado por um emissor rápido demais. É controlado pela **janela de recepção** (*Receive Window* — `rwnd`) anunciada pelo receptor no cabeçalho TCP a cada segmento.
   - **Controle de congestionamento (*Congestion Control*):** protege a **rede intermediária** (roteadores e enlaces) contra sobrecarga. O emissor mantém uma **janela de congestionamento** (`cwnd`) estimada dinamicamente por algoritmos como *Slow Start*, *Congestion Avoidance*, CUBIC ou BBR. A taxa efetiva de envio é limitada pelo menor valor entre `rwnd` e `cwnd`.
4. **Head-of-Line (HoL) Blocking do TCP:** como o TCP garante entrega estritamente ordenada de um único fluxo de bytes, se o pacote #2 se perder na rede mas os pacotes #3, #4 e #5 já tiverem chegado, o sistema operacional **retém** os pacotes #3, #4 e #5 no buffer do kernel até que o pacote #2 seja retransmitido com sucesso. No HTTP/2 (que multiplexa dezenas de arquivos sobre uma única conexão TCP), a perda de um único pacote TCP trava momentaneamente a entrega de todos os fluxos daquela conexão — limitação que motivou a criação do HTTP/3 sobre QUIC/UDP.

**Exemplo prático (diagrama do Handshake TCP + início do HTTPS):**

```text
Cliente (198.51.100.25:54321)                        Servidor (203.0.113.10:443)
      │                                                           │
      │ 1. TCP SYN  [Seq = 1000]                                  │
      │ ────────────────────────────────────────────────────────► │
      │                                                           │  (1 RTT de
      │ 2. TCP SYN, ACK  [Seq = 5000, Ack = 1001]                 │   conexão TCP)
      │ ◄──────────────────────────────────────────────────────── │
      │                                                           │
      │ 3. TCP ACK  [Seq = 1001, Ack = 5001]                      │
      │ ────────────────────────────────────────────────────────► │
      │ ═══ Conexão TCP ESTABLISHED (inicia Handshake TLS) ══════ │
```

**Possíveis problemas de segurança:**

1. **SYN Flood (Inundação SYN — ataque DoS/DDoS):** o atacante envia milhares de segmentos `SYN` (geralmente com IPs de origem falsificados) para a porta 80/443 do servidor e nunca envia o terceiro pacote (`ACK`). Sem proteção, o servidor aloca memória na fila de conexões semiabertas (`SYN_RCVD` / *SYN backlog*) esperando pelos `ACKs` que jamais virão, até esgotar a fila e impedir clientes reais de se conectarem.
2. **TCP Session Hijacking (Sequestro de conexão TCP) e injeção de `RST`:** se um atacante conseguir adivinhar ou observar (em rede local) a quádrupla da conexão (`IP origem, porta origem, IP destino, porta destino`) e os números de sequência de 32 bits em uso, ele pode injetar pacotes forjados na conexão ou derrubá-la enviando um segmento com a flag `RST` (*Reset*).
3. **Exaustão de conexões e ataques *Slowloris* (camada 7 sobre TCP):** um atacante abre milhares de conexões TCP completas com o servidor Web e envia cabeçalhos HTTP de forma extremamente lenta (ex.: 1 byte a cada 15 segundos), mantendo os *workers* ou descritores de arquivo do servidor ocupados até derrubar o serviço com pouquíssima banda.

**Configuração segura:**

- **[REQUISITO]** Manter habilitado no sistema operacional do servidor o mecanismo de **SYN Cookies** (no Linux: `net.ipv4.tcp_syncookies = 1`), que codifica o estado inicial da conexão de forma criptográfica no próprio número de sequência `ISN` do `SYN-ACK`, evitando alocar memória na tabela de conexões antes de o cliente provar que é real no terceiro passo (`ACK`).
- **[REQUISITO]** Utilizar **TLS (HTTPS)** sobre o TCP: mesmo que alguém tente injetar segmentos TCP na rede, os registros TLS possuem autenticação criptográfica (AEAD) e números de sequência próprios; qualquer byte injetado invalida a tag de integridade e encerra a sessão com segurança.
- **[RECOMENDAÇÃO]** Configurar *timeouts* agressivos de leitura de cabeçalho e corpo no servidor Web/proxy reverso (ex.: `client_header_timeout 10s; client_body_timeout 10s;` no Nginx) e limites de conexões simultâneas por IP (`limit_conn`) para mitigar ataques *Slowloris* e exaustão de sockets.
- **[BOA PRÁTICA]** Garantir geração aleatória forte de números de sequência iniciais (ISN, RFC 6528 — padrão nos kernels modernos) para inviabilizar adivinhação de sequência fora do caminho (*off-path TCP hijacking*).

<a id="s1-3-4"></a>
#### 1.3.4 UDP (User Datagram Protocol)

**O que é:** o **UDP** (*User Datagram Protocol* — Protocolo de Datagrama de Usuário, RFC 768) é um protocolo da camada de Transporte (camada 4) **sem conexão** (*connectionless*), minimalista e de baixa sobrecarga (*overhead*), que envia mensagens discretas chamadas **datagramas** sem estabelecer handshake prévio e sem garantias nativas de entrega, ordenação ou controle de congestionamento.

**Para que serve:** atender aplicações em que a **baixa latência** é mais crítica do que a retransmissão automática no kernel, ou nas quais a própria camada superior implementa seu próprio controle de confiabilidade e criptografia sob medida. Na Web moderna, o UDP é essencial em quatro frentes:

1. **DNS tradicional (UDP/53):** consultas e respostas rápidas em 1 único RTT, sem gastar 1 RTT extra apenas para abrir uma conexão TCP.
2. **HTTP/3 sobre QUIC (UDP/443):** o protocolo **QUIC** (RFC 9000), base do **HTTP/3** (RFC 9114), roda sobre UDP e implementa no espaço de usuário (dentro do navegador e do servidor) confiabilidade, controle de congestionamento por fluxo (*stream*) e TLS 1.3 integrado. Como cada *stream* QUIC é independente, a perda de um pacote UDP afeta **apenas o recurso daquele stream**, eliminando o *Head-of-Line Blocking* da camada de transporte que afetava o TCP no HTTP/2.
3. **WebRTC (*Web Real-Time Communication*):** videoconferências, chamadas de voz e compartilhamento de tela no navegador usam UDP (com SRTP — *Secure Real-time Transport Protocol* — e DTLS — *Datagram TLS*) porque, em áudio e vídeo ao vivo, um pacote de voz atrasado 300 ms não serve mais; é preferível descartar o quadro perdido e exibir o quadro seguinte em tempo real.
4. ** Jogos online e streaming de baixa latência:** atualização de posição e estado em tempo real.

**Como funciona:**

O cabeçalho UDP possui apenas **8 bytes** fixos (contra 20 a 60 bytes do TCP): Porta de Origem (16 bits), Porta de Destino (16 bits), Comprimento total (16 bits) e Checksum (16 bits — opcional em IPv4, obrigatório em IPv6). A aplicação chama o socket, o sistema operacional empacota os dados no datagrama UDP, entrega ao IP e o envia imediatamente na rede.

**Exemplo prático (comparação de abertura de sessão HTTP/2 sobre TCP+TLS vs. HTTP/3 sobre UDP+QUIC):**

```text
HTTP/2 (sobre TCP + TLS 1.3)                 HTTP/3 (sobre UDP + QUIC c/ TLS 1.3 nativo)
Cliente                    Servidor          Cliente                            Servidor
  │── 1. TCP SYN ────────────►│                │── 1. QUIC Initial (ClientHello) ─►│
  │◄─ 2. TCP SYN-ACK ─────────│ 1º RTT         │◄─ 2. QUIC Handshake (ServerHello, │ 1º RTT
  │── 3. TCP ACK + TLS Hello ►│                │      Cert, Finished) ─────────────│
  │◄─ 4. TLS ServerHello... ──│ 2º RTT         │── 3. ACK + GET /index.html ──────►│
  │── 5. TLS Finished + GET ─►│                │   (Em retomada 0-RTT, o GET já    │
  │   (2 RTTs até enviar HTTP)│                │    vai no pacote #1!)             │
```

**Possíveis problemas de segurança:**

1. **Ataques DDoS de Reflexão e Amplificação UDP:** como o UDP não tem handshake de 3 vias para comprovar que o IP de origem realmente pertence a quem enviou o pacote, um atacante pode enviar pequenos pacotes UDP para servidores mal configurados na Internet (resolvedores DNS abertos, servidores NTP, Memcached, SSDP/UPnP, CLDAP) falsificando o **IP de origem** para que seja o IP da **vítima**. Os servidores respondem com datagramas muito maiores (fator de amplificação de 10x a mais de 50.000x no caso de Memcached UDP) diretamente para a vítima, saturando seu link de rede.
2. **Vazamento de IP real via WebRTC (*WebRTC IP Leak*):** o WebRTC no navegador utiliza protocolos auxiliares sobre UDP (**STUN** — *Session Traversal Utilities for NAT* — e **ICE**) para descobrir os endereços IP locais da interface de rede e o IP público atrás do NAT, podendo expor o IP real de um usuário mesmo quando ele utiliza uma VPN mal configurada ou proxy apenas HTTP.

**Configuração segura:**

- **[REQUISITO]** Nunca expor serviços baseados em UDP que gerem amplificação (como Memcached na porta 11211/UDP ou resolvedores DNS recursivos abertos na porta 53/UDP) para a Internet pública.
- **[REQUISITO]** Em protocolos modernos sobre UDP (como QUIC / HTTP/3, RFC 9000), exigir que o pacote inicial do cliente (`QUIC Initial Packet`) tenha pelo menos **1.200 bytes** de tamanho (preenchido com *padding*) e validar o endereço por token antes de enviar respostas grandes — regra embutida na RFC 9000 que impede que o HTTP/3 seja usado como amplificador DDoS (limite de amplificação de 3x antes da validação do endereço).
- **[RECOMENDAÇÃO]** Aplicar *Response Rate Limiting* (RRL) em servidores DNS autoritativos públicos para mitigar abusos de reflexão.
- **[BOA PRÁTICA]** Ao habilitar HTTP/3 no servidor Web, abrir a porta **UDP/443** no firewall e anunciar o suporte via header `Alt-Svc: h3=":443"; ma=86400` nas respostas HTTPS.

<a id="s1-3-5"></a>
#### 1.3.5 HTTP (Hypertext Transfer Protocol)

**O que é:** o **HTTP** (*Hypertext Transfer Protocol* — Protocolo de Transferência de Hipertexto) é o protocolo da camada de Aplicação (camada 7) que rege a comunicação entre clientes e servidores na Web. Sua semântica atual (métodos, códigos de status, cabeçalhos, cache) é padronizada de forma unificada na **RFC 9110** (*HTTP Semantics*), independentemente da versão de transporte na rede.

**Para que serve:** padronizar um idioma universal de requisições e respostas para buscar documentos HTML, folhas de estilo, scripts, imagens, vídeos, submeter formulários e trocar dados estruturados (JSON, XML, Protobuf) entre APIs.

**Como funciona — evolução das versões do HTTP (tema recorrente em provas):**

| Versão | Especificação | Transporte | Formato no fio | Principais características e avanços |
|---|---|---|---|---|
| **HTTP/0.9** (1991) | Documentação original do CERN | TCP | Texto ASCII | Apenas uma linha (`GET /pagina.html`), sem headers, sem códigos de status, só devolvia HTML e fechava a conexão imediatamente |
| **HTTP/1.0** (1996) | RFC 1945 | TCP | Texto ASCII | Introduziu **headers**, **códigos de status**, `Content-Type` (arquivos além de HTML) e métodos `POST` e `HEAD`. Por padrão, abria **uma nova conexão TCP por requisição** (a menos que negociado `Connection: keep-alive` não oficial) |
| **HTTP/1.1** (1997 / 2022) | RFC 2068 → RFC 2616 → **RFC 9112** | TCP | Texto ASCII | **Conexões persistentes por padrão** (`keep-alive`), header **`Host` obrigatório** (permitindo vários sites no mesmo IP — *Virtual Hosting*), *chunked transfer encoding*, cache avançado (`ETag`, `Cache-Control`). Sofre de **HoL Blocking de aplicação**: na prática, para baixar 6 arquivos em paralelo, o navegador abre até **6 conexões TCP** por origem |
| **HTTP/2** (2015 / 2022) | RFC 7540 → **RFC 9113** | TCP (na prática sempre com TLS — `h2`) | **Binário** (*frames*) | **Multiplexação completa** de várias requisições e respostas em paralelo sobre **uma única conexão TCP** (usando *streams* e *frames*), **compressão de cabeçalhos HPACK** (RFC 7541), priorização de fluxos e *Server Push* (hoje descontinuado no Chrome **[Obsoleto]**). Ainda sofre de HoL Blocking na camada **TCP** se houver perda de pacotes |
| **HTTP/3** (2022) | **RFC 9114** | **QUIC sobre UDP** (`h3`, RFC 9000) | **Binário** (*frames* QUIC) | Substitui o TCP pelo **QUIC (sobre UDP)** com **TLS 1.3 nativo obrigatório**, **elimina o HoL Blocking entre fluxos** na camada de transporte, reduz o tempo de conexão (1-RTT e 0-RTT), usa **compressão de cabeçalhos QPACK** (RFC 9204) e suporta **migração de conexão** (trocar do Wi-Fi para o 5G mudando de IP sem derrubar o download graças ao *Connection ID*) |

**Anatomia completa de uma mensagem HTTP/1.1 bruta (RFC 9112):**

No HTTP/1.1, as mensagens são texto estruturado em quatro partes, onde cada linha de controle e de cabeçalho termina obrigatoriamente com a sequência **CRLF** (`\r\n`, *Carriage Return + Line Feed*, bytes `0x0D 0x0A`) e o bloco de cabeçalhos é separado do corpo por uma **linha em branco** (`\r\n\r\n`):

**1. Exemplo de Requisição HTTP/1.1 bruta:**

```http
POST /api/v1/login HTTP/1.1
Host: api.loja-ficticia.test
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/130.0.0.0
Content-Type: application/json; charset=utf-8
Accept: application/json
Content-Length: 49

{"email":"ana@exemplo.test","senha":"S3nhaForte"}
```

- **Start-line (*Request-line*):** `POST` (método) + espaço + `/api/v1/login` (alvo da requisição / *request-target*) + espaço + `HTTP/1.1` (versão do protocolo) + `\r\n`.
- **Header fields (Cabeçalhos):** pares `Nome: Valor\r\n` com metadados sobre o host, formato do corpo e preferências do cliente.
- **Linha vazia (`\r\n`):** delimita o fim exato dos cabeçalhos.
- **Message body (Corpo da mensagem — opcional):** os `49` bytes de payload JSON enviados ao servidor.

**2. Exemplo de Resposta HTTP/1.1 bruta:**

```http
HTTP/1.1 200 OK
Date: Fri, 30 Oct 2026 10:15:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 43
Cache-Control: no-store
Set-Cookie: __Host-sid=9f8e7d6c5b4a; Path=/; Secure; HttpOnly; SameSite=Lax

{"autenticado":true,"usuario":"Ana Silva"}
```

- **Start-line (*Status-line*):** `HTTP/1.1` (versão) + espaço + `200` (código de status de 3 dígitos) + espaço + `OK` (*reason phrase* textual, que foi abolida no HTTP/2 e HTTP/3) + `\r\n`.
- **Header fields:** metadados da resposta, instruções de cache e de cookies.
- **Linha vazia (`\r\n`)** seguida pelo **Corpo da resposta**.

**Possíveis problemas de segurança (HTTP puro sem TLS):**

1. **Ausência de confidencialidade (Texto claro):** em `http://` (porta 80), toda a requisição e toda a resposta — incluindo senhas de formulários, cookies de sessão (`Cookie`), tokens `Authorization`, dados pessoais (CPF, cartão) e páginas visitadas — trafegam **em texto claro**. Qualquer equipamento no caminho (roteador Wi-Fi de cafeteria, provedor, switch mal configurado ou atacante executando *ARP Spoofing* com Wireshark/`tcpdump`) lê tudo instantaneamente.
2. **Ausência de integridade (Manipulação em trânsito):** um intermediário na rede pode alterar a resposta HTML/JavaScript em trânsito — por exemplo, injetando um script malicioso de captura de senhas (*keylogger*), anúncios indesejados ou trocando o arquivo `.exe`/`.zip` que o usuário estava baixando por um *malware*.
3. **Ausência de autenticidade (Impersonação):** o cliente não tem nenhuma garantia criptográfica de que o servidor que respondeu na porta 80 é realmente o dono de `loja-ficticia.test` e não um impostor na rede local.

**Configuração segura:**

- **[REQUISITO]** Nunca transmitir credenciais, tokens, cookies de sessão ou dados pessoais sobre HTTP puro (`http://`).
- **[REQUISITO]** Configurar a porta 80 (`http://`) **apenas** para emitir um redirecionamento permanente imediato (`301 Moved Permanently` ou `308 Permanent Redirect`) para a mesma URL em `https://` (porta 443), acompanhado de **HSTS** nas respostas HTTPS.
- **[RECOMENDAÇÃO]** Habilitar **HTTP/2** (`h2`) e **HTTP/3** (`h3`) sobre TLS nos servidores Web/CDNs para melhorar desempenho e resiliência de conexão.

<a id="s1-3-6"></a>
#### 1.3.6 HTTPS (HTTP over TLS)

**O que é:** o **HTTPS** (*Hypertext Transfer Protocol Secure*) não é um protocolo diferente do HTTP, mas sim o próprio protocolo HTTP (1.1, 2 ou 3) encapsulado sobre uma camada criptográfica **TLS** (*Transport Layer Security* — Segurança da Camada de Transporte, cujas versões seguras atuais são a **TLS 1.2**, RFC 5246, e a **TLS 1.3**, RFC 8446). Historicamente era chamado de *SSL* (*Secure Sockets Layer*), termo hoje tecnicamente **[Obsoleto]** (SSL 2.0, SSL 3.0, TLS 1.0 e TLS 1.1 foram formalmente depreciados pelo IETF — veja RFC 8996 — mas o nome "SSL" ainda aparece por tradição na interface do DevTools e em diretivas como `ssl_certificate` do Nginx).

**Para que serve:** fornecer a tríade de segurança em trânsito para a comunicação Web:

1. **Confidencialidade (Criptografia):** ninguém no caminho consegue ler o caminho da URL (`/minha-conta?cpf=...`), os headers, os cookies ou o corpo da requisição/resposta.
2. **Integridade (Autenticação de mensagem — AEAD):** ninguém no caminho consegue modificar, remover, duplicar ou reordenar um único byte da mensagem HTTP sem que o receptor detecte imediatamente a adulteração e aborte a conexão.
3. **Autenticidade (Identificação via Certificados X.509):** o navegador comprova matematicamente que está conversando com o servidor legítimo detentor do domínio (ex.: `loja-ficticia.test`), impedindo ataques *Man-in-the-Middle* (MITM).

> [!IMPORTANT]
> **O que o HTTPS protege vs. o que o HTTPS NÃO protege (pegadinha clássica de prova):**
> - **O que um observador na rede AINDA VÊ mesmo com HTTPS:** o **endereço IP** de destino, a **porta** (443), o volume aproximado de dados e o tempo das mensagens, a consulta DNS (se não usar DoH/DoT) e, tradicionalmente, o **nome do domínio (host)** no campo **SNI** do handshake TLS (a menos que **ECH** — *Encrypted Client Hello* — esteja ativo).
> - **O que o observador NÃO VÊ no HTTPS:** o método (`GET`/`POST`), o **caminho e a query string** (`/busca?q=segredo`), todos os **headers HTTP** (`Cookie`, `Authorization`, `User-Agent`) e o **corpo** da requisição e da resposta.
> - **Atenção:** ter HTTPS ("cadeado" ou conexão segura) **não significa** que o site está livre de vulnerabilidades de aplicação (como SQL Injection, XSS, CSRF ou IDOR) nem que o dono do site é honesto (sites de *phishing* também emitem certificados HTTPS gratuitos em segundos!). O HTTPS garante apenas que o canal entre o navegador e aquele domínio é criptografado e autêntico.

**Como funciona — conceitos fundamentais do ecossistema TLS/HTTPS:**

- **Criptografia Híbrida (Assimétrica + Simétrica):**
  - Durante o **Handshake TLS**, utiliza-se **criptografia assimétrica** (chave pública e chave privada): troca de chaves efêmera **ECDHE** (*Elliptic Curve Diffie-Hellman Ephemeral*, que garante **Forward Secrecy** / Sigilo Direto — mesmo que a chave privada do servidor vaze no futuro, o tráfego gravado hoje não poderá ser descriptografado) e assinatura digital (RSA ou ECDSA) para autenticar o servidor.
  - Uma vez derivado o segredo compartilhado da sessão, toda a transmissão dos dados HTTP usa **criptografia simétrica autenticada (AEAD — *Authenticated Encryption with Associated Data*)**, como `AES-128-GCM`, `AES-256-GCM` ou `ChaCha20-Poly1305`, que é milhares de vezes mais rápida em hardware do que a criptografia assimétrica.
- **Certificados Digitais X.509 (RFC 5280), Autoridades Certificadoras (CA) e Cadeia de Confiança:**
  - Como o navegador sabe que a chave pública apresentada por `loja-ficticia.test` não pertence a um impostor? Porque o servidor apresenta um **Certificado X.509** contendo o nome do domínio (na extensão **SAN — *Subject Alternative Name***), o período de validade (`Not Before` / `Not After`), a chave pública do servidor e a **assinatura digital de uma Autoridade Certificadora (CA)** (ex.: Let's Encrypt, DigiCert, GlobalSign, Sectigo).
  - A **Cadeia de Confiança (*Chain of Trust*)** possui três níveis:
    1. **Root CA (CA Raiz):** seu certificado autoassinado já vem instalado de fábrica no repositório de confiança (*Trust Store*) do sistema operacional ou do navegador. A chave privada da Root CA fica guardada offline em cofres físicos com HSM (*Hardware Security Module*).
    2. **Intermediate CA (CA Intermediária):** assinada pela Root CA. É quem assina os certificados dos sites no dia a dia. Se houver um incidente, revoga-se a intermediária sem inutilizar a raiz.
    3. **Leaf / End-Entity Certificate (Certificado de Folha / do Servidor):** o certificado de `loja-ficticia.test`, assinado pela CA Intermediária. O servidor Web deve enviar na conexão tanto o seu certificado de folha quanto o(s) certificado(s) intermediário(s) (`fullchain.pem`).
- **SNI (*Server Name Indication*, RFC 6066):** como o handshake TLS acontece **antes** de o navegador enviar a requisição HTTP (e, portanto, antes do header `Host: loja-ficticia.test`), como um servidor que hospeda 50 domínios no mesmo endereço IP sabe qual certificado TLS deve apresentar? Graças à extensão **SNI**: logo na primeira mensagem do handshake TLS (`ClientHello`), o navegador informa em texto claro o nome do host desejado (`loja-ficticia.test`). Evolução recente: o **ECH (*Encrypted Client Hello*)** **[Varia por versão]** criptografa o SNI real usando uma chave pública distribuída via DNS (registro `HTTPS`), expondo na rede apenas um nome público do provedor/CDN (*outer SNI*).
- **Mixed Content (Conteúdo Misto):** ocorre quando a página principal é carregada com segurança via **`https://`**, mas o seu HTML, CSS ou JavaScript tenta carregar sub-recursos (imagens, scripts, folhas de estilo, iframes, chamadas `fetch`) via **`http://`** inseguro. A especificação W3C *Mixed Content Level 2* divide isso em duas categorias que os navegadores modernos tratam de forma rigorosa:
  - **Blockable (Ativo) Mixed Content:** `<script src="http://...">`, `<link rel="stylesheet" href="http://...">`, `<iframe>`, `fetch()`/`XMLHttpRequest`, `<object>`. Como um script ou CSS interceptado em HTTP pode assumir controle total da página HTTPS inteira, **todos os navegadores bloqueiam conteúdo misto ativo por padrão** (exibindo erro vermelho no Console e na aba Network).
  - **Optionally-blockable (Passivo) Mixed Content:** `<img>`, `<audio>`, `<video>`. Navegadores atuais (como o Chrome) tentam primeiro **atualizar automaticamente** (*auto-upgrade*) `http://` para `https://`; se o servidor não responder em HTTPS, o recurso também é bloqueado **[Varia por versão]**.
- **HSTS (*HTTP Strict Transport Security*, RFC 6797):** como vimos em [1.2.2](#s1-2-2) e aprofundaremos na [seção 5.2](./05-headers-de-seguranca.md#s5), o header `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` obriga o navegador a usar exclusivamente HTTPS em todas as conexões futuras para aquele domínio e impede que o usuário clique em "continuar mesmo assim" diante de erros de certificado inválido.

**Exemplo prático (inspecionando o handshake TLS e certificado com `curl`):**

```bash
$ curl -vI https://loja-ficticia.test
* Host loja-ficticia.test:443 was resolved.
* Connected to loja-ficticia.test (203.0.113.10) port 443
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=loja-ficticia.test
*  start date: Oct  1 00:00:00 2026 GMT
*  expire date: Dec 30 23:59:59 2026 GMT
*  subjectAltName: host "loja-ficticia.test" matched cert's "loja-ficticia.test"
*  issuer: C=US; O=Let's Encrypt; CN=E5
*  SSL certificate verify ok.
```

**Possíveis problemas de segurança:**

1. **Suporte a protocolos obsoletos (SSL 2.0/3.0, TLS 1.0, TLS 1.1) e cifras fracas (RC4, 3DES, CBC, RSA key exchange sem Forward Secrecy):** permite ataques de downgrade e descriptografia parcial (como POODLE, BEAST, LUCKY13, SWEET32, ROBOT).
2. **Certificados inválidos, expirados, autoassinados em produção ou com nome divergente (`ERR_CERT_COMMON_NAME_INVALID`, `ERR_CERT_DATE_INVALID`, `ERR_CERT_AUTHORITY_INVALID`):** ou cadeia incompleta (servidor envia apenas o certificado de folha e esquece a CA intermediária, funcionando em alguns desktops que fazem download extra via AIA — *Authority Information Access* — mas falhando em dispositivos móveis, `curl` e chamadas de API).
3. **Desativação de validação TLS no código back-end:** desenvolvedores que configuram `NODE_TLS_REJECT_UNAUTHORIZED=0` no Node.js ou `verify=False` no `requests` do Python para "fazer funcionar em homologação" e esquecem isso em produção tornam toda a criptografia inútil contra ataques MITM, pois o cliente aceita qualquer certificado falsificado sem verificar a assinatura da CA.
4. **SSL Stripping (Downgrade de HTTPS para HTTP no primeiro acesso):** quando o usuário digita `loja-ficticia.test` em uma rede hostil e o domínio não usa HSTS Preload, um atacante intercepta uma eventual requisição HTTP inicial e mantém a vítima conversando em HTTP com o atacante enquanto o atacante conversa em HTTPS com o servidor real.

**Configuração segura:**

- **[REQUISITO]** Habilitar **apenas TLS 1.2 e TLS 1.3** no servidor Web/balanceador, desativando completamente SSL 2.0, SSL 3.0, TLS 1.0 e TLS 1.1, e utilizar apenas suítes com **ECDHE** (*Forward Secrecy*) e cifras **AEAD** (`AES-GCM` e `ChaCha20-Poly1305`).
- **[REQUISITO]** Configurar o arquivo de certificado completo (**certificado de folha + intermediárias**, ex.: `fullchain.pem`), automatizar a renovação antes do vencimento (protocolo **ACME**, RFC 8555) e **jamais desativar a verificação de certificado** em clientes HTTP de produção.
- **[REQUISITO]** Redirecionar todo tráfego HTTP (porta 80) para HTTPS (porta 443) e enviar o header **`Strict-Transport-Security`** (HSTS).
- **[RECOMENDAÇÃO]** Utilizar a diretiva `Content-Security-Policy: upgrade-insecure-requests` durante migrações para converter automaticamente referências `http://` legadas em `https://` e evitar quebras por *Mixed Content*.
- **[BOA PRÁTICA]** Habilitar **OCSP Stapling** no servidor (para que o próprio servidor entregue a prova recente e assinada de que seu certificado não foi revogado, economizando latência e preservando a privacidade do usuário) e monitorar logs públicos de **Certificate Transparency (CT, RFC 6962)** para detectar emissões não autorizadas de certificados para o seu domínio.

<a id="s1-4"></a>
### 1.4 Métodos HTTP

Os **métodos HTTP** (frequentemente chamados de **verbos HTTP**, definidos na RFC 9110, seção 9, e na RFC 5789 para o `PATCH`) indicam a **semântica da ação desejada** sobre o recurso identificado pela URL.

Antes de analisar cada método individualmente, domine três propriedades formais definidas pela RFC 9110 que caem em quase toda prova de Desenvolvimento Web e Redes:

1. **Método Seguro (*Safe Method* — RFC 9110, seção 9.2.1):** um método é *safe* quando sua semântica é **exclusivamente de leitura** — isto é, o cliente não solicita nem espera qualquer alteração de estado no servidor como efeito da requisição (ex.: `GET`, `HEAD`, `OPTIONS`, `TRACE`).
   - *Atenção:* "seguro" (*safe*) aqui significa **sem efeitos colaterais de negócio** (*read-only*), e **não** "protegido contra ataques hackers". Além disso, um servidor pode gravar uma linha de log de auditoria ou incrementar uma métrica interna ao atender um `GET` sem violar a RFC, desde que não haja efeito colateral observável ou cobrado do cliente (como apagar um registro ou fechar uma compra).
2. **Método Idempotente (*Idempotent Method* — RFC 9110, seção 9.2.2):** um método é *idempotente* quando o efeito colateral no estado do servidor de **executar a mesma requisição $N$ vezes ($N \ge 1$) é exatamente idêntico ao efeito de executá-la uma única vez** (ex.: `GET`, `HEAD`, `OPTIONS`, `TRACE`, `PUT`, `DELETE`).
   - *Por que isso importa?* Se a conexão cair antes de a resposta chegar, o navegador, proxy ou biblioteca HTTP sabe que pode **repetir automaticamente** uma requisição idempotente (`GET`, `PUT`, `DELETE`) com segurança. Já o **`POST` não é idempotente** (repetir um `POST /pagamentos` pode cobrar o cartão do cliente duas vezes!), e o **`PATCH` também não é garantido como idempotente** pela especificação.
   - *Nota técnica:* idempotência refere-se ao **estado resultante no servidor**, e não necessariamente ao código de status retornado: o primeiro `DELETE /produtos/42` pode retornar `200 OK` (ou `204 No Content`) e o segundo `DELETE /produtos/42` pode retornar `404 Not Found`, mas o método continua sendo **idempotente** porque, após 1 ou 10 chamadas, o estado final no servidor é o mesmo (o produto 42 não existe mais).
3. **Cacheável (*Cacheable* — RFC 9110, seção 9.2.3):** indica se as respostas a esse método podem ser armazenadas para reutilização futura por caches do navegador ou CDNs (`GET` e `HEAD` são cacheáveis por padrão; `POST` e `PATCH` só são cacheáveis em condições raríssimas com cabeçalhos de frescor explícitos e `Content-Location`, e na prática de navegadores/CDNs não são cacheados; `PUT`, `DELETE`, `OPTIONS`, `CONNECT` e `TRACE` **não** são cacheáveis).

<a id="s1-4-1"></a>
#### 1.4.1 GET

**O que é:** o método **`GET`** solicita a **recuperação de uma representação** do recurso especificado na URL. É o método mais utilizado da Web, sendo *safe*, **idempotente** e **cacheável**.

**Para que serve:** carregar páginas HTML, imagens, folhas CSS, scripts JS, baixar arquivos e consultar dados em APIs REST (listagens, buscas, filtros e detalhes de itens).

**Como funciona:** os parâmetros de filtro ou busca são passados no próprio caminho (*path parameters*, ex.: `/usuarios/42`) ou na *query string* (`?categoria=livros&pagina=2`). Pela RFC 9110, uma requisição `GET` **não deve conter corpo (*body*)** — embora não seja proibido sintaticamente no parser, a RFC define que um payload em `GET` não tem semântica definida e proxies, WAFs, CDNs e a própria API `fetch()` do navegador podem rejeitá-lo ou descartá-lo.

**Exemplo prático (HTTP bruto e `fetch()`):**

```http
GET /v1/produtos?categoria=eletronicos&ordem=preco_asc HTTP/1.1
Host: api.loja-ficticia.test
Accept: application/json
Cookie: __Host-sid=9f8e7d6c5b4a
```

```javascript
// Consulta via fetch() (GET é o método padrão quando omitido)
const url = new URL("https://api.loja-ficticia.test/v1/produtos");
url.searchParams.set("categoria", "eletronicos");
url.searchParams.set("ordem", "preco_asc");

const resp = await fetch(url, {
  method: "GET",
  credentials: "include",
  headers: { "Accept": "application/json" }
});
const dados = await resp.json();
```

**Possíveis problemas de segurança:**

1. **Ações de escrita/alteração de estado via `GET` (violação de método *safe*):** criar endpoints como `GET /usuarios/42/excluir` ou `GET /carrinho/finalizar` é um erro grave. Primeiro, extensões de pré-carregamento (*link prefetching* / *Speculation Rules* do navegador), antivírus corporativos e robôs de busca seguem links `GET` automaticamente e acabarão disparando a exclusão. Segundo, torna a aplicação trivialmente vulnerável a **CSRF (Cross-Site Request Forgery)** mesmo com `SameSite=Lax`, pois basta um atacante inserir `<img src="https://loja-ficticia.test/usuarios/42/excluir">` ou um link em outro site para executar a ação na sessão da vítima.
2. **Envio de senhas, tokens ou dados pessoais (CPF, cartão) em requisições `GET`:** como o `GET` usa a URL, tudo na *query string* fica gravado em texto claro no **histórico do navegador**, nos **logs de acesso** do servidor Web (Nginx/Apache), nos **logs de proxies corporativos e CDNs**, na aba **Network** e pode vazar para sites de terceiros através do header **`Referer`** quando o usuário clica em um link externo.

**Configuração segura:**

- **[REQUISITO]** Garantir que endpoints `GET` sejam estritamente **somente leitura** (*side-effect free* para o estado de negócio). Qualquer criação, edição, exclusão, login ou logout deve exigir `POST`, `PUT`, `PATCH` ou `DELETE`.
- **[REQUISITO]** Jamais trafegar senhas, tokens de autenticação, chaves de API ou dados pessoais sensíveis (PII — *Personally Identifiable Information*) na *query string* de uma requisição `GET`.
- **[RECOMENDAÇÃO]** Validar e limitar o tamanho de parâmetros de paginação (`?limit=50`) no servidor para evitar que um `GET /v1/produtos?limit=99999999` cause negação de serviço (DoS) no banco de dados.

<a id="s1-4-2"></a>
#### 1.4.2 POST

**O que é:** o método **`POST`** solicita que o recurso de destino **processe a representação enviada no corpo da requisição** de acordo com a semântica própria daquele recurso. **Não é *safe*** e **não é idempotente**.

**Para que serve:** criar novos recursos (onde o servidor gera o identificador, ex.: `POST /v1/pedidos` → cria o pedido `#901`), submeter formulários HTML de login/cadastro, fazer upload de arquivos (`multipart/form-data`), disparar operações de negócio complexas (processar um pagamento, enviar um e-mail) ou realizar buscas complexas cujos filtros não cabem ou são sensíveis demais para a URL.

**Como funciona:** os dados são enviados no **corpo da requisição** (*request body*), cujo formato é indicado pelo header `Content-Type` — tipicamente `application/json`, `application/x-www-form-urlencoded` ou `multipart/form-data`. Quando um `POST` resulta na criação de um novo recurso, a resposta ideal é **`201 Created`** acompanhada do header `Location: /v1/pedidos/901`.

**Exemplo prático (HTTP bruto e `fetch()`):**

```http
POST /v1/pedidos HTTP/1.1
Host: api.loja-ficticia.test
Content-Type: application/json
Accept: application/json
Cookie: __Host-sid=9f8e7d6c5b4a
X-CSRF-Token: 4b8e2f1a9c0d7e6f5a4b3c2d1e0f9a8b
Content-Length: 58

{"itens":[{"produtoId":42,"quantidade":2}],"cupom":"DEZ"}
```

```javascript
const resp = await fetch("https://api.loja-ficticia.test/v1/pedidos", {
  method: "POST",
  credentials: "include",
  headers: {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "X-CSRF-Token": obterTokenCsrf()
  },
  body: JSON.stringify({
    itens: [{ produtoId: 42, quantidade: 2 }],
    cupom: "DEZ"
  })
});
```

**Possíveis problemas de segurança:**

1. **CSRF (*Cross-Site Request Forgery*):** formulários HTML clássicos (`<form method="POST">`) com `Content-Type` simples (`application/x-www-form-urlencoded`, `multipart/form-data` ou `text/plain`) podem ser submetidos automaticamente a partir de uma página maliciosa de terceiro sem passar por *preflight* CORS; se a sessão depender apenas de cookies sem proteção anti-CSRF, a ação será executada em nome da vítima.
2. **Mass Assignment (Atribuição em Massa / *Over-posting*):** se o back-end pega o JSON do `POST /v1/usuarios` e o repassa diretamente para o modelo do banco de dados (`Usuario.create(req.body)`), um atacante pode inspecionar a API no DevTools e adicionar campos extras no JSON, como `{"nome":"Ana", "role":"admin", "saldo":999999}`, escalando privilégios.
3. **Duplicidade por falta de idempotência:** se o usuário clicar duas vezes seguidas em "Pagar" ou a rede oscilar, duas requisições `POST` idênticas podem gerar cobrança dupla.

**Configuração segura:**

- **[REQUISITO]** Proteger endpoints `POST` autenticados por cookie contra **CSRF** (usando tokens anti-CSRF sincronizados, cookies `SameSite=Lax`/`Strict` e exigência de `Content-Type: application/json` validado no servidor).
- **[REQUISITO]** Usar listas de permissão explícitas de campos aceitos (*allowlist* / DTOs / *schemas* como Zod, Pydantic ou Bean Validation) para impedir **Mass Assignment**.
- **[RECOMENDAÇÃO]** Em operações financeiras ou críticas via `POST`, suportar uma chave de idempotência no cabeçalho (ex.: `Idempotency-Key: <UUID-v4>`) para que o servidor detecte e ignore retransmissões acidentais da mesma operação.

<a id="s1-4-3"></a>
#### 1.4.3 PUT

**O que é:** o método **`PUT`** solicita que o estado do recurso identificado pela URL seja **substituído integralmente** (ou criado naquela URL exata, se ainda não existir) pela representação enviada no corpo da requisição. **Não é *safe***, mas **é idempotente** (enviar o mesmo documento completo 1 ou 5 vezes deixa o recurso exatamente com aquele estado final).

**Para que serve:** substituir por completo um recurso cujo endereço exato o cliente já conhece (ex.: atualizar todos os dados cadastrais de `/v1/enderecos/15`, ou fazer upload de um arquivo para um caminho conhecido em um bucket de armazenamento como `PUT /arquivos/relatorio-2026.pdf`).

**Como funciona:** diferente do `PATCH` (que envia apenas os campos alterados), a semântica estrita do `PUT` exige que o cliente envie a **representação completa** do recurso. Se um campo opcional for omitido no `PUT`, o servidor deve interpretá-lo como removido/nulo. Se o recurso já existia e foi substituído, retorna-se `200 OK` ou `204 No Content`; se foi criado naquele URI, retorna-se `201 Created`.

**Exemplo prático (HTTP bruto e `fetch()`):**

```http
PUT /v1/enderecos/15 HTTP/1.1
Host: api.loja-ficticia.test
Content-Type: application/json
Cookie: __Host-sid=9f8e7d6c5b4a
Content-Length: 96

{"rua":"Av. Paulista, 1000","cidade":"Sao Paulo","uf":"SP","cep":"01310-100","principal":true}
```

```javascript
await fetch("https://api.loja-ficticia.test/v1/enderecos/15", {
  method: "PUT",
  credentials: "include",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    rua: "Av. Paulista, 1000",
    cidade: "Sao Paulo",
    uf: "SP",
    cep: "01310-100",
    principal: true
  })
});
```

**Possíveis problemas de segurança:**

1. **IDOR / BOLA (*Broken Object Level Authorization*):** o usuário autenticado `id=10` altera o ID na URL para `PUT /v1/enderecos/15` (que pertence ao usuário `id=99`). Se o back-end atualizar o endereço `15` sem verificar se `endereco.usuario_id == usuario_logado.id`, qualquer usuário poderá sobrescrever dados de terceiros.
2. **Método `PUT` habilitado no sistema de arquivos do servidor Web (WebDAV / upload arbitrário):** em servidores Web legados mal configurados, permitir `PUT` diretamente sobre o diretório público permitia que um atacante enviasse `PUT /shell.php` ou `PUT /pagina.html`, gravando código malicioso no servidor.
3. **Perda acidental de dados (*Lost Update*) e Mass Assignment:** sobrescrever campos que outro processo acabou de atualizar ou aceitar campos restritos (como `id`, `criadoEm`, `aprovado`).

**Configuração segura:**

- **[REQUISITO]** Validar no servidor a **autorização em nível de objeto (BOLA/IDOR)**: confirmar que o recurso apontado na URL pertence ao usuário autenticado antes de aplicar o `PUT`.
- **[REQUISITO]** Desabilitar o método `PUT` em diretórios estáticos do servidor Web, permitindo-o apenas nas rotas específicas da API de aplicação e com validação de *schema* (DTO).
- **[RECOMENDAÇÃO]** Utilizar cabeçalhos condicionais como `If-Match: "etag-atual"` para controle de concorrência otimista (retornando `412 Precondition Failed` se outra pessoa tiver alterado o registro antes).

<a id="s1-4-4"></a>
#### 1.4.4 PATCH

**O que é:** o método **`PATCH`** (introduzido pela RFC 5789) solicita a aplicação de **modificações parciais** a um recurso. **Não é *safe*** e **não é obrigatoriamente idempotente**.

**Para que serve:** atualizar **apenas um ou alguns campos** de um recurso sem precisar trafegar e substituir o objeto inteiro (ex.: alterar somente o `telefone` do perfil do usuário ou mudar o `status` de um pedido para `"enviado"`).

**Como funciona:** o corpo da requisição contém apenas o conjunto de alterações (*patch document*), que pode ser um JSON parcial com os campos a mesclar (**JSON Merge Patch**, RFC 7396 — `application/merge-patch+json`, ou `application/json` comum na maioria das APIs REST) ou uma lista de operações sequenciais (**JSON Patch**, RFC 6902 — `application/json-patch+json`, como `[{"op": "replace", "path": "/telefone", "value": "11999990000"}]`).
- *Por que o `PATCH` não é sempre idempotente?* Na maioria das vezes (`{"telefone": "11999990000"}`), repeti-lo produz o mesmo resultado; porém, um documento `PATCH` também pode descrever uma operação relativa como `[{"op": "add", "path": "/tentativasLogin", "value": 1}]` (incrementar contador ou anexar elemento a uma lista), cujo efeito muda a cada repetição.

**Exemplo prático (HTTP bruto e `fetch()`):**

```http
PATCH /v1/usuarios/me HTTP/1.1
Host: api.loja-ficticia.test
Content-Type: application/json
Cookie: __Host-sid=9f8e7d6c5b4a
Content-Length: 31

{"telefone":"+5511999990000"}
```

```javascript
await fetch("https://api.loja-ficticia.test/v1/usuarios/me", {
  method: "PATCH",
  credentials: "include",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ telefone: "+5511999990000" })
});
```

**Possíveis problemas de segurança:**

1. **Broken Object Property Level Authorization (BOPLA / Mass Assignment em atualizações parciais):** como o `PATCH` costuma usar funções genéricas de mesclagem no back-end (ex.: `Object.assign(usuario, req.body)` ou `findByIdAndUpdate(id, req.body)`), é extremamente comum que o desenvolvedor pense apenas no campo `telefone` exibido na tela e esqueça que o atacante pode enviar `{"telefone": "...", "isAdmin": true, "emailVerificado": true}` na mesma requisição `PATCH`.
2. **Prototype Pollution (Poluição de Protótipo em JavaScript/Node.js):** funções recursivas de *deep merge* inseguras ao processar payloads `PATCH` contendo chaves como `"__proto__": {"isAdmin": true}` ou `"constructor": {"prototype": ...}` podem contaminar o protótipo global de objetos da aplicação.

**Configuração segura:**

- **[REQUISITO]** Filtrar estritamente quais propriedades do objeto podem ser modificadas por aquele perfil de usuário (ex.: permitir apenas `nome` e `telefone` no schema do `PATCH /v1/usuarios/me`), rejeitando qualquer propriedade desconhecida ou administrativa.
- **[REQUISITO]** Rejeitar chaves especiais como `__proto__`, `constructor` e `prototype` em parsers e rotinas de mesclagem de objetos JSON.

<a id="s1-4-5"></a>
#### 1.4.5 DELETE

**O que é:** o método **`DELETE`** solicita a **remoção** da associação entre a URL informada e o recurso atual (na prática de APIs REST, exclui ou inativa o registro). **Não é *safe***, mas **é idempotente**.

**Para que serve:** remover um item específico identificado na URL (ex.: `DELETE /v1/carrinho/itens/88` ou `DELETE /v1/sessoes/atual` para fazer logout).

**Como funciona:** o identificador do recurso a ser removido vai na própria URL (`/v1/comentarios/305`). Requisições `DELETE` normalmente **não possuem corpo**. O servidor processa a exclusão e responde com `204 No Content` (sem corpo), `200 OK` (com uma confirmação JSON) ou `202 Accepted` (quando a exclusão foi agendada para processamento assíncrono em fila).

**Exemplo prático (HTTP bruto e `fetch()`):**

```http
DELETE /v1/carrinho/itens/88 HTTP/1.1
Host: api.loja-ficticia.test
Cookie: __Host-sid=9f8e7d6c5b4a
X-CSRF-Token: 4b8e2f1a9c0d7e6f5a4b3c2d1e0f9a8b
```

```javascript
const resp = await fetch("https://api.loja-ficticia.test/v1/carrinho/itens/88", {
  method: "DELETE",
  credentials: "include",
  headers: { "X-CSRF-Token": obterTokenCsrf() }
});
// resp.status === 204 (No Content)
```

**Possíveis problemas de segurança:**

1. **Falta de autorização no servidor (IDOR / Broken Access Control em `DELETE`):** o atacante autenticado faz um loop de `DELETE /v1/comentarios/1` até `DELETE /v1/comentarios/10000`. Se o back-end apenas verificar "o usuário está logado?" em vez de verificar "este usuário é o dono do comentário ou é moderador?", todo o banco de dados será apagado.
2. **Ausência de trilha de auditoria em exclusões físicas (`HARD DELETE`):** apagar fisicamente registros sensíveis sem registrar *quem*, *quando* e *de qual IP* solicitou o `DELETE` impede investigações forenses.

**Configuração segura:**

- **[REQUISITO]** Verificar autenticação **e** autorização granular (propriedade do objeto e papel RBAC/ABAC) no back-end antes de executar qualquer `DELETE`.
- **[RECOMENDAÇÃO]** Adotar exclusão lógica (*soft delete*, preenchendo uma coluna `deleted_at`) para registros críticos de negócio e gravar log de auditoria imutável da operação.

<a id="s1-4-6"></a>
#### 1.4.6 OPTIONS

**O que é:** o método **`OPTIONS`** solicita informações sobre as **opções de comunicação disponíveis** para o recurso de destino (ou para o servidor como um todo, usando `OPTIONS * HTTP/1.1`). É um método ***safe*** e **idempotente**, mas **não é cacheável** pelo cache HTTP padrão (embora o resultado de *preflight* CORS tenha seu próprio cache controlado por `Access-Control-Max-Age`).

**Para que serve:** possui dois usos principais na Web:
1. **Descoberta de métodos suportados:** o servidor responde com o header `Allow: GET, HEAD, POST, OPTIONS`.
2. **Requisição de *Preflight* do CORS (*Cross-Origin Resource Sharing*):** antes de o navegador permitir que um script em `https://app.exemplo.test` envie uma requisição "não simples" (como um `DELETE`, `PUT`, `PATCH` ou um `POST` com `Content-Type: application/json` ou header `Authorization`) para outra origem (`https://api.loja-ficticia.test`), o próprio navegador dispara automaticamente uma requisição **`OPTIONS`** perguntando ao servidor de destino se aquela origem, aquele método e aqueles headers são permitidos (veja [seção 1.11 — CORS](#s1) e [seção 5.10](./05-headers-de-seguranca.md#s5)).

**Como funciona:** na requisição de *preflight* CORS, o navegador envia `OPTIONS` com os cabeçalhos `Origin`, `Access-Control-Request-Method` e `Access-Control-Request-Headers`. O servidor deve responder sem executar nenhuma lógica de negócio (geralmente com `204 No Content` ou `200 OK`) contendo os cabeçalhos `Access-Control-Allow-*`.

**Exemplo prático (HTTP bruto de Preflight CORS e chamada manual com `fetch()`):**

```http
OPTIONS /v1/pedidos HTTP/1.1
Host: api.loja-ficticia.test
Origin: https://painel.loja-ficticia.test
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type, x-csrf-token
```

Resposta do servidor:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://painel.loja-ficticia.test
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type, X-CSRF-Token
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 600
Vary: Origin
```

```javascript
// Consulta explícita de métodos suportados via fetch()
const resp = await fetch("https://api.loja-ficticia.test/v1/pedidos", {
  method: "OPTIONS"
});
console.log(resp.headers.get("Allow")); // "GET, POST, OPTIONS"
```

**Possíveis problemas de segurança:**

1. **CORS permissivo demais no `OPTIONS`:** responder ao `OPTIONS` refletindo cegamente qualquer header `Origin` recebido junto com `Access-Control-Allow-Credentials: true` abre uma vulnerabilidade crítica que permite a sites maliciosos lerem dados autenticados dos usuários.
2. **Execução acidental de lógica ou exigência indevida de autenticação no `OPTIONS`:** segundo o padrão Fetch/CORS, o navegador **não envia cookies nem header `Authorization` na requisição de preflight `OPTIONS`**. Se o desenvolvedor exigir autenticação antes de responder ao `OPTIONS`, o *preflight* falhará com `401` e o CORS quebrará — o que às vezes leva o desenvolvedor desavisado a desativar a autenticação na rota inteira (para todos os métodos!), criando uma falha gravíssima.

**Configuração segura:**

- **[REQUISITO]** No tratamento de *preflight* (`OPTIONS`), validar o header `Origin` contra uma **lista fechada de origens confiáveis** (*allowlist*) antes de emitir `Access-Control-Allow-Origin`.
- **[REQUISITO]** Responder ao método `OPTIONS` no middleware de CORS **antes** do middleware de autenticação (retornando `204 No Content` sem executar consultas de negócio), mantendo a autenticação obrigatória para `GET`, `POST`, `PUT`, `PATCH` e `DELETE`.
- **[BOA PRÁTICA]** Configurar `Access-Control-Max-Age` (ex.: `600` a `86400` segundos) para que o navegador armazene a permissão do *preflight* em cache e não precise enviar um `OPTIONS` extra antes de cada chamada à API.

<a id="s1-4-7"></a>
#### 1.4.7 HEAD

**O que é:** o método **`HEAD`** é idêntico ao método `GET`, com uma única regra fundamental: **o servidor deve devolver exatamente os mesmos cabeçalhos HTTP que devolveria para um `GET`, mas NÃO deve enviar o corpo da resposta (*response body*)**. É ***safe***, **idempotente** e **cacheável**.

**Para que serve:** verificar se um link ou recurso existe (`200` vs `404`), descobrir o tamanho de um arquivo grande antes de baixá-lo (`Content-Length`), checar o tipo de mídia (`Content-Type`) ou verificar se a versão em cache mudou (`ETag` / `Last-Modified`) sem gastar banda transferindo megabytes de conteúdo. Também é muito utilizado por balanceadores de carga e monitores de disponibilidade (*health checks*).

**Como funciona:** o cliente envia `HEAD /arquivos/instalador.iso HTTP/1.1`. O servidor calcula os metadados e responde com `HTTP/1.1 200 OK`, `Content-Length: 4294967296` (4 GB), `Content-Type: application/octet-stream` e encerra a resposta imediatamente após a linha em branco (`\r\n\r\n`), transferindo 0 bytes de corpo.

**Exemplo prático (HTTP bruto e `fetch()`):**

```http
HEAD /downloads/relatorio-anual.pdf HTTP/1.1
Host: loja-ficticia.test
```

```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Length: 15728640
Last-Modified: Thu, 29 Oct 2026 18:00:00 GMT
ETag: "f00baa-624000"
Accept-Ranges: bytes
```

```javascript
// Descobrindo o tamanho de um PDF sem baixá-lo
const resp = await fetch("https://loja-ficticia.test/downloads/relatorio-anual.pdf", {
  method: "HEAD"
});
if (resp.ok) {
  const bytes = Number(resp.headers.get("Content-Length"));
  console.log(`O arquivo possui ${(bytes / 1048576).toFixed(2)} MB`);
}
```

**Possíveis problemas de segurança:**

1. **Bypass de controle de acesso por método (*Method-Based Access Control Bypass*):** em configurações incorretas de servidores Web ou proxies (onde o desenvolvedor restringe apenas `<Limit GET POST>` no Apache ou valida `if (req.method === 'GET')` em um middleware customizado), um atacante que envia `HEAD` para uma rota restrita pode passar direto pela checagem; se a rota desencadear efeitos colaterais ou vazar informações em cabeçalhos customizados ou pelo tempo de resposta (*timing side-channel*), ocorre vazamento de dados.

**Configuração segura:**

- **[REQUISITO]** Garantir que todas as regras de autenticação e autorização aplicadas ao método `GET` sejam igualmente aplicadas ao método `HEAD` (frameworks modernos como Express, Django e Spring já tratam `HEAD` automaticamente a partir das rotas `GET`).

<a id="s1-4-8"></a>
#### 1.4.8 Outros métodos (CONNECT, TRACE) e tabela comparativa

Além dos sete métodos principais usados no desenvolvimento diário, a RFC 9110 define mais dois métodos padronizados cujo funcionamento e riscos de segurança são frequentemente cobrados em provas de Cybersecurity e Redes:

##### Método `CONNECT`

- **O que é e para que serve:** o método **`CONNECT`** solicita que um **proxy HTTP intermediário** estabeleça um **túnel TCP/IP bidirecional transparente** até o servidor de destino informado na linha de requisição (`CONNECT loja-ficticia.test:443 HTTP/1.1`).
- **Como funciona:** quando um navegador está configurado para usar um *forward proxy* corporativo e precisa acessar um site **HTTPS**, o navegador não pode simplesmente mandar a requisição HTTP para o proxy ler, pois o tráfego precisa ser criptografado de ponta a ponta entre o navegador e o servidor final. Então o navegador envia `CONNECT loja-ficticia.test:443 HTTP/1.1` ao proxy; o proxy abre uma conexão TCP até `loja-ficticia.test:443`, responde `HTTP/1.1 200 Connection Established` ao navegador e, a partir desse instante, apenas repassa os bytes criptografados do TLS de um lado para o outro como um túnel cego. Não é *safe*, não é idempotente e não é cacheável.
- **Problema de segurança e configuração segura:** se um servidor Web ou proxy reverso aceitar requisições `CONNECT` vindas da Internet aberta (configuração de *Open Proxy*), atacantes poderão usá-lo como trampolim para ocultar a origem de ataques externos ou para **invadir a rede interna da empresa** (`CONNECT 10.0.0.5:22 HTTP/1.1` ou `CONNECT 127.0.0.1:6379 HTTP/1.1`).
  - **[REQUISITO]** Desabilitar completamente o método `CONNECT` em servidores Web de origem e proxies reversos; em *forward proxies* corporativos, exigir autenticação e permitir `CONNECT` apenas da rede interna para portas autorizadas (como `443`).

##### Método `TRACE`

- **O que é e para que serve:** o método **`TRACE`** executa um teste de diagnóstico (*loop-back*) da mensagem: o servidor final devolve ao cliente, no corpo de uma resposta `200 OK` com `Content-Type: message/http`, **o texto exato da requisição HTTP que chegou até ele** (permitindo ver se proxies intermediários adicionaram ou modificaram cabeçalhos no caminho). É *safe* e idempotente, mas não cacheável.
- **Problema de segurança (XST — *Cross-Site Tracing*):** como o `TRACE` ecoa no corpo da resposta todos os cabeçalhos recebidos pelo servidor — incluindo cabeçalhos sensíveis anexados automaticamente como `Cookie` (mesmo com a flag `HttpOnly`!) e `Authorization` —, historicamente um ataque de XSS combinado com `TRACE` (conhecido como **XST**) permitia que um script malicioso lesse cookies `HttpOnly` refletidos na resposta do `TRACE`. Navegadores modernos hoje bloqueiam o método `TRACE` nas APIs `fetch()` e `XMLHttpRequest` (**Forbidden method** no Fetch Standard), mas servidores com `TRACE` ativo ainda podem expor cabeçalhos internos adicionados por proxies reversos (como chaves internas de API ou cabeçalhos de autenticação de borda).
  - **[REQUISITO]** Manter o método **`TRACE` desabilitado** em todos os servidores Web de produção (ex.: `TraceEnable off` no Apache; o Nginx já rejeita `TRACE` com `405 Not Allowed` por padrão).

##### Tabela comparativa completa dos métodos HTTP (RFC 9110 e RFC 5789)

| Método | *Safe* (Somente leitura)? | Idempotente? | Cacheável? | Corpo na requisição (*Request Body*)? | Corpo na resposta (*Response Body*)? | Uso principal na Web e em APIs REST |
|---|---|---|---|---|---|---|
| **`GET`** | **Sim** | **Sim** | **Sim** | Não (sem semântica definida; deve ser evitado) | Sim | Buscar documentos, ativos estáticos e consultar recursos sem alterar estado |
| **`HEAD`** | **Sim** | **Sim** | **Sim** | Não | **Não** (proibido pela RFC; retorna apenas os headers) | Obter metadados (`Content-Length`, `ETag`, status) sem baixar o corpo |
| **`OPTIONS`** | **Sim** | **Sim** | Não* | Opcional (raro) | Opcional | Descobrir métodos suportados (`Allow`) e executar *Preflight* de **CORS** |
| **`TRACE`** | **Sim** | **Sim** | Não | Não | Sim (`message/http` ecoando a requisição) | Diagnóstico de caminho HTTP (**deve ser desabilitado em produção** por risco de XST) |
| **`PUT`** | Não | **Sim** | Não | **Sim** (representação completa) | Opcional (`200 OK` com corpo ou `204 No Content`) | **Substituir integralmente** um recurso existente (ou criá-lo em URI conhecido) |
| **`DELETE`** | Não | **Sim** | Não | Opcional (na prática não utilizado) | Opcional (`204 No Content` ou `200 OK`) | **Remover** o recurso identificado pela URL |
| **`POST`** | Não | **Não** | Raro** | **Sim** | Sim (`201 Created`, `200 OK`, etc.) | **Criar** recursos subordinados, submeter formulários, uploads e processar ações |
| **`PATCH`** | Não | **Não** | Raro** | **Sim** (documento de alterações) | Sim (`200 OK` ou `204 No Content`) | **Atualizar parcialmente** campos específicos de um recurso (RFC 5789) |
| **`CONNECT`** | Não | **Não** | Não | Não no início (depois vira túnel TCP bruto) | Resposta `200` inicial seguida do túnel | Abrir túnel TCP (geralmente para HTTPS na porta 443) através de *forward proxy* |

*\*Nota sobre `OPTIONS`: não é cacheável pelo cache HTTP geral, mas navegadores armazenam o resultado do preflight no cache dedicado de CORS conforme `Access-Control-Max-Age`.*
*\*\*Nota sobre `POST` e `PATCH`: a RFC permite cacheamento apenas se houver cabeçalhos explícitos de frescor e `Content-Location`, mas navegadores e CDNs não fazem cache de `POST`/`PATCH` por padrão.*

<a id="s1-5"></a>
### 1.5 Códigos de Status HTTP

Toda resposta HTTP inicia com um **código de status de 3 dígitos** (RFC 9110, seção 15), que informa ao cliente o resultado da tentativa do servidor de compreender e atender à requisição. Na aba **Network** do Chrome DevTools ([seção 2.1](./02a-devtools-network.md#s2-1)), a coluna **Status** é o primeiro indicador de diagnóstico de funcionamento e de anomalias de segurança.

<a id="s1-5-1"></a>
#### 1.5.1 Classes de status (1xx a 5xx)

O **primeiro dígito** do código define a **classe semântica** da resposta. Os dois últimos dígitos não têm agrupamento matemático formal, servindo apenas para distinguir cada situação específica dentro da classe:

| Classe | Nome da Classe | Significado Técnico (RFC 9110) | Responsabilidade Primária | Exemplos Clássicos |
|---|---|---|---|---|
| **`1xx`** | **Informacional** (*Informational*) | A requisição foi recebida e o processamento continua. São respostas **provisórias** (consistem apenas na linha de status e headers opcionais, **nunca têm corpo**) enviadas antes da resposta final | Coordenação de protocolo entre cliente e servidor | `100 Continue`, `101 Switching Protocols`, `103 Early Hints` |
| **`2xx`** | **Sucesso** (*Successful*) | A requisição foi recebida, compreendida e aceita com sucesso pelo servidor | Operação concluída normalmente | `200 OK`, `201 Created`, `204 No Content`, `206 Partial Content` |
| **`3xx`** | **Redirecionamento** (*Redirection*) | Uma ação adicional precisa ser tomada pelo agente do usuário (navegador) para completar a requisição — seja seguir para uma nova URL (`Location`) ou reutilizar uma cópia local em cache | Navegador / Cache | `301 Moved Permanently`, `302 Found`, `304 Not Modified`, `307 Temporary Redirect`, `308 Permanent Redirect` |
| **`4xx`** | **Erro do Cliente** (*Client Error*) | A requisição contém sintaxe inválida, parâmetros incorretos, falta de autenticação, falta de permissão, aponta para um recurso inexistente ou excede limites de taxa | **Cliente** (exceto se o servidor estiver mal configurado) | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Content`, `429 Too Many Requests` |
| **`5xx`** | **Erro do Servidor** (*Server Error*) | O servidor falhou ao cumprir uma requisição aparentemente válida (exceção não tratada no código, banco de dados fora do ar, falha ou timeout no proxy reverso/gateway) | **Servidor / Infraestrutura de Back-end** | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout` |

> [!TIP]
> **Regra de bolso para provas e depuração:**
> - Deu **`4xx`**? O servidor está dizendo *"A culpa é sua (cliente): você montou a requisição errada, não se identificou, não tem permissão ou pediu algo que não existe"*.
> - Deu **`5xx`**? O servidor está dizendo *"A culpa é minha (servidor): você fez um pedido válido, mas meu código quebrou, meu banco caiu ou o servidor de aplicação atrás do proxy não respondeu a tempo"*.

<a id="s1-5-2"></a>
#### 1.5.2 Códigos de status detalhados

A seguir, analisamos individualmente os **27 códigos de status HTTP** mais importantes para provas acadêmicas, desenvolvimento de APIs e auditorias de segurança.

##### `100 Continue`

- **O que é e quando ocorre:** resposta provisória que indica que a parte inicial da requisição (os cabeçalhos) foi recebida e ainda não foi rejeitada pelo servidor, autorizando o cliente a prosseguir enviando o corpo da requisição. Ocorre quando um cliente precisa fazer upload de um corpo muito grande (ex.: um vídeo de 500 MB) e quer saber **antes de gastar banda enviando os bytes** se o servidor vai aceitar a requisição (validando autenticação, tamanho `Content-Length` e tipo `Content-Type`). Para isso, o cliente envia nos headers `Expect: 100-continue`. Se estiver tudo certo, o servidor responde `HTTP/1.1 100 Continue` e só então o cliente transmite o corpo; se o token for inválido ou o arquivo grande demais, o servidor já responde direto com `401` ou `413` sem receber os 500 MB.
- **Exemplo HTTP:**
  ```http
  # 1. Cliente envia apenas os headers com Expect: 100-continue
  PUT /v1/videos/aula01.mp4 HTTP/1.1
  Host: api.loja-ficticia.test
  Authorization: Bearer eyJhbGci...
  Content-Type: video/mp4
  Content-Length: 524288000
  Expect: 100-continue

  # 2. Servidor aprova os headers preliminarmente
  HTTP/1.1 100 Continue

  # 3. Cliente envia os 500 MB do corpo e recebe a resposta final:
  HTTP/1.1 201 Created
  Location: /v1/videos/aula01.mp4
  ```
- **Implicações de segurança e configuração segura:** **[RECOMENDAÇÃO]** Validar autenticação e o limite máximo de `Content-Length` **antes** de emitir `100 Continue`, evitando que clientes não autenticados consumam banda e buffers de entrada do servidor com uploads gigantes.

##### `101 Switching Protocols`

- **O que é e quando ocorre:** confirma que o servidor entendeu e aceitou o pedido do cliente (feito via cabeçalho `Upgrade`) para **trocar o protocolo de aplicação** que está rodando sobre aquela conexão TCP. O caso mais clássico na Web moderna é o handshake de abertura de **WebSockets** (RFC 6455): a conversa começa como uma requisição HTTP/1.1 comum (`GET /chat` com `Connection: Upgrade` e `Upgrade: websocket`) e, após o servidor responder `101 Switching Protocols`, aquela conexão TCP deixa de falar HTTP e passa a trafegar quadros (*frames*) binários/texto bidirecionais de WebSocket. No Chrome DevTools (aba *Network* → filtro *WS*), toda conexão WebSocket ativa aparece com status **`101`**.
- **Exemplo HTTP:**
  ```http
  GET /ws/notificacoes HTTP/1.1
  Host: api.loja-ficticia.test
  Origin: https://loja-ficticia.test
  Connection: Upgrade
  Upgrade: websocket
  Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
  Sec-WebSocket-Version: 13

  HTTP/1.1 101 Switching Protocols
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
  ```
- **Implicações de segurança e configuração segura:** conexões WebSocket **não são protegidas pela Same-Origin Policy / CORS padrão da mesma forma que `fetch()`**; um site malicioso consegue tentar abrir um WebSocket para `wss://api.loja-ficticia.test/ws` e o navegador anexará os cookies! Esse ataque é chamado de **CSWSH (*Cross-Site WebSocket Hijacking*)**.
  - **[REQUISITO]** Antes de responder `101 Switching Protocols`, o servidor WebSocket **deve validar estritamente o cabeçalho `Origin`** contra a lista de domínios confiáveis e verificar a autenticação/sessão do usuário.

##### `200 OK`

- **O que é e quando ocorre:** indica que a requisição foi bem-sucedida. O significado exato do payload retornado depende do método usado: em `GET`, o corpo traz a representação do recurso pedido; em `HEAD`, traz apenas os cabeçalhos sem corpo; em `POST`, `PUT` ou `PATCH`, traz a representação do resultado do processamento ou o estado atualizado do recurso.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/json; charset=utf-8
  Cache-Control: no-store

  {"id":42,"nome":"Notebook Pro","preco":5499.90}
  ```
- **Implicações de segurança e configuração segura:** uma armadilha comum (e anti-padrão arquitetural) é retornar **`200 OK` para erros de autenticação ou de servidor** (ex.: `HTTP/1.1 200 OK` com corpo `{"sucesso": false, "erro": "Não autorizado"}` ou páginas de erro amigáveis que devolvem status `200` — o chamado *Soft 404* / *Soft 500*). Isso engana sistemas de monitoramento, WAFs, caches (que podem armazenar em cache uma página de erro achando que é um `200 OK` válido!) e ferramentas de auditoria.
  - **[RECOMENDAÇÃO]** Usar `200 OK` apenas quando a operação realmente teve sucesso, reservando códigos `4xx` e `5xx` reais para falhas.

##### `201 Created`

- **O que é e quando ocorre:** indica que a requisição foi bem-sucedida e resultou na **criação de um ou mais novos recursos** no servidor (típico após um `POST /v1/usuarios` ou um `PUT` que criou um recurso novo). Pela RFC 9110, a resposta `201 Created` deve apontar o endereço do novo recurso criado no cabeçalho **`Location`** e, normalmente, traz no corpo a representação do item recém-criado.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 201 Created
  Location: https://api.loja-ficticia.test/v1/usuarios/704
  Content-Type: application/json; charset=utf-8

  {"id":704,"nome":"Carlos Mendes","email":"carlos@exemplo.test"}
  ```
- **Implicações de segurança e configuração segura:** ao retornar no corpo do `201 Created` o objeto recém-salvo no banco de dados, muitos frameworks serializam a entidade inteira da tabela — **vazando o hash da senha recém-criada (`password_hash`), tokens internos de ativação de conta ou flags administrativas** (*Excessive Data Exposure*).
  - **[REQUISITO]** Nunca serializar a entidade bruta do banco de dados na resposta do `201 Created`; utilizar um DTO de saída que omita hashes de senha e segredos internos.

##### `204 No Content`

- **O que é e quando ocorre:** indica que o servidor processou a requisição com **sucesso**, mas **não há conteúdo (corpo) adicional a enviar** na resposta. É o código padrão ideal para:
  1. Exclusões bem-sucedidas (`DELETE /v1/itens/88`).
  2. Atualizações (`PUT` / `PATCH`) de "salvar rascunho" onde o cliente na tela já possui os dados atualizados e só precisa da confirmação de que foi salvo.
  3. Respostas a requisições de *preflight* CORS (`OPTIONS`).
  4. Endpoints de telemetria/beacon (`POST /v1/metricas`).
  - *Regra sintática da RFC 9110:* uma resposta `204 No Content` **nunca deve conter corpo de mensagem** (termina na primeira linha em branco após os cabeçalhos).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 204 No Content
  Date: Fri, 30 Oct 2026 10:25:00 GMT
  ```
- **Implicações de segurança e configuração segura:** **[BOA PRÁTICA]** No front-end, ao consumir APIs com `fetch()`, verificar se `resp.status === 204` antes de chamar `await resp.json()`, pois chamar `resp.json()` em um corpo vazio de `204` lança `SyntaxError: Unexpected end of JSON input` no Console.

##### `206 Partial Content`

- **O que é e quando ocorre:** indica que o servidor está cumprindo com sucesso uma **requisição de intervalo (*Range Request*, RFC 9110 seção 14)** para um recurso, entregando apenas a fatia de bytes solicitada pelo cliente no cabeçalho `Range` (ex.: `Range: bytes=0-1048575`). É o mecanismo que permite:
  1. **Streaming de vídeo e áudio HTML5 (`<video>`, `<audio>`):** o usuário pode pular para o minuto 45 de um filme sem precisar baixar os 44 minutos anteriores.
  2. **Pausar e retomar downloads grandes** no navegador sem começar do zero.
  3. **Downloads paralelos** em múltiplos blocos.
- **Exemplo HTTP:**
  ```http
  # Requisição do navegador pedindo o primeiro 1 MB de um vídeo de 50 MB:
  GET /midia/aula-seguranca.mp4 HTTP/1.1
  Host: cdn.loja-ficticia.test
  Range: bytes=0-1048575

  # Resposta do servidor com a fatia solicitada:
  HTTP/1.1 206 Partial Content
  Content-Type: video/mp4
  Content-Range: bytes 0-1048575/52428800
  Content-Length: 1048576
  Accept-Ranges: bytes

  <...1.048.576 bytes binários do vídeo...>
  ```
- **Implicações de segurança e configuração segura:** historicamente, cabeçalhos `Range` com centenas de intervalos sobrepostos (`Range: bytes=0-1,0-2,0-3...`) causaram vulnerabilidades clássicas de **Negação de Serviço (DoS)** por exaustão de memória/CPU (como o famoso bug *Apache Killer* CVE-2011-3192) e estouros de inteiro em servidores Web/caches.
  - **[RECOMENDAÇÃO]** Limitar o número máximo de intervalos aceitos em um único header `Range` (servidores modernos já impõem esse limite nativamente) e validar a sintaxe do intervalo (retornando `416 Range Not Satisfiable` quando o intervalo pedido for inválido).

##### `301 Moved Permanently`

- **O que é e quando ocorre:** indica que o recurso solicitado recebeu uma **nova URL permanente** (informada no header `Location`) e que todas as referências futuras devem usar o novo endereço. Caches armazenam o redirecionamento `301` por padrão e motores de busca (Google, Bing) transferem a relevância de indexação (SEO) da URL antiga para a nova.
- **Comportamento histórico de método (Atenção para a prova!):** por razões históricas dos primeiros navegadores, quando um cliente envia um **`POST`** e recebe um **`301 Moved Permanently`** (ou `302 Found`), os navegadores **transformam o método da segunda requisição em `GET`** e descartam o corpo original! (Para redirecionar de forma permanente **proibindo** a troca de `POST` para `GET`, criou-se o código **`308 Permanent Redirect`**).
- **Exemplo HTTP:**
  ```http
  # Redirecionamento clássico de HTTP para HTTPS ou de domínio antigo para novo:
  GET /catalogo HTTP/1.1
  Host: loja-ficticia.test

  HTTP/1.1 301 Moved Permanently
  Location: https://www.loja-ficticia.test/produtos
  Content-Length: 0
  ```
- **Implicações de segurança e configuração segura:** como o navegador guarda o `301` em cache de forma agressiva, um erro de configuração ou um ataque de *Cache Poisoning* que gere um `301` malicioso ficará preso no cache dos clientes por muito tempo. Além disso, nunca construa o valor de `Location` concatenando parâmetros da URL sem validação (vulnerabilidade de **Open Redirect** — [seção 3.13](./03a-vulnerabilidades-parte1.md#s3)).

##### `302 Found`

- **O que é e quando ocorre:** indica que o recurso reside **temporariamente** em uma URL diferente (informada no cabeçalho `Location`). Como a mudança é temporária, o cliente deve continuar usando a URL original em requisições futuras. É o código mais usado no padrão **Post/Redirect/Get (PRG)**: o usuário envia `POST /login` (ou `POST /checkout`), o servidor processa os dados, define o cookie de sessão e responde `302 Found` com `Location: /painel`; o navegador então faz um `GET /painel` (evitando aquele aviso de "Confirmar reenvio do formulário" se o usuário apertar `F5`).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 302 Found
  Location: /painel
  Set-Cookie: __Host-sid=a1b2c3d4e5; Path=/; Secure; HttpOnly; SameSite=Lax
  Cache-Control: no-store
  ```
- **Implicações de segurança e configuração segura:**
  1. **Open Redirect (Redirecionamento Aberto):** em telas de login que aceitam `/login?redirect=https://site-malicioso.example`, se o servidor colocar o valor de `redirect` diretamente no header `Location:` do `302 Found`, atacantes usarão o link oficial da empresa para redirecionar vítimas a páginas de *phishing*.
  2. **Execução continuada após o redirect no código back-end (*Execution After Redirect — EAR*):** se uma rota restrita faz `res.redirect('/login')` (que apenas configura o status `302` e o header `Location`), mas o programador **esquece o `return` logo depois**, o código abaixo continua executando e envia o HTML do painel administrativo no corpo da própria resposta `302`! O navegador segue o redirect, mas um atacante usando `curl` ou o DevTools lê o corpo do `302` com todos os dados secretos.
  - **[REQUISITO]** Validar destinos de redirecionamento permitiendo apenas caminhos relativos locais (iniciados por `/` seguido de caractere alfanumérico, bloqueando `//dominio-externo.com` que é URL relativa de protocolo!) ou domínios presentes em uma *allowlist* estrita.
  - **[REQUISITO]** Sempre encerrar imediatamente o fluxo da função (`return res.redirect(...)`) ao emitir um redirecionamento de controle de acesso.

##### `304 Not Modified`

- **O que é e quando ocorre:** indica que a versão do recurso armazenada no **cache do cliente continua válida (igual à do servidor)** e, portanto, não há necessidade de o servidor retransmitir o corpo do recurso. Ocorre em resposta a uma **requisição condicional** (`GET` ou `HEAD` contendo `If-None-Match: "<etag>"` ou `If-Modified-Since: <data>`). Pela RFC 9110, uma resposta `304 Not Modified` **nunca possui corpo**: o navegador simplesmente lê os bytes que já estavam no seu cache local e atualiza o tempo de validade deles.
- **Exemplo HTTP:**
  ```http
  GET /assets/app.bundle.js HTTP/1.1
  Host: loja-ficticia.test
  If-None-Match: "v2.4.1-9a8b7c"

  HTTP/1.1 304 Not Modified
  ETag: "v2.4.1-9a8b7c"
  Cache-Control: public, max-age=3600
  ```
- **Implicações de segurança e configuração segura:** *ETags* persistem mesmo quando o usuário fecha o navegador; se um servidor gerar um `ETag` único por visitante para um arquivo estático (como um pixel ou imagem), ele pode abusar do mecanismo `If-None-Match` / `304 Not Modified` para **rastrear usuários sem cookies** (*ETag tracking*). Além disso, dados sensíveis de sessão nunca devem depender de revalidação `304` em computadores compartilhados (devem usar `Cache-Control: no-store`).

##### `307 Temporary Redirect`

- **O que é e quando ocorre:** é o equivalente moderno e rigoroso do `302` para **redirecionamento temporário**, com uma diferença crucial garantida pela RFC 9110: no `307 Temporary Redirect`, **o agente do usuário NÃO PODE alterar o método HTTP nem o corpo da requisição** ao seguir o redirecionamento! Se o cliente enviou um `POST` com um corpo JSON e recebeu `307 Temporary Redirect` com `Location: /v2/pedidos`, o cliente repetirá um **`POST` com o mesmo corpo JSON** no novo endereço.
  - *Curiosidade do Chrome DevTools:* quando um domínio possui política **HSTS** ativa e o navegador converte `http://` para `https://` internamente antes de ir à rede, a aba *Network* exibe uma linha sintética **`307 Internal Redirect`**.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 307 Temporary Redirect
  Location: https://api-contingencia.loja-ficticia.test/v1/pagamentos
  ```
- **Implicações de segurança e configuração segura:** **[CAUTION]** Nunca redirecione um `POST` com `307` (ou `308`) para um domínio de terceiro não confiável, pois o navegador reenviará automaticamente o corpo original do `POST` (que pode conter dados sensíveis) para a nova URL informada em `Location`.

##### `308 Permanent Redirect`

- **O que é e quando ocorre:** especificado originalmente na RFC 7538 e consolidado na RFC 9110, é o par permanente do `307` (e a versão estrita do `301`): indica que o recurso mudou **permanentemente** para a URL em `Location` e **proíbe a alteração do método HTTP e do corpo** ao seguir o redirecionamento (um `POST` continua sendo `POST`; um `DELETE` continua sendo `DELETE`).
- **Exemplo HTTP:**
  ```http
  # Mudança permanente de endpoint de API preservando métodos POST/PUT/DELETE:
  POST /api/pedidos HTTP/1.1
  Host: api.loja-ficticia.test

  HTTP/1.1 308 Permanent Redirect
  Location: /v1/pedidos
  ```
- **Implicações de segurança e configuração segura:** **[RECOMENDAÇÃO]** Ao configurar redirecionamento de `http://` para `https://` em APIs REST (que recebem `POST`, `PUT`, `DELETE`), prefira `308 Permanent Redirect` (ou rejeite HTTP em APIs diretamente) para evitar que clientes de API convertam silenciosamente um `POST` em `GET` como fariam diante de um `301`.

##### `400 Bad Request`

- **O que é e quando ocorre:** indica que o servidor não pode ou não irá processar a requisição devido a algo percebido como um **erro do cliente** na estrutura do pedido: JSON malformado (ex.: falta de uma chave ou vírgula inválida no corpo), sintaxe de cabeçalho HTTP corrompida, parâmetro de URL com formato impossível ou cabeçalho `Host` ausente/inválido no HTTP/1.1.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 400 Bad Request
  Content-Type: application/problem+json; charset=utf-8

  {"type":"https://api.loja-ficticia.test/erros/json-invalido","title":"JSON malformado","status":400}
  ```
- **Implicações de segurança e configuração segura:** mensagens de erro `400` excessivamente verbosas de *parsers* (como parsers XML ou deserializadores JSON/YAML) podem revelar caminhos internos de arquivos no servidor ou a biblioteca exata utilizada.
  - **[RECOMENDAÇÃO]** Retornar mensagens de erro padronizadas (ex.: formato **Problem Details for HTTP APIs**, RFC 9457) que informem qual campo falhou sem expor detalhes internos da pilha do servidor.

##### `401 Unauthorized`

- **O que é e quando ocorre:** apesar do nome histórico em inglês dizer *"Unauthorized"*, na especificação HTTP (RFC 9110, seção 15.5.2) o código **`401` significa NÃO AUTENTICADO (*Unauthenticated*)**! Ele indica que a requisição não foi aplicada porque **carece de credenciais de autenticação válidas** para o recurso de destino (ex.: o usuário ainda não fez login, o cookie de sessão não foi enviado, ou o token JWT no header `Authorization` está ausente, expirado ou com assinatura inválida). Pela RFC 9110, toda resposta `401` **deve** incluir um cabeçalho **`WWW-Authenticate`** indicando qual esquema de autenticação é aceito (como `Bearer` ou `Basic`).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 401 Unauthorized
  WWW-Authenticate: Bearer realm="api.loja-ficticia.test", error="invalid_token", error_description="O token expirou"
  Content-Type: application/json; charset=utf-8

  {"erro":"Autenticacao necessaria ou sessao expirada."}
  ```
- **Implicações de segurança e configuração segura:** **[BOA PRÁTICA]** Quando uma API retorna `401 Unauthorized`, o cliente sabe que **se autenticar (fazer login ou renovar o token com um *refresh token*) resolverá o problema**. Não revele detalhes que ajudem um atacante a enumerar contas ou depurar chaves criptográficas de assinatura no corpo do `401`.

##### `403 Forbidden`

- **O que é e quando ocorre:** indica que o servidor entendeu a requisição, **mas se recusa a autorizá-la** (*"Eu sei quem você é, mas você não tem permissão para acessar este recurso"*). Diferente do `401` (falta de autenticação), no **`403 Forbidden`** o usuário geralmente **já está autenticado** (ex.: logado como `"cliente"`), porém tenta acessar uma rota exclusiva de `"administrador"` (`GET /v1/admin/relatorios`), tenta acessar o pedido de outro cliente (IDOR bloqueado), falha na validação do token anti-CSRF ou é bloqueado por uma regra do **WAF** / lista de controle de IP. Autenticar-se novamente com as mesmas credenciais **não mudará o resultado**.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 403 Forbidden
  Content-Type: application/json; charset=utf-8

  {"erro":"Acesso negado: seu perfil nao possui permissao para esta operacao."}
  ```
- **Implicações de segurança e configuração segura:** em certos cenários sensíveis (como ao acessar `/v1/documentos/987654` que pertence a outro usuário), retornar `403 Forbidden` quando o documento existe (mas é de outra pessoa) e `404 Not Found` quando o documento não existe permite que um atacante **enumere quais IDs existem no banco de dados**. Por isso, a própria RFC 9110 autoriza que um servidor que queira **esconder a própria existência** de um recurso restrito responda com **`404 Not Found`** em vez de `403 Forbidden`.

##### `404 Not Found`

- **O que é e quando ocorre:** indica que o servidor de origem não encontrou nenhuma representação atual para o recurso solicitado na URL (ou não deseja revelar que existe para um cliente sem permissão).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 404 Not Found
  Content-Type: application/json; charset=utf-8

  {"erro":"Produto nao encontrado."}
  ```
- **Implicações de segurança e configuração segura:** durante a fase de reconhecimento (*reconnaissance*), ferramentas automatizadas de *fuzzing* de diretórios (como `ffuf`, `gobuster`, `dirsearch`) disparam milhares de requisições procurando arquivos esquecidos (`/.env`, `/.git/config`, `/backup.sql`, `/admin`). Além disso, páginas `404` mal construídas que escrevem na tela *"A página `/busca/<script>alert(1)</script>` não foi encontrada"* sem escapar a URL causam **XSS Refletido**!
  - **[REQUISITO]** Nunca refletir o caminho da URL na página `404` sem codificação HTML adequada (prevenção de XSS) e usar `404` uniforme quando quiser ocultar a existência de recursos privados contra enumeração.

##### `405 Method Not Allowed`

- **O que é e quando ocorre:** indica que a URL existe e é reconhecida pelo servidor, mas **o método HTTP utilizado na requisição não é suportado** por aquele recurso específico (por exemplo, enviar `DELETE /v1/login` ou enviar `POST` para um arquivo estático `.html` ou para uma rota que só aceita `GET`). Pela RFC 9110, o servidor **deve** incluir na resposta `405` o cabeçalho **`Allow`** listando quais métodos aquela URL aceita.
- **Exemplo HTTP:**
  ```http
  DELETE /v1/catalogo HTTP/1.1
  Host: api.loja-ficticia.test

  HTTP/1.1 405 Method Not Allowed
  Allow: GET, HEAD, OPTIONS
  Content-Type: application/json; charset=utf-8

  {"erro":"Metodo DELETE nao permitido para este endpoint."}
  ```
- **Implicações de segurança e configuração segura:** **[RECOMENDAÇÃO]** Garantir que o servidor Web e o framework da API rejeitem prontamente com `405 Method Not Allowed` (ou `501 Not Implemented`) qualquer verbo não previsto na rota, evitando que verbos exóticos (`TRACE`, `PUT`, `DELETE`) acionem comportamentos padrão inseguros.

##### `409 Conflict`

- **O que é e quando ocorre:** indica que a requisição não pôde ser completada devido a um **conflito com o estado atual do recurso de destino**. É muito utilizado em APIs REST para erros de regra de estado ou unicidade, tais como:
  1. Tentativa de cadastrar um usuário com um `email` ou `cpf` que **já existe** no banco.
  2. Tentativa de cancelar um pedido cujo status atual já é `"entregue"`.
  3. Conflito de edição concorrente (duas pessoas editando a mesma versão de um documento ao mesmo tempo).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 409 Conflict
  Content-Type: application/json; charset=utf-8

  {"erro":"Conflito de estado: o pedido #901 ja foi enviado e nao pode ser alterado."}
  ```
- **Implicações de segurança e configuração segura:** em formulários públicos de cadastro (`POST /v1/usuarios`), retornar `409 Conflict — Este e-mail já está cadastrado` permite **Enumeração de Usuários (*User Enumeration*)**: um atacante descobre se uma pessoa específica tem conta no sistema (o que é crítico em bancos, clínicas médicas ou plataformas sensíveis).
  - **[RECOMENDAÇÃO]** Em fluxos públicos sensíveis à privacidade, evitar confirmar diretamente se um e-mail/CPF já está cadastrado: envie uma mensagem neutra na tela (*"Se o endereço for válido, enviaremos um link de confirmação por e-mail"*) e trate o aviso de conta já existente por e-mail fora de banda.

##### `410 Gone`

- **O que é e quando ocorre:** semelhante ao `404 Not Found`, porém mais específico: informa que o recurso existia antes naquela URL, mas foi **intencionalmente e permanentemente removido** e não possui endereço de redirecionamento. Motores de busca que recebem `410 Gone` removem a URL do índice muito mais rápido do que quando recebem `404`. Também é útil para links de convite ou promoções que já encerraram definitivamente.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 410 Gone
  Content-Type: application/json; charset=utf-8

  {"erro":"Esta oferta encerrou permanentemente."}
  ```
- **Implicações de segurança e configuração segura:** **[BOA PRÁTICA]** Pode ser usado ao desativar permanentemente versões antigas e inseguras de uma API (ex.: `/v0/usuarios` retorna `410 Gone`), deixando claro para os clientes que a versão foi descomissionada e não está apenas temporariamente fora do ar.

##### `413 Content Too Large` (anteriormente *Payload Too Large* / *Request Entity Too Large*)

- **O que é e quando ocorre:** indica que o servidor se recusa a processar a requisição porque o **corpo da requisição (*request payload*) é maior do que o servidor está disposto ou apto a processar** (por exemplo, tentar enviar uma foto de 25 MB quando o limite configurado é de 5 MB). A RFC 9110 renomeou oficialmente este código para `413 Content Too Large` (em RFCs anteriores chamava-se `Payload Too Large` na RFC 7231 e `Request Entity Too Large` na RFC 2616 — todos os três nomes podem aparecer em provas!).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 413 Content Too Large
  Retry-After: 3600
  Content-Type: application/json; charset=utf-8

  {"erro":"O arquivo enviado excede o tamanho maximo permitido de 5 MB."}
  ```
- **Implicações de segurança e configuração segura:** aceitar corpos de requisição sem limite de tamanho é uma vulnerabilidade clássica de **Negação de Serviço (DoS por exaustão de memória RAM e disco)**: um único atacante envia um JSON de 2 GB ou um upload infinito e derruba o processo do Node.js/Python/Java com *Out Of Memory (OOM)*.
  - **[REQUISITO]** Configurar limites rígidos de tamanho de payload tanto no proxy reverso (ex.: `client_max_body_size 2m;` no Nginx) quanto no parser do back-end (ex.: `express.json({ limit: '100kb' })`), rejeitando excessos imediatamente com **`413 Content Too Large`**.

##### `415 Unsupported Media Type`

- **O que é e quando ocorre:** indica que o servidor se recusa a aceitar a requisição porque o **formato do corpo (*media type*) informado no header `Content-Type`** (ou a codificação em `Content-Encoding`) **não é suportado** pelo endpoint — por exemplo, enviar `Content-Type: text/xml` ou `text/plain` para um endpoint REST que aceita exclusivamente `application/json`, ou tentar fazer upload de um `.exe` onde só se aceita `image/png` e `image/jpeg`.
- **Exemplo HTTP:**
  ```http
  POST /v1/pedidos HTTP/1.1
  Host: api.loja-ficticia.test
  Content-Type: text/plain

  HTTP/1.1 415 Unsupported Media Type
  Accept-Post: application/json
  Content-Type: application/json; charset=utf-8

  {"erro":"Formato nao suportado. Envie Content-Type: application/json."}
  ```
- **Implicações de segurança e configuração segura:** exigir `Content-Type: application/json` e rejeitar qualquer outro `Content-Type` com **`415 Unsupported Media Type`** é uma **defesa em profundidade crucial contra CSRF** e contra **XXE (*XML External Entity*)**! Por quê? Porque um `<form>` HTML malicioso de outro site só consegue enviar `application/x-www-form-urlencoded`, `multipart/form-data` ou `text/plain`; se o seu servidor recusar esses três tipos com `415` e exigir estritamente `application/json`, qualquer tentativa cross-origin via JS será obrigada pelo navegador a passar por *preflight* CORS (`OPTIONS`) e será bloqueada!
  - **[REQUISITO]** Validar estritamente o cabeçalho `Content-Type` em todas as rotas `POST`/`PUT`/`PATCH` e retornar `415 Unsupported Media Type` para tipos não esperados.

##### `418 I'm a teapot` (RFC 2324 / RFC 9110)

- **O que é e quando ocorre:** criado originalmente em 1º de abril de 1998 na **RFC 2324** (*Hyper Text Coffee Pot Control Protocol — HTCPCP/1.0*, uma sátira oficial do IETF) para indicar que *"o servidor se recusa a coar café porque é um bule de chá, de forma permanente"*. Devido à sua enorme popularidade como *easter egg* na cultura de engenharia de software e ao fato de muitos sistemas e WAFs reais terem passado a usá-lo (por exemplo, para rejeitar requisições automatizadas de *bots* ou *scrapers* indesejados), a **RFC 9110 (seção 15.5.19)** reservou permanentemente o código **`418` (Unused)** para que ele nunca seja atribuído a outra função oficial no HTTP.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 418 I'm a teapot
  Content-Type: text/plain; charset=utf-8

  I'm a teapot: requisicao bloqueada pela politica anti-bot.
  ```
- **Implicações de segurança e configuração segura:** **[BOA PRÁTICA]** Embora apareça frequentemente em questões de prova como curiosidade sobre as RFCs do HTTP, em APIs corporativas sérias prefira códigos semânticos padronizados (`403 Forbidden` ou `429 Too Many Requests`) para bloqueios de segurança, facilitando a integração com ferramentas de SIEM (*Security Information and Event Management*) e monitoramento.

##### `422 Unprocessable Content` (anteriormente *Unprocessable Entity*)

- **O que é e quando ocorre:** originado no WebDAV (RFC 4918 como *Unprocessable Entity*) e hoje incorporado ao núcleo do HTTP pela **RFC 9110 (seção 15.5.21)** com o nome **`422 Unprocessable Content`**, indica que o servidor entende perfeitamente o `Content-Type` (não é `415`) e a sintaxe do JSON está perfeita (não é um `400` de erro de sintaxe), mas **não foi possível processar as instruções contidas devido a erros semânticos ou de validação de negócio** nos campos enviados (ex.: `"idade": -5`, `"cpf": "111.111.111-11"` com dígitos verificadores inválidos, ou `"dataFim"` anterior à `"dataInicio"`).
  - *Nota para a prova:* tanto `400 Bad Request` quanto `422 Unprocessable Content` são aceitos para erros de validação de campos, mas quando uma prova pergunta qual código diferencia **erro de validação semântica em um JSON sintaticamente válido**, a resposta exata é **`422`**.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 422 Unprocessable Content
  Content-Type: application/problem+json; charset=utf-8

  {
    "title": "Falha na validacao dos dados",
    "status": 422,
    "erros": [
      {"campo": "quantidade", "mensagem": "Deve ser um numero inteiro maior ou igual a 1."},
      {"campo": "cep", "mensagem": "Formato de CEP invalido."}
    ]
  }
  ```
- **Implicações de segurança e configuração segura:** **[REQUISITO]** Toda validação de regras de negócio que gere `422` deve ocorrer obrigatoriamente no servidor, mesmo que o front-end já possua a mesma validação visual nos formulários.

##### `429 Too Many Requests`

- **O que é e quando ocorre:** definido na RFC 6585, indica que o usuário/cliente **enviou requisições demais em um determinado intervalo de tempo (*Rate Limiting* — limitação de taxa)**. A resposta deve explicar o limite atingido e, preferencialmente, incluir o cabeçalho **`Retry-After`** indicando quantos segundos (ou até qual data/hora) o cliente deve aguardar antes de tentar uma nova requisição.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 429 Too Many Requests
  Retry-After: 60
  RateLimit-Limit: 5
  RateLimit-Remaining: 0
  RateLimit-Reset: 60
  Content-Type: application/json; charset=utf-8

  {"erro":"Limite de tentativas excedido. Aguarde 60 segundos antes de tentar novamente."}
  ```
- **Implicações de segurança e configuração segura:** o código **`429 Too Many Requests`** é uma das defesas mais importantes em Cybersecurity Web, pois protege contra:
  1. **Ataques de Força Bruta (*Brute Force*) e *Credential Stuffing*** em endpoints de `/login`, `/recuperar-senha` e validação de códigos MFA/OTP de 6 dígitos.
  2. **Scraping abusivo e enumeração massiva de dados** (varrer CPFs ou IDs de clientes).
  3. **Negação de Serviço (DoS) de camada de aplicação** e abuso financeiro (disparar milhares de SMS ou chamadas a modelos de Inteligência Artificial pagos).
  - **[REQUISITO]** Implementar *rate limiting* com retorno `429 Too Many Requests` e header `Retry-After` em todos os endpoints de login, recuperação de senha, envio de SMS/e-mail, geração de relatórios pesados e rotas públicas de API, combinando limites por endereço IP e por identificador de conta/usuário.

##### `500 Internal Server Error`

- **O que é e quando ocorre:** código genérico de erro de servidor que indica que o back-end encontrou uma **condição inesperada (uma exceção não tratada no código, erro de sintaxe SQL, ponteiro nulo / `TypeError` no servidor, falha de conexão com o banco)** que o impediu de atender à requisição.
- **Exemplo HTTP (Seguro vs. Inseguro):**
  ```http
  # EXEMPLO INSEGURO (Vazando Stack Trace e estrutura do banco — CWE-209):
  HTTP/1.1 500 Internal Server Error
  Content-Type: text/html

  <h1>Error: ER_PARSE_ERROR: You have an error in your SQL syntax near '' OR '1'='1' at line 1</h1>
  <pre>at Query.Sequence._packetToError (/var/www/loja/node_modules/mysql/lib/protocol/sequences/Sequence.js:47:14)
  at /var/www/loja/src/controllers/authController.js:82:19</pre>

  # EXEMPLO SEGURO (Mensagem genérica c/ ID de correlação; detalhe fica só no log interno):
  HTTP/1.1 500 Internal Server Error
  Content-Type: application/json; charset=utf-8

  {"erro":"Ocorreu um erro interno inesperado.","correlationId":"req-8f9e2a1b-2026"}
  ```
- **Implicações de segurança e configuração segura (Atenção máxima para a prova e para auditoria no DevTools!):**
  - Quando um atacante testa payloads (como uma aspa simples `'` em um parâmetro) e a API responde com **`500 Internal Server Error`**, isso é um **forte indício de que a entrada quebrou a lógica interna ou uma consulta SQL (possível *SQL Injection*)**!
  - Pior ainda: se o servidor estiver com modo de depuração ativo (`DEBUG=True` no Django/Flask, `APP_ENV=local` no Laravel ou middleware de erro padrão de desenvolvimento), a resposta `500` **vaza o *Stack Trace* completo**, revelando nomes de tabelas e colunas do banco de dados, versões exatas de bibliotecas, caminhos internos do servidor (`/var/www/...`) e até variáveis de ambiente com senhas do banco!
  - **[REQUISITO]** Em produção, desativar modos de *debug* e configurar um tratador global de exceções (*global error handler*) que registre o *stack trace* completo **apenas nos logs internos restritos do servidor**, retornando ao cliente no `500` somente uma mensagem genérica padronizada e um identificador único de rastreamento (`correlationId`).

##### `502 Bad Gateway`

- **O que é e quando ocorre:** indica que o servidor que está respondendo ao cliente atua como um **gateway ou proxy reverso** (ex.: Nginx, Cloudflare, AWS ALB) e, ao tentar repassar a requisição para o servidor de aplicação de origem (*upstream* — ex.: processo Node.js, Gunicorn/Python, PHP-FPM ou Tomcat), **recebeu uma resposta inválida ou a conexão com o back-end foi recusada/encerrada abruptamente** (por exemplo, o processo do Node.js caiu ou travou antes de responder).
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 502 Bad Gateway
  Server: gateway
  Content-Type: application/json; charset=utf-8

  {"erro":"Falha temporaria de comunicacao com o servico de aplicacao."}
  ```
- **Implicações de segurança e configuração segura:** páginas padrão de erro `502`/`503`/`504` geradas pelo proxy reverso frequentemente exibem no rodapé a versão exata do servidor (ex.: `nginx/1.24.0 (Ubuntu)`) e esquecem de incluir cabeçalhos de CORS e de segurança (fazendo o front-end ver um erro confuso de CORS em vez de saber que foi um `502`).
  - **[RECOMENDAÇÃO]** Configurar páginas/JSONs personalizados para erros `502`, `503` e `504` no proxy reverso, ocultando a versão do software (`server_tokens off;`) e mantendo os cabeçalhos de segurança e CORS necessários (`always` na diretiva `add_header` do Nginx).

##### `503 Service Unavailable`

- **O que é e quando ocorre:** indica que o servidor **não está pronto para manipular a requisição no momento**, geralmente por estar em **manutenção programada** ou temporariamente **sobrecarregado** (fila de conexões cheia, pool de conexões com o banco esgotado, proteção anti-DDoS ativa). É uma condição temporária e deve, sempre que possível, vir acompanhada do cabeçalho **`Retry-After`** informando a previsão de retorno.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 503 Service Unavailable
  Retry-After: 120
  Cache-Control: no-store
  Content-Type: application/json; charset=utf-8

  {"erro":"Servico em manutencao programada. Retorno previsto em 2 minutos."}
  ```
- **Implicações de segurança e configuração segura:** **[REQUISITO]** Sempre incluir `Cache-Control: no-store` em respostas `503` (e demais `5xx`) para impedir que uma CDN ou proxy intermediário armazene a página de indisponibilidade em cache e continue exibindo "Fora do ar" mesmo depois que o sistema já voltou a funcionar.

##### `504 Gateway Timeout`

- **O que é e quando ocorre:** indica que o servidor, atuando como **gateway ou proxy reverso** (Nginx, API Gateway, CDN), enviou a requisição para o servidor de aplicação (*upstream*), mas **o servidor de aplicação demorou demais e não respondeu dentro do tempo limite (*timeout*) configurado** (por exemplo, o proxy reverso espera no máximo 30 segundos, mas uma consulta pesada ao banco de dados ou uma chamada a um serviço externo de nota fiscal travou por 60 segundos).
- **Diferença clássica de prova (`502` vs `503` vs `504`):**
  - **`502 Bad Gateway`:** o proxy tentou falar com o back-end e recebeu uma **resposta quebrada ou conexão recusada** (`ECONNREFUSED` / processo morto).
  - **`503 Service Unavailable`:** o serviço está **sobrecarregado ou em manutenção** e recusa novas tarefas no momento.
  - **`504 Gateway Timeout`:** o proxy conseguiu entregar o pedido ao back-end, mas o back-end **demorou além do limite de tempo (*timeout*)** sem responder.
- **Exemplo HTTP:**
  ```http
  HTTP/1.1 504 Gateway Timeout
  Content-Type: application/json; charset=utf-8

  {"erro":"O processamento excedeu o tempo limite. Tente novamente em instantes."}
  ```
- **Implicações de segurança e configuração segura:** se um atacante consegue disparar um `504 Gateway Timeout` com uma requisição específica (por exemplo, enviando uma expressão regular complexa — ataque **ReDoS**, ou uma consulta sem paginação, ou testando *Blind SQL Injection* baseado em tempo com `SLEEP(10)` ou `pg_sleep(10)`), isso revela tanto um vetor de **Negação de Serviço (DoS)** quanto possíveis vulnerabilidades de injeção cega por tempo!
  - **[RECOMENDAÇÃO]** Definir *timeouts* em todas as chamadas de banco de dados e integrações HTTP externas do back-end (menores que o *timeout* do gateway) e mover processamentos demorados (como geração de relatórios PDF grandes) para filas assíncronas (retornando `202 Accepted` imediatamente e permitindo consultar o progresso depois).

<a id="s1-5-3"></a>
#### 1.5.3 Tabela-resumo de códigos HTTP

Use a tabela consolidada abaixo para revisão rápida antes da prova:

| Código | Nome Oficial (RFC 9110 / RFCs) | Classe | Resumo Técnico Direto | Ponto de Atenção em Segurança / Prova |
|---|---|---|---|---|
| **`100`** | `Continue` | 1xx | Cabeçalhos iniciais aprovados; cliente pode enviar o corpo (`Expect: 100-continue`) | Validar autenticação e `Content-Length` antes de aprovar uploads grandes |
| **`101`** | `Switching Protocols` | 1xx | Servidor aceita trocar de protocolo na conexão (ex.: upgrade para **WebSocket**) | Validar header `Origin` para evitar *Cross-Site WebSocket Hijacking* |
| **`200`** | `OK` | 2xx | Requisição atendida com sucesso; resposta contém o resultado | Evitar *Soft 404/500* (retornar `200` quando houve erro de negócio/permissão) |
| **`201`** | `Created` | 2xx | Novo recurso criado com sucesso (geralmente via `POST`); traz header `Location` | Cuidado para não serializar campos secretos (`password_hash`) do objeto criado |
| **`204`** | `No Content` | 2xx | Sucesso, porém **sem corpo na resposta** (típico em `DELETE`, `PUT` e `OPTIONS`) | Proibido ter corpo; `resp.json()` falha se chamado sem checar `status === 204` |
| **`206`** | `Partial Content` | 2xx | Entrega fatia de bytes solicitada via header `Range` (streaming e retomada) | Limitar múltiplos intervalos em `Range` para evitar DoS |
| **`301`** | `Moved Permanently` | 3xx | Redirecionamento **permanente** (`Location`); navegadores podem mudar `POST` p/ `GET` | Cacheável por padrão; validar `Location` contra *Open Redirect* |
| **`302`** | `Found` | 3xx | Redirecionamento **temporário** (`Location`); navegadores mudam `POST` p/ `GET` (PRG) | Principal vetor de **Open Redirect** se `Location` vier de parâmetro não validado |
| **`304`** | `Not Modified` | 3xx | Cache local ainda é válido (`If-None-Match` / `If-Modified-Since`); **sem corpo** | Economiza banda; não deve ser usado como substituto de `no-store` em dados secretos |
| **`307`** | `Temporary Redirect` | 3xx | Redirecionamento **temporário** que **preserva método e corpo** (`POST` continua `POST`) | Também aparece no DevTools como `307 Internal Redirect` quando há **HSTS** |
| **`308`** | `Permanent Redirect` | 3xx | Redirecionamento **permanente** que **preserva método e corpo** (`POST` continua `POST`) | Preferível ao `301` para redirecionar endpoints de API que recebem `POST`/`PUT` |
| **`400`** | `Bad Request` | 4xx | Erro sintático ou estrutural na requisição do cliente (JSON inválido, parâmetros ruins) | Não vazar detalhes internos do parser na mensagem de erro |
| **`401`** | `Unauthorized` | 4xx | **Não autenticado**: faltam credenciais válidas ("Quem é você?"). Exige `WWW-Authenticate` | Pegadinha n.º 1 de prova: `401` é falha de **autenticação**, não de autorização! |
| **`403`** | `Forbidden` | 4xx | **Não autorizado**: servidor sabe quem é, mas **nega permissão** ("Você não pode!") | Falha de **autorização** (ou bloqueio de WAF/CSRF); relogar não resolve |
| **`404`** | `Not Found` | 4xx | Recurso não encontrado na URL (ou ocultado intencionalmente) | Pode substituir `403` para impedir enumeração de IDs privados; escapar URL contra XSS |
| **`405`** | `Method Not Allowed` | 4xx | A URL existe, mas não aceita aquele verbo HTTP; exige header `Allow` | Bloqueia verbos indevidos (como `DELETE` ou `TRACE` em rotas de leitura) |
| **`409`** | `Conflict` | 4xx | Conflito com o estado atual do recurso (e-mail duplicado, versão concorrente) | Cuidado com **enumeração de usuários** ao acusar CPF/e-mail já existente |
| **`410`** | `Gone` | 4xx | Recurso foi removido **permanentemente** e de forma intencional | Útil para descomissionar versões antigas de APIs |
| **`413`** | `Content Too Large` | 4xx | Corpo da requisição excede o limite máximo permitido pelo servidor | Defesa essencial contra **DoS por upload/JSON gigante** (`client_max_body_size`) |
| **`415`** | `Unsupported Media Type` | 4xx | Formato `Content-Type` enviado pelo cliente não é aceito pelo endpoint | Exigir `application/json` e rejeitar outros com `415` ajuda a mitigar CSRF e XXE |
| **`418`** | `I'm a teapot` | 4xx | Código humorístico da RFC 2324 (HTCPCP) reservado na RFC 9110 | Usado em provas como questão sobre RFCs ou em WAFs para respostas anti-bot |
| **`422`** | `Unprocessable Content` | 4xx | Sintaxe do JSON está certa, mas falhou na **validação semântica/regras de negócio** | Diferencia erro de sintaxe bruta (`400`) de erro de validação de campos (`422`) |
| **`429`** | `Too Many Requests` | 4xx | Cliente excedeu o limite de requisições por tempo (**Rate Limiting**); usa `Retry-After` | Defesa crítica contra **Força Bruta**, *Credential Stuffing*, *Scraping* e DoS |
| **`500`** | `Internal Server Error` | 5xx | Exceção não tratada ou falha interna inesperada no código/banco do servidor | **Risco crítico de vazar Stack Trace / erros SQL**; em testes, pode indicar SQLi |
| **`502`** | `Bad Gateway` | 5xx | Proxy reverso/gateway recebeu resposta inválida ou conexão recusada do back-end | Processo da aplicação caiu; ocultar versão do servidor na página de erro |
| **`503`** | `Service Unavailable` | 5xx | Servidor temporariamente sobrecarregado ou em manutenção (`Retry-After`) | Usar `Cache-Control: no-store` para não cachear a queda na CDN |
| **`504`** | `Gateway Timeout` | 5xx | Proxy reverso esperou pela resposta do back-end além do tempo limite (*timeout*) | Pode indicar consulta lenta (DoS) ou teste de *Blind SQL Injection* (`SLEEP`) |

<a id="s1-5-4"></a>
#### 1.5.4 Pegadinhas clássicas de prova sobre status HTTP

1. **Pegadinha `401 Unauthorized` vs. `403 Forbidden`:**
   - **A armadilha:** o candidato lê a palavra em inglês *"Unauthorized"* no código `401` e marca que `401` significa falta de autorização (permissão). **Errado!**
   - **A resposta correta:** **`401 Unauthorized` = Falha de AUTENTICAÇÃO** (o cliente não provou quem é: não enviou token/cookie ou enviou um token expirado/inválido). **`403 Forbidden` = Falha de AUTORIZAÇÃO** (o cliente já está autenticado, mas seu perfil não tem permissão para aquela ação).
2. **Pegadinha `301` / `302` vs. `307` / `308` (Preservação do método HTTP):**
   - **A armadilha:** achar que `302` e `307` são 100% iguais, ou que `301` e `308` são 100% iguais.
   - **A resposta correta:**
     - **Temporários:** `302 Found` (permite que o navegador troque `POST` por `GET`) vs. `307 Temporary Redirect` (**proíbe** trocar o método: `POST` continua `POST`).
     - **Permanentes:** `301 Moved Permanently` (permite que o navegador troque `POST` por `GET`) vs. `308 Permanent Redirect` (**proíbe** trocar o método: `POST` continua `POST`).
3. **Pegadinha do `304 Not Modified`:**
   - **A armadilha:** como começa com `3`, achar que o `304` redireciona o usuário para outra URL, ou achar que é um erro.
   - **A resposta correta:** o `304 Not Modified` **não muda de URL e não é erro**: é a confirmação do servidor de que o arquivo no cache do navegador continua atualizado. A resposta `304` **não tem corpo**, economizando transferência de dados.
4. **Pegadinha `400 Bad Request` vs. `422 Unprocessable Content` vs. `415 Unsupported Media Type`:**
   - Se o cliente enviou `Content-Type: application/xml` para uma API que só aceita JSON → **`415 Unsupported Media Type`**.
   - Se o cliente enviou `Content-Type: application/json`, mas o JSON está truncado `{"nome": "Ana"` (erro de sintaxe) → **`400 Bad Request`**.
   - Se o cliente enviou `Content-Type: application/json` com JSON perfeito `{"nome": "", "idade": -10}`, mas violou as regras de validação dos campos → **`422 Unprocessable Content`** (embora `400` também seja tolerado genericamente).
5. **Pegadinha `502` vs. `503` vs. `504` em arquiteturas com Proxy Reverso / Balanceador:**
   - O container do Node.js travou/morreu e recusou a conexão TCP do Nginx → **`502 Bad Gateway`**.
   - O servidor está em janela de manutenção ou sem *workers* disponíveis na fila → **`503 Service Unavailable`**.
   - O Node.js aceitou a conexão, mas ficou preso 60 segundos rodando uma query pesada e o Nginx desistiu de esperar → **`504 Gateway Timeout`**.

<a id="s1-6"></a>
### 1.6 Headers HTTP de Requisição e Resposta

Os **headers HTTP** (cabeçalhos HTTP, definidos na RFC 9110, seções 6 a 14) são pares de metadados enviados pelo cliente na **requisição** e pelo servidor na **resposta**. Eles controlam negociação de conteúdo, autenticação, cookies, cache, compressão, conexão, rastreamento de contexto e políticas críticas de segurança.

<a id="s1-6-1"></a>
#### 1.6.1 Estrutura e funcionamento dos headers

**O que é e como funciona a sintaxe:**

1. **Formato `Nome: Valor`:** no HTTP/1.1, cada linha de cabeçalho consiste em um nome de campo (*field-name*), seguido por dois-pontos (`:`), espaço opcional (*OWS — Optional Whitespace*), o valor do campo (*field-value*) e o terminador de linha `CRLF` (`\r\n`).
2. **Case-insensitivity (Insensibilidade a maiúsculas/minúsculas nos nomes):** pela RFC 9110 (seção 5.1), **o nome de qualquer header HTTP é *case-insensitive***: `Content-Type`, `content-type` e `CONTENT-TYPE` são exatamente o mesmo cabeçalho! No **HTTP/2** (RFC 9113) e no **HTTP/3** (RFC 9114), inclusive, a especificação exige que todos os nomes de cabeçalhos sejam convertidos para **letras minúsculas** antes da compressão binária (HPACK/QPACK) — motivo pelo qual, ao inspecionar conexões `h2` ou `h3` na aba *Network* do Chrome DevTools, você verá os cabeçalhos escritos em minúsculas (`content-type`, `cache-control`, `set-cookie`).
   - *Atenção:* o **nome** do header é *case-insensitive*, mas o **valor** pode ser *case-sensitive* dependendo do cabeçalho (por exemplo, o valor de um token em `Authorization: Bearer ...`, de um cookie em `Cookie:` ou de uma `ETag` **diferencia** maiúsculas de minúsculas!).
3. **Múltiplos valores e repetição:** quando um cabeçalho aceita uma lista de valores (como `Accept`, `Cache-Control`), múltiplos cabeçalhos com o mesmo nome podem ser combinados em uma única linha separados por vírgula (`,`). **Exceção importante da RFC 9110:** o cabeçalho **`Set-Cookie` nunca deve ser concatenado com vírgula** (porque datas em `Expires=Wed, 21 Oct 2026...` contêm vírgulas!); por isso, um servidor que define 3 cookies envia 3 linhas `Set-Cookie:` separadas na resposta.
4. **Pseudo-headers do HTTP/2 e HTTP/3 (`:method`, `:path`, `:authority`, `:scheme`, `:status`):**
   - No HTTP/2 e no HTTP/3 não existe mais a "primeira linha de texto" (`GET /produtos HTTP/1.1` ou `HTTP/1.1 200 OK`). Em seu lugar, os metadados de controle da mensagem são transmitidos como **pseudo-cabeçalhos** iniciados pelo caractere dois-pontos (`:`), que aparecem no topo do painel *Headers* do DevTools:
     - Na requisição (4 pseudo-headers): **`:method`** (ex.: `GET`), **`:scheme`** (ex.: `https`), **`:authority`** (substitui o header `Host`, ex.: `loja-ficticia.test`) e **`:path`** (ex.: `/produtos?page=1`).
     - Na resposta (1 pseudo-header): **`:status`** (ex.: `200`).
   - Clientes não podem criar cabeçalhos customizados que comecem com `:`; esse prefixo é exclusivo do protocolo.
5. **Forbidden Request Headers (Cabeçalhos proibidos ao JavaScript no navegador):**
   - Segundo o **Fetch Standard** do WHATWG, existem cabeçalhos de requisição que são controlados **exclusivamente pelo próprio navegador** e que o código JavaScript (`fetch()` ou `XMLHttpRequest`) **é proibido de definir ou sobrescrever**. Se um script tentar fazer `fetch(url, { headers: { "Cookie": "a=b", "Origin": "https://banco.com", "Host": "outro.com" } })`, o navegador simplesmente ignora essas alterações silenciosamente.
   - **Principais *Forbidden Request Headers*:** `Cookie`, `Host`, `Origin`, `Referer`, `Content-Length`, `Connection`, `Upgrade`, `Via`, `TE`, `Transfer-Encoding` e **todos os cabeçalhos que começam com `Sec-` ou `Proxy-`** (como `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User`, `Sec-CH-UA`).
   - *Por que isso é vital para a segurança?* Porque o servidor pode confiar que um cabeçalho como `Origin` ou `Sec-Fetch-Site` vindo de um navegador real **não foi falsificado por um script malicioso via XSS ou página de terceiro**! (Atenção: um atacante fora do navegador, usando `curl` ou Python, pode enviar qualquer cabeçalho; portanto, `Origin` e `Sec-Fetch-*` protegem contra ataques **via navegador da vítima**, como CSRF e CORS, mas não substituem autenticação/autorização no servidor).

<a id="s1-6-2"></a>
#### 1.6.2 Principais headers de requisição

Abaixo estão detalhados os principais cabeçalhos enviados pelo cliente ao servidor, seu funcionamento e suas implicações de segurança:

##### 1. `Host` (e `:authority` no HTTP/2 e HTTP/3)
- **O que é e para que serve:** informa o nome de domínio (e a porta, se não for a padrão 80/443) do servidor ao qual a requisição se destina. Obrigatório em todas as requisições HTTP/1.1 (RFC 9112), permite a **hospedagem virtual (*Virtual Hosting*)**, onde um único endereço IP atende dezenas de sites diferentes.
- **Exemplo:** `Host: loja-ficticia.test` ou `Host: api.loja-ficticia.test:8443`.
- **Segurança:** embora o JavaScript do navegador não possa alterá-lo (*forbidden header*), ferramentas externas (`curl -H "Host: evil.com" https://loja-ficticia.test`) podem manipulá-lo, causando ataques de **Host Header Injection** (veja [1.6.4](#s1-6-4)).

##### 2. `User-Agent` e Client Hints (`Sec-CH-UA*`)
- **O que é e para que serve:** identifica o software cliente, sistema operacional e motor de renderização que está fazendo a requisição (navegador, app móvel, `curl`, robô do Google). Por privacidade (para reduzir *Browser Fingerprinting*), o Chrome e navegadores baseados em Chromium implementaram a **redução do `User-Agent` (*User-Agent Reduction*)** **[Varia por versão]**, congelando informações detalhadas na string clássica e migrando os detalhes para os cabeçalhos **User-Agent Client Hints** (`Sec-CH-UA`, `Sec-CH-UA-Mobile`, `Sec-CH-UA-Platform`).
- **Exemplo:**
  ```http
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
  Sec-CH-UA: "Chromium";v="130", "Google Chrome";v="130", "Not?A_Brand";v="99"
  Sec-CH-UA-Mobile: ?0
  Sec-CH-UA-Platform: "Windows"
  ```
- **Segurança:** o `User-Agent` é **100% controlado pelo cliente** (pode ser trocado até pelo botão *More network conditions* no próprio DevTools). **[REQUISITO]** Nunca basear controles de segurança ou autenticação no `User-Agent`, e sanitizá-lo ao exibi-lo em painéis administrativos de logs (vetor clássico de *Stored XSS* em painéis de auditoria e de *Log4Shell*).

##### 3. `Accept`, `Accept-Encoding` e `Accept-Language` (Negociação de Conteúdo)
- **O que são e para que servem:** permitem que o cliente informe ao servidor quais formatos de mídia (**`Accept`**), algoritmos de compressão (**`Accept-Encoding`**) e idiomas humanos (**`Accept-Language`**) ele compreende e prefere, opcionalmente ponderados pelo fator de qualidade `q=` (de `0` a `1`, padrão `1.0`).
- **Exemplo:**
  ```http
  Accept: application/json, text/plain;q=0.8, */*;q=0.5
  Accept-Encoding: gzip, deflate, br, zstd
  Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7
  ```
  - Em `Accept-Encoding`, os algoritmos modernos suportados pelo Chrome incluem `gzip`, `deflate`, `br` (**Brotli**, RFC 7932) e **`zstd`** (**Zstandard**, RFC 8878) **[Varia por versão]**.

##### 4. `Authorization`
- **O que é e para que serve:** transporta credenciais de autenticação do cliente para o servidor no formato `<Esquema> <Credencial>`. Os dois esquemas mais cobrados em provas são:
  - **`Basic` (RFC 7617):** `Authorization: Basic <base64(usuario:senha)>`. Atenção: Base64 é apenas codificação de texto (reversível instantaneamente), **não é criptografia**! Só pode ser usado sobre HTTPS.
  - **`Bearer` (RFC 6750):** `Authorization: Bearer <token>`, padrão universal para envio de *Access Tokens* OAuth 2.0 e **JWT (JSON Web Tokens)** em APIs REST.
- **Exemplo:** `Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...`
- **Segurança:** diferentemente do cabeçalho `Cookie`, o cabeçalho `Authorization` **não é enviado automaticamente pelo navegador** ao carregar imagens/formulários de outros sites — o que torna APIs que autenticam exclusivamente via header `Authorization` imunes a ataques **CSRF** tradicionais! Por outro lado, para o JavaScript enviar esse header, o token geralmente precisa ser lido pelo JS (ficando exposto se houver **XSS**, a menos que se use um padrão *Backend-for-Frontend* com cookies `HttpOnly`).

##### 5. `Cookie`
- **O que é e para que serve:** envia de volta ao servidor os pares `nome=valor` (separados por ponto e vírgula `; `) que foram previamente gravados pelo servidor via `Set-Cookie` (RFC 6265) e cujo domínio, caminho, tempo de vida e políticas (`Secure`, `SameSite`) são compatíveis com a requisição atual.
- **Exemplo:** `Cookie: __Host-sid=9f8e7d6c5b4a; tema=escuro`
- **Segurança:** no cabeçalho `Cookie` da requisição **não vão os atributos** (`HttpOnly`, `Secure`, `SameSite`, `Path`, `Max-Age`) — esses atributos só aparecem no `Set-Cookie` da resposta! Como o navegador anexa `Cookie` automaticamente, é indispensável configurar `SameSite` e proteções anti-CSRF.

##### 6. `Referer` (assim mesmo, com erro de grafia histórico na RFC!)
- **O que é e para que serve:** informa a URL (completa ou apenas a origem, dependendo da `Referrer-Policy` ativa — veja [seção 5.5](./05-headers-de-seguranca.md#s5)) da página anterior de onde partiu o clique no link ou o carregamento do sub-recurso (imagem, script, CSS).
- **Exemplo:** `Referer: https://loja-ficticia.test/produtos/busca?q=notebook`
- **Segurança:** se a URL da página atual contiver dados sensíveis na query string (ex.: `https://loja-ficticia.test/resetar-senha?token=SEGREDO123`) e a página carregar uma imagem externa ou o usuário clicar em um link externo sem uma política `Referrer-Policy` restritiva, o navegador enviará a URL inteira com o token secreto no header `Referer` para o servidor externo!
  - **[REQUISITO]** Nunca colocar tokens secretos em URLs e configurar o header de resposta `Referrer-Policy: strict-origin-when-cross-origin` (hoje o padrão nos navegadores modernos quando nenhuma política é especificada) ou `no-referrer` em páginas sensíveis.

##### 7. `Origin`
- **O que é e para que serve:** informa exclusivamente a **origem** (`esquema + "://" + host + [":porta"]`, **sem caminho e sem query string**) que iniciou a requisição. É enviado automaticamente pelo navegador em todas as requisições **CORS** (*cross-origin*) e em requisições de mutação (`POST`, `PUT`, `PATCH`, `DELETE`), mesmo *same-origin*. Em contextos opacos (como `<iframe>` com `sandbox` sem `allow-same-origin`, esquemas `data:`/`file:` ou redirecionamentos cross-origin), pode assumir o valor literal `Origin: null`.
- **Exemplo:** `Origin: https://painel.loja-ficticia.test`
- **Segurança:** peça central da segurança **CORS** e ótima defesa auxiliar contra **CSRF** (como é um *forbidden header*, nenhum script malicioso em outro site consegue forjar o header `Origin` no navegador da vítima).

##### 8. `Content-Type` e `Content-Length`
- **O que são e para que servem:** quando a requisição possui um corpo (`POST`, `PUT`, `PATCH`), **`Content-Type`** declara o tipo MIME e a codificação do payload enviado (ex.: `application/json; charset=utf-8`, `application/x-www-form-urlencoded` ou `multipart/form-data; boundary=----WebKitFormBoundary...`), e **`Content-Length`** informa o tamanho exato desse corpo em **bytes** (octetos).
- **Exemplo:**
  ```http
  Content-Type: application/json; charset=utf-8
  Content-Length: 49
  ```

##### 9. `If-None-Match` e `If-Modified-Since` (Requisições Condicionais de Cache)
- **O que são e para que servem:** enviados pelo navegador quando ele já possui uma cópia do recurso em cache local, mas precisa **revalidar** com o servidor se houve alteração:
  - **`If-None-Match`:** envia o identificador `ETag` da versão em cache (ex.: `If-None-Match: "v2.4.1-9a8b7c"`). Tem precedência sobre `If-Modified-Since`.
  - **`If-Modified-Since`:** envia a data da última modificação conhecida (ex.: `If-Modified-Since: Thu, 29 Oct 2026 18:00:00 GMT`).
  - Se o arquivo não mudou, o servidor responde **`304 Not Modified`** sem corpo; se mudou, responde **`200 OK`** com o arquivo novo.

##### 10. `Cache-Control` (na requisição)
- **O que é e para que serve:** embora seja mais famoso na resposta, o cliente também envia `Cache-Control` na requisição para ditar como caches intermediários e locais devem responder (por exemplo, quando você aperta `Ctrl+F5` ou marca **Disable cache** no DevTools, o Chrome envia `Cache-Control: no-cache` e `Pragma: no-cache` para forçar resposta fresca da origem).

##### 11. `Connection` e `Upgrade`
- **O que são e para que servem:** cabeçalhos de controle de salto único (*hop-by-hop headers*, válidos apenas no HTTP/1.1 e proibidos no HTTP/2 e HTTP/3). `Connection: keep-alive` mantém a conexão TCP aberta para reuso (padrão no HTTP/1.1); `Connection: close` pede o fechamento após a resposta; e o par `Connection: Upgrade` + `Upgrade: websocket` solicita a troca de protocolo (`101 Switching Protocols`).

##### 12. Fetch Metadata Request Headers (`Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User`)
- **O que são e para que servem:** família moderna de cabeçalhos enviados automaticamente pelos navegadores em conexões seguras (`https://`) que informam ao servidor o **contexto exato** de como aquela requisição nasceu (permitindo bloquear ataques CSRF, Clickjacking e XS-Leaks no servidor com políticas de *Resource Isolation*):
  - **`Sec-Fetch-Site`:** relação entre a origem da página que iniciou o pedido e o servidor de destino:
    - `none`: ação direta do usuário (digitar a URL na barra de endereços ou clicar em um favorito).
    - `same-origin`: requisição vinda exatamente da mesma origem (`esquema + host + porta`).
    - `same-site`: requisição vinda de um subdomínio do mesmo site registrável (*eTLD+1*, ex.: `app.loja-ficticia.test` chamando `api.loja-ficticia.test`).
    - `cross-site`: requisição disparada por um site externo totalmente diferente (`https://outro-site.example`) — **alerta imediato para possível CSRF se for uma rota de escrita!**
  - **`Sec-Fetch-Mode`:** modo da requisição: `navigate` (navegação de página), `cors` (chamada `fetch`/XHR com CORS), `no-cors` (carregamento de `<img>`, `<script>`), `same-origin`, `websocket`.
  - **`Sec-Fetch-Dest`:** onde o dado será usado: `document` (aba principal), `iframe`, `script`, `style`, `image`, `font`, `empty` (usado por `fetch()`/XHR).
  - **`Sec-Fetch-User`:** `?1` quando a navegação foi disparada por ativação explícita do usuário (clique ou teclado); omitido quando disparada automaticamente por script.
- **Exemplo:**
  ```http
  Sec-Fetch-Site: same-origin
  Sec-Fetch-Mode: cors
  Sec-Fetch-Dest: empty
  ```

##### 13. `Range`
- **O que é e para que serve:** solicita apenas um intervalo específico de bytes de um arquivo (ex.: `Range: bytes=0-1023`), recebendo em troca `206 Partial Content`.

##### 14. `X-Forwarded-For` (e cabeçalhos de proxy `X-Forwarded-Proto`, `Forwarded`)
- **O que é e para que serve:** cabeçalho *de facto* adicionado por proxies reversos, CDNs e balanceadores de carga para informar ao servidor de aplicação qual era o **endereço IP original do cliente** (já que, na conexão TCP direta com o servidor de aplicação, o IP de origem visível no socket é o IP do balanceador de carga). Quando a requisição passa por múltiplos proxies, o formato é uma lista separada por vírgulas:
  ```http
  X-Forwarded-For: <ip-cliente>, <ip-proxy-1>, <ip-proxy-2>
  ```
  O padrão formal da IETF que unifica `X-Forwarded-For`, `X-Forwarded-Proto` e `X-Forwarded-Host` é o header **`Forwarded`** (RFC 7239, ex.: `Forwarded: for=198.51.100.25;proto=https;host=loja-ficticia.test`).
- **Risco crítico de falsificação (*IP Spoofing via Header*):** qualquer cliente malicioso pode enviar na sua requisição um cabeçalho `X-Forwarded-For: 127.0.0.1` ou `X-Forwarded-For: 198.51.100.99`. Se o proxy reverso apenas concatenar novos IPs no final (resultando em `X-Forwarded-For: 127.0.0.1, 203.0.113.55`) e a aplicação back-end ler ingenuamente o **primeiro IP da esquerda** (ou se não houver proxy reverso filtrando o header), o atacante consegue **burlar o *rate limiting* de força bruta** (inventando um IP falso a cada tentativa de senha) e **burlar restrições de acesso baseadas em IP interno (`127.0.0.1`)**! Veja como configurar com segurança em [1.6.4](#s1-6-4).

##### 15. `X-Requested-With`
- **O que é e para que serve:** cabeçalho não padronizado (`X-Requested-With: XMLHttpRequest`) popularizado por bibliotecas como jQuery e Axios para indicar ao servidor que a requisição veio de uma chamada AJAX/JavaScript e não de uma navegação HTML normal. Como **não é um cabeçalho simples do CORS**, um site externo não consegue enviar `X-Requested-With` sem passar por *preflight* `OPTIONS` — sendo usado historicamente como defesa complementar contra CSRF.

<a id="s1-6-3"></a>
#### 1.6.3 Principais headers de resposta

A seguir estão os principais cabeçalhos enviados pelo servidor ao cliente na resposta HTTP, seu funcionamento prático e implicações de segurança:

##### 1. `Content-Type`
- **O que é e para que serve:** informa ao navegador o **tipo de mídia (MIME type)** do corpo da resposta e a codificação de caracteres (`charset`), determinando como o navegador deve processar o arquivo (renderizar como HTML, executar como JavaScript, aplicar como CSS, exibir como imagem ou interpretar como JSON).
- **Exemplos:** `Content-Type: text/html; charset=utf-8`, `Content-Type: application/json; charset=utf-8`, `Content-Type: image/webp`.
- **Segurança (*MIME Sniffing* e XSS):** se o servidor enviar um `Content-Type` incorreto (por exemplo, retornar texto fornecido pelo usuário com `Content-Type: text/html` em vez de `application/json` ou `text/plain`), ou se omitir o `Content-Type` permitindo que o navegador "adivinhe" o formato olhando os primeiros bytes do arquivo (*MIME Sniffing*), um arquivo de imagem ou texto enviado por um atacante contendo `<script>...</script>` pode acabar sendo executado como HTML/JS!
  - **[REQUISITO]** Sempre enviar o `Content-Type` exato com `charset=utf-8` acompanhado do header de segurança **`X-Content-Type-Options: nosniff`** (veja [seção 5.3](./05-headers-de-seguranca.md#s5)).

##### 2. `Content-Length`
- **O que é e para que serve:** indica o tamanho exato do corpo da resposta (já comprimido, se houver `Content-Encoding`) em **bytes** (octetos). No HTTP/1.1, permite que o navegador saiba exatamente onde uma resposta termina e onde a próxima começa dentro da mesma conexão TCP persistente (`keep-alive`).
- **Exemplo:** `Content-Length: 14520`

##### 3. `Content-Encoding`
- **O que é e para que serve:** informa qual algoritmo de **compressão sem perdas** foi aplicado pelo servidor sobre o corpo da resposta para economizar banda de rede (`gzip`, `br` — Brotli, `zstd` — Zstandard, `deflate`). O navegador descomprime os bytes transparentemente antes de entregá-los ao DOM ou ao `fetch()`.
- **Exemplo:** `Content-Encoding: br`
- **Segurança (Ataques de canal lateral por compressão — BREACH / CRIME):** quando uma resposta HTTP comprimida (`gzip`/`br`) contém ao mesmo tempo **um segredo fixo** (como um token CSRF na página) e **um texto refletido controlado pelo atacante** na mesma resposta, um atacante que observe o tamanho exato dos pacotes TLS na rede pode adivinhar o segredo caractere por caractere medindo quando a taxa de compressão aumenta (ataque **BREACH**). Mitiga-se com cookies `SameSite`, mascaramento aleatório de tokens CSRF a cada resposta ou desativando compressão em respostas com segredos refletidos.

##### 4. `Set-Cookie`
- **O que é e para que serve:** instrui o navegador a armazenar um cookie (par `nome=valor`) e define seus atributos de escopo e segurança (RFC 6265 / rascunho RFC 6265bis).
- **Exemplo seguro completo:**
  ```http
  Set-Cookie: __Host-sid=9f8e7d6c5b4a3f2e1d0c; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=3600
  ```
- **Resumo dos atributos (aprofundados na [seção 2.2 — Application](./02b-devtools-application.md#s2-2)):**
  - `HttpOnly`: **proíbe o acesso via JavaScript** (`document.cookie`), protegendo o cookie contra roubo direto por **XSS**.
  - `Secure`: garante que o cookie **só seja transmitido em conexões criptografadas HTTPS**.
  - `SameSite=Strict | Lax | None`: controla o envio do cookie em requisições *cross-site*, protegendo contra **CSRF** (`Lax` é o padrão nos navegadores modernos quando o atributo é omitido; `None` exige obrigatoriamente `Secure`).
  - Prefixos de nome `__Host-` (exige `Secure`, `Path=/` e proíbe atributo `Domain`, blindando o cookie contra subdomínios comprometidos) e `__Secure-` (exige `Secure`).

##### 5. `Cache-Control`, `ETag`, `Last-Modified` e `Expires`
- **O que são e para que servem:** controlam o armazenamento e a revalidação de respostas nos caches do navegador e das CDNs (detalhes completos na [seção 1.13](#s1) e [seção 5.12](./05-headers-de-seguranca.md#s5)):
  - **`Cache-Control`:** header principal moderno com diretivas como `no-store` (proíbe gravar qualquer cópia em cache — **obrigatório para dados bancários/pessoais sensíveis**), `no-cache` (pode guardar em cache, mas obriga a revalidar no servidor antes de usar), `private` (apenas cache do navegador do usuário, proibido em CDNs compartilhadas), `public` (permitido em CDNs), `max-age=<segundos>` e `immutable`.
  - **`ETag` (*Entity Tag*):** impressão digital (hash ou identificador de versão entre aspas, ex.: `ETag: "686897696a7c876b7e"`) da representação atual do recurso para revalidação via `If-None-Match`.
  - **`Last-Modified`:** data e hora em que o recurso foi modificado pela última vez (`Last-Modified: Thu, 29 Oct 2026 18:00:00 GMT`) para revalidação via `If-Modified-Since`.
  - **`Expires`:** data/hora absoluta de expiração do cache (`Expires: Sat, 30 Oct 2027 10:00:00 GMT`). É um header legado do HTTP/1.0; quando `Cache-Control: max-age=...` está presente, **`max-age` tem prioridade e ignora o `Expires`**.

##### 6. `Location`
- **O que é e para que serve:** possui dois papéis dependendo do status da resposta:
  1. Em respostas **`3xx` (`301`, `302`, `303`, `307`, `308`)**: indica a **URL de destino** para onde o navegador deve redirecionar automaticamente.
  2. Em respostas **`201 Created`**: informa a URL onde o recurso recém-criado pode ser acessado.
- **Exemplo:** `Location: https://loja-ficticia.test/minha-conta`

##### 7. `Server` e `X-Powered-By`
- **O que são e para que servem:** informam qual software de servidor Web/proxy reverso (**`Server`**) e qual tecnologia/framework de aplicação (**`X-Powered-By`**) geraram a resposta.
- **Exemplos (Inseguros — excesso de informação):**
  ```http
  Server: Apache/2.4.49 (Unix) OpenSSL/1.1.1k
  X-Powered-By: Express
  X-Powered-By: PHP/8.1.2-1ubuntu2.14
  ```
- **Segurança:** não trazem benefício funcional ao usuário final e entregam de bandeja para scanners automatizados o modelo e a versão exata da pilha tecnológica (*Banner Grabbing* — veja [1.6.4](#s1-6-4)).

##### 8. `Vary`
- **O que é e para que serve:** instrui caches (especialmente CDNs e proxies compartilhados) sobre **quais cabeçalhos da requisição foram levados em conta pelo servidor para escolher ou montar aquela resposta**.
- **Exemplos e importância crítica:**
  - `Vary: Accept-Encoding`: diz à CDN para guardar cópias separadas para clientes que aceitam `br`, `gzip` ou sem compressão.
  - **`Vary: Origin`:** **obrigatório** quando o servidor retorna um `Access-Control-Allow-Origin` dinâmico (baseado em uma *allowlist* de origens)! Se o servidor esquecer `Vary: Origin`, uma CDN pode guardar em cache a resposta com `Access-Control-Allow-Origin: https://app1.loja-ficticia.test` e entregá-la depois para `https://app2.loja-ficticia.test`, quebrando o CORS ou causando *Cache Poisoning*.

##### 9. Cabeçalhos de CORS (`Access-Control-*`)
- **O que são e para que servem:** controlam quais origens externas podem ler respostas da aplicação no navegador (detalhes em [1.11](#s1) e [5.10/5.11](./05-headers-de-seguranca.md#s5)):
  - `Access-Control-Allow-Origin`: especifica a origem permitida (`https://app.loja-ficticia.test`) ou `*` (apenas para recursos públicos sem credenciais).
  - `Access-Control-Allow-Credentials: true`: permite que o código JS da origem autorizada leia a resposta quando a requisição inclui cookies ou `Authorization`.
  - `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Expose-Headers`, `Access-Control-Max-Age`.

##### 10. `WWW-Authenticate`
- **O que é e para que serve:** enviado obrigatoriamente em respostas **`401 Unauthorized`** (RFC 9110) para informar ao cliente qual esquema de autenticação deve ser utilizado para acessar o recurso.
- **Exemplo:** `WWW-Authenticate: Bearer realm="api", error="invalid_token"` (Obs.: se enviado como `WWW-Authenticate: Basic realm="Area Restrita"`, faz o navegador abrir aquela caixa de diálogo nativa de usuário e senha).

##### 11. `Retry-After`
- **O que é e para que serve:** enviado em respostas **`429 Too Many Requests`** (Rate Limiting), **`503 Service Unavailable`** (manutenção) ou `3xx` para indicar quantos segundos (ex.: `Retry-After: 60`) ou até qual data HTTP (`Retry-After: Fri, 30 Oct 2026 12:00:00 GMT`) o cliente deve esperar antes de fazer uma nova tentativa.

##### 12. `Content-Disposition`
- **O que é e para que serve:** controla se o navegador deve exibir o arquivo diretamente dentro da aba (**`inline`**) ou forçar a caixa de **download para salvar em disco (`attachment`)**, sugerindo o nome do arquivo (RFC 6266).
- **Exemplo:**
  ```http
  Content-Disposition: attachment; filename="fatura-outubro-2026.pdf"
  ```
- **Segurança:** quando uma aplicação permite que usuários façam upload de arquivos (PDFs, textos, SVGs, imagens) e outros usuários os baixem, servir esses arquivos em um domínio separado (*sandbox domain*, ex.: `userdata.loja-ficticia.test`) com **`Content-Disposition: attachment`** e `X-Content-Type-Options: nosniff` impede que um arquivo malicioso (como um `.svg` ou `.html` disfarçado contendo JavaScript) seja renderizado `inline` no navegador da vítima e execute um **Stored XSS**!

##### 13. `Transfer-Encoding`
- **O que é e para que serve:** cabeçalho de salto único (*hop-by-hop*) específico do **HTTP/1.1** (RFC 9112) que especifica como o corpo da mensagem é codificado para transferência entre dois nós. O valor clássico é **`Transfer-Encoding: chunked`** (transferência em blocos): quando o servidor começa a enviar uma resposta gerada dinamicamente (ou streaming) **sem saber de antemão qual será o tamanho total final (`Content-Length`)**, ele omite o `Content-Length` e envia o corpo em pedaços (*chunks*), onde cada pedaço é precedido pelo seu tamanho em hexadecimal, encerrando com um bloco de tamanho `0\r\n\r\n`. (No HTTP/2 e no HTTP/3, `Transfer-Encoding: chunked` não existe mais porque o próprio protocolo já divide tudo em *frames* binários nativamente).
- **Segurança (*HTTP Request Smuggling*):** se uma requisição maliciosa contiver **simultaneamente** `Content-Length` e `Transfer-Encoding: chunked` (ou variações ofuscadas deste cabeçalho), e o proxy reverso na borda usar um deles enquanto o servidor back-end usar o outro para delimitar onde termina uma requisição e começa a seguinte, ocorre a vulnerabilidade crítica de **HTTP Request Smuggling (Contrabando de Requisição HTTP)** — permitindo que um atacante injete requisições clandestinas na fila de outros usuários!
  - **[REQUISITO]** Manter proxies reversos e servidores de aplicação atualizados, normalizar tráfego na borda (ex.: usar HTTP/2 fim a fim ou rejeitar estritamente qualquer requisição HTTP/1.1 que contenha tanto `Content-Length` quanto `Transfer-Encoding`, conforme exige a RFC 9112).

<a id="s1-6-4"></a>
#### 1.6.4 Problemas de segurança em headers e configuração segura

Na auditoria de uma aplicação (e nas questões de prova de Cybersecurity), quatro vulnerabilidades ligadas diretamente à manipulação ou exposição de headers HTTP merecem atenção especial:

##### 1. Banner Grabbing e Exposição de Versão em `Server` e `X-Powered-By`

- **Como ocorre:** por padrão, servidores Web e frameworks anunciam orgulhosamente seu nome e versão exata nos cabeçalhos de resposta:
  ```http
  Server: nginx/1.18.0 (Ubuntu)
  X-Powered-By: Express
  X-AspNet-Version: 4.0.30319
  ```
- **Qual é o risco:** na técnica de reconhecimento chamada **Banner Grabbing** (captura de banner), atacantes e *bots* varrem milhões de IPs coletando esses cabeçalhos. Se amanhã sair uma vulnerabilidade crítica (CVE) para `nginx/1.18.0` ou uma versão específica do PHP, o atacante filtra instantaneamente todos os servidores vulneráveis sem precisar adivinhar a tecnologia. Embora ocultar a versão não substitua a aplicação de *patches* (não é segurança apenas por obscuridade), é uma medida fundamental de **defesa em profundidade** para dificultar o reconhecimento automatizado.
- **Como corrigir:**
  - **[RECOMENDAÇÃO]** No **Nginx**: adicionar `server_tokens off;` no bloco `http` (passa a exibir apenas `Server: nginx` sem o número da versão).
  - **[RECOMENDAÇÃO]** No **Apache**: configurar `ServerTokens Prod` e `ServerSignature Off`.
  - **[RECOMENDAÇÃO]** No **Express (Node.js)**: chamar `app.disable("x-powered-by")` (ou usar o middleware `helmet()`, que já remove `X-Powered-By` automaticamente).
  - **[RECOMENDAÇÃO]** No **PHP (`php.ini`)**: definir `expose_php = Off`.

##### 2. CRLF Injection e HTTP Response Splitting

- **Como ocorre:** como vimos em [1.3.5](#s1-3-5), no HTTP/1.1 cada cabeçalho é separado do próximo pela sequência **CRLF** (`\r\n`, bytes `0x0D 0x0A`, ou `%0d%0a` quando codificado na URL), e dois CRLFs seguidos (`\r\n\r\n`) encerram os cabeçalhos e iniciam o corpo da página HTML! Se uma aplicação pegar um parâmetro controlado pelo usuário (ex.: `?idioma=pt-BR` ou `?redirect=/home`) e concatená-lo diretamente no valor de um header de resposta (`Set-Cookie` ou `Location`) sem filtrar quebras de linha `\r` e `\n`, o atacante pode injetar `%0d%0a`:
  ```text
  Parâmetro enviado pelo atacante:
  ?tema=escuro%0d%0aSet-Cookie:%20sessao=FIXADA_PELO_ATACANTE%0d%0a%0d%0a<script>alert(1)</script>
  ```
  Se o servidor vulnerável concatenar isso sem validação, a resposta HTTP será "partida ao meio" (**HTTP Response Splitting**):
  ```http
  HTTP/1.1 200 OK
  Set-Cookie: pref=escuro
  Set-Cookie: sessao=FIXADA_PELO_ATACANTE

  <script>alert(1)</script>...
  ```
- **Qual é o risco:** permite injetar cabeçalhos arbitrários (fixação de sessão via `Set-Cookie`, desativação de filtros de segurança, envenenamento de cache) e até injetar um corpo HTML inteiro executando **XSS**!
- **Como corrigir:**
  - **[REQUISITO]** Utilizar sempre as APIs nativas modernas do framework para definir headers e redirecionamentos (Node.js, Python/Django/Flask, Go, Java/Spring e servidores HTTP/2+ já lançam erro automaticamente se encontrarem caracteres `\r` ou `\n` dentro do nome ou valor de um header) e validar estritamente qualquer entrada de usuário antes de incluí-la em cabeçalhos.

##### 3. Host Header Injection (Injeção do Cabeçalho `Host`)

- **Como ocorre:** muitas aplicações precisam gerar links absolutos (especialmente ao enviar **e-mails de recuperação de senha**: *"Clique aqui para redefinir sua senha: `https://.../reset?token=XYZ`"*). Um erro clássico e perigoso de programação é montar esse link lendo o domínio diretamente do cabeçalho `Host` da requisição HTTP (`https://${req.headers.host}/reset?token=${token}`).
- **Como o ataque funciona (Password Reset Poisoning):**
  1. O atacante vai à tela de "Esqueci minha senha", digita o e-mail da vítima (`vitima@empresa.test`) e intercepta a requisição `POST /recuperar-senha` alterando o cabeçalho para `Host: site-do-atacante.example` (ou adicionando `X-Forwarded-Host: site-do-atacante.example`).
  2. Se o servidor confiar no cabeçalho `Host` recebido, ele enviará um e-mail legítimo para a vítima contendo o link `https://site-do-atacante.example/reset?token=SEGREDO_DA_VITIMA`!
  3. Quando a vítima clicar no link dentro do e-mail oficial, o navegador dela enviará o `token` de redefinição diretamente para o servidor do atacante, que toma a conta da vítima na hora!
  - Outros impactos de *Host Header Injection* incluem **Web Cache Poisoning** (envenenar links de scripts/imagens em páginas em cache na CDN) e **SSRF** em proxies reversos mal configurados que roteiam requisições com base em qualquer `Host` informado.
- **Como corrigir:**
  - **[REQUISITO]** Nunca usar `req.headers.host` ou `X-Forwarded-Host` para construir URLs em e-mails de recuperação de senha ou redirecionamentos; utilizar uma variável de configuração fixa do ambiente no servidor (ex.: `APP_PUBLIC_URL="https://loja-ficticia.test"`).
  - **[REQUISITO]** Configurar o servidor Web (Nginx/Apache) e o framework (ex.: `ALLOWED_HOSTS` no Django) para aceitar **apenas** os nomes de domínio oficiais da aplicação e definir um bloco `default_server` no Nginx que descarte (`return 444;`) requisições com header `Host` desconhecido.

##### 4. Falsificação de `X-Forwarded-For` (*IP Spoofing via Header*)

- **Como ocorre e como corrigir:** como vimos em [1.6.2](#s1-6-2), qualquer cliente externo pode enviar cabeçalhos `X-Forwarded-For`, `X-Real-IP` ou `True-Client-IP` forjados na requisição.
  - **[REQUISITO]** Configurar a aplicação back-end (ex.: diretiva `app.set('trust proxy', ...)` no Express, `ForwardedHeadersMiddleware` no ASP.NET Core ou `real_ip_header` + `set_real_ip_from` no Nginx) para **só aceitar cabeçalhos `X-Forwarded-*` quando a conexão TCP vier comprovadamente do IP interno do seu balanceador de carga / proxy reverso**, e avaliar a lista `X-Forwarded-For` da **direita para a esquerda** (pegando o primeiro IP adicionado pelo seu proxy confiável na borda, e nunca o primeiro IP da esquerda que foi escrito pelo cliente!).

##### 5. Visão geral dos Headers de Segurança HTTP

Além dos cabeçalhos funcionais estudados nesta seção, a defesa moderna de aplicações Web no navegador apoia-se em um conjunto dedicado de **Headers de Segurança de Resposta**, que serão estudados um a um, com exemplos de configuração em Nginx, Apache e Express, na **[Seção 5 — Headers de Segurança HTTP](./05-headers-de-seguranca.md#s5)**:

| Header de Segurança | Defesa Principal que Proporciona | Referência |
|---|---|---|
| **`Content-Security-Policy` (CSP)** | Mitiga **XSS** e injeção de dados restringindo de quais origens scripts, estilos, imagens, conexões e frames podem ser carregados | [Seção 5.1](./05-headers-de-seguranca.md#s5) |
| **`Strict-Transport-Security` (HSTS)** | Obriga o navegador a usar **exclusivamente HTTPS**, prevenindo *SSL Stripping* e downgrade de protocolo | [Seção 5.2](./05-headers-de-seguranca.md#s5) |
| **`X-Content-Type-Options: nosniff`** | Proíbe o navegador de adivinhar o tipo MIME (*MIME Sniffing*), evitando execução acidental de arquivos como HTML/JS | [Seção 5.3](./05-headers-de-seguranca.md#s5) |
| **`X-Frame-Options`** (e `CSP: frame-ancestors`) | Impede que a página seja embutida em `<iframe>` de outros sites, prevenindo **Clickjacking (UI Redressing)** | [Seção 5.4](./05-headers-de-seguranca.md#s5) |
| **`Referrer-Policy`** | Controla quanta informação da URL atual é enviada no header `Referer` ao navegar para outros sites | [Seção 5.5](./05-headers-de-seguranca.md#s5) |
| **`Permissions-Policy`** | Restringe o acesso da página e de iframes a APIs sensíveis do navegador (`camera`, `microphone`, `geolocation`, `usb`) | [Seção 5.6](./05-headers-de-seguranca.md#s5) |
| **`Cross-Origin-Opener-Policy` (COOP)** | Isola a janela da página (`window.opener`) contra janelas *cross-origin*, mitigando ataques XS-Leaks e *Reverse Tabnabbing* | [Seção 5.7](./05-headers-de-seguranca.md#s5) |
| **`Cross-Origin-Resource-Policy` (CORP)** | Controla quais origens podem incorporar aquele recurso específico (`same-origin`, `same-site`, `cross-origin`) | [Seção 5.8](./05-headers-de-seguranca.md#s5) |
| **`Cross-Origin-Embedder-Policy` (COEP)** | Exige permissão explícita (CORP/CORS) de todos os sub-recursos carregados para habilitar isolamento *cross-origin* | [Seção 5.9](./05-headers-de-seguranca.md#s5) |

> [!TIP]
> Na prova e em auditorias reais com o Chrome DevTools ([seção 2.1](./02a-devtools-network.md#s2-1) e [seção 6](./06-checklist-e-comparativo.md#s6)), sempre clique na requisição principal do documento HTML (filtro **Doc**) na aba **Network** e inspecione os **Response Headers**: é ali que os headers de segurança (`Content-Security-Policy`, `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`) causam seu maior impacto de proteção!

---

---

> *(Início)* | [🏠 **Voltar ao Sumário Principal**](../README.md) | [1B. Fundamentos Web (1.7 a 1.13) ➡️](./01b-fundamentos-web-parte2.md)
