# Hashing

One of the optimizations within Oxen is to use the [xxHash algorithm](https://github.com/Cyan4973/xxHash) to hash the file contents. This is a very fast hashing algorithm that is designed to be very memory efficient. It is also very fast to compute.

Compared to SHA or MD5 hashes which can hash data at < 1GB/s, xxHash can hash data at 30GB/s. This is a significant improvement for large files, and speeds up the process of adding and committing files to Oxen.


