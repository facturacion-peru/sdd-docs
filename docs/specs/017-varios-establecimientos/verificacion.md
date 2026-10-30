# Verificación 017: Varios establecimientos

## T026 · API contra SUNAT beta y la base local (2026-10-30)

Base local de desarrollo (empresa demo, SUNAT beta validado), rama `017-varios-establecimientos` de la API.

| Prueba | Resultado |
|---|---|
| **CE-001:** segundo local `0001` «Sucursal Centro» (JR. DE LA UNION 456), serie `B002` en ese local, entrada de 5 unidades y una boleta emitida por el servicio real | ✅ SUNAT beta aceptó **B002-00000001** («La Boleta numero B002-1, ha sido aceptada», código 0). El XML lleva `<cbc:AddressTypeCode>0001</cbc:AddressTypeCode>` y la dirección del local; el stock bajó solo en el local (4) y el principal quedó igual (30) |
| **Migración de la base local** (aplicada en T003) | ✅ Ningún lote, movimiento, ticket ni comprobante quedó sin local: 47 lotes, 123 movimientos, 16 tickets y 33 comprobantes en el principal de la empresa demo, y lo de la otra empresa demo en su principal. Los dos vendedores demo (`vendedor@demo.test` y el inactivo) quedaron asignados al principal |

**CE-003 (mismas cifras antes y después):** la base local se migró en T003, antes de que existiera esta verificación, así que no hay una foto de «antes» para comparar a mano. Lo cubre la prueba automática `tests/Integration/Establishments/EstablishmentMigrationTest.php`, que siembra datos en el esquema anterior, migra y compara el inicio, el stock, las listas y los saldos de los movimientos, en SQLite y en PostgreSQL.

**Queda para el recorrido con la app** (T026 pasos 3 a 5 y F022, con el usuario): entrada en el principal y traslado al segundo local, venta del vendedor del segundo local, nota de crédito con devolución y alertas por local; con un solo local, igual que en `main`; 360 px y teléfono real.
