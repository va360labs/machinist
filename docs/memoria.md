# Memoria de decisiones

## 2026-11-18 — Encargo #1: renovaciones concurrentes de la señal del daemon

- Las pruebas **confirman el fallo**: dos `renew_daemon_owner` simultáneas sobre una señal caducada devuelven `true` las dos (lectura y escritura separadas, sin transacción). Un aviso de pausa duplicado y tardío hace que `initialize_schedule_cursor_after_pause` descarte el cursor vigente y se salte la ejecución que ya tocaba.
- Las pruebas fijan el **comportamiento actual** (nombres con `currently_`) en lugar de usar `#[ignore]` o fallar: así documentan el fallo sin romper la CI. Quien lo arregle debe cambiar la cuenta esperada a 1 y esperar que se encole la ejecución de las 12:03.
- El cruce se fuerza con una tercera conexión que retiene `BEGIN IMMEDIATE`. Si alguna lectura llegara tarde, vería la señal ya renovada y devolvería `false`, así que un resultado de 2 demuestra que hubo cruce. La espera de 500 ms deja margen de sobra frente al tiempo de espera de 5 s.
- No se añade prueba de extremo a extremo con `FactoryDaemon::run`: dependería de tiempos reales. Se reproduce la rama del bucle con horas fijas.
