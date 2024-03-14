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
    Crear un nuevo contador

    ```shell
    curl -X POST http://localhost:4000/counters --json '{"value": 0}'
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

## Accediendo a los datos del servidor desde React

Para manejar las solicitudes HTTP de manera efectiva y eficiente en React, podemos utilizar la combinación de Axios para realizar las peticiones y la librería SWR para el fetching y caching de datos. Esto nos permite crear aplicaciones más rápidas y con mejor respuesta al usuario.

### Instalación de Axios y SWR

Primero, necesitas instalar `axios` para manejar las peticiones HTTP y `swr` para el manejo de datos. Ejecuta el siguiente comando en tu terminal para instalar ambas librerías:

```shell
npm add axios swr
```

Para simplificar las peticiones HTTP y establecer una URL base para todas las llamadas, configura un cliente Axios. Crea un archivo en src/utils/fetcher.ts y define tu cliente Axios y una función fetcher que utilizará SWR para hacer las peticiones.

### Configuración del Cliente Axios

```ts title="src/utils/fetcher.ts"
import axios from 'axios'

export const client = axios.create({
  baseURL: 'http://localhost:4000',
})

export async function fetcher(url: string) {
  return client.get(url).then((response) => response.data)
}
```

### Uso de SWR con el Fetcher

Una vez que tienes tu función fetcher definida, puedes usarla con SWR en tus componentes React para acceder a los datos del servidor. SWR manejará automáticamente el caching, la revalidación, y otras optimizaciones.

```
'use client'

import useSWR from 'swr'

import { client, fetcher } from '@/utils/fetcher'

import { CounterProperties } from './types'

async function updateCounter(id: number, value: number) {
  return await client.patch(`/counters/${id}`, { value })
}

function useCounter(id: number) {
  const { data, isLoading, mutate } = useSWR(`/counters/${id}`, fetcher)

  const counter: number = data?.value ?? 0

  const onIncrement = async (value: number) => {
    const {
      data: { value: updatedValue },
    } = await updateCounter(id, counter + value)
    mutate({ ...data, value: updatedValue }, false)
  }

  const onDecrement = async (value: number) => {
    const {
      data: { value: updatedValue },
    } = await updateCounter(id, counter - value)
    mutate({ ...data, value: updatedValue }, false)
  }

  const onReset = async () => {
    const {
      data: { value: updatedValue },
    } = await updateCounter(id, 0)
    mutate({ ...data, value: updatedValue }, false)
  }

  return { counter, isLoading, onDecrement, onIncrement, onReset }
}

export function Counter({ id, step }: CounterProperties) {
  const { counter, isLoading, onDecrement, onIncrement, onReset } =
    useCounter(id)

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
          <Button isIconOnly size="md" aria-label="Decrement counter by step">
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
            onClick={() => onDecrement(1)}
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
            onClick={() => onIncrement(1)}
          >
            <ChevronRightIcon
              className="text-gray-600 dark:text-gray-400"
              height="1.3rem"
            />
            <div className="absolute right-1 bottom-0 font-bold text-[10px]">
              +{step}
            </div>
          </Button>
          <Button isIconOnly size="md" aria-label="Increment counter by step">
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