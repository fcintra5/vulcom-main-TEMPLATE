# Conversa: revisão do README do Vulcom para GitHub Codespaces

Data: 30/09/2026  
Repositório: https://github.com/faustocintra/vulcom-main  
Revisão analisada: `51ee2451e57b6482693f79399d94dd51cafc4d08`

> Exportação das mensagens da conversa relacionadas à tarefa. As referências da resposta foram convertidas em links Markdown. Chamadas de ferramentas e metadados operacionais foram omitidos. A análise abaixo registra o estado observado naquele momento; não houve alterações no repositório nem teste de execução em um codespace. As propostas não equivalem a mudanças já implementadas.

## Usuário — solicitação inicial

Acesse o repositório [https://github.com/faustocintra/vulcom-main](https://github.com/faustocintra/vulcom-main) e verifique se as instruções do README.md estão corretas para a configuração e a execução do projeto em um codespace do GitHub. O README faz menção a um nome de repositório `vulcom-main-YYYY-S` porque o repositório que está sendo analisado é um *template* para outros repositórios que serão criados semestralmente.

## Assistente — atualização inicial

Vou conferir o README e os arquivos de configuração do projeto, considerando que `vulcom-main-YYYY-S` é o nome esperado dos repositórios criados a partir do template.

## Assistente — atualização durante a análise

Encontrei dois pontos importantes: as URLs `localhost` não atendem ao acesso pelo navegador no Codespaces, e a sequência de inicialização pode executar o seed duas vezes no Prisma 6, causando conflito no usuário `admin`. Estou conferindo também a aprovação de scripts do npm, pois esse comando depende da versão instalada.

## Assistente — análise entregue

**O README está parcialmente correto, mas precisa de ajustes para funcionar no Codespaces pelo navegador.** Conferi o código, os arquivos de configuração e as dependências da revisão `51ee2451`. A análise foi estática, sem iniciar um codespace.

A referência a **`vulcom-main-YYYY-S` está adequada ao propósito do template**: o professor cria o repositório semestral e os alunos fazem o *fork* dele. Basta explicitar que `YYYY` e `S` devem ser substituídos pelo ano e semestre.

Os principais pontos encontrados são:

| Ponto | Avaliação |
|---|---|
| *Fork* e criação do codespace | Fluxo correto. O botão pode aparecer como **Create codespace on main** ou `+`. |
| `npm install` e `npm run dev` nas duas pastas | Correspondem aos scripts existentes. |
| Banco de dados | Usa **SQLite**; não exige servidor de banco nem `DATABASE_URL`. |
| URLs `localhost` | Precisam de adaptação para o acesso pelo navegador no Codespaces. |
| CORS | A configuração atual autoriza somente origens locais. |
| Migração seguida de seed | Pode executar o seed duas vezes e causar erro. |
| Aprovação de scripts do npm | O comando existe, mas depende da versão do npm. |
| Ambiente padronizado | Falta um `devcontainer.json` que fixe as ferramentas utilizadas. |

**1. A comunicação entre front-end e back-end é o principal problema**

No [README](https://github.com/faustocintra/vulcom-main/blob/main/README.md), o front-end recebe:

```ini
VITE_API_BASE="http://localhost:8888"
```

Entretanto, as requisições de [myfetch.js](https://github.com/faustocintra/vulcom-main/blob/main/front-end/src/lib/myfetch.js) são executadas no navegador do aluno. Nesse contexto, `localhost` aponta para o computador dele. Para acessar serviços do codespace pelo navegador, é preciso usar o endereço encaminhado pelo GitHub. [Fonte: GitHub Docs — solução de problemas de encaminhamento de portas](https://docs.github.com/en/codespaces/troubleshooting/troubleshooting-port-forwarding-for-github-codespaces).

Mantendo duas URLs, seria necessário configurar:

```ini
# front-end/.env.local
VITE_API_BASE="https://<NOME-DO-CODESPACE>-8888.app.github.dev"
VITE_AUTH_TOKEN_NAME="_auth"
```

```ini
# back-end/.env
ALLOWED_ORIGINS="https://<NOME-DO-CODESPACE>-5173.app.github.dev"
```

Os endereços devem ser copiados da aba **PORTS/PORTAS**. Porém, isso ainda exige considerar a autenticação das portas privadas do GitHub: o `myfetch` atual não envia credenciais nas chamadas entre origens, e essas portas possuem uma autenticação própria, independente do JWT da aplicação. [Fonte: GitHub Docs — segurança no Codespaces](https://docs.github.com/en/codespaces/reference/security-in-github-codespaces).

**Para esse template, recomendo usar o proxy do Vite.** Ele permite acessar a aplicação pela porta 5173 e encaminhar as chamadas à API internamente para a porta 8888.

Uma configuração proposta para `front-end/vite.config.js` seria:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

const codespaceHost =
  process.env.CODESPACE_NAME &&
  process.env.GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN
    ? `${process.env.CODESPACE_NAME}-5173.${process.env.GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}`
    : null

export default defineConfig({
  plugins: [react()],
  base: './',
  server: {
    host: '0.0.0.0',
    port: 5173,
    strictPort: true,
    allowedHosts: codespaceHost ? [codespaceHost] : [],
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:8888',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, '')
      }
    }
  }
})
```

Nesse caso, o `.env.local` fica independente do nome do codespace:

```ini
VITE_API_BASE="/api"
VITE_AUTH_TOKEN_NAME="_auth"
```

O Vite recebe `/api/users/login` e encaminha para `http://127.0.0.1:8888/users/login`. Esse recurso é previsto pela configuração oficial do Vite. A proposta acima exige alterar o arquivo do projeto; **o proxy ainda não existe no repositório**. [Fonte: Vite — opções do servidor](https://vite.dev/config/server-options).

O README deve orientar a manter os dois processos executando e abrir a porta **5173** pela aba **PORTS/PORTAS**, com visibilidade privada.

**2. A inicialização do banco deve evitar o seed duplicado**

O projeto usa Prisma **6.5.0**, conforme o arquivo de dependências travadas. Nessa versão, `prisma migrate dev` executa o seed quando cria um banco inexistente. Portanto, o `npx prisma db seed` seguinte repete a operação. [Fonte: Prisma ORM v6 — seeding](https://www.prisma.io/docs/orm/v6/prisma-migrate/workflows/seeding).

O [seed.js](https://github.com/faustocintra/vulcom-main/blob/main/back-end/prisma/seed.js) usa `create()` para inserir `admin`, cujo nome de usuário e e-mail são únicos. A segunda execução falha por duplicidade.

Para preservar as três etapas explícitas do README, use:

```bash
npx prisma generate
npx prisma migrate dev --name create-tables --skip-seed
npx prisma db seed
```

Acrescente que essa sequência é para a **primeira configuração**. Ao retomar um codespace já preparado, normalmente basta iniciar os dois servidores.

**3. O comando de aprovação de scripts não está errado**

```bash
npm install-scripts approve --all
```

Esse comando está documentado no npm atual. O problema é apresentá-lo como uma etapa universal sem definir a versão do npm. Além disso, os dois `package.json` já contêm autorizações `allowScripts` para versões específicas. [Fonte: npm — install-scripts](https://docs.npmjs.com/cli/v12/commands/npm-install-scripts/).

Sugiro condicionar essa etapa à existência de scripts pendentes e indicar como conferir:

```bash
node --version
npm --version
npm install-scripts ls
```

O último comando deve ser usado em uma versão do npm que o suporte. Se houver bloqueios no back-end, eles também precisam ser tratados antes dos comandos do Prisma.

**4. Há pequenos ajustes de clareza e reprodutibilidade**

- **Informar os caminhos completos dos arquivos de ambiente.** São `back-end/.env.example` e `front-end/.env.local.example`. Prefira copiá-los, preservando os exemplos.
- **Corrigir os comentários sobre autenticação.** `VITE_AUTH_TOKEN_NAME` nomeia a chave do `localStorage`, conforme o código atual. Não é o nome de um cookie, e não precisa tecnicamente coincidir com `AUTH_COOKIE_NAME`. A referência a `VITE_AUTH_COOKIE_NAME` também está desatualizada.
- **Retirar `npm audit fix` da configuração obrigatória.** Ele pode alterar as dependências travadas. Para uma turma, convém avaliar as atualizações no template e distribuir uma combinação previamente verificada.
- **Definir Node.js e npm.** O Faker 10.6.0 exige versões recentes de Node.js; Node 18 não atende. Um `.devcontainer/devcontainer.json` com uma versão compatível de Node 24 e npm definido tornaria o ambiente mais previsível.
- **Documentar o primeiro login:** usuário `admin`, senha `Vulcom@DSM`, conforme o seed.
- **Orientar a reinicialização dos servidores após alterações nos arquivos `.env`.**

**As correções prioritárias são a comunicação pelo proxy/portas e a prevenção do seed duplicado.** O nome semestral do repositório pode ser mantido. Não fiz alterações no GitHub.

## Usuário — solicitação de exportação

Exporte esta conversa como um arquivo Markdown para ser usado como contexto do Codex.

