# react-patterns
This is a collection of usefull react patterns for day to day use.

### branch()
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

### WithPropsLogger()
logs props of a component

```
const withPropsLogger = (prefix = '') => WrappedComponent => props => {
  console.log(`${prefix}[Props]:`, props)
  return <WrappedComponent {...props} />
}
```

```
withPropsLogger(Component)
```

---

### withMockData()
```
const withMockData = (mockData, delay) => WrappedComponent => {
  class WithMockDataWrapper extends React.Component {
    state = {
      data: [],
      useDefault: true
    }

    componentDidMount() {
      this.props.addTimeout(() => this.setState({data: mockData}), delay)
    }

    componentWillUnmount() {
      this.props.clearTimeouts()
    }

    render() {
      return <WrappedComponent data={this.state.data} {...this.props} />
    }
  }

  return WithMockDataWrapper
}
```

```
withMockData({data: 'This is mock data'})(Component)
```

---



