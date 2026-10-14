# Repaso

## 🛒 Carrito + Reducer

1. **Agregar un producto al carrito**
   - ¿Desde dónde se dispara la acción?
   - ¿Qué `action` debería enviarse?
   - ¿Qué hace el reducer?
   - ¿Cómo llega el nuevo estado a la interfaz?

2. **Agregar nuevamente un producto que ya está en el carrito**
   - ¿Qué debería detectar el reducer?
   - ¿Se agrega otro elemento o aumenta `quantity`?
   - ¿Qué estado debería devolver?

3. **Eliminar un producto del carrito**
   - ¿Qué información necesita la acción?
   - ¿Qué responsabilidad tiene el reducer?
   - ¿Qué componentes deberían actualizarse?

4. **Calcular el total del carrito**
   - ¿Dónde conviene obtener este dato?
   - ¿Es estado real o estado derivado?
   - ¿Qué ocurre cuando cambia la cantidad de un producto?

5. **Vaciar el carrito después de confirmar una orden**
   - ¿En qué momento debería vaciarse?
   - ¿Qué pasa si la creación de la orden falla?
   - ¿Qué acciones/contextos participan?

## 👤 Auth + Firebase

6. **Un usuario intenta realizar el checkout sin estar autenticado**
   - ¿Dónde detectarías la situación?
     - UI/Login/Auth/ProtectedRoute/Firestore Rules
   - ¿Qué contexto proporciona esa información?
   - ¿Qué debería ocurrir con el carrito?

7. **Registro de un nuevo usuario**
   - ¿Qué ocurre primero: Firebase Authentication o Firestore?
   - ¿Qué información se guarda en cada lugar?
   - ¿Cómo relacionamos el usuario autenticado con su documento en Firestore?

8. **Inicio de sesión y recuperación de la sesión**
   - ¿Cómo sabe React que el usuario ya está autenticado?
   - ¿Qué ocurre al recargar la página?
   - ¿Qué responsabilidad tiene `AuthProvider`?

9. **Cerrar sesión**
   - ¿Qué debería cambiar en Firebase?
   - ¿Qué debería cambiar en el estado de React?
   - ¿Qué partes de la aplicación deberían reaccionar?

## 📦 Productos + Firestore

10. **Mostrar el catálogo**
    - ¿De dónde vienen los productos?
    - ¿Qué servicio consulta Firestore?
    - ¿Dónde se almacena el resultado?
    - ¿Cómo llega finalmente al componente?

11. **Un producto modifica su stock**
    - ¿Quién puede realizar esa modificación?
    - ¿Dónde se realiza realmente la actualización?
    - ¿Qué papel cumplen las reglas de Firestore?

12. **Un usuario intenta modificar directamente el precio de un producto**
    - ¿Dónde debería bloquearse?
    - ¿Puede confiarse solamente en una validación del frontend?
    - ¿Qué debería establecer Firestore?

### 🔎 Filtros y catálogo

13. **Filtrar productos por categoría**
    - ¿Dónde debe vivir el estado del filtro?
    - ¿Se modifica la fuente original de productos?
    - ¿Cómo obtenemos la lista filtrada?

14. **Buscar productos por nombre**
    - ¿Qué ocurre cuando el usuario escribe?
    - ¿Qué datos necesitamos?
    - ¿El filtro se realiza en frontend o consultando Firestore?

15. **Combinar búsqueda + categoría + ordenamiento**
    - ¿En qué orden aplicarías cada operación?
    - ¿Qué estado necesitas mantener?
    - ¿Qué información debería permanecer sin modificar?

---

## Ejercicios Tipo

### 🟡 1. Persistencia del Carrito/Favoritos 💾

- ¿Dónde almacenarían la información? BBDD
  - user: { id, name, email, cart: [ { p1, quantity}, {p2, quantity } ] }
  - Colección propia: cart: { id, uid, products: [ { p1, quantity}, {p2, quantity } ] }
- Servicio:
  - ¿Cuándo guardarían el carrito? Cuando el carrito cambie
- ¿Cuándo lo recuperarían? Login

### 🟢 2. Reviews de productos ⭐

- **Nueva funcionalidad:**
- Los usuarios autenticados pueden dejar una valoración de 1 a 5 estrellas y un comentario sobre un producto.
- Las reviews deben mostrarse en el detalle del producto.

1. Types

- products: [ { id, name, review: { id, pid, rating: number, comentario: string, ... } }]

2. Guardar en BBDD: Colección nueva ó en Colección de productos
3. Servicio

- ¿Qué operaciones en Firestore necesitamos?
  - Firestore Rules: Usuario customer que compró el producto - Orden de compra
  - CRUD
  - Comentarios indebidos: IA
- ¿Cómo calculamos el promedio de estrellas?

4. UI: Formulario controlado

UI CONTEXT SERVICE FireStore

### 🟡 3. Ordenar productos

- **Nueva funcionalidad:**
- El usuario puede ordenar el catálogo por precio tanto de forma ascendente como descendente.

- ¿Dónde vive el criterio de ordenamiento?
  - Servicio/Firestore Indexado
- ¿Es estado?
  - ProductsContext
- ¿Se realiza sobre los productos originales?
  - Snapshot
- ¿El ordenamiento se hace en frontend o Firestore?
  - Firestore (Indexado)
- ¿Cómo combinarlo con los filtros existentes?
  - Indexado

### 🔴 4. Cancelar una orden

- **Nueva funcionalidad:**
- Un usuario puede cancelar una orden mientras se encuentre en estado `pendiente`.
- Una orden `en proceso` o `completada` no puede cancelarse.

- ¿Dónde validamos el estado?
  - Firestore - orden
- ¿Quién tiene permitido cancelar?
  - Solamente el dueño de la orden: Front/FirestoreRules
- ¿Qué pasa con el stock?
  - Restaurar Stock ante cancelación
- ¿Deberíamos devolver las unidades al stock?
  - SI

### 🔥 5. "Comprar nuevamente"

- **Nueva funcionalidad:**
- En el historial de órdenes, el usuario tiene un botón **"Comprar nuevamente"** que agrega al carrito los productos de una orden anterior.

Determinar:

- ¿De dónde obtenemos los productos?
  - Firestore - Order
- ¿Qué información necesitamos de la orden?
  - id de productos
- ¿Usamos el reducer existente?
  - Si
- ¿Qué ocurre si un producto ya está en el carrito?
- ¿Qué ocurre si un producto ya no existe?
- ¿Qué ocurre si ahora tiene menos stock?
- ¿Qué pasa si cambió el precio?
- ¿Dónde hacemos esas validaciones?

### 🟢 6. Productos recientemente vistos 👀

- **Nueva funcionalidad:**
- El usuario puede consultar los últimos 5 productos que visitó.

Definir:

- ¿Dónde almacenamos los productos vistos?
  - BBDD: user: { id, email, ultimosVistos: [ pid1, pid2, pid3 ] }
  - Servicio
- ¿Necesitamos un nuevo Context?
- ¿Guardamos el producto completo o solamente sus `id`?
- ¿Qué ocurre si visita nuevamente un producto?
- ¿Qué ocurre al recargar la página?
- ¿Lo persistimos en Firestore o en el navegador?

---
