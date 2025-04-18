内存列存数据格式：Apache Arrow，它有一个非常大的愿景：提供内存数据分析 (in-memory analytics) 的开发平台，让数据在异构大数据系统间移动、处理地更快

参考资料：

- <https://tech.ipalfish.com/blog/2020/12/08/apache_arrow_summary/>

Feather 数据格式

- <https://arrow.apache.org/docs/python/feather.html>



数据存储格式选择：Parquet、HDF还是Feather？

- <https://www.sitstars.com/archives/113/>

可以看到，三种不同大小的数据集中，feather 读写速度都一枝独秀，大小占用中规中矩。Parquet 在小数据集上表现较差，但随着数据量的增加，其读写速度相比与其他格式就有了很大优势，在大数据集上，Parquet 的读取速度甚至能和 feather 一较高下，可以想象数据量突破 2G 后，Parquet 的读取速度可能就是最快的了。但是在写方面，Parquet 一直没有表现出超越 feather 的势头。Parquet 另外一个优势就是压缩率高，占用空间相比于其他格式一直是比较小的。hdf 比较尴尬，在小数据集上打不过 feather，在大数据集上的读取速度甚至比不上 csv。至于 csv 就不用说了，各方面拉跨。