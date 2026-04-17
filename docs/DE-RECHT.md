# DE-Recht fuer Palestine Drinks Shop

Pflicht-Module die wir selbst bauen muessen. Basis: Medusa v2 + Next.js.

## Pflicht (ab Tag 1)

### 1. Pfand (Getraenkepfand)
- **Anforderung**: Einweg-Pfand 0,25 EUR pro Dose muss GETRENNT vom Produktpreis ausgewiesen werden (PflichtPfandV)
- **Loesung**:
  - Custom Field `deposit_cents` auf `product_variant` (integer, default 25)
  - Custom Cart-Subscriber: addiert Pfand als separate Line-Item-Gruppe
  - Storefront: Zeigt "Zzgl. 0,25 EUR Pfand pro Dose" unter Preis
  - Rechnung: Pfand als eigene Position mit USt-Schluessel 19 %

### 2. Grundpreis (Preisangabenverordnung)
- **Anforderung**: Bei Getraenken muss Grundpreis pro Liter angegeben werden (EUR/L), gut lesbar in Naehe des Verkaufspreises
- **Loesung**:
  - Custom Fields `volume_ml` (integer) + `pack_size` (integer) auf `product_variant`
  - Helper `calcUnitPrice(price, volume_ml, pack_size) => "12,13 EUR/L"`
  - Storefront Product Card + PDP: Grundpreis unter Preis anzeigen
  - Bei 24er-Pack 330ml zu 31,20 EUR: 3,94 EUR/L

### 3. Rechnung (GoBD-konform)
- **Anforderung**:
  - Fortlaufende Rechnungsnummer (keine Luecken)
  - Pflichtangaben: Datum, Firma, Kunde, USt-ID NAIMO, USt-Schluessel, Steuerbetrag
  - GoBD-konforme Archivierung (unveraenderlich, 10 Jahre)
- **Loesung**:
  - Medusa `order.placed` Subscriber → PDF generieren
  - Custom Counter-Service: `invoice_sequence` Tabelle mit atomarem Increment
  - Format: `RE-2026-000001`
  - PDF via `puppeteer` oder `@react-pdf/renderer`
  - Ablage: Minio `invoices/` Bucket + DB-Referenz
  - Email-Versand als Attachment

### 4. Widerrufsrecht (BGB §312g, 14 Tage)
- **Anforderung**:
  - Widerrufsbelehrung im Checkout vor Bestellbestaetigung
  - Muster-Widerrufsformular als PDF/Download
  - Button "Zahlungspflichtig bestellen" (§312j BGB)
- **Loesung**:
  - Checkout-Step: Pflicht-Checkbox "Ich habe die Widerrufsbelehrung gelesen"
  - Order-Button explizit: "Jetzt zahlungspflichtig bestellen"
  - Separate Seite `/widerruf` + PDF-Download
  - Bestellbestaetigungs-Email enthaelt Widerrufsbelehrung

### 5. Impressum + Datenschutz + AGB
- **Anforderung**: Jederzeit erreichbar (Footer), vor Kauf akzeptiert
- **Loesung**:
  - Footer-Links: Impressum, Datenschutz, AGB, Widerruf, Versand, Kontakt
  - Checkout: Pflicht-Checkbox "AGB + Datenschutz akzeptiert"
  - Bei NAIMO Trade UG: LUCID-Nummer (Verpackungsregister) ins Impressum
  - Cookie-Banner TCF v2.2 konform

### 6. Preisangabe inkl. MwSt
- **Anforderung**: Endpreise inkl. 19 % MwSt anzeigen (B2C)
- **Medusa-Konfig**:
  - Region Deutschland: `tax_rate: 19`, `automatic_taxes: true`, `includes_tax: true`
  - Alle Produktpreise als Brutto eintragen
- **Storefront**: "Preise inkl. 19 % MwSt, zzgl. Versand + Pfand"

### 7. Doppelter Opt-in Newsletter
- **Anforderung**: Nur mit Confirmation-Email Einwilligung speichern
- **Loesung**: Supabase Table `newsletter_subscribers` + Resend/Mailgun fuer Confirmation

## Zahlung + Versand (Woche 2)

### 8. Zahlarten
- Stripe (Kreditkarte, SEPA Lastschrift, Klarna)
- PayPal
- Vorkasse (manuell)
- ggf. Rechnungskauf via Klarna/Ratepay

### 9. Versand
- **DHL** (Paketversand, ca. 5 EUR bei 24er-Karton)
- **DPD** als Alternative
- Versandkosten-Matrix: Gewicht-basiert (24er ca. 9 kg)
- Freier Versand ab z.B. 60 EUR
- Spedition fuer Palette (B2B)
- Kein Versand nach AT/CH (erstmal) — DE only

### 10. Retouren
- Retourenlabel im Kundenkonto
- Schrittweise via Medusa Return Flow

## Buchhaltung (Woche 3)

### 11. DATEV-Export
- Monatlicher Export CSV mit:
  - Datum, Rechnungsnummer, Netto, MwSt, Brutto
  - Sachkonto 8400 (Erloese 19 %)
  - Gegenkonto (Bank/PayPal/Stripe)
- Scheduled Job: 1. des Monats fuer Vormonat
- Download als ZIP aus Admin

### 12. Buchhaltungs-Automation via n8n
- Rechnungen monatlich an Steuerberater
- Zahlungseingang-Matching (Banking → Orders)
- OSS-Meldung EU-Versand

## Rechts-Sicherheit (nice to have)

- **Trusted Shops** Integration (Kaeuferschutz-Badge, Conversion +15%)
- **Haendlerbund** oder **IT-Recht Kanzlei**: AGB/Datenschutz/Widerruf aktuell halten
- **eKomi** oder **Trustpilot**: Rechtssicheres Rezensions-System
- **SSL** via Traefik Let's Encrypt (automatisch via Coolify)

## Umsetzungs-Reihenfolge

1. Woche 1: Pfand + Grundpreis + Rechnung + Widerruf + AGB-Hooks
2. Woche 2: Stripe + DHL + Versand-Kosten-Matrix
3. Woche 3: DATEV-Export + Retouren
4. Woche 4: Trusted Shops + Newsletter Opt-in + Launch
