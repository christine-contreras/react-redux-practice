# React Redux Notes

## 3 Core Concepts

- Store: holds the state of you app (on source of truth)
- Action: describes what happened
- Reducer: ties the store and actions together

## 3 Principles

1. the state of your app is stored in an object tree within a single store
2. the only way to change that state is to emit an action, an object describing what happened
   - action has a type: action name
3. to specify how that state tree is transformed by actions, you write pure reducers (pur functions)
   - (previousState, action) => newState

### Breakdown of principles

- you have your react app and you have your redux STORE. the app can not directly make changes to the store
- in order to make a change you must DISPATCH an ACTION
- The reducer handles the action and updates the current state
- as soon as the state is updated the value is passed on to the app and the view is updated

## Actions

- the only way to app can interact with the store
- carry information for your app to the store
- are plain JS objects
- have a TYPE property that indicates the type of action being performed

``js
const BUY_CAKE = 'BUY_CAKE'

{
type: BUY_CAKE,
}
``

### Action Creator

- function that returns an action

``js
const BUY_CAKE = 'BUY_CAKE'

const buyCake = () => {
return {
type: BUY_CAKE,
info: 'redux action'
}
}
``

## Reducer

- how the app state changes based on the actions send to the store
- state has to be represented by one object
- accepts state and action as arguments
- set state to initial state by default
- make a copy of state as to not mutate

``js
// (previousState, action) => newState

const initialState = {
numOfCakes: 10,
}

const reducer = (state = initialState, action) => {
switch(action.type) {
case BUY_CAKE: return {
...state
numOfCakes: state.numOfCakes - 1
}

        default: return state
    }

}
``

## Redux Store

- brings the actions and reducers together
- holds app state
- allows access to the state via getState()
- Alows state to be updated via dispatch(action)
- Registers listeners via subscribe(listener)

``js
import redux from 'redux'
const createStore = redux.createStore

const store = createStore(reducer)

console.log('initial state', store.getState())

const unsubscribe = store.subscribe(() => console.log('update state',
store.getState()))

store.dispatch(buyCake())

unsubscribe()
``

## Multiple Reducers

``js
const BUY_CAKE = 'BUY_CAKE'
const BUY_ICE_CREAM = 'BUY_ICE_CREAM'

const buyCake = () => {
return {
type: BUY_CAKE,
info: 'redux action'
}
}

const buyIceCream = () => {
return {
type: BUY_ICE_CREAM,
}
}

const initialCakeState = {
numOfCakes: 10,
}

const initialIceCreamState = {
numOfIceCreams: 20
}

const cakeReducer = (state = initialCakeState, action) => {
switch(action.type) {

    case BUY_CAKE: return {
    ...state
    numOfCakes: state.numOfCakes - 1
    }

        default: return state
    }

}

const iceCreamReducer = (state = initialIceCreamState, action) => {
switch(action.type) {

    case BUY_ICE_CREAM: return {
    ...state
    numOfIceCreams: state.numOfIceCreams - 1
    }

        default: return state
    }

}

const combineReducers = redux.combineReducers
const rootReducers = combineReducers({
cake: cakeReducer,
iceCream: iceCreamReducer
})

const store = createStore(rootReducers)

store.dispatch(buyCakes())
store.dispatch(buyIceCream())
``

if you want to reach the value it would now be state.cake.numOfCakes

## Middleware

- the suggest way to extend Redux with custom functionality
- 3rd party extension point between dispatching and actions

``js
const applyMiddleware = redux.applyMiddleware

const store = createStore(rootReducer, applyMiddleware(logger, saga))
``

## Async Actions

- ex: api call to fetch data

``js
const initialState = {
loading: false,
data: [],
error: ''
}

//Actions
FETCH_USERS_REQUEST = 'FETCH_USERS_REQUEST'
FETCH_USERS_SUCCESS = 'FETCH_USERS_SUCCESS'
FETCH_USERS_FAILURE = 'FETCH_USERS_FAILURE'

//Action Creators
const fetchUsersRequest = () => {
type: FETCH_USERS_REQUEST
}

const fetchUsersSuccess = () => {
type: FETCH_USERS_SUCCESS,
payload: users
}

const fetchUsersFailure = () => {
type: FETCH_USERS_FAILURE,
payload: error
}

//Reducer
const fetchReducer = (state = initialState, action) => {
switch(action.type) {
case FETCH_USERS_REQUEST:
return {
...state,
loading: true
}

        case FETCH_USERS_SUCCESS:
        return {
            ...state,
            loading: false,
            users: action.payload,
            error: ''
        }

        case FETCH_USERS_FAILURE:
        return {
            ...state,
            loading: false,
            users: []
            error: action.payload,
        }
    }

}

const store = createStore(reducer)

``

## Async Action Creators

- axios (optional)
- redux-thunk
- redux-saga

action creators usually return an object. with thunk it returns a function
``js

const fetchUsers = () => {
return function(dispatch) {
dispatch(fetchUsersRequest())

        axios.get('https://jsonplaceholder.typicode.com/users')
        .then(response => {
            const users = response.data
            dispatch(fetchUsersSuccess(users))

        })
        .catch(error => {
            //error.message
            dispatch(fetchUsersFailure(error.message))
        })
    }

const store = createStore(reducer, applyMiddleware(thunkMiddleware))
store.subscribe(() => { console.log(store.getState())})

store.dispatch(fetchUsers())
}
``
