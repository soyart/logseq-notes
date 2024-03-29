## [Format](https://parquet.apache.org/docs/file-format/configurations/)
	- Configuration
		- [[Parquet row group]] size
			- Large row groups (512M - 1G) recommended
			- Because we may need to read entire row groups, HDFS block size should be larger than row group size
		- [[Parquet page]] size
			- Smaller page size (8K) recommended (more fine-grained control)
			- Larger page size = less page header overhead
	- Metadata
		- File metadata
		- Column (chunk) metadata
		- Thrift metadata
			- Serialized with `TCompactProtocol`
	- [[Parquet page]]
		- Represented by 3 parts, encoded back-to-back (page size in Configuration refer to these 3 combined)
			- [Optional] definition-level data
			- [Optional] repetition-level data
			- [Required] encoded values
		- [Encodings](https://parquet.apache.org/docs/file-format/data-pages/encodings/)
			- Plain encoding - simple encoding
				- [PLAIN = 0]
			- Dictionary encoding
				- Dictionary is built with keys being column values, values are then encoded/compressed with RLE/Bit-Packing Hybrid
				- If dictionary got too large, plain encoding will be used
			- RLE/Bit-Packing Hybrid