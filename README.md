<a href="http://blog.vjeux.com/2011/javascript/jdataview-read-binary-file.html">jDataView</a> - A unique way to read a binary file in the browser.
================================

jDataView provides a standard way to read binary files in all the browsers. It follows the [DataView Specification](http://www.khronos.org/registry/webgl/doc/spec/TypedArray-spec.html#6) and even extends it for a more practical use.

Explanation
=========

There are three ways to read a binary file from the browser.

* The first one is to download the file through XHR with [charset=x-user-defined](https://developer.mozilla.org/en/using_xmlhttprequest#Receiving_binary_data). You get the file as a **String**, and you have to rewrite all the decoding functions (getUint16, getFloat32, ...). All the browsers support this.

* Then browsers that implemented WebGL also added **ArrayBuffers**. It is a plain buffer that can be read with views called **TypedArrays** (Int32Array, Float64Array, ...). You can use them to decode the file but this is not very handy. It has big drawback, it can't read non-aligned data. It is supported by Firefox 4 and Chrome 7.

* A new revision of the specification added **DataViews**. It is a view around your buffer that can read arbitrary data types directly through functions: getUint32, getFloat64 ... Only Chrome 9 supports it.

jDataView provides the DataView API for all the browsers using the best available option between Strings, TypedArrays and DataViews.

API
===
See the specification for a detailed API. [http://www.khronos.org/registry/webgl/doc/spec/TypedArray-spec.html](http://www.khronos.org/registry/webgl/doc/spec/TypedArray-spec.html#6)

Constructor
-----------------
* new **jDataView**(buffer, offset, length, littleEndian=true)
    * buffer can be either a String or an ArrayBuffer
    * littleEndian is a default value for the view

Specification API
-------------------------
The wrapper satisfies all the specification getters.

* **getInt8**(byteOffset)
* **getUint8**(byteOffset)
* **getInt16**(byteOffset, littleEndian)
* **getUint16**(byteOffset, littleEndian)
* **getInt32**(byteOffset, littleEndian)
* **getUint32**(byteOffset, littleEndian)
* **getFloat32**(byteOffset, littleEndian)
* **getFloat64**(byteOffset, littleEndian)


Extended Specification
---------------------------------
Addition of a littleEndian argument to the constructor. It will be the default value of the getters if their littleEndian value is undefined.

* **jDataView**(buffer, offset, length, littleEndian=true)

The byteOffset parameter is now optional. If you omit it, it will read right after the latest read offset. You can interact with the internal pointer with those two functions.

* **seek**(byteOffset)
    * Moves the internal pointer to the position
* **tell**()
    * Returns the current position

Addition of getChar and getString utilities.

* **getChar**(byteOffset)
* **getString**(length, byteOffset)

Addition of createBuffer, a utility to easily create buffers with the latest available storage type (String or ArrayBuffer).

* **createBuffer**(byte1, byte2, ...)

Shortcomings
==========

* Only the Read API is being wrapped, jDataView does not provide any `set` method.
* The Float64 implementation on strings does not have full precision.
* I found that most files we want to read are in littleEndian due to x86 architecture. I changed the default behavior of getters to be littleEndian instead of bigEndian.

Example
======
First we need a file. Either you get it through XHR or use the createBuffer utility.

```javascript
var file = jDataView.createBuffer(
	0x10, 0x01, 0x00, 0x00, // Int32 - 272
	0x90, 0xcf, 0x1b, 0x47, // Float32 - 39887.5625
	0, 0, 0, 0, 0, 0, 0, 0, // 8 blank bytes
	0x4d, 0x44, 0x32, 0x30, // String - MD20
	0x61                    // Char - a
);
```

Now we use the DataView as defined in the specification, the only thing that changes is the j before jDataView.

```javascript
var view = new jDataView(file);
var version = view.getInt32(0); // 272
var float = view.getFloat32(4); // 39887.5625
```

The wrapper extends the specification to make the DataView easier to use.

```javascript
var view = new jDataView(file);
// A position counter is managed. Remove the argument to read right after the last read.
version = view.getInt32(); // 272
float = view.getFloat32(); // 39887.5625

// You can move around with tell() and seek()
view.seek(view.tell() + 8);

// Two helpers: getChar and getString will make your life easier
var tag = view.getString(4); // MD20
var char = view.getChar(); // a
```

You can use a <a href="http://blog.vjeux.com/2011/javascript/jquery-binary-ajax.html">patched version of jQuery</a> that supports ArrayBuffer for AJAX.

```javascript
$.get(
  'data.bin',
  function (data) {
    var view = new jDataView(data);
    var tag = view.getString(4); // 'MD20'
    var version = view.getUint32(); // 732
  },
  'binary'
);
```

Changelog
========
* **21 September 2011**: Added a missing ```littleEndian``` argument on getInt16.
* **28 April 2011**: Seeking to the end of file no longer throws an error.
* **26 April 2011**: Fixed a bug with extremely large unsigned 32bit being considered as signed. ([Solution](http://stackoverflow.com/questions/1240408/reading-bytes-from-a-javascript-string/2954435#2954435)). 
* **8 April 2011**: Added littleEndian argument on the constructor. Opera 11.50 does not fully implement DataView, improved check.

Demos
==== 

* A <a href="http://fooo.fr/~vjeux/github/jsDataView/demo/untar/untar.html">simple tar viewer</a>. It is a "Hello World" demo of how easy it is to use the library.

* A <a href="http://fooo.fr/~vjeux/github/jsWoWModelViewer/modelviewer.html">World of Warcraft Model Viewer</a>. It uses jDataView to read the binary file and then WebGL to display it.
<a href="http://fooo.fr/~vjeux/github/jsWoWModelViewer/modelviewer.html"><img src="http://fooo.fr/~vjeux/github/jsWoWModelViewer/images/modelviewer.png"></a>

* A <a href="http://www.visual-experiments.com/2011/04/05/photosynth-webgl-viewer/">PhotoSynth WebGL Viewer</a> by Visual Experiments. It uses jDataView to read the binary file and then WebGL to display it.
<a href="http://www.visual-experiments.com/2011/04/05/photosynth-webgl-viewer/"><img src="http://i.imgur.com/HRHXo.jpg"/></a>

Please tell me if you made something with jDataView :)

Licence: [Do What The Fuck You Want To Public License](http://sam.zoy.org/wtfpl/)
