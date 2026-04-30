# Instruções para o Claude Code

> Regras marcadas com `[TS]` se aplicam apenas a projetos TypeScript.

## Estilo de Código

### Blocos condicionais sempre com chaves
Nunca escrever `if` inline sem chaves. Sempre usar bloco explícito.

```js
// ❌ Errado
if (isValid) doSomething();

// ✅ Certo
if (isValid) {
  doSomething();
}
```

### Sem comentários no código
O código deve ser autoexplicativo por meio de nomes claros de funções, variáveis e estrutura. Comentários são desnecessários e devem ser evitados.

```js
// ❌ Errado
// Filtra apenas os usuários ativos
const result = users.filter(u => u.active);

// ✅ Certo
const activeUsers = users.filter(user => user.isActive);
```

### Funções com responsabilidade única
Cada função deve fazer apenas uma coisa. Se uma função precisa coordenar várias etapas, ela deve agir como orquestradora — delegando o trabalho para subfunções nomeadas, sem implementar a lógica inline.

```js
// ❌ Errado — mistura coordenação com implementação
async function handleCreateOrder(request) {
  const body = await request.json();
  if (!body.items || body.items.length === 0) {
    throw new Error("No items");
  }
  const order = { id: crypto.randomUUID(), items: body.items };
  await db.insert("orders", order);
  await emailService.send({ to: body.email, subject: "Pedido confirmado" });
  return { success: true, orderId: order.id };
}

// ✅ Certo — orquestradora delega para funções com responsabilidade única
async function handleCreateOrder(request) {
  const orderData = parseOrderRequest(request);
  const validatedOrder = validateOrder(orderData);
  const savedOrder = await saveOrder(validatedOrder);
  await sendOrderConfirmation(savedOrder);
  return buildOrderResponse(savedOrder);
}
```

O sinal de que uma função está fazendo coisas demais é quando seu nome precisaria de "e" para descrever o que ela faz — ex: `validateAndSave`, `fetchAndFormat`.

### Nomes completos em variáveis, constantes e parâmetros
Nunca usar nomes de uma única letra ou abreviações em variáveis, constantes ou parâmetros de funções — inclusive em callbacks e arrow functions.

```js
// ❌ Errado
const userIds = users.map(u => u.id);
const total = items.reduce((acc, i) => acc + i.price, 0);

// ✅ Certo
const userIds = users.map(user => user.id);
const total = items.reduce((accumulator, item) => accumulator + item.price, 0);
```

### Early return ao invés de else aninhado
Retornar ou lançar erro o quanto antes para evitar aninhamento. Nunca usar `else` quando o bloco anterior já encerra o fluxo com `return` ou `throw`.

```js
// ❌ Errado
function getDiscount(user) {
  if (user.isActive) {
    if (user.isPremium) {
      return 0.3;
    } else {
      return 0.1;
    }
  } else {
    return 0;
  }
}

// ✅ Certo
function getDiscount(user) {
  if (!user.isActive) {
    return 0;
  }

  if (user.isPremium) {
    return 0.3;
  }

  return 0.1;
}
```

### Booleanos com prefixo descritivo
Variáveis e propriedades booleanas devem sempre usar prefixos como `is`, `has`, `should` ou `can` para deixar claro que se trata de um valor verdadeiro/falso.

```js
// ❌ Errado
const active = true;
const premiumUser = false;
const emailSent = true;

// ✅ Certo
const isActive = true;
const isPremiumUser = false;
const hasEmailBeenSent = true;
```

### `[TS]` Tipagem explícita de parâmetros e retorno de funções
Sempre tipar os parâmetros e o retorno de funções. Quando os parâmetros descrevem uma entidade, agrupá-los em um `type` ou `interface` nomeado. Identificadores simples como `id` ou `slug` podem ser passados individualmente.

```ts
// ❌ Errado
async function createUser(name, age, gender) {
  ...
}

// ✅ Certo — parâmetros descrevem uma entidade, usar objeto tipado
type CreateUserParams = {
  name: string;
  age: number;
  gender: string;
}

async function createUser(createUserParams: CreateUserParams): Promise<User> {
  ...
}

// ✅ Certo — identificador simples, passar individualmente
async function getUserById(id: string): Promise<User> {
  ...
}

// ✅ Certo — identificador + entidade, combinar os dois estilos
async function updateUser(id: string, updateUserParams: UpdateUserParams): Promise<User> {
  ...
}
```