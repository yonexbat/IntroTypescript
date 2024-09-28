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

## Unions

## Special types
### any

```typescript
const json = JSON.parse("55");  // json has type any
```

### undefined

### null

### never

### intrinsic
The implementation of this type is provided by the compiler. For example:
```typescript
/**
 * Convert first character of string literal type to uppercase
 */
type Capitalize<S extends string> = intrinsic;
```

## Basics
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

## Advanced stuff
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

