fs              = require("fs")
{exec, spawn}   = require("child_process")
path            = require("path")
idl             = require("idl")

compile = (sources) ->
  coffee =          spawn "coffee", "-c -o lib".split(/\s/).concat(sources)
  coffee.stderr.on  "data", (buffer) -> puts buffer.toString()
  coffee.on         "exit", (status) -> process.exit(1) if status != 0

currentBranch = (callback) ->
  branches =        ""
  git =             spawn "git", [ "branch" ]
  git.stdout.on     "data", (buffer) -> branches += buffer.toString()
  git.stderr.on     "data", (buffer) -> puts buffer.toString()
  git.on            "exit", (status) ->
    process.exit(1) if status != 0
    branch = /\*\s+(.*)/.exec(branches)[1]
    callback(branch)

task "gitignore", "create a .gitignore for node-ec2 based on git branch", ->
  currentBranch (branch) ->
    gitignore = '''
                .gitignore
                lib-cov
                .DS_Store
                **/.DS_Store
                
                '''

    if branch is "gh-pages"
      gitignore += '''
                   lib/packet.js
                   lib/pattern.js
                   '''
    else if branch is "master"
      gitignore += '''
                   documentation
                   site
                   index.html
                   lib/packet.js
                   lib/pattern.js
                   '''
    fs.writeFile(".gitignore", gitignore)

task "index", "rebuild the Node IDL landing page.", ->
  package = JSON.parse fs.readFileSync "package.json", "utf8"
  console.log(package)
  idl.generate "#{package.name}.idl", "index.html"

task "docco", "rebuild the CoffeeScript docco documentation.", ->
  exec "rm -rf documentation && docco src/*.coffee && cp -rf docs documentation && rm -r docs", (err) ->
    throw err if err

task "compile", "compile the CoffeeScript into JavaScript", ->
  path.exists "./lib", (exists) ->
    fs.mkdirSync("./lib", parseInt(755, 8)) if not exists
    sources = fs.readdirSync("src")
    sources = "src/" + source for source in sources when source.match(/\.coffee$/)
    compile sources

task "coverage", "run coverage"
  exec "expresso coverage.js  --coverage"

task "test", "run tests"
  exec "vows vows/*.js  --spec"

task "clean", "rebuild the CoffeeScript docco documentation.", ->
  currentBranch (branch) ->
    if branch is "master"
      exec "rm -rf site documentation lib/packet.js lib/pattern.js", (err) ->
        throw err if err
