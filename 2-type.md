# 2 - Use type para Tipos Complexo

## 1. Tipo para Representar um Objeto de Usuário
Em vez de repetir a estrutura de um objeto de usuário em vários lugares, crie um type reutilizável.

```typescript
type User = {
  id: number;
  name: string;
  email?: string; // Campo opcional
};

function printUser(user: User): void {
  console.log(`ID: ${user.id}, Name: ${user.name}`);
}

const newUser: User = {
  id: 1,
  name: "Alice",
};

printUser(newUser);
```

### Retorno de APIs

Quando sua aplicação consome uma API que retorna objetos do tipo User, o tipo pode ser reutilizado para tipar os dados recebidos.

```typescript
async function fetchUser(userId: number): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  return response.json(); // Garante que o retorno seja do tipo User
}

fetchUser(1).then((user) => {
  console.log(user.name); // TypeScript garante que 'name' existe no objeto
});
```

### Listas de Usuários

Se você estiver lidando com múltiplos usuários, o tipo User pode ser usado para tipar arrays de usuários.

```typescript
const users: User[] = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob" },
];

users.forEach((user) => printUser(user));
```

### Campos Opcionais

O campo email foi marcado como opcional (?) para refletir casos em que nem todos os usuários possuem e-mail (ex.: convidados). Isso é útil em sistemas onde diferentes tipos de usuários podem ter atributos variáveis.

```typescript
const guest: User = { id: 3, name: "Guest" }; // Sem e-mail
const registeredUser: User = { id: 4, name: "Registered", email: "user@example.com" };

console.log(guest.email); // undefined
console.log(registeredUser.email); // "user@example.com"
```

### Uso em Componentes Frontend
Em aplicações React, você pode usar o tipo User para tipar as props de um componente.

```typescript
type User = {
  id: number;
  name: string;
  email?: string;
};

type UserCardProps = {
  user: User;
};

function UserCard({ user }: UserCardProps): JSX.Element {
  return (
    <div>
      <h2>{user.name}</h2>
      {user.email && <p>Email: {user.email}</p>}
    </div>
  );
}

const user: User = { id: 1, name: "Alice", email: "alice@example.com" };
<UserCard user={user} />;
```

### Uso em Sistemas de Autenticação
O tipo User pode ser usado para representar o usuário autenticado no sistema.

```typescript
function getAuthenticatedUser(): User | null {
  const isAuthenticated = true; // Exemplo fictício
  return isAuthenticated
    ? { id: 1, name: "Alice", email: "alice@example.com" }
    : null;
}

const authenticatedUser = getAuthenticatedUser();
if (authenticatedUser) {
  console.log(`Welcome, ${authenticatedUser.name}`);
} else {
  console.log("User not authenticated");
}
```

### Benefícios no Longo Prazo

#### Escalabilidade:

Se a aplicação crescer e você adicionar mais campos ao objeto User (ex.: role, createdAt, etc.), o tipo centralizado será automaticamente atualizado em todas as partes da aplicação.

#### Integração com Outras Bibliotecas:

Bibliotecas como Redux, React Query ou Apollo Client podem usar o tipo User para tipar estados, consultas ou resultados de mutações.

#### Colaboração em Equipe:

Em equipes, o uso de tipos bem definidos evita mal-entendidos sobre a estrutura dos dados, facilitando a colaboração.


## 2. Tipo para Representar Respostas de API
Se você trabalha com APIs, pode criar tipos para representar diferentes respostas.

```typescript
type ApiResponse<T> = {
  data: T;
  status: number;
  error?: string;
};

type UserData = {
  id: number;
  name: string;
};

const response: ApiResponse<UserData> = {
  data: { id: 1, name: "Alice" },
  status: 200,
};
```

Ao trabalhar com APIs, é comum que as respostas retornadas sigam um padrão consistente. Usar tipos para representar essas respostas ajuda a organizar o código, garantir a segurança de tipos e simplificar o consumo das APIs.

No exemplo fornecido:

ApiResponse<T> é um tipo genérico que representa a estrutura padrão de uma resposta da API.
UserData é o tipo específico de dados retornados pela API neste caso.
Este padrão permite reutilizar a estrutura ApiResponse para diferentes tipos de dados, como UserData, ProductData, ou outros.

### Como Aplicar no Mundo Real?

#### Definição de um Tipo Genérico de Resposta

O tipo genérico ApiResponse<T> tem três campos principais:

- data: Os dados retornados pela API (dinâmicos, definidos por T).
- status: O código de status HTTP da resposta (ex.: 200, 404, 500).
- error: Uma mensagem de erro opcional, caso a API retorne um problema.

```typescript
type ApiResponse<T> = {
  data: T;
  status: number;
  error?: string;
};
```

##### Resposta com Dados de Usuário

Imagine uma API que retorna informações de um usuário:

```typescript
type UserData = {
  id: number;
  name: string;
  email: string;
};

const userResponse: ApiResponse<UserData> = {
  data: { id: 1, name: "Alice", email: "alice@example.com" },
  status: 200,
};

console.log(userResponse.data.name); // "Alice"
```

##### Resposta com Lista de Produtos

Outra API pode retornar uma lista de produtos. Você reutiliza o mesmo tipo genérico:

```typescript
type Product = {
  id: number;
  name: string;
  price: number;
};

const productResponse: ApiResponse<Product[]> = {
  data: [
    { id: 1, name: "Laptop", price: 1500 },
    { id: 2, name: "Smartphone", price: 700 },
  ],
  status: 200,
};

console.log(productResponse.data[0].name); // "Laptop"
```

##### Resposta de Erro

Caso a API retorne um erro, o campo error é preenchido:

```typescript
const errorResponse: ApiResponse<null> = {
  data: null,
  status: 404,
  error: "User not found",
};

console.log(errorResponse.error); // "User not found"
```

##### Consumindo APIs com Funções Assíncronas

###### Definir uma Função Genérica

Crie uma função que consuma uma API e use o tipo genérico ApiResponse<T> para tipar o retorno.

```typescript
async function fetchApi<T>(url: string): Promise<ApiResponse<T>> {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return { data, status: response.status };
  } catch (error) {
    return { data: null as any, status: 500, error: error.message };
  }
}
```

Agora, você pode usar essa função para consumir qualquer endpoint.

Consumir a API de usuários:

```typescript
async function getUser(userId: number): Promise<ApiResponse<UserData>> {
  return fetchApi<UserData>(`/api/users/${userId}`);
}

getUser(1).then((response) => {
  if (response.status === 200) {
    console.log(response.data?.name); // "Alice"
  } else {
    console.error(response.error); // Possível mensagem de erro
  }
});
```

Consumir a API de produtos:

```typescript
async function getProducts(): Promise<ApiResponse<Product[]>> {
  return fetchApi<Product[]>("/api/products");
}

getProducts().then((response) => {
  if (response.status === 200) {
    console.log(response.data?.map((product) => product.name)); // ["Laptop", "Smartphone"]
  }
});
```

##### Integração com Ferramentas Externas
O tipo ApiResponse também pode ser usado para integrar ferramentas como Redux ou React Query:

Com Redux:

```typescript
type FetchUsersAction = {
  type: "FETCH_USERS_SUCCESS";
  payload: ApiResponse<UserData[]>;
};

const action: FetchUsersAction = {
  type: "FETCH_USERS_SUCCESS",
  payload: {
    data: [
      { id: 1, name: "Alice", email: "alice@example.com" },
      { id: 2, name: "Bob", email: "bob@example.com" },
    ],
    status: 200,
  },
};
```

Com React Query:

```typescript
import { useQuery } from "react-query";

function useUsers() {
  return useQuery<ApiResponse<UserData[]>>("users", () =>
    fetchApi<UserData[]>("/api/users")
  );
}
const { data, error } = useUsers();
console.log(data?.data); // Lista de usuários
```

***Eu pessoalmente prefiro usar o ReactQuery.***

### Por que usar tipos para respostas de API?

#### Consistência:

APIs geralmente retornam respostas no mesmo formato (ex.: status, dados, mensagens de erro). Um tipo padrão para respostas ajuda a padronizar como o código manipula esses dados.

#### Reutilização:

Usar tipos genéricos evita duplicação de código. O mesmo tipo ApiResponse<T> pode ser reutilizado para diferentes tipos de dados.

#### Segurança de Tipos:

Garantir que os dados da API sejam usados corretamente. Se o tipo esperado for diferente, o TypeScript exibirá um erro.

#### Melhoria na Manutenção:

Se o formato da resposta da API mudar, é mais fácil atualizar o tipo e propagar as alterações por toda a aplicação.

#### Facilidade no Debugging:

Saber exatamente quais campos estão disponíveis em cada resposta ajuda a depurar mais rapidamente.




## 3. Tipo para Funções com Callbacks
Facilite a criação de funções com callbacks complexos usando type.

```typescript
type Callback<T> = (error: Error | null, result: T | null) => void;

function fetchData(callback: Callback<string>): void {
  setTimeout(() => {
    callback(null, "Data fetched successfully");
  }, 1000);
}

fetchData((error, result) => {
  if (error) {
    console.error(error.message);
  } else {
    console.log(result);
  }
});
```

## Explicando o Exemplo

### O Tipo Callback<T>

O tipo Callback<T> representa uma função que:

Recebe dois parâmetros:
error: Um objeto de erro (Error) ou null se não houver erro.
result: Um valor genérico T ou null se houver erro.
Não retorna nada (void), pois seu propósito é lidar com o fluxo da função chamadora.
```typescript
type Callback<T> = (error: Error | null, result: T | null) => void;
```

### Função fetchData

A função fetchData simula uma operação assíncrona (como buscar dados de uma API). Ela:

Recebe um callback do tipo Callback<string> para lidar com o resultado.
Usa setTimeout para simular um atraso de 1 segundo.
Chama o callback com:
null como erro (indicando sucesso).
"Data fetched successfully" como resultado.
```typescript
function fetchData(callback: Callback<string>): void {
  setTimeout(() => {
    callback(null, "Data fetched successfully");
  }, 1000);
}
```

### Uso de fetchData

A função fetchData é chamada com um callback que:

- verifica se houve um erro:
  - se error não for null, registra a mensagem de erro no console.
- lida com o resultado:
  - se result não for null, exibe o resultado no console.

```typescript
fetchData((error, result) => {
  if (error) {
    console.error(error.message);
  } else {
    console.log(result); // "Data fetched successfully"
  }
});
```

### Por que isso é útil?

#### Reutilização:

O tipo Callback<T> pode ser usado em várias funções que seguem a mesma estrutura de erro e resultado.
Padronização:

Todas as funções que utilizam callbacks seguem uma assinatura consistente, facilitando o entendimento do código.

#### Segurança de Tipos:

O TypeScript garante que:

- o error é sempre do tipo Error | null.
- o result é sempre do tipo genérico T | null.
- isso previne erros de implementação.

#### Legibilidade:

O tipo nomeado (Callback<T>) torna a assinatura do callback mais clara e fácil de ler.

## Processamento de Arquivos
Simule a leitura de um arquivo que usa callbacks para retornar o conteúdo ou um erro.

```typescript
type FileCallback = Callback<string>;

function readFile(path: string, callback: FileCallback): void {
  setTimeout(() => {
    if (path === "invalid.txt") {
      callback(new Error("File not found"), null);
    } else {
      callback(null, "File content: Hello, World!");
    }
  }, 1000);
}

// Uso
readFile("example.txt", (error, content) => {
  if (error) {
    console.error("Error reading file:", error.message);
  } else {
    console.log(content); // "File content: Hello, World!"
  }
});
```

## Outros exemplos:

### Chamadas de API
Implemente uma função que faz chamadas de API e retorna os dados ou um erro.

```typescript
type ApiResponse = {
  status: number;
  data: any;
};

type ApiCallback = Callback<ApiResponse>;

function fetchApi(endpoint: string, callback: ApiCallback): void {
  setTimeout(() => {
    if (endpoint === "/error") {
      callback(new Error("API endpoint not found"), null);
    } else {
      callback(null, { status: 200, data: { message: "Success!" } });
    }
  }, 1000);
}

// Uso
fetchApi("/data", (error, response) => {
  if (error) {
    console.error("API Error:", error.message);
  } else {
    console.log("API Response:", response?.data.message); // "Success!"
  }
});
```

### Eventos com Callbacks
Implemente um sistema que escute eventos e use callbacks para notificar os resultados.

```typescript
type EventCallback = Callback<Event>;

function listenToEvent(eventName: string, callback: EventCallback): void {
  const event = { type: eventName, timestamp: Date.now() }; // Simula um evento
  if (eventName === "error") {
    callback(new Error("Event type not supported"), null);
  } else {
    callback(null, event);
  }
}

// Uso
listenToEvent("click", (error, event) => {
  if (error) {
    console.error("Error in event:", error.message);
  } else {
    console.log(`Event occurred: ${event?.type} at ${event?.timestamp}`);
  }
});
```

### Fila de Tarefas
Implemente um sistema que processa tarefas em uma fila e retorna o resultado ou um erro.

```typescript
type Task = { id: number; name: string };
type TaskCallback = Callback<Task>;

function processTask(task: Task, callback: TaskCallback): void {
  setTimeout(() => {
    if (task.id === 0) {
      callback(new Error("Invalid task ID"), null);
    } else {
      callback(null, task);
    }
  }, 500);
}

// Uso
processTask({ id: 1, name: "Send Email" }, (error, task) => {
  if (error) {
    console.error("Task Error:", error.message);
  } else {
    console.log("Task Processed:", task?.name); // "Send Email"
  }
});
```

### Benefícios:

#### Consistência: 

Todas as funções que utilizam callbacks seguem um padrão claro e previsível.

#### Reutilização: 

O mesmo tipo pode ser usado em diferentes funções.

#### Segurança de Tipos: 

O TypeScript verifica a conformidade com o tipo definido.

#### Facilidade de Depuração: 

Saber que o callback sempre segue o mesmo padrão facilita o rastreamento de problemas.

#### Clareza: 

O código é mais legível e compreensível para outros desenvolvedores.


## 4. Tipo para Unir Vários Tipos (Union)
O uso de Union Types (|) em TypeScript permite combinar vários tipos possíveis em um único tipo. No exemplo, PaymentMethod representa uma união de strings, cada uma correspondendo a um método de pagamento válido.

Esse recurso é especialmente útil quando:

- o valor de uma variável ou argumento pode ser de vários tipos, mas precisa ser restrito a opções específicas.
- você deseja evitar o uso de valores não válidos (ex.: métodos de pagamento inexistentes).
- quer facilitar a legibilidade e garantir segurança de tipos ao desenvolver.

### Métodos de Pagamento em um Sistema de E-commerce

Em sistemas de e-commerce, métodos de pagamento são limitados a opções específicas (ex.: cartão de crédito, PayPal, transferência bancária). O uso de PaymentMethod garante que apenas métodos válidos sejam utilizados.

```typescript
type PaymentMethod = "credit_card" | "paypal" | "bank_transfer";

function processPayment(method: PaymentMethod): void {
  console.log(`Processing payment with: ${method}`);
}

processPayment("credit_card");
// processPayment("cash"); // Erro: Argumento não é um PaymentMethod válido.
```

### Estados de Pedido

Imagine um sistema onde um pedido pode estar em um dos seguintes estados: "pending", "shipped", "delivered", "cancelled". Você pode usar um Union Type para representar esses estados.

```typescript
type OrderStatus = "pending" | "shipped" | "delivered" | "cancelled";

function updateOrderStatus(status: OrderStatus): void {
  console.log(`Pedido atualizado para: ${status}`);
}

updateOrderStatus("shipped");    // OK
updateOrderStatus("processing"); // Erro: "processing" não é um OrderStatus válido.
```

### Tipos de Respostas da API

Uma API pode retornar diferentes tipos de respostas, dependendo do sucesso ou erro da operação. Um Union Type pode representar essas possibilidades.

```typescript
type ApiResponse = 
  | { status: "success"; data: any }
  | { status: "error"; message: string };

function handleApiResponse(response: ApiResponse): void {
  if (response.status === "success") {
    console.log("Dados recebidos:", response.data);
  } else {
    console.error("Erro:", response.message);
  }
}

handleApiResponse({ status: "success", data: { id: 1, name: "Alice" } }); // OK
handleApiResponse({ status: "error", message: "Falha na requisição" });   // OK
```

### Eventos do Sistema

Em sistemas que lidam com eventos (ex.: logs ou interações), você pode usar Union Types para limitar os tipos de eventos permitidos.

```typescript
type EventType = "click" | "hover" | "keydown";

function logEvent(event: EventType): void {
  console.log(`Evento registrado: ${event}`);
}

logEvent("click");  // OK
logEvent("resize"); // Erro: "resize" não é um EventType válido.
```

### Tipos de Usuários em um Sistema

Em sistemas com diferentes tipos de usuários (ex.: admin, usuário comum, convidado), você pode usar Union Types para definir permissões específicas.

```typescript
type UserRole = "admin" | "user" | "guest";

function getDashboard(role: UserRole): string {
  switch (role) {
    case "admin":
      return "Admin Dashboard";
    case "user":
      return "User Dashboard";
    case "guest":
      return "Guest Dashboard";
  }
}

console.log(getDashboard("admin")); // "Admin Dashboard"
console.log(getDashboard("moderator")); // Erro
```

### Benefícios no Longo Prazo

#### Evita Bugs:

Tipos inválidos são detectados antes de o código ser executado.

#### Centralização:

Adicionar ou modificar opções é fácil e todas as referências ao tipo são automaticamente atualizadas.

#### Segurança e Legibilidade:

Garante que desenvolvedores entendam quais valores são permitidos sem consultar outras partes do código.


## 5. Tipo para Estruturas de Dados Aninhadas
Tipos podem ser usados para descrever estruturas de dados mais complexas, como uma lista de produtos em um e-commerce.

```typescript
type Product = {
  id: number;
  name: string;
  price: number;
  tags: string[];
};

type Cart = {
  items: Product[];
  total: number;
};

const cart: Cart = {
  items: [
    { id: 1, name: "Laptop", price: 1000, tags: ["electronics", "computer"] },
    { id: 2, name: "Mouse", price: 50, tags: ["electronics", "accessory"] },
  ],
  total: 1050,
};
```
Este exemplo mostra como usar tipos em TypeScript para descrever estruturas de dados complexas e aninhadas. No caso apresentado, temos uma aplicação de e-commerce onde:

Cada produto é representado por um objeto Product.
O carrinho de compras (Cart) contém uma lista de produtos (Product[]) e um total do preço dos itens.
Ao modelar estruturas de dados como essas em TypeScript:

Você melhora a consistência do código, garantindo que a estrutura seja sempre respeitada.
Garante segurança de tipos, evitando que dados inválidos sejam usados ou retornados.
Melhora a legibilidade do código, tornando mais claro o formato esperado dos dados.

Detalhando o Exemplo
1. Definição do Tipo Product
O tipo Product descreve a estrutura de um produto, com os seguintes campos:

id (número): Um identificador único para o produto.
name (string): O nome do produto.
price (número): O preço do produto.
tags (array de strings): Uma lista de palavras-chave associadas ao produto.
```typescript
type Product = {
  id: number;
  name: string;
  price: number;
  tags: string[];
};
```

2. Definição do Tipo Cart
O tipo Cart representa o carrinho de compras e contém:

items (array de Product): Uma lista de produtos adicionados ao carrinho.
total (número): O valor total dos produtos no carrinho.
```typescript
type Cart = {
  items: Product[];
  total: number;
};
```

3. Instanciando um Carrinho
O objeto cart segue a estrutura do tipo Cart, com dois produtos no campo items e o total de seus preços em total.

```typescript
const cart: Cart = {
  items: [
    { id: 1, name: "Laptop", price: 1000, tags: ["electronics", "computer"] },
    { id: 2, name: "Mouse", price: 50, tags: ["electronics", "accessory"] },
  ],
  total: 1050,
};
```

Por que isso é útil no mundo real?
Modelagem Clara de Dados:

Modelar dados complexos (como produtos e carrinhos) facilita o entendimento do formato esperado e como eles são manipulados na aplicação.
Reutilização:

Tipos como Product podem ser usados em várias partes do sistema:
Listas de produtos na loja.
Produtos em carrinhos.
Produtos em pedidos.
Validação Automática:

O TypeScript verifica automaticamente se os objetos seguem as estruturas definidas, ajudando a evitar erros de dados malformados.
Autocompletes e Segurança:

IDEs como VS Code oferecem sugestões e verificações com base nesses tipos, tornando o desenvolvimento mais produtivo e seguro.
Cenários Reais de Uso
1. Listagem de Produtos
O tipo Product pode ser usado para tipar a resposta de uma API que retorna produtos disponíveis em uma loja.

```typescript
async function fetchProducts(): Promise<Product[]> {
  return [
    { id: 1, name: "Keyboard", price: 30, tags: ["electronics", "computer"] },
    { id: 2, name: "Monitor", price: 150, tags: ["electronics", "display"] },
  ];
}

fetchProducts().then((products) => {
  products.forEach((product) => {
    console.log(`Produto: ${product.name}, Preço: ${product.price}`);
  });
});
```

2. Adicionar Produtos ao Carrinho
A estrutura do tipo Cart garante que os produtos adicionados ao carrinho sigam a definição do tipo Product.

```typescript
function addToCart(cart: Cart, product: Product): Cart {
  const updatedItems = [...cart.items, product];
  const updatedTotal = updatedItems.reduce((sum, item) => sum + item.price, 0);

  return {
    items: updatedItems,
    total: updatedTotal,
  };
}

// Exemplo de uso
const newProduct: Product = { id: 3, name: "Headphones", price: 200, tags: ["electronics", "audio"] };
const updatedCart = addToCart(cart, newProduct);

console.log(updatedCart);
// Saída:
// {
//   items: [...produtos existentes, { id: 3, name: "Headphones", price: 200, tags: ["electronics", "audio"] }],
//   total: 1250
// }
```

3. Gerar Relatórios de Carrinho
O tipo Cart pode ser usado para gerar relatórios, como a quantidade total de itens e a soma dos preços.

```typescript
function generateCartSummary(cart: Cart): string {
  return `O carrinho possui ${cart.items.length} itens. Total: R$ ${cart.total.toFixed(2)}`;
}

console.log(generateCartSummary(cart));
// Saída: "O carrinho possui 2 itens. Total: R$ 1050.00"
```

4. Ordenação de Produtos
O tipo Product pode ser usado para criar funções que ordenam produtos por preço ou outro critério.

```typescript
function sortProductsByPrice(products: Product[], ascending: boolean = true): Product[] {
  return products.sort((a, b) => (ascending ? a.price - b.price : b.price - a.price));
}

// Exemplo
const sortedProducts = sortProductsByPrice(cart.items);
console.log(sortedProducts);
// Saída (ordenado por preço): [{ id: 2, ... }, { id: 1, ... }]
```

Benefícios no Longo Prazo
Consistência:

A modelagem clara dos dados reduz o risco de inconsistências ou uso inadequado.
Reutilização:

Tipos como Product e Cart podem ser usados em várias partes do sistema, como APIs, validações, e interfaces de usuário.
Escalabilidade:

Se novos campos forem adicionados aos tipos (ex.: descontos para Product ou impostos para Cart), as mudanças são propagadas automaticamente para todo o código que usa esses tipos.
Facilidade de Depuração:

Saber exatamente quais campos estão disponíveis em cada tipo facilita o rastreamento de bugs e compreensão do código.



## 6. Tipo para Representar Configurações
Se o sistema depende de configurações, um type pode simplificar a definição.

```typescript
type Config = {
  baseUrl: string;
  retries: number;
  timeout: number;
};

const appConfig: Config = {
  baseUrl: "https://api.example.com",
  retries: 3,
  timeout: 5000,
};
```

## 7. Tipo para Compor Propriedades (Intersection)
Use & para combinar tipos e criar novos.

```typescript
type Timestamps = {
  createdAt: Date;
  updatedAt: Date;
};

type UserWithTimestamps = User & Timestamps;

const detailedUser: UserWithTimestamps = {
  id: 1,
  name: "Alice",
  createdAt: new Date(),
  updatedAt: new Date(),
};
```

## 8. Tipo para Reutilizar Campos Opcionais
Use Partial<T> para tornar todas as propriedades de um tipo opcionais.

```typescript
type PartialUser = Partial<User>;

const updateUser: PartialUser = {
  name: "Bob", // Somente alguns campos precisam ser atualizados.
};
```

## 9. Tipo para Restringir Campos Específicos
Use Pick<T, K> para selecionar apenas algumas propriedades de um tipo.

```typescript
type UserContactInfo = Pick<User, "email" | "name">;

const contact: UserContactInfo = {
  name: "Alice",
  email: "alice@example.com",
};
```

## 10. Tipo para Excluir Propriedades
Use Omit<T, K> para excluir propriedades específicas de um tipo.

```typescript
type UserWithoutEmail = Omit<User, "email">;

const userWithoutEmail: UserWithoutEmail = {
  id: 1,
  name: "Alice",
};
```

## 11. Tipo Genérico com Condicional
Use condicionais em tipos para criar soluções mais flexíveis.

```typescript
type AdminUser<T> = T extends { role: "admin" } ? T & { permissions: string[] } : T;

type Admin = AdminUser<{ id: number; name: string; role: "admin" }>;

const adminUser: Admin = {
  id: 1,
  name: "Alice",
  role: "admin",
  permissions: ["manage_users", "delete_posts"],
};
```

## 12. Tipo para Representar Eventos Dinâmicos
Crie tipos para um sistema de eventos.

```typescript
type EventPayload = {
  login: { userId: number };
  logout: { timestamp: Date };
};

type Event<K extends keyof EventPayload> = {
  type: K;
  payload: EventPayload[K];
};

const loginEvent: Event<"login"> = {
  type: "login",
  payload: { userId: 123 },
};

```