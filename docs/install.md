
Crear un boilerplate desde cero para hacer funcionar múltiples herramientas de Node.js, como React, NextUI, Jest y Storybook, puede ser una tarea desafiante y propensa a errores, especialmente para los novatos. Los problemas de configuración, las dependencias en conflicto y las incompatibilidades pueden ser una fuente de frustración significativa. Esto puede desanimar a los principiantes y hacer que el proceso de inicio sea más largo y complicado de lo deseado, en lugar de permitirles centrarse directamente en el desarrollo de sus aplicaciones.

## Instalar Next

Aunque Next tiene una forma canónica de instalarse a través de la herramienta `npx`, el proyecto tienen una [multitud de ejemplos](https://github.com/vercel/next.js/tree/canary/examples) donde podemos descargar next ya integrado con alguna librería que nos interese.

```bash
npx create-next-app@latest taller-de-next-con-typescript-code
```

!!! info
    Las opciones que tienes que marcar en el instalador son las siguientes:

        ✔ Would you like to use TypeScript? … Yes
        ✔ Would you like to use ESLint? … Yes
        ✔ Would you like to use Tailwind CSS? … Yes
        ✔ Would you like to use `src/` directory? … Yes
        ✔ Would you like to use App Router? (recommended) … Yes
        ✔ Would you like to customize the default import alias? … No


Después de lo cual podemos abrir el nuevo directorio con nuestro IDE y ejecutar el servidor de desarrollo:

```bash
cd taller-de-next-con-typescript-code
npm run dev
```

![Imagen de inicio de una aplicación Next](images/next-landingpage.png)

## Linting

El **linting** es una práctica esencial en el desarrollo de software que consiste en analizar el código fuente en busca de errores, inconsistencias y malas prácticas. **ESLint** es una herramienta de linting ampliamente utilizada en el ecosistema de JavaScript que permite definir reglas y convenciones de estilo para garantizar que el código sea legible y consistente. Por otro lado, **Prettier** es una herramienta de formateo de código que se centra en la apariencia del código, asegurando que esté correctamente estructurado y con una disposición uniforme. La principal diferencia entre ambos radica en su enfoque: ESLint se centra en reglas y convenciones de estilo, mientras que Prettier se concentra en el formato visual. Ambas herramientas son altamente complementarias y se suelen utilizar juntas en proyectos para mantener un código limpio, legible y estéticamente agradable.

Aunque Next ya nos da un entorno con _ESLint_ configurado, es demasiado básico.
Vamos a añadir el soporte de prettier y algunos plugins más para ordenar los
imports.

Procedemos a instalar las dependencias de desarrollo:

```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-import \
eslint-plugin-prettier eslint-plugin-simple-import-sort eslint-plugin-sort \
eslint-plugin-unused-imports @typescript-eslint/eslint-plugin \
@typescript-eslint/parser
```

Y a continuación vamos a editar el archivo `.eslintrc.json` con este contenido:

```json title=".eslintrc.json"
{
  "env": {
    "node": true,
    "jest": true
  },
  "extends": [
    "next",
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint",
    "import",
    "simple-import-sort",
    "sort",
    "unused-imports"
  ],
  "root": true,
  "rules": {
    "@typescript-eslint/no-shadow": "error",
    "@typescript-eslint/no-unused-vars": [
      "error",
      { "ignoreRestSiblings": true }
    ],
    "camelcase": "error",
    "default-case-last": "error",
    "default-param-last": "error",
    "dot-notation": "error",
    "eqeqeq": "error",
    "import/first": "error",
    "import/newline-after-import": "error",
    "import/no-duplicates": "error",
    "no-shadow": "off",
    "simple-import-sort/exports": "error",
    "simple-import-sort/imports": "error",
    "sort/destructuring-properties": "error",
    "sort/object-properties": "error",
    "sort/type-properties": "error",
    "unused-imports/no-unused-imports": "error"
  }
}
```

!!! note
    No vamos a explicar lo que significa cada opción, se recomienda ir a la
    documentacion de ESLint y de cada plugin para ver el significado de las
    reglas utilizadas. Estas reglas son solo un ejemplo y cada persona
    o grupo de desarrollo puede tener las suyas propias. Así que no deben tomarse
    como algo dogmático.

Ya tenemos configurado ESLint y ahora vamos a configurar prettier añadiendo este
fichero a la raíz del proyecto:

```json title=".prettierrc.json"
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all"
}
```

Ahora si ejecutamos el linter en la línea de comandos:

```bash
npm run lint
```

Veremos que encuentra algunos errores de lintado. Podemos corregirlos automáticamente ejecutando lo siguiente:

```bash
npm run lint -- --fix
```

!!! info
    Obviamente no es muy cómodo. Podemos hacerlo de forma automática al guardar
    el fichero en vscode creando este fichero:

    ```json title=".vscode/settings.json"
    {
    "editor.codeActionsOnSave": {
        "source.fixAll": true,
        "source.organizeImports": true
    },
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true
    }
    ```

    Esta configuración de vscode es exclusiva para este proyecto, se puede
    configurar para que lo haga en todos los que creemos si configuramos
    estas opciones en el entorno de usuario. 
    Se recomienda ver el [Taller de VSCode](https://aulasoftwarelibre.github.io/taller-vscode/).

## Git hooks

Hemos visto ya como formatear automáticamente nuestros ficheros desde vscode,
pero eso tiene un problema, si algún fichero se edita de otra manera o hay
algún problema con el editor, puede que se nos escape algún fichero con el 
formato incorrecto.

Necesitamos estar completamente seguro que no vamos a commitear a nuestro
repositorio de git ningún fichero que viole nuestras reglas de lintado.

Para eso _git_ proporciona una serie de _hooks_, que son scripts que se ejecutan
en determinadas acciones (_pull_, _commit_, _pre-commit_, ...). Podemos
aprovecharlos para comprobar que los ficheros son correctos.

!!! info
    Saber Git es esencial para los programadores ya que facilita el seguimiento de cambios, la colaboración en equipo y la gestión eficiente del código fuente, lo que garantiza un desarrollo de software ordenado y eficaz. Se recomienda seguir el [Taller de Git](https://aulasoftwarelibre.github.io/taller-de-git/).

### Configurar _lint-staged_

Con este paquete podremos comprobar el lintado de nuestros archivos antes de 
hacer commit. Instalamos los paquetes necesarios. Husky se encarga de configurar
los hooks y lint-staged es el script que se lanza para comprobar que los
ficheros modificados son correctos:

```bash
npm install --save-dev husky lint-staged
```

Y creamos el archivo de configuración:

```javascript title=".lintstagedrc.js"
const path = require('path')
 
const buildEslintCommand = (filenames) =>
  `next lint --fix --file ${filenames
    .map((f) => path.relative(process.cwd(), f))
    .join(' --file ')}`
 
module.exports = {
  '*.{js,jsx,ts,tsx}': [buildEslintCommand],
}
```

!!! note
    Hemos activado el fix automático, si lo queremos quitar hay que borrar el
    flag `--fix` al comando next.
    Hay que tener en cuenta que esto modifica los ficheros que se van a comitear,
    y por tanto habrá diferencia entre lo que hemos querido guardar y lo guardado.
    No es mayor problema si sabemos lo que estamos haciendo, porque por lo general
    el lintado no ocasiona bugs.


Y ahora configuramos husky para que ejecute lint-staged antes de cada commit:

```
npm pkg set scripts.prepare="husky install"
npm run prepare
npx husky add .husky/pre-commit "npx lint-staged"
```

### Configurar _conventional commits_

[Conventional Commits](https://www.conventionalcommits.org) es una convención de nomenclatura para los mensajes de confirmación (commits) en proyectos de desarrollo de software. Su objetivo es estandarizar la estructura y el contenido de los mensajes de confirmación, lo que facilita la comprensión y el seguimiento de los cambios en un repositorio. Los mensajes de confirmación siguen un formato específico, que incluye un encabezado conciso y opcionalmente un cuerpo y pies de página, todos diseñados para proporcionar información clara sobre qué cambios se realizaron en el código.

La estructura de un mensaje de commit es la siguiente:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Vamos a ayudarnos de husky para configurarlo. Instalamos como siempre las
dependencias:

```bash
npm install --save-dev @commitlint/{config-conventional,cli}
echo "module.exports = {extends: ['@commitlint/config-conventional']};" > .commitlintrc.js
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

Y una vez hecho todo eso intentamos ya, por fin, guardar los cambios y veremos
como actúan los hooks de git:

```bash
git add .
git commit -m "chore: configure linting, lint-staged and conventional commits"
```

!!! note
    Si queremos ver como falla si el mensaje no sigue conventional commits
    podemos ejecutar esta prueba:

    ```bash
    git commit --allow-empty -m "invalid message"
    ```

## Jest

Para completar nuestro entorno, vamos a configurar jest. Jest es un marco de pruebas (testing framework) ampliamente utilizado en el desarrollo de aplicaciones JavaScript y React. Sirve para automatizar y simplificar la realización de pruebas unitarias, de integración y funcionales en el código de tu aplicación. Jest facilita la creación de pruebas al proporcionar un conjunto de utilidades y funciones que permiten escribir y ejecutar pruebas de manera eficiente, asegurando que tu código funcione como se espera y reduciendo la probabilidad de errores en producción.

Instalamos las dependencias:

```bash
npm install --save-dev jest jest-environment-jsdom @testing-library/react \
@testing-library/jest-dom
```

Y creamos el archivo de configuración:

```js title="jest.config.mjs"
import nextJest from 'next/jest.js'
 
const createJestConfig = nextJest({
  // Provide the path to your Next.js app to load next.config.js and .env files in your test environment
  dir: './',
})
 
// Add any custom config to be passed to Jest
/** @type {import('jest').Config} */
const config = {
  // Add more setup options before each test is run
  // setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
 
  testEnvironment: 'jest-environment-jsdom',
}
 
// createJestConfig is exported this way to ensure that next/jest can load the Next.js config which is async
export default createJestConfig(config)
```

Y con el siguiente comando podremos añadir un comando test a npm:

```bash
npm pkg set scripts.test="jest"
```

!!! question "Ejercicio"
    Se deja como ejercicio, guardar los cambios en git con la configuración de
    jest y siguiendo conventional commits para el mensaje.

!!! tip
    A partir de aquí ya tendríamos una plantilla básica para trabajar con Next.
    Puede usarse ya como [plantillas de repositorios para Github](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-template-repository) o esperar a la siguiente sección donde se
    añadirá una librería de componentes y la herramienta _Storybook_, aunque eso
    provoca que la plantilla sea menos versátil.
