# An introduction to typescript
I assume you  already know *javascript* and already worked a bit with *typescript*. Now you would like to get a better understanding of typescript.

## Hello typescript

Explicit type:

```typescript
let m1: string = 'hello';
```

Implicit type:

```typescript
let m1 = 'hello';
```

The type can be infered by the declaration, so the type is implicitly defined.

## Unions and intersection types
This sample here clarifies the difference between union (|) and intersection (&)

```typescript
type MyUnion1 = {a: string} | {b: string};
const x1_all: MyUnion1 = {a: "hello", b: "world"}
const x1_onlyOne: MyUnion1 = {a: "hello"} 

type MyUnion2 = {a: string} & {b: string};
const x2_all: MyUnion2 = {a: "hello", b: "world"}
const x2_onlyOne: MyUnion2 = {a: "hello"} // Error in typescript here
```

## Special types

### any

```typescript
const json = JSON.parse("55");  // json has type any
```

### undefined
*undefined* means that variable has not been defined. The undefined type is a primitive type that has only one value: *undefined*

### null
The *null* value represents the intentional absence of any object value. Indication that a variable points to no object.

### never
Type *never* has never a value.
```typescript
 const x: never = undefined; // Will not compile. There is no way to assign a value to a never variable.
```

### intrinsic
The implementation of this type is provided by the compiler. For example:
```typescript
/**
 * Convert first character of string literal type to uppercase
 */
type Capitalize<S extends string> = intrinsic;
```
### Empty object type
It represents any value that is not __null__ or __undefined__. 

Samples:
```typescript
type Sample = {};
const x1: Sample = "abc";
const x2: Sample = 4;
```

Invalid sample:
```typescript
type Sample = {};
const x3: Sample = null; //not valid
```

Sometimes, for autocomplete, libraries use this trick:
```typescript
type Sample = "a" | "b" | string & {};
const x: Sample = "anystring really"
```
VS Code will help with autocomplete in editor. It proposes a or b or anystring. Bit of a hack though.

## Basics

### like a constant
Pin down to a single value. 

```typescript
type X = 13;
const p: X = 13;
const z: X = 12; // error
```

String is more common:

```typescript
type X = 'a' | 'b';
const p: X = 'a';
const z: X = 'some random string'; // error
```

### keyof

```typescript
type SampleType = {
  A: string,
  B: string
};

type AllKey = keyof SampleType; // 'A' | 'B'

const x: AllKey = 'A'; 
```

### typeof
This exists in javascript as well, but in typescript it is more useful:

```typescript
let a = { x: 10, y: 3 };

type TypeOfA = typeof a; // {x: number; y: number;}

let b: TypeOfA =  { x: 1, y: 1 };
```


```typescript
function f() {
  return { x: 10, y: 3 };
}

type FuncDecl = typeof f; // () => {x: number; y: number;}

type P = ReturnType<typeof f>; // {x: number; y: number;}

let x: P = {
  x: 2, y:4
}
```

### Tuples
```typescript
let ourTuple: [number, boolean, string] = [123,true,'hello'];
```
Tuples with a rest:

```typescript
let ourTuple: [number, boolean, string, ...any[]] = [123,true,'hello', 2131231, 2344];
```

In the example above, the rest is declared as ...any[]. This is equivalent to  ...any. If you inspect the type of ourTuple in the sample below, it will be *let ourTuple: [number, boolean, string, ...any[]]*

```typescript
let ourTuple: [number, boolean, string, ...any] = [123,true,'hello', 2131231, 2344]; // same as let ourTuple: [number, boolean, string, ...any[]]
```

### Array\<any\> vs any[]
See https://stackoverflow.com/questions/15860715/typescript-array-vs-any

*Array\<any>* is equivalent to *any[]*

## Advanced stuff

### Conditional types

Here a sample that uses conditional types and never to flatten an array:

```typescript
type Flatten<T> = T extends any[] ? T[number] : T;
 
// Extracts out the element type.
type Str = Flatten<string[]>;
     
type Str = string
 
// Leaves the type alone.
type Num = Flatten<number>;
```
#### the infer keyword
The infer keyword can be used in conditional types. Let typescript do its best to infer the type. 
Here a very basic example to get the type of the first element in a tuple.

```typescript
type MyTuple = [boolean, number];
type First<T> = T extends [infer FirstElem, any] ? FirstElem : never; // boolean
const x: First<MyTuple> = true;
```


In this example, the type of the return value of a function is evaluated:
```typescript
**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;

```

Here another basic example with string type:

```typescript
type ExtractStringType<T> = T extends `${infer U}` ? U : never;
const x: ExtractStringType<'hello'> = 'hello';
```

There can be nested conditional types:
```typescript
type ExtractNumberType<T> = T extends `${infer U}` ? `${U}` extends `${number}` ? U : never : never;
const p: ExtractNumberType<'123'> = '123';
const t: ExtractNumberType<'abc'> = 'abc'; // error!! not possible, only numbers allowed
```

### Mapped types

In the following example, *OptionsFlags\<Type>* replaces all properties with boolean properties:

```typescript
type SampleType = {
  A: string,
  B: string
};

type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type OptionsOfSampleType = OptionsFlags<SampleType>; // {A: boolean, B: boolean}

const x: OptionsOfSampleType = {
  A: true,
  B: false
};
```

It is also possible to make properties optional:

```typescript
type SampleType = {
  A: string,
  B: string
};

type MyPartial<Type> = {
  [T in keyof Type]+?: Type[T];
};

type PartialOfSampleType = MyPartial<SampleType>; // {A?: boolean | undefined; B?: boolean | undefined}

const x: PartialOfSampleType = {
  A: 'A'
};
```

And you can rename properties:
```typescript
type SampleType = {
  A: string,
  B: string
};

type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

type GetterSample = Getters<SampleType>;

let x: GetterSample = {
  getA: function (): string {
    return 'PorpAValue';
  },
  getB: function (): string {
    return 'PorpBValue';
  }
};
```

