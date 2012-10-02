fs = require("fs")
path = require("path")
watcher = require("watch")
util = require("util")
mkdirp = require("mkdirp")

{spawn} = require("child_process")
exec = (cmd, args=[], options={}, cb=->) ->
  opts = stdio: 'inherit'
  opts[k] = v for k, v of options
  bin = spawn(cmd, args, opts)
  bin.on 'exit', cb

handlebars = ->
  exec 'handlebars', ['-f', 'public/views.js', 'src/client/views/']

build = (watch)->
  mkdirp 'public', ->
    args = if watch then ['-w'] else []
    exec 'coffee', args.concat(['-cbo', 'lib/', 'src/server/'])
    exec 'jspackage', args.concat([
      '-l', 'src/public/vendor',
      '-l', './node_modules/mpd/lib',
      'src/client/app', 'public/app.js'
    ])
    exec 'stylus', args.concat(['-o', 'public/', 'src/client/'])

    # fuck you handlebars
    if watch
      watcher.watchTree 'src/client/views', ignoreDotFiles: true, ->
        handlebars()
        util.log "generated public/views.js"
    else
      handlebars()

    npm_args = if watch then ['run', 'dev'] else ['run', 'build']
    exec 'npm', npm_args, cwd: path.resolve(__dirname, 'mpd.js')

watch = -> build('w')

task "watch", -> watch()

task "build", -> build()

task "clean", ->
  exec "rm", ['-rf', './public', './lib']

task "dev", ->
  watch()
  runServer = -> exec "node-dev", ["lib/server.js"],
    stdio: [process.stdin, process.stdout, process.stderr, 'ipc']
  setTimeout runServer, 1000
