### A - ¿Qué Rayos son los Hooks? (Hooks Pattern Introduction)

1.  **Definición:**

    - Piensa en los Hooks como "ganchos" que te permiten "enganchar" características de React como el estado (`state`) y los métodos del ciclo de vida (como cuando un componente aparece o desaparece) ¡directamente en tus componentes funcionales!
    - Antes de los Hooks (introducidos en React 16.8), si querías que un componente tuviera su propio estado o reaccionara a su ciclo de vida, _tenías_ que usar componentes de clase (Class Components). Los Hooks te permiten hacer todo eso, pero con funciones, que suelen ser más sencillas.
    - Aunque no son un "patrón de diseño" en el sentido estricto, cambian mucho la forma en que diseñas tus componentes en React.

2.  **Notas o advertencias:**
    - ¡Ojo! Esta chuleta se basa en la documentación proporcionada. React evoluciona, pero nos centraremos en estos conceptos clave tal como se explican aquí.
    - Los Hooks vinieron a resolver algunos dolores de cabeza que daban los componentes de clase. Veamos cuáles eran.

### B - Los Componentes de Clase: La Vida Antes de los Hooks (Class Components)

1.  **Definición:**

    - Eran la forma _obligatoria_ de crear componentes con estado interno o que necesitaban ejecutar código en momentos específicos de su vida (montaje, actualización, desmontaje).
    - Usaban la sintaxis de clases de JavaScript (ES2015).

2.  **Ejemplo (Estructura Típica):**

    ```javascript
    class MyComponent extends React.Component {
      // 1. Constructor: Para inicializar estado y "bindear" métodos
      constructor() {
        super(); // Llama al constructor de React.Component
        this.state = {
          /* aquí va tu estado inicial */
        };

        // Esto era necesario para que 'this' funcionara bien dentro de tus métodos
        this.customMethodOne = this.customMethodOne.bind(this);
        this.customMethodTwo = this.customMethodTwo.bind(this);
      }

      // 2. Métodos de Ciclo de Vida: Código en momentos clave
      componentDidMount() {
        /* Código al montar el componente */
      }
      componentWillUnmount() {
        /* Código antes de desmontar */
      }

      // 3. Métodos Personalizados: Tu lógica
      customMethodOne() {
        /* ... */
      }
      customMethodTwo() {
        /* ... */
      }

      // 4. Render: Qué se va a mostrar en pantalla
      render() {
        return {
          /* JSX aquí */
        };
      }
    }
    ```

    **Explicación del ejemplo:**

    - El `constructor` preparaba el componente, definiendo el `state` inicial y asegurando que los métodos personalizados (`customMethodOne`, etc.) supieran a qué `this` referirse.
    - Los métodos como `componentDidMount` se ejecutaban automáticamente en fases específicas (como cuando el componente aparece por primera vez en pantalla).
    - `render` era (y sigue siendo) el método que devuelve qué pintar en la interfaz.

3.  **Notas o advertencias:**
    - Aunque todavía puedes usar clases, los Hooks suelen ser la opción preferida hoy en día por las razones que veremos a continuación.

### C - Problemillas Comunes con las Clases: ¿Por Qué Cambiar?

#### 1. **Entender las Clases de ES2015 podía ser un Rollo**

- **El Lío:** Si tenías un componente funcional simple y de repente necesitabas añadirle estado, ¡zas!, tenías que reescribirlo _entero_ como una clase.
- **El Dolor:** Esto implicaba entender conceptos como `this`, `constructor`, `super()`, y por qué diablos había que hacer `.bind(this)`. Para alguien empezando, ¡era mucho que asimilar!

- **Ejemplo (Antes y Después):**

  - _Antes (Funcional Simple):_
    ```javascript
    function Button() {
      return <div className="btn">disabled</div>;
    }
    ```
  - _Después (Clase con Estado para Habilitar/Deshabilitar):_

    ```javascript
    export default class Button extends React.Component {
      constructor() {
        super();
        this.state = { enabled: false }; // Añadimos estado
      }

      render() {
        const { enabled } = this.state;
        const btnText = enabled ? "enabled" : "disabled";

        return (
          <div
            className={`btn enabled-${enabled}`}
            // Actualizamos el estado al hacer clic
            onClick={() => this.setState({ enabled: !enabled })}
          >
            {btnText}
          </div>
        );
      }
    }
    ```

    _(Puedes ver este código en acción [aquí](https://codesandbox.io/embed/throbbing-currying-2lp9w))_

  **Explicación del ejemplo:**
  Para que el botón cambiara de "disabled" a "enabled" al hacer clic, tuvimos que convertir la función simple en una clase completa, añadir un `constructor`, definir `this.state`, y usar `this.setState` para cambiarlo. ¡Más código y más conceptos!

#### 2. **Reestructurar para Compartir Lógica era Complicado (Wrapper Hell)**

- **El Lío:** Para reutilizar lógica entre componentes (como conectar a un estado global o añadir props), se usaban patrones como [Higher Order Components (HOCs)](https://www.patterns.dev/posts/hoc-pattern) o [Render Props](https://www.patterns.dev/posts/render-props-pattern).
- **El Dolor:** Estos patrones te obligaban a "envolver" tus componentes con otros componentes. Si necesitabas varias de estas lógicas compartidas, terminabas con un montón de capas anidadas en tu árbol de componentes (el famoso "Wrapper Hell"). ¡Era difícil seguir el flujo de datos!

- **Ejemplo (Visual del Wrapper Hell):**
  ```html
  <WrapperOne>
    <WrapperTwo>
      <WrapperThree>
        <WrapperFour>
          <WrapperFive>
            <Component>
              <h1>¡Al fin llegué al componente!</h1>
            </Component>
          </WrapperFive>
        </WrapperFour>
      </WrapperThree>
    </WrapperTwo>
  </WrapperOne>
  ```
  **Explicación del ejemplo:** Imagina depurar algo aquí... ¡uf!

#### 3. **Componentes Complejos y Lógica Enredada**

- **El Lío:** A medida que un componente de clase crecía, la lógica relacionada con diferentes funcionalidades se mezclaba dentro de los mismos métodos de ciclo de vida.
- **El Dolor:** Era difícil ver qué partes del código correspondían a qué funcionalidad. Por ejemplo, en `componentDidMount` podías tener código para cargar datos, suscribirte a eventos del navegador y configurar temporizadores, todo junto. ¡Hacía difícil depurar y mantener!

- **Ejemplo (Lógica Mezclada):**

  - Imagina un componente que muestra un contador y el ancho de la ventana.
  - _Código de Clase (Simplificado):_

    ```javascript
     // ... (constructor con state para count y width)

     componentDidMount() {
       // Lógica para el ancho de ventana
       this.handleResize();
       window.addEventListener("resize", this.handleResize);
       // ¡Aquí podría haber más lógica para el contador si fuera necesario!
     }

     componentWillUnmount() {
       // Limpieza para el ancho de ventana
       window.removeEventListener("resize", this.handleResize);
       // ¡Aquí podría haber más limpieza para el contador!
     }

     // Métodos para el contador
     increment = () => { /* ... */ };
     decrement = () => { /* ... */ };

     // Método para el ancho de ventana
     handleResize = () => { /* ... */ };

     render() { /* Muestra Count y Width */ }
    ```

    _(Puedes ver el código completo [aquí](https://codesandbox.io/embed/bold-brown-bzhpw))_

  - _Visualización del Enredo:_
    ![Flow chart de clase con lógica mezclada](https://res.cloudinary.com/ddxwdqwkr/image/upload/f_auto/v1641930124/patterns.dev/clasvshoks1.001.png)

  **Explicación del ejemplo:** Fíjate cómo la lógica del contador (`increment`, `decrement`) y la del ancho de la ventana (`handleResize`, `addEventListener`) están en el mismo componente. Los métodos del ciclo de vida (`componentDidMount`, `componentWillUnmount`) tienen que manejar ambas cosas.

### D - ¡Llegan los Hooks al Rescate!

1.  **Definición:**
    - Son funciones especiales (siempre empiezan con `use`, como `useState`) que te permiten usar estado y otras características de React en componentes funcionales.
    - Vienen a solucionar los problemas de las clases:
      - ✅ Puedes añadir estado a componentes funcionales sin reescribirlos.
      - ✅ Puedes manejar efectos secundarios (como en los ciclos de vida) de forma más organizada.
      - ✅ Puedes crear tu propia lógica reutilizable con estado (Custom Hooks).

### E - El Hook de Estado: `useState`

1.  **Definición:**

    - Es el Hook fundamental para añadir estado a un componente funcional.
    - Cuando lo llamas, le pasas el valor inicial del estado.
    - Te devuelve un array con dos cosas:
      1.  El valor _actual_ del estado.
      2.  Una _función_ para actualizar ese estado.

2.  **Ejemplo:**

    - _Sintaxis:_
      ```javascript
      //      [valorActual, funcionParaActualizar] = React.useState(valorInicial);
      const [nombre, setNombre] = React.useState("Invitado");
      ```
    - _Refactorizando el Input (de Clase a Función con Hook):_

      - _Clase (Recordatorio):_
        ```javascript
        class Input extends React.Component {
          constructor() {
            /* ... this.state = { input: "" }; ... */
          }
          handleInput(e) {
            this.setState({ input: e.target.value });
          }
          render() {
            /* <input onChange={this.handleInput} value={this.state.input} /> */
          }
        }
        ```
      - _Función con `useState`:_

        ```javascript
        import React, { useState } from "react"; // Importamos useState

        function Input() {
          // 1. Llamamos a useState con "" como valor inicial
          //    Nos devuelve 'input' (valor actual) y 'setInput' (función para actualizar)
          const [input, setInput] = useState("");

          // 2. Usamos 'input' para el valor del input
          // 3. Usamos 'setInput' en el onChange para actualizar el estado
          return (
            <input
              onChange={(e) => setInput(e.target.value)} // ¡Más directo!
              value={input}
              placeholder="Escribe algo..."
            />
          );
        }
        export default Input;
        ```

        _(Puedes ver este código en acción [aquí](https://codesandbox.io/embed/nervous-hoover-oicu6))_

    **Explicación del ejemplo:**

    - `useState("")` inicializa nuestro estado `input` como una cadena vacía.
    - `input` (la variable) siempre tendrá el valor actual del campo de texto.
    - `setInput(nuevoValor)` es la función que llamamos para cambiar el estado, lo que hará que el componente se vuelva a renderizar con el nuevo valor. ¡Adiós `this.state` y `this.setState`!

### F - El Hook de Efecto: `useEffect`

1.  **Definición:**

    - Te permite realizar "efectos secundarios" en componentes funcionales. ¿Qué es un efecto secundario? Cualquier cosa que interactúe con el "mundo exterior" fuera de tu componente:
      - Llamadas a APIs (fetch de datos).
      - Suscripciones (a eventos del navegador, WebSockets).
      - Manipulación directa del DOM (aunque menos común en React).
      - Temporizadores (`setTimeout`, `setInterval`).
    - Combina el propósito de `componentDidMount`, `componentDidUpdate`, y `componentWillUnmount` de las clases.

2.  **¿Cómo funciona?**

    - Le pasas una función (el "efecto"). Esta función se ejecutará _después_ de que React haya actualizado el DOM.
    - Opcionalmente, le pasas un segundo argumento: un array de dependencias.
      - `useEffect(() => { /* efecto */ }, [])`: El array vacío `[]` significa que el efecto se ejecuta **solo una vez**, después del primer renderizado (similar a `componentDidMount`).
      - `useEffect(() => { /* efecto */ }, [valor1, valor2])`: El efecto se ejecuta **después del primer renderizado Y cada vez que `valor1` o `valor2` cambien**. (Similar a `componentDidUpdate` pero más controlado).
      - `useEffect(() => { /* efecto */ })`: Sin array de dependencias (¡cuidado!), el efecto se ejecuta **después de cada renderizado**.
    - **Limpieza:** Si tu efecto necesita "limpieza" (ej: cancelar una suscripción o un temporizador), puedes devolver una función dentro de tu efecto. React ejecutará esta función de limpieza antes de desmontar el componente o antes de volver a ejecutar el efecto si sus dependencias cambiaron (similar a `componentWillUnmount`).
      ```javascript
      useEffect(() => {
        // Efecto (ej: suscribirse)
        const subscription = subscribeToSomething();
        // Función de limpieza
        return () => {
          // Limpiar (ej: desuscribirse)
          unsubscribe(subscription);
        };
      }, []); // Dependencias
      ```

3.  **Ejemplo (Loguear el valor del Input):**

    - Queremos mostrar en la consola lo que el usuario escribe en el input _cada vez que cambia_.

    ```javascript
    import React, { useState, useEffect } from "react"; // Importamos useEffect

    export default function Input() {
      const [input, setInput] = useState("");

      // Efecto que se ejecuta cuando 'input' cambia
      useEffect(() => {
        // El código dentro se ejecuta después de renderizar, si 'input' cambió
        console.log(`El usuario escribió: ${input}`);
        // No necesitamos limpieza aquí
      }, [input]); // <-- Array de dependencias: ¡Solo re-ejecuta si 'input' cambia!

      return (
        <input
          onChange={(e) => setInput(e.target.value)}
          value={input}
          placeholder="Escribe algo..."
        />
      );
    }
    ```

    _(Puedes ver este código en acción [aquí](https://codesandbox.io/embed/blissful-ramanujan-p237n))_

    **Explicación del ejemplo:**

    - El `useEffect` se ejecutará la primera vez que el componente se monte y, después, cada vez que el valor de `input` (que está en el array de dependencias `[input]`) cambie.
    - Esto nos permite reaccionar a los cambios de estado (o props) de una manera organizada.

### G - ¡Crea tus Propios Hooks! (Custom Hooks)

1.  **Definición:**

    - Son simplemente funciones JavaScript normales cuyo nombre empieza por `use`.
    - La magia está en que **pueden llamar a otros Hooks** (como `useState` o `useEffect`) dentro de ellas.
    - Te permiten extraer lógica de componente (que incluye estado o efectos) para que puedas **reutilizarla** fácilmente en diferentes componentes. ¡Son súper poderosos para mantener tu código limpio y DRY (Don't Repeat Yourself)!

2.  **Regla de Oro:** Los nombres de los Custom Hooks **deben** empezar con `use`. Esto le permite a React y a herramientas como linters aplicar las [reglas de los Hooks](https://reactjs.org/docs/hooks-rules.html) (ej: no llamar Hooks dentro de condicionales o bucles).

3.  **Ejemplo (Hook `useKeyPress` para detectar si una tecla está presionada):**

    - _El Custom Hook (`useKeyPress.js`):_

      ```javascript
      import React, { useState, useEffect } from "react";

      function useKeyPress(targetKey) {
        // 1. Estado para saber si la tecla está presionada
        const [keyPressed, setKeyPressed] = useState(false);

        // 2. Funciones para manejar los eventos keydown y keyup
        function handleDown({ key }) {
          // Usamos destructuring para obtener 'key' del evento
          if (key === targetKey) {
            setKeyPressed(true);
          }
        }

        function handleUp({ key }) {
          if (key === targetKey) {
            setKeyPressed(false);
          }
        }

        // 3. Efecto para añadir y quitar los event listeners
        useEffect(() => {
          // Añadir listeners cuando el hook se monta (o targetKey cambia)
          window.addEventListener("keydown", handleDown);
          window.addEventListener("keyup", handleUp);

          // Función de limpieza: quitar listeners cuando el hook se desmonta
          return () => {
            window.removeEventListener("keydown", handleDown);
            window.removeEventListener("keyup", handleUp);
          };
        }, []); // <- Array vacío: los listeners se añaden una vez

        // 4. Devolver el estado actual
        return keyPressed;
      }

      export default useKeyPress; // Exportamos el hook
      ```

    - _Usando el Custom Hook en el componente `Input`:_

      ```javascript
      import React, { useState, useEffect } from "react";
      import useKeyPress from "./useKeyPress"; // Importamos nuestro hook

      export default function Input() {
        const [input, setInput] = useState("");
        // Usamos nuestro hook para diferentes teclas
        const pressQ = useKeyPress("q");
        const pressW = useKeyPress("w");
        const pressL = useKeyPress("l");

        // Efectos para reaccionar cuando las teclas son presionadas
        useEffect(() => {
          if (pressQ) console.log("¡El usuario presionó Q!");
        }, [pressQ]); // Dependencia: pressQ

        useEffect(() => {
          if (pressW) console.log("¡El usuario presionó W!");
        }, [pressW]); // Dependencia: pressW

        useEffect(() => {
          if (pressL) console.log("¡El usuario presionó L!");
        }, [pressL]); // Dependencia: pressL

        return (
          <input
            onChange={(e) => setInput(e.target.value)}
            value={input}
            placeholder="Escribe algo... (prueba Q, W, L)"
          />
        );
      }
      ```

      _(Puedes ver este código en acción [aquí](https://codesandbox.io/embed/billowing-pine-xplez))_

    **Explicación del ejemplo:**

    - Toda la lógica para detectar si una tecla está presionada (manejar estado, añadir/quitar listeners) está encapsulada en `useKeyPress`.
    - El componente `Input` ahora solo _usa_ el hook (`useKeyPress('q')`) y obtiene un simple booleano (`pressQ`) que le dice si la tecla 'q' está presionada o no. ¡Mucho más limpio y reutilizable!

4.  **Notas o advertencias:**
    - ¡No reinventes la rueda! Hay muchísimos Custom Hooks geniales creados por la comunidad. Antes de crear uno, busca si ya existe.
    - Algunas colecciones populares:
      - [React Use](https://github.com/streamich/react-use)
      - [useHooks](https://usehooks.com/)
      - [Collection of React Hooks](https://nikgraf.github.io/react-hooks/)

### H - Ejemplo Completo: Refactorizando el Contador y Ancho de Ventana con Hooks

1.  **El Objetivo:** Tomar el ejemplo anterior (que mezclaba lógica de contador y ancho de ventana en una clase) y reescribirlo usando Hooks para separar las responsabilidades.

2.  **La Solución con Hooks:**

    - _Crear un Custom Hook para el contador (`useCounter`):_
      ```javascript
      function useCounter() {
        const [count, setCount] = useState(0);
        const increment = () => setCount(count + 1);
        const decrement = () => setCount(count - 1);
        // Devuelve el estado y las funciones para manipularlo
        return { count, increment, decrement };
      }
      ```
    - _Crear un Custom Hook para el ancho de la ventana (`useWindowWidth`):_

      ```javascript
      function useWindowWidth() {
        const [width, setWidth] = useState(window.innerWidth);

        useEffect(() => {
          const handleResize = () => setWidth(window.innerWidth);
          window.addEventListener("resize", handleResize);
          // ¡Importante la limpieza!
          return () => window.removeEventListener("resize", handleResize);
        }, []); // Se ejecuta solo una vez para añadir el listener

        return width; // Devuelve solo el valor del ancho
      }
      ```

    - _Usar los Hooks en el componente `App`:_

      ```javascript
      import React, { useState, useEffect } from "react"; // Necesarios para los hooks internos
      import "./styles.css";
      import { Count } from "./Count"; // Componente para mostrar contador
      import { Width } from "./Width"; // Componente para mostrar ancho

      // ... (definiciones de useCounter y useWindowWidth aquí arriba) ...

      export default function App() {
        // Usamos nuestros custom hooks
        const counter = useCounter(); // Obtiene { count, increment, decrement }
        const width = useWindowWidth(); // Obtiene el valor del ancho

        return (
          <div className="App">
            {/* Pasamos los datos y funciones del hook counter */}
            <Count
              count={counter.count}
              increment={counter.increment}
              decrement={counter.decrement}
            />
            <div id="divider" />
            {/* Pasamos el dato del hook width */}
            <Width width={width} />
          </div>
        );
      }
      ```

      _(Puedes ver este código en acción [aquí](https://codesandbox.io/embed/eloquent-bhabha-2w0ll))_

3.  **Visualización del Cambio:**

    - _Antes (Clase):_ Lógica mezclada.
      ![Flow chart de clase con lógica mezclada](https://res.cloudinary.com/ddxwdqwkr/image/upload/f_auto/v1641930124/patterns.dev/clasvshoks1.001.png)
    - _Después (Hooks):_ Lógica separada y reutilizable.
      ![Flow chart con Hooks y lógica separada](https://res.cloudinary.com/ddxwdqwkr/image/upload/f_auto/v1641930050/patterns.dev/classicalvshooks2.001.png)

4.  **Explicación del ejemplo:**
    - ¡Mira qué limpio queda el componente `App`! Toda la lógica compleja está ahora dentro de los Custom Hooks (`useCounter`, `useWindowWidth`).
    - `App` solo llama a los hooks y pasa los datos/funciones resultantes a los componentes hijos.
    - La lógica está **separada por funcionalidad**, es **reutilizable** y mucho más **fácil de entender y probar**.

### I - Otros Hooks Útiles (Breve Vistazo)

#### 1. **`useContext`**

- **Definición:** Te permite "suscribirte" a un [Contexto de React](https://reactjs.org/docs/context.html) directamente desde un componente funcional, sin necesidad de usar `Context.Consumer` ni pasar props manualmente a través de muchos niveles (evita el "prop drilling").
- **Uso:** Le pasas el objeto de contexto (creado con `React.createContext`) y te devuelve el valor actual de ese contexto.
- **Nota:** El componente que usa `useContext` se volverá a renderizar automáticamente si el valor del contexto cambia.

#### 2. **`useReducer`**

- **Definición:** Es una alternativa a `useState` para manejar lógica de estado más compleja. Es especialmente útil cuando el próximo estado depende del anterior o cuando tienes múltiples sub-valores en tu estado.
- **Uso:** Funciona de manera similar a los _reducers_ en Redux. Le pasas una función _reducer_ `(state, action) => newState` y un estado inicial. Te devuelve el estado actual y una función `dispatch` para enviar acciones que actualicen el estado.
- **Ventaja:** Puede ayudar a organizar mejor la lógica de actualización del estado y optimizar el rendimiento en algunos casos.

### J - Ventajas Claras de Usar Hooks (Pros)

#### 1. **Menos Código y Más Claro**

- **La Idea:** Los Hooks te permiten agrupar la lógica por _funcionalidad_ (ej: todo lo relacionado con el fetch de datos en un `useEffect`) en lugar de por _método de ciclo de vida_ (donde se mezclaba todo). Esto hace el código más corto y fácil de seguir.
- **Ejemplo (Comparación Rápida):**
  - _Clase con Estado:_
    ```javascript
    class TweetSearchResults extends React.Component {
      constructor(props) {
        super(props);
        this.state = { filterText: "", inThisLocation: false };
        this.handleFilterTextChange = this.handleFilterTextChange.bind(this);
        // ... más binding ...
      }
      handleFilterTextChange(filterText) {
        this.setState({ filterText });
      }
      // ... más handlers ...
      render() {
        /* ... usa this.state y this.handle... */
      }
    }
    ```
  - _Función con Hooks:_
    `javascript
    const TweetSearchResults = ({ tweets }) => {
      const [filterText, setFilterText] = useState("");
      const [inThisLocation, setInThisLocation] = useState(false);
      // ¡No hay constructor, no hay binding!
      return (
        <div>
          <SearchBar
            filterText={filterText} /* ... */
            setFilterText={setFilterText} /* Pasa la función directamente */
          />
          <TweetList tweets={tweets} filterText={filterText} /* ... */ />
        </div>
      );
    };
    `
    **Explicación:** ¡Mira cuánto código repetitivo (constructor, binding) desaparece con los Hooks!

#### 2. **Simplifica Componentes Complejos**

- **La Idea:** Las clases de JS pueden ser liosas (recuerda `this`). Los Hooks te permiten usar las características de React (estado, ciclo de vida, contexto) sin escribir clases, favoreciendo un estilo de programación funcional más sencillo.

#### 3. **Reutilización de Lógica con Estado ¡Fácil!**

- **La Idea:** Antes, reutilizar lógica _con estado_ era complicado (HOCs, Render Props). Con los Custom Hooks, puedes extraer esa lógica (estado + efectos) en una función reutilizable. ¡Es como tener bloques de construcción súper poderosos!

#### 4. **Compartir Lógica No Visual**

- **La Idea:** Los Hooks son perfectos para extraer y compartir lógica que no está directamente atada a la interfaz visual (ej: lógica de fetch de datos, manejo de formularios, suscripciones).

### K - Cosillas a Tener en Cuenta (Cons / Downsides)

1.  **Hay Reglas:** Los Hooks tienen reglas estrictas (ej: solo llamarlos en el nivel superior de tu función, no dentro de bucles o condicionales). Necesitas un linter (como ESLint con el plugin de React Hooks) para que te avise si las rompes, porque si no, ¡pueden pasar cosas raras!
2.  **Curva de Aprendizaje:** Aunque `useState` es sencillo, `useEffect` puede ser un poco más complicado de dominar al principio, especialmente entender bien el array de dependencias y la función de limpieza. ¡Requiere práctica!
3.  **Posible Mal Uso:** Hooks como `useCallback` y `useMemo` son para optimizaciones, pero usarlos innecesariamente puede complicar el código más de lo que ayuda. Hay que saber cuándo son realmente necesarios.

### L - Hooks vs. Clases: ¿Cuándo Usar Qué?

1.  **La Tendencia:** Desde su introducción, los Hooks se han vuelto la forma preferida y recomendada por el equipo de React para escribir componentes. Para nuevos desarrollos, **casi siempre querrás usar componentes funcionales con Hooks**.
2.  **¿Y las Clases?** Todavía funcionan y las encontrarás en bases de código antiguas. No hay una necesidad _urgente_ de reescribir todas tus clases a Hooks si funcionan bien, pero para código nuevo, los Hooks suelen ser la mejor opción.
3.  **Resumen Rápido de Diferencias:**

    | Característica           | React Hooks (Funciones)                    | Clases                                      |
    | :----------------------- | :----------------------------------------- | :------------------------------------------ |
    | **Jerarquía**            | Evita anidamiento excesivo (Wrapper Hell)  | Puede llevar a múltiples niveles (HOCs/RPs) |
    | **Complejidad (`this`)** | No hay `this` del que preocuparse          | Requiere entender `this` y `bind`           |
    | **Reutilización Lógica** | Fácil con Custom Hooks                     | Más complejo (HOCs, Render Props, Mixins)   |
    | **Código**               | Generalmente más conciso                   | Más código repetitivo (boilerplate)         |
    | **Uniformidad**          | Una forma más unificada de escribir comps. | Diferencias entre funcional y clase         |
