# NAME

repconn - a super simple Clojure nREPL client.

# RATIONALE

> This is the Unix philosophy: Write programs that do one thing and do it well.
> Write programs to work together. Write programs to handle text streams,
> because that is a universal interface.
>
> \- Douglas McIlroy


One pain point in Clojure programming and JVM in general is the slow
startup time. Compiling and running even small programs feels sluggish.
This makes the language a less obvious choice for your command line utilities.
Luckily, there are few problems that can't be solved with couple of lines of
Perl \<3

Repconn is a smallish TCP client that slings your code to a running nREPL
server for evaluation. This way small single-file programs can be made to start
significantly faster.

See the examples below for more details.


# REQUIREMENTS

Unless you use Windows you should have recent enough Perl installed. The code
uses only core modules so you don't need to install anything from cpan.

You need to define an environment variable for the server port. You can use any
free port:

    $ export REPCONN_NREPL_PORT=55555

You also need to have a running nREPL server listening on that port. If you use
Leiningen you can start a headless server like this:

    $ nohup lein repl :headless :port $REPCONN_NREPL_PORT >/dev/null &


# INSTALLATION

Download the repconn script, make it executable and place it in your $PATH. /usr/local/bin/ is a good location.

With curl:

    $ curl https://raw.githubusercontent.com/ajk/clojure-repconn/master/script/repconn -o repconn

    $ chmod 0755 repconn

    $ sudo mv repconn /usr/local/bin/


# EXAMPLES

Run code as an argument or from pipe:

    $ repconn '(println "Hello world!")'

    $ echo '(println "Hello" (first argv))' |repconn - Clojure!


Run code from a stand-alone file. Pay attention to the shebang line, this is where it gets interesting:

```clojure

#!/usr/local/bin/repconn
(ns user)

(println "Hello world!")
```

Save the snippet above in a file called hello.clj and run it:

    # pass the filename as argument...
    $ repconn hello.clj

    # ...or run it as an executable
    $ chmod 0755 hello.clj

    $ ./hello.clj


Now we're cooking with fire! What about stdin and stdout? Let's try a poor version of grep:


```clojure

#!/usr/local/bin/repconn
(ns user)

; command-line parameters are automatically
; provided for you in a vector named argv
(def regex (re-pattern (first argv)))

(doseq [line (line-seq *in*)]
  (if (re-find regex line)
    (println line)))
```


Put that in a file, make it executable and try it out:


    $ ./poor-grep aardvark < /usr/share/dict/american-english


Requiring dependencies works as you would expect:

```clojure

#!/usr/local/bin/repconn
(ns user
  (:require [clojure.data.json :as json]
            [org.httpkit.client :as http]))

(-> @(http/get "http://jsonip.com")
    :body
    (json/read-str :key-fn keyword)
    :ip
    println)
```

# PROJECT MATURITY

Although this is a new project it does what it says on the tin. I would suggest
testing thoroughly before mission-critical use.

Most of the originally planned feature set is implemented but please let me
know if you have great ideas.



# SEE ALSO

Alternative projects with similiar goals:

- [Grenchman](http://leiningen.org/grench.html)

- [lein-repls and Cljsh](https://github.com/franks42/lein-repls)


# COPYRIGHT

Copyright 2016 Antti Koskinen

# LICENSE

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see http://www.gnu.org/licenses/

# CONTRIBUTING

Feel free to send bug reports and pull requests on this repository.
