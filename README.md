Starts Chromium or Google Chrome in headless mode (using Xvfb) and forwards the output from JavaScript console to stdout.

## Usage
`./run-headless-chromium.js [options]`

All `[options]` are directly passed to Chromium. All console messages are forwarded to the
standard output of this process. Every message generated by `console.log`, `console.warn`, etc.
is printed with a newline character at the end, unless the message ends with `\x03\b`. This is
a special character sequence interpreted as "not end of newline".

In addition to the JavaScript messages, errors from Chromium are also printed in the console.
This behavior can be controlled by two environment variables:

* `LOG_CR_VERBOSITY` - Only print messages from Chromium with this verbosity level.  
  Allowed values: Any permutation of `INFO|WARNING|ERROR|ERROR_REPORT|FATAL|VERBOSE|UNKNOWN`,
  defaults to `ERROR|ERROR_REPORT|FATAL`. Use `LOG_CR_VERBOSITY=.` to show all messages.
* `LOG_CR_HIDE_PATTERN` - Exclude messages matching this pattern (case-insensitive).  
  Allowed values: Any regular expression (ECMAScript/JavaScript syntax),
  defaults to `kwallet`.

## Example
Headless Chromium is ideal for unit testing, e.g. with [Jasmine](http://jasmine.github.io/):

### Jasmine 1.3

```html
<script src="jasmine/lib/jasmine-core/jasmine.js"></script>
<script src="jasmine/src/console/ConsoleReporter.js"></script>
<script>
var jasmineEnv = jasmine.getEnv();

var consoleReporter = new jasmine.ConsoleReporter(
    function print(message) {
        // Append magic bytes to signal that the line has not ended yet.
        // This is needed because ConsoleReporter will add the trailing newlines
        // if desired, i.e. it expects console.log to behave as print, not println.
        console.log(message + '\x03\b');
    },
    function onComplete() {
        // Magic string to signal completion of the tests
        console.info('All tests completed!');
    },
    // showColors (whether to generate colorful output)
    true
);

jasmineEnv.addReporter(consoleReporter);
jasmineEnv.execute();
</script>
```

### Jasmine 2.0

```html
<script src="jasmine/lib/jasmine-core/jasmine.js"></script>
<script src="jasmine/lib/console/console.js"></script>
<script>
var jasmineEnv = jasmine.getEnv();

var consoleReporter = new jasmine.ConsoleReporter({
    print: function print(message) {
        console.log(message + '\x03\b');
    },
    onComplete: function onComplete() {
        console.info('All tests completed!');
    },
    showColors: true
});

jasmineEnv.addReporter(consoleReporter);
jasmineEnv.execute();
</script>
```
