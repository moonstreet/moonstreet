---
title: "Starting With React"
date: 2020-02-02T11:30:36+01:00
image: ""
author: "Jacqueline"
tags: ["reactjs"]
categories: ["reactjs"]
---

# Getting started with React. Need to know modern JavaScript

Disclaimer: I just want to be able to build simple web apps. I do not want to be a professional web dev by no means. Just want to be able to create a little bit of frontend to some api's. So let's get going!

## Declaring variables

Using const:  

```js
alert(name1)
const name = 'Jacqueline'
// name = "Marie" will render an error: Assignment to a constant variable
alert(name)
```

Using var:

```js
if (true) {
  var name2="Marlies"
  alert(name2)
}

//this will work even it is out scope, because name2 is part of the WINDOW>OBJECT
//This is why Javascript is so ugly
console.log(name2)
```

Using let:

```js
if (true) {
  let name3="Marlies"
  alert(name3)
}

console.log(name3) //index.js:26 Uncaught ReferenceError: name3 is not defined
```

## Templating

The "old way':

```js
let fname = 'Jacqueline'
let mname = 'Patricia'
let age = prompt('Guess my age!')
let result = 'Name ' + fname + 'Middlename ' + mname + "Age" + age 
alert(result)
```
A new way, very nice!

```js
let newresult = `Hi my name is ${fname} ${mname} and I am  ${age} years old`
alert(newresult)
```

## Default parameters

```js
function welcome(user='Jacqueline', message='Ja hoor') {
alert(`Hello ${user} ${message}`)
}

welcome('jacqueline') //will also display: ja hoor!
```


## Arrow Functions

Just a function how we used to write them:

```js
function greeting(message) {
    return console.log(${message})
}
```

But now: look ma, no arguments!

```
let hello = () => alert('Hello everybody!')
console.log(hello())
```

Arrow function with one argument:

```js
let message = mytext => console.log(`The message is.. ${mytext}`)
console.log(message('here is a log line'))
```

Two arguments and error check: 

```js
let logMessage = (title, body) => {
  if (!title) {
    throw new Error('A title is required');
  }

  if (!body) {
    throw new Error('A body is required');
  }

  return `log line: ${title} with body ${body}`
}

console.log(logMessage('something broke', 'unknown error'))
```

## This keyword

The JavaScript this keyword refers to the object it belongs to. It is import for object method binding.
https://www.w3schools.com/js/js_this.asp

For example a todo list.

```
let todo = { 
  tasks:['learn React','learn Javascript', 'learn CRUD'],
  printTasks: function() {
    console.log(this.tasks.join(' - '))
  }
}

alert(todo.printTasks())
```

Here another example:

```
let taskList = { 

  tasks: ['React','MongoDb','CRUD'],

  printTasks: function() {
    setTimeout(() => console.log(this.tasks.join(' - ')), 3000);
  }

};

alert(taskList.printTasks());
```

## Object construction

```
let Todo = { 
  name: "Writing",
  category: "Book",
  duration: 1
}

console.log(Todo.name);

//instantiate the object
let {category, name} = Todo 
category = 'Study';
console.log(Todo.category);
```

## Class constructor super

```
class Holiday {

  constructor(destination, days) {
    this.destination = destination;
    this.days = days;
  }

  info() {
    console.log(`We have been ${this.days} days in ${this.destination}`)
  }

}

console.log(Holiday.prototype)
const trip = new Holiday('Bonaire', 14) 
console.log(trip.info());
```

## Inheritance

```
class Holiday { 

  constructor(destination, days) { 
    this.destination = destination;
    this.days = days;
  }

  info() { 
    console.log(`We have been ${this.days} days in ${this.destination}`)
  }

}

console.log(Holiday.prototype)
const trip = new Holiday('Bonaire', 14)
console.log(trip.info());


class ShortHoliday extends Holiday { 
  constructor(destination, days, city) { 
    super(destination,days);
    this.city = city;
  }

  info() { 
    super.info();
    console.log(`And visit ${this.city} when there`);
  }

}

const cityTrip = new ShortHoliday('France',3,'Paris')
cityTrip.info()
```

I gues this is enough javascript for now.
