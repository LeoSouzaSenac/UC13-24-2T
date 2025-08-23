# 🔑 10️⃣ Autenticação com JWT (JSON Web Token)

O **JWT (JSON Web Token)** é um padrão para transmitir informações seguras entre o cliente e o servidor como um objeto JSON **assinado digitalmente**. Ele é amplamente usado para **autenticação e autorização** em APIs REST.

Um **JWT** é um **código gerado pelo servidor** que diz quem você é. Ele é usado para **autenticar usuários** sem precisar ficar pedindo login toda hora.
Ele funciona como uma **chave de acesso** que prova quem está logado.

* Quando o usuário tenta acessar uma página privada, como `/perfil/3`, ele envia o token junto com a requisição.
* O servidor **verifica o token**.

  * Se for válido e mostrar que o usuário está logado, o acesso é liberado.
  * Se for inválido ou estiver faltando, o acesso é negado.
 
>"Ah mas eu só acesso uma determinada página se eu tiver email e senha. Pra que JWT?"
>Ok, você só acessa a página se souber login e senha. Certo. Mas imagine que você fez login e entrou na página do seu perfil. Agora, se alguém copiar a URL exata da sua página, essa pessoa não deveria conseguir acessar seus dados, certo? É aí que entra o JWT: ele funciona como uma chave de segurança. Mesmo que alguém tenha a URL, sem o token correto o servidor não libera o acesso às informações privadas. Assim, o JWT garante que apenas o usuário que está logado realmente consiga acessar suas próprias páginas e dados.

**Exemplo de token:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiZW1haWwiOiJhbGljZUBtYWlsLmNvbSJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

* Esse código é **único para o usuário**.
* Graças ao token, **apenas o usuário logado consegue acessar suas páginas e dados**, mesmo que alguém tente digitar URLs diferentes.

💡 **Resumo:** o JWT é a forma de o servidor saber “essa pessoa está logada e pode ver isso”, garantindo segurança nas rotas privadas.


Ele tem **três partes**:

1. **Header (cabeçalho)** – define o tipo e o algoritmo:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

2. **Payload (corpo)** – os dados que o token carrega, por exemplo:

```json
{
  "id": 1,
  "email": "alice@test.com"
}
```

3. **Signature (assinatura)** – garante que o token não foi alterado:

```
4lqH_XjG0ZxRz6J9dRZGzV5Z3HkFZ1K5P0mQ6mE1q1M
```

💡 **Resumo rápido:**

* O payload é quem você é (ID, email, etc).
* A assinatura garante que ninguém mexeu no token.
* Todo o token é **um código que você envia ao servidor para provar quem você é**.
* 
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
JWT_EXPIRES_IN=86400
```

> `JWT_SECRET` é a chave que será usada para **assinar o token**.
> `JWT_EXPIRES_IN` define **quanto tempo o token é válido** (ex: `1d` = 1 dia, `2h` = 2 horas, `86400` também é igual a 1 dia (86400 segundos = 1 dia)).

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
import { generateToken } from '../utils/jwt' // Importa a função que gera o JWT

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
      delete safe.password // Remove a senha antes de enviar os dados ao cliente
  
      // ===============================
      // GERANDO O TOKEN JWT
      // ===============================
      // Aqui estamos criando um token que contém o id e o email do usuário
      // Esse token será enviado ao cliente e usado para autenticação em outras rotas
      const token = generateToken({ id: user.id, email: user.email })
      // O token é um código seguro que representa o usuário logado.
      // Exemplo de token:
      // eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiZW1haWwiOiJhbGljZUBtYWlsLmNvbSIsImlhdCI6MTY5Mjk2MDAwMCwiZXhwIjoxNjkyOTY4MDAwfQ.RANDOMHASH
  
      // ===============================
      // ENVIANDO O TOKEN AO CLIENTE
      // ===============================
      // O cliente deve guardar esse token (normalmente em localStorage ou cookies)
      // e enviá-lo em cada requisição que precisa de autenticação
      // o backend só envia o token ao cliente, não guarda
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
import { verifyToken } from '../utils/jwt'

// Middleware para proteger rotas que exigem autenticação
export const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  // Pega o header de autorização da requisição
  const authHeader = req.headers.authorization

  // Se não houver header ou ele não começar com "Bearer ", retorna erro 401 (não autorizado)
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'Token não fornecido' })
  }

  // Extrai o token do header (remove o "Bearer " antes)
  const token = authHeader.split(' ')[1]

  // Verifica se o token é válido chamando a função verifyToken
  // Retorna o payload decodificado se válido, ou null se inválido/expirado
  const decoded = verifyToken(token)

  // Se o token for inválido, retorna erro 401
  if (!decoded) {
    return res.status(401).json({ message: 'Token inválido' })
  }

  // Armazena o payload decodificado no req para que outros middlewares ou controllers possam acessar
  // Ex.: req.user terá id e email do usuário logado
  (req as any).user = decoded

  // Chama o próximo middleware ou controller
  next()
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
import { authMiddleware } from '../middlewares/auth'

const router = Router()
const controller = new UserController()

router.get('/', authMiddleware, controller.list.bind(controller))
router.get('/:id', authMiddleware, controller.getById.bind(controller))
router.post('/', authMiddleware, controller.create.bind(controller))
router.put('/:id', authMiddleware, controller.update.bind(controller))
router.delete('/:id', authMiddleware, controller.remove.bind(controller))

export default router
```

> Agora, todas as requisições precisam enviar o token no cabeçalho `Authorization: Bearer <token>` para acessar as rotas protegidas.
