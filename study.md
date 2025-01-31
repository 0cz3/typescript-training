# TypeScript

静的型付け言語。型システムを用いてプログラムの正しさを実行前にチェックできる

参考にしたページ

- [サバイバルTypeScript](https://typescriptbook.jp/)
- [TypeScript Deep Dive](https://typescript-jp.gitbook.io/deep-dive)

## 型推論

型注釈がついていない変数でもコンテキストに基づいて自動的に型を推測する。
型推論できるものについては基本型注釈を記述しないで良い。

## 型注釈(type annotation)

コンパイラに与えるヒント。書き加えると、コンパイラはより細かいチェックをしてくれる
変数や定数、関数のパラメータ、関数の戻り値などのデータを定義する際に書く`データ: 型`

```ts
function increment(num: number) {
//                 ^^^^^^^^型注釈
  return num + 1;
}
console.log(increment("999"));
```

memo
上記だと型強制が行われずエラーとなる
型強制 (type coercion)
型が異なる2つの値を処理するとき、暗黙的に別の型へ変換されること
`999 + 1 = 1000`
`"999" + 1 = 9991 //型強制`

### オブジェクトの型注釈

プロパティに対して型をつける。キーと値の型のペアを書く。ネストしてもかける

```ts
let box: { width: number; height: number };
//       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^型注釈
box = { width: 1080, height: 720 };
```

### 配列の型注釈

```ts
//Type[]
let array: number[];
array = [1, 2, 3];
//Array<T>
let array: Array<number>;
array = [1, 2, 3];
```

## コンパイル

TypeScriptのコードはコンパイル時にjavascriptに変換される
NodeはTypeScriptを解釈しない

## 型エイリアス

typeを使って型に名前をつけて使い回す

```ts
type StringOrNumber = string | number;
let value: StringOrNumber;
value = "hello"; // string型が代入可能
value = 123; // number型も代入可能
```

## 基本的な型

JSと同じ型.number,string,boolean...
memo
numberは浮動小数点。少数やマイナス可


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
any型の値をより安全にする、型アサーションの制約を回避する際に利用される

型アサーションの制約を回避する際に利用

```ts
const str = "a";
const num = str as unknown as number;
```

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

### void

値が存在しないことを示す。関数が何も返さない場合に使用

### never

決して何も返さないことを示す。エラーを投げる関数や無限ループの関数の戻り値として使用する。


```ts
function throwError(): never {
  throw new Error();
}
// 関数宣言の場合の型推論はvoidになる。関数宣言やアロー関数ではneverの推論になる
```

memo
[網羅性チェックにも使用](https://typescriptbook.jp/reference/statements/never#never%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E7%B6%B2%E7%BE%85%E6%80%A7%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF)
条件分岐で全ての型についての処理が網羅されているかチェックする

```ts
type Extension = "js" | "ts" | "json";
function printLang(ext: Extension): void {
  switch (ext) {
    case "js":
      console.log("JavaScript");
      break;
    case "ts":
      console.log("TypeScript");
      break;
    default:
      const exhaustivenessCheck: never = ext;
      // jsonが入ってくるが、neverにはneverしか入らないためエラーとなる
      break;
  }
}
```

下記のようしておくとより使用しやすい

```ts
class ExhaustiveError extends Error {
  constructor(value: never, message = `Unsupported type: ${value}`) {
    super(message);
  }
}

function printLang(ext: Extension): void {
  switch (ext) {
    case "js":
      console.log("JavaScript");
      break;
    case "ts":
      console.log("TypeScript");
      break;
    default:
      throw new ExhaustiveError(ext);
  }
}
```

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
    //処理
  }
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

## 読み取り専用配列

値の変更をさせない

```ts
const numbers: readonly number[] = [1, 2, 3];
const strings: ReadonlyArray<string> = ["hello", "world"];
```

## 列挙型 (Enum)

定数のセットを作って使い回す。型を作ると同時にオブジェクトも作られる

```ts
// パスカルケース
enum Position {
  Top,
  Right,
  Bottom,
  Left,
}

//同義
enum Position {
  Top = "Top",
  Right = "Right",
  Bottom = "Bottom,
  Left = "Left",
}
```



## タプル型

配列の要素数と要素の型を固定。[型, 型, 型]で表す

```ts
let tuple: [string, number];
tuple = ["hello", 10]; // 代入できる
tuple = [10, "hello"]; // 順序が正しくないため、代入できない
tuple = ["hello", 10, "world"]; // 要素が多すぎるため代入できない (pushはできる、その場合参照時にエラーが出る)
```

## readonlyプロパティ

プロパティに代入させない

```ts
let obj: { readonly name: string; age: number };
obj = { name: "John", age: 20 };
obj.name = "Tom"; //エラー
```

## オプションプロパティ

?をつけたプロパティは省略可能

```ts
let obj: { name: string; age?: number };
obj = { name: "John" };
```

memo オブジェクトメソッド
関数をプロパティに持つオブジェクトを定義できる

```ts
const obj = {
  a: 1,
  b: 2,
  sum(): number {
    return this.a + this.b;
  },
};
```

## インデックス型

オブジェクトのフィールド名をあえて指定せず、プロパティのみを指定したい場合使用

```ts
let obj: {
  [K: string]: number;
};
// let obj2: Record<string, number>;でも同義

obj = { a: 1, b: 2 }; // OK
obj.c = 4; // OK
obj["d"] = 5; // OK
```

## ユニオン型

```ts
let numberOrUndefined: number | undefined;
// 配列
type List = (string | number)[];
// 絞り込み
if (typeof maybeUserId === "string") {
  const userId: string = maybeUserId; // この分岐内では文字列型に絞り込まれるため、代入できる。
}
```

## インターセクション型

オブジェクトの定義を合成させる。考え方はユニオン型と相対するもの

```ts
type TwoDimensionalPoint = {
  x: number;
  y: number;
};
type Z = {
  z: number;
};
type ThreeDimensionalPoint = TwoDimensionalPoint & Z;
const p: ThreeDimensionalPoint = {
  x: 0,
  y: 1,
  z: 2,
};
```

## リテラル型 (literal type)

プリミティブ型の特定の値だけを代入可能にする。マジックナンバーやステートの表現に用いる

```ts
const isTrue: true = true;
const num: 123 = 123;
const str: "foo" = "foo"; //fooしか入らない

const num: 1 | 2 | 3 = 1; // 1, 2, 3しか受け入れない
```

## 関数への型づけ

パラメータと戻り値につける。パラメータはanyになるため必ず型をつける。戻り値は型推論してくれることが多い

```ts
//アロー関数
const greet = (name: string): string => {
  return `Hello ${name}`;
};
const greet: (name: string) => string = name => {
  return `Hello ${name}`;
};

//関数宣言
function greet(name: string): string {
  return `Hello ${name}`;
}

//呼び出し
console.log(greet("John"));
'Hello John'

//callback (arg: [引数の型]) => [戻り値の型]
function greetNewUser(func: (name: string) => string) {
  console.log(func("ご新規"));
}
function hello(name: string) {
  return `こんにちは！${name}さん！！`;
}
function goodMorning(name: string) {
  return `おはようございます！${name}さん！！`;
}
/* funcの型をvoidにするとhelloやgoodMorningではエラーは起きないが、
greetNewUserの処理で使われているfuncでエラーが起きる */

//呼び出し
// こんにちは！ご新規さん!!
greetNewUser(hello);
// おはようございます！ご新規さん!!
greetNewUser(goodMorning);
```

## クラスへの型づけ

```ts
class Person {
  name: string;
  age: number;
  // private age: number;などアクセス修飾子と併用可
  // private readonly age: number;のようにreadonly修飾子と併用可
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  introduce(): void {
    console.log(`My name is ${this.name} and I am ${this.age} years old.`);
  }
}
```

## this引数 (this parameter)

使用するコンテキストによってthisの意味するところが変わってしまうため、これらがどのコンテキストで使用されるべきなのかをTypeScriptに伝える
thisは呼び出す側は意識する必要はなく、第2引数以降を指定する
アロー関数の場合は考慮不要

```ts
class Male {
  private name: string;
  public constructor(name: string) {
    this.name = name;
  }
  public toString(): string {
    return `Monsieur ${this.name}`;
  }
}

class Female {
  private name: string;
  public constructor(name: string) {
    this.name = name;
  }
  public toString(this: Female): string {
    return `Madame ${this.name}`;
  }
}
// Femaleを呼び出す際のthisは必ずFemale型であると伝える
```

## type importとtype export

型のみエクスポートして使用できる

```ts
// type.ts----------
export type MyObject = {
  name: string;
  age: number;
};
// main.ts----------
import type { MyObject } from "./types";
const obj: MyObject = {
  name: "TypeScript",
  age: 3,
};
```

## typeof型演算子

変数名から型を逆算できる。

```ts
const object = {
  name: "TypeScript",
  version: 3.9,
};
type ObjectType = typeof object;
// カーソルを当てると、type ObjectType = { name: string; version: number;}と表示される
```

## keyof型演算子

オブジェクトの型からプロパティ名を型として返す

```ts
type Person = {
  name: string;
};
type PersonKey = keyof Person;
// カーソルを当てると、type PersonKey = "name"と表示される
```

2つ以上のプロパティがあるオブジェクトの型にkeyofを使った場合は、すべてのプロパティ名がユニオン型で返される

```ts
type Book = {
  title: string;
  price: number;
  rating: number;
};
type BookKey = keyof Book;
// 上は次と同じ意味になる
type BookKey = "title" | "price" | "rating";
```

## Required<T>

Tのすべてのプロパティからオプショナルであることを意味する?を取り除く

## Partial<T>

Tのすべてのプロパティをオプションプロパティにする

呼び出し時に必要な引数以外の記述を省略することもできる

```ts
function findUsers({
  surname,
  middleName,
  givenName,
  age,
  address,
  nationality,
}: FindUsersArgs = {}) {//省略された場合はデフォルトの引数{}が呼び出しされる
  // ...
}
findUsers();
findUsers({ age: 22 });
```

## Record<Keys, Type>

プロパティのキーがKeysであり、プロパティの値がTypeであるオブジェクトの型を作る

```ts
type StringNumber = Record<string, number>;
const value: StringNumber = { a: 1, b: 2, c: 3 };
```

## Pick<T, Keys>

オブジェクトから特定のプロパティだけを拾い出す
元の方が変更になった場合も追従してくれる

```ts
type User = {
  surname: string;
  middleName?: string;
  givenName: string;
  age: number;
  address?: string;
  nationality: string;
  createdAt: string;
  updatedAt: string;
};
type Person = Pick<User, "surname" | "middleName" | "givenName">;
```

## Omit<T, Keys>

TからKeysで指定したプロパティを除いたobject型を返す。
KeysにTには無いプロパティキーを指定しても、TypeScriptコンパイラーは指摘しないので注意。
（タイポに気をつける、Tのプロパティを修正した場合は、こちらの第二引数ももれなく修正する）

## Exclude<T, U>

ユニオン型TからUで指定した型を取り除いたユニオン型を返す

```ts
type Grade = "A" | "B" | "C" | "D" | "E";
type PassGrade = Exclude<Grade, "D" | "E">;
// → type PassGrade = "A" | "B" | "C"
```

## Extract<T, U>

ユニオン型TからUで指定した型だけを抽出した型を返す

```ts
type Grade = "A" | "B" | "C" | "D" | "E";
type FailGrade = Extract<Grade, "D" | "E">;
// → type FailGrade = "D" | "E"
```

## NonNullable<T>

Tからnullとundefinedを取り除いたユニオン型を返す

## ReturnType<T>

関数型Tの戻り値を取得する
typeofと併用して実際の関数の戻り値を取得できる

```ts
const isEven = (num: number) => {
  return num / 2 === 0;
};
type isEvenRetType = ReturnType<typeof isEven>;
//type isEvenRetType = boolean
```

## Awaited<T>

Promiseが何重にネストしていても、最終的な解決値の型を取得できる

## Mapped Types

既存の型から新しい型を生成

```ts
type Person = {
  name: string;
  age: number;
};
type ReadOnlyPerson = { readonly [K in keyof Person]: Person[K] };
/* type ReadOnlyPerson = {
    readonly name: string;
    readonly age: number;
} */
```

## インデックスアクセス型 (indexed access types)

プロパティ名に対応する型を取得。key ofを使用すると前プロパティの方が得られる

```ts
type Foo = { a: number; b: string; c: boolean };
type T = Foo[keyof Foo];
// type T = string | number | boolean
```

[number]を使用して配列型にも使用可能

```ts
type MixedArray = (string | undefined)[];
type T = MixedArray[number];
// type T = string | undefined

const array = [null, "a", "b"];
type T = (typeof array)[number];
// type T = string | null  typeof型演算子と組み合わせて、配列の値から要素の型を導く
```

## satisfies演算子「satisfies operator」

satisfies T(Tは型)で、その値が型Tを満たすことを検証。データの型を判定したいだけの時に使用

```ts
const pikachu: Pokemon = {
  name: "pikachu",
  no: 25,
  genre: "mouse pokémon",
  height: "0.4m",
  weight: "6.0kg",
};
const raichu = {
  name: "raichu",
  no: 26,
  genre: "mouse pokémon",
  height: "0.8m",
  weight: "30.0kg",
} satisfies Pokemon;
```

## インターフェースと型エイリアスの違い

 |   |   |   |
|---|---|---|
|内容|インターフェース|型エイリアス|
|継承|可能|不可。ただし交差型で表現は可能|
|継承による上書き|上書きまたはエラー|フィールド毎に交差型が計算される|
|同名のものを宣言|定義がマージされる|エラー|
|Mapped Types|使用不可|使用可能|

インターフェースと型エイリアスの使い分けについて明確な違いはないが、
Googleが公開しているTypeScriptのスタイルガイドの型エイリアスvsインターフェースの項目では、プリミティブな値やユニオン型やタプルの型定義をする場合は型エイリアスを利用し、オブジェクトの型を定義する場合はインターフェースを使うことを推奨している。

## Promise
romiseの型を指定する場合はジェネリクスを伴いPromise<T>と記述。
TにはPromiseが履行された(fulfilled)ときに返す値の型を指定
