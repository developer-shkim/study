> 댄 밴더캄, 『이펙티브 타입스크립트』, 프로그래밍 인사이트(2021) 를 읽고 간단히 정리했습니다. [예제 코드 저장소 링크](https://github.com/danvk/effective-typescript)입니다.

<!-- omit in toc -->

# 목차

- [1장\. 타입스크립트 알아보기](#1장-타입스크립트-알아보기)
  - [개념](#개념)
  - [타입스크립트 설정](#타입스크립트-설정)
  - [타입스크립트 컴파일러 역할](#타입스크립트-컴파일러-역할)
  - [런타임에는 타입 체크가 불가능하다](#런타임에는-타입-체크가-불가능하다)
  - [타입시스템을 열려(open) 있다\.](#타입시스템을-열려open-있다)
  - [any 타입 지양하기](#any-타입-지양하기)
- [2장\. 타입스크립트의 타입 시스템](#2장-타입스크립트의-타입-시스템)
  - [타입은 할당 가능한 값들의 집합이다](#타입은-할당-가능한-값들의-집합이다)
  - [타입 관점 vs 값 관점](#타입-관점-vs-값-관점)
  - [타입 선언 vs 타입 단언](#타입-선언-vs-타입-단언)
  - [함수 표현식(expression) vs 함수 선언식(statement)](#함수-표현식expression-vs-함수-선언식statement)
  - [타입 vs 인터페이스](#타입-vs-인터페이스)
  - [타입 반복 줄이기](#타입-반복-줄이기)
  - [동적 데이터에 인덱스 시그니처 사용](#동적-데이터에-인덱스-시그니처-사용)
  - [매핑된 타입을 사용하여 값을 동기화하기](#매핑된-타입을-사용하여-값을-동기화하기)
- [3장\. 타입 추론](#3장-타입-추론)
  - [타입을 명시해야 하는 이유](#타입을-명시해야-하는-이유)
  - [타입 넓히기(widening)](#타입-넓히기widening)
  - [타입 좁히기(narrowing)](#타입-좁히기narrowing)
- [4장\. 타입 설계](#4장-타입-설계)
  - [유효한 상태만 표현하는 타입을 지향하기](#유효한-상태만-표현하는-타입을-지향하기)
  - [인터페이스의 유니온 지향하기](#인터페이스의-유니온-지향하기)
  - [string 타입 남용하지 않기](#string-타입-남용하지-않기)

# 1장. 타입스크립트 알아보기

### 개념

- 모든 자바스크립트는 타입스크립트이지만, 일부 자바스크립트만이 타입 체크를 통과한다.
- 타입 체크를 통과하더라도 여전히 런타임에 오류가 발생할 수 있다.
- 변수 정의에 오타가 있고, 사용처에서 올바르게 사용했을 때 타입스크립트는 사용처가 오타라고 판단할 수 있다.
- 명시적으로 정의에 타입을 선언하면 변수에 오타가 있음을 알 수 있으므로 의도를 분명하게 하는 것이 좋다.

### 타입스크립트 설정

- 타입스크립트를 어떻게 사용할지 동료, 도구가 알게 하기 위해서 가급적 tsconfig.json 을 사용하는 것이 좋다.
- `noImplicitAny`(암시적 any)
  - 암시적 any 타입을 허용하지 않음
  - 타입 정의 없이 코드를 사용했을 때, 암시적으로 any 라는 타입으로 간주됨
  - 위 옵션이 설정되어있을 경우 오류로 처리하여 타입을 정의하게 함
  ```ts
  function add(a, b) { ... } // 암시적으로 any
  function add(a: number, b: number) { ... }
  ```
- `strictNullChecks`
  - null, undefined 를 모든 타입에서 허용하지 않도록 설정함
  - 타입에 null 을 추가하거나 if 문으로 체크 또는 단언문(`!.`) 설정
  ```ts
  const x: number = null; // 오류
  const x: number | null = null; // 통과
  ```

### 타입스크립트 컴파일러 역할

- 타입스크립트를 자바스크립트로 트랜스파일
- 코드 타입오류 체크
  > 트랜스파일: 소스코드를 다른 형태의 소스코드로 변환하지만 여전히 컴파일되어야하므로 컴파일과 구분함
- 유의사항
  - 코드에 오류가 있을 때 컴파일 오류가 아니라 타입 체크 오류가 있다는 것이 기술적으로 맞다. 코드 생성만이 컴파일이기 때문이다.
  - 컴파일과 타입 체크는 독립적이어서 타입 오류가 있는 코드도 컴파일은 가능하다.

### 런타임에는 타입 체크가 불가능하다

- 자바스크립트 컴파일 과정에서 모든 인터페이스, 타입, 타입 구문은 제거된다.
- 런타임에 타입 체크하는 방법
  - (1) 속성을 체크
  - (2) 타입을 명시적으로 저장하는 태그 기법
  - (3) 인터페이스 대신 클래스로 사용 => 타입, 값 모두 사용 가능

```ts
interface Shape {
    width: number;
}

interface Rectangle extends Square{
    height: number;
}

function calc(shape: Shape) {
    if (shape instanceof Rectangle) { ... }
    // (1) 방법
    // instanceof 는 런타임에 체크됨
    // Rectancle 은 타입이므로 런타임 시점에 아무 역할도 할 수 없음

    if ('height' in shape) { ... }
    // 런타임에 타입 정보를 유지할 수 있음
}
```

```ts
interface Square {
    kind: 'square'
    width: number;
}

interface Rectangle {
    kind: 'rectangle';
    height: number;
    width: number;
}

type Shape = Square | Rectangle;    // tagged union

function calc(shape: Shape) {
    if (shape.kind === 'rectangle') { ... }
    // (2) 방법
}
```

```ts
class Square { ... }

class Rectangle extends Square { ... }

type Shape = Square | Rectangle;    // 타입으로 참조

function calc(shape: Shape) {
    if (shape instanceof Rectangle) { ... } // 값으로 참조
    // (3) 방법
}
```

### 타입시스템을 열려(open) 있다.

- 함수 호출에 사용되는 매개변수의 속성들이 타입에 선언된 속성만을 가지지 않는다.
- `for (const el of Object.keys(obj))` 에서 el 은 암시적으로 any 타입이 된다.
- 타입에 선언되지 않은 속성일 수 있음을 감안하기 때문이다.
- open <-> sealed, precise

### any 타입 지양하기

근거

- `as any` 를 사용하면 선언된 타입에 관계없이 다른 타입으로 할당할 수 있게 되어 타입 안전성이 없어진다.
- 함수 시그니처를 무시하여 약속된 타입이 아닌 다른 타입으로 호출할 수 있다.
- 코드 에디터에서 자동완성, 도움말 기능을 지원받지 못한다.

# 2장. 타입스크립트의 타입 시스템

### 타입은 할당 가능한 값들의 집합이다

- 모든 숫자값의 집합은 `number` 타입이고, 42는 타입에 해당하고 'Korea' 해당하지 않는다.
- 가장 작은 집합은 공집합으로 `never` 타입니다.
- 그 다음으로 작은 집합은 한 가지 값만 포함하는 `unit`(`literal`) 타입이다.
- `할당 가능한` = `~의 원소` = `~의 부분 집합` 으로 해석할 수 있다.
- 타입 체커의 주요 역할은 하나의 타입이 다른 타입의 부분 집합인지 검사하는 것이다.
- union(합집합): `|`, intersection(교집합): `&`

```ts
interface Person {
  name: string;
  birth: Date;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;

const ps: PersonSpan = {
  name: 'tsts',
  birth: new Date('2024/06/29'),
  death: new Date('2024/06/30'),
};

type A = keyof (Person | Lifespan);
type B = keyof (Person & Lifespan);

const a: A = 'birth'; // type A = "birth"
const b: B = 'death'; // type B = "death" | "name" | "birth"
```

### 타입 관점 vs 값 관점

class, enum, typeof 연산자는 상황에 따라 타입과 값 모두 가능한 예약어이다.

- class
  - 타입 관점에서는 형태가 사용되고
  - 값 관점에서는 생성자가 사용된다.
- typeof 연산자
  - 타입 관점에서는 타입스크립트 타입을 반환하고
  - 값 관점에서는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환한다.
    - 자바스크립트는 6개의 런타임 타입이 존재한다. (string, number, boolean, undefined, object, function)

```ts
class Cylinder {}

type T = typeof Cylinder; // type T = typeof Cylinder
type TT = InstanceType<typeof Cylinder>; // type TT = Cylinder
```

### 타입 선언 vs 타입 단언

- 변수에 값을 할당하고 타입을 부여하는 방식은 선언, 단언이 있다.

```ts
interface Person {
  name: string;
}

const alice: Person = { name: 'Alice' }; // 선언
const bob = { name: 'Bob' } as Person; // 단언 (assertions)
```

- 타입 선언을 사용하는 게 낫다.

  - 단언은 인터페이스를 만족하지 않아도 통과한다.

  ```ts
  const emptyAlice: Person = {}; // 에러
  const emptyBob = {} as Person; // 통과
  ```

  - 단언은 잉여 속성 체크가 되지 않는다.

  ```ts
  const alice: Person = {
    name: 'Alice',
    wrongProperty: 'wrong value',
  }; // Person 에 wrongProperty 없다는 에러 발생

  const bob = {
    name: 'Bob',
    wrongProperty: 'wrong value',
  } as Person; // 통과
  ```

- 타입 단언은 타입 체커보다 개발자가 판단한 타입이 더 정확할 때만 사용하자.(예: DOM 엘리먼트)
- 타입 단언은 A 가 B 의 서브 타입일 때에만 B as A 로 쓸 수 있다.
- 잉여 속성 체크(Excess Property Checks)

  - 타입 명시된 변수에 리터럴 할당할 때 해당 타입의 속성이 있는지, 그 외 속성은 없는지 확인
  - 리터럴이 아닌 변수를 할당할 때에는 체크되지 않음(예: 임시 변수 추가)

  ```ts
  interface Person {
    name: string;
    age?: number;
  }

  const p1: Person = { name: 'shkim', agge: 10 }; // Person 에 agge 가 없다는 에러 발생

  const p2 = { name: 'shkim', agge: 10 };
  const p3: Person = p2; // 통과
  ```

- 공통 속성 체크
  - 선택적 속성만 가지는 weak 타입에 동작
  - 임시 변수를 추가하더라도 동작

### 함수 표현식(expression) vs 함수 선언식(statement)

- 타입스크립트에서는 함수 표현식을 사용하는 것이 좋다.
- 매개변수, 반환값까지 전체를 타입으로 선언하여 재사용할 수 있기 때문이다.
- 함수에 타입을 정의하면 함수 구현에서는 정의하지 않아도 자동으로 추론

```ts
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;

declare function fetch( ... ) // 라이브러리의 함수 정의

const myFetch: typeof fetch = ...  // 라이브러리 함수 타입으로 선언
```

### 타입 vs 인터페이스

- 공통점

  ```ts
  // 인덱스 시그니처
  type TDict = { [key: string]: string };
  interface IDict {
    [key: string]: string;
  }

  // 제네릭
  type TPair<T> = {
    first: T;
    second: T;
  }

  interface IPair<T> {
    first: T;
    second: T
  }

  // 확장
  interface IStateWithPop extends TState {
    population: number;
  }
  type TStateWithPop = IState & { populate: number };

  // 클래스에서 구현
  class StateT implements TState ...
  class StateT implements IState ...
  ```

- 다른 점

  - 타입은 유니온, 매핑된 타입, 조건부 타입 같은 고급 기능 사용 가능

  ```ts
  interface VariableMap {
    [name: string]: Input | Output;
  }

  // 인터페이스는 아래 처럼 못함ㅠ
  type NamedVariable = (Input | Output) & { name: string };
  type Pair = [number, number];
  type StringList = string[];
  type NamedNums = [string, ...number[]];

  // 비슷하게 구현은 할 수 있음
  interface Tuple {
    0: number;
    1: number;
    length: 2;
  }
  ```

  - 인터페이스는 보강(augument) 가능
    - 속성을 확장하는 선언 병합(declaration merging) 가능
    - API 가 변경될 때 사용자가 인터페이스를 통해 필드 병합 가능
    - 예) lib.es5.d.ts, lib.es2015.d.ts 에 선언된 Array 인터페이스 병합

  ```ts
  interface IState {
    name: string;
  }

  interface IState {
    population: number;
  }

  const test: IState = {
    name: 'aaa',
    population: 4,
  };
  ```

### 타입 반복 줄이기

- 매핑된 타입, Pick 과 같은 제네릭 타입 사용

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

// TopNavState 를 State 의 부분집합으로 본다면.
interface TopNavState = {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}

// 매핑된 타입을 사용하여 반복을 줄일 수 있음
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
};
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

- 인덱싱

```ts
interface SaveAction {
  type: 'save';
}

interface LoadAction {
  type: 'load';
}

type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load';

// Action 유니온을 인덱싱
type ActionType = Action['type']; // 'save' | 'load'
```

- keyof

```ts
interface Options {
  width: number;
  height: number;
}

interface OptionsUpdate {
  width?: number;
  height?: number;
}

class UIWidget {
  constructor(init: Options) {}
  update(options: OptionsUpdate) {}
}

// 매핑된 타입, keyof 로 구현 (Partial 로도 사용 가능)
type OptionsUpdate = { [k in keyof Options]?: Options[k] };
```

- typeof: 자바스크립트 런타임 연산자와 다른 타입스크립트 연산자

```ts
const OPTIONS = {
  width: 640;
  height: 480;
}

// 값의 형태에 해당하는 타입 정의
type Options = typeof OPTIONS;
```

### 동적 데이터에 인덱스 시그니처 사용

- 런타임 때까지 객체의 속성을 알 수 없을 경우에만 인덱스 시그니처 사용
- 안전한 접근을 고려한다면 인덱스 시그니처 값 타입에 undefined 추가

```ts
type Vec3D = Record<'x' | 'y' | 'z', number>; // Record 제네릭 타입
type Vec3D = { [k in 'x' | 'y' | 'z']: number }; // 매핑된 타입
// type Vec3D = { x: number, y: number, z: number };

type ABC = { [k in 'a' | 'b' | 'c']: k extends 'b' ? string : number }; // 조건부 타입
```

### 매핑된 타입을 사용하여 값을 동기화하기

- REQUIRES_UPDATE 는 항상 ScatterProps 와 동일한 속성을 가져야 한다는 정보를 제공

```ts
const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = {
  xs: true,
  ys: true,
  onClick: false
}

let k: keyOf ScatterProps;
for (k in oldProps) {
  if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
    return true;
  }
}
return false;
```

# 3장. 타입 추론

### 타입을 명시해야 하는 이유

- 객체 리터럴 정의 시 잉여 속성 체크가 동작 -> 사용되는 곳이 아닌 선언 시에 오류 표시
- 함수 선언 시 의도된 반환 타입 명시 -> 사용되는 곳이 아닌 선언 시에 오류 표시
- 시그니처를 먼저 작성하면 TDD 효과처럼 원하는 모양을 얻게 됨
- 명명된 타입 사용

```ts
interface Vector2D {
  x: number;
  y: number;
}

function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y }; // Vector2D 가 아닌 { x: number, y: number } 로 추론
}
```

> eslint 의 no-inferrable-types 를 사용하면 모든 타입 구문이 정말 필요한지 확인 가능

### 타입 넓히기(widening)

- 런타임에 모든 변수는 유일한 값을 가진다.
- 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에는 가능한 값들의 집합인 타입을 가진다.
- 타입 체커가 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다. = 타입 넓히기
- `as const` 를 사용해서 최대한 좁은 타입으로 추론할 수 있다.

```ts
const v1 = {
  x: 1,
  y: 2,
}; //  { x: number; y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
}; //  { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const; // { readonly x: 1; readonly y: 2 }
```

### 타입 좁히기(narrowing)

- if 문으로 null 체크
- instanceOf, 속성명 in 변수, Array.isArray 등으로 좁히기
- 태그된 유니온 방식
- 커스텀 함수로 사용자 정의 타입 가드 활용

```ts
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}
```

# 4장. 타입 설계

### 유효한 상태만 표현하는 타입을 지향하기

```ts
// 오류이면서 동시에 로딩 중일 수 있음
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

// 애플리케이션 상태를 제대로 표현하기
interface RequestPending {
  state: 'pending';
}

interface RequestError {
  state: 'error';
  error: string;
}

interface RequestSuccess {
  state: 'ok';
  pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}
```

### 인터페이스의 유니온 지향하기

```ts
// 나쁜 예: 유니온의 인터페이스
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}

// 좋은 예: 인터페이스의 유니온
interface FillLayer {
  type: 'fill'; // 태그로 타입의 범위를 좁힐 수 있음
  layout: FillLayout;
  paint: FillPaint;
}

interface LineLayer {
  type: 'line';
  layout: LineLayout;
  paint: LinePaint;
}

interface PointLayer {
  type: 'point';
  layout: PointLayout;
  paint: PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer;
```

### string 타입 남용하지 않기

```ts
// 나쁜 예: 매개변수와 반환타입에 any 가 있음
function pluck(records: any[], key: string): any[] {
  return records.map((r) => r[key]);
}

// 보완 1단계: 제네릭 타입 사용
function pluck<T>(records: T[], key: string): any[] {
  return records.map((r) => r[key]); // error: key 범위가 너무 넒음
}

function pluck<T>(records: T[], key: keyof T): any[] {
  return records.map((r) => r[key]); // error 해결
}

// 좋은 예: 타입 좁히기
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][];

pluck(albums, 'releaseDate'); // Date[]
pluck(albums, 'artist'); // string[]
```
