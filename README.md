# Intro

Este informe detalla una metodología técnica de múltiples etapas utilizada para filtrar la viabilidad mayorista de marcas de ropa de alta demanda. El objetivo principal fue distinguir la infraestructura B2B oficial y segura de los entornos de reventa no autorizados o de alto riesgo. A través de la extracción automatizada de SERP, verificación de dominios y auditoría de pasarelas de pago, se han identificado cuatro marcas principales —Lululemon, Vuori, Ryderwear y Pangaia— con marcos de asociación profesional viables.

## Metodología Técnica

### Fase I: Extracción Automatizada Multimotor

Se implementó una herramienta propietaria basada en una API de Go y CLI para extraer datos de las páginas de resultados de motores de búsqueda (SERP) en cinco índices globales: Google, Yandex, Baidu, Bing y DuckDuckGo.

- **Parámetros:** Se utilizaron diversas cadenas de búsqueda (ej. "strategic sales", "dealer application", "B2B portal") y ventanas de fechas específicas para identificar canales profesionales persistentes.
- **Repositorio Inicial de Datos:** Los resultados en formato JSON se consolidaron mediante Python (utilizando una lógica de concatenación/fusión) para identificar solapamientos y dominios oficiales de alto ranking.

### Fase II: Auditoría de Dominios y Seguridad

Para mitigar el riesgo de sitios de "mercado gris" o de suplantación de identidad (phishing) dirigidos a compradores mayoristas, se aplicó una capa de filtrado secundaria:

- **Análisis de API Whois:** Se auditaron los dominios para verificar la longevidad de su registro. Los dominios establecidos recientemente (que a menudo indican sitios de estafa temporales) fueron descartados del conjunto de datos.
- **Catalogación de Pasarelas de Pago:** Se realizó una simulación del recorrido del usuario hasta el carrito para cada candidato. Los sitios fueron clasificados inmediatamente como *Estafa/Alto Riesgo* y excluidos si los métodos de pago disponibles se limitaban a:
  - Criptomonedas (Irreversibles)
  - CashApp / Zelle / Transferencias bancarias
  - Cuentas de transferencias entre pares (P2P) no comerciales

## Hallazgos

Los siguientes canales superaron con éxito la auditoría de seguridad y proporcionan puntos de entrada oficiales B2B/Mayoristas para minoristas de boutiques legítimos.

| Marca       | Nombre del Programa              | URL de Acceso Principal                                        |
|-------------|----------------------------------|----------------------------------------------------------------|
| Lululemon   | Ventas Estratégicas              | [shop.lululemon.com/about/strategic-sales](https://shop.lululemon.com/about/strategic-sales) |
| Vuori       | Distribuidor Autorizado          | [help.vuoriclothing.com/en_us/wholesaler](https://help.vuoriclothing.com/en_us/wholesaler) |
| Ryderwear   | Portal Mayorista                 | [wholesale.ryderwear.com](https://wholesale.ryderwear.com)     |
| Pangaia     | Mercancía Corporativa Responsable| [pangaia.com/responsible-merchandise](https://pangaia.com/responsible-merchandise) |

## Marcas No Viables

Durante la auditoría, las siguientes marcas fueron señaladas por tener una infraestructura B2B en línea insignificante o inexistente para boutiques independientes:

- **Alo Yoga:** Los resultados estaban altamente saturados por lotes de liquidación de terceros y revendedores no autorizados; no se identificó un portal B2B directo y verificado.
- **Spanx:** El perfil actual de SERP carece de una aplicación B2B de autoservicio dedicada para la expansión de tiendas físicas independientes.

## Conclusión

Para las boutiques que buscan expandir su catálogo de ropa de alto rendimiento, **Vuori** y **Ryderwear** ofrecen las infraestructuras mayoristas digitales más optimizadas y listas para aplicar. **Lululemon** se mantiene como un socio estratégico altamente selectivo, mientras que **Pangaia** se enfoca en colaboraciones B2B alineadas con su misión. Se recomienda evitar cualquier oferta de "lotes" de terceros descubierta durante las búsquedas iniciales, ya que estas fallaron consistentemente el protocolo de seguridad de la Fase II.
