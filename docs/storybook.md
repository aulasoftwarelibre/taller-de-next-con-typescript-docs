Storybook es una herramienta sumamente valiosa en el desarrollo de aplicaciones, ya que permite la visualización y exploración de componentes de forma aislada, lo que significa que puedes ver y probar un componente específico sin tener que ejecutar toda la aplicación. Esto resulta extremadamente útil durante el proceso de desarrollo, ya que te permite concentrarte en la construcción y refinamiento de componentes individuales de manera eficiente, sin la necesidad de navegar por la aplicación en su totalidad.

Además de su capacidad para facilitar el desarrollo y la depuración de componentes, Storybook es también una herramienta clave para la creación de un "design system" o sistema de diseño. Un "design system" es un conjunto cohesivo de principios de diseño, componentes reutilizables y directrices de estilo que garantizan la coherencia visual y de interacción en toda la aplicación. Storybook es un lugar ideal para compilar y documentar estos componentes y sus variaciones, lo que permite a los diseñadores y desarrolladores colaborar de manera efectiva para definir y mantener un diseño consistente en toda la aplicación. En resumen, Storybook no solo simplifica el proceso de desarrollo y pruebas, sino que también sirve como una herramienta fundamental en la creación y gestión de un sistema de diseño sólido y coherente.

Sin embargo, no suele ser sencillo de configurar. Aunque la herramienta de instalación que proporciona suele reconocer el framework, normalmente no es capaz de reconocer los temas que utilizamos o ciertas configuraciones de typescript como las rutas.

## Instalación

En vez de dirigirnos a la web principal, vamos a indicar los pasos a seguir. Lo primero es instalar storybook dentro de nuestro proyecto.

```bash
npx storybook@latest init
```

Seleccionaremos todas las respuestas por defecto. Con esto ya tendremos una serie de componentes básicos para que podamos ver su funcionamiento:

```bash
npm run storybook
```

Se nos abrirá esta página con algunos componentes de ejemplo:

![Landing page de Storybook](images/storybook-landingpage.png)


!!! warning
    Storybook crea una serie de historias de ejemplo en el directorio `src/stories`. Recuerda no guardarlas en git.

## Creación de una historia

Una historia en Storybook es una representación visual y funcional de un componente de React en un estado particular o variación. Cada historia describe cómo se ve y se comporta un componente en un contexto específico, lo que facilita la visualización y documentación de su uso y apariencia.

Vamos a crear la historia del componente que creamos antes:

```typescript title="src/stories/components/ThemeSwitcher.stories.ts"
import ThemeSwitcher from "@/components/ThemeSwitcher";
import { Meta, StoryObj } from "@storybook/react";

const meta = {
    title: 'Components/ThemeSwitcher',
    component: ThemeSwitcher,
} satisfies Meta<typeof ThemeSwitcher>

export default meta;
type Story = StoryObj<typeof meta>;

export const Basic: Story = {}
```

!!! info
    Es probable que storybook se vuelva inestable cuando se añade un nuevo fichero.
    Si fuera así hay que reiniciarlo.

Y aparecerá una nueva sección llamada componentes que tendrá nuestra historia. Desgraciadamente no se verá como se espera. Eso es debido a que tenemos que configurar nuestro entorno para que sea el mismo qeu tiene next. Para ello debemos modificar los archivos que están en el directorio `.storybook`.

Este fichero permite usar los alias de typescript:

```typescript title=".storybook/main.ts" hl_lines="2 19-27" 
import type { StorybookConfig } from "@storybook/nextjs";
import path from "path";

const config: StorybookConfig = {
  stories: ["../src/**/*.mdx", "../src/**/*.stories.@(js|jsx|mjs|ts|tsx)"],
  addons: [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-onboarding",
    "@storybook/addon-interactions",
  ],
  framework: {
    name: "@storybook/nextjs",
    options: {},
  },
  docs: {
    autodocs: "tag",
  },
  webpackFinal: async (config) => {
    if (config.resolve) {
      config.resolve.alias = {
        ...config.resolve.alias,
        "@": path.resolve(__dirname, "../src"),
      };
    }
    return config;
  },
};
export default config;
```

Este fichero nos permite cargar los estilos:

```typescript title=".storybook/preview.tsx" linenums="1" hl_lines="2-5 9-15"
import type { Preview } from "@storybook/react";
import * as React from "react";

import "../src/app/globals.css";
import { Providers } from "../src/app/providers";


const preview: Preview = {
  decorators: [
    (Story) => 
      <Providers>
        <Story />
      </Providers>
    ,
  ],
  parameters: {
    actions: { argTypesRegex: "^on[A-Z].*" },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/,
      },
    },
  },
};

export default preview;
```

!!! warning
    El fichero `.storybook/preview.ts` debe ser renombrado a `.storybook/preview.tsx`.

Ahora si reiniciamos _Storybook_ veremos nuestra historia de forma correcta. Podemos ya incluso borrar los componentes de ejemplo.

![Storybook configurado](images/storybook-configured.png)

