title: Flowscript
author:
  name: Vladimir Kurchatkin
  twitter: vkurchatkin
  github: vkurchatkin
output: index.html
controls: true
style: style.css

--

# Flowscript

--

### Что это

* Flow
* ES201x
* async/await
* идиомы и паттерны

--

### Flow

Статический анализатор JS-кода

* От фейсбука
* Как Тайпскрипт, но не совсем
* Только compile-time конструкции

```sh
npm install --save-dev flow-bin
```

---

### Типы есть

* Можно их держать в голове
* Можно их записать в readme
* Можно использовать jsdoc
* А можно записать в формате, пригодном для автоматической валидации

---

### Пример

```js
type User = {
  id: string,
  login: string,
  passwordHash: string,
  createdAt: number,
  updatedAt: number
};

function printUser(user: User) {
  console.log(`${user.log} created at ${createdAt}`);
}

printUser({
  id: '1',
  login: 'foo',
  passwordHash: 'bar',
  createdAt: Date.now(),
  updatedAt: Date.now()
});

```

---

### Что в коробке

* Структурные типы
* Классы
* Интерфейсы
* Null-safety
* Дженерики
* Type inference
* Type refinement

---

### Дженерики и Null-safety

* Дженерики и Null-safety — главный профит статической типизации
* Tony Hoare: «I call it (nulls) my billion-dollar mistake»
* Flow, Swift, Rust, Kotlin — хорошо
* Go, Java, Scala — плохо

---

### Дженерики

```js
type Int<T: string> = {
  val: number,
  unit: T
};

function add<T: string>(a: Int<T>, b: Int<T>): Int<T> {
  return { val: a.val + b.val, unit: a.unit };
}


function meters(val: number): Int<'METERS'> {
  return {
    val,
    unit: 'METERS'
  };
}

function miles(val: number): Int<'MILES'> {
  return {
    val,
    unit: 'MILES'
  };
}


add(meters(300), miles(10)); // <-- ошибка
```

Flow поддерживает ковариантность и контрвариантность.

---

### Null-safety

```js
function test(a: ?string) {
  a.toLowerCase(); // <-- нельзя

  if (a) {
    a.toLowerCase(); // <-- можно
  }
}
```

---

### Паттерны flow: Enum

Можно забыть про `constants.js`

```js
type MyEnum =
  | 'FOO'
  | 'BAR'
  | 'BAZ'
;

const val: MyEnum = 'FOO';
```

Для продвинутых:

```js
const MyEnum = {
  FOO: 'FOO',
  BAR: 'BAR',
  BAZ: 'BAZ'
};

type MyEnumT = $Keys<typeof MyEnum>

```

---

### Паттерны Flow: Disjoint union

Полиморфизм в стиле OCaml:

```js
type Phone =
  | { os: 'android', runJava: () => {} }
  | { os: 'ios', runSwift: () => {} }
;

function runCode(phone: Phone) {
  if (phone.os === 'android') {
    phone.runJava(); // <-- безопасно
  } else {
    phone.runSwift();
  }
}
```

---

### Паттерны Flow: Result

`throw` и `reject` только для исключительных ситуаций

```js
type Result<T> =
  | { success: true, val: T}
  | { success: false, error: string }
;

function divide(a: number, b: number): Result<number> {
  if (b !== 0) {
    return { success: true, val: a / b };
  }

  return { success: false, error: 'DIVISION_BY_ZERO'
  };
}

const r = divide(foo, bar);

if (!r.success) {
  console.log(r.error); // <-- flow знает нужный вариант
} else {
  console.log(r.val);
}
```

---

### Паттерны Flow: Store

Дано:

```js
type Store<T> = {
  get(key: string): ?T;
  put(key: string, val: T): void;
}

type Doc = {
  id: string,
  login: string
};

```

Надо добавить новое обязательное поле `role`:

```js
type Role = 'admin' | 'user';
```

---

### Паттерны Flow: Store

Плохо:

```js
type Store<T> = {
  get(key: string): ?T;
  put(key: string, val: T): void;
}

type Doc = {
  id: string,
  login: string,
  role: Role // <- небезопасно читать
};

```

---

### Паттерны Flow: Store

Хорошо:

```js
type Store<+R, -W> = {
  get(key: string): ?R;
  put(key: string, val: W): void;
}

type RDoc = {
  id: string,
  login: string,
  role?: Role // <-- при чтении поля может не быть
};

type WDoc = {
  id: string,
  login: string,
  role: Role // <-- при записи поле обязательно
};
```

---

### Паттерны Flow: Restructuring

```js

type FooBar = {
  foo: string,
  bar: number
}

function serialize(val: FooBar): string {
  // в val могут быть и другие поля, это вредно для forward compatibility
  const { foo, bar } = val;
  return JSON.stringify({ foo, bar });
}

```

---

### Подводные камни

* Баги
* `any`
* Мало готовых деклараций
* Нет аннотаций для immutability, как следствие
* Массивы и объекты инвариантны

---

### Side note

Если для API сложно написать декларакции, то API — плохое.


```js

type Handler = (req: Req, res: Res, next?: (s?: string) => void);

type App = Handler & {
  get: (key: string) => mixed
     & (...handlers: Array<Handler>) => App
     & (handlers: Array<Handler>) => App
     &  (handlers: Array<Handler>, ...moreHandlers: Array<Handler>) => App
     & (path: string | RegExp | Array<string | RegExp>, ...handlers: Array<Handler>) => App
     & (path: string | RegExp | Array<string | RegExp>, handlers: Array<Handler>) => App
     & (path: string | RegExp | Array<string | RegExp>, handlers: Array<Handler>, ...moreHandlers: Array<Handler>) => App
}
```

---

### Type driven development

То же что, Test driver development, но типы вместо тестов.

1. Определить типы
2. Написать оболочку реализации
3. Сделать так, чтобы Flow не жаловался
4. Внести правки (и в типы, и в реализацию)
5. GOTO 3

Но тесты никто не отменял.

---

### Почему бы не использовать другой (полноценный) язык

Рассмотрели: Go, Swift, Rust, Scala

* Null-safety: нет в Go и Scala
* Generics: нет в Go
* async-await: нет нигде (но в Go и не надо)
* GC: нет в Swift и Rust
* Ad-hoc polymorphic structs: нет нигде
* Классовое ООП: хрен с ним
* Огромная экосистема: нет в Swift и Rust (пока)

---

### Отдельно про Go

Good parts:

* возврат ошибки: `Result`
* каналы: промисы; может быть, свои каналы
* горутины: `async-await`
* структурная типизация: `type Foo = { foo: string }`
* простая компиляция, бинарник на выходе
* производительность
* предсказуемый выход:

```js
async main() {
  const r = await runApp(app);
  process.exit(r);
}
```




---

### Отдельно про Go

Bad parts:

* многопоточность
* нет Null-safety (даже `this` нужно проверять)
* нет дженериков
* GOPATH
* нет type inference
