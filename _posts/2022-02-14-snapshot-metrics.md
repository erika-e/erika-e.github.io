---
title: 'Snapshot Your Metrics Models'
date: 2022-02-14
description: Don't let source data changes scare your stakeholders with changes to important KPIs. Add snapshots and tests to important metrics to be notified if something goes wrong with a key metric.
featured_image: '/images/posts/scary_data.jpg'
---

## Why Snapshot Your Metrics?

Metrics like your company's financials are important. Your stakeholders have
some of these numbers memorized. They trust you to make sure these numbers are
right. They'll notice right away if historical values change when they
shouldn't.

A classic post on the dbt Discourse explains how to use snapshots to ensure
your metrics aren't changing on you. [Build Snapshot-BasedTests to Detect Regressions in Your Historic Data](https://discourse.getdbt.com/t/build-snapshot-based-tests-to-detect-regressions-in-historic-data/1478)
was written in 2020 by [Evgeny Rubtsov](https://www.linkedin.com/in/evgeny-rubtsov-48884322/).
Back then it required custom tests and a special macro.

I implemented this recently for one of our financial models. I took advantage of
✨new✨ dbt features to avoid writing anything custom. Here's how it works.

1. 📸 Create a snapshot of your important metric model
2. 🧪 Add a test to ensure `dbt_valid_to` is always null, meaning your data hasn't changed
3. 😎 Rest easy knowing regressions will pop up as test failures
4. 🛠️ If failures do occur, update your test after root cause investigation

Fortunately (unfortunately?) the system worked and my test did detect a
regression. I was able to take quick action before our stakeholders noticed.
I also dug in on the root cause so we can be assured this won't happen again.

### 1. Make a Snapshot of Your Model

Let's say you have a financial model, like sales by month by customer. Your
model might look something like this:

| month_of | customer_id | sales |
| --- | --- | ---
| 2022-01-01 | 8675 | 2000 |
| 2022-01-01 | 5309 | 9000 |

In this case, we don't have a unique key we can use in our snapshot. We'll have
to make one. This is quite easy to do with [`dbt_utils.surrogate_key`](https://github.com/dbt-labs/dbt-utils#surrogate_key-source)

<!--- 
Ok, this super annoying and gross
Useful posts: https://www.bytedude.com/jekyll-syntax-highlighting-and-line-numbers/
https://rachelmad.github.io/entries/2016/11/06/code-in-jekyll

Still not happy with how the code blocks are looking, improve this in the future
at some point. Can't use SQL syntax highlighting with Jinja, it makes error
styling around curlies characters.
--->
{% highlight text linenos %}
-- assume your real model is above this point
-- add a surrogate key like so

select
  month_of
, customer_id
, sales
, {% raw %}{{ dbt_utils.surrogate_key(['month_of', 'customer_id']) }}{% endraw %} as surrogate_key
from final
{% endhighlight %}

Next up let's make our snapshot.

{% highlight text linenos %}
{% raw %}
{% snapshot sales_snapshot %}

{{
    config(
      target_database='analytics',
      target_schema='analytics_snapshots',
      unique_key='surrogate_key',

      strategy='check',
      check_cols=['month_of'
                  , 'customer_id'
                  , 'sales']
    )
}}

select
*
from {{ ref('sales') }}
where month_of <= current_date - interval '1 day'

{% endsnapshot %}
{% endraw %}
{% endhighlight %}

Let's break down what's going on in the snapshot. We're snapshotting our sales
model, with the `check` strategy. This means every time the snapshot runs, it
will compare the previous values of the columns specified in the `check_cols`
list against their current values.

If there's a difference, a new row will be inserted into the snapshot with the
new values. On the old row, the `dbt_valid_to` timestamp will be updated to the
time the snapshot ran. The newly inserted row will have a `dbt_valid_from` equal
to this time, and a `dbt_valid_to` that's null.

We can give ourselves a grace period with the `where` line in the SQL query.

### 2. Adding Tests

We'll see a change in the snapshot model if something changes. Let's add a test
to notify us if this happens. Back in 2020, a custom test was required. Now,
`dbt_utils` has you covered with `expression_is_true`.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Love that I never have to write custom tests in dbt. There&#39;s almost always a way to do it with dbt_utils instead!<br><br>Haven&#39;t had to use dbt-expectations yet but I&#39;ve heard lots of great things.</p>&mdash; Erika Pullum she/hers (@erikapullum) <a href="https://twitter.com/erikapullum/status/1489246319474524160?ref_src=twsrc%5Etfw">February 3, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

{% highlight yaml linenos %}
version: 2

models:
  - name: sales_snapshot
    tests:
      - dbt_utils.expression_is_true:
          expression: "dbt_valid_to is null"
{% endhighlight %}

If our metrics change, the snapshot will insert new rows. The test will fail.
We'll know a problem has occurred an we can investigate and determine what
caused it.

The test failing tells us that something changed in the source that shouldn't
have.  This is good, because we'll know about the problem. We need to fix the
test though, because a failure every day could cause us to miss something else
that's important.

### 3. Updating the Test After a Problem

Let's say one of our sales numbers changed, for `customer_id` `8675`. The
snapshot will now have two records for that customer:

| month_of | customer_id | sales | dbt_valid_from | dbt_valid_to |
| --- | --- | --- | --- | --- |
| 2022-01-01 | 8675 | 2000 | 2022-01-02 | 2022-02-14 |
| 2022-01-01 | 8675 | 3000 | 2022-02-14 | null |

Our test will be failing due to the first row in the table, and we'll get a
message something like this:

{% highlight cmd linenos %}
06:10:53  Completed with 1 error and 0 warnings:
06:10:53  
06:10:53  Failure in test dbt_utils_expression_is_true_sales_snapshot_dbt_valid_to_is_null (snapshots/snapshots.yml)
06:10:53    Got 1 result, configured to fail if >0
{% endhighlight %}

Since we've understood the failure mode already, we need to fix the failing
test. It has done its job by alerting us -- we don't want it failing every day!

One method for fixing the failing test is to remove, edit, or update the rows in
the snapshot. Since dbt `0.21.0` severity configurations mean we can get the
test back to passing without modifying the snapshot table.

[Severity configurations](https://docs.getdbt.com/reference/resource-configs/severity)
let us specify a severity of `error` or `warn` for a test. They also allow us to
specify the number of result rows the test must return to be considered failure.

This error message gave us exactly what we need to get the test passing. We can
update the severity configs so that the test is passing again.

{% highlight yaml linenos %}
version: 2

models:
  - name: sales_snapshot
    tests:
      - dbt_utils.expression_is_true:
          expression: "dbt_valid_to is null"
          config: #update when exceptions occur
            severity: error
            error_if: ">1"
            warn_if: ">1"
{% endhighlight %}

This requires a PR, which is probably a good thing! This gives you an
opportunity to document what caused the failure, what corrective actions you
took, and why the change to the test is needed.

## What Should You Snapshot Like This?

This strategy is effective, but it takes work to maintain it. What types of
models is it good for?

- Models that cannot be wrong, like financials or key KPIs
- Data that's aggregated by a date dimension, like a week or a month
- Metrics that shouldn't change after a certain amount of time

Have thoughts? Find me on Twitter and let me know!
