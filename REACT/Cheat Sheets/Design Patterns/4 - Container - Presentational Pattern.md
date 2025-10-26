## A - Patrón Contenedor/Presentacional (Container/Presentational Pattern)

#### 1. **Definición:**

Piensa en esto como una forma de organizar tu código en React para que sea más ordenado y fácil de manejar. La idea principal es dividir tus componentes (las piezas de tu interfaz) en dos tipos:

- **Presentacionales:** Se preocupan de **CÓMO** se ven las cosas. Son como los diseñadores de interiores de tu app.
- **Contenedores:** Se preocupan de **QUÉ** datos mostrar y de dónde vienen. Son como los ingenieros que conectan todo por detrás.

Separar estas responsabilidades (lo que se llama "separación de intereses" o _separation of concerns_) hace que tu código sea más limpio.

#### 2. **Ejemplo:**

Vamos a ver un ejemplo donde queremos mostrar 6 imágenes de perritos que obtenemos de internet.

- La parte **Presentacional** sería el componente que simplemente recibe una lista de URLs de imágenes y las muestra (`<img>`). No sabe _de dónde_ salieron esas imágenes, solo las pinta en pantalla.
- La parte **Contenedor** sería el componente que _va a internet_, busca esas 6 URLs de imágenes de perritos y luego le _pasa_ esa lista al componente presentacional para que las muestre.

Veremos los componentes específicos en las secciones B y C.

#### 3. **Notas o advertencias:**

- El objetivo es separar la **lógica** (obtener datos, manejar estado) de la **vista** (cómo se muestra la interfaz).
- Aunque útil, hoy en día tenemos otras herramientas como los Hooks de React que a veces hacen este patrón menos necesario (¡lo veremos más adelante!).

---

## B - Componentes Presentacionales (Presentational Components)

#### 1. **Definición:**

Estos componentes son los artistas de tu aplicación. Su única misión es **mostrar** la información que reciben a través de `props` (parámetros que le pasan desde fuera).

- Se centran en el **aspecto visual** (HTML, CSS, estilos).
- **No modifican** los datos que reciben. ¡Son solo para mirar!
- Generalmente **no tienen estado propio** (a menos que sea algo puramente visual, como si un menú está abierto o cerrado).
- Reciben sus datos de los **Componentes Contenedores** (ver sección C).

#### 2. **Ejemplo:**

Este es un componente presentacional (`DogImages`) que recibe una lista de URLs de perritos (`dogs`) y crea una etiqueta `<img>` por cada una.

```jsx
// DogImages.js
import React from "react";

// Recibe 'dogs' como prop
export default function DogImages({ dogs }) {
  // Simplemente mapea sobre la lista y crea elementos <img>
  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />);
}
```

**Explicación del ejemplo:**
Este componente no sabe cómo se obtuvieron las `dogs`. Solo sabe que es una lista de URLs y su trabajo es mostrarlas como imágenes. ¡Simple y directo!

#### 3. **Notas o advertencias:**

- Son fáciles de **reutilizar** porque no dependen de cómo se obtienen los datos. Puedes usar `DogImages` en cualquier lugar donde tengas una lista de URLs de imágenes.
- Son fáciles de **probar** porque, dada la misma `prop`, siempre mostrarán lo mismo.
- A menudo son **componentes funcionales** (como en el ejemplo), ya que no suelen necesitar estado o métodos de ciclo de vida complejos.

---

## C - Componentes Contenedores (Container Components)

#### 1. **Definición:**

Estos son los "cerebros" detrás de la operación. Su trabajo principal es:

- Manejar la **lógica**: obtener datos (por ejemplo, de una API), manejar el estado de la aplicación.
- **Pasar los datos** necesarios a los Componentes Presentacionales que "contienen".
- Generalmente **no tienen estilos propios** ni renderizan mucho HTML directamente. Suelen renderizar uno o más componentes presentacionales.

#### 2. **Ejemplo:**

Este es un componente contenedor (`DogImagesContainer`) que obtiene las imágenes de perritos y luego usa el componente `DogImages` (nuestro componente presentacional de la sección B) para mostrarlas.

```jsx
// DogImagesContainer.js
import React from "react";
import DogImages from "./DogImages"; // Importa el componente presentacional

export default class DogImagesContainer extends React.Component {
  constructor() {
    super();
    // Tiene su propio estado para guardar las URLs de los perros
    this.state = {
      dogs: [],
    };
  }

  // Cuando el componente se monta...
  componentDidMount() {
    // ...va a buscar las imágenes a la API
    fetch("https://dog.ceo/api/breed/labrador/images/random/6")
      .then((res) => res.json())
      // Cuando las recibe, actualiza su estado
      .then(({ message }) => this.setState({ dogs: message }));
  }

  // Renderiza el componente presentacional y le pasa los datos
  render() {
    return <DogImages dogs={this.state.dogs} />;
  }
}
```

**Explicación del ejemplo:**
`DogImagesContainer` se encarga de la tarea "sucia": hablar con la API (`fetch`), guardar las URLs en su `state`, y finalmente, le dice a `DogImages`: "¡Oye, toma estas URLs (`this.state.dogs`) y muéstralas!".

#### 3. **Notas o advertencias:**

- Son los que conectan tu aplicación con el "mundo exterior" (APIs, estado global, etc.).
- Antes de los Hooks, solían ser **componentes de clase** porque necesitaban manejar estado (`this.state`) y ciclo de vida (`componentDidMount`).

---

## D - Alternativa con Hooks (Hooks Alternative)

#### 1. **Definición:**

Con la llegada de los **Hooks** de React (funciones especiales como `useState` y `useEffect`), ahora podemos añadir lógica y estado a los componentes funcionales. Esto a menudo nos permite lograr la misma separación de lógica y vista **sin necesidad de crear un componente Contenedor separado**.

Podemos crear **Hooks personalizados** (custom hooks) para encapsular la lógica (como obtener datos).

#### 2. **Ejemplo:**

Primero, creamos un Hook personalizado `useDogImages` que hace el trabajo de buscar las imágenes:

```jsx
// useDogImages.js
import { useState, useEffect } from "react";

export default function useDogImages() {
  const [dogs, setDogs] = useState([]); // Estado para guardar las URLs

  useEffect(() => {
    // Lógica para buscar las imágenes (igual que antes)
    fetch("https://dog.ceo/api/breed/labrador/images/random/6")
      .then((res) => res.json())
      .then(({ message }) => setDogs(message));
  }, []); // El array vacío [] significa que esto se ejecuta solo una vez

  return dogs; // El hook devuelve la lista de perros
}
```

Ahora, nuestro componente `DogImages` (que antes era puramente presentacional) puede usar este Hook directamente para obtener los datos:

```jsx
// DogImages.js (versión con Hook)
import React from "react";
import useDogImages from "./useDogImages"; // Importa el Hook personalizado

export default function DogImages() {
  // Llama al Hook para obtener los datos
  const dogs = useDogImages();

  // Sigue haciendo lo mismo: mostrar las imágenes
  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />);
}
```

**Explicación del ejemplo:**
¡Mira qué genial! Ya no necesitamos el componente `DogImagesContainer`. El componente `DogImages` sigue siendo bastante "presentacional" en espíritu (su objetivo principal es mostrar `<img>`), pero ahora usa el hook `useDogImages` para obtener los datos que necesita. La lógica de obtención de datos está encapsulada y separada en el Hook.

#### 3. **Notas o advertencias:**

- Los Hooks permiten separar lógica y vista de una manera más directa, a menudo reduciendo la cantidad de "capas" de componentes.
- Esto no significa que el patrón Contenedor/Presentacional esté "mal", pero los Hooks ofrecen una alternativa muy popular y eficiente hoy en día.

---

## E - Ventajas (Pros) del Patrón Contenedor/Presentacional

#### 1. **Definición:**

¿Por qué la gente usaba (y a veces sigue usando) este patrón? Tiene sus puntos buenos:

- **Separación de Intereses Clara:** Mantiene la lógica de datos separada de la lógica de la interfaz de usuario. ¡Orden ante todo!
- **Reutilización:** Los componentes presentacionales son como piezas de LEGO genéricas. Puedes usarlos en muchos sitios diferentes con datos distintos.
- **Colaboración Más Fácil:** Un diseñador o maquetador que no sepa mucho de la lógica de la app puede modificar los componentes presentacionales sin miedo a romper algo complejo.
- **Testing Sencillo:** Probar los componentes presentacionales es fácil. Les das unas `props` y compruebas que renderizan lo esperado. No necesitas simular llamadas a API ni nada complicado para ellos.

#### 2. **Ejemplo:**

Imagina que tienes un componente presentacional `UserProfileCard` que muestra nombre, foto y bio. Puedes usarlo en la página de perfil del usuario (obteniendo los datos de un contenedor que llama a la API del usuario) y también en una lista de amigos (donde otro contenedor le pasa los datos de cada amigo). El `UserProfileCard` es el mismo, ¡solo cambian los datos que recibe!

#### 3. **Notas o advertencias:**

- La principal ventaja es la **organización** y la **claridad** que aporta a proyectos más grandes.

---

## F - Desventajas (Cons) del Patrón Contenedor/Presentacional

#### 1. **Definición:**

No todo es perfecto. Este patrón también tiene algunos inconvenientes, sobre todo comparado con el enfoque moderno de Hooks:

- **Puede ser Excesivo (Overkill):** Para aplicaciones pequeñas o componentes simples, crear dos componentes (uno contenedor y uno presentacional) puede sentirse como mucho trabajo innecesario.
- **Alternativa con Hooks:** Como vimos en la sección D, los Hooks a menudo logran una separación similar con menos código y sin necesidad de componentes de clase (aunque ahora los componentes funcionales también pueden tener estado con Hooks).
- **Capa Extra:** Introduce una capa adicional de componentes (el contenedor) que a veces solo sirve para pasar `props` hacia abajo.

#### 2. **Ejemplo:**

Si solo necesitas mostrar un dato simple que viene de una API, crear un `DataContainer` que llama a la API y un `DataPresenter` que solo muestra ese dato, podría ser más complicado que simplemente usar `useState` y `useEffect` dentro de un único componente funcional.

#### 3. **Notas o advertencias:**

- Hoy en día, con los Hooks, muchos desarrolladores prefieren encapsular la lógica en Hooks personalizados en lugar de usar estrictamente el patrón Contenedor/Presentacional, especialmente porque ya no necesitamos componentes de clase para manejar estado o ciclo de vida.
