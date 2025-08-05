# Projeto TypeScript do Zero

Este projeto mostra como criar, configurar e rodar um projeto básico em TypeScript.

## ✅ Requisitos

- Node.js instalado (versão recomendada: LTS)
- npm ou yarn

---

## 🛠️ Passos para criar o projeto

### 1. Criar a pasta do projeto

```bash
mkdir meu-projeto-typescript && cd meu-projeto-typescript
````

### 2. Inicializar o `package.json`

```bash
npm init -y
```

### 3. Instalar o TypeScript como dependência de desenvolvimento

```bash
npm install typescript
```

### 4. Criar o arquivo de configuração `tsconfig.json`

```bash
npx tsc --init
```

Ou crie manualmente:

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```
### 5. Instalar as bibliotecas necessárias
```bash
npm install express  
npm install @types/express -D
npm install mysql2
```
---

## 📁 Estrutura de Diretórios

```
meu-projeto-typescript/
├── src/
│   └── server.ts
├── package.json
├── tsconfig.json
```

---

## ✍️ Criar um arquivo TypeScript de exemplo

Crie o arquivo `src/index.ts`:

```ts
const sayHello = (name: string): void => {
  console.log(`Olá, ${name}!`);
};

sayHello('TypeScript');
```

---

## ⚙️ Adicionar scripts ao `package.json`

```json
"scripts": {
  "build": "tsc",
  "start": "tsc && node dist/index.js",
  "dev": "npx ts-node-dev src/server.ts"
}
```

---

## 🚀 Rodando o projeto

### 1. Apenas compilar o projeto

```bash
npm run build
```

### 2. Compilar e executar o projeto

```bash
npm start
```

### 3. Rodar o script do server
```bash
npm run dev
```

---

## ✅ Pronto!

Seu ambiente TypeScript está configurado e pronto para desenvolvimento!
