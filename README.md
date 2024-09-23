# Table_Migrator for REDAXO


```php
// Beispiel für die Verwendung:
$migration = new \klxm\migrator\TableMigrator('forcal_events', 'neue_events_tabelle');

// Einfaches Mapping
$migration->addMapping('id', 'id')
          ->addMapping('title', 'name_1')
          ->addMapping('description', 'text_1')
          ->addMapping('location', 'venue')
          ->addMapping('category', 'category');

// Kombiniertes Mapping für Start- und Enddatum/zeit
$migration->addCombinedMapping('start_datetime', ['start_date', 'start_time'], function($date, $time) {
    return $date . ' ' . $time;
})
->addCombinedMapping('end_datetime', ['end_date', 'end_time'], function($date, $time) {
    return $date . ' ' . $time;
});

// Verarbeitetes Mapping für die Beschreibung (entfernt HTML-Tags)
$migration->addProcessedMapping('clean_description', 'text_1', function($value) {
    return strip_tags($value);
});

// Forcal RRule Mapping
$migration->addForcalRRuleMapping('rrule', [
    'repeat', 'repeat_year', 'repeat_week', 'repeat_month', 
    'repeat_month_week', 'repeat_day', 'end_repeat_date'
]);

// Führt die Migration durch
$migration->migrate();
```
