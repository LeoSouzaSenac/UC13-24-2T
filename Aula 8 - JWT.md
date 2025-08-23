# 🔑 10️⃣ Autenticação com JWT (JSON Web Token)

O **JWT (JSON Web Token)** é um padrão para transmitir informações seguras entre o cliente e o servidor como um objeto JSON **assinado digitalmente**. Ele é amplamente usado para **autenticação e autorização** em APIs REST.

---

## 🔹 Para que serve o JWT?

* Permite autenticar usuários sem armazenar sessão no servidor.
* O token gerado contém informações do usuário (payload) e uma assinatura que garante que o token não foi alterado.
* Pode ter tempo de expiração definido (`expiresIn`).
* Ao fazer requisições, o cliente envia o token no cabeçalho `Authorization: Bearer <token>`.

---

## 🔹 Como funciona no backend

1. O usuário faz login enviando **email** e **senha**.
2. O backend valida as credenciais.
3. Se válidas, o backend gera um **JWT** contendo o ID e o email do usuário.
4. O token é enviado para o cliente.
5. O cliente envia o token em todas as requisições protegidas.
6. O backend verifica o token antes de acessar rotas protegidas.

---

## 🔹 1️⃣ Instalar a biblioteca JWT

```bash
npm install jsonwebtoken
npm install --save-dev @types/jsonwebtoken
```

Adicionar variáveis no `.env`:

```
JWT_SECRET=minhaChaveSecreta123
JWT_EXPIRES_IN=1d
```

> `JWT_SECRET` é a chave que será usada para **assinar o token**.
> `JWT_EXPIRES_IN` define **quanto tempo o token é válido** (ex: `1d` = 1 dia, `2h` = 2 horas).

---

## 🔹 2️⃣ Criar função utilitária para gerar token

**src/utils/jwt.ts**

```ts
import jwt from 'jsonwebtoken'

interface Payload {
  id: number
  email: string
}

export const generateToken = (payload: Payload) => {
  return jwt.sign(payload, process.env.JWT_SECRET!, {
    expiresIn: process.env.JWT_EXPIRES_IN || '1d'
  })
}
```

> `jwt.sign` cria um token assinado.
> `payload` é o conteúdo do token (informações do usuário).
> `expiresIn` define o tempo de expiração.

---

## 🔹 3️⃣ Alterar AuthController para retornar token

**src/controllers/AuthController.ts**

```ts
import { Request, Response } from 'express'
import { UserService } from '../services/UserService'
import { generateToken } from '../utils/jwt'

const service = new UserService()

export class AuthController {
  async register(req: Request, res: Response) {
    try {
      const user = await service.create(req.body)
      const token = generateToken({ id: user.id, email: user.email })
      const safe: any = { ...user }
      delete safe.password
      res.status(201).json({ user: safe, token })
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

      const token = generateToken({ id: user.id, email: user.email })
      const safe: any = { ...user }
      delete safe.password

      res.json({ user: safe, token })
    } catch (e: any) {
      res.status(400).json({ message: e.message })
    }
  }
}
```

> Agora, ao registrar ou fazer login, o backend retorna o **JWT** junto com os dados do usuário.

---

## 🔹 4️⃣ Criar middleware para rotas protegidas

**src/middlewares/authMiddleware.ts**

```ts
import { Request, Response, NextFunction } from 'express'
import jwt from 'jsonwebtoken'

export interface AuthRequest extends Request {
  user?: any
}

export const authMiddleware = (req: AuthRequest, res: Response, next: NextFunction) => {
  const authHeader = req.headers.authorization
  if (!authHeader) return res.status(401).json({ message: 'Token não fornecido' })

  const [, token] = authHeader.split(' ')
  if (!token) return res.status(401).json({ message: 'Token inválido' })

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!)
    req.user = decoded
    next()
  } catch (err) {
    res.status(401).json({ message: 'Token inválido ou expirado' })
  }
}
```

> Esse middleware verifica se o token existe, é válido e não expirou.
> Se válido, adiciona o payload em `req.user` para que possamos usar em rotas.

---

## 🔹 5️⃣ Proteger rotas com JWT

Exemplo de rota protegida **GET /users**:

```ts
import { Router } from 'express'
import { UserController } from '../controllers/UserController'
import { authMiddleware } from '../middlewares/authMiddleware'

const router = Router()
const controller = new UserController()

router.get('/', authMiddleware, controller.list.bind(controller))
router.get('/:id', authMiddleware, controller.getById.bind(controller))
router.post('/', controller.create.bind(controller))
router.put('/:id', authMiddleware, controller.update.bind(controller))
router.delete('/:id', authMiddleware, controller.remove.bind(controller))

export default router
```

> Agora, todas as requisições precisam enviar o token no cabeçalho `Authorization: Bearer <token>` para acessar as rotas protegidas.
