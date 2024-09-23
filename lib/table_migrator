<?php
namespace klxm\migrator;

class TableMigrator
{
    private $oldTable;
    private $newTable;
    private $mappings;
    private $sql;

    public function __construct($oldTable, $newTable)
    {
        $this->oldTable = $oldTable;
        $this->newTable = $newTable;
        $this->mappings = [];
        $this->sql = \rex_sql::factory();
    }

    public function addMapping($newField, $oldField)
    {
        $this->mappings[$newField] = $oldField;
        return $this;
    }

    public function addCombinedMapping($newField, $oldFields, $combineFunction)
    {
        $this->mappings[$newField] = ['fields' => $oldFields, 'function' => $combineFunction];
        return $this;
    }

    public function addProcessedMapping($newField, $oldField, $processFunction)
    {
        $this->mappings[$newField] = ['field' => $oldField, 'process' => $processFunction];
        return $this;
    }

    public function addRelatedMapping($newField, $oldField, $relatedTable, $relatedField, $defaultValue = '')
    {
        $this->mappings[$newField] = [
            'type' => 'related',
            'field' => $oldField,
            'relatedTable' => $relatedTable,
            'relatedField' => $relatedField,
            'defaultValue' => $defaultValue
        ];
        return $this;
    }

    public function migrate()
    {
        $data = $this->sql->getArray("SELECT * FROM {$this->oldTable}");

        foreach ($data as $row) {
            $newData = [];
            foreach ($this->mappings as $newField => $mapping) {
                if (is_array($mapping)) {
                    if (isset($mapping['fields'])) {
                        // Combined mapping
                        $values = array_map(function($field) use ($row) {
                            return $row[$field];
                        }, $mapping['fields']);
                        $newData[$newField] = $mapping['function'](...$values);
                    } elseif (isset($mapping['field']) && isset($mapping['process'])) {
                        // Processed mapping
                        $newData[$newField] = $mapping['process']($row[$mapping['field']]);
                    } elseif (isset($mapping['type']) && $mapping['type'] === 'related') {
                        // Related mapping
                        $relatedValue = $this->getRelatedValue(
                            $row[$mapping['field']],
                            $mapping['relatedTable'],
                            $mapping['relatedField'],
                            $mapping['defaultValue']
                        );
                        $newData[$newField] = $relatedValue;
                    }
                } else {
                    $newData[$newField] = $row[$mapping];
                }
            }

            $this->insertIntoNewTable($newData);
        }

        echo "Migration abgeschlossen!";
    }

    private function getRelatedValue($id, $relatedTable, $relatedField, $defaultValue = 'Unbekannt')
    {
        if ($id === null || $id === '') {
            return $defaultValue;
        }
        
        $id = (int)$id; // Ensure the id is an integer
        
        try {
            $relatedObject = \rex_yform_manager_dataset::get($id, $relatedTable);
            if ($relatedObject) {
                $value = $relatedObject->getValue($relatedField);
                return $value !== null && $value !== '' ? $value : $defaultValue;
            }
            return $defaultValue;
        } catch (\Exception $e) {
            // Log the error or handle it as appropriate for your application
            error_log("Error retrieving related value: " . $e->getMessage());
            return $defaultValue;
        }
    }

    private function insertIntoNewTable($data)
    {
        $fields = implode(', ', array_keys($data));
        $placeholders = ':' . implode(', :', array_keys($data));

        $query = "INSERT INTO {$this->newTable} ($fields) VALUES ($placeholders)";
        $this->sql->setQuery($query, $data);
    }

    public function truncateAndStripHTML($text, $maxLength = 256)
    {
        $text = strip_tags($text);
        $text = html_entity_decode($text, ENT_QUOTES | ENT_HTML5, 'UTF-8');
        
        if (strlen($text) <= $maxLength) {
            return $text;
        }

        $truncated = substr($text, 0, $maxLength);
        
        // Find the position of the last sentence end within the truncated text
        $lastSentenceEnd = 0;
        preg_match_all('/[.!?](?=\s|$)/u', $truncated, $matches, PREG_OFFSET_CAPTURE);
        
        if (!empty($matches[0])) {
            $lastSentenceEnd = $matches[0][count($matches[0]) - 1][1] + 1;
        }

        // If we found a sentence end, use it; otherwise, find the last complete word
        if ($lastSentenceEnd > 0) {
            return trim(substr($text, 0, $lastSentenceEnd));
        } else {
            $lastSpace = strrpos($truncated, ' ');
            return $lastSpace !== false ? trim(substr($text, 0, $lastSpace)) : $truncated;
        }
    }
}

