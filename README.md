	var file = cDataView.createBinaryStream(
		0x10, 0x01, 0x00, 0x00, // Int32 - 272
		0x90, 0xcf, 0x1b, 0x47, // Float32 - 39887.5625
		0, 0, 0, 0, 0, 0, 0, 0, // 8 blank bytes
		0x4d, 0x44, 0x32, 0x30, // String - MD20
		0x61                    // Char - a
	);

Now we use the DataView as defined in the specification, the only thing that changes is the c before cDataView.
    var view = new cDataView(file);
    var version = view.getInt32(0); // 272
    var float = view.getFloat32(4); // 39887.5625

The wrapper extends the specification to make the DataView easier to use.
    var view = new cDataView(file);
    // A position counter is managed. Remove the argument to read right after the last read.
    version = view.getInt32(); // 272
    float = view.getFloat32(); // 39887.5625

    // You can move around with tell() and seek()
    view.seek(view.tell() + 8);

    // Two helpers: getChar and getString will make your life easier
    var tag = view.getString(undefined, 4); // MD20
    var char = view.getChar(); // a