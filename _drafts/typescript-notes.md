value: todo lo que le podemos asignar a una variable

type = properties + methods . Ejemplo: string, que tiene propiedades y sus métodos.

types are "labels" to describe different properties and methods that a value has.

type => easy way to refer to the different properties and functions that a value has

## Types

* Primitive Types: number, boolean, string, symbol, void, null, undefined

* Object Types: functions, arrays, classes, object

## Why types matter

* Types are used by the compiler to catch errors during development

* Helps to understand what values are when reading the code

## Type annotations and Type inference

* Type annotations: code that we add to tell Typescript what type of a value a variable will refer to

* Type inference: Typescript tries to figure out what type of value a variable refers to

### Annotations with variables

Type annotation example:

``` javascript
let apples: number = 5
```

### any

* is a type as string or boolean
* means TS has no idea what this is - can't chck for correct property references
* avoid 'any' variables except when you have no choice

## Divers

### Type Definition Files

In typescript we can use javascript libraries (all the stuff that is on npm) but requires those libraries to have all its methods types defined and it is not always the case. For this cases exists `Type Definition Files` which are files than do a mapping between the original JS method API into another with have the type of all its methods defined. So, it is like an adapter.

The `Type Definition Files` are created by the community. Most popular libraries have one, but less known need to be created by some volunteer.

You can look for `Type Definition Files` on npm by placing: @type/name-of-the-library in the searching bar. Example: @type/faker.

### Export

For exporting a member of a file (method, attribute, class), we must to use the `export` keyword:

``` typescript
// User.ts
export class User {
  name: string
  location: {
    lat: number
    lng: number
  }
```

Doing so, we can then import this class definition on another file:

``` typescript
// index.ts
import { User } from './User'
```

### Union types

When using `Union Types` as method arguments, `Typescript` do a pairing of the different properties of the types involved in the union, in order to see which properties can be refered on the variable of the union type. Example:

``` typescript
export class User {
  name: string
  location: {
    lat: number
    lng: number
  }
}

export class Company {
  companyName: string
  catchPhrase: string
  location: {
    lat: number
    lng: number
  }
}

addMarker(mappable: User | Company): void {
	new google.maps.Marker({
	  map: this.googleMap,
	  position: {
	    lat: mappable.location.lat,
	    lng: mappable.location.lng
	  }
	})
}
```

In this case, `Typescript` check which properties to limit which number of properties could be refered on `mappable` inside the `addMarker` method. In this case, `mappable` can only refer to `location` property.

And now, we have only two types involved in the union. What if in the future we add new types to this union? It will be unmantainable. Maybe, we need a more reusable approach.

Defining an interface:

``` typescript
// Instructions to every other class
// on how they can be an argument to 'addMarker'
interface Mappable {
  location: {
    lat: number
    lng: number
  }
}
```

Now, refactoring the method:

``` typescript
addMarker(mappable: Mappable): void {
	new google.maps.Marker({
	  map: this.googleMap,
	  position: {
	    lat: mappable.location.lat,
	    lng: mappable.location.lng
	  }
	})
}
```

## Configuring Typescript Compiler

* To create a `tsconfig.json`, in your project:

``` commandline
tsc --init
```

The settings inside this file allow us to customize the typescript compiler.

Some interesting properties to sey up here:

``` typescript
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    // "incremental": true,                   /* Enable incremental compilation */
    "target": "es5",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "outDir": "./build",                        /* Redirect output structure to the directory. */
    "rootDir": "./src",

} 
```

Here we are indicating with `outDir` where the `output Dir` of the transpiled code will be, and where the source code will be with `rootDir`.

Now, doing `tsc` we can see how our output files are placed into the `build` folder:

``` typescript
tsc
```

* live code reloading

Just running `tsc` with the `w` (watch mode) option.

``` typescript
tsc -w
```

## Enums

Indicates that it is a collection of closed related values. It does not bring any perfomance improvement. The benefits of enums lays on the semantics it brings.

``` typescript
enum MatchResult {
  HomeWin = 'H',
  AwayWin = 'A',
  Draw = 'D'
}


for (let match of matches) {
  if (match[1] === 'Man United' && match[5] === MatchResult.HomeWin) {
    manUnitedWins++
  } else if (match[2] === 'Man United' && match[5] === MatchResult.AwayWin) {
    manUnitedWins++
  }
}

```

Some things to mention:

* the enum syntax follows almost the same rules as regular js objects
* creates an object with the same keys and values when transpiled to js code.
* use it when you have a small fixed set of values that are very closely related and known at compile time.

## Type Assertions

Is a way of overriding `Typescript` default behavior when asserting types. For example:

``` typescript
enum MatchResult {
  HomeWin = 'H',
  AwayWin = 'A',
  Draw = 'D'
}


this.data = fs
  .readFileSync(this.filename, {
    encoding: 'utf-8'
  })
  .split('\n')
  .map((row: string): string[] => row.split(','))
  .map((row: string[]): any => {
    return [
      dateStringToDate(row[0]),
      row[1],
      row[2],
      parseInt(row[3]),
      parseInt(row[4]),
      row[5] as MatchResult   // we are sure this value can be 'H', 'A' or 'D' so we tell TS to consider it as a MatchResult
    ]
  })

```

## Generics

``` typescript
class HoldAnything<TypeOfData> {
  data: TypeOfData
}

let holdNumber = new HoldAnything<number>()
holdNumber.data = 123

let holdString = new HoldAnything<string>()
holdString.data = 'hola'
```