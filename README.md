# Forkando e criando um _codespace_ para este repositório

1. Faça _login_ no [GitHub](https://github.com).
2. Acesse [https://github.com/faustocintra/vulcom-main-YYYY-S](https://github.com/faustocintra/vulcom-main-YYYY-S).
3. Clique sobre o botão `[Fork]` no canto superior direito.
4. Na página seguinte ("Create new fork"), não altere nada, simplesmente clique sobre o botão `[Create fork]`. Aguarde.
5. Confira se a URL mostrada no navegador corresponde a "https://github.com/**<SEU USUÁRIO>**/vulcom-main-YYYY-S".
6. Clique sobre o botão verde `[Code]` e, em seguida:
  - No _popup_, clique sobre a aba `Codespaces`.
  - Clique sobre o botão `+` para criar um _codespace_ para o repositório.
  - Será aberta, no próprio navegador, uma aba do Visual Studio Code _online_ enquanto o _codespace_ é criado. Aguarde o término da criação.
  
> Um _codespace_ do GitHub é um ambiente de desenvolvimento hospedado na nuvem que pode trazer editor, linguagens, ferramentas e dependências totalmente configurados para um projeto, dispensando sua instalação no computador do usuário. Acessível pelo navegador de qualquer computador conectado à Internet, ele facilita o início das atividades, padroniza o ambiente entre os participantes e permite continuar o trabalho em diferentes máquinas, preservando arquivos e configurações. 

----

# Configurando o _back-end_

### Configuração das variáveis de ambiente

Renomeie o arquivo `.env.example` para `.env`. Ajuste o conteúdo do arquivo para o seguinte:
```ini
# Renomeie este arquivo para .env e preencha os valores abaixo

# Gere uma chave de token em https://jwtsecrets.com/
# (O token secret é simplesmente uma string aleatória)
TOKEN_SECRET=""

# Nome do cookie de autenticação, p. ex. _auth
# # (mesmo valor de VITE_AUTH_COOKIE_NAME no .env.local do front-end)
AUTH_COOKIE_NAME="_auth"

# URLs do front-end a partir do qual serão aceitas requisições
ALLOWED_ORIGINS="http://localhost:5173,http://127.0.0.1:5173"
```

> **IMPORTANTE**: gere o _token secret_ conforme indicado no comentário do arquivo e preencha o valor da variável `TOKEN_SECRET`.

## Instalação das dependências

Abra um terminal no VS Code. Nele, execute os comandos:
```
cd back-end
npm install
```

Caso apareça uma mensagem alertando sobre vulnerabilidades detectadas, execute:
```
npm audit fix
```

## Criação do banco de dados e dos registros iniciais

Ainda no terminal, na pasta `back-end`, execute:
```
npx prisma generate
npx prisma migrate dev --name create-tables
npx prisma db seed
```

## Executando o projeto

Estando dentro da pasta `back-end`, execute:
```
npm run dev
```

----

# Configurando o _front-end_

### Configuração das variáveis de ambiente

Renomeie o arquivo `.env.local.example` para `.env.local`. Ajuste o conteúdo do arquivo para o seguinte:
```ini
# Renomeie este arquivo para .env.local e preencha os valores abaixo

# Preencha com a URL do back-end
VITE_API_BASE="http://localhost:8888"

# Preencha com o nome do cookie de autenticação
# (mesmo valor de AUTH_COOKIE_NAME no .env do back-end)
VITE_AUTH_TOKEN_NAME="_auth"
```

## Instalação das dependências

Abra um segundo terminal no VS Code. Nele, execute os comandos:
```
cd front-end
npm install
```

Caso apareça uma mensagem alertando sobre vulnerabilidades detectadas, execute, dentro da pasta `front-end`:
```
npm audit fix
```

Autorize a execução dos _scripts_ pós-instalação, executando, dentro da pasta `front-end`:
```
npm install-scripts approve --all
```

## Executando o projeto

Estando dentro da pasta `front-end`, execute:
```
npm run dev
```