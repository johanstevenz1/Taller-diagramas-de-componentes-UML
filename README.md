# Taller de diagramas de componentes UML

Solución de **MediCare** y **EduManager** basada en los casos del taller y en la lectura *Conectando el diseño orientado a objetos: interfaces, relaciones y vista funcional mediante componentes UML* (Bloque 5), especialmente los apartados 20–24 y las recomendaciones de los apartados 25–27.

## Archivos de entrega

| Caso | Proyecto editable de Visual Paradigm | Vista general | Colaboración analizada |
|---|---|---|---|
| MediCare | [MediCare.vpp](MediCare.vpp) | [Componentes](MediCare_01_Componentes.png) | [Reserva de cita](MediCare_02_Reserva_de_cita.png) |
| EduManager | [EduManager.vpp](EduManager.vpp) | [Componentes](EduManager_01_Componentes.png) | [Inscripción](EduManager_02_Inscripcion.png) |

Los proyectos y las imágenes se crean, guardan y exportan con **Visual Paradigm 18.1**, usando su Open API dentro de la aplicación. Los PNG son exportaciones del motor de Visual Paradigm. Cada VPP contiene dos diagramas de componentes editables; los componentes de la vista general se reutilizan en la vista de colaboración. La extensión nativa correcta es **.vpp**.

**Edición instalada:** Visual Paradigm Enterprise en evaluación. Sus exportaciones incluyen la marca de agua que añade el programa. Se conserva la exportación original; para exportar sin esa marca se necesita abrir el VPP con una licencia que lo permita. Véase la [información oficial sobre exportaciones y marcas de agua](https://www.visual-paradigm.com/support/faq.jsp).

## Cómo leer los diagramas

- Un rectángulo marcado `«component»` representa un módulo con una responsabilidad funcional reconocible. Sus operaciones resumen capacidades públicas; no son clases internas ni una implementación en código.
- Una dependencia discontinua apunta **desde quien necesita un servicio hacia quien lo ofrece**. No representa el orden de ejecución.
- Las interfaces de la segunda vista se muestran en notación rectangular `«Interface»`. Una relación de uso expresa que el consumidor requiere ese contrato. Una realización, con triángulo hueco hacia la interfaz, expresa que el proveedor lo implementa.
- Las notas explican reglas y el escenario de colaboración. Los colores facilitan la lectura y no añaden semántica UML.
- Se conserva el nivel funcional pedido por la lectura: no se incluyen servidores, bases de datos, protocolos, microservicios ni detalles de despliegue. Tampoco se introducen herencia o rombos de composición sin una razón del dominio.

## Caso 1 — MediCare

| Componente | Responsabilidad principal | Colaboración necesaria |
|---|---|---|
| Gestión de pacientes | Registrar pacientes y mantener sus datos personales y de contacto. | Citas obtiene identidad/contacto; Historia identifica al titular. |
| Gestión de citas | Solicitar, consultar, modificar y cancelar citas. Coordinar la confirmación. | Pacientes, Agenda y disponibilidad, Notificaciones. |
| Agenda y disponibilidad | Consultar la agenda de un médico y reservar/liberar intervalos sin doble ocupación. | Profesionales suministra horarios, especialidades y excepciones. |
| Gestión de profesionales | Administrar médicos, especialidades, horarios y disponibilidad planificada. | Provee información a Agenda. |
| Atención clínica | Registrar síntomas, diagnóstico, observaciones y recomendaciones de cada consulta. | Citas, Historia clínica, Órdenes y prescripciones. |
| Historia clínica | Conservar y consultar longitudinalmente las atenciones de cada paciente. | Pacientes y Órdenes y prescripciones. |
| Órdenes y prescripciones | Emitir y recuperar órdenes de exámenes y prescripciones vinculadas a una atención. | Recibe de Atención el identificador de la atención correspondiente. |
| Notificaciones | Avisar al paciente de la confirmación, modificación o cancelación de una cita. | Citas entrega el evento y los datos del destinatario. |

**Solicitar una cita:** Citas consulta al paciente y solicita a Agenda una reserva condicional. Agenda aplica los horarios de Profesionales y comprueba la ocupación. La comprobación y la reserva se tratan como una operación indivisible, para que dos solicitudes concurrentes no obtengan el mismo intervalo. Citas confirma únicamente si obtiene la reserva y después solicita la notificación. Si el registro falla, libera la reserva; si el horario está ocupado, rechaza esa opción.

La modificación de una cita requiere reservar el nuevo intervalo antes de liberar el anterior; la cancelación libera la ocupación. Ambos cambios generan el aviso correspondiente. La agenda del médico se consulta mediante Agenda; pacientes y médicos consultan el historial mediante Historia clínica. Las órdenes y prescripciones se recuperan por el identificador de la atención, por lo que no quedan como documentos aislados.

## Caso 2 — EduManager

| Componente | Responsabilidad principal | Colaboración necesaria |
|---|---|---|
| Planes de estudio | Administrar programas, planes, asignaturas y sus prerrequisitos. | Provee reglas a Prerrequisitos y el plan a Historial. |
| Oferta académica | Administrar y consultar grupos con asignatura, profesor, horario y capacidad máxima. | Provee grupos a Inscripción y Calificaciones, capacidad a Cupos e intervalos a Horarios. |
| Inscripción académica | Coordinar la inscripción y consultar inscripciones/estudiantes inscritos por grupo. | Consulta Oferta y requiere tres validadores independientes. |
| Validación de prerrequisitos | Comprobar las condiciones curriculares de una asignatura. | Planes de estudio e Historial académico. |
| Control de cupos | Consultar y reservar/liberar las ocupaciones de los grupos. | Capacidad máxima definida en Oferta. |
| Compatibilidad de horarios | Detectar cruces entre grupos ya inscritos y seleccionados en la solicitud. | Horarios de Oferta y selecciones que controla para cada estudiante. |
| Evaluaciones y calificaciones | Registrar actividades/notas y porcentajes; calcular y publicar la definitiva. | Grupos docentes de Oferta, inscritos de Inscripción y Notificaciones. |
| Historial académico | Consultar resultados publicados y asignaturas aprobadas, reprobadas y pendientes. | Calificaciones y Planes de estudio. |
| Notificaciones académicas | Informar a los estudiantes que sus resultados definitivos están publicados. | Calificaciones le entrega los destinatarios inscritos y el aviso. |

**Realizar una inscripción:** Inscripción consulta el grupo de Oferta y requiere `IPrerrequisitos`, `ICompatibilidadHoraria` e `ICupos`. Prerrequisitos compara reglas del plan con asignaturas aprobadas del historial. Horarios valida y reserva temporalmente la selección del estudiante. Cupos reserva una plaza si todavía existe disponibilidad. Inscripción registra la inscripción solamente cuando todas las condiciones son válidas. Ante un rechazo o fallo, libera las reservas obtenidas. La reserva de cupos es indivisible y las selecciones se coordinan por estudiante para evitar conflictos entre solicitudes concurrentes.

**Publicar resultados:** el profesor consulta sus grupos y estudiantes inscritos, registra notas y configura los porcentajes de evaluación. Calificaciones calcula `definitiva = Σ(nota de la actividad × porcentaje / 100)`, verifica que los porcentajes sumen 100 y publica el resultado. Solo la publicación solicita el aviso a los estudiantes. Historial consume las definitivas publicadas y el plan para identificar materias aprobadas, reprobadas y pendientes.

## Decisiones y límites del modelo

- Citas e Inscripción coordinan; los componentes especializados conservan las reglas de disponibilidad, prerrequisitos, cupos y horarios. Esto evita un módulo que concentre todas las responsabilidades.
- La reserva condicional y la liberación ante fallos son decisiones de diseño para cumplir las restricciones de no duplicación; no se impone una tecnología de persistencia o transacciones.
- Para notas no se inventa un umbral de aprobación ni un catálogo de porcentajes. Son políticas institucionales configurables que el caso no especifica. Un resultado pendiente significa una asignatura exigida por el plan que todavía no está aprobada.
- La lectura sirve de referencia conceptual. No se copia su esquema ilustrativo de flechas: las dependencias se orientan según el consumidor y el proveedor de cada servicio.

## Revisión

Se revisan las exportaciones y se reabren los proyectos con la API de Visual Paradigm para comprobar sus elementos nativos: MediCare contiene 8 componentes y EduManager contiene 9. Cada proyecto incluye 2 diagramas, 3 relaciones de uso y 3 realizaciones de interfaz en su vista de colaboración. El archivo VPP permite cambiar nombres, operaciones, posiciones y relaciones de manera individual.

Los diagramas siguen el mecanismo documentado por el fabricante para [crear diagramas de componentes con Open API](https://knowhow.visual-paradigm.com/openapi/component-diagram/).
