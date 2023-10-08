En este capítulo, exploraremos una forma más avanzada de gestionar el estado en aplicaciones React utilizando los conceptos de reducers y contextos. Aprenderemos cómo los reducers, implementados con React Hooks, pueden ser una herramienta poderosa para administrar estados complejos y realizar actualizaciones controladas. Además, discutiremos cuándo y por qué es apropiado utilizar reducers en casos de uso específicos, lo que te permitirá tomar decisiones informadas al diseñar la lógica de tu aplicación React.

Los reducers son particularmente valiosos en situaciones donde el estado de la aplicación es complejo y necesita ser actualizado de manera controlada. Ejemplos de estos casos incluyen la gestión de formularios con múltiples campos interdependientes, la administración de un carrito de compras en línea con numerosos productos y cantidades, o el control de flujos de datos asincrónicos en aplicaciones que requieren comunicación en tiempo real.

## Contexto avanzado

En el anterior ejemplo, el mismo contexto tenia el estado y la función que lo
actualiza. En este ejemplo los vamos a separar.

Este archivo define las constantes `INCREMENT`, `DECREMENT`, y `RESET` que representan las acciones posibles en el contexto del contador. Luego, define los tipos de datos relacionados con el estado del contador y las acciones que pueden modificarlo. Esto proporciona una estructura sólida para la gestión del estado del contador y las actualizaciones asociadas.

```ts title="src/state/AdvancedCounterProvider/types.ts"
export const INCREMENT = Symbol()
export const DECREMENT = Symbol()
export const RESET = Symbol()

export type CounterState = number

export type CounterDispatcher = (action: CounterActions) => void

export interface CounterIncrementAction {
  type: typeof INCREMENT
  value: number
}

export interface CounterDecrementAction {
  type: typeof DECREMENT
  value: number
}

export interface CounterResetAction {
  type: typeof RESET
}

export type CounterActions =
  | CounterIncrementAction
  | CounterDecrementAction
  | CounterResetAction
```

En este archivo, se crean dos contextos: `CounterStateContext` y `CounterDispatcherContext`. El primero almacena el estado actual del contador, mientras que el segundo almacena la función del dispatcher que se utilizará para modificar el estado. Separar estos dos contextos permite un control más preciso del estado y sus actualizaciones.

```ts title="src/state/AdvancedCounterProvider/context.ts"
import React from 'react'

import { CounterDispatcher, CounterState } from './types'

export const CounterStateContext = React.createContext<
  CounterState | undefined
>(undefined)

export const CounterDispatcherContext = React.createContext<
  CounterDispatcher | undefined
>(undefined)
```

Este archivo define el reducer, que es una función que toma el estado actual y una acción, y devuelve el nuevo estado. El reducer se encarga de gestionar cómo el estado del contador cambia en respuesta a diferentes acciones, como incrementar, decrementar o reiniciar. Esto proporciona un control centralizado y predecible sobre las actualizaciones del estado. El estado, como en el caso del `useState` es inmutable.
No debemos modificarlo, solo devolver el nuevo valor. Cuidado cuando se actualicen
objectos porque estos se suelen copiar por referencia.

```ts title="src/state/AdvancedCounterProvider/reducers.ts"
import React from 'react'

import {
  CounterActions,
  CounterState,
  DECREMENT,
  INCREMENT,
  RESET,
} from './types'

export const reducer: React.Reducer<CounterState, CounterActions> = (
  state,
  action,
) => {
  switch (action.type) {
    case INCREMENT: {
      const nextState = state + action.value

      return nextState
    }
    case DECREMENT: {
      const nextState = state - action.value

      return nextState
    }
    case RESET: {
      return 0
    }
    default: {
      return state
    }
  }
}
```

Aquí se encuentra el hook personalizado `useAdvancedCounter`, que utiliza los contextos `CounterStateContext` y `CounterDispatcherContext` para acceder al estado del contador y las funciones de modificación. Este hook simplifica la interacción con el contexto del contador al proporcionar métodos como `incrementCounter`, `decrementCounter`, y `resetCounter` que pueden ser utilizados en componentes que necesiten interactuar con el estado del contador.


```ts title="src/state/AdvancedCounterProvider/hooks.ts"
import React, { MouseEventHandler, useCallback } from 'react'

import { CounterDispatcherContext, CounterStateContext } from './contexts'
import { DECREMENT, INCREMENT, RESET } from './types'

export const useAdvancedCounter = () => {
  const state = React.useContext(CounterStateContext)
  const dispatch = React.useContext(CounterDispatcherContext)

  if (state === undefined || dispatch === undefined) {
    throw new Error('useAdvancedCounter must be used within a CounterProvider')
  }

  const incrementCounter: MouseEventHandler<HTMLButtonElement> = useCallback(
    ({ currentTarget: { dataset } }) => {
      dispatch({
        type: INCREMENT,
        value: Number(dataset.step),
      })
    },
    [dispatch],
  )

  const decrementCounter: MouseEventHandler<HTMLButtonElement> = useCallback(
    ({ currentTarget: { dataset } }) => {
      dispatch({
        type: DECREMENT,
        value: Number(dataset.step),
      })
    },
    [dispatch],
  )

  const resetCounter = useCallback(() => {
    dispatch({
      type: RESET,
    })
  }, [dispatch])

  return {
    counter: state,
    decrementCounter,
    incrementCounter,
    resetCounter,
  }
}
```

Este componente es el proveedor de contexto para el contador avanzado. Utiliza el reducer definido en `reducers.ts` para administrar el estado del contador y proporciona los contextos `CounterStateContext` y `CounterDispatcherContext` para que los componentes descendientes puedan acceder al estado y las funciones de modificación.

```ts title="src/state/AdvancedCounterProvider/AdvancedCounterProvider.tsx"
import React from 'react'

import { CounterDispatcherContext, CounterStateContext } from './contexts'
import { reducer } from './reducers'

export default function AdvancedCounterProvider({
  children,
}: {
  children: React.ReactNode
}) {
  const [state, dispatch] = React.useReducer(reducer, 0)

  return (
    <CounterDispatcherContext.Provider value={dispatch}>
      <CounterStateContext.Provider value={state}>
        {children}
      </CounterStateContext.Provider>
    </CounterDispatcherContext.Provider>
  )
}
```

Este archivo exporta el `AdvancedCounterProvider` y el hook personalizado `useAdvancedCounter`, lo que facilita su importación en otros archivos de la aplicación sin tener que preocuparse por las rutas de los archivos.

```ts title="src/state/AdvancedCounterProvider/index.ts"
export { default } from './AdvancedCounterProvider'
export { useAdvancedCounter } from './hooks'
```

## Nuevo AdvancedCounter

Este nuevo counter requiere explicación adicional, pero antes vamos a crear un
nuevo componente que aproveche este nuevo contexto.

Recordad que hay que agregar el contexto:

```ts title="src/app/counter/layout.tsx"
'use client'

import Menu from '@/components/Menu'
import AdvancedCounterProvider from '@/state/AdvancedCounterProvider'
import CounterProvider from '@/state/CounterProvider'

export default function CounterLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <CounterProvider>
      <AdvancedCounterProvider>
        <div className="container mx-auto">
          <div className="flex flex-col gap-2">
            <Menu />
            <div>{children}</div>
          </div>
        </div>
      </AdvancedCounterProvider>
    </CounterProvider>
  )
}
```

Y ahora creamos el nuevo componente:

```ts title="src/components/AdvancedCounter/types.ts"
export interface AdvancedCounterProps {
  id: number
  step: number
}
```

```ts title="src/components/AdvancedCounter/AdvancedCounter.tsx"
'use client'

import {
  ArrowPathIcon,
  ChevronDoubleLeftIcon,
  ChevronDoubleRightIcon,
  ChevronLeftIcon,
  ChevronRightIcon,
} from '@heroicons/react/24/outline'
import {
  Button,
  ButtonGroup,
  Card,
  CardFooter,
  CardHeader,
} from '@nextui-org/react'

import { useAdvancedCounter } from '@/state/AdvancedCounterProvider'

import { AdvancedCounterProps } from './types'

export default function AdvancedCounter({ id, step }: AdvancedCounterProps) {
  const { counter, decrementCounter, incrementCounter, resetCounter } =
    useAdvancedCounter()

  return (
    <Card className="w-[240px] bg-gradient-to-br from-violet-500 to-fuchsia-500">
      <CardHeader className="flex-col !items-start">
        <div className="flex flex-col">
          <p className="text-tiny text-white/60 uppercase font-bold">
            Contador #{id}
          </p>
          <p className="text-white font-medium text-large">
            El contador vale {counter}.
          </p>
        </div>
      </CardHeader>
      <CardFooter className="justify-center">
        <ButtonGroup>
          <Button
            isIconOnly
            size="md"
            aria-label="Decrement counter faster"
            onClick={decrementCounter}
            data-step={step * 10}
          >
            <ChevronDoubleLeftIcon
              className="text-gray-600 dark:text-gray-400"
              height="1.3rem"
            />
            <div className="absolute right-1 bottom-0 font-bold text-[10px]">
              -{step * 10}
            </div>
          </Button>
          <Button
            isIconOnly
            size="md"
            aria-label="Decrement counter"
            onClick={decrementCounter}
            data-step={step}
          >
            <ChevronLeftIcon
              className="text-gray-600 dark:text-gray-400"
              height="1.3rem"
            />
            <div className="absolute right-1 bottom-0 font-bold text-[10px]">
              -{step}
            </div>
          </Button>
          <Button
            isIconOnly
            size="md"
            aria-label="Reset counter"
            onClick={resetCounter}
          >
            <ArrowPathIcon
              className="text-gray-600 dark:text-gray-400"
              height="1.3rem"
            />
          </Button>
          <Button
            isIconOnly
            size="md"
            aria-label="Increment counter"
            onClick={incrementCounter}
            data-step={step}
          >
            <ChevronRightIcon
              className="text-gray-600 dark:text-gray-400"
              height="1.3rem"
            />
            <div className="absolute right-1 bottom-0 font-bold text-[10px]">
              +{step}
            </div>
          </Button>
          <Button
            isIconOnly
            size="md"
            aria-label="Increment counter faster"
            onClick={incrementCounter}
            data-step={step * 10}
          >
            <ChevronDoubleRightIcon
              className="text-gray-600 dark:text-gray-400"
              height="1.3rem"
            />
            <div className="absolute right-1 bottom-0 font-bold text-[10px]">
              +{step * 10}
            </div>
          </Button>
        </ButtonGroup>
      </CardFooter>
    </Card>
  )
}
```

```ts title="src/components/AdvancedCounter/index.ts"
export { default } from './AdvancedCounter'
```

Y por último cargamos el componente en la página:

```ts title="src/app/counter/page.tsx"
import AdvancedCounter from '@/components/AdvancedCounter'
import Counter from '@/components/Counter'
import CounterContainerContext from '@/components/CounterContainerContext'
import CounterContainerShared from '@/components/CounterContainerShared'

export default function CounterPage() {
  return (
    <div className="flex flex-col gap-2">
      <h2>State Counters</h2>
      <div className="flex flex-row gap-2">
        {Array.from({ length: 3 }, (_, id) => (
          <Counter key={id} id={id} step={1} />
        ))}
      </div>
      <h2>Stateless Counters</h2>
      <CounterContainerShared />
      <h2>Context Counters</h2>
      <CounterContainerContext />
      <h2>Dispatched Counters</h2>
      <div className="flex flex-row gap-2">
        {Array.from({ length: 3 }, (_, id) => (
          <AdvancedCounter key={id} id={id} step={id + 1} />
        ))}
      </div>
    </div>
  )
}
```

## Cambios

¿En qué cambia el uso de reducers con el `useState`? Básicamente la diferencia
radica en que con `useState` la lógica y la representación están unidas, mientras
que ahora el componente `AdvancerCounter` no sabe nada de cómo se producen los
cambios, solo indica intenciones.

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
