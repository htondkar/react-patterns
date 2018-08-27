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


### withRedirect()
Redirects user based on userType

```
/**
 * provides a redirect method based on userType
 * @param {object} urlByUser
 * @param {string} urlByUser.ADMIN
 * @param {string} urlByUser.REGULAR
 * @param {string} urlByUser.SOME_OTHER_TYPE
 */
const withRedirect = urlByUser => Component => {
  class WithRedirect extends React.Component {
    getUserType = () => this.props.userType

    redirect = param => {
      const userType = this.getUserType()

      if (!(userType in urlByUser)) this.error(userType)

      const urlToRedirect = urlByUser[userType].replace('{}', param)
      browserHistory.push(urlToRedirect)
    }

    error(userType) {
      throw new Error(
        `current user is a ${userType},
         which is not included in the
         withRedirect options`
      )
    }

    render() {
      return <Component {...this.props} redirectBasedOnUserType={this.redirect} />
    }
  }

  WithRedirect.displayName = `withRedirect(${Component.displayName || Component.name})`

  const mapStateToProps = state => ({ userType: state.userData.userType })
  return connect(mapStateToProps)(WithRedirect)
}

export default withRedirect

```

Usage:

```
withData({
  ADMIN: '/dashboard/stats/{}', // "{}" is a placeholder for params, etc...
  REGULAR: '/dashboard/profile/{}'
})(Component)
```

In the wrapped component: 

```
onDone() {
  props.redirectBasedOnUser()
}
```

---

### withPropsMapper()
modifies props with a mapperObject

```
/**
 * iterates over all keys in the mapperObject and if the same key exists on the props,
 * applies the mapper function on the prop
 * @param {object} mapperObject
 * @param {(value, allProps) => newValue } mapperObject.any
 */
const withPropsMapper = (mapperObject = {}) => WrappedComponent => {
  const WithPropsMapper = props => {
    // call the appropriate method on the mapperObject and pass the value and all props
    const propsReducer = (acc, currentMapperKey) => {
      acc[currentMapperKey] = mapperObject[currentMapperKey](
        props[currentMapperKey],
        props
      )
      return acc
    }

    // filter mapper object keys and reduce them to new props.
    const mappedProps = Object.keys(mapperObject).reduce(propsReducer, {})

    // let the mapped props override the default
    return <WrappedComponent {...props} {...mappedProps} />
  }

  WithPropsMapper.displayName = `WithPropsMapper(${WrappedComponent.displayName ||
    WrappedComponent.name})`

  return WithPropsMapper
}
```
Usage: 

```
const propsMapper = { orderId: (_, props) => props.match.params.orderId }
export default withPropsMapper(propsMapper)(MyComponent)
```

---





