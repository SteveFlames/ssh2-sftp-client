
# Table of Contents

1.  [Overview](#orgf1a39ab)
2.  [Installation](#org5f5c1fc)
3.  [Basic Usage](#org1230e93)
4.  [Version 6.x Changes](#orgf3a37de)
    1.  [Version 6.0.1](#org75ce52a)
    2.  [Version 6.0.0 Changes](#orgdf7d82a)
5.  [Documentation](#org8cbb843)
    1.  [Specifying Paths](#orgc458cfe)
    2.  [Methods](#orgcd22839)
        1.  [new SftpClient(name) ===> SFTP client object](#org658e86e)
        2.  [connect(config) ===> SFTPstream](#org28aa4ed)
        3.  [list(path, pattern) ==> Array[object]](#org9be9cf9)
        4.  [exists(path) ==> boolean](#orgc5fc3da)
        5.  [stat(path) ==> object](#org1dda009)
        6.  [get(path, dst, options) ==> String|Stream|Buffer](#org535a076)
        7.  [fastGet(remotePath, localPath, options) ===> string](#org04c9225)
        8.  [put(src, remotePath, options) ==> string](#org709ca5d)
        9.  [fastPut(localPath, remotePath, options) ==> string](#orgef53ab9)
        10. [append(input, remotePath, options) ==> string](#org8e5c753)
        11. [mkdir(path, recursive) ==> string](#org44df074)
        12. [rmdir(path, recursive) ==> string](#org9fa1a5d)
        13. [delete(path, noErrorOK) ==> string](#orgfcc2d1b)
        14. [rename(fromPath, toPath) ==> string](#org6150dcb)
        15. [posixRename(fromPath, toPath) ==> string](#org4484482)
        16. [chmod(path, mode) ==> string](#org201af56)
        17. [realPath(path) ===> string](#org516ad1f)
        18. [cwd() ==> string](#orge635443)
        19. [uploadDir(srcDir, dstDir, filter) ==> string](#org3337440)
        20. [downloadDir(srcDir, dstDir, filter) ==> string](#orge49e580)
        21. [end() ==> boolean](#org5d7cb70)
        22. [Add and Remove Listeners](#org0f218dd)
6.  [Platform Quirks & Warnings](#orgefe2bd9)
    1.  [Server Capabilities](#orgea8c46e)
    2.  [Promises & Events](#orgef8ee0b)
    3.  [Windows Based Servers](#orgd10fdaf)
    4.  [Don't Re-use SftpClient Objects](#org553c32d)
7.  [FAQ](#org778c557)
    1.  [Remote server drops connections with only an end event](#orgdd19a09)
    2.  [How can I pass writable stream as dst for get method?](#orgd0c7d17)
    3.  [How can I upload files without having to specify a password?](#org9fa578f)
    4.  [How can I connect through a Socks Proxy](#org0c91c49)
    5.  [Timeout while waiting for handshake or handshake errors](#org2d7770e)
    6.  [How can I limit upload/download speed](#org97e33a3)
8.  [Examples](#orge1dc473)
9.  [Troubleshooting](#orgb2ace97)
    1.  [Common Errors](#org99cbc3b)
        1.  [Not returning the promise in a `then()` block](#org5c0086e)
        2.  [Mixing Promise Chains and Async/Await](#orgf77aa39)
        3.  [Try/catch and Error Handlers](#org61a7a3f)
        4.  [Server Differences](#orgad1a96a)
        5.  [Avoid Concurrent Operations](#org487c6e9)
    2.  [Debugging Support](#orge6f232c)
10. [Logging Issues](#org9299df9)
11. [Pull Requests](#org740d3e3)
12. [Contributors](#org43384a1)



<a id="orgf1a39ab"></a>

# Overview

an SFTP client for node.js, a wrapper around [SSH2](https://github.com/mscdex/ssh2)  which provides a high level
convenience abstraction as well as a Promise based API.

Documentation on the methods and available options in the underlying modules can
be found on the [SSH2](https://github.com/mscdex/ssh2) and [SSH2-STREAMS](https://github.com/mscdex/ssh2-streams/blob/master/SFTPStream.md)  project pages.

Current stable release is **v6.0.1**.

Code has been tested against Node versions 12.18.2 and 13.14.0

Node versions < 10.x are not supported.

<span class="underline">WARNING</span> There is currently a regression error with versions of node later than
version 14.0. It appears that when using streams with chunk sizes which exceed
the high water mark for the stream, a drain event is no longer emitted. As a
result, streams with sufficient data will hang indefinitely. This appears to
affect fastput, fastget, put and possibly get operations. Until this issue is
resolved and a new version of ssh2/ssh2-streams is released, using node v14 is
not recommended.

A bug report hass been logged against the ssh2-streams library as [issue 156](https://github.com/mscdex/ssh2-streams/issues/156).

<span class="underline">UPDATE</span>: The author of the upstream ssh2 and ssh2-streams module has decided on
a re-write of the ssh2 module to address the above issues as well as some other
design limitations and to allow the module to better fit in with newer versions
of node. As part of that process, the functionality of ssh2-streams is being
incorporated into the main ssh2 module and the ssh2-streams module is being
deprecated. This will require a significant update to this module and may result
in some API changes, depending on what changes in the re-write of ssh2.

To support these changes, a new branch called *version-6* has been created. This
branch will use the newest version of ssh2 and for now is very much experimental
and subject to change.

<span class="underline">UPDATE</span>: Apparently the change in core node which cause the issue with ssh2 has
been rolled back in node version 15.3.0. Testing seems to indicate the above
issue does not exist in that version of node.


<a id="org5f5c1fc"></a>

# Installation

    npm install ssh2-sftp-client


<a id="org1230e93"></a>

# Basic Usage

    let Client = require('ssh2-sftp-client');
    let sftp = new Client();
    
    sftp.connect({
      host: '127.0.0.1',
      port: '8080',
      username: 'username',
      password: '******'
    }).then(() => {
      return sftp.list('/pathname');
    }).then(data => {
      console.log(data, 'the data info');
    }).catch(err => {
      console.log(err, 'catch error');
    });


<a id="orgf3a37de"></a>

# Version 6.x Changes


<a id="org75ce52a"></a>

## Version 6.0.1

-   Fix issue with connect retry not releasing 'ready' listener.
-   Add finally clauses to all promises to ensure release of temporary listeners.
-   Add nyc module to improve test coverage
-   Added additional utils tests to improve test coverage
-   Removed some unnecessary util functions to reduce code size


<a id="orgdf7d82a"></a>

## Version 6.0.0 Changes

-   Added new optional argument *notFoundOK* to `delete()` method. If true, no
    error is thrown when trying to delete a file which does not exist. Default is
    false.
-   Added new filter argument to `uploadDir()` and `downloadDir()` methods. The
    filter argument is a regular expression used to match the files and
    directories to be included in the upload or download. Defaults to match all
    files and directories.
-   New event handling approach. Have standardised the adding and removing of
    event handlers as well as added event handlers for *close* and *end* events.
    Some sftp servers appear to abruptly terminate connections without issuing an
    *error* event. This could result in failing to resolve the promise associated
    with the action being performed. Adding *close* and *end* listeners to reject
    a promise should prevent the scripts from hanging when the remote server does
    not return either a status or an error.
-   Replace the `retry` library module with the `promise-retry` library module.
    This module also uses the `retry` library, but at a higher level abstracted
    for use with Promises. As a result, the number of failed retries is no longer
    returned as part of the error message when all attempted connection retries
    fail.
-   Reduced argument verification code. Version 5.x added a lot of additional
    argument verification code in an attempt to provide more meaningful error
    messages. While this goal was achieved, it did have a significant performance
    impact, especially with respect to small file transfers. This additional
    argument verification has now been removed in favour of faster code. The
    downside is that some error messages have changed and in some cases, will not
    be as meaningful or will be more generic in nature. This seems as a reasonable
    compromise. It may result in increased difficulty tracking down why a script
    is failing, but once a script is working, it should be more efficient. If your
    code relies on the text in error messages, you will need to verify whether the
    text is still the same in any testing and adjust where necessary.


<a id="org8cbb843"></a>

# Documentation

The connection options are the same as those offered by the underlying SSH2
module. For full details, please see [SSH2 client methods](https://github.com/mscdex/ssh2#user-content-client-methods)

All the methods will return a Promise, except for `on()` and
`removeListener()`, which are typically only used in special use cases.


<a id="orgc458cfe"></a>

## Specifying Paths

The convention with both FTP and SFTP is that paths are specified using a
'nix' style i.e. use `/` as the path separator. This means that even if your
SFTP server is running on a win32 platform, you should use `/` instead of `\`
as the path separator. For example, for a win32 path of `C:\Users\fred` you
would actually use `/C:/Users/fred`. If your win32 server does not support
the 'nix' path convention, you can try setting the `remotePathSep` property
of the `SftpClient` object to the path separator of your remote server. This
**might** work, but has not been tested. Please let me know if you need to do
this and provide details of the SFTP server so that I can try to create an
appropriate environment and adjust things as necessary. At this point, I'm
not aware of any win32 based SFTP servers which do not support the 'nix' path
convention.

All remote paths must either be absolute e.g. `/absolute/path/to/file` or
they can be relative with a prefix of either `./` (relative to current remote
directory) or `../` (relative to parent of current remote directory) e.g.
`./relative/path/to/file` or `../relative/to/parent/file`. It is also
possible to do things like `../../../file` to specify the parent of the
parent of the parent of the current remote directory. The shell tilde (`~`)
and common environment variables like `$HOME` are NOT supported.

It is important to recognise that the current remote directory may not always
be what you may expect. A lot will depend on the remote platform of the SFTP
server and how the SFTP server has been configured. When things don't seem to
be working as expected, it is often a good idea to verify your assumptions
regarding the remote directory and remote paths. One way to do this is to
login using a command line program like `sftp` or `lftp`.

There is a small performance hit for using `./` and `../` as the module must
query the remote server to determine what the root path is and derive the
absolute path. Using absolute paths are therefore more efficient and likely
more robust.

When specifying file paths, ensure to include a full path i.e. include the
remote filename. Don't expect the module to append the local file name to the
path you provide. For example, the following will not work

    client.put('/home/fred/test.txt', '/remote/dir');

will not result in the file `test.txt` being copied to
`/remote/dir/test.txt`. You need to specify the target filename as well e.g.

    client.put('/home/fred/test.txt', '/remote/dir/test.txt');

Note that the remote file name does not have to be the same as the local file
name. The following works fine;

    client.put('/home/fred/test.txt', '/remote/dir/test-copy.txt');

This will copy the local file `test.txt` to the remote file `test-copy.txt`
in the directory `/remote/dir`.


<a id="orgcd22839"></a>

## Methods


<a id="org658e86e"></a>

### new SftpClient(name) ===> SFTP client object

Constructor to create a new `ssh2-sftp-client` object. An optional `name` string
can be provided, which will be used in error messages to help identify which
client has thrown the error.

1.  Constructor Arguments

    -   **name:** string. An optional name string used in error messages

2.  Example Use

        'use strict';
        
        const Client = require('ssh2-sftp-client');
        
        const config = {
          host: 'example.com',
          username: 'donald',
          password: 'my-secret'
        };
        
        const sftp = new Client('example-client');
        
        sftp.connect(config)
          .then(() => {
            return sftp.cwd();
          })
          .then(p => {
            console.log(`Remote working directory is ${p}`);
            return sftp.end();
          })
          .catch(err => {
            console.log(`Error: ${err.message}`); // error message will include 'example-client'
          });


<a id="org28aa4ed"></a>

### connect(config) ===> SFTPstream

Connect to an sftp server. Full documentation for connection options is
available [here](https://github.com/mscdex/ssh2#user-content-client-methods)

1.  Connection Options

    This module is based on the excellent [SSH2](https://github.com/mscdex/ssh2#client) module. That module is a general SSH2
    client and server library and provides much more functionality than just SFTP
    connectivity. Many of the connect options provided by that module are less
    relevant for SFTP connections. It is recommended you keep the config options to
    the minimum needed and stick to the options listed in the `commonOpts` below.
    
    The `retries`, `retry_factor` and `retry_minTimeout` options are not part of the
    SSH2 module. These are part of the configuration for the [retry](https://www.npmjs.com/package/retry) package and what
    is used to enable retrying of sftp connection attempts. See the documentation
    for that package for an explanation of these values.
    
        // common options
        
        let commonOpts {
          host: 'localhost', // string Hostname or IP of server.
          port: 22, // Port number of the server.
          forceIPv4: false, // boolean (optional) Only connect via IPv4 address
          forceIPv6: false, // boolean (optional) Only connect via IPv6 address
          username: 'donald', // string Username for authentication.
          password: 'borsch', // string Password for password-based user authentication
          agent: process.env.SSH_AGENT, // string - Path to ssh-agent's UNIX socket
          privateKey: fs.readFileSync('/path/to/key'), // Buffer or string that contains
          passphrase: 'a pass phrase', // string - For an encrypted private key
          readyTimeout: 20000, // integer How long (in ms) to wait for the SSH handshake
          strictVendor: true // boolean - Performs a strict server vendor check
          debug: myDebug // function - Set this to a function that receives a single
                        // string argument to get detailed (local) debug information.
          retries: 2 // integer. Number of times to retry connecting
          retry_factor: 2 // integer. Time factor used to calculate time between retries
          retry_minTimeout: 2000 // integer. Minimum timeout between attempts
        };
        
        // rarely used options
        
        let advancedOpts {
          localAddress,
          localPort,
          hostHash,
          hostVerifier,
          agentForward,
          localHostname,
          localUsername,
          tryKeyboard,
          authHandler,
          keepaliveInterval,
          keepaliveCountMax,
          sock,
          algorithms,
          compress
        };

2.  Example Use

        sftp.connect({
          host: example.com,
          port: 22,
          username: 'donald',
          password: 'youarefired'
        });


<a id="org9be9cf9"></a>

### list(path, pattern) ==> Array[object]

Retrieves a directory listing. This method returns a Promise, which once
realised, returns an array of objects representing items in the remote
directory.

-   **path:** {String} Remote directory path
-   **pattern:** (optional) {string|RegExp} A pattern used to filter the items included in the returned
    array. Pattern can be a simple *glob*-style string or a regular
    expression. Defaults to `/.*/`.

1.  Example Use

        const Client = require('ssh2-sftp-client');
        
        const config = {
          host: 'example.com',
          port: 22,
          username: 'red-don',
          password: 'my-secret'
        };
        
        let sftp = new Client;
        
        sftp.connect(config)
          .then(() => {
            return sftp.list('/path/to/remote/dir');
          })
          .then(data => {
            console.log(data);
          })
          .then(() => {
            sftp.end();
          })
          .catch(err => {
            console.error(err.message);
          });

2.  Return Objects

    The objects in the array returned by `list()` have the following properties;
    
        {
          type: // file type(-, d, l)
          name: // file name
          size: // file size
          modifyTime: // file timestamp of modified time
          accessTime: // file timestamp of access time
          rights: {
            user:
            group:
            other:
          },
          owner: // user ID
          group: // group ID
        }

3.  Pattern Filter

    The filter options can be a regular expression (most powerful option) or a
    simple *glob*-like string where \* will match any number of characters, e.g.
    
        foo* => foo, foobar, foobaz
        *bar => bar, foobar, tabbar
        *oo* => foo, foobar, look, book
    
    The *glob*-style matching is very simple. In most cases, you are best off using
    a real regular expression which will allow you to do more powerful matching and
    anchor matches to the beginning/end of the string etc.


<a id="orgc5fc3da"></a>

### exists(path) ==> boolean

Tests to see if remote file or directory exists. Returns type of remote object
if it exists or false if it does not.

1.  Example Use

        const Client = require('ssh2-sftp-client');
        
        const config = {
          host: 'example.com',
          port: 22,
          username: 'red-don',
          password: 'my-secret'
        };
        
        let sftp = new Client;
        
        sftp.connect(config)
          .then(() => {
            return sftp.exists('/path/to/remote/dir');
          })
          .then(data => {
            console.log(data);          // will be false or d, -, l (dir, file or link)
          })
          .then(() => {
            sftp.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org1dda009"></a>

### stat(path) ==> object

Returns the attributes associated with the object pointed to by `path`.

-   **path:** String. Remote path to directory or file on remote server

1.  Attributes

    The `stat()` method returns an object with the following properties;
    
        let stats = {
          mode: 33279, // integer representing type and permissions
          uid: 1000, // user ID
          gid: 985, // group ID
          size: 5, // file size
          accessTime: 1566868566000, // Last access time. milliseconds
          modifyTime: 1566868566000, // last modify time. milliseconds
          isDirectory: false, // true if object is a directory
          isFile: true, // true if object is a file
          isBlockDevice: false, // true if object is a block device
          isCharacterDevice: false, // true if object is a character device
          isSymbolicLink: false, // true if object is a symbolic link
          isFIFO: false, // true if object is a FIFO
          isSocket: false // true if object is a socket
        };

2.  Example Use

        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.stat('/path/to/remote/file');
          })
          .then(data => {
            // do something with data
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org535a076"></a>

### get(path, dst, options) ==> String|Stream|Buffer

Retrieve a file from a remote SFTP server. The `dst` argument defines the
destination and can be either a string, a stream object or undefined. If it is a
string, it is interpreted as the path to a location on the local file system
(path should include the file name). If it is a stream object, the remote data
is passed to it via a call to pipe(). If `dst` is undefined, the method will put
the data into a buffer and return that buffer when the Promise is resolved. If
`dst` is defined, it is returned when the Promise is resolved.

In general, if your going to pass in a string as the destination, you are
better off using the `fastGet()` method.

-   **path:** String. Path to the remote file to download
-   **dst:** String|Stream. Destination for the data. If a string, it
    should be a local file path.
-   **options:** Options for the `get()` command (see below).

1.  Options

    The options object can be used to pass options to the underlying readStream used
    to read the data from the remote server.
    
        {
          flags: 'r',
          encoding: null,
          handle: null,
          mode: 0o666,
          autoClose: true
        }
    
    Most of the time, you won't want to use any options. Sometimes, it may be useful
    to set the encoding. For example, to 'utf-8'. However, it is important not to do
    this for binary files to avoid data corruption.

2.  Example Use

        let client = new Client();
        
        let remotePath = '/remote/server/path/file.txt';
        let dst = fs.createWriteStream('/local/file/path/copy.txt');
        
        client.connect(config)
          .then(() => {
            return client.get(remotePath, dst);
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });
    
    -   **Tip:** See examples file in the Git repository for more examples. You can pass
        any writeable stream in as the destination. For example, if you pass in
        `zlib.createGunzip()` writeable stream, you can both download and
        decompress a gzip file 'on the fly'.


<a id="org04c9225"></a>

### fastGet(remotePath, localPath, options) ===> string

Downloads a file at remotePath to localPath using parallel reads for faster
throughput. This is the simplest method if you just want to download a file.

-   **remotePath:** String. Path to the remote file to download
-   **localPath:** String. Path on local file system for the downloaded file. The
    local path should include the filename to use for saving the
    file.
-   **options:** Options for `fastGet()` (see below)

1.  Options

        {
          concurrency: 64, // integer. Number of concurrent reads to use
          chunkSize: 32768, // integer. Size of each read in bytes
          step: function(total_transferred, chunk, total) // callback called each time a
                                                          // chunk is transferred
        }
    
    -   **Warning:** Some servers do not respond correctly to requests to alter chunk
        size. This can result in lost or corrupted data.

2.  Sample Use

        let client = new Client();
        let remotePath = '/server/path/file.txt';
        let localPath = '/local/path/file.txt';
        
        client.connect(config)
          .then(() => {
            client.fastGet(remotePath, localPath);
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org709ca5d"></a>

### put(src, remotePath, options) ==> string

Upload data from local system to remote server. If the `src` argument is a
string, it is interpreted as a local file path to be used for the data to
transfer. If the `src` argument is a buffer, the contents of the buffer are
copied to the remote file and if it is a readable stream, the contents of that
stream are piped to the `remotePath` on the server.

-   **src:** string | buffer | readable stream. Data source for data to copy to the
    remote server.
-   **remotePath:** string. Path to the remote file to be created on the server.
-   **options:** object. Options which can be passed to adjust the write stream used
    in sending the data to the remote server (see below).

1.  Options

    The following options are supported;
    
        {
          flags: 'w',  // w - write and a - append
          encoding: null, // use null for binary files
          mode: 0o666, // mode to use for created file (rwx)
          autoClose: true // automatically close the write stream when finished
        }
    
    The most common options to use are mode and encoding. The values shown above are
    the defaults. You do not have to set encoding to utf-8 for text files, null is
    fine for all file types. However, using utf-8 encoding for binary files will
    often result in data corruption.

2.  Example Use

        let client = new Client();
        
        let data = fs.createReadStream('/path/to/local/file.txt');
        let remote = '/path/to/remote/file.txt';
        
        client.connect(config)
          .then(() => {
            return client.put(data, remote);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });
    
    -   **Tip:** If the src argument is a path string, consider just using `fastPut()`.


<a id="orgef53ab9"></a>

### fastPut(localPath, remotePath, options) ==> string

Uploads the data in file at `localPath` to a new file on remote server at
`remotePath` using concurrency. The options object allows tweaking of the fast put process.

-   **localPath:** string. Path to local file to upload
-   **remotePath:** string. Path to remote file to create
-   **options:** object. Options passed to createWriteStream (see below)

1.  Options

        {
          concurrency: 64, // integer. Number of concurrent reads
          chunkSize: 32768, // integer. Size of each read in bytes
          mode: 0o755, // mixed. Integer or string representing the file mode to set
          step: function(total_transferred, chunk, total) // function. Called every time
          // a part of a file was transferred
        }
    
    -   **Warning:** There have been reports that some SFTP servers will not honour
        requests for non-default chunk sizes. This can result in data loss
        or corruption.

2.  Example Use

        let localFile = '/path/to/file.txt';
        let remoteFile = '/path/to/remote/file.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            client.fastPut(localFile, remoteFile);
          })
          .then(() => {
            client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org8e5c753"></a>

### append(input, remotePath, options) ==> string

Append the `input` data to an existing remote file. There is no integrity
checking performed apart from normal writeStream checks. This function simply
opens a writeStream on the remote file in append mode and writes the data passed
in to the file.

-   **input:** buffer | readStream. Data to append to remote file
-   **remotePath:** string. Path to remote file
-   **options:** object. Options to pass to writeStream (see below)

1.  Options

    The following options are supported;
    
        {
          flags: 'a',  // w - write and a - append
          encoding: null, // use null for binary files
          mode: 0o666, // mode to use for created file (rwx)
          autoClose: true // automatically close the write stream when finished
        }
    
    The most common options to use are mode and encoding. The values shown above are
    the defaults. You do not have to set encoding to utf-8 for text files, null is
    fine for all file types. Generally, I would not attempt to append binary files.

2.  Example Use

        let remotePath = '/path/to/remote/file.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.append(Buffer.from('Hello world'), remotePath);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org44df074"></a>

### mkdir(path, recursive) ==> string

Create a new directory. If the recursive flag is set to true, the method will
create any directories in the path which do not already exist. Recursive flag
defaults to false.

-   **path:** string. Path to remote directory to create
-   **recursive:** boolean. If true, create any missing directories in the path as
    well

1.  Example Use

        let remoteDir = '/path/to/new/dir';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.mkdir(remoteDir, true);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org9fa1a5d"></a>

### rmdir(path, recursive) ==> string

Remove a directory. If removing a directory and recursive flag is set to
`true`, the specified directory and all sub-directories and files will be
deleted. If set to false and the directory has sub-directories or files, the
action will fail.

-   **path:** string. Path to remote directory
-   **recursive:** boolean. If true, remove all files and directories in target
    directory. Defaults to false

**Note**: There has been at least one report that some SFTP servers will allow
non-empty directories to be removed even without the recursive flag being set to
true. While this is not standard behaviour, it is recommended that users verify
the behaviour of rmdir if there are plans to rely on the recursive flag to
prevent removal of non-empty directories.

1.  Example Use

        let remoteDir = '/path/to/remote/dir';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.rmdir(remoteDir, true);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="orgfcc2d1b"></a>

### delete(path, noErrorOK) ==> string

Delete a file on the remote server.

-   **path:** string. Path to remote file to be deleted.

-   **noErrorOK:** boolean. If true, no error is raised when you try to delete a
    non-existent file. Default is false.

1.  Example Use

        let remoteFile = '/path/to/remote/file.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.delete(remoteFile);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org6150dcb"></a>

### rename(fromPath, toPath) ==> string

Rename a file or directory from `fromPath` to `toPath`. You must have the
necessary permissions to modify the remote file.

-   **fromPath:** string. Path to existing file to be renamed
-   **toPath:** string. Path to new file existing file is to be renamed to. Should
    not already exist.

1.  Example Use

        let from = '/remote/path/to/old.txt';
        let to = '/remote/path/to/new.txt';
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.rename(from, to);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org4484482"></a>

### posixRename(fromPath, toPath) ==> string

This method uses the openssh POSIX rename extension introduced in OpenSSH 4.8.
The advantage of this version of rename over standard SFTP rename is that it is
an atomic operation and will allow renaming a resource where the destination
name exists. The POSIX rename will also work on some filesystems which do not
support standard SFTP rename because they don't support the system hardlink()
call. The POSIX rename extension is available on all openSSH servers from 4.8
and some other implementations. This is an extension to the standard SFTP
protocol and therefore is not supported on all sSFTP servers.

-   **fromPath:** string. Path to existing file to be renamed.
-   **toPath:** string. Path for new name. If it already exists, it will be replaced
    by file specified in fromPath

    let from = '/remote/path/to/old.txt';
    let to = '/remote/path/to/new.txt';
    let client = new Client();
    
    client.connect(config)
      .then(() => {
        return client.posixRename(from, to);
      })
      .then(() => {
        return client.end();
      })
      .catch(err => {
        console.error(err.message);
      });


<a id="org201af56"></a>

### chmod(path, mode) ==> string

Change the mode (read, write or execute permissions) of a remote file or
directory.

-   **path:** string. Path to the remote file or directory
-   **mode:** octal. New mode to set for the remote file or directory

1.  Example Use

        let path = '/path/to/remote/file.txt';
        let newMode = 0o644;  // rw-r-r
        let client = new Client();
        
        client.connect(config)
          .then(() => {
            return client.chmod(path, newMode);
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org516ad1f"></a>

### realPath(path) ===> string

Converts a relative path to an absolute path on the remote server. This method
is mainly used internally to resolve remote path names.

**Warning**: Currently, there is a platform inconsistency with this method on
win32 platforms. For servers running on non-win32 platforms, providing a path
which does not exist on the remote server will result in an empty e.g. '',
absolute path being returned. On servers running on win32 platforms, a
normalised path will be returned even if the path does not exist on the remote
server. It is therefore advised not to use this method to also verify a path
exists. instead, use the `exist()` method.

-   **path:** A file path, either relative or absolute. Can handle '.' and '..', but
    does not expand '~'.


<a id="orge635443"></a>

### cwd() ==> string

Returns what the server believes is the current remote working directory.


<a id="org3337440"></a>

### uploadDir(srcDir, dstDir, filter) ==> string

Upload the directory specified by `srcDir` to the remote directory specified by
`dstDir`. The `dstDir` will be created if necessary. Any sub directories within
`srcDir` will also be uploaded. Any existing files in the remote path will be
overwritten.

The upload process also emits 'upload' events. These events are fired for each
successfully uploaded file. The `upload` event calls listeners with 1 argument,
an object which has properties source and destination. The source property is
the path of the file uploaded and the destination property is the path to where
the file was uploaded to. The purpose of this event is to provide some way for
client code to get feedback on the upload progress. You can add your own lisener
using the `on()` method.

The optionsl *filter* argument is a regular expression which can be used to
select which files and directories to include in the upload.

-   **srcDir:** A local file path specified as a string
-   **dstDir:** A remote file path specified as a string
-   **filter:** A regular expression used to filter which files and directories to
    include in the upload

1.  Example

            'use strict';
        
            // Example of using the uploadDir() method to upload a directory
            // to a remote SFTP server
        
            const path = require('path');
            const SftpClient = require('../src/index');
        
            const dotenvPath = path.join(__dirname, '..', '.env');
            require('dotenv').config({path: dotenvPath});
        
            const config = {
        host: process.env.SFTP_SERVER,
        username: process.env.SFTP_USER,
        password: process.env.SFTP_PASSWORD,
        port: process.env.SFTP_PORT || 22
            };
        
            async function main() {
        const client = new SftpClient('upload-test');
        const src = path.join(__dirname, '..', 'test', 'testData', 'upload-src');
        const dst = '/home/tim/upload-test';
        
        try {
          await client.connect(config);
          client.on('upload', info => {
            console.log(`Listener: Uploaded ${info.source}`);
          });
          let rslt = await client.uploadDir(src, dst);
          return rslt;
        } finally {
          client.end();
        }
            }
        
            main()
        .then(msg => {
          console.log(msg);
        })
        .catch(err => {
          console.log(`main error: ${err.message}`);
        });


<a id="orge49e580"></a>

### downloadDir(srcDir, dstDir, filter) ==> string

Download the remote directory specified by `srcDir` to the local file system
directory specified by `dstDir`. The `dstDir` directory will be created if
required. All sub directories within `srcDir` will also be copied. Any existing
files in the local path will be overwritten. No files in the local path will be
deleted.

The method also emites `download` events to provide a way to monitor download
progress. The download event listener is called with one argument, an object
with two properties, source and destination. The source property is the path to
the remote file that has been downloaded and the destination is the local path
to where the file was downloaded to. You can add a listener for this event using
the `on()` method.

The optional *filter* argument is a regular expression which can be used to
select which files and directories will be downloaded from the remote server.

-   **srcDir:** A remote file path specified as a string
-   **dstDir:** A local file path specified as a string
-   **filter:** A regular expression used to match the files and directories to be
    downloaded

1.  Example

        'use strict';
        
        // Example of using the downloadDir() method to upload a directory
        // to a remote SFTP server
        
        const path = require('path');
        const SftpClient = require('../src/index');
        
        const dotenvPath = path.join(__dirname, '..', '.env');
        require('dotenv').config({path: dotenvPath});
        
        const config = {
          host: process.env.SFTP_SERVER,
          username: process.env.SFTP_USER,
          password: process.env.SFTP_PASSWORD,
          port: process.env.SFTP_PORT || 22
        };
        
        async function main() {
          const client = new SftpClient('upload-test');
          const dst = '/tmp';
          const src = '/home/tim/upload-test';
        
          try {
            await client.connect(config);
            client.on('download', info => {
        console.log(`Listener: Download ${info.source}`);
            });
            let rslt = await client.downloadDir(src, dst);
            return rslt;
          } finally {
            client.end();
          }
        }
        
        main()
          .then(msg => {
            console.log(msg);
          })
          .catch(err => {
            console.log(`main error: ${err.message}`);
          });


<a id="org5d7cb70"></a>

### end() ==> boolean

Ends the current client session, releasing the client socket and associated
resources. This function also removes all listeners associated with the client.

1.  Example Use

        let client = new Client();
        
        client.connect(config)
          .then(() => {
            // do some sftp stuff
          })
          .then(() => {
            return client.end();
          })
          .catch(err => {
            console.error(err.message);
          });


<a id="org0f218dd"></a>

### Add and Remove Listeners

Although normally not required, you can add and remove custom listeners on the
ssh2 client object. This object supports a number of events, but only a few of
them have any meaning in the context of SFTP. These are

-   **error:** An error occurred. Calls listener with an error argument.
-   **end:** The socket has been disconnected. No argument.
-   **close:** The socket was closed. Boolean argument which is true when the socket
    was closed due to errors.

1.  on(eventType, listener)

    Adds the specified listener to the specified event type. It the event type is
    `error`, the listener should accept 1 argument, which will be an Error object. If
    the event type is `close`, the listener should accept one argument of a boolean
    type, which will be true when the client connection was closed due to errors.

2.  removeListener(eventType, listener)

    Removes the specified listener from the event specified in eventType. Note that
    the `end()` method automatically removes all listeners from the client object.


<a id="orgefe2bd9"></a>

# Platform Quirks & Warnings


<a id="orgea8c46e"></a>

## Server Capabilities

All SFTP servers and platforms are not equal. Some facilities provided by
`ssh2-sftp-client` either depend on capabilities of the remote server or the
underlying capabilities of the remote server platform. As an example,
consider `chmod()`. This command depends on a remote filesystem which
implements the 'nix' concept of users and groups. The *win32* platform does
not have the same concept of users and groups, so `chmod()` will not behave
in the same way.

One way to determine whether an issue you are encountering is due to
`ssh2-sftp-client` or due to the remote server or server platform is to use a
simple CLI sftp program, such as openSSH's sftp command. If you observe the
same behaviour using plain `sftp` on the command line, the issue is likely
due to server or remote platform limitations. Note that you should not use a
GUI sftp client, like `Filezilla` or `winSCP` as such GUI programs often
attempt to hide these server and platform incompatibilities and will take
additional steps to simulate missing functionality etc. You want to use a CLI
program which does as little as possible.

One way to determine whether an issue you are encountering is due to
`ssh2-sftp-client` or due to the remote server or server platform is to use a
simple CLI sftp program, such as openSSH's sftp command. If you observe the
same behaviour using plain `sftp` on the command line, the issue is likely
due to server or remote platform limitations. Note that you should not use a
GUI sftp client, like `Filezilla` or `winSCP` as such GUI programs often
attempt to hide these server and platform incompatibilities and will take
additional steps to simulate missing functionality etc.


<a id="orgef8ee0b"></a>

## Promises & Events

The reality of the current Node environment is that Promises and Events don't
play nicely together. Part of the problem is that events are asynchronous in
nature and can occur at any time. It is very difficult to ensure an event is
captured inside a Promise and handled appropriately. More information can be
found in the Node documentation for Events.

Node v12 has introduced some experimental features to make working with
Events and Promises a little easier. At this stage, we are not using these
features because they are experimental and because it would mean you cannot
use this module with Node v10. Use of these features will likely be examined
more closely once they become stable and non-experimental.

So, what does this mean for this module? The `ssh2-sftp-client` module works
hard to ensure things work as expected. In most cases, events are handled
appropriately. However, there are some edge cases where events may not be
handled and you may see an uncaught error exception. The most common place to
see this is when you keep an SFTP connection open, but don't use it for some
time. When the connection is open, but no methods are active (running), there
are no error handlers defined. Should an error event be emitted (for exmaple,
because the network connection has been lost), there is no handler and you
will get an uncaught error exception.

One way to handle this is to add your own error handler using the on()
method. Note however, you need to be careful how many times your error
handler is added. If you begin to see a warning about a possible memory leak,
it is an indication your error handler is being added multiple times (Node
will generate this warning if it finds more than 11 listeners attached to an
event emitter).

The other issue that can occur is that in some rare cases, the error message
you get will be potentially misleading. For example, SFTP servers running on
Windows appear to emit an *ECONNRESET* error in addition to the main error
(for example, for failed authentication). This can result in an error which
looks like a connection was reset by the remote host when in fact the real
error was due to bad authentication (bad password or bad username). This
situation can be made even worse by some platforms which deliberately hide
the real error for security reasons e.g. does not report an error indicating
a bad username because that information can be used to try and identify
legitimate usernames. While this module attempts to provide meaningful error
messages which can assist developers track down problems, it is a good idea
to consider these errors with a grain of salt and verify the error when
possible.


<a id="orgd10fdaf"></a>

## Windows Based Servers

It appears that when the sftp server is running on Windows, a *ECONNRESET*
error signal is raised when the end() method is called. Unfortunately, this
signal is raised after a considerable delay. This means we cannot remove the
error handler used in the end() promise as otherwise you will get an uncaught
exception error. Leaving the handler in place, even though we will ignore
this error, solves that issue, but unfortunately introduces a new problem.
Because we are not removing the listener, if you re-use the client object for
subsequent connections, an additional error handler will be added. If this
happens more than 11 times, you will eventually see the Node warning about a
possible memory leak. This is because node monitors the number of error
handlers and if it sees more than 11 added to an object, it assumes there is
a problem and generates the warning.

The best way to avoid this issue is to not re-use client objects. Always
generate a new sftp client object for each new connection.


<a id="org553c32d"></a>

## Don't Re-use SftpClient Objects

Due to an issue with *ECONNRESET* error signals when connecting to Windows
based SFTP servers, it is not possible to remove the error handler in the
end() method. This means that if you re-use the SftpClient object for
multiple connections e.g. calling connect(), then end(), then connect() etc,
you run the risk of multiple error handlers being added to the SftpClient
object. After 11 handlers have been added, Node will generate a possible
memory leak warning.

To avoid this problem, don't re-use SftpClient objects. Generate a new
SftpClient object for each connection. You can perform multiple actions with
a single connection e.g. upload multiple files, download multiple files etc,
but after you have called end(), you should not try to re-use the object with
a further connect() call. Create a new object instead.


<a id="org778c557"></a>

# FAQ


<a id="orgdd19a09"></a>

## Remote server drops connections with only an end event

Many SFTP servers have rate limiting protection which will drop connections once
a limit has been reached. In particular, openSSH has the setting `MaxStartups`,
which can be a tuple of the form `max:drop:full` where `max` is the maximum
allowed unauthenticated connections, `drop` is a percentage value which
specifies percentage of connections to be dropped once `max` connections has
been reached and `full` is the number of connections at which point all
subsequent connections will be dropped. e.g. `10:30:60` means allow up to 10
unauthenticated connections after which drop 30% of connection attempts until
reaching 60 unauthenticated connections, at which time, drop all attempts.

Clients first make an unauthenticated connection to the SFTP server to begin
negotiation of protocol settings (cipher, authentication method etc). If you are
creating multiple connections in a script, it is easy to exceed the limit,
resulting in some connections being dropped. As SSH2 only raises an 'end' event
for these dropped connections, no error is detected. The `ssh2-sftp-client` now
listens for `end` events during the connection process and if one is detected,
will reject the connection promise.

One way to avoid this type of issue is to add a delay between connection
attempts. It does not need to be a very long delay - just sufficient to permit
the previous connection to be authenticated. In fact, the default setting for
openSSH is `10:30:60`, so you really just need to have enough delay to ensure
that the 1st connection has completed authentication before the 11th connection
is attempted.


<a id="orgd0c7d17"></a>

## How can I pass writable stream as dst for get method?

If the dst argument passed to the get method is a writeable stream, the remote
file will be piped into that writeable. If the writeable you pass in is a
writeable stream created with `fs.createWriteStream()`, the data will be written
to the file specified in the constructor call to `createWriteStream()`.

The writeable stream can be any type of write stream. For example, the below code
will convert all the characters in the remote file to upper case before it is
saved to the local file system. This could just as easily be something like a
gunzip stream from `zlib`, enabling you to decompress remote zipped files as you
bring them across before saving to local file system.

    'use strict';
    
    // Example of using a writeable with get to retrieve a file.
    // This code will read the remote file, convert all characters to upper case
    // and then save it to a local file
    
    const Client = require('../src/index.js');
    const path = require('path');
    const fs = require('fs');
    const through = require('through2');
    
    const config = {
      host: 'arch-vbox',
      port: 22,
      username: 'tim',
      password: 'xxxx'
    };
    
    const sftp = new Client();
    const remoteDir = '/home/tim/testServer';
    
    function toupper() {
      return through(function(buf, enc, next) {
        next(null, buf.toString().toUpperCase());
      });
    }
    
    sftp
      .connect(config)
      .then(() => {
        return sftp.list(remoteDir);
      })
      .then(data => {
        // list of files in testServer
        console.dir(data);
        let remoteFile = path.join(remoteDir, 'test.txt');
        let upperWtr = toupper();
        let fileWtr = fs.createWriteStream(path.join(__dirname, 'loud-text.txt'));
        upperWtr.pipe(fileWtr);
        return sftp.get(remoteFile, upperWtr);
      })
      .then(() => {
        return sftp.end();
      })
      .catch(err => {
        console.error(err.message);
      });


<a id="org9fa578f"></a>

## How can I upload files without having to specify a password?

There are a couple of ways to do this. Essentially, you want to setup SSH keys
and use these for authentication to the remote server.

One solution, provided by @KalleVuorjoki is to use the SSH agent
process. **Note**: SSH<sub>AUTH</sub><sub>SOCK</sub> is normally created by your OS when you load the
ssh-agent as part of the login session.

    let sftp = new Client();
    sftp.connect({
      host: 'YOUR-HOST',
      port: 'YOUR-PORT',
      username: 'YOUR-USERNAME',
      agent: process.env.SSH_AUTH_SOCK
    }).then(() => {
      sftp.fastPut(/* ... */)
    }

Another alternative is to just pass in the SSH key directly as part of the
configuration.

    let sftp = new Client();
    sftp.connect({
      host: 'YOUR-HOST',
      port: 'YOUR-PORT',
      username: 'YOUR-USERNAME',
      privateKey: fs.readFileSync('/path/to/ssh/key')
    }).then(() => {
      sftp.fastPut(/* ... */)
    }


<a id="org0c91c49"></a>

## How can I connect through a Socks Proxy

This solution was provided by @jmorino.

    import { SocksClient } from 'socks';
    import SFTPClient from 'ssh2-sftp-client';
    
    const host = 'my-sftp-server.net';
    const port = 22; // default SSH/SFTP port on remote server
    
    // connect to SOCKS 5 proxy
    const { socket } = await SocksClient.createConnection({
      proxy: {
        host: 'my.proxy', // proxy hostname
        port: 1080, // proxy port
        type: 5, // for SOCKS v5
      },
      command: 'connect',
      destination: { host, port } // the remote SFTP server
    });
    
    const client = new SFTPClient();
    client.connect({
      host,
      sock: socket, // pass the socket to proxy here (see ssh2 doc)
      username: '.....',
      privateKey: '.....'
    })
    
    // client is connected


<a id="org2d7770e"></a>

## Timeout while waiting for handshake or handshake errors

Some users have encountered the error 'Timeout while waiting for handshake' or
'Handshake failed, no matching client->server ciphers. This is often due to the
client not having the correct configuration for the transport layer algorithms
used by ssh2. One of the connect options provided by the ssh2 module is
`algorithm`, which is an object that allows you to explicitly set the key
exchange, ciphers, hmac and compression algorithms as well as server
host key used to establish the initial secure connection. See the SSH2
documentation for details. Getting these parameters correct usually resolves the
issue.


<a id="org97e33a3"></a>

## How can I limit upload/download speed

If you want to limit the amount of bandwidth used during upload/download of
data, you can use a stream to limit throughput. The following example was
provided by *kennylbj*. Note that there is a caveat that we must set the
`autoClose` flag to false to avoid calling an extra `_read()` on a closed stream
that may cause \_get Permission Denied error in ssh2-streams.

    
    
    const Throttle = require('throttle');
    const progress = require('progress-stream');
    
    // limit download speed
    const throttleStream = new Throttle(config.throttle);
    
    // download progress stream
    const progressStream = progress({
      length: fileSize,
      time: 500,
    });
    progressStream.on('progress', (progress) => {
      console.log(progress.percentage.toFixed(2));
    });
    
    const outStream = createWriteStream(localPath);
    
    // pipe streams together
    throttleStream.pipe(progressStream).pipe(outStream);
    
    try {
      // set autoClose to false
      await client.get(remotePath, throttleStream, { autoClose: false });
    } catch (e) {
      console.log('sftp error', e);
    } finally {
      await client.end();
    }


<a id="orge1dc473"></a>

# Examples

I have started collecting example scripts in the example directory of the
repository. These are mainly scripts I have put together in order to investigate
issues or provide samples for users. They are not robust, lack adequate error
handling and may contain errors. However, I think they are still useful for
helping developers see how the module and API can be used.


<a id="orgb2ace97"></a>

# Troubleshooting

The `ssh2-sftp-client` module is essentially a wrapper around the `ssh2` and
`ssh2-streams` modules, providing a higher level `promise` based API. When you
run into issues, it is important to try and determine where the issue lies -
either in the ssh2-sftp-client module or the underlying `ssh2` and
`ssh2-streams` modules. One way to do this is to first identify a minimal
reproducible example which reproduces the issue. Once you have that, try to
replicate the functionality just using the `ssh2` and `ssh2-streams` modules. If
the issue still occurs, then you can be fairly confident it is something related
to those later 2 modules and therefore and issue which should be referred to the
maintainer of that module.

The `ssh2` and `ssh2-streams` modules are very solid, high quality modules with
a large user base. Most of the time, issues with those modules are due to client
misconfiguration. It is therefore very important when trying to diagnose an
issue to also check the documentation for both `ssh2` and `ssh2-streams`. While
these modules have good defaults, the flexibility of the ssh2 protocol means
that not all options are available by default. You may need to tweak the
connection options, ssh2 algorithms and ciphers etc for some remote servers. The
documentation for both the `ssh2` and `ssh2-streams` module is quite
comprehensive and there is lots of valuable information in the issue logs.

If you run into an issue which is not repeatable with just the `ssh2` and
`ssh2-streams` modules, then please log an issue against the `ssh2-sftp-client`
module and I will investigate. Please note the next section on logging issues.

Note also that in the repository there are two useful directories. The first is
the examples directory, which contain some examples of using `ssh2-sftp-client`
to perform common tasks. A few minutes reviewing these examples can provide that
additional bit of detail to help fix any problems you are encountering.

The second directory is the validation directory. I have some very simple
scripts in this directory which perform basic tasks using only the `ssh2`
modules (no `ssh2-sftp-client` module). These can be useful when
trying to determine if the issue is with the underlying `ssh2` module or
the `ssh2-sftp-client` wrapper module.


<a id="org99cbc3b"></a>

## Common Errors

There are some common errors people tend to make when using Promises or
Asyc/Await. These are by far the most common problem found in issues logged
against this module. Please check for some of these before logging your
issue.


<a id="org5c0086e"></a>

### Not returning the promise in a `then()` block

All methods in `ssh2-sftp-client` return a Promise. This means methods are
executed *asynchrnously*. When you call a method inside the `then()` block
of a promise chain, it is critical that you return the Promise that call
generates. Failing to do this will result in the `then()` block completing
and your code starting execution of the next `then()`, `catch()` or
`finally()` block before your promise has been fulfilled. For example, the
following will not do what you expect

    sftp.connect(config)
      .then(() => {
        sftp.fastGet('foo.txt', 'bar.txt');
      }).then(rslt => {
        console.log(rslt);
        sftp.end();
      }).catch(e => {
        console.error(e.message);
      });

In the above code, the `sftp.end()` method will almost certainly be called
before `sftp.fastGet()` has been fulfilled (unless the *foo.txt* file is
really small!). In fact, the whole promise chain will complete and exit even
before the `sftp.end()` call has been fulfilled. The correct code would be
something like

    sftp.connect(config)
      .then(() => {
        return sftp.fastGet('foo.txt', 'bar.txt');
      }).then(rslt => {
        console.log(rslt);
        return sftp.end();
      }).catch(e => {
        console.error(e.message);
      });

Note the `return` statements. These ensure that the Promise returned by the
client method is returned into the promise chain. It will be this promise
the next block in the chain will wait on to be fulfilled before the next
block is executed. Without the return statement, that block will return the
default promise for that block, which essentially says *this block has been
fulfilled*. What you really want is the promise which says *your sftp client
method call has been fulfilled*.

A common symptom of this type of error is for file uploads or download to
fail to complete or for data in those files to be truncated. What is
happening is that the connection is being ended before the transfer has
completed.


<a id="orgf77aa39"></a>

### Mixing Promise Chains and Async/Await

Another common error is to mix Promise chains and async/await calls. This is
rarely a great idea. While you can do this, it tends to create complicated
and difficult to maintain code. Select one approach and stick with it. Both
approaches are functionally equivalent, so there is no reason to mix up the
two paradigms. My personal preference would be to use async/await as I think
that is more *natural* for most developers. For example, the following is
more complex and difficult to follow than necessary (and has a bug!)

    sftp.connect(config)
      .then(() => {
        return sftp.cwd();
      }).then(async (d) => {
        console.log(`Remote directory is ${d}`);
        try {
          await sftp.fastGet(`${d}/foo.txt`, `./bar.txt`);
        }.catch(e => {
          console.error(e.message);
        });
      }).catch(e => {
        console.error(e.message);
      }).finally(() => {
        sftp.end();
      });

The main bug in the above code is the `then()` block is not returning the
Promise generated by the call to `sftp.fastGet()`. What it is actually
returning is a fulfilled promise which says the `then()` block has been run
(note that the await'ed promise is not being returned and is therefore
outside the main Promise chain). As a result, the `finally()` block will be
executed before the await promise has been fulfilled.

Using async/await inside the promise chain has created unnecessary
complexity and leads to incorrect assumptions regarding how the code will
execute. A quick glance at the code is likely to give the impression that
execution will wait for the `sftp.fastGet()` call to be fulfilled before
continuing. This is not the case. The code would be more clearly expressed
as either

    sftp.connect(config)
      .then(() => {
        return sftp.cwd();
      }).then(d => {
        console.log(`remote dir ${d}`);
        return sftp.fastGet(`${d}/foot.txt`, 'bar.txt');
      }).catch(e => {
        console.error(e.message);
      }).finally(() => {
        return sftp.end();
      });

**or, using async/await**

    async function doSftp() {
      try {
        let sftp = await sftp.connect(conf);
        let d = await sftp.cwd();
        console.log(`remote dir is ${d}`);
        await sftp.fastGet(`${d}/foo.txt`, 'bat.txt');
      } catch (e) {
        console.error(e.message);
      } finally () {
        await sftp.end();
      }
    }


<a id="org61a7a3f"></a>

### Try/catch and Error Handlers

Another common error is to try and use a try/catch block to catch event
signals, such as an error event. In general, you cannot use try/catch blocks
for asynchronous code and expect errors to be caught by the `catch` block.
Handling errors in asynchronous code is one of the key reasons we now have
the Promise and async/await frameworks.

The basic problem is that the try/catch block will have completed execution
before the asynchronous code has completed. If the asynchronous code has not
compleed, then there is a potential for it to raise an error. However, as
the try/catch block has already completed, there is no *catch* waiting to
catch the error. It will bubble up and probably result in your script
exiting with an uncaught exception error.

Error events are essentially asynchronous code. You don't know when such
events will fire. Therefore, you cannot use a try/catch block to catch such
event errors. Even creating an error handler which then throws an exception
won't help as the key problem is that your try/catch block has already
executed. There are a number of alternative ways to deal with this
situation. However, the key symptom is that you see occasional uncaught
error exceptions that cause your script to exit abnormally despite having
try/catch blocks in your script. What you need to do is look at your code
and find where errors are raised asynchronously and use an event handler or
some other mechanism to manage any errors raised.


<a id="orgad1a96a"></a>

### Server Differences

Not all SFTP servers are the same. Like most standards, the SFTP protocol
has some level of interpretation and allows different levels of compliance.
This means there can be differences in behaviour between different servers
and code which works with one server will not work the same with another.
For example, the value returned by *realpath* for non-existent objects can
differ significantly. Some servers will throw an error for a particular
operation while others will just return null, some servers support
concurrent operations (such as used by fastGet/fastPut) while others will
not and of course, the text of error messages can vary significantly. In
particular, we have noticed significant differences across different
platforms. It is therefore advisable to do comprehensive testing when the
SFTP server is moved to a new platform. This includes moving from to a cloud
based service even if the underlying platform remains the same. I have
noticed that some cloud platforms can generate unexpected events, possibly
related to additional functionality or features associated with the cloud
implementation. For example, it appears SFTP servers running under Azure
will generate an error event when the connection is closed even when the
client has requested the connection be terminated. The same SFTP server
running natively on Windows does not appear to exhibit such behaviour.


<a id="org487c6e9"></a>

### Avoid Concurrent Operations

Technically, SFTP should be able to perform multiple operations
concurrently. As node is single threaded, what we a really talking about is
running multiple execution contexts as a pool where node will switch
contexts when each context is blocked due to things like waiting on network
data etc. However, I have found this to be extremely unreliable and of very
little benefit from a performance perspective. My recommendation is to
therefore avoid executing multiple requests over the same connection in
parallel (for example, generating multiple `get()` promises and using
something like `Promise.all()` to resolve them.

If you are going to try and perform concurrent operations, you need to test
extensively and ensure you are using data which is large enough that context
switching does occur (i.e. the request is not completed in a single run).
Some SFTP servers will handle concurrent operations better than others.


<a id="orge6f232c"></a>

## Debugging Support

You can add a `debug` property to the config object passed in to `connect()` to
turn on debugging. This will generate quite a lot of output. The value of the
property should be a function which accepts a single string argument. For example;

    config.debug = msg => {
      console.error(msg);
    };

Enabling debugging can generate a lot of output. If you use console.error() as
the output (as in the example above), you can redirect the output to a file
using shell redirection e.g.

    node script.js 2> debug.log

If you just want to see debug messages from `ssh2-sftp-client` and exclude debug
messages from the underlying `ssh2` and `ssh2-streams` modules, you can filter
based on messages which start with 'CLIENT' e.g.

    {
      debug: (msg) => {
        if (msg.startsWith('CLIENT')) {
          console.error(msg);
        }
      }
    }


<a id="org9299df9"></a>

# Logging Issues

Please log an issue for all bugs, questions, feature and enhancement
requests. Please ensure you include the module version, node version and
platform.

I am happy to try and help diagnose and fix any issues you encounter while using
the `ssh2-sftp-client` module. However, I will only put in effort if you are
prepared to put in the effort to provide the information necessary to reproduce
the issue. Things which will help

-   Node version you are using
-   Version of ssh2-sftp-client you are using
-   Platform your client is running on (Linux, macOS, Windows)
-   Platform and software for the remote SFTP server when possible
-   Example of your code or a minimal script which reproduces the issue you are
    encountering. By far, the most common issue is incorrect use of the module
    API. Example code can usually result in such issues being resolved very
    quickly.

Perhaps the best assistance is a minimal reproducible example of the issue. Once
the issue can be readily reproduced, it can usually be fixed very quickly.


<a id="org740d3e3"></a>

# Pull Requests

Pull requests are always welcomed. However, please ensure your changes pass all
tests and if your adding a new feature, that tests for that feature are
included. Likewise, for new features or enhancements, please include any
relevant documentation updates.

This module will adopt a standard semantic versioning policy. Please indicate in
your pull request what level of change it represents i.e.

-   **Major:** Change to API or major change in functionality which will require an
    increase in major version number.
-   **Minor:** Minor change, enhancement or new feature which does not change
    existing API and will not break existing client code.
-   **Bug Fix:** No change to functionality or features. Simple fix of an existing
    bug.


<a id="org43384a1"></a>

# Contributors

This module was initially written by jyu213. On August 23rd, 2019, theophilusx
took over responsibility for maintaining this module. A number of other people
have contributed to this module, but until now, this was not tracked. My
intention is to credit anyone who contributes going forward.

Thanks to the following for their contributions -

-   **jyu213:** Original author
-   **theophilusx:** Current maintainer
-   **henrytk:** Documentation fix
-   **waldyrious:** Documentation fixes
-   **james-pellow:** Cleanup and fix for connect method logic
-   **jhorbulyk:** Contributed posixRename() functionality
-   **teenangst:** Contributed fix for error code 4 in stat() method
-   **kennylbj:** Contributed example of using a throttle stream to limit
    upload/download bandwidth.
-   **anton-erofeev:** Documentation fix

