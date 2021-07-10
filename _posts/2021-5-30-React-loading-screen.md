---
layout: post
title: Different approach to loading screen in React
published: true
comments: true
excerpt_separator: <!--more-->
---

Most of the loading screens I saw, based on boolean variable `loading`. Then based on it, component `loadingScreen` is returned or the actual page. The more data I wanted to load, the more `if statements` I had to make to check if I am still loading. More such ifs sounds like a bad idea to me 🤷‍♀️.

<!--more-->

I wanted to make my `LoadingScreen` component smart enough to figure out, if it should be still displayed.

Let's keep loading screen simple for this example. If it has children, display them. Else, use default loader.

```js
// LoadingScreen.js
const LoadingScreen = (props) => {
  return (
    <div class="bg-gray-600 h-screen w-screen fixed">
      {props.children ? (
        <div>{props.children}</div>
      ) : (
        <div>Default loader...</div>
      )}
    </div>
  );
};
```

Since loader has to decide if data is already loaded, it needs to have access to those data. From the main component point of view it will look like this.

```js
// MainPage.js
const MainPage = (props) => {
    const [data, setData] = useState(undefined);

    useEffect(() => {
        if(typeof props.data !== 'undefined'){
            var keyValuePairs = Object.values(props.data).map((key) => <li key={key}>{key}</li>);
            setData(keyValuePairs);
        }else{
            props.makeRequest();
        }
    }, [props.data])

    return (
        <>
            <LoadingScreen toLoad={[data]}/>
            <div>
                <h2>Conent:</h2>
                <div>
                    {data}
                </div>
            </div>

        </>
    )
}

const mapStateToProps = (state) => {
    return {
        data: state.main.data
    }
}

const mapDispatchToProps = dispatch => ({
    makeRequest: () => dispatch(getData());
})

```

The simplest way of checking if data is already loaded, is checking if all elements in array `toLoad` are not `undefined`.

Let's add such check to `LoadingScreen` component.

```js
// LoadingScreen.js
const LoadingScreen = (props) => {
  const isDataLoaded = () => {
    for (var i in props.toLoad) {
      if (typeof props.toLoad[i] === "undefined") {
        return false;
      }
    }
    return true;
  };

  return isDataLoaded() ? null : (
    <div class="bg-gray-600 h-screen w-screen fixed">
      {props.children ? (
        <div>{props.children}</div>
      ) : (
        <div>Default loader...</div>
      )}
    </div>
  );
};
```

And that's it! `LoadingScreen` will be displayed till data will stay `undefined`. Another approach is to check if data is equal to it's initial state.

```js
// MainPage.js
<LoadingScreen toLoad={[data]} toLoadInitialState={[initialData]} />
```

And the check will be:

```js
// LoadingScreen.js

const isDataLoaded = () => {
  for (var i in props.toLoad) {
    if (props.toLoad[i] === props.toLoadInitialState[i]) {
      return false;
    }
  }
  return true;
};
```

Of course the problem will be when obtained data will be equal initial data but in most of my cases it does the job.

It is about one month since I started to learn React so feel free to point out any rookie mistake I made 😉.
