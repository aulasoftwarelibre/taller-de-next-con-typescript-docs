Ya tenemos Next instalado, pero ahora necesitamos integrar el resto de componentes que vamos a necesitar para trabajar: Jest, Storybook y NextUI.

## NextUI

Esta librería de componentes se apoya en TailwindCSS con lo que su instalación nos dará muy pocos problemas y viene bien descrita en la [web oficial](https://storiesv2.nextui.org/docs/frameworks/nextjs). En cualquier caso, vamos a instalarlo con el paquete `@heroicons/react`, que nos permitirá tener un paquete de iconos en la aplicación.

```bash
npm install @nextui-org/react framer-motion @heroicons/react
```

### Dark theme

Para poder aplicar el tema oscuro o claro en nuestra aplicación, necesitamos configurar una librería llamada `next-theme`. En este caso NextUI también lo explica en la [web oficial](https://storiesv2.nextui.org/docs/customization/dark-mode).

Si hemos seguido las instrucciones correctamente, tendremos un fichero providers con el siguiente código:

```typescript title="src/app/providers.tsx"
"use client";

import {NextUIProvider} from '@nextui-org/react'
import {ThemeProvider as NextThemesProvider} from "next-themes";

export function Providers({children}: { children: React.ReactNode }) {
  return (
    <NextUIProvider>
      <NextThemesProvider attribute="class" defaultTheme="dark">
        {children}
      </NextThemesProvider>
    </NextUIProvider>
  )
}
```

!!! info
    La notación ```"use client";``` es exclusiva de algunos frameworks con soporte de `React Server Components`. Se verá su uso durante el curso.

### Nuestro primer componente: ThemeSwitcher

Para poder ver en funcionamiento el selector de tema claro/oscuro, vamos a crear un _switch_ para cambiar de un tema a otro.

Para eso crearemos el siguiente componente:

```typescript title="src/components/ThemeSwitcher/index.tsx"
"use client";

import { MoonIcon, SunIcon } from "@heroicons/react/24/solid";
import { Switch } from "@nextui-org/react";
import { useTheme } from "next-themes";
import { useEffect, useState } from "react";

export default function ThemeSwitcher() {
  const [mounted, setMounted] = useState(false); // (1)!
  const { theme, setTheme } = useTheme(); // (2)!

  useEffect(() => {
    setMounted(true);
  }, [setMounted]); // (3)!

  if (!mounted) return null;

  return (
    <Switch
      defaultSelected={theme === "light"}
      size="md"
      color="secondary"
      onClick={() => setTheme(theme === "dark" ? "light" : "dark")}
      startContent={<SunIcon />}
      endContent={<MoonIcon />}
    />
  );
}
```

1. Hook para comprobar si el componente ya ha sido montado.
2. Hook proporcionado por next-themes para conocer el tema y cambiarlo.
3. Esta función se ejecuta una sola vez, justo después de que el componente se haya montado, y cambia el estado del hook a _true_.


Y para que funcione vamos a hacer un par de cambios. Eliminamos los estilos que nos trae por defecto la plantilla de Next:

```css title="src/app/globals.css"
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Y añadimos nuestro componente a la página principal:

```typescript title="src/app/page.tsx" hl_lines="2 13"
import Image from 'next/image'
import ThemeSwitcher from '@/components/ThemeSwitcher'

export default function Home() {
  return (
    <main className="..">
      <div className="...">
        <p className="...">
          Get started by editing&nbsp;
          <code className="...">src/app/page.tsx</code>
        </p>
        <div className="...">
          <ThemeSwitcher /> // (1)!
          <a className="...">
            By{' '} 
            <Image src="/vercel.svg" alt="Vercel Logo"
               className="dark:invert" width={100} height={24} priority />
          </a>
        </div>
      </div>
```

1. Añadimos el componente antes del logo de Vercel. El IDE importará automáticamente el componente.

Ahora, ya tendremos la página principal con nuestro componente para cambiar al tema claro:

![Alt text](images/next-light-theme.png)

