<img src="/assets/main.jpeg" width="1920px"/>

This repository was created to help react developers optimize their codebase and figure out a way to write less boilerplate code. Most of the hooks and utils in this repo were not created by me, they are borrowed from blogs, community resources, and also from the other developers I worked with. My goal is just to accumulate knowledge and share it with others. Please note, not all hooks and utils here are in their original form, sometimes I made some changes to fit my goals or modified them based on what works better in my experience. So if you find any mistakes, please let me know.

Without further do, let's cut to the chase!

</br>

# `useStickyState`

There are situations when we need to persist state between sessions, which obviously leads us to deal with localSorage values. To serve this purpose i like to use hook called useStickyState:

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
        
```javascript
const SomeComponent() {
  const [person, setPerson] = useStickyState('Some Key', 'some value');
}
```
        
It's used just like React.useState, except it takes two arguments: a default value, and a key. The second argument, key, will be used as the localStorage key. It's important that each useStickyState instance uses a unique value.
</details>

## :bulb: Quick typescript note

The hook simply infers the type of the value passed in, but when i use it in my projects i prefer strict typization to view all the values that might be stored in the localStorage.

```typescript
/** All possible keys for usePersistedState */
enum Keys {
    ZoneAlert = 'disable_zone_alert_for_date',
}

/** All possible key: value pairs for usePersistedState */
type Entries = {
    [Keys.ZoneAlert]: ZoneAlertDefaultState;
};

const useStickyState = <T extends Entries[Keys], U extends Keys>(defaultValue: T, key: U) => ...
```

</br>

# `parseError`

This sounds familiar, right?

```javascript
try {
  //some code here
} catch (error) {
  notify({ message: error.message });
}
```

But if we use typescript, it will yield at us that the error actually has type 'unknown' and we can't access the 'message' property without an additional type check. Utility function parseError gives us an opportunity to handle this case in a simple and secure way:

## :pencil2: Code

```typescript
type ErrorWithMessage = {
  message: string;
};

type ErrorWithStatus = {
  status: number;
};

function isErrorWithMessage(error: unknown): error is ErrorWithMessage {
  return (
    typeof error === "object" &&
    error !== null &&
    "message" in error &&
    typeof (error as Record<string, unknown>).message === "string"
  );
}

function isErrorWithStatus(error: unknown): error is ErrorWithStatus {
  return (
    typeof error === "object" &&
    error !== null &&
    "status" in error &&
    typeof (error as Record<string, unknown>).status === "number"
  );
}

function toErrorWithMessage(maybeError: unknown): ErrorWithMessage {
  if (isErrorWithMessage(maybeError)) return maybeError;

  if (typeof maybeError === "string" && maybeError) {
    return new Error(maybeError);
  }

  try {
    return new Error(
      maybeError ? JSON.stringify(maybeError) : i18n("error.message-default")
    );
  } catch {
    // fallback in case there's an error stringifying the maybeError
    // like with circular references for example.
    return new Error(String(maybeError));
  }
}

export function getErrorMessage(error: unknown) {
  return toErrorWithMessage(error).message;
}

const WELL_KNOWN_STATUSES_MAP: Record<string, string | undefined> = {
  403: i18n("title.error-403"),
  404: i18n("title.error-404"),
  500: i18n("title.error-500"),
};

export const parseError = (error: unknown) => {
  const status = isErrorWithStatus(error) ? String(error.status) : "500";

  return {
    description: getErrorMessage(error),
    status,
    title:
      WELL_KNOWN_STATUS_KEYSETS_MAP[status] ||
      i18nK("title.error-default") ||
      "",
  };
};
```

<details>
  <summary><h2>:technologist: Usage example</h2></summary>

```typescript
try {
  //some code here
} catch (error) {
  const { title, description, status } = parseError(error);

  notify({ title, description, status });
}
```

</details>

Now we can be sure that our 'catch' case handles properly because we use type safe function, which covers all failure scenario and prevents your code from unexpected collapse.

</br>

# `useFilters`

I think every frontend engineer at least once created a page with entities filters. Hook 'useFilters' can help you perform this typical task with ease.

## :pencil2: Code

```typescript
interface UseFiltersParams<T extends {}, F extends {}, O extends {}> {
  initialData: T[];
  initialFilters: F;
  filterFn: (data: T[], filter: F, options?: O) => T[];
  options?: O;
}

const useFilters = <T extends {}, F extends {}, O extends {}>(
  params: UseFiltersParams<T, F, O>
) => {
  const { initialData, initialFilters, filterFn, options } = params;

  const [filter, setFilter] = React.useState(initialFilters);
  const [filteredValue, setFilteredValue] = React.useState(initialData);

  React.useEffect(() => {
    setFilteredValue(filterFn(initialData, filter, options));
  }, [initialData, filterFn, filter, options]);

  const onFilterChange = React.useCallback(
    (currentFilter: F) => {
      setFilteredValue(filterFn(initialData, currentFilter, options));
    },
    [initialData, filterFn, options]
  );

  const handleChange = React.useCallback(
    (currentFilter: F) => {
      setFilter(currentFilter);
      onFilterChange(currentFilter);
    },
    [onFilterChange]
  );

  return [filteredValue, filter, handleChange] as const;
};
```

<details>
  <summary><h2>:technologist: Usage example</h2></summary>

```typescript
const INITIAL_FILTERS: RuleFilter = {
  tagOrDescriptionOrId: "",
  ruleGroup: "",
  ruleType: "",
};

const getFilteredRules = (rules: Rule[], filters: RuleFilter) => {
  const {
    tagOrDescriptionOrId,
    ruleGroup,
    ruleType,
    paranoiaLevel,
    activeState,
    blocking,
  } = filters;

  let result = rules;

  if (ruleGroup) {
    result = filterByField(result, "ruleGroup", ruleGroup);
  }

  if (ruleType) {
    result = filterByField(result, "ruleType", ruleType);
  }

  if (tagOrDescriptionOrId) {
    result = filterBySomeFields(
      result,
      ["description", "id", "tags"],
      tagOrDescriptionOrId
    );
  }

  return result;
};

const [filteredRules, filter, onFilterChange] = useFilters({
  initialData: rules,
  initialFilters: INITIAL_FILTERS_BASE,
  filterFn: getFilteredRules,
});
```

</details>

The hook can easily be upgraded to persist filters to the url string,i just showed you the basic version here.

</br>

# `useDebounce`

I think that debounce needs no introduction, it's an indispensable helper on any front-end project, allowing us to do a lot of cool stuff, like preventing unnecessary api calls when user is typing in a search field. Here is a hook implementation:

## :pencil2: Code

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timerId = setTimeout(() => {
      setDebounced(value);
    }, delay);

    return () => {
      clearTimeout(timerId);
    };
  }, [value, delay]);

  return debounced;
}
```

<details>
  <summary><h2>:technologist: Usage example</h2></summary>

```typescript
const [query, setQuery] = useState(""); // updates without a delay
const debouncedQuery = useDebounce(query, 1000); // updates with a 1 second delay
```

</details>

</br>

# `useToggle`

Often when implementing new features we want to toggle something (modals, switchers, etc.), so why don't make this logic reusable?

## :pencil2: Code

```typescript
const useToggle = (
  initialValue: boolean
): [boolean, (nextValue?: boolean) => void] => {
  const [value, setValue] = useState(initialValue);

  function toggle(nextValue?: boolean) {
    //we can pass an optional argument or the state simply will be changed to the opposite value
    setValue((current) => nextValue ?? !current);
  }

  return [value, toggle];
};
```

<details>
  <summary><h2>:technologist: Usage example</h2></summary>

```typescript
const [on, toggle] = useToggle(true);

toggle();
//or
toggle(true);
```

</details>
