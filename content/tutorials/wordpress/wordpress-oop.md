# WordPress: Der objektorientierte Ansatz
Plugins erlauben es, WordPress um neue Funktionalitäten zu erweitern. Wie das Content-Management-System selbst werden sie in PHP geschrieben. Dabei wird ein objektorientierter Ansatz empfohlen, nicht zuletzt, um Namenskonflikte mit anderen Plugins zu vermeiden.[^1]

Objekte wurden bereits relativ zu Beginn in PHP eingeführt, mit PHP 3 zunächst als syntaktischer Zucker für assoziative Arrays.[^2] PHP 4 brachte Methoden und durch PHP 5 wurde es möglich, Sichtbarkeiten (`public`, `private`, `protected`) festzulegen und im Fehlerfall Exceptions zu werfen. Die Sprache wurde seither stetig um neue Features erweitert und heute kann man sagen, dass das objektorientierte Paradigma der Programmierung in einer modernen Form von PHP unterstützt wird.

## Namensräume und Verzeichnisstruktur

## Automatischer Import von Klassendateien

## Serialisierung und Deserialisierung
WordPress stellt Funktionen bereit, mit denen Daten auf komfortable Weise in der Datenbank abgelegt und von dort wieder abgerufen werden können. Beim Speichern von Objekten ist jedoch zu beachten, dass nur *serielle* Daten wie Zahlen, Strings oder Arrays in Datenbanken gespeichert werden können. Komplexe Datenstrukturen und Objekte müssen zunächst *serialisiert* werden.

- Es gibt eine Hauptdatei für das Plugin im Wurzelverzeichnis. Dieses enthält die Informationen über das Plugin.
- Man sollte eine Klasse haben, die das gesamte Plugin repräsentiert. Diese Klasse wird in der Hauptdatei vom Plugin instanziiert. Der Namespace dieser Klasse entspricht dem Klassennamen:
```php
<?php

namespace MyPlugin;

class MyPlugin {
    public function __construct() {
        // Initialization code for your plugin
    }

    public function activate() {
        // Activation code
    }

    public function deactivate() {
        // Deactivation code
    }

    // Other methods and properties for your main plugin functionality
}

// Create an instance of the main plugin class
$my_plugin_instance = new MyPlugin();

// Register activation and deactivation hooks
register_activation_hook(__FILE__, array($my_plugin_instance, 'activate'));
register_deactivation_hook(__FILE__, array($my_plugin_instance, 'deactivate'));
```
Wenn das Plugin in einer *anderen* Datei instanzziiert wird, dann braucht man noch `use MyPlugin\MyPlugin;`.

- Klassen-Dateien werden in `/includes` eingefügt. Sie sollten einen Namespace verwenden. Angenommen das Plugin heißt `ModernPriceList` und die Klasse heißt `Service`.
Dateiname: /includes/ModernPriceListService.php
Dateiinhalt: 
```php
namespace ModernPriceList;

class Service {
...
}
```

- Damit Klassendateien automatisch importiert werden, wenn ein neuer Klassenname im Code angetroffen wird:
```php
// Inside your main plugin file or a separate file for autoloading
spl_autoload_register('modern_price_lists_autoloader');

// Import the namespace for the Service class
use ModernPriceList\Service;

function modern_price_lists_autoloader($class_name) {
    $namespace = 'ModernPriceList\\';

    // Define your base directory
    $base_dir = plugin_dir_path(__FILE__) . 'includes/';

    // Convert class name to file path
    $file_path = $base_dir . str_replace('\\', '/', $namespace . $class_name) . '.php';

    // Include the file if it exists
    if (file_exists($file_path)) {
        include $file_path;
    }
}

// Now you can create an instance of the Service class
$my_new_service = new Service();
```
Beachte, dass trotz des Autoloads noch immer der Namespace importiert werden muss? Sonst muss man `new ModernPriceList.Service()` etc. schreiben.

- Wenn man `namespace ModernPriceList` hat, kann man einfach `use Service` (statt `use ModernPriceList\Service`) schreiben. Voraussetzung ist, dass `namespace ModernPriceList;` in der Datei von `Service` ist. Es wird angenommen, dass `Service` im Namespace `ModernPriceList` ist.
DAS SCHEINT NICHT ZU STIMMEN!

- Objekte müssen serialisiert werden, bevor sie in WordPress-Optionen gespeichert werden können.
```php
private function add_service($service) {
    // Get the existing services from the options
    $existing_services = get_option('services', array());

    // Serialize the Service object before storing in the array
    $existing_services[] = serialize($service);

    // Save the updated array back to the options
    update_option('services', $existing_services);
}
```
Das neue Service-Objekt (das Argument) wird mit `serialize()` zu einem String (serialized string) umgewandelt bevor es dann dem Array hinzugefügt wird, das von der Option repräsentiert wird.
Wenn Objekte aus dem Option-Array wieder herausgenommen werden, müssen sie wieder NICHT UNBEDINGT umgewandelt (deserialized) werden. `get_option()` führt die Deserialization automatisch durch.
"If the option value was serialized, then it will be unserialized when it is returned. In this case the type will be the same. For example, storing a non-scalar value like an array will return the same array."
https://developer.wordpress.org/reference/functions/get_option/
```php
public function services_callback(): void {
    $services = get_option('services', array());

    foreach ($services as $i => $service) {
        // $service = unserialize($service); UNNÖTIG
        ...
    }
}
```

## Fußnoten
[^1]: https://developer.wordpress.org/plugins/plugin-basics/best-practices/#object-oriented-programming-method.
[^2]: Zeev Suraski, einer der beiden Hauptentwickler von PHP in dieser Zeit, hat kurz vor der Veröffentlichung von PHP 5 einen interessanten [Artikel](https://www.devx.com/web-development-zone/10007/) geschrieben, in dem er die Motivation für eine weitreichendere Objektorientierung darlegt. [JetBrains](https://www.jetbrains.com/lp/php-25/) gibt einen allgemeinen Überblick über die Meilensteine in PHP's Geschichte.

## Quellen
https://wpshout.com/is-wordpress-object-oriented/
