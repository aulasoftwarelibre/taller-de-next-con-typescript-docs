En este capítulo, exploraremos una forma más avanzada de gestionar el estado en aplicaciones React utilizando los conceptos de reducers y contextos. Aprenderemos cómo los reducers, implementados con React Hooks, pueden ser una herramienta poderosa para administrar estados complejos y realizar actualizaciones controladas. Además, discutiremos cuándo y por qué es apropiado utilizar reducers en casos de uso específicos, lo que te permitirá tomar decisiones informadas al diseñar la lógica de tu aplicación React.

Los reducers son particularmente valiosos en situaciones donde el estado de la aplicación es complejo y necesita ser actualizado de manera controlada. Ejemplos de estos casos incluyen la gestión de formularios con múltiples campos interdependientes, la administración de un carrito de compras en línea con numerosos productos y cantidades, o el control de flujos de datos asincrónicos en aplicaciones que requieren comunicación en tiempo real.

## Contexto con reducers

En el anterior ejemplo, el mismo contexto tenía el estado y la función que lo actualiza. En este ejemplo los vamos a separar para que el estado viaje por un contexto y la función `dispatch` por otro.

### Acciones

Este archivo define las constantes `INCREMENT` y `RESET`, las dos acciones que mantiene el contador para conservar la misma API que vimos en la sección anterior (`onIncrement` y `onReset`). Además describe los tipos asociados a cada acción y expone los action creators que usaremos desde los componentes.

```ts title="src/state/counter-provider/actions.ts"
export const INCREMENT = 'counter/increment' as const
export const RESET = 'counter/reset' as const

interface CounterIncrementAction {
  type: typeof INCREMENT
  value: number
}

interface CounterResetAction {
  type: typeof RESET
}

export type CounterAction = CounterIncrementAction | CounterResetAction

export const increment = (value: number): CounterIncrementAction => ({
  type: INCREMENT,
  value,
})

export const reset = (): CounterResetAction => ({ type: RESET })
```

`INCREMENT` siempre transporta un `value: number`, el mismo paso (`step`) que reciben nuestros componentes. `RESET` no necesita payload porque basta con devolver el estado inicial. Marcamos las constantes con `as const` para que TypeScript trate esas cadenas como literales exactos, de modo que el reducer pueda discriminar las acciones sin comprobaciones adicionales.

En los componentes esto se traduce en llamadas muy legibles:

```ts
dispatch(increment(5)) // equivale a onIncrement(5)
dispatch(reset()) // equivale a onReset()
```

Así podemos mantener la interfaz `onIncrement`/`onReset` mientras la implementación interna pasa a Redux-style actions.

### Reducer

Este archivo define el reducer, que es una función que toma el estado actual y una acción, y devuelve el nuevo estado. Aquí nos encargamos de gestionar cómo cambia el contador en respuesta a las acciones `INCREMENT` y `RESET`, manteniendo la lógica bien localizada y fácil de testear. El estado, como en el caso del `useState`, es inmutable: no debemos modificarlo, solo devolver el nuevo valor. Cuidado cuando se actualicen objetos porque estos se suelen copiar por referencia.

```ts title="src/state/counter-provider/reducer.ts"
import { type CounterAction, INCREMENT, RESET } from './actions'

export type CounterState = number
export type CounterDispatcher = React.Dispatch<CounterAction>

export const initialCounterState: CounterState = 0

export const reducer: React.Reducer<CounterState, CounterAction> = (
  state,
  action
) => {
  switch (action.type) {
    case INCREMENT: {
      const step = Number(action.value)
      return state + step
    }

    case RESET:
      return initialCounterState

    default:
      return state
  }
}
```
Aquí se encuentra el hook personalizado `useCounter`, que utiliza los contextos `CounterStateContext` y `CounterDispatcherContext` para acceder al estado del contador y a la función `dispatch`. Este hook encapsula la interacción con el contexto y expone la misma interfaz que teníamos con `useState`: `onIncrement` y `onReset`. Así los componentes que ya consumían esas funciones pueden seguir funcionando sin cambios mientras la implementación interna evoluciona.


### Hook

```ts title="src/state/counter-provider/use-counter.ts"
import { useCallback, useContext } from 'react'
import { increment, reset } from './actions'
import {
  CounterDispatcherContext,
  CounterStateContext,
} from './counter-provider'

export function useCounter() {
  const state = useContext(CounterStateContext)
  const dispatch = useContext(CounterDispatcherContext)

  if (state === undefined || dispatch === undefined) {
    throw new Error('useCounter must be used within a <CounterProvider>')
  }

  const incrementCounter = useCallback(
    (step = 1) => {
      dispatch(increment(step))
    },
    [dispatch]
  )

  const resetCounter = useCallback(() => {
    dispatch(reset())
  }, [dispatch])

  return {
    counter: state,
    onIncrement: incrementCounter,
    onReset: resetCounter,
  }
}
```

### Context provider

Cuando el estado y el dispatcher viajan juntos, cualquier cambio en el valor dispara un re-render en todos los componentes que consumen el contexto, aunque solo necesiten la función para lanzar acciones. Al separarlos, los componentes que únicamente despachan acciones no se ven afectados por las variaciones del estado y mantenemos a salvo la interfaz pública (`onIncrement`, `onReset`) que ya conocíamos.

Este componente es el proveedor de contexto para el contador. Utiliza el reducer definido en `reducer.ts` para administrar el estado y proporciona los contextos `CounterStateContext` y `CounterDispatcherContext` para que los componentes descendientes puedan acceder al valor y a la función `dispatch`. Separar ambos contextos evita renderizados innecesarios y nos permite conservar la misma firma externa que en el capítulo anterior.


```ts title="src/state/counter-provider/counter-provider.tsx"
import { createContext, useReducer } from 'react'
import {
  type CounterDispatcher,
  type CounterState,
  initialCounterState,
  reducer,
} from './reducer'

export const CounterStateContext = createContext<CounterState | undefined>(
  undefined
)
export const CounterDispatcherContext = createContext<
  CounterDispatcher | undefined
>(undefined)

export function CounterProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initialCounterState)

  return (
    <CounterStateContext.Provider value={state}>
      <CounterDispatcherContext.Provider value={dispatch}>
        {children}
      </CounterDispatcherContext.Provider>
    </CounterStateContext.Provider>
  )
}
```


## Cambios

¿En qué cambia el uso de reducers con el `useState`? Básicamente la diferencia
radica en que con `useState` la lógica y la representación están unidas, mientras
que ahora el componente `StatelessCounterContainer` no sabe nada de cómo se producen los
cambios, solo indica intenciones.

!!! question

    1. Crea una nueva acción llamada DECREMENT y separa los incrementos de los decrementos.
    2. Crea un Counter que dentro use el hook `useCounter` con las tres acciones (incremento, decremento y reinicio). 

    El objetivo es que los botones llamen a su correspondiente acción de decremento e incremento.

Ejemplo:

```ts
// con useState, el componente tiene que conocer la lógica y el estado previo
<Component onClick={() => setState(NEW VALUE LOGIC)}>

// con reducers, solo indica la intencion (ACTION) y los argumentos (ARGS)
// pero no necesita saber el valor del estado porque el reducer que lo ejecuta
<Component onClick={() => dispatch({type: ACTION, value: ARGS})}>
```

En realidad, podemos decir que con reducers, los componentes lanzan eventos y
los reducers actúan como event handlers.


## Conclusiones

El uso de reducers en React, especialmente cuando se trabaja con testing, proporciona varias ventajas significativas:

1. Previsibilidad: Los reducers se basan en funciones puras que toman un estado actual y una acción para producir un nuevo estado. Esto hace que las actualizaciones de estado sean predecibles y evita efectos secundarios no deseados, lo que facilita la prueba de componentes que utilizan el estado gestionado por reducers. Por eso son ideales para pruebas unitarias.

1. Separación de responsabilidades: Al dividir la lógica de actualización del estado en reducers, se logra una clara separación de responsabilidades en la aplicación. Esto facilita la prueba a nivel de unidad de los reducers sin la necesidad de interactuar con componentes de la interfaz de usuario.

1. Reducción de Acoplamiento: Al utilizar reducers, reduces el acoplamiento entre componentes y el estado global de la aplicación. Esto facilita la prueba de componentes de manera aislada sin depender de otros componentes o lógica de la aplicación.

Sin embargo, hay un aumento en la complejidad y en el número de ficheros. Solo
se debe aplicar en componentes y situaciones complejas, con un estado muy grande.
