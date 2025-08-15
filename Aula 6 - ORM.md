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
// Importa o "reflect-metadata", que é essencial para o TypeORM funcionar.
// Ele habilita o uso de decorators (@Entity, @Column, etc.) para mapear classes em tabelas do banco. Mapear classes em tabelas do banco significa transformar cada classe do código em uma tabela do banco de dados, onde cada propriedade da classe vira uma coluna e cada objeto vira um registro.
import 'reflect-metadata';

// Importa a classe DataSource do TypeORM.
// O DataSource é a configuração principal de conexão com o banco de dados.
// Ele sabe qual banco usar, onde conectar, quais entidades existem, etc.
import { DataSource } from 'typeorm';

// Importa a biblioteca dotenv, que serve para carregar variáveis de ambiente do arquivo .env
// Isso evita colocar senhas e dados sensíveis diretamente no código.
import * as dotenv from "dotenv";

// Importa as entidades do projeto. Entidades são justamente as classes que representam tabelas do banco de dados dentro do código, descrevendo seus campos (colunas) e relações com outras tabelas.
// Aqui o TypeORM precisa saber quais classes representam tabelas do banco.
import { User } from '../models/User';
import { Post } from '../models/Post';

// Carrega as variáveis de ambiente do arquivo .env para o process.env
// Agora podemos acessar process.env.DB_HOST, process.env.DB_USER, etc.
dotenv.config();

// Aqui pegamos as variáveis de ambiente definidas no arquivo .env
// O destructuring facilita, pegando cada valor e guardando em uma constante.
// process.env é um objeto do Node.js que guarda variáveis do arquivo .env
const { DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME } = process.env;

// Criamos e exportamos a configuração principal do banco de dados usando o TypeORM.
// O DataSource vai ser usado em qualquer parte do projeto onde precisarmos interagir com o banco.
/* É uma classe do TypeORM que representa a configuração e a conexão com o banco de dados. Quando você cria uma instância dela, você define:
    - Tipo do banco (mysql, postgres, etc.)
    - Host, porta, usuário, senha
    - Quais entidades usar
    - Se vai sincronizar tabelas automaticamente (synchronize)
*/
export const AppDataSource = new DataSource({
    // Qual tipo de banco vamos usar. Pode ser "mysql", "postgres", "sqlite", etc.
    type: 'mysql',

    // Endereço onde o banco está rodando. Pode ser um IP, localhost, etc.
    host: DB_HOST,

    // Porta de conexão do banco. MySQL por padrão usa 3306.
    port: Number(DB_PORT || "3306"),

    // Nome do usuário para acessar o banco de dados.
    username: DB_USER,

    // Senha do usuário do banco de dados.
    password: DB_PASSWORD,

    // Nome do banco de dados que vamos usar.
    database: DB_NAME,

    // O synchronize: true cria automaticamente as tabelas e colunas com base nas entidades.
    // ⚠️ Importante: Isso é útil apenas em desenvolvimento.
    // Em produção, deve ser false, para não apagar ou alterar dados automaticamente.
    synchronize: true, 

    // logging: true faz o TypeORM mostrar no terminal todos os comandos SQL que ele está executando.
    logging: true,

    // Aqui registramos as entidades (as classes que representam tabelas).
    // O TypeORM precisa saber quais são para criar o mapeamento com o banco.
    entities: [User, Post],
});

```

---

## 🧍 Criando a Entidade User

Arquivo **`src/models/User.ts`**:

```ts
// Importa os decorators e funções do TypeORM para criar a entidade e mapear colunas e relacionamentos
// Decorators são uma funcionalidade do TypeScript (e do JavaScript moderno) que permitem adicionar comportamento extra a classes, métodos ou propriedades de forma declarativa, usando o símbolo @. É por causa deles que conseguimos 'transformar' as classes e as propriedades dela em tabelas e colunas no nosso banco de dados. Cada decorator, cada @, diz ao ORM o que aquela classe ou propriedade representa.
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';

// Importa a entidade Post, para definir a relação entre User e Post
import { Post } from './Post';

// @Entity('users') indica que esta classe representa a tabela "users" no banco
@Entity('users')
export class User {
    // @PrimaryGeneratedColumn() indica que o campo é a chave primária (PK) e será auto-incrementado
    id: number;

    // @Column define que esta propriedade será uma coluna no banco
    // length: 100 → tamanho máximo do campo
    // nullable: false → não pode ser nulo
    @Column({ length: 100, nullable: false })
    name: string;

    // @Column com unique: true garante que o valor será único na tabela (não pode repetir)
    @Column({ unique: true })
    email: string;

    /*
        @OneToMany indica que um usuário pode ter vários posts (1:N)
        () => Post → função que retorna a entidade relacionada
        post => post.user → indica a propriedade na entidade Post que referencia o usuário
        O TypeORM usa isso para criar a relação e a chave estrangeira automaticamente
        Essa é apenas uma das pontas desta ligação. Devemos declarar o restante na entidade Post, para que o ORM consiga entender completamente             toda a relação que estamos tentando definir.
    */
    @OneToMany(() => Post, post => post.user)
    posts: Post[]; // Como um usuário pode ter muitos posts, usamos um array pois a propriedade posts vai armazenar uma lista de posts que pertencem àquele usuário.
}

```

---

## 📝 Criando a Entidade Post

Arquivo **`src/models/Post.ts`**:

```ts
// Importa os decorators e funções do TypeORM
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';

// Importa a entidade User, para definir a relação entre Post e User
import { User } from './User';

// @Entity('posts') indica que esta classe representa a tabela "posts" no banco
@Entity('posts')
export class Post {
    // @PrimaryGeneratedColumn() indica que o campo é a chave primária (PK) e será auto-incrementado
    id: number;

    // @Column define que esta propriedade será uma coluna no banco
    // type: "varchar" → tipo texto
    // length: 100 → tamanho máximo
    // nullable: false → não pode ser nulo
    @Column({ type: "varchar", length: 100, nullable: false })
    title: string;

    /*
        @ManyToOne indica que vários posts podem pertencer a um único usuário (N:1)
        () => User → função que retorna a entidade relacionada
        user => user.posts → indica a propriedade na entidade User que referencia os posts
        O TypeORM usa isso para criar a chave estrangeira automaticamente
    */
    @ManyToOne(() => User, user => user.posts)
    user: User; // Cada post terá exatamente um usuário dono
}

```

---

## 🎮 Criando Controllers

Arquivo **`src/controllers/UserController.ts`**:

```ts
// Importa tipos do Express para lidar com requisições e respostas
import { Request, Response } from 'express';

// Importa a instância do DataSource, que é a conexão com o banco
import { AppDataSource } from '../config/data-source';

// Importa a entidade User para trabalhar com a tabela de usuários
import { User } from '../models/User';

// Cria a classe UserController, que contém os métodos para manipular usuários
export class UserController {
    // Cria o repositório do User, que permite fazer operações no banco
    // O repositório é como uma “camada de acesso ao banco” fornecida pelo TypeORM. É um objeto fornecido pelo TypeORM que sabe como fazer operações no banco para uma entidade específica. Sem esta linha de código, não conseguimos interagir com o banco.
    private userRepository = AppDataSource.getRepository(User);

    // Método que lista todos os usuários
    // async → usamos await dentro dele para esperar o resultado do banco
    async list(req: Request, res: Response) {
        // find é uma função do repositório do TypeORM que busca um ou mais registros de uma entidade no banco.
        // se chamarmos assim: 'const users = await userRepository.find();' ele busca todos os usuários, retornando um array de User.
        // O TypeORM permite passar um objeto com opções dentro do find() para filtrar, ordenar ou incluir relações.
        /*
        O {} é um objeto de opções.
        
        Ele permite informar como a busca deve ser feita, por exemplo:
        
        relations → quais relações devem ser carregadas
        
        where → condições (where: { name: 'João' })
        
        order → ordenação (order: { name: 'ASC' })

        relations recebe uma lista de propriedades que representam relações entre entidades.
        
        No seu caso, ['posts'] diz: “ao buscar os usuários, carregue também todos os posts de cada usuário”.
        
        É um array porque você pode incluir várias relações ao mesmo tempo (poderia ser relations: ['posts', 'comments', 'profile'])
        */
        // find({ relations: ['posts'] }) → busca todos os usuários
        // e carrega também os posts de cada usuário (faz um JOIN automaticamente)
        const users = await this.userRepository.find({ relations: ['posts'] });

        // Retorna os usuários em formato JSON na resposta
        return res.json(users);
    }

    // Método que cria um novo usuário
    async create(req: Request, res: Response) {
        // Pega os dados do corpo da requisição
        const { name, email } = req.body;

        // Cria um novo objeto User (ainda não salva no banco)
        // Ele cria um objeto da entidade User (é como se usássemos new User() etc etc., só que pronto para ser usado no próximo método - que salva no banco de dados)
        const user = this.userRepository.create({ name, email });

        // Salva o usuário no banco de dados
        await this.userRepository.save(user);

        // Retorna o usuário criado com status 201 (Created)
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

routes.get('/users', userController.list);
routes.post('/users', userController.create);
routes.post('/posts', postController.create);

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
/*
  .initialize() é um método do ORM que inicia a conexão com o banco (que nem fazíamos com o createPool() da bilioteca do mysql2) e preparar todos os recursos antes de usar. Abre a conexão com o banco usando as configurações (host, porta, usuário, senha, banco), carrega as entidades (models/tabelas), executa sincronização (se synchronize: true estiver definido), que é o que cria as tabelas. Initialize é assíncrono, portanto retorna uma Promise. O que fica dentro de .then() é o que acontece se der certo, e o que fica no .catch() é o que acontece se houver erro.
*/
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
