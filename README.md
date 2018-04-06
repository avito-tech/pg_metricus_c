# PG Metricus

### Info

pg_metricus is an extension written in C for sending metrics in the socket (Brubeck aggregator, Graphite, etc.) from pl/pgsql code.

If a sending is executed inside a transaction, the metrics will delivered even if the transaction is aborted.

### Installation

In the directory where you downloaded pg_metricus, run

```
make install
```

You need set control variables in postgresql.conf. 
These can be added/changed at anytime with a simple reload. See the documentation for more details.:

```
pg_metricus.host = '10.9.5.164'
pg_metricus.port = 8124
```

Log into PostgreSQL and run the following commands. 
Schema is optional (but recommended) and can be whatever you wish, but it cannot be changed after installation.

```plpgsql
create schema metricus;
create extension pg_metricus schema metricus;
```

### Format

For Brubeck aggregator:
```plpgsql
select metricus.send_metric(format(E'%s.%s:%s|%s\n', 
    metric_path, 
    metric_name, 
    metric_value, 
    metric_type
));
```

For Graphite:
```plpgsql
select metricus.send_metric(format(E'%s.%s %s %s \n', 
    metric_path, 
    metric_name, 
    metric_value, 
    extract(epoch from now())::integer
));
```

### Example

```plpgsql
do language plpgsql $$
declare
	x1 timestamp;
	x2 timestamp;
	v_val_hstore text;
begin

	x1 = clock_timestamp();

	v_val_hstore = get_val_hstore();

	x2 = clock_timestamp();

	perform metricus.send_metric(format(E'%s.%s:%s|%s\n', 
        'db.sql.metric', 
        'get_val_hstore_duration', 
        extract(millisecond from (x2 - x1))::bigint::text, 
        'ms'
    ));

end
$$;
```

### Author

Nikolay Vorobev (nvorobev@avito.ru)

### License

MIT
