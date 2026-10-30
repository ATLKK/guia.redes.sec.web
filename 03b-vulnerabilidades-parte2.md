> [⬅️ 3A. Vulnerabilidades Web (3.0 a 3.9)](./03a-vulnerabilidades-parte1.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [4. Redes de Computadores e Portas ➡️](./04-redes-de-computadores.md)

---

<a id="s3-10"></a>
### 3.10 Directory Traversal (e Directory Listing / Exposição de Diretórios)

##### 1. Definição

O termo **Directory Traversal** (Travessia de Diretórios — catalogado no **CWE**, *Common Weakness Enumeration* ou Enumeração de Fraquezas Comuns, sob o código **CWE-22**) refere-se à falha de segurança na qual uma aplicação web permite que um usuário manipule referências a arquivos ou pastas na requisição **HTTP** (*Hypertext Transfer Protocol* — Protocolo de Transferência de Hipertexto) para navegar além do diretório-raiz (*web root* ou diretório base de arquivos públicos) pretendido pelo desenvolvedor. Na literatura e em provas de certificação e concursos, essa falha frequentemente aparece associada ou contrastada com **Directory Listing** (Listagem de Diretórios, **CWE-548**), uma falha de **Security Misconfiguration** (Configuração Incorreta de Segurança — categoria **A05:2021** e **A02:2025** do **OWASP Top 10**, mantido pela *Open Web Application Security Project* — Projeto Aberto de Segurança em Aplicações Web) em que o servidor web exibe automaticamente o índice completo de arquivos e subpastas quando nenhum arquivo de índice padrão (como `index.html` ou `index.php`) está presente no diretório requisitado.

Enquanto o **Directory Listing** expõe passivamente a estrutura e os arquivos de um diretório acessível pela **URL** (*Uniform Resource Locator* — Localizador Uniforme de Recursos), o **Directory Traversal** permite escapar ativamente de uma pasta restrita usando sequências de navegação hierárquica, como `../` (*dot-dot-slash* em sistemas Unix/Linux) ou `..\` (*dot-dot-backslash* em sistemas Microsoft Windows), comprometendo a confidencialidade do servidor.

> [!NOTE]
> A distinção e a sinonímia técnica entre **Directory Traversal** e **Path Traversal**, bem como técnicas avançadas de codificação de caracteres e canonicalização de caminhos no sistema operacional, são aprofundadas na subseção [3.17 Path Traversal](#s3-17). Aqui em 3.10, focamos na navegação indevida pela árvore de diretórios da aplicação e na exposição de diretórios no servidor web.

##### 2. Como funciona

Em um servidor web ou aplicação back-end, os arquivos servidos ao público devem ficar confinados a um diretório específico do sistema de arquivos — por exemplo, `/var/www/loja-ficticia/public/relatorios/`. Quando a aplicação oferece uma funcionalidade de visualização ou download de documentos passando o nome do arquivo ou subpasta por parâmetro (como `GET /download?pasta=2026&arquivo=balanco.pdf`), o código do servidor concatena os valores recebidos diretamente ao caminho base no disco.

Se não houver higienização nem validação da fronteira do diretório, um agente malicioso pode substituir o parâmetro por sequências `../`, que nos sistemas de arquivos **POSIX** (*Portable Operating System Interface* — Interface Portátil de Sistema Operacional) e Windows representam o **diretório-pai** (*parent directory*). Cada `../` faz o resolvedor do sistema operacional subir um nível na árvore de diretórios:

```text
Caminho base configurado:   /var/www/loja-ficticia/public/relatorios/
Entrada do usuário:         ../../../../etc/passwd
Caminho concatenado:        /var/www/loja-ficticia/public/relatorios/../../../../etc/passwd

Resolução passo a passo pelo sistema de arquivos:
1. /var/www/loja-ficticia/public/relatorios/..  -> /var/www/loja-ficticia/public
2. /var/www/loja-ficticia/public/..             -> /var/www/loja-ficticia
3. /var/www/loja-ficticia/..                    -> /var/www
4. /var/www/..                                  -> /
5. / + etc/passwd                               -> /etc/passwd
```

Já no caso complementar de **Directory Listing**, quando o usuário acessa `https://loja-ficticia.test/uploads/` ou `https://loja-ficticia.test/backups/` e a diretiva de autoindexação do servidor (como `autoindex on` no Nginx ou `Options +Indexes` no Apache HTTP Server) está habilitada, o servidor gera uma página **HTML** (*Hypertext Markup Language* — Linguagem de Marcação de Hipertexto) listando todos os arquivos daquele diretório. Isso permite descobrir arquivos de backup (`.bak`, `.old`, `.sql.gz`), chaves privadas, arquivos `.env` ou código-fonte que jamais deveriam ser conhecidos pelo público.

##### 3. Por que acontece

As causas-raiz dessas vulnerabilidades combinam falhas de codificação e falhas de configuração de infraestrutura:

1. **Concatenação direta de entrada não confiável em chamadas de sistema de arquivos**: uso de funções como `fs.readFile()` (Node.js), `open()` (Python), `file_get_contents()` (PHP) ou `FileInputStream` (Java) interpolando diretamente parâmetros de query string, corpo **JSON** (*JavaScript Object Notation* — Notação de Objetos JavaScript), parâmetros de rota ou até mesmo cookies.
2. **Ausência de validação de fronteira (*jail* / *chroot* lógico)**: o desenvolvedor verifica apenas se o arquivo existe, mas não verifica se o caminho final resolvido continua sendo um descendente estrito do diretório permitido.
3. **Diretivas de autoindexação ativas em produção**: servidores web ou frameworks deixados com listagem automática de diretórios habilitada por padrão ou por conveniência durante o desenvolvimento.
4. **Armazenamento de arquivos sensíveis dentro do *DocumentRoot***: colocar arquivos `.git/`, `.env`, `config.inc`, dumps de banco de dados (`.sql`) ou arquivos compactados (`.zip`, `.tar.gz`) dentro da pasta pública servida diretamente pelo servidor HTTP.

##### 4. Exemplo didático de código vulnerável

Considere um endpoint em **Node.js com Express** de um portal acadêmico de laboratório (`https://portal.exemplo.test`) que permite a leitura de manuais em formato texto ou **PDF** (*Portable Document Format* — Formato Portátil de Documento) armazenados em uma pasta pública, acompanhado de uma configuração de Nginx com listagem de diretórios ativa:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/manuais.js (Ambiente de Laboratório)
const express = require('express');
const fs = require('fs');
const path = require('path');
const router = express.Router();

const DIRETORIO_MANUAIS = '/var/www/portal/public/manuais';

router.get('/api/manual', (req, res) => {
  const { categoria, arquivo } = req.query;

  // VULNERABILIDADE: concatenação direta sem validar se o caminho escapa da pasta base.
  // Atenção: mesmo path.join() resolve "../" permitindo sair de DIRETORIO_MANUAIS!
  const caminhoCompleto = path.join(DIRETORIO_MANUAIS, categoria || '', arquivo || '');

  if (!fs.existsSync(caminhoCompleto)) {
    return res.status(404).json({ erro: `Arquivo não encontrado em: ${caminhoCompleto}` });
  }

  // Envia o conteúdo bruto do arquivo solicitado sem verificar a qual diretório pertence
  const conteudo = fs.readFileSync(caminhoCompleto, 'utf8');
  res.type('text/plain').send(conteudo);
});

module.exports = router;
```

E no arquivo de configuração do servidor reverso Nginx (`/etc/nginx/conf.d/portal.conf`):

```nginx
# CONFIGURAÇÃO VULNERÁVEL A DIRECTORY LISTING
server {
    listen 443 ssl;
    server_name portal.exemplo.test;
    root /var/www/portal/public;

    location /manuais/ {
        # VULNERABILIDADE: exibe a lista completa de arquivos se não houver index.html
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }
}
```

Note dois erros graves no código acima: primeiro, `path.join('/var/www/portal/public/manuais', '../../.env')` resulta em `/var/www/portal/.env`, escapando perfeitamente do diretório público; segundo, quando o arquivo não existe, a resposta `404` vaza o caminho físico completo do servidor (`caminhoCompleto`), facilitando o mapeamento da estrutura de pastas pelo atacante.

##### 5. Como identificar

Durante uma revisão de código (**SAST** — *Static Application Security Testing* ou Teste Estático de Segurança de Aplicação) ou inspeção de arquitetura, busque por:

- Parâmetros HTTP cujos valores contenham nomes de arquivos, extensões (`.pdf`, `.png`, `.tmpl`, `.html`) ou nomes de pastas (`?dir=`, `?folder=`, `?file=`, `?template=`, `?lang=`, `?doc=`).
- Uso de APIs (*Application Programming Interfaces* — Interfaces de Programação de Aplicações) de leitura de disco (`fs.readFile`, `fs.createReadStream`, `res.sendFile`, `open`, ` include`, `require`) alimentadas por variáveis provenientes de `req.query`, `req.params`, `req.body` ou `req.cookies`.
- Configurações de servidor web contendo `autoindex on;` (Nginx), `Options Indexes` ou `Options +Indexes` (Apache), `directoryBrowse enabled="true"` (**IIS** — *Internet Information Services*), ou middlewares como `serve-index` no Express.
- Páginas retornando o título padrão `Index of /caminho` ao acessar URLs terminadas em `/` sem arquivo de índice.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

> [!CAUTION]
> **Aviso Legal e Ético**: Todos os procedimentos descritos a seguir destinam-se exclusivamente ao aprendizado acadêmico e à execução em ambientes de laboratório formalmente autorizados (como **DVWA** — *Damn Vulnerable Web Application*, **OWASP Juice Shop**, **OWASP WebGoat**, **PortSwigger Web Security Academy** ou containers locais do estudante). Realizar testes de intrusão ou leitura não autorizada de arquivos em sistemas de terceiros constitui crime previsto na legislação brasileira (Art. 154-A do Código Penal — Lei nº 12.737/2012 e alterações da Lei nº 14.155/2021).

Em um laboratório controlado executando a aplicação de exemplo em `http://localhost:3000`:

1. **Mapeamento do ponto de entrada**: o estudante observa que, ao clicar para baixar um regulamento no portal do laboratório, o navegador faz a requisição:
   ```http
   GET /api/manual?categoria=graduacao&arquivo=regulamento.txt HTTP/1.1
   Host: localhost:3000
   ```
2. **Teste de Directory Listing**: o estudante acessa diretamente `https://portal.exemplo.test/manuais/` no navegador. Se o servidor responder com `Index of /manuais/` exibindo links para `regulamento.txt`, `rascunho_confidencial.txt` e `../`, fica comprovada a exposição de diretórios (CWE-548).
3. **Teste de saída de diretório relativo (Traversal)**: o estudante testa se a aplicação resolve referências de diretório atual (`./`) e diretório-pai (`../`). Primeiro, envia `?categoria=graduacao/..&arquivo=graduacao/regulamento.txt`. Se o mesmo arquivo for retornado com status `200 OK`, confirma-se que o servidor avalia sequências de navegação.
4. **Leitura de arquivo de demonstração fora do *web root***: no laboratório Linux (como o módulo *File Inclusion / Path Traversal* do DVWA ou WebGoat), o estudante altera o parâmetro para subir até a raiz do sistema de arquivos e ler um arquivo padrão de leitura pública do sistema operacional, ou o arquivo de flag do exercício:
   ```http
   GET /api/manual?categoria=../../../../../../&arquivo=etc/passwd HTTP/1.1
   Host: localhost:3000
   ```
5. **Análise do resultado**: se a resposta HTTP `200 OK` trouxer linhas como `root:x:0:0:root:/root:/bin/bash`, fica demonstrado que o processo da aplicação escapou do diretório `/var/www/portal/public/manuais` e acessou a raiz `/` do sistema operacional.

##### 7. Como corrigir

A correção efetiva exige **defesa em profundidade**, combinando referências indiretas (ou validação estrita por lista de permissão), extração apenas do nome base (*basename*), verificação canônica do caminho resolvido e desativação de listagem de diretórios no servidor web.

- **[REQUISITO]** Nunca utilizar entrada do usuário diretamente em caminhos do sistema de arquivos sem antes canonicalizar o caminho com `path.resolve()` / `fs.realpathSync()` e verificar se ele inicia estritamente com o diretório base permitido acrescido do separador de diretório.
- **[REQUISITO]** Desabilitar a listagem automática de diretórios (`autoindex off` no Nginx; `Options -Indexes` no Apache) em todos os diretórios servidos ao público, exceto em repositórios públicos de espelhamento (*mirrors*) onde essa listagem seja o requisito funcional explícito.
- **[RECOMENDAÇÃO]** Substituir nomes de arquivos na URL por identificadores indiretos (como um **UUID** — *Universally Unique Identifier* ou Identificador Único Universal, ou um ID numérico mapeado em banco de dados ou dicionário estático).
- **[BOA PRÁTICA]** Executar o processo da aplicação web com um usuário de sistema operacional de privilégio mínimo (nunca como `root`), restrito apenas à leitura da pasta específica de arquivos públicos.

Veja o código corrigido completo em Node.js/Express e a configuração segura do Nginx:

```javascript
// ARQUIVO CORRIGIDO: src/routes/manuais.js
const express = require('express');
const fs = require('fs');
const path = require('path');
const router = express.Router();

const DIRETORIO_MANUAIS = path.resolve('/var/www/portal/public/manuais');

// Lista de permissão (Allowlist) para categorias válidas
const CATEGORIAS_PERMITIDAS = new Set(['graduacao', 'pos-graduacao', 'extensao']);

// Expressão regular permitindo apenas letras, números, hífen, sublinhado e extensão .txt ou .pdf
const REGEX_NOME_ARQUIVO = /^[a-zA-Z0-9_-]+\.(txt|pdf)$/;

router.get('/api/manual', (req, res) => {
  const categoria = String(req.query.categoria || '');
  const arquivo = String(req.query.arquivo || '');

  // 1. Validação por allowlist da categoria
  if (!CATEGORIAS_PERMITIDAS.has(categoria)) {
    return res.status(400).json({ erro: 'Categoria inválida.' });
  }

  // 2. Extração apenas do nome base (remove qualquer ../ ou subdiretório) e validação de formato
  const nomeSeguro = path.basename(arquivo);
  if (nomeSeguro !== arquivo || !REGEX_NOME_ARQUIVO.test(nomeSeguro)) {
    return res.status(400).json({ erro: 'Nome de arquivo inválido.' });
  }

  // 3. Resolução canônica e verificação estrita de prefixo (Fronteira de Diretório)
  const diretorioCategoria = path.resolve(DIRETORIO_MANUAIS, categoria);
  const caminhoResolvido = path.resolve(diretorioCategoria, nomeSeguro);

  if (!caminhoResolvido.startsWith(diretorioCategoria + path.sep)) {
    return res.status(403).json({ erro: 'Acesso negado fora do diretório permitido.' });
  }

  // 4. Verificação de existência sem vazar o caminho físico do servidor na mensagem de erro
  if (!fs.existsSync(caminhoResolvido)) {
    return res.status(404).json({ erro: 'Manual solicitado não encontrado.' });
  }

  // 5. Proteção contra links simbólicos (symlinks) apontando para fora da pasta
  const caminhoReal = fs.realpathSync(caminhoResolvido);
  if (!caminhoReal.startsWith(diretorioCategoria + path.sep)) {
    return res.status(403).json({ erro: 'Acesso negado.' });
  }

  res.sendFile(caminhoReal);
});

module.exports = router;
```

E a configuração corrigida do servidor Nginx:

```nginx
# CONFIGURAÇÃO SEGURA DO NGINX
server {
    listen 443 ssl;
    server_name portal.exemplo.test;
    root /var/www/portal/public;

    # Desativa listagem de diretórios globalmente no bloco server
    autoindex off;

    # Bloqueia acesso a arquivos ocultos (iniciados por ponto, como .env, .git, .htaccess)
    location ~ /\. {
        deny all;
        return 404;
    }

    location /manuais/ {
        autoindex off;
    }
}
```

##### 8. Como testar se a correção funcionou

1. **Teste de caminhos legítimos (Não regressão)**: envie `GET /api/manual?categoria=graduacao&arquivo=regulamento.txt` e confirme que a resposta é `200 OK` com o documento esperado.
2. **Teste de sequências de travessia**: envie `GET /api/manual?categoria=graduacao&arquivo=../../.env` e `GET /api/manual?categoria=../../&arquivo=etc/passwd`. Verifique que ambas retornam `400 Bad Request` com mensagem genérica, sem expor caminhos internos do servidor.
3. **Teste de *symlink* no laboratório**: crie um link simbólico dentro da pasta de teste apontando para `/etc/hosts` e tente requisitá-lo; confirme que a verificação `fs.realpathSync()` bloqueia a leitura com `403 Forbidden`.
4. **Teste de Directory Listing**: acesse `https://portal.exemplo.test/manuais/` e `https://portal.exemplo.test/.env` pelo navegador ou via `curl -i` e verifique que o servidor retorna `403 Forbidden` ou `404 Not Found`, nunca a listagem `Index of /`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Na coluna **Name** ou **Path** e na subaba **Payload** (*Query String Parameters*), observam-se parâmetros como `file=`, `path=`, `folder=`, `template=` ou `doc=` carregando caminhos de pastas (`relatorios/2026/arquivo.pdf`) ou sequências `../`.
  - Na subaba **Response** e **Preview**, quando vulnerável a *Directory Listing*, uma requisição a um diretório (`/uploads/`, `/assets/backups/`) retorna status `200 OK` com `Content-Type: text/html` cujo corpo exibe `<title>Index of /uploads</title>` e hiperlinks para arquivos internos.
  - Quando vulnerável a *Directory Traversal* ou vazamento de caminho (*Full Path Disclosure*), respostas de erro (`404` ou `500`) na subaba **Response** exibem mensagens como `ENOENT: no such file or directory, open '/var/www/portal/public/manuais/...'`, revelando o sistema operacional e a árvore exata de diretórios do servidor.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Em **Cookies** ou **Local Storage**, algumas aplicações legadas guardam preferências de idioma ou tema usando caminhos de arquivo (ex.: cookie `user_theme=../../etc/passwd` ou `lang=pt-br/main.php`), que o servidor lê sem validar.
  - Em **Frames** (seção inferior da barra lateral da aba Application), é possível visualizar a árvore de documentos e recursos carregados na página e identificar diretórios estáticos abertos ou arquivos de configuração indevidamente indexados.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Funções JavaScript do front-end que montam dinamicamente URLs de download (ex.: `fetch('/api/manual?arquivo=' + inputUsuario)`) ou mensagens de erro de `fetch()` não tratadas que imprimem no Console o JSON de erro vindo do back-end contendo o caminho físico completo do servidor.

---

<a id="s3-11"></a>
### 3.11 File Upload Vulnerabilities (Vulnerabilidades de Upload de Arquivos)

##### 1. Definição

As **File Upload Vulnerabilities** (Vulnerabilidades de Upload de Arquivos — associadas principalmente ao **CWE-434**: *Unrestricted Upload of File with Dangerous Type*, ou Upload Irrestrito de Arquivo com Tipo Perigoso) ocorrem quando uma aplicação web permite que o usuário envie arquivos para o servidor de armazenamento sem validar rigorosamente o seu nome, tipo real, conteúdo, dimensões, tamanho ou modo de execução/entrega posterior.

Dependendo de como o arquivo enviado é processado e de onde ele é armazenado, uma falha de upload pode levar desde um ataque de **Cross-Site Scripting Armazenado** (**Stored XSS** — execução de scripts maliciosos no navegador de outros usuários, **CWE-79**), passando por **Negação de Serviço** (**DoS** — *Denial of Service*, por exaustão de disco ou memória, **CWE-400**), sobrescrita de arquivos críticos do sistema via *Path Traversal* no nome do arquivo, até o impacto máximo de **Execução Remota de Código** (**RCE** — *Remote Code Execution*) no próprio servidor web por meio do envio de um *web shell* (script executável no servidor). No **OWASP Top 10**, essas falhas se enquadram principalmente em **A04:2021 / A06:2025 (Insecure Design)**, **A03:2021 / A05:2025 (Injection)** e **A05:2021 / A02:2025 (Security Misconfiguration)**.

##### 2. Como funciona

O envio de arquivos na web utiliza tradicionalmente requisições `POST` ou `PUT` com o cabeçalho `Content-Type: multipart/form-data`. Nessa codificação, o corpo da requisição HTTP é dividido em partes separadas por um delimitador (*boundary*), e cada arquivo enviado inclui metadados controlados inteiramente pelo cliente:

```http
POST /api/perfil/avatar HTTP/1.1
Host: loja-ficticia.test
Content-Type: multipart/form-data; boundary=----FormBoundary7MA4YWxkTrZu0gW

------FormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="foto.png"
Content-Type: image/png

<bytes binários do arquivo>
------FormBoundary7MA4YWxkTrZu0gW--
```

Se a aplicação confiar cegamente no cabeçalho `Content-Type: image/png` enviado pelo navegador ou apenas verificar se a string `filename` contém `.png`, diversos vetores de exploração tornam-se possíveis:

1. **Execução no Servidor (*Web Shell* / RCE)**: em servidores que interpretam scripts diretamente a partir do diretório de arquivos públicos (como PHP, ASP.NET clássico ou JSP), o atacante envia um arquivo `shell.php` (ou `avatar.png.php`) contendo código do lado do servidor, mas declara `Content-Type: image/png` na requisição multipart. Se o arquivo for salvo em `/var/www/html/uploads/shell.php` e esse diretório tiver permissão de execução de scripts, basta acessar `https://loja-ficticia.test/uploads/shell.php` para que o servidor execute o código.
2. **Execução no Cliente (Stored XSS via SVG ou HTML)**: mesmo que o servidor seja em Node.js, Python ou Go (que não executam arquivos `.php` da pasta `uploads`), se a aplicação aceitar imagens **SVG** (*Scalable Vector Graphics* — Gráficos Vetoriais Escaláveis, que são documentos baseados em **XML** — *Extensible Markup Language*) ou arquivos `.html` e servi-los na mesma origem da aplicação com `Content-Type: image/svg+xml` ou `text/html` (ou sem o header `X-Content-Type-Options: nosniff`), qualquer `<script>` embutido no SVG será executado no contexto da sessão de quem visualizar o arquivo diretamente.
3. **Sobrescrita de arquivos (*Path Traversal* no `filename`)**: se o campo `filename` na parte multipart for enviado como `filename="../../.ssh/authorized_keys"` ou `filename="../views/login.ejs"`, e o servidor salvar o arquivo concatenando o `originalname` diretamente no caminho de destino, arquivos vitais do servidor serão sobrescritos.
4. **Ataques a analisadores (*Parsers*) e Negação de Serviço**: envio de arquivos **XML/SVG/DOCX** contendo entidades externas (**XXE** — *XML External Entity*), imagens com dimensões de cabeçalho adulteradas (*Pixel Flood* / *Decompression Bomb*, como um PNG de 40 KB que se expande para 20 GB em memória **RAM** — *Random Access Memory* ao gerar miniatura), ou arquivos de vários gigabytes para esgotar o disco.

##### 3. Por que acontece

As vulnerabilidades de upload ocorrem porque desenvolvedores frequentemente confundem **metadados declarados pelo cliente** com **propriedades reais do arquivo**:

- **Validação apenas no front-end**: usar `<input type="file" accept=".jpg,.png">` ou validar a extensão via JavaScript antes do envio. Um usuário malicioso intercepta a requisição ou usa `curl` / `fetch()` no Console para ignorar o front-end.
- **Confiança no `Content-Type` da parte multipart**: o cabeçalho `Content-Type: image/jpeg` dentro do `multipart/form-data` é definido pelo cliente e pode ser alterado trivialmente para qualquer valor.
- **Validação frágil de extensão (Lista de bloqueio / *Blocklist* ou Regex defeituosa)**: bloquear apenas `.php` esquecendo `.phtml`, `.phar`, `.php5`, `.shtml`, `.svg`, `.html`, ou usar `filename.includes('.png')` (que aprova `malicioso.png.php`).
- **Preservação do nome original do arquivo**: salvar o arquivo no disco usando o `originalname` enviado pelo cliente, abrindo brecha para caracteres especiais, dupla extensão e *Path Traversal*.
- **Servir uploads na mesma origem e com permissão de execução**: hospedar arquivos enviados por usuários no mesmo domínio principal (`loja-ficticia.test`) onde residem os cookies de sessão e permitir que o servidor web interprete scripts naquela pasta.

##### 4. Exemplo didático de código vulnerável

Abaixo está um endpoint de upload de comprovantes e avatares em **Node.js com Express e Multer**, representativo de falhas comuns encontradas em laboratórios de segurança:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/upload.js (Ambiente de Laboratório)
const express = require('express');
const multer = require('multer');
const path = require('path');
const router = express.Router();

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    // VULNERABILIDADE 1: salva diretamente dentro da pasta pública da mesma origem
    cb(null, '/var/www/loja-ficticia/public/uploads');
  },
  filename: (req, file, cb) => {
    // VULNERABILIDADE 2: usa o nome original enviado pelo usuário sem sanitizar
    cb(null, file.originalname);
  }
});

const upload = multer({
  storage: storage,
  // VULNERABILIDADE 3: sem limite de tamanho (limits: { fileSize: ... }) -> Risco de DoS!
  fileFilter: (req, file, cb) => {
    // VULNERABILIDADE 4: confia apenas no MIME type declarado pelo cliente no header HTTP
    // e permite image/svg+xml, que pode conter JavaScript executável!
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('Apenas imagens são permitidas!'), false);
    }
  }
});

router.post('/api/upload-avatar', upload.single('avatar'), (req, res) => {
  // Retorna a URL pública onde o arquivo ficou disponível imediatamente
  res.json({
    sucesso: true,
    url: `/uploads/${req.file.originalname}`
  });
});

module.exports = router;
```

##### 5. Como identificar

Para identificar vulnerabilidades de upload durante uma auditoria:

- Localize todos os formulários e endpoints que aceitam `multipart/form-data`, envio binário (`application/octet-stream`) ou imagens codificadas em Base64 dentro de payloads JSON.
- Verifique na resposta HTTP ou no DOM onde o arquivo enviado fica hospedado: se a URL gerada mantém o nome original escolhido pelo usuário (ex.: `/uploads/meu_arquivo.png`), há forte indício de que o servidor não renomeia o arquivo com um identificador seguro.
- Inspecione os headers HTTP retornados quando o arquivo enviado é baixado ou visualizado: verifique se faltam `X-Content-Type-Options: nosniff`, `Content-Disposition: attachment` e `Content-Security-Policy: sandbox`.
- No código-fonte, verifique se há limite de tamanho (`fileSize`), verificação de assinatura binária (*Magic Bytes* / *file signature*) ou recodificação da imagem (como reprocessamento via biblioteca `sharp` ou `Pillow`).

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório autorizado (como **OWASP Juice Shop**, **DVWA** ou **PortSwigger Web Security Academy**):

1. **Envio de linha de base (*Baseline*)**: o estudante envia uma imagem PNG legítima de 1x1 pixel (`teste.png`) pelo formulário do laboratório e observa a requisição na aba **Network** do DevTools e o caminho retornado (`/uploads/teste.png`).
2. **Teste de validação de tipo via SVG contendo script didático**: como o código vulnerável aceita qualquer `file.mimetype.startsWith('image/')`, o estudante cria em seu ambiente local um arquivo `avatar_teste.svg` contendo um payload inofensivo de prova de conceito:
   ```xml
   <?xml version="1.0" standalone="no"?>
   <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
   <svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
     <rect width="100" height="100" style="fill:rgb(0,0,255);" />
     <script type="text/javascript">
       alert('XSS em Laboratorio via SVG - Origem: ' + document.domain);
     </script>
   </svg>
   ```
3. **Execução do teste**: o estudante faz o upload de `avatar_teste.svg` e navega diretamente para `http://localhost:3000/uploads/avatar_teste.svg`. Quando o navegador renderiza o SVG diretamente na mesma origem da aplicação sem CSP restritiva, a caixa de alerta é exibida, comprovando que qualquer script ali presente teria acesso ao `localStorage`, `sessionStorage` e aos cookies não-`HttpOnly` do domínio.
4. **Teste de falsificação de `Content-Type` (em laboratórios PHP como DVWA)**: o estudante envia um arquivo de teste contendo `<?php echo "Teste de execucao de laboratorio: " . phpversion(); ?>` declarando `Content-Type: image/jpeg` no corpo multipart. Se o servidor salvar o arquivo com extensão `.php` e executá-lo ao ser acessado via navegador, comprova-se a falha de execução remota no laboratório.

##### 7. Como corrigir

A proteção de funcionalidades de upload exige uma cadeia de controles cumulativos no servidor:

- **[REQUISITO]** Validar a extensão do arquivo contra uma **lista de permissão estrita (*allowlist*)** em letras minúsculas (ex.: apenas `.jpg`, `.jpeg`, `.png`, `.webp` — excluindo `.svg`, `.html`, `.xml` se não forem estritamente necessários e sanitizados).
- **[REQUISITO]** Nunca utilizar o nome de arquivo enviado pelo usuário (`originalname`) para salvar no sistema de arquivos. Gerar sempre um nome aleatório criptograficamente seguro (ex.: `crypto.randomUUID() + '.png'`).
- **[REQUISITO]** Validar os **Magic Bytes** (assinatura binária dos primeiros bytes do arquivo, como `89 50 4E 47 0D 0A 1A 0A` para PNG ou `FF D8 FF` para JPEG) e, para imagens, **recodificar/reprocessar** a imagem no servidor (eliminando metadados **EXIF** — *Exchangeable Image File Format* e qualquer payload oculto).
- **[REQUISITO]** Definir limites estritos de tamanho de arquivo (`fileSize`), quantidade de arquivos e dimensões máximas de pixels.
- **[RECOMENDAÇÃO]** Armazenar os arquivos fora do diretório público da aplicação (ou em um serviço de armazenamento de objetos dedicado, como Amazon **S3** — *Simple Storage Service* ou equivalente) e servi-los a partir de um **domínio separado (*sandbox domain*)** sem cookies de sessão (ex.: `arquivos-usuarios.exemplo-cdn.test` em vez de `loja-ficticia.test`).
- **[REQUISITO]** Ao servir arquivos enviados por usuários, incluir obrigatoriamente os cabeçalhos `X-Content-Type-Options: nosniff`, `Content-Security-Policy: default-src 'none'; sandbox` e, para arquivos que não precisam ser exibidos inline, `Content-Disposition: attachment; filename="arquivo.ext"`.

Veja a implementação corrigida completa utilizando Node.js, Express, Multer em memória e reprocessamento seguro com `sharp`:

```javascript
// ARQUIVO CORRIGIDO: src/routes/upload.js
const express = require('express');
const multer = require('multer');
const crypto = require('crypto');
const path = require('path');
const fs = require('fs/promises');
const sharp = require('sharp');
const router = express.Router();

// Armazenamento fora da pasta pública de scripts (fora do web root executável)
const DIRETORIO_ARMAZENAMENTO = path.resolve('/var/data/loja-ficticia/avatars');

// Processa em memória com limite rígido de 2 MB para inspecionar antes de gravar no disco
const upload = multer({
  storage: multer.memoryStorage(),
  limits: {
    fileSize: 2 * 1024 * 1024, // Máximo de 2 MB
    files: 1
  }
});

// Verificação explícita de Magic Bytes para PNG e JPEG
function validarMagicBytes(buffer) {
  if (!buffer || buffer.length < 8) return null;
  // Assinatura PNG: 89 50 4E 47 0D 0A 1A 0A
  const ehPng = buffer[0] === 0x89 && buffer[1] === 0x50 && buffer[2] === 0x4E && buffer[3] === 0x47;
  // Assinatura JPEG: FF D8 FF
  const ehJpeg = buffer[0] === 0xFF && buffer[1] === 0xD8 && buffer[2] === 0xFF;
  if (ehPng) return 'png';
  if (ehJpeg) return 'jpeg';
  return null;
}

router.post('/api/upload-avatar', upload.single('avatar'), async (req, res) => {
  try {
    if (!req.file || !req.file.buffer) {
      return res.status(400).json({ erro: 'Nenhum arquivo enviado.' });
    }

    // 1. Verifica os Magic Bytes reais do buffer (ignora Content-Type e filename do cliente)
    const formatoDetectado = validarMagicBytes(req.file.buffer);
    if (!formatoDetectado) {
      return res.status(415).json({ erro: 'Formato de arquivo não suportado. Envie PNG ou JPEG.' });
    }

    // 2. Gera nome 100% aleatório com UUID (elimina Path Traversal e dupla extensão)
    const idArquivo = crypto.randomUUID();
    const nomeArquivoFinal = `${idArquivo}.webp`;
    const caminhoFinal = path.resolve(DIRETORIO_ARMAZENAMENTO, nomeArquivoFinal);

    // 3. Recodifica a imagem com limite de dimensões (previne Pixel Flood e remove payloads/EXIF)
    await sharp(req.file.buffer, { limitInputPixels: 4096 * 4096 })
      .resize(512, 512, { fit: 'cover' })
      .webp({ quality: 85 })
      .toFile(caminhoFinal);

    return res.status(201).json({
      sucesso: true,
      avatarId: idArquivo
    });
  } catch (err) {
    return res.status(400).json({ erro: 'Não foi possível processar a imagem enviada.' });
  }
});

// Endpoint dedicado para servir o avatar com headers de isolamento de segurança
router.get('/api/avatar/:id', async (req, res) => {
  const id = String(req.params.id || '');
  if (!/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i.test(id)) {
    return res.status(400).end();
  }

  const caminho = path.resolve(DIRETORIO_ARMAZENAMENTO, `${id}.webp`);
  res.setHeader('Content-Type', 'image/webp');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('Content-Security-Policy', "default-src 'none'; sandbox");
  res.setHeader('Cross-Origin-Resource-Policy', 'same-origin');
  return res.sendFile(caminho);
});

module.exports = router;
```

##### 8. Como testar se a correção funcionou

1. **Teste de arquivo legítimo**: envie uma imagem PNG ou JPEG válida menor que 2 MB; verifique que a resposta é `201 Created` retornando apenas um `avatarId` no formato UUID e que a imagem recodificada em `.webp` é servida sem metadados EXIF.
2. **Teste de falsificação de MIME e extensão**: envie o arquivo `avatar_teste.svg` (ou um arquivo de texto renomeado para `foto.png` com `Content-Type: image/png`). Confirme que a validação de *Magic Bytes* e o parser `sharp` rejeitam o arquivo com status `415 Unsupported Media Type` ou `400 Bad Request`.
3. **Teste de limite de tamanho**: envie um arquivo de 5 MB e confirme que o Multer interrompe a leitura imediatamente, retornando erro de limite excedido sem esgotar a memória do servidor.
4. **Inspeção de headers de entrega**: acesse `GET /api/avatar/<uuid>` e confirme a presença de `Content-Type: image/webp`, `X-Content-Type-Options: nosniff` e `Content-Security-Policy: default-src 'none'; sandbox`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Na requisição `POST` de upload, a subaba **Payload** mostra os campos do `multipart/form-data`, permitindo ver como o navegador envia `filename` e `Content-Type`.
  - Na subaba **Response** do upload, se o servidor devolver o caminho `/uploads/<mesmo_nome_enviado>`, identifica-se imediatamente que o nome original do usuário foi preservado no servidor.
  - Na requisição `GET` subsequente que carrega o arquivo enviado, a subaba **Headers** (*Response Headers*) permite auditar se faltam `X-Content-Type-Options: nosniff`, `Content-Security-Policy` ou `Content-Disposition`, e se o arquivo está sendo servido a partir do mesmo domínio principal da aplicação (`Domain` na tabela de requisições).
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Se um arquivo `.svg` ou `.html` malicioso for aberto diretamente na mesma origem da aplicação, ele compartilha a mesma entrada em **Local Storage**, **Session Storage**, **IndexedDB** e **Cookies** visível na aba Application daquela origem.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Se a validação de upload existir apenas no front-end, é possível observar no Console mensagens de erro geradas por scripts do cliente antes mesmo que qualquer requisição apareça na aba Network.
  - Caso o servidor implemente corretamente uma política `Content-Security-Policy: default-src 'none'; sandbox` nos arquivos enviados, qualquer tentativa de execução de script dentro de um SVG gerará imediatamente um erro vermelho de violação de **CSP** (*Content Security Policy* — Política de Segurança de Conteúdo) no Console.

---

<a id="s3-12"></a>
### 3.12 Clickjacking (UI Redressing)

##### 1. Definição

O **Clickjacking** (Sequestro de Clique), tecnicamente denominado **UI Redressing** (Redesenho ou Camuflagem de Interface de Usuário — **UI**, *User Interface*) e catalogado sob o **CWE-1021** (*Improper Restriction of Rendered UI Layers or Frames*), é uma vulnerabilidade de segurança do lado do cliente na qual um atacante incorpora uma página web legítima e sensível dentro de um quadro embutido (`<iframe>`) transparente ou visualmente camuflado em uma página maliciosa sob seu controle. O objetivo é enganar o usuário autenticado para que ele clique em botões, links ou controles da aplicação-alvo achando que está interagindo apenas com a página visível do atacante.

No **OWASP Top 10**, a ausência das proteções que impedem o enquadramento indevido de páginas críticas enquadra-se em **A05:2021 / A02:2025 (Security Misconfiguration)** e **A04:2021 / A06:2025 (Insecure Design)**. Diferentemente do **CSRF** (*Cross-Site Request Forgery* — Falsificação de Requisição entre Sites, estudado na [Seção 3.3](./03a-vulnerabilidades-parte1.md#s3)), em que uma requisição é disparada silenciosamente por código da página atacante (e que pode ser bloqueado por tokens anti-CSRF), no **Clickjacking** o usuário está interagindo diretamente com a página real da aplicação legítima carregada dentro do `<iframe>`. Portanto, **tokens anti-CSRF não impedem ataques de Clickjacking**, pois o próprio formulário legítimo carregado dentro do `<iframe>` já contém o token CSRF válido da sessão do usuário!

##### 2. Como funciona

A mecânica do ataque baseia-se nas propriedades de posicionamento e transparência das folhas de estilo **CSS** (*Cascading Style Sheets* — Folhas de Estilo em Cascata), especificamente `position: absolute`, `z-index` e `opacity`:

1. O usuário já está autenticado no site legítimo `https://banco-ficticio.test` (ou em um painel administrativo `https://painel.loja-ficticia.test`).
2. O usuário visita uma página de isca (`https://sorteio-malicioso.test`) que exibe um botão chamativo: *"Clique aqui para resgatar seu cupom de R$ 100"*.
3. Posicionado exatamente sobre esse botão (`z-index: 2`), a página de isca carrega um `<iframe src="https://painel.loja-ficticia.test/configuracoes/excluir-conta">` com `opacity: 0.0001` (invisível ao olho humano, mas totalmente clicável pelo navegador) e deslocado com coordenadas CSS (`top`, `left`) de modo que o botão *"Confirmar Exclusão da Conta"* do `<iframe>` fique exatamente embaixo do cursor do mouse quando o usuário aponta para o botão do cupom.
4. Ao clicar, o evento de clique é entregue pelo navegador ao documento dentro do `<iframe>` superior. Se os cookies de sessão puderem ser enviados naquele contexto (por exemplo, se estiverem com `SameSite=None; Secure`, ou se a página embutida realizar uma ação que não dependa de cookies bloqueados, como autorizar permissões OAuth/WebRTC ou copiar tokens em fluxos interativos), a ação sensível é executada legitimamente na origem-alvo.

```text
+-------------------------------------------------------------------+
| PÁGINA VISÍVEL (https://sorteio-malicioso.test)    [z-index: 1]   |
|                                                                   |
|     "Parabéns! Você ganhou um brinde!"                            |
|                                                                   |
|     +-------------------------------------------------------+     |
|     | [ Botão Visível: RESGATAR PRÊMIO AGORA ]              |     |
|     |   ^                                                   |     |
|     |   | Sobreposição exata com opacity: 0 (Invisível)     |     |
|     |   v                                                   |     |
|     | [ Botão no Iframe: CONFIRMAR EXCLUSÃO DE CONTA ]      |     |
|     | (<iframe src="https://painel.loja-ficticia.test/...">)|     |
|     +-------------------------------------------------------+     |
|                                                    [z-index: 2]   |
+-------------------------------------------------------------------+
```

Variantes modernas incluem:
- **Likejacking**: induzir curtidas, compartilhamentos ou inscrições em redes sociais.
- **Cursorjacking**: ocultar ou deslocar visualmente a imagem do cursor via CSS para enganar a percepção espacial do usuário.
- **Drag-and-Drop Clickjacking**: induzir o usuário a arrastar elementos (como em um jogo falso) que na verdade copiam e soltam tokens ou dados sensíveis entre frames.

##### 3. Por que acontece

O Clickjacking acontece porque, por padrão histórico da arquitetura da Web, qualquer documento HTML pode incorporar qualquer outra URL em um elemento `<iframe>`, a menos que o servidor da página incorporada instrua explicitamente o navegador a proibir essa renderização em quadro por meio dos cabeçalhos de resposta HTTP:
- Diretiva **`frame-ancestors`** do cabeçalho **`Content-Security-Policy`** (**CSP**); e/ou
- Cabeçalho legado **`X-Frame-Options`** (`DENY` ou `SAMEORIGIN`).

Além disso, desenvolvedores às vezes tentam impedir o enquadramento usando apenas scripts JavaScript no cliente (*frame-busting scripts*, como `if (top !== self) top.location = self.location;`), os quais podem ser facilmente neutralizados pelo atacante usando o atributo HTML5 `<iframe sandbox="allow-forms allow-scripts">` (que permite scripts e formulários dentro do iframe, mas bloqueia a navegação do `top` frame por não incluir `allow-top-navigation`).

##### 4. Exemplo didático de código vulnerável

Veja um endpoint em **Node.js/Express** de um painel administrativo que renderiza uma página de confirmação crítica (desativação do segundo fator de autenticação — **2FA**, *Two-Factor Authentication*) sem enviar nenhum cabeçalho de restrição de enquadramento:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/seguranca.js (Ambiente de Laboratório)
const express = require('express');
const router = express.Router();

// VULNERABILIDADE: A resposta HTML não inclui Content-Security-Policy: frame-ancestors
// nem X-Frame-Options. Qualquer site externo pode embutir esta página em um <iframe>!
router.get('/conta/desativar-2fa', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html lang="pt-BR">
    <head><title>Segurança da Conta</title></head>
    <body>
      <h2>Desativar Autenticação em Duas Etapas (2FA)</h2>
      <form method="POST" action="/conta/desativar-2fa">
        <input type="hidden" name="csrfToken" value="${req.session.csrfToken}">
        <button id="btn-confirmar" type="submit" style="width:220px;height:50px;">
          Desativar 2FA Agora
        </button>
      </form>
    </body>
    </html>
  `);
});

module.exports = router;
```

##### 5. Como identificar

1. **Inspeção de Headers HTTP**: verifique as respostas das páginas HTML da aplicação (especialmente páginas de login, autorização OAuth, alteração de configurações, transferências e confirmações). Se não houver `Content-Security-Policy` contendo a diretiva `frame-ancestors` (`'none'` ou `'self'`) nem `X-Frame-Options` (`DENY` ou `SAMEORIGIN`), a página é passível de enquadramento.
2. **Avaliação de Cookies e Estado**: verifique como os cookies de sessão estão configurados (`SameSite=Lax` ou `Strict` impedem o envio de cookies em iframes *cross-site*, mitigando parte dos cenários autenticados, mas páginas que usam `SameSite=None; Secure` ou que não dependem de cookies para a ação visada continuam totalmente expostas).
3. **Verificação de *Frame-Busters* em JS**: procure no código front-end por verificações `window.top !== window.self` usadas como única defesa sem os cabeçalhos HTTP correspondentes.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório autorizado (como **PortSwigger Web Security Academy** ou container local):

1. **Criação da página de teste (*Proof of Concept* — PoC)**: o estudante cria um arquivo HTML local (`teste_clickjacking.html`) apontando o `<iframe>` para a página de laboratório (`http://localhost:3000/conta/desativar-2fa`) com opacidade reduzida (`0.3` durante a calibração para enxergar o alinhamento):
   ```html
   <!DOCTYPE html>
   <html lang="pt-BR">
   <head>
     <meta charset="UTF-8">
     <title>Teste Didático de Clickjacking em Laboratório</title>
     <style>
       #isca {
         position: absolute;
         top: 90px;
         left: 20px;
         width: 220px;
         height: 50px;
         z-index: 1;
         background: #28a745;
         color: #fff;
         border: none;
         font-size: 16px;
       }
       #alvo-iframe {
         position: absolute;
         top: 0;
         left: 0;
         width: 600px;
         height: 300px;
         opacity: 0.25; /* Visível em 25% para demonstração em aula; no ataque real seria 0.0001 */
         z-index: 2;
         border: 2px dashed red;
       }
     </style>
   </head>
   <body>
     <button id="isca">Testar Alinhamento de Clique</button>
     <iframe id="alvo-iframe" src="http://localhost:3000/conta/desativar-2fa"></iframe>
   </body>
   </html>
   ```
2. **Observação do resultado**: se a página `http://localhost:3000/conta/desativar-2fa` for carregada normalmente dentro do `<iframe>` e o clique sobre o botão verde acionar o botão `#btn-confirmar` dentro do quadro, a vulnerabilidade está confirmada no laboratório.

##### 7. Como corrigir

A defesa contra Clickjacking é simples, determinística e deve ser aplicada no servidor via cabeçalhos de resposta HTTP (detalhados em profundidade na [Seção 5 — Headers de Segurança HTTP](./05-headers-de-seguranca.md#s5)):

- **[REQUISITO]** Enviar a diretiva `frame-ancestors` no cabeçalho **`Content-Security-Policy`**:
  - Use `Content-Security-Policy: frame-ancestors 'none';` se a página **nunca** deve ser embutida em um `<frame>`, `<iframe>`, `<embed>` ou `<object>`.
  - Use `Content-Security-Policy: frame-ancestors 'self';` se apenas páginas da **mesma origem** puderem embuti-la.
  - Use `Content-Security-Policy: frame-ancestors 'self' https://parceiro.exemplo.test;` se um domínio específico precisar embuti-la.
- **[RECOMENDAÇÃO]** Enviar conjuntamente o cabeçalho **`X-Frame-Options: DENY`** (ou `SAMEORIGIN`) para garantir defesa em profundidade e compatibilidade com clientes ou ferramentas de auditoria legadas.
- **[RECOMENDAÇÃO]** Configurar cookies de sessão com `SameSite=Lax` ou `SameSite=Strict` para que o navegador não envie cookies de autenticação em requisições de terceiros dentro de iframes cross-site.

Veja o código corrigido completo em Node.js/Express:

```javascript
// ARQUIVO CORRIGIDO: src/routes/seguranca.js
const express = require('express');
const router = express.Router();

// Middleware de proteção anti-Clickjacking para rotas da aplicação
router.use((req, res, next) => {
  // Defesa primária padrão W3C (CSP Level 2+)
  res.setHeader('Content-Security-Policy', "default-src 'self'; frame-ancestors 'none'");
  // Defesa complementar clássica (RFC 7034)
  res.setHeader('X-Frame-Options', 'DENY');
  next();
});

router.get('/conta/desativar-2fa', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html lang="pt-BR">
    <head><title>Segurança da Conta</title></head>
    <body>
      <h2>Desativar Autenticação em Duas Etapas (2FA)</h2>
      <form method="POST" action="/conta/desativar-2fa">
        <input type="hidden" name="csrfToken" value="${req.session.csrfToken}">
        <button id="btn-confirmar" type="submit">Desativar 2FA Agora</button>
      </form>
    </body>
    </html>
  `);
});

module.exports = router;
```

##### 8. Como testar se a correção funcionou

1. **Verificação de Headers**: faça uma requisição `GET /conta/desativar-2fa` e confirme na resposta a presença simultânea de `Content-Security-Policy: ... frame-ancestors 'none'` e `X-Frame-Options: DENY`.
2. **Reexecução do PoC em laboratório**: abra novamente o arquivo `teste_clickjacking.html` no navegador. O navegador recusará a renderização do documento dentro do `<iframe>`, exibindo uma página de erro interna do Chrome (*"localhost recusou a conexão"*) e registrando o bloqueio no Console.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Ao filtrar por **Doc** (Documentos HTML) e clicar na requisição principal da página, na subaba **Headers** -> **Response Headers**, nota-se a ausência de `X-Frame-Options` e a ausência da diretiva `frame-ancestors` dentro de `Content-Security-Policy`.
  - Quando o bloqueio funciona em um teste com `<iframe>`, a requisição do documento dentro do iframe aparece na aba Network com status de bloqueio ou alerta de política de segurança.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Na seção **Frames** (no fim da barra lateral esquerda da aba Application), ao selecionar o frame principal (`top`), o painel **Security & Isolation** informa a política de incorporação e se há diretivas de segurança aplicadas ao documento.
  - Em **Storage -> Cookies**, é possível verificar se os cookies de sessão estão com `SameSite=None` (o que agrava drasticamente o impacto do Clickjacking por permitir que o iframe seja carregado já autenticado em contexto cross-site).
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Quando uma página protegida é colocada dentro de um `<iframe>` não autorizado, o Chrome registra imediatamente um erro vermelho explicito no Console:
    `Refused to frame 'https://painel.loja-ficticia.test/' because an ancestor violates the following Content Security Policy directive: "frame-ancestors 'none'".` ou
    `Refused to display 'https://painel.loja-ficticia.test/' in a frame because it set 'X-Frame-Options' to 'deny'.`

---

<a id="s3-13"></a>
### 3.13 Open Redirect (Redirecionamento Aberto)

##### 1. Definição

O **Open Redirect** (Redirecionamento Aberto ou Redirecionamento Não Validado — catalogado no **CWE-601**: *URL Redirection to Untrusted Site*) é uma vulnerabilidade na qual uma aplicação web aceita uma entrada controlada pelo usuário (geralmente em um parâmetro de query string como `?redirect=`, `?next=`, `?returnUrl=`, `?url=` ou `?continue=`) e a utiliza para redirecionar o navegador para uma **URL externa arbitrária**, sem validar se o destino pertence ao próprio domínio ou a uma lista estrita de domínios confiáveis.

No **OWASP Top 10**, o Open Redirect relaciona-se diretamente a **A01:2021 / A01:2025 (Broken Access Control)**. Embora isoladamente seja por vezes classificado como de severidade baixa ou média, na prática ele funciona como um **multiplicador crítico de impacto** em três cenários reais:
1. **Campanhas de Phishing de Alta Credibilidade**: a URL inicial clicada pela vítima começa com o domínio oficial e certificado TLS verdadeiro da instituição (`https://login.banco-ficticio.test/auth?continue=https://banco-falso.test`), enganando usuários atentos e filtros de e-mail corporativos.
2. **Roubo de Códigos e Tokens em Fluxos OAuth 2.0 / OIDC (*OpenID Connect*)**: se o endpoint de autorização ou a página de `redirect_uri` tiver um Open Redirect, o código de autorização (`?code=...`) ou o token no fragmento (`#access_token=...`) pode ser vazado para o servidor do atacante via cabeçalho `Referer` ou preservação de fragmento de URL pelo navegador durante redirecionamentos `3xx`.
3. **Contorno (*Bypass*) de Filtros de SSRF e CSP**: usado como trampolim para fazer um servidor interno (em ataques de **SSRF** — *Server-Side Request Forgery*) ou um script restrito por CSP acessar recursos externos confiando na origem inicial.

##### 2. Como funciona

É muito comum que aplicações redirecionem o usuário de volta para a página que ele tentava acessar antes de fazer login:

```text
Fluxo normal pretendido:
1. Usuário acessa: https://loja-ficticia.test/checkout
2. Sistema redireciona para: https://loja-ficticia.test/login?next=/checkout
3. Após autenticar com sucesso, o servidor lê "next" e responde:
   HTTP/1.1 302 Found
   Location: /checkout
```

O problema surge quando o servidor não restringe o parâmetro `next` a caminhos internos relativos e aceita URLs absolutas ou relativas ao protocolo:

```text
Fluxo explorado no Open Redirect:
1. Vítima recebe link: https://loja-ficticia.test/login?next=https://clone-malicioso.test/sessao-expirada
2. Vítima confia no domínio "loja-ficticia.test", digita seu usuário e senha reais no formulário verdadeiro.
3. O servidor autentica a vítima e imediatamente envia:
   HTTP/1.1 302 Found
   Location: https://clone-malicioso.test/sessao-expirada
4. O navegador cai na página falsa do atacante (que exibe "Senha incorreta, digite novamente e informe seu token SMS"), e a vítima entrega a credencial sem perceber que mudou de domínio!
```

Além do cabeçalho HTTP `Location` (com códigos `301`, `302`, `303`, `307`, `308`), o Open Redirect também ocorre no **DOM** quando código JavaScript no front-end lê a URL e executa `window.location.href = params.get('next')` ou `window.location.replace(...)` — o que, se aceitar o pseudo-protocolo `javascript:alert(1)`, converte-se instantaneamente em um **DOM-based XSS**!

##### 3. Por que acontece

1. **Ausência total de validação do destino**: passar `req.query.next` diretamente para `res.redirect(req.query.next)`.
2. **Validações ingênuas com expressões regulares ou `startsWith`**:
   - Testar apenas `url.startsWith('/')` sem bloquear `//atacante.test` (**URL relativa ao protocolo** — *protocol-relative URL*, que o navegador interpreta como `https://atacante.test`) ou `/\atacante.test` (que os navegadores modernos normalizam convertendo a barra invertida `\` em `/`, resultando em `//atacante.test`!).
   - Testar apenas `url.includes('loja-ficticia.test')`, o que aprova `https://loja-ficticia.test.atacante.test` ou `https://atacante.test/?origem=loja-ficticia.test`.
   - Testar `url.startsWith('https://loja-ficticia.test')` sem verificar o fim do host, o que aprova `https://loja-ficticia.test.atacante.test`.

##### 4. Exemplo didático de código vulnerável

Observe uma rota de login e logout em **Node.js/Express** que tenta fazer uma validação superficial com `startsWith('/')`, mas continua vulnerável a bypasses clássicos de prova:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/auth.js (Ambiente de Laboratório)
const express = require('express');
const router = express.Router();

router.get('/logout', (req, res) => {
  const destino = req.query.returnUrl || '/';

  req.session.destroy(() => {
    // VULNERABILIDADE 1: se o desenvolvedor tentou validar apenas se começa com "/"
    // um payload como "//evil.exemplo.test" ou "/\evil.exemplo.test" começa com "/"
    // mas redireciona o navegador para fora do domínio!
    if (destino.startsWith('/')) {
      return res.redirect(destino);
    }

    // VULNERABILIDADE 2: verifica apenas se o domínio consta em algum lugar da string
    if (destino.includes('loja-ficticia.test')) {
      return res.redirect(destino);
    }

    return res.redirect('/');
  });
});

module.exports = router;
```

E um exemplo equivalente de **DOM-based Open Redirect** no front-end:

```javascript
// SCRIPT FRONT-END VULNERÁVEL: public/js/pos-login.js
const params = new URLSearchParams(window.location.search);
const continuarPara = params.get('continue');
if (continuarPara) {
  // VULNERABILIDADE CRÍTICA: aceita "https://externo.test" e até "javascript:alert(document.domain)"!
  window.location.href = continuarPara;
}
```

##### 5. Como identificar

- Procure na aba **Network** ou no código-fonte por parâmetros de redirecionamento: `redirect`, `redirect_to`, `redirect_uri`, `return`, `returnTo`, `returnUrl`, `next`, `url`, `target`, `dest`, `destination`, `continue`, `goto`, `out`, `view`.
- Verifique respostas HTTP com status `301`, `302`, `303`, `307` ou `308` cujo cabeçalho `Location` reflita valores enviados na requisição.
- No código JavaScript do cliente, procure por atribuições a `location`, `location.href`, `location.assign()`, `location.replace()` ou `window.open()` que utilizem dados extraídos de `location.search` ou `location.hash`.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório autorizado:

1. **Teste de URL externa direta**: o estudante acessa `http://localhost:3000/logout?returnUrl=https://example.com`. Se o servidor responder com `302 Found` e `Location: https://example.com`, o redirecionamento aberto é imediato.
2. **Teste de contorno de filtro de barra inicial (`//` e `/\`)**: quando a aplicação exige que o parâmetro comece com `/`, o estudante testa em laboratório:
   - `http://localhost:3000/logout?returnUrl=//example.com` (URL relativa ao esquema)
   - `http://localhost:3000/logout?returnUrl=/%5cexample.com` (`%5c` é a barra invertida `\` codificada em URL, normalizada pelo navegador para `/`)
3. **Teste de contorno de filtro de substring de domínio**: quando a aplicação exige que o nome do domínio apareça na string, o estudante testa:
   - `http://localhost:3000/logout?returnUrl=https://example.com?origem=loja-ficticia.test`
   - `http://localhost:3000/logout?returnUrl=https://loja-ficticia.test@example.com` (sintaxe de credencial `usuario@host` na URI, onde o host real acessado é `example.com`).

##### 7. Como corrigir

A estratégia mais segura é evitar passar URLs completas por parâmetro e, quando o redirecionamento pós-login for necessário, analisá-lo com o parser padrão **WHATWG `URL`** garantindo que a origem (`origin`) resultante seja idêntica à origem da aplicação:

- **[REQUISITO]** Analisar a URL de destino usando a classe padrão `new URL(destino, baseOrigin)` e validar estritamente se `urlAnalisada.origin === baseOrigin` e se o protocolo é `https:` (ou `http:` em desenvolvimento local), rejeitando qualquer discrepância.
- **[REQUISITO]** No redirecionamento relativo, rejeitar qualquer string que comece com `//`, `/\` ou contenha caracteres de controle/barra invertida, redirecionando apenas para `urlAnalisada.pathname + urlAnalisada.search + urlAnalisada.hash`.
- **[RECOMENDAÇÃO]** Utilizar um mapa de chaves fixas (ex.: `?next=checkout` -> `/carrinho/checkout`, `?next=perfil` -> `/minha-conta`) em vez de aceitar caminhos livres na URL.

Veja o código corrigido completo (Back-end em Node.js/Express e Front-end em JavaScript):

```javascript
// ARQUIVO CORRIGIDO (BACK-END): src/routes/auth.js
const express = require('express');
const router = express.Router();

const ORIGEM_OFICIAL = 'https://loja-ficticia.test';

function obterCaminhoInternoSeguro(inputUrl, caminhoPadrao = '/') {
  if (typeof inputUrl !== 'string' || inputUrl.trim() === '') {
    return caminhoPadrao;
  }

  // Bloqueia explicitamente barras invertidas, caracteres de controle e esquemas relativos "//"
  if (inputUrl.includes('\\') || inputUrl.startsWith('//') || /[\x00-\x1f]/.test(inputUrl)) {
    return caminhoPadrao;
  }

  try {
    // O parser WHATWG URL resolve o input contra a origem oficial
    const urlResolvida = new URL(inputUrl, ORIGEM_OFICIAL);

    // Verifica estritamente se a origem final permanece exatamente na ORIGEM_OFICIAL
    if (urlResolvida.origin !== ORIGEM_OFICIAL) {
      return caminhoPadrao;
    }

    // Retorna apenas o caminho relativo sanitizado (pathname + search + hash)
    return `${urlResolvida.pathname}${urlResolvida.search}${urlResolvida.hash}`;
  } catch {
    return caminhoPadrao;
  }
}

router.get('/logout', (req, res) => {
  const destinoSeguro = obterCaminhoInternoSeguro(req.query.returnUrl, '/');
  req.session.destroy(() => {
    return res.redirect(303, destinoSeguro);
  });
});

module.exports = router;
```

E no front-end:

```javascript
// SCRIPT FRONT-END CORRIGIDO: public/js/pos-login.js
function redirecionarSeguro(parametroContinue) {
  if (!parametroContinue || parametroContinue.includes('\\') || parametroContinue.startsWith('//')) {
    window.location.replace('/');
    return;
  }
  try {
    const destino = new URL(parametroContinue, window.location.origin);
    // Garante mesma origem (impede domínios externos e bloqueia pseudo-protocolos como javascript:)
    if (destino.origin === window.location.origin) {
      window.location.replace(destino.pathname + destino.search + destino.hash);
      return;
    }
  } catch {
    // URL malformada
  }
  window.location.replace('/');
}
```

##### 8. Como testar se a correção funcionou

1. **Caminho interno legítimo**: teste `/logout?returnUrl=/minha-conta?aba=pedidos` e confirme que o cabeçalho `Location` retornado é `/minha-conta?aba=pedidos`.
2. **Bypasses clássicos**: teste `/logout?returnUrl=https://example.com`, `/logout?returnUrl=//example.com`, `/logout?returnUrl=/\example.com`, `/logout?returnUrl=https://loja-ficticia.test.example.com` e `/logout?returnUrl=javascript:alert(1)`. Confirme que **todos** caem no fallback seguro `Location: /`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - **Importante**: como o redirecionamento faz o navegador mudar de página, é **obrigatório marcar a caixa `Preserve log`** (Preservar registro) na barra superior da aba Network antes de testar! Caso contrário, o registro da requisição `302` será apagado assim que a página de destino carregar.
  - Na tabela de requisições, observa-se uma linha com **Status `301`, `302`, `307` ou `308`** seguida de uma nova requisição cujo **Domain** mudou para um domínio externo.
  - Na coluna **Initiator**, é possível ver exatamente qual requisição anterior (ou qual linha de JavaScript, no caso de redirecionamento via DOM) disparou a navegação para o domínio externo.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Algumas Single Page Applications (**SPAs** — Aplicações de Página Única) salvam a URL de retorno em `sessionStorage.setItem('redirectAfterLogin', ...)` ou em um cookie temporário (`redirect_to=...`) antes de enviar o usuário para o provedor de identidade. Inspecionar **Session Storage** e **Cookies** revela se esse valor pode ser manipulado.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Com a opção **Preserve log** marcada no Console, é possível rastrear avisos de navegação ou usar *breakpoints* de evento (*Event Listener Breakpoints -> Load -> `beforeunload`*) para pausar a execução exatamente na linha de código JavaScript que altera `window.location.href`.

---

<a id="s3-14"></a>
### 3.14 Session Hijacking (Sequestro de Sessão)

##### 1. Definição

O **Session Hijacking** (Sequestro de Sessão — relacionado ao **CWE-384**, **CWE-522** e **CWE-614**) consiste na captura, interceptação, predição ou exfiltração do identificador de sessão ativo (*Session ID* ou token de acesso, como um **JWT** — *JSON Web Token*, **RFC 7519**) de um usuário legitimamente autenticado, permitindo que o atacante assuma a identidade da vítima perante o servidor web sem precisar conhecer sua senha ou passar pelo segundo fator de autenticação (**MFA** — *Multi-Factor Authentication*).

No **OWASP Top 10**, o sequestro de sessão enquadra-se em **A07:2021 / A07:2025 (Identification and Authentication Failures)** e, quando decorrente de transmissão sem criptografia ou cookies sem o atributo `Secure`, também em **A02:2021 / A04:2025 (Cryptographic Failures)**.

> [!IMPORTANT]
> **Diferença fundamental de prova (Session Hijacking vs. Session Fixation)**:
> - No **Session Hijacking (3.14)**, a vítima autentica-se primeiro, o servidor gera uma sessão válida para ela e, **depois**, o atacante **rouba ou descobre** esse token já autenticado (via XSS, interceptação de rede, vazamento em URL/logs ou baixa entropia).
> - No **Session Fixation ([3.15](#s3-15))**, o atacante **escolhe ou obtém previamente** um identificador de sessão **antes** do login, força o navegador da vítima a usar esse identificador específico e espera a vítima se autenticar nele (quando o servidor falha em regenerar o ID de sessão no momento do login).

##### 2. Como funciona

Como o protocolo HTTP é inerentemente *stateless* (sem estado), após o login o servidor identifica o usuário em cada requisição subsequente exclusivamente pelo envio do identificador de sessão (geralmente em um cookie `Cookie: session_id=...` ou no cabeçalho `Authorization: Bearer ...`). Se um atacante obtiver esse identificador enquanto ele estiver válido no servidor, basta incluí-lo em suas próprias requisições HTTP para personificar a vítima instantaneamente.

Os quatro vetores clássicos pelos quais ocorre o **Session Hijacking** são:

1. **Exfiltração via Cross-Site Scripting (XSS)**: se o cookie de sessão foi configurado **sem a flag `HttpOnly`** (ou se o token JWT foi salvo no `localStorage` ou `sessionStorage`), qualquer script injetado via XSS pode ler `document.cookie` ou `localStorage.getItem('token')` e enviá-lo para um servidor externo via `fetch()`.
2. **Interceptação em Rede (*Network Sniffing* / *Man-in-the-Middle* — MitM)**: se a aplicação trafega sobre HTTP sem criptografia TLS, ou se o cookie foi emitido **sem a flag `Secure`** (permitindo que uma única requisição HTTP em texto plano para o mesmo domínio faça o navegador enviar o cookie sem criptografia), um atacante na mesma rede Wi-Fi ou no caminho de roteamento captura o cookie em trânsito.
3. **Exposição de Token na URL, Cabeçalho `Referer` ou Logs**: quando o identificador de sessão é passado na query string (`?sid=abc123...` ou `;jsessionid=...`), ele fica gravado no histórico do navegador, nos logs de proxies corporativos e servidores web, e vaza no cabeçalho `Referer` quando o usuário clica em um link externo.
4. **Predição ou Força Bruta de ID de Sessão (Baixa Entropia — CWE-330)**: se o servidor gera IDs de sessão sequenciais, baseados em *timestamp* (`Date.now()`) ou com geradores pseudoaleatórios não criptográficos (`Math.random()`), o atacante calcula os IDs das sessões ativas de outros usuários.

##### 3. Por que acontece

- Omissão das flags de segurança **`HttpOnly`**, **`Secure`** e **`SameSite`** na criação do cookie de sessão (`Set-Cookie`), ou ausência dos prefixos de proteção **`__Host-`** / **`__Secure-`** (**RFC 6265bis**).
- Armazenamento de tokens de sessão de longa duração em `localStorage` ou `sessionStorage`, onde ficam totalmente expostos ao DOM e a qualquer script de terceiros comprometido.
- Uso de geradores de números pseudoaleatórios previsíveis para criar identificadores de sessão (em vez de um **CSPRNG** — *Cryptographically Secure Pseudo-Random Number Generator*, com pelo menos 128 bits de entropia, conforme diretriz do **OWASP Session Management Cheat Sheet**).
- Ausência de expiração por inatividade (*idle timeout*) e expiração absoluta (*absolute timeout*), ou falha em invalidar a sessão no lado do servidor quando o usuário clica em *"Sair"* (apagando o cookie apenas no navegador, mas mantendo o token válido no banco/memória do servidor).

##### 4. Exemplo didático de código vulnerável

Veja um exemplo em **Node.js/Express** de gerenciamento manual de sessão que acumula as falhas clássicas cobradas em provas e auditorias:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/sessao-insegura.js (Ambiente de Laboratório)
const express = require('express');
const router = express.Router();

const sessoesAtivas = new Map();

router.post('/api/login', (req, res) => {
  const { email, senha } = req.body;
  const usuario = autenticarUsuarioNoBanco(email, senha);

  if (!usuario) {
    return res.status(401).json({ erro: 'Credenciais inválidas.' });
  }

  // VULNERABILIDADE 1: ID de sessão previsível baseado em timestamp + Math.random()!
  const sessionId = `${Date.now()}-${Math.floor(Math.random() * 1000)}`;
  sessoesAtivas.set(sessionId, { userId: usuario.id, role: usuario.role });

  // VULNERABILIDADE 2: Cookie emitido sem HttpOnly, sem Secure, sem SameSite e sem expiração!
  res.setHeader('Set-Cookie', `sid=${sessionId}; Path=/`);

  // VULNERABILIDADE 3: Também retorna o token no corpo para ser salvo no localStorage
  return res.json({
    mensagem: 'Login realizado',
    sid: sessionId
  });
});

router.post('/api/logout', (req, res) => {
  // VULNERABILIDADE 4: Limpa o cookie no cliente, mas NÃO invalida a sessão no servidor!
  // Se o token já tiver sido copiado, continuará funcionando indefinidamente.
  res.clearCookie('sid');
  return res.json({ mensagem: 'Logout concluído.' });
});

module.exports = router;
```

##### 5. Como identificar

1. **Inspeção da tabela de Cookies na aba Application**: verifique se o cookie que mantém a sessão autenticada possui marcas de seleção (✔) nas colunas **`HttpOnly`** e **`Secure`**, qual o valor em **`SameSite`** (`Lax` ou `Strict`) e se o valor do cookie parece curto, sequencial ou decodificável como timestamp/contador.
2. **Inspeção do Local Storage / Session Storage**: verifique se tokens de acesso (`access_token`, `jwt`, `auth_token`, `session`) estão armazenados em texto legível pelo JavaScript.
3. **Teste de invalidação no Logout**: copie o valor do cookie de sessão antes de clicar em *"Sair"*, faça o logout e, em seguida, reenvie uma requisição autenticada usando o cookie antigo copiado. Se o servidor ainda responder `200 OK`, a sessão não foi invalidada no servidor!

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório autorizado (como **DVWA** no módulo *Weak Session IDs* ou **OWASP WebGoat**):

1. **Demonstração de leitura via Console/XSS quando falta `HttpOnly`**: após fazer login na aplicação vulnerável do laboratório, o estudante abre a aba **Console** do DevTools e digita `document.cookie`. Como o cookie `sid` não possui a flag `HttpOnly`, seu valor é retornado imediatamente na string (ex.: `'sid=1730280000123-482'`), demonstrando que qualquer payload XSS poderia exfiltrá-lo.
2. **Demonstração de personificação em outro contexto (Navegador Anônimo)**: o estudante abre uma janela anônima (*Incognito*) sem fazer login, abre o DevTools na aba **Application -> Cookies**, adiciona manualmente o cookie `sid` com o valor copiado da primeira janela e recarrega a página `/minha-conta`. A janela anônima passa a exibir a conta autenticada do usuário sem jamais ter digitado senha.
3. **Demonstração de persistência pós-logout**: na janela original, o estudante clica em *"Sair"*. Em seguida, atualiza a janela anônima que ainda contém o cookie copiado: como o servidor não removeu o `sid` de `sessoesAtivas`, a sessão sequestrada permanece ativa mesmo após o logout da vítima.

##### 7. Como corrigir

A proteção robusta contra o sequestro de sessão exige controles criptográficos na geração do ID, atributos estritos no cookie e gerenciamento do ciclo de vida no servidor:

- **[REQUISITO]** Gerar identificadores de sessão exclusivamente com um gerador criptograficamente seguro (**CSPRNG**, como `crypto.randomBytes(32).toString('hex')` no Node.js ou `secrets.token_hex(32)` no Python), garantindo no mínimo **128 bits** (idealmente 256 bits) de entropia pura.
- **[REQUISITO]** Configurar todo cookie de sessão com os atributos **`HttpOnly`** (bloqueia leitura via `document.cookie`), **`Secure`** (exige transmissão exclusivamente sobre HTTPS) e **`SameSite=Lax`** ou **`SameSite=Strict`** (protege contra envio em requisições cross-site).
- **[RECOMENDAÇÃO]** Utilizar o prefixo de cookie **`__Host-`** (ex.: `__Host-sid`), que força o navegador a exigir `Secure`, `Path=/` e ausência de atributo `Domain` (impedindo que subdomínios vulneráveis leiam ou sobrescrevam o cookie).
- **[REQUISITO]** Invalidar e apagar imediatamente o registro da sessão no banco de dados / Redis do servidor no momento do logout, além de aplicar expiração por inatividade (*idle timeout*, ex.: 15 a 30 minutos) e expiração absoluta (*absolute timeout*, ex.: 8 a 12 horas).

Veja o código corrigido completo em Node.js/Express:

```javascript
// ARQUIVO CORRIGIDO: src/routes/sessao-segura.js
const express = require('express');
const crypto = require('crypto');
const router = express.Router();

const sessoesAtivas = new Map();
const IDLE_TIMEOUT_MS = 15 * 60 * 1000;     // 15 minutos de inatividade
const ABSOLUTE_TIMEOUT_MS = 8 * 60 * 60 * 1000; // 8 horas de duração máxima

function gerarSessionIdSeguro() {
  // 32 bytes = 256 bits de entropia criptográfica (CSPRNG)
  return crypto.randomBytes(32).toString('hex');
}

router.post('/api/login', (req, res) => {
  const { email, senha } = req.body;
  const usuario = autenticarUsuarioNoBanco(email, senha);

  if (!usuario) {
    return res.status(401).json({ erro: 'Credenciais inválidas.' });
  }

  const agora = Date.now();
  const sessionId = gerarSessionIdSeguro();

  sessoesAtivas.set(sessionId, {
    userId: usuario.id,
    role: usuario.role,
    criadoEm: agora,
    ultimoAcessoEm: agora
  });

  // Uso do prefixo __Host- com HttpOnly, Secure, SameSite=Lax e Max-Age
  res.cookie('__Host-sid', sessionId, {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    path: '/',
    maxAge: ABSOLUTE_TIMEOUT_MS
  });

  return res.status(200).json({ mensagem: 'Autenticado com sucesso.' });
});

router.post('/api/logout', (req, res) => {
  const sessionId = req.cookies['__Host-sid'];
  if (sessionId) {
    // DESTRÓI a sessão no lado do servidor imediatamente!
    sessoesAtivas.delete(sessionId);
  }
  res.clearCookie('__Host-sid', {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    path: '/'
  });
  return res.status(200).json({ mensagem: 'Sessão encerrada com segurança.' });
});

module.exports = router;
```

##### 8. Como testar se a correção funcionou

1. **Teste no Console**: após autenticar, execute `document.cookie` na aba **Console**. O cookie `__Host-sid` **não deve aparecer** no resultado, confirmando a eficácia do `HttpOnly`.
2. **Teste na aba Application**: verifique em **Storage -> Cookies** que o cookie `__Host-sid` possui 64 caracteres hexadecimais aleatórios e está marcado com `HttpOnly` (✔), `Secure` (✔) e `SameSite: Lax`.
3. **Teste de revogação no servidor**: copie o cookie antes do logout, acione `POST /api/logout` e tente reutilizar o cookie copiado em uma requisição subsequente. O servidor deve responder `401 Unauthorized`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Na resposta da requisição de login (`POST /api/login`), o cabeçalho `Set-Cookie` aparece sem `HttpOnly` ou sem `Secure`, ou a subaba **Response** exibe um token de sessão sendo trafegado na query string da URL.
  - Na coluna **Scheme** ou **Protocol**, requisições feitas sobre `http` (sem TLS) enviando o cabeçalho `Cookie` expõem a sessão em texto claro na rede.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Em **Storage -> Cookies**, as colunas **`HttpOnly`** e **`Secure`** aparecem em branco (desmarcadas) para o cookie de sessão, ou a coluna **`Value`** exibe um número sequencial curto ou previsível.
  - Em **Storage -> Local Storage**, observa-se um token de sessão armazenado e acessível por qualquer script em execução na página.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Ao avaliar a expressão `document.cookie` no prompt do Console, o identificador de sessão é impresso em texto claro (o que jamais ocorre quando o cookie é protegido por `HttpOnly`).

---

<a id="s3-15"></a>
### 3.15 Session Fixation (Fixação de Sessão)

##### 1. Definição

O **Session Fixation** (Fixação de Sessão — catalogado especificamente no **CWE-384**: *Session Fixation*) é uma vulnerabilidade de gerenciamento de sessão na qual a aplicação web **não invalida nem regenera o identificador de sessão existente** no momento em que um usuário passa do estado anônimo (não autenticado) para o estado autenticado (ou quando eleva seu nível de privilégio). Isso permite que um atacante forneça ou plante previamente um identificador de sessão conhecido por ele no navegador da vítima antes do login; assim que a vítima se autentica, aquele mesmo identificador "fixado" passa a estar associado à conta autenticada da vítima.

No **OWASP Top 10**, o Session Fixation pertence à categoria **A07:2021 / A07:2025 (Identification and Authentication Failures)**.

##### 2. Como funciona

Muitos frameworks criam uma sessão anônima assim que o visitante abre a loja virtual ou a tela de login pela primeira vez (para guardar o carrinho de compras ou o idioma antes do login). O ataque de Fixação de Sessão explora a manutenção desse mesmo ID após a autenticação em três etapas bem definidas:

```text
ETAPA 1 — Preparação (Obtenção ou Escolha do Session ID pelo Atacante):
  O atacante acessa https://loja-ficticia.test/login e recebe um ID anônimo válido:
  sid = "SESSAO_CONHECIDA_PELO_ATACANTE_999"
  (Ou, em sistemas que aceitam IDs arbitrários, o próprio atacante inventa o ID).

ETAPA 2 — Fixação (Implantação do ID no Navegador da Vítima):
  O atacante faz o navegador da vítima adotar esse ID antes do login, usando:
  (a) Parâmetro de URL (em sistemas que aceitam Session ID via GET/URL rewriting):
      https://loja-ficticia.test/login?sid=SESSAO_CONHECIDA_PELO_ATACANTE_999
  (b) Injeção de cookie a partir de um subdomínio vulnerável (Cookie Tossing):
      Em https://blog-antigo.loja-ficticia.test, um script define:
      document.cookie = "sid=SESSAO_CONHECIDA_PELO_ATACANTE_999; Domain=.loja-ficticia.test; Path=/"
  (c) Acesso físico breve a um terminal compartilhado (laboratório, quiosque, biblioteca)
      antes de o próximo usuário fazer login.

ETAPA 3 — Autenticação da Vítima e Assunção da Sessão:
  1. A vítima digita seu usuário e senha reais na tela de login legítima, enviando junto o
     cookie sid="SESSAO_CONHECIDA_PELO_ATACANTE_999".
  2. O servidor valida a senha e (ERRO CRÍTICO!) apenas adiciona req.session.userId = vitima.id
     ao objeto da sessão existente, SEM trocar o ID "SESSAO_CONHECIDA_PELO_ATACANTE_999".
  3. Agora, o atacante (que já conhecia esse ID desde a Etapa 1) está autenticado na conta da vítima!
```

##### 3. Por que acontece

1. **Ausência de chamada a `req.session.regenerate()` no login**: o desenvolvedor apenas atribui propriedades (`req.session.user = usuario`) à sessão pré-existente sem solicitar ao gerenciador de sessões que destrua o ID antigo e emita um novo `Set-Cookie` com um identificador recém-gerado.
2. **Aceitação de IDs de sessão arbitrários (*Permissive Session Generation*)**: servidores que, ao receberem um cookie `sid=qualquer_coisa` inexistente no banco, criam uma nova sessão usando exatamente a chave enviada pelo cliente em vez de ignorá-la e gerar uma chave própria.
3. **Aceitação de identificadores de sessão via Query String ou Form Data**: configurações legadas (como `session.use_trans_sid = 1` ou `session.use_only_cookies = 0` no PHP, ou URL rewriting `;jsessionid=` em servidores Java antigos) que permitem fixar a sessão simplesmente enviando um link para a vítima.
4. **Cookies de sessão sem o prefixo `__Host-` em arquiteturas com múltiplos subdomínios**: permitindo que um subdomínio menos seguro (`evento.loja-ficticia.test`) fixe um cookie com `Domain=.loja-ficticia.test` que será lido pelo sistema principal (`app.loja-ficticia.test`).

##### 4. Exemplo didático de código vulnerável

Observe um endpoint de login em **Node.js com `express-session`** que realiza a autenticação da senha corretamente, mas esquece de regenerar o identificador da sessão ao autenticar o usuário:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/login-fixation.js (Ambiente de Laboratório)
const express = require('express');
const router = express.Router();

router.post('/api/login', async (req, res) => {
  const { email, senha } = req.body;
  const usuario = await verificarCredenciais(email, senha);

  if (!usuario) {
    return res.status(401).json({ erro: 'Usuário ou senha incorretos.' });
  }

  // VULNERABILIDADE CRÍTICA (CWE-384 - Session Fixation):
  // A aplicação reutiliza o mesmo req.sessionID que o navegador já possuía antes do login!
  // Nenhum novo header Set-Cookie é emitido para substituir o ID anônimo anterior.
  req.session.autenticado = true;
  req.session.usuarioId = usuario.id;
  req.session.perfil = usuario.perfil;

  return res.status(200).json({
    mensagem: 'Login efetuado com sucesso.',
    perfil: usuario.perfil
  });
});

module.exports = router;
```

##### 5. Como identificar

O teste de identificação de **Session Fixation** é um dos mais diretos e objetivos em auditorias de segurança web:

1. Acesse a tela de login da aplicação ainda **sem estar autenticado** e anote o valor atual do cookie de sessão (ex.: `connect.sid = s%3Aabc123...`).
2. Se ainda não houver cookie na tela de login, defina manualmente no DevTools um cookie com o nome do cookie de sessão da aplicação ou adicione um item ao carrinho anônimo para que o servidor emita um cookie pré-login.
3. Realize o login normalmente com uma conta de teste.
4. Compare o valor do cookie de sessão **após o login** com o valor anotado **antes do login**:
   - Se o valor do cookie **permaneceu exatamente igual** após o login, a aplicação é **vulnerável a Session Fixation**!
   - Se a resposta do `POST /login` incluiu um cabeçalho `Set-Cookie` com um **novo valor** e o valor antigo deixou de funcionar, a aplicação está protegida.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório autorizado:

1. **Inspeção Pré-Login**: o estudante abre `http://localhost:3000/login`, abre o DevTools em **Application -> Cookies** e observa que o servidor atribuiu o cookie anônimo `connect.sid` com o valor `ID_ANONIMO_INICIAL` (ou define um cookie de teste).
2. **Execução do Login**: com a aba **Network** aberta (e `Preserve log` ativo), o estudante submete o formulário de login com as credenciais da conta de laboratório (`aluno@exemplo.test`).
3. **Verificação da Falha**:
   - Na aba **Network**, o estudante clica na requisição `POST /api/login` e inspeciona os **Response Headers**: nota-se que **não há** cabeçalho `Set-Cookie` emitindo um novo identificador.
   - Na aba **Application -> Cookies**, o estudante constata que o valor de `connect.sid` após o login continua sendo `ID_ANONIMO_INICIAL`, agora vinculado à conta autenticada de `aluno@exemplo.test`.

##### 7. Como corrigir

A prevenção contra Session Fixation consiste na **rotação obrigatória do identificador de sessão** em qualquer mudança de estado de autenticação ou nível de privilégio:

- **[REQUISITO]** Sempre chamar a função de regeneração de sessão do framework (como `req.session.regenerate()` no Express, `request.session.cycle_key()` no Django, `session_regenerate_id(true)` no PHP ou `HttpServletRequest.changeSessionId()` no Java/Jakarta EE) **imediatamente após** validar as credenciais do usuário e **antes** de gravar os dados da conta autenticada na sessão.
- **[REQUISITO]** Garantir que o ID de sessão antigo seja completamente removido do armazenamento do servidor no ato da regeneração.
- **[REQUISITO]** Aceitar identificadores de sessão exclusivamente via cookies HTTP (nunca via query string ou parâmetros de formulário) e ignorar qualquer ID de sessão enviado pelo cliente que não exista previamente no armazenamento do servidor.
- **[RECOMENDAÇÃO]** Utilizar o prefixo `__Host-` no nome do cookie de sessão para impedir que subdomínios vizinhos injetem (*fixem*) cookies com atributo `Domain` abrangente.

Veja o código corrigido completo em Node.js com `express-session`, preservando de forma segura apenas dados não sensíveis se necessário (como itens de um carrinho) em uma nova sessão limpa:

```javascript
// ARQUIVO CORRIGIDO: src/routes/login-seguro.js
const express = require('express');
const router = express.Router();

router.post('/api/login', async (req, res, next) => {
  try {
    const { email, senha } = req.body;
    const usuario = await verificarCredenciais(email, senha);

    if (!usuario) {
      return res.status(401).json({ erro: 'Usuário ou senha incorretos.' });
    }

    // Opcional: se havia um carrinho anônimo, copie apenas os IDs validados antes de destruir a sessão antiga
    const itensCarrinhoAnonimo = Array.isArray(req.session.carrinho) ? [...req.session.carrinho] : [];

    // CORREÇÃO ESSENCIAL CONTRA SESSION FIXATION:
    // Destrói o ID de sessão antigo no servidor e gera um ID criptograficamente novo!
    req.session.regenerate((err) => {
      if (err) return next(err);

      // Grava os dados do usuário APENAS na nova sessão recém-criada
      req.session.autenticado = true;
      req.session.usuarioId = usuario.id;
      req.session.perfil = usuario.perfil;
      req.session.carrinho = itensCarrinhoAnonimo;

      // Salva a nova sessão e envia o novo Set-Cookie ao navegador
      req.session.save((errSave) => {
        if (errSave) return next(errSave);
        return res.status(200).json({
          mensagem: 'Login efetuado com sucesso com nova sessão.',
          perfil: usuario.perfil
        });
      });
    });
  } catch (erro) {
    return next(erro);
  }
});

module.exports = router;
```

##### 8. Como testar se a correção funcionou

1. Anote o valor do cookie de sessão na página de login antes de autenticar (`ID_ANTES`).
2. Faça o login com a conta de teste e inspecione a requisição `POST /api/login` na aba **Network**: confirme que os **Response Headers** contêm um cabeçalho `Set-Cookie` atribuindo um valor completamente novo (`ID_DEPOIS` $\neq$ `ID_ANTES`).
3. Tente fazer uma requisição autenticada enviando manualmente o `ID_ANTES`: confirme que o servidor rejeita o ID antigo com `401 Unauthorized`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Ao selecionar a requisição de autenticação (`POST /api/login`), compare a subaba **Headers** -> **Request Headers** (`Cookie: sid=VALOR_A`) com **Response Headers**: em uma aplicação vulnerável a Session Fixation, **não aparece um `Set-Cookie` trocando `sid` para `VALOR_B`**, e as requisições seguintes continuam enviando `Cookie: sid=VALOR_A`.
  - Na subaba **Cookies** dentro dos detalhes da requisição `POST /api/login`, é possível comparar lado a lado **Request Cookies** e **Response Cookies** para verificar instantaneamente se houve rotação do identificador.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Em **Storage -> Cookies**, mantendo a tabela visível enquanto se executa o login (ou recarregando a visualização após o login), observa-se que a coluna **`Value`** do cookie de sessão não sofreu alteração entre o estado visitante e o estado logado.
  - Caso um subdomínio tenha injetado um cookie fixado, observa-se na tabela de Cookies **duas entradas com o mesmo `Name`** (uma com `Domain: .loja-ficticia.test` e outra com `Domain: app.loja-ficticia.test`), um sinal clássico de ataque de *Cookie Tossing* para fixação de sessão.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Em ambientes de teste onde o cookie não é `HttpOnly`, ou ao depurar scripts de subdomínios, comandos que gravam `document.cookie = "sid=..."` com atributo `domain=` podem ser rastreados nos scripts carregados na página.

---

<a id="s3-16"></a>
### 3.16 Command Injection (Injeção de Comandos do Sistema Operacional)

##### 1. Definição

O **Command Injection** (Injeção de Comandos do Sistema Operacional, também chamado de **OS Command Injection** — *Operating System Command Injection*, catalogado sob o **CWE-78**) é uma vulnerabilidade crítica na qual uma aplicação web constrói uma string de comando de sistema operacional concatenando dados controlados pelo usuário e a entrega para ser interpretada por um **shell** do sistema (como `/bin/sh`, `/bin/bash` no Linux/Unix ou `cmd.exe` / `powershell.exe` no Windows). Isso permite que o atacante injete metacaracteres separadores de comandos e execute comandos arbitrários diretamente no servidor subjacente, herdando todas as permissões de sistema do processo da aplicação web.

No **OWASP Top 10**, o Command Injection é um dos representantes mais severos da categoria **A03:2021 / A05:2025 (Injection)**, pois concede diretamente **Execução Remota de Código (RCE)** no servidor.

> [!NOTE]
> **Distinção importante para provas (Command Injection vs. Code Injection)**:
> - **OS Command Injection (CWE-78)**: o atacante injeta comandos do **sistema operacional** (ex.: `whoami`, `id`, `cat /etc/passwd`, `ping`, `nslookup`) executados por um shell (`/bin/sh` ou `cmd.exe`).
> - **Code Injection (CWE-94)**: o atacante injeta código da **própria linguagem de programação** da aplicação (ex.: JavaScript dentro de `eval()`, Python dentro de `exec()`, ou PHP dentro de `eval()`), que por sua vez pode ou não chamar o sistema operacional depois.

##### 2. Como funciona

Quando um programa em Node.js (`child_process.exec`), Python (`os.system` ou `subprocess.run(..., shell=True)`), PHP (`system`, `exec`, `shell_exec`, `passthru`, operador crase `` `...` ``) ou C (`system`) invoca um comando através de um interpretador de comandos (*shell*), o shell analisa a string inteira em busca de **metacaracteres de controle** antes de executar o programa principal.

Se uma ferramenta de diagnóstico de rede do painel administrativo executa:
```bash
ping -c 4 <DESTINO_INFORMADO_PELO_USUARIO>
```
e o usuário informa `192.0.2.1; id`, o shell `/bin/sh` interpreta o ponto e vírgula (`;`) como o fim do primeiro comando e o início de um segundo comando independente:

```text
String montada pela aplicação:
  ping -c 4 192.0.2.1; id

Interpretação pelo /bin/sh:
  Comando 1: ping -c 4 192.0.2.1
  Comando 2: id   --> Executado com as permissões do usuário do servidor web!
```

A tabela a seguir resume os principais metacaracteres e operadores de encadeamento interpretados pelos shells em ambientes Unix/Linux e Windows:

| Operador / Metacaractere | Ambientes Suportados | Comportamento no Shell |
| :--- | :--- | :--- |
| `;` (Ponto e vírgula) | Unix/Linux (`sh`, `bash`), PowerShell | Executa o segundo comando incondicionalmente após o término do primeiro (`cmd1 ; cmd2`). |
| `&` (E comercial simples) | Unix/Linux (background) e Windows `cmd.exe` (sequencial) | No `cmd.exe`, executa `cmd1` e depois `cmd2`. No Unix, envia `cmd1` para segundo plano e executa `cmd2` imediatamente. |
| `&&` (AND lógico) | Unix/Linux e Windows | Executa `cmd2` somente se `cmd1` terminar com código de saída `0` (sucesso). |
| `\|\|` (OR lógico) | Unix/Linux e Windows | Executa `cmd2` somente se `cmd1` falhar (código de saída diferente de `0`). |
| `\|` (Pipe / Tubulação) | Unix/Linux e Windows | Conecta a saída padrão (`stdout`) de `cmd1` na entrada padrão (`stdin`) de `cmd2`. |
| `` `cmd` `` (Crases / Backticks) | Unix/Linux (`sh`, `bash`) | **Substituição de comando**: executa `cmd` primeiro e substitui o resultado dentro da linha principal. |
| `$(cmd)` (Cifrão-parênteses) | Unix/Linux (`sh`, `bash`), PowerShell | **Substituição de comando** moderna (pode ser aninhada) executada antes do comando principal. |
| `\n` (`%0a` — Quebra de linha) | Unix/Linux e Windows | Atua como separador de linha de comando, iniciando um novo comando na linha seguinte. |

Além do Command Injection **In-Band** (onde a saída do comando aparece diretamente na resposta HTTP), existe o **Blind Command Injection** (Injeção de Comando Cega), em que a aplicação não retorna o `stdout` na resposta HTTP. Nesse caso, a confirmação em laboratório é feita por tempo de resposta (ex.: injetando `sleep 5` ou `ping -c 5 127.0.0.1` e observando o atraso no **TTFB** — *Time To First Byte*) ou por interação de rede fora de banda (**OAST** — *Out-of-Band Application Security Testing*).

##### 3. Por que acontece

1. **Uso de APIs de execução que invocam um shell intermediário**: funções como `child_process.exec()` no Node.js ou `subprocess.Popen(cmd, shell=True)` no Python abrem `/bin/sh -c "..."` ou `cmd.exe /d /s /c "..."` por padrão.
2. **Desconhecimento de alternativas nativas da linguagem**: invocar binários de linha de comando (`ping`, `curl`, `convert`, `zip`, `grep`, `wkhtmltopdf`) para tarefas que poderiam ser feitas com bibliotecas nativas seguras da própria linguagem.
3. **Injeção de Argumentos (*Argument Injection* — CWE-88)**: mesmo quando não se usa shell, passar entradas iniciadas por hífen (`-` ou `--`) como argumentos posicionais para um binário externo sem o delimitador de fim de opções (`--`), fazendo o binário interpretar o dado do usuário como uma flag de configuração perigosa (ex.: `--output=/var/www/public/shell.php`).

##### 4. Exemplo didático de código vulnerável

Considere um endpoint em **Node.js/Express** de um painel de gerência de rede (`https://noc.exemplo.test`) que verifica a conectividade com um servidor usando `child_process.exec`:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/diagnostico.js (Ambiente de Laboratório)
const express = require('express');
const { exec } = require('child_process');
const router = express.Router();

router.post('/api/diagnostico/ping', (req, res) => {
  const { host } = req.body;

  // VULNERABILIDADE CRÍTICA (CWE-78):
  // exec() invoca /bin/sh -c (Linux) ou cmd.exe (Windows) e interpreta metacaracteres (; | && $() ` `)
  const comando = `ping -c 2 ${host}`;

  exec(comando, { timeout: 8000 }, (erro, stdout, stderr) => {
    return res.status(200).json({
      comandoExecutado: comando,
      saida: stdout || stderr
    });
  });
});

module.exports = router;
```

##### 5. Como identificar

- **No código-fonte (SAST)**: audite todas as chamadas a `child_process.exec`, `child_process.execSync`, `os.system`, `os.popen`, `subprocess.run(..., shell=True)`, `Runtime.getRuntime().exec()`, `system()`, `shell_exec()`, `passthru()`, `popen()`.
- **Na superfície da aplicação (DAST)**: procure funcionalidades que claramente interagem com utilitários do sistema operacional: ferramentas de diagnóstico de rede (`ping`, `traceroute`, `nslookup`, `whois`), conversores de documentos ou mídia (`ffmpeg`, `ImageMagick`, `Ghostscript`, `LibreOffice`, `wkhtmltopdf`), geração de backups compactados (`tar`, `zip`) ou envio de e-mail via binário local (`sendmail`).

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um laboratório autorizado (como o módulo **Command Injection** do **DVWA** ou do **PortSwigger Web Security Academy**):

1. **Requisição normal de referência**: o estudante envia `{"host": "127.0.0.1"}` para `POST /api/diagnostico/ping` e observa o retorno normal de 2 pacotes ICMP Echo Reply na resposta JSON.
2. **Teste de encadeamento com ponto e vírgula ou `&&` (In-Band)**: o estudante envia no laboratório `{"host": "127.0.0.1 && whoami"}`. A resposta JSON traz o resultado do `ping` seguido do nome do usuário do sistema operacional na última linha (ex.: `www-data` ou `node`).
3. **Teste de confirmação por tempo (Blind Command Injection)**: simulando um cenário onde a aplicação suprime a saída de texto, o estudante envia `{"host": "127.0.0.1 && sleep 5"}` e observa na aba **Network** do DevTools que o tempo da requisição (**Waiting for server response / TTFB**) salta de ~20 ms para mais de **5.000 ms**, comprovando matematicamente que o comando `sleep 5` foi executado pelo kernel do sistema operacional.

##### 7. Como corrigir

A correção definitiva de Command Injection baseia-se em três pilares hierárquicos:

1. **[RECOMENDAÇÃO] Evitar completamente chamar comandos do sistema operacional** quando existir biblioteca nativa na linguagem (ex.: usar o módulo nativo `dns.promises.lookup()` em vez de chamar o binário `nslookup` no shell).
2. **[REQUISITO] Se for indispensável executar um binário externo, usar exclusivamente APIs parametrizadas que NÃO invocam o shell** — como **`child_process.execFile()`** ou **`child_process.spawn(..., { shell: false })`** no Node.js, e **`subprocess.run(['ping', '-c', '2', host], shell=False)`** no Python — passando os argumentos como um **array de strings separado** e incluindo o delimitador `--` antes do argumento do usuário para evitar *Argument Injection*.
3. **[REQUISITO] Validar a entrada do usuário contra uma lista de permissão estrita (*allowlist*) ou expressão regular rigorosa** (ex.: aceitar exclusivamente endereços IP válidos ou nomes de domínio alfanuméricos estritos).

Veja o código corrigido completo em Node.js/Express utilizando `execFile` (sem shell) e validação estrita de IP/Hostname:

```javascript
// ARQUIVO CORRIGIDO: src/routes/diagnostico.js
const express = require('express');
const { execFile } = require('child_process');
const net = require('net');
const router = express.Router();

// Regex estrita para hostname RFC 1123 (apenas letras, números, ponto e hífen)
const REGEX_HOSTNAME_ESTRITO = /^(?=.{1,253}$)(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,63}$/;

function ehDestinoPermitido(valor) {
  if (typeof valor !== 'string') return false;
  // Permite apenas IPv4/IPv6 sintaticamente válidos ou hostnames estritos
  return net.isIP(valor) !== 0 || REGEX_HOSTNAME_ESTRITO.test(valor);
}

router.post('/api/diagnostico/ping', (req, res) => {
  const host = String(req.body.host || '').trim();

  // 1. Validação por allowlist de formato (bloqueia qualquer metacaractere ou flag iniciada por "-")
  if (!ehDestinoPermitido(host) || host.startsWith('-')) {
    return res.status(400).json({ erro: 'Endereço de destino inválido.' });
  }

  // 2. Uso de execFile(): NÃO abre /bin/sh nem cmd.exe!
  // Os argumentos são passados diretamente ao processo via execve() como array separado.
  // O marcador "--" impede que o argumento "host" seja interpretado como uma opção do comando ping.
  execFile('/bin/ping', ['-c', '2', '-W', '2', '--', host], { timeout: 5000, shell: false }, (erro, stdout) => {
    if (erro) {
      return res.status(502).json({ erro: 'Host inacessível ou tempo limite esgotado.' });
    }
    return res.status(200).json({ saida: stdout });
  });
});

module.exports = router;
```

##### 8. Como testar se a correção funcionou

1. **Teste de entrada válida**: envie `{"host": "192.0.2.1"}` ou `{"host": "exemplo.test"}` e confirme que o comando executa dentro dos limites esperados.
2. **Teste de metacaracteres de shell**: envie `{"host": "127.0.0.1; id"}`, `{"host": "127.0.0.1 && sleep 5"}` e `{"host": "$(whoami)"}`. Confirme que todos são barrados imediatamente na validação com status `400 Bad Request` e tempo de resposta inferior a 5 ms.
3. **Teste de injeção de argumento (*Argument Injection*)**: envie `{"host": "-v"}` ou `{"host": "-i 0.1"}` e confirme que tanto o filtro `host.startsWith('-')` quanto o separador `--` impedem a interpretação como flag.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Na subaba **Payload**, identificam-se campos enviando parâmetros que sugerem argumentos de linha de comando (`host`, `ip`, `cmd`, `domain`, `options`, `format`).
  - Na subaba **Response**, em vulnerabilidades *In-Band*, a resposta contém textos típicos de saída de terminal (`stdout`/`stderr`), como banners de utilitários Linux/Windows, `uid=33(www-data)` ou erros de sintaxe do shell (`/bin/sh: 1: Syntax error: ...`).
  - Na subaba **Timing** (e na coluna **Waterfall**), em testes de *Blind Command Injection* com `sleep`, a barra verde **Waiting for server response (TTFB)** apresenta uma duração exata proporcional ao tempo injetado no comando (ex.: `5.01 s` para `sleep 5`).
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Raramente armazena artefatos diretos de Command Injection, mas pode conter em **Local Storage** ou **Cookies** preferências ou configurações de exportação que são enviadas ao servidor em segundo plano e concatenadas em comandos de geração de relatórios.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Se a resposta de erro `500 Internal Server Error` do back-end for impressa no Console pelo código front-end, mensagens como `Error: Command failed: ping -c 2 ...` expõem literalmente a linha de comando executada pelo servidor.

---

<a id="s3-17"></a>
### 3.17 Path Traversal (Travessia de Caminho e Canonicalização de Arquivos)

##### 1. Definição

O **Path Traversal** (Travessia de Caminho — registrado oficialmente no **CWE-22**: *Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')*) consiste na manipulação da resolução de caminhos de arquivos para acessar ou gravar arquivos arbitrários fora do diretório restrito pretendido pela aplicação. No **OWASP Top 10**, enquadra-se primariamente em **A01:2021 / A01:2025 (Broken Access Control)**.

**Relação e Sinonímia Técnica entre Directory Traversal (3.10) e Path Traversal (3.17)**:
Na taxonomia oficial do **MITRE CWE** e da **OWASP**, *Directory Traversal* e *Path Traversal* compartilham o mesmo identificador (**CWE-22**) e são frequentemente tratados como sinônimos em provas. Contudo, em engenharia de segurança avançada, faz-se uma distinção didática importante:
- Enquanto na subseção [3.10 Directory Traversal](#s3-10) estudamos a navegação hierárquica básica entre pastas (`../`) e a exposição de diretórios (*Directory Listing*),
- O termo **Path Traversal** abrange de forma mais ampla **todas as técnicas de manipulação e evasão na resolução e canonicalização de caminhos de arquivos**, incluindo **caminhos absolutos** (`/etc/passwd`, `C:\Windows\win.ini`), **caminhos UNC** (*Universal Naming Convention* no Windows: `\\servidor\compartilhamento`), **codificações simples e duplas de URL** (`%2e%2e%2f`, `%252e%252e%252f`), **sequências aninhadas** (`....//`), **diferenças entre separadores de sistemas operacionais** (`/` vs `\`), **truncamento por byte nulo** (`%00`, histórico) e **falhas de canonicalização** (*Canonicalization Errors*).

##### 2. Como funciona

Para impedir o `../` básico visto na subseção 3.10, muitos desenvolvedores implementam filtros ingênuos de substituição de strings (como `input.replace('../', '')`). O **Path Traversal avançado** explora as diferenças entre **como o filtro analisa a string** e **como o sistema operacional ou servidor web decodifica e canonicaliza o caminho final**.

Veja a tabela completa das técnicas de evasão e codificação cobradas em provas de Segurança da Informação:

| Técnica de Path Traversal | Exemplo de Payload | Como contorna filtros superficiais |
| :--- | :--- | :--- |
| **Caminho Absoluto Direto** | `?file=/etc/passwd` ou `?file=C:\Windows\win.ini` | Não contém `../`! Se o servidor usar `path.resolve('/base', '/etc/passwd')` (Node.js) ou `os.path.join('/base', '/etc/passwd')` (Python), o segundo argumento iniciado por `/` **descarta completamente** o diretório base `/base`! |
| **Sequências Aninhadas (Não-recursivas)** | `....//....//....//etc/passwd` ou `..././..././etc/passwd` | Se o código fizer `str.replace(/\.\.\//g, '')` apenas uma vez (não recursivamente), ao remover o `../` interno de `....//`, os caracteres restantes se juntam formando um novo `../`! |
| **Separador Invertido (Windows vs. Unix)** | `..\..\..\Windows\win.ini` ou `../..\etc/passwd` | No Windows, tanto `/` quanto `\` funcionam como separadores de diretório nas APIs Win32. Se o filtro procurar apenas por `../`, `..\` passa livremente. |
| **URL Encoding Simples** | `%2e%2e%2f%2e%2e%2fetc%2fpasswd` ou `..%2f..%2f` | `%2e` = `.` e `%2f` = `/`. Contorna filtros que inspecionam a URL bruta **antes** da etapa de decodificação de URL. |
| **Double URL Encoding (Dupla Codificação)** | `%252e%252e%252f` ou `..%255c..%255c` | `%25` é o caractere `%` codificado. Na 1ª decodificação (ex.: no proxy reverso/WAF), `%252e%252e%252f` vira `%2e%2e%2f` (não tem `../` literal). Na 2ª decodificação (no back-end), vira `../`! |
| **Codificação UTF-8 Superlonga (*Overlong UTF-8*)** `[Obsoleto]` | `%c0%ae%c0%ae/%c0%af` | Historicamente explorado em servidores antigos (como IIS 4/5 e parsers Java antigos) que aceitavam representações UTF-8 não-canônicas de 2 bytes para caracteres ASCII de 1 byte. |
| **Null Byte Injection (`%00`)** `[Obsoleto]` | `../../etc/passwd%00.png` | Em versões antigas do PHP (< 5.3.4) e ambientes onde strings da linguagem passavam para funções em C (terminadas em `\0`), a linguagem validava que a string terminava em `.png`, mas a chamada `open()` em C parava de ler no byte nulo `\0`, abrindo `/etc/passwd`. |
| **Validação de Prefixo Sem Barra Final** | `/var/www/public-secret/chaves.txt` | Se o diretório base é `/var/www/public` e o código verifica apenas `caminho.startsWith('/var/www/public')` **sem a barra final** `/`, uma pasta irmã chamada `/var/www/public-secret` passa na verificação! |

##### 3. Por que acontece

A causa-raiz do Path Traversal avançado é a **violação da ordem correta entre Decodificação, Canonicalização e Validação**:

```text
ORDEM INCORRETA (VULNERÁVEL):
  [Entrada Bruta] -> [1. Valida/Filtra "../"] -> [2. Decodifica/Desaninha] -> [3. Abre no SO]
  (O atacante passa o payload codificado pela etapa 1; a etapa 2 reconstrói o "../"!)

ORDEM CORRETA (SEGURA — Canonicalização antes da Validação):
  [Entrada Bruta] -> [1. Decodifica completamente] -> [2. Canonicaliza para Caminho Absoluto Real]
                  -> [3. Valida se inicia com "/base_permitida/"] -> [4. Abre no SO]
```

> [!WARNING]
> **Armadilha clássica em Python e Node.js com Caminhos Absolutos**:
> Em **Python**, `os.path.join('/var/www/uploads', '/etc/passwd')` retorna `'/etc/passwd'` (descarta o primeiro argumento se o segundo começa com `/`).
> Em **Node.js**, `path.resolve('/var/www/uploads', '/etc/passwd')` também retorna `'/etc/passwd'`!
> Por isso, nunca basta usar `os.path.join` ou `path.resolve`; é **obrigatório** validar o resultado final após a resolução!

##### 4. Exemplo didático de código vulnerável

Veja um endpoint em **Python (Flask)** que tenta bloquear o Path Traversal removendo `../` com `.replace()`, mas cai em três armadilhas clássicas (sequência aninhada `....//`, caminho absoluto `/etc/passwd` em `os.path.join`, e falta de barra final na checagem de prefixo):

```python
# ARQUIVO VULNERÁVEL: app_relatorios.py (Ambiente de Laboratório)
import os
from flask import Flask, request, abort, send_file

app = Flask(__name__)
BASE_DIR = "/var/www/app/relatorios"

@app.route("/api/baixar")
def baixar_relatorio():
    nome = request.args.get("arquivo", "")

    # FALHA 1: Sanitização não-recursiva! "....//" transforma-se em "../" após o replace!
    nome_filtrado = nome.replace("../", "").replace("..\\", "")

    # FALHA 2: Em Python, se nome_filtrado começar com "/" (ex.: "/etc/passwd"),
    # os.path.join() DESCARTA completamente o BASE_DIR e retorna "/etc/passwd"!
    caminho = os.path.join(BASE_DIR, nome_filtrado)

    # FALHA 3: Se usar normpath com startsWith sem barra final ("/var/www/app/relatorios"),
    # permite acessar a pasta irmã "/var/www/app/relatorios_confidenciais/segredo.txt"!
    caminho_normalizado = os.path.normpath(caminho)
    if not caminho_normalizado.startswith(BASE_DIR):
        abort(403, "Acesso negado")

    return send_file(caminho_normalizado)
```

##### 5. Como identificar

1. **Em revisão de código**: procure por `.replace('../', '')`, `os.path.join(base, user_input)` sem verificação posterior, `path.resolve(base, user_input)` sem verificação posterior, ou checagens `.startsWith(BASE_DIR)` que não concatenam `os.sep` / `path.sep` ao final de `BASE_DIR`.
2. **Em testes de caixa-preta**: verifique como a aplicação reage a pontos e barras codificados (`%2e`, `%2f`, `%5c`), a sequências dobradas (`....//`) e a caminhos iniciados por `/` em parâmetros de arquivo.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

No laboratório **PortSwigger Web Security Academy** (série de laboratórios de *File Path Traversal*) ou em container local executando a aplicação Python acima:

1. **Teste de sequência simples**: o estudante envia `GET /api/baixar?arquivo=../../../etc/passwd`. O `.replace("../", "")` transforma a entrada em `etc/passwd`, resultando em `404 Not Found`.
2. **Teste de sequência aninhada (`....//`)**: o estudante envia `GET /api/baixar?arquivo=....//....//....//....//etc/passwd`.
   - O método `.replace("../", "")` remove o `../` central de cada `....//`, deixando `../../../../etc/passwd`.
   - Contudo, na etapa seguinte (`os.path.normpath`), o caminho vira `/etc/passwd` e para na verificação `startswith(BASE_DIR)`.
3. **Exploração da falha de prefixo sem barra final (Partial Path Traversal — CWE-22)**: sabendo que existe um diretório irmão `/var/www/app/relatorios_confidenciais/chaves.txt` no laboratório, o estudante envia:
   ```http
   GET /api/baixar?arquivo=....//relatorios_confidenciais/chaves.txt HTTP/1.1
   Host: localhost:5000
   ```
   - `nome.replace("../", "")` transforma `....//relatorios_confidenciais/chaves.txt` em `../relatorios_confidenciais/chaves.txt`.
   - `os.path.normpath("/var/www/app/relatorios/../relatorios_confidenciais/chaves.txt")` resolve para `"/var/www/app/relatorios_confidenciais/chaves.txt"`.
   - Como essa string começa com `"/var/www/app/relatorios"`, a condição `caminho_normalizado.startswith(BASE_DIR)` retorna `True`, vazando o arquivo confidencial!

##### 7. Como corrigir

A técnica padrão-ouro para eliminar todas as variantes de Path Traversal é **canonicalizar o caminho completo com resolução de links simbólicos (`os.path.realpath` em Python, `fs.realpathSync` / `path.resolve` em Node.js, `Path.toRealPath()` em Java)** e verificar a contenção estrita dentro do diretório base:

- **[REQUISITO]** Nunca tentar "limpar" `../` usando `.replace()` ou regex de remoção; entradas suspeitas devem ser **rejeitadas integralmente** ( *Fail Closed*).
- **[REQUISITO]** Resolver o caminho real absoluto (`realpath`) tanto do diretório base quanto do arquivo alvo e usar `os.path.commonpath([base_real, alvo_real]) == base_real` (em Python) ou `alvoReal.startsWith(baseReal + path.sep)` (em Node.js).

Veja o código corrigido completo em Python (Flask):

```python
# ARQUIVO CORRIGIDO: app_relatorios.py
import os
import re
from flask import Flask, request, abort, send_file

app = Flask(__name__)
# Resolve o caminho canônico real do diretório base na inicialização
BASE_DIR_REAL = os.path.realpath("/var/www/app/relatorios")
REGEX_NOME_PERMITIDO = re.compile(r"^[a-zA-Z0-9_-]+\.(pdf|csv|txt)$")

@app.route("/api/baixar")
def baixar_relatorio():
    nome = request.args.get("arquivo", "")

    # 1. Validação estrita por allowlist do nome do arquivo (sem barras nem pontos duplos)
    if not REGEX_NOME_PERMITIDO.match(nome):
        abort(400, "Nome de arquivo inválido.")

    # 2. Construção e canonicalização completa (resolve '..', '.' e links simbólicos)
    caminho_candidato = os.path.join(BASE_DIR_REAL, nome)
    caminho_real = os.path.realpath(caminho_candidato)

    # 3. Verificação segura de hierarquia usando os.path.commonpath (imune ao bug de pasta irmã!)
    if os.path.commonpath([BASE_DIR_REAL, caminho_real]) != BASE_DIR_REAL:
        abort(403, "Acesso negado.")

    if not os.path.isfile(caminho_real):
        abort(404, "Arquivo não encontrado.")

    return send_file(caminho_real)
```

##### 8. Como testar se a correção funcionou

1. Envie payloads com codificação simples (`%2e%2e%2f`), dupla (`%252e%252e%252f`), sequências aninhadas (`....//`), barras invertidas (`..\`), caminhos absolutos (`/etc/passwd`) e pastas irmãs (`....//relatorios_confidenciais/chaves.txt`).
2. Confirme que todos retornam `400 Bad Request` ou `403 Forbidden`, enquanto um arquivo legítimo (`relatorio_2026.pdf`) retorna `200 OK`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Na subaba **Payload** (*Query String Parameters*), o DevTools oferece o botão **View URL-encoded** vs. **View decoded**: alternar entre esses dois modos permite ver exatamente como sequências `%2e%2e%2f` ou `%252e%252e%252f` estão sendo enviadas na requisição bruta.
  - Em ataques contra arquivos estáticos mal roteados pelo proxy reverso, a coluna **Status** e a subaba **Response** mostram se uma URL contendo `/static/..%2fconfig.json` retornou o arquivo de configuração do back-end.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Sob **Frames -> Top -> Resources**, podem aparecer templates ou arquivos carregados dinamicamente por endpoints que recebem caminhos relativos como parâmetro.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - Ao testar chamadas via `fetch('/api/baixar?arquivo=' + encodeURIComponent('...'))` diretamente no REPL do Console, é possível verificar como o navegador normaliza barras (`/..`) na própria URL antes mesmo de enviá-la na rede (por exemplo, `fetch('/pasta/../segredo')` é normalizado pelo próprio navegador para `/segredo`, razão pela qual parâmetros de query string `?arquivo=../` ou codificações `%2e%2e%2f` são os vetores reais analisados).

---

<a id="s3-18"></a>
### 3.18 Race Conditions (Condições de Corrida / TOCTOU)

##### 1. Definição

As **Race Conditions** (Condições de Corrida — catalogadas no **CWE-362**: *Concurrent Execution using Shared Resource with Improper Synchronization*) ocorrem quando o comportamento e o resultado de um sistema dependem da ordem ou do tempo exato em que múltiplas threads, processos ou requisições concorrentes são escalonados e executados sobre um estado compartilhado (como saldo em banco de dados, estoque de produto, limite de uso de cupom ou arquivo no sistema operacional).

A subcategoria mais cobrada em segurança de aplicações web é o **TOCTOU** (*Time-of-Check to Time-of-Use* — Tempo da Verificação até o Tempo do Uso, **CWE-367**), que acontece quando a aplicação primeiro **verifica** uma condição de segurança ou regra de negócio (ex.: *"Este cupom de desconto já foi utilizado?"* ou *"O saldo da conta é $\ge$ R$ 100?"*) e, alguns milissegundos depois, **utiliza** o resultado dessa verificação para efetivar a alteração de estado (ex.: *"Aplicar desconto e marcar cupom como usado"* ou *"Debitar R$ 100"*), sem que essas duas etapas formem uma operação **atômica** e isolada. No **OWASP Top 10**, enquadra-se em **A04:2021 / A06:2025 (Insecure Design)**.

##### 2. Como funciona

Em uma aplicação web assíncrona moderna (Node.js, Python ASGI, Go, Java, PHP-FPM com múltiplos workers), cada requisição HTTP recebida é processada concorrentemente. Quando a aplicação realiza uma leitura (`SELECT`) seguida de uma escrita (`UPDATE`/`INSERT`) em comandos separados sem bloqueio pessimista/otimista ou atomicidade de banco de dados, cria-se uma **janela de corrida (*race window*)** entre a leitura e a gravação — que dura tipicamente de 2 ms a 50 ms (tempo da viagem de rede até o banco de dados).

Se um usuário disparar **10 requisições simultâneas** (por exemplo, usando multiplexação **HTTP/2** *Single-Packet Attack* ou `Promise.all` com `fetch()`) para resgatar um cupom de uso único, todas as 10 requisições podem executar o `SELECT` antes que a primeira requisição consiga concluir o `UPDATE`:

```text
LINHA DO TEMPO DE UM ATAQUE TOCTOU (Resgate de Cupom de Uso Único de R$ 50):
Estado Inicial no Banco: cupom "BEMVINDO50", usado = false, saldo_carteira = R$ 0

Tempo   Requisição A (Thread/Task 1)            Requisição B (Thread/Task 2)
-----   -------------------------------------   -------------------------------------
t=1ms   SELECT usado FROM cupons                SELECT usado FROM cupons
        -> Retorna: usado = false (OK!)         -> Retorna: usado = false (OK!)
t=4ms   Verifica if (!usado) -> PASSA!          Verifica if (!usado) -> PASSA!
        (Janela de vulnerabilidade TOCTOU)      (Janela de vulnerabilidade TOCTOU)
t=8ms   Adiciona R$ 50 na carteira              Adiciona R$ 50 na carteira
t=12ms  UPDATE cupons SET usado = true          UPDATE cupons SET usado = true
-----   -------------------------------------   -------------------------------------
Resultado Final: O cupom de uso único foi creditado DUAS vezes (R$ 100), burlando a regra!
```

##### 3. Por que acontece

1. **Padrão *Read-Modify-Write* não atômico no código da aplicação**: ler o valor do banco para uma variável na memória da aplicação (`const saldo = await getSaldo()`), calcular o novo valor em código (`if (saldo >= valor) novoSaldo = saldo - valor`) e gravar de volta (`await setSaldo(novoSaldo)`) fora de uma transação com bloqueio adequado.
2. **Nível de isolamento padrão do banco de dados insuficiente**: a maioria dos bancos de dados relacionais (como PostgreSQL, MySQL/InnoDB, SQL Server e Oracle) utiliza por padrão o nível de isolamento **`READ COMMITTED`** (ou `REPEATABLE READ` sem bloqueio de leitura no MySQL), no qual duas leituras `SELECT` comuns (sem `FOR UPDATE`) leem a mesma versão confirmada da linha simultaneamente sem aguardar uma à outra!
3. **Uso de `async/await` sobre variáveis compartilhadas em memória**: em servidores Node.js (que têm uma única thread de JS, mas **intercalam a execução no Event Loop** sempre que encontram um `await` de I/O de rede ou banco), duas requisições paralelas intercalam seus passos entre cada `await`.
4. **Ausência de chaves de idempotência (*Idempotency Keys*) e restrições de unicidade (`UNIQUE`) no banco de dados**.

##### 4. Exemplo didático de código vulnerável

Veja um endpoint em **Node.js/Express com PostgreSQL (`pg`)** para transferência de pontos ou resgate de voucher que implementa a lógica de verificação e atualização em etapas separadas vulneráveis a TOCTOU:

```javascript
// ARQUIVO VULNERÁVEL: src/routes/carteira.js (Ambiente de Laboratório)
const express = require('express');
const db = require('../database'); // Pool de conexões PostgreSQL
const router = express.Router();

router.post('/api/carteira/sacar', async (req, res) => {
  const usuarioId = req.session.usuarioId;
  const valorSaque = Number(req.body.valor);

  if (!Number.isFinite(valorSaque) || valorSaque <= 0) {
    return res.status(400).json({ erro: 'Valor inválido.' });
  }

  // ETAPA 1 — TIME OF CHECK (Leitura sem bloqueio de linha e fora de operação atômica)
  const { rows } = await db.query(
    'SELECT saldo FROM contas WHERE usuario_id = $1',
    [usuarioId]
  );

  const saldoAtual = Number(rows[0]?.saldo || 0);

  // Se 5 requisições paralelas chegarem aqui juntas quando saldoAtual = 100 e valorSaque = 100,
  // TODAS as 5 passarão por este "if" antes que o UPDATE abaixo seja executado!
  if (saldoAtual < valorSaque) {
    return res.status(400).json({ erro: 'Saldo insuficiente.' });
  }

  // Simula latência de validação externa ou processamento (amplia a janela TOCTOU)
  await registrarAuditoriaExterna(usuarioId, valorSaque);

  // ETAPA 2 — TIME OF USE (Atualização tardia sem revalidar o saldo na cláusula WHERE)
  await db.query(
    'UPDATE contas SET saldo = saldo - $1 WHERE usuario_id = $2',
    [valorSaque, usuarioId]
  );

  return res.status(200).json({ mensagem: 'Saque aprovado!', valor: valorSaque });
});

module.exports = router;
```

##### 5. Como identificar

- Procure fluxos financeiros ou de limite de uso: saques, transferências, resgate de cupons/gift cards, votação/curtidas únicas, geração de convites limitados, ou criação de contas com mesmo e-mail/username.
- No código-fonte, identifique sequências `await db.query('SELECT ...')` seguidas de `if (...)` e `await db.query('UPDATE ...')` sem transação explícita (`BEGIN ... COMMIT`), sem cláusula `SELECT ... FOR UPDATE` e sem verificação da condição dentro da própria cláusula `WHERE` do `UPDATE`.

##### 6. Como ocorre o teste em um ambiente autorizado de laboratório

Em um ambiente de laboratório autorizado (como **PortSwigger Web Security Academy** — módulo *Race Conditions* ou **OWASP Juice Shop**):

1. **Verificação de estado inicial**: a conta do estudante no laboratório possui saldo de `R$ 100,00`.
2. **Disparo concorrente via Console do DevTools (`Promise.all`)**: com a sessão autenticada no laboratório, o estudante utiliza `Promise.all()` na aba **Console** para disparar 5 requisições `fetch()` exatamente ao mesmo tempo, aproveitando a multiplexação da conexão:
   ```javascript
   // Teste educacional em laboratório autorizado: disparo de 5 requisições concorrentes
   await Promise.all(
     Array.from({ length: 5 }, () =>
       fetch('/api/carteira/sacar', {
         method: 'POST',
         headers: { 'Content-Type': 'application/json' },
         body: JSON.stringify({ valor: 100 })
       }).then(r => r.json())
     )
   ).then(resultados => console.table(resultados));
   ```
3. **Análise do resultado**: no servidor vulnerável, 3 a 5 das requisições retornam `200 OK ("Saque aprovado!")` porque leram `saldo = 100` simultaneamente antes do primeiro `UPDATE`, deixando o saldo da conta no banco negativo (`-R$ 400,00`).

##### 7. Como corrigir

A eliminação de condições de corrida em aplicações web baseia-se em **atomicidade no banco de dados**, **bloqueio pessimista (`SELECT ... FOR UPDATE`)** e **restrições de integridade no esquema**:

- **[REQUISITO]** Realizar a verificação e a atualização em uma **única instrução SQL atômica condicional** (`UPDATE contas SET saldo = saldo - $1 WHERE usuario_id = $2 AND saldo >= $1 RETURNING saldo`) verificando o número de linhas afetadas (`rowCount === 1`), **ou** envolver a operação em uma transação de banco de dados usando **bloqueio de linha explícito (`SELECT ... FOR UPDATE`)**.
- **[REQUISITO]** Adicionar restrições **`CHECK (saldo >= 0)`** e índices **`UNIQUE`** (ex.: `UNIQUE (usuario_id, cupom_id)`) diretamente nas tabelas do banco de dados como última linha de defesa intransponível.
- **[RECOMENDAÇÃO]** Exigir um cabeçalho **`Idempotency-Key`** (UUID gerado pelo cliente para a operação) em endpoints de pagamento e transferência, armazenado com chave única no banco ou Redis.

Veja o código corrigido completo em Node.js/Express com transação ACID (*Atomicity, Consistency, Isolation, Durability*), bloqueio pessimista e atualização atômica condicional:

```javascript
// ARQUIVO CORRIGIDO: src/routes/carteira.js
const express = require('express');
const db = require('../database');
const router = express.Router();

// Nota de Esquema no PostgreSQL (Defesa em Profundidade no Banco):
// ALTER TABLE contas ADD CONSTRAINT chk_saldo_nao_negativo CHECK (saldo >= 0);

router.post('/api/carteira/sacar', async (req, res) => {
  const usuarioId = req.session.usuarioId;
  const valorSaque = Number(req.body.valor);

  if (!Number.isFinite(valorSaque) || valorSaque <= 0) {
    return res.status(400).json({ erro: 'Valor inválido.' });
  }

  const client = await db.connect();
  try {
    await client.query('BEGIN');

    // CORREÇÃO PRIMÁRIA: Operação ATÔMICA condicional!
    // O banco bloqueia a linha, verifica "saldo >= $1" e subtrai em um único passo indivisível.
    // Se 5 requisições concorrentes chegarem juntas com saldo = 100 e valorSaque = 100,
    // a 1ª atualiza (rowCount = 1, saldo vira 0) e as outras 4 falham na cláusula WHERE (rowCount = 0)!
    const resultado = await client.query(
      `UPDATE contas
          SET saldo = saldo - $1
        WHERE usuario_id = $2
          AND saldo >= $1
      RETURNING saldo`,
      [valorSaque, usuarioId]
    );

    if (resultado.rowCount === 0) {
      await client.query('ROLLBACK');
      return res.status(409).json({ erro: 'Saldo insuficiente para realizar o saque.' });
    }

    await client.query(
      'INSERT INTO historico_saques (usuario_id, valor) VALUES ($1, $2)',
      [usuarioId, valorSaque]
    );

    await client.query('COMMIT');
    return res.status(200).json({
      mensagem: 'Saque aprovado com segurança!',
      novoSaldo: resultado.rows[0].saldo
    });
  } catch (erro) {
    await client.query('ROLLBACK');
    return res.status(500).json({ erro: 'Erro ao processar transação.' });
  } finally {
    client.release();
  }
});

module.exports = router;
```

##### 8. Como testar se a correção funcionou

1. Restaure o saldo da conta de laboratório para `R$ 100,00`.
2. Execute novamente o script `Promise.all` com 5 ou 20 requisições simultâneas de saque de `R$ 100,00`.
3. Confirme que **exatamente uma (1)** requisição retorna `200 OK` (com `novoSaldo: 0`) e **todas as demais** retornam `409 Conflict ("Saldo insuficiente...")`, mantendo o saldo final em `R$ 0,00`.

##### 9. Como ela pode aparecer durante uma análise no DevTools

- **Aba Network ([Seção 2.1](./02a-devtools-network.md#s2-1))**:
  - Na coluna **Waterfall** (Cascata de tempo), requisições vulneráveis a condições de corrida (ou disparadas por um botão de formulário que não é desabilitado no primeiro clique — *Double Submit*) aparecem com barras horizontais **paralelas e sobrepostas no mesmo instante de tempo**, compartilhando o mesmo **Connection ID** (em conexões `h2` / `h3`).
  - Se mais de uma requisição paralela para uma operação de uso único retornar **Status `200 OK`**, fica evidenciada a ausência de controle de concorrência/idempotência no servidor.
- **Aba Application ([Seção 2.2](./02b-devtools-application.md#s2-2))**:
  - Aplicações que tentam controlar limites de uso apenas salvando uma flag no cliente (`localStorage.setItem('cupomAplicado', 'true')`) podem ser identificadas inspecionando o **Local Storage** ou **Session Storage**.
- **Aba Console ([Seção 2.3](./02c-devtools-console.md#s2-3))**:
  - A aba Console permite usar `Promise.all()` com `fetch()` e `console.table()` (como demonstrado no item 6) para auditar em ambientes de homologação se um endpoint crítico é idempotente e resistente a requisições concorrentes.

---

<a id="s3-19"></a>
### 3.19 Matriz Comparativa das 18 Vulnerabilidades Web

Para consolidar o estudo completo de toda a **Seção 3** ([itens 3.1 a 3.9 na parte anterior](./03a-vulnerabilidades-parte1.md#s3) e [itens 3.10 a 3.18 nesta parte](#s3-10)), a tabela a seguir relaciona cada uma das **18 vulnerabilidades**, seu mapeamento no **OWASP Top 10 (2021 e 2025)**, seu código **CWE principal**, a **causa-raiz técnica**, **como identificá-la no Chrome DevTools** e a **defesa primária exigida em prova**:

| # | Vulnerabilidade | OWASP Top 10 (2021 / 2025) | CWE Principal | Causa-Raiz Técnica | Evidência Principal no Chrome DevTools | Defesa Primária (Padrão-Ouro) |
| :-: | :--- | :--- | :--- | :--- | :--- | :--- |
| **3.1** | **SQL Injection (SQLi)** | A03:2021 / A05:2025 (*Injection*) | CWE-89 | Concatenação direta de entrada do usuário em comandos SQL sem separação entre código e dados. | **Network**: erros de banco na subaba *Response* (`SQL syntax`), retorno de linhas extras ou atraso no *TTFB* (*Blind SQLi*). | **Prepared Statements** (*Parameterized Queries* / Consultas Parametrizadas) com vinculação de variáveis (*bind variables*). |
| **3.2** | **Cross-Site Scripting (XSS)** | A03:2021 / A05:2025 (*Injection*) | CWE-79 | Inserção de dados não confiáveis no HTML/DOM sem codificação contextual ou uso de *sinks* inseguros (`innerHTML`, `eval`). | **Network/Elements**: payload refletido sem escape no HTML; **Console**: execução de JS ou alertas de violação de CSP. | **Codificação de saída contextual** (*Context-Aware Output Encoding*), uso de `textContent`, sanitização (**DOMPurify**) e **CSP** (`nonce-` / *Trusted Types*). |
| **3.3** | **Cross-Site Request Forgery (CSRF)** | A01:2021 / A01:2025 (*Broken Access Control*) | CWE-352 | Confiar apenas no envio automático de cookies pelo navegador em ações de escrita sem verificar a intenção da origem. | **Network**: `POST`/`PUT` sem token anti-CSRF no *Payload*/*Headers*; **Application**: cookie de sessão com `SameSite=None` ou sem `SameSite`. | **Tokens Anti-CSRF** imprevisíveis por sessão/requisição + cookies com **`SameSite=Lax`** ou **`SameSite=Strict`** + validação de `Origin`/`Sec-Fetch-Site`. |
| **3.4** | **Server-Side Request Forgery (SSRF)** | A10:2021 / A01:2025 (*SSRF / Broken Access Control*) | CWE-918 | O servidor faz requisições HTTP/rede para URLs fornecidas pelo usuário sem validar o destino e o IP resolvido. | **Network**: parâmetros de payload contendo URLs completas (`?url=https://...`, `?webhook=...`); *Response* trazendo metadados internos. | **Allowlist estrita** de domínios/protocolos, resolução DNS com **bloqueio de faixas de IP privadas/loopback/link-local** (`127.0.0.1`, `169.254.169.254`) e desativação de redirecionamentos no cliente HTTP do servidor. |
| **3.5** | **Broken Access Control** | A01:2021 / A01:2025 (*Broken Access Control*) | CWE-284 / CWE-862 | Ausência de verificação de autorização vertical (papel/permissão) ou horizontal no lado do servidor a cada requisição. | **Network**: rota administrativa (`/api/admin/...`) respondendo `200 OK` para sessão de usuário comum; ocultação apenas via CSS/JS no cliente. | **Controle de acesso centralizado no servidor (*Deny by Default*)**, validando permissões do usuário autenticado em todas as rotas. |
| **3.6** | **Authentication Failures** | A07:2021 / A07:2025 (*Authentication Failures*) | CWE-287 / CWE-307 | Senhas fracas, ausência de limite de tentativas (*brute force*), enumeração de usuários ou tokens mal validados. | **Network**: centenas de `POST /login` sem bloqueio `429 Too Many Requests`; mensagens ou tempos diferentes revelando se o e-mail existe. | **MFA/2FA** (WebAuthn), **Rate Limiting** com `429`, mensagens de erro genéricas e uniformes em tempo constante, e hashing com **Argon2id / bcrypt**. |
| **3.7** | **Security Misconfiguration** | A05:2021 / A02:2025 (*Security Misconfiguration*) | CWE-16 / CWE-209 | Configurações padrão inseguras, modo *debug* ativo em produção, *stack traces* expostos, CORS `*` com credenciais ou falta de headers. | **Network**: *Response Headers* expondo `Server`/`X-Powered-By`, ausência de CSP/HSTS, *Response* `500` com *stack trace*; **Console**: avisos de CORS/arquivos `.map`. | **Hardening automatizado**, desativação de *debug* e *stack traces* ao cliente, remoção de banners de versão e aplicação de **Headers de Segurança HTTP**. |
| **3.8** | **Cryptographic Failures** | A02:2021 / A04:2025 (*Cryptographic Failures*) | CWE-311 / CWE-327 | Tráfego em texto claro (HTTP), TLS obsoleto (1.0/1.1), algoritmos quebrados (MD5, SHA-1, DES, ECB) ou chaves *hardcoded*. | **Network**: coluna *Scheme* com `http` ou *Mixed Content*; **Application/Sources**: chaves secretas expostas em bundles JS ou senhas sem TLS. | **HTTPS (TLS 1.3 / 1.2 forte)** com **HSTS**, algoritmos **AEAD** (`AES-GCM`, `ChaCha20-Poly1305`), **Argon2id** para senhas e gestão segura de segredos (*KMS/Vault*). |
| **3.9** | **IDOR / BOLA** | A01:2021 / A01:2025 (*Broken Access Control*) | CWE-639 | O servidor busca um registro pelo ID enviado na requisição (`/faturas/1052`) sem checar se aquele registro pertence ao usuário logado. | **Network**: alteração do ID na URL ou JSON (`1051` -> `1052`) retorna `200 OK` com dados pessoais (CPF, pedidos) de outro usuário. | **Verificação obrigatória de propriedade do objeto (*Object-Level Authorization*) no servidor** (`WHERE id = $1 AND usuario_id = $2`) + uso complementar de UUIDs. |
| **3.10** | **Directory Traversal & Directory Listing** | A01:2021 e A05:2021 / A01:2025 e A02:2025 | CWE-22 / CWE-548 | Navegação por `../` fora da pasta pública por falta de validação de fronteira, ou autoindexação (`autoindex on`) ativa no servidor web. | **Network**: parâmetros `?arquivo=../../` ou resposta `200 OK` exibindo `<title>Index of /uploads</title>` na subaba *Preview*. | **Desativar `autoindex` (`Options -Indexes`)**, usar `path.basename()`, validar por *allowlist* e verificar prefixo canônico com `realpath`. |
| **3.11** | **File Upload Vulnerabilities** | A04:2021 / A06:2025 (*Insecure Design*) | CWE-434 | Confiar na extensão ou `Content-Type` informado pelo cliente, preservar o nome original e servir uploads na mesma origem com execução. | **Network**: `POST` multipart aceitando `.svg`/`.php`; resposta retornando `/uploads/<nome_original>` no mesmo domínio sem `nosniff`. | **Validar Magic Bytes e extensão por *allowlist***, renomear com **UUID**, **recodificar imagens**, armazenar fora do *web root* em domínio separado com `X-Content-Type-Options: nosniff` e `CSP: sandbox`. |
| **3.12** | **Clickjacking (UI Redressing)** | A05:2021 / A02:2025 (*Security Misconfiguration*) | CWE-1021 | Ausência de cabeçalhos que proíbam páginas sensíveis de serem renderizadas dentro de um `<iframe>` em sites de terceiros. | **Network**: documento HTML sem `CSP: frame-ancestors` e sem `X-Frame-Options`; **Console**: erro `Refused to frame` quando corrigido. | Enviar **`Content-Security-Policy: frame-ancestors 'none'`** (ou `'self'`) em conjunto com **`X-Frame-Options: DENY`** (ou `SAMEORIGIN`). |
| **3.13** | **Open Redirect** | A01:2021 / A01:2025 (*Broken Access Control*) | CWE-601 | Redirecionar (`302 Location` ou `location.href`) para URL informada em parâmetro (`?next=`) sem validar se a origem final é interna. | **Network** (com *Preserve log*): resposta `301`/`302` com header `Location: https://externo.test` ou `//externo.test` refletindo a query string. | Validar o destino com o parser **`new URL(input, ORIGEM)`** exigindo `url.origin === ORIGEM`, bloqueando `\\` e `//`, ou usar mapa de rotas fixas. |
| **3.14** | **Session Hijacking** | A07:2021 / A07:2025 (*Authentication Failures*) | CWE-522 / CWE-614 | Exposição do token de sessão por falta de `HttpOnly`/`Secure`, guarda de JWT em `localStorage`, IDs previsíveis ou falta de revogação no logout. | **Application**: cookie de sessão sem ✔ em `HttpOnly`/`Secure`, ou token em *Local Storage*; **Console**: `document.cookie` exibe o ID de sessão. | Gerar ID com **CSPRNG ($\ge$ 128 bits)**, cookie **`__Host-`** com **`HttpOnly; Secure; SameSite=Lax`**, timeouts curtos e destruição da sessão no servidor no logout. |
| **3.15** | **Session Fixation** | A07:2021 / A07:2025 (*Authentication Failures*) | CWE-384 | O servidor mantém o mesmo identificador de sessão que existia antes do login após autenticar o usuário. | **Network / Application**: a resposta do `POST /login` não emite um novo `Set-Cookie` e o valor do cookie de sessão permanece idêntico antes e depois do login. | **Regenerar obrigatoriamente o ID da sessão no momento do login (`req.session.regenerate()`)**, invalidando o ID pré-login e aceitando sessões apenas via cookie. |
| **3.16** | **Command Injection** | A03:2021 / A05:2025 (*Injection*) | CWE-78 | Passar entrada do usuário em uma string executada por um shell do sistema operacional (`exec`, `system`, `shell=True`). | **Network**: *Response* contendo saída de terminal (`uid=`, `stdout`) ou *Timing (TTFB)* atrasado exatamente pelo tempo de um `sleep`. | Evitar chamadas ao SO; quando inevitável, usar **`execFile` / `spawn(..., { shell: false })`** com argumentos em array separado, marcador `--` e *allowlist* estrita. |
| **3.17** | **Path Traversal** | A01:2021 / A01:2025 (*Broken Access Control*) | CWE-22 | Falhas de canonicalização ao filtrar `../` antes da decodificação completa ou aceitar caminhos absolutos, `....//` e `%252e%252e%252f`. | **Network**: *Payload* exibindo parâmetros codificados (`%2e%2e%2f`) que retornam `200 OK` com conteúdo de arquivos de configuração do sistema. | **Canonicalizar o caminho real (`realpath` / `path.resolve`) após decodificação** e validar contenção hierárquica estrita (`commonpath` ou `startsWith(base + sep)`). |
| **3.18** | **Race Conditions (TOCTOU)** | A04:2021 / A06:2025 (*Insecure Design*) | CWE-362 / CWE-367 | Janela de tempo entre a leitura da condição (`SELECT`) e a gravação (`UPDATE`) permitindo que requisições paralelas burlem limites. | **Network**: múltiplas requisições paralelas na mesma faixa do *Waterfall* retornando todas `200 OK` para uma ação que deveria ser única. | **Operações atômicas no banco (`UPDATE ... WHERE saldo >= $1`)**, transações com **`SELECT ... FOR UPDATE`**, constraints `UNIQUE`/`CHECK` e **Idempotency Keys**. |

---

---

> [⬅️ 3A. Vulnerabilidades Web (3.0 a 3.9)](./03a-vulnerabilidades-parte1.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [4. Redes de Computadores e Portas ➡️](./04-redes-de-computadores.md)
