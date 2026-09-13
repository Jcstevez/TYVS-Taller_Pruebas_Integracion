\# Matriz de Pruebas — Registraduría



\## Pruebas de Integración (H2) y Unitarias con Mocks — Juan Camilo Estevez



| # | Clase | Método de test | Tipo | Escenario | Resultado esperado |

|---|-------|----------------|------|-----------|---------------------|

| 1 | `RegistryIT` | `shouldRegisterValidPerson` | Integración (H2) | Persona válida nueva | `VALID`, queda en BD |

| 2 | `RegistryIT` | `shouldPersistValidVoterAndRejectDuplicates` | Integración (H2) | Mismo `id` registrado dos veces | 1ra → `VALID`, 2da → `DUPLICATED` (detectado por la BD real) |

| 3 | `RegistryWithMockTest` | `shouldReturnDuplicatedWhenRepoSaysExists` | Unitaria (mock) | Mock dice que el `id=7` ya existe | `DUPLICATED`, nunca llama `save()` |

| 4 | `RegistryWithMockTest` | `shouldSaveWhenPersonIsValid` | Unitaria (mock) | Mock dice que `id=8` no existe | `VALID`, sí llama `save()` |

| 5 | `RegistryWithMockTest` | `shouldWrapPersistenceFailure` | Unitaria (mock) | `existsById(9)` lanza `SQLException` | La excepción se envuelve |

| 6 | `RegistryWithMockTest` | `shouldRejectUnderageWithoutTouchingRepository` | Unitaria (mock) | Persona de 17 años | `UNDERAGE`, sin tocar el repo |

| 7 | `RegistryWithMockTest` | `shouldReturnInvalidWhenPersonIsNull` | Unitaria (mock) | `person = null` | `INVALID`, cero interacciones |

| 8 | `RegistryWithMockTest` | `shouldReturnInvalidWhenIdIsNotPositive` | Unitaria (mock) | `id = 0` | `INVALID` |

| 9 | `RegistryWithMockTest` | `shouldReturnDeadWhenPersonIsNotAlive` | Unitaria (mock) | `alive = false` | `DEAD` |

| 10 | `RegistryWithMockTest` | `shouldReturnInvalidAgeWhenAgeIsNegative` | Unitaria (mock) | `edad = -1` (valor límite) | `INVALID\_AGE` |

| 11 | `RegistryWithMockTest` | `shouldReturnInvalidAgeWhenAgeExceedsMaximum` | Unitaria (mock) | `edad = MAX\_AGE + 1` | `INVALID\_AGE` |

| 12 | `RegistryWithMockTest` | `shouldReturnUnderageWhenAgeIsZero` | Unitaria (mock) | `edad = 0` (frontera) | `UNDERAGE` |

| 13 | `RegistryWithMockTest` | `shouldAcceptTheMaximumAge` | Unitaria (mock) | `edad = MAX\_AGE` (frontera superior) | `VALID` |



\## Pruebas de Sistema, Testcontainers y Contract Testing — Sofy Alejandra Prada Murillo



| # | Clase | Método de test | Tipo | Escenario | Resultado esperado |

|---|-------|----------------|------|-----------|---------------------|

| 14 | `RegistryControllerIT` | `shouldRegisterValidPerson` | Sistema (HTTP) | `POST /register`, persona válida | `200 OK`, body `"VALID"` |

| 15 | `RegistryControllerIT` | `shouldReturnDuplicatedWhenIdAlreadyRegistered` | Sistema (HTTP) | Mismo `id` dos veces por HTTP | `200 OK`, body `"DUPLICATED"` |

| 16 | `RegistryControllerIT` | `shouldReturnUnderageWhenPersonIsMinor` | Sistema (HTTP) | `age=17` | `200 OK`, body `"UNDERAGE"` |

| 17 | `RegistryControllerIT` | `shouldReturnDeadWhenPersonIsNotAlive` | Sistema (HTTP) | `alive=false` | `200 OK`, body `"DEAD"` |

| 18 | `RegistryControllerIT` | `shouldReturnBadRequestWhenGenderIsNotValid` | Sistema (HTTP) | `gender="X"` inválido | `400 BAD REQUEST` |

| 19 | `RegistryRepositoryPostgresIT` | `shouldPersistAndRejectDuplicate` | Integración (Testcontainers/PostgreSQL) | Votante válido y duplicado contra BD real | `VALID` y `DUPLICATED`, igual que H2 pero validado contra PostgreSQL real |

| 20-23 | `RegistryRepositoryPostgresIT` | (4 casos adicionales) | Integración (Testcontainers/PostgreSQL) | Reglas de negocio replicadas contra PostgreSQL | Coinciden con el comportamiento validado en H2 |

| 24 | `CertificadoServicePactTest` | `emiteCertificadoCuandoElVotanteEsValido` | Contract (Pact, consumidor) | Votante válido | Contrato espera `VALID` |

| 25 | `CertificadoServicePactTest` | `noEmiteCertificadoCuandoElVotanteEstaDuplicado` | Contract (Pact, consumidor) | Votante duplicado | Contrato espera `DUPLICATED` |

| 26-27 | `RegistraduriaProviderPactIT` | (2 verificaciones) | Contract (Pact, proveedor) | Reproduce las interacciones del pacto contra la API real | La API real cumple el contrato declarado por el consumidor |



\## Resumen de ejecución



| Suite | Tests | Failures | Errors | Skipped |

|---|---|---|---|---|

| Unitarias + Mocks (`RegistryWithMockTest`) | 11 | 0 | 0 | 0 |

| Integración H2 (`RegistryIT`) | 2 | 0 | 0 | 0 |

| Sistema HTTP (`RegistryControllerIT`) | 5 | 0 | 0 | 0 |

| Integración Testcontainers (`RegistryRepositoryPostgresIT`) | 5 | 0 | 0 | 0 |

| Contract - consumidor (`CertificadoServicePactTest`) | 2 | 0 | 0 | 0 |

| Contract - proveedor (`RegistraduriaProviderPactIT`) | 2 | 0 | 0 | 0 |

| \*\*Total\*\* | \*\*27\*\* | \*\*0\*\* | \*\*0\*\* | \*\*0\*\* |



\*\*Cobertura combinada (JaCoCo, unitarias + integración + sistema):\*\* 91% instrucciones, 70% branches.

