# matrix-js-sdk

## Índice

- [Enquadramento da Componente](#enquadramento-da-componente)
- [Pré-requisitos](#pré-requisitos)
- [Instalação e Execução Local](#instalação-e-execução-local)
- [Configuração](#configuração)
- [Testes](#testes)
- [Build e Deployment](#build-e-deployment)
- [Estrutura do Repositório](#estrutura-do-repositório)
- [Contribuição e Governação do Código](#contribuição-e-governação-do-código)
- [Documentação Complementar](#documentação-complementar)
- [Licenciamento e SBOM](#licenciamento-e-sbom)
- [Segurança](#segurança)
- [Versão Entregue e Correspondência com o EA em Produção](#versão-entregue-e-correspondência-com-o-ea-em-produção)

---

## Enquadramento da Componente

O `matrix-js-sdk` é uma fork DGE do SDK `matrix-org/matrix-js-sdk` v27.0.0. Fornece o SDK Matrix Client-Server para JavaScript e TypeScript, utilizado internamente pela `matrix-react-sdk` como camada de comunicação com o servidor Matrix.

Este repositório é o componente de mais baixo nível da stack DGEChat de 4 repositórios:

```text
dgechat-desktop (ponto de entrada — Web e Desktop)
|-- element-desktop (wrapper Electron)
|-- element-web ("skin" para matrix-react-sdk)
|-- matrix-react-sdk (componente UI principal)
`-- matrix-js-sdk <-- este repositório (SDK Matrix Client-Server)
```

**Identificação EA:** SDK JavaScript/TypeScript — camada de comunicação com o servidor Matrix no contexto DGEChat.

---

## Pré-requisitos

| Dependência | Versão / Detalhe |
| --- | --- |
| Node.js | >= 18.0.0 (requerido pelo SDK) |
| Yarn | 1.x (recomendado em substituição a npm) |
| TypeScript | ^5.0.0 (compilação do projeto) |
| libolm | opcional, necessário para cifra ponta-a-ponta |

Não são necessárias variáveis de ambiente adicionais para desenvolvimento local.

---

## Instalação e Execução Local

**Instalação como dependência (projeto consumidor):**

```bash
yarn add matrix-js-sdk
```

**Desenvolvimento local (com linkagem para matrix-react-sdk):**

### 1. Clonar e instalar dependências

```bash
git clone https://github.com/DGEChat/matrix-js-sdk
cd matrix-js-sdk
yarn install
```

### 2. Compilar e expor via yarn link

```bash
yarn build
yarn link
```

### 3. Ligar em matrix-react-sdk

```bash
cd ../matrix-react-sdk
yarn link matrix-js-sdk
```

O ponto de entrada é `lib/index.js` (Node.js) ou `lib/browser-index.js` (browser).

---

## Configuração

**Instanciar o cliente Matrix:**

```javascript
import * as sdk from "matrix-js-sdk";
const client = sdk.createClient({
    baseUrl: "https://matrix.example.com",
    accessToken: "<token>",
    userId: "@user:example.com",
});
```

**Parâmetros principais do `createClient`:**

| Parâmetro | Descrição | Tipo |
| --- | --- | --- |
| `baseUrl` | URL base do servidor Matrix | String |
| `accessToken` | Token de acesso Matrix | String |
| `userId` | ID do utilizador Matrix | String |
| `fetchFn` | Função fetch alternativa (ponyfill) | Function |

**SonarQube (`sonar-project.properties`):**

| Parâmetro | Valor |
| --- | --- |
| `sonar.sources` | `src` |
| `sonar.tests` | `spec` |
| `sonar.exclusions` | `docs,examples,git-hooks` |
| `sonar.javascript.lcov.reportPaths` | `coverage/lcov.info` |
| `sonar.testExecutionReportPaths` | `coverage/jest-sonar-report.xml` |

**Cifra ponta-a-ponta (E2E):**

Para ativar E2E, fornecer a biblioteca `libolm` antes de inicializar o SDK, e chamar `await matrixClient.initCrypto()` antes de `matrixClient.startClient()`.

---

## Testes

```bash
# Executar todos os testes unitários (Jest)
yarn test

# Executar testes em modo watch
yarn test:watch

# Executar testes com cobertura (LCOV + SonarQube XML)
yarn coverage

# Verificar tipos TypeScript
yarn lint:types

# Executar linting (ESLint + Prettier)
yarn lint
```

> **Nota:** O teste `sync-browserify.spec.ts` requer um build de browser prévio (`yarn build`) para passar.

Relatórios de cobertura gerados em `coverage/lcov.info` e `coverage/jest-sonar-report.xml` (SonarQube).

---

## Build e Deployment

```bash
# Build completo (lib/ + browser bundle dist/)
yarn build

# Apenas compilar TypeScript e gerar tipos (sem browser bundle)
yarn build:dev

# Watch mode (recompila em alterações)
yarn start
```

O processo de build inclui:

- Compilação Babel para `lib/` (Node.js entry point)
- Bundle browser via Browserify para `dist/browser-matrix.js`
- Minificação para `dist/browser-matrix.min.js`

**Geração de documentação API (Typedoc):**

```bash
yarn gendoc
cd _docs
python -m http.server 8005
# Aceder a http://localhost:8005
```

Não existe workflow CI/CD configurado para publicação automática. A publicação no npm deve ser feita manualmente com `yarn prepublishOnly` (aciona o build completo automaticamente).

---

## Estrutura do Repositório

```text
matrix-js-sdk/
├── src/                        # código-fonte TypeScript
│   ├── client.ts               # MatrixClient — cliente principal
│   ├── crypto/                 # implementação de criptografia
│   ├── crypto-api/             # API de criptografia (Rust crypto)
│   ├── common-crypto/          # utilitários comuns de criptografia
│   ├── browser-index.ts        # entry point para browser
│   └── index.ts                # entry point Node.js
├── spec/                       # testes Jest
├── examples/                   # exemplos de uso (browser, node)
├── docs/                       # documentação adicional
├── git-hooks/                  # hooks git locais
├── lib/                        # build compilado (gerado)
├── dist/                       # browser bundle (gerado)
├── sonar-project.properties    # configuração SonarQube
├── tsconfig.json               # configuração TypeScript
├── tsconfig-build.json         # configuração TypeScript para build
└── typedoc.json                # configuração documentação Typedoc
```

**Convenções:**

- Código TypeScript em `src/`, testes em `spec/`
- Cada módulo principal tem ficheiro `.ts` em `src/`
- Rust crypto integrado via `@matrix-org/matrix-sdk-crypto-js`

---

## Contribuição e Governação do Código

**Workflow de branches:**

- Branch principal: `develop` (PRs dirigidos aqui)
- Branch estável: `master` (apenas releases)
- Pull requests obrigatórios para qualquer alteração

**Verificação antes de submeter PR:**

- Executar `yarn lint` sem erros
- Executar `yarn test` — todos os testes devem passar
- Se adicionar APIs públicas, documentar com TSDoc e regenerar com `yarn gendoc`

**Convenções de desenvolvimento:**

- TypeScript estrito — sem `any` implícito
- Testes unitários em `spec/` para novas funcionalidades
- Eventos emitidos via `EventEmitter` seguindo convenções Matrix

---

## Documentação Complementar

- [dgechat-desktop — ponto de entrada DGEChat](https://github.com/DGEChat/dgechat-desktop)
- [matrix-react-sdk — repositório UI DGEChat](https://github.com/DGEChat/matrix-react-sdk)
- [Matrix Client-Server API Specification](https://spec.matrix.org/latest/client-server-api/)
- [Typedoc — documentação API gerada](http://matrix-org.github.io/matrix-js-sdk/index.html)
- [Upstream: matrix-org/matrix-js-sdk](https://github.com/matrix-org/matrix-js-sdk)

---

## Licenciamento e SBOM

**Licença:** Apache-2.0

| Nome | Versão | Fornecedor | Tipo de Licença | Tipo | purl |
| --- | --- | --- | --- | --- | --- |
| @babel/runtime | ^7.12.5 | Babel / OpenJS Foundation | MIT | runtime | `pkg:npm/%40babel%2Fruntime@7.12.5` |
| @matrix-org/matrix-sdk-crypto-js | ^0.1.1 | matrix.org | Apache-2.0 | runtime | `pkg:npm/%40matrix-org%2Fmatrix-sdk-crypto-js@0.1.1` |
| another-json | ^0.2.0 | — | MIT | runtime | `pkg:npm/another-json@0.2.0` |
| bs58 | ^5.0.0 | — | MIT | runtime | `pkg:npm/bs58@5.0.0` |
| content-type | ^1.0.4 | — | MIT | runtime | `pkg:npm/content-type@1.0.4` |
| jwt-decode | ^3.1.2 | — | MIT | runtime | `pkg:npm/jwt-decode@3.1.2` |
| loglevel | ^1.7.1 | — | MIT | runtime | `pkg:npm/loglevel@1.7.1` |
| matrix-events-sdk | 0.0.1 | matrix.org | Apache-2.0 | runtime | `pkg:npm/matrix-events-sdk@0.0.1` |
| matrix-widget-api | ^1.3.1 | matrix.org | Apache-2.0 | runtime | `pkg:npm/matrix-widget-api@1.3.1` |
| oidc-client-ts | ^2.2.4 | Duende Software | Apache-2.0 | runtime | `pkg:npm/oidc-client-ts@2.2.4` |
| p-retry | 4 | Sindre Sorhus | MIT | runtime | `pkg:npm/p-retry@4` |
| sdp-transform | ^2.14.1 | — | MIT | runtime | `pkg:npm/sdp-transform@2.14.1` |
| unhomoglyph | ^1.0.6 | — | MIT | runtime | `pkg:npm/unhomoglyph@1.0.6` |
| uuid | 9 | — | MIT | runtime | `pkg:npm/uuid@9` |
| jest | ^29.0.0 | — | MIT | dev | `pkg:npm/jest@29.0.0` |
| typescript | ^5.0.0 | Microsoft | Apache-2.0 | dev | `pkg:npm/typescript@5.0.0` |

---

## Segurança

- **Gestão de segredos:** Credenciais Matrix (tokens de acesso, chaves de sessão) são geridas pela plataforma DGEChat; não incluídas no repositório.
- **Confirmação:** O repositório não contém passwords, API keys, tokens, certificados privados ou connection strings reais.
- **Cifra ponta-a-ponta:** Implementada via libolm (protocolos Olm/Megolm); requer inicialização explícita com `initCrypto()`.
- **Reporte de vulnerabilidades:** Reportar via issues no repositório ou por contacto direto com os responsáveis técnicos do projeto.

---

## Versão Entregue e Correspondência com o EA em Produção

| Campo | Valor |
| --- | --- |
| Versão | 27.0.0 |
| Tag/Release | — |
| Commit hash | *(a preencher)* |
| Data de release | *(a preencher)* |
| Ambiente | — |

---

> Developer Documentation (original below)

---

## Warning: This information is only partially applicable to DGEChat, look here instead: [dgechat-desktop](https://github.com/DGEChat/dgechat-desktop)

```text
dgechat-desktop (recommended starting point to build DGEChat for Web and Desktop)
|-- element-desktop (electron wrapper)
|-- element-web ("skin" for matrix-react-sdk)
|-- matrix-react-sdk (most of the development happens here)
`-- matrix-js-sdk <-- this repo (Matrix client js sdk)
```

## Matrix JavaScript SDK (Developer Documentation)

This is the [Matrix](https://matrix.org) Client-Server SDK for JavaScript and TypeScript. This SDK can be run in a
browser or in Node.js.

The Matrix specification is constantly evolving - while this SDK aims for maximum backwards compatibility, it only
guarantees that a feature will be supported for at least 4 spec releases. For example, if a feature the js-sdk supports
is removed in v1.4 then the feature is *eligible* for removal from the SDK when v1.8 is released. This SDK has no
guarantee on implementing all features of any particular spec release, currently. This can mean that the SDK will call
endpoints from before Matrix 1.1, for example.

## Quickstart

### In a browser

#### Note, the browserify build has been deprecated — please use a bundler like webpack or vite instead

Download the browser version from
[matrix-js-sdk releases](https://github.com/matrix-org/matrix-js-sdk/releases/latest) and add that as a
`<script>` to your page. There will be a global variable `matrixcs`
attached to `window` through which you can access the SDK. See below for how to
include libolm to enable end-to-end-encryption.

The browser bundle supports recent versions of browsers. Typically this is ES2015
or `> 0.5%, last 2 versions, Firefox ESR, not dead` if using
[browserlists](https://github.com/browserslist/browserslist).

Please check [the working browser example](examples/browser) for more information.

### In Node.js

Ensure you have the latest LTS version of Node.js installed.
This library relies on `fetch` which is available in Node from v18.0.0 - it should work fine also with polyfills.
If you wish to use a ponyfill or adapter of some sort then pass it as `fetchFn` to the MatrixClient constructor options.

Using `yarn` instead of `npm` is recommended. Please see the Yarn [install guide](https://classic.yarnpkg.com/en/docs/install)
if you do not have it already.

`yarn add matrix-js-sdk`

```javascript
import * as sdk from "matrix-js-sdk";
const client = sdk.createClient({ baseUrl: "https://matrix.org" });
client.publicRooms(function (err, data) {
    console.log("Public Rooms: %s", JSON.stringify(data));
});
```

See below for how to include libolm to enable end-to-end-encryption. Please check
[the Node.js terminal app](examples/node) for a more complex example.

You can also use the sdk with [Deno](https://deno.land/) (`import npm:matrix-js-sdk`) but its not officialy supported.

To start the client:

```javascript
await client.startClient({ initialSyncLimit: 10 });
```

You can perform a call to `/sync` to get the current state of the client:

```javascript
client.once("sync", function (state, prevState, res) {
    if (state === "PREPARED") {
        console.log("prepared");
    } else {
        console.log(state);
        process.exit(1);
    }
});
```

To send a message:

```javascript
const content = {
    body: "message text",
    msgtype: "m.text",
};
client.sendEvent("roomId", "m.room.message", content, "", (err, res) => {
    console.log(err);
});
```

To listen for message events:

```javascript
client.on("Room.timeline", function (event, room, toStartOfTimeline) {
    if (event.getType() !== "m.room.message") {
        return; // only use messages
    }
    console.log(event.event.content.body);
});
```

By default, the `matrix-js-sdk` client uses the `MemoryStore` to store events as they are received. For example to iterate through the currently stored timeline for a room:

```javascript
Object.keys(client.store.rooms).forEach((roomId) => {
    client.getRoom(roomId).timeline.forEach((t) => {
        console.log(t.event);
    });
});
```

### What does this SDK do?

This SDK provides a full object model around the Matrix Client-Server API and emits
events for incoming data and state changes. Aside from wrapping the HTTP API, it:

- Handles syncing (via `/initialSync` and `/events`)
- Handles the generation of "friendly" room and member names.
- Handles historical `RoomMember` information (e.g. display names).
- Manages room member state across multiple events (e.g. it handles typing, power
  levels and membership changes).
- Exposes high-level objects like `Rooms`, `RoomState`, `RoomMembers` and `Users`
  which can be listened to for things like name changes, new messages, membership
  changes, presence changes, and more.
- Handle "local echo" of messages sent using the SDK. This means that messages
  that have just been sent will appear in the timeline as 'sending', until it
  completes. This is beneficial because it prevents there being a gap between
  hitting the send button and having the "remote echo" arrive.
- Mark messages which failed to send as not sent.
- Automatically retry requests to send messages due to network errors.
- Automatically retry requests to send messages due to rate limiting errors.
- Handle queueing of messages.
- Handles pagination.
- Handle assigning push actions for events.
- Handles room initial sync on accepting invites.
- Handles WebRTC calling.

Later versions of the SDK will:

- Expose a `RoomSummary` which would be suitable for a recents page.
- Provide different pluggable storage layers (e.g. local storage, database-backed)

## Usage

### Conventions

#### Emitted events

The SDK will emit events using an `EventEmitter`. It also
emits object models (e.g. `Rooms`, `RoomMembers`) when they
are updated.

```javascript
// Listen for low-level MatrixEvents
client.on("event", function (event) {
    console.log(event.getType());
});

// Listen for typing changes
client.on("RoomMember.typing", function (event, member) {
    if (member.typing) {
        console.log(member.name + " is typing...");
    } else {
        console.log(member.name + " stopped typing.");
    }
});

// start the client to setup the connection to the server
client.startClient();
```

#### Promises and Callbacks

Most of the methods in the SDK are asynchronous: they do not directly return a
result, but instead return a [Promise](http://documentup.com/kriskowal/q/)
which will be fulfilled in the future.

The typical usage is something like:

```javascript
  matrixClient.someMethod(arg1, arg2).then(function(result) {
    ...
  });
```

Alternatively, if you have a Node.js-style `callback(err, result)` function,
you can pass the result of the promise into it with something like:

```javascript
matrixClient.someMethod(arg1, arg2).nodeify(callback);
```

The main thing to note is that it is problematic to discard the result of a
promise-returning function, as that will cause exceptions to go unobserved.

Methods which return a promise show this in their documentation.

Many methods in the SDK support *both* Node.js-style callbacks *and* Promises,
via an optional `callback` argument. The callback support is now deprecated:
new methods do not include a `callback` argument, and in the future it may be
removed from existing methods.

### Examples

This section provides some useful code snippets which demonstrate the
core functionality of the SDK. These examples assume the SDK is setup like this:

```javascript
import * as sdk from "matrix-js-sdk";
const myUserId = "@example:localhost";
const myAccessToken = "QGV4YW1wbGU6bG9jYWxob3N0.qPEvLuYfNBjxikiCjP";
const matrixClient = sdk.createClient({
    baseUrl: "http://localhost:8008",
    accessToken: myAccessToken,
    userId: myUserId,
});
```

#### Automatically join rooms when invited

```javascript
matrixClient.on("RoomMember.membership", function (event, member) {
    if (member.membership === "invite" && member.userId === myUserId) {
        matrixClient.joinRoom(member.roomId).then(function () {
            console.log("Auto-joined %s", member.roomId);
        });
    }
});

matrixClient.startClient();
```

#### Print out messages for all rooms

```javascript
matrixClient.on("Room.timeline", function (event, room, toStartOfTimeline) {
    if (toStartOfTimeline) {
        return; // don't print paginated results
    }
    if (event.getType() !== "m.room.message") {
        return; // only print messages
    }
    console.log(
        // the room name will update with m.room.name events automatically
        "(%s) %s :: %s",
        room.name,
        event.getSender(),
        event.getContent().body,
    );
});

matrixClient.startClient();
```

Output:

```text
  (My Room) @megan:localhost :: Hello world
  (My Room) @megan:localhost :: how are you?
  (My Room) @example:localhost :: I am good
  (My Room) @example:localhost :: change the room name
  (My New Room) @megan:localhost :: done
```

#### Print out membership lists whenever they are changed

```javascript
matrixClient.on("RoomState.members", function (event, state, member) {
    const room = matrixClient.getRoom(state.roomId);
    if (!room) {
        return;
    }
    const memberList = state.getMembers();
    console.log(room.name);
    console.log(Array(room.name.length + 1).join("=")); // underline
    for (var i = 0; i < memberList.length; i++) {
        console.log("(%s) %s", memberList[i].membership, memberList[i].name);
    }
});

matrixClient.startClient();
```

Output:

```text
  My Room
  =======
  (join) @example:localhost
  (leave) @alice:localhost
  (join) Bob
  (invite) @charlie:localhost
```

## API Reference

A hosted reference can be found at
[matrix-js-sdk API Reference](http://matrix-org.github.io/matrix-js-sdk/index.html)

This SDK uses [Typedoc](https://typedoc.org/guides/doccomments) doc comments. You can manually build and
host the API reference from the source files like this:

```bash
yarn gendoc
cd _docs
python -m http.server 8005
```

Then visit `http://localhost:8005` to see the API docs.

## End-to-end encryption support

The SDK supports end-to-end encryption via the Olm and Megolm protocols, using
[libolm](https://gitlab.matrix.org/matrix-org/olm). It is left up to the
application to make libolm available, via the `Olm` global.

It is also necessary to call `await matrixClient.initCrypto()` after creating a new
`MatrixClient` (but **before** calling `matrixClient.startClient()`) to
initialise the crypto layer.

If the `Olm` global is not available, the SDK will show a warning, as shown
below; `initCrypto()` will also fail.

```text
Unable to load crypto module: crypto will be disabled: Error: global.Olm is not defined
```

If the crypto layer is not (successfully) initialised, the SDK will continue to
work for unencrypted rooms, but it will not support the E2E parts of the Matrix
specification.

To provide the Olm library in a browser application:

- download the transpiled libolm (from [packages.matrix.org/npm/olm/](https://packages.matrix.org/npm/olm/)).
- load `olm.js` as a `<script>` *before* `browser-matrix.js`.

To provide the Olm library in a node.js application:

- `yarn add https://packages.matrix.org/npm/olm/olm-3.1.4.tgz`
  (replace the URL with the latest version you want to use from
  [packages.matrix.org/npm/olm/](https://packages.matrix.org/npm/olm/))
- `global.Olm = require('olm');` *before* loading `matrix-js-sdk`.

If you want to package Olm as dependency for your node.js application, you can
use `yarn add https://packages.matrix.org/npm/olm/olm-3.1.4.tgz`. If your
application also works without e2e crypto enabled, add `--optional` to mark it
as an optional dependency.

## Contributing

*This section is for people who want to modify the SDK. If you just
want to use this SDK, skip this section.*

First, you need to pull in the right build tools:

```bash
yarn install
```

### Building

To build a browser version from scratch when developing:

```bash
yarn build
```

To run tests (Jest):

```bash
yarn test
```

> **Note**
> The `sync-browserify.spec.ts` requires a browser build (`yarn build`) in order to pass

To run linting:

```bash
yarn lint
```
