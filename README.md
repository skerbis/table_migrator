# Table_Migrator for REDAXO

# TableMigrator

`TableMigrator` ist eine PHP-Klasse, die entwickelt wurde, um die Migration von Daten zwischen Datenbanktabellen zu erleichtern. Sie bietet eine flexible und erweiterbare Lösung für die Übertragung von Daten aus einer alten Tabellenstruktur in eine neue, mit Unterstützung für verschiedene Arten von Feldmappings und Datenverarbeitung.

## Funktionen

- Einfaches Mapping von Feldern zwischen alter und neuer Tabelle
- Kombinieren mehrerer Felder in ein neues Feld
- Verarbeiten von Feldinhalten während der Migration
- Herstellen von Beziehungen zu verwandten Tabellen
- Automatisches Einfügen von migrierten Daten in die neue Tabelle

## Installation

Fügen Sie die `TableMigrator`-Klasse zu Ihrem REDAXO-Projekt hinzu. Stellen Sie sicher, dass sie im Namespace `klxm\migrator` liegt.

## Verwendung

### Grundlegende Verwendung

```php
use klxm\migrator\TableMigrator;

$migrator = new TableMigrator('old_table', 'new_table');
$migrator->addMapping('new_field', 'old_field')
         ->migrate();
```

### Erweiterte Mappings

```php
// Kombiniertes Mapping
$migrator->addCombinedMapping('full_name', ['first_name', 'last_name'], 
    function($firstName, $lastName) {
        return $firstName . ' ' . $lastName;
    }
);

// Verarbeitetes Mapping
$migrator->addProcessedMapping('description', 'old_description', 
    function($text) {
        return $migrator->truncateAndStripHTML($text, 200);
    }
);

// Verwandtes Mapping
$migrator->addRelatedMapping('category_name', 'category_id', 'rex_categories', 'name', 'Uncategorized');
```

### Durchführen der Migration

Nach dem Einrichten aller Mappings, führen Sie die Migration durch:

```php
$migrator->migrate();
```

## Methoden

- `addMapping($newField, $oldField)`: Einfaches Feldmapping
- `addCombinedMapping($newField, $oldFields, $combineFunction)`: Kombiniert mehrere Felder
- `addProcessedMapping($newField, $oldField, $processFunction)`: Verarbeitet Feldinhalte
- `addRelatedMapping($newField, $oldField, $relatedTable, $relatedField, $defaultValue)`: Stellt Beziehungen her
- `migrate()`: Führt die Migration durch
- `truncateAndStripHTML($text, $maxLength)`: Hilfsmethode zum Kürzen und Bereinigen von HTML

## Hinweise

- Diese Klasse wurde für die Verwendung mit REDAXO und YForm entwickelt.
- Stellen Sie sicher, dass Sie Backups Ihrer Daten haben, bevor Sie eine Migration durchführen.
- Testen Sie die Migration gründlich in einer Entwicklungsumgebung, bevor Sie sie in der Produktion einsetzen.

## Lizenz

[Ihre Lizenzinformationen hier einfügen]

## Beitragen

Wenn Sie zu diesem Projekt beitragen möchten, erstellen Sie bitte einen Pull Request oder öffnen Sie ein Issue für Diskussionen und Vorschläge.$migrator->migrate();
```

## Beispiele

### Benutzertabelle Migration

```php
$migrator = new TableMigrator('old_users', 'new_users');
$migrator->addMapping('id', 'user_id')
         ->addMapping('email', 'email_address')
         ->addProcessedMapping('password', 'password_hash', function($value) {
             return password_hash($value, PASSWORD_DEFAULT);
         })
         ->addCombinedMapping('full_name', ['first_name', 'last_name'], function($first, $last) {
             return $first . ' ' . $last;
         });
$migrator->migrate();
```

### Produktkatalog Migration

```php
$migrator = new TableMigrator('old_products', 'new_products');
$migrator->addMapping('id', 'product_id')
         ->addMapping('name', 'product_name')
         ->addProcessedMapping('price', 'price_cents', function($value) {
             return $value * 100;
         })
         ->addCombinedMapping('full_sku', ['sku_prefix', 'sku_number'], function($prefix, $number) {
             return $prefix . '-' . str_pad($number, 5, '0', STR_PAD_LEFT);
         });
$migrator->migrate();
```

### Forcal zu Kalendertabelle

### Kurzes Beispiel 


```php
$migrator = new TableMigrator('rex_forcal_entries', 'rex_calendar');
$migrator->addMapping('id', 'id')
         ->addMapping('name', 'name_1')
         ->addMapping('description', 'text_1')
         ->addMapping('start_date', 'start_date')
         ->addMapping('end_date', 'end_date')
         ->addForcalRRuleMapping('rrule', ['repeat', 'repeat_year', 'repeat_week', 'repeat_month', 'repeat_month_week', 'repeat_day', 'end_repeat_date']);
$migrator->migrate();
```

#### Ausführlich

```php
$migrator = new TableMigrator('rex_forcal_entries', 'rex_calendar');

$migrator->addMapping('id', 'id')
         ->addMapping('name', 'name_1')
         ->addMapping('teaser', 'teaser_1')
         ->addMapping('description', 'text_1')
         ->addMapping('start_date', 'start_date')
         ->addMapping('end_date', 'end_date')
         ->addMapping('start_time', 'start_time')
         ->addMapping('end_time', 'end_time')
         ->addMapping('all_day', 'full_time')
         ->addMapping('category_id', 'category')
         ->addMapping('location_id', 'venue')
         ->addMapping('status', 'status')
         ->addMapping('createdate', 'createdate')
         ->addMapping('updatedate', 'updatedate')
         ->addMapping('created_by', 'createuser')
         ->addMapping('updated_by', 'updateuser')
         ->addMapping('image', 'image')
         ->addMapping('file', 'file')
         ->addMapping('lang', 'lang_1')
         ->addProcessedMapping('event_type', 'type', function($value) {
             // Konvertiere Forcal Event-Typen in entsprechende Kalender-Typen
             switch ($value) {
                 case 'one_time':
                     return 'single';
                 case 'repeat':
                     return 'recurring';
                 default:
                     return 'other';
             }
         })
         ->addForcalRRuleMapping('rrule', [
             'repeat', 'repeat_year', 'repeat_week', 'repeat_month', 
             'repeat_month_week', 'repeat_day', 'end_repeat_date'
         ])
         ->addProcessedMapping('location', 'awoloc', function($value) {
             // Konvertiere AWO-spezifische Standortinformationen
             $locations = json_decode($value, true);
             if ($locations && isset($locations[0]['name'])) {
                 return $locations[0]['name'];
             }
             return '';
         })
         ->addCombinedMapping('additional_info', ['lang_image_1', 'lang_images_1'], function($image, $images) {
             // Kombiniere zusätzliche Bildinformationen
             $info = [];
             if ($image) $info['main_image'] = $image;
             if ($images) $info['gallery'] = json_decode($images, true);
             return json_encode($info);
         });

$migrator->migrate();
```

In diesem ausführlichen Beispiel:

1. Wir mappen grundlegende Felder wie `id`, `name`, `description`, `start_date`, `end_date` etc. direkt.

2. Wir verwenden `addProcessedMapping` für das `event_type` Feld, um Forcal-spezifische Ereignistypen in allgemeinere Kalendertypen zu konvertieren.

3. Die `addForcalRRuleMapping` Methode wird verwendet, um die komplexen Wiederholungsregeln von Forcal in das standardisierte iCal RRULE Format zu konvertieren.

4. Wir nutzen `addProcessedMapping` für das `awoloc` Feld, um AWO-spezifische Standortinformationen zu extrahieren und zu konvertieren.

5. Mit `addCombinedMapping` fassen wir zusätzliche Bildinformationen in einem JSON-Feld zusammen.

6. Felder wie `createdate`, `updatedate`, `created_by`, und `updated_by` werden direkt gemappt, um Audit-Informationen zu erhalten.

7. Wir behalten auch Felder wie `image` und `file` bei, um Anhänge oder zugehörige Medien zu migrieren.

Dieses Beispiel zeigt, wie flexibel der `TableMigrator` sein kann, um selbst komplexe Migrationsszenarien zu bewältigen. Es berücksichtigt die spezifischen Strukturen und Datenformate der Forcal-Tabelle und transformiert sie in ein allgemeineres Kalenderformat.

Beachten Sie, dass Sie möglicherweise die Zielfelder in Ihrer neuen Kalendertabelle entsprechend anpassen müssen, um alle diese Informationen aufnehmen zu können. Außerdem könnten je nach Ihrer spezifischen Implementierung weitere Anpassungen erforderlich sein.

## Praxis-Beispiel: 

```php
// Usage
$migrator = new TableMigratorx('rex_forcal_entries', 'rex_yformcalendar');
$migrator->addMapping('name', 'name_1')
    ->addProcessedMapping('description', 'text_1', function($text) use ($migrator) {
        return $migrator->truncateAndStripHTML($text);
    })
    ->addRelatedMapping('location', 'awoloc', 'rex_orte', 'name')

    ->addCombinedMapping('dtstart', ['start_date', 'start_time'], function($date, $time) {
        return $date . ' ' . $time;
    })
    ->addCombinedMapping('dtend', ['end_date', 'end_time'], function($date, $time) {
        return $date . ' ' . $time;
    })
    ->addProcessedMapping('all_day', 'full_time', function($value) {
        return $value === null ? 0 : ($value ? 1 : 0);
    })
    ->addMapping('categories', 'category');
        // Fields that couldn't be directly mapped:
        // ->addMapping('createdate', 'createdate')
        // ->addMapping('updatedate', 'updatedate')
        // ->addMapping('created_by', 'createuser')
        // ->addMapping('updated_by', 'updateuser')
        // ->addMapping('status', 'status')
        // ->addMapping('teaser', 'teaser_1')
        // ->addMapping('image', 'image')
        // ->addMapping('file', 'file')
        // ->addMapping('lang', 'lang_1')
        // ->addMapping('category_id', 'category')
        // ->addMapping('location_id', 'venue')
$migrator->migrate();
?>

```




## Hinweise

- Stellen Sie sicher, dass die Zieltabelle bereits existiert und die korrekten Felder enthält.
- Testen Sie die Migration zuerst in einer Entwicklungsumgebung, bevor Sie sie auf Produktionsdaten anwenden.

## ToDo

- Implementierung von Fehlerbehandlung und Logging
- Hinzufügen von Unterstützung für Batch-Verarbeitung bei großen Datenmengen
- Erweiterung um Funktionen zum Rückgängigmachen von Migrationen

## Beitrag

Beiträge sind willkommen! Bitte erstellen Sie ein Issue oder einen Pull Request auf GitHub.

## Lizenz

[MIT License](https://opensource.org/licenses/MIT)





