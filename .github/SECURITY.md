# Política de seguridad

Esta política se aplica a todos los repositorios de la
[Planasa Digital Apps Organization](https://github.com/Planasa-Digital-Apps-Organization)
que no definan un `SECURITY.md` propio.

## Reportar una vulnerabilidad

**No abras un issue público para reportar vulnerabilidades.** En su lugar:

- **Email**: `security@planasa.com` <!-- TODO: confirmar buzón final -->
- **Asunto**: `[SECURITY] <repo>: <breve descripción>`
- **Contenido sugerido**: pasos para reproducir, impacto observado, versión / commit afectado, propuesta de mitigación si la tienes.

Si la vulnerabilidad afecta a producción (datos clientes, credenciales expuestas, RCE), marca el asunto con `[CRITICAL]` y notifica también por el canal interno de IT.

## SLA de respuesta

| Severidad | Acknowledge | Mitigación / parche |
| --- | --- | --- |
| Critical | < 24 h | < 7 días |
| High | < 48 h | < 14 días |
| Medium / Low | < 5 días | next planned release |

<!-- TODO: validar SLAs con el equipo. Placeholders razonables hasta confirmación. -->

## Divulgación coordinada

- No publiques detalles técnicos antes de que la mitigación esté desplegada en producción.
- Una vez resuelta, publicaremos un advisory en el repo afectado (GitHub Security Advisories) referenciando el CVE si aplica.
- Acreditamos al reportador salvo que prefiera anonimato.

## Out of scope

- Vulnerabilidades en dependencias de terceros sin impacto demostrable en nuestros repos (reportar al upstream).
- Problemas de configuración local del desarrollador (no son seguridad de producto).
- Ingeniería social no relacionada con la superficie del producto.
