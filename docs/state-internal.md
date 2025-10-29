
## Gestionar el estado dentro del componente

El hook `useState` en React permite gestionar el estado local de un componente funcional. Al utilizar `useState`, podemos declarar variables de estado y realizar actualizaciones en respuesta a eventos o acciones del usuario. El hook consta de dos partes: el valor actual del estado y una función para actualizar ese valor. Cuando llamamos a la función de actualización, React re-renderiza el componente con el nuevo estado, lo que proporciona una experiencia dinámica y reactiva en nuestras aplicaciones. En esta sección, exploraremos cómo utilizar `useState` para mantener y actualizar el estado interno de nuestros componentes de manera efectiva.

`useState` acepta dos tipos de inicialización. La forma más habitual es pasar directamente el valor inicial:

```ts
const [counter, setCounter] = useState(0)
```

!!! info "Inicialización perezosa"

    Si el cálculo del estado inicial necesita más lógica (por ejemplo, leer del `localStorage` o procesar datos), podemos pasar una función que devuelva ese valor. React solo ejecuta esa función la primera vez que monta el componente, evitando trabajo extra en cada renderizado.

    ```ts
    const [counter, setCounter] = useState(() => {
      const stored = window.localStorage.getItem('counter')
      return stored ? Number(stored) : 0
    })
    ```

    Usa esta variante más avanzada cuando el valor inicial depende de operaciones costosas o de efectos secundarios. Así mantienes el componente ligero en renderizados posteriores.

La función de actualización que devuelve `useState` también admite dos formas de uso. Podemos pasar el valor final directamente o proporcionar una función que reciba el estado anterior y devuelva el nuevo.

Usa `setCounter(nuevoValor)` cuando el resultado es independiente del estado previo. Es la opción ideal para resets como `setCounter(0)` o para establecer flags booleanos.

!!! example "Asignación directa"

    ```ts
    const [counter, setCounter] = useState(0)
    setCounter(1)
    ```

Si el nuevo valor depende del existente, pasa una función: `setCounter(previous => previous + step)`. React ejecuta esa función con el estado más reciente, incluso si varios eventos se disparan antes de renderizar de nuevo.

!!! example "Actualización derivada"

    ```ts
    const [counter, setCounter] = useState(0)
    setCounter(previous => previous + 1)
    ```

Trabajar con la función evita condiciones de carrera: `setCounter(counter + step)` reutiliza el valor que estaba disponible en el render anterior, de modo que varios clics rápidos podrían perder incrementos. La variante funcional garantiza que siempre operas sobre el estado vigente. Elegir entre ambas alternativas hará que tus componentes se comporten de forma predecible aunque actualices el estado desde distintos manejadores o efectos.

## Incrementar el contador

Los hooks **siempre** deben aparecer al comienzo de las funciones donde se declaran.
Por lo tanto, vamos a reemplazar la variable counter, por el hook `useState`, el cual nos devolverá la variable `counter`
y la función `setCounter` que usaremos para cambiar el estado/valor de dicha variable:

``` { .typescript .no-copy title="src/components/counter/counter.tsx" hl_lines="1 7"}
import { useState } from 'react'

// ...

export function Counter({ id, step }: CounterProperties) {

  const [counter, setCounter] = useState(0)

  return (
    <Card className="w-60 bg-linear-to-br from-violet-500 to-fuchsia-500">
```

Y ahora podemos usar la función setCounter para actualizar el valor de counter.
Por ejemplo modificamos el botón de incrementar el contados añadiendo a la 
propiedad _onPress_ una función que al ser ejecutada incremente el valor de counter.


``` { .typescript .no-copy title="src/components/counter/counter.tsx" hl_lines="3"}
<Button 
  isIconOnly size="md" aria-label="Increment counter"
  onPress={() => setCounter(previous => previous + step)}
>
  <ChevronRightIcon
    className="text-gray-600 dark:text-gray-400"
    height="1.3rem"
  />
  <div className="absolute right-1 bottom-0 font-bold text-[10px]">
    +{step}
  </div>
</Button>
```

!!! info
    Es importante entender que los estados son inmutable, o deben tratarse
    como si lo fueran. Si queremos modificarlos debemos usar siempre la función
    set proporcionada. De lo contrario React no sabrá que tiene que volver
    a renderizar el componente.

En el manejador de clic usamos la actualización derivada descrita en la sección anterior (`setCounter(previous => previous + step)`) para asegurarnos de que el incremento se aplica siempre sobre el estado más reciente.

Ahora al pulsado el botón de incrementar el contador veremos que ya funciona 
correctamente.

!!! question "Ejercicio"
    Completa el resto de acciones de los botones de incrementar y decrementar por
    pasos y de reiniciar el contador a cero.


### Componente CounterContainer

Ahora vamos a crear otro componente para visualizar a la vez a tres contadores

```ts title="src/components/counter/counter-container.tsx"
import { Counter } from './counter'

export function CounterContainer() {
  return (
    <div className="flex flex-row gap-2">
      <Counter id={1} step={1} />
      <Counter id={2} step={1} />
      <Counter id={3} step={1} />
    </div>
  )
}

```

## Visualizar el componente en la aplicación Next

Hasta ahora hemos usado Storybook para visualizar los componentes, vamos a crear
nuestra primera página para usarlo allí. 

Vamos a crear los siguientes archivos.

### Componente Menu

```ts title="src/components/menu/menu.tsx"
'use client'

import { Navbar, NavbarBrand, NavbarContent } from '@heroui/react'

import { ThemeSwitcher } from '@/components/theme-switcher'

export function Menu() {
  return (
    <Navbar position="static">
      <NavbarBrand>Curso de React</NavbarBrand>
      <NavbarContent justify="end">
        <ThemeSwitcher />
      </NavbarContent>
    </Navbar>
  )
}
```

### Componente CounterLayout

```ts title="src/app/counter/layout.tsx"
'use client'

import { Menu } from '@/components/menu/menu'

export default function CounterLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="container mx-auto">
      <div className="flex flex-col gap-2">
        <Menu />
        <div>{children}</div>
      </div>
    </div>
  )
}
```

### Componente CounterPage

```ts title="src/app/counter/page.tsx"
import { CounterContainer } from '@/components/counter/counter-container'

export default function CounterPage() {
  return (
    <div className="flex flex-col gap-2">
      <h2>State Counters</h2>
      <div className="flex flex-row gap-2">
        <CounterContainer />
      </div>
    </div>
  )
}
```

Y ahora debes tener disponible una nueva página `/counter` con tres contadores en
[http://localhost:3000/counter](http://localhost:3000/counter) y con un menú.

![Página counter](images/counter-page.png)

!!! info
    Los layout reciben un parámetro children que es la combinación de todos los
    layouts de los directorios superiores más la página superior. Es decir, que
    lo que se está renderizando es equivalente a:

    ```html
    <RootLayout> // Este el layout en src/app/layout.tsx
      <CounterLayout>
        <CounterPage />
      </CounterLayout>
    </RootLayout>
    ```

    Es muy útil para heredar plantillas entre subrutas. Más información en
    [la documentación de Next](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts).


### Conclusión

Si pruebas los tres componentes, verás que cada uno tiene su propio estado y que
no se comparten entre ellos. Es decir, si incrementas uno, el resto queda como estaba.
