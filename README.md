# gsfpy
Python wrapper for the C implementation of the Generic Sensor Format library.

## Install as pip package into external project
    SSH: pip install git+ssh://git@github.com/UKHO/gsfpy.git@master
    HTTPS: pip install git+https://github.com/UKHO/gsfpy.git@master

## Development environment
Set up the gsfpy project in a local development environment as follows:

    git clone git@github.com:UKHO/gsfpy.git
    virtualenv gsfpy/ (--always-copy)
    cd gsfpy/
    source bin/activate
    python3 -m pip install -e .

## Run tests
Set up the development environment as above, then:

    python3 -m pip install pytest
    python3 -m pytest -s --verbose ./tests/

## Examples of usage
### Open/close a GSF file
    import gsfpy
    from ctypes import *
    from gsfpy.enums import FileMode

    mode = FileMode.GSF_READONLY
    c_int_ptr = POINTER(c_int)
    p_gsf_fileref = c_int_ptr(c_int(0))

    # Note - pass file paths as byte strings
    retValOpen = gsfpy.gsfOpen(b'path/to/file.gsf', mode, p_gsf_fileref)
    retValClose = gsfpy.gsfClose(p_gsf_fileref[0])

### Read from a GSF file
    import gsfpy
    from ctypes import *
    from gsfpy.enums import FileMode, RecordType

    mode = FileMode.GSF_READONLY
    c_int_ptr = POINTER(c_int)
    p_gsf_fileref = c_int_ptr(c_int(0))
    
    commentID = c_gsfDataID()
    commentID.recordID = c_uint(RecordType.GSF_RECORD_COMMENT.value)

    c_gsfDataID_ptr = POINTER(c_gsfDataID)
    p_dataID = c_gsfDataID_ptr(commentID)

    c_gsfRecords_ptr = POINTER(c_gsfRecords)
    p_rec = c_gsfRecords_ptr(c_gsfRecords())

    c_ubyte_ptr = POINTER(c_ubyte)
    p_stream = c_ubyte_ptr()

    retValOpen = gsfpy.gsfOpen(b'path/to/file.gsf', mode, p_gsf_fileref)
    bytesRead = gsfpy.gsfRead(p_gsf_fileref[0], c_int(RecordType.GSF_RECORD_COMMENT.value), p_dataID, p_rec, p_stream, 0)

    # Retrieve the value of a field from the comment that was read.
    # Note the use of ctypes.string_at() to get POINTER(c_char) contents.
    print(string_at(p_rec.contents.comment.comment))

    retValClose = gsfpy.gsfClose(p_gsf_fileref[0])

### Write to a GSF file
    import gsfpy
    from ctypes import *
    from gsfpy.enums import FileMode, RecordType

    createMode = FileMode.GSF_CREATE
    c_int_ptr = POINTER(c_int)
    p_gsf_fileref = c_int_ptr(c_int(0))

    commentID = c_gsfDataID()
    commentID.recordID = c_uint(RecordType.GSF_RECORD_COMMENT.value)

    c_gsfDataID_ptr = POINTER(c_gsfDataID)
    p_dataID = c_gsfDataID_ptr(commentID)

    # Initialize the contents of the record that will be written.
    # Note use of ctypes.create_string_buffer() to set POINTER(c_char) contents.
    c_gsfRecords_ptr = POINTER(c_gsfRecords)
    p_rec = c_gsfRecords_ptr(c_gsfRecords())
    p_rec.contents.comment.comment_time.tvsec = c_int(1000)
    p_rec.contents.comment.comment_length = c_int(17)
    p_rec.contents.comment.comment = create_string_buffer(b'My first comment')

    retValOpenCreate = gsfpy.gsfOpen(b'path/to/new-file.gsf', createMode, p_gsf_fileref)
    bytesWritten = gsfpy.gsfWrite(p_gsf_fileref[0], p_dataID, p_rec)
    retValClose = gsfpy.gsfClose(p_gsf_fileref[0])

### Troubleshoot
    # The gsfIntError() and gsfStringError() functions are useful for
    # diagnostics. They return an error code and corresponding error
    # message, respectively.
    retValIntError = gsfpy.gsfIntError()
    retValStringError = gsfpy.gsfStringError()
    print(retValStringError)

## Generic Sensor Format Documentation
Generic Sensor Format specification: see https://github.com/schwehr/generic-sensor-format/blob/master/doc/GSF_lib_03-06.pdf

Generic Sensor Format C library v3.06 specification: see https://github.com/schwehr/generic-sensor-format/blob/master/doc/GSF_spec_03-06.pdf

## Acknowledgements
C implementation of the GSF library provided by [Leidos](https://www.leidos.com/products/ocean-marine) under the LGPL license v2.1.

libgsf3_06.so was built from the Leidos C code using Make scripts based on those from [schwehr/generic-sensor-format](https://github.com/schwehr/generic-sensor-format/)