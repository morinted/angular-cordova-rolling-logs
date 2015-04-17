# Angular Rolling Logs for Cordova Apps

An AngularJS plugin that uses cordova-plugin-file to provide rolling logs
on the device.

Ideal for usage with Cordova apps or libraries like Ionic, you can set up
logs that are saved on the device and rolled over at a size that you specify.

# Getting started

## Installing

To start, you'll need to have a Cordova project using Angular. You'll want to
install the Cordova File plugin:

    cordova plugin add org.apache.cordova.file

Then, add `rolling-log.js` from this repository into your project, and add a
reference to it in your `index.html`, such as:

    <script src="app/lib/rolling-log.js"></script>

## Usage

### Importing

Now that the application is installed, it's time to use it. Anywhere that you
would like to use the logger, you must depend on it. The module is named
`rollingLog` and the factory (service) is named `$roll`. To set it up with
a controller, you might use it like this:

    angular.module('myModule', ['rollingLog'])
    .controller('MyModuleController', [
        '$scope',
        '$roll',
        function ($scope, $roll) {
            $roll.log('Success!')
        };
    ]);

### API

The logger does not start immediately, because it must be started after device
ready. Furthermore, there may be conditions where you do not wish to log. For
that reason, it must be started manually. It also accepts configuration data,
which is better served before starting.

#### Configuration

There is one configuration function, **setConfig**, that takes an
object formatted like the default settings below:

    var defaultSettings = {
        logSize: 25600, // Size of files, in bytes
        eventBuffer: 25, // Number of events before write
        debug: false, // Write debug messages
        console: false, // Write to JS console with $log
        writeOnPause: false,
        prefix: 'log',
        directory: 'dataDirectory',
    };
    $roll.setConfig(defaultSettings);

You can hand it any and all of the options above. Their behavior is:

| Option Name | Description |
| ----------- | ----------- |
| logSize | Integer: Max size of each log file, in bytes. Total storage used at full logging will be twice this. Default is 25.6KB (rather small). |
| eventBuffer | Integer: The logs will wait this number of logs before writing to the file. This way, we are not constantly writing to the file. |
| debug | Boolean: Write messages to file that were called using $roll.debug |
| console | Boolean: Also write messages to console, using Angular $log |
| writeOnPause | Boolean: Add a listener that waits for device pause (screen off, changing app) and write on that event, disregarding the current eventBuffer. |
| prefix | String: the prefix name of your log. Consider the app name to make your log files more recognizable. |
| directory | String: The [Cordova directory constant](https://github.com/apache/cordova-plugin-file#where-to-store-files) where you want to store the logs. By default, dataDirectory is selected -- a non-user-accessable directory. If you want to make your logs user-accessible, refer to the list and consider other options such as 'externalDataDirectory' |

#### Starting the Logger

After your platform is ready and you've provided your configuration,
you can start the logger. To do so, just use:

| Function | Description |
| -------- | ----------- |
| $roll.start() | Returns a promise that resolves after the logging has started. It will reject the promise if there is no Cordova defined or if there is an issue writing to the specified directory. |

Sample usage:

    $roll.start()
        .then(function (success) {
            // It worked and we are started
            $roll.info(success);
            return;
        }, function (error) {
            // Something went wrong... no file saving!
            // But the console should still work.
            $roll.info(error);
            return;
        })
        .finally(function () {
            // No matter if we only have the console or
            // the file system, too, we're going to print
            // some info.
            $roll.info('Device info: ' +
                JSON.stringify(ionic.Platform.device()));
        });

#### Logging Functions

The API is meant to mirror the AngularJS $log behavior, and adds configuration.

The main usage is with:

| Function    | Description |
| ----------- | ----------- |
| $roll.log(String) | Logs the string, if enableConsole is true, it will pass on the string to $log.log() |
| $roll.info(String) | Logs the string, if enableConsole is true, it will pass on the string to $log.info() |
| $roll.error(String) | Logs the string, if enableConsole is true, it will pass on the string to $log.error() |
| $roll.debug(String) | Logs the string, if debug is true. Furthermore, if enableConsole is true, it will pass on the string to $log.debug() |

# Contributing

Please use JSLint and Unix-style line breaks to keep consistent formatting.