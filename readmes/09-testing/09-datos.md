# RESUMEN TESTING

## FIXTURES:

Datos de prueba reutilizables que representan entidades de nuestra aplicación.

Permiten mantener datos consistentes entre diferentes tests y evitar repetir la creación de objetos manualmente.

## RENDER-WITH-PROVIDERS:

Función auxiliar para renderizar componentes dentro del árbol de Providers necesario para que funcionen correctamente.

También permite precargar estados iniciales, por ejemplo, un usuario autenticado, productos, carrito, etc.

De esta manera, los tests pueden simular diferentes escenarios de la aplicación.

## TEST DE REDUCER:

El Reducer tiene Funciones Puras: - Siempre devuelve el mismo resultado para los mismos argumentos. - No produce efectos secundarios, es decir, no modifica nada fuera de su propio ámbito ni depende de un estado externo que pueda cambiar.

Por lo tanto el Reducer: - No depende de React - No depende del DOM - No depende de Firebase - Recibe: Estado actual + Acción - Retorna: Nuevo estado

Por lo tanto, no necesitamos renderizar componentes ni utilizar React, testeamos únicamente la lógica de transformación del estado.

## TEST DE HOOKS:

Testeamos el comportamiento de un hook y las piezas de lógica que utiliza.

Testeamos reducer (si lo tiene), contexto y provider.

El objetivo es verificar su comportamiento desde la perspectiva del consumidor, sin necesidad de testear la UI.

## TEST DE COMPONENTES:

Testeamos el comportamiento observable del componente a través de la interfaz: 1. Interactuamos con el componente como lo haría un usuario: hacer clic, escribir, seleccionar opciones, etc. 2. Verificamos que la interfaz muestre el resultado esperado después de cada interacción.

No testeamos directamente los detalles internos de implementación, sino el comportamiento visible para el usuario.

Si el componente utiliza Context y Reducer, una prueba exitosa puede validar que estas piezas funcionan correctamente en conjunto con la UI para el escenario probado.

## MSW (MOCK SERVICE WORKER):

Intercepta las peticiones HTTP que realiza la aplicación y devuelve respuestas controladas por el test.

Por ejemplo:
/api/presign → respuesta mockeada

Permite testear la comunicación con APIs sin depender de un backend real (S3).

## CHECKOUT:

Testeamos que el flujo de compra coordine correctamente las distintas responsabilidades: - Interacción del usuario: agregar un producto al carrito e iniciar la compra. - Estado del carrito: verificar los productos, las cantidades y el total. - Autenticación: comprobar que el usuario esté autenticado. - Construcción del pedido: verificar que los datos enviados sean correctos. - Llamada al servicio de órdenes: comprobar que se invoque con los datos esperados. - Actualización del estado: verificar que el carrito se vacíe después de completar la compra.

NO testeamos Firestore ni su comunicación real. El servicio de órdenes se mockea para aislar el flujo de compra de la base de datos.

El objetivo es comprobar que Checkout coordina correctamente la interacción entre la interfaz, el carrito, los contextos y el servicio de órdenes, desde la acción del usuario hasta la actualización del estado de la aplicación.

Importante: este test verifica la integración entre los componentes y sus dependencias dentro del flujo de compra, pero no garantiza que la persistencia en Firestore funcione correctamente.

# Datos Extra

## Instalación de Coverage

```bash
npm install -D @vitest/coverage-v8
```

## vite.config.ts

```ts
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import path from "path";
import { defineConfig } from "vitest/config";

// https://vite.dev/config/
export default defineConfig({
  plugins: [react(), tailwindcss()],

  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },

  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: "./src/test/setup.ts",

    coverage: {
      provider: "v8",
      reporter: ["text", "html"],
    },
  },
});
```

## .github/workflows/ci.yml En Raíz del Proyecto

```yml
name: CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      # 1. Clona repositorio
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Instala Node
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22

      # 3. Instala dependencias
      - name: Install dependencies
        run: npm ci

      # 4. Ejecuta tests
      - name: Run tests
        run: npm test

      # 5. Ejecuta coverage
      - name: Run coverage
        run: npm run test:coverage
```

## vercel.json En Raíz del Proyecto

```json
{
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

## src/test/checkoutFlow.test.tsx

```tsx
// src/test/checkoutFlow.test.tsx
import { ProductCard } from "@/components/common/ProductCard";
import { CheckoutPage } from "@/pages/CheckoutPage";
import { screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { vi } from "vitest";
import { customerUserFixture, productFixture } from "./fixtures";
import { renderWithProviders } from "./renderWithProviders";

// Importamos el servicio REAL para luego tiparlo como Mock:
import { createOrderFromCart } from "@/services/orders.service";

// Mock del módulo completo:
// Vitest "hoistea" vi.mock(), por eso usamos vi.fn() inline
// y luego accedemos a los mocks mediante vi.mocked(...)
vi.mock("@/services/orders.service", () => ({
  createOrderFromCart: vi.fn(),
}));

describe("Flow: checkout", () => {
  test("el usuario puede completar una compra", async () => {
    // 1. Convertimos el servicio en versión mockeada:
    const mockedCreateOrderFromCart = vi.mocked(createOrderFromCart);

    // 2. Configuramos respuesta falsa del backend:
    // El servicio real devolvería el ID generado por Firestore.
    mockedCreateOrderFromCart.mockResolvedValue("order_001");

    const user = userEvent.setup();

    // 3. Renderizamos:
    // - ProductCard => permite agregar al carrito
    // - CheckoutPage => consume el mismo CartContext
    renderWithProviders(
      <>
        <ProductCard product={productFixture} />
        <CheckoutPage />
      </>,
      {
        // Simulamos usuario logueado:
        preloadedUser: customerUserFixture,
      },
    );

    // 4. Agregar producto al carrito:
    await user.click(
      screen.getByRole("button", {
        name: /Agregar al carrito/i,
      }),
    );

    // 5. Verificar que el producto aparece en Checkout:
    expect(screen.getByText(`${productFixture.name} x 1`)).toBeInTheDocument();

    // 6. Ejecutar compra:
    await user.click(
      screen.getByRole("button", {
        name: /Comprar/i,
      }),
    );

    // 7. Verificamos que Firestore Service fue invocado:
    expect(mockedCreateOrderFromCart).toHaveBeenCalledTimes(1);

    // 8. Verificamos payload correcto:
    expect(mockedCreateOrderFromCart).toHaveBeenCalledWith({
      userId: customerUserFixture.uid,
      total: productFixture.price,
      items: [
        {
          productId: productFixture.id,
          name: productFixture.name,
          image: productFixture.image,
          description: productFixture.description,
          price: productFixture.price,
          quantity: 1,
          categoryId: productFixture.categoryId,
        },
      ],
    });

    // 9. Luego de comprar, el carrito debe quedar vacío:
    expect(screen.getByText(/El carrito está vacío/i)).toBeInTheDocument();
  });
});
```
