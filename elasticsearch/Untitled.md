Trying to create too many buckets. Must be less than or equal to: [65535] 
search.max_buckets/1.5 每个分片允许的最大buckets
search.max_buckets/1.5/keys  每个key允许的最大聚合窗口数量
每个分片300key，聚合下来1500key，说明分片分布不均匀，导致虽然发片查询不报错，聚合后就报错