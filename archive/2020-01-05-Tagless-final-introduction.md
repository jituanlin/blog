# Tagless final introduction

## What is Tagless? Why use it?
In functional programming world, `tagless final` is a design pattern used to achieve 2 goals:
1. Left the **effect** free.
2. Left the programming **implemation** of **semantic** free, 
so we program in **semantic** level.

## Too abstract? Let's look some example

### A example about print something

Suppose we need to:
 1. print a array from left to right,then
 2. print it from right to left again
Easy! right? Let's do it:
```Typescript
const main = (array: string[]) => {
  array.forEach(s => console.log(s));
  array.reverse().forEach(s => console.log(s));
};
main(["one", "two", "three"]);
```
Nice! We run it, check the screen output, it work!
But, should we run and check the screen output every
time for test it? 

It will be a big problem if we need test our code hand by 
hand every time when we facing a big codebase.

### `Dependence inject`?
Dependence inject is a cool thing in OOP world, 
we do not resist hugging it.
```typescript
interface Console {
  log: (x: any) => void;
}

const getMain = (console: Console) => (array: string[]) => {
  array.forEach(s => console.log(s));
  array.reverse().forEach(s => console.log(s));
};

const test = () => {
  const output = [];
  const console = {
    log: x => output.push(x)
  };
  getMain(console)(["one", "two", "three"]);
  assert.deepEqual(output, ["one", "two", "three", "three", "two", "one", ,]);
};
```



