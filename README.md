# sander

A Promise-based power tool for common filesystem tasks in node.js.

## Installation

```bash
npm install sander
```

## Another wrapper around `fs`? Really?

Yup. Working with the low-level `fs` API is the fastest road to callback hell, and while a lot of the existing `fs` wrappers add a whole load of missing features, they don't really mitigate the fundamental suckiness of working with the filesystem in a painful, imperative way, which forces you to handle errors at every step of the journey towards the centre of the node.js [pyramid of doom](http://stackoverflow.com/search?q=pyramid+of+doom).

**Enough! Manual filing is tedious - you need a power tool.** Instead of writing this...

```js
var path = require( 'path' ),
    fs = require( 'fs' ),
    mkdirp = require( 'mkdirp' );

var dest = path.resolve( basedir, filename );

mkdirp( path.dirname( dest ), function ( err ) {
  if ( err ) throw err;

  fs.writeFile( dest, filecontents, function ( err ) {
    if ( err ) throw err;
    doTheNextThing();
  });
});
```

...write this:

```js
require( 'sander' ).writeFile( basedir, filename, filecontents ).then( doTheNextThing );
```


## Conventions

### Promises

All methods return a Promise. If you're not familiar with Promises, [read up on them on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) - they're coming in ES6 and are already supported in many browsers, and I guarantee they'll make your life easier.

(Node doesn't natively support promises yet - we're using [es6-promise](https://github.com/jakearchibald/es6-promise) for maximum compatibility. For convenience, the `Promise` constructor is exposed as `sander.Promise`.)

### Intermediate folder creation

When writing files and folders, intermediate folders are automatically created as necessary. (I've never encountered a situation where I wanted an `ENOENT` error instead of having this be done for me.)

### Automatic path resolution

Wherever appropriate, method arguments are joined together with `path.resolve()` - so the following are equivalent:

```js
sander.readFile( 'foo', 'bar', 'baz' );
sander.readFile( path.resolve( 'foo', 'bar', 'baz' ) );
sander.readFile( 'foo/bar/baz' ); // or 'foo\bar\baz' on Windows
```

### Methods that involve two paths

Some operations, such as renaming files, require two paths to be specified. The convention for handling this in sander is as follows:

```js
sander.rename( basedir, oldname ).to( basedir, newname );
```



## Methods

### `fs` methods

In addition to the extra methods (listed below), all `fs` methods except `watch`/`watchFile`/`unwatchFile`/`createReadStream`/`createWriteStream` have `sander` equivalents. The synchronous methods (those ending `Sync`) are the same as the `fs` originals except that path resolution and intermediate folder creation are automatically handled (see [conventions](#conventions), above). All async methods return a promise.

For more information about what these methods do, consult the [node documentation](http://nodejs.org/api/fs.html).

In the list below, `...paths` indicates you can use one or more strings in sequence, as per the automatic path resolution convention. An `fd` argument refers to a file descriptor, which you'd generate with `sander.open()` or `sander.openSync()`.

```js
sander.appendFile(...paths, data, [options])
sander.appendFileSync(...paths, data, [options])
sander.chmod(...paths, {mode: mode})
sander.chmodSync(...paths, {mode: mode})
sander.chown(...paths, uid, gid)
sander.chownSync(...paths, uid, gid)
sander.close(fd)
sander.closeSync(fd)
sander.fchmod(fd, {mode: mode})
sander.fchmodSync(fd, {mode: mode})
sander.fchown(fd, uid, gid)
sander.fchownSync(fd, uid, gid)
sander.fstat(fd)
sander.fstatSync(fd)
sander.fsync(fd)
sander.fsyncSync(fd)
sander.ftruncate(fd, len)
sander.ftruncateSync(fd, len)
sander.futimes(fd, atime, mtime)
sander.futimesSync(fd, atime, mtime)
sander.lchmod(...paths, {mode: mode})
sander.lchmodSync(...paths, {mode: mode})
sander.lchown(...paths, uid, gid)
sander.lchownSync(...paths, uid, gid)
sander.link(...paths).to(...paths)
sander.linkSync(...paths).to(...paths)
sander.lstat(...paths)
sander.lstatSync(...paths)
sander.mkdir(...paths, [{mode: mode}])
sander.mkdirSync(...paths, [{mode: mode}])
sander.open(...paths, flags, [{mode: mode}])
sander.openSync(...paths, flags, [{mode: mode}])
sander.read(fd, buffer, offset, length, position)
sander.readSync(fd, buffer, offset, length, position)
sander.readdir(...paths)
sander.readdirSync(...paths)
sander.readFile(...paths, [options])
sander.readFileSync(...paths, [options])
sander.readlink(...paths)
sander.readlinkSync(...paths)
sander.realpath(...paths, [cache])
sander.realpathSync(...paths, [cache])
sander.rename(...paths).to(...paths)
sander.renameSync(...paths).to(...paths)
sander.rmdir(...paths)
sander.rmdirSync(...paths)
sander.stat(...paths)
sander.statSync(...paths)
sander.symlink(...paths).to(...paths, [{type: type}])
sander.symlinkSync(...paths).to(...paths, [{type: type}])
sander.truncate(...paths, len)
sander.truncateSync(...paths, len)
sander.unlink(...paths)
sander.unlinkSync(...paths)
sander.utimes(...paths, atime, mtime)
sander.utimesSync(...paths, atime, mtime)
sander.write(fd, buffer, offset, length, position)
sander.writeSync(fd, buffer, offset, length, position)
sander.writeFile(...paths, data, [options])
sander.writeFileSync(...paths, data, [options])
```

Note that with the `chmod`/`fchmod`/`lchmod`/`symlink`/`mkdir`/`open` methods (and their synchronous equivalents), the `mode` and `type` arguments must be passed as objects with a `mode` or `type` property. This is so that sander knows which arguments should be treated as parts of a path (because they're strings) and which shouldn't.


### Extra methods

```js
// Copy a file using streams. `readOptions` is passed to `fs.createReadStream`,
// while `writeOptions` is passed to `fs.createWriteStream`
sander.copyFile(...paths, [readOptions]).to(...paths, [writeOptions]);

// Copy a file synchronously. `readOptions`, is passed to `fs.readFileSync`,
// while `writeOptions` is passed to `fs.writeFileSync`
sander.copyFileSync(...paths, [readOptions]).to(...paths, [writeOptions]);
```

More extras coming soon, e.g. `copydir`/`copydirSync`/`rimraf`/`rimrafSync`.


### License

MIT
