fs            = require('fs');
path          = require('path');
_             = require('underscore');
CurlyCS       = require('./lib/curly-cs');
{:spawn, :exec} = require('child_process');
helpers       = require('./lib/curly-cs/helpers');

# Built file header.
header = """
  /**
   * CurlyCS Compiler v#{CurlyCS.VERSION}
   *
   * Copyright 2016, Feijian Sun
   * Released under the MIT License
   */
""";

# Build the CurlyCS language from source.
build = (cb) -> {
  files = fs.readdirSync('src');
  files = ('src/' + file for file in files when file.match(/\.(lit)?ccs$/));
  run(['-c', '-o', 'lib/curly-cs'].concat(files), cb);
}

# Run a CurlyCS Script
run = (args, cb) -> {
  proc = spawn('node', ['bin/ccs'].concat(args));

  proc.stderr.on('data', (buffer) -> { console.log(buffer.toString()); });
  proc.on       ('exit', (status) -> {
    process.exit(1) if status != 0;
    cb() if typeof(cb) is 'function';
  });
}

task('build', 'build the CurlyCS language from source', build);

task('build:full', 'rebuild the source twice', -> {
  build(-> {
    build(-> {
      csPath = './lib/curly-cs';
      csDir  = path.dirname(require.resolve(csPath));

      for mod of require.cache when csDir is mod[0 ... csDir.length] {
        delete(require.cache[mod]);
      }
    });
  });
});

task('build:parser', 'rebuild the Jison parser (run build first)', -> {
  helpers.extend(global, require('util'));
  require('jison');
  parser = require('./lib/curly-cs/grammar').parser;
  fs.writeFile('lib/curly-cs/parser.js', parser.generate());
});

task('build:browser', 'rebuild the merged script for inclusion in the browser', -> {
  code = '';
  for name in ['helpers', 'rewriter', 'lexer', 'parser', 'scope', 'nodes', 'sourcemap', 'curly-cs', 'browser'] {
    code += """
      require['./#{name}'] = (function() {
        var exports = {}, module = {exports: exports};
        #{fs.readFileSync("lib/curly-cs/#{name}.js")}
        return module.exports;
      })();
    """;
  }
  code = """
    (function(root) {
      var CurlyCS = function() {
        function require(path){ return require[path]; }
        #{code}
        return require['./curly-cs'];
      }();

      if (typeof define === 'function' && define.amd) {
        define(function() { return CurlyCS; });
      } else {
        root.CurlyCS = CurlyCS;
      }
    }(this));
  """;
  unless process.env.MINIFY is 'false' {
    {:code} = require('uglify-js').minify(code, {fromString: true});
  }
  fs.writeFileSync('extras/curly-cs.js', header + '\n' + code);
});

