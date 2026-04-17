# Palestine Drinks E-Commerce

Ecommerce-Shop fuer Palestine Drinks Deutschland (NAIMO Trade UG).

## Stack

- **Backend**: Medusa v2 (Node/TypeScript) mit Postgres + Redis
- **Storefront**: Next.js 15 (React 19, App Router)
- **Hosting**: Coolify auf VPS `72.62.151.115` (Naim Obeid)
- **Domain (Test)**: `ecom.palestine-drinks.de`
- **Images**: Minio (S3) auf `minio.naim-obeid.de`

## Struktur

```
palestine-drinks-ecom/
├── palestine-drinks/              # Medusa v2 Backend (API + Admin)
├── palestine-drinks-storefront/   # Next.js Storefront
├── docker-compose.yml             # Postgres + Redis lokal
├── .gitignore
└── README.md
```

## Lokale Entwicklung

```bash
# Postgres + Redis starten
docker compose up -d

# Backend (http://localhost:9000/app = Admin)
cd palestine-drinks && npm run dev

# Storefront (http://localhost:8000)
cd palestine-drinks-storefront && npm run dev
```

## Deploy

Coolify pullt automatisch aus GitHub und baut Docker-Images.
Nach Push:

```bash
curl -X POST "http://72.62.151.115:8000/api/v1/applications/<uuid>/start" \
  -H "Authorization: Bearer $COOLIFY_TOKEN"
```

## DE-Recht Module (geplant)

- Pfand-Feld auf Produkt (0,25 EUR/Dose)
- Grundpreis-Anzeige (EUR/Liter, EUR/100ml)
- Rechnungs-PDF mit fortlaufender Nummer (GoBD)
- Widerrufsbelehrung + Muster-Widerrufsformular
- Doppelter Opt-in Newsletter
- DATEV-Export
- DHL-Versand-Integration
- Klarna + PayPal + SEPA + Stripe

## Firma

NAIMO Trade UG
Karlstraße 40, 45739 Oer-Erkenschwick
+49 2368 8900789
info@palestine-drinks.de
