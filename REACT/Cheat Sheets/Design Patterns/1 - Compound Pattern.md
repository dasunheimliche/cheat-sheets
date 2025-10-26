## A - Compound Component Pattern (Patrón de Componente Compuesto)

#### 1. **Definición:**

Piensa en componentes que son como mejores amigos: funcionan juntos, comparten información (estado) y lógica, y realmente no tienen mucho sentido el uno sin el other. El **Compound Component Pattern** es una técnica para crear estos grupos de componentes que colaboran para realizar una tarea específica.

Es ideal para cosas como menús desplegables (`select`), listas con opciones, o cualquier conjunto de componentes que necesiten un estado y comportamiento compartidos.

#### 2. **Objetivo Principal:**

Permitir que varios componentes trabajen en equipo, compartiendo un estado interno y lógica, sin que tú tengas que manejar toda esa complejidad desde fuera. Hacen que el uso del conjunto se sienta como si fuera un solo componente mágico.

---

Ahora, veamos cómo podemos implementar esto usando una herramienta genial de React: la **Context API**.

## B - Implementación con Context API: El "Cerebro" Compartido (`FlyOut` y Context)

#### 1. **Definición:**

Para que nuestros componentes "amigos" puedan compartir información (como si un menú está abierto o cerrado), necesitamos un lugar central para guardar esa información (el estado) y una forma de pasarla. Aquí es donde entra la **Context API** de React.

Crearemos un componente principal (llamémoslo `FlyOut`) que manejará el estado compartido (por ejemplo, `open` para saber si el menú está visible y `toggle` para cambiar ese estado). Este componente usará un `Context.Provider` para "transmitir" ese estado a todos sus componentes hijos que lo necesiten.

#### 2. **Ejemplo:**

```javascript
import React, { useState, createContext } from "react";

// 1. Creamos el Contexto: Es como crear un canal de radio privado.
const FlyOutContext = createContext();

// 2. Creamos el Componente Principal (Wrapper): Este maneja el estado y lo provee.
function FlyOut(props) {
  // Guarda el estado: ¿está abierto o cerrado? Inicialmente cerrado (false).
  const [open, setOpen] = useState(false);

  // Función para cambiar el estado (abrir/cerrar)
  const toggle = () => setOpen(!open);

  return (
    // 3. Usamos el Provider: Transmitimos 'open' y 'toggle' a los hijos.
    <FlyOutContext.Provider value={{ open, toggle }}>
      {props.children} {/* Aquí irán los componentes hijos */}
    </FlyOutContext.Provider>
  );
}

// ¡Ojo! Aún no exportamos FlyOut directamente, lo haremos más adelante.
```

**Explicación del ejemplo:**

- `createContext()`: Crea el "canal" (`FlyOutContext`) por donde viajará la información.
- `FlyOut`: Es el componente "padre" que tiene el estado (`open`) y la función para cambiarlo (`toggle`).
- `FlyOutContext.Provider`: Envuelve a los componentes hijos (`props.children`) y les da acceso a `value={{ open, toggle }}`. Cualquier componente dentro de este Provider podrá "sintonizar" el canal y recibir `open` y `toggle`.

#### 3. **Notas o advertencias:**

- Este componente `FlyOut` es el corazón del patrón en esta implementación. Guarda el estado y lo comparte.
- `props.children` es una prop especial en React que contiene cualquier cosa que pongas _dentro_ de las etiquetas `<FlyOut>...</FlyOut>` cuando lo uses.

---

## C - Implementación con Context API: Los "Ayudantes" (`Toggle`, `List`, `Item`)

#### 1. **Definición:**

Ahora necesitamos los componentes "ayudantes" que usarán la información compartida. Estos serán los botones para abrir/cerrar, la lista que se muestra/oculta, y los elementos de esa lista. Usarán el hook `useContext` para "sintonizar" el `FlyOutContext` y obtener `open` y `toggle`.

#### 2. **Ejemplo:**

```javascript
import React, { useContext } from "react";
// Asumimos que FlyOutContext ya fue creado como en el paso B
// import { FlyOutContext } from './FlyOut'; // (Si estuviera en otro archivo)
import Icon from "./Icon"; // Un componente de icono cualquiera

// Componente para el botón que abre/cierra
function Toggle() {
  // 1. Sintonizamos el contexto para obtener 'open' y 'toggle'
  const { open, toggle } = useContext(FlyOutContext);

  // 2. Al hacer clic, llamamos a 'toggle' para cambiar el estado
  return (
    <div onClick={toggle}>
      {" "}
      {/* Usamos la función toggle del contexto */}
      <Icon /> {/* Muestra un icono */}
    </div>
  );
}

// Componente para la lista que se muestra/oculta
function List({ children }) {
  // 1. Sintonizamos para obtener 'open'
  const { open } = useContext(FlyOutContext);

  // 2. Solo muestra la lista (<ul>) si 'open' es true
  return open ? <ul>{children}</ul> : null;
}

// Componente para cada elemento de la lista
function Item({ children }) {
  // Este es simple, solo envuelve el contenido en un <li>
  return <li>{children}</li>;
}

// ¡Ojo! Aún no los hemos conectado formalmente a FlyOut. Eso viene ahora.
```

**Explicación del ejemplo:**

- `useContext(FlyOutContext)`: Es la forma en que `Toggle` y `List` acceden a los valores (`open`, `toggle`) que `FlyOut` está proveyendo.
- `Toggle`: Llama a la función `toggle` del contexto cuando se hace clic.
- `List`: Usa el valor `open` del contexto para decidir si debe renderizarse (mostrarse) o no.
- `Item`: Es un componente simple que no necesita el contexto directamente, pero vive dentro de `List`.

#### 3. **Notas o advertencias:**

- ¡Mira qué limpios quedan estos componentes! No manejan estado propio, solo usan el que les llega del contexto.
- Para que `useContext` funcione, estos componentes _deben_ ser renderizados en algún lugar _dentro_ del `FlyOutContext.Provider` (es decir, dentro de `<FlyOut>...</FlyOut>`).

---

## D - Implementación con Context API: ¡Ensamblando Todo!

#### 1. **Definición:**

Tenemos el "cerebro" (`FlyOut`) y los "ayudantes" (`Toggle`, `List`, `Item`). Ahora, vamos a hacer algo ingenioso: adjuntaremos los ayudantes como propiedades del cerebro. Es como si `FlyOut` tuviera sus propias herramientas incorporadas: `FlyOut.Toggle`, `FlyOut.List`, `FlyOut.Item`. Esto hace que importar y usar el conjunto sea súper fácil.

#### 2. **Ejemplo:**

**Paso 1: Adjuntar los componentes como propiedades (en el archivo `FlyOut.js`)**

```javascript
// ... (código de FlyOut, Toggle, List, Item de los pasos B y C) ...

// ¡La magia! Hacemos que Toggle, List, e Item sean "parte" de FlyOut.
FlyOut.Toggle = Toggle;
FlyOut.List = List;
FlyOut.Item = Item;

// Ahora sí, exportamos el componente FlyOut con sus "herramientas" adjuntas.
export { FlyOut }; // O export default FlyOut;
```

**Paso 2: Usar el componente compuesto (en otro archivo, ej: `FlyoutMenu.js`)**

```javascript
import React from "react";
// Solo necesitamos importar FlyOut, ¡los demás vienen con él!
import { FlyOut } from "./FlyOut";

export default function FlyoutMenu() {
  return (
    // Usamos FlyOut como contenedor principal
    <FlyOut>
      {/* Usamos los componentes "adjuntos" */}
      <FlyOut.Toggle /> {/* El botón para abrir/cerrar */}
      <FlyOut.List>
        {" "}
        {/* La lista que aparecerá */}
        <FlyOut.Item>Editar</FlyOut.Item> {/* Opción 1 */}
        <FlyOut.Item>Eliminar</FlyOut.Item> {/* Opción 2 */}
      </FlyOut.List>
    </FlyOut>
  );
}
```

**Explicación del ejemplo:**

- Al hacer `FlyOut.Toggle = Toggle;`, estamos diciendo: "La propiedad llamada `Toggle` en el objeto `FlyOut` es el componente `Toggle` que definimos".
- Cuando usamos `<FlyOut.Toggle />`, React entiende que debe renderizar nuestro componente `Toggle`.
- Lo más genial es que solo necesitamos importar `FlyOut`. ¡Todo lo demás (`Toggle`, `List`, `Item`) viene "pegado" a él!
- El componente `FlyoutMenu` no necesita manejar ningún estado (`open` o `toggle`). Toda esa lógica está encapsulada dentro de `FlyOut`.

#### 3. **Notas o advertencias:**

- Esta técnica de adjuntar componentes como propiedades es muy común en librerías de UI y hace que el código sea muy legible y organizado.
- Puedes ver un ejemplo funcional aquí: [CodeSandbox - Context API](https://codesandbox.io/embed/provider-pattern-2-ck29r)

---

## E - Implementación Alternativa: `React.Children.map` y `React.cloneElement`

#### 1. **Definición:**

Hay otra forma de lograr algo similar sin usar Context API directamente para pasar las props. El componente padre (`FlyOut`) puede "inspeccionar" a sus hijos directos usando `React.Children.map` y luego "clonarlos" usando `React.cloneElement`, añadiéndoles las props necesarias (`open`, `toggle`) en el proceso.

#### 2. **Ejemplo:**

```javascript
import React, { useState } from "react";

export function FlyOut(props) {
  const [open, setOpen] = useState(false);
  const toggle = () => setOpen(!open);

  return (
    <div>
      {/* Mapeamos sobre los hijos directos */}
      {React.Children.map(props.children, (child) =>
        // Clonamos cada hijo y le pasamos 'open' y 'toggle' como props
        React.cloneElement(child, { open, toggle })
      )}
    </div>
  );
}

// Los componentes hijos (Toggle, List, Item) ahora recibirían 'open' y 'toggle'
// directamente como props, en lugar de usar useContext.
// Ejemplo (simplificado):
// function Toggle({ open, toggle }) { /* ... usar open y toggle ... */ }
// function List({ open, children }) { /* ... usar open ... */ }

// La forma de adjuntarlos como propiedades (FlyOut.Toggle, etc.) y usarlos
// sería la misma que en el método con Context API.
```

**Explicación del ejemplo:**

- `React.Children.map(props.children, ...)`: Recorre cada hijo inmediato que se le pasa a `<FlyOut>`.
- `React.cloneElement(child, { open, toggle })`: Crea una copia de cada `child` pero le añade (o sobrescribe) las props `open` y `toggle`.
- Los componentes hijos (`Toggle`, `List`) ya no necesitarían `useContext`. Simplemente recibirían `open` y `toggle` como cualquier otra prop.

#### 3. **Notas o advertencias:**

- **¡Cuidado!** Este método tiene una limitación importante: **solo funciona con los hijos directos**. Si envuelves un `FlyOut.Toggle` dentro de un `<div>` u otro componente, ¡dejará de recibir las props `open` y `toggle`! (Ver sección G - Cons).
- También existe un pequeño riesgo de **colisión de nombres de props**. Si un hijo ya tenía una prop llamada `open` o `toggle`, `cloneElement` la sobrescribirá.
- Puedes ver un ejemplo de esta técnica aquí: [CodeSandbox - Children.map](https://codesandbox.io/embed/provider-pattern-2-j9l1k) (Aunque este ejemplo específico aún parece usar Context en los hijos, la idea principal es que `cloneElement` les pasa las props).

---

## F - Pros (Ventajas) del Compound Component Pattern

#### 1. **Estado encapsulado:**

Los componentes manejan su propio estado interno (`open`, `toggle`). Quien usa `<FlyOut>` no tiene que preocuparse por eso. ¡Menos lío para ti!

#### 2. **Uso sencillo y declarativo:**

La forma de usarlo (`<FlyOut><FlyOut.Toggle />...</FlyOut>`) es muy clara y describe bien la estructura, sin necesidad de pasar manualmente el estado de un lado a otro.

#### 3. **Importaciones limpias:**

Gracias a la técnica de adjuntar componentes como propiedades (`FlyOut.Toggle`), solo necesitas importar el componente principal (`FlyOut`). ¡Mantiene tus importaciones ordenadas!

#### 4. **Flexibilidad:**

Puedes decidir qué partes del componente compuesto usar y en qué orden (dentro de lo lógico, claro).

---

## G - Cons (Desventajas y Cuidados) del Compound Component Pattern

#### 1. **Limitación de anidación (con `React.Children.map`):**

Si usas el método de `React.Children.map` y `React.cloneElement` (Sección E), **solo los hijos directos** reciben las props inyectadas. Si pones un `<div>` u otro componente entre `FlyOut` y `FlyOut.Toggle`, la conexión se rompe.

```javascript
// ¡ESTO ROMPE EL MÉTODO de React.Children.map!
<FlyOut>
  <div>
    {" "}
    {/* Este div extra impide que Toggle reciba las props */}
    <FlyOut.Toggle />
    <FlyOut.List>...</FlyOut.List>
  </div>
</FlyOut>
```

- **Nota:** La implementación con **Context API (Secciones B, C, D) no tiene este problema**, ya que el contexto está disponible para cualquier descendiente, sin importar cuántos niveles de anidación haya.

#### 2. **Posible colisión de Props (con `React.cloneElement`):**

Cuando usas `React.cloneElement` (en el método de `React.Children.map`), las props que inyectas (`open`, `toggle`) se fusionan superficialmente (`shallow merge`) con las props existentes del hijo. Si el hijo ya tenía una prop con el mismo nombre, ¡su valor original será sobrescrito! Hay que tener cuidado con los nombres.

#### 3. **Puede ser un poco "mágico":**

Para alguien que no conoce el patrón, podría no ser inmediatamente obvio cómo `FlyOut.Toggle` obtiene acceso al estado de `FlyOut`, especialmente con el método de Context API. Pero una vez que entiendes el patrón, ¡es genial!
