# Consumindo APIs em React usando **ViaCEP**

> Neste guia prático, você vai criar do zero um app React simples onde o usuário digita um **CEP** e o app busca o **endereço** na API pública do **ViaCEP**. Você aprenderá, passo a passo, a usar `fetch` (a API nativa do navegador para requisições HTTP) e a tratar estados de **carregamento**, **sucesso** e **erro**.

---

## 📦 Pré-requisitos

* **Node.js** (versão LTS). Verifique com:

  ```bash
  node -v
  npm -v
  ```
* Conhecimento básico de **JavaScript** e **React** (componentes e hooks `useState`).

> **Por que não Create React App?** Vamos usar **Vite** com **npm**, que é leve e rápido para iniciar projetos React modernos.

---

## 🚀 1) Crie o projeto React com Vite

No terminal, execute:

```bash
npm create vite@latest via-cep-react -- --template react
cd via-cep-react
npm install
npm run dev
```

Abra o endereço mostrado no terminal (geralmente `http://localhost:5173`). Você verá a página inicial do Vite.

---

## 🧹 2) Limpe o projeto base

Vamos deixar o esqueleto do app pronto:

**Estrutura final esperada:**

```
via-cep-react/
├─ index.html
├─ package.json
├─ vite.config.js
└─ src/
   ├─ main.jsx
   ├─ App.jsx
   ├─ index.css
   └─ App.css
```

Substitua/crie os arquivos abaixo com o conteúdo indicado.

---

## 🧠 3) Antes de codar: o que é `fetch` e como funciona?

`fetch` é a **API nativa** dos navegadores para fazer **requisições HTTP** (GET, POST, etc.).

* **Retorno**: `fetch(url)` devolve uma **Promise** que resolve para um **objeto `Response`**.
* **Status HTTP**: você confere com `response.ok` (true para 2xx) e `response.status`.
* **Corpo da resposta**: converta para JSON com `await response.json()`.
* **Erros**: erros de rede disparam `catch`. Erros HTTP (404/500) **não** disparam `catch` automaticamente — por isso sempre cheque `response.ok`.

Fluxo típico:

```js
const url = `https://viacep.com.br/ws/${cep}/json/`;
const response = await fetch(url, { method: 'GET', headers: { Accept: 'application/json' } });
if (!response.ok) throw new Error(`HTTP ${response.status}`);
const data = await response.json();
```

> **ViaCEP** retorna `{ erro: true }` quando o CEP não existe. Vamos tratar isso no código.

---

## 🧩 4) Código do app

### `src/main.jsx`

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### `src/index.css`

```css
/* Reset simples + base */
* { box-sizing: border-box; }
html, body, #root { height: 100%; }
body {
  margin: 0;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Arial, "Apple Color Emoji", "Segoe UI Emoji";
  background: #0f172a; /* slate-900 */
  color: #e2e8f0;      /* slate-200 */
}

:root {
  --card: #111827;      /* gray-900 */
  --muted: #94a3b8;     /* slate-400 */
  --primary: #22c55e;   /* green-500 */
  --primary-700: #15803d;/* green-700 */
}

.container {
  max-width: 720px;
  margin: 48px auto;
  padding: 24px;
}

h1 {
  font-size: 28px;
  margin: 0 0 16px;
}

.subtitle { color: var(--muted); margin-bottom: 24px; }

.card {
  background: var(--card);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 10px 20px rgba(0,0,0,0.25);
}

.form { margin: 12px 0 8px; }
.label { display: block; font-size: 14px; color: var(--muted); margin-bottom: 6px; }

.inputRow { display: flex; gap: 8px; }
input[type="text"] {
  flex: 1;
  padding: 12px 14px;
  border-radius: 12px;
  border: 1px solid rgba(255,255,255,0.1);
  background: #0b1220; /* slate-950-ish */
  color: inherit;
  outline: none;
}
input[type="text"]:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(34,197,94,0.18);
}

button {
  padding: 12px 16px;
  border: none;
  border-radius: 12px;
  background: var(--primary);
  color: #052e16; /* green-950 */
  font-weight: 600;
  cursor: pointer;
}
button:disabled { opacity: .6; cursor: not-allowed; }
button:hover:not(:disabled) { background: var(--primary-700); color: #e2e8f0; }

.alert {
  margin-top: 12px;
  background: #3f1d1d;
  color: #fecaca; /* red-200 */
  border: 1px solid #7f1d1d; /* red-900 */
  padding: 10px 12px;
  border-radius: 12px;
}

.result { margin-top: 16px; }
.result ul { list-style: none; padding: 0; margin: 0; }
.result li { padding: 6px 0; border-bottom: 1px dashed rgba(255,255,255,0.06); }
.result li:last-child { border-bottom: none; }

footer { margin-top: 24px; color: var(--muted); }
footer a { color: var(--primary); text-decoration: none; }
footer a:hover { text-decoration: underline; }
```

### `src/App.jsx`

```jsx
import { useState } from 'react'  // Importa o hook useState do React, usado para criar estados na função
import './App.css'                // Importa o arquivo de estilos CSS para estilizar a página

export default function App() {
  // Lembrando:
  // 1º - variável que guarda o estado atual
  // 2º - função usada para atualizar o estado
  // 3º - dentro de useState, colocamos o valor inicial que o estado vai ter

  // Aqui criamos uma variável para guardar o valor do CEP e uma função que atualiza o CEP
  const [cep, setCep] = useState('')

  // Aqui criamos uma outra variável que guarda um endereço e uma função que atualiza esse endereço
  const [address, setAddress] = useState(null)

  // Função chamada sempre que o usuário digita algo no input
  function handleCepChange(e) {
    // Mantém apenas números e limita o CEP a 8 caracteres
    // e.target.value = valor digitado no input
    setCep(e.target.value.replace(/\D/g, '').slice(0, 8)) 
  }

  // Função chamada quando o usuário envia o formulário (clica no botão "Buscar")
  async function handleSubmit(e) {
    e.preventDefault()  // Evita que a página recarregue ao enviar o formulário
    setAddress(null)    // Limpa o endereço anterior, caso exista

    // Verifica se o CEP tem exatamente 8 dígitos
    if (cep.length !== 8) return // Se não tiver, simplesmente retorna e não faz nada

    try {
      // Faz a requisição para a URL da API do ViaCEP
      // A URL é construída concatenando o CEP digitado
      const response = await fetch(`https://viacep.com.br/ws/${cep}/json/`, {
        method: 'GET',                        // Método HTTP da requisição
        headers: { Accept: 'application/json' }, // Cabeçalhos da requisição: diz que esperamos JSON
      })

      // Converte a resposta da requisição em objeto JavaScript
      const data = await response.json()

      // Verifica se a API retornou erro (CEP inexistente)
      if (data.erro) return

      // Atualiza o estado com o endereço retornado
      setAddress(data)
    } catch (err) {
      // Captura qualquer erro que aconteça durante a requisição ou conversão
      console.error('Erro ao buscar CEP:', err)
    }
  }

  // JSX da interface: HTML dentro do React
  return (
    <div className="container"> {/* Container principal da página */}
      <h1>Busca CEP (ViaCEP)</h1> {/* Título */}
      <p className="subtitle">Digite um CEP e descubra o endereço completo.</p> {/* Subtítulo */}

      <div className="card"> {/* Card que envolve o formulário e resultados */}
        <form onSubmit={handleSubmit} className="form"> {/* Formulário que chama handleSubmit */}
          <label className="label" htmlFor="cep">CEP</label> {/* Label do input */}
          <div className="inputRow"> {/* Linha com input e botão */}
            <input
              id="cep"                            
              type="text"                         
              placeholder="Somente números (ex: 01001000)" 
              value={cep}                         // Valor controlado pelo estado
              onChange={handleCepChange}          // Chama handleCepChange ao digitar
              inputMode="numeric"                 // Sugere teclado numérico em dispositivos móveis
              autoFocus                            // Foca o input ao abrir a página
            />
            <button type="submit">
              Buscar {/* Texto do botão */}
            </button>
          </div>
        </form>

        {/* Mostra o endereço encontrado, caso exista */}
        {address && (
          <div className="result">
            <h2>Endereço</h2>
            <ul>
              <li><strong>Logradouro:</strong> {address.logradouro || '—'}</li>
              <li><strong>Bairro:</strong> {address.bairro || '—'}</li>
              <li><strong>Cidade:</strong> {address.localidade || '—'}</li>
              <li><strong>UF:</strong> {address.uf || '—'}</li>
              <li><strong>CEP:</strong> {address.cep || cep}</li>
              <li><strong>DDD:</strong> {address.ddd || '—'}</li>
            </ul>
          </div>
        )}

        {/* Footer com crédito da API */}
        <footer>
          <small>
            Dados por <a href="https://viacep.com.br" target="_blank" rel="noreferrer">ViaCEP</a>
          </small>
        </footer>
      </div>
    </div>
  )
}

```

### `src/App.css` (opcional, aqui vazio só para exemplo)

```css
/* Você pode colocar estilos específicos do App aqui, se quiser. */
```

> **Teste rápido:** use o CEP `01001000` (Praça da Sé – SP) para ver o resultado.

---

## 🧪 5) Entendendo o ciclo completo da busca

1. **Usuário digita** o CEP no input (aceitamos apenas dígitos e limitamos a 8).
2. Ao **enviar o formulário**, chamamos `handleSubmit` e **validamos** o CEP.
3. Chamamos `fetch` para `https://viacep.com.br/ws/CEP/json/`.
4. Checamos `response.ok` e depois convertemos com `response.json()`.
5. Se vier `{ erro: true }`, exibimos **“CEP não encontrado.”**
6. No sucesso, **salvamos** o objeto no estado `address` e **renderizamos** a lista.
7. Sempre controlamos `loading` para desabilitar o botão e mostrar feedback.

---

## 🧯 6) Tratamento de erros — boas práticas

* **Valide** o CEP antes da requisição (8 dígitos).
* **`try/catch`** para capturar erros de rede.
* **Checar `response.ok`** para tratar 4xx/5xx.
* A API ViaCEP pode responder com **`{ erro: true }`** — trate esse caso.
* Opcional: implemente **timeout** com `AbortController` se quiser limitar o tempo de espera.

Exemplo de timeout curto (opcional):

```js
const controller = new AbortController()
const id = setTimeout(() => controller.abort(), 8000)
const response = await fetch(url, { signal: controller.signal })
clearTimeout(id)
```

---

## 🧭 7) Dicas de UX

* **Máscara** visual para CEP: `00000-000` (a lógica interna pode seguir usando apenas dígitos).
* **Enter** envia o formulário (já suportado via `<form onSubmit={...}>`).
* **Desabilite** o botão durante o carregamento.
* **Mensagens claras** para “CEP inválido”, “não encontrado” e erro de rede.

---

## 🧩 8) Como a resposta do ViaCEP chega (exemplo)

Para `01001000` a estrutura é semelhante a:

```json
{
  "cep": "01001-000",
  "logradouro": "Praça da Sé",
  "complemento": "lado ímpar",
  "bairro": "Sé",
  "localidade": "São Paulo",
  "uf": "SP",
  "ibge": "3550308",
  "gia": "1004",
  "ddd": "11",
  "siafi": "7107"
}
```

> Quando o CEP não existe, a API retorna `{ "erro": true }`.

---

## 🧱 9) Perguntas frequentes (FAQ)

**• Preciso de chave de API?**
Não. O ViaCEP é público e não requer autenticação.

**• Posso usar com `axios`?**
Sim, mas aqui focamos em `fetch`. Com axios, seria `axios.get(url)` e `response.data`.

**• E CORS?**
O ViaCEP envia os cabeçalhos de CORS necessários. Outras APIs podem não enviar — nesse caso, seria preciso um proxy ou ajustar a origem no servidor.

**• Posso buscar automaticamente ao digitar 8 dígitos?**
Sim. Você pode disparar a busca no `onChange` quando `cep.length === 8`.

---

## 🧠 10) Próximos passos / Desafios

* Criar um **hook** `useViaCep(cep)` que encapsula a lógica de busca.
* Adicionar **máscara** `00000-000` no input e normalizar para 8 dígitos internamente.
* Mostrar um **skeleton** ou spinner durante `loading`.
* Salvar **histórico** de CEPs buscados no `localStorage`.
* Cobrir com **testes** (ex.: React Testing Library) mockando `fetch`.

---

## ✅ Conclusão

Você acabou de consumir uma API real no React usando `fetch`, tratar respostas e exibir o resultado de forma amigável. Essa base é a mesma para **qualquer** API REST: montar a URL, chamar `fetch`, verificar `response.ok`, converter com `response.json()` e renderizar os dados.

> Dica final: crie um componente reutilizável para o **Campo CEP** (com máscara e validação) e use-o em outros formulários do seu projeto.
