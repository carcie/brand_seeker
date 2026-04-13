# Pipeline de Localización, Verificación y Monitoreo de Cadena de Suministro
## 1. Intro
El objetivo de este sistema es reemplazar la búsqueda manual con un pipeline ETL (Extraer, Transformar, Cargar) programático. El sistema identifica lotes de ropa al por mayor (específicamente de las marcas **Alo, Lululemon, OC, Ryder, Vuori y Pangaia**), audita la legitimidad de la fuente mediante una verificación de múltiples factores y valida el historial físico de la cadena de suministro a través de datos de comercio internacional.
El resultado final es un conjunto de datos estructurado y de alta integridad, listo para el modelado de viabilidad financiera (opcional).

## 2. Arquitectura del Sistema
La arquitectura del pipeline se compone de cuatro microservicios modulares:
 1. **Servicio de Ingesta (Descubrimiento):** Realiza búsquedas (scraping) y consultas periódicas a APIs de mercados en busca de inventario específico.
 2. **Servicio de Auditoría (Validación):** Verifica programáticamente la antigüedad del dominio, el registro empresarial y la ubicación física de los almacenes.
 3. **Servicio de Procedencia (Cadena de Suministro):** Contrasta la fuente con registros de comercio global y conocimientos de embarque (Bill of Lading - BOL) para verificar el flujo real de inventario.
 4. **Servicio de Normalización (Entrega):** Estandariza los datos en un esquema JSON para su posterior análisis financiero.
## 3. Fase 1: Servicio de Ingesta (Descubrimiento)
**Objetivo:** Monitoreo programático de plataformas de liquidación de Nivel 1 (Tier 1) y puntos de venta de excedentes de tiendas departamentales.
**Enfoque Técnico:**
 * **Endpoints Objetivo:** B-Stock, Direct Liquidation, 888 Lots, DNC Wholesale, BrandsGateway, BrandsDistribution, Styliafoe y otros liquidadores especializados en marcas premium.
 * **Infraestructura:** Python workers programados (Celery/Redis) utilizando Playwright para renderizado de DOM dinámico, BeautifulSoup para análisis estático o servicios pagados como Octoparse.


```python
import requests
from bs4 import BeautifulSoup
import time

TARGET_BRANDS = ["Lululemon", "Vuori", "Alo", "Pangaia", "Ryder", "OC"]

def poll_listings(base_url):
    headers = {'User-Agent': 'Mozilla/5.0 (Apparel-Sourcing-Bot/1.0)'}
    try:
        response = requests.get(base_url, headers=headers, timeout=10)
        soup = BeautifulSoup(response.text, 'html.parser')
        listings = soup.find_all('div', class_='auction-card')
        
        results = []
        for item in listings:
            title = item.find('h2').text
            if any(brand.lower() in title.lower() for brand in TARGET_BRANDS):
                results.append({
                    "title": title,
                    "source_url": item.find('a')['href'],
                    "scraped_at": time.time()
                })
        return results
    except Exception as e:
        print(f"Error de Ingesta: {e}")
        return []

```
## 4. Fase 2: Servicio de Auditoría
**Objetivo:** Mitigación automatizada de riesgos para filtrar operaciones fraudulentas mediante análisis de metadatos y geoespacial.
### A. Auditoría de Integridad del Dominio
Verifica que el dominio tenga un historial suficiente y no sea un sitio temporal creado para estafas.


```python
import whois
from datetime import datetime

def check_domain_trust(domain):
    try:
        w = whois.whois(domain)
        creation_date = w.creation_date[0] if isinstance(w.creation_date, list) else w.creation_date
        age_days = (datetime.now() - creation_date).days
        return age_days > 365 # Umbral: 1 año
    except:
        return False

```

## 2. Arquitectura del Sistema
La arquitectura del pipeline se compone de cuatro microservicios modulares:
 1. **Servicio de Ingesta (Descubrimiento):** Realiza búsquedas (scraping) y consultas periódicas a APIs de mercados en busca de inventario específico.
 2. **Servicio de Auditoría (Validación):** Verifica programáticamente la antigüedad del dominio, el registro empresarial y la ubicación física de los almacenes.
 3. **Servicio de Procedencia (Cadena de Suministro):** Contrasta la fuente con registros de comercio global y conocimientos de embarque (Bill of Lading - BOL) para verificar el flujo real de inventario.
 4. **Servicio de Normalización (Entrega):** Estandariza los datos en un esquema JSON para su posterior análisis financiero.
## 3. Fase 1: Servicio de Ingesta (Descubrimiento)
**Objetivo:** Monitoreo programático de plataformas de liquidación de Nivel 1 (Tier 1) y puntos de venta de excedentes de tiendas departamentales.
**Enfoque Técnico:**
 * **Endpoints Objetivo:** B-Stock, Direct Liquidation, 888 Lots y liquidadores especializados en marcas premium.
 * **Infraestructura:** Trabajadores de Python programados (Celery/Redis) utilizando Playwright para renderizado de DOM dinámico o BeautifulSoup para análisis estático.

   
```python
import requests
from bs4 import BeautifulSoup
import time

TARGET_BRANDS = ["Lululemon", "Vuori", "Alo", "Pangaia", "Ryder", "OC"]

def poll_listings(base_url):
    headers = {'User-Agent': 'Mozilla/5.0 (Apparel-Sourcing-Bot/1.0)'}
    try:
        response = requests.get(base_url, headers=headers, timeout=10)
        soup = BeautifulSoup(response.text, 'html.parser')
        listings = soup.find_all('div', class_='auction-card')
        
        results = []
        for item in listings:
            title = item.find('h2').text
            if any(brand.lower() in title.lower() for brand in TARGET_BRANDS):
                results.append({
                    "title": title,
                    "source_url": item.find('a')['href'],
                    "scraped_at": time.time()
                })
        return results
    except Exception as e:
        print(f"Error de Ingesta: {e}")
        return []

```
## 4. Fase 2: Servicio de Auditoría (Verificación)
**Objetivo:** Mitigación automatizada de riesgos para filtrar operaciones fraudulentas mediante análisis de metadatos y geoespacial.
### A. Auditoría de Integridad del Dominio
Verifica que el dominio tenga un historial suficiente y no sea un sitio temporal creado para estafas.
```python
import whois
from datetime import datetime

def check_domain_trust(domain):
    try:
        w = whois.whois(domain)
        creation_date = w.creation_date[0] if isinstance(w.creation_date, list) else w.creation_date
        age_days = (datetime.now() - creation_date).days
        return age_days > 365 # Umbral: 1 año
    except:
        return False

```
### B. Auditoría Geoespacial de Almacenes
Utiliza la API de Google Maps para confirmar que la dirección del proveedor corresponde a una zona industrial/comercial y no a una residencia u oficina virtual.
```python
import googlemaps

gmaps = googlemaps.Client(key='YOUR_API_KEY')

def verify_warehouse_location(address):
    res = gmaps.geocode(address)
    if res:
        location_type = res[0]['types']
        # Identificar banderas rojas residenciales o no industriales
        red_flags = ['residential', 'subpremise', 'postal_code']
        return not any(flag in location_type for flag in red_flags)
    return False

```
## 5. Fase 3: Servicio de Procedencia (Monitoreo de Cadena de Suministro)
**Objetivo:** Confirmar que el vendedor es un consignatario documentado de las marcas objetivo a través de datos aduaneros.
**Enfoque Técnico:**
 * **Fuente de Datos:** Integración con APIs de inteligencia comercial (ej. Panjiva, ImportYeti o Vizion).
 * **Lógica de Validación:** Comparar el "Consignatario" (vendedor) con el "fabricante" (ej. Lululemon Athletica Canada) para encontrar registros de BLs coincidentes en los últimos 12 a 24 meses.
```python
def verify_provenance(company_name, target_brand):
    # Simulación de llamada a una API de Comercio Global
    api_url = f"https://api.trademonitor.com/v1/shipments?consignee={company_name}"
    response = requests.get(api_url, auth=('user', 'pass'))
    shipments = response.json().get('data', [])
    
    matches = [s for s in shipments if target_brand.lower() in str(s).lower()]
    return len(matches) > 0, {"match_count": len(matches)}

```
## 6. Fase 4: Normalización (Esquema de Entrega)
**Objetivo:** Generar el objeto final para el equipo de Viabilidad Financiera.
**Ejemplo de Carga Útil (Payload) JSON:**
```json
{
  "lead_metadata": {
    "brand": "Vuori",
    "vendor": "Empresa Puenta LLC",
    "source_url": "https://sitiodepalletsoropadelujo.com/vuori-lot-99"
  },
  "trust_score": {
    "domain_age_days": 840,
    "location_verified": true,
    "location_class": "Industrial Warehouse",
    "is_scam_likely": false
  },
  "supply_chain_verification": {
    "provenance_status": "VERIFIED",
    "last_shipment_date": "2026-02-10",
    "verified_shipper": "VUORI INC / ASIA LOGISTICS",
    "annual_volume_kg": 12500
  },
  "analysis_ready": true
}


def verify_warehouse_location(address):
    res = gmaps.geocode(address)
    if res:
        location_type = res[0]['types']
        # este es un ejemplo de como identificar redflags
        red_flags = ['residential', 'subpremise', 'postal_code']
        return not any(flag in location_type for flag in red_flags)
    return False

```

