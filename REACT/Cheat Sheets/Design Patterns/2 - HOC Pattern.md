## A - ¿Qué es un Higher Order Component (HOC)?

#### 1. **Definición:**

Piensa en un HOC como una "fábrica de componentes". Es una **función** especial en JavaScript que:

1.  **Recibe** un componente como si fuera un ingrediente.
2.  **Le añade** alguna funcionalidad extra o props (como un superpoder o un accesorio).
3.  **Devuelve** un **nuevo** componente mejorado con esa funcionalidad añadida.

**En corto:** Es una técnica para **reutilizar lógica** entre componentes. Su nombre original en inglés es _Higher Order Component_, ¡así lo encontrarás en la documentación!

#### 2. **Notas o advertencias:**

- El objetivo principal es mantener tu código `DRY` (Don't Repeat Yourself - No te repitas).
- No modifican el componente original, ¡crean uno nuevo!

---

## B - Ejemplo Sencillo: Añadiendo Estilos (`withStyles`)

#### 1. **Definición:**

Supongamos que quieres aplicar el mismo `padding` y `margin` a varios botones o textos en tu app. En lugar de copiar y pegar los estilos, creamos un HOC llamado `withStyles`.

#### 2. **Ejemplo:**

```javascript
// --- El HOC: "withStyles" ---
// Recibe un Componente...
function withStyles(Component) {
  // ...y devuelve un NUEVO componente (una función en este caso)
  return (props) => {
    // 1. Define la lógica/datos que quieres añadir (los estilos)
    const style = { padding: "0.2rem", margin: "1rem" };
    // 2. Renderiza el Componente original, pasándole los nuevos estilos
    //    y cualquier otra prop que ya tuviera (...props)
    return <Component style={style} {...props} />;
  };
}

// --- Componentes Originales ---
const Button = () => <button>¡Haz clic!</button>;
const Text = () => <p>Hola Mundo!</p>;

// --- Usando el HOC ---
// Creamos versiones "mejoradas" de Button y Text
const StyledButton = withStyles(Button);
const StyledText = withStyles(Text);

// Ahora puedes usar <StyledButton /> y <StyledText /> en tu app,
// y ya tendrán los estilos aplicados.
```

**Explicación del ejemplo:**

- La función `withStyles` toma un componente (`Button` o `Text`).
- Dentro, crea y devuelve _otro_ componente funcional (`props => { ... }`).
- Este nuevo componente calcula o define la lógica extra (el objeto `style`).
- Finalmente, renderiza el componente original (`<Component ... />`), inyectándole la nueva `prop` (`style={style}`) y asegurándose de pasarle todas las `props` que pudiera tener antes (`{...props}`).
- `StyledButton` y `StyledText` son los nuevos componentes listos para usar, ¡ya con estilos!

---

## C - Ejemplo Práctico: Indicador de Carga (`withLoader`) - Creación

#### 1. **Definición:**

Vamos a crear un HOC más útil: `withLoader`. Este HOC se encargará de:

1.  Recibir un componente y una URL de API.
2.  Mostrar un mensaje "Loading..." mientras espera los datos de la URL.
3.  Una vez que llegan los datos, mostrar el componente original pasándole esos datos.

#### 2. **Ejemplo (El HOC `withLoader.js`):**

```javascript
import React, { useEffect, useState } from "react";

// Recibe el Componente a mostrar (Element) y la URL de donde cargar datos
export default function withLoader(Element, url) {
  // Devuelve un nuevo componente funcional
  return (props) => {
    // Estado para guardar los datos (inicialmente null)
    const [data, setData] = useState(null);

    // Efecto para cargar los datos cuando el componente se monta
    useEffect(() => {
      async function getData() {
        const res = await fetch(url); // Pide los datos a la URL
        const data = await res.json(); // Convierte la respuesta a JSON
        setData(data); // Guarda los datos en el estado
      }

      getData(); // Llama a la función para cargar datos
    }, []); // El array vacío [] significa que esto se ejecuta solo una vez

    // --- Lógica condicional ---
    // Si aún no hay datos (!data)...
    if (!data) {
      return <div>Loading...</div>; // Muestra el mensaje de carga
    }

    // Si ya hay datos...
    // Renderiza el Componente original (Element)
    // Pasándole sus props originales (...props) Y los datos cargados (data={data})
    return <Element {...props} data={data} />;
  };
}
```

**Explicación del ejemplo:**

- El HOC `withLoader` es una función que toma `Element` (el componente a envolver) y `url` (la API de donde sacar datos).
- Devuelve un _nuevo_ componente.
- Este nuevo componente usa `useState` para manejar el estado `data` (donde irán los datos de la API).
- Usa `useEffect` para hacer la llamada a la `url` (usando `fetch`) _solo una vez_ cuando se monta.
- Mientras `data` sea `null` (porque aún no ha llegado la respuesta), muestra "Loading...".
- Cuando `fetch` termina y `setData` actualiza el estado, el componente se vuelve a renderizar. Ahora `data` ya no es `null`, así que renderiza el `Element` original, pasándole una nueva `prop` llamada `data` con los datos recibidos.

#### 3. **Notas o advertencias:**

- Este HOC es súper reutilizable porque le puedes pasar cualquier componente y cualquier URL. ¡No está atado a una API específica!

---

## D - Ejemplo Práctico: Indicador de Carga (`withLoader`) - Uso

#### 1. **Definición:**

Ahora, veamos cómo usar nuestro `withLoader` con un componente real, como uno que muestra imágenes de perritos (`DogImages`).

#### 2. **Ejemplo (Usando el HOC en `DogImages.js`):**

```javascript
import React from "react";
import withLoader from "./withLoader"; // Importamos nuestro HOC

// --- Componente Original (DogImages) ---
// Este componente AHORA ESPERA recibir una prop llamada "data"
function DogImages(props) {
  // Asume que props.data tiene la estructura { message: [...] }
  // y mapea sobre el array de URLs de imágenes
  return props.data.message.map((dog, index) => (
    <img src={dog} alt="Dog" key={index} />
  ));
}

// --- Exportación "Mejorada" ---
// En lugar de exportar DogImages directamente...
// export default DogImages; <--- NO HACEMOS ESTO

// ...exportamos el RESULTADO de llamar a withLoader:
export default withLoader(
  DogImages, // El componente que queremos envolver
  "https://dog.ceo/api/breed/labrador/images/random/6" // La URL específica para este caso
);
```

**Explicación del ejemplo:**

1.  Importamos el HOC `withLoader`.
2.  Definimos nuestro componente `DogImages` de forma normal, pero ahora _espera_ recibir los datos a través de `props.data`. Ya no se preocupa por _cómo_ cargar los datos.
3.  Al final del archivo, en lugar de `export default DogImages`, hacemos `export default withLoader(DogImages, "URL_DE_LA_API")`.
4.  Esto significa que lo que realmente se exporta no es `DogImages` puro, sino la versión "mejorada" que `withLoader` creó para él, ya configurada con la URL de la API de perritos.
5.  Cuando uses `<DogImages />` en tu app, primero verás "Loading...", y luego, cuando los datos lleguen, el HOC renderizará el `DogImages` original pasándole los datos en `props.data`.

#### 3. **Notas o advertencias:**

- ¡Fíjate qué limpio queda el componente `DogImages`! Toda la lógica de carga está encapsulada en el HOC `withLoader`. Esto es genial para la **separación de conceptos**.

---

## E - Componiendo HOCs (¡Más Poder!)

#### 1. **Definición:**

¡Puedes usar más de un HOC en el mismo componente! Esto se llama "composición". Imagina que además de cargar datos, quieres que tu lista de perritos muestre un texto "Hovering!" cuando pasas el ratón por encima. Podemos crear otro HOC para eso (`withHover`) y combinarlo con `withLoader`.

#### 2. **Ejemplo:**

```javascript
// --- Otro HOC: withHover.js ---
import React, { useState } from "react";

export default function withHover(Element) {
  return props => {
    const [hovering, setHover] = useState(false); // Estado para saber si el ratón está encima

    return (
      <Element
        {...props} // Pasa las props originales
        hovering={hovering} // Añade la prop "hovering"
        // Añade manejadores de eventos del ratón
        onMouseEnter={() => setHover(true)}
        onMouseLeave={() => setHover(false)}
      />
    );
  };
}

// --- En DogImages.js ---
import React from "react";
import withLoader from "./withLoader";
import withHover from "./withHover"; // Importamos el nuevo HOC

function DogImages(props) {
  // Ahora recibe props de AMBOS HOCs: "data" de withLoader y "hovering" de withHover
  return (
    // Usamos los manejadores onMouseEnter/Leave que nos pasó withHover
    <div {...props}>
      {/* Muestra el div si props.hovering es true */}
      {props.hovering && <div id="hover">Hovering!</div>}
      <div id="list">
        {/* Sigue usando props.data de withLoader */}
        {props.data.message.map((dog, index) => (
          <img src={dog} alt="Dog" key={index} />
        ))}
      </div>
    </div>
  );
}

// --- Exportación Compuesta ---
// ¡Envolvemos un HOC dentro del otro!
export default withHover( // 1. El HOC más externo: withHover
  withLoader( // 2. El HOC interno: withLoader
    DogImages, // 3. El componente base
    "https://dog.ceo/api/breed/labrador/images/random/6" // 4. La URL para withLoader
  )
);
```

**Explicación del ejemplo:**

1.  Creamos `withHover`, un HOC que añade una prop booleana `hovering` y los manejadores `onMouseEnter`/`onMouseLeave` al componente que envuelve.
2.  En `DogImages.js`, ahora importamos _ambos_ HOCs.
3.  El componente `DogImages` ahora usa `props.data` (de `withLoader`) y `props.hovering` (de `withHover`). También necesita esparcir `{...props}` en su `div` principal para que los `onMouseEnter`/`onMouseLeave` funcionen.
4.  La magia está en la exportación: `withHover(withLoader(DogImages, url))`. Se lee de adentro hacia afuera:
    - Primero, `withLoader` envuelve a `DogImages` y le da la prop `data`.
    - Luego, `withHover` envuelve _el resultado anterior_ y le añade la prop `hovering` y los manejadores.
5.  El componente final `DogImages` recibe las props combinadas de todos los HOCs.

#### 3. **Notas o advertencias:**

- Componer HOCs es poderoso, pero puede hacer que el árbol de componentes se vea muy anidado (`<withHover><withLoader><DogImages /></withLoader></withHover>`) y a veces sea un poco difícil seguir de dónde viene cada prop.

---

## F - Una Alternativa Moderna: React Hooks

#### 1. **Definición:**

Con la llegada de los **React Hooks**, muchas de las cosas que hacíamos con HOCs ahora se pueden hacer de forma más directa _dentro_ del componente funcional usando Hooks personalizados. Los Hooks nos permiten "enganchar" lógica reutilizable (como manejo de estado o efectos) a nuestros componentes.

#### 2. **Ejemplo (Reemplazando `withHover` con un Hook `useHover`):**

```javascript
// --- El Hook Personalizado: useHover.js ---
import { useState, useRef, useEffect } from "react";

export default function useHover() {
  const [hovering, setHover] = useState(false); // Estado interno del hook
  const ref = useRef(null); // Ref para apuntar al elemento DOM

  const handleMouseOver = () => setHover(true);
  const handleMouseOut = () => setHover(false);

  useEffect(() => {
    const node = ref.current; // El elemento DOM real
    if (node) {
      // Añade los listeners directamente al nodo
      node.addEventListener("mouseover", handleMouseOver);
      node.addEventListener("mouseout", handleMouseOut);

      // Función de limpieza: se ejecuta cuando el componente se desmonta
      return () => {
        node.removeEventListener("mouseover", handleMouseOver);
        node.removeEventListener("mouseout", handleMouseOut);
      };
    }
  }, [ref.current]); // Se vuelve a ejecutar si el nodo cambia

  // El hook devuelve la ref y el estado "hovering"
  return [ref, hovering];
}

// --- En DogImages.js (usando el Hook) ---
import React from "react";
import withLoader from "./withLoader"; // Todavía usamos withLoader (podría ser un hook también!)
import useHover from "./useHover"; // Importamos el Hook

function DogImages(props) {
  // ¡Usamos el hook DENTRO del componente!
  const [hoverRef, hovering] = useHover(); // Obtenemos la ref y el estado

  return (
    // Asignamos la ref al div que queremos observar
    <div ref={hoverRef} {...props}>
      {/* Usamos el estado "hovering" directamente */}
      {hovering && <div id="hover">Hovering!</div>}
      <div id="list">
        {props.data.message.map((dog, index) => (
          <img src={dog} alt="Dog" key={index} />
        ))}
      </div>
    </div>
  );
}

// La exportación ahora solo usa withLoader
export default withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);
```

**Explicación del ejemplo:**

1.  Creamos `useHover`, una función que empieza con `use` (convención para Hooks).
2.  Dentro, usa `useState` para el estado `hovering` y `useRef` para crear una `ref`. Una `ref` es como un puntero a un elemento del DOM.
3.  Usa `useEffect` para añadir y quitar (`return () => ...`) los event listeners (`mouseover`, `mouseout`) directamente al elemento DOM al que apunta la `ref`.
4.  El Hook devuelve un array con la `ref` y el valor de `hovering`.
5.  En `DogImages`, llamamos a `useHover()` y obtenemos `hoverRef` y `hovering`.
6.  Asignamos `hoverRef` al `div` principal usando el atributo `ref={hoverRef}`.
7.  Usamos la variable `hovering` directamente para mostrar o no el texto.
8.  ¡Ya no necesitamos el HOC `withHover`! La lógica está ahora _dentro_ del componente gracias al Hook.

#### 3. **Notas o advertencias:**

- Los Hooks a menudo hacen que el código sea más fácil de leer y el árbol de componentes menos profundo (menos anidamiento).
- ¡Ojo! Los Hooks solo se pueden usar dentro de componentes funcionales o de otros Hooks.

---

## G - HOCs vs. Hooks: ¿Cuándo Usar Cuál?

#### 1. **Definición:**

Tanto HOCs como Hooks sirven para reutilizar lógica, pero tienen sus momentos ideales. Los Hooks son a menudo la opción preferida hoy en día, pero los HOCs siguen siendo útiles.

#### 2. **Comparación Rápida:**

- **Anidamiento:**
  - **HOCs:** Pueden crear muchos niveles de anidamiento en tu árbol de componentes si compones varios. (`<Hoc1><Hoc2><Componente /></Hoc2></Hoc1>`)
  - **Hooks:** Mantienen el árbol más plano, ya que la lógica se usa _dentro_ del componente.
- **Flexibilidad:**
  - **HOCs:** Geniales cuando la misma lógica, _sin cambios_, se aplica a muchos componentes. El componente envuelto no necesita saber mucho sobre el HOC.
  - **Hooks:** Muy flexibles. Fáciles de personalizar para cada componente que los usa. Permiten extraer y compartir lógica más granular.
- **Props:**
  - **HOCs:** Pasan lógica/datos a través de props. Cuidado con los choques de nombres de props (ver sección "Contras").
  - **Hooks:** Devuelven valores (estado, funciones, refs) que usas directamente en el componente. Menos riesgo de colisión de nombres si se usan bien.

#### 3. **Cuándo usar HOCs (según el texto):**

- Cuando necesitas aplicar **exactamente la misma lógica**, sin personalización, a **muchos** componentes.
- Cuando el componente base puede funcionar perfectamente **sin** la lógica añadida por el HOC (son como mejoras opcionales).

#### 4. **Cuándo usar Hooks (según el texto):**

- Cuando la lógica necesita ser **personalizada** para cada componente que la usa.
- Cuando la lógica solo la usan **pocos** componentes.
- Cuando la lógica añade **muchas props** diferentes al componente (con Hooks, puedes devolver un objeto o array con todo lo necesario).

#### 5. **Notas o advertencias:**

- Como dicen los docs de React: _"En la mayoría de los casos, los Hooks serán suficientes y pueden ayudar a reducir el anidamiento en tu árbol."_
- ¡No es una guerra! A veces, incluso pueden coexistir.

---

## H - Caso de Estudio: Apollo Client (De HOC a Hook)

#### 1. **Definición:**

Apollo Client es una biblioteca popular para manejar datos con GraphQL en React. Es un buen ejemplo de cómo la comunidad ha adoptado los Hooks. Antes, usaba principalmente HOCs; ahora, los Hooks son la forma recomendada.

#### 2. **Antes (con HOC `graphql()`):**

```javascript
// (Código muy simplificado del ejemplo del texto)
import { graphql } from "react-apollo"; // HOC de Apollo v2/v3

class Input extends React.Component {
  // ...lógica del componente...
  handleClick = () => {
    // El HOC inyecta una prop "mutate"
    this.props.mutate({ variables: { message: this.state.message } });
  };
  // ...render...
}

// Envolvemos el componente con el HOC para darle la capacidad de mutar datos
export default graphql(ADD_MESSAGE)(Input);
```

**Problema:** Si necesitabas datos de varias fuentes o varias mutaciones, tenías que componer (anidar) múltiples HOCs `graphql()`, lo cual podía volverse confuso.

#### 3. **Ahora (con Hook `useMutation`):**

```javascript
// (Código muy simplificado del ejemplo del texto)
import { useMutation } from "@apollo/react-hooks"; // Hook de Apollo v3+

export default function Input() {
  const [message, setMessage] = useState("");
  // Usamos el Hook DENTRO del componente
  const [addMessage] = useMutation(ADD_MESSAGE, {
    variables: { message },
  });

  return (
    // ...render...
    // Usamos la función "addMessage" devuelta por el Hook directamente
    <button onClick={addMessage}>Add</button>
    // ...
  );
}
```

**Ventaja:** ¡Mucho más directo! Si necesitas más datos o mutaciones, simplemente llamas a otro Hook (`useQuery`, `useMutation`, etc.) dentro del mismo componente. Menos código, más claro de dónde viene la lógica.

#### 4. **Notas o advertencias:**

- Este cambio muestra cómo los Hooks pueden mejorar la experiencia del desarrollador y simplificar el código en casos prácticos.

---

## I - Pros de los HOCs

#### 1. **Reutilización de Lógica:**

¡Es su superpoder principal! Escribes la lógica una vez (en el HOC) y la aplicas a tantos componentes como quieras. Mantiene tu código **DRY** (Don't Repeat Yourself).

#### 2. **Separación de Conceptos (Separation of Concerns):**

Ayudan a separar responsabilidades. Por ejemplo, el componente `DogImages` se enfoca en _mostrar_ imágenes, mientras que `withLoader` se enfoca en _cargarlas_. Cada uno hace su trabajo sin meterse en el del otro.

#### 3. **Composición:**

Puedes combinar varios HOCs para añadir múltiples capas de funcionalidad a un componente (como vimos con `withHover` y `withLoader`).

---

## J - Contras de los HOCs

#### 1. **Colisión de Nombres de Props (Prop Collisions):**

¡Este es un clásico! ¿Qué pasa si tu HOC (`withStyles`) quiere pasar una prop llamada `style`, pero tu componente (`Button`) ya tenía su propia prop `style`?

```javascript
// HOC que añade 'style'
function withStyles(Component) {
  return (props) => {
    const style = { padding: "0.2rem", margin: "1rem" };
    // ¡Aquí la prop 'style' del HOC podría sobreescribir la original!
    return <Component style={style} {...props} />;
  };
}

// Componente que YA tiene una prop 'style'
const Button = () => <button style={{ color: "red" }}>Click me!</button>;

// Al usarlo, el 'color: red' original se pierde!
const StyledButton = withStyles(Button);
```

**Solución:** El HOC debe ser más inteligente, por ejemplo, fusionando los estilos o usando un nombre de prop menos común.

```javascript
// HOC mejorado que fusiona estilos
function withStyles(Component) {
  return (props) => {
    const hocStyle = { padding: "0.2rem", margin: "1rem" };
    // Fusiona el estilo del HOC con el estilo original del componente (si existe)
    const mergedStyle = { ...hocStyle, ...props.style };
    return <Component {...props} style={mergedStyle} />; // Pasa las props originales PRIMERO
  };
}
```

#### 2. **Infierno de Envolturas (Wrapper Hell) y Depuración:**

Cuando compones muchos HOCs, tu árbol de componentes en las herramientas de desarrollo de React puede verse así: `<WithAuth><WithRouter><WithStyles><MyComponent /></WithStyles></WithRouter></WithAuth>`. Esto puede hacer difícil rastrear de dónde viene una prop específica o depurar problemas. ¿Qué HOC pasó esta prop? ¿En qué orden se aplican?

#### 3. **Ofuscación de Props:**

No siempre es obvio qué props le llegarán al componente final solo mirando el componente base. Tienes que saber qué props inyecta cada HOC en la cadena.
