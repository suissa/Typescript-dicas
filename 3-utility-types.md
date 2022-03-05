# 3 - Use Utility Types

O TypeScript fornece *Utility Types* que facilitam a manipulação de tipos existentes, permitindo criar variações desses tipos de maneira eficiente e reutilizável. Se você ainda nunca usou algum desses está mais que na hora. Esses Utility Types são:

- `Partial`
- `Pick`
- `Omit`
- `Required`
- `Record`
- `Extract`
- `Exclude`
- `NonNullable`
- `ReturnType`
- `Awaited `

Ao utilizar algum desses *Utility Types* você demonstra maior conhecimento do Typescript além de deixar seu código muito mais profissional e dinâmico.

## 1. Partial<T>

O tipo utilitário `Partial<T>` transforma todas as propriedades de um tipo em opcionais (?).

### Definição

```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

Ele percorre todas as propriedades do tipo `T` usando `keyof`.
Cada propriedade é marcada como opcional (?).

### Definição do Tipo Base

```ts
type User = { 
  id: number; 
  name: string; 
  email: string; 
};
```

### Criando um Tipo Partial

O `Partial<User>` transforma todas as propriedades de User em opcionais:

```ts
type PartialUser = Partial<User>;
```

Antes:
```ts
const user: User = { id: 1, name: "Alice", email: "alice@example.com" };
```

Depois:
```ts
const partialUser: PartialUser = { name: "Alice" }; // OK, propriedades opcionais
```

### Cenários Reais de Uso

#### 1. Atualização Parcial de Dados

Em sistemas onde você precisa atualizar apenas parte de um objeto, o `Partial<T> `permite criar tipos que refletem esse comportamento.

Exemplo: Atualização de Perfil

Imagine que um endpoint de API permita atualizar somente alguns campos de um perfil de usuário:

```ts
async function updateUser(id: number, updates: Partial<User>): void {
  // mongoose
  return await User.findOneAndUpdate(id, updates)
}

// Atualizando apenas o nome
await updateUser(1, { name: "Bob" });
// Atualizando o e-mail
await updateUser(2, { email: "bob@example.com" });
// Atualizando múltiplos campos
await updateUser(3, { name: "Charlie", email: "charlie@example.com" });
```

Por exemplo no Nest.js, você pode criar um `UpdateDTO` dessa forma:

```ts
export class User {
  id: number;
  name: string;
  email: string;
  password: string;
  isActive: boolean;
}
```

E agora criamos o `UpdateUserDTO`:

```ts
import { PartialType } from '@nestjs/mapped-types';
import { User } from './user.entity';

// Todas as propriedades de User agora são opcionais no UpdateUserDto
export class UpdateUserDto extends PartialType(User) {}
```

> O `PartialType` é o utilitário fornecido pelo NestJS para aplicar a lógica de `Partial`

Benefício:

Você não precisa criar manualmente uma variação do tipo `User` com campos opcionais.
A função aceita apenas campos válidos de `User`.

#### 2. Formulários Dinâmicos

O `Partial<T>` é útil para modelar formulários que não requerem todos os campos de um tipo.

Exemplo: Formulário de Cadastro

```ts
function validateForm(form: Partial<User>): boolean {
  if (form.email && !form.email.includes("@")) {
    console.error("E-mail inválido");
    return false;
  }
  console.log("Formulário válido:", form);
  return true;
}

// Testando o formulário
validateForm({ name: "Alice" }); // OK
validateForm({ email: "invalid-email" }); // E-mail inválido
```

##### Benefício:

Permite lidar com objetos parciais sem comprometer a segurança de tipos.

#### 3. Configurações Opcionais

Ao trabalhar com configurações de sistema, o `Partial<T>` permite lidar com opções onde nem todos os campos precisam ser definidos.

Exemplo: Configurações de Aplicação

```ts
type AppConfig = {
  theme: string;
  language: string;
  notifications: boolean;
};

function initializeApp(config: Partial<AppConfig>): void {
  const defaultConfig: AppConfig = {
    theme: "light",
    language: "en",
    notifications: true,
  };

  const finalConfig = { ...defaultConfig, ...config };
  console.log("Configuração Final:", finalConfig);
}

// Usando apenas alguns campos de configuração
initializeApp({ theme: "dark" });
// Saída: Configuração Final: { theme: "dark", language: "en", notifications: true }
```

##### Benefício:

O `Partial<T>` facilita a definição de configurações personalizadas sem a necessidade de especificar todos os campos.

#### 4. Funções de Criação de Objetos

Se você precisa criar objetos com alguns campos padrão e permitir a sobrescrita de outros, o `Partial<T>` é útil.

Exemplo: Criar Usuários

```ts
function createUser(overrides: Partial<User>): User {
  const defaultUser: User = { id: 0, name: "Unknown", email: "unknown@example.com" };
  return { ...defaultUser, ...overrides };
}

// Criando um usuário com campos personalizados
const newUser = createUser({ name: "Alice" });
console.log(newUser);
// Saída: { id: 0, name: "Alice", email: "unknown@example.com" }
```

##### Benefício:

Permite criar objetos com campos padrão e personalizados de forma segura.

---

## 2. Required<T>

O `Required<T>` transforma todas as propriedades de um tipo em obrigatórias, mesmo que originalmente fossem opcionais.

### 1. Garantir dados em APIs

Imagine um sistema de e-commerce onde o `Product` contém campos opcionais durante o processo de criação, mas todos os campos precisam ser preenchidos antes de salvar o produto no banco de dados.

```ts
type Product = {
  id?: number; // Gerado automaticamente, opcional na criação
  name?: string;
  price?: number;
};

type CompleteProduct = Required<Product>;

function saveProduct(product: CompleteProduct): void {
  console.log("Salvando produto:", product);
}

// Erro: todos os campos agora são obrigatórios
// saveProduct({ name: "Notebook" });

saveProduct({ id: 1, name: "Notebook", price: 2500 });
// Saída: Salvando produto: { id: 1, name: 'Notebook', price: 2500 }

```

#### Por que usar?

Garante que o banco de dados receba todos os campos necessários.
O TypeScript exige que todos os campos sejam fornecidos, reduzindo a chance de erros.


### 2. Garantir dados ao processar pagamentos

```ts
type Order = {
  orderId?: string;
  amount?: number;
  currency?: string;
};

type CompleteOrder = Required<Order>;

function processPayment(order: CompleteOrder): void {
  console.log("Processando pagamento:", order);
}

// Erro: todos os campos agora são obrigatórios
// processPayment({ orderId: "12345", amount: 100 });

processPayment({ orderId: "12345", amount: 100, currency: "USD" });
// Saída: Processando pagamento: { orderId: '12345', amount: 100, currency: 'USD' }
```

#### Por que usar?

Garante que todas as informações necessárias para o pagamento estejam presentes antes de processar a transação.
Evita erros em integrações com gateways de pagamento que exigem dados completos.

### 3. Completar dados de configuração

Em sistemas que usam configurações dinâmicas, como aplicativos com suporte a temas, você pode usar `Required<T>` para garantir que todas as propriedades de configuração estejam preenchidas antes de aplicar o tema.

```ts
type ThemeConfig = {
  primaryColor?: string;
  secondaryColor?: string;
  fontSize?: number;
};

type CompleteThemeConfig = Required<ThemeConfig>;

function applyTheme(config: CompleteThemeConfig): void {
  console.log("Aplicando tema com configuração:", config);
}

// Erro: todos os campos agora são obrigatórios
// applyTheme({ primaryColor: "#000000" });

applyTheme({ primaryColor: "#000000", secondaryColor: "#FFFFFF", fontSize: 16 });
// Saída: Aplicando tema com configuração: { primaryColor: '#000000', secondaryColor: '#FFFFFF', fontSize: 16 }
```

### 4. Gerar relatórios completos

Ao gerar relatórios, você pode usar `Required<T>` para garantir que todas as informações obrigatórias estejam disponíveis antes de criar ou salvar o relatório.

```ts
type Report = {
  reportId?: string;
  title?: string;
  createdBy?: string;
  data?: any[];
};

type CompleteReport = Required<Report>;

function generateReport(report: CompleteReport): void {
  console.log("Gerando relatório:", report);
}

// Erro: todos os campos agora são obrigatórios
// generateReport({ reportId: "R-123", title: "Relatório de Vendas" });

generateReport({
  reportId: "R-123",
  title: "Relatório de Vendas",
  createdBy: "Alice",
  data: [{ product: "Laptop", sales: 10 }],
});
// Saída: Gerando relatório: { reportId: 'R-123', title: 'Relatório de Vendas', createdBy: 'Alice', data: [...] }
```

#### Por que usar?

Garante a integridade do relatório, impedindo que informações incompletas sejam processadas ou exibidas.
Ajuda a identificar rapidamente campos ausentes.


### 5. Mock de Dados para Testes
Ao criar mocks para testes, pode ser necessário que todos os campos estejam presentes para simular um cenário completo.

```ts
type User = {
  id?: number;
  name?: string;
  email?: string;
};

type CompleteUser = Required<User>;

const mockUser: CompleteUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
};

console.log("Mock de usuário completo:", mockUser);
// Saída: Mock de usuário completo: { id: 1, name: 'Alice', email: 'alice@example.com' }
```

#### Por que usar?

Garante que os mocks sejam completos e representem cenários reais durante os testes.
Reduz inconsistências ou falhas ao simular dados incompletos.

### 6. Integrações Externas

Quando você envia dados para APIs externas, o Required<T> pode garantir que todas as informações obrigatórias estejam presentes.

```ts
type IntegrationPayload = {
  apiKey?: string;
  endpoint?: string;
  payload?: any;
};

type CompleteIntegrationPayload = Required<IntegrationPayload>;

function sendToIntegration(payload: CompleteIntegrationPayload): void {
  console.log("Enviando dados para integração:", payload);
}

// Erro: todos os campos agora são obrigatórios
// sendToIntegration({ apiKey: "1234" });

sendToIntegration({
  apiKey: "1234",
  endpoint: "https://api.example.com",
  payload: { data: "value" },
});
// Saída: Enviando dados para integração: { apiKey: '1234', endpoint: 'https://api.example.com', payload: { data: 'value' } }
```

#### Por que usar?

Garante que a integração receba todos os dados necessários para funcionar corretamente.
Evita falhas causadas por campos ausentes.

---

## 3. Readonly<T>
Torna todas as propriedades de um tipo somente leitura, impedindo modificações.

Eu já escrevi sobre os usos do [Readonly aqui](https://github.com/suissa/Typescript-dicas/blob/master/1-readonly.md)

---

## 4. Pick<T, K>
Extrai apenas propriedades específicas de um tipo.

### Como funciona Pick?

Definição:

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

T: O tipo original.
K: Um conjunto de propriedades (keyof T) que serão extraídas de T.

### Exemplos Reais de Uso do Pick

#### 1. Minimizar dados expostos

Imagine um sistema que precisa enviar informações do usuário ao frontend, mas sem expor dados sensíveis como a senha.

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type PublicUser = Pick<User, "id" | "name" | "email">;

const user: PublicUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
};

// O campo "password" não está disponível
// console.log(user.password); // Erro: Propriedade não existe no tipo 'PublicUser'.
```

Por exemplo no Nest.js, você pode fazer um GetDTO dessa forma:

```ts
export class User {
  id: number;
  name: string;
  email: string;
  password: string;
  isActive: boolean;
}
```

Reusando a `class User` assim:

```ts
import { PickType } from '@nestjs/mapped-types';
import { User } from './user.entity';

// Expondo apenas `id`, `name` e `email` no DTO
export class GetUserDto extends PickType(User, ['id', 'name', 'email'] as const) {}
```

> O `PickType` é o utilitário do Nest.js para aplicar a lógica do `Pick`

Bem fácil né? E aí voc~e usa dessa forma:

```ts
import { Controller, Get, Param } from '@nestjs/common';
import { GetUserDto } from './dto/get-user.dto';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  async getUser(@Param('id') id: number): Promise<GetUserDto> {
    const user = await this.userService.findOne(id);
    // Retornando apenas os campos definidos no DTO
    return {
      id: user.id,
      name: user.name,
      email: user.email,
    };
  }
}
```

Se necessário, você pode estender o DTO para adicionar propriedades adicionais.

```ts
export class GetUserDto extends PickType(User, ['id', 'name', 'email'] as const) {
  additionalField: string;
}
```

##### Por que usar?

Garante que apenas os dados permitidos sejam expostos.
Ajuda a evitar vazamento de informações sensíveis.

#### 2. Restrições em funções
Quando você deseja criar uma função que só trabalha com parte de um objeto, o Pick pode ser usado para limitar o escopo.

Exemplo: Atualizar Nome e E-mail

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type UpdateUser = Pick<User, "id" | "name" | "email">;

function updateUser(user: UpdateUser): void {
  console.log(`Atualizando usuário ${user.id} com nome ${user.name} e e-mail ${user.email}`);
}

updateUser({ id: 1, name: "Alice", email: "alice@example.com" });
// Erro: Propriedade "password" não faz parte do tipo 'UpdateUser'.
// updateUser({ id: 1, name: "Alice", email: "alice@example.com", password: "123456" });
```

##### Por que usar?

Garante que a função aceite apenas os campos permitidos.
Simplifica a assinatura da função, excluindo campos irrelevantes ou sensíveis.

#### 3. Resposta de API personalizada
Imagine uma API que retorna objetos com muitas propriedades, mas você deseja que o cliente receba apenas algumas delas.

Exemplo: API de Produtos

```ts
type Product = {
  id: number;
  name: string;
  description: string;
  price: number;
  stock: number;
  category: string;
};

type PublicProduct = Pick<Product, "id" | "name" | "price">;

const product: PublicProduct = {
  id: 101,
  name: "Smartphone",
  price: 1200,
};

// O frontend não recebe detalhes como descrição ou estoque
```

##### Por que usar?

Reduz o tamanho da resposta da API, enviando apenas os campos necessários.
Evita a exposição de informações internas, como estoque.

#### 4. Criação de interfaces para componentes

Ao trabalhar com componentes em frameworks como React, o Pick é útil para reutilizar tipos já definidos.

Exemplo: Propriedades de Componente

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type UserCardProps = Pick<User, "name" | "email">;

function UserCard({ name, email }: UserCardProps): JSX.Element {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}

<UserCard name="Alice" email="alice@example.com" />;
```

##### Por que usar?

Permite criar propriedades de componentes baseadas em tipos existentes.
Reutiliza a lógica já definida, evitando duplicação de código.

#### 5. Validação de Ddados em formulários
Em formulários, você pode querer trabalhar apenas com os campos que o usuário preenche, sem considerar outros campos do tipo original.

Exemplo: Formulário de Registro

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type RegistrationForm = Pick<User, "name" | "email" | "password">;

function validateRegistration(form: RegistrationForm): boolean {
  if (!form.email.includes("@")) {
    console.error("E-mail inválido");
    return false;
  }
  console.log("Formulário válido:", form);
  return true;
}

validateRegistration({ name: "Alice", email: "alice@example.com", password: "securepassword" });
```

##### Por que usar?

Permite trabalhar apenas com os campos relevantes para o formulário.
Garante que apenas os dados esperados sejam manipulados.

#### 6. Logs e Auditorias
Para logs ou auditorias, pode ser útil registrar apenas parte de um objeto, excluindo informações irrelevantes ou sensíveis.

Exemplo: Registro de Atividade de Usuário

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type UserLog = Pick<User, "id" | "name">;

function logUserActivity(user: UserLog): void {
  console.log(`Usuário ${user.name} (ID: ${user.id}) realizou uma ação.`);
}

logUserActivity({ id: 1, name: "Alice" });
```

##### Por que usar?

Reduz o risco de expor informações sensíveis em logs.
Mantém os logs limpos e relevantes.

### Vantagens do Pick

#### Reutilização de tipos:

Reaproveita tipos existentes em diferentes contextos sem precisar redefini-los.

#### Redução de dados:

Restringe os dados manipulados a um subconjunto necessário.

#### Melhoria de segurança:

Evita o vazamento de informações sensíveis, como senhas ou dados internos.

#### Manutenção simplificada:

Se o tipo original mudar, os tipos derivados com Pick serão atualizados automaticamente.

### Resumo

O Pick<T, K> é uma ferramenta poderosa para lidar com dados em TypeScript, especialmente em:

- restrições de acesso (APIs, logs, componentes).
- redução de dados expostos ou manipulados.
- criação de tipos específicos para cenários reutilizando tipos base.

----

## 5. Omit<T, K>
O `Omit<T, K>` é um utilitário do TypeScript usado para criar novos tipos a partir de um existente, excluindo propriedades específicas. Isso é especialmente útil em cenários onde você quer manipular ou expor objetos sem campos sensíveis ou irrelevantes.

### 1. Remover dados sensíveis

Imagine um sistema onde os dados de usuários incluem informações sensíveis, como senhas. Você precisa enviar esses dados ao frontend para exibir informações do usuário, mas sem expor campos como password.

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type SafeUser = Omit<User, "password">;

const user: SafeUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
};

// O campo "password" foi omitido
```

#### Por que usar?

Garante que informações sensíveis não sejam enviadas ou manipuladas em locais onde não deveriam estar.
Melhora a segurança ao evitar vazamento de dados críticos.

### 2. Create DTO (CreateUserDto)

O Create DTO é usado ao criar um novo usuário. Ele não inclui o campo id, pois este geralmente é gerado automaticamente (ex.: no banco de dados).

Todos os campos obrigatórios para criar um novo usuário devem estar presentes.

```ts
type CreateUserDto = Omit<User, "id">;

const createUserDto: CreateUserDto = {
  name: "Alice",
  email: "alice@example.com",
  password: "securepassword",
  isActive: true,
};
```

#### Por que usar?

Evita erros ao excluir campos gerados automaticamente no processo de criação.
Facilita a reutilização da estrutura de tipos existente.

### 3. DTO de atualização parcial (UpdateUserDto)

Usando Omit combinado com Partial, você pode criar um DTO para atualizações parciais, excluindo campos como id e permitindo que apenas os campos relevantes sejam atualizados.

```ts
type UpdateUserDto = Partial<Omit<User, "id">>;

const updateUserDto: UpdateUserDto = {
  email: "newemail@example.com",
};

// O campo "id" foi removido e os demais campos são opcionais
```

#### Por que usar?

Garante que o ID do recurso não seja atualizado inadvertidamente.
Permite atualizações flexíveis sem duplicação de tipos.

### 4. Integrações externas

Ao integrar com APIs externas, você pode usar Omit para limpar dados irrelevantes ou confidenciais antes de enviar o objeto.

```ts
type PaymentDetails = {
  cardNumber: string;
  cardHolder: string;
  expirationDate: string;
  cvv: string;
};

type SafePaymentDetails = Omit<PaymentDetails, "cvv">;

function processPayment(details: SafePaymentDetails): void {
  console.log("Processando pagamento:", details);
}

const payment: SafePaymentDetails = {
  cardNumber: "1234-5678-9012-3456",
  cardHolder: "Alice",
  expirationDate: "12/25",
};

// O CVV não é incluído no processamento
processPayment(payment);
```

#### Por que usar?

Reduz riscos de segurança ao evitar que dados sensíveis sejam enviados desnecessariamente.
Simplifica a interface de integração.

### 5. Filtrar campos em relatórios

Em relatórios ou logs, você pode querer remover informações irrelevantes para simplificar os dados exibidos.

```ts
type Transaction = {
  id: string;
  amount: number;
  date: string;
  customerDetails: string;
};

type ReportTransaction = Omit<Transaction, "customerDetails">;

const transaction: ReportTransaction = {
  id: "TX12345",
  amount: 100,
  date: "2025-01-01",
};

// Apenas os dados necessários para o relatório estão presentes
console.log("Relatório de Transação:", transaction);
```

#### Por que usar?

Foca apenas nos campos relevantes para o relatório.
Evita incluir dados desnecessários ou confidenciais.

### 6. Criar tipos para componentes em frontend

Ao construir componentes React ou de outros frameworks, você pode usar Omit para evitar conflitos de propriedades ou limitar o escopo.

```tsx
type ButtonProps = {
  label: string;
  onClick: () => void;
  disabled?: boolean;
};

type IconButtonProps = Omit<ButtonProps, "label"> & {
  icon: string;
};

function IconButton({ icon, onClick, disabled }: IconButtonProps): JSX.Element {
  return (
    <button onClick={onClick} disabled={disabled}>
      <i className={icon}></i>
    </button>
  );
}

<IconButton icon="fa-check" onClick={() => console.log("Clicked")} />;
```

#### Por que usar?

Remove propriedades irrelevantes (label) e adiciona novas específicas ao componente (icon).
Mantém a tipagem consistente e reutilizável.

### 7. Criação de logs de auditoria

Logs de auditoria geralmente precisam capturar ações específicas realizadas, sem armazenar informações sensíveis.

```ts
type UserAction = {
  userId: number;
  action: string;
  timestamp: string;
  details: string;
  sensitiveData: string;
};

type AuditLog = Omit<UserAction, "sensitiveData">;

const log: AuditLog = {
  userId: 1,
  action: "LOGIN",
  timestamp: "2025-01-01T10:00:00Z",
  details: "Usuário acessou o sistema",
};

// O dado sensível foi removido
console.log("Log de auditoria:", log);
```

#### Resumo

O `Omit<T, K>` é extremamente útil para:

- remover dados sensíveis em sistemas (ex.: senhas, CVVs).
- criar DTOs personalizados, como CreateDto e UpdateDto.
- filtrar campos desnecessários em logs, relatórios ou respostas de APIs.
- reduzir a complexidade de componentes ou interfaces em frontends.
- manter segurança e clareza, evitando a exposição de campos irrelevantes ou perigosos.

---

## 6. Record<K, T>
O `Record<K, T>` é um *utility type* do TypeScript que permite criar um tipo de objeto onde as chaves (`K`) são de um tipo específico, e os valores (`T`) seguem outro tipo. É especialmente útil para representar tabelas, mapas ou dicionários em que você precisa garantir a consistência do tipo das chaves e valores.

### 1. Mapear usuários por ID

No exemplo inicial, mapeamos usuários por seu ID, onde cada ID é uma chave e o valor é um objeto `User`.

```ts
type User = {
  id: number;
  name: string;
};

type UserRecord = Record<number, User>;

const users: UserRecord = {
  1: { id: 1, name: "Alice" },
  2: { id: 2, name: "Bob" },
};

console.log(users[1].name); // "Alice"
```

#### Por que usar?

Garante que todas as chaves são números e os valores são objetos User.
Reduz a chance de erros ao manipular dados mapeados por ID.

### 2. Representar configurações dinâmicas

Se sua aplicação permite configurações dinâmicas (ex.: temas ou preferências do usuário), `Record<K, T>` é uma maneira eficiente de modelar esses dados.

```ts
type Config = {
  value: string;
  description: string;
};

type Configs = Record<string, Config>;

const appConfigs: Configs = {
  theme: { value: "dark", description: "Tema escuro" },
  language: { value: "en", description: "Idioma padrão" },
};

console.log(appConfigs.theme.value); // "dark"
```

#### Por que usar?

Representa configurações como pares chave-valor de forma segura.
Garante que cada configuração siga o mesmo formato.

### 3. Localização de textos (i18n)

Em sistemas multilíngues, você pode usar `Record` para mapear chaves de texto (`K`) para traduções específicas (`T`).

```ts
type Translations = Record<string, string>;

const translations: Translations = {
  greeting: "Hello",
  farewell: "Goodbye",
  thankYou: "Thank you",
};

console.log(translations.greeting); // "Hello"
```

#### Por que usar?

Facilita a manutenção de textos traduzidos em sistemas internacionais.
Garante que todos os valores sejam strings, evitando erros como `undefined`.


### 4. Tabelas de produtos por categoria

Imagine que você tenha um catálogo de produtos organizados por categorias.

```ts
type Product = {
  id: number;
  name: string;
  price: number;
};

type ProductCatalog = Record<string, Product[]>;

const catalog: ProductCatalog = {
  electronics: [
    { id: 1, name: "Laptop", price: 2000 },
    { id: 2, name: "Smartphone", price: 1200 },
  ],
  clothing: [
    { id: 3, name: "T-shirt", price: 50 },
    { id: 4, name: "Jeans", price: 100 },
  ],
};

console.log(catalog.electronics[0].name); // "Laptop"
```

#### Por que usar?

Garante que as chaves são categorias e os valores são listas de produtos válidos.
Ajuda a organizar e consultar dados estruturados.

### 5. Controle de Estoque

Você pode usar Record para criar um sistema que rastreia a quantidade de estoque disponível para cada produto.

```ts
type Stock = Record<string, number>;

const stock: Stock = {
  "product-1": 100,
  "product-2": 50,
  "product-3": 0,
};

console.log(stock["product-1"]); // 100
```

#### Por que usar?

Permite associar o ID de um produto à sua quantidade em estoque.
Simplifica operações como atualização e consulta de estoque.

### 6. Mapear permissões por usuário

Em um sistema de controle de acesso, você pode usar Record para mapear IDs de usuários para suas permissões.

```ts
type Permissions = "read" | "write" | "delete";

type UserPermissions = Record<number, Permissions[]>;

const permissions: UserPermissions = {
  1: ["read", "write"],
  2: ["read"],
  3: ["read", "write", "delete"],
};

console.log(permissions[3]); // ["read", "write", "delete"]
```

#### Por que usar?

Modela permissões de forma clara e segura.
Garante que as permissões associadas a cada usuário sejam válidas.

### 7. Criar logs de eventos

Em sistemas que rastreiam eventos, Record pode ser usado para organizar logs por tipos de eventos.

```ts
type EventLog = {
  timestamp: string;
  details: string;
};

type Logs = Record<string, EventLog[]>;

const systemLogs: Logs = {
  error: [
    { timestamp: "2025-01-01T12:00:00Z", details: "Erro de conexão" },
  ],
  info: [
    { timestamp: "2025-01-01T12:30:00Z", details: "Sistema inicializado" },
  ],
};

console.log(systemLogs.error[0].details); // "Erro de conexão"
```

#### Por que usar?

Garante que os logs sejam organizados por categoria (ex.: error, info).
Facilita a consulta e análise de eventos.

### 8. Mapeamento de códigos de erros

Se sua aplicação retorna códigos de erro específicos, você pode usar Record para mapear códigos para mensagens descritivas.

```ts
type ErrorMessages = Record<number, string>;

const errors: ErrorMessages = {
  404: "Recurso não encontrado",
  401: "Não autorizado",
  500: "Erro interno do servidor",
};

console.log(errors[404]); // "Recurso não encontrado"
```

#### Por que usar?

Permite que erros sejam tratados de forma consistente em todo o sistema.
Facilita a manutenção ao centralizar mensagens de erro.

### Resumo


O `Record<K, T>` é extremamente útil para:

- permite modelar tabelas, mapas e dicionários de maneira explícita.
- garante que tanto as chaves quanto os valores sejam consistentes.
- facilita a reutilização de tipos já existentes para criar estruturas específicas.
- útil em diversos cenários, como logs, mapeamentos e tabelas dinâmicas.


## 7. Extract<T, U>

O `Extract<T, U>` é um *utility type* do TypeScript que extrai apenas os tipos que estão presentes em ambos os conjuntos (T e U). Ele é especialmente útil quando você deseja trabalhar com um subconjunto específico de valores de um tipo mais amplo, garantindo que os valores permitidos sejam estritamente controlados.

### 1. Filtrar eventos permitidos

Imagine que você tenha um conjunto de eventos em um sistema, mas só queira trabalhar com alguns específicos.

```ts
type AllEvents = "click" | "hover" | "focus" | "scroll";
type AllowedEvents = Extract<AllEvents, "click" | "hover">;

const event: AllowedEvents = "click"; // OK
// const event: AllowedEvents = "scroll"; // Erro
```

#### Por que usar?

Restringe o uso a eventos permitidos, ajudando a evitar comportamentos não intencionais.
Facilita a documentação, indicando quais eventos são suportados.

### 2. Controle de Permissões

Em um sistema de controle de acesso, você pode ter várias permissões, mas algumas ações específicas só podem ser executadas por usuários com permissões restritas.

```ts
type AllPermissions = "read" | "write" | "delete" | "execute";
type UserPermissions = Extract<AllPermissions, "read" | "write">;

const userAction: UserPermissions = "read"; // OK
// const userAction: UserPermissions = "execute"; // Erro
```

#### Por que usar?

Garante que ações sensíveis sejam realizadas apenas por usuários com permissões adequadas.
Torna o código mais seguro e fácil de entender.

### 3. Validação de campos permitidos em formulários

Imagine que você tenha um formulário com vários campos, mas apenas alguns deles podem ser validados automaticamente.

```ts
type FormFields = "username" | "password" | "email" | "phone";
type ValidatableFields = Extract<FormFields, "email" | "phone">;

const fieldToValidate: ValidatableFields = "email"; // OK
// const fieldToValidate: ValidatableFields = "username"; // Erro
```

#### Por que usar?

Restringe os campos que podem ser validados automaticamente.
Garante que apenas os campos suportados sejam usados em funções de validação.


### 4. API de requisições permitidas

Se você tem uma API com múltiplos endpoints, mas alguns são exclusivos para determinadas operações, o `Extract` pode filtrar os endpoints permitidos para cada caso.

```ts
type AllEndpoints = "/users" | "/posts" | "/comments" | "/admin";
type PublicEndpoints = Extract<AllEndpoints, "/posts" | "/comments">;

const endpoint: PublicEndpoints = "/posts"; // OK
// const endpoint: PublicEndpoints = "/admin"; // Erro
```

#### Por que usar?

Filtra os endpoints disponíveis para diferentes níveis de acesso.
Facilita a manutenção ao definir claramente os endpoints permitidos.

### 5. Gerenciamento de tipos de notificações

Em um sistema de notificações, você pode ter vários tipos de mensagens, mas deseja trabalhar apenas com notificações críticas.

```ts
type AllNotifications = "info" | "warning" | "error" | "success";
type CriticalNotifications = Extract<AllNotifications, "error" | "warning">;

const notification: CriticalNotifications = "error"; // OK
// const notification: CriticalNotifications = "success"; // Erro
```

#### Por que usar?

Ajuda a focar em mensagens importantes, como erros e alertas.
Evita tratar notificações irrelevantes em cenários críticos.

### 6. Definição de tipos em tabelas dinâmicas

Se você tem uma tabela com várias colunas, mas deseja trabalhar apenas com um subconjunto específico em uma operação, o `Extract` pode ajudar a filtrar as colunas relevantes.

```ts
type TableColumns = "id" | "name" | "email" | "createdAt";
type EditableColumns = Extract<TableColumns, "name" | "email">;

const column: EditableColumns = "email"; // OK
// const column: EditableColumns = "id"; // Erro
```

#### Por que usar?

Restringe as operações a colunas editáveis, melhorando a consistência e segurança.
Facilita a manutenção ao indicar claramente quais colunas são editáveis.

### 7. Filtros de produtos permitidos

Em um sistema de e-commerce, você pode ter vários filtros disponíveis, mas nem todos são aplicáveis a determinadas categorias.

```ts
type AllFilters = "price" | "brand" | "size" | "color";
type ClothingFilters = Extract<AllFilters, "size" | "color">;

const filter: ClothingFilters = "size"; // OK
// const filter: ClothingFilters = "price"; // Erro
```

#### Por que usar?

Limita os filtros disponíveis com base na categoria de produto.
Garante que apenas filtros relevantes sejam exibidos e processados.

### 9. Tipos de eventos permitidos para um componente

Se você tem um componente que aceita diferentes eventos, o Extract pode ajudar a restringir os tipos de eventos que ele manipula.

```ts
type DOMEvents = "click" | "dblclick" | "mouseenter" | "mouseleave";
type ButtonEvents = Extract<DOMEvents, "click" | "dblclick">;

const buttonEvent: ButtonEvents = "click"; // OK
// const buttonEvent: ButtonEvents = "mouseenter"; // Erro
```

#### Por que usar?

Garante que o componente manipule apenas eventos suportados.
Facilita a documentação e o uso do componente.

### Resumo

O `Extract<T, U>` é extremamente útil para:

- restringe os valores permitidos a um subconjunto de tipos válidos.
- torna explícito quais valores podem ser usados em um determinado contexto.
- se o conjunto original mudar, o Extract garante que os subconjuntos se ajustem automaticamente.
- previne o uso de valores não permitidos, melhorando a segurança do código.

## 8. Exclude<T, U>

O `Exclude<T, U>` é um * * do TypeScript que cria um novo tipo removendo os tipos que estão em `U` do conjunto `T`. Ele é útil para criar tipos que filtram valores indesejados ou irrelevantes em um determinado contexto.

### 1. Remover tipos de eventos indesejados

Imagine que você tem uma lista de eventos disponíveis, mas deseja excluir aqueles que não são relevantes.

```ts
type AllEvents = "click" | "hover" | "focus" | "scroll";
type DisallowedEvents = Exclude<AllEvents, "scroll" | "focus">;

const event: DisallowedEvents = "click"; // OK
// const event: DisallowedEvents = "scroll"; // Erro
```

#### Por que usar?

Filtra eventos irrelevantes, garantindo que apenas os permitidos sejam manipulados.
Reduz o risco de manipulação incorreta de eventos não desejados.


### 2. Controle de Permissões

Imagine um sistema de permissões onde você deseja excluir ações sensíveis (como exclusões) de certos usuários.

```ts
type AllPermissions = "read" | "write" | "delete" | "execute";
type RestrictedPermissions = Exclude<AllPermissions, "delete" | "execute">;

const userPermission: RestrictedPermissions = "read"; // OK
// const userPermission: RestrictedPermissions = "delete"; // Erro
```

#### Por que usar?

Impede que usuários acessem ou realizem ações restritas.
Garante que apenas permissões seguras sejam atribuídas.

### 3. Campos opcionais em formulários

Se você tem um formulário com vários campos, mas deseja desabilitar ou remover certos campos em um contexto específico, o `Exclude` pode ser útil.

```ts
type FormFields = "username" | "password" | "email" | "phone";
type EditabledFields = Exclude<FormFields, "email">;

const editableField: EditabledFields = "username"; // OK
// const editableField: EditabledFields = "email"; // Erro
```

#### Por que usar?

Restringe os campos disponíveis para edição ou manipulação.
Facilita a criação de contextos específicos para diferentes tipos de formulários.


### 4. Restrições de filtros em E-commerce

Imagine que você tem uma lista de filtros disponíveis, mas deseja excluir alguns que não se aplicam a certas categorias de produtos.

```ts
type AllFilters = "price" | "brand" | "size" | "color";
type RestrictedFilters = Exclude<AllFilters, "size" | "color">;

const filter: RestrictedFilters = "price"; // OK
// const filter: RestrictedFilters = "size"; // Erro
```

#### Por que usar?

Remove filtros irrelevantes para determinadas categorias de produtos.
Garante que apenas filtros válidos sejam exibidos e processados.

### 5. Restrições de Endpoints

Em uma API, você pode querer limitar os endpoints disponíveis para certos usuários, excluindo aqueles que são administrativos ou perigosos.

```ts
type Endpoints = "/users" | "/posts" | "/admin" | "/logs";
type PublicEndpoints = Exclude<Endpoints, "/admin" | "/logs">;

const endpoint: PublicEndpoints = "/users"; // OK
// const endpoint: PublicEndpoints = "/admin"; // Erro
```

#### Por que usar?

Restringe o acesso de usuários a endpoints sensíveis ou internos.
Facilita a criação de níveis de acesso na API.

### 6. Remoção de tipos em notificações

Se você tem diferentes tipos de notificações, mas deseja excluir aquelas que não devem ser exibidas para certos usuários, o Exclude pode ajudar.

```ts
type Notifications = "info" | "warning" | "error" | "success";
type NonCriticalNotifications = Exclude<Notifications, "error">;

const notification: NonCriticalNotifications = "info"; // OK
// const notification: NonCriticalNotifications = "error"; // Erro
```
#### Por que usar?

Remove notificações desnecessárias ou inadequadas para o contexto.
Torna o sistema de notificações mais direcionado e relevante.


### 7. Excluir propriedades de objetos

Imagine que você tenha um tipo de objeto e deseje excluir certos campos ao criar outro tipo.

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

type PublicUserKeys = Exclude<keyof User, "password">;

const publicKey: PublicUserKeys = "email"; // OK
// const publicKey: PublicUserKeys = "password"; // Erro
```

#### Por que usar?

Remove campos sensíveis ao criar versões públicas de tipos.
Facilita a criação de tipos reutilizáveis para diferentes contextos.


### 8. Configurações de sistema excluindo modos específicos

Se você tem configurações de sistema com vários modos, mas deseja excluir modos avançados para usuários comuns, o Exclude é ideal.

```ts
type Modes = "basic" | "advanced" | "developer";
type RestrictedModes = Exclude<Modes, "developer">;

const mode: RestrictedModes = "basic"; // OK
// const mode: RestrictedModes = "developer"; // Erro
```

#### Por que usar?

Garante que usuários comuns não tenham acesso a modos avançados.
Facilita o controle de permissões de configuração.


### 9. Remover opções de configuração dinâmicas

Em sistemas com várias opções de configuração, você pode querer excluir algumas opções que não são aplicáveis em certos contextos.


```ts
type ConfigOptions = "theme" | "language" | "debugMode" | "apiKeys";
type RestrictedConfig = Exclude<ConfigOptions, "apiKeys" | "debugMode">;

const config: RestrictedConfig = "theme"; // OK
// const config: RestrictedConfig = "apiKeys"; // Erro
```

#### Por que usar?

Remove opções que não fazem sentido ou são perigosas para determinados contextos.
Simplifica a interface de configuração.

### Resumo 

O `Exclude<T, U>` é extremamente útil para:

- impede o uso de tipos indesejados ou não permitidos.
- define explicitamente quais valores ou tipos devem ser excluídos.
- centraliza a lógica de exclusão de tipos, tornando-a fácil de modificar e reaproveitar.
- aplicável em eventos, permissões, formulários, notificações, APIs, e mais.

## 9. NonNullable<T>

O `NonNullable<T>` é um *utility type* do TypeScript que remove os valores `null` e `undefined` de um tipo. Ele é particularmente útil quando você precisa garantir que certos valores não sejam nulos ou indefinidos em cenários onde isso pode causar erros ou comportamento indesejado.

### 1. Garantir dados em variáveis de usuário

Imagine um sistema onde o nome do usuário pode ser null em alguns pontos do fluxo, mas você quer garantir que ele não seja nulo antes de exibir ou processá-lo.

```ts
type User = {
  id: number;
  name: string | null;
};

type NonNullableUser = NonNullable<User["name"]>;

const userName: NonNullableUser = "Alice"; // OK
// const userName: NonNullableUser = null; // Erro
```

#### Por que usar?

Garante que o nome do usuário não seja null ou undefined antes de utilizá-lo.
Previne erros ao tentar acessar métodos de string em um valor nulo.

### 2. Funções que retornam valores obrigatórios

Imagine uma função que busca dados de um banco, onde o retorno pode ser undefined se nenhum dado for encontrado, mas você quer garantir que o valor retornado seja válido antes de usá-lo.

```ts
function getUserName(id: number): string | undefined {
  const users = { 1: "Alice", 2: "Bob" };
  return users[id];
}

type ValidUserName = NonNullable<ReturnType<typeof getUserName>>;

const userName: ValidUserName = "Bob"; // OK
// const userName: ValidUserName = undefined; // Erro
```

#### Por que usar?

Garante que a variável userName tenha um valor válido, reduzindo a necessidade de verificações adicionais.


### 3. Garantir dados após validação

Em sistemas onde os dados podem inicialmente ser nulos ou indefinidos, você pode usar o NonNullable para garantir que eles sejam válidos após uma validação.

```ts
function validateName(name: string | null): NonNullable<string> {
  if (name === null) {
    throw new Error("Nome inválido");
  }
  return name;
}

const validatedName = validateName("Alice"); // OK
// const validatedName = validateName(null); // Erro
```

#### Por que usar?

Reduz o risco de operar em valores inválidos após a validação.
Melhora a segurança do código em fluxos que dependem de dados obrigatórios.

### 4. Trabalhar com propriedades de objetos

Imagine um objeto onde algumas propriedades podem ser undefined, mas você deseja garantir que certas propriedades estejam sempre presentes em um determinado contexto.

```ts
type Config = {
  apiKey?: string;
  endpoint?: string;
};

type RequiredConfig = NonNullable<Config["apiKey"]>;

const apiKey: RequiredConfig = "123456"; // OK
// const apiKey: RequiredConfig = undefined; // Erro
```

#### Por que usar?

Garante que configurações críticas, como uma chave de API, não sejam indefinidas.
Previne falhas em integrações que dependem dessas propriedades.


### 5. Restrições em formulários

Em um formulário, os valores iniciais podem ser nulos ou indefinidos, mas você quer garantir que certos campos sejam preenchidos antes de salvar.

```ts
type FormValues = {
  username: string | undefined;
  email: string | null;
};

type ValidFormValues = {
  username: NonNullable<FormValues["username"]>;
  email: NonNullable<FormValues["email"]>;
};

const validValues: ValidFormValues = {
  username: "Alice", // OK
  email: "alice@example.com", // OK
};
// const invalidValues: ValidFormValues = { username: undefined, email: null }; // Erro
```

#### Por que usar?

Garante que os valores obrigatórios do formulário estejam preenchidos antes de enviar os dados.
Melhora a confiabilidade da aplicação ao processar os dados.


### 6. Garantir respostas de APIs

Ao consumir APIs, algumas propriedades podem ser opcionais. O NonNullable pode garantir que certas respostas sejam sempre válidas antes de serem usadas.

```ts
type ApiResponse = {
  data?: {
    id: number;
    name: string | null;
  };
};

type ValidResponse = NonNullable<ApiResponse["data"]>;

const response: ValidResponse = { id: 1, name: "Alice" }; // OK
// const response: ValidResponse = undefined; // Erro
```

#### Por que usar?

Previne erros causados por dados incompletos ou inválidos recebidos de APIs.
Simplifica a manipulação de dados retornados.


### 7. Trabalhar com Promises

Ao usar `Promise` em sistemas assíncronos, você pode garantir que o valor resolvido não seja nulo ou indefinido. Aqui vamos usar `Awaited` com `NonNullable` para garantir que o retorno não seja nem nulo nem indefinido.

```ts
type FetchResult = string | null | undefined;

type ValidResult = NonNullable<Awaited<FetchResult>>;

async function fetchData(): Promise<ValidResult> {
  return "Fetched Data";
}

fetchData().then((data) => console.log(data)); // OK
```

#### Por que usar?

Garante que o valor resolvido pela Promise seja válido.
Previne problemas em fluxos assíncronos com valores inesperados.

### Resumo

O `NonNullable<T>` é extremamente útil para:

- remove null e undefined, garantindo que os valores sejam válidos.
- evita operações em valores nulos ou indefinidos, como acessos a métodos inexistentes.
- torna explícito que certos valores precisam ser obrigatórios após validação.
- garante consistência em fluxos de dados críticos, como formulários, APIs e configurações.


## 10. ReturnType<T>

O `ReturnType<T>` é um utility type do TypeScript que infere o tipo de retorno de uma função. Ele é especialmente útil para criar tipos derivados dinamicamente com base nas funções existentes, garantindo consistência e evitando duplicação de código.

### 1. Reutilizar o tipo de retorno de funções

Imagine que você tenha uma função que retorna objetos complexos e deseja reutilizar o tipo desses objetos em outras partes do sistema.

```ts
function getUser(): { id: number; name: string } {
  return { id: 1, name: "Alice" };
}

type UserReturnType = ReturnType<typeof getUser>;

const user: UserReturnType = { id: 2, name: "Bob" };
```

### Por que usar?

Evita a necessidade de declarar manualmente o tipo de retorno.
Garante que o tipo reutilizado esteja sempre em sincronia com a função.

### 2. Tipos dinâmicos em APIs

Ao consumir uma API, você pode usar o ReturnType<T> para inferir o tipo do retorno de uma função que busca dados.

```ts
function fetchUser(): Promise<{ id: number; email: string }> {
  return Promise.resolve({ id: 1, email: "user@example.com" });
}

type FetchUserResponse = ReturnType<typeof fetchUser>;

async function processUser(): Promise<Awaited<FetchUserResponse>> {
  const user = await fetchUser();
  console.log(user.email); // OK
  return user;
}
```

#### Por que usar?

Simplifica a definição de tipos baseados em funções assíncronas.
Torna o código mais limpo e fácil de manter.



### 3. Garantir consistência em Serviços

Em aplicações grandes, você pode garantir que o retorno de um serviço ou repositório seja consistente em toda a aplicação usando ReturnType.

```ts
class UserService {
  getUserById(id: number): { id: number; name: string } {
    return { id, name: "User" };
  }
}

type UserServiceReturn = ReturnType<UserService["getUserById"]>;

const user: UserServiceReturn = { id: 1, name: "Alice" }; // OK
```

#### Por que usar?

Evita inconsistências ao reutilizar o mesmo tipo em diferentes partes da aplicação.
Facilita refatorações, pois os tipos derivados são ajustados automaticamente.

### 4.  Inferir tipos em testes

Ao testar funções ou serviços, você pode usar o ReturnType<T> para garantir que os objetos de teste correspondam ao retorno esperado.

```ts
function createProduct(): { id: number; name: string; price: number } {
  return { id: 1, name: "Laptop", price: 1500 };
}

type Product = ReturnType<typeof createProduct>;

const mockProduct: Product = { id: 2, name: "Phone", price: 800 }; // OK
// const invalidProduct: Product = { id: 2, name: "Phone" }; // Erro: Faltando a propriedade "price"
```

#### Por que usar?

Garante que os mocks em testes estejam alinhados com o retorno da função real.
Reduz erros causados por discrepâncias nos tipos.

### 5. Criação de interfaces para componentes frontend

Se você tem uma função que fornece dados para um componente, o ReturnType pode ser usado para inferir dinamicamente o tipo das propriedades do componente.

```ts
function getComponentProps(): { title: string; description: string } {
  return { title: "Welcome", description: "This is a description." };
}

type ComponentProps = ReturnType<typeof getComponentProps>;

function Component(props: ComponentProps): JSX.Element {
  return (
    <div>
      <h1>{props.title}</h1>
      <p>{props.description}</p>
    </div>
  );
}

<Component title="Hello" description="Dynamic typing with ReturnType" />;
```

#### Por que usar?

Garante que o componente e a função fornecedora de dados estejam sincronizados.
Evita duplicação de tipos ao definir manualmente as propriedades.

### 6. Trabalhar com Middlewares ou Interceptores

Em sistemas que usam middlewares (como NestJS ou Express), o ReturnType pode inferir o tipo de dados retornados para evitar duplicação de tipos.

```ts
function processRequest(): { status: number; message: string } {
  return { status: 200, message: "OK" };
}

type MiddlewareResponse = ReturnType<typeof processRequest>;

function middleware(): MiddlewareResponse {
  return processRequest();
}
```

#### Por que usar?

Facilita a criação de middlewares consistentes.
Reduz a complexidade ao lidar com tipos derivados de funções.

### 7. Validação de dados com Serviços

Em sistemas onde você precisa validar dados retornados de um serviço antes de processá-los, o `ReturnType` pode ajudar.

```ts
function validateUser(): { id: number; isValid: boolean } {
  return { id: 1, isValid: true };
}

type ValidationResult = ReturnType<typeof validateUser>;

function logValidationResult(result: ValidationResult): void {
  console.log(`User ID: ${result.id}, Valid: ${result.isValid}`);
}

logValidationResult({ id: 2, isValid: false }); // OK
```

#### Por que usar?

Garante que os tipos derivados de funções de validação sejam consistentes em todo o sistema.
Facilita o rastreamento e log de resultados validados.

### 8. Inferir Tipos em chamadas dinâmicas

Em sistemas que chamam funções dinamicamente, como carregadores de módulos, o `ReturnType` pode ajudar a inferir os tipos sem precisar defini-los manualmente.

```ts
function loadModule(): { moduleName: string; version: string } {
  return { moduleName: "ModuleA", version: "1.0.0" };
}

type ModuleInfo = ReturnType<typeof loadModule>;

const module: ModuleInfo = { moduleName: "ModuleB", version: "2.0.0" }; // OK
```

#### Por que usar?

Evita duplicação de tipos ao trabalhar com módulos carregados dinamicamente.
Garante consistência nos tipos em sistemas modulares.


## 11. Awaited 

O `Awaited<T>` é um utility type introduzido no TypeScript para ajudar a manipular tipos relacionados a *Promises* ou valores que podem ser resolvidos de forma assíncrona.

Ele é usado para extrair o tipo do valor que uma Promise resolve ou para lidar com tipos que podem ser Promises ou valores síncronos. Simplificando: `Awaited<T>` obtém o tipo "interno" que uma `Promise` resolve.

### Por que usar Awaited?

Facilitar a manipulação de tipos assíncronos.
Tornar o código mais legível e explícito ao trabalhar com Promise ou funções assíncronas.
Evitar a necessidade de lidar manualmente com Promise aninhadas.

### Como Awaited<T> funciona?

Definição do Awaited (simplificada):

```ts
type Awaited<T> =
  T extends PromiseLike<infer U> ? Awaited<U> : T;
```

### Explicação:

- T extends PromiseLike<infer U>:
  - Se T é uma Promise ou algo similar, ele extrai o tipo interno (chamado de U).
- Awaited<U>:
  - Se U também é uma Promise, ele continua "desembrulhando" até chegar a um tipo que não seja uma Promise.
- Caso contrário, retorna T:
  - Se T não é uma Promise, retorna o próprio tipo.


### 1. Resolver o tipo de uma promise

Imagine que você tenha uma função que retorna uma Promise.

```ts
type MyPromise = Promise<string>;

type Resolved = Awaited<MyPromise>; // Resolved é "string"
```

Sem `Awaited`, seria necessário desembrulhar manualmente o tipo da `Promise`. Com ele, o tipo interno é extraído automaticamente.



### 2. Função Assíncrona

Quando você usa async/await, o TypeScript retorna automaticamente uma Promise. O Awaited pode ser usado para extrair o tipo resolvido.

```ts
async function getData(): Promise<number> {
  return 42;
}

type ResolvedType = Awaited<ReturnType<typeof getData>>;
// ResolvedType é "number"
```


### 3. Lidar com Promises aninhadas

Se você tem Promises aninhadas, Awaited resolve todos os níveis de aninhamento.

```ts
type NestedPromise = Promise<Promise<Promise<number>>>;

type Resolved = Awaited<NestedPromise>; // Resolved é "number"
```

O `Awaited` continua "desembrulhando" até chegar ao tipo final, no caso, number.



### 4. Combinação com valores síncronos e assíncronos

O `Awaited` também lida com tipos que podem ser uma `Promise` ou um valor direto.

```ts
type MaybePromise<T> = T | Promise<T>;

type Result = Awaited<MaybePromise<string>>;
// Result é "string"
```

### 5. Trabalhando com Arrays e Promises

Se você tem uma função que retorna um array de Promises, o Awaited pode ajudar a extrair o tipo do valor resolvido.

```ts
type PromiseArray = Promise<string>[];

type ResolvedArray = Awaited<PromiseArray[number]>;
// ResolvedArray é "string"
```

- PromiseArray[number]: Acessa o tipo de um elemento do array.
- Awaited<...>: Resolve o tipo da Promise.

### Quando Usar Awaited?

#### Funções Assíncronas:

Sempre que você quiser trabalhar com o tipo resolvido de uma Promise retornada por uma função.

#### Lidar com Promises aninhadas:

Quando o tipo retornado é uma Promise<Promise<T>> e você precisa do tipo mais interno.

#### Generalizar funções que aceitam Promises ou valores diretos:

Para funções que podem lidar com valores síncronos ou assíncronos.

#### Validação de tipos em APIs:

Ao lidar com chamadas de API assíncronas, você pode usar Awaited para determinar o tipo final dos dados.
