# react-patterns
This is a collection of usefull react patterns for day to day use.

### Branc HOC
```
const branch = (test, ComponentOnPass, ComponentOnFail) => props => test()
  ? <ComponentOnPass {...props} />
  : ComponentOnFail
    ? <ComponentOnFail {...props} />
    : null
```

```
branch(testFunction, Component1, Component2)
```

---
