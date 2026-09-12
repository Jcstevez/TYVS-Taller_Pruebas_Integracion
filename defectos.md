# Registro de Defectos — Registraduría

Este documento recopila defectos detectados durante la ejecución de las pruebas de integración (H2) y unitarias con mocks del proyecto Registraduría.

---

## Formato 1: Lista detallada (narrativa)

### Defecto 01 — Edad negativa mal clasificada como menor de edad

- **Capa afectada:** Dominio (`Registry.registerVoter`)
- **Caso de prueba:** `RegistryWithMockTest.shouldReturnInvalidAgeWhenAgeIsNegative`
- **Entrada:** `Person(name="Imposible", id=12, age=-1, gender=UNIDENTIFIED, alive=true)`
- **Resultado esperado:** `INVALID_AGE`
- **Resultado obtenido (antes del fix):** `UNDERAGE`
- **Causa probable:** La validación evaluaba `age < MIN_AGE` sin distinguir entre una edad negativa (dato imposible) y una edad menor pero válida (ej: 17 años).
- **Tipo de prueba:** Unitaria (mock)
- **Estado:** Resuelto — se agregó la constante `INVALID_AGE` y la regla `age < 0 || age > MAX_AGE` evaluada ANTES que la regla de minoría de edad.
- **Prioridad:** Alta

> **Valor límite asociado:** la frontera entre las dos clases de equivalencia es la edad `0` (frontera inferior válida = recién nacido, `-1` = dato imposible). Cubierta también por `shouldReturnUnderageWhenAgeIsZero`.

---

### Defecto 02 — Duplicado no detectado si solo se prueba con mock

- **Capa afectada:** Infraestructura (`RegistryRepository`) / cobertura de pruebas
- **Caso de prueba:** Comparación entre `RegistryWithMockTest.shouldReturnDuplicatedWhenRepoSaysExists` y `RegistryIT.shouldPersistValidVoterAndRejectDuplicates`
- **Entrada:** Dos personas con el mismo `id` (ej: `id=100`)
- **Resultado esperado:** El segundo registro debe devolver `DUPLICATED`
- **Observación:** En el test con mock, el `DUPLICATED` lo simula el desarrollador con `when(repo.existsById(7)).thenReturn(true)` — **no prueba que la base de datos real detecte el duplicado**. Solo `RegistryIT` (con H2 real) valida que la restricción de unicidad funciona de verdad en la capa de persistencia.
- **Causa probable:** Confiar únicamente en pruebas con mocks daría falsa confianza: la lógica de `Registry` podría estar bien pero el `RegistryRepository` podría no tener la restricción de unicidad configurada, y ningún mock lo detectaría.
- **Tipo de prueba:** Integración (H2) — es la que realmente cierra este riesgo
- **Estado:** Resuelto — cubierto por `RegistryIT`, que sí usa la base de datos real
- **Prioridad:** Media

---

## Formato 2: Tabla de defectos (bug tracking)

| ID | Caso de Prueba | Entrada | Resultado Esperado | Resultado Obtenido | Causa Probable | Estado |
|----|----------------|---------|---------------------|----------------------|-----------------|--------|
| 01 | `shouldReturnInvalidAgeWhenAgeIsNegative` | `age=-1` | `INVALID_AGE` | `UNDERAGE` (antes del fix) | No distinguía edad imposible de menor de edad | Resuelto |
| 02 | Mock vs H2 en duplicados | `id` repetido | `DUPLICATED` verificado contra BD real | Mock solo simula el resultado | Mock no valida restricción real de unicidad | Resuelto |

---

## Convenciones de Estado

- **Abierto** → El defecto aún no se corrige.
- **En progreso** → El defecto está siendo trabajado.
- **Resuelto** → El defecto fue corregido y validado con pruebas.