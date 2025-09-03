# Criando um Script para Gerar Projetos TypeScript Automaticamente

Que tal aprender um pouco sobre automatização, usando Typescript? Vamos aplicar alguns conceitos que aprendemos em aula para criar um projeto que cria **automaticamente** outros projetos, com tudo já configurado!
O projeto que vamos aprender a criar gera projetos prontos com:

* Estrutura de pastas (`src`, `dist`)
* `tsconfig.json` com configuração inicial
* `package.json` atualizado com scripts `build` e `start`
* Arquivo inicial `index.ts`

---

## Objetivo do Script

O script:

1. Pergunta o nome do projeto ao usuário.
2. Cria a pasta do projeto **uma pasta acima da atual**.
3. Inicializa o projeto com `npm init -y`.
4. Instala dependências (`express`, `typeorm`, `reflect-metadata`, etc.).
5. Instala dependências de desenvolvimento (`typescript`, `ts-node-dev`, tipagens).
6. Cria `tsconfig.json` já configurado para **decorators**.
7. Atualiza `package.json` com scripts úteis.
8. Cria a estrutura **MVC** dentro de `src/`.
9. Gera arquivos iniciais (`server.ts`, `app.ts`, `data-source.ts`, `index.ts`, `.env`).

* Crie um projeto de nome `projeto-automatico`, e siga todo o passo a passo para criá-lo, como sempre fazemos.

Instale estas dependências nele:

```bash
# Instala TypeScript como dependência de desenvolvimento
npm install typescript --save-dev

# Instala tipos do Node.js para o TypeScript
npm install @types/node --save-dev

# Instala readline-sync para capturar input do usuário
npm install readline-sync

# (Opcional, mas recomendado) Tipagem do readline-sync
npm install @types/readline-sync --save-dev

```

---

## 2. Bibliotecas

Antes de começarmos a escrever o script, é importante entender quais bibliotecas iremos utilizar e para que cada uma serve.

### 1. `fs` (File System)
- Biblioteca nativa do Node.js.
- Permite criar, ler, escrever e deletar arquivos e pastas.
- Funções principais usadas no script:
  - `fs.mkdirSync(path, options)` → cria uma pasta.  
    - `path`: string com o caminho da pasta.  
    - `options.recursive`: se `true`, cria pastas intermediárias (pastas que ficam antes de chegar na pasta final) caso não existam.  
  
  - `fs.writeFileSync(path, data)` → cria ou sobrescreve arquivos.  
    - `path`: caminho do arquivo.  
    - `data`: conteúdo do arquivo (string).  

  - `fs.readFileSync(path, encoding)` → lê o conteúdo de um arquivo.  
    - `path`: caminho do arquivo.  
    - `encoding`: ex. `"utf-8"`.    "utf-8" faz com que o conteúdo seja retornado como texto legível e não como código binário.
    - Retorna uma string com o conteúdo do arquivo.

### 2. `child_process` (`execSync`)
- Biblioteca nativa do Node.js para executar comandos do terminal.  
- Função usada no script:
  - `execSync(command, options)` → executa um comando de forma síncrona.  
    - `command`: string com o comando do terminal, ex: `"npm init -y"`.  
    - `options.stdio`: `"inherit"` para mostrar a saída diretamente no terminal.  
    
- Permite rodar comandos como `npm install`, `tsc`, ou qualquer comando do sistema.

### 3. `path`
- Biblioteca nativa do Node.js para manipulação de caminhos de arquivos.  
- Função usada:
  - `path.join(...paths: string[])` → junta várias strings de caminho de forma segura, usando o separador correto do sistema operacional (`/` ou `\`).  
  - Retorno: string com o caminho completo.

### 4. `readline-sync`
- Biblioteca externa para ler input do usuário.  
- Função usada:
  - `readlineSync.question(prompt)` → mostra o prompt no terminal e retorna o que o usuário digitou como string.
- Permite que o script pergunte o nome do projeto antes de criar pastas e arquivos.


## Interfaces que vamos utilizar neste script

Para organizar melhor o código e garantir que o TypeScript entenda os tipos de cada objeto, **precisamos criar duas interfaces**:

1. **TsConfig**  
   - Representa a estrutura do arquivo `tsconfig.json`.  
   - Define os tipos das opções do compilador, pastas incluídas e excluídas.

2. **PackageJson**  
   - Representa a estrutura do arquivo `package.json`.  
   - Define os tipos de propriedades como `name`, `version` e `scripts`.

### Boas práticas
- É considerado **boa prática** separar cada interface em arquivos diferentes, especialmente em projetos maiores.  
- Por exemplo:
  - `interfaces/TsConfig.ts` → contém apenas a interface `TsConfig`.  
  - `interfaces/PackageJson.ts` → contém apenas a interface `PackageJson`.  
- Isso deixa o código mais organizado e facilita a manutenção.


### **TsConfig.ts**

```ts
// Interface para tipar o tsconfig.json
export interface TsConfig {
  compilerOptions: {
    target: string;
    module: string;
    moduleResolution: string;
    outDir: string;
    rootDir: string;
    strict: boolean;
    esModuleInterop: boolean;
    experimentalDecorators: boolean;
    emitDecoratorMetadata: boolean;
    strictPropertyInitialization: boolean;
  };
  include: string[];
}
```

### **PackageJson.ts**

```ts
// Interface para tipar o package.json
export interface PackageJson {
  name?: string;
  version?: string;
  description?: string;
  main?: string;
  scripts: {
    test: string;
    build: string;
    start: string;
    dev: string;
  };
  keywords?: string[];
  author?: string;
  license?: string;
  [key: string]: unknown;
}
```

## 1. Importando bibliotecas

```ts
import * as fs from "fs";                       // Para criar pastas e arquivos
import * as path from "path";                   // Para criar caminhos seguros
import { execSync } from "child_process";       // Para rodar comandos do terminal
import * as readlineSync from "readline-sync";  // Para perguntar algo ao usuário
import { TsConfig } from "./interfaces/TsConfig";       // Tipagem do tsconfig
import { PackageJson } from "./interfaces/PackageJson"; // Tipagem do package.json
```

**Explicação simples:**

* `fs` → manipula arquivos e pastas.
* `path` → cria caminhos corretos independente do sistema operacional.
* `execSync` → roda comandos do terminal dentro do script.
* `readlineSync` → permite perguntar algo ao usuário no terminal.
* `TsConfig` e `PackageJson` → garantem que os objetos criados estejam com os tipos corretos.

---

## 2. Função principal

Toda a lógica do script vai dentro de uma função chamada `createTsProject()`.

```ts
function createTsProject(): void {
  // Código do script vai aqui
}
```

---

## 3. Perguntar o nome do projeto

```ts
const projectName: string = readlineSync.question("Digite o nome do projeto: ");

if (!projectName) {
  console.log("Nome do projeto não pode ser vazio!");
  return;
}
```

**Explicação:**

* `readlineSync.question("...")` → pede para o usuário digitar algo.
* Se o usuário não digitar nada, mostramos uma mensagem e encerramos a função com `return`.

---

## 4. Definir o caminho do projeto

```ts
const projectPath: string = path.join("..", projectName);
```

* `".."` → indica que a pasta será criada **uma acima da pasta atual**.
* `projectName` → nome digitado pelo usuário.
* `path.join()` → garante que o caminho seja correto em qualquer sistema (Windows/Linux/Mac).

---

## 5. Criar a pasta do projeto

```ts
fs.mkdirSync(projectPath, { recursive: true });
```

* `recursive: true` → cria pastas intermediárias automaticamente se não existirem.
* Exemplo: se `"../MeuProjeto"` não existir, ele será criado.

---

## 6. Entrar na pasta do projeto

```ts
process.chdir(projectPath);
console.log("📦 Inicializando o projeto...");
```

* `process.chdir()` → muda o diretório atual do Node para a pasta do projeto.
* Assim, todos os comandos seguintes vão acontecer dentro desta pasta.

---

## 7. Inicializar npm e instalar dependências

```ts
// Inicializa npm
execSync("npm init -y", { stdio: "inherit" });

// Instala dependências de runtime
execSync(
  "npm install express cors dotenv bcrypt jsonwebtoken typeorm reflect-metadata",
  { stdio: "inherit" }
);

// Instala dependências de desenvolvimento
execSync(
  "npm install -D typescript @types/node @types/express @types/cors @types/bcrypt @types/jsonwebtoken ts-node-dev",
  { stdio: "inherit" }
);
```

* `execSync("comando", { stdio: "inherit" })` → executa o comando no terminal e mostra a saída.
* Dependências de runtime → necessárias para rodar a aplicação (`express`, `typeorm`, etc.).
* Dependências de dev → necessárias apenas durante o desenvolvimento (`typescript`, tipagens, `ts-node-dev`).

---

## 8. Criar o `tsconfig.json`

```ts
const tsConfig: TsConfig = {
  compilerOptions: {
    target: "ES6",
    module: "commonjs",
    moduleResolution: "Node",
    outDir: "dist",
    rootDir: "src",
    strict: true,
    esModuleInterop: true,
    experimentalDecorators: true,
    emitDecoratorMetadata: true,
    strictPropertyInitialization: false
  },
  include: ["src"]
};

fs.writeFileSync("tsconfig.json", JSON.stringify(tsConfig, null, 2));
```

* Define como o TypeScript vai compilar os arquivos.
* `experimentalDecorators` e `emitDecoratorMetadata` → permitem usar decorators, importante para TypeORM.

---

## 9. Atualizar `package.json` com scripts

```ts
const packageJsonRaw: string = fs.readFileSync("package.json", "utf-8");
const packageJson: PackageJson = JSON.parse(packageJsonRaw);

packageJson.scripts = {
  test: 'echo "Error: no test specified" && exit 1',
  build: "tsc",
  start: "tsc && node dist/server.js",
  dev: "ts-node-dev --respawn --transpile-only src/server.ts"
};

fs.writeFileSync("package.json", JSON.stringify(packageJson, null, 2));
```

* Lemos o `package.json` criado pelo `npm init`.
* Modificamos os scripts:

  * `build` → compila TS para JS
  * `start` → compila e roda o projeto
  * `dev` → roda o projeto em modo de desenvolvimento com recarga automática
* Salvamos o `package.json` atualizado.

---

## 10. Criar estrutura MVC dentro de `src/`

```ts
const srcFolders = ["entities", "controllers", "middlewares", "routes", "services", "utils"];
fs.mkdirSync("src");
srcFolders.forEach(folder => fs.mkdirSync(`src/${folder}`));
```

* Criamos a pasta `src` e subpastas típicas de projetos MVC.
* Cada pasta terá funções específicas:

  * `entities` → modelos do banco
  * `controllers` → lógica de requisições
  * `middlewares` → funções intermediárias
  * `routes` → rotas da API
  * `services` → lógica de negócios
  * `utils` → funções utilitárias

---

## 11. Criar arquivos iniciais

```ts
fs.writeFileSync("src/app.ts", "");
fs.writeFileSync("src/data-source.ts", "");
fs.writeFileSync("src/server.ts", "");
fs.writeFileSync("src/index.ts", `console.log("Projeto ${projectName} rodando!");`);
fs.writeFileSync(".env", "# Variáveis de ambiente\nPORT=3000\n");
```

* `app.ts` → configura o Express
* `server.ts` → inicializa o servidor
* `data-source.ts` → conexão com o banco
* `index.ts` → arquivo de teste inicial (`console.log`)
* `.env` → variáveis de ambiente, como `PORT`

---

## 12. Mensagem final de sucesso

```ts
console.log(`\n✅ Projeto "${projectName}" criado com sucesso em "${projectPath}"`);
console.log("👉 Para começar:");
console.log(`cd ../${projectName}`);
console.log("npm run dev");
```

* Mostra no terminal que o projeto foi criado.
* Mostra comandos para entrar na pasta e rodar em modo desenvolvimento.

---

## 13. Rodando a função principal

```ts
createTsProject();
```

* Chamamos a função para iniciar o script assim que ele for executado.

---











