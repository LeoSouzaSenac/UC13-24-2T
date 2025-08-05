# **Aula 3: Rotas, Requisições, Respostas e Middlewares no Express**

## 🎯 **Objetivos da Aula**

* Entender **o que são rotas** no Express.
* Compreender o conceito de **requisição e resposta**.
* Aprender a **estrutura de uma rota**.
* Introduzir **middlewares**, entender seu propósito e como usá-los.

---

## 🛫 **1. O que é uma Rota?**

No Express, uma **rota** define como a aplicação responde a uma **requisição HTTP** feita para um **endereço específico**.

### 🛤 **Analogia para entender melhor**

🔹 **Imagine que você está em uma cidade e precisa encontrar um restaurante.** Você abre um mapa (navegador) e pesquisa pelo nome do restaurante (URL). Ao clicar nele, você recebe informações como horário de funcionamento e cardápio (dados da resposta).

> No Express, o caminho que seu pedido segue é definido por **rotas**, e cada uma responde de maneira diferente dependendo do que você solicitar.

---

## 🔄 **2. O que é uma Requisição e uma Resposta?**

Sempre que interagimos com um site ou aplicativo, estamos enviando **requisições** (requests) e recebendo **respostas** (responses).

|           Termo          |                                          O que significa?                                          |
| :----------------------: | :------------------------------------------------------------------------------------------------: |
| **Requisição (Request)** |   É o pedido feito pelo usuário para o servidor. Pode conter dados como um formulário preenchido.  |
|  **Resposta (Response)** | É o que o servidor retorna para o usuário. Pode ser uma mensagem, um arquivo ou até mesmo um erro. |

---

### 🧪 **Exemplo no Express**

```ts
app.get('/saudacao', (req: Request, res: Response): Response => {
  return res.send('Olá, jovem programador!');
});
```

✅ O navegador **faz uma requisição GET para `/saudacao`**.
✅ O servidor **responde com a mensagem "Olá, jovem programador!"**.

---

## 🚦 **3. Como Criar Rotas no Express?**

Aqui está a estrutura básica de uma rota no Express:

```ts
app.metodo('/caminho', (req: Request, res: Response): Response => {
  return res.resposta();
});
```

---

### 🌍 **Principais Métodos HTTP**

|   Método   |            O que faz?            |     Exemplo     |
| :--------: | :------------------------------: | :-------------: |
|   **GET**  |      Busca dados do servidor     |   `/usuarios`   |
|  **POST**  |    Envia dados para o servidor   |   `/usuarios`   |
|   **PUT**  |    Atualiza um recurso inteiro   | `/usuarios/:id` |
|  **PATCH** | Atualiza parcialmente um recurso | `/usuarios/:id` |
| **DELETE** |         Remove um recurso        | `/usuarios/:id` |

---

### 🔧 **PUT vs PATCH – Qual a diferença?**

| Método | Atualiza tudo? | Usado para...             |
| ------ | -------------- | ------------------------- |
| PUT    | Sim            | Substituir todo o recurso |
| PATCH  | Não            | Alterar parte do recurso  |
---

## ✅ Diferença Principal

| Método    | Atualização  | Envia o recurso completo? | Substitui ou modifica?                 |
| --------- | ------------ | ------------------------- | -------------------------------------- |
| **PUT**   | **Completa** | **Sim**                   | **Substitui** o recurso inteiro        |
| **PATCH** | **Parcial**  | **Não**                   | **Modifica** apenas os campos enviados |

---

## 🔧 Exemplo prático

Imagine que temos um recurso **usuário** com esta estrutura:

```json
{
  "id": 1,
  "nome": "João",
  "email": "joao@email.com",
  "idade": 30
}
```

---

### ✅ PUT – Atualização Completa

Se você quiser atualizar o usuário usando `PUT`, você precisa enviar **o objeto inteiro** (mesmo que só um campo mude). Caso contrário, os campos ausentes podem ser apagados.

#### Requisição:

```http
PUT /usuarios/1
Content-Type: application/json

{
  "id": 1,
  "nome": "João Silva",
  "email": "joao@email.com",
  "idade": 30
}
```

#### Resultado:

Atualiza o usuário com os novos dados, **substituindo** o recurso original pelo novo.

---

### ✅ PATCH – Atualização Parcial

Com `PATCH`, você só precisa enviar **os campos que deseja alterar**.

#### Requisição:

```http
PATCH /usuarios/1
Content-Type: application/json

{
  "nome": "João Silva"
}
```

#### Resultado:

Apenas o campo `nome` será alterado, os outros permanecerão iguais.

---

## 🧠 Dica Final

* Use **PUT** quando quiser **substituir** o recurso inteiro.
* Use **PATCH** quando quiser **atualizar apenas alguns campos**.

---

## Status - o que são?
Status (ou códigos de status HTTP) são números que indicam o resultado de uma requisição feita a um servidor. Eles são enviados pelo servidor (por exemplo, uma API) como parte da resposta para dizer ao cliente (navegador, app, etc.) o que aconteceu com a requisição.

Por exemplo, se você conseguiu criar um novo usuário, o status é o 201. Se você tentou acessar uma rota que não existe (por ter digitado ela errado, por exemplo) o status é o 404.

### ATENÇÃO: O Express não "adivinha" o status correto, ele só usa o padrão (200) se você não informar nenhum. Ou seja, se você não indicar o status correto, ele sempre vai retornar o 200, seja para sucesso, seja erro, etc.

Portanto, **é responsabilidade do desenvolvedor** indicar o status adequado conforme o resultado da requisição (erro, sucesso, recurso criado, não encontrado, etc).

Exemplo básico:
```ts
// O status 200 diz: "A requisição foi bem-sucedida"
// O conteúdo (mensagem) é enviado junto (já convertido para o formato JSON, que é aceito praticamente por qualquer programa)
res.status(200).json({ mensagem: "Tudo certo!" });

```

### 📊 **Principais Status HTTP e Seus Significados**

|  Código |      Significado      |               Quando Usar?               |
| :-----: | :-------------------: | :--------------------------------------: |
| **200** |           OK          |    Quando a requisição é bem-sucedida    |
| **201** |        Created        |      Quando um novo recurso é criado     |
| **204** |       No Content      | Sucesso, mas sem resposta (ex: exclusão) |
| **400** |      Bad Request      |          Requisição mal formada          |
| **404** |       Not Found       |        Rota ou recurso não existe        |
| **500** | Internal Server Error |           Deu ruim no servidor           |

---

## 💡 **Exemplos Práticos com Dados**

```ts
// /usuarios/:id => Parâmetros de rota
app.get('/usuarios/:id', (req: Request, res: Response): Response => {
  const id = req.params.id;
  return res.send(`Usuário de ID: ${id}`);
});

// /buscar?termo=javascript => Query string
app.get('/buscar', (req: Request, res: Response): Response => {
  const termo = req.query.termo;
  return res.send(`Buscando por: ${termo}`);
});

// JSON no corpo (body)
app.post('/dados', (req: Request, res: Response): Response => {
  const { nome } = req.body;
  return res.send(`Nome recebido: ${nome}`);
});
```

---

## 🛠 **Exemplo completo com todas as rotas básicas**

```ts
import express, { Application, Request, Response } from 'express';

const app: Application = express();
const PORT: number = 3000;

app.use(express.json());

// 🔹 GET
app.get('/usuarios', (req: Request, res: Response): Response => {
  return res.status(200).json({ mensagem: 'Lista de usuários' });
});

// 🔹 POST
app.post('/usuarios', (req: Request, res: Response): Response => {
  const { nome } = req.body;
  if (!nome) return res.status(400).json({ mensagem: 'Nome é obrigatório!' });
  return res.status(201).json({ mensagem: `Usuário ${nome} criado com sucesso!` });
});

// 🔹 PUT
app.put('/usuarios/:id', (req: Request, res: Response): Response => {
  return res.status(200).json({ mensagem: 'Usuário atualizado completamente!' });
});

// 🔹 PATCH
app.patch('/usuarios/:id', (req: Request, res: Response): Response => {
  return res.status(200).json({ mensagem: 'Usuário atualizado parcialmente!' });
});

// 🔹 DELETE
app.delete('/usuarios/:id', (req: Request, res: Response): Response => {
  const { id } = req.params;
  if (!id) return res.status(400).json({ mensagem: 'ID não enviado' });
  return res.status(204).send(); // Sem conteúdo
});
```

---

## 🏗 **4. O que é um Middleware?**

Um **middleware** é uma função que roda **entre** a requisição e a resposta. Ele pode:

✔️ Modificar ou validar a requisição   
✔️ Bloquear ou liberar o acesso a certas rotas   
✔️ Adicionar logs, cabeçalhos, etc.    

---

### 🧙 **Analogias para facilitar**  

🔹 **O porteiro do seu prédio:** Antes de entrar, ele pode verificar se você é um morador ou um visitante.  

🔹 **Gandalf na ponte de Khazad-dûm (Senhor dos Anéis):**

  - Ele analisa quem está tentando passar *(requisição)*.  
  - Se for um membro da `Sociedade do Anel`, ele permite a passagem *(`next()`)*.  
  - Se for um `Balrog` *(requisição errada)*, ele bloqueia o caminho *(`res.status(403)` - acesso negado)*.

<div align="center">
    <img src="https://media.tenor.com/uUnfd6BfpEgAAAAC/you-shall-not-pass.gif" alt="Gandalf falando You Shall Not Pass">
    <p>
        Fonte: <em><a href="https://ar.inspiredpencil.com/pictures-2023/gandalf-you-shall-not-pass-gif" target="_blank">https://ar.inspiredpencil.com/pictures-2023/gandalf-you-shall-not-pass-gif</a></em>
    </p>
</div>

```ts
// Define um middleware chamado "porteiroMiddleware"
// Middleware é uma função que intercepta a requisição antes de ela chegar à rota final
// NextFunction é o tipo da função next(). Se você não chamar next(), a requisição fica presa no middleware e não chega na rota final
const porteiroMiddleware = (req: Request, res: Response, next: NextFunction) => {
  
  // Exibe no console o caminho da URL acessada na requisição
  console.log(`📢 Requisição recebida em: ${req.url}`);

  // Chama a função "next" para permitir que a requisição continue para o próximo middleware ou rota
  next();
};

// Aplica o middleware "porteiroMiddleware" de forma global
// Isso significa que ele será executado em **todas as requisições**, independentemente da rota
app.use(porteiroMiddleware);
```

---


## ❌ **Rota 404 para caminhos inválidos**

```ts
// Middleware que trata requisições que não bateram em nenhuma rota definida
app.use((req: Request, res: Response): Response => {
  
  // Retorna uma resposta com status HTTP 404 (Não Encontrado)
  // E envia um JSON com a mensagem personalizada
  return res.status(404).json({ mensagem: 'Rota não encontrada!' });
});

//Esse middleware não chama next(). Isso é proposital.

//Ele é executado por último, quando nenhuma das rotas anteriores foi atendida. Detalhe: ele deve ser criado por último, depois de todas as rotas.

//Ou seja: se o Express chegar até aqui, é porque nenhuma rota válida foi encontrada para a URL solicitada → então ele retorna um erro 404.
```

---

## 🧪 **5. Exercícios para Praticar**

1️⃣ **Crie uma rota GET `/sobre` que retorna um JSON com seu nome, idade e descrição.**

2️⃣ **Adicione um middleware que registre a hora exata da requisição no console. Pesquise que recursos (classes, interfaces, bibliotecas, métodos, etc) usar e faça comentários explicando seu uso.**

```
Requisição feita em: 2025-02-17T18:30:12.345Z
```

3️⃣ **Crie uma rota POST `/comentarios` que recebe um JSON com "texto".**

* Retorne 400 se o corpo da requisição estiver vazio
* Retorne 201 se recebido corretamente

4️⃣ **Crie uma rota DELETE `/comentarios/:id`**

* Retorne 204 ao excluir
* Retorne 400 se o ID não for enviado

5️⃣ (**Desafio bônus**) **Crie um middleware que bloqueie requisições feitas entre 00h e 06h. Pesquise que recursos (classes, interfaces, bibliotecas, métodos, etc) usar e faça comentários explicando seu uso.**


**Atenção: comente todos os seus códigos, pois uma surpresa o aguarda ao final. HA HA HA**

![Gif du mau](https://i.gifer.com/1aQu.gif)

---

## ✅ **Resumo da Aula**

✅ Entendemos **requisições, respostas e rotas** no Express    
✅ Aprendemos **os métodos HTTP mais comuns**    
✅ Diferenciamos **PUT vs PATCH**    
✅ Usamos e entendemos **middlewares** com analogias    
✅ Criamos rotas práticas e exercícios para fixar    
✅ Implementamos uma **rota fallback 404**    
