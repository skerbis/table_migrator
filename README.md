# Table_Migrator for REDAXO

Hier ist eine vollständige README für die `TableMigrator` Klasse mit Erklärungen zu den Methoden und Kurzbeispielen:

# TableMigrator

`TableMigrator` ist eine flexible PHP-Klasse für die Migration von Daten zwischen Datenbanktabellen. Sie bietet verschiedene Mapping-Optionen zur Transformation und Kombination von Feldern.

## Installation

Fügen Sie die `TableMigrator.php` Datei zu Ihrem Projekt hinzu und importieren Sie sie:

```php
require_once 'path/to/TableMigrator.php';
use klxm\migrator\TableMigrator;
```

## Grundlegende Verwendung

```php
$migrator = new TableMigrator('old_table', 'new_table');
$migrator->addMapping('new_field', 'old_field');
$migrator->migrate();
```

## Methoden

### Constructor

```php
public function __construct($oldTable, $newTable)
```

Initialisiert einen neuen TableMigrator.

```php
$migrator = new TableMigrator('old_users', 'new_users');
```

### addMapping

```php
public function addMapping($newField, $oldField)
```

Fügt ein einfaches 1:1 Mapping zwischen Feldern hinzu.

```php
$migrator->addMapping('username', 'user_login');
```

### addCombinedMapping

```php
public function addCombinedMapping($newField, $oldFields, $combineFunction)
```

Kombiniert mehrere Felder zu einem neuen Feld.

```php
$migrator->addCombinedMapping('full_name', ['first_name', 'last_name'], function($first, $last) {
    return $first . ' ' . $last;
});
```

### addProcessedMapping

```php
public function addProcessedMapping($newField, $oldField, $processFunction)
```

Wendet eine Funktion auf ein Feld an.

```php
$migrator->addProcessedMapping('hashed_password', 'password', function($value) {
    return password_hash($value, PASSWORD_DEFAULT);
});
```

### addForcalRRuleMapping

```php
public function addForcalRRuleMapping($newField, $oldFields)
```

Spezielle Methode zur Konvertierung von Forcal-Wiederholungsregeln in iCal RRULE Format.

```php
$migrator->addForcalRRuleMapping('rrule', ['repeat', 'repeat_year', 'repeat_week', 'repeat_month', 'repeat_month_week', 'repeat_day', 'end_repeat_date']);
```

### migrate

```php
public function migrate()
```

Führt die Migration basierend auf den definierten Mappings durch.

```php
$migrator->migrate();
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





