# Debugging Notes

# Node

## Version Problems

Sometimes the version of node can be too up to date and will cause issues when running with our apps because they were created previous to that version.

### Local Fix

We can have the student install a Node Version Manager and downgrade their local version of node to something that the app will work with.

### Deployment Fix

For a Heroku deploy or similar you can state what version of Node you would like to use in the root package.json.

If the app is working locally then run

    node -v

And put that version of node in the package.json in a new object called "engines" like so

```javascript
    "engines": {
     "node": "<node.version.here>"
    }
```

# Mongo

## data/db is not found (or something like that)

In rare cases the data/db directory where mongo creates its databases becomes unwritable or dissapears or some bullshit happens that I have no clue why or how. But we can fix it.

Simplest thing to do is create a new folder with the same structure as data/db. So something like "mongodb-data/db". We will create this folder in the root directory of their computer and then we need to change the path that mongo is taking to get to its database directory. To do this you can run

    mongod --dbpath ~/path/to/the/directory

# GraphQL

## 400 Bad Requests

Sometimes the bad request exists because they are not passing in the variable correctly. For example

```javascript
    const [fooBar, {error}] = useMutation(FOO_BAR)

    function exampleCall(exampleParam){

    const {loading, data} = await fooBar({
        variables: exampleParam
     })
    }
```

This is wrong (often times) because graphQL is expecting the format of the variables passed in to be

```javascript
{
  exampleParam: "exampleString";
}
```

However it is currently only receiving

```javascript
examplestring;
```

To get around this the student may either put

```javascript
    const [fooBar, {error}] = useMutation(FOO_BAR)

    function exampleCall(exampleParam){

    const {loading, data} = await fooBar({
        variables: exampleParam: exampleParam //This is the thing that changed
    })
    }

    //or

    const [fooBar, {error}] = useMutation(FOO_BAR)

    function exampleCall(exampleParam){

    const {loading, data} = await fooBar({
        variables: { exampleParam } //Notice the curlies around exampleParam
    })
    }
```

this will format the variables section properly so graphQL receives what it is expecting. If you are confused console log

```javascript
// Will log exampleParam
console.log(exampleParam);

// Will log exampleParam: "exampleParam"
console.log({ exampleParam });
```

# React

## Cannot update a component while rendering a different component

Found a good article on this issue that is linked below

[Stack Overflow Link](https://stackoverflow.com/questions/62336340/cannot-update-a-component-warning)

This has happened when I had a child component changing the state of a parent component. When I did the parent would render and entirely different component and I believe that's why the error is thrown. You can avoid this by putting your setState hook into a useEffect so it will catch on a render cycle instead of being simultanious. Or something.

```javascript
    //bad version
    function Parent(){
        const [state, setState] = useState("")

        switch (state) {
            case "":
                return <Child setState={setState}>
            case "next":
                return <DiffComponent>
            default:
                break
        }
    }

    function Child({setState}){
        const [foo, setFoo] = useState(false)

        function changeBoolean(){
            foo = !foo
        }

        // While this if statement will check and work properly in a js sense, 
        // react will throw the cannot update err unless it is wrapped in an if statement
        if(foo === true) {
            setState("next")
        }

        return <button onClick={changeBoolean}>Click Me</button>
    }
```

```javascript
//good version
 function Parent(){
        const [state, setState] = useState("")

        switch (state) {
            case "":
                return <Child setState={setState}>
            case "next":
                return <DiffComponent>
            default:
                break
        }
    }

    function Child({setState}){
        const [foo, setFoo] = useState(false)

        function changeBoolean(){
            setFoo(prevState => !prevState)
        }

        useEffect(() => {
            if(foo === true) {
                setState("next")
            }
        }, [foo])//useEffect will run because foo's state has changed, running the if statement and changing the parent state

        return <button onClick={changeBoolean}>Click Me</button>
    }
```

# CSS

## Centering something that is using view width (vw)

An interesting trick I found is that if you are using postition absolute on an element and you want it to remain centered while also using vw for telling it how large to be is that you can take the vw value you have put in, subtract it from 100, divide that by 2, and that will give you the value you can put in for a position property like left or right. Here's a way less confusing example

```html
<div class="center-me">BS in here</div>
```

```css
.center-me{
    position: absolute;
    width: 70vw; /* 100vw - 70vw = 30vw */
    left: 15%; /* 30/2 = 15% from the left side of the screen */
}
```
This works because you are saying to put the element 15% from the left side of the screen. Since the width of the element takes up another 70% of the screen you end up with the 85% of the screen width taken up with 15% just being essentially padding on the left side and that leaves another 15% for the other side of the screen to be blank space. So, to sum up, 

    half of the leftover vw on the left ---> the element in the middle <--- the other half of the leftover vw on the right