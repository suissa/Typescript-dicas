# 1 - Use readonly para Imutabilidade

## 1. Protegendo Propriedades de Objetos

Se você deseja garantir que certas propriedades de um objeto não sejam alteradas 
após a inicialização:

```typescript
type User = {
  readonly id: number;
  name: string;
};

const user: User = {
  id: 1,
  name: "Alice",
};

user.name = "Bob"; // Ok
// user.id = 2; // Erro: Cannot assign to 'id' because it is a read-only property.
```


## 2. Protegendo Arrays
O modificador readonly em arrays garante que não será possível alterar, adicionar ou remover elementos.

```typescript
const colors: readonly string[] = ["red", "green", "blue"];

// colors.push("yellow"); // Erro: Property 'push' does not exist on type 'readonly string[]'.
// colors[0] = "purple"; // Erro: Index signature in type 'readonly string[]' only permits reading.

console.log(colors[0]); // "red"
```

## 3. Classes com Propriedades Imutáveis
Use readonly em propriedades de classes para criar objetos que possuem atributos imutáveis.

```typescript
class Point {
  readonly x: number;
  readonly y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const point = new Point(10, 20);
// point.x = 15; // Erro: Cannot assign to 'x' because it is a read-only property.
```

## 4. Protegendo Parâmetros de Funções
Você pode usar readonly em parâmetros para evitar alterações internas em arrays passados para uma função.

```typescript
function printNames(names: readonly string[]): void {
  // names.push("John"); // Erro: Property 'push' does not exist on type 'readonly string[]'.
  names.forEach((name) => console.log(name));
}

const employees = ["Alice", "Bob", "Charlie"];
printNames(employees);
```

## 5. Objetos com Nested readonly
Propriedades aninhadas também podem ser protegidas com readonly.

```typescript
type Config = {
  readonly database: {
    readonly host: string;
    readonly port: number;
  };
};

const config: Config = {
  database: {
    host: "localhost",
    port: 5432,
  },
};

// config.database.host = "127.0.0.1"; // Erro: Cannot assign to 'host' because it is a read-only property.
```

## 6. Usando Readonly<T> Utility Type
TypeScript fornece o utilitário Readonly<T> para transformar todas as propriedades de um tipo em somente leitura.

```typescript
type User = {
  id: number;
  name: string;
};

const readonlyUser: Readonly<User> = {
  id: 1,
  name: "Alice",
};

// readonlyUser.name = "Bob"; // Erro: Cannot assign to 'name' because it is a read-only property.
```

## 7. Imutabilidade em Redux ou Estados
Em sistemas baseados em estado, como Redux, readonly ajuda a garantir que os estados não sejam mutados acidentalmente.

```typescript
type State = {
  readonly user: {
    readonly id: number;
    readonly name: string;
  };
  readonly tasks: readonly string[];
};

const state: State = {
  user: { id: 1, name: "Alice" },
  tasks: ["Task 1", "Task 2"],
};

// state.user.name = "Bob"; // Erro
// state.tasks.push("Task 3"); // Erro
```

8. Propriedades de Classes com readonly e Inicialização Automática
readonly funciona bem com inicializadores de classe.

```typescript
class User {
  readonly id: number;
  readonly createdAt = new Date();

  constructor(id: number) {
    this.id = id;
  }
}

const user = new User(101);
// user.createdAt = new Date(); // Erro: Cannot assign to 'createdAt' because it is a read-only property.
```

## Cenários reais

### 1. Configurações Imutáveis de Aplicação
Imagine que você tenha um conjunto de configurações carregadas uma vez no início da aplicação e que não devem ser alteradas em nenhum momento.

```typescript
type AppConfig = {
  readonly apiUrl: string;
  readonly port: number;
  readonly env: "development" | "production";
};

const config: AppConfig = {
  apiUrl: "https://api.example.com",
  port: 8080,
  env: "production",
};

// config.apiUrl = "https://new-api.example.com"; // Erro: Cannot assign to 'apiUrl' because it is a read-only property.
console.log(config.env); // "production"
```

#### Configurações de Ambiente
Em aplicações Node.js ou backend, as configurações carregadas de variáveis de ambiente podem ser marcadas como imutáveis para evitar alterações acidentais.

```typescript
type AppConfig = {
  readonly appName: string;
  readonly environment: "development" | "staging" | "production";
  readonly port: number;
  readonly db: {
    readonly host: string;
    readonly user: string;
    readonly password: string;
  };
};

const config: AppConfig = {
  appName: "MyAwesomeApp",
  environment: "production",
  port: 3000,
  db: {
    host: "localhost",
    user: "admin",
    password: "securepassword",
  },
};

// Tentativa de alterar valores
// config.environment = "development"; // Erro
// config.db.password = "newpassword"; // Erro

console.log(config);
```

**Isso garante que valores críticos, como credenciais de banco de dados ou ambiente de execução, não sejam alterados após a inicialização.**

#### Configuração de Rotas Backend
Em APIs RESTful, as rotas do sistema podem ser definidas como imutáveis para garantir que os endpoints permaneçam consistentes.

```typescript
type RouteConfig = {
  readonly path: string;
  readonly method: "GET" | "POST" | "PUT" | "DELETE";
  readonly description: string;
};

const routes: readonly RouteConfig[] = [
  { path: "/users", method: "GET", description: "Fetch all users" },
  { path: "/users/:id", method: "GET", description: "Fetch user by ID" },
  { path: "/users", method: "POST", description: "Create a new user" },
];

// Tentativa de alterar
// routes[0].path = "/customers"; // Erro
// routes.push({ path: "/products", method: "GET", description: "Fetch products" }); // Erro

console.log(routes);
```

**Isso protege as definições de rotas contra mudanças acidentais que poderiam quebrar integrações de API.**

#### Configurações de Serviços Externos
Ao configurar integrações com APIs externas, como AWS ou Firebase, as chaves e URLs podem ser protegidas como somente leitura.

```typescript
type ExternalServiceConfig = {
  readonly serviceName: string;
  readonly apiKey: string;
  readonly endpoint: string;
};

const awsConfig: ExternalServiceConfig = {
  serviceName: "AWS S3",
  apiKey: "AWS_KEY_12345",
  endpoint: "https://s3.amazonaws.com",
};

// Tentativa de alterar
// awsConfig.apiKey = "NEW_KEY"; // Erro
// awsConfig.endpoint = "https://new-endpoint.com"; // Erro

console.log(awsConfig);
```

**Isso garante a segurança das chaves de acesso e evita alterações que poderiam causar falhas de integração.**

#### Configuração de Frontend
Em aplicações React ou Angular, as configurações para temas ou endpoints podem ser marcadas como imutáveis.

```typescript
type FrontendConfig = {
  readonly theme: {
    readonly primaryColor: string;
    readonly secondaryColor: string;
  };
  readonly apiBaseUrl: string;
};

const config: FrontendConfig = {
  theme: {
    primaryColor: "#ff5733",
    secondaryColor: "#333333",
  },
  apiBaseUrl: "https://api.myfrontendapp.com",
};

// Tentativa de alterar
// config.theme.primaryColor = "#000000"; // Erro
// config.apiBaseUrl = "http://localhost:3000"; // Erro

console.log(config.theme.primaryColor); // "#ff5733"
```

**Isso arante consistência no tema visual e na URL base das chamadas de API.**

#### Configuração de Permissões
Em sistemas de controle de acesso, as permissões e papéis podem ser definidas como imutáveis para evitar alterações dinâmicas.

```typescript
type RoleConfig = {
  readonly role: string;
  readonly permissions: readonly string[];
};

const roles: readonly RoleConfig[] = [
  { role: "admin", permissions: ["create", "read", "update", "delete"] },
  { role: "user", permissions: ["read"] },
];

// Tentativa de alterar
// roles[0].permissions.push("manage_users"); // Erro
// roles[1].role = "guest"; // Erro

console.log(roles);
```

**Isso garante que os papéis e permissões sejam consistentes durante a execução do sistema.**

#### Configuração de Localização
Em sistemas multilíngues, as traduções e idiomas suportados podem ser armazenados como configurações imutáveis.

```typescript
type LocalizationConfig = {
  readonly defaultLanguage: string;
  readonly supportedLanguages: readonly string[];
};

const localization: LocalizationConfig = {
  defaultLanguage: "en",
  supportedLanguages: ["en", "es", "fr", "de"],
};

// Tentativa de alterar
// localization.defaultLanguage = "es"; // Erro
// localization.supportedLanguages.push("pt"); // Erro

console.log(localization);
```

**Isso previne alterações que poderiam causar inconsistências no suporte a idiomas.**

#### Configuração de Taxas ou Valores Fixos
Sistemas que utilizam valores fixos, como taxas ou impostos, podem proteger esses dados contra alterações.

```typescript
type TaxConfig = {
  readonly vat: number; // Imposto sobre valor agregado
  readonly serviceFee: number;
};

const taxConfig: TaxConfig = {
  vat: 0.2, // 20%
  serviceFee: 5.0, // Taxa fixa
};

// Tentativa de alterar
// taxConfig.vat = 0.25; // Erro
// taxConfig.serviceFee = 10; // Erro

console.log(`VAT: ${taxConfig.vat}, Service Fee: ${taxConfig.serviceFee}`);
```

**Isso evita alterações acidentais em valores críticos que impactam o cálculo de preços.**

#### Quando usar configurações imutáveis?

- Dados sensíveis: Credenciais, chaves de API, URLs de serviços externos.
- Valores críticos: Taxas, valores fixos, configurações de ambiente.
- Definições globais: Configurações de tema, rotas, localizações.
- Segurança: Proteção contra alterações não intencionais ou maliciosas.
- Consistência: Garante que configurações permaneçam as mesmas durante toda a execução da aplicação.



### 2. Modelos de Dados Imutáveis em um Sistema de Inventário
Em sistemas de inventário, os dados como o ID de um produto não devem mudar após serem criados já que cada produto é geralmente identificado por um ID único (como um número ou um UUID). Esse identificador único é usado para rastrear, atualizar e gerenciar os produtos no sistema. Garantir que o ID de um produto seja imutável é essencial para evitar inconsistências e erros no banco de dados ou no sistema como um todo.

```typescript
type InventoryItem = {
  readonly id: string;
  name: string;
  stock: number;
};

const item: InventoryItem = {
  id: "ABC123",
  name: "Notebook",
  stock: 50,
};

// item.id = "XYZ456"; // Erro: Cannot assign to 'id' because it is a read-only property.
item.stock -= 1; // Ok, apenas o ID é imutável.
```

### 3. Estados de Redux ou Gerenciadores de Estado
No gerenciamento de estados, como com Redux, os estados devem ser imutáveis para evitar mutações acidentais.

```typescript
type State = {
  readonly user: {
    readonly id: number;
    readonly name: string;
  };
  readonly todos: readonly string[];
};

const state: State = {
  user: { id: 1, name: "Alice" },
  todos: ["Task 1", "Task 2"],
};

// state.user.name = "Bob"; // Erro: Cannot assign to 'name' because it is a read-only property.
// state.todos.push("Task 3"); // Erro: Property 'push' does not exist on type 'readonly string[]'.
console.log(state.todos[0]); // Ok: "Task 1"
```

#### Como alterar o estado corretamente?
Em sistemas como Redux, a forma correta de alterar o estado é criar uma nova cópia das propriedades que precisam ser atualizadas, sem modificar o estado existente. Isso é feito frequentemente em um reducer.

Exemplo com Redux:

```typescript
function reducer(state: State, action: { type: string; payload?: any }): State {
  switch (action.type) {
    case "UPDATE_USER_NAME":
      // Retorna uma nova cópia do estado com o nome do usuário atualizado
      return {
        ...state,
        user: {
          ...state.user,
          name: action.payload,
        },
      };

    case "ADD_TODO":
      // Retorna uma nova cópia do estado com um novo todo adicionado
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };

    default:
      return state; // Estado inalterado
  }
}
```

Ações:

```typescript
const updateUserNameAction = { type: "UPDATE_USER_NAME", payload: "Bob" };
const addTodoAction = { type: "ADD_TODO", payload: "Task 3" };

const newState1 = reducer(state, updateUserNameAction);
console.log(newState1.user.name); // "Bob"

const newState2 = reducer(newState1, addTodoAction);
console.log(newState2.todos); // ["Task 1", "Task 2", "Task 3"]
```

##### Benefícios da Imutabilidade no Gerenciamento de Estado

###### Debugging mais Fácil:

Como o estado antigo e o novo são preservados, é possível rastrear mudanças ao longo do tempo usando ferramentas como o Redux DevTools.


###### Previsibilidade:

A imutabilidade garante que o estado só mude de maneira controlada, reduzindo o risco de alterações não intencionais.

###### Time-Travel Debugging:

A imutabilidade permite que você "volte no tempo" para estados anteriores, pois os estados antigos não são sobrescritos.

###### Escalabilidade:

Em projetos grandes, a imutabilidade facilita a manutenção, especialmente quando vários desenvolvedores estão colaborando.


### 4. Listas de Constantes Compartilhadas
Se você tem uma lista de constantes que será usada em vários lugares do sistema, readonly ajuda a evitar alterações.

```typescript
const CATEGORIES: readonly string[] = ["electronics", "books", "clothing"];

// CATEGORIES.push("toys"); // Erro: Property 'push' does not exist on type 'readonly string[]'.
// CATEGORIES[0] = "furniture"; // Erro: Index signature in type 'readonly string[]' only permits reading.

console.log(CATEGORIES.includes("books")); // Ok: true
```

Imagine uma empresa de delivery que só atende determinadas cidades. Essas cidades são armazenadas como uma lista fixa de constantes imutáveis, já que a área de cobertura não muda com frequência e deve ser consistente em todo o sistema.

Essa lista será usada em diversas partes da aplicação:

- No frontend, para mostrar apenas as cidades disponíveis no formulário de checkout.
- No backend, para validar pedidos e garantir que só sejam aceitos para cidades atendidas.
- Em relatórios, para garantir consistência na análise de dados.

Exemplo: Lista de Cidades Atendidas

```typescript
const SUPPORTED_CITIES: readonly string[] = [
  "São Paulo",
  "Rio de Janeiro",
  "Belo Horizonte",
  "Curitiba",
  "Porto Alegre",
];
```

#### Benefícios do Uso de readonly Aqui

##### Consistência: 

A lista não pode ser alterada em nenhuma parte do sistema, garantindo que todos os módulos usem os mesmos dados.

##### Segurança: 

Evita que métodos como push ou atribuições diretas alterem a lista acidentalmente.

##### Reutilização: 

A mesma lista pode ser usada no frontend, backend e até em integrações externas.

### 5. Resposta de APIs

Imutabilidade em respostas de APIs evita que os dados sejam alterados inadvertidamente após o recebimento.
A imutabilidade em respostas de APIs é usada quando você deseja garantir que os dados recebidos de uma API permaneçam consistentes e inalterados durante o ciclo de vida da aplicação. Isso é especialmente útil em situações onde alterações nos dados podem causar erros ou comportamento inesperado.

Exemplo:

```typescript
type ApiResponse = {
  readonly status: number;
  readonly data: {
    readonly id: number;
    readonly name: string;
  };
};

const response: ApiResponse = {
  status: 200,
  data: {
    id: 1,
    name: "Alice",
  },
};

// response.status = 404; // Erro
// response.data.name = "Bob"; // Erro
console.log(response.data.name); // Ok: "Alice"
```

Isso vai te ajudar a:

#### Ter maior segurança dos dados
Quando você precisa garantir que dados sensíveis ou críticos, como IDs de usuários, tokens de autenticação ou informações financeiras, não sejam acidentalmente modificados:

Exemplo:

```typescript
type ApiResponse = {
  readonly status: number;
  readonly data: {
    readonly userId: number;
    readonly balance: number;
  };
};

const response: ApiResponse = {
  status: 200,
  data: {
    userId: 12345,
    balance: 5000,
  },
};

// response.data.balance = 1000; // Erro: balance não pode ser alterado
```

**Isso previne bugs onde informações sensíveis poderiam ser sobrescritas inadvertidamente.**

#### Debugging mais Fácil
Se os dados de uma API forem alterados acidentalmente durante o processamento, pode ser difícil depurar a origem do problema. Tornando-os readonly, você garante que o estado inicial seja preservado para análise.

Exemplo:
```typescript
type ApiResponse = {
  readonly data: {
    readonly id: number;
    readonly name: string;
  };
};

const response: ApiResponse = {
  data: { id: 1, name: "Alice" },
};

// Durante o processamento
function modifyResponse(res: ApiResponse): void {
  // res.data.name = "Bob"; // Erro: Não é permitido alterar a resposta
  console.log(res.data.name); // "Alice"
}
```

**Isso garante que o estado original dos dados seja preservado para futuras verificações.**

#### Gerenciamento de Estados Imutáveis

Em sistemas que utilizam bibliotecas de gerenciamento de estado (como Redux), a imutabilidade das respostas de API ajuda a evitar que o estado global seja corrompido acidentalmente.

Exemplo:

```typescript
type UserState = {
  readonly user: {
    readonly id: number;
    readonly name: string;
  };
};

const initialState: UserState = {
  user: { id: 1, name: "Alice" },
};

// Redux reducer
function userReducer(state = initialState, action: any): UserState {
  switch (action.type) {
    case "UPDATE_NAME":
      // state.user.name = action.payload; // Erro
      return {
        user: { ...state.user, name: action.payload }, // Atualização segura
      };
    default:
      return state;
  }
}
```

**Isso arante que o estado nunca seja alterado diretamente, mantendo a consistência do fluxo de dados.**


#### Evitar Efeitos Colaterais em Funções
Se os dados de uma API forem passados para diferentes funções, a imutabilidade garante que nenhuma função altere os dados diretamente.

Exemplo:

```typescript
type ApiResponse = {
  readonly data: {
    readonly id: number;
    readonly value: number;
  };
};

function processResponse(response: ApiResponse): number {
  // response.data.value = 10; // Erro
  return response.data.value * 2;
}

const apiResponse: ApiResponse = { data: { id: 1, value: 5 } };
console.log(processResponse(apiResponse)); // 10
```

**Isso previne alterações inesperadas nos dados durante o processamento.**

#### Controle de Versionamento
Se os dados de uma API forem alterados dinamicamente em diferentes partes da aplicação, isso pode quebrar integrações ou causar incompatibilidades.

Exemplo:

```typescript
type ApiResponse = {
  readonly version: string;
  readonly data: any;
};

const response: ApiResponse = {
  version: "1.0.0",
  data: { message: "Success" },
};

// response.version = "2.0.0"; // Erro
```

**Isso ajuda a rastrear e manter a compatibilidade de dados.**


#### Quando Usar?
Use a imutabilidade em respostas de API sempre que você precisar:

Preservar o estado original dos dados.
Evitar erros causados por mutações não intencionais.
Manter a integridade do estado global ou local.
Proteger dados sensíveis ou críticos contra alterações.
Garantir consistência em aplicações colaborativas.

### 7. Rotas de Navegação em um Frontend
Rotas definidas no frontend normalmente são constantes e não devem ser modificadas.

```typescript
type Route = {
  readonly path: string;
  readonly component: string;
};

const routes: readonly Route[] = [
  { path: "/", component: "Home" },
  { path: "/about", component: "About" },
];

// routes.push({ path: "/contact", component: "Contact" }); // Erro
// routes[0].path = "/home"; // Erro
console.log(routes[0].path); // Ok: "/"
```

### 8. Dados Sensíveis em Aplicações
Dados como IDs de usuários ou tokens de autenticação devem ser imutáveis após serem definidos.

```typescript
type UserSession = {
  readonly userId: number;
  readonly token: string;
  isActive: boolean;
};

const session: UserSession = {
  userId: 12345,
  token: "abcdefg12345",
  isActive: true,
};

// session.userId = 67890; // Erro: Cannot assign to 'userId' because it is a read-only property.
session.isActive = false; // Ok, status pode ser alterado.
```

### 9. Configurações de Serviços Terceirizados
As credenciais ou configurações de serviços externos devem ser protegidas contra alterações.

```typescript
type ExternalServiceConfig = {
  readonly apiKey: string;
  readonly endpoint: string;
};

const serviceConfig: ExternalServiceConfig = {
  apiKey: "API_KEY_123456",
  endpoint: "https://api.example.com",
};

// serviceConfig.apiKey = "NEW_KEY"; // Erro
console.log(serviceConfig.endpoint); // Ok: "https://api.example.com"
```
