# Aerotermia por demanda de TRVs

Blueprint de Home Assistant que enciende y apaga el switch de una aerotermia (o caldera) a partir de la demanda de calor de las válvulas termostáticas (TRV) de los radiadores. Un solo blueprint, sin ayudantes.

[![Abre tu Home Assistant e importa el blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Falexgonza-rgz%2Fha-blueprint-aerotermia-trv%2Fblob%2Fmain%2Faerotermia_demanda_trv.yaml)

## Qué hace

- Un radiador "demanda calor" cuando el atributo `hvac_action` de su entidad `climate` es `heating`.
- Hay **dos niveles de radiadores**:
  - **Prioritarios**: basta con que demanden calor N de ellos (por defecto 1) para arrancar la aerotermia.
  - **Resto**: hacen falta M a la vez (por defecto 2).
- **Detección automática**: todas las entidades `climate` cuyo id empiece por un prefijo (por defecto `climate.trv_`) se añaden solas al resto, salvo las prioritarias y las excluidas. Una TRV nueva con ese prefijo entra sin tocar el blueprint. Si vacías el prefijo, se desactiva y solo cuenta la lista manual.
- **Sin ayudantes ni sondeos**: funciona por eventos, con disparadores de plantilla y retardos con `for:`.

## Protecciones

- **Retardos** antes de encender y de apagar, para ignorar parpadeos de las válvulas.
- **Tiempos mínimos** en marcha y de parada, para evitar ciclados cortos del compresor.
- **Protección hidráulica**: si **ninguna** TRV demanda calor, el agua no tiene adónde ir. En ese caso la aerotermia se apaga tras un retardo muy corto (30 s por defecto) y **sin esperar el tiempo mínimo en marcha**. El retardo de apagado normal y el tiempo mínimo en marcha solo se aplican mientras al menos un radiador sigue pidiendo calor.
- **Reconciliación** al arrancar Home Assistant o cuando el switch vuelve de `unavailable`: espera hasta 5 minutos a que las TRVs estén disponibles y entonces decide.
- Antes de actuar, vuelve a comprobar la demanda tras cada espera. Si cambió, no hace nada.

## Instalación

1. Pulsa el botón de importación de arriba, o copia `aerotermia_demanda_trv.yaml` a `config/blueprints/automation/<tu carpeta>/`.
2. En **Ajustes → Automatizaciones → Blueprints**, crea una automatización a partir de él.
3. Elige el switch de la aerotermia y tus radiadores prioritarios. El resto se detecta solo si sus ids empiezan por `climate.trv_`.

Requiere **Home Assistant 2024.10 o superior** (sintaxis `triggers:` / `actions:`).

Si antes tenías una automatización que calculaba la demanda y otra que arrancaba la aerotermia, desactívalas: este blueprint las sustituye.

## Entradas

| Entrada | Por defecto | Descripción |
| --- | --- | --- |
| Switch de la aerotermia | | Switch que se enciende y apaga según la demanda |
| Radiadores prioritarios | vacío | Entidades `climate` que por sí solas justifican arrancar |
| Mínimo de prioritarios demandando calor | 1 | Cuántos prioritarios deben demandar a la vez |
| Prefijo de detección automática | `climate.trv_` | Se añaden al resto las `climate` con ese prefijo |
| Resto de radiadores (a mano) | vacío | Radiadores del resto que no empiecen por el prefijo |
| Radiadores excluidos | vacío | Entidades con el prefijo que no deben contar |
| Mínimo del resto demandando calor | 2 | Cuántos del resto deben demandar a la vez |
| Retardo antes de encender | 2 min | La demanda debe mantenerse este tiempo |
| Retardo antes de apagar (con algún radiador demandando) | 2 min | Aplica solo si queda algún radiador abierto |
| Retardo antes de apagar sin ningún radiador demandando | 30 s | Apagado casi inmediato si no hay circulación |
| Tiempo mínimo en marcha (con algún radiador demandando) | 5 min | No aplica si no queda ningún radiador |
| Tiempo mínimo parada | 5 min | Tiempo mínimo antes de volver a encender |

## Limitaciones y avisos

- Asume que `hvac_action: heating` significa válvula abierta y `idle` significa cerrada. Si tus TRVs reportan otra cosa, ajusta la plantilla.
- La lógica está validada de forma simulada sobre las plantillas, pero **controla un equipo real**: pruébalo con retardos cortos y vigila los primeros ciclos.
- Consulta el manual de tu equipo: puede tener ya sus propias protecciones de tiempo mínimo entre arranques del compresor.
- La entidad `climate` de la propia aerotermia no empieza por `climate.trv_`, por lo que no cuenta como radiador.
