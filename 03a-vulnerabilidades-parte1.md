> [⬅️ 2.3 Chrome DevTools — Aba Console](./02c-devtools-console.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [3B. Vulnerabilidades Web (3.10 a 3.19) ➡️](./03b-vulnerabilidades-parte2.md)

---

<a id="s3"></a>
## 3. Cybersecurity: Vulnerabilidades Web

A **Segurança da Informação (Cybersecurity)** aplicada ao desenvolvimento web estuda como proteger aplicações, APIs (Application Programming Interfaces — Interfaces de Programação de Aplicações), servidores, bancos de dados e usuários contra acessos não autorizados, manipulação de dados, exfiltração de informações confidenciais e interrupção de serviços. Enquanto nas seções anteriores estudamos o funcionamento estrutural da Web ([Seção 1](./01a-fundamentos-web-parte1.md#s1)) e a instrumentação de diagnóstico no navegador com o Chrome DevTools ([Seção 2](./02a-devtools-network.md#s2)), nesta seção analisaremos como falhas de projeto, implementação ou configuração transformam recursos legítimos do protocolo HTTP (Hypertext Transfer Protocol — Protocolo de Transferência de Hipertexto), do DOM (Document Object Model — Modelo de Objeto de Documento) e do banco de dados em vetores de ataque.

Na literatura acadêmica e nas normas técnicas internacionais — como a família **ISO/IEC 27000** (International Organization for Standardization / International Electrotechnical Commission) e os guias do **NIST** (National Institute of Standards and Technology — Instituto Nacional de Padrões e Tecnologia dos EUA) —, a proteção de um sistema apoia-se fundamentalmente na chamada **Tríade CID** (em inglês, **CIA Triad**):

1. **Confidencialidade (*Confidentiality*)**: garantia de que a informação seja acessível exclusivamente às entidades (usuários, processos ou sistemas) devidamente autorizadas. Uma falha que permite ler o CPF ou o cartão de crédito de terceiros viola a confidencialidade.
2. **Integridade (*Integrity*)**: salvaguarda da exatidão e da completude da informação, impedindo que dados ou fluxos de execução sejam alterados, inseridos ou excluídos de maneira não autorizada ou acidental. Um ataque que modifica o saldo de uma conta ou injeta código JavaScript malicioso em uma página viola a integridade.
3. **Disponibilidade (*Availability*)**: garantia de que usuários autorizados obtenham acesso tempestivo e confiável aos dados e aos recursos computacionais sempre que necessário. Ataques de negação de serviço — **DoS** (Denial of Service) ou **DDoS** (Distributed Denial of Service — Negação de Serviço Distribuída) — ou falhas que travam o banco de dados violam a disponibilidade.

Além da tríade clássica, arquiteturas modernas exigem **Autenticidade** (certeza da identidade da origem de uma mensagem ou sessão) e **Não Repúdio** (ou irretratabilidade — impossibilidade de uma entidade negar a autoria de uma ação registrada em trilha de auditoria íntegra).

Para dominar a análise de segurança em provas acadêmicas e auditorias técnicas, é indispensável distinguir quatro conceitos fundamentais que frequentemente são confundidos:

| Conceito | Definição Técnica | Exemplo no Contexto Web |
| :--- | :--- | :--- |
| **Ativo (*Asset*)** | Qualquer dado, componente de software, infraestrutura ou processo que possua valor para a organização ou para os usuários. | Banco de dados de clientes, chaves privadas TLS (Transport Layer Security — Segurança da Camada de Transporte), tokens de sessão, reputação institucional. |
| **Vulnerabilidade (*Vulnerability*)** | Fraqueza ou falha no design, na implementação, na operação ou na configuração de um ativo que pode ser explorada por uma ameaça. | Concatenação direta de variáveis de entrada do usuário em uma instrução SQL (Structured Query Language — Linguagem de Consulta Estruturada). |
| **Ameaça (*Threat*)** | Qualquer circunstância, evento ou agente (externo ou interno, intencional ou acidental) com potencial de causar dano a um ativo explorando uma vulnerabilidade. | Atacante malicioso, botnet automatizada de varredura, insider mal-intencionado. |
| **Exploit** | Código, sequência de comandos, requisição HTTP artesanal ou técnica específica construída para tirar proveito prático de uma vulnerabilidade. | Envio do payload `' UNION SELECT username, password_hash FROM users--` em um parâmetro vulnerável. |
| **Risco (*Risk*)** | Combinação da **probabilidade** de uma ameaça explorar uma vulnerabilidade com o **impacto** resultante sobre o negócio e os titulares de dados. | Alto risco de vazamento em massa de dados pessoais sujeito a sanções legais e perdas financeiras. |

Para catalogar e padronizar o estudo dessas falhas mundialmente, a comunidade de segurança utiliza dois identificadores essenciais mantidos pela corporação **MITRE** com apoio governamental e comunitário:
- **CWE (Common Weakness Enumeration — Enumeração de Fraquezas Comuns)**: dicionário que classifica os *tipos abstratos de fraquezas* de software ou hardware (por exemplo, **CWE-89** para SQL Injection, **CWE-79** para Cross-Site Scripting, **CWE-352** para Cross-Site Request Forgery).
- **CVE (Common Vulnerabilities and Exposures — Vulnerabilidades e Exposições Comuns)**: registro de *ocorrências específicas e públicas* de uma vulnerabilidade descoberta em um produto ou versão concreta (por exemplo, `CVE-2021-44228`, conhecida como *Log4Shell* na biblioteca Apache Log4j).

> [!CAUTION]
> **Ética Profissional, Legalidade e Uso Exclusivo em Laboratório Autorizado:**
> Todo o conhecimento técnico apresentado nesta apostila tem finalidade **estritamente acadêmica, defensiva e educacional**. Executar testes de intrusão (*pentests*), enviar payloads de teste ou manipular parâmetros para acessar dados de terceiros em aplicações reais na Internet **sem autorização prévia, expressa e documentada do proprietário do sistema é crime**.
> No Brasil, o **Art. 154-A do Código Penal** (incluído pela **Lei nº 12.737/2012** — conhecida como Lei Carolina Dieckmann — e atualizado com penas mais severas pela **Lei nº 14.155/2021**) tipifica o crime de *Invasão de Dispositivo Informático*, punindo quem invade dispositivo alheio, conectado ou não à rede de computadores, com o fim de obter, adulterar ou destruir dados ou informações sem autorização expressa ou tácita do titular, ou instalar vulnerabilidades para obter vantagem ilícita. Além disso, a exposição de dados pessoais infringe a **LGPD (Lei Geral de Proteção de Dados Pessoais — Lei nº 13.709/2018)**.
>
> **[REQUISITO]** Pratique os procedimentos de identificação e verificação descritos nesta seção **exclusivamente** em ambientes locais e plataformas de treinamento legalmente projetadas para esse fim:
> 1. **OWASP Juice Shop**: aplicação moderna em Node.js, Express e Angular repleta de desafios práticos cobrindo todo o OWASP Top 10 (executável localmente via Docker em `http://localhost:3000`).
> 2. **OWASP WebGoat**: ambiente didático interativo em Java com lições guiadas passo a passo sobre falhas de segurança web.
> 3. **DVWA (Damn Vulnerable Web Application)**: aplicação clássica em PHP/MySQL com níveis ajustáveis de segurança (*Low*, *Medium*, *High*, *Impossible*) para comparar código vulnerável e código corrigido.
> 4. **PortSwigger Web Security Academy**: plataforma online gratuita com laboratórios isolados sob demanda para cada classe de vulnerabilidade web.

---

<a id="s3-0"></a>
### 3.0 Panorama do OWASP Top 10

A **OWASP (Open Worldwide Application Security Project)** é uma fundação internacional sem fins lucrativos, aberta e comunitária, dedicada a melhorar a segurança de softwares em todo o mundo. Seus projetos, ferramentas, guias de teste (**OWASP WSTG — Web Security Testing Guide**) e padrões de verificação arquitetural (**OWASP ASVS — Application Security Verification Standard**) constituem a principal referência técnica acadêmica e corporativa na área de segurança de aplicações.

O documento mais célebre da fundação é o **OWASP Top 10**, um relatório de conscientização atualizado periodicamente que classifica as **dez categorias mais críticas de riscos de segurança em aplicações web**. A classificação não se baseia apenas na opinião de especialistas, mas na combinação de dados empíricos de centenas de milhares de aplicações testadas (telemetria de ferramentas e relatórios de auditoria mapeados por CWEs) com pesquisas globais da indústria.

<a id="s3-0-1"></a>
#### 3.0.1 Evolução do OWASP Top 10: de 2021 para 2025

Compreender a transição entre as edições **OWASP Top 10:2021** e **OWASP Top 10:2025** é fundamental para provas e concursos, pois demonstra como o cenário de ameaças evoluiu: categorias antes isoladas foram consolidadas em causas-raiz mais amplas, e novos riscos ligados à cadeia de suprimentos de software e ao tratamento inadequado de exceções ganharam destaque.

A tabela a seguir compara lado a lado as dez categorias oficiais das edições **2021** e **2025**:

| Posição | OWASP Top 10:2021 | OWASP Top 10:2025 | Principais Mudanças e Observações Técnicas |
| :---: | :--- | :--- | :--- |
| **A01** | **A01:2021 – Broken Access Control** (Quebra de Controle de Acesso) | **A01:2025 – Broken Access Control** (Quebra de Controle de Acesso) | Permanece em 1º lugar como o risco mais grave e frequente. Na edição **2025**, a categoria **incorporou o SSRF** (Server-Side Request Forgery, que era A10 em 2021), reconhecendo que forçar o servidor a acessar recursos internos não autorizados é, em essência, uma falha de controle de acesso e isolamento. |
| **A02** | **A02:2021 – Cryptographic Failures** (Falhas Criptográficas) | **A02:2025 – Security Misconfiguration** (Configuração Incorreta de Segurança) | *Security Misconfiguration* subiu de A05 (2021) para **A02 (2025)** devido à crescente complexidade de configurações em nuvem, containers, CORS, headers e frameworks modernos. |
| **A03** | **A03:2021 – Injection** (Injeção) | **A03:2025 – Software Supply Chain Failures** (Falhas na Cadeia de Suprimentos de Software) | Evolução e expansão significativa da antiga categoria A06:2021 (*Vulnerable and Outdated Components*). Passa a cobrir não apenas bibliotecas desatualizadas, mas também pacotes maliciosos em gerenciadores (npm, PyPI), comprometimento de pipelines de CI/CD (Continuous Integration / Continuous Deployment) e dependências transitivas. |
| **A04** | **A04:2021 – Insecure Design** (Design Inseguro) | **A04:2025 – Cryptographic Failures** (Falhas Criptográficas) | Passou de A02 (2021) para **A04 (2025)**. Continua englobando ausência de TLS, algoritmos obsoletos (MD5, SHA-1), gerenciamento precário de chaves e uso de geradores pseudoaleatórios inseguros. |
| **A05** | **A05:2021 – Security Misconfiguration** (Configuração Incorreta de Segurança) | **A05:2025 – Injection** (Injeção) | Passou de A03 (2021) para **A05 (2025)**. Agrupa SQL Injection, Cross-Site Scripting (XSS), Command Injection, LDAP Injection e Template Injection. A queda relativa na posição reflete o uso mais difundido de ORMs e frameworks modernos com escape automático, embora o impacto continue crítico quando ocorre. |
| **A06** | **A06:2021 – Vulnerable and Outdated Components** (Componentes Vulneráveis e Desatualizados) | **A06:2025 – Insecure Design** (Design Inseguro) | Passou de A04 (2021) para **A06 (2025)**. Foca em falhas conceituais e de regra de negócio (ausência de modelagem de ameaças, fluxos sem proteção contra automação/abuso e condições de corrida arquiteturais). |
| **A07** | **A07:2021 – Identification and Authentication Failures** (Falhas de Identificação e Autenticação) | **A07:2025 – Authentication Failures** (Falhas de Autenticação) | Nome simplificado em 2025, mantendo a posição **A07**. Abrange força bruta, *credential stuffing*, gerenciamento inseguro de sessões, tokens previsíveis e falta de MFA (Multi-Factor Authentication — Autenticação Multifator). |
| **A08** | **A08:2021 – Software and Data Integrity Failures** (Falhas de Integridade de Software e Dados) | **A08:2025 – Software or Data Integrity Failures** (Falhas de Integridade de Software ou Dados) | Mantém a posição **A08**. Cobre desserialização insegura, carregamento de scripts de CDNs (Content Delivery Networks) sem verificação de integridade (SRI — Subresource Integrity) e atualizações sem validação de assinatura digital. |
| **A09** | **A09:2021 – Security Logging and Monitoring Failures** (Falhas de Registro e Monitoramento de Segurança) | **A09:2025 – Security Logging and Alerting Failures** (Falhas de Registro e Alerta de Segurança) | Mantém a posição **A09**, mas enfatiza a palavra **Alerting** (alerta): não basta gravar logs passivamente se a equipe de resposta não receber alertas acionáveis em tempo hábil durante um ataque. |
| **A10** | **A10:2021 – Server-Side Request Forgery (SSRF)** | **A10:2025 – Mishandling of Exceptional Conditions** (Tratamento Inadequado de Condições Excepcionais) | **Nova categoria em 2025** (enquanto SSRF foi integrado em A01:2025). Aborda erros não tratados, falhas em aberto (*fail-open*, onde uma exceção concede acesso em vez de bloqueá-lo), vazamento de estados inconsistentes e esgotamento de recursos por tratamento deficiente de erros. |

> [!NOTE]
> **E o OWASP API Security Top 10?**
> Além do Top 10 voltado a aplicações web gerais, a OWASP publica um ranking específico para APIs REST/GraphQL (como o **OWASP API Security Top 10:2023**). Nele, a vulnerabilidade número 1 (**API1:2023**) é o **BOLA (Broken Object Level Authorization)**, que é o equivalente moderno em APIs ao clássico **IDOR (Insecure Direct Object Reference)** estudado no item [3.9](#s3-9).

<a id="s3-0-2"></a>
#### 3.0.2 Mapeamento das Vulnerabilidades desta Seção para o OWASP Top 10

Para facilitar o estudo integrado, a tabela abaixo mostra como as vulnerabilidades detalhadas nesta seção (itens 3.1 a 3.9, e 3.10 a 3.18 na continuação da seção) se encaixam nas categorias do OWASP Top 10 (2021 e 2025) e em seus respectivos identificadores CWE:

| Item na Apostila | Vulnerabilidade | Categoria OWASP Top 10:2021 | Categoria OWASP Top 10:2025 | CWE Principal |
| :--- | :--- | :--- | :--- | :--- |
| **[3.1](#s3-1)** | SQL Injection (SQLi) | A03:2021 – Injection | A05:2025 – Injection | CWE-89 |
| **[3.2](#s3-2)** | Cross-Site Scripting (XSS) | A03:2021 – Injection | A05:2025 – Injection | CWE-79 |
| **[3.3](#s3-3)** | Cross-Site Request Forgery (CSRF) | A01:2021 – Broken Access Control | A01:2025 – Broken Access Control | CWE-352 |
| **[3.4](#s3-4)** | Server-Side Request Forgery (SSRF) | A10:2021 – SSRF | A01:2025 – Broken Access Control | CWE-918 |
| **[3.5](#s3-5)** | Broken Access Control | A01:2021 – Broken Access Control | A01:2025 – Broken Access Control | CWE-284 / CWE-285 |
| **[3.6](#s3-6)** | Authentication Failures | A07:2021 – Auth Failures | A07:2025 – Authentication Failures | CWE-287 |
| **[3.7](#s3-7)** | Security Misconfiguration | A05:2021 – Security Misconfiguration | A02:2025 – Security Misconfiguration | CWE-16 / CWE-209 |
| **[3.8](#s3-8)** | Cryptographic Failures | A02:2021 – Cryptographic Failures | A04:2025 – Cryptographic Failures | CWE-310 / CWE-327 |
| **[3.9](#s3-9)** | IDOR / BOLA | A01:2021 – Broken Access Control | A01:2025 – Broken Access Control | CWE-639 |
| **3.10 / 3.17** | Directory / Path Traversal | A01:2021 – Broken Access Control | A01:2025 – Broken Access Control | CWE-22 |
| **3.11** | File Upload Vulnerabilities | A04:2021 / A03:2021 | A06:2025 / A05:2025 | CWE-434 |
| **3.12** | Clickjacking (UI Redressing) | A05:2021 – Security Misconfiguration | A02:2025 – Security Misconfiguration | CWE-1021 |
| **3.13** | Open Redirect | A01:2021 – Broken Access Control | A01:2025 – Broken Access Control | CWE-601 |
| **3.14** | Session Hijacking | A07:2021 – Auth Failures | A07:2025 – Authentication Failures | CWE-384 / CWE-522 |
| **3.15** | Session Fixation | A07:2021 – Auth Failures | A07:2025 – Authentication Failures | CWE-384 |
| **3.16** | Command Injection | A03:2021 – Injection | A05:2025 – Injection | CWE-78 |
| **3.18** | Race Conditions (TOCTOU) | A04:2021 – Insecure Design | A06:2025 – Insecure Design | CWE-362 / CWE-367 |

---

<a id="s3-1"></a>
### 3.1 SQL Injection (SQLi)

##### 1. Definição

**SQL Injection (SQLi — Injeção de SQL)**, catalogada sob a **CWE-89** e integrante da categoria **Injection** (**A03:2021** / **A05:2025** no OWASP Top 10), é uma vulnerabilidade crítica que ocorre quando dados não confiáveis fornecidos pelo usuário são inseridos diretamente na estrutura de um comando SQL sem a devida separação entre o **plano de código (instrução)** e o **plano de dados (parâmetros)**. Como resultado, o interpretador do SGBD (Sistema Gerenciador de Banco de Dados — como PostgreSQL, MySQL, MariaDB, Microsoft SQL Server ou Oracle) passa a tratar a entrada do usuário como parte da própria sintaxe do comando SQL, permitindo que um atacante altere a lógica da consulta original.

O impacto de uma vulnerabilidade de SQL Injection é frequentemente catastrófico: ela pode permitir contornar mecanismos de autenticação (*login bypass*), ler dados arbitrários de qualquer tabela do banco (violação de confidencialidade), modificar ou apagar registros (violação de integridade) e, em configurações mal isoladas, ler arquivos do sistema operacional ou até executar comandos no servidor de banco de dados — levando ao comprometimento total da infraestrutura (**RCE — Remote Code Execution**, ou Execução Remota de Código).

##### 2. Como funciona

Em uma aplicação web típica, o servidor back-end recebe parâmetros vindos do cliente (via query string da URL, corpo JSON de um `POST`, cabeçalhos HTTP como `User-Agent`/`Cookie` ou campos de formulário) e realiza consultas ao banco de dados. Quando o desenvolvedor constrói a consulta concatenando ou interpolando strings, os caracteres especiais da linguagem SQL — como aspas simples (`'`), ponto e vírgula (`;`), operadores booleanos (`OR`, `AND`), palavras-chave (`UNION`, `SELECT`, `SLEEP`) e sequências de comentário (`--`, `#`, `/* */`) — ganham poder sintático.

Suponha que um back-end execute a seguinte consulta para autenticar um usuário ou buscar um produto:

```sql
SELECT id, nome, preco FROM produtos WHERE categoria = '$entrada_usuario' AND ativo = 1;
```

Se o usuário enviar o valor legítimo `eletronicos`, a consulta resultante será:

```sql
SELECT id, nome, preco FROM produtos WHERE categoria = 'eletronicos' AND ativo = 1;
```

Porém, se o usuário enviar a sequência didática `' OR '1'='1' -- `, a string interpolada transforma a árvore sintática da consulta em:

```sql
SELECT id, nome, preco FROM produtos WHERE categoria = '' OR '1'='1' -- ' AND ativo = 1;
```

Analisando como o SGBD processa essa instrução adulterada:
1. O primeiro caractere `'` **fecha prematuramente** o literal de string iniciado antes de `$entrada_usuario`.
2. O operador lógico `OR '1'='1'` introduz uma condição booleana **universalmente verdadeira** (tautologia), fazendo com que a cláusula `WHERE` seja avaliada como `TRUE` para todas as linhas da tabela.
3. O marcador de comentário `-- ` (seguido de espaço, padrão SQL ANSI e exigido no MySQL) instrui o banco de dados a **ignorar todo o restante da linha**, descartando a verificação `AND ativo = 1` e evitando um erro de sintaxe causado pela aspa simples original que sobraria no final.

Na literatura técnica, o SQL Injection é dividido em **três classes principais**, conforme o canal pelo qual os dados são extraídos:

| Classe de SQLi | Subtipo | Funcionamento Técnico |
| :--- | :--- | :--- |
| **1. In-Band SQLi (Clássico)** | **Error-Based (Baseado em Erro)** | O atacante força o banco de dados a gerar um erro sintático ou de conversão de tipo cujos detalhes (ex.: versão do banco ou resultado de uma subconsulta) são exibidos diretamente na resposta HTTP enviada ao navegador. |
| **1. In-Band SQLi (Clássico)** | **Union-Based (Baseado em UNION)** | Utiliza o operador `UNION SELECT` para concatenar os resultados da consulta original com os resultados de uma segunda consulta escolhida pelo atacante (ex.: extrair colunas da tabela `usuarios` ou do catálogo `information_schema`). Exige que ambas as consultas tenham o mesmo número de colunas e tipos de dados compatíveis. |
| **2. Inferential SQLi (Blind / Cego)** | **Boolean-Based Blind** | A aplicação não exibe dados do banco nem mensagens de erro na tela, mas muda sutilmente seu comportamento (ex.: retorna HTTP `200` com "Produto disponível" quando a condição injetada é `TRUE` e HTTP `404` quando é `FALSE`). O atacante infere os dados caractere por caractere fazendo perguntas de "sim ou não" (ex.: `SUBSTRING(senha,1,1) = 'a'`). |
| **2. Inferential SQLi (Blind / Cego)** | **Time-Based Blind** | A resposta visual da aplicação é idêntica em qualquer caso. O atacante injeta funções que atrasam a execução do banco caso uma condição seja verdadeira (ex.: `pg_sleep(5)` no PostgreSQL ou `SLEEP(5)` no MySQL) e mede o **TTFB (Time To First Byte — tempo até o primeiro byte)** da resposta. |
| **3. Out-of-Band SQLi (Fora de Banda)** | **DNS / HTTP Exfiltration** | Usado quando a resposta da requisição é assíncrona ou totalmente silenciosa. O comando SQL força o servidor de banco de dados a realizar uma resolução DNS ou requisição HTTP para um domínio controlado pelo testador, exfiltrando os dados no subdomínio consultado. |

> [!NOTE]
> **SQL Injection de Segunda Ordem (*Second-Order SQLi*):**
> Nem todo SQL Injection ocorre no exato momento em que o dado é enviado. No *Second-Order SQLi*, o aplicativo armazena a entrada do usuário de forma segura no banco (por exemplo, ao cadastrar o nome de usuário `admin'--`), mas posteriormente outra rotina interna lê esse valor do próprio banco de dados e, por confiar que "veio do banco", concatena-o diretamente em uma nova consulta SQL sem parametrização.

##### 3. Por que acontece

A causa-raiz do SQL Injection é a **mistura de código e dados na mesma representação textual** (falha na separação de canais de comando e entrada). Especificamente, ela decorre de:
- **Concatenação ou interpolação de strings** (`+`, `f"{var}"` em Python, template literals `` `${var}` `` em JavaScript, `.` em PHP) para montar instruções SQL dinâmicas.
- **Crença equivocada em filtros manuais (*blocklists*)**: tentar remover palavras como `SELECT` ou `UNION` com `replace()` é facilmente contornado (ex.: `SeLeCt`, `SELSELECTECT`, comentários inline `UN/**/ION`).
- **Uso incorreto de ORMs (Object-Relational Mappers — Mapeadores Objeto-Relacionais)**: embora ORMs modernos (Prisma, Sequelize, TypeORM, Hibernate, SQLAlchemy, Django ORM) usem consultas parametrizadas por padrão em seus métodos de alto nível, desenvolvedores às vezes recorrem a métodos de consulta bruta (`sequelize.query()`, `cursor.execute()`, `EntityManager.createNativeQuery()`) interpolando variáveis diretamente, ou passam parâmetros não validados em cláusulas `ORDER BY` e nomes de colunas.
- **Confiança cega em dados já armazenados ou em cabeçalhos HTTP**: achar que cookies, cabeçalhos `X-Forwarded-For` ou dados vindos de outras tabelas não podem conter metacaracteres SQL.

##### 4. Exemplo didático de código vulnerável

Considere um endpoint de API REST de uma loja fictícia de laboratório (`https://api.loja-ficticia.test/api/v1/pedidos/busca`) desenvolvido em **Node.js (Express)** com o driver `pg` (PostgreSQL). O objetivo do endpoint é listar os pedidos do usuário autenticado filtrando pelo código de status ou termo de busca:

```javascript
// [CÓDIGO VULNERÁVEL - APENAS PARA ESTUDO EM LABORATÓRIO]
const express = require('express');
const { Pool } = require('pg');
const app = express();
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

app.get('/api/v1/pedidos/busca', async (req, res) => {
  const termo = req.query.termo || '';
  const usuarioId = req.user.id; // Obtido da sessão autenticada

  // ERRO CRÍTICO: Interpolação direta de `termo` via Template Literal!
  const sqlQuery = `
    SELECT id, codigo_rastreio, valor_total, status
    FROM pedidos
    WHERE usuario_id = ${usuarioId}
      AND codigo_rastreio = '${termo}'
  `;

  try {
    const resultado = await pool.query(sqlQuery);
    return res.status(200).json({ pedidos: resultado.rows });
  } catch (erro) {
    // SEGUNDO ERRO: Exposição direta do erro do SGBD na resposta JSON (Error-Based SQLi)
    return res.status(500).json({
      erro: 'Falha ao executar consulta no banco de dados',
      detalhes_sql: erro.message,
      query_executada: sqlQuery
    });
  }
});
```

Neste código vulnerável, mesmo que o `usuarioId` venha da sessão autenticada do servidor, o parâmetro `termo` recebido em `req.query.termo` pode fechar a aspa simples e adicionar operadores `OR` ou `UNION SELECT`, anulando o filtro `WHERE usuario_id = ${usuarioId}` e expondo pedidos ou credenciais de todos os clientes do banco.

##### 5. Como identificar

A identificação de SQL Injection combina **análise de caixa-branca** (revisão de código-fonte e SAST — Static Application Security Testing) e **análise de caixa-preta** (inspeção de tráfego em ambiente autorizado):

1. **Na revisão de código (Caixa-Branca)**:
   - Procure por chamadas ao banco de dados que utilizem concatenação (`+`), interpolação de strings (`` `SELECT ... ${var}` `` em JS, `f"SELECT ... {var}"` ou `.format()` em Python) em vez de marcadores de posição (`$1, $2` ou `?`).
   - Verifique cláusulas dinâmicas onde parâmetros preparados não podem substituir identificadores SQL diretamente (como `ORDER BY ${coluna}` ou `FROM ${tabela}`), conferindo se há validação contra uma lista estrita (*allowlist*).
2. **No comportamento da aplicação (Caixa-Preta em Laboratório)**:
   - Respostas HTTP `500 Internal Server Error` acionadas quando um parâmetro recebe uma única aspa simples (`'`) ou dupla (`"`), acompanhadas de mensagens características do SGBD, tais como:
     - PostgreSQL: `ERROR: unterminated quoted string at or near "'"`
     - MySQL / MariaDB: `You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version`
     - SQLite: `SQLite3::SQLException: near "'": syntax error`
     - Microsoft SQL Server: `Unclosed quotation mark after the character string`
   - Diferença sistemática de resposta quando se envia uma condição verdadeira (`' AND '1'='1`) versus uma condição falsa (`' AND '1'='2`).

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente controlado de treinamento (como o **OWASP Juice Shop** ou um container local da aplicação acima), a verificação técnica segue um método científico e incremental:

1. **Passo 1 — Sondagem de Metacaractere (Teste de Quebra Sintática)**:
   - O estudante acessa a busca com uma aspa simples codificada na URL:
     `GET https://api.loja-ficticia.test/api/v1/pedidos/busca?termo=BR123'`
   - **Observação**: A aplicação deixa de retornar `200 OK` (lista vazia) e retorna `500 Internal Server Error` informando `unterminated quoted string`. Isso indica que a aspa não foi tratada como literal de texto, mas sim interpretada pelo parser SQL.
2. **Passo 2 — Confirmação Lógica (Par Booleano Verdadeiro vs. Falso)**:
   - O estudante envia duas requisições sequenciais para confirmar a interpretação lógica sem causar erro de sintaxe:
     - Requisição A (Falsa): `?termo=BR123' AND '1'='2` $\rightarrow$ Retorna `{"pedidos": []}` (0 registros).
     - Requisição B (Tautologia): `?termo=BR123' OR '1'='1` $\rightarrow$ Retorna todos os pedidos da tabela, inclusive de outros usuários, pois `usuario_id = X AND codigo = '' OR '1'='1'` é avaliado como `(usuario_id = X AND codigo = '') OR TRUE` (devido à precedência do operador `AND` sobre o `OR` em SQL!).
3. **Passo 3 — Verificação de Extração via `UNION` (em laboratório didático)**:
   - Sabendo que a consulta original retorna 4 colunas (`id`, `codigo_rastreio`, `valor_total`, `status`), em um laboratório didático o aluno testa:
     `?termo=' UNION SELECT 1, username, 0, email FROM usuarios-- `
   - O JSON retornado pela API passa a exibir os nomes de usuário e e-mails cadastrados na tabela `usuarios`.

##### 7. Como corrigir

**[REQUISITO]** A defesa primária e obrigatória contra SQL Injection é o uso de **Consultas Parametrizadas (*Parameterized Queries*)**, também chamadas de **Prepared Statements (Instruções Preparadas)**.

Quando utilizamos *Prepared Statements*, a comunicação entre a aplicação e o SGBD ocorre em duas etapas distintas no protocolo binário do banco:
1. Primeiro, a aplicação envia o **molde da consulta SQL** com marcadores de posição (`$1, $2` no PostgreSQL; `?` no MySQL/SQLite; `:nome` no Oracle). O banco faz o *parsing*, compila a árvore de execução da consulta e fixa sua estrutura lógica.
2. Em seguida, a aplicação envia os **valores dos parâmetros separadamente**. Como a árvore sintática já está compilada, é **matematicamente impossível** que o valor de `$1` altere a lógica do comando: se o usuário digitar `' OR '1'='1`, o banco procurará literalmente por um código de rastreio cujo texto seja `' OR '1'='1`.

Veja o código anterior **completamente corrigido**, aplicando *Prepared Statements*, validação de entrada, princípio do menor privilégio e tratamento seguro de erros:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO]
const express = require('express');
const { Pool } = require('pg');
const app = express();

// [BOA PRÁTICA] Conectar ao banco usando uma conta com Princípio do Menor Privilégio
// (apenas SELECT/INSERT/UPDATE nas tabelas da aplicação, sem permissão de SUPERUSER)
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

app.get('/api/v1/pedidos/busca', async (req, res) => {
  const termo = String(req.query.termo || '').trim();
  const usuarioId = Number(req.user.id);

  // [RECOMENDAÇÃO] Defesa em profundidade: validar formato esperado (Allowlist / Regex)
  // Ex.: códigos de rastreio têm apenas letras maiúsculas e números (até 20 caracteres)
  if (termo && !/^[A-Z0-9]{1,20}$/.test(termo)) {
    return res.status(400).json({
      erro: 'Formato de código de rastreio inválido.'
    });
  }

  // [REQUISITO] Uso de Prepared Statement com marcadores posicionais ($1, $2)
  // Os valores são passados separadamente no array `values`!
  const sqlQuery = `
    SELECT id, codigo_rastreio, valor_total, status
    FROM pedidos
    WHERE usuario_id = $1
      AND codigo_rastreio = $2
  `;
  const values = [usuarioId, termo];

  try {
    const resultado = await pool.query(sqlQuery, values);
    return res.status(200).json({ pedidos: resultado.rows });
  } catch (erro) {
    // [REQUISITO] Registrar o erro detalhado APENAS no log interno do servidor
    console.error('[ERRO INTERNO SQL]', { timestamp: new Date().toISOString(), erro: erro.message });

    // Retornar mensagem genérica ao cliente, sem vazar estrutura de tabelas ou sintaxe SQL
    return res.status(500).json({
      erro: 'Ocorreu um erro interno ao processar sua solicitação.'
    });
  }
});
```

> [!TIP]
> **E quando precisamos parametrizar `ORDER BY` ou nomes de colunas?**
> Marcadores de *Prepared Statements* (`$1`, `?`) só aceitam **valores literais**, não nomes de tabelas, nomes de colunas ou palavras-chave (`ASC`/`DESC`). Nesses casos, **[REQUISITO]** nunca concatene a string do usuário: valide a entrada contra uma **lista de permissões estrita (*allowlist*)** mapeada no código:
> ```javascript
> const colunasPermitidas = { data: 'criado_em', valor: 'valor_total', status: 'status' };
> const colunaOrdenacao = colunasPermitidas[req.query.ordenarPor] || 'criado_em';
> const direcao = req.query.dir === 'asc' ? 'ASC' : 'DESC';
> // Agora `colunaOrdenacao` e `direcao` vêm exclusivamente de constantes fixas do próprio código!
> ```

##### 8. Como testar se a correção funcionou

Após aplicar a correção, execute uma bateria de verificação em três níveis:
1. **Teste com Metacaracteres SQL**: envie `?termo=BR123'` e `?termo=' OR '1'='1`.
   - **Resultado esperado**: Se a validação de formato (regex) estiver ativa, a API deve responder imediatamente com `400 Bad Request`. Se o campo aceitar texto livre (como um campo de comentário ou nome), a API deve responder com `200 OK` e uma lista vazia `{"pedidos": []}`, **nunca** retornando registros de outros usuários e **nunca** gerando status `500` por erro de sintaxe SQL.
2. **Teste de Regressão Funcional**: envie um código de rastreio válido pertencente ao usuário logado (`?termo=BR99887766`) e confirme que o pedido legítimo continua sendo retornado com `200 OK`.
3. **Teste de Tempo (Blind Time-Based)**: envie um payload com função de atraso (`?termo=' OR pg_sleep(5)--`) e confirme na aba **Network** do DevTools que o tempo de resposta (**TTFB**) permanece na casa de poucos milissegundos (ex.: `15 ms`), provando que a função `pg_sleep` não foi executada pelo banco.

##### 9. Como ela pode aparecer durante uma análise no DevTools

Durante uma auditoria técnica utilizando o **Chrome DevTools** (veja a [Seção 2](./02a-devtools-network.md#s2)), indícios e evidências de SQL Injection aparecem nas seguintes abas:

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Coluna Status e Aba Response / Preview**: Ao enviar uma entrada com aspa simples (`'`), observa-se uma resposta com status `500 Internal Server Error` (ou às vezes `200 OK` com mensagem de erro no HTML/JSON) contendo stack traces de drivers de banco de dados (`PDOException`, `PSQLException`, `mysql_fetch_array()`, `ORA-01756`, `SQLSTATE[42000]`).
  - **Aba Payload (Query String Parameters / Request Payload)**: Mostra os parâmetros exatos sendo enviados ao servidor. Se um parâmetro do front-end envia trechos de SQL brutos (por exemplo, `?filtro=preco > 100 AND categoria = 'livros'`), é um fortíssimo sinal de alerta de que o back-end concatena essa string diretamente em uma cláusula `WHERE`!
  - **Aba Timing (Waiting for server response — TTFB)**: Em testes autorizados de *Time-Based Blind SQLi*, a barra verde de **Waiting for server response** salta de valores normais (ex.: `40 ms`) para exatamente o tempo injetado na função de atraso (ex.: `5.04 s` para `SLEEP(5)`).
  - **Coluna Size (Transfer / Resource Size)**: Em falhas baseadas em tautologia (`OR '1'='1'`) ou `UNION`, o tamanho da resposta aumenta drasticamente (ex.: de `450 B` para `85 kB`), pois o servidor despeja centenas de linhas da tabela no JSON.
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Erros `SyntaxError: Unexpected token < in JSON at position 0` no JavaScript do cliente quando o front-end esperava um JSON da API, mas o servidor back-end quebrou com um erro fatal de SQL e devolveu uma página HTML de erro do PHP/Django/Express contendo a mensagem do banco de dados.
  - Objetos de erro de API impressos por blocos `catch (err) { console.error(err.response.data); }` revelando o campo `sqlMessage` ou a query completa no Console.
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Em aplicações legadas ou mal projetadas, cookies armazenados em **Storage > Cookies** que guardam filtros ou identificadores lidos diretamente em consultas SQL pelo servidor sem parametrização (*Cookie-based SQL Injection*).

---

<a id="s3-2"></a>
### 3.2 Cross-Site Scripting (XSS — Refletido, Armazenado e DOM-Based)

##### 1. Definição

**Cross-Site Scripting (XSS)**, catalogado sob a **CWE-79** e classificado na categoria **Injection** (**A03:2021** / **A05:2025** no OWASP Top 10), é uma vulnerabilidade em que um atacante consegue injetar scripts maliciosos (geralmente **JavaScript**, mas também marcação HTML ativa) em páginas visualizadas por outros usuários de uma aplicação confiável.

O motivo pelo qual o XSS é tão perigoso reside na **Same-Origin Policy (SOP — Política de Mesma Origem, detalhada na [Seção 1.11](./01a-fundamentos-web-parte1.md#s1))**: como o script injetado é entregue dentro da própria página da aplicação vulnerável (por exemplo, `https://loja-ficticia.test`), o navegador da vítima não tem como distinguir o JavaScript legítimo do desenvolvedor daquele injetado pelo atacante. Consequentemente, o script malicioso é executado com **acesso total ao contexto daquela origem**: ele pode ler tokens guardados em `localStorage` ou `sessionStorage`, ler cookies que não possuam a flag `HttpOnly` (`document.cookie`), capturar teclas digitadas (*keylogging*), modificar o DOM para exibir formulários falsos de login (*phishing* interno) e disparar requisições autenticadas (`fetch()` / `XMLHttpRequest`) em nome da vítima.

> [!NOTE]
> **Por que a sigla é "XSS" e não "CSS"?**
> Historicamente, o termo *Cross-Site Scripting* abreviava-se como CSS, mas a comunidade de segurança adotou a sigla **XSS** para evitar ambiguidade com **CSS (Cascading Style Sheets — Folhas de Estilo em Cascata)**.

##### 2. Como funciona

O XSS ocorre sempre que um fluxo de dados percorre o caminho entre uma **fonte não confiável (*Source*)** e um **sumidouro de renderização ou execução (*Sink*)** no navegador sem passar por codificação de saída (*Output Encoding*) ou sanitização contextual adequada. Existem **três modalidades clássicas de XSS**, cujas diferenças arquiteturais são presença obrigatória em provas acadêmicas:

| Tipo de XSS | Onde o Payload é Processado | Persistência | Fluxo de Execução Passo a Passo |
| :--- | :--- | :--- | :--- |
| **1. XSS Refletido (*Reflected / Non-Persistent XSS*)** | **Servidor (Back-End)** | **Não persistente** (dura apenas uma requisição). | 1. O atacante cria um link contendo o script em um parâmetro (ex.: `?busca=<script>...</script>`) e o envia à vítima.<br>2. A vítima clica no link.<br>3. O servidor recebe o parâmetro e o "reflete" imediatamente no HTML da resposta (ex.: *"Resultados para: `<script>...</script>`"*) sem escapar os caracteres `<` e `>`.<br>4. O navegador da vítima interpreta a resposta e executa o script. |
| **2. XSS Armazenado (*Stored / Persistent XSS*)** | **Servidor + Banco de Dados** | **Persistente** (afeta todos que abrirem a página). | 1. O atacante envia o payload em um campo salvo no servidor (ex.: comentário em produto, nome do perfil, mensagem de fórum).<br>2. O servidor grava a string no banco de dados.<br>3. Sempre que qualquer usuário (ou administrador) acessa a página onde aquele comentário é listado, o servidor entrega o HTML contendo o script armazenado.<br>4. O navegador de cada visitante executa o script automaticamente, sem precisar clicar em links suspeitos. |
| **3. XSS Baseado em DOM (*DOM-Based XSS*)** | **Cliente (Front-End / JavaScript)** | Geralmente não persistente (ou persistido no `localStorage`). | 1. O servidor entrega um arquivo HTML/JS estático e totalmente limpo (o payload pode nem ir ao servidor se estiver no fragmento `#` da URL).<br>2. O próprio código JavaScript legítimo da página lê um dado de uma **Source** controlável pelo usuário (ex.: `location.search`, `location.hash`, `window.name`, `postMessage`) e o escreve diretamente em um **Sink** perigoso do DOM (ex.: `element.innerHTML`, `document.write()`, `eval()`).<br>3. O navegador interpreta a alteração do DOM e executa o script. |

Para diagnosticar o **DOM-Based XSS**, é essencial conhecer os pares de **Sources (Fontes)** e **Sinks (Sumidouros)**:

- **Principais *Sources* (Fontes de entrada no JavaScript)**:
  - `location.search` (query string da URL: `?msg=...`)
  - `location.hash` (fragmento da URL: `#aba=...` — note que o fragmento `#` **nunca é enviado ao servidor HTTP**, tornando o DOM XSS invisível aos logs tradicionais de servidor e WAFs!)
  - `location.href` / `document.URL` / `document.documentURI`
  - `document.referrer`
  - `window.name`
  - `event.data` em ouvintes `window.addEventListener('message', ...)`
  - `localStorage` e `sessionStorage` (quando alimentados por parâmetros externos)
- **Principais *Sinks* Perigosos (Sumidouros de execução no DOM)**:
  - **Sinks de HTML**: `element.innerHTML`, `element.outerHTML`, `element.insertAdjacentHTML()`, `document.write()`, `document.writeln()`
  - **Sinks de Execução de Código JS**: `eval()`, `Function()`, `setTimeout(string)`, `setInterval(string)`
  - **Sinks de Navegação / URL (aceitam pseudo-protocolo `javascript:`)**: `location.href = ...`, `location.assign(...)`, `location.replace(...)`, `anchorElement.href = ...`, `iframeElement.src = ...`

> [!WARNING]
> **Por que `<script>alert(1)</script>` nem sempre funciona com `innerHTML`?**
> Conforme a especificação oficial do HTML5 (WHATWG), tags `<script>` inseridas dinamicamente via `element.innerHTML = "<script>alert(1)</script>"` **não são executadas** pelo navegador por motivos de segurança e ciclo de parsing. Porém, isso **NÃO significa que `innerHTML` seja seguro**: um atacante usa manipuladores de eventos em elementos como imagens ou SVGs — por exemplo, `<img src="x" onerror="alert(1)">` ou `<svg onload="alert(1)">` — que **são executados imediatamente** quando inseridos via `innerHTML`!

##### 3. Por que acontece

O XSS ocorre porque a aplicação insere dados controláveis pelo usuário em um contexto do documento sem aplicar a **codificação de saída específica para aquele contexto** ou utiliza APIs inseguras do DOM:
- **No Back-End (SSR — Server-Side Rendering)**: concatenar variáveis em templates HTML ou usar diretivas que desativam o escape automático do motor de template (como `<%- variavel %>` no EJS, `{{ variavel | safe }}` no Django/Jinja2, `{!! $variavel !!}` no Laravel Blade).
- **No Front-End (SPA — Single-Page Application)**: usar `element.innerHTML` em vez de `element.textContent`, ou usar propriedades de escape explícito em frameworks modernos sem sanitizar antes — como `dangerouslySetInnerHTML` no React, `v-html` no Vue.js ou `[innerHTML]` no Angular.
- **Em atributos `href` ou `src`**: validar se uma URL é segura apenas verificando se não tem `<` ou `>`, esquecendo que um usuário pode informar `javascript:alert(document.domain)` como link de "Website do perfil".

##### 4. Exemplo didático de código vulnerável

Abaixo apresentamos um cenário duplo em uma aplicação fictícia (`https://loja-ficticia.test/avaliacoes`): um **XSS Refletido** no servidor (Node.js/Express) e um **DOM-Based XSS** no script de front-end que renderiza os comentários:

```javascript
// [CÓDIGO VULNERÁVEL 1 — BACK-END (XSS Refletido)]
app.get('/busca', (req, res) => {
  const termo = req.query.q || '';
  // ERRO: O valor de `termo` é interpolado diretamente na resposta HTML sem codificação de entidades!
  res.setHeader('Content-Type', 'text/html; charset=utf-8');
  res.send(`
    <!DOCTYPE html>
    <html lang="pt-BR">
    <body>
      <h2>Resultados da busca para: ${termo}</h2>
      <div id="mensagem-status"></div>
      <div id="lista-avaliacoes"></div>
      <script src="/static/avaliacoes.js"></script>
    </body>
    </html>
  `);
});
```

```javascript
// [CÓDIGO VULNERÁVEL 2 — FRONT-END /static/avaliacoes.js (DOM-Based XSS e Stored XSS)]
// 1. Lê parâmetro da URL (Source: location.hash) e injeta com innerHTML (Sink)
const hashMsg = decodeURIComponent(window.location.hash.slice(1));
if (hashMsg) {
  document.getElementById('mensagem-status').innerHTML = `Aviso: ${hashMsg}`;
}

// 2. Busca avaliações salvas no banco e monta cartões HTML com innerHTML (Stored XSS)
fetch('/api/v1/avaliacoes')
  .then(res => res.json())
  .then(avaliacoes => {
    const container = document.getElementById('lista-avaliacoes');
    avaliacoes.forEach(item => {
      // ERRO CRÍTICO: `item.autor`, `item.comentario` e `item.siteAutor` vêm do banco sem sanitização!
      container.innerHTML += `
        <article class="card">
          <h4><a href="${item.siteAutor}">${item.autor}</a></h4>
          <p>${item.comentario}</p>
        </article>
      `;
    });
  });
```

Veja como este pequeno trecho contém **três pontos de injeção**:
1. Em `/busca?q=<script>alert(1)</script>`, o servidor reflete a tag `<script>` diretamente no corpo do HTML (**XSS Refletido**).
2. Em `/busca#<img src=x onerror=alert(1)>`, o JavaScript lê `location.hash` e grava em `.innerHTML` (**DOM-Based XSS**).
3. Se um usuário salvar uma avaliação com `comentario: '<img src=x onerror=alert(1)>'` ou `siteAutor: 'javascript:alert(1)'`, todo visitante terá o script executado (**XSS Armazenado**).

##### 5. Como identificar

1. **Mapeamento de Entrada e Reflexão**: identifique todos os pontos onde dados fornecidos pelo usuário (query strings, campos de cadastro, cabeçalhos, fragmentos de URL) reaparecem na resposta HTTP ou são manipulados pelo DOM.
2. **Inspeção do Contexto de Saída**: verifique **onde** o dado aparece no HTML:
   - Dentro do corpo de uma tag (`<div>DADO</div>`)?
   - Dentro de um atributo entre aspas (`<input value="DADO">`)?
   - Dentro de um atributo de URL (`<a href="DADO">`)?
   - Dentro de um bloco `<script>const usuario = "DADO";</script>`?
3. **Varredura por Sinks no Código JS**: busque no código-fonte por `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `eval`, `dangerouslySetInnerHTML` e `v-html`.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um laboratório autorizado (como o **PortSwigger Web Security Academy** ou **DVWA**):

1. **Passo 1 — Sondagem com String Canário Inofensiva**:
   - Antes de usar qualquer tag de script, o analista envia uma string alfanumérica única seguida de caracteres especiais de marcação, por exemplo: `testeXSS123<>"'`.
   - Em seguida, abre o código-fonte da resposta (ou a aba **Network > Response** do DevTools) e procura por `testeXSS123`.
   - Se a resposta contiver `testeXSS123&lt;&gt;&quot;&#39;`, o servidor aplicou codificação HTML corretamente. Se contiver `testeXSS123<>"'` literalmente sem codificação, a aplicação é vulnerável a injeção de marcação!
2. **Passo 2 — Prova de Conceito Didática (*Proof of Concept — PoC*)**:
   - No parâmetro refletido em contexto HTML, envia-se o payload didático clássico:
     `<script>alert(document.domain)</script>` ou `<img src=x onerror=alert(document.domain)>`
   - Nota acadêmica: usa-se `alert(document.domain)` (ou `console.log(window.origin)`) em vez de apenas `alert(1)` para comprovar em qual origem (SOP) o script está sendo executado (garantindo que não está isolado em um `iframe` com `sandbox` vazio).

##### 7. Como corrigir

A correção definitiva contra XSS exige uma estratégia em camadas (defesa em profundidade):

1. **[REQUISITO] Codificação de Saída Contextual (*Contextual Output Encoding*) no Back-End**:
   Todo dado dinâmico inserido em um documento HTML deve ter seus caracteres especiais convertidos nas respectivas **Entidades HTML** antes de ser enviado ao navegador:
   - `&` $\rightarrow$ `&amp;`
   - `<` $\rightarrow$ `&lt;`
   - `>` $\rightarrow$ `&gt;`
   - `"` $\rightarrow$ `&quot;`
   - `'` $\rightarrow$ `&#x27;` (ou `&#39;`)
2. **[REQUISITO] Uso de APIs Seguras do DOM (`textContent` / `createElement`) no Front-End**:
   Ao inserir texto puro no DOM via JavaScript, **nunca** use `innerHTML`; utilize sempre `element.textContent` (ou `innerText`), que trata qualquer string estritamente como nó de texto e jamais interpreta tags HTML.
3. **[REQUISITO] Sanitização rigorosa quando HTML rico for indispensável**:
   Se a aplicação realmente precisar permitir formatação HTML (como um editor de texto com negrito e listas), sanitize a string usando uma biblioteca dedicada e mantida ativamente, como o **DOMPurify** (`DOMPurify.sanitize(htmlSuja)`), antes de atribuí-la ao DOM.
4. **[REQUISITO] Validação de protocolo em URLs dinâmicas**:
   Ao preencher atributos `href` ou `src` com links enviados por usuários, valide com o construtor `new URL()` se o protocolo (`url.protocol`) é estritamente `'http:'` ou `'https:'`, bloqueando `'javascript:'` e `'data:'`.
5. **[RECOMENDAÇÃO] Defesa em profundidade com Headers e Cookies ([Seção 5](./05-headers-de-seguranca.md#s5))**:
   - Configure a flag **`HttpOnly`** nos cookies de sessão para que, mesmo que ocorra um XSS, o atacante não consiga ler o cookie via `document.cookie`.
   - Implemente uma política **CSP (Content-Security-Policy)** forte (sem `'unsafe-inline'` e sem `'unsafe-eval'`, usando `nonce` ou hashes e, quando possível, **Trusted Types** com `require-trusted-types-for 'script'`).

Veja o código anterior **completamente corrigido**:

```javascript
// [CÓDIGO CORRIGIDO 1 — BACK-END]
function escapeHtml(str) {
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}

app.get('/busca', (req, res) => {
  const termoSeguro = escapeHtml(req.query.q || '');

  // [REQUISITO] Definir Content-Type explícito e headers de defesa em profundidade
  res.setHeader('Content-Type', 'text/html; charset=utf-8');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';"
  );

  res.send(`
    <!DOCTYPE html>
    <html lang="pt-BR">
    <body>
      <h2>Resultados da busca para: ${termoSeguro}</h2>
      <div id="mensagem-status"></div>
      <div id="lista-avaliacoes"></div>
      <script src="/static/avaliacoes.js"></script>
    </body>
    </html>
  `);
});
```

```javascript
// [CÓDIGO CORRIGIDO 2 — FRONT-END /static/avaliacoes.js]
function urlSegura(urlInformada) {
  try {
    const parsed = new URL(urlInformada);
    return (parsed.protocol === 'http:' || parsed.protocol === 'https:') ? parsed.href : '#';
  } catch {
    return '#';
  }
}

// 1. Uso de `textContent` em vez de `innerHTML` (imune a DOM XSS)
const hashMsg = decodeURIComponent(window.location.hash.slice(1));
if (hashMsg) {
  document.getElementById('mensagem-status').textContent = `Aviso: ${hashMsg}`;
}

// 2. Construção segura de nós do DOM com createElement + textContent + validação de URL
fetch('/api/v1/avaliacoes')
  .then(res => res.json())
  .then(avaliacoes => {
    const container = document.getElementById('lista-avaliacoes');
    avaliacoes.forEach(item => {
      const article = document.createElement('article');
      article.className = 'card';

      const h4 = document.createElement('h4');
      const link = document.createElement('a');
      link.href = urlSegura(item.siteAutor);
      link.rel = 'noopener noreferrer';
      link.textContent = item.autor; // Seguro: nunca interpreta tags HTML

      const p = document.createElement('p');
      p.textContent = item.comentario; // Seguro: nunca interpreta tags HTML

      h4.appendChild(link);
      article.appendChild(h4);
      article.appendChild(p);
      container.appendChild(article);
    });
  });
```

##### 8. Como testar se a correção funcionou

1. Envie novamente os payloads de teste (`<script>alert(1)</script>`, `<img src=x onerror=alert(1)>` e `javascript:alert(1)` no campo de site).
2. Verifique na tela do navegador que o texto `<script>alert(1)</script>` aparece **impresso literalmente na página como texto comum**, sem que nenhuma janela de alerta seja disparada e sem que elementos HTML extras sejam criados na árvore do DOM.
3. Inspecione o link criado para `javascript:alert(1)` e confirme que o atributo `href` foi convertido com segurança para `#`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Aba Response (código-fonte bruto vindo do servidor)**: Permite diferenciar imediatamente **XSS Refletido/Armazenado** de **DOM-Based XSS**. Se a tag `<img src=x onerror=...>` já vier escrita não escapada no HTML bruto da aba **Response**, a falha é no servidor (Refletido ou Armazenado). Se o HTML bruto da aba **Response** estiver limpo, mas a tag aparecer no painel **Elements** (DOM vivo), a falha é um **DOM-Based XSS** causado pelo JavaScript do cliente!
  - **Requisições de exfiltração suspeitas**: Se um script XSS já estiver ativo na página, a aba **Network** mostrará requisições inesperadas (do tipo `Fetch/XHR`, `Img` ou `Ping`) enviadas para domínios externos desconhecidos carregando tokens ou cookies na query string (ex.: `https://coletor-externo.test/log?c=eyJhbGci...`).
  - **Ausência do header `Content-Security-Policy`** nos **Response Headers** do documento principal (`Doc`).
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Se o site possuir uma **Content Security Policy (CSP)** ativa e houver uma tentativa de XSS, o Console exibirá um erro vermelho detalhado de violação de CSP:
    `Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'self'"`.
  - Se o analista digitar `document.cookie` no Console e visualizar o cookie de sessão (`SID=...`), fica comprovado que o cookie **não possui a flag `HttpOnly`** e poderá ser roubado caso ocorra um XSS.
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Em **Storage > Local Storage / Session Storage**, a presença de tokens JWT (`access_token`, `id_token`) ou dados pessoais sensíveis evidencia que qualquer XSS na origem conseguirá exfiltrar a sessão inteira com uma única linha (`localStorage.getItem('access_token')`).
  - Em **Storage > Cookies**, a coluna **HttpOnly** desmarcada (vazia) para cookies de autenticação mostra vulnerabilidade direta à leitura via XSS.

---

<a id="s3-3"></a>
### 3.3 Cross-Site Request Forgery (CSRF)

##### 1. Definição

**Cross-Site Request Forgery (CSRF ou XSRF — Falsificação de Requisição Entre Sites)**, catalogada sob a **CWE-352** e classificada dentro de **Broken Access Control** (**A01:2021** e **A01:2025** no OWASP Top 10), é uma vulnerabilidade em que um site malicioso (ou e-mail/fórum externo) induz o navegador de um usuário já autenticado em uma aplicação confiável a enviar, **sem o conhecimento ou intenção do usuário**, uma requisição HTTP que executa uma ação de mudança de estado na aplicação confiável (como transferir valores, alterar o e-mail ou senha da conta, excluir um registro ou promover permissões).

Diferentemente do XSS — em que o atacante tenta **executar código dentro da origem confiável para ler dados** —, o CSRF explora a **confiança que o servidor deposita nos cookies enviados automaticamente pelo navegador**: no CSRF, o atacante não precisa ler a resposta da requisição; basta que o comando chegue ao servidor acompanhado do cookie de sessão válido da vítima.

##### 2. Como funciona

Por padrão histórico da arquitetura Web, quando um servidor define um cookie de sessão (`Set-Cookie: session_id=xyz123; Path=/`), o navegador passa a anexar esse cookie (`Cookie: session_id=xyz123`) **automaticamente em toda requisição HTTP destinada àquele domínio**, independentemente de a requisição ter sido iniciada pelo usuário clicando em um botão dentro do próprio site (`https://banco.exemplo.test`) ou disparada por um formulário oculto hospedado em outro site (`https://sorteio-malicioso.test`) — a menos que políticas restritivas de `SameSite` estejam ativas.

Para que um ataque de CSRF tenha sucesso, **três condições** precisam ocorrer simultaneamente:
1. **Uma ação relevante que altera estado**: existe um endpoint no servidor que executa uma operação sensível (ex.: `POST /conta/alterar-email`).
2. **Autenticação baseada exclusivamente em credenciais enviadas automaticamente pelo navegador**: o servidor identifica o usuário apenas pelo cookie de sessão (ou autenticação HTTP Basic/Digest/Certificado Cliente) e não exige nenhum segredo adicional na requisição.
3. **Ausência de parâmetros imprevisíveis**: o atacante conhece antecipadamente todos os nomes de campos que o endpoint espera receber (ex.: `novo_email=atacante@exemplo.test`) e não há nenhum token criptográfico imprevisível exigido pelo servidor.

> [!IMPORTANT]
> **Por que a Same-Origin Policy (SOP) não bloqueia o CSRF por padrão?**
> Conforme visto na [Seção 1.11](./01a-fundamentos-web-parte1.md#s1), a **Same-Origin Policy** impede que a origem `A` **leia** a resposta de uma requisição feita para a origem `B` (*cross-origin read*), mas ela **permite** o envio de chamadas simples de escrita (*cross-origin writes*), como o envio de formulários HTML `<form method="POST">` com `Content-Type: application/x-www-form-urlencoded`, `multipart/form-data` ou `text/plain`, além do carregamento de links e imagens (`GET`). Quando o navegador bloqueia a leitura da resposta no CSRF, o estrago no servidor **já aconteceu**!

##### 3. Por que acontece

O CSRF acontece porque a aplicação não diferencia uma **requisição intencional originada na própria interface do sistema** (*same-origin / same-site*) de uma **requisição forjada disparada por uma página de terceiros** (*cross-site*):
- Endpoints que modificam dados (`POST`, `PUT`, `DELETE`) confiam cegamente apenas na presença do cookie de sessão.
- Pior ainda: aplicações que violam a semântica HTTP (RFC 9110) e usam o método **`GET` para realizar alterações de estado** (ex.: `GET /usuarios/excluir?id=42` ou `GET /perfil/mudar-email?email=x`). Nesse caso, até mesmo uma tag de imagem `<img src="https://loja-ficticia.test/perfil/mudar-email?email=x">` colocada em um site externo ou mesmo que o cookie tenha `SameSite=Lax` (no caso de navegação top-level via link) conseguirá disparar a ação!
- Cookies de sessão configurados com `SameSite=None; Secure` sem necessidade ou sem a devida validação de **Anti-CSRF Token** no servidor.

##### 4. Exemplo didático de código vulnerável

Considere o endpoint de alteração de e-mail de cadastro em uma aplicação fictícia de laboratório (`https://loja-ficticia.test/conta/email`):

```javascript
// [CÓDIGO VULNERÁVEL NO BACK-END — Node.js / Express]
app.use(express.urlencoded({ extended: false }));

app.post('/conta/email', async (req, res) => {
  // Verifica apenas se existe sessão ativa via Cookie (ex.: express-session)
  if (!req.session || !req.session.usuarioId) {
    return res.status(401).send('Não autenticado');
  }

  const novoEmail = req.body.novo_email;
  // ERRO CRÍTICO: Nenhuma verificação de CSRF Token, de header Origin ou de senha atual!
  await db.query('UPDATE usuarios SET email = $1 WHERE id = $2', [
    novoEmail,
    req.session.usuarioId
  ]);

  return res.status(200).send('E-mail atualizado com sucesso!');
});
```

Em um ambiente de laboratório, a página externa simulada (`http://laboratorio-externo.test/promo.html`) conteria apenas o seguinte HTML auto-submetido:

```html
<!-- [EXEMPLO DIDÁTICO DE PÁGINA EXTERNA EM LABORATÓRIO] -->
<!DOCTYPE html>
<html>
<body>
  <form id="csrf-form" action="https://loja-ficticia.test/conta/email" method="POST">
    <input type="hidden" name="novo_email" value="invasor@laboratorio.test" />
  </form>
  <script>
    // Submete o formulário automaticamente assim que a página é aberta
    document.getElementById('csrf-form').submit();
  </script>
</body>
</html>
```

##### 5. Como identificar

1. **Verificação de Métodos de Escrita**: localize todas as requisições que criam, alteram ou apagam dados no servidor (`POST`, `PUT`, `PATCH`, `DELETE`, ou `GET` mal utilizado).
2. **Inspeção de Credenciais e Tokens**: verifique como a autenticação é transmitida:
   - Se a requisição usa **exclusivamente** o header `Authorization: Bearer <token>` (injetado via JavaScript) e **não aceita cookies**, ela é naturalmente imune a CSRF clássico, pois um site externo não consegue adicionar o header `Authorization` da vítima.
   - Se a requisição depende de **Cookies**, verifique se existe um token anti-CSRF imprevisível no corpo ou em um header customizado (`X-CSRF-Token`), como estão configurados os atributos `SameSite` do cookie de sessão e se o servidor valida os headers `Origin` e `Sec-Fetch-Site`.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

1. **Passo 1 — Análise da Requisição Legítima**:
   - No laboratório, o estudante faz login na aplicação (`https://loja-ficticia.test`), abre a aba **Network** do DevTools e altera seu e-mail legitimamente pela interface.
   - Ele inspeciona a requisição `POST /conta/email` e nota que: (a) o `Content-Type` é `application/x-www-form-urlencoded`; (b) o corpo possui apenas `novo_email=aluno@exemplo.test` (sem campo `_csrf`); e (c) o cookie de sessão não possui `SameSite=Strict` ou `Lax` adequadamente aliado a tokens.
2. **Passo 2 — Simulação de Origem Cruzada**:
   - O estudante hospeda uma página HTML simples em outra porta/domínio local de teste (`http://outro-dominio.test:8080/teste-csrf.html`) contendo um `<form>` apontando para `https://loja-ficticia.test/conta/email` e o submete no mesmo navegador onde está logado.
   - Se o e-mail da conta na `loja-ficticia.test` for alterado pela submissão vinda de `outro-dominio.test`, a vulnerabilidade de CSRF está comprovada.

##### 7. Como corrigir

A proteção robusta contra CSRF baseia-se em **defesa em profundidade**, combinando tokens criptográficos com atributos modernos de cookies e validação de metadados da requisição:

1. **[REQUISITO] Anti-CSRF Tokens (*Synchronizer Token Pattern*)**:
   O servidor gera um token criptograficamente aleatório, único e vinculado à sessão do usuário autenticado. Esse token é inserido em um campo oculto dos formulários (`<input type="hidden" name="csrf_token" value="...">`) ou enviado em um cabeçalho HTTP customizado (`X-CSRF-Token`) nas chamadas `fetch()`. Como a **Same-Origin Policy** impede que um site externo leia a página da aplicação confiável, o atacante jamais consegue descobrir o valor do token da vítima.
2. **[REQUISITO] Atributo `SameSite` nos Cookies de Sessão (`Strict` ou `Lax`)**:
   - `SameSite=Strict`: o navegador **jamais** envia o cookie em requisições originadas fora do mesmo site (nem mesmo ao clicar em um link normal). Ideal para ações financeiras ou painéis administrativos.
   - `SameSite=Lax`: o navegador bloqueia o envio do cookie em sub-recursos cross-site e em formulários `POST` cross-site, enviando-o apenas em navegações de nível superior (*top-level navigation*) com métodos seguros (`GET`).
3. **[RECOMENDAÇÃO] Validação de `Origin`, `Referer` e `Sec-Fetch-Site` no Servidor**:
   Para qualquer requisição de mudança de estado (`POST`, `PUT`, `PATCH`, `DELETE`), o servidor deve conferir se o header `Origin` corresponde exatamente à origem esperada (`https://loja-ficticia.test`) e se o header de Fetch Metadata `Sec-Fetch-Site` é `same-origin`.
4. **[REQUISITO] Respeitar a Semântica HTTP (RFC 9110)**:
   **Nunca** utilize requisições `GET` ou `HEAD` para operações que modifiquem dados no servidor.

Veja o código anterior **completamente corrigido**:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO NO BACK-END — Node.js / Express]
const crypto = require('crypto');

// Configuração segura do Cookie de Sessão:
// Set-Cookie: __Host-sid=...; Path=/; Secure; HttpOnly; SameSite=Lax

// Middleware para gerar token CSRF vinculado à sessão
app.get('/conta/configuracoes', (req, res) => {
  if (!req.session.csrfToken) {
    req.session.csrfToken = crypto.randomBytes(32).toString('hex');
  }
  res.render('configuracoes', { csrfToken: req.session.csrfToken });
});

// Middleware de proteção contra CSRF para rotas de escrita
function validarCsrf(req, res, next) {
  // 1. Validar Origin e Sec-Fetch-Site (Defesa em Profundidade)
  const origin = req.headers.origin;
  const fetchSite = req.headers['sec-fetch-site'];
  if (origin && origin !== 'https://loja-ficticia.test') {
    return res.status(403).json({ erro: 'Origem não autorizada (CSRF bloqueado).' });
  }
  if (fetchSite && fetchSite !== 'same-origin') {
    return res.status(403).json({ erro: 'Requisição cross-site bloqueada.' });
  }

  // 2. Validar Anti-CSRF Token com comparação em tempo constante
  const tokenEnviado = req.body.csrf_token || req.headers['x-csrf-token'] || '';
  const tokenSessao = req.session?.csrfToken || '';

  const bufEnviado = Buffer.from(String(tokenEnviado));
  const bufSessao = Buffer.from(String(tokenSessao));

  if (
    !tokenSessao ||
    bufEnviado.length !== bufSessao.length ||
    !crypto.timingSafeEqual(bufEnviado, bufSessao)
  ) {
    return res.status(403).json({ erro: 'Token CSRF inválido ou ausente.' });
  }

  next();
}

app.post('/conta/email', validarCsrf, async (req, res) => {
  if (!req.session?.usuarioId) {
    return res.status(401).json({ erro: 'Não autenticado.' });
  }

  const novoEmail = String(req.body.novo_email || '').trim();
  await db.query('UPDATE usuarios SET email = $1 WHERE id = $2', [
    novoEmail,
    req.session.usuarioId
  ]);

  return res.status(200).json({ mensagem: 'E-mail atualizado com sucesso!' });
});
```

##### 8. Como testar se a correção funcionou

1. Submeta o formulário a partir da página externa de laboratório sem o campo `csrf_token` (ou com um token pertencente a outra sessão): a aplicação deve rejeitar a requisição imediatamente com status **`403 Forbidden`** e o dado no banco deve permanecer inalterado.
2. Na aba **Network** do DevTools, marque a opção **Show filtered out request cookies** na sub-aba **Cookies** da requisição cross-site e verifique que o navegador sequer enviou o cookie de sessão graças ao atributo `SameSite=Lax` ou `SameSite=Strict`.
3. Submeta o formulário legitimamente dentro da própria aplicação: a requisição deve retornar `200 OK`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Aba Payload (Form Data / Request Payload) e Request Headers**: Ao inspecionar requisições `POST`, `PUT`, `PATCH` ou `DELETE` autenticadas por cookies, a **ausência** de um campo como `csrf_token` / `_token` no Payload e a **ausência** de um header customizado como `X-CSRF-Token` ou `X-XSRF-TOKEN` nos Request Headers indicam forte suspeita de vulnerabilidade a CSRF.
  - **Coluna Method**: Ações de exclusão, aprovação ou alteração de dados sendo disparadas via método **`GET`** (visível na coluna **Method**) representam uma vulnerabilidade grave de CSRF.
  - **Request Headers (`Origin` e `Sec-Fetch-Site`)**: Mostram se a requisição partiu de `same-origin` ou `cross-site`.
  - **Sub-aba Cookies da requisição**: Permite ver se o cookie de sessão foi enviado ou se foi filtrado pelo navegador por causa da política `SameSite` (exibido em amarelo quando *Show filtered out request cookies* está ativo).
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Em **Storage > Cookies**, verifique a coluna **SameSite** do cookie de sessão. Se estiver configurada como **`None`** (ou vazia em navegadores antigos sem o padrão `Lax`), o cookie será enviado em todas as requisições cross-site de terceiros.
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Avisos ou erros relacionados a cookies `SameSite=None` sem o atributo `Secure`, ou logs de erro `403 Forbidden` quando um token CSRF expira durante o uso da aplicação.

---

<a id="s3-4"></a>
### 3.4 Server-Side Request Forgery (SSRF)

##### 1. Definição

**Server-Side Request Forgery (SSRF — Falsificação de Requisição do Lado do Servidor)**, catalogado sob a **CWE-918** (categoria dedicada **A10:2021** e consolidado dentro de **A01:2025 – Broken Access Control** no OWASP Top 10), é uma vulnerabilidade em que um atacante induz a aplicação servidora (back-end) a realizar requisições de rede (HTTP, HTTPS ou outros protocolos) para um destino arbitrário não previsto pelo desenvolvedor.

Enquanto no **CSRF** a vítima que realiza a requisição forjada é o **navegador do cliente**, no **SSRF** quem dispara a requisição forjada é o **próprio servidor da aplicação**. Isso permite que um atacante externo utilize o servidor web como um "proxy involuntário" (*pivô*) para atravessar firewalls de perímetro e acessar recursos que jamais estariam expostos diretamente na Internet — como serviços administrativos rodando em `localhost` (`127.0.0.1`), bancos de dados e microsserviços na rede privada interna (faixas **RFC 1918**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) ou o serviço de metadados da infraestrutura em nuvem (`169.254.169.254`).

##### 2. Como funciona

Muitas aplicações web modernas possuem funcionalidades legítimas que exigem que o servidor busque uma URL informada pelo usuário, tais como:
- Importar uma foto de perfil a partir de um link (`https://...`);
- Gerar uma pré-visualização (*link preview* / Open Graph) de uma página compartilhada no chat;
- Configurar **Webhooks** (onde o usuário informa uma URL para a qual o servidor enviará notificações de eventos);
- Geradores de PDF no servidor que convertem HTML em PDF buscando folhas de estilo e imagens externas;
- Importadores de arquivos XML/SVG ou leitores de feeds RSS.

Se o servidor simplesmente receber o parâmetro `url` enviado pelo cliente e passá-lo diretamente para uma biblioteca de cliente HTTP (`fetch`, `axios`, `requests.get()` em Python, `cURL` em PHP), um atacante pode substituir a URL externa por alvos sensíveis internos:

| Vetor de Alvo em SSRF | Exemplo de URL Forjada | Impacto Técnico |
| :--- | :--- | :--- |
| **Loopback do próprio servidor** | `http://127.0.0.1:8080/admin` ou `http://localhost:6379/` | Acessa painéis administrativos internos ou bancos em memória (como Redis/Memcached) que confiam em conexões vindas de `127.0.0.1` sem exigir senha. |
| **Rede Interna Privada (RFC 1918)** | `http://10.0.15.42:9200/_cat/indices` ou `http://192.168.1.1/` | Varre portas internas (*port scanning* via tempo de resposta/erros) e interage com microsserviços internos (ex.: Elasticsearch, Kubernetes API, Consul) protegidos apenas pelo firewall externo. |
| **Serviço de Metadados de Nuvem (IMDS)** | `http://169.254.169.254/latest/meta-data/iam/security-credentials/` | Endereço *link-local* (RFC 3927) usado por provedores de nuvem (AWS, GCP, Azure, DigitalOcean) para expor metadados e **credenciais temporárias de IAM** da máquina virtual. No IMDSv1, uma simples requisição `GET` retorna chaves de acesso da nuvem! |
| **Esquemas de URL Alternativos** | `file:///etc/passwd`, `dict://127.0.0.1:6379/INFO`, `gopher://...` | Quando a biblioteca HTTP suporta múltiplos protocolos (como `libcurl`), o atacante pode ler arquivos locais do disco (`file://`) ou enviar comandos TCP brutos para serviços internos (`gopher://`, `dict://`). |

Assim como no SQL Injection, o SSRF divide-se em dois tipos quanto ao retorno:
1. **SSRF Clássico (*In-Band / Full SSRF*)**: o conteúdo da resposta obtida pelo servidor ao consultar o alvo interno é devolvido diretamente ao navegador do atacante (ex.: o servidor baixa `http://127.0.0.1:8080/status` e entrega o resultado como "imagem" ou texto para o usuário).
2. **SSRF Cego (*Blind SSRF*)**: o servidor dispara a requisição para o alvo interno, mas **não devolve o corpo da resposta** ao usuário (por exemplo, retorna apenas `{"status": "Webhook testado"}`). Ainda assim, o atacante pode inferir a existência de serviços internos medindo o tempo de resposta (**TTFB**) ou os códigos de erro HTTP (`200` vs `502` vs `504`), ou disparar ações internas que não exigem leitura da resposta.

##### 3. Por que acontece

O SSRF ocorre porque a aplicação **confia em uma URL ou nome de host fornecido pelo cliente** sem validar:
1. **O esquema (protocolo)** da URL (`http:` / `https:` vs `file:`, `gopher:`, `ftp:`);
2. **O domínio de destino** contra uma lista estrita de domínios permitidos (*allowlist*);
3. **O endereço IP real resultante da resolução DNS**: muitos desenvolvedores tentam bloquear a string `"localhost"` ou `"127.0.0.1"` com expressões regulares (*blocklist*), mas esquecem que um atacante pode contornar filtros textuais facilmente usando:
   - Endereço IPv6 de loopback (`http://[::1]/`);
   - Representações alternativas de IP (decimal inteiro `http://2130706433/`, octal `http://0177.0.0.1/` ou hexadecimal `http://0x7f.0.0.1/`);
   - Um domínio público próprio que possui um registro DNS `A` apontando para `127.0.0.1` ou `169.254.169.254`, ou técnicas de **DNS Rebinding** (onde o DNS resolve para um IP público durante a checagem de segurança e muda para `127.0.0.1` com `TTL=0` no milissegundo seguinte, quando o cliente HTTP faz a requisição real);
   - **Redirecionamento HTTP (`301`/`302`/`307`)**: o servidor valida que `https://site-permitido.test/avatar` é externo, mas ao acessá-lo recebe um `302 Found` com `Location: http://169.254.169.254/...` e a biblioteca HTTP segue o redirecionamento automaticamente sem revalidar o novo destino!

##### 4. Exemplo didático de código vulnerável

Veja um endpoint em **Node.js (Express)** criado para buscar e exibir metadados ou avatares de uma URL externa informada pelo usuário (`POST /api/v1/importar-avatar`):

```javascript
// [CÓDIGO VULNERÁVEL A SSRF — APENAS PARA ESTUDO EM LABORATÓRIO]
const express = require('express');
const app = express();
app.use(express.json());

app.post('/api/v1/importar-avatar', async (req, res) => {
  const { urlImagem } = req.body;

  try {
    // ERRO CRÍTICO: O servidor faz `fetch()` diretamente para qualquer URL enviada pelo usuário,
    // seguindo redirecionamentos por padrão e retornando o conteúdo bruto na resposta!
    const resposta = await fetch(urlImagem);
    const conteudoTexto = await resposta.text();

    return res.status(200).json({
      statusRemoto: resposta.status,
      tipoConteudo: resposta.headers.get('content-type'),
      previewDados: conteudoTexto.slice(0, 1000)
    });
  } catch (erro) {
    return res.status(500).json({
      erro: 'Não foi possível conectar ao host informado',
      detalhe: erro.message
    });
  }
});
```

##### 5. Como identificar

1. **Inspeção de Parâmetros de Rede**: procure em todas as requisições da aplicação por parâmetros cujos valores sejam URLs completas (`?url=https://...`, `webhookUrl`, `callback`, `avatar_url`, `template_uri`), nomes de host ou caminhos de servidor.
2. **Revisão de Código Back-End**: localize chamadas de clientes HTTP (`fetch`, `axios`, `got`, `http.request`, `requests`, `urllib`, `curl_exec`, `HttpClient`) e verifique de onde vêm os argumentos de URL/host e se o seguimento automático de redirecionamentos (`redirect: 'follow'`) está habilitado.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um laboratório dedicado (como o **PortSwigger Web Security Academy** ou um container Docker local com um serviço interno simulado na porta `8080`):

1. **Passo 1 — Identificação do Comportamento Normal**:
   - O estudante envia `{"urlImagem": "https://exemplo.test/logo.png"}` e observa na resposta que o servidor busca o arquivo e retorna seus dados.
2. **Passo 2 — Teste de Acesso a Loopback no Laboratório**:
   - No ambiente de laboratório, existe uma interface administrativa interna rodando apenas em `http://127.0.0.1:8080/admin` (inacessível diretamente pelo navegador do aluno).
   - O aluno envia `{"urlImagem": "http://127.0.0.1:8080/admin"}` no corpo do `POST /api/v1/importar-avatar`.
   - O servidor web faz a requisição para sua própria interface de loopback (`127.0.0.1:8080`) e devolve o HTML do painel administrativo dentro do campo `previewDados` do JSON!

##### 7. Como corrigir

A mitigação correta de SSRF exige validação rigorosa na **camada de aplicação** combinada com isolamento na **camada de rede**:

1. **[REQUISITO] Lista de Permissões Estrita (*Allowlist*) de Domínios e Protocolos**:
   Se a aplicação só precisa buscar imagens de parceiros conhecidos (ex.: `cdn.loja-ficticia.test`), valide o protocolo (`https:`) e o `hostname` exato contra uma *allowlist* após fazer o *parsing* com `new URL()`.
2. **[REQUISITO] Resolução de DNS e Bloqueio de Faixas de IP Privadas/Reservadas (quando URLs arbitrárias forem exigidas pelo negócio)**:
   Se a regra de negócio exigir buscar URLs arbitrárias da Internet (ex.: gerador de *link preview* ou *webhook*), você deve:
   - Exigir exclusivamente `http:` ou `https:`;
   - Resolver o DNS do `hostname` e verificar se **todos** os IPs retornados são **IPs públicos globais**, bloqueando loopback (`127.0.0.0/8`, `::1`), link-local/metadados (`169.254.0.0/16`, `fe80::/10`), redes privadas RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), CGNAT (`100.64.0.0/10`) e endereços únicos locais IPv6 (`fc00::/7`);
   - **Desativar o seguimento automático de redirecionamentos HTTP** (`redirect: 'error'` ou `'manual'`);
   - Validar o `Content-Type` da resposta e **nunca** devolver a resposta bruta inesperada ao cliente.
3. **[RECOMENDAÇÃO] Proteção de Infraestrutura e Nuvem (Defesa em Profundidade)**:
   - Exija o **IMDSv2** em instâncias de nuvem (que requer uma requisição `PUT` prévia com cabeçalho customizado `X-aws-ec2-metadata-token-ttl-seconds` para obter um token de sessão e limita o número de saltos de rede a `1`).
   - Isole o serviço que faz chamadas externas em uma sub-rede separada (DMZ de saída) ou use um *egress proxy* com regras de firewall que proíbam conexões para a rede interna.

Veja o código anterior **corrigido com validação de URL, resolução DNS, bloqueio de IPs privados e bloqueio de redirecionamentos**:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO CONTRA SSRF]
const dns = require('dns').promises;
const net = require('net');

// Função auxiliar que verifica se um endereço IPv4 ou IPv6 é privado, loopback ou reservado
function isIpPrivadoOuReservado(ip) {
  if (net.isIPv4(ip)) {
    const partes = ip.split('.').map(Number);
    const [a, b] = partes;
    if (a === 0 || a === 10 || a === 127) return true; // 0.0.0.0/8, 10.0.0.0/8, 127.0.0.0/8
    if (a === 169 && b === 254) return true;           // 169.254.0.0/16 (Link-local / Cloud Metadata!)
    if (a === 172 && b >= 16 && b <= 31) return true;  // 172.16.0.0/12 (RFC 1918)
    if (a === 192 && b === 168) return true;           // 192.168.0.0/16 (RFC 1918)
    if (a === 100 && b >= 64 && b <= 127) return true; // 100.64.0.0/10 (CGNAT)
    if (a >= 224) return true;                         // Multicast e Reservados
    return false;
  }
  if (net.isIPv6(ip)) {
    const norm = ip.toLowerCase();
    return (
      norm === '::1' ||
      norm === '::' ||
      norm.startsWith('fe80:') || // Link-local
      norm.startsWith('fc') ||    // Unique Local (fc00::/7)
      norm.startsWith('fd') ||
      norm.startsWith('::ffff:')  // IPv4-mapped IPv6
    );
  }
  return true;
}

app.post('/api/v1/importar-avatar', async (req, res) => {
  try {
    const parsedUrl = new URL(String(req.body.urlImagem || ''));

    // 1. [REQUISITO] Permitir exclusivamente protocolo HTTPS (e porta padrão 443)
    if (parsedUrl.protocol !== 'https:' || (parsedUrl.port && parsedUrl.port !== '443')) {
      return res.status(400).json({ erro: 'Apenas URLs HTTPS na porta padrão são permitidas.' });
    }

    // 2. [REQUISITO] Resolver o DNS e garantir que NENHUM IP aponte para rede interna/metadados
    const enderecos = await dns.lookup(parsedUrl.hostname, { all: true });
    if (enderecos.length === 0 || enderecos.some(reg => isIpPrivadoOuReservado(reg.address))) {
      return res.status(403).json({ erro: 'Host de destino não permitido pelas políticas de segurança.' });
    }

    // 3. [REQUISITO] Desabilitar seguimento de redirecionamentos (`redirect: 'error'`) e definir timeout
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 3000);

    const resposta = await fetch(parsedUrl.href, {
      method: 'GET',
      redirect: 'error',
      signal: controller.signal
    });
    clearTimeout(timeout);

    // 4. [REQUISITO] Validar Content-Type estritamente antes de processar e nunca ecoar resposta bruta
    const contentType = resposta.headers.get('content-type') || '';
    if (!['image/png', 'image/jpeg', 'image/webp'].includes(contentType.split(';')[0].trim())) {
      return res.status(415).json({ erro: 'O recurso remoto não é uma imagem válida.' });
    }

    return res.status(200).json({ mensagem: 'Avatar validado e importado com sucesso.' });
  } catch (erro) {
    return res.status(400).json({ erro: 'Não foi possível validar a imagem fornecida.' });
  }
});
```

##### 8. Como testar se a correção funcionou

1. Envie `http://127.0.0.1:8080/admin`, `https://169.254.169.254/latest/meta-data/` e `file:///etc/passwd`: a API deve rejeitar imediatamente com status `400 Bad Request` ou `403 Forbidden`.
2. Teste um domínio cujo registro DNS aponte para `127.0.0.1` (ex.: `localtest.me` em laboratório): a validação pós-resolução (`dns.lookup`) deve detectar o IP `127.0.0.1` e retornar `403 Forbidden`.
3. Teste uma URL externa que retorne `302 Found` redirecionando para `http://127.0.0.1`: a opção `redirect: 'error'` deve abortar a requisição com segurança.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Aba Payload (Query String Parameters / Request Payload)**: Presença de parâmetros que transportam URLs completas ou caminhos de servidores remotos (ex.: `GET /api/proxy?target=https://api-parceira.test/dados.json` ou `POST /gerar-pdf` com `{"urlCabecalho": "http://..."}`). Sempre que o navegador envia uma URL como parâmetro para o servidor buscar, aquele endpoint é um candidato primário à auditoria de SSRF!
  - **Aba Response / Preview**: Respostas da API que devolvem o corpo bruto, os cabeçalhos HTTP ou mensagens de erro de conexão (`ECONNREFUSED 10.0.0.12:6379`, `getaddrinfo ENOTFOUND`) geradas pelo cliente HTTP do servidor back-end.
  - **Aba Timing (Waiting for server response — TTFB)**: Variações acentuadas no TTFB ao testar diferentes portas ou hosts em laboratório (ex.: conexão recusada rapidamente em `10 ms` para porta fechada versus *timeout* de `3000 ms` para porta filtrada por firewall interno).
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Erros de JavaScript revelando endpoints internos de proxy usados pelo front-end para contornar restrições de CORS (por exemplo, um desenvolvedor que criou `/api/cors-proxy?url=...` para evitar configurar CORS corretamente — prática que quase sempre introduz uma falha grave de SSRF!).
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Configurações salvas no `localStorage` ou em cookies contendo endpoints de API ou URLs de webhooks que o cliente envia ao servidor nas requisições subsequentes.

---

<a id="s3-5"></a>
### 3.5 Broken Access Control (Quebra de Controle de Acesso)

##### 1. Definição

**Broken Access Control (Quebra de Controle de Acesso)**, mapeada em múltiplas CWEs (como **CWE-284**, **CWE-285** e **CWE-862**) e classificada em **1º lugar absoluto no OWASP Top 10 tanto em 2021 (A01:2021) quanto em 2025 (A01:2025)**, ocorre quando uma aplicação falha em impor restrições adequadas sobre **o que usuários autenticados (ou anônimos) têm permissão para fazer, acessar ou modificar**.

Para não confundir na prova, lembre-se da distinção fundamental vista na [Seção 1.7.4](./01a-fundamentos-web-parte1.md#s1):
- **Autenticação (*Authentication — AuthN*)**: responde à pergunta *"Quem é você?"* (verificar identidade via login/senha/MFA).
- **Autorização ou Controle de Acesso (*Authorization / Access Control — AuthZ*)**: responde à pergunta *"O que você tem permissão para fazer?"* (verificar se o usuário autenticado `João` pode acessar o relatório financeiro ou excluir um produto).

Quando o controle de acesso falha, um usuário comum consegue visualizar dados confidenciais de terceiros, executar funções exclusivas de administradores ou alterar recursos fora do seu escopo de permissão.

##### 2. Como funciona

O controle de acesso em aplicações web é dividido em **três dimensões clássicas**:

1. **Controle de Acesso Vertical (*Privilege Escalation / Escalação Vertical de Privilégios*)**:
   Separa usuários com **níveis hierárquicos ou papéis diferentes** (por exemplo: Visitante Anônimo $\rightarrow$ Cliente Comum $\rightarrow$ Gerente $\rightarrow$ Administrador do Sistema). Ocorre quebra vertical quando um Cliente Comum consegue acessar um endpoint administrativo como `POST /api/v1/admin/usuarios/promover` ou abrir o painel `/admin/relatorios`.
2. **Controle de Acesso Horizontal (*Escalação Horizontal de Privilégios*)**:
   Separa usuários que possuem **o mesmo nível de permissão**, impedindo que um acesse os recursos privados do outro (por exemplo: o Cliente `A` não pode ver os exames médicos ou pedidos do Cliente `B`). O caso mais famoso de falha horizontal é o **IDOR / BOLA**, que estudaremos especificamente no item [3.9](#s3-9).
3. **Controle de Acesso Dependente de Contexto (*Context-Dependent / Business Logic Access Control*)**:
   Garante que um usuário só possa executar ações respeitando a **máquina de estados e o fluxo de negócio** da aplicação. Exemplo: um usuário não pode pular diretamente da etapa `1 (Carrinho)` para a etapa `4 (Confirmação de Pedido Pago)` sem passar pela etapa `3 (Pagamento Aprovado)`, nem pode editar os itens de um pedido depois que o status mudou para `"Enviado"`.

Quanto ao modelo arquitetural, sistemas seguros implementam o controle de acesso usando:
- **RBAC (Role-Based Access Control — Controle de Acesso Baseado em Papéis)**: as permissões são atribuídas a papéis (`admin`, `editor`, `leitor`) e os usuários recebem papéis.
- **ABAC (Attribute-Based Access Control — Controle de Acesso Baseado em Atributos)**: a decisão avalia atributos do sujeito (departamento, cargo), do recurso (dono do documento, status do pedido) e do ambiente (horário, origem de rede).

##### 3. Por que acontece

A quebra de controle de acesso ocorre principalmente pelos seguintes erros de projeto e implementação:
- **"Segurança por Obscuridade" apenas na Interface (Front-End)**: o desenvolvedor esconde o botão *"Painel Admin"* no React/Angular/HTML com um `if (user.role === 'admin')`, mas **esquece de validar a permissão no endpoint da API no servidor (Back-End)**. Qualquer pessoa que enviar a requisição HTTP diretamente para a rota da API conseguirá executá-la!
- **Confiança em parâmetros de permissão enviados pelo cliente**: ler o papel do usuário a partir de um cookie não assinado (`Cookie: admin=true`), de um header customizado ou do próprio corpo JSON enviado pelo navegador (`{"nome": "Carlos", "role": "admin"}` — vulnerabilidade conhecida como **Mass Assignment / Atribuição em Massa**).
- **Verificação de acesso apenas no método `GET`**: proteger `GET /admin/config` exigindo perfil de administrador, mas esquecer o middleware de autorização na rota `POST /admin/config` ou `PUT /admin/config`.
- **Política *Default Allow* (Permitir por padrão)**: configurar o roteador do servidor para liberar todas as rotas por padrão, exigindo que o desenvolvedor lembre de adicionar manualmente um decorador `@require_admin` em cada nova rota criada (basta esquecer em uma rota para abrir uma brecha crítica).

##### 4. Exemplo didático de código vulnerável

Considere uma API REST em **Node.js (Express)** de um sistema acadêmico fictício (`https://api.exemplo.test`) onde apenas coordenadores (`role === 'coordenador'`) deveriam poder aprovar matrículas ou alterar o papel de usuários:

```javascript
// [CÓDIGO VULNERÁVEL A BROKEN ACCESS CONTROL — ESTUDO EM LABORATÓRIO]

// Rota 1: Atualização de perfil do próprio usuário (Vulnerável a Mass Assignment / Escalação Vertical)
app.put('/api/v1/perfil', autenticarSessao, async (req, res) => {
  const usuarioId = req.user.id;
  // ERRO 1: O servidor aceita `role` vindo diretamente do req.body enviado pelo cliente!
  const { nome, telefone, role } = req.body;

  await db.query(
    'UPDATE usuarios SET nome = $1, telefone = $2, role = COALESCE($3, role) WHERE id = $4',
    [nome, telefone, role, usuarioId]
  );
  return res.status(200).json({ mensagem: 'Perfil atualizado!' });
});

// Rota 2: Exclusão de turma (Vulnerável a falta de verificação de Role no Back-End)
// O desenvolvedor escondeu o botão "Excluir Turma" no Front-End para alunos,
// mas colocou apenas `autenticarSessao` (AuthN) sem verificar `autorizarPapel` (AuthZ)!
app.delete('/api/v1/admin/turmas/:turmaId', autenticarSessao, async (req, res) => {
  await db.query('DELETE FROM turmas WHERE id = $1', [req.params.turmaId]);
  return res.status(200).json({ mensagem: 'Turma excluída com sucesso.' });
});
```

##### 5. Como identificar

1. **Matriz de Acesso (Papel $\times$ Endpoint)**: liste todos os endpoints da API e os papéis existentes no sistema (Anônimo, Usuário Comum, Operador, Admin). Verifique no código-fonte do servidor se cada rota possui um middleware explícito de autorização (`AuthZ`), e não apenas de autenticação (`AuthN`).
2. **Inspeção de Bundles JavaScript no Front-End**: arquivos JavaScript compilados frequentemente contêm a lista completa de rotas da aplicação (incluindo rotas `/admin/...` ou `/internal/...`), mesmo quando o usuário logado é um cliente comum.
3. **Verificação de *Mass Assignment***: observe se endpoints de criação/edição (`POST`/`PUT`/`PATCH`) aceitam propriedades extras como `role`, `isAdmin`, `permissoes` ou `saldo`.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um laboratório autorizado, realiza-se o teste com **duas contas de níveis diferentes** (Conta A = Administrador/Coordenador; Conta B = Aluno/Usuário Comum):

1. **Passo 1 — Mapeamento com a Conta de Maior Privilégio**:
   - O estudante faz login como Coordenador (Conta A) e executa uma ação administrativa legítima (`DELETE /api/v1/admin/turmas/10`), observando a estrutura exata da requisição na aba **Network** do DevTools.
2. **Passo 2 — Repetição com a Conta de Menor Privilégio**:
   - O estudante faz login como Aluno (Conta B) e envia a mesma chamada `DELETE /api/v1/admin/turmas/10` usando a sessão/token do Aluno.
   - **Diagnóstico**: Se o servidor responder `200 OK` (em vez de `403 Forbidden`), está comprovada a falha de **Escalação Vertical de Privilégios**.
3. **Passo 3 — Teste de *Mass Assignment* no Perfil**:
   - Logado como Aluno, ao atualizar o próprio perfil em `PUT /api/v1/perfil`, o estudante adiciona o campo `"role": "coordenador"` no JSON enviado e verifica se o servidor altera seu nível de acesso.

##### 7. Como corrigir

1. **[REQUISITO] Impor Controle de Acesso Exclusivamente no Servidor (Back-End)**:
   Toda decisão de autorização deve ser tomada no servidor, consultando a identidade e as permissões da sessão validada no lado do servidor. Ocultar botões no front-end serve apenas para usabilidade (UX), **nunca** para segurança.
2. **[REQUISITO] Adotar Política *Deny by Default* (Negar por Padrão)**:
   Configure um middleware global que bloqueie qualquer requisição que não tenha sido explicitamente autorizada para o papel do usuário.
3. **[REQUISITO] Prevenir *Mass Assignment* com DTOs / Allowlist de Campos**:
   Extraia do `req.body` **exclusivamente** os campos que o usuário tem permissão de editar (`nome`, `telefone`), ignorando e jamais vinculando ao banco de dados campos sensíveis como `role`, `is_admin`, `creditos` ou `status_pagamento`.

Veja o código anterior **completamente corrigido**:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO — CONTROLE DE ACESSO BASEADO EM PAPÉIS (RBAC)]

// Middleware centralizado e reutilizável de Autorização (AuthZ)
function exigirPapel(...papeisPermitidos) {
  return (req, res, next) => {
    // 1. Garante que o usuário está autenticado (AuthN)
    if (!req.user || !req.user.id) {
      return res.status(401).json({ erro: 'Autenticação necessária.' });
    }
    // 2. Garante que o papel do usuário (lido da sessão do servidor!) está na lista permitida (AuthZ)
    if (!papeisPermitidos.includes(req.user.role)) {
      console.warn(`[AUDITORIA AUTHZ] Acesso negado para usuario=${req.user.id} (role=${req.user.role}) em ${req.method} ${req.originalUrl}`);
      return res.status(403).json({ erro: 'Acesso negado: permissão insuficiente.' });
    }
    next();
  };
}

// Rota 1 Corrigida: Allowlist estrita de campos editáveis (Sem Mass Assignment!)
app.put('/api/v1/perfil', autenticarSessao, async (req, res) => {
  const usuarioId = req.user.id;
  // Extrai APENAS `nome` e `telefone`. Mesmo que o cliente envie `role: "coordenador"`, é ignorado!
  const nome = String(req.body.nome || '').trim();
  const telefone = String(req.body.telefone || '').trim();

  await db.query(
    'UPDATE usuarios SET nome = $1, telefone = $2 WHERE id = $3',
    [nome, telefone, usuarioId]
  );
  return res.status(200).json({ mensagem: 'Perfil atualizado com segurança!' });
});

// Rota 2 Corrigida: Exige explicitamente o papel 'coordenador' ou 'admin' no servidor
app.delete(
  '/api/v1/admin/turmas/:turmaId',
  autenticarSessao,
  exigirPapel('coordenador', 'admin'),
  async (req, res) => {
    await db.query('DELETE FROM turmas WHERE id = $1', [req.params.turmaId]);
    return res.status(200).json({ mensagem: 'Turma excluída com sucesso.' });
  }
);
```

##### 8. Como testar se a correção funcionou

1. Envie `PUT /api/v1/perfil` contendo `{"nome": "Aluno", "telefone": "11999999999", "role": "coordenador"}` e consulte o perfil em seguida: o campo `role` deve permanecer `"aluno"`.
2. Com a sessão de um usuário `"aluno"`, envie `DELETE /api/v1/admin/turmas/10`: a API deve retornar obrigatoriamente **`403 Forbidden`** (e não `200 OK`), mantendo o registro intacto no banco.
3. Sem enviar credenciais de sessão (usuário anônimo), tente acessar a mesma rota: a API deve retornar **`401 Unauthorized`**.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Coluna Status (`200 OK` vs `401 Unauthorized` vs `403 Forbidden`)**: Lembre-se da regra clássica de prova detalhada na [Seção 1.5](./01a-fundamentos-web-parte1.md#s1): **`401 Unauthorized`** significa falha de **autenticação** (usuário não identificado), enquanto **`403 Forbidden`** significa falha de **autorização** (o servidor sabe quem você é, mas você não tem permissão). Quando uma rota administrativa acessada por um usuário comum retorna `200 OK` em vez de `403 Forbidden`, há uma quebra de controle de acesso!
  - **Aba Response / Preview**: Respostas JSON do endpoint de login ou perfil que retornam flags como `"isAdmin": false, "role": "user", "permissions": [...]`, seguidas por requisições `PUT`/`POST` onde o front-end reenvia esse mesmo objeto completo de volta para o servidor.
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - **Storage > Cookies / Local Storage**: Presença de cookies em texto puro ou flags no `localStorage` como `role=user`, `admin=0` ou `plano=gratuito`. Se alterar `admin=0` para `admin=1` na aba **Application** liberar apenas a tela visual no navegador (mas as chamadas de API na aba **Network** falharem com `403 Forbidden`), o servidor está seguro; porém, se o próprio servidor ler o cookie `admin=1` sem assinatura criptográfica e liberar os dados na aba **Network**, há uma vulnerabilidade crítica de **Broken Access Control**!
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Variáveis globais expostas em `window` (como `window.__INITIAL_STATE__` ou `window.currentUser`) ou bundles JavaScript carregados que revelam caminhos ocultos de rotas de administração (`/api/internal/export-all-users`).

---

<a id="s3-6"></a>
### 3.6 Authentication Failures (Falhas de Autenticação)

##### 1. Definição

**Authentication Failures (Falhas de Autenticação)** — denominada *Identification and Authentication Failures* em **A07:2021** e simplificada para *Authentication Failures* em **A07:2025** no OWASP Top 10 (associada principalmente à **CWE-287**, **CWE-307** e **CWE-798**) — abrange todas as fragilidades nos mecanismos que verificam a **identidade de um usuário, dispositivo ou serviço** e gerenciam o ciclo de vida de sua sessão autenticada.

Quando a autenticação falha, um atacante consegue assumir a identidade de usuários legítimos ou administradores (*Account Takeover — ATO*), passando a operar no sistema como se fosse o próprio titular da conta.

##### 2. Como funciona

Um subsistema de autenticação web não se resume à tela de login: ele engloba o cadastro de credenciais, o formulário de login, a verificação de segundo fator (**MFA — Multi-Factor Authentication**), o fluxo de *"Esqueci minha senha"*, a emissão e renovação de tokens/cookies de sessão e o encerramento da sessão (*logout*). As falhas de autenticação exploram brechas em qualquer um desses elos:

| Vetor de Ataque à Autenticação | Funcionamento Técnico |
| :--- | :--- |
| **1. Brute Force (Força Bruta)** | O atacante testa sistematicamente milhares de senhas diferentes contra uma **única conta específica** (ex.: `admin@loja-ficticia.test`) até encontrar a combinação correta, aproveitando-se da ausência de limitação de taxa (*rate limiting*). |
| **2. Credential Stuffing (Preenchimento de Credenciais)** | Explora a reutilização de senhas pelos usuários. O atacante utiliza listas massivas de pares `e-mail:senha` vazados de **outros sites** na Internet e testa automaticamente esses pares no endpoint de login da aplicação-alvo. |
| **3. Password Spraying (Pulverização de Senhas)** | Para evitar o bloqueio de uma conta por excesso de tentativas (*account lockout*), o atacante escolhe **uma única senha muito comum** (ex.: `Mudar@123` ou `Empresa2026!`) e a testa uma única vez contra **milhares de usuários diferentes** da organização. |
| **4. Enumeração de Usuários (*User Enumeration*)** | A aplicação revela se um e-mail ou CPF está cadastrado no banco por meio de **mensagens de erro diferentes** (*"Usuário não encontrado"* vs. *"Senha incorreta"*), **códigos HTTP diferentes** (`404` vs. `401`) ou **diferença no tempo de resposta (*Timing Attack*)** (demorar `250 ms` calculando o hash bcrypt quando o usuário existe vs. responder em `3 ms` quando o usuário não existe). |
| **5. Falhas na Recuperação de Senha** | Tokens de *"Esqueci minha senha"* gerados com valores previsíveis (timestamp, `Math.random()`, ID sequencial), que não expiram rapidamente ou que são vazados para terceiros via cabeçalho `Referer` quando a página de redefinição carrega scripts externos. |
| **6. Persistência Indevida de Sessão** | Quando o usuário clica em *"Sair" (Logout)* ou altera sua senha por suspeita de invasão, a aplicação apenas apaga o cookie no navegador, mas **não invalida a sessão ou os tokens no servidor**, permitindo que um token capturado anteriormente continue funcionando. |

##### 3. Por que acontece

As falhas de autenticação ocorrem por decisões inadequadas de implementação:
- Ausência de controle de taxa de requisições (**Rate Limiting** com status `429 Too Many Requests`), atraso progressivo (*exponential backoff*) ou desafios anti-automação em rotas de login, cadastro e recuperação de senha.
- Permitir senhas fracas, curtas ou presentes em listas conhecidas de senhas comprometidas (ex.: `123456`, `password`, `admin123`), contrariando as diretrizes do **NIST SP 800-63B**.
- Retornar mensagens de erro excessivamente detalhadas no login ou executar o cálculo caro de hash (Argon2id/bcrypt) apenas quando o e-mail existe no banco (criando um canal lateral de tempo — *timing side-channel*).
- Ausência de **MFA (Autenticação Multifator)** ou de suporte a chaves resistentes a phishing (**WebAuthn / Passkeys / FIDO2**) para contas privilegiadas.

##### 4. Exemplo didático de código vulnerável

Considere um endpoint de login (`POST /api/v1/auth/login`) em **Node.js (Express)** que acumula quatro falhas clássicas de autenticação cobradas em provas:

```javascript
// [CÓDIGO VULNERÁVEL A FALHAS DE AUTENTICAÇÃO — ESTUDO EM LABORATÓRIO]
app.post('/api/v1/auth/login', async (req, res) => {
  const { email, senha } = req.body;

  // ERRO 1: Ausência total de Rate Limiting (permite milhares de tentativas por segundo)
  const resultado = await db.query('SELECT id, email, senha_hash, role FROM usuarios WHERE email = $1', [email]);
  const usuario = resultado.rows[0];

  if (!usuario) {
    // ERRO 2: Enumeração explícita de usuários (mensagem diferente + retorno imediato em ~2ms!)
    return res.status(404).json({ erro: 'Este endereço de e-mail não está cadastrado no sistema.' });
  }

  // ERRO 3: Só executa o bcrypt (~250ms) se o usuário existir, criando também Enumeração por Tempo!
  const senhaCorreta = await bcrypt.compare(senha, usuario.senha_hash);
  if (!senhaCorreta) {
    return res.status(401).json({ erro: 'A senha informada está incorreta para este usuário.' });
  }

  // ERRO 4: Reutiliza a sessão existente sem regenerar o ID de sessão e não exige MFA para admin
  req.session.usuarioId = usuario.id;
  req.session.role = usuario.role;
  return res.status(200).json({ mensagem: 'Login realizado', usuario: { id: usuario.id, email: usuario.email } });
});
```

##### 5. Como identificar

1. **Teste de Mensagens e Status de Erro**: envie um login com e-mail inexistente (`naoexiste@exemplo.test`) e outro com e-mail existente mas senha errada. Compare o código HTTP, o texto da resposta JSON e o tempo de resposta (**TTFB**).
2. **Teste de Limite de Tentativas (*Rate Limiting*)**: verifique se o envio de 20 tentativas consecutivas de login com senha incorreta em poucos segundos aciona o bloqueio temporário (`429 Too Many Requests` com header `Retry-After`).
3. **Política de Senha e Logout**: verifique se o sistema aceita cadastrar a senha `123` e se uma sessão antiga continua funcionando após o logout.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um laboratório autorizado (como o **DVWA** ou **OWASP Juice Shop**):

1. **Passo 1 — Diagnóstico de Enumeração de Usuários (Mensagem e Timing)**:
   - O estudante envia uma tentativa para `usuario_inexistente@loja-ficticia.test` e observa na aba **Network**: Status `404 Not Found`, corpo `{"erro": "Este endereço de e-mail não está cadastrado..."}` e `TTFB = 4 ms`.
   - Em seguida, envia para `admin@loja-ficticia.test` com senha errada e observa: Status `401 Unauthorized`, corpo `{"erro": "A senha informada está incorreta..."}` e `TTFB = 260 ms`.
   - Com isso, fica provado que qualquer agente externo consegue descobrir quais clientes possuem conta na loja!
2. **Passo 2 — Verificação de Ausência de Rate Limiting**:
   - No laboratório, um script simples envia 30 requisições sequenciais e todas recebem `401 Unauthorized` sem qualquer bloqueio `429 Too Many Requests`.

##### 7. Como corrigir

1. **[REQUISITO] Respostas Uniformes e Mitigação de Timing Attack (Contra Enumeração)**:
   - Retorne exatamente o **mesmo código de status (`401 Unauthorized`)** e a **mesma mensagem genérica** (*"E-mail ou senha inválidos."*) tanto quando o e-mail não existe quanto quando a senha está errada. Nos fluxos de *"Esqueci minha senha"*, responda sempre: *"Se o e-mail informado estiver cadastrado, você receberá as instruções em instantes"*.
   - Quando o usuário não for encontrado no banco, execute mesmo assim uma verificação contra um **hash fictício constante (*dummy hash*)** para que o tempo de CPU gasto com **Argon2id** ou **bcrypt** seja idêntico (~250 ms) em ambos os casos!
2. **[REQUISITO] Rate Limiting Estrito e Proteção contra Força Bruta**:
   Aplique limitação de taxa por endereço IP e por identificador de conta (retornando `429 Too Many Requests` com o cabeçalho `Retry-After`).
3. **[REQUISITO] Regeneração de Sessão no Login e Invalidação no Logout**:
   Chame `req.session.regenerate()` imediatamente após autenticar o usuário (prevenindo *Session Fixation*) e `req.session.destroy()` no servidor durante o logout, além de invalidar sessões ativas quando o usuário alterar a senha.
4. **[RECOMENDAÇÃO] MFA (Autenticação Multifator) e Políticas NIST SP 800-63B**:
   Exija senhas com mínimo de 8 a 12 caracteres (permitindo frases-senha longas de até 64+ caracteres), bloqueie senhas presentes em listas de vazamentos conhecidos e ofereça MFA via TOTP (Time-Based One-Time Password) ou WebAuthn/Passkeys.

Veja o código anterior **completamente corrigido**:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO — AUTENTICAÇÃO RESISTENTE A ENUMERAÇÃO E FORÇA BRUTA]
const rateLimit = require('express-rate-limit');
const bcrypt = require('bcrypt');

// Hash bcrypt fictício pré-gerado no boot do servidor para igualar o tempo de resposta (Anti-Timing Attack)
const DUMMY_BCRYPT_HASH = '$2b$12$KIX9r3b2qV1m7pL8xW0zO.u9Y5vN4cM2jH1gF6dS8aP0eR3tY7uW.';

// [REQUISITO] Rate Limiter: máximo de 5 tentativas a cada 15 minutos por IP
const limiterLogin = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  standardHeaders: true, // Inclui headers RateLimit-* e Retry-After
  legacyHeaders: false,
  message: { erro: 'Muitas tentativas de login. Tente novamente após 15 minutos.' }
});

app.post('/api/v1/auth/login', limiterLogin, async (req, res) => {
  const email = String(req.body.email || '').trim().toLowerCase();
  const senha = String(req.body.senha || '');

  const resultado = await db.query(
    'SELECT id, email, senha_hash, role, mfa_habilitado FROM usuarios WHERE email = $1',
    [email]
  );
  const usuario = resultado.rows[0];

  // [REQUISITO] Executa SEMPRE o bcrypt.compare (contra o hash real ou contra o DUMMY_BCRYPT_HASH)
  // Isso garante que o tempo de resposta (~250ms) seja idêntico se o usuário existir ou não!
  const hashAlvo = usuario ? usuario.senha_hash : DUMMY_BCRYPT_HASH;
  const senhaValida = await bcrypt.compare(senha, hashAlvo);

  if (!usuario || !senhaValida) {
    // [REQUISITO] Mesma mensagem genérica e mesmo status 401 para ambos os casos
    return res.status(401).json({ erro: 'Credenciais inválidas (e-mail ou senha incorretos).' });
  }

  // [REQUISITO] Regenerar o ID da sessão após autenticar (previne Session Fixation)
  req.session.regenerate((err) => {
    if (err) {
      return res.status(500).json({ erro: 'Erro interno ao iniciar sessão.' });
    }
    req.session.usuarioId = usuario.id;
    req.session.role = usuario.role;
    return res.status(200).json({
      mensagem: 'Autenticação concluída com sucesso.',
      usuario: { id: usuario.id, email: usuario.email }
    });
  });
});
```

##### 8. Como testar se a correção funcionou

1. Envie uma tentativa com e-mail inexistente e outra com e-mail válido + senha errada: confirme que ambas retornam **`401 Unauthorized`**, o **mesmo corpo JSON** (`"Credenciais inválidas..."`) e **tempos de resposta semelhantes** (~250 ms na aba **Timing**).
2. Envie 6 tentativas seguidas com senha incorreta: confirme que a 6ª requisição é bloqueada com status **`429 Too Many Requests`** e cabeçalho `Retry-After`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Coluna Status e Aba Response**: Diferença nos códigos HTTP (`404` vs `401`) ou nas mensagens de erro do endpoint `/login` ou `/recuperar-senha` permitindo enumeração de contas.
  - **Aba Timing (Waiting for server response — TTFB)**: Discrepância sistemática de tempo (ex.: `3 ms` para usuário inexistente versus `300 ms` para usuário existente) visível claramente na barra verde do Waterfall/Timing.
  - **Response Headers (`Retry-After`, `RateLimit-Remaining`)**: A ausência de bloqueio `429 Too Many Requests` após dezenas de chamadas rápidas indica falta de proteção contra força bruta.
  - **Aba Payload**: Senhas sendo enviadas na URL via método `GET` (ex.: `GET /login?user=a&senha=123`) ou sobre protocolo `http://` não criptografado (visível na coluna **Scheme**).
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - **Storage > Cookies**: Observar se o valor do cookie de sessão (`session_id`) muda após o login e se é efetivamente removido/invalidado via `Set-Cookie` com expiração no passado (`Max-Age=0`) ou header `Clear-Site-Data: "cookies", "storage"` após clicar em Logout.
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Desenvolvedores que esquecem logs de depuração no formulário de login imprimindo credenciais (`console.log("Enviando login:", payload)`), deixando a senha em texto claro registrada no Console do navegador.

---

<a id="s3-7"></a>
### 3.7 Security Misconfiguration (Configuração Incorreta de Segurança)

##### 1. Definição

**Security Misconfiguration (Configuração Incorreta de Segurança)** — classificada em **A05:2021** e que subiu para a **2ª posição mais crítica em A02:2025** no OWASP Top 10 (mapeada na **CWE-16** e **CWE-209**) — ocorre quando um servidor web, banco de dados, framework de aplicação, container ou serviço em nuvem é implantado em produção com configurações padrão inseguras, modos de depuração (*debug*) ativos, permissões abertas em excesso, cabeçalhos de segurança ausentes ou exposição acidental de arquivos técnicos e detalhes internos da pilha de software.

Mesmo um código-fonte perfeitamente escrito sem SQL Injection ou XSS torna-se completamente vulnerável se o servidor onde ele roda expuser o arquivo `.env` com as senhas do banco de dados ou deixar um painel administrativo com a senha padrão `admin:admin`.

##### 2. Como funciona

Em arquiteturas modernas, uma aplicação web depende de múltiplas camadas de configuração: sistema operacional, container Docker/Kubernetes, proxy reverso (Nginx/Apache), runtime/framework (Node.js/Express, Python/Django/Flask, Java/Spring Boot, PHP/Laravel) e serviços de nuvem (buckets S3, políticas de CORS). Uma falha de configuração em qualquer uma dessas camadas expõe a aplicação das seguintes formas:

1. **Modo de Depuração (*Debug Mode*) Ativo em Produção**:
   Frameworks como Django (`DEBUG = True`), Laravel (`APP_DEBUG=true`), Flask (`debug=True`) ou Express em `NODE_ENV=development` exibem páginas de erro interativas completas quando ocorre uma exceção (`500 Internal Server Error`). Essas páginas revelam o **Stack Trace (rastro de pilha)** completo, trechos do código-fonte do servidor, caminhos absolutos de diretórios (`/var/www/app/...`) e, muitas vezes, **todas as variáveis de ambiente** (`DATABASE_URL`, `AWS_SECRET_ACCESS_KEY`, `JWT_SECRET`) diretamente no navegador!
2. **Exposição de Arquivos Ocultos, de Configuração ou de Código-Fonte**:
   O servidor web (Nginx/Apache) serve por engano pastas ou arquivos que deveriam ser bloqueados:
   - Diretório **`.git/`** (`https://loja-ficticia.test/.git/config` — permite reconstruir todo o código-fonte privado da empresa!);
   - Arquivo **`.env`** contendo chaves de API e senhas do banco;
   - Arquivos de **Source Maps (`.js.map`)** em produção, que permitem reconstruir o código TypeScript/React original com todos os comentários internos dos desenvolvedores;
   - Arquivos de backup deixados na raiz pública (`backup.sql`, `config.php.bak`, `dump.tar.gz`).
3. **Contas Padrão, Listagem de Diretórios e Recursos Desnecessários**:
   Manter credenciais de fábrica (`admin/admin`, `root/root`, `postgres/postgres`), deixar o *Directory Listing* (`autoindex on`) ativado no servidor web, ou deixar habilitados métodos HTTP de diagnóstico como `TRACE` ou endpoints internos de monitoramento (como `/actuator/env` e `/actuator/heapdump` no Spring Boot ou `/server-status` no Apache) abertos para a Internet.
4. **CORS Permissivo e Ausência de Headers de Segurança**:
   Configurar `Access-Control-Allow-Origin` refletindo qualquer origem com `Access-Control-Allow-Credentials: true` (permitindo que qualquer site malicioso leia dados privados da API) ou omitir headers essenciais como `Strict-Transport-Security`, `Content-Security-Policy` e `X-Content-Type-Options` (estudados em profundidade na [Seção 5](./05-headers-de-seguranca.md#s5)).

##### 3. Por que acontece

- **Diferença entre ambientes (*Environment Drift*)**: configurações cómodas para o ambiente local do desenvolvedor (`DEBUG=true`, CORS aberto para `*`, Source Maps públicos) são copiadas acidentalmente para o ambiente de produção por falta de um processo automatizado de *hardening* (endurecimento de segurança).
- **Padrões inseguros (*Insecure Defaults*)**: ferramentas e frameworks antigos que priorizam facilidade inicial de uso em vez de segurança por padrão (*Secure by Default*).
- **Vazamento de metadados (*Banner Grabbing*)**: manter cabeçalhos padrão como `Server: Apache/2.4.49 (Unix)` ou `X-Powered-By: Express / PHP/8.1.2`, que informam imediatamente a um scanner automatizado quais CVEs públicas existem para aquela versão exata.

##### 4. Exemplo didático de código vulnerável

Veja um servidor **Node.js (Express)** configurado de forma insegura e implantado diretamente em produção:

```javascript
// [CONFIGURAÇÃO VULNERÁVEL EM PRODUÇÃO — ESTUDO EM LABORATÓRIO]
const express = require('express');
const cors = require('cors');
const app = express();

// ERRO 1: Não desativa o header `X-Powered-By: Express` e não adiciona headers de segurança (Helmet)

// ERRO 2: Configuração de CORS gravemente insegura (reflete qualquer origem + permite credenciais!)
app.use(cors({
  origin: true,      // Reflete qualquer header `Origin` enviado pelo atacante!
  credentials: true  // Permite que sites externos leiam respostas autenticadas por cookies!
}));

// ERRO 3: Serve toda a pasta raiz do projeto como estática (`__dirname`),
// expondo acidentalmente `.env`, `package.json`, `.git` e o próprio código do servidor!
app.use(express.static(__dirname));

app.get('/api/v1/relatorio', async (req, res, next) => {
  try {
    throw new Error('Falha de conexão com postgres://admin:SenhaSuperSecreta@10.0.1.50:5432/prod_db');
  } catch (err) {
    next(err);
  }
});

// ERRO 4: Handler de erro verboso que devolve o Stack Trace e variáveis internas ao cliente!
app.use((err, req, res, next) => {
  res.status(500).json({
    erro: err.message,
    stack: err.stack,
    ambiente: process.env // Vaza TODAS as variáveis de ambiente do servidor no JSON!
  });
});
```

##### 5. Como identificar

1. **Inspeção de Cabeçalhos de Resposta HTTP**: verifique a presença de `Server`, `X-Powered-By`, `X-AspNet-Version`, políticas de `Access-Control-Allow-*` e a ausência de headers de proteção (`CSP`, `HSTS`, `X-Content-Type-Options`, `X-Frame-Options`).
2. **Análise de Respostas de Erro (`400`, `404`, `500`)**: provoque erros benignos (como enviar um JSON malformado ou acessar uma rota inexistente `/rota-inexistente-123`) e verifique se o servidor devolve páginas padrão do framework com stack traces.
3. **Verificação de Arquivos Sensíveis e Source Maps**: confira se a aplicação publica arquivos `.js.map` ou permite acesso a caminhos de configuração.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório (como o **OWASP Juice Shop**):

1. **Passo 1 — Análise de Headers e Erros**:
   - O estudante abre a aba **Network** do DevTools e observa nos **Response Headers** a linha `X-Powered-By: Express` e a ausência de `Content-Security-Policy`.
   - Ao enviar um caractere inválido em um endpoint da API, recebe um status `500` contendo o *stack trace* completo com caminhos de arquivos do servidor (`/app/routes/order.js:45:12`).
2. **Passo 2 — Teste de Configuração de CORS**:
   - No laboratório, o estudante verifica que, ao enviar uma requisição com `Origin: https://qualquer-site.test`, o servidor responde com `Access-Control-Allow-Origin: https://qualquer-site.test` e `Access-Control-Allow-Credentials: true`.

##### 7. Como corrigir

1. **[REQUISITO] Desativar Modo Debug e Mensagens de Erro Verbosas em Produção**:
   Defina `NODE_ENV=production` (ou `DEBUG=False`), registre *stack traces* exclusivamente em arquivos/serviços de log internos e retorne mensagens padronizadas e genéricas ao cliente.
2. **[REQUISITO] Isolar Diretório Público e Bloquear Arquivos Ocultos**:
   Sirva arquivos estáticos apenas de uma subpasta dedicada (`public/` ou `dist/`), bloqueie qualquer acesso a arquivos iniciados por ponto (`location ~ /\. { deny all; }` no Nginx) e não publique arquivos `.map` em produção.
3. **[REQUISITO] Configurar CORS com Allowlist Estrita e Adicionar Headers de Segurança**:
   Permita apenas origens explicitamente listadas, remova cabeçalhos de versão (`app.disable('x-powered-by')` e `server_tokens off;` no Nginx) e aplique headers de segurança padronizados.

Veja o código anterior **completamente corrigido**:

```javascript
// [CONFIGURAÇÃO CORRIGIDA E ENDURECIDA (HARDENED) PARA PRODUÇÃO]
const express = require('express');
const path = require('path');
const cors = require('cors');
const helmet = require('helmet');
const app = express();

// 1. [REQUISITO] Remove o cabeçalho `X-Powered-By: Express` e aplica headers de segurança HTTP via Helmet
app.disable('x-powered-by');
app.use(helmet());

// 2. [REQUISITO] Allowlist estrita de origens permitidas no CORS
const ORIGENS_PERMITIDAS = ['https://loja-ficticia.test', 'https://app.loja-ficticia.test'];
app.use(cors({
  origin: (origin, callback) => {
    if (!origin || ORIGENS_PERMITIDAS.includes(origin)) {
      return callback(null, true);
    }
    return callback(new Error('Origem bloqueada pela política de CORS'));
  },
  credentials: true
}));

// 3. [REQUISITO] Servir APENAS a subpasta `/public` com dotfiles bloqueados
app.use('/static', express.static(path.join(__dirname, 'public'), {
  dotfiles: 'deny',
  index: false // Desativa listagem/indexação automática de diretórios
}));

// 4. [REQUISITO] Handler global de erros silencioso para o cliente e detalhado apenas no log interno
app.use((err, req, res, next) => {
  console.error('[ERRO INTERNO]', {
    rota: req.originalUrl,
    mensagem: err.message,
    stack: err.stack
  });

  res.status(500).json({
    erro: 'Erro interno do servidor. Se o problema persistir, contate o suporte.'
  });
});
```

##### 8. Como testar se a correção funcionou

1. Inspecione os **Response Headers** na aba **Network**: confirme que `X-Powered-By` desapareceu e que `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff` e `X-Frame-Options` estão presentes.
2. Tente acessar `https://loja-ficticia.test/static/.env`: o servidor deve retornar `403 Forbidden` ou `404 Not Found`.
3. Provoque uma exceção em uma rota de teste e confirme que a resposta `500` traz apenas a mensagem genérica, sem *stack trace* e sem variáveis de ambiente.

##### 9. Como ela pode aparecer durante uma análise no DevTools

A vulnerabilidade de **Security Misconfiguration** é uma das mais visíveis diretamente no **Chrome DevTools**:
- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Sub-aba Headers > Response Headers**:
    - Exposição de versões exatas em `Server: nginx/1.18.0 (Ubuntu)` ou `X-Powered-By: PHP/7.4.3`.
    - Ausência de headers de segurança (`Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`).
    - Configuração perigosa de CORS (`Access-Control-Allow-Origin: *` em endpoints privados ou reflexão de `Origin` combinada com `Access-Control-Allow-Credentials: true`).
  - **Sub-aba Response / Preview**: Respostas de erro `500` ou `400` exibindo páginas de debug do Django/Laravel/Spring/Express com trechos de código do servidor e caminhos de diretórios internos.
  - **Lista de Requisições**: Carregamento de arquivos `.js.map` (Source Maps) abertos ao público em produção.
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Logs deixados por modo de desenvolvimento (`Development mode is enabled`), objetos de configuração com chaves de API de terceiros impressos no Console, ou reconstrução completa da árvore de arquivos TypeScript originais (visível também na aba **Sources** graças aos arquivos `.map` expostos).
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - **Frames > Security & Isolation**: Mostra imediatamente se o documento está sem isolamento de origem cruzada (**COOP — Cross-Origin-Opener-Policy** e **COEP — Cross-Origin-Embedder-Policy** não configurados).
  - **Storage > Cookies**: Cookies configurados sem os atributos `Secure`, `HttpOnly` e `SameSite`.

---

<a id="s3-8"></a>
### 3.8 Cryptographic Failures (Falhas Criptográficas)

##### 1. Definição

**Cryptographic Failures (Falhas Criptográficas)** — classificada como **A02:2021** e **A04:2025** no OWASP Top 10 (anteriormente chamada de *Sensitive Data Exposure* até 2017, e mapeada nas **CWE-310**, **CWE-319**, **CWE-327** e **CWE-338**) — refere-se à ausência de criptografia ou ao uso incorreto, fraco ou obsoleto de primitivas criptográficas para proteger **dados em trânsito (*data in transit*)** e **dados em repouso (*data at rest*)**, bem como na geração de segredos e tokens.

O foco desta categoria mudou de "Exposição de Dados Sensíveis" (que era o sintoma) para "Falhas Criptográficas" (que é a causa-raiz): sempre que senhas, números de cartão de crédito (sujeitos ao padrão **PCI DSS — Payment Card Industry Data Security Standard**), prontuários médicos ou dados pessoais protegidos pela **LGPD** são transmitidos sem TLS, armazenados em texto claro ou protegidos com algoritmos quebrados (como MD5 ou SHA-1), ocorre uma falha criptográfica.

##### 2. Como funciona

Na engenharia de segurança web, as falhas criptográficas manifestam-se em **cinco domínios técnicos principais**, muito cobrados em provas acadêmicas:

| Domínio Criptográfico | Prática Insegura / Vulnerável | Por que Falha Tecnicamente | Padrão Seguro Moderno (`[REQUISITO]`) |
| :--- | :--- | :--- | :--- |
| **1. Proteção em Trânsito** | Transmitir login/dados via **HTTP puro** (porta 80), permitir **Mixed Content** (página HTTPS carregando scripts/formulários via HTTP) ou aceitar **SSL 2.0/3.0, TLS 1.0 e TLS 1.1** (`[Obsoleto]`, depreciados pela RFC 8996). | O tráfego HTTP é texto claro legível por qualquer nó intermediário (*Man-in-the-Middle — MitM*). Protocolos legados sofrem ataques conhecidos (POODLE, BEAST) e não garantem *Forward Secrecy*. | **HTTPS obrigatório** em todo o site com **TLS 1.3** (RFC 8446) ou **TLS 1.2** com suites ECDHE + AEAD, aliado ao header **HSTS** (`Strict-Transport-Security`, RFC 6797). |
| **2. Armazenamento de Senhas (Hashing)** | Salvar senhas em texto puro, usar criptografia reversível, ou usar hashes rápidos como **MD5**, **SHA-1** ou mesmo **SHA-256 simples sem *salt* e sem fator de trabalho**. | Placas de vídeo (GPUs) modernas calculam **bilhões de hashes MD5 ou SHA-256 por segundo**, quebrando senhas instantaneamente por força bruta ou *Rainbow Tables* (tabelas pré-computadas) se não houver *salt* único e lentidão intencional. | Funções de derivação de chave (KDF) **lentas por design, com *salt* aleatório único por senha e uso intensivo de memória**: **Argon2id** (vencedor do *Password Hashing Competition*), **bcrypt**, **scrypt** ou **PBKDF2-HMAC-SHA256** (com $\ge 600.000$ iterações). |
| **3. Criptografia Simétrica de Dados em Repouso** | Usar cifras obsoletas (**DES**, **3DES**, **RC4**) ou usar AES no modo **ECB (Electronic Codebook)** ou CBC sem autenticação de integridade e com IV (Vetor de Inicialização) fixo. | O modo **AES-ECB** cifra blocos idênticos de 16 bytes gerando textos cifrados idênticos (vazando padrões visuais e lógicos) e não detecta adulteração do texto cifrado. | Criptografia Autenticada com Dados Associados (**AEAD**): **AES-256-GCM** ou **ChaCha20-Poly1305**, gerando um **IV/Nonce aleatório único** a cada operação. |
| **4. Geração de Números Aleatórios** | Gerar tokens de sessão, códigos de recuperação de senha ou *salts* usando **`Math.random()`** (JavaScript) ou `rand()` (PHP/C). | `Math.random()` é um **PRNG (Pseudo-Random Number Generator)** estatístico **não criptográfico**: seu estado interno pode ser previsto a partir de poucas saídas observadas! | Usar exclusivamente um **CSPRNG (Cryptographically Secure Pseudo-Random Number Generator)**: `crypto.randomBytes()` / `crypto.randomUUID()` no Node.js ou `window.crypto.getRandomValues()` no navegador. |
| **5. Gestão de Chaves e Segredos** | Embutir chaves privadas, segredos de JWT ou chaves de criptografia diretamente no código-fonte (*hardcoded*) ou enviá-las no bundle JavaScript do front-end. | Qualquer pessoa com acesso ao repositório Git ou qualquer usuário que abrir o DevTools no navegador poderá extrair a chave mestra e decifrar/forjar dados. | Armazenar chaves em cofres de segredos (**KMS / HSM / Vault**) ou variáveis de ambiente restritas no servidor, com rotação periódica de chaves. |

> [!IMPORTANT]
> **Pegadinha Clássica de Prova: Codificação $\neq$ Hashing $\neq$ Criptografia!**
> - **Codificação (*Encoding* — ex.: Base64, Base64URL, Hex, URL Encoding)**: **NÃO é segurança!** Serve apenas para representar dados binários em caracteres de texto seguros para transporte. Qualquer pessoa reverte Base64 instantaneamente com `atob()` sem precisar de nenhuma chave.
> - **Hashing (ex.: SHA-256, Argon2id, bcrypt)**: função matemática **unidirecional (irreversível)**. Não existe "descriptografar um hash"; para verificar uma senha, aplica-se a função de hash novamente sobre a entrada e comparam-se os resultados.
> - **Criptografia (*Encryption* — ex.: AES-256-GCM, RSA, ECDHE)**: transformação **bidirecional (reversível)** que protege a confidencialidade dos dados e exige uma **chave criptográfica** para decifrar o texto cifrado de volta ao texto original.

##### 3. Por que acontece

- Confusão conceitual entre algoritmos de hash rápidos para verificação de integridade de arquivos (SHA-256) e funções de hash de senha que precisam ser computacionalmente lentas e resistentes a GPU (Argon2id/bcrypt).
- Uso de `Math.random()` por conveniência ao gerar tokens de redefinição de senha ou identificadores de sessão.
- Crença equivocada de que "cifrar no JavaScript do cliente com uma chave fixa no código" protege os dados, esquecendo que o código-fonte do cliente é público e inspecionável pelo DevTools.

##### 4. Exemplo didático de código vulnerável

Considere um módulo de cadastro de usuário e geração de token de recuperação de senha em **Node.js** que apresenta três falhas criptográficas graves:

```javascript
// [CÓDIGO VULNERÁVEL A FALHAS CRIPTOGRÁFICAS — ESTUDO EM LABORATÓRIO]
const crypto = require('crypto');

// ERRO 1: Chave de criptografia hardcoded no código-fonte e algoritmo AES-ECB sem IV!
const CHAVE_FIXA = '1234567890123456'; // 16 bytes fixos e previsíveis

function cadastrarUsuarioVulneravel(email, senha, cpf) {
  // ERRO 2: Hash de senha com MD5 sem salt (quebrado em microssegundos por Rainbow Tables/GPU!)
  const senhaHash = crypto.createHash('md5').update(senha).digest('hex');

  // ERRO 3: Criptografia de CPF usando modo inseguro AES-128-ECB (sem IV e sem autenticação GCM)
  const cipher = crypto.createCipheriv('aes-128-ecb', Buffer.from(CHAVE_FIXA), null);
  let cpfCifrado = cipher.update(cpf, 'utf8', 'hex');
  cpfCifrado += cipher.final('hex');

  // ERRO 4: Token de recuperação de senha gerado com Math.random() (PRNG previsível!)
  const tokenReset = Math.random().toString(36).substring(2);

  return { email, senhaHash, cpfCifrado, tokenReset };
}
```

##### 5. Como identificar

1. **Inspeção de Rede e TLS**: verifique se todas as páginas e chamadas de API utilizam `https://`, se formulários de login não enviam dados via HTTP, se há recursos carregados via `http://` (*Mixed Content*) e se o certificado TLS e o protocolo (`TLS 1.3` / `TLS 1.2`) são válidos.
2. **Revisão de Código e Banco de Dados**: procure no código-fonte por referências a `md5`, `sha1`, `createCipher`, `ecb`, `des`, `rc4`, `Math.random()` e chaves literais (`SECRET = "..."`).
3. **Análise de Tokens e Respostas de API**: observe se respostas JSON vazam hashes de senha (ex.: strings hexadecimais de 32 caracteres típicas de MD5: `e10adc3949ba59abbe56e057f20f883e`, que corresponde a `123456`!) ou se tokens recebidos têm baixa entropia.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

1. **Passo 1 — Identificação de Hash Fraco ou Exposto**:
   - Em um desafio do **OWASP Juice Shop**, o estudante obtém a resposta de uma API que expõe o campo `password` contendo uma string hexadecimal de 32 caracteres (128 bits = formato clássico de **MD5** sem salt).
2. **Passo 2 — Identificação de PRNG Previsível ou Criptografia no Cliente**:
   - Ao solicitar a recuperação de senha no laboratório, o estudante observa que o token recebido no link (`?token=0.847291...` ou sequencial baseado no timestamp) foi gerado por `Math.random()` ou `Date.now()`, sendo facilmente previsível.

##### 7. Como corrigir

1. **[REQUISITO] Senhas sempre com Argon2id ou bcrypt (ou scrypt / PBKDF2 com alto custo)**:
   Nunca use MD5, SHA-1 ou SHA-256 direto para senhas. Utilize bibliotecas padrão como `argon2` ou `bcrypt` (custo $\ge 12$), que já geram e embutem automaticamente um *salt* criptograficamente aleatório de 128 bits no próprio resultado.
2. **[REQUISITO] Criptografia Simétrica Autenticada (AES-256-GCM) com IV Aleatório e Chave em Cofre/Ambiente**:
   Para dados sensíveis que precisam ser recuperados no futuro (como documentos fiscais ou chaves de integração), utilize **AES-256-GCM** com um IV de 12 bytes (96 bits) gerado via `crypto.randomBytes(12)` a cada operação, salvando o `iv` e a `authTag` (tag de integridade de 16 bytes) junto ao texto cifrado.
3. **[REQUISITO] Tokens Sempre com CSPRNG (`crypto.randomBytes`) e Hash no Banco**:
   Gere tokens de recuperação de senha com pelo menos 32 bytes (256 bits) de entropia via `crypto.randomBytes(32).toString('hex')` e salve no banco de dados o **hash SHA-256** desse token (assim, mesmo que o banco seja lido, ninguém consegue usar os tokens de reset pendentes!).

Veja o código anterior **completamente corrigido**:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO — BOAS PRÁTICAS CRIPTOGRÁFICAS MODERNAS]
const crypto = require('crypto');
const bcrypt = require('bcrypt');

// [REQUISITO] Chave de 256 bits (32 bytes em hex) carregada de cofre/variável de ambiente segura
const CHAVE_MESTRA_BYTES = Buffer.from(process.env.ENCRYPTION_KEY_HEX, 'hex'); // 32 bytes

// 1. Criptografia Simétrica Autenticada (AEAD) com AES-256-GCM e IV único de 96 bits (12 bytes)
function cifrarDadoSensivel(textoClaro) {
  const iv = crypto.randomBytes(12); // CSPRNG: IV único para cada registro!
  const cipher = crypto.createCipheriv('aes-256-gcm', CHAVE_MESTRA_BYTES, iv);

  const cifrado = Buffer.concat([cipher.update(textoClaro, 'utf8'), cipher.final()]);
  const authTag = cipher.getAuthTag(); // Garante integridade e autenticidade do dado cifrado

  // Armazena IV + AuthTag + TextoCifrado codificados em Base64
  return `${iv.toString('base64')}:${authTag.toString('base64')}:${cifrado.toString('base64')}`;
}

// 2. Cadastro com Hashing de Senha Forte (bcrypt com salt automático e fator de custo 12)
// e geração de Token com CSPRNG (256 bits de entropia)
async function cadastrarUsuarioSeguro(email, senha, cpf) {
  const CUSTO_BCRYPT = 12;
  const senhaHash = await bcrypt.hash(senha, CUSTO_BCRYPT);

  const cpfCifrado = cifrarDadoSensivel(cpf);

  // [REQUISITO] Token gerado com CSPRNG (32 bytes = 256 bits de entropia real)
  const tokenResetBruto = crypto.randomBytes(32).toString('hex');
  // Salva apenas o hash do token no banco para proteger contra vazamento de tabela
  const tokenResetHashBanco = crypto.createHash('sha256').update(tokenResetBruto).digest('hex');

  return { email, senhaHash, cpfCifrado, tokenResetParaEmail: tokenResetBruto, tokenResetHashBanco };
}
```

##### 8. Como testar se a correção funcionou

1. Cadastre dois usuários com a **mesma senha** e o **mesmo CPF** em ambiente de teste: verifique no banco de dados que os dois hashes `bcrypt` são **completamente diferentes** (graças ao *salt* único de cada um) e que os dois textos cifrados `AES-256-GCM` também são **completamente diferentes** (graças ao IV aleatório de 12 bytes).
2. Altere 1 único bit do texto cifrado salvo no banco e tente decifrá-lo: o `cipher.getAuthTag()` do modo GCM deve lançar imediatamente um erro de autenticação (`Unsupported state or unable to authenticate data`), comprovando a proteção de integridade!

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Coluna Scheme / Protocol**: Requisições trafegando em `http` (texto claro) em vez de `https`, ou recursos bloqueados/alertados por **Mixed Content**.
  - **Response Headers**: Ausência do header `Strict-Transport-Security` (HSTS), deixando o primeiro acesso vulnerável a ataques de *SSL Stripping*.
  - **Aba Payload e Aba Response**: Senhas, números de cartão ou CPFs trafegando meramente codificados em **Base64** (que o analista decodifica em 1 segundo no Console com `atob("...")`), ou APIs vazando hashes MD5/SHA-1 no JSON de resposta (*Excessive Data Exposure*).
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Avisos amarelos ou erros vermelhos de **Mixed Content**:
    `Mixed Content: The page at 'https://loja-ficticia.test/' was loaded over HTTPS, but requested an insecure resource 'http://cdn.loja-ficticia.test/script.js'. This request has been blocked; the content must be served over HTTPS.`
  - Erros de API Web Crypto (`DOMException: The operation is insecure`) quando se tenta usar `window.crypto.subtle` em um contexto não seguro (`http://`).
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - **Storage > Local Storage / Cookies**: Dados sensíveis "disfarçados" em Base64 no `localStorage`, ou cookies de sessão sem a flag **`Secure`** (o que permite que o navegador envie o cookie em texto claro caso o usuário faça uma requisição `http://` para o mesmo domínio!).
  - **Inspeção de Código Front-End (aba Sources)**: Chaves privadas de API, segredos de assinatura JWT ou chaves AES (`CryptoJS.AES.encrypt(..., "chaveSecreta123")`) expostos diretamente nos arquivos `.js` enviados ao navegador.

---

<a id="s3-9"></a>
### 3.9 Insecure Direct Object Reference (IDOR / BOLA)

##### 1. Definição

**Insecure Direct Object Reference (IDOR — Referência Direta Insegura a Objetos)**, catalogada sob a **CWE-639** e integrante de **A01:2021 / A01:2025 – Broken Access Control** no OWASP Top 10, é uma vulnerabilidade específica e extremamente comum de controle de acesso (geralmente **horizontal**) que ocorre quando uma aplicação utiliza um identificador fornecido diretamente pelo cliente (como um `id` numérico na URL, na query string ou no corpo JSON) para buscar, modificar ou excluir um registro no banco de dados ou um arquivo no servidor **sem verificar se o usuário autenticado na sessão atual é realmente o proprietário ou tem permissão sobre aquele objeto específico**.

No ecossistema moderno de APIs REST e GraphQL, o IDOR recebeu um nome ainda mais preciso no **OWASP API Security Top 10** (onde ocupa o **1º lugar isolado — API1:2023**): **BOLA (Broken Object Level Authorization — Quebra de Autorização no Nível do Objeto)**. Em provas e entrevistas técnicas, **IDOR e BOLA referem-se exatamente ao mesmo problema estrutural**: o servidor autentica o usuário (*sabe que você é o usuário de ID `105`*), mas ao atender `GET /api/v1/faturas/8902`, consulta apenas `WHERE id = 8902` e esquece de validar `AND usuario_id = 105`!

##### 2. Como funciona

Quase toda aplicação web referencia entidades de negócio (pedidos, faturas, mensagens privadas, prontuários, comprovantes em PDF, endereços) por meio de sua chave primária no banco de dados. Quando o usuário legítimo acessa sua página *"Meus Pedidos"*, o front-end faz uma chamada como:

```http
GET /api/v1/pedidos/1042 HTTP/1.1
Host: api.loja-ficticia.test
Cookie: __Host-sid=sessao_do_usuario_maria_id_55
```

Se o back-end da aplicação verificar apenas que o cookie `__Host-sid` é válido (autenticação OK), mas buscar o pedido no banco executando:

```sql
SELECT * FROM pedidos WHERE id = 1042;
```

sem conferir se `pedidos.usuario_id` é igual a `55` (o ID de Maria na sessão), qualquer outro cliente logado que alterar o número na URL para `1041`, `1043`, `1044`... receberá o JSON completo com nome, CPF, endereço residencial e itens comprados por todos os outros clientes da loja!

O IDOR / BOLA pode ocorrer em diferentes locais da requisição HTTP:
1. **No Path da URL (RESTful)**: `GET /api/v1/documentos/5002`, `DELETE /api/v1/enderecos/88`
2. **Na Query String**: `GET /gerar-boleto?fatura_id=9120`
3. **No Corpo da Requisição (JSON / Form Data — inclusive em operações de escrita)**:
   Ao enviar um comentário ou alterar uma senha em `PUT /api/v1/usuarios/senha` com o corpo `{"user_id": 55, "nova_senha": "..."}`, se o servidor usar o `user_id` vindo do JSON em vez do ID guardado na sessão do servidor, um atacante troca `"user_id": 55` por `"user_id": 1` e redefine a senha de outro usuário!
4. **Em Nomes de Arquivos Estáticos Previsíveis**: `GET /downloads/notas-fiscais/nf_2026_00045.pdf`.

> [!WARNING]
> **Pegadinha Clássica de Prova: "Trocar ID numérico sequencial (1, 2, 3) por UUID v4 resolve o IDOR?"**
> **NÃO!** Substituir um ID auto-incremental (`1042`) por um **UUID v4** (`f47ac10b-58cc-4372-a567-0e02b2c3d479`) dificulta a enumeração cega por tentativa e erro, sendo uma excelente **[BOA PRÁTICA]** de defesa em profundidade, mas **NÃO corrige a vulnerabilidade de IDOR/BOLA por si só**!
> Isso seria apenas *Segurança por Obscuridade*: UUIDs frequentemente vazam em logs, no cabeçalho `Referer`, em URLs compartilhadas, em endpoints públicos de listagem (ex.: avaliações de produtos que exibem o `user_uuid` do autor) ou no histórico do navegador. **[REQUISITO]** A única correção real para o IDOR/BOLA é validar no servidor, em toda requisição, se o usuário autenticado tem permissão de acesso àquele objeto!

##### 3. Por que acontece

- O desenvolvedor confunde **Autenticação** (validar que o token JWT ou cookie de sessão é válido) com **Autorização no Nível do Objeto** (validar que aquele objeto específico pertence àquele usuário).
- Acreditar que, como a interface visual do aplicativo só lista na tela os links dos pedidos do próprio usuário logado, ninguém jamais requisitará um ID diferente daqueles exibidos na tela.
- Confiar em um `usuario_id` enviado pelo front-end no corpo JSON ou na query string em vez de extrair o `usuario_id` diretamente da sessão autenticada no servidor (`req.session.usuarioId` ou `req.user.sub`).

##### 4. Exemplo didático de código vulnerável

Veja um endpoint em **Node.js (Express)** que retorna os detalhes e o endereço de entrega de um pedido específico (`GET /api/v1/pedidos/:pedidoId`):

```javascript
// [CÓDIGO VULNERÁVEL A IDOR / BOLA — ESTUDO EM LABORATÓRIO]
app.get('/api/v1/pedidos/:pedidoId', autenticarSessao, async (req, res) => {
  const pedidoId = Number(req.params.pedidoId);

  // O middleware `autenticarSessao` garantiu que `req.user.id` está logado,
  // MAS a consulta abaixo busca o pedido APENAS pelo `id` da URL,
  // sem verificar se o pedido pertence ao `req.user.id`!
  const resultado = await db.query(
    `SELECT id, usuario_id, nome_destinatario, cpf_cliente, endereco_entrega, valor_total
     FROM pedidos
     WHERE id = $1`,
    [pedidoId]
  );

  if (resultado.rows.length === 0) {
    return res.status(404).json({ erro: 'Pedido não encontrado.' });
  }

  // ERRO CRÍTICO: Devolve os dados pessoais e endereço de qualquer pedido existente no banco!
  return res.status(200).json(resultado.rows[0]);
});
```

##### 5. Como identificar

1. **Na Revisão de Código (Caixa-Branca)**:
   - Verifique todas as consultas ao banco de dados que recebem um ID vindo de `req.params`, `req.query` ou `req.body` (`findById(req.params.id)`, `WHERE id = $1`).
   - Confira se a consulta inclui obrigatoriamente o vínculo de propriedade com o sujeito autenticado (`AND usuario_id = $2` ou uma verificação explícita de política de acesso `podeAcessarPedido(req.user, pedido)`).
2. **Na Inspeção de Tráfego (Caixa-Preta em Laboratório)**:
   - Identifique todas as rotas que contêm identificadores de recursos (`/usuarios/123`, `/faturas/456`, `?docId=789`).

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um laboratório autorizado (como o **OWASP Juice Shop**, **WebGoat** ou **PortSwigger Web Security Academy**), o teste padrão para IDOR/BOLA utiliza **duas contas de teste de mesmo nível** criadas pelo próprio estudante (`Usuario_Teste_A` com ID `10` e `Usuario_Teste_B` com ID `11`):

1. **Passo 1 — Geração de Objeto na Conta A**:
   - O estudante faz login como `Usuario_Teste_A` e cria um pedido legítimo, que recebe o identificador `pedidoId = 501`. Ao acessá-lo, observa na aba **Network** a chamada `GET /api/v1/pedidos/501` retornando `200 OK`.
2. **Passo 2 — Troca de Sessão para a Conta B (Mesmo Nível Horizontal)**:
   - Em outra janela anônima (ou após fazer login como `Usuario_Teste_B`), o estudante tenta acessar o recurso pertencente à Conta A requisitando `GET /api/v1/pedidos/501` com os cookies/token de `Usuario_Teste_B`.
3. **Passo 3 — Análise do Resultado**:
   - Se o servidor retornar `200 OK` exibindo os dados do pedido `501` (que pertence a `Usuario_Teste_A`) para `Usuario_Teste_B`, a vulnerabilidade de **IDOR / BOLA** está confirmada.

##### 7. Como corrigir

1. **[REQUISITO] Verificação Obrigatória de Propriedade (*Object-Level Authorization*) no Banco de Dados**:
   Toda consulta que busca, atualiza ou exclui um objeto individual deve filtrar simultaneamente pelo ID do objeto **e** pelo ID do usuário extraído da sessão confiável do servidor (`WHERE id = $1 AND usuario_id = $2`), ou passar por uma camada de política de autorização (RBAC/ABAC) que valide se o usuário logado tem direito de acesso àquele registro.
2. **[REQUISITO] Nunca confiar em IDs de usuário vindos do corpo ou query string**:
   O identificador do usuário logado deve vir **exclusivamente** do estado de sessão no servidor (`req.session.usuarioId`) ou do payload assinado e validado do token (`req.user.id`). Quando uma rota referenciar apenas dados do próprio usuário logado, prefira endpoints implícitos como `GET /api/v1/meus-pedidos` ou `/api/v1/perfil/me` em vez de `/api/v1/usuarios/:id`.
3. **[BOA PRÁTICA] Usar Identificadores Imprevisíveis (UUID v4 / ULID) + Retorno Padronizado `404`**:
   Combine a verificação obrigatória de permissão no servidor com o uso de UUIDs públicos e retorne `404 Not Found` (ou `403 Forbidden`, conforme a política da API) quando um usuário tentar acessar um ID que pertence a outra pessoa, evitando revelar até mesmo se o ID consultado existe ou não no banco.

Veja o código anterior **completamente corrigido**:

```javascript
// [CÓDIGO CORRIGIDO E SEGURO CONTRA IDOR / BOLA]
app.get('/api/v1/pedidos/:pedidoId', autenticarSessao, async (req, res) => {
  const pedidoId = Number(req.params.pedidoId);
  // [REQUISITO] O ID do usuário vem EXCLUSIVAMENTE da sessão autenticada no servidor!
  const usuarioAutenticadoId = Number(req.user.id);

  if (!Number.isInteger(pedidoId) || pedidoId <= 0) {
    return res.status(400).json({ erro: 'Identificador de pedido inválido.' });
  }

  // [REQUISITO] A cláusula WHERE exige que o pedido exista E pertença ao usuário logado
  // (exceto se o usuário tiver papel explícito de 'auditor_admin' validado na sessão)
  const isAdmin = req.user.role === 'auditor_admin';
  const sqlQuery = isAdmin
    ? `SELECT id, codigo_rastreio, nome_destinatario, endereco_entrega, valor_total
       FROM pedidos WHERE id = $1`
    : `SELECT id, codigo_rastreio, nome_destinatario, endereco_entrega, valor_total
       FROM pedidos WHERE id = $1 AND usuario_id = $2`;

  const params = isAdmin ? [pedidoId] : [pedidoId, usuarioAutenticadoId];
  const resultado = await db.query(sqlQuery, params);

  if (resultado.rows.length === 0) {
    // Retorna 404 mesmo que o pedido exista para outro cliente, evitando enumeração de existência!
    return res.status(404).json({ erro: 'Pedido não encontrado.' });
  }

  return res.status(200).json(resultado.rows[0]);
});
```

##### 8. Como testar se a correção funcionou

1. Logado com `Usuario_Teste_B`, envie `GET /api/v1/pedidos/501` (onde `501` pertence a `Usuario_Teste_A`): a API deve retornar **`404 Not Found`** (ou `403 Forbidden`) e nenhum dado do pedido `501` deve ser retornado no corpo da resposta.
2. Logado com `Usuario_Teste_A` (o verdadeiro dono do pedido `501`), envie `GET /api/v1/pedidos/501`: a API deve retornar **`200 OK`** com os dados esperados.
3. Teste também as rotas de escrita (`PUT /api/v1/pedidos/501/cancelar`, `DELETE /api/v1/enderecos/88`) com `Usuario_Teste_B` para garantir que a checagem de propriedade (`AND usuario_id = $2`) foi aplicada tanto na leitura quanto na modificação e exclusão!

##### 9. Como ela pode aparecer durante uma análise no DevTools

O **Chrome DevTools** é a ferramenta primária para detectar indícios de IDOR/BOLA durante uma auditoria:

- **Na aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Colunas Name, Path e URL**: Endpoints que expõem chaves primárias sequenciais diretamente na URL, como `GET /api/v1/usuarios/42/cartoes`, `GET /api/v1/faturas?id=1009` ou `GET /arquivos/holerite_funcionario_312.pdf`.
  - **Aba Payload (Request Payload / Form Data)**: Requisições `POST`, `PUT` ou `PATCH` onde o front-end envia explicitamente o ID do proprietário no corpo JSON — por exemplo, `POST /api/v1/mensagens` enviando `{"remetente_id": 42, "texto": "Olá"}` ou `GET /api/v1/extrato` enviando `{"conta_id": 778}`. Sempre que o front-end precisa "dizer" ao servidor quem é o dono do recurso por meio de um parâmetro editável, há altíssimo risco de IDOR!
  - **Aba Response / Preview (combinada com *Copy as fetch* ou *Edit and Resend*)**: Em ambiente autorizado de laboratório, ao usar a função do DevTools para repetir uma requisição alterando o ID de `501` para `500`, a aba **Response** exibe código `200 OK` com dados pessoais (`nome`, `email`, `cpf`, `endereco`) de outro usuário cadastrado.
- **Na aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - **Storage > Local Storage / Session Storage**: Aplicações SPA mal projetadas que salvam `user_id: "42"` no `localStorage`, leem esse valor no JavaScript e o concatenam nas chamadas `fetch('/api/pedidos?user_id=' + localStorage.getItem('user_id'))`, sem que o servidor valide se o `user_id` da query string bate com o dono do cookie de sessão!
- **Na aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Execução de `fetch()` no REPL do Console durante testes em laboratório autorizado comprovando que chamadas para IDs adjacentes retornam objetos JSON completos de terceiros sem bloqueio de autorização.

---

---

> [⬅️ 2.3 Chrome DevTools — Aba Console](./02c-devtools-console.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [3B. Vulnerabilidades Web (3.10 a 3.19) ➡️](./03b-vulnerabilidades-parte2.md)
