# Decoradores TypeScript 
_TypeScript Decorators_

Com a introdução de Classes em TypeScript e ES6, agora existem certos cenários que exigem recursos adicionais para dar suporte a anotação ou modificação de classes e membros de classe.

Os decoradores fornecem uma maneira de adicionar anotação e uma sintaxe de metaprogramação para declarações de classes e membros. Decoradores são uma [proposta de estágio 2](https://github.com/tc39/proposal-decorators) até a data desta tradução (May 20, 2022) para JavaScript e estão disponíveis como um recurso experimental do TypeScript.

<details open>
<summary>Observação</summary>
Decoradores são um recurso experimental que pode mudar em versões futuras.
</details>

Para habilitar o suporte experimental para decoradores, você deve habilitar a opção [experimentalDecorators](https://www.typescriptlang.org/tsconfig#experimentalDecorators) do compilador ou no seu arquivo de configuração `tsconfig.json`

**Linha de comando:**

```sh
tsc --target ES5 --experimentalDecorators
```


**tsconfig.json:**

```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
```


## Decoradores
Um decorador é um tipo especial de declaração que pode ser anexada a uma declaração de classe, método, acessador, propriedade ou parâmetro. Decoradores usam o formulário `@expression`, onde `expression` deve avaliar uma função que será chamada em tempo de execução com informações sobre a declaração decorada.


Por exemplo, dado o decorador `@sealed`, podemos escrever a função sealed da seguinte forma:
```ts
function sealed(target) {
  // Faça algo com target
}
```

  

## Fábricas de decoradores

Se quisermos personalizar como um decorador é aplicado a uma declaração, podemos escrever uma fábrica (factory) de decoradores.

Uma *fábrica de decoradores* é simplesmente uma função que retorna a expressão que será chamada pelo decorator em tempo de execução.


Podemos escrever da seguinte forma:
```ts
function color(valor: string) {
  // esta é a fábrica

  return function (target) {
    // este é o decorador
  }
}
```

## Composição de decoradores
Vários decoradores podem ser escritos e aplicados a uma declaração, por exemplo, em uma única linha:
```ts
@f @g x
```

Em várias linhas
```ts
@f

@g

x
```

Quando vários decoradores se aplicam a uma única declaração, sua avaliação é semelhante à composição de funções em matemática. Neste modelo, ao compor as funções f e g, a composição resultante ( *f* ∘ *g* )( *x* ) é equivalente a *f* ( *g* ( *x* )).

Assim, as etapas a seguir são executadas ao avaliar várioss decoradores em uma única declaração no TypeScript:

1. As expressões para cada decorador são avaliadas de cima para baixo.
2. Os resultados sÃo então chamados como funções de baixo para cima.
3. Se fôssemos usar fábricas de decoradores, podemos observar essa ordem de avaliação com o seguinte exemplo:
```ts
function first() {
  console.log('first(): factory evaluated')
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log('first(): called')
  }
}

function second() {
  console.log('second(): factory evaluated')
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log('second(): called')
  }
}

class ExampleClass {
  @first()
  @second()
  method() {}
}
```
[Experimentar](https://www.typescriptlang.org/play/#code/PTAEAEFMA8AdIE4EsC2kB2AXAhgGwCKQDGA9gtpmQM4BQAZgK7pGZInqh1IJWYAUASlABvGqFCl0VErkgA6XCQDmfAERce-AQC5O2FmQCeoSADc8DCpAAmqgQG4xoBJEwMEHRs1btQfHAhKrrrY6IYANKCwCCTwCJiGANKQhrq8yOhKkdaQVETIsJQIugAKMXEJhHkFRUKi4uKS0rIKymoavIK6RHiytg5OAL6OgzT0TCxsHFTE7NaCIk5NMvKKKqozkvM6egYIxmYWVv2O4i5uHpwTPhz+2IHBoKERUeWICcmpoOlImdm5+SQhTIpTe8UMVUBwIQdScjXYzVWbQ2s3Q226vRsdlOoGGNFGNCIuGwVCooAAotBsChYLIAMLE0mLcTgDpaJzgTZzQRONCYAAWJG2IlGgyAA)

Com isso, vemos no console:
```ts
first(): factory evaluated

second(): factory evaluated

second(): called

first(): called
```


## Avaliação do decorador
Há uma ordem bem definida de como os decoradores aplicados a várias declaração dentro de uma classe são aplicados:

1. *Parameter Decorators*, seguidos por *Method*, *Accessor* ou *Property Decorators* são aplicados para cada membro de instância.
2. *Parameter Decorators*, seguidoss popr *Method*, *Accessor* ou *Property Decorators* são aplicados para cada membro estático.
3. *Parameter Decorators* são aplicados para o construtor.
4. *Class Decorators* são aplicados para a classe.

## Decoradores de classe
Um *Class Decorator* é declarado imediatamente antes de uma declaração de classe. O decorador de classe é aplicado ao construtor da classe e pode ser usado para observar, modificar ou substituir uma definição de classe. Um *class decorator* não pode ser usado em um arquivo de declaração ou em qualquer outro contexto de ambiente (como em uma *declare class*).

A expressão para o *class decorator* será chamada como uma função em tempo de execução, com o construtor da classe decorada com seu único argumento.

Se o class decorator retornar um valor, ele substituirá a declaração de classe pela função construtora fornecida.

<details open>
<summary>Observação</summary>
Caso você opte por retornar uma nova função construtora, deve-se tomar cuidado para manter o protótipo original. A lógica que aplica os decoradores em tempo de execução **não** fará isso por você.
</details>

O seguinte é um exemplo de um class decorator (`@sealed`) aplicado a classe `BugReport`.
```ts
@sealed
class BugReport {
  type = 'report'
  title: string
  constructor(t: string) {
    this.title = t
  }
}
```
[Experimentar](https://www.typescriptlang.org/play/#code/PTAEAEFMA8AdIE4EsC2kB2AXAhgGwCKQDGA9gtpmQM4BQAZgK7pGZInqhWR6QAmAFKXRVMCBizIAuUADEmLNugCUoAN41QoAPIAjAFbFMAOi55B7EWIkIlAbg3b9hk91znho8ZQRHYCEpSYAJ7wdjQAvjQgoAC0cUQMmHExNOCmuHw0RLjYVFSgAEIMAOYASpCwZJhqDsHwoAC8oABECBVVzfaarJgZ0pZI6MX2DkKWXmT8mP2ig8Uq6prdABZIVEY9GY2gmF2gkeFAA)

Podemos definir o decorador `@sealed` usando a seguinte declaração de função:
```ts
function sealed(constructor: Function) {
  Object.seal(constructor)
  Object.seal(constructor.prototype)
}
```

Quando `@sealed` for executado, ele selará tanto o construtor quanto seu protótipo e, portanto, impedirá que qualquer funcionalidade adicional seja adicionada ou removida desta classe durante o tempo de execução, acessando `BugReport.prototype` ou definindo propriedades em `BugReport` em si (observe que as classes ES2015 são realmente apenas açucar sintático para funções de construtor baseadas em protótipo). Este decorator **não** impede que classes sejam sub-classes de `BugReport`.

A seguir, temos um exemplo de como substituir o construtor para definir novos padrões.
```ts
function reportableClassDecorator<T extends {new (...args: any[]): {}}>(
  constructor: T
) {
  return class extends constructor {
    reportingURL = 'http://www...'
  }
}

@reportableClassDecorator
export class BugReport {
  type = 'report'
  title: string
  constructor(t: string) {
    this.title = t
  }
}

const bug = new BugReport('Precisa do modo escuro')
console.log(bug.title) // Imprime "Precisa do modo escuro"
console.log(bug.type) // Imprime "report"

// Observe que o decorador não altera o tipo do TypeScript
// e, portanto, a nova propriedade `reporting URL` _não é_
// conhecida pelo sistema de tipos:
bug.reportingURL
// Property 'reportingURL' does not exist on type 'BugReport'.
```

Caso isso seja um problema, por exemplo, quando a classe decorada está em outro local onde você não possui acesso, isso pode ser resolvido criando uma interface usando o mesmo nome da classe decorada:
```ts
interface BugReport {
  reportingURL: string
}
```


## Decoradores de método
Um *Method Decorator* é declarado imediatamente antes de uma declaração de método. O decorador é aplicado ao *PropertyDescription* *(descritor de propriedade)* do método e pode ser usado para observar, modificar ou substituir uma definição de método. Um decorador de método não pode ser usado em um arquivo de declaração, em uma sobrecarga ou em qualquer outro contexto de ambiente (como em uma classe declarada usando `declare`).

A expressão para o decorador de método será chhamada como uma função em tempo de execução, com os três argumentos a seguir:
1. A função construtora da classe para um membro estático ou o protótipo da classe para um membro de instância.
2. O nome do membro.
3. O *property descriptor* para o membro.

<details open>
O *PropertyDescriptor* será `undefined`se o destino do script for menor que `ES5`.
</details>

Se o decorador do método retornar um valor, ele será usado como descritor de propriedade do método.

**Tabela de conteúdo**
<summary>Observação</summary>
O valor de retorno será ignorado se o destino do seu script for menor que `ES5`.
</details>


Veja a seguir um exemplo de decorador de método (`@enumerable`) aplicado a um método na classe `Greeter`:
```ts
class Greeter {
  greeting: string
  constructor(message: string) {
    this.greeting = message
  }
  @enumerable(false)
  greet() {
    return 'Hello, ' + this.greeting
  }
}
```

Podemos definir o decorador `@enumerable` usando a seguinte declaração de função:
```ts
function enumerable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.enumerable = value
  }
}
```

O decorador `@enumerable(false)` aqui é uma fábrica de decoradores. Quando o decorador `@enumerable(false)` é chamado, ele modifica a propriedade enumerável do descritor de propriedade.

## Decoradores de Acessor (acessadores)
Um *Decorador de Acessador* é declarado logo antes de uma declaração de acessador. (`get` ou `set`). O decorador do acessador é aplicado ao descritor de propriedade do acessador e pode ser usado para observar, modificar ou substituir as definições de um acessador. Um decorador de acesso não pode ser usado em um arquivo de declaração ou em qualquer outro contexto de ambiente (como em uma classe de declaração).

<details open>
<summary>Observação</summary>
O TypeScript não permite decorar o acessador `get` e `set` para um único membro. Em vez disso, todos os decoradores do membro devem ser aplicados ao primeiro acessador especificado na ordem do documento. Isso ocorre porque os decoradores se aplicam a um descritor de propriedade, que combbina o acessador `get` e `set` , e não cada declaração separadamente.
</details>

A expressão para o decorador do acessador será chamada como uma função em tempo de execução, com os três argumentos a seguir:
1. A função construtora (*Constructor*) da classe para um membro estático ou o protótipo da classe para um membro de instância.
2. O nome do membro. (*Nome da classe*)
3. O descritor de propriedade (*PropertyDescriptor*) para o membro.
  

<details open>
<summary>Observação</summary>
O descritor de propriedade (*PropertyDescriptor*) será `undefined` se a propriedade `"target”` do seu `tsconfig.json` for menor que `”ES5"`
</details>

Se o decorador do acessador retornar um valor, ele será usado como o descritor de propriedade do membro.

<details open>
<summary>Observação</summary>
O valor de retorno será ignorado se o `“target”` do seu `tsconfig.json` for menor que `“ES5”`.
</details>

Veja a seguir um exemplo de um decorador de acesso (`@configurable`) aplicado a um membro da classe `Point`:
```ts
class Point {
  private _x: number
  private _y: number
  constructor(x: number, y: number) {
    this._x = x
    this._y = y
  }
  @configurable(false)
  get x() {
    return this._x
  }
  @configurable(false)
  get y() {
    return this._y
  }
}
```

Podemos definir o decorador `@configurable` usando a seguinte declaração de função:
```ts
function configurable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    descriptor.configurable = value
  }
}
``` 

## Decoradores de propriedade
Um *Property Decorator* é declarado logo antes de uma declaração de propriedade. Ele não pode ser usado em um arquivo de declaração ou em qualquer outro contexto de ambiente (como em uma declaração de classe).

A expressão para o *property decorator* será chamada como uma função em tempo de execução, com os dois argumentos a seguir:
1. A função construtora da classe para um membro estático ou o protótipo da classe para um membro de instância.
2. O nome do membro.

<details open>
<summary>Observação</summary>
Um *Property Descriptor* não é fornecido como um argumento para um *Property Decorator* devido à forma como os *Property Decorator* são inicializados no TypeScript. Isso ocorre porque atualmente não há nenhum mecanismo para descrever uma propriedade de instância ao definir membros de um protótipo e nenhuma maneira de observar ou modificar o inicializador de uma propriedade. O valor de retorno também é ignorado. Como tal, um *Property Decorator* só pode ser usado para observar que uma propriedade de um nome específico foi declarada para uma classe.
</details>

Podemos usar essas informações para registrar metadados sobre a propriedade, como no exemplo a seguir:
```ts
class Greeter {
  @format('Hello, %s')
  greeting: string

  constructor(message: string) {
    this.greeting = message
  }

  greet() {
    let formatString = getFormat(this, 'greeting')
    return formatString.replace('%s', this.greeting)
  }
}
```

Podemos então definir o decorador `@format` e as funções `getFormat` usando as seguintes declarações de função:
```ts
import 'reflect-metadata'

const formatMetadataKey = Symbol('format')

function format(formatString: string) {
  return Reflect.metadata(formatMetadataKey, formatString)
}

function getFormat(target: any, propertyKey: string) {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey)
}
```

O decorador `@format("Hello, %s")` aqui é uma fábrica de decoradores. Quando `@format("Hello, %s")` é chamado, ele adiciona uma entrada de metadados para a propriedade usando a função `Reflect.metadata` da biblioteca `reflect-metadata`. Quando `getFormat` é chamado, ele lê o valor de metadados para o formato.

> [!NOTE]- Observação
> Este exemplo requer a biblioteca `reflect-metadata`. Consulte [Metadados](https://www.typescriptlang.org/docs/handbook/decorators.html#metadata) para obter mais informações sobre a biblioteca `reflect-metadata`.

## Decoradores de Parâmetros
Um *Parameter Decorator* é declarado logo antes de uma declaração de parâmetro. Ele é aplicado à função de um construtor de classe ou declaração de método. Um decorador de parâmetro não pode ser usado em um arquivo de declaração, uma sobrecarga ou em qualquer outro contexto de ambiente (como em uma classe de declaração).

A expressão para o decorador de parâmetros será chamada como uma função em tempo de execução, com os três argumentos a seguir:
1. A função construtora da classe para um membro estático ou o protótipo da classe para um membro de instância.
2. O nome do membro.
3. O índice ordinal do parâmetro na lista de parâmetros da função.


<details open>
<summary>Observação</summary>
Um *parameter decorator* só pode ser usado para observar que um parâmetro foi declarado em um método.
</details>

O valor de retorno do decorador de parâmetro é ignorado.

Veja a seguir um exemplo de decorador de parâmetro (`@required`) aplicado ao parâmetro de um membro da classe `BugReport`:
```ts
class BugReport {
  type = 'report'
  title: string

  constructor(t: string) {
    this.title = t
  }

  @validate
  print(@required verbose: boolean) {
    if (verbose) {
      return `type: ${this.type}\ntitle: ${this.title}`
    } else {
      return this.title
    }
  }
}
```

Podemos então definir os decoradores `@required` e `@validate` usando as seguintes declarações de função:
```ts
import 'reflect-metadata'

const requiredMetadataKey = Symbol('required')

function required(
  target: Object,
  propertyKey: string | symbol,
  parameterIndex: number
) {
  let existingRequiredParameters: number[] =
    Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || []

  existingRequiredParameters.push(parameterIndex)

  Reflect.defineMetadata(
    requiredMetadataKey,
    existingRequiredParameters,
    target,
    propertyKey
  )
}

function validate(
  target: any,
  propertyName: string,
  descriptor: TypedPropertyDescriptor<Function>
) {
  let method = descriptor.value!

  descriptor.value = function () {
    let requiredParameters: number[] = Reflect.getOwnMetadata(
      requiredMetadataKey,
      target,
      propertyName
    )

    if (requiredParameters) {
      for (let parameterIndex of requiredParameters) {
        if (
          parameterIndex >= arguments.length ||
          arguments[parameterIndex] === undefined
        ) {
          throw new Error('Missing required argument.')
        }
      }
    }

    return method.apply(this, arguments)
  }
}
```

O decorador `@required` adiciona uma entrada de metadados que marca o parâmetro como obrigatório. O decorador `@validate` então envolve o método de saudação existente em uma função que valida os argumentos antes de invocar o método original.

<details open>
<summary>Observação</summary>
Este exemplo requer a biblioteca `reflect-metadata`. Consulte [Metadados](https://www.typescriptlang.org/docs/handbook/decorators.html#metadata) para obter mais informações sobre a biblioteca `reflect-metadata`.
</details>

## Metadados
Alguns exemplos usam a biblioteca `reflect-metadata` que adiciona um *polyfill* para uma [API de metadados experimental](https://github.com/rbuckton/ReflectDecorators). Esta biblioteca ainda não faz parte do padrão *ECMAScript (JavaScript)*. No entanto, assim que os decoradores forem oficialmente adotados como parte do padrão *ECMAScript*, essas extensões serão propostas para adoção.

Você pode instalar esta biblioteca via npm:
```bash
npm i reflect-metadata --save
```

O TypeScript inclui suporte experimental para emitir certos tipos de metadados para declarações que possuem decoradores. Para habilitar esse suporte experimental, você deve definir a opção do compilador (`”compilerOptions”`) [`emitDecoratorMetadata`](https://www.typescriptlang.org/tsconfig#emitDecoratorMetadata) na linha de comando ou em seu `tsconfig.json`:

```bash
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata
```

**tsconfig.json**
```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

Quando ativado, desde que a biblioteca de `reflect-metadata` tenha sido importada, informações adicionais sobre o tipo de tempo de design serão expostas no tempo de execução.

Podemos ver isso em ação no exemplo a seguir:
```ts
import 'reflect-metadata'

class Point {
  constructor(public x: number, public y: number) {}
}

class Line {
  private _start: Point

  private _end: Point

  @validate
  set start(value: Point) {
    this._start = value
  }

  get start() {
    return this._start
  }

  @validate
  set end(value: Point) {
    this._end = value
  }

  get end() {
    return this._end
  }
}

function validate<T>(
  target: any,
  propertyKey: string,
  descriptor: TypedPropertyDescriptor<T>
) {
  let set = descriptor.set!

  descriptor.set = function (value: T) {
    let type = Reflect.getMetadata('design:type', target, propertyKey)

    if (!(value instanceof type)) {
      throw new TypeError(`Invalid type, got ${typeof value} not ${type.name}.`)
    }

    set.call(this, value)
  }
}

const line = new Line()

line.start = new Point(0, 0)

// @ts-ignore
// line.end = {}
// Fails at runtime with:
// > Invalid type, got object not Point
```

O compilador TypeScript injetará informações de tipo em tempo de design usando o decorador `@Reflect.metadata`. Você pode considerá-lo o equivalente ao seguinte TypeScript:
```tsx
class Line {
  private _start: Point

  private _end: Point

  @validate
  @Reflect.metadata('design:type', Point)
  set start(value: Point) {
    this._start = value
  }

  get start() {
    return this._start
  }

  @validate
  @Reflect.metadata('design:type', Point)
  set end(value: Point) {
    this._end = value
  }

  get end() {
    return this._end
  }
}
```

<details open>
<summary>Observação</summary>
Os metadados do decoradores são um recurso experimental e podem apresentar alterações importantes em versões futuras.
</details>