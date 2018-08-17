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

### withLoader()

```
const withLoader = WrappedComponent => {
  const withLoader = props => branch(
    () => props.loading,
    withProps({message: props.loadingMessage})(Loading), // show the loading
    WrappedComponent
  )(props)

  return withLoader
}
```

---

### hasError()
```
const hasError = (ErrorComponent = Error) => WrappedComponent => {
  const HasError = props =>
    branch(props.hasError, ErrorComponent, WrappedComponent)(props)

  return HasError
}
```

```
hasError(CustomErrorComponent)(Component)
```

---

### withData()
fetches data and provides it to the component locally

```
const withData = ({url, params, loadingMessage}) => WrappedComponent => {
  class WithData extends React.Component {
    state = {
      data: [],
      hasError: false,
      error: {
        title: 'Cannot retrieve Real Posts',
        message: 'Could not retrieve Real Posts from supplied API.'
      },
      useDefault: false,
      loading: false,
      loadingMessage
    }

    componentDidMount() {
      this.setState({loading: true})

      axios.get(url, { params })
      .then(({data}) => {
        this.setState({
          data,
          loading: false,
          hasError: false,
          useDefault: data.length === 0
        })
      })
      .catch(error => {
        console.log(error)
        this.setState({
          hasError: true,
          loading: false
        })
      })
    }

    render() {
      return (
        <WrappedComponent 
          {...this.state}
          {...this.props}
        />
      )
    }
  }

  return WithData
}
```

```
withData({
  url: 'https://jsonplaceholder.typicode.com/posts',
  params: {
    _limit: 10,
    page: 2
  },
  loadingMessage: 'Loading posts from JSON Placeholder...'
})(Component)
```

---






