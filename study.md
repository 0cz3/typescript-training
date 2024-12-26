# TypeScript

静的型付け言語。型システムを用いてプログラムの正しさを実行前にチェックできる

参考にしたページ

- [サバイバルTypeScript](https://typescriptbook.jp/)
- [TypeScript Deep Dive](https://typescript-jp.gitbook.io/deep-dive)


## 型推論

型注釈がついていない変数でもコンテキストに基づいて自動的に型を推測する

## 型注釈(type annotation)

コンパイラに与えるヒント。書き加えると、コンパイラはより細かいチェックをしてくれる

```ts
function increment(num: number) {
//                 ^^^^^^^^型注釈
  return num + 1;
}
console.log(increment("999"));
```

上記だと型強制が行われずエラーとなる

型強制 (type coercion)
型が異なる2つの値を処理するとき、暗黙的に別の型へ変換されること
`999 + 1 = 1000`
`"999" + 1 = 9991 //型強制`

## コンパイル

TypeScriptのコードはコンパイル時にjavascriptに変換される
NodeはTypeScriptを解釈しない

## 特殊な型

### any
なんでも代入できる。値に対する操作の制限がないため安全性は高くない
どのような型の変数にも代入できる

```ts
const value: any = 10;
const int: number = value;
const bool: boolean = value;
const str: string = value;
const obj: object = value;
```

操作もできる

```ts
const a: any = 100; // 代入できる
console.log(a * 3); // 操作もできる

```

型付けに時間をかけたくない、とりあえず動かしたいなどコンパイラーのチェックを抑制したいときに使用できる
memo：
暗黙のany
型注釈もなく型推論もできない場合、TypeScriptが変数の型をanyにすること
noImplicitAnyをtrueにすると暗黙のanyに警告を出すことができる

### unknown

何でも代入できる型。その値に対する操作は制限され、型の安全性が保たれる。「型安全なany型」

具体的な型へは代入できない

```ts
const x: unknown = 100; // 代入はできる
console.log(x * 3); // 操作はできない
//'x' is of type 'unknown'.
```

操作できない

```ts
const value: unknown = 10;
const int: number = value;
//Type 'unknown' is not assignable to type 'number'.
```

プロパティへのアクセス、メソッドの呼び出しもできない

```ts
const value: unknown = 10;
value.toFixed();
// 'value' is of type 'unknown'.
const obj: unknown = { name: "オブジェクト" };
obj.name;
//'obj' is of type 'unknown'.
```

型の絞り込み
unknownを使うためには、typeofやinstanceofなど型ガードを行って型を絞り込む必要がある。型ガード以降はガードした型として扱う
typeofで型ガード

```ts
const value: unknown = "";
// 型ガード
if (typeof value === "string") {
  // ここブロックではvalueはstring型として扱える
  console.log(value.toUpperCase());
}
```

型ガード関数で型ガード

```ts
// 型ガード関数
function isObject(value: unknown): value is object {
  return typeof value === "object" && value !== null;
}
const value: unknown = { a: 1, b: 2 };
// 型ガード（型ガード関数isObject呼び出し）
if (isObject(value)) {
  console.log(Object.keys(value));
  // ここでは、valueはobject型として扱える
}
```

型ガード関数
TypeScriptのコンパイラはifやswitchといった制御フローの各場所での変数の型を分析している。それを利用して型ガードする

```ts
function isDuck(animal: Animal): animal is Duck {
  return animal instanceof Duck;
}
```

animal is Duckの部分は型述語。関数isDuck()がtrueを返す時のifのブロックの中ではanimalはDuck型として解釈される

void
値が存在しないことを示す。関数が何も返さない場合に使用



## 高度な型表現

ユニオン型
複数の方のどれか。初期値nullの処理などに使用

```ts
type NullableString = string | null;
```

ダブル型
配列の各要素に異なる型を指定できる型

```ts
type Response = [number, string];
const response: Response = [200, "OK"];
```

## 構造的部分型システム

オブジェクトの形状（つまり、オブジェクトがどのようなプロパティとメソッドを有しているか）に基づいて型を判断

構造的型付け(structural typing)
型が持つプロパティやメソッドの構造が同一であれば、異なる名前を持つ型同士でも互換性があると見なす

```ts
class Person {
  walk() {}
}
class Dog {
  walk() {}
}
const person = new Person();
const dog: Dog = person; // コンパイルエラーにならない
```

構造的部分型
型が持つプロパティやメソッドの構造に着目して、基本型（親クラス）と部分型（子クラス）の関係性を判断

```ts
class Shape {
  area(): number {
    return 0;
  }
}
 
class Circle {
  radius: number;
  constructor(radius: number) {
    this.radius = radius;
  }
  area(): number {
    return Math.PI * this.radius ** 2;
  }
}

const shape: Shape = new Circle(10);
```

CircleクラスはShapeクラスのareaメソッドを持っており、追加でradiusプロパティを定義しているため、CircleはShapeの部分型として扱われる。これは、CircleがShapeの持つ構造（ここではareaメソッド）を含んでいるため。その結果、Shape型の変数にCircle型のインスタンスを代入することが可能

TypeScriptにおいてextendsは部分型の判定ではなく、親クラスの機能やインターフェイスを継承するために使用する

```ts
class Animal {
  walk() {}
}
 
class Dog extends Animal {
  walk(speed: number) {} // コンパイルエラーになる
}
```

引数を取らないというインターフェイスが守られていない、エラーになる
privateメンバーや、型アサーションを使って名前的型付けのように扱うことも可能

構造的部分型システムによりテスト対象の依存物の注入が簡単にできる

```ts
type User = { id: number; name: string };

class UserApi {
  async getUser(id: number): Promise<User | undefined> {
    
}

class UserService {
  private api: UserApi;

  constructor(api: UserApi) {
    this.api = api;
  }
  async userExists(id: number): Promise<boolean> {
    const user = await this.api.getUser(id);
    return user !== undefined;
  }
}
```

memo
名前的型付け(nominal typing)は、型の名前に基づいて型の区別を行う方法

オブジェクトの形状（オブジェクトがどのようなプロパティとメソッドを有しているか）に基づいて型を判断。Javaなど。

```java
class Person {}
class Dog {}
class Main {
    public static void main(String[] args) {
        Person person = new Person();
        Dog dog = person; // コンパイルエラー: 不適合な型
    }
}
```

名前的部分型
クラスやインターフェースの継承を通じて、型間の親子関係（基本型と部分型の関係）が形成される

```java
class Shape {}
class Circle extends Shape {}
Shape shape = new Circle();
```

## ジェネリクス

コードを共通化した際に型を変数のように扱う

before

```ts
// 重複した3つの関数
function chooseRandomlyString(v1: string, v2: string): string {
  return Math.random() <= 0.5 ? v1 : v2;
}
function chooseRandomlyNumber(v1: number, v2: number): number {
  return Math.random() <= 0.5 ? v1 : v2;
}
function chooseRandomlyURL(v1: URL, v2: URL): URL {
  return Math.random() <= 0.5 ? v1 : v2;
}
```

after

```ts
function chooseRandomly<T>(v1: T, v2: T): T {
  return Math.random() <= 0.5 ? v1 : v2;
}
chooseRandomly<string>("勝ち", "負け");
chooseRandomly<number>(1, 2);
chooseRandomly<URL>(urlA, urlB);
```






