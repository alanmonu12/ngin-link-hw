No. Pin	Nombre	Tipo	Descripción
1	D	I (Entrada)	Entrada de datos de transmisión CAN (LOW para estados dominantes y HIGH para estados de bus recesivos). También conocido como TXD o entrada del driver.
2	GND	GND	Conexión a tierra (Ground).
3	VCC	Alimentación	Voltaje de alimentación del transceptor de 3.3V.
4	R	O (Salida)	Salida de datos de recepción CAN (LOW para estados dominantes y HIGH para estados de bus recesivos). También conocido como RXD o salida del receptor.
5	Vref	O (Salida)	Pin de salida de voltaje de referencia equivalente a VCC / 2.
6	CANL	I/O (Ent/Sal)	Línea de bus CAN de nivel bajo (Low level).
7	CANH	I/O (Ent/Sal)	Línea de bus CAN de nivel alto (High level).
8	RS	I (Entrada)	

Pin de selección de modo:

• Pull-down fuerte a GND = Modo de alta velocidad.

• Pull-up fuerte a VCC = Modo de bajo consumo.

• Pull-down mediante resistencia de 10kΩ a 100kΩ a GND = Modo de control de pendiente (slope control).

