<img src="/assets/main.jpeg" width="1920px"/>

This repository was created to help react developers optimise their codebase and figure out the way to write less boilerplate code. Most of the hooks in this repo were not created by me, they are borrowed from blogs, community resources and also from the other developers i worked with. My goal is just to accumulate knowledge and share it with others. Please note, not all hooks here are in their original form,  sometimes i made some changes to fit my goals or modified them based on what works better in my experience. So if you find any mistakes, please let me know.

Without further do, let's cut to the chase!

# `useStickyState`

There are situations when we need to persist state between sessions, which obviously leads us to deal with localSorage values. To serve this purpose i like to use hook called useStickyState.

### `Code`

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
### `Usage`

It's used just like React.useState, except it takes two arguments: a default value, and a key:

```javascript
const SomeComponent() {
  const [person, setPerson] = useStickyState('Josh Comeau', 'the-creator-of-this-hook');
}
```

The second argument, key, will be used as the localStorage key. It's important that each useStickyState instance uses a unique value.

### `Quick typescript note`
This hook is not strictly typed and simply infers the type of the value passed in.
