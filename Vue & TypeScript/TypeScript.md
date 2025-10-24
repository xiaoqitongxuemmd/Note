# TypeScript

## 数据类型

- number
- boolean
- string
- array
  - `const name: string[] = ['', '']` 推荐方式
  - `const name: Array<string> = ['', '']`
- object 不推荐使用
  - `const info = { name: 'why', age: 18 }`
- null/undefined
- symbol
- any
- unknown
  - unknown 只能赋值给 unknown 和 any 类型的变量
- void
- never
- tuple
  - `const info: [string, number, number] = ['a', 1, 2]`
  - 数组只存放相同类型的数据，而元组可以存放不同的类型
- 可以通过 | 声明联合类型
- 可以通过 interface / type 声明类型
- 可以使用 as 作为类型断言
- 使用 ! 做非空类型断言 `message!.length`
- 可选链 ?.
  - 当对象不存在时会发生短路，直接返回 undefined
  - 当对象属性存在时，才会继续执行
- !! 操作符可以将其他类型变量转换为 boolean 类型
- ?? 操作符可以空值合并
  - `const content = message ?? 'default'` 等价于 `const content = message ? message 'default'`

## 类型缩小

- typeof
- === 等
- instanceof 
- in

## 类

```typescript
class Person {
    public name: string;
    private age: number;

    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    public greet() {
        console.log(`Hello, my name is ${this.name} and I am ${this.age} years old.`);
    }

    private getAge() {
        return this.age;
    }
}

const person = new Person('Alice', 25);
console.log(person.name); // 输出: Alice
// console.log(person.age); // Error: Property 'age' is private.
person.greet(); // 输出: Hello, my name is Alice and I am 25 years old.
// person.getAge(); // Error: Method 'getAge' is private.
```

typescript 中类使用 extends 关键字继承
```typescript
class Person {
    constructor(public name: string, public age: number) {}

    public greet() {
        console.log(`Hello, my name is ${this.name} and I am ${this.age} years old.`);
    }
}

class Employee extends Person {
    constructor(name: string, age: number, public position: string) {
        super(name, age); // 调用基类的构造函数
    }

    public work() {
        console.log(`${this.name} is working as a ${this.position}.`);
    }
}

const employee = new Employee('Alice', 25, 'Developer');
employee.greet(); // 输出: Hello, my name is Alice and I am 25 years old.
employee.work(); // 输出: Alice is working as a Developer.
```

抽象类
```typescript
abstract class Animal {
    constructor(public name: string) {}

    public greet() {
        console.log(`Hello, I am ${this.name}.`);
    }

    abstract makeSound(): void;
}

class Dog extends Animal {
    constructor(name: string) {
        super(name);
    }

    public makeSound() {
        console.log('Woof woof!');
    }
}

class Cat extends Animal {
    constructor(name: string) {
        super(name);
    }

    public makeSound() {
        console.log('Meow meow!');
    }
}

const dog = new Dog('Buddy');
dog.greet(); // 输出: Hello, I am Buddy.
dog.makeSound(); // 输出: Woof woof!

const cat = new Cat('Whiskers');
cat.greet(); // 输出: Hello, I am Whiskers.
cat.makeSound(); // 输出: Meow meow!
```

## 泛型

```typescript
function foo<Type>(arg: Type): Type {
    return arg
}
```