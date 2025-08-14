# 🎓 **Aula 6 – Criando e Consultando com ORM (TypeORM + MVC)**

## 🎯 Objetivos da Aula

- Entender o que é um **ORM** e como o **TypeORM** facilita o trabalho com banco de dados.
- Comparar o uso de SQL manual (`mysql2/promise`) com ORM.
- Criar entidades e mapear para tabelas com **decorators**.
- Implementar **relacionamentos** (One-to-Many, Many-to-One) no TypeORM.
- Usar `relations` para simular **JOINs** automaticamente.
- Estruturar um projeto **MVC sem View** (Model, Controller, Routes).

---

## 🧩 O que é um ORM?

**ORM** significa **Object-Relational Mapping** (*Mapeamento Objeto-Relacional*).

📌 É uma forma de interagir com o banco de dados **usando objetos e métodos**, sem precisar escrever SQL puro o tempo todo.

**Vantagens de usar ORM:**
- 📉 Menos repetição de código SQL.
- 🛡️ Menos risco de SQL Injection (tratamento automático).
- 🗂️ Melhor organização do código.
- 🔄 Portabilidade entre bancos de dados diferentes.

---

## 🧠 Diferença: SQL manual vs TypeORM

### No **mysql2/promise** (SQL manual):
```ts
const [rows] = await connection.query(
  'SELECT * FROM usuarios WHERE id = ?',
  [id]
);
````

### No **TypeORM**:

```ts
const user = await userRepository.findOneBy({ id });
```

💡 No SQL manual → você escreve a query.
💡 No ORM → você descreve o que quer, e ele gera a query para você.

---

## 🏗️ Criando o Projeto do Zero

### 1 Criar pasta do projeto

```bash
mkdir 14-08 && cd 14-08
```

### 2 Iniciar o projeto

```bash
npm init -y
```

### 3 Instalar dependências

```bash
npm install express typeorm reflect-metadata mysql2 dotenv
npm install -D typescript @types/node @types/express ts-node-dev
```

### 4 Criar o `tsconfig.json`

```bash
npx tsc --init
```

---

## ⚙️ Configurando o TypeScript

Coloque isso dentro do **`tsconfig.json`**:

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strictPropertyInitialization": false
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

**Explicando:**

* `experimentalDecorators` → habilita uso de `@Entity`, `@Column`, etc.
* `emitDecoratorMetadata` → necessário para o TypeORM saber tipos das colunas.
* `strictPropertyInitialization: false` → evita erro em propriedades que o ORM preenche sozinho.

---

## 📂 Estrutura de Pastas

```
src/
 ├── config/
 │    └── data-source.ts    # Configuração de conexão com o banco
 ├── controllers/           # Lógica de negócio (Controllers)
 ├── models/                # Entidades do banco (Models)
 ├── routes/                # Arquivos de rotas
 ├── server.ts               # Ponto de entrada do servidor
 └── .env                    # Variáveis de ambiente
```

---

## 🔑 Configurando a conexão com o banco

Arquivo **`src/config/data-source.ts`**:

```ts
import 'reflect-metadata';
import { DataSource } from 'typeorm';
import * as dotenv from "dotenv";
import { User } from '../models/User';
import { Post } from '../models/Post';

dotenv.config();

const { DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME } = process.env;

export const AppDataSource = new DataSource({
    type: 'mysql',
    host: DB_HOST,
    port: Number(DB_PORT || "3306"),
    username: DB_USER,
    password: DB_PASSWORD,
    database: DB_NAME,
    synchronize: true, // Apenas em desenvolvimento, isso cria todas as tabelas para nós
    logging: true,
    entities: [User, Post], // Entidades registradas
});
```

---

## 🧍 Criando a Entidade User

Arquivo **`src/models/User.ts`**:

```ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { Post } from './Post';

@Entity('users')
export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column({ length: 100, nullable: false })
    name: string;

    @Column({ unique: true })
    email: string;

    @OneToMany(() => Post, post => post.user)
    posts: Post[];
}
```

---

## 📝 Criando a Entidade Post

Arquivo **`src/models/Post.ts`**:

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';
import { User } from './User';

@Entity('posts')
export class Post {
    @PrimaryGeneratedColumn()
    id: number;

    @Column({ type: "varchar", length: 100, nullable: false })
    title: string;

    @ManyToOne(() => User, user => user.posts)
    user: User;
}
```

---

## 🎮 Criando Controllers

Arquivo **`src/controllers/UserController.ts`**:

```ts
import { Request, Response } from 'express';
import { AppDataSource } from '../config/data-source';
import { User } from '../models/User';

export class UserController {
    private userRepository = AppDataSource.getRepository(User);

    async list(req: Request, res: Response) {
        const users = await this.userRepository.find({ relations: ['posts'] });
        return res.json(users);
    }

    async create(req: Request, res: Response) {
        const { name, email } = req.body;
        const user = this.userRepository.create({ name, email });
        await this.userRepository.save(user);
        return res.status(201).json(user);
    }
}
```

Arquivo **`src/controllers/PostController.ts`**:

```ts
import { Request, Response } from 'express';
import { AppDataSource } from '../config/data-source';
import { Post } from '../models/Post';
import { User } from '../models/User';

export class PostController {
    private postRepository = AppDataSource.getRepository(Post);
    private userRepository = AppDataSource.getRepository(User);

    async create(req: Request, res: Response) {
        const { title, userId } = req.body;
        const user = await this.userRepository.findOneBy({ id: userId });
        if (!user) return res.status(404).json({ message: 'User not found' });

        const post = this.postRepository.create({ title, user });
        await this.postRepository.save(post);
        return res.status(201).json(post);
    }
}
```

---

## 🌐 Criando as Rotas

Arquivo **`src/routes/index.ts`**:

```ts
import { Router } from 'express';
import { UserController } from '../controllers/UserController';
import { PostController } from '../controllers/PostController';

const routes = Router();
const userController = new UserController();
const postController = new PostController();

routes.get('/users', (req, res) => userController.list(req, res));
routes.post('/users', (req, res) => userController.create(req, res));
routes.post('/posts', (req, res) => postController.create(req, res));

export default routes;
```

---

## 🚀 Criando o Servidor

Arquivo **`src/server.ts`**:

```ts
import "reflect-metadata";
import express, { Application } from "express";
import router from "./routes";
import { AppDataSource } from "./config/data-source";

const app: Application = express();
const PORTA: number = 3000;

app.use(express.json());

AppDataSource.initialize()
    .then(() => {
        console.log("📦 Banco conectado com sucesso");
        app.use(router);

        app.listen(PORTA, () => {
            console.log(`🚀 Servidor rodando na porta ${PORTA}`);
        });
    })
    .catch((err) => console.error("❌ Erro ao conectar no banco:", err));
```

---

## 📄 Arquivo `.env`

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=root
DB_NAME=meubanco
```

---

## 📝 Exercícios Práticos

1. Criar as entidades `Category` e `Product`.
2. Relacionar `Category` com `Product` (One-to-Many).
3. Criar rota `/products` que traga a categoria junto (`relations`).
4. Criar rota `/users/posts` que traga todos usuários e seus posts.

---

## ✅ Resumo

* ORM = converte objetos/classes em tabelas.
* TypeORM usa **decorators** para mapear tabelas.
* Estrutura **MVC sem View** → separa Models, Controllers e Rotas.
* `relations` carrega dados relacionados (JOIN).
* Sempre inicializar o `DataSource` antes de usar o repositório.
