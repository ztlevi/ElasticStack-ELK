# Kibana https://logz.io/blog/kibana-tutorial/

See `./logstash/pipeline/logstash.conf` for more details about using csv as source input.

Kibana/Index Pattern -> Create Index pattern -> Input demo-csv\* -> select @timestamp

## Logical Statements

You can use logical statements in searches in these ways:

- USA AND Firefox
- USA OR Firefox
- (USA AND Firefox) OR Windows
- -USA
- !USA
- +USA
- NOT USA

### Tips and gotchas

1.  You need to make sure that you use the proper format such as capital letters to define logical
    terms like AND or OR
2.  You can use parentheses to define complex, logical statements
3.  You can use -,! and NOT to define negative terms
