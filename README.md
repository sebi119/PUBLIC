# PUBLIC

# Oracle: Daten innerhalb der Datenbank archivieren und komprimieren

Ja, gibt es! Oracle bietet mehrere Möglichkeiten, Daten **innerhalb der Datenbank** zu komprimieren.

---

## Optionen in Oracle

### 1. Advanced Compression (einfachste Lösung)

Direkt auf Tabellenebene aktivieren:

```sql
-- Bestehende Tabelle auf Compression umstellen
ALTER TABLE orders COMPRESS FOR OLTP;

-- Oder gleich beim Erstellen
CREATE TABLE orders_archive (...)
  COMPRESS FOR QUERY HIGH; -- stärkere Kompression für Archivdaten
```

| Compress-Typ | Wann sinnvoll |
|---|---|
| `COMPRESS FOR OLTP` | Produktivtabellen mit INSERT/UPDATE |
| `COMPRESS FOR QUERY HIGH` | Archivdaten, hauptsächlich lesend |
| `COMPRESS FOR ARCHIVE HIGH` | Maximale Kompression, selten gelesen |

> ⚠️ Erfordert die **Advanced Compression Lizenz**

---

### 2. In-Database Archiving (ohne separate Tabelle!)

Oracle 12c+ hat ein eingebautes Feature genau dafür – Zeilen als „archiviert" markieren, aber **in derselben Tabelle** lassen:

```sql
-- Feature auf Tabelle aktivieren
ALTER TABLE orders ROW ARCHIVAL;

-- Zeilen als archiviert markieren
UPDATE orders
SET ora_archive_state = '1'  -- '0' = aktiv, '1' = archiviert
WHERE created_at < ADD_MONTHS(SYSDATE, -24);

COMMIT;
```

Applikationen sehen dann **automatisch nur aktive Zeilen**:

```sql
-- Session auf "nur aktive Zeilen" einstellen
ALTER SESSION SET ROW ARCHIVAL VISIBILITY = ACTIVE;
-- ab jetzt sind archivierte Zeilen unsichtbar

-- Alle Zeilen sehen (z.B. für Archiv-Abfragen)
ALTER SESSION SET ROW ARCHIVAL VISIBILITY = ALL;
```

**Vorteil:** Keine zweite Tabelle nötig, kombinierbar mit Compression.

---

### 3. Kombination: In-Database Archiving + Compression

Das ist die eleganteste Lösung:

```sql
-- Tabelle mit beiden Features
ALTER TABLE orders ROW ARCHIVAL;
ALTER TABLE orders COMPRESS FOR ARCHIVE HIGH;

-- Archivierte Zeilen werden komprimiert gespeichert
-- Aktive Zeilen bleiben normal zugänglich
```

---

## Welche Lösung passt wann?

| Situation | Empfehlung |
|---|---|
| Keine Extra-Tabelle gewünscht | **In-Database Archiving** |
| Maximale Platzersparnis | **Compression FOR ARCHIVE HIGH** |
| Beides kombiniert | **In-Database Archiving + Compression** |
| Keine Advanced-Lizenz | Separate Archivtabelle + Standard Compression |

---

> **Habt ihr die Advanced Compression Lizenz, oder muss es ohne gehen?**
