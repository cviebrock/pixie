# Pixie

A simple, on-the-fly image resizing server written in NodeJS.

> **NOTE**: Not production-ready yet, and my JS isn't the best so _caveat emptor_.



## Installation

Get the source code (clone it, download a zip, whatever), then:

``` shell
cd pixie
cp pixie.config.json.example pixie.config.json
yarn
yarnpkg start
```

There are probably some libraries that your OS will need to build the
[sharp](http://sharp.dimens.io/) image resizing library that Pixie uses.
See their docs for more details on what prerequisites you might need 
and how to get them.

After starting Pixie, it will log information about any new files it 
generates, and any errors, to stdout.



## Configuration

Configuration is handled by the `.env` file.

#### PORT

The port that Pixie will listen on.  Defaults to 3000.

#### PATH

The root path for images that will be served.  Defaults to the `public` 
subdirectory of Pixie, but you probably want to set this to the (absolute) 
storage path of your other application that is using Pixie.

#### HASH_KEY

A random string that is used to generate image hashes, mitigating DOS attacks.  See 
[Hashing](#hashing) below for more details.  If left empty, then no hash 
protection is enforced.

#### HASH_LENGTH

The length of the generated hash strings.  Defaults to 8.



## Requesting Images

Any original file that already exists in the `ROOT` directory will be served 
immediately (via ExpressJS's `use()` middleware).

A request for a resized image takes the following format:

```
FILENAME~~WIDTHxHEIGHT[~HASH].EXT[.FORMAT]
```

So, for example if the original image is `test_image.jpg`, and you would like a
160x90 JPEG version of the image, the file would be called:

```
test_image~~160x90.jpg
```

If you wanted a .webp version of the same image, simply add `.webp` on to the 
file:

```
test_image~~160x90.jpg.webp
```

If you want to only specify one of the dimensions (width or height) and have 
Pixie resize the image while maintaining aspect ratio, then just set the other 
dimension as zero:

```
test_image~~320x0.jpg
```

Generated files are served and also stored in the `ROOT` directory, so that 
subsequent requests to that file can be served immediately without requiring 
resizing again.

If you have hashing enabled -- which you should! -- then the hash is added to
the requested file name: 

```
test_image~~160x90~60609adb.jpg
test_image~~160x90~60609adb.jpg.webp
```



## Hashing

You probably don't want Pixie to generate images of arbitrary sizes.  To prevent
this, a hashing algorithm is implemented to verify that the requested image is
valid.  The hash is the MD5 of the original file name, the new width and height 
of the resized image, and the secret `HASH_KEY` configuration value.  This MD5 
is then truncated to the rightmost number of characters as defined by 
`HASH_LENGTH` (simply to keep file names a reasonable length).

Some sample code (in PHP):

```php
define('HASH_KEY', 'SomeSecretString');
define('HASH_LENGTH', 8);

$file = 'test_image.jpg';
$newWidth = 160;
$newHeight = 90;

$hash = md5($file . $newWidth . $newHeight . HASH_KEY);
$hash = substr($hash, -1 * HASH_LENGTH); 
```

As long as you maintain consistency between the key and length in both Pixie's 
configuration and your application, then resized files will have deterministic 
names.



## Issues / Planned Features

Pixie relies on Node and ExpressJS to deal with race conditions (e.g. thousands 
of requests for the same image that needs to be generated).  This could be an 
issue on high-traffic sites, but Node's single thread system should take care 
of things.  More work is planned in this area.

Image sizes are restricted to 9999x9999 pixels.  This could be made into 
configurable options, but those seem like reasonable limits for most cases.

There is no consideration for restricting images from being up-sized (and thus 
possibly losing some detail).  Again, this could be configurable in future 
versions.


## Copyright and License

[Pixie](https://github.com/cviebrock/pixie) was written by 
[Colin Viebrock](http://viebrock.ca) and is released under the 
[MIT License](LICENSE.md).

Copyright (c) 2017 Colin Viebrock
