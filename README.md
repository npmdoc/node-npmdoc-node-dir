# api documentation for  [node-dir (v0.1.16)](https://github.com/fshost)  [![npm package](https://img.shields.io/npm/v/npmdoc-node-dir.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-node-dir) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-node-dir.svg)](https://travis-ci.org/npmdoc/node-npmdoc-node-dir)
#### asynchronous file and directory operations for Node.js

[![NPM](https://nodei.co/npm/node-dir.png?downloads=true)](https://www.npmjs.com/package/node-dir)

[![apidoc](https://npmdoc.github.io/node-npmdoc-node-dir/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-node-dir_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-node-dir/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-node-dir/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-node-dir/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Nathan Cartwright",
        "email": "fshost@yahoo.com",
        "url": "https://github.com/fshost"
    },
    "bugs": {
        "url": "https://github.com/fshost/node-dir/issues"
    },
    "dependencies": {
        "minimatch": "^3.0.2"
    },
    "description": "asynchronous file and directory operations for Node.js",
    "devDependencies": {
        "mocha": "~1.13.0",
        "should": "~2.0.2"
    },
    "directories": {
        "lib": "lib"
    },
    "dist": {
        "shasum": "d2ef583aa50b90d93db8cdd26fcea58353957fe4",
        "tarball": "https://registry.npmjs.org/node-dir/-/node-dir-0.1.16.tgz"
    },
    "engines": {
        "node": ">= 0.10.5"
    },
    "gitHead": "03510a2089a385b8161d9f480c702811241bbfa2",
    "homepage": "https://github.com/fshost",
    "keywords": [
        "node-dir",
        "directory",
        "dir",
        "subdir",
        "file",
        "asynchronous",
        "Node.js",
        "fs"
    ],
    "license": "MIT",
    "main": "index",
    "maintainers": [
        {
            "name": "fshost",
            "email": "fshost@yahoo.com"
        }
    ],
    "name": "node-dir",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/fshost/node-dir.git"
    },
    "scripts": {
        "test": "mocha --reporter spec"
    },
    "version": "0.1.16"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module node-dir](#apidoc.module.node-dir)
1.  [function <span class="apidocSignatureSpan">node-dir.</span>files (dir, type, callback, ignoreType)](#apidoc.element.node-dir.files)
1.  [function <span class="apidocSignatureSpan">node-dir.</span>paths (dir, combine, callback)](#apidoc.element.node-dir.paths)
1.  [function <span class="apidocSignatureSpan">node-dir.</span>readFiles (dir, options, callback, complete)](#apidoc.element.node-dir.readFiles)
1.  [function <span class="apidocSignatureSpan">node-dir.</span>readFilesStream (dir, options, callback, complete)](#apidoc.element.node-dir.readFilesStream)
1.  [function <span class="apidocSignatureSpan">node-dir.</span>subdirs (dir, callback)](#apidoc.element.node-dir.subdirs)

#### [module node-dir.paths](#apidoc.module.node-dir.paths)
1.  [function <span class="apidocSignatureSpan">node-dir.</span>paths (dir, combine, callback)](#apidoc.element.node-dir.paths.paths)
1.  [function <span class="apidocSignatureSpan">node-dir.paths.</span>files (dir, type, callback, ignoreType)](#apidoc.element.node-dir.paths.files)
1.  [function <span class="apidocSignatureSpan">node-dir.paths.</span>subdirs (dir, callback)](#apidoc.element.node-dir.paths.subdirs)



# <a name="apidoc.module.node-dir"></a>[module node-dir](#apidoc.module.node-dir)

#### <a name="apidoc.element.node-dir.files"></a>[function <span class="apidocSignatureSpan">node-dir.</span>files (dir, type, callback, ignoreType)](#apidoc.element.node-dir.files)
- description and source-code
```javascript
function files(dir, type, callback, ignoreType) {

    var pending,
        results = {
            files: [],
            dirs: []
        };
    var done = function() {
        if (ignoreType || type === 'all') {
            callback(null, results);
        } else {
            callback(null, results[type + 's']);
        }
    };

    var getStatHandler = function(statPath, lstatCalled) {
        return function(err, stat) {
            if (err) {
                if (!lstatCalled) {
                    return fs.lstat(statPath, getStatHandler(statPath, true));
                }
                return callback(err);
            }
            if (stat && stat.isDirectory() && stat.mode !== 17115) {
                if (type !== 'file') {
                    results.dirs.push(statPath);
                }
                files(statPath, type, function(err, res) {
                    if (err) return callback(err);
                    if (type === 'all') {
                        results.files = results.files.concat(res.files);
                        results.dirs = results.dirs.concat(res.dirs);
                    } else if (type === 'file') {
                        results.files = results.files.concat(res.files);
                    } else {
                        results.dirs = results.dirs.concat(res.dirs);
                    }
                    if (!--pending) done();
                }, true);
            } else {
                if (type !== 'dir') {
                    results.files.push(statPath);
                }
                // should be the last statement in statHandler
                if (!--pending) done();
            }
        };
    };

    if (typeof type !== 'string') {
        ignoreType = callback;
        callback = type;
        type = 'file';
    }

    fs.stat(dir, function(err, stat) {
        if (err) return callback(err);
        if(stat && stat.mode === 17115) return done();

        fs.readdir(dir, function(err, list) {
            if (err) return callback(err);
            pending = list.length;
            if (!pending) return done();
            for (var file, i = 0, l = list.length; i < l; i++) {
                file = path.join(dir, list[i]);
                fs.stat(file, getStatHandler(file));
            }
        });
    });
}
```
- example usage
```shell
...
'''


#### files( dir, callback )
Asynchronously iterate the files of a directory and its subdirectories and pass an array of file paths to a callback.

'''javascript
dir.files(__dirname, function(err, files) {
    if (err) throw err;
    console.log(files);
});
'''

Note that for the files and subdirs the object returned is an array, and thus all of the standard array methods are available for
 use in your callback for operations like filters or sorting. Some quick examples:
...
```

#### <a name="apidoc.element.node-dir.paths"></a>[function <span class="apidocSignatureSpan">node-dir.</span>paths (dir, combine, callback)](#apidoc.element.node-dir.paths)
- description and source-code
```javascript
function paths(dir, combine, callback) {

    var type;

    if (typeof combine === 'function') {
        callback = combine;
        combine = false;
    }

    exports.files(dir, 'all', function(err, results) {
        if (err) return callback(err);
        if (combine) {

            callback(null, results.files.concat(results.dirs));
        } else {
            callback(null, results);
        }
    });
}
```
- example usage
```shell
...

#### paths(dir, [combine], callback )
Asynchronously iterate the subdirectories of a directory and its subdirectories and pass an array of both file and directory paths
 to a callback.

Separated into two distinct arrays (paths.files and paths.dirs)

'''javascript
dir.paths(__dirname, function(err, paths) {
    if (err) throw err;
    console.log('files:\n',paths.files);
    console.log('subdirs:\n', paths.dirs);
});
'''
...
```

#### <a name="apidoc.element.node-dir.readFiles"></a>[function <span class="apidocSignatureSpan">node-dir.</span>readFiles (dir, options, callback, complete)](#apidoc.element.node-dir.readFiles)
- description and source-code
```javascript
function readFiles(dir, options, callback, complete) {
    if (typeof options === 'function') {
        complete = callback;
        callback = options;
        options = {};
    }
    if (typeof options === 'string') options = {
        encoding: options
    };
    options = extend({
        recursive: true,
        encoding: 'utf8',
        doneOnErr: true
    }, options);
    var files = [];

    var done = function(err) {
        if (typeof complete === 'function') {
            if (err) return complete(err);
            complete(null, files);
        }
    };

    fs.readdir(dir, function(err, list) {
        if (err)  {
            if (options.doneOnErr === true) {
              if (err.code === 'EACCES') return done();
              return done(err);
            }
        }
        var i = 0;

        if (options.reverse === true ||
            (typeof options.sort == 'string' &&
                (/reverse|desc/i).test(options.sort))) {
            list = list.reverse();
        } else if (options.sort !== false) list = list.sort();

        (function next() {
            var filename = list[i++];
            if (!filename) return done(null, files);
            var file = path.join(dir, filename);
            fs.stat(file, function(err, stat) {
                if (err && options.doneOnErr === true) return done(err);
                if (stat && stat.isDirectory()) {
                    if (options.recursive) {
                        if (options.matchDir && !matches(filename, options.matchDir)) return next();
                        if (options.excludeDir && matches(filename, options.excludeDir)) return next();
                        readFiles(file, options, callback, function(err, sfiles) {
                            if (err && options.doneOnErr === true) return done(err);
                            files = files.concat(sfiles);
                            next();
                        });
                    } else next();
                } else if (stat && stat.isFile()) {
                    if (options.match && !matches(filename, options.match)) return next();
                    if (options.exclude && matches(filename, options.exclude)) return next();
                    if (options.filter && !options.filter(filename)) return next();
                    if (options.shortName) files.push(filename);
                    else files.push(file);
                    fs.readFile(file, options.encoding, function(err, data) {
                        if (err) {
                            if (err.code === 'EACCES') return next();
                            if (options.doneOnErr === true) {
                                return done(err);
                            }
                        }
                        if (callback.length > 3)
                            if (options.shortName) callback(null, data, filename, next);
                            else callback(null, data, file, next);
                            else callback(null, data, next);
                    });
                }
                else {
                    next();
                }
            });
        })();

    });
}
```
- example usage
```shell
...

A reverse sort can also be achieved by setting the sort option to 'reverse', 'desc', or 'descending' string value.

examples

'''javascript
// display contents of files in this script's directory
dir.readFiles(__dirname,
function(err, content, next) {
    if (err) throw err;
    console.log('content:', content);
    next();
},
function(err, files){
    if (err) throw err;
...
```

#### <a name="apidoc.element.node-dir.readFilesStream"></a>[function <span class="apidocSignatureSpan">node-dir.</span>readFilesStream (dir, options, callback, complete)](#apidoc.element.node-dir.readFilesStream)
- description and source-code
```javascript
function readFilesStream(dir, options, callback, complete) {
    if (typeof options === 'function') {
        complete = callback;
        callback = options;
        options = {};
    }
    if (typeof options === 'string') options = {
        encoding: options
    };
    options = extend({
        recursive: true,
        encoding: 'utf8',
        doneOnErr: true
    }, options);
    var files = [];

    var done = function(err) {
        if (typeof complete === 'function') {
            if (err) return complete(err);
            complete(null, files);
        }
    };

    fs.readdir(dir, function(err, list) {
        if (err)  {
            if (options.doneOnErr === true) {
              if (err.code === 'EACCES') return done();
              return done(err);
            }
        }
        var i = 0;

        if (options.reverse === true ||
            (typeof options.sort == 'string' &&
                (/reverse|desc/i).test(options.sort))) {
            list = list.reverse();
        } else if (options.sort !== false) list = list.sort();

        (function next() {
            var filename = list[i++];
            if (!filename) return done(null, files);
            var file = path.join(dir, filename);
            fs.stat(file, function(err, stat) {
                if (err && options.doneOnErr === true) return done(err);
                if (stat && stat.isDirectory()) {
                    if (options.recursive) {
                        if (options.matchDir && !matches(filename, options.matchDir)) return next();
                        if (options.excludeDir && matches(filename, options.excludeDir)) return next();
                        readFilesStream(file, options, callback, function(err, sfiles) {
                            if (err && options.doneOnErr === true) return done(err);
                            files = files.concat(sfiles);
                            next();
                        });
                    } else next();
                } else if (stat && stat.isFile()) {
                    if (options.match && !matches(filename, options.match)) return next();
                    if (options.exclude && matches(filename, options.exclude)) return next();
                    if (options.filter && !options.filter(filename)) return next();
                    if (options.shortName) files.push(filename);
                    else files.push(file);
                    var stream = fs.createReadStream(file);
                    if (options.encoding !== null) {
                        stream.setEncoding(options.encoding);
                    }
                    stream.on('error',function(err) {
                      if (options.doneOnErr === true) return done(err);
                      next();
                    });
                    if (callback.length > 3)
                        if (options.shortName) callback(null, stream, filename, next);
                        else callback(null, stream, file, next);
                        else callback(null, stream, next);
                }
                else {
                  next();
                }
            });
        })();

    });
}
```
- example usage
```shell
...
},
function(err, files){
    if (err) throw err;
    console.log('finished reading files:', files);
});

// display contents of huge files in this script's directory
dir.readFilesStream(__dirname,
function(err, stream, next) {
    if (err) throw err;
    var content = '';
    stream.on('data',function(buffer) {
        content += buffer.toString();
    });
    stream.on('end',function() {
...
```

#### <a name="apidoc.element.node-dir.subdirs"></a>[function <span class="apidocSignatureSpan">node-dir.</span>subdirs (dir, callback)](#apidoc.element.node-dir.subdirs)
- description and source-code
```javascript
function subdirs(dir, callback) {
    exports.files(dir, 'dir', function(err, subdirs) {
        if (err) return callback(err);
        callback(null, subdirs);
    });
}
```
- example usage
```shell
...

Also note that if you need to work with the contents of the files asynchronously, please use the readFiles method.  The files and
 subdirs methods are for getting a list of the files or subdirs in a directory as an array.

#### subdirs( dir, callback )
Asynchronously iterate the subdirectories of a directory and its subdirectories and pass an array of directory paths to a callback
.

'''javascript
dir.subdirs(__dirname, function(err, subdirs) {
    if (err) throw err;
    console.log(subdirs);
});
'''

#### paths(dir, [combine], callback )
Asynchronously iterate the subdirectories of a directory and its subdirectories and pass an array of both file and directory paths
 to a callback.
...
```



# <a name="apidoc.module.node-dir.paths"></a>[module node-dir.paths](#apidoc.module.node-dir.paths)

#### <a name="apidoc.element.node-dir.paths.paths"></a>[function <span class="apidocSignatureSpan">node-dir.</span>paths (dir, combine, callback)](#apidoc.element.node-dir.paths.paths)
- description and source-code
```javascript
function paths(dir, combine, callback) {

    var type;

    if (typeof combine === 'function') {
        callback = combine;
        combine = false;
    }

    exports.files(dir, 'all', function(err, results) {
        if (err) return callback(err);
        if (combine) {

            callback(null, results.files.concat(results.dirs));
        } else {
            callback(null, results);
        }
    });
}
```
- example usage
```shell
...

#### paths(dir, [combine], callback )
Asynchronously iterate the subdirectories of a directory and its subdirectories and pass an array of both file and directory paths
 to a callback.

Separated into two distinct arrays (paths.files and paths.dirs)

'''javascript
dir.paths(__dirname, function(err, paths) {
    if (err) throw err;
    console.log('files:\n',paths.files);
    console.log('subdirs:\n', paths.dirs);
});
'''
...
```

#### <a name="apidoc.element.node-dir.paths.files"></a>[function <span class="apidocSignatureSpan">node-dir.paths.</span>files (dir, type, callback, ignoreType)](#apidoc.element.node-dir.paths.files)
- description and source-code
```javascript
function files(dir, type, callback, ignoreType) {

    var pending,
        results = {
            files: [],
            dirs: []
        };
    var done = function() {
        if (ignoreType || type === 'all') {
            callback(null, results);
        } else {
            callback(null, results[type + 's']);
        }
    };

    var getStatHandler = function(statPath, lstatCalled) {
        return function(err, stat) {
            if (err) {
                if (!lstatCalled) {
                    return fs.lstat(statPath, getStatHandler(statPath, true));
                }
                return callback(err);
            }
            if (stat && stat.isDirectory() && stat.mode !== 17115) {
                if (type !== 'file') {
                    results.dirs.push(statPath);
                }
                files(statPath, type, function(err, res) {
                    if (err) return callback(err);
                    if (type === 'all') {
                        results.files = results.files.concat(res.files);
                        results.dirs = results.dirs.concat(res.dirs);
                    } else if (type === 'file') {
                        results.files = results.files.concat(res.files);
                    } else {
                        results.dirs = results.dirs.concat(res.dirs);
                    }
                    if (!--pending) done();
                }, true);
            } else {
                if (type !== 'dir') {
                    results.files.push(statPath);
                }
                // should be the last statement in statHandler
                if (!--pending) done();
            }
        };
    };

    if (typeof type !== 'string') {
        ignoreType = callback;
        callback = type;
        type = 'file';
    }

    fs.stat(dir, function(err, stat) {
        if (err) return callback(err);
        if(stat && stat.mode === 17115) return done();

        fs.readdir(dir, function(err, list) {
            if (err) return callback(err);
            pending = list.length;
            if (!pending) return done();
            for (var file, i = 0, l = list.length; i < l; i++) {
                file = path.join(dir, list[i]);
                fs.stat(file, getStatHandler(file));
            }
        });
    });
}
```
- example usage
```shell
...
'''


#### files( dir, callback )
Asynchronously iterate the files of a directory and its subdirectories and pass an array of file paths to a callback.

'''javascript
dir.files(__dirname, function(err, files) {
    if (err) throw err;
    console.log(files);
});
'''

Note that for the files and subdirs the object returned is an array, and thus all of the standard array methods are available for
 use in your callback for operations like filters or sorting. Some quick examples:
...
```

#### <a name="apidoc.element.node-dir.paths.subdirs"></a>[function <span class="apidocSignatureSpan">node-dir.paths.</span>subdirs (dir, callback)](#apidoc.element.node-dir.paths.subdirs)
- description and source-code
```javascript
function subdirs(dir, callback) {
    exports.files(dir, 'dir', function(err, subdirs) {
        if (err) return callback(err);
        callback(null, subdirs);
    });
}
```
- example usage
```shell
...

Also note that if you need to work with the contents of the files asynchronously, please use the readFiles method.  The files and
 subdirs methods are for getting a list of the files or subdirs in a directory as an array.

#### subdirs( dir, callback )
Asynchronously iterate the subdirectories of a directory and its subdirectories and pass an array of directory paths to a callback
.

'''javascript
dir.subdirs(__dirname, function(err, subdirs) {
    if (err) throw err;
    console.log(subdirs);
});
'''

#### paths(dir, [combine], callback )
Asynchronously iterate the subdirectories of a directory and its subdirectories and pass an array of both file and directory paths
 to a callback.
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
