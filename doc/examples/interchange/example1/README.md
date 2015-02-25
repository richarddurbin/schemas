This example consists of a folder with three significant files: "Sequence.fa",
"Reference.avro", and "ReferenceSet.avro". Together they define an example
`ReferenceSet` in GA4GH interchange format.

"Reference.avro" is generated from "Reference.json", and "ReferenceSet.avro" is
"generated from "ReferenceSet.json".

In the JSON files, note the use of tags on Avro unions. Null values for unions
may be simple JSON nulls. Union values which are user-defined record types are
tagged as the fully qualified type name ("org.ga4gh.models.Position"). Union
values which are strings are tagged as "string", and union values which are
arrays (of any type) are tagged as "array". (It does not appear to be documented
what Avro will do if you try to tag a union value as "array" if the union
unifies two or more array types.)

The JSON files are compressed into Avro format as follows:

```shell
# Start in the root of the repository (schemas)
cd schemas

# Get the Avro command-line tools
curl -o avro-tools.jar \
    http://www.us.apache.org/dist/avro/avro-1.7.7/java/avro-tools-1.7.7.jar

# Create .avsc files each describing an individual References API record type
# from the monolithic references.avdl file
mkdir -p target/schemas/
java -jar avro-tools.jar idl2schemata src/main/resources/avro/references.avdl \
    target/schemas/
    
# Make Reference.avro from Reference.json, according to Reference.avsc
java -jar avro-tools.jar fromjson \
    --schema-file target/schemas/Reference.avsc \
    doc/examples/interchange/example1/Reference.json \
    > doc/examples/interchange/example1/Reference.avro
    
# Make ReferenceSet.avro from ReferenceSet.json, according to ReferenceSet.avsc
java -jar avro-tools.jar fromjson \
    --schema-file target/schemas/ReferenceSet.avsc \
    doc/examples/interchange/example1/ReferenceSet.json \
    > doc/examples/interchange/example1/ReferenceSet.avro
```

To dump files back out in JSON:

```shell
java -jar avro-tools.jar tojson \
    doc/examples/interchange/example1/Reference.avro

java -jar avro-tools.jar tojson \
    doc/examples/interchange/example1/ReferenceSet.avro
```
    

