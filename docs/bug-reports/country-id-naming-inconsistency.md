# Bug: Inconsistencia de naming `countryId`/`country_id` — Análisis y Plan de Corrección

> **Fecha:** 2026-07-05  
> **Severidad:** Alta — afecta creación de crushes (funcionalidad core)  
> **Tipo:** Cross-cutting (backend + iOS)  
> **Actualización 2026-07-XX:** El comportamiento de `POST /api/crushes` cambió arquitectónicamente. La app iOS ahora genera el UUID client-side antes de enviar el POST. El backend acepta el UUID, valida e inserta (idempotencia: 200 OK si ya existe). El POST no devuelve body. Este bug report queda **parcialmente obsoleto**: el naming mismatch sigue siendo relevante para otros endpoints, pero el problema específico de la respuesta de creación ya no aplica.

---

## 1. El error en el screenshot

```
We couldn't save it
We couldn't process the server response. Issue with field 'countryId'.
Server message: {"data":{"message":"Crush created successfully",
"id":"993da567-8839-426e-8d77-d0a22d542077"}}
```

**Diagnóstico:** El endpoint `POST /api/crushes` devuelve un objeto incompleto (`message` + `id`), pero el iOS espera un objeto `CrushViewData` completo que incluye `countryId` como campo obligatorio. El decoder falla porque `countryId` no está presente en la respuesta JSON.

**Causa raíz:** No hay modelo de respuesta separado para la creación de crush. El backend devuelve un success message genérico; el iOS intenta decodear ese JSON como un `CrushViewData` completo.

---

## 2. Problema subyacente: inconsistencia de naming en toda la API

### 2.1 Backend: dos convenciones coexisten

| Lugar | Convención | Campo |
|---|---|---|
| `CrushResource.php` | ✅ snake_case | `country_id` |
| `CrushSummaryResource.php` | ✅ snake_case | `country_id` |
| `DatabaseQueries.php` (arrays manuales) | ❌ camelCase | `countryId` |
| `PostCrushRequest` (validación entrada) | ✅ snake_case | `country_id` |
| `CrushController::store()` (respuesta) | ✅ obsoleto — ahora no devuelve body (client-side UUID) | N/A — la app ya conoce el UUID |

### 2.2 iOS: espera camelCase en todo

| Modelo | Propiedad Swift | CodingKeys | Espera en JSON |
|---|---|---|---|
| `CrushViewData` | `countryId: Int` | `case countryId` (sin mapping) | `countryId` (camelCase) ❌ |
| `CrushRowData` | `countryId: Int` | `case countryId` (sin mapping) | `countryId` (camelCase) ❌ |
| `TimelineBundleItemModel.Crush` | `countryId: Int` | implícito | `countryId` (camelCase) ❌ |
| `ExperienceTimelineItem.Crush` | `countryId: Int` | implícito | `countryId` (camelCase) ❌ |
| `RelationshipTimelineItem.Crush` | `countryId: Int` | implícito | `countryId` (camelCase) ❌ |
| `MapCountryDetailsAndData` | `countryId: Int` | `case countryId` (sin mapping) | `countryId` (camelCase) ❌ |

### 2.3 iOS: envío correcto, recepción incorrecta

- **Request** (iOS → Backend): `country_id` (snake_case) ✅ correcto.
- **Response** (Backend → iOS): Backend devuelve `country_id` (snake_case) en Resources, pero iOS espera `countryId` (camelCase). ❌ esto falla.

---

## 3. Por qué el error aparece AHORA

> **⚠️ Obsoleto tras client-side UUID generation:** La app iOS ahora genera el UUID del crush antes de enviar el POST. El endpoint `POST /api/crushes` **ya no devuelve body**. Si el UUID ya existe, el backend devuelve 200 OK (idempotencia). La app ya conoce el ID y no necesita parsear respuesta.
>
> El análisis siguiente describe el estado del bug **antes** del cambio arquitectónico. Se conserva como referencia histórica.

El endpoint `POST /api/crushes` **no devuelve el objeto crush completo**. Solo devuelve:
```json
{"data": {"message": "Crush created successfully", "id": "..."}}
```

El iOS decodifica esto como `APIResponse<CrushViewData>`. Como `CrushViewData` tiene `countryId` como campo obligatorio, el decoder falla con `keyNotFound`.

**Pero:** Si el endpoint devolviera el objeto completo usando `CrushResource` (que ya usa `country_id` en snake_case), el decoder **también** fallaría porque iOS espera `countryId` (camelCase), no `country_id` (snake_case).

Hay **dos problemas** que se solapan:
1. **Bug inmediato:** El endpoint de creación no devuelve el objeto completo.
2. **Bug subyacente:** Si devolviera el objeto completo, el naming mismatch entre `country_id` (backend) y `countryId` (iOS) haría que TODOS los endpoints de crush fallen.

---

## 4. Archivos que necesitan cambio

### 4.1 Backend (api/)

| Archivo | Línea | Cambio necesario | Prioridad |
|---|---|---|---|
| `CrushController.php` (store) | ~209-241 | Devolver objeto crush completo usando `CrushResource::make($crush)` en lugar de `['message' => ..., 'id' => ...]` | Alta |
| `DatabaseQueries.php` | 190, 221, 350, 416 | Cambiar `countryId` → `country_id` en arrays manuales | Alta |
| `DatabaseQueries.php` | Verificar todos los arrays manuales | Buscar otros campos camelCase (`genderId`, `duckyId`, `matchStatusId`, `currentStateId`, etc.) | Alta |
| `storage/responses/crushes.index.new.json` | Múltiples | Actualizar ejemplos a snake_case | Media |
| `.scribe/endpoints/04.yaml` | Múltiples | Regenerar documentación Scribe | Media |
| `mdl-api-postman-collection.json` | Múltiples | Actualizar colección Postman | Baja |

### 4.2 iOS (ios/)

| Archivo | Línea | Cambio necesario | Prioridad |
|---|---|---|---|
| `HTTPClient.swift` | 313 | Añadir `decoder.keyDecodingStrategy = .convertFromSnakeCase` al `JSONDecoder` | Alta |
| `HTTPClient.swift` | Verificar | Asegurar que no hay modelos que dependan de camelCase y se rompan con `convertFromSnakeCase` | Alta |
| `CrushViewData.swift` | 54 | Eliminar `CodingKeys` personalizado si `convertFromSnakeCase` lo hace innecesario, O mapear `case countryId = "country_id"` | Alta |
| `CrushRowData.swift` | 48 | Eliminar `CodingKeys` personalizado, O mapear `case countryId = "country_id"` | Alta |
| `CrushService.swift` | 16-29 | Verificar que `createCrush` sigue funcionando con respuesta completa | Alta |
| `UITestingDataProvider.swift` | Múltiples | Actualizar mocks de respuesta a `country_id` (snake_case) | Alta |
| `TimelineDecodingTests.swift` | 60 | Verificar que el test sigue pasando (ya usa `convertFromSnakeCase`) | Media |
| `MapCountryDetailsAndData.swift` | 13 | Mapear `case countryId = "country_id"` | Media |

### 4.3 Documentación (product/)

| Documento | Cambio |
|---|---|
| `product/docs/plans/active/email-verification/README.md` | Referencia: esta inconsistencia debe resolverse antes de que la feature de verificación de email se implemente |
| `product/docs/decisions/` (nuevo) | Documentar la decisión de usar snake_case en toda la API y el retroactive fix de este bug |

---

## 5. Solución recomendada

### Paso 1: Corregir el decoder de iOS (cambio de una línea, impacto global)

En `ios/MDLApp/Shared/Services/Networking/HTTPClient.swift`, línea 313:

```swift
// ANTES:
let decoder = JSONDecoder()

// DESPUÉS:
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

**Impacto:** Todos los modelos que usan `countryId` en Swift se mapean automáticamente desde `country_id` en JSON. No hay que tocar cada modelo individualmente.

**Riesgo:** Si hay modelos que esperan camelCase y el backend los devuelve así, `convertFromSnakeCase` no afecta (por ejemplo, `myField` en JSON se mapea a `myField` en Swift, no a `my_field`). Pero si hay un campo como `URL` o `UUID`, `convertFromSnakeCase` no lo toca. Los únicos riesgos son campos con guiones bajos que NO deberían convertirse, pero eso es raro en este codebase.

**Verificación:** Ejecutar `TimelineDecodingTests` (que ya usa `convertFromSnakeCase`) para confirmar que el decoder funciona con los modelos existentes.

### Paso 2: Corregir el endpoint de creación de crush (backend)

> **⚠️ Obsoleto tras client-side UUID generation:** Este paso ya no aplica. La app iOS ahora genera el UUID del crush antes de enviar el POST. El endpoint `POST /api/crushes` acepta el UUID en el payload, valida e inserta (idempotencia: 200 OK si ya existe), y **no devuelve body**. El iOS ya no necesita parsear un ID de respuesta.
>
> Se conserva el análisis original como referencia histórica:

~~En `api/app/Http/Controllers/CrushController.php`, método `store()`:~~

```php
// COMPORTAMIENTO ANTERIOR (ya no aplica):
return response()->json([
    'data' => [
        'message' => 'Crush created successfully',
        'id' => $crushEntity->id()
    ]
], 201);

// PROPUESTA ANTERIOR (ya no aplica):
$crush = Crush::find($crushEntity->id());
return response()->json([
    'data' => new CrushResource($crush)
], 201);
```

**Nuevo comportamiento:**
```php
// La app envía el UUID generado client-side en el payload.
// El backend valida e inserta (o devuelve 200 OK si ya existe).
// No hay body en la respuesta.
```

### Paso 3: Corregir `DatabaseQueries.php` (backend)

Cambiar todos los `countryId` → `country_id` en arrays manuales. También buscar otros campos camelCase (`genderId`, `duckyId`, `matchStatusId`, `currentStateId`, etc.) y convertirlos a snake_case.

### Paso 4: Actualizar mocks de UI Testing (iOS)

En `UITestingDataProvider.swift`, cambiar todos los `countryId` → `country_id` en los mocks de respuesta JSON.

### Paso 5: Verificar todos los modelos de iOS

Buscar todos los `CodingKeys` personalizados en modelos que reciben JSON del backend. Si un modelo ya tiene `CodingKeys` con `countryId = "country_id"`, el `convertFromSnakeCase` lo ignora (el mapping explícito gana). Si no tiene mapping, `convertFromSnakeCase` lo maneja automáticamente.

**Decisión:** Si `convertFromSnakeCase` se aplica globalmente, los `CodingKeys` personalizados que hacen mapping manual son redundantes pero no dañinos. Se pueden dejar o eliminar. La decisión de eliminarlos es de refactoring futuro.

### Paso 6: Tests

- [ ] `POST /api/crushes` devuelve objeto completo con `country_id`.
- [ ] iOS parsea `country_id` correctamente como `countryId` en Swift.
- [ ] `TimelineDecodingTests` sigue pasando.
- [ ] Todos los mocks de UI Testing usan `country_id`.
- [ ] End-to-end: crear un crush desde el simulador → no hay error de `countryId`.

---

## 6. Notas para el implementador

### Backend (api/)
- Usar MCP `search_graph` para encontrar TODOS los arrays manuales en `DatabaseQueries.php` que usen camelCase.
- Imitar `CrushResource` para devolver el objeto completo en `store()`.
- Los Resources (`CrushResource`, `CrushSummaryResource`) ya usan snake_case — no tocarlos.

### iOS (ios/)
- Una sola línea en `HTTPClient.swift` arregla el naming de TODOS los modelos de respuesta.
- Verificar que no hay modelos que dependan de camelCase y se rompan (ej. campos que en el backend se envían en camelCase por alguna razón legítima).
- Los mocks de `UITestingDataProvider` deben actualizarse para reflejar el contrato real del backend.

### Cross-cutting
- Este bug debe resolverse **antes** de implementar la feature de verificación de email, porque el email-verification también agregará nuevos endpoints y modelos que seguirán la convención snake_case.
- Documentar esta decisión en `product/docs/decisions/` como "API Naming Convention: snake_case everywhere".

---

## 7. Decisiones pendientes de revisión humana

1. **¿Usar `convertFromSnakeCase` globalmente en `HTTPClient` o mapear campo por campo?**  
   Recomendación: `convertFromSnakeCase` global. Es una línea, arregla todo, y es consistente con la decisión de usar snake_case en la API. Los modelos que ya tienen `CodingKeys` personalizados con mapping explícito seguirán funcionando (el mapping explícito gana sobre `convertFromSnakeCase`).

2. **¿Qué hacer con los `CodingKeys` personalizados existentes?**  
   Recomendación: Dejarlos por ahora. Eliminarlos es refactoring que puede hacerse en una fase posterior. No son dañinos.

3. **¿Hay otros campos camelCase en `DatabaseQueries.php`?**  
   El explorador encontró `countryId`. Pero `DatabaseQueries.php` tiene muchos arrays manuales. Necesita una revisión completa de todos los campos.

---

## 8. Alcance completo: TODOS los campos camelCase encontrados

### 8.1 Backend (`DatabaseQueries.php`) — arrays manuales con camelCase

| Línea | Campo camelCase | Debería ser |
|---|---|---|
| 190 | `countryId` | `country_id` |
| 195 | `stateId` | `state_id` |
| 197 | `startDate` | `start_date` |
| 204 | `satisfactionId` | `satisfaction_id` |
| 221 | `countryId` | `country_id` |
| 241 | `stateId` | `state_id` |
| 243 | `startDate` | `start_date` |
| 250 | `satisfactionId` | `satisfaction_id` |
| 350 | `countryId` | `country_id` |
| 416 | `countryId` | `country_id` |
| 424 | `favCountries` | `fav_countries` |
| 425 | `lovCountries` | `lov_countries` |
| 426 | `popTraits` | `pop_traits` |

### 8.2 Backend (`routes/api.php`) — parámetros de ruta con camelCase

| Ruta | Parámetro | Debería ser |
|---|---|---|
| `GET /api/crushes/{crushId}/view` | `crushId` | `crush_id` |
| `PATCH /api/crushes/{crushId}` | `crushId` | `crush_id` |
| `PATCH /api/crushes/{crushId}/bundle` | `crushId` | `crush_id` |
| `DELETE /api/crushes/{crushId}` | `crushId` | `crush_id` |
| `GET /api/experiences/{crushId}` | `crushId` | `crush_id` |
| `GET /api/experiences/{experienceId}/view` | `experienceId` | `experience_id` |
| `POST /api/experiences/{experienceId}/relationship` | `experienceId` | `experience_id` |
| `PATCH /api/experiences/{experienceId}` | `experienceId` | `experience_id` |
| `DELETE /api/experiences/{experienceId}` | `experienceId` | `experience_id` |
| `POST /api/relationships/{relationshipId}` | `relationshipId` | `relationship_id` |
| `POST /api/relationships/{relationshipId}/experience` | `relationshipId` | `relationship_id` |
| `PATCH /api/relationships/{relationshipId}` | `relationshipId` | `relationship_id` |
| `DELETE /api/relationships/{relationshipId}` | `relationshipId` | `relationship_id` |

> **Nota:** Los parámetros de ruta en Laravel son nombres de variables PHP, no claves JSON. Cambiarlos a snake_case en la URL puede ser breaking para clientes existentes. Sin embargo, para consistencia con la convención de la API, deberían evaluarse. Los parámetros de ruta no afectan el JSON de respuesta, solo la URL.

### 8.3 iOS (`UITestingDataProvider.swift`) — mocks con camelCase

| Líneas | Campos camelCase en mocks | Deberían ser |
|---|---|---|
| 38-41 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 48-51 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 58-61 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 120-123 | `genderId`, `duckyId`, `matchStatusId`, `currentStateId` | `gender_id`, `ducky_id`, `match_status_id`, `current_state_id` |
| 166-169 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 175 | `stateId` | `state_id` |
| 190-193 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 199 | `stateId` | `state_id` |
| 221-224 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 245-248 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 254 | `stateId` | `state_id` |
| 276-279 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |
| 285 | `stateId` | `state_id` |
| 307-310 | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | `match_status_id`, `current_state_id`, `gender_id`, `ducky_id` |

### 8.4 Tests (`tests/`) — aserciones con camelCase

| Archivo | Línea | Campo camelCase | Debería ser |
|---|---|---|---|
| `TimelineIntegrationTest.php` | 300 | `stateId` | `state_id` |
| `RelationshipStateTransitionTest.php` | 109, 161 | `stateId` | `state_id` |
| `RelationshipExperienceLinkTest.php` | 221 | `stateId` | `state_id` |
| `JsonStructureContractTest.php` | 74 | `genderId` | `gender_id` |

> **Nota:** Los tests que usan `stateId` en `error.fields` parecen validar la estructura de errores de validación. Si el backend devuelve errores con `state_id` (snake_case), los tests deben actualizarse. Los Controllers (`RelationshipController.php`, `ExperienceController.php`) ya leen `state_id` del request, pero los errores de validación pueden devolver `stateId` si el Form Request usa camelCase internamente.

### 8.5 Scribe documentation — ejemplos de respuesta con camelCase

| Archivo | Campos camelCase en documentación | Deberían ser |
|---|---|---|
| `.scribe/endpoints/04.yaml` | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | snake_case |
| `storage/responses/crushes.index.new.json` | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | snake_case |
| `storage/framework/views/47b615cd...php` (cached) | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | snake_case |
| `resources/views/scribe/index.blade.php` | `matchStatusId`, `currentStateId`, `genderId`, `duckyId` | snake_case |

> **Nota:** La documentación de Scribe y los ejemplos de `storage/responses/` están desactualizados. Regenerar Scribe tras corregir el backend.

---

## 9. Plan de corrección ampliado

### Fase 1: Backend — `DatabaseQueries.php` (alta prioridad)

Cambiar TODOS los campos camelCase en arrays manuales a snake_case. Archivo crítico porque devuelve el timeline y el mapa, que son funcionalidades core.

### Fase 2: Backend — `CrushController::store()` (alta prioridad)

Devolver el objeto crush completo usando `CrushResource` en lugar de `['message' => ..., 'id' => ...]`.

### Fase 3: iOS — `HTTPClient.swift` decoder (alta prioridad)

Una línea: `decoder.keyDecodingStrategy = .convertFromSnakeCase`. Esto arregla TODOS los modelos de respuesta de golpe.

### Fase 4: iOS — `UITestingDataProvider.swift` (alta prioridad)

Actualizar todos los mocks de respuesta a snake_case. Esto es crítico porque los tests de UI usan estos mocks y si reflejan camelCase, los tests pasan pero la app real falla.

### Fase 5: Backend — Tests (media prioridad)

Actualizar `TimelineIntegrationTest`, `RelationshipStateTransitionTest`, `RelationshipExperienceLinkTest`, `JsonStructureContractTest` para usar snake_case en aserciones.

### Fase 6: Backend — Documentación Scribe (baja prioridad)

Regenerar Scribe y actualizar ejemplos de respuesta en `storage/responses/`.

### Fase 7: Backend — Parámetros de ruta (evaluar)

Los parámetros de ruta en `routes/api.php` (`crushId`, `experienceId`, `relationshipId`) son camelCase. Para consistencia completa, evaluar cambiarlos a snake_case. **Riesgo:** URLs con camelCase pueden estar hardcodeadas en el iOS. Verificar antes de cambiar. Los parámetros de ruta no afectan el JSON, así que esto es cosmético para la URL.

---

## 10. Impacto en features activas

### Vault E2EE
- **Sin impacto directo.** El Vault no consume endpoints de crush, timeline, ni mapa. Pero el fix del decoder de iOS (`convertFromSnakeCase`) es global y afecta a todos los modelos. Verificar que los modelos del Vault no se rompen.

### Email Verification
- **Sin impacto directo.** Los nuevos endpoints de email-verification deben diseñarse con snake_case desde el inicio (ya está en el plan). El decoder global de iOS se beneficiará automáticamente.

---

## 11. Checklist de verificación final (end-to-end)

- [ ] Crear un crush desde el simulador → la app genera UUID client-side, envía POST, backend devuelve 200 OK (sin body). No hay error de `countryId`.
- [ ] Ver lista de crushes → `country_id` se parsea correctamente.
- [ ] Ver timeline → `state_id`, `satisfaction_id` se parsean correctamente.
- [ ] Ver mapa → `country_id`, `fav_countries`, `lov_countries`, `pop_traits` se parsean correctamente.
- [ ] Editar un crush → `country_id` se envía y recibe correctamente.
- [ ] Idempotencia: enviar el mismo UUID dos veces → backend devuelve 200 OK, no hay duplicado.
- [ ] Todos los tests de backend pasan (`make test`).
- [ ] Todos los tests de iOS pasan (`Cmd+U` en Xcode).
- [ ] Documentación Scribe regenerada y actualizada.
- [ ] Mocks de UI Testing actualizados a snake_case.
