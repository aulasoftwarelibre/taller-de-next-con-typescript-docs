# Integración con servicios REST

En el desarrollo de aplicaciones modernas, la capacidad de interactuar eficientemente con servicios externos a través de APIs es fundamental. Esta sección está dedicada a explorar cómo podemos integrar nuestra aplicación Next.js con diferentes servicios API, enfocándonos principalmente en la comunicación con APIs REST.

## Protocolo HTTP

La comunicación entre el frontend de una aplicación y el backend se realiza a menudo a través de APIs REST, utilizando el protocolo HTTP. Este método de integración permite a las aplicaciones web interactuar con bases de datos y servidores sin necesidad de recargar la página, ofreciendo una experiencia de usuario fluida y dinámica. En esta sección, exploraremos los tipos de peticiones HTTP más comunes en APIs REST y cómo se utilizan para diferentes operaciones CRUD (Crear, Leer, Actualizar, Eliminar).

### Tipos de peticiones en API REST

- **GET**: Se utiliza para solicitar datos de un recurso específico. Las peticiones GET deben ser solo de lectura y no modificar los datos del servidor Una característica importante de GET es que los parámetros de la solicitud se envían en la URL. Los navegadores tienden a cachear las respuestas de las peticiones GET, lo que puede ser útil para mejorar la eficiencia y velocidad de carga de las aplicaciones web.

- **POST**: Se usa para enviar datos a un servidor para crear o actualizar un recurso. Los datos se envían en el cuerpo de la solicitud, no en la URL, lo que permite enviar información más compleja y segura. A diferencia de GET, POST no es cacheado por los navegadores, lo que lo hace más seguro para operaciones que implican cambios o la transferencia de información sensible.

- **PUT**: Similar a POST, pero se utiliza principalmente para actualizar recursos existentes. Si el recurso no existe, PUT puede crear uno nuevo.

- **DELETE**: Como su nombre indica, se utiliza para eliminar recursos especificados.

- **PATCH**: Se usa para aplicar actualizaciones parciales a un recurso. A diferencia de PUT, que reemplaza completamente el recurso, PATCH modifica solo los campos especificados.

### Diferencias clave entre GET y POST

- **Caché del navegador**: GET puede ser cacheado, lo que reduce el tiempo de carga y mejora la eficiencia de la aplicación. POST, al contener datos en el cuerpo del mensaje, no se cachea, proporcionando una capa adicional de seguridad para datos sensibles.
- **Ubicación de los datos**: En GET, los datos se pasan a través de la URL, lo que puede limitar la cantidad de información enviada y es menos seguro. En POST, los datos se envían en el cuerpo de la solicitud, permitiendo mayor cantidad de información y mayor seguridad.
- **Uso**: GET se utiliza para solicitar datos sin afectarlos, mientras que POST se utiliza para enviar datos al servidor, afectando cambios o creando recursos.

### Creando un servidor REST

Como este tutorial trata sobre frontend, no vamos a desarrollar un back. Usaremos un servidor de pruebas que
permite enviar y guardar datos desde un fichero de texto.

Para ello vamos a ejecutar lo siguiente:

```shell
npm add --save-dev json-server
npx npm-add-script -f -k db:start -v "mkdir -p node_modules/.cache && echo '{\"counters\":[{\"id\": 1, \"value\": 0}]}' > node_modules/.cache/db.json && json-server --port 4000 --watch node_modules/.cache/db.json"
```

Y ya podemos ejecutarlo en un terminal aparte:

```shell
npm run db:start
```

Podemos acceder desde el navegador, que por defecto hace peticiones GET, y visualizar los datos que contiene:

* Ver todos los contadores: [http://localhost:4000/counters](http://localhost:4000/counters)
* Ver un contador por su id: [http://localhost:4000/counters/1](http://localhost:4000/counters/1)

Para crear datos, necesitamos usar POST. No podemos usar el navegador, pero podemos usar un cliente como `curl`:

!!! example
    Vamos a crear un par de contadores

    ```shell
    curl -X POST http://localhost:4000/counters --json '{"id": "2", "value": 0}'
    curl -X POST http://localhost:4000/counters --json '{"id": "3", "value": -10}'
    ```

Para actualizar datos, tendremos que usar la url del contador. Dependiendo del servidor, deberemos usar POST, PUT o PATCH.

!!! example
    Si el ejemplo anterior nos devolvió un contador con el id `2`, podemos actualizarlo así:

    ```shell
    curl -X PATCH http://localhost:4000/counters/2 --json '{"value": 10}'
    ```    

    O podemos usar PUT:

    ```shell
    curl -X PUT http://localhost:4000/counters/2 --json '{"value": 10}'
    ```

    Podemos ver ahora que deberíamos tener tres contadores:

      ```shell
      curl -X GET http://localhost:4000/counters
      ```

## Accediendo a los datos del servidor desde React

Para manejar las solicitudes HTTP de manera efectiva y eficiente en React, podemos utilizar la combinación de Axios para realizar las peticiones y la librería SWR para el fetching y caching de datos. Esto nos permite crear aplicaciones más rápidas y con mejor respuesta al usuario.

### Instalación de SWR

Primero, instala `swr`, que nos ayudará con el fetching, el cacheo y la revalidación de datos. Ejecuta el siguiente comando en tu terminal:

```shell
npm add swr
```

Para simplificar las peticiones HTTP vamos a centralizar la URL base del servicio y una función `fetcher` que reutilizaremos desde SWR. Crea el archivo `src/utils/fetcher.ts` con el siguiente contenido.

### Configuración del fetcher

```ts title="src/utils/fetcher.ts"
export const API_BASE_URL = 'http://localhost:4000'

export async function fetcher(url: string) {
  const response = await fetch(`${API_BASE_URL}${url}`)

  if (!response.ok) {
    throw new Error('Request failed')
  }

  return response.json()
}
```


### Uso de SWR con el Fetcher

Una vez que tienes tu función fetcher definida, puedes usarla con SWR en tus componentes React para acceder a los datos del servidor. SWR manejará automáticamente el caching, la revalidación y otras optimizaciones. Para mantener el ejemplo ordenado, vamos a separar la lógica en un servicio y un hook reutilizable antes de llegar al componente.

### Servicio del contador

```ts title="src/services/counter.ts"
import { API_BASE_URL } from '@/utils/fetcher'

export async function updateCounter(id: number, value: number) {
  const response = await fetch(`${API_BASE_URL}/counters/${id}`, {
    body: JSON.stringify({ value }),
    headers: { 'Content-Type': 'application/json' },
    method: 'PATCH',
  })

  if (!response.ok) {
    throw new Error('Unable to update counter')
  }

  return response.json()
}
```

### Hook `useCounter`

```ts title="src/hooks/use-counter.ts"
'use client'

import useSWR from 'swr'

import { fetcher } from '@/utils/fetcher'
import { updateCounter } from '@/services/counter'

export function useCounter(id: number) {
  const { data, isLoading, mutate } = useSWR(`/counters/${id}`, fetcher)

  const counter: number = data?.value ?? 0

  const onIncrement = async (value: number) => {
    const updated = await updateCounter(id, counter + value)

    mutate({ ...(data ?? {}), value: updated.value }, false)
  }

  const onReset = async () => {
    const updated = await updateCounter(id, 0)

    mutate({ ...(data ?? {}), value: updated.value }, false)
  }

  return { counter, isLoading, onIncrement, onReset }
}
```

### Componente `Counter`

Vamos a actualizar nuestro componente `Counter` para que ahora use la API en
vez de almacenar el estado localmente. Si recargamos la página, lo primero que
notaremos es que ahora los contadores tienen el valor por defecto que les hemos
colocado. Si usamos los botones para modificarlos y recargamos, el nuevo valor
se reflejará.

```ts title="src/components/counter/counter.tsx"
'use client'

import {
  Button,
  ButtonGroup,
  Card,
  CardFooter,
  CardHeader,
  Skeleton,
} from '@heroui/react'
import {
  ArrowPathIcon,
  ChevronDoubleLeftIcon,
  ChevronDoubleRightIcon,
  ChevronLeftIcon,
  ChevronRightIcon,
} from '@heroicons/react/24/solid'

import { useCounter } from '@/hooks/use-counter'
import { CounterProperties } from './types'

export function Counter({ id, step }: CounterProperties) {
  const { counter, isLoading, onIncrement, onReset } = useCounter(id)

  if (isLoading) {
    return (
      <Card className="w-[240px] h-[132px]">
        <Skeleton>
          <div className="h-24 rounded-lg bg-default-300"></div>
        </Skeleton>
      </Card>
    )
  }

  return (
    <Card className="w-[240px] bg-gradient-to-br from-violet-500 to-fuchsia-500">
      <CardHeader className="flex-col items-start">
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
            aria-label="Decrement counter by step"
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
            onClick={() => onReset()}
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
            aria-label="Increment counter by step"
            onClick={() => onIncrement(step * 10)}
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

Dentro del hook reaprovechamos la misma función `onIncrement` para sumar o restar valores, pasando números negativos cuando queremos decrementar. Así evitamos duplicar lógica y mantenemos el ejemplo centrado en un único flujo de mutación: llamar al endpoint con el nuevo valor y refrescar la caché de SWR.
