# FITSIO.jl

A [Julia](http://julialang.org) package for reading and writing
Flexible Image Transport System (FITS) files, based on the
[cfitsio](http://heasarc.gsfc.nasa.gov/fitsio/) library.

The interface is inspired by Erin Sheldon's
[fitsio](https://github.com/esheldon/fitsio) Python package.

## Installation

FITSIO is available for Julia 0.5 and later versions, and can be
installed with [Julia](https://github.com/julialang/julia.jl)'s
built-in package manager. In a Julia session run the command

```julia
julia> Pkg.update()
julia> Pkg.add("FITSIO")
```

On Linux or OS X, if it isn't already installed on your system,
the cfitsio library is automatically downloaded and compiled
(in your Julia packages directory). On Windows, a compiled dll will be
downloaded.

## Usage

To open an existing file for reading:
```julia
julia> using FITSIO

julia> f = FITS("file.fits")
File: file.fits
Mode: "w" (read-write)
HDUs: Num  Name  Type   
      1          Image  
      2          Table  
```
(At the REPL, information about the file contents is shown.)


A FITS file consists of one or more header-data units (HDUs),
concatenated one after the other. The `FITS` object therefore is
represented as a collection of these HDUs.

Get information about the first HDU:
```julia
julia> f[1]
File: file.fits
HDU: 1
Type: Image
Datatype: Float64
Datasize: (800, 800)
```

Iterate over HDUs in the file:
```julia
julia> for hdu in f; println(typeof(hdu)); end
FITSIO.ImageHDU
FITSIO.TableHDU
```


Each HDU can contain image data, or table data (either binary or
ASCII-formatted). For image extensions, get the size of the image
without reading it:
```julia
julia> ndims(f[1])
    2

julia> size(f[1])
(800,800)

julia> size(f[1], 2)
800
```


Read an image from disk:
```julia
julia> data = read(f[1]);  # read an image from disk

julia> data = read(f[1], :, 790:end);  # read just a subset of image
```


Show info about a binary table:
```julia
julia> f[2]
File: file.fits
HDU: 2
Type: Table
Rows: 20
Columns: Name  Size  Type    TFORM  
         col2        String  5A     
         col1        Int64   1K     
```


Read a column from the table:
```julia
 julia> data = read(f[2], "col1")
```


Read the entire header into memory and get values from it:
```julia
julia> header = read_header(f[1]);  # read the entire header from disk

julia> length(header)  # total number of records in header
17

julia> haskey(header, "NAXIS1")  # check if a key exists
true

julia> header["NAXIS1"]  # get value by keyword
800

julia> header[4]  # get value by position
800

julia> get_comment(header, "NAXIS")  # get comment for a given keyword
"length of data axis 1"
```


Read just a single header record without reading the entire header:
```julia
julia> read_key(f[1], 4)  # by position
("NAXIS1",800,"length of data axis 1")

julia> read_key(f[1], "NAXIS1")  # read by keyword
(800,"length of data axis 1")
```


Manipulate a header in memory:
```julia
julia> header["NEWKEY"] = 10  # change or add a keyword

julia> set_comment!(header, "NEWKEY", "this is a comment")
```


Close the file:
```julia
julia> close(f)
```
(`FITS` objects are also closed automatically when garbage collected.)


Open a new file for writing:
```julia
julia> f = FITS("newfile.fits", "w");
```
The second argument can be `"r"` (read-only; default), `"r+"`
(read-write) or `"w"` (write). In "write" mode, any existing file of
the same name is overwritten.


Write an image to the file:
```julia
julia> data = reshape([1:100], 5, 20);

julia> write(f, data)  # Write a new image extension with the data
```
To write some header keywords in the new extension, pass a
`FITSHeader` instance as a keyword: `write(f, data; header=header)`


Write a table to the file:
```julia
julia> data = Dict("col1"=>[1., 2., 3.], "col2"=>[1, 2, 3]);

julia> write(f, data)  # write a new binary table to a new extension
```
