// NOTHING TO DO WITH GENERICS
const addOne = (a: number): number => {
  return a + 1
}

const addTwo = (a: number): number => {
  return a + 2
}

// a more generic approach

const add = (a: number, b: number): number => {
  return a + b
}

// another example

class HoldNumber {
  data: number
}

let holdNumber = new HoldNumber()
holdNumber.data = 123

class HoldString {
  data: string
}

let holdString = new HoldString()
holdString.data = 'asdasd'

// using GENERICS
class HoldAnything<TypeOfData> {
  data: TypeOfData
}

let holdNumber = new HoldAnything<number>()
holdNumber.data = 123

let holdString = new HoldAnything<string>()
holdString.data = 'hola'