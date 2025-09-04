# Class-Validator no Backend TypeScript

O **class-validator** é uma biblioteca usada para **validar dados em runtime** no backend.  
Ela é muito utilizada junto com **DTOs (Data Transfer Objects)** para garantir que os dados que chegam à API estão corretos, completos e seguem regras específicas.

---

## 1️⃣ Por que usar class-validator

Mesmo usando TypeScript, os tipos só são verificados **em tempo de desenvolvimento**.  
Quando a API está rodando, qualquer cliente (Postman, frontend, app móvel, scripts) pode enviar dados inválidos.

💡 Exemplo de problema sem validação:

```ts
// req.body = { name: "", email: "leo.com", password: "123" }
const user = req.body;
console.log(user.name.toLowerCase()); // ❌ erro: name está vazio ou inválido
````

Com **class-validator**, você valida os dados antes que eles cheguem no controller ou no banco, evitando erros ou dados inconsistentes.

---

## 2️⃣ Instalação

```bash
npm install class-validator class-transformer
```

* `class-validator` → valida os dados
* `class-transformer` → transforma objetos comuns em instâncias de classes, necessárias para a validação

---

## 3️⃣ Decorators principais do class-validator

| Decorator          | Descrição                                              |
| ------------------ | ------------------------------------------------------ |
| `@IsNotEmpty()`    | Campo obrigatório, não pode ser vazio                  |
| `@IsEmail()`       | Verifica se o valor é um e-mail válido                 |
| `@MinLength(n)`    | Define tamanho mínimo da string                        |
| `@MaxLength(n)`    | Define tamanho máximo da string                        |
| `@IsInt()`         | Verifica se é número inteiro                           |
| `@Min(n)`          | Valor mínimo (para números)                            |
| `@Max(n)`          | Valor máximo (para números)                            |
| `@IsBoolean()`     | Verifica se é booleano                                 |
| `@IsOptional()`    | Campo opcional (não falha se estiver ausente)          |
| `@Matches(regex)`  | Valida se a string corresponde a uma expressão regular |
| `@IsDate()`        | Verifica se é uma data válida                          |
| `@IsArray()`       | Verifica se é um array                                 |
| `@ArrayMinSize(n)` | Tamanho mínimo de um array                             |
| `@ArrayMaxSize(n)` | Tamanho máximo de um array                             |

💡 Você pode combinar vários decorators em um mesmo campo.

---

## 4️⃣ Criando um DTO com validação

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

* As mensagens personalizadas ajudam a retornar **erros claros para o frontend**.

---

## 5️⃣ Middleware de validação

Para validar automaticamente os dados recebidos em uma rota, podemos criar um middleware genérico:

```ts
import { plainToInstance } from "class-transformer";
import { validate } from "class-validator";
import { Request, Response, NextFunction } from "express";

export function validateDTO(dtoClass: any) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const dtoObj = plainToInstance(dtoClass, req.body); // transforma em classe
    const errors = await validate(dtoObj); // valida

    if (errors.length > 0) {
      // retorna apenas as mensagens de erro
      return res.status(400).json(errors.map(err => err.constraints));
    }

    next();
  };
}
```

---

## 6️⃣ Usando o middleware no router

```ts
import { Router } from "express";
import { createUser } from "../controllers/userController";
import { CreateUserDTO } from "../dtos/CreateUserDTO";
import { validateDTO } from "../middlewares/validate";

const router = Router();

router.post("/users", validateDTO(CreateUserDTO), createUser);

export default router;
```

💡 Com isso, qualquer payload enviado para `/users` **será validado antes do controller**.
Campos ausentes, tipos incorretos ou valores inválidos retornarão **400 Bad Request**.

---

## 7️⃣ Exemplo prático de payload inválido

Se o cliente enviar:

```json
{
  "name": "",
  "email": "teste.com",
  "password": "123"
}
```

A API vai retornar:

```json
[
  { "name": "O nome é obrigatório" },
  { "email": "E-mail inválido" },
  { "password": "Senha deve ter no mínimo 6 caracteres" }
]
```

---

## 8️⃣ Dicas e boas práticas

1. Sempre use **DTOs + class-validator** para dados que vêm do cliente.
2. Combine **decorators** para criar regras complexas.
3. Use **mensagens personalizadas** para facilitar debug e UX.
4. Lembre-se: mesmo que o front-end valide, **o backend deve validar sempre**.

---

## 9️⃣ Referências

* [Documentação oficial do class-validator](https://github.com/typestack/class-validator)
* [Documentação do class-transformer](https://github.com/typestack/class-transformer)

Quer que eu faça isso?
```
