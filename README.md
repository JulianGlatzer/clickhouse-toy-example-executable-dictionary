# clickhouse-toy-example-executable-dictionary
This toy example illustrates an from our side unexpected behaviour in clickhouse. We create a dictionary (cache-layout) which uses an executable as source. If a query uses a key in this dictionary several times, the request to the executable includes this key several times. We would have expected that for performance reasons clickhouse requests the value for each key from the executable only once.

Steps to reproduce
1. run docker compose with docker compose -f docker-compose.yml
2. In clickhouse client (clickhouse client --user testuser --password test --host localhost), run select number, dictGet('lambdadict','value',(number%2)) from system.numbers limit 10. For this query, I would have expected, that only the key 0 and 1 are requested from the executavle
3. The executable is tee /tmp/input.txt | sort | uniq | awk '{printf "%s\t%s\n", $1, $1*2}' | tee /tmp/output.txt (ignore the sort | uniq as it's just a workaround I will explain later) Go to /tmp/input.txt and you can see the following content 
```
0
1
0
1
0
1
0
1
0
1
```
instead of
```
0
1
```
Assuming that the awk command represents an expensive operation, this is not ideal. We have created a workaround by using sort | uniq and apparently clickhouse does not expect the same number of lines for the output as for the input of the executable. It still feels, it would be better to ensure that the unique set of keys is requested by clickhouse itself.