# JER: JavaScript Execution Recorder

TODO: Let Anders pick a proper name

JER records facts about concrete executions of JavaScript programs.

The collection of facts is done as a Jalangi-analysis, and the values are stored as JSON entries in a text file.
In this readme, the text file of JSON entries is known as a "log file".
 
Caveat: This is an unpolished prototype-tool, known limitations are listed in the bottom of this readme, and on the issue tracker of this project.   
 
For examples of how to use the collected values, see the later parts of this readme. 

## Collected facts 

- collected facts are values of expressions, variables and properties at certain locations in programs
- facts are *not* qualified by contexts, they are purely syntactic
- collected values are rarely concrete values (due to efficiency and space concerns)
- collected values are abstracted wrt. the value lattice of [TAJS](https://github.com/cs-au-dk/TAJS)
  - user-allocated objects are abstracted by their allocation site
  - many natively allocated objects are abstracted by their canonical name
  - strings and numbers are abstracted immediately
    
The [JalangiLogFiles](JalangiLogFiles) directory contains some example log files.

## Using collected values 
 
- The [log-readers](log-readers) directory contains implementations for reading the log files.
- The [log-readers/java/src/dk/au/cs/casa/jer/types](log-readers/java/src/dk/au/cs/casa/jer/types) contains Java files that describes what an entry in a log file can look like (see JavaDoc for further information).
- The [log-readers/java/](log-readers/java/) log-reader implementation can be thought of as the log-reader reference implementation.

### Using log files in Java
 
Java version 1.8+ is required to run this implementation.
 
Example: 
```java
Set<IEntry> logEntries = new dk.au.cs.casa.jer.LogParser("myLog.log").getEntries();
  
```

- The script [scripts/make-log-reader-jar.sh](scripts/make-log-reader-jar.sh) produces a jar file at dist/jer.jar.
- jer.jar depends on gson, it is present at [log-readers/java/lib/gson-2.3.1.jar](log-readers/java/lib/gson-2.3.1.jar)


## Collecting more values

If the example log files that have been provided with this project are not sufficient, more can be created.

### Prerequisites

- java (& javac) 1.6+ is required to run the Java-parts of this project
- run `npm install` to install Jalangi and other dependencies


### Creating log files from plain JavaScript files

The file [scripts/createLogFiles](scripts/createLogFiles) can be used to create log files for all JavaScript files in the [test]() directory (recursively).
It does so by essentially recording values appearing during `node test/x/y/z/file.js`.

The log files will be placed in the directory [JalangiLogFiles](JalangiLogFiles) in a subdirectory corresponding to the location of the JavaScript files.

Example:

```
EXAMPLE
```

Creating a log file for a single JavaScript application can be done using [scripts/execute-standalone](scripts/execute-standalone).
 
Example:
```
EXAMPLE
```

### Creating log files from HTML files

The creation of a log file for a HTML file is more involved than for a plain JavaScript file, for two reasons: 
a server that can save log files needs to be started, the HTML file needs to be interacted with in a browser.

The process of obtaining a log file for an HTML file looks like this:

1. run [scripts/instrumentHTMLFiles.sh](scripts/instrumentHTMLFiles.sh) (The instrumented files are placed in a folder named instrumentedHTMLFiles)
2. Run [script/startServer.sh](script/startServer.sh) to start the nodeJS server
3. Open the instrumented HTML-file you wish to create log file for in a browser	
4. Interact with the page and press p to save the log file  (The log file is saved in a folder named nodeJSServer/unchangedLogFiles)
5. Repeat step 3 and 4 for every HTML file that should have a log file 

Due to the way Jalangi uses source locations in HTML files an extra post-processing step is required for every HTML file in nodeJSServer/unchangedLogFiles.

7. Run [scripts/postProcessHTMLLogs.sh](scripts/postProcessHTMLLogs.sh) to adjust the source locations in log files the log files in nodeJSServer/unchangedLogFiles, the adjusted files will be placed in the JalanggiLogFiles directory.

A concrete example of this process:
```
EXAMPLE
```



## Use case: Testing unsoundness of a static analysis

The collected values can be used to find concrete examples of unsoundness in a static analysis.

The collection of values is done using a dynamic analysis that will observe a subset of all potential values.
In order to be sound, a static analysis will over-approximate the set of potential values in the program.
Specifically, the over-approximation needs to include all of the dynamically observed values.
 
A dynamically observed value is not over-approximated by the static analysis, then the analysis is unsound, with the dynamic value as a proof.

Example:

```javascript
EXAMPLE
```

Note that the collected string and number values are abstracted immediately regardless of whether they could be represented by a single concrete value.
This means that a precise and sound analysis can actually under-approximate the collected string and number values without being unsound.

## Misc. limiations and oddities

- Semantic limitations and bugs of [Jalangi](https://github.com/Samsung/jalangi2) will influence the logs
  - the most serious limitation is the improper treatment of the with-statement
- The exact choice of source locations used in the log files can be discussed, for now it is recommended to work around surprising special cases in the tools that use the log files
  - (an alternative to source locations matching is an id assigned to every AST-node using a deterministic tree traversal) 
- log files for JavaScript files is done where only a single JavaScript file is instrumented, obtaining a log file for an entire application is not currently possible
- log files for JavaScript files will have nodejs-semantics and **not** browser-semantics, e.g. the value of `this` is not the global object.

- TODO phantomjs automation of browser interaction for easy reproduction of log files
- TODO cleanup in nodeJSServer: it uses way to many node-packages
- TODO actual JavaDoc in Java log parsing files

## Contributing

Pull requests bug reports on the issue tracker are welcome.