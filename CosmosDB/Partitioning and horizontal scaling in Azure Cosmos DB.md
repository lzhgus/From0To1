## Logical partition 
---
A logical partition consists of a set of items that have the same partition key. 
Each logical partition can store up to 20GB of data. Good partition key choices have a wide range of possible values. For example, in a container where all items contain a `foodGroup` property, the data within the `Beef Products` logical partition can grow up to 20 GB
You can use Azure Monitor Alerts to [monitor if a logical partition's size is approaching 20 GB](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-alert-on-logical-partition-key-storage-size).


## Physical partitions 
---
A container is scaled by distributing data and throughpht across physical partitions. 

The number of physical partitions in your container depends on the following:
-   The amount of throughput provisioned (each individual physical partition can provide a throughput of up to 10,000 request units per second). The 10,000 RU/s limit for physical partitions implies that logical partitions also have a 10,000 RU/s limit, as each logical partition is only mapped to one physical partition.
    
-   The total data storage (each individual physical partition can store up to 50GB data).

## Managing logical partitions 
---
Azure Cosmos DB hashes the partition key value of an item. The hashed result determines the physical partition. Then, Azure Cosmos DB allocates the key space of partition key hashes evenly across the physical partitions. 

## Replica sets
---
Each physical partition consists of a set of replicas, also referred to as a [_replica set_](https://docs.microsoft.com/en-us/azure/cosmos-db/global-dist-under-the-hood).

Typically, smaller containers only require a single physical partition, but they will still have at least 4 replicas.

## Choose a partition key 
---
A partition key has two components: `Partition key path` and the `Partition key value`. 

For `all` containers, your partition key should: 

- Be a property that has a value which does not change. If a property is your partition key, you can't update that property's value. 
- Should only contain `String` values - or numbers should ideally be converted into a `String`, if there is any chance that they are outside the boundaries of double precision numbers according to [IEEE 754 binary64](https://www.rfc-editor.org/rfc/rfc8259#ref-IEEE754). The [Json specification](https://www.rfc-editor.org/rfc/rfc8259#section-6) calls out the reasons why using numbers outside of this boundary in general is a bad practice due to likely interoperability problems. These concerns are especially relevant for the partition key column, because it is immutable and requires data migration to change it later.
- Have a high cardinality. In other words, the property should have a wide range of possible values.
- Spread request unit (RU) consumption and data storage evenly across all logical partitions. This ensures even RU consumption and storage distribution across your physical partitions.


## Use item ID as the partition key

If your container has a property that has a wide range of possible values, it is likely a great partition key choice. One possible example of such a property is the _item ID_. For small read-heavy containers or write-heavy containers of any size, the _item ID_ is naturally a great choice for the partition key.

The system property _item ID_ exists in every item in your container. You may have other properties that represent a logical ID of your item. In many cases, these are also great partition key choices for the same reasons as the _item ID_.

The _item ID_ is a great partition key choice for the following reasons:

-   There are a wide range of possible values (one unique _item ID_ per item).
-   Because there is a unique _item ID_ per item, the _item ID_ does a great job at evenly balancing RU consumption and data storage.
-   You can easily do efficient point reads since you'll always know an item's partition key if you know its _item ID_.

Some things to consider when selecting the _item ID_ as the partition key include:

-   If the _item ID_ is the partition key, it will become a unique identifier throughout your entire container. You won't be able to have items that have a duplicate _item ID_.
-   If you have a read-heavy container that has a lot of [physical partitions](https://docs.microsoft.com/en-us/azure/cosmos-db/partitioning-overview#physical-partitions), queries will be more efficient if they have an equality filter with the _item ID_.
-   You can't run stored procedures or triggers across multiple logical partitions.