# IDIOMATIC REDUX
These are *short* notes I took while watching [Dan Abramov](https://github.com/gaearon)'s course on [Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)

For a more thorough summary of the course I recommend going through [this repo](https://github.com/tayiorbeii/egghead.io_idiomatic_redux_course_notes) by [Taylor Bell](https://github.com/tayiorbeii)

## 01. Simplifying the Arrow Functions
- action creators as arrow functions can be used to reduce the amount of code
- use concise method notation inside `mapDispatchToProps` since it's shorter

## 02. Supplying the Initial State
- initial state may be passed inside `createStore` and will override the default value in the reducer
- specifying the full state tree in `createStore` is discouraged as it breaks encapsulation
- however, if it's only hydrating persisted data created by redux itself, then it's perfectly fine as encapsulation is still kept

## 03. Persisting the State to the Local Storage
- localStorage may be used to persist data
- setting and getting should be wrapped in try/catch since these operations might be blocked by the browser's privacy settings
- uuid may be used instead of a counter to avoid ids collision when re-running the app
- `node_uuid` may be used for that
- `throttle` from `lodash` can be used to ensure that we don't call the expensive `JSON.stringify` too often

## 04. Refactoring the Entry Point
- extract `configureStore` and the `Root` component from the app's entry point
- this makes testing easier

## 05. Adding `React Router` to the Project
- `React Router` - standard stuff

## 06. Navigating with `React Router <Link>`
- use `Link` from `React Router` to control the part of the state that is kept in the URL

## 07. Filtering Redux State with `React Router` Params
- `React Router` will inject URL parameters in the `props.params` object
- when using a router, backend should be configured to always serve the JS entry point so the frontend router can kick in
- since the router becomes the source of truth for this part of the state, we can remove the reducer/actions related with it

## 08. Using `withRouter()` to Inject the Params into Connected Components
- `withRouter` will inject the router as well as other related props as params (it hides the manual context injection from you)
- useful for using the router in deeply nested component
- requires v3.0

## 09. Using `mapDispatchToProps()` Shorthand Notation
- use a hash inside `mapDispatchToProps` to map prop names to action creator functions

## 10. Colocating Selectors with Reducers
- place selectors in the same file as the corresponding reducer
- reducer will be the default export and selectors will be named exports
- the root reducer will also expose the selectors as named exports
- this way knowledge of shape of the state is limited to a single file

## 11. Normalizing the State Shape
- we may use a hash map instead of an array to store objects
- combineReducers may be used multiple times
- as reducers grow it may be useful to refactor some parts into separate files
- this works nice when combining object spread operator and computer property operator
```js
return {
    ...state
    [action.id]: todo(state[action.id], action)
}
```

## 12. Wrapping `dispatch()` to Log Actions
- when creating the store, we can override the `dispatch` function to add custom logic, such as logging
- this is practically a manual way of writing a middleware

## 13. Adding a Fake Backend to the Project
- demonstrating how to mock a backend API

## 14. Fetching Data on Route Change
- we cannot override lifecycle hooks in generated components (like the ones returned by calling `connect`)
- in order to override `componentDidMount` we use a higher-level component and pass it to connect
- we also need to override `componentDidUpdate` since when props change the component isn't going to call `componentDidMount` a second time

## 15. Dispatching Actions with the Fetched Data
- extracting the data fetching to a function inside the component
- adding `receiveTodos` as an action creator that is called when the API call is successful
- use `import * as actions` to namespace all the actions in a single object

## 16. Wrapping `dispatch()` to Recognize Promises
- data fetching can be written as an action creator (`fetchTodos`)
- the asynchronous `fetchTodos` method will return a promise and use the then method to pass the result through the `receiveTodos` action creator
- however, by default, redux only supports the dispatching of plain object and cannot handle a promise object
- we can use the same technique from `LESSON 12` to enhance the store with a dispatch function that can handle promises
- taking the sync/async handling away from the components reduces their responsibility an knowledge which is a good thing!

## 17. The Middleware Chain
- we recognized a repeated pattern of overwriting the `store.dispatch` method in order to add custom functionality
- this can be refactored to be a more strict interface of getting the store and the "next" dispatch method and returning an enhanced dispatch method
- this interface is known as a `middleware`
```js
store => next => enhancedDispatch
```
- the middleware chain will be applied in reverse so the order of middlewares defined in the array will reflect the order of calling the dispatch methods returned by them

## 18. Applying Redux Middleware
- redux ships with a utility function that applied middlewares when creating the store called `applyMiddleware`
- it may be passed as a 2nd or 3rd (optional) argument to the `createStore` method. this argument is called `enhancer`
- many middlewares (including logger and promise support) are available as `npm` packages

## 19. Updating the State with the Fetched Data
- refactoring the reducers to deal with data that is fetched incrementally from the backend API
- the store now has a lookup table of `todosById` and arrays of `idsByFilter` for every filter

## 20. Refactoring the Reducers
- refactoring reducers to be more DRY
- reducers export selectors so the knowledge of the shape of the state tree is encapsulated

## 21. Displaying Loading Indicators
- adding a boolean flag to the lists to indicate an in-flight fetch operation
- as we add a new field and change the shape of the state tree, we now see the advantage of encapsulating access to the state tree inside the reducer that handles it

## 22. Dispatching Actions Asynchronously with Thunks
- introducing the `thunk` middleware
- this middleware lets us dispatch functions instead of objects to the store
- the function will get `dispatch` as an argument and it is able to call it multiple times across the time span of an async operation
- this technique is more powerful than the previously introduced promise middleware since it enables us to describe complex async flows while a promise only expresses the async value of a single operation

## 23. Avoiding Race Conditions with Thunks
- a `thunk` can be used to avoid unnecessary network requests (if a requests is already being made, and hasn't returned yet)
- in order to do so, the thunk needs access to the store. it can get it using the `getState` second argument provided to the thunk
- as a convention, it's convenient to always return a promise even if the API hasn't been called

## 24. Displaying Error Messages
- when writing thunks, we can leverage the 2nd argument of the `then` method (the rejection handler) to dispatch an error action
- for better consistency and clarity we can postfix async actions with `REQUEST/SUCCESS/FAILURE`
- we handle the error action in a combined reducer which is for setting and clearing the error message
- a reusable error message component with `errorMessage, onRetry` props is rendered whenever we get a truthy error property
- using `catch` instead of using the rejection callback has a downside where it will also catch any errors thrown by reducers or components that are subscribed to the store

## 25. Creating Data on the Server
- demonstrates how to work against a mocked `addTodo` backend endpoint

## 26. Normalizing API Responses with `normalizr`
- the `normalizr` utility can be used to normalize server responses
- this helps reduce code and avoid writing special case handlers in reducers

## 27. Updating Data on the Server
- completes the app by connecting toggleTodo to a backend endpoint that updates the data on the server
- because we introduced `normalizr` before, the `byId` reducer doesn't need to change
- we still need to update the all/active/completed lists when the operation is successful
