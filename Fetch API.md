# Guia: Entendendo o `fetch` em TypeScript

O `fetch` é uma função nativa do JavaScript (e também do TypeScript, já que ele compila para JavaScript) que permite **consumir APIs** e buscar dados pela internet. Ou seja, é através dessa funçãoq eu nós conseguimos utilizar alguma API, seja ela feita por outras pessoas ou a do nosso próprio backend.
É ele que "entra" nas rotas e faz as requisições.
Ele trabalha de forma **assíncrona**, ou seja, não trava a aplicação enquanto espera a resposta.

---

## Como o `fetch` funciona

Quando usamos `fetch`, ele retorna uma **Promise**. Isso significa que precisamos tratar o resultado de forma assíncrona, usando:

* `.then()` e `.catch()`
* ou `async/await`

Fluxo simplificado:

```
1. Você faz a requisição com fetch → fetch("url")
2. O navegador envia a requisição
3. O servidor responde
4. O fetch retorna uma Promise com a resposta
5. Você transforma a resposta em JSON → response.json()
6. Usa os dados no seu código
```

O **`fetch` é usado para…** 👇

1. **Comunicar o front-end com o back-end** (React → servidor).
2. **Consumir rotas RESTful** (GET, POST, PUT, DELETE).
3. **Trabalhar com APIs externas** (ex.: ViaCEP, GitHub API, OpenWeather).
4. **Integrar o site/app a serviços de terceiros** (pagamentos, mapas, redes sociais).

Resumindo: o `fetch` é a **ponte entre o navegador (ou React) e um servidor/API** 🚀.

---

## Exemplo 1: Usando `.then()`

```typescript
// Definindo o tipo esperado da API do ViaCEP
interface Endereco {
  cep: string;
  logradouro: string;
  bairro: string;
  localidade: string;
  uf: string;
}

fetch("https://viacep.com.br/ws/01001000/json/")
  .then((response) => {
    if (!response.ok) {
      throw new Error("Erro na requisição");
    }
    return response.json();
  })
  .then((data: Endereco) => {
    console.log("Endereço:", data);
  })
  .catch((error) => {
    console.error("Erro:", error);
  });
```

---

## Exemplo 2: Usando `async/await`

```typescript
interface Endereco {
  cep: string;
  logradouro: string;
  bairro: string;
  localidade: string;
  uf: string;
}

async function buscarEndereco(cep: string): Promise<void> {
  try {
    const response = await fetch(`https://viacep.com.br/ws/${cep}/json/`);

    if (!response.ok) {
      throw new Error("Erro na requisição");
    }

    const data: Endereco = await response.json();
    console.log("Endereço encontrado:", data);
  } catch (error) {
    console.error("Erro ao buscar CEP:", error);
  }
}

// Chamando a função
buscarEndereco("01001000");
```

---

## Exemplo 3: Enviando dados com `POST`

Além de buscar dados, o `fetch` também pode **enviar dados** para uma API.

```typescript
interface Usuario {
  nome: string;
  email: string;
}

async function cadastrarUsuario(usuario: Usuario): Promise<void> {
  try {
    const response = await fetch("https://exemplo.com/api/usuarios", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(usuario),
    });

    if (!response.ok) {
      throw new Error("Erro ao cadastrar usuário");
    }

    const data = await response.json();
    console.log("Usuário cadastrado com sucesso:", data);
  } catch (error) {
    console.error(error);
  }
}

cadastrarUsuario({ nome: "Maria", email: "maria@email.com" });
```

---

## Resumindo

* `fetch` é usado para **consumir APIs**.
* Ele retorna uma **Promise**.
* Podemos usar `.then()` ou `async/await`.
* Serve tanto para **buscar (GET)** quanto para **enviar dados (POST, PUT, DELETE)**.
