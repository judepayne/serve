# Serve

**Babashka script to dynamically serve a file.**


    serve --help
    
    <serve> is a utility that watches for changes in a file whose content can be served over http, e.g. text, json, svg and serves that content in your browser. When the file is changed, <serve> updates.
    Usage:
    serve <file>
    Options:
      -f, --file <file>           The file to serve. Can be supplied as the main argument without the --file/-f flag.
      -p, --port <port> 8080      The port to host at. Should be a long.
      -i, --ip   <ip>   localhost The ip address to host at.
      -h, --help <help>           Prints this help.


## Installation

#### Manually
With [babashka](https://github.com/babashka/babashka) installed, copy the `serve` script to somewhere in your path and make executable, e.g. `chmod 755 serve`.

#### Install with bbin
bbin is a package manager for babashka.
With [babashka](https://github.com/babashka/babashka) and [bbin](https://github.com/babashka/bbin) installed,

     bbin install https://raw.githubusercontent.com/judepayne/serve/main/serve.clj


## License

`serve` is released under the [MIT License](LICENSE)
