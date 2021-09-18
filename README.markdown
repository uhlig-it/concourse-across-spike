# Spike on Concourse' `across` step

This is a spike on dynamically creating Concourse pipeline instances from an external dataset. It started as a [discussion on GitHub](https://github.com/concourse/concourse/discussions/7175), and was finally possible with [Concourse v7.4.0](https://github.com/concourse/concourse/releases/tag/v7.4.0).

# Idea

We have a list of regions where we compile a sales report for in a task. The list of regions is somewhat dynamic, and we decided to create a separate pipeline instance for each region.

We will create a "meta" pipeline `regional-sales-instances` that updates all instances of the `regional-sales` pipeline whenever the list of regions changes.

TODO How to we deal with regions going away? Perhaps we'd delete or archive the pipeline.

# Setting the meta pipeline

```command
$ fly \
    --target "$CONCOURSE_TARGET" \
  set-pipeline \
    --pipeline regional-sales-instances \
    --config pipelines/regional-sales-instances.yml \
    --load-vars-from="private-config.yml"
```

# Sales Data

The sales data for this spike was faked with the following command:

```command
$ for region in $(jq < meta/regions.json --raw-output '.[]'); do
  for month in $(seq 9); do
    mkdir -p sales-data/"$region"
    echo "sku,units" > "sales-data/$region/2021-0$month.csv"
    repeat 5 echo "$RANDOM,$RANDOM" >> "sales-data/$region/2021-0$month.csv"
  done
done
```

In a real-world pipeline, this could be consumed from a resource (e.g. an S3 bucket).
