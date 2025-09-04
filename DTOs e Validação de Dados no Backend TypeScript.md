# DTOs e Validação de Dados no Backend TypeScript

Neste material, vamos aprender o que são **DTOs (Data Transfer Objects)**, como aplicar **validação de dados**, e como ambos são usados para criar um backend **mais seguro, organizado e confiável**.

---

## 1️⃣ O que é um DTO?

**DTO (Data Transfer Object)** é um **objeto que define o formato dos dados** que entram ou saem da sua API.  

Ele serve para:

- Padronizar o que a API **aceita como entrada** (requests).  
- Padronizar o que a API **retorna como saída** (responses).  
- Evitar vazamento de informações sensíveis do banco de dados.  
- Facilitar a comunicação entre front-end e back-end.

### Exemplo DTO de entrada (criação de usuário):

```ts
// src/dtos/CreateUserDTO.ts
export class CreateUserDTO {
  name: string;
  email: string;
  password: string;
}
````

### Exemplo DTO de saída (resposta ao front-end):

```ts
// src/dtos/UserResponseDTO.ts
export class UserResponseDTO {
  id: number;
  name: string;
  email: string;
}
```

---

## 2️⃣ O que é validação de dados?

**Validação de dados** é o processo de **verificar se os dados que chegam na API estão corretos e seguem as regras definidas**.

Ela ajuda a:

* Evitar que dados inválidos entrem no banco.
* Garantir que a API não quebre com payloads inesperados.
* Dar respostas claras ao usuário quando algo está errado.
* Aumentar a **segurança** do backend.

---

## 3️⃣ Como DTO e validação se relacionam

* O **DTO define a forma e os campos** (contrato).
* A **validação garante que os dados respeitem essa forma e regras**.

💡 Resumindo:

| Conceito  | Função                                                        |
| --------- | ------------------------------------------------------------- |
| DTO       | Define **quais campos e tipos** a API espera                  |
| Validação | Garante que os **dados enviados respeitam o contrato** do DTO |

---

## 4️⃣ Validação com `class-validator`

### Instalação

```bash
npm install class-validator class-transformer
```

### DTO com validação:

```ts
import { IsEmail, IsNotEmpty, MinLength } from "class-validator";

export class CreateUserDTO {
  @IsNotEmpty({ message: "O nome é obrigatório" })
  name: string;

  @IsEmail({}, { message: "E-mail inválido" })
  email: string;

  @MinLength(6, { message: "Senha deve ter no mínimo 6 caracteres" })
  password: string;
}
```

### Middleware de validação:

```ts
import { plainToInstance } from "class-transformer";
import { validate } from "class-validator";
import { Request, Response, NextFunction } from "express";

export function validateDTO(dtoClass: any) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const dtoObj = plainToInstance(dtoClass, req.body);
    const errors = await validate(dtoObj);

    if (errors.length > 0) {
      return res.status(400).json(errors.map(err => err.constraints));
    }

    next();
  };
}
```

### Usando no controller:

```ts
import { Router } from "express";
import { createUser } from "../controllers/userController";
import { CreateUserDTO } from "../dtos/CreateUserDTO";
import { validateDTO } from "../middlewares/validate";

const router = Router();

router.post("/users", validateDTO(CreateUserDTO), createUser);

export default router;
```

---

## 5️⃣ Validação com `zod`

### Instalação

```bash
npm install zod
```

### Schema e tipo TypeScript:

```ts
import { z } from "zod";

export const createUserSchema = z.object({
  name: z.string().min(1, "O nome é obrigatório"),
  email: z.string().email("E-mail inválido"),
  password: z.string().min(6, "Senha deve ter no mínimo 6 caracteres"),
});

export type CreateUserDTO = z.infer<typeof createUserSchema>;
```

### Middleware de validação:

```ts
import { Request, Response, NextFunction } from "express";
import { ZodSchema } from "zod";

export function validateSchema(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json(result.error.format());
    }

    req.body = result.data;
    next();
  };
}
```

### Usando no controller:

```ts
import { Router } from "express";
import { createUser } from "../controllers/userController";
import { createUserSchema } from "../schemas/userSchema";
import { validateSchema } from "../middlewares/validateSchema";

const router = Router();

router.post("/users", validateSchema(createUserSchema), createUser);

export default router;
```

---

## 6️⃣ Resumo prático

1. **DTO** → define o formato dos dados que entram/saem.
2. **Validação** → garante que os dados recebidos seguem o DTO.
3. Bibliotecas recomendadas:

   * `class-validator` → ótimo com classes e decoradores.
   * `zod` → ótimo com funções e tipos TS.
4. Sempre **use DTO + validação** para manter sua API **segura, organizada e confiável**.

---

## 7️⃣ Dica para os alunos

> Pense no DTO como o **contrato** e na validação como a **fiscalização do contrato**.
> Assim, nada de dados inesperados entra na sua aplicação, e o backend fica mais robusto.

---

