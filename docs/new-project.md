En este capítulo, aprenderemos a crear un nuevo proyecto de React utilizando Next.js y el framework de diseño NextUI. React es una biblioteca de JavaScript ampliamente utilizada para construir interfaces de usuario interactivas, mientras que Next.js es un framework de React que facilita la creación de aplicaciones web avanzadas con rutas y renderización del lado del servidor. NextUI es una biblioteca de componentes que proporciona componentes de interfaz de usuario preestilizados y personalizables, lo que acelera el proceso de diseño y desarrollo.

## Instalación

Crear un boilerplate desde cero para hacer funcionar múltiples herramientas de Node.js, como React, NextUI, Vitest y Storybook, puede ser una tarea desafiante y propensa a errores, especialmente para los novatos. Los problemas de configuración, las dependencias en conflicto y las incompatibilidades pueden ser una fuente de frustración significativa. Esto puede desanimar a los principiantes y hacer que el proceso de inicio sea más largo y complicado de lo deseado, en lugar de permitirles centrarse directamente en el desarrollo de sus aplicaciones.

## Dependencias

### Frameworks: Next.js

Aunque es posible agregar React a un proyecto existente o crear una aplicación desde cero, la documentación oficial de React aconseja considerar el uso de un framework como Next.js para proyectos web más grandes. Esto se debe a que los frameworks proporcionan una estructura y herramientas adicionales que facilitan el desarrollo, el enrutamiento y la optimización del rendimiento de la aplicación.

En este curso, seguiremos el consejo de la documentación oficial y utilizaremos Next.js como nuestro framework principal para proyectos de React. Next.js simplifica la configuración inicial, proporciona rutas claras y ofrece una renderización del lado del servidor, lo que mejora el rendimiento y la experiencia del usuario.

### Librerías de componentes: NextUI

Este curso se enfoca en el desarrollo de aplicaciones React y no en el diseño web en sí. Con el propósito de agilizar el proceso de diseño y mantener un aspecto profesional en nuestras aplicaciones, optaremos por utilizar NextUI como nuestra biblioteca de componentes.

NextUI es una biblioteca de componentes de interfaz de usuario que se encuentra preestilizada y lista para ser utilizada. Una de sus ventajas notables es que está construida sobre Tailwind CSS, un popular framework de diseño que simplifica la creación de interfaces atractivas y responsivas. NextUI, al aprovechar Tailwind CSS, nos permite personalizar fácilmente el aspecto y la sensación de nuestros componentes sin necesidad de escribir CSS personalizado.

### Herramientas de diseño: Storybook

Storybook es una importante herramienta en el desarrollo de React debido a su capacidad para permitir el desarrollo aislado de componentes, generar documentación en vivo con historias que muestran usos variados, fomentar la colaboración entre equipos y detectar errores de manera temprana. Esta herramienta agiliza el proceso de desarrollo al centrarse en componentes individuales, mejora la calidad del código al prevenir errores antes de la integración y facilita la comunicación entre diseñadores y desarrolladores al proporcionar una representación visual de los componentes en su estado real, lo que contribuye a una experiencia de usuario más sólida.

### Entornos de testing: Vitest

Vitest es una herramienta de pruebas moderna y rápida que se utiliza en el ecosistema de React, diseñada como una alternativa más eficiente a Jest. Proporciona un conjunto de bibliotecas y utilidades específicas para la realización de pruebas en aplicaciones React. Algunas de las bibliotecas más destacadas incluyen `@testing-library/react` para interactuar con componentes React y `@testing-library/jest-dom` para realizar afirmaciones sobre su comportamiento y apariencia en un entorno de prueba, aunque es importante notar que, a pesar del nombre `jest-dom`, es completamente compatible con Vitest gracias a su API compatible con Jest. Vitest es conocido por su alta velocidad y eficiencia. Además, Vitest facilita la ejecución de pruebas unitarias y de integración, ofreciendo características como la ejecución en paralelo y la recopilación de cobertura de código, lo que lo convierte en una excelente opción para el desarrollo y la garantía de calidad de aplicaciones React.