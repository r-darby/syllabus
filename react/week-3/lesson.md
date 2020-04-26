# React 3

**What will we learn today?**

- [Recap](#recap)
- [Unmounting](#unmounting)
- [The Circle of Life](#the-circle-of-life)
- [Fetching Data in React](#fetching-data-in-react)
- [Working with forms in React](#working-with-forms-in-react)

## Recap

<!-- TODO: check this after reviewing updates to week 2 -->

Last week we looked at using props and state to create React components that change with user input ([interactive example](https://codesandbox.io/s/7j21mrq08x)):

```js
function Counter() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1)
  }

  return (
    <div>
      Count: {count}
      <button onClick={increment}>Click me!</button>
    </div>
  );
}
```

We also looked at fetching data in our React components: <!-- TODO: add interactive example -->

```js
function DataFetcher() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('URL')
      .then(res => res.json())
      .then(data => {
        setData(data)
        setError(null)
      })
      .catch(err => {
        setError(err)
        setData(null)
      })
  }, []);

  if (!data) {
    return 'Loading...'
  } else {
    return (
      <div>
        Your data is {data.thing}
      </div>
    )
  }
}
```

## Unmounting

So far we've looked at components that are always rendered in the browser. However (and this is often the case in large applications), we might want to control whether components are shown or not. Let's look at a Toggle component ([interactive example](https://codesandbox.io/s/xmo8oo514)):

```js
const Message = () => (
  <p>I'm shown when this.state.isShown is true ✅</p>
);

class Toggle extends Component {
  constructor(props) {
    super(props);
    this.state = { isShown: false };
  }

  toggle = () => {
    this.setState((previousState) => { 
    	return { isShown: !previousState.isShown } 
	 });
  };

  render() {
    return (
      <div>
        {this.state.isShown ? <Message /> : null}
        <button onClick={this.toggle}>Toggle</button>
      </div>
    );
  }
}
```

If you open up dev tools, you will see that the element changes based on the `isShown` state. The hidden element is not hidden with CSS, it is actually removed from the DOM. This is because `this.state.isShown` is `false` which means the Toggle component returns `null` for that part of the JSX. If you return `null` in JSX then React will render nothing at all.

## The Circle of Life

When a component is within the DOM, we call it *mounted*. When a component is removed from the DOM, we call it *unmounted*. When we change state like in the unmounting example above, we can switch between these statuses. This gives us a clue that components go through a *lifecycle* of different statuses. We have seen 2 of the statuses: mounting and unmounting, there is also a third called *updating*.

We can hook into this lifecycle through special component methods that are added by React's `Component` class. They are run at different points of the lifecycle, often before and after they change to a different status. The method names contain `will` or `did` based on whether they run before or after a status change.

This diagram shows the React component lifecycle:

![React component lifecycle](../assets/lifecycle.png)

Let's look at how we can use one of the lifecycle methods ([interactive example](https://codesandbox.io/s/m5z2v36x1y)):

```js
class Lifecycle extends Component {
  componentDidMount() {
    console.log('componentDidMount');
  }

  render() {
    return <div>Hello World</div>;
  }
}
```

> **Exercise A**
> Open the `pokedex` application that we have been working on for the last 2 weeks and open the `CaughtPokemon.js` file
> 1. Add a `constructor` method to the `CaughtPokemon` component. Within this method add a `console.log('constructor')`
> 2. Add a `componentDidMount` method to the `CaughtPokemon` component. Within this method add a `console.log('componentDidMount')`. You don't need to return anything from this method
> 3. Repeat the same step above with the `componentDidUpdate` and `componentWillUnmount` methods
> 4. Try interacting with the `CaughtPokemon` component in your web browser (clicking the button) while looking at the JavaScript console. What order do the logs appear?
> 5. The `componentWillUnmount` method will never be called. Can you explain why?

We'll now focus on a few of the lifecycle hooks and see how they are used.

### `componentDidMount` and `componentWillUnmount`

The `componentDidMount` method runs after a component has finished rendering to the DOM. The component is now waiting for a props change or input from the user. It is called only once. We use this lifecycle hook to make changes outside of the component (sometimes these are called *side effects*).

The `componentWillUnmount` method runs when a component has been unmounted from the DOM. It is used to "clean up" the component as it is no longer being shown. Often we need to close down or cancel the changes we made in `componentDidMount`.

To look at these in more detail, we'll create a Clock component in an exercise.

> **Exercise B**
> Open the `pokedex` React application again
> 1. Create a new file called `Clock.js` in the `src` directory
> 2. Copy and paste in the code below ([interactive version](https://codesandbox.io/s/p9q2wq069j)):

```js
import React, { Component } from 'react';

class Time extends Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }
  
  tick = () => {
    console.log('tick');
    this.setState({
      date: new Date()
    });
  };
  
  render() {
    return (
      <div>{this.state.date.toLocaleTimeString()}</div>
    );
  }
}

class Clock extends Component {
  constructor(props) {
    super(props);
    this.state = { isShowingClock: true };
  }
  
  toggle = () => {
    this.setState((previousState) => {
      return { isShowingClock: !previousState.isShowingClock };
    });
  };
  
  render() {
    return (
      <div>
        {this.state.isShowingClock && <Time />}
        <button onClick={this.toggle}>Toggle time</button>
      </div>
    );
  }
}

export default Clock;
```

> 3. In `App.js` import the `Clock` component with `import Clock from './Clock'`
> 4. Then render the `Clock` component in the `App` component (hint: `<Clock />`)
> 5. Now change the `Time` component (notice that there are 2 components defined in this file) add a `componentDidMount` method
> 6. Within the `componentDidMount` method use `setInterval` to call `this.tick` every 1000 milliseconds (hint: `setInterval(this.tick, 1000)`)
> 7. Now open the JavaScript console your web browser. What is happening? Can you explain why?
> 8. Keep looking at the JavaScript console and try clicking the "Toggle time" button. What do you think the problem is here? How can we fix it?
> 9. Change the `componentDidMount` method to assign `this.timer` to the output of `setInterval` (hint: `this.timer = setInterval(this.tick, 1000)`)
> 10. Add a `componentWillUnmount` method to the `Time` component
> 11. In the `componentWillUnmount` method, remove the timer by calling `clearInterval(this.timer)`
> 12. Try clicking the "Toggle time" button again, like in step 9. How have we solved the problem?

## Fetching Data in React

Most web applications will load data from the server. How do we do this in React? The component lifecycle is very important - we don't want to be calling our API at the wrong time, or multiple times with the same data!

If we tried to fetch data in our `render` method, it would make a request every time props or state changed. This would create lots of unnecessary requests. As we saw above, `componentDidMount` is called only once when the component is first rendered and so it is an ideal place for making requests. Let's look at an example ([interactive example](https://codesandbox.io/s/4rkovwq0kw)):

```js
class MartianPhotoFetcher extends Component {
  componentDidMount() {
    fetch(`https://api.nasa.gov/mars-photos/api/v1/rovers/curiosity/photos?earth_date=${this.props.date}`);
  }

  render() {
    // We don't don't what the img src is when we render :(
    return <img src={src} />;
  }
}
```

This example isn't very useful! We can't use the data returned from the server in `render` because the request is asynchronous :( We need React to re-render once the request is resolved - a perfect use for state! Let's look at an example ([interactive example](https://codesandbox.io/s/5kk53yx6ll))

```js
class MartianPhotoFetcher extends Component {
  constructor(props) {
    super(props);
    this.state = {
      imgSrc: null
    };
  }
  
  componentDidMount() {
    fetch(`https://api.nasa.gov/mars-photos/api/v1/rovers/curiosity/photos?earth_date=${this.props.date}`)
      .then(res => res.json())
      .then(data => {
        this.setState({
          imgSrc: data.photos[0].img_src
        })
      });
  }
  
  render() {
    return <img src={this.state.imgSrc} />;
  }
}
```

Now we can see the Martian photo that we fetched from the server!

However we have a bit of a problem - when we first render the component, we don't have the photo `src` yet. We first have to initialise it to `null` in the constructor. This shows us that we're missing something from our UI - a *loading status*.

Let's look at showing a different UI when the request is loading ([interactive example](https://codesandbox.io/s/93zr0xz32r)):

```js
class MartianPhotoFetcher extends Component {
  constructor(props) {
    super(props);
    this.state = {
      isLoading: true,
      imgSrc: null
    };
  }
  
  componentDidMount() {
    fetch(`https://api.nasa.gov/mars-photos/api/v1/rovers/curiosity/photos?earth_date=${this.props.date}&api_key=gnesiqnKCJMm8UTYZYi86ZA5RAnrO4TAR9gDstVb`)
      .then(res => res.json())
      .then(data => {
        this.setState({
          isLoading: false,
          imgSrc: data.photos[0].img_src
        })
      });
  }
  
  render() {
    if (this.state.isLoading) {
      return <span>Loading... 👽</span>;
    } else {
      return <img src={this.state.imgSrc} />;
    }
  }
}
```

Here are the steps that the component takes:

- Initialise `isLoading` to `true`
- In `render`, show a loading message because `isLoading` is true
- Once rendered, `componentDidMount` will trigger the API request
- When the request resolves, we set the `isLoading` state to false and set the data that we want
- Changing state triggers a re-render, and because `isLoading` is false we render the Martian photo

We can still improve our component! What happens if we make a request that fails? Our request will error, but we won't show the error in the browser. Let's see how we can fix it ([interactive example](https://codesandbox.io/s/6v9qo90r2r)).

First we have to deal with annoying quirk of `fetch` - it doesn't reject the promise on HTTP errors. We can fix this by adding another `.then` before we convert to JSON:

```js
.then((res) => {
  if (res.status >= 200 && res.status < 300) {
    return res;
  } else {
    throw new Error('HTTP error');
  }
})
```

Now we can add our solution - a `.catch` on the `fetch` call. Here we reset the loading state and add the error to state.

```js
.catch((err) => {
  this.setState({
    isLoading: false,
    err: err
  });
})
```

Now we can check if there's an error in state and render out an error message:

```js
render() {
  if (this.state.isLoading) {
    return <span>Loading... 👽</span>;
  } else if (this.state.error) {
    return <span>Something went wrong 😭</span>;
  } else {
    return <img src={this.state.imgSrc} />;
  }
}
```

> **Exercise C**
> Open the `pokedex` React application again and open the `src/BestPokemon.js` file
> 1. If you haven't already, convert the `BestPokemon` component to a class component
> 2. Create a `constructor` method (hint: remember to call `super(props)`)
> 3. Set the initial state to have a key named `pokemonNames` that is assigned to an empty array `[]`
> 4. Add a `componentDidMount` method to the component
> 5. Within the `componentDidMount` method call the `fetch()` function with this URL: `https://pokeapi.co/api/v2/pokedex/1/`. What will this do?
> 6. Add a `.then()` handler into the `fetch` function (hint: remember this needs to come immediately after the `fetch()` call) which converts the response from JSON (hint: `.then(res => res.json())`)
> 8. Add a second `.then()` handler after the one we just added, where the callback function will receive an argument called `data`
> 9. Within the second `.then()` callback function, log out the data that we just received (hint: `console.log(data.pokemon_entries[0].pokemon_species.name)`)
> 10. Now change the `console.log()` to log out an array instead, with the first, fourth and seventh Pokemon (hint: `console.log([data.pokemon_entries[0].pokemon_species.name, data.pokemon_entries[3].pokemon_species.name, data.pokemon_entries[6].pokemon_species.name])`)
> 11. Now again within the `.then()` callback function, call `this.setState()` to set the `pokemonNames` key and assign it to the array that we just logged out (you can copy/paste it)
> 12. Inside the `render` method, remove the old `pokemonNames` variable and replace it with `this.state.pokemonNames`. What do you see in your web browser?
> 13. Add an `isLoading` piece of state, which is initialised to `true`
> 14. When calling `this.setState()` inside the `.then()` handler, also set `isLoading` to `false`
> 15. In the `render` method check if `this.state.isLoading` is `true` and return a loading message (e.g. `<span>Loading...</span>`). Otherwise if `this.state.isLoading` is `false` then render the loop as we did before
> 16. **(STRETCH GOAL)** Add some error handling which renders an error message
> 17. **(STRETCH GOAL)** Explore the data returned from the API. See if you can show some more interesting Pokemon information in your app (hint: try `console.log`ging different data returned from the API)

## Working with forms in React

Modern web applications often involve interacting with forms such as creating an account, adding a blog post or posting a comment. This would involve using inputs, buttons and various form elements and being able to get the values entered by users to do something with it (like display them on a page or send them in a POST request). So, how do we do this in React?

A popular pattern for building forms and collect user data is the *controlled component* pattern. A pattern is a repeated solution to a problem that is useful in multiple similar cases. Let's have a look at an example ([interactive example](https://codesandbox.io/s/4jq1yqy8kx)):

```js
class SimpleReminder extends Component {
  constructor(props) {
    super(props);
    this.state = {
      reminder: ""
    };
  }

  handleChange = event => {
    this.setState({
      reminder: event.target.value
    });
  };

  render() {
    return (
      <div>
        <input
          type="text"
          placeholder="Some reminder"
          value={this.state.reminder}
          onChange={this.handleChange}
        />
        <p>Today I need to remember to... {this.state.reminder}</p>
      </div>
    );
  }
}
```

We're controlling the `value` of the input by using the value from the `reminder` state. This means that we can only change the value by updating the state. It is done using the `onChange` attribute and the method `handleChange` which is called every time the input value changes (typically when a new character is added or removed). If you didn't call `this.setState()` in the `handleChange` method, then the input's value would never change and it would appear as if you couldn't type in the input! Finally, the value we keep in the `reminder` state is displayed on the screen as today's reminder. 

In addition, instead of just saving the value of the input in the state, we could have also transformed the string before we set it with `this.setState()`, for example by calling `toUpperCase()` on the string.

Let's have a look at a more complex example where we want to build a form to let users enter information to create a personal account ([interactive example](https://codesandbox.io/s/m7p083zn6p)):

```js
class CreateAccountForm extends Component {
  constructor(props) {
    super(props);
    this.state = {
      username: "",
      email: "",
      password: ""
    };
  }

  handleChange = event => {
    this.setState({
      [event.target.name]: event.target.value
    });
  };

  submit = () => {
    console.log("Do something with the form values...");
    console.log(`Username = ${this.state.username}`);
    console.log(`Email = ${this.state.email}`);
    console.log(`Password = ${this.state.password}`);
  };

  render() {
    return (
      <div>
        <div>
          <input
            type="text"
            name="username"
            placeholder="Username"
            value={this.state.username}
            onChange={this.handleChange}
          />
        </div>
        <div>
          <input
            type="text"
            name="email"
            placeholder="Email"
            value={this.state.email}
            onChange={this.handleChange}
          />
        </div>
        <div>
          <input
            type="password"
            name="password"
            placeholder="Password"
            value={this.state.password}
            onChange={this.handleChange}
          />
        </div>
        <button type="submit" onClick={this.submit}>
          Create account
        </button>
      </div>
    );
  }
}
```

We now have three different inputs named `username`, `email` and `password`, and we keep each entered value in a state with the same name. The method `handleChange` is reused to keep track of each change of value. The trick here is to use the name of the input element to update the corresponding state. Finally, when the user clicks on the submit button, the `submit` method is called to process the values. They are currently just displayed in the console but you could imagine validating the format of these values and sending them in a POST request.

**Additional note:** Have you seen this strange syntax in the `setState` of `handleChange`? It's called a computed property name. In a Javascript object, you can use a variable wrapped in square brackets which acts as a dynamic key, such as: 

```
const myFirstKey = "key1";
const myFirstValue = "value1";
const dynamicKeyObject = { [myFirstKey]: myFirstValue };
console.log(dynamicKeyObject); // => { key1: "value1" }
```

> **Exercise D**
> Open the `pokedex` React application again and open the `src/CaughtPokemon.js` file. In this exercise, instead of recording the number of caught Pokemons, we are going to record the names of each Pokemon you caught.
> 1. Make sure the `CaughtPokemon` component is written as a class component
> 2. Add an `<input>` in the `render` method before the `button` (hint: `<input type="text" />`)
> 3. Add a `value` property to the `<input>` set to the state `pokemonNameInput`
> 4. Initialize the state `pokemonNameInput` in the constructor to an empty string `''` (you can try to set something else than an empty string and verify that this value is correctly displayed in your input)
> 5. Create a new `handleInputChange` method
> 6. Add a `onChange` handler to the `<input>` that will call `this.handleInputChange`
> 7. Add a parameter called `event` to the `handleInputChange` method and add a `console.log` with `event.target.value`. In your browser, try writting something in the `<input>`. What do you see in the JavaScript console?
> 8. Use `setState` in `handleInputChange` to record `event.target.value` in the state `pokemonNameInput`. In your browser, try writting something in the `<input>`. What do you see this time in the JavaScript console?
> 9. We are now going to save the user input when clicking on the `<button>`. Initialize `caughtPokemon` to an empty array `[]` instead of 0 in the `constructor`. In the `render`, use `.length` to display the number of items in the state array `caughtPokemon` (hint: it should still display `0` on the screen). Finally, delete the content of `catchPokemon` method (it should be empty, we will rewrite it later).
> 10. In `catchPokemon` method, create a variable `newCaughtPokemon` set to the state `caughtPokemon` and add the value of the state `pokemonNameInput` to it (hint: use `push()` to add a new item in an array).
> 11. In `catchPokemon` method, use `setState` to record the variable `newCaughtPokemon` in the state `caughtPokemon`. Open your browser, enter a pokemon name in the `<input>` and click on the button. Can you see the number of caught pokemon incrementing as you click on the button?
> 12. We are now going to display the names of the caught pokemon. In the `render` method, add a `<ul>` element and use the `.map()` method on the `caughtPokemon` state to loop over each pokemon and return a `<li>` element for each.
> 13. Empty the `<input>` after clicking on the button. For this, in `catchPokemon` method, set the state of `pokemonNameInput` to an empty string `''`.
> 14: **(STRETCH GOAL)** Make sure the user cannot add a pokemon to the `caughtPokemon` state if the value of `pokemonNameInput` state is empty.


## Further Reading

There is a new update to React, which adds a new feature called *Hooks*. It allows you to access the special super powers of state and lifecycle in regular function components. There is an extensive guide in the [official React tutorial](https://reactjs.org/docs/hooks-intro.html).

# Homework

{% include "./homework.md" %}
