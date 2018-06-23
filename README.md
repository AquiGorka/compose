# Compose

React component composition helper. Benefits:

- Split view from store and/or side effects, basically from any non-view logic.
- Split stores from side effects.
- No need for extra libraries, keep everything in Components.

```js

// view should be able to handle the possible states (eg. loading)
const View = ({ value, valueLoading }) => valueLoading
  ? <div>Loading...</div>
  : <div>{value}</div>

// store: get, put, post, delete, etc
class Store extends Component {
  state = { value: null, loading: false }

  render() {
    const { children, ...rest } = this.props
    const { value, loading } = this.state

    <Fragment>
      {Children.map(children, child =>
        cloneElement(child, {
          ...rest,
          valueGet: this.get,
          valuePost: this.post,
          valueLoading: loading,
          value
        }),
      )}
    </Fragment>
  }

  get = () => {
    // fetch value
    this.setState({ loading: true })
    setTimeout(() => this.setState({ value: 1, loading: false }), 1000)
  }

  post = () => { /*...*/ }
}

// side effects, like fetching value on mount
class Saga extends Component {
  componentDidMount() {
    this.props.valueGet()
  }

  render() {
    const { children, ...props } = this.props
    const { valuePost, ...rest } = props

    <Fragment>
      {Children.map(children, child =>
        cloneElement(child, { ...rest, valuePost: this.post }),
      )}
    </Fragment>
  }

  post = data => {
    // handle side effects, then dispatch the action further up
    // possible side effect eg. some other request
    this.props.post(data)
  }
}

const SagaStore = Compose(Store, Saga)

export { SagaStore as Store, View }
export default Compose(SagaStore, View)
```
