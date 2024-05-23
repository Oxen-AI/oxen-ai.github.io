# Improvements

Here lies a list of improvements that could be made to Oxen. This is a living document as I document the codebase it's self and go.... Oh yeah! That worked but could be better.

## Compress the data in the .oxen/versions/files/ directories

Right now we store the data in the .oxen/versions/files/ directories in a very uncompressed format. This is not efficient for text files that could be compressed. We could compress the data using zlib like git does.

### Benefits
* Reduces the size of the .oxen/versions/files/ directories

### Drawbacks
* Time to compress and decompress the data (especially on large files that we want to be quickly accessible on the oxen server)


