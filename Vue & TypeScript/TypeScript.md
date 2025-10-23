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

