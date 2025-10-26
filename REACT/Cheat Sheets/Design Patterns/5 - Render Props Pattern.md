## A - ¿Qué es el Patrón "Render Props"? (Render Props Pattern)

#### 1. **Definición Sencilla:**

Imagina que tienes un componente de React, pero en lugar de que _él mismo_ decida qué dibujar en pantalla, le das una "instrucción" especial a través de sus `props`. Esta instrucción es una **función** que sabe cómo dibujar algo (devuelve JSX). El componente principal simplemente ejecuta esa función que le pasaste. ¡Eso es un "Render Prop"!

En corto: Es una prop cuyo valor es una función que retorna JSX. El componente usa esa función para renderizar.

#### 2. **Ejemplo Básico:**

Digamos que tenemos un componente `Titulo` que no hace nada más que mostrar lo que le digamos usando una prop llamada `render`.

```jsx
// El componente que recibe la "instrucción" (render prop)
const Titulo = (props) => props.render();

// Cómo lo usamos
<Titulo render={() => <h1>✨ ¡Soy un render prop! ✨</h1>} />;
```

**Explicación del ejemplo:**
El componente `Titulo` recibe una prop llamada `render`. Esta prop es una función `() => <h1>...</h1>`. Dentro de `Titulo`, simplemente llamamos a `props.render()`, lo que ejecuta la función y ¡PUM! se renderiza el `<h1>`.

#### 3. **Notas Importantes:**

- **Reusabilidad:** Lo genial es que el componente `Titulo` es súper reutilizable. Puedes usarlo muchas veces pasándole diferentes funciones a `render` para mostrar distintos títulos.
- **No tiene que llamarse `render`:** Aunque se llame "Render Prop", la prop no _tiene_ que llamarse `render`. Cualquier prop que sea una función y se use para renderizar JSX cuenta como una.

  ```jsx
  // Componente que usa varias render props con nombres específicos
  const ContenedorTitulos = (props) => (
    <>
      {props.renderPrimerTitulo()}
      {props.renderSegundoTitulo()}
    </>
  );

  // Cómo lo usamos
  <ContenedorTitulos
    renderPrimerTitulo={() => <h1>✨ ¡Primer Título! ✨</h1>}
    renderSegundoTitulo={() => <h2>🔥 ¡Segundo Título! 🔥</h2>}
  />;
  ```

---

## B - El Poder Real: Pasar Datos con Render Props

#### 1. **Definición:**

La verdadera magia ocurre cuando el componente que _recibe_ la render prop (como nuestro `Titulo` o `ContenedorTitulos`) tiene alguna información o estado propio y quiere **pasárselo** a la función render prop para que lo use.

Piensa en ello como: "Oye, función render prop, aquí tienes estos datos que calculé o que tengo guardados, úsalos para dibujar lo que tengas que dibujar".

#### 2. **Ejemplo Conceptual:**

```jsx
// Un componente que tiene datos y los pasa a su render prop
function ComponenteConDatos(props) {
  const datosImportantes = { usuario: "Ana", nivel: 5 };

  // Llama a la función render prop PASÁNDOLE los datos
  return props.render(datosImportantes);
}

// Cómo lo usamos, recibiendo los datos en la función
<ComponenteConDatos
  render={(info) => (
    <div>
      <p>Usuario: {info.usuario}</p>
      <p>Nivel: {info.nivel}</p>
    </div>
  )}
/>;
```

**Explicación del ejemplo:**
`ComponenteConDatos` tiene un objeto `datosImportantes`. Cuando llama a `props.render()`, le pasa ese objeto como argumento. La función que le dimos a `render` (la que empieza con `(info) => ...`) recibe esos datos en su parámetro `info` y los usa para mostrar el nombre y nivel del usuario.

---

## C - Caso Práctico: El Convertidor de Temperatura

#### 1. **El Problema:**

Imagina una app con un campo para escribir grados Celsius (`Input`), y otros dos componentes (`Kelvin`, `Fahrenheit`) que deben mostrar la temperatura convertida.

![Diagrama inicial del problema de estado](https://www.patterns.dev/img/react-renderprops-1.png)

El componente `Input` tiene el valor (estado), pero `Kelvin` y `Fahrenheit` son hermanos y no pueden acceder directamente a ese valor.

#### 2. **Solución 1: "Levantar el Estado" (Lifting State Up)**

Una forma es mover el estado (el valor de la temperatura) al componente padre común (`App`). `App` guarda el estado y lo pasa como props a `Input`, `Kelvin` y `Fahrenheit`.

![Diagrama de levantar el estado](https://www.patterns.dev/img/react-renderprops-2.png)

- **Desventaja:** En apps grandes, esto puede hacer que muchos componentes se re-rendericen innecesariamente cada vez que cambia el estado, afectando el rendimiento.

#### 3. **Solución 2: ¡Usar Render Props!**

Hacemos que el componente `Input` (que tiene el estado) acepte una render prop. `Input` se encargará de llamar a esa función pasándole el valor actual de la temperatura.

```jsx
// Input.js - Ahora acepta una render prop
import React, { useState } from "react";

function Input(props) {
  const [value, setValue] = useState(""); // Estado local

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Temp en °C"
      />
      {/* Llama a la render prop pasándole el estado 'value' */}
      {props.render(value)}
    </>
  );
}

// App.js - Usa Input con la render prop
import React from "react";
import Input from "./Input"; // Asegúrate de importar Input
import Kelvin from "./Kelvin"; // Y los otros componentes
import Fahrenheit from "./Fahrenheit";
import "./styles.css";

export default function App() {
  return (
    <div className="App">
      <h1>☃️ Convertidor de Temperatura 🌞</h1>
      <Input
        render={(
          valorActual // La función recibe el valor de Input
        ) => (
          <>
            {/* Kelvin y Fahrenheit reciben el valor directamente */}
            <Kelvin value={valorActual} />
            <Fahrenheit value={valorActual} />
          </>
        )}
      />
    </div>
  );
}

// Kelvin.js y Fahrenheit.js (simplificados)
function Kelvin({ value }) {
  return <div className="temp">{parseInt(value || 0) + 273.15}K</div>;
}

function Fahrenheit({ value }) {
  return <div className="temp">{(parseInt(value || 0) * 9) / 5 + 32}°F</div>;
}
```

**Explicación del ejemplo:**
Ahora `Input` maneja su propio estado `value`. Cuando renderiza, llama a `props.render(value)`. En `App`, le pasamos una función a `render`. Esa función recibe `valorActual` (que es el `value` de `Input`) y la usa para renderizar `Kelvin` y `Fahrenheit`, pasándoles ese valor. ¡Problema resuelto sin levantar el estado hasta `App`!

![Diagrama de la solución con Render Props](https://www.patterns.dev/img/react-renderprops-3.png)

---

## D - Variante Común: `children` como Función

#### 1. **Definición:**

En React, puedes pasar una función como "hijo" directo de un componente (entre las etiquetas `<Componente>...</Componente>`). ¡Esto también es una forma de Render Prop! El componente accede a esta función a través de `props.children`.

#### 2. **Ejemplo (Convertidor de Temperatura con `children`):**

```jsx
// Input.js - Usando props.children en lugar de props.render
import React, { useState } from "react";

function Input(props) {
  const [value, setValue] = useState("");

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Temp en °C"
      />
      {/* Llama a la función pasada como children, pasándole el estado 'value' */}
      {props.children(value)}
    </>
  );
}

// App.js - Pasando la función como children
import React from "react";
import Input from "./Input";
import Kelvin from "./Kelvin";
import Fahrenheit from "./Fahrenheit";
import "./styles.css";

export default function App() {
  return (
    <div className="App">
      <h1>☃️ Convertidor de Temperatura 🌞</h1>
      <Input>
        {/* La función ahora va aquí, entre las etiquetas */}
        {(valorActual) => (
          <>
            <Kelvin value={valorActual} />
            <Fahrenheit value={valorActual} />
          </>
        )}
      </Input>
    </div>
  );
}
```

**Explicación del ejemplo:**
Es casi idéntico al anterior, pero en `App.js`, la función que renderiza `Kelvin` y `Fahrenheit` se pasa _dentro_ de las etiquetas `<Input>...</Input>`. Dentro de `Input.js`, en lugar de llamar a `props.render(value)`, llamamos a `props.children(value)`. Funciona igual, ¡es solo una convención diferente y muy popular!

---

## E - Render Props vs. Hooks (Una Mirada Rápida)

#### 1. **Contexto:**

Los Hooks de React (como `useState`, `useEffect`, `useContext`, etc.) son una forma más moderna de manejar estado y lógica reutilizable en componentes funcionales. En muchos casos, pueden reemplazar a los Render Props y a los [Higher Order Components (HOCs)](https://www.patterns.dev/posts/hoc-pattern).

#### 2. **Ejemplo (Apollo Client - Antes y Después):**

El texto muestra un ejemplo con Apollo Client (una librería para GraphQL). Antes de los Hooks, se usaban componentes como `<Query>` o `<Mutation>` con Render Props (usando `children` como función) para obtener datos o ejecutar acciones.

```jsx
// Ejemplo con Render Prop (Componente Mutation de Apollo)
import { Mutation } from "react-apollo";
import { ADD_MESSAGE } from "./resolvers";

// ... dentro del render de un componente de clase ...
<Mutation mutation={ADD_MESSAGE} variables={{ message: this.state.message }}>
  {(
    addMessage // Función como children (render prop)
  ) => (
    <div className="input-row">
      <input onChange={this.handleChange} type="text" />
      <button onClick={addMessage}>Add</button> {/* Usa la función recibida */}
    </div>
  )}
</Mutation>;
```

Con Hooks, el código se vuelve a menudo más directo y menos anidado:

```jsx
// Mismo ejemplo con Hooks (Hook useMutation de Apollo)
import React, { useState } from "react";
import { useMutation } from "@apollo/react-hooks";
import { ADD_MESSAGE } from "./resolvers";

export default function Input() {
  const [message, setMessage] = useState("");
  // El Hook devuelve la función directamente
  const [addMessage] = useMutation(ADD_MESSAGE, { variables: { message } });

  return (
    <div className="input-row">
      <input onChange={(e) => setMessage(e.target.value)} type="text" />
      <button onClick={addMessage}>Add</button> {/* Usa la función del Hook */}
    </div>
  );
}
```

**Explicación del ejemplo:**
El Hook `useMutation` nos da directamente la función `addMessage` sin necesidad de anidar componentes y pasar funciones como `children`. ¡Más limpio!

#### 3. **Notas:**

- **Anidamiento ("Nesting Hell"):** Usar muchos Render Props anidados puede hacer el código difícil de leer (aunque a veces menos que con HOCs). Los Hooks suelen evitar este problema.
- **¿Render Props está obsoleto?** ¡No del todo! Sigue siendo un patrón válido y útil, especialmente si trabajas con componentes de clase o librerías que aún no usan Hooks extensivamente. ¡Entenderlo es importante!

---

## F - Ventajas y Desventajas de Render Props

#### 1. **Ventajas (Pros 👍):**

- **Reutilización Clara:** Excelente para compartir lógica o estado entre componentes.
- **Evita Colisiones de Nombres:** A diferencia de los HOCs, tú decides cómo llamar a las props que recibes en la función render prop, así que no hay riesgo de que una prop inyectada por el HOC sobreescriba una tuya sin que te des cuenta.
- **Props Explícitas:** Sabes exactamente qué datos está pasando el componente contenedor a tu función render prop porque son los argumentos de esa función. ¡No hay magia oculta!
- **Separación de Intereses:** Ayuda a separar la lógica (en el componente contenedor) de la presentación (en la función render prop).

#### 2. **Desventajas (Cons 👎):**

- **Reemplazado en parte por Hooks:** Para muchos casos de uso comunes (compartir estado, lógica, conectar a contextos), los Hooks ofrecen una sintaxis más directa y menos anidada en componentes funcionales.
- **Anidamiento Potencial:** Si necesitas datos de varias fuentes que usan render props, puedes terminar anidando componentes `<DataSource1>{data1 => <DataSource2>{data2 => ...}</DataSource2>}</DataSource1>`, lo cual puede ser un poco engorroso.
- **Limitaciones en la Función Render Prop:** La función que pasas como render prop es solo eso, una función. No puedes usar Hooks o métodos de ciclo de vida _dentro_ de ella directamente (aunque el componente _contenedor_ sí puede usar todo eso).
