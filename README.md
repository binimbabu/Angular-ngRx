ngRx


ngRx helps with state management. ngRx is a state management solution (a package into Angular projects to assist in managing application-wide state. State is data used in one or more components in the application, that changes over time (eg: user clicked a button and those data changes should lead to change in the application, typically to a UI update). ngRx is a used to manage complex state instead of managing in components or services.
ngRx is a state management solution, a library that helps with managing state data in the application (ngRx a library clearly defined way of managing application state).
ngRx state management starts with the following:-
once ngRx installed in Angular project then create a store
Create a state store (thats store is where data is stored and managed by ngRx, store is where components reach out to read the data ). When component needs some data and it should update when the data changes, it can read data from that store and can listen to data changes. So, Angular can update the UI as the data in the store changes. For reading the data and extracting the data from the store it can leverage a concept called Selectors. With the help of store and selectors components can get the globally managed state data. With ngRx you have one central store in the application that manages all data used in entire application and different components can then get the data they need from store. Data change over time for example because a button was clicked or form was submitted. In ngRx to change the data in store, components must dispatch actions (which describes the changes that need to be performed and add extra data that is needed to change the data in the store). And then these actions are picked up by so called Reducers (reducers are defined for setting up this entire state management system). In Reducers have the logic that gets triggered based on those actions to change the data in the store. effects or side effects for example HTTP requests that should be sent to backend that should maybe also triggered when certain actions occur (so action doesn't just lead to data in that store being changed and also HTTP requests sent to backend ) 


ng add  @ngrx/store


After running the above command we get the below in imports in app.module.ts

StoreModule.forRoot({}, {})

which is used for creating an empty store.

To get data to store or changes data in store we need a reducer, in ngRx data is changed in store using Reducer. It is the reducer which sets up the initial value and changes the data over time in store.

Create a 'store' folder in app where the file (here i.e counter.reducer.ts) that defines the reducers are there in this file. 
'counter.reducer.ts' add the reducer thats responsible for managing data that belongs to counterpart of this app in this file.


app/store/counter.reducer.ts


import { createReducer } from '@ngrx/store';

const initialState = 0;
export const counterReducer = createReducer(initialState);




'counterReducer' will be a reducer responsible for managing counter data.
Reducer can be created with createReducer (imported from import { createReducer } from '@ngrx/store';)' . Reducer is a function that takes data as input and splits out the updated state (update counter state here where updated state to be stored in store). 'createReducer' needs atleast 1 argument here that 1 argument is 'initialState', where 'initialState' is the piece of global store. 'initialState' can be a number, text, Boolean value, object with counter value, can be an array. ( in this case 'initialState' is a number hence we set the value to 0 ).
We can make the 'counterReducer' available in 'StoreModule.forRoot({}, {})' in app.module.ts (like below) where we pass as an objevt here we pass key as 'counter' and value will be the 'counterReducer'.


 app.module.ts

StoreModule.forRoot({
    counter: counterReducer
  })],


the above code will construct a store that combines all the data from all these reducers and different pieces of data coming from different Reducers will be stored in one global store object by using those keys here 'counter'. So, these keys used for assigning the Reducer can be used for extracting data from store (data managed by Reducer).


Alternative way of creating reducer:-


app/store/counter.reducer.ts


export function counterReducer(state = initialState) {
  return state;
}


Here 'state' as the argument to 'counterReducer' is the current state managed by ngRx because this reducer function (here 'counterReducer' ) will be later executed by ngRx. This 'counterReducer' reducer function will be executed whenever an action occurs or when it initially sets up the store (similar to 'createReducer' created above). When 'counterReducer function is executed then it expects to return updated new state. Initially 'counterReducer' will set value to 'initialState' since the state hasn't changed and has no value.


Read Data from store



counter-output.component.ts


import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { Store } from '@ngrx/store';

@Component({
  selector: 'app-counter-output',
  templateUrl: './counter-output.component.html',
  styleUrls: ['./counter-output.component.css'],
})
export class CounterOutputComponent {
  count$: Observable<number>;
  constructor(private store: Store<{ counter: number }>) {
    this.count$ = store.select('counter');
  }
}



in counter-output.component.ts file we inject 'Store' which is globally managed store.
'store.select' used to select data from store


 constructor(private store: Store<{ counter: number }>) 

Here in the above code we are telling typescript that the Store will be an object that has 'counter' key because we used 'counter' key in app.module.ts for assigning the Reducer. Data stored under the key (here 'counter') will be a number and it will be a number because in the app my counter Reducer will return a number (i.e in 'counterReducer' function in app/store/counter.reducer.ts ) 

When selecting values from store we get an Observable (advantage of using Observable is the data updated when in the Store gets changed). As the data in store changes as the Observable is used in 'counter-output.component.ts' to get the value in store (in the variable count$ ) the 'counter-output.component.ts' will listen to that change and update the UI.


counter-output.component.html

<p class="counter">{{ count$ | async }}</p>

Here async pipe used to listen to observable 




Actions and State changing reducers

To change the value in reducer we need actions. To dispatch actions for example from inside your components ngRx can call those reducers, so that reducers can then work on the latest state thats managed by ngRx and update accordingly.

In 'store' folder we add another file counter.actions.ts ( this file is to define actions that can be dispatched in this application).
For example in the counter application to increment and decrement the counter we need actions.


store/counter.actions.ts


import { createAction } from '@ngrx/store';

export const increment = createAction('[Counter] Increment');



'createAction' first argument is type/ identifier of the action. The identifier must be unique ( dont use the same identifier for different actions) and it is of type string. '[Counter]' in the identifier describes the application and 'Increment' in the identifier denotes the action. This action 'increment' need to be used in the Reducer.


app/store/counter.reducer.ts


import { createReducer } from '@ngrx/store';

const initialState = 0;
createReducer(
  initialState,
  on(increment, (state) => state + 1)
);





Value created by another function provided by ngRx store and that function is 'on'. 'on is used as a function to the second argument in 'createReducer'. In 'on' first argument passed is the action created i.e here 'increment' , second argument in 'on' is a function itself (function that will be executed by ngRx whenever 'increment' action and this function should contain logic that updates the state, this function will automatically receive current state as an input (here 'state'). But this 'state' shouldn't be directly change the existing state but instead always produce a new value, if the 'initialState' used is an array or object. This function will return old state + 1 and that will be used and stored by NgRx as the new state value for the slice of data managed by Reducer ). Store consist of multiple pieces of data, but the piece of data managed by this Reducer will be updated whenever the increment action is dispatched with the action of this function.


Dispatching Actions


Inject ' constructor(private store: Store) {}' in 'counter-controls.component.ts' file from 'import { Store } from '@ngrx/store';'

In the increment method in 'counter-controls.component.ts' dispatch the 'increment' action from importing 'increment' from 'counter.actions' (  'import { increment } from '../store/counter.actions';')

 increment() {
    this.store.dispatch(increment());
  }



counter-controls.component.ts


import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { increment } from '../store/counter.actions';

@Component({
  selector: 'app-counter-controls',
  templateUrl: './counter-controls.component.html',
  styleUrls: ['./counter-controls.component.css'],
})
export class CounterControlsComponent {
  constructor(private store: Store) {}

  increment() {
    this.store.dispatch(increment());
  }

  decrement() {}
}



When action is dispatched then only  'increment()' action works in 'counter.actions'  executed (action has to be dispatched for the action to be executed), 'increment()' action not executed in counter.reducer.ts file 'on' operation.
After action dispatched in the 'counter.reducer.ts ' it will listen and define what should happen to the state when action occurs in  reducer 'counter.reducer.ts. (dispatching the increment action which is changing the counter with help of reducer)



counter.reducer.ts


import { createReducer, on } from '@ngrx/store';
import { increment } from './counter.actions';

const initialState = 0;
export const counterReducer = createReducer(
  initialState,
  on(increment, (state) => state + 1)
);




Attaching Data to actions

'createAction' in 'counter.actions.ts' can have second argument which is an object that describes the kind of data that can be attached to action which can be done by extra 'props' function (provided by '@ngrx/store')  (i.e import { createAction, props } from '@ngrx/store';) 
'createAction' in 'counter.actions.ts' second argument passed can execute props as a function where we props is a generic type which describes the kind of data can be attached to action where we can add a key named value (can be the type of value  attached to props which describes the shape of the data that should be attached to the action, here the counter value to be incremented so given like this 'props<{value: number}>()'  ) to the props type.


counter.actions.ts

import { createAction, props } from '@ngrx/store';

export const increment = createAction(
  '[Counter] Increment',
  props<{ value: number }>()
);

export const decrement = createAction(
  '[Counter] Decrement',
  props<{ value: number }>()
);



To extract this data go to the reducer and in the 'on' function where the increment action occurs can accept a second argument an 'action' argument in 'counter.reducer.ts'. 'action' argument has a 'value' in 'counter.reducer.ts' because we used 'value' as key name in 'counter.actions.ts'. In this way we can extract 'action.value' from reducer.




counter.reducer.ts


const initialState = 0;
export const counterReducer = createReducer(
  initialState,
  on(increment, (state, action) => state + action.value),
  on(decrement, (state, action) => state - action.value)
);


In 'counter.controls.component.ts' we pass the second argument in the dispatch action as '2', so now counter value will be incremented by 2

  increment() {
    this.store.dispatch(increment({ value: 2 }));
  }



counter.controls.component.ts


import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { increment } from '../store/counter.actions';

@Component({
  selector: 'app-counter-controls',
  templateUrl: './counter-controls.component.html',
  styleUrls: ['./counter-controls.component.css'],
})
export class CounterControlsComponent {
  constructor(private store: Store) {}

  increment() {
    this.store.dispatch(increment({ value: 2 }));
  }

  decrement() {
    this.store.dispatch(decrement({ value: 2 }));
  }
}






Handling action without createReducer



export function counterReducer(state = initialState, action: any) {
  return state;
}


second argument in 'counterReducer' is 'action' with type (here type is 'any'). In theory, a reducer can handle any action dispatched anywhere in the application.

Inside the reducer we need to update the state depending on which action is dispatched. each reducer handles its slice and its state of this slice thats managed by 'counterReducer' here. 'action.type' comes from action here 'counter.actions.ts'. If the 'action.type === '[Counter] Increment' then the new state should be produced by this 'counterReducer' should be old state with the value included in dispactched action. But if ther is unknown 'action.type'in this reducer then return the unchanged state.




counter.reducer.ts



import { createReducer, on } from '@ngrx/store';
import { decrement, increment } from './counter.actions';

export function counterReducer(state = initialState, action: any) {
  if (action.type === '[Counter] Increment') {
    return state + action.value;
  } else if (action.type === '[Counter] Decrement') {
    return state - action.value;
  }
  return state;
}




Alternative way of defining Actions


Instead of defining actions the below way

import { createAction, props } from '@ngrx/store';

export const increment = createAction(
  '[Counter] Increment',
  props<{ value: number }>()
);

export const decrement = createAction(
  '[Counter] Decrement',
  props<{ value: number }>()
);


we can call Action object where 'IncrementAction' class implements 'Action' which is imported from '@ngrx/store'. Then the default type is set as '[Counter] Increment'. constructor 'value' becomes a property of objects that are created based on this class (i.e IncrementAction) 
