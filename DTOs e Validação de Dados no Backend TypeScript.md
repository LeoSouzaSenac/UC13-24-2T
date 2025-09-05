# DTOs e Validação de Dados no Backend TypeScript

Neste material, vamos aprender o que são **DTOs (Data Transfer Objects)**, como aplicar **validação de dados**, e como ambos são usados para criar um backend **mais seguro, organizado e confiável**.  
Além disso, vamos ver **exemplos reais de problemas que podem acontecer** se não aplicarmos essas práticas.

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

💡 **Imagine o cenário:**
Um usuário poderia tentar enviar:

```json
{ "name": "", "email": "leo@", "password": "123" }

```

Sem validação, você iria salvar ou processar esses dados ruins.



---

## 2️⃣ O que é validação de dados?

**Validação de dados** é o processo de **verificar se os dados que chegam na API estão corretos e seguem as regras definidas**.

Ela ajuda a:

* Evitar que dados inválidos entrem no banco.
* Garantir que a API não quebre com payloads inesperados, que é quando o cliente envia algo diferente do que você espera (campos a mais no body, por exemplo)
* Dar respostas claras ao usuário quando algo está errado.
* Aumentar a **segurança** do backend.

💡 **Exemplo prático:**
Se alguém tentar criar um usuário com:

```json
{
  "name": "",
  "email": "leo.com",
  "password": "123"
}
```

A validação vai retornar:

* Nome obrigatório
* E-mail inválido
* Senha muito curta

Sem validação, o backend poderia **criar um usuário com dados incorretos**, ou até quebrar se algum campo for usado de forma inesperada.

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

## Por que usar **classes** e não **interfaces** para DTOs

### 1️⃣ Interfaces só existem em **tempo de desenvolvimento**

```ts
interface CreateUserDTO {
  name: string;
  email: string;
  password: string;
}
```

* ✅ TypeScript vai verificar os tipos enquanto você escreve o código.
* ❌ Mas **em runtime (quando a aplicação está rodando)**, a interface **não existe**, ou seja, não é possível inspecionar ou validar os dados recebidos.

💡 Exemplo prático:

```ts
// req.body = { name: "Leo", email: "leo@teste.com", password: "123456", isAdmin: true }
const user: CreateUserDTO = req.body;

console.log(user.isAdmin); // existe, mesmo que não esteja na interface!
```

Mesmo que você defina os tipos, **qualquer campo extra enviado pelo cliente ainda vai existir no objeto**, porque interfaces não protegem em runtime.

---

### 2️⃣ Classes existem em **runtime**

```ts
import { IsEmail, IsNotEmpty, MinLength } from "class-validator";

export class CreateUserDTO {
  @IsNotEmpty() name: string;
  @IsEmail() email: string;
  @MinLength(6) password: string;
}
```

* ✅ Bibliotecas como `class-validator` podem **inspecionar os objetos em runtime** e aplicar validações.
* ✅ Você ainda mantém os tipos do TypeScript.
* ✅ Campos extras enviados pelo cliente **não passam pela validação** e não entram no seu DTO.

💡 Exemplo prático:

Se o cliente enviar:

```json
{
  "name": "",
  "email": "leo.com",
  "password": "123",
  "isAdmin": true
}
```

* O middleware de validação vai retornar erro, indicando:

  * `name` obrigatório
  * `email` inválido
  * `password` muito curto
* O campo `isAdmin` **é ignorado**, garantindo que só os dados corretos entrem no sistema.

---

### 3️⃣ Quando usar interface mesmo assim

* **Interfaces** são úteis dentro da aplicação, para **tipar variáveis e contratos entre services e controllers**.
* Mas para **receber dados externos na API e validar**, **classe + validação é obrigatória**.

💡 Analogia:

* **Interface = contrato escrito no papel** (TypeScript)
* **Classe + validação = inspetor que verifica os dados reais antes de entrar no prédio**

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

💡 **Exemplo prático:**

* Usuário envia `{ name: "", email: "leo.com", password: "123" }`
* O middleware retorna:

```json
[
  { "name": "O nome é obrigatório" },
  { "email": "E-mail inválido" },
  { "password": "Senha deve ter no mínimo 6 caracteres" }
]
```

---

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

💡 **Exemplo prático:**

* Frontend envia `{ name: "", email: "teste@", password: "123" }`
* Middleware retorna erro com mensagens detalhadas, evitando que dados inválidos cheguem ao serviço.

---

## 6️⃣ Por que não basta apenas TypeScript?

TypeScript **valida tipos em tempo de compilação**, mas **não existe em runtime**.
Se alguém enviar dados diretamente via Postman, Insomnia, app móvel ou curl, **TypeScript sozinho não impede que dados errados entrem**.

* **TypeScript = proteção durante o desenvolvimento**
* **DTO + validação = proteção em runtime, quando a API já está rodando**

💡 Analogia:

* **TS = cinto de segurança enquanto você escreve o carro**
* **DTO + validação = airbag que protege quando o carro já está rodando**

---

## 7️⃣ Resumo prático

1. **DTO** → define o formato dos dados que entram/saem.
2. **Validação** → garante que os dados recebidos seguem o DTO.
3. **TypeScript sozinho não valida dados externos** → precisa de validação em runtime.
4. Bibliotecas recomendadas:

   * `class-validator` → ótimo com classes e decoradores.
   * `zod` → ótimo com funções e tipos TS.
5. Sempre **use DTO + validação** para manter a API **segura, organizada e confiável**.

---

## 8️⃣ Dica para os alunos

> Pense no DTO como o **contrato** e na validação como a **fiscalização do contrato**.
> Assim, nada de dados inesperados entra na sua aplicação, o backend fica robusto e os erros são tratados de forma clara.


---

### CreateUSerDTO.ts
```ts
// src/dtos/CreateUserDTO.ts
import { IsEmail, IsNotEmpty, Matches, MaxLength, MinLength } from "class-validator";

export class CreateUserDTO {
  @IsNotEmpty({ message: "O nome é obrigatório" })
  @Matches(/^[A-Za-zÀ-ÿ\s]+$/, { message: "Nome deve conter apenas letras e espaços" })
  @MaxLength(50, { message: "Nome deve ter no máximo 50 caracteres" })
  name: string;

  @IsEmail({}, { message: "E-mail inválido" })
  @MaxLength(100, { message: "E-mail deve ter no máximo 100 caracteres" })
  email: string;

  @MinLength(6, { message: "Senha deve ter no mínimo 6 caracteres" })
  @Matches(/(?=.*[a-z])/, { message: "Senha deve conter pelo menos uma letra minúscula" })
  @Matches(/(?=.*[A-Z])/, { message: "Senha deve conter pelo menos uma letra maiúscula" })
  @Matches(/(?=.*\d)/, { message: "Senha deve conter pelo menos um número" })
  @Matches(/(?=.*[@$!%*?&])/, { message: "Senha deve conter pelo menos um caractere especial (@$!%*?&)" })
  password: string;
}

```


### validateDTO.ts
```ts
// src/middlewares/validateDTO.ts

// Importa funções para transformar objetos e validar classes
import { plainToInstance } from "class-transformer"; // Converte objetos comuns (JSON) em instâncias de classes
import { validate } from "class-validator";         // Função que valida uma instância usando decorators
import { Request, Response, NextFunction } from "express"; // Tipos do Express

// Função que cria um middleware de validação para qualquer DTO
export function validateDTO(dtoClass: any) {
  // Retorna uma função compatível com Express (middleware)
  return async (req: Request, res: Response, next: NextFunction) => {
    // Converte o corpo da requisição (req.body) em uma instância da classe DTO
    // Isso é necessário para que o class-validator consiga ler os decorators (@IsEmail, @IsNotEmpty, etc.)
    const dtoObj = plainToInstance(dtoClass, req.body);

    // Valida a instância do DTO, é uma função do class-validator que verifica se está tudo como definimos
    // Retorna um array de erros, se houver algum campo inválido
    const errors = await validate(dtoObj);

    // Se houver erros, retorna resposta 400 (Bad Request) com mensagens
    if (errors.length > 0) {
        // O validate retorna um array de objetos, um para cada campo que falhou na validação.
        
      // errors.map(err => err.constraints) serve para extrair apenas as mensagens de erro, ignorando outras informações
      // 'constraints' contém mensagens de erro geradas pelos decorators do DTO
      // Exemplo: { name: "O nome é obrigatório", email: "E-mail inválido" }
      return res.status(400).json(errors.map(err => err.constraints));
    }

    // Se não houver erros, passa para o próximo middleware ou controller
    next();
  };
}
```
