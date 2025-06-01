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
{}: It represents any value that is not __null__ or __undefined__. 

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

## Value is optional

In the following sample, the value is optional, but key is not:

```typescript
type A = {
  keyIsRequired: string | undefined
}

// error!, key keyIsRequired is not optional, only its value is:
const a: A = {

};

// ok:
const a: A = {
  keyIsRequired: undefined
};
```

### Utility types
This types are available globally in any typescript project.
#### Pick and Omit ####
Work on the structure of objects
```typescript
type Sample = {
  a: string,
  b: number,
  c: Date
};

type A = Omit<Sample, "a" | "b">; // Will result in {c: Date}
const x: A = {c: new Date()};

type B = Pick<Sample, "a">; // Will result in {a: string}
const y: B = {a: "hello"};
```

#### Extract and Exclude
Work on unions

```typescript
type Sample1 = {
  discriminator: "A",
  b: number,
  c: Date
};

type Sample2 = {
  discriminator: "B",
  d: number,
};

type SampleUnion = Sample1 | Sample2;

type A = Extract<SampleUnion, {discriminator: "A"}>; // Will result type Sample1
const x: A = {
  discriminator:  "A",
  b: 344,
  c: new Date()
}
type B = Exclude<SampleUnion, {discriminator: "A"}>; // Will result type Sample2
const y: B = {
  discriminator:  "B",
  d: 23
}
```

## Advanced stuff

### Conditional types

Very simple sample
```typescript
interface Animal {
  makeNoise(): void;
}
interface Cat extends Animal {
  miau(): void;
}
 
type Example1 = Cat extends Animal ? number : string; //Will be number
const p: Example1 = 33;
```

Here a sample that uses conditional types to flatten an array:

```typescript
type Flatten<T> = T extends any[] ? T[number] : T;
 
// Extracts out the element type, the result is just the string type
type Str = Flatten<string[]>;
     
const x: Str = "abc";
 
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

### Symbol
In typescript, you can create a new symbol like this:

```typescript
const mySymbol: unique symbol = Symbol();
```

Symbols can be used as keys to objects:

```typescript
let myObj = {[mySymbol]: "hello world"};
console.log(myObj[mySymbol]);
```

The following statement just declares a symbol, but does not really create a symbol. It compiles to nothing (when compiled to javascript):
```typescript
declare const metersSymbol: unique symbol;
```

#### Branded types
Symbols can be used for branded types. In the following sample, no symbol is really created in the resulting code. It just checkts type on compile time, not runtime.
In this sample, it prevents the user to confuse ther parameters ind method addMeters. The first
argument is of type *meters* and the second argument must be of type *kilometers*.

```typescript
declare const metersSymbol: unique symbol;
declare const kilometersSymbol: unique symbol;

type Meters = number & { [metersSymbol]: void };
type Kilometers = number & { [kilometersSymbol]: void };

function meters(value: number): Meters {
  return value as Meters;
}

function kilometers(value: number): Kilometers {
  return value as Kilometers;
}

function addMeters(value1: Meters, value2: Kilometers): Meters
{
  return (value1 + value2*1000) as Meters;
}

console.log(addMeters(meters(23), kilometers(1)));
```
