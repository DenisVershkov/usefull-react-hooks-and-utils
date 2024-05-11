<img src="/assets/main.jpeg" width="1920px"/>

This repository was created to help react developers optimize their codebase and figure out a way to write less boilerplate code. Most of the hooks and utils in this repo were not created by me, they are borrowed from blogs, community resources, and also from the other developers I worked with. My goal is just to accumulate knowledge and share it with others. Please note, not all hooks and utils here are in their original form, sometimes I made some changes to fit my goals or modified them based on what works better in my experience. So if you find any mistakes, please let me know.

Without further do, let's cut to the chase!

</br>

# `useStickyState`

There are situations when we need to persist state between sessions, which obviously leads us to deal with localSorage values. To serve this purpose i like to use hook called useStickyState:
</br>

## :pencil2: Code

```typescript
const useStickyState = <T>(defaultValue: T, key: string) => {
const [value, setValue] = React.useState<T>(() => {
const stickyValue = window.localStorage.getItem(key);

        return stickyValue !== null ? JSON.parse(stickyValue) : defaultValue;
    });

    React.useEffect(() => {
        window.localStorage.setItem(key, JSON.stringify(value));
    }, [key, value]);

    return [value, setValue] as const;
};
```

<details>
  <summary><h2>:technologist: Usage example</h2></summary>     
        
```js
const SomeComponent() {
  const [person, setPerson] = useStickyState('Josh Comeau', 'the-creator-of-this-hook');
}
```
        
It's used just like React.useState, except it takes two arguments: a default value, and a key. The second argument, key, will be used as the localStorage key. It's important that each useStickyState instance uses a unique value.
</details>

## :bulb: Quick typescript note
This hook is not strictly typed and simply infers the type of the value passed in.

</br>

# `getErrorMessage`

Usually developers do something like this:

```js
try {
  //some code here
} catch (error) {
  notify({message: error.message})
}
```

But if we use typescript, it will yield at us that the error actually has type 'unknown' and we can't access the 'message' property without an additional type check. Utility function getErrorMessage gives us an ooportunity to handle this case in a simple and nice way:
</br>

### :pencil2: Code

```ts
type ErrorWithMessage = {
  message: string
}

function isErrorWithMessage(error: unknown): error is ErrorWithMessage {
  return (
    typeof error === 'object' &&
    error !== null &&
    'message' in error &&
    typeof (error as Record<string, unknown>).message === 'string'
  )
}

function toErrorWithMessage(maybeError: unknown): ErrorWithMessage {
  if (isErrorWithMessage(maybeError)) return maybeError

  try {
    return new Error(JSON.stringify(maybeError))
  } catch {
    // fallback in case there's an error stringifying the maybeError
    // like with circular references for example.
    return new Error(String(maybeError))
  }
}

function getErrorMessage(error: unknown) {
  return toErrorWithMessage(error).message
}
```
</br>

<details>
  <summary>:technologist: Usage example</summary>

```ts
try {
  //some code here
} catch (error) {
  notify({message: getErrorMessage(error)})
}
```
</details>

Now we can be sure that our 'catch' case handles properly because we use type safe function, which covers all failure scenario and prevents your code from unexpected collapse.

</br>

# `useDebounce`

I think that debounce needs no introduction, it's an indispensable helper on any front-end project, allowing us to do a lot of cool stuff, like preventing unnecessary api calls when user is typing in a search field. Here is a hook implementation:

### :pencil2: Code

```ts
export function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value)

  useEffect(() => {
    const timerId = setTimeout(() => {
      setDebounced(value)
    }, delay);

    return () => {
      clearTimeout(timerId)
    }
  }, [value, delay])

  return debounced
}
}
```

<details>
  <summary>:technologist: Usage example</summary>
  </br>

imagine we want to make a request to an api to get a list of autocomplete options:

```ts
const [query, setQuery] = useState('') //the actual input state
const searchQuery = useDebounce(query, 1000) //the state we can use to make a request since it is updated in 1 second after the user stops typing
```
</details>

</br>

# `useToggle`

Often when implementing new features we want to toggle something (modals, switchers, etc.), so why don't make this logic reusable?

### :pencil2: Code

```ts
const useToggle = (initialValue: boolean): [boolean, (nextValue?: boolean) => void] => {
  const [value, setValue] = useState(initialValue)

  function toggle(nextValue?: boolean) {
//we can pass an optional argument or the state simply will be changed to the opposite value
    setValue(current => nextValue ?? !current)
  }

  return [value, toggle]
}
```

<details>
  <summary>:technologist: Usage example</summary>
  </br>


```ts
const [on, toggle] = useToggle(true);

toggle()
//or
toggle(true)
```
</details>
