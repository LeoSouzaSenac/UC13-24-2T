# 🛠 Criando um Backend TypeScript + TypeORM do Zero com Bcrypt, MVC e .env

Este guia ensina passo a passo a criar um backend completo usando:

- Node.js + TypeScript  
- Express.js  
- TypeORM (MySQL ou outro banco)  
- Bcrypt para criptografia de senhas  
- MVC (Controllers, Services, Entities, Routes)  
- Variáveis de ambiente com `.env`  

As rotas incluídas serão:

- `POST /auth/register` – criar usuário  
- `POST /auth/login` – login com bcrypt  
- `GET /users` – listar todos  
- `GET /users/:id` – listar por id  
- `PUT /users/:id` – atualizar  
- `DELETE /users/:id` – deletar  

---

## 🔐 Sobre Bcrypt

**Bcrypt** é uma biblioteca para **criptografar senhas**, garantindo que elas não sejam armazenadas em texto puro no banco de dados. Ou seja, ao invés de aparecer 'senha123', ela é armazenada criptografada. 

- **Instalação**:

```bash
npm install bcrypt
npm install --save-dev @types/bcrypt
````

* **Uso básico**:

```ts
import bcrypt from 'bcrypt'

const password = 'minhaSenha123'
const saltRounds = 10 // saltRounds define quantas vezes o algoritmo de hashing vai gerar e aplicar o "salt" à sua senha antes de criar o hash final.
// salt é um valor aleatório adicionado à senha antes de gerar o hash. Ele garante que senhas iguais resultem em hashes diferentes, protegendo contra ataques. Quanto maior o número, mais seguro, mas também mais lento o cálculo do hash.
// Criar hash
const hash = await bcrypt.hash(password, saltRounds)

// Comparar senha
const isValid = await bcrypt.compare('minhaSenha123', hash) // compara a senha digitada com o hash
console.log(isValid) // true
```

> O bcrypt adiciona automaticamente um salt seguro ao hash, dificultando ataques de força bruta.

---

## 1️⃣ Iniciar o projeto

```bash
mkdir meu-backend
cd meu-backend
npm init -y
npm install express typeorm mysql2 reflect-metadata dotenv bcrypt cors
npm install --save-dev typescript ts-node-dev @types/node @types/express @types/bcrypt
```

Criar `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "ES2020",
    "moduleResolution": "Node",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src"]
}
```

---

## 2️⃣ Criar estrutura de pastas (MVC)

```
src/
 ├── app.ts
 ├── server.ts
 ├── data-source.ts
 ├── entities/
 │    └── User.ts
 ├── controllers/
 │    ├── AuthController.ts
 │    └── UserController.ts
 ├── services/
 │    └── UserService.ts
 ├── routes/
 │    ├── auth.routes.ts
 │    ├── user.routes.ts
 │    └── index.ts
 └── middlewares/
      └── errorHandler.ts
```

---

## 3️⃣ Configurar `.env`

Crie `.env`:

```
PORT=3000

DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=senha
DB_NAME=meubanco

BCRYPT_SALT_ROUNDS=10
```

---

## 4️⃣ Configurar TypeORM

**src/data-source.ts**

```ts
import 'reflect-metadata'
import { DataSource } from 'typeorm'
import { config } from 'dotenv'
import { User } from './entities/User'

config() // carrega variáveis do .env

export const AppDataSource = new DataSource({
  type: 'mysql',
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT) || 3306,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  entities: [User],
  synchronize: true, // cria tabelas automaticamente (apenas dev!)
  logging: false
})
```

---

## 5️⃣ Criar entidade User

**src/entities/User.ts**

```ts
import { Entity, PrimaryGeneratedColumn, Column, BeforeInsert, BeforeUpdate, CreateDateColumn, UpdateDateColumn } from 'typeorm'
import bcrypt from 'bcrypt'

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id!: number

  @Column({ length: 120 })
  name!: string

  @Column({ unique: true, length: 160 })
  email!: string

  @Column()
  password!: string

  @CreateDateColumn()
  createdAt!: Date

  @UpdateDateColumn()
  updatedAt!: Date

  // Antes de salvar ou atualizar, criptografa a senha
  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword() {
    // Evita re-hash se a senha já estiver hasheada
// this.password é a senha do usuário que será salva ou atualizada no banco
// O bcrypt gera hashes que sempre começam com $2
// se ela já começa com isso, significa que já foi criptografada e não precisamos criptografar de novo
    if (!this.password.startsWith('$2')) {
      // convertemos usando paseINT pois o tudo que vem do .env sempre é lido como string
      const rounds = parseInt(process.env.BCRYPT_SALT_ROUNDS || '10', 10)
// bcrypt.hash é uma função assíncrona
// usamos await pois o cálculo pode levar algum tempo dependendo do saltRounds
      this.password = await bcrypt.hash(this.password, rounds)
    }
  }

  // Método para comparar senha com hash
// primeiro retorna a promise, que depois será resolvida como boolean true ou false
  async validatePassword(plain: string): Promise<boolean> {
    return bcrypt.compare(plain, this.password)
  }
}
```

---

## 6️⃣ Criar UserService

**src/services/UserService.ts**

```ts
import { AppDataSource } from '../data-source'
import { User } from '../entities/User'

export class UserService {
  private repo = AppDataSource.getRepository(User)

// Função para criar um novo usuário
  async create(data: { name: string; email: string; password: string }) {
    // Verifica se já existe
    const exists = await this.repo.findOne({ where: { email: data.email } })
    if (exists) throw new Error('E-mail já cadastrado')
    const user = this.repo.create(data)
    return await this.repo.save(user)
  }

// Método para encontrar todos os usuários
async findAll() {
  // Busca todos os usuários no banco de dados usando o repositório do TypeORM
  const users = await this.repo.find()
  
  // map é um método de arrays que cria um novo array a partir de outro, aplicando uma função a cada elemento.
  // ele percorre todo o array e aplica a cada item do array uma função (que passamos como argumento do map)
  return users.map(u => {
    // Cria uma cópia do usuário atual (u) usando spread operator para não alterar o objeto original
    const clone: any = { ...u }

    // Remove a propriedade 'password' do clone, garantindo que a senha não será enviada na resposta
    delete clone.password

    // Retorna o clone sem senha, que será incluído no novo array gerado pelo map
    return clone
  })

  // O método retorna um array de usuários, todos sem a propriedade 'password'
}

  async findById(id: number) {
  // Busca um usuário no banco de dados pelo ID usando o repositório do TypeORM
  const user = await this.repo.findOne({ where: { id } })

  // Se nenhum usuário for encontrado, lança um erro
  if (!user) throw new Error('Usuário não encontrado')

  // Cria uma cópia do usuário usando spread operator para não alterar o objeto original
  const clone: any = { ...user }

  // Remove a propriedade 'password' do clone, garantindo que a senha não será enviada na resposta
  delete clone.password

  // Retorna o clone do usuário sem a senha
  return clone
}

  // Partial<User> indica que o objeto data pode ter qualquer propriedade do User, ou seja, não é obrigatório enviar todas. Por exemplo, quero atualizar apenas o nome, ou email e senha.
async update(id: number, data: Partial<User>) {
  // Busca o usuário pelo ID usando o repositório
  const user = await this.repo.findOne({ where: { id } })

  // Se nenhum usuário for encontrado, lança um erro
  if (!user) throw new Error('Usuário não encontrado')

  // Se data contém a senha, definimos explicitamente no usuário para que o hashPassword seja chamado
  if (data.password) {
    user.password = data.password
  }

  // Copia apenas as outras propriedades existentes em data para user, sem alterar a senha novamente
  const { password, ...rest } = data
  // Object.assign() é um método do Object que copia propriedades de um ou mais objetos para um objeto alvo.
  Object.assign(user, rest)

  // Antes de salvar, o TypeORM chama @BeforeUpdate() no User,
  // que cuida do hash da senha se ela estiver presente e não estiver hasheada
  return await this.repo.save(user)
}


  async remove(id: number) {
  // Busca o usuário pelo ID usando o repositório do TypeORM
  const user = await this.repo.findOne({ where: { id } })

  // Se nenhum usuário for encontrado, lança um erro
  if (!user) throw new Error('Usuário não encontrado')

  // Remove o usuário do banco de dados
  // TypeORM garante que apenas este registro será deletado
  await this.repo.remove(user)

  // Retorna uma mensagem de confirmação
  return { message: 'Usuário removido' }
}

  async findByEmail(email: string) {
  // Busca um usuário no banco de dados pelo email usando o repositório do TypeORM
  // Retorna o usuário inteiro (incluindo a senha), ou undefined se não existir
  return this.repo.findOne({ where: { email } })
}

```

---

## 7️⃣ Criar UserController

**src/controllers/UserController.ts**

```ts
import { Request, Response } from 'express'
import { UserService } from '../services/UserService'

const service = new UserService()

export class UserController {
  async create(req: Request, res: Response) {
    try {
      const user = await service.create(req.body)
      res.status(201).json(user)
    } catch (e: any) {
      res.status(400).json({ message: e.message })
    }
  }

  async list(req: Request, res: Response) {
    const users = await service.findAll()
    res.json(users)
  }

  async getById(req: Request, res: Response) {
    try {
      const user = await service.findById(Number(req.params.id))
      res.json(user)
    } catch (e: any) {
      res.status(404).json({ message: e.message })
    }
  }

  async update(req: Request, res: Response) {
    try {
      const user = await service.update(Number(req.params.id), req.body)
      res.json(user)
    } catch (e: any) {
      res.status(400).json({ message: e.message })
    }
  }

  async remove(req: Request, res: Response) {
    try {
      const result = await service.remove(Number(req.params.id))
      res.json(result)
    } catch (e: any) {
      res.status(404).json({ message: e.message })
    }
  }
}
```

---

## 8️⃣ Criar AuthController

**src/controllers/AuthController.ts**

```ts
import { Request, Response } from 'express'
import { UserService } from '../services/UserService'

const service = new UserService()

export class AuthController {
  async register(req: Request, res: Response) {
    try {
      const user = await service.create(req.body)
      res.status(201).json(user)
    } catch (e: any) {
      res.status(400).json({ message: e.message })
    }
  }

  async login(req: Request, res: Response) {
    try {
      const { email, password } = req.body
      const user = await service.findByEmail(email)
      if (!user) return res.status(404).json({ message: 'Usuário não encontrado' })

      const valid = await user.validatePassword(password)
      if (!valid) return res.status(401).json({ message: 'Senha inválida' })

      const safe: any = { ...user }
      delete safe.password

      res.json({ user: safe })
    } catch (e: any) {
      res.status(400).json({ message: e.message })
    }
  }
}
```

---

## 9️⃣ Criar rotas

**src/routes/user.routes.ts**

```ts
import { Router } from 'express'
import { UserController } from '../controllers/UserController'

const router = Router()
const controller = new UserController()

router.get('/', controller.list.bind(controller))
router.get('/:id', controller.getById.bind(controller))
router.post('/', controller.create.bind(controller))
router.put('/:id', controller.update.bind(controller))
router.delete('/:id', controller.remove.bind(controller))

export default router
```

---

```ts
// src/routes/auth.routes.ts
import { Router } from 'express'
import { AuthController } from '../controllers/AuthController'

const router = Router()
const controller = new AuthController()

router.post('/register', controller.register.bind(controller))
router.post('/login', controller.login.bind(controller))

export default router
```

---

```ts
// src/routes/index.ts
import { Router } from 'express'
import authRoutes from './auth.routes'
import userRoutes from './user.routes'

const router = Router()

router.use('/auth', authRoutes)
router.use('/users', userRoutes)

export default router
```

---

```ts
// src/app.ts
import express from 'express'
import { config } from 'dotenv'
import routes from './routes'

config()
const app = express()
app.use(express.json())
app.use(routes)

export default app
```

---

```ts
// src/server.ts
import app from './app'
import { AppDataSource } from './data-source'

const PORT = process.env.PORT || 3000

AppDataSource.initialize()
  .then(() => {
    app.listen(PORT, () => console.log(`Servidor rodando na porta ${PORT}`))
  })
  .catch(err => console.error('Erro ao conectar no banco:', err))
```





