# React-Typescript-Redux

## **1. Action 및 Reducer**

관리할 상태값이 많아질 경우 복잡할 것 같음 > types, actions, reducer를 분리하여 작성

📎 _modules/todos/types.ts_

```tsx
const ADD_TODO = 'todos/ADD_TODO';
const TOGGLE_TODO = 'todos/TOGGLE_TODO';

/** Action 타입 */
type addTodoAction = {
    type: typeof ADD_TODO;
    payload: string;
};
type toggleTodoAction = {
    type: typeof TOGGLE_TODO;
    payload: number;
};

/** State 타입 */
export type Todo = {
    id: number;
    text: string;
    done: boolean;
};

/** Reducer와 ActionCreator 정의 시 사용할 수 있게 export */
export type TodosStateType = Todo[];
export type TodosActionTypes = addTodoAction | toggleTodoAction;
```

<br>

📎 _modules/todos/actions.ts_

```tsx
import { TodosActionTypes } from './types';

/** return 타입을 정의하면 검사 및 IDE 자동완성을 받을 수 있음 */
export const addTodo = (text: string): TodosActionTypes => ({ type: 'todos/ADD_TODO', payload: text });
export const toggleTodo = (id: number): TodosActionTypes => ({ type: 'todos/TOGGLE_TODO', payload: id });
```

<br>

📎 _modules/todos/reducer.ts_

```tsx
import { TodosStateType, TodosActionTypes } from './types';

/** return 타입을 정의하면 검사 및 IDE 자동완성을 받을 수 있음 */
const initialState: TodosStateType = [
    {
        id: 0,
        text: 'Something 1',
        done: true
    }
]

export default function reducer(state: TodosStateType = initialState, action: TodosActionTypes): TodosStateType {
    switch(action.type) {
        case 'todos/ADD_TODO':
            return {...}
        case 'todos/TOGGLE_TODO':
            return {...}
        default:
            return state
    }
}
```

<br>

## 2. Store

📎 _modules/index.ts_

```tsx
import { combineReducers } from 'redux';
import todos from 'modules/todos/reducer';

const rootReducer = combineReducers({
    todos,
});

export default rootReducer;

/** store에 있는 값에 접근할 때 도움을 받기 위해서 */
export type RootState = ReturnType<typeof rootReducer>;
```

<br>

## 3. 커스텀 Hooks

Action별로 hooks를 작성하여 필요한 곳에 필요한 것만 불러올 수 있도록 함.

📎 _useTodos.ts_

```tsx
import { useSelector } from 'react-redux';
import { RootState } from 'modules';

export default function useTodos() {
    /** RootState를 타입으로 지정하여 rootReducer에 포함된 reducer들의 타입을 보여줌 */
    const todos = useSelector((state: RootState) => state.todos);
    return todos;
}
```

<br>

📎 _useAddTodo.ts_

```tsx
import { useCallback } from 'react';
import { useDispatch } from 'react-redux';
import { addTodo } from 'modules/todos/actions';

export default function useAddTodo() {
    const dispatch = useDispatch();
    return useCallback((text: string) => dispatch(addTodo(text)), [dispatch]);
}
```

## **4. 컴포넌트에서 커스텀 Hooks 사용**

📎 _TodoList.ts_

```tsx
import React from 'react';
import useTodos from 'hooks/todos/useTodos';

export default function TodoList () {
    /** useTodos에서 state.todos를 반환하기 때문에 */
    const todos = useTodos();

    {...생략}
}
```

📎 _TodoInput.ts_

```tsx
import React, { useState } from 'react';
import useAddTodo from 'hooks/todos/useAddTodo';

export default function TodoInput () {
    const [value, setValue] = useState('');
    const addTodo = useAddTodo();

    const onSubmit = (e: FormEvent): void => {
        e.preventDefault();
        addTodo(text);
        setValue('');
    }

    {...생략}
}
```

<br>

# 결론

1. 타입을 설정하여 검사 및 IDE의 도움을 받을 수 있음. <br>
2. Hooks를 사용하여 프리젠테이셔널 / 컨테이너로 분리하지 않고 사용할 수 있음. <br>
3. 디렉토리 구조의 고민이 필요함 > modules에서 reducer별로 `actions, types, reduce`로 분리하여 작성할지, 기존대로 `Ducks 패턴`을 사용할지. <br>
4. interface를 사용할지, type을 사용할지. <br>
5. typesafe-actions, immer, redux-saga 등과 함께 사용해 볼 예정.
