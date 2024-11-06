---
title: "event_time"
id: "event-time"
sidebar_label: "event_time"
resource_types: [models, seeds, source]
description: "dbt uses event_time to understand when an event occurred. When defined, event_time enables microbatch incremental models and more refined comparison of datasets during Advanced CI."
datatype: string
---

Available in dbt Cloud Versionless and dbt Core v1.9 and higher.

<Tabs>
<TabItem value="model" label="Models">

<File name='dbt_project.yml'>

```yml
models:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>


<File name='models/properties.yml'>

```yml
models:
  - name: model_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```
</File>

<File name="models/modelname.sql">

```sql
{{ config(
    event_time='my_time_field'
) }}
```

</File>

</TabItem>

<TabItem value="seeds" label="Seeds">

<File name='dbt_project.yml'>

```yml
seeds:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<File name='seeds/properties.yml'>

```yml
seeds:
  - name: seed_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```

</File>
</TabItem>

<TabItem value="snapshot" label="Snapshots">

<File name='dbt_project.yml'>

```yml
snapshots:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<VersionBlock firstVersion="1.9">
<File name='snapshots/properties.yml'>

```yml
snapshots:
  - name: snapshot_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

<File name="models/modlename.sql">

```sql

{{ config(
    event_time: 'my_time_field'
) }}
```

</File>


import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>
</VersionBlock>



</TabItem>

<TabItem value="sources" label="Sources">

<File name='dbt_project.yml'>

```yml
sources:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<File name='models/properties.yml'>

```yml
sources:
  - name: source_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```

</File>
</TabItem>
</Tabs>

## Definition

Set the `event_time` to the name of the field that represents the timestamp of the event, as opposed to a date-like data loading date.  You can configure `event_time` for a [model](/docs/build/models), [seed](/docs/build/seeds), or [source](/docs/build/sources) in your `dbt_project.yml` file, property YAML file, or config block. 

Here are some examples of good and bad `event_time` columns:
✅ Good:

- `account_created_at` &mdash; This represents the specific time when an account was created, making it a fixed event in time.
- `session_began_at` &mdash; This captures the exact timestamp when a user session started, which won’t change and directly ties to the event.

❌ Bad:

- `_fivetran_synced` &mdash; This isn't the time that the event happened, it's the time that the event was ingested.
- `last_updated_at` &mdash; This isn't a good use case as this will keep changing over time. 

`event_time` is required for [Incremental microbatch](/docs/build/incremental-microbatch) and [Advanced CI's compare changes](/docs/deploy/advanced-ci#speeding-up-comparisons) in CI/CD workflows, where it ensures the same time-slice of data is correctly compared between your CI and production environments.

When you configure `event_time`, it enables compare changes to:

- Compare data in CI versus production for overlapping times only, reducing false discrepancies.
- Handle scenarios where CI has "fresher" data than production, by using only the overlapping timeframe, allowing you to avoid incorrect row-count changes.
- Account for subset data builds in CI without flagging filtered-out rows as "deleted" when compared with production.

## Examples

<Tabs> 

<TabItem value="model" label="Models">

Here's an example in the `dbt_project.yml` file:

<File name='dbt_project.yml'>

```yml
models:
  my_project:
    user_sessions:
      +event_time: session_start_time
```
</File>

Example in a properties YAML file:

<File name='models/properties.yml'>

```yml
models:
  - name: user_sessions
    config:
      event_time: session_start_time
```

</File>

Example in sql model config block:

<File name="models/user_sessions.sql">

```sql
{{ config(
    event_time='session_start_time'
) }}
```

</File> 

This setup sets `session_start_time` as the `event_time` for the `user_sessions` model. This makes sure the compare changes process uses this timestamp for time-slice comparisons or incremental microbatching.
</TabItem> 

<TabItem value="seeds" label="Seeds">

Here's an example in the `dbt_project.yml` file:

<File name='dbt_project.yml'>

```yml
seeds:
  my_project:
    my_seed:
      +event_time: record_timestamp
```

</File>

Example in a seed properties YAML:

<File name='seeds/properties.yml'>

```yml
seeds:
  - name: my_seed
    config:
      event_time: record_timestamp
```
</File>

This setup sets `record_timestamp` as the `event_time` for `my_seed`. It ensures that the `record_timestamp` is used consistently in [Advanced CI's compare changes](/docs/deploy/advanced-ci#speeding-up-comparisons) or [incremental microbatching](/docs/build/incremental-microbatch).

</TabItem> 
<TabItem value="sources" label="Sources">

Here's an example of source properties YAML file:

<File name='models/properties.yml'>

```yml
sources:
  - name: source_name
    tables:
      - name: table_name
        config:
          event_time: event_timestamp
```
</File>

This setup sets `event_timestamp` as the `event_time` for the specified source table.

</TabItem> 
</Tabs>
