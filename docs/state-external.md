Vamos a imaginar un nuevo caso de uso donde queramos compartir el mismo estado
entre diferentes componentes. En este caso, lo que interesa es que nuestros
componentes sean sin estado (_stateless_). En el caso de nuestro contador
eso significa que al no poder usar hooks, tanto `counter` como `setCounter`
viven fuera de nuestro componente y tanto el valor como los handlers vienen
dados como argumentos.

!!! info
    Recuerda que esto no son casos reales, solo ejemplos de cómo aplicar
    distintos patrones.

Para ello vamos a crear un nuevo componente llamado `StatelessCounter` para
evitar perder el trabajo que teníamos hecho.

## Componente StatelessCounter

Creamos el contrato de las propiedades:

```ts title="src/components/StatelessCounter/types.ts"
export interface StatelessCounterProps {
  counter: number
  id: number
  onIncrement: (amount: number) => void
  onReset: () => void
  step: number
}
```

!!! question "Ejercicio"
    Si quieres, puedes intentar hacerlo por tu cuenta con lo que ya sabes.
    Básicamente es copiar el código del componente base (antes de usar el hook)
    y utilizar el fichero de propiedades anterior.

    La función `onIncrement` suma al valor de counter el valor indicado.
    La función `onReset` resetea el valor a cero.

    Haz los ficheros: `StatelessCounter.tsx`, `index.tsx` y `StatelessCounter.stories.ts`.
    El fichero de Storybook no necesita como argumento el `onIncrement` ni `onReset`
    porque lo inyecta automáticamente.

    De todas maneras el código resuelto se encuentra a continuación. Tampoco te
    preocupes si no lo haces exactamente igual, no hay una única solución.

Y ahora el resto de ficheros del componente:

```ts title="src/components/StatelessCounter/StatelessCounter.tsx"
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

import { StatelessCounterProps } from './types'

export default function StatelessCounter({
  counter,
  id,
  onIncrement,
  onReset,
  step,
}: StatelessCounterProps) {
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
            aria-label="Decrement counter with step"
            onClick={() => onIncrement(-step * 10)}
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
            onClick={() => onIncrement(-step)}
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
            onClick={onReset}
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
            onClick={() => onIncrement(step)}
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
            onClick={() => onIncrement(step * 10)}
            aria-label="Increment counter with step"
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

```ts title="src/components/StatelessCounter/index.ts"
export { default } from './StatelessCounter'
```

Y por último la historia:

```ts title="src/stories/components/StatelessCounter.stories.ts"
import { Meta, StoryObj } from '@storybook/react'

import StatelessCounter from '@/components/StatelessCounter'

const meta = {
  component: StatelessCounter,
  title: 'Components/StatelessCounter',
} satisfies Meta<typeof StatelessCounter>

export default meta
type Story = StoryObj<typeof meta>

export const Basic: Story = {
  args: {
    counter: 0,
    id: 1,
    step: 1,
  },
}
```

Si nos vamos al panel de Storybook, ahora veremos que el contador como tal no
funciona, obvio porque no tiene estado. Sin embargo si pulsamos algún botón
veremos que la pestaña actions refleja las acciones que se ejecutan en el handler:

![Pestaña acciones de Storybook](images/state-story-actions.png)

## Pasar el estado entre componentes

Ahora ya podemos crear el estado del contador y pasarlo como argumento. Pero no
es buena idea que el estado esté en la propia `CounterPage`, porque en la nueva
versión de Next perderíamos la propiedad de que esa página fuera un Server
React Component.

Es mejor crear un componente que contenga a nuestros contadores sin estado:

```ts title=""
import { useCallback, useState } from 'react'

import StatelessCounter from '../StatelessCounter'

export default function CounterContainerShared() {
  const [counter, setCounter] = useState(0)

  const onIncrement = useCallback(
    (amount: number) => setCounter(counter + amount),
    [counter, setCounter],
  )

  const onReset = useCallback(() => setCounter(0), [setCounter])

  return (
    <div className="flex flex-row gap-2">
      {Array.from({ length: 3 }, (_, id) => (
        <StatelessCounter
          key={id}
          id={id}
          step={id + 1} // Step incrementa como id + 1
          counter={counter}
          onIncrement={onIncrement}
          onReset={onReset}
        />
      ))}
    </div>
  )
}
```

```ts title="src/components/CounterContainerShared/index.ts"
export { default } from './CounterContainerShared'
```

Y añadimos el componente a nuestra página:


```ts title="src/app/counter/page.tsx"
import Counter from '@/components/Counter'
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
    </div>
  )
}
```


## Conclusión

Ahora podemos ver dos filas de componentes, la primera no comporten estado
y la segunda sí. Usar una u otra dependerá de nuestro caso de uso. Por lo general
no habrá muchas situaciones en las que tengamos que crear componentes sin estado.

![Página con los dos tipos de componentes](images/state-external.png)

Para aquellos casos en los que necesitemos compartirlo, veremos que probablemente
lo necesitemos en distintos puntos del documento que no estén próximos entre sí.
Si ese fuera el caso, o se usa [composición](https://legacy.reactjs.org/docs/composition-vs-inheritance.html)
si es posible, o necesitamos usar contextos, que veremos en la siguiente sección.
