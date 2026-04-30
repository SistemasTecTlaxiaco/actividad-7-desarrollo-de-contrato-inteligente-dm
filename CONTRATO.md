# SECCIÓN: DESARROLLO

Se diseñó el contrato en **Soroban CLI** utilizando el lenguaje **Rust**, estructurado bajo una arquitectura modular que separa la lógica de estado y la lógica de activos.

## 1. Lógica del Contador
Implementada para gestionar operaciones aritméticas persistentes en la blockchain:
* **`increment(env: Env) -> u32`**: Accede al almacenamiento de instancia (`instance storage`), recupera el valor de `COUNTER` (o 0 si no existe), le suma uno y guarda el nuevo resultado.
* **`decrement(env: Env) -> u32`**: Reduce el valor almacenado. Utiliza la operación `saturating_sub(1)` para garantizar que, si el contador es 0, no ocurra un error de bajo flujo (*underflow*).
* **`get(env: Env) -> u32`**: Función de lectura simple que permite consultar el valor actual almacenado en la memoria del contrato.
* **`reset(env: Env)`**: Función administrativa que sobrescribe el valor de `COUNTER` con 0, reiniciando el estado.

## 2. Lógica de Tokens (Estándar Básico)
Define el ciclo de vida de los activos digitales mediante las siguientes funciones técnicas:
* **`mint(env: Env, to: Address, amount: u128) -> u128`**: Crea una nueva cantidad de tokens. Actualiza el mapa de saldos (`BALANCES`) y el suministro total (`TOTAL_SUPPLY`) utilizando `checked_add` para prevenir errores de desbordamiento.
* **`transfer(env: Env, from: Address, to: Address, amount: u128) -> bool`**: Ejecuta la transferencia verificando la solvencia del emisor. Si los fondos son suficientes, actualiza ambos saldos; de lo contrario, retorna `false`.
* **`balance(env: Env, account: Address) -> u128`**: Función de consulta que accede al mapa de saldos. Implementa `unwrap_or(0)` para asegurar que direcciones nuevas retornen un saldo de cero sin errores.
* **`total_supply(env: Env) -> u128`**: Recupera el valor global de activos circulantes emitidos por el contrato.
* **`burn(env: Env, from: Address, amount: u128) -> bool`**: Reduce el saldo de un usuario y el suministro total, garantizando la consistencia mediante el uso de resta saturada.

## 3. Implementación Técnica y Seguridad
Dentro del bloque `impl ContadorContract`, se destaca el uso del objeto `env` para interactuar con el almacenamiento. Se aplicaron patrones de seguridad críticos:
* **Validación de Desbordamiento:** Uso de `checked_add` y `.expect("overflow")` para prevenir vulnerabilidades aritméticas en tipos `u128`.
* **Consistencia de Datos:** La lógica asegura que la suma de todos los saldos individuales siempre coincida con el `total_supply`.
* **Gestión de Errores:** Uso de `unwrap_or_else` para inicializar mapas de datos en la primera interacción, evitando fallos por claves inexistentes.

## 4. Suite de Pruebas y Validación (QA)
Para garantizar la fiabilidad del contrato, se implementó una suite de **10 pruebas automatizadas** con un 100% de éxito.

### Prueba de Integración: `test_multiple_operations`
Simula un ciclo de vida completo del contrato:
1. **Inicialización:** Creación de identidades independientes (`acc1`, `acc2`, `acc3`).
2. **Distribución:** Ejecución de operaciones `mint` y validación de suministro.
3. **Movimiento:** Transferencias entre cuentas para probar la lógica de saldos.
4. **Reducción:** Invocación de la función `burn`.
5. **Verificación:** Validación mediante macros `assert_eq!` para confirmar que los estados finales coinciden con la matemática esperada.

## 5. Despliegue del Contrato
El contrato fue compilado exitosamente y desplegado en el entorno de red. Las validaciones lógicas incluidas aseguran que el contrato falle de manera segura ante entradas incorrectas, protegiendo la integridad de los fondos y la estabilidad de la red Stellar.
