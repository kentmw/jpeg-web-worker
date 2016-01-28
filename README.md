# jpeg-web-worker
Generate a jpeg using a web worker

## Motivation
The browser can generate a jpeg for you from an HTML canvas:

```
var canvas = document.getElementById("myCanvas");
var img    = canvas.toDataURL("image/jpg");
```
Which you can write to an image tag using:
```
document.write('<img src="'+img+'"/>');
```
or create as a blob (which can be sent as a part of a form) using:
```
var blob = canvas.toBlob( callback , 'image/jpeg' );
```

But what if the `toDataURL ` or `toBlob` call takes too long. Maybe the image is large or you are performing this call many times on different canvases. It would be nice to have the heavy duty of converting the canvas data to a jpeg off of the main application process. Perhaps through a web worker. This library provides the web worker file.

## Usage

**If you're working in the browser**
Getting the web worker file out of npm modules to be used by browser can be tricky. You can either copy the index.js into your project directory directly and rename it. Or you can use [workerify](https://github.com/shama/workerify) to convert this module to a blob to be used with browserify.

**API**

The message to be posted
```
{
  image: {
    data: [An RGBA array. This can typically be created by a canvas's context.getImageData],
    height: [Integer],
    width: [Integer]
  },
  quality: [OPTIONAL, defaults to 50. Integer from 0 (low quality) - 100 (high quality)]
}
```

The message to be received
```
{
  data: {
    data: [An RGBA Uint8Array, this time in JPEG encoding. Can be used to generate a jpeg blob.],
    height: [Integer],
    width: [Integer]
  }
}
```

**Example**

```
var canvas = document.getElementById("myCanvas");
var context = canvas.getContext('2d');
var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
var worker = new Worker('jpeg-web-worker.js');
worker.postMessage({
  image: imageData,
  quality: 50
});
worker.onmessage = function(e) {
  // e.data is the imageData of the jpeg. {data: U8IntArray, height: int, width: int}
  // you can still convert the jpeg imageData into a blog like this:
  var blob = new Blob( [e.data.data], {type: 'image/jpeg'} );
}
```

## Credits

This library relies almost entirely on an adaptation of the encoder from the [jpeg-js]() library. This library was built to work in a node.js environment and not in the browser, let alone in a web worker environment.


## License

jpeg-js library uses a 3-clause BSD license as follows:

Copyright (c) 2014, Eugene Ware
All rights reserved.
  
Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:  

1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.  
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.  
3. Neither the name of Eugene Ware nor the names of its contributors
   may be used to endorse or promote products derived from this software
   without specific prior written permission.  
  
THIS SOFTWARE IS PROVIDED BY EUGENE WARE ''AS IS'' AND ANY
EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL EUGENE WARE BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

The original license information about the encoder including the original port to javascript by  Andreas Ritter, and the adobe license can be found [on the jpeg-js's readme](https://github.com/eugeneware/jpeg-js#encoding)