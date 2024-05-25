# Plans to Scale

## S3 File Gateway

Right now we are cookin' on SSDs on EC2 Instances 👨‍🍳 this is fast, but we can only mount up to 15TB drives on each instance. Eventually we need to scale to 100s of TBs of storage. Looks like we can simply mount S3 as a filesystem, but there are concerns on performance with the network file system here. Will have to do some tests.

https://aws.amazon.com/blogs/storage/mounting-amazon-s3-to-an-amazon-ec2-instance-using-a-private-connection-to-s3-file-gateway/

We could also store the version files in S3 and keep the rocksdb(s) on the SSDs. The merkle trees and commits databases etc are much smaller but would be slow to query if we put on NFS. In theory. All this has to be tested.

## Block Level Deduplication

XetHub boasts block level deduplication as a means of compression of their data.

https://about.xethub.com/blog/benchmarking-xethub-vs-dvc-lfs-lakefs

This is great for compression, especially when you add one row to a CSV for example. What it is not great for is serving up large files, because you will have to reconstruct them before you serve them.

I think it would be nice to have configurable block level deduplication on the client and server. N commits back maybe? Kind of like "cold storage" that you can turn on and off, acknowledging that queries will be slower, but you save storage.

## Compress Version Files

We currently store all the version files in their raw form in the .oxen/versions/ directory. This is fine for small files, but for large files it is not efficient. We can compress these files using a tool like zstd.

Worth testing this with a large file to see how it affects serving.



