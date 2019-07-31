# How to rename field names in jsonb in array of objects

Roadmaps:

- [ ] Improve example
- [ ] Unittest

```
CREATE OR REPLACE FUNCTION jsonb_rename_fields_in_array_of_objects(data jsonb, path varchar, renaming jsonb) RETURNS JSONB AS $$
DECLARE
    i INTEGER;
    renaming_row RECORD;
BEGIN
    FOR i IN 0..jsonb_array_length(data->path)-1 LOOP
        FOR renaming_row IN select * from jsonb_each(renaming) LOOP
            IF data->path->i->renaming_row.key IS NOT NULL THEN
                data = jsonb_set(
                    data,
                    ('{'|| path || ', ' || i || ', ' || renaming_row.value || '}')::text[],
                    (data->path->i->renaming_row.key)::jsonb,
                    true
                )::jsonb;
            END IF;

            data = data #- ('{'|| path || ', ' || i || ', ' || renaming_row.key || '}')::text[];
        END LOOP;
    END LOOP;

    RETURN data;
END
$$ LANGUAGE plpgsql;
```

How to use:

```
select
    jsonb_pretty(jsonb_rename_fields_in_array_of_objects(
        "jsonb_field",
        'my_path',
        '{
            "field_a" : "field_1",
            "field_b" : "field_2"
        }')
    )
from
    my_table where id='a435bfcb-8500-4c8f-a4a2-b2b5c4c467b5';
```
