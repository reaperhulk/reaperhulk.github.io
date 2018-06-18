---
layout: post
title: Data Driven Decisions Using PyPI Download Statistics
---

What platform do my users run on? How many users do I have? Is it okay to drop support for *something*?

Open source library developers have an analytics problem. How do you answer questions like this when people are embedding your software as a component of their own product?

For example, you, the developer of the library `frinkiac-iac` would like to drop support for Python 2.6 because it is old, unsupported by upstream, and restricting your use of newer Python features. Is it okay to do this? Since you don't have any data you may attempt to compensate for this lack of insight in a variety of ways:

* Reach out through your community (mailing lists, bug trackers, IRC channels) to tentatively suggest the idea.

* Do a release with a deprecation warning with a timeline for removal that will potentially be massively pushed back as users complain.[^3]

* Just remove it and see if anybody complains.

* Wait for a major package to do it and follow their lead.


Each of these choices can work, but frequently result in major breakage for users or an unacceptably long timeline for removal resulting in significant maintenance burden[^4]. You need data, but it's so hard to collect it. [Can't someone else do it?](https://frinkiac.com/caption/S09E22/750715)[^1]

## BigQuery

[Quietly announced](https://mail.python.org/pipermail/distutils-sig/2016-May/028986.html) back in May 2016, PyPI download statistics are the most powerful tool a Python developer can use to inform their decision making. To unleash its [awesome power](https://frinkiac.com/caption/S09E23/465781) start by loading the [dataset](https://bigquery.cloud.google.com/dataset/the-psf:pypi). **Preemptive apology:** If you are a new Google Cloud user you may be confronted with an unpleasant set of modals asking you to agree to various things and create a new project. What you actually need to do is get to billing and start a free trial[^5]. Once you've got the BigQuery console available you should be able to click that link again and access the page. Now you can click Compose Query!

BigQuery uses a SQL-like language. A [query reference](https://cloud.google.com/bigquery/docs/reference/legacy-sql) is available to help you write your own queries. Additionally, you can see the set of available fields you can query against by clicking `downloads` on the left pane and looking at the schema. One note: `file.project` is the [normalized name](https://www.python.org/dev/peps/pep-0503/#normalized-names) of the project.

With this robust data set we can make substantially more informed decisions, but here are some examples to help you out. All the examples here use what Google references as "legacy SQL". Despite the name this is the default for queries, but feel free to use the [standard SQL](https://cloud.google.com/bigquery/docs/reference/standard-sql/) if you'd like!

### Query: Python Versions

In the original scenario we were curious if we could drop support for Python 2.6. What does that query look like? For the following examples we'll be using `cryptography` since that's my primary project and the one I run queries on most commonly.

```sql
SELECT
  REGEXP_EXTRACT(details.python, r"^([^\.]+\.[^\.]+)") as python_version,
  COUNT(*) as download_count,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
WHERE
  file.project = 'cryptography'
GROUP BY
  python_version,
ORDER BY
  download_count DESC
LIMIT 100
```

And our results (as of 2016-12-09):

<table style="width:100%">
  <tr><th>Row</th><th>python_version</th><th>download_count</th></tr>
  <tr><td>1</td><td>2.7</td><td>2208835</td></tr>
  <tr><td>2</td><td>3.5</td><td>202515</td></tr>
  <tr><td>3</td><td>2.6</td><td>109134</td></tr>
  <tr><td>4</td><td>null</td><td>91076</td></tr>
  <tr><td>5</td><td>3.4</td><td>87837</td></tr>
  <tr><td>6</td><td>3.3</td><td>5623</td></tr>
  <tr><td>7</td><td>3.6</td><td>1354</td></tr>
  <tr><td>8</td><td>3.7</td><td>643</td></tr>
  <tr><td>9</td><td>1.17</td><td>341</td></tr>
  <tr><td>10</td><td>3.2</td><td>270</td></tr>
  <tr><td>11</td><td>3.1</td><td>2</td></tr>
  <tr><td>12</td><td>2.4</td><td>1</td></tr>
  <tr><td>13</td><td>2.5</td><td>1</td></tr>
</table>

As you can see, Python 2.6 makes up 109,134 out of 2,707,632 downloads in the
past 30 days. This represents roughly 4% of downloads. Is that low enough to
drop support?[^6].

`null` also makes up approximately 3.4% of downloads. These
are downloads from PyPI using clients that do not support sending the statistics
we're querying against. This can be an older version of `pip` or alternate
clients. You also see 341 downloads from 1.17, which is...who knows!
When making maintenance decisions you should factor these unknowns as you
feel appropriate.

### Query: OpenSSL versions

`cryptography` supports a wide variety of OpenSSL versions. However, supporting
0.9.8 and 1.0.0 are a significant challenge since they are missing many of the
features we need (and are no longer supported by upstream). Let's craft a
query to see what versions of OpenSSL are in use:

```sql
SELECT
  details.system.name,
  REGEXP_EXTRACT(details.openssl_version, r"^OpenSSL ([^ ]+) ") as openssl_version,
  COUNT(*) as download_count,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
WHERE
  details.openssl_version IS NOT NULL
GROUP BY
  details.system.name,
  openssl_version,
HAVING
  download_count >= 100
ORDER BY
  download_count DESC
LIMIT 100
```

This gives us a view of what OpenSSL is linked against Python broken down by platform:

<table style="width:100%">
  <tr><th>Row</th><th>details_system_name</th><th>openssl_version</th><th>download_count</th></tr>
  <tr><td>1</td><td>Linux</td><td>1.0.1f</td><td>56152115</td></tr>
  <tr><td>2</td><td>Linux</td><td>1.0.1t</td><td>26301089</td></tr>
  <tr><td>3</td><td>Linux</td><td>1.0.2g</td><td>25892721</td></tr>
  <tr><td>4</td><td>Linux</td><td>1.0.1e-fips</td><td>24082526</td></tr>
  <tr><td>5</td><td>Linux</td><td>1.0.1</td><td>19199858</td></tr>
  <tr><td>6</td><td>Darwin</td><td>0.9.8zh</td><td>15528451</td></tr>
  <tr><td>7</td><td>Linux</td><td>1.0.1k-fips</td><td>9029119</td></tr>
  <tr><td>8</td><td>Linux</td><td>1.0.2j</td><td>8499043</td></tr>
  <tr><td>9</td><td>Windows</td><td>1.0.2h</td><td>5037984</td></tr>
  <tr><td>10</td><td>Darwin</td><td>1.0.2j</td><td>3077958</td></tr>
</table>

While I haven't provided the entire set of results it turns out less than 100,000
downloads out of 210,063,137 were made using OpenSSL 1.0.0. 0.9.8 holds a much
greater share, but only due to Darwin (aka macOS...aka OS X). In cryptography's
case we statically link wheels on Mac and Windows so we can ignore the OpenSSL
version on those platforms. Looks like dropping 0.9.8 and 1.0.0 is probably safe![^7]

### Query: Most Popular Projects

Maybe you just want to know how popular your package is relative to others in
the past 30 days.

```sql
SELECT
  file.project,
  COUNT(*) as total_downloads,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
GROUP BY
  file.project
ORDER BY
  total_downloads DESC
LIMIT 100
```

### Query: How Many Downloads Did My Project Get?

Or maybe just your package:

```sql
SELECT
  COUNT(*) as total_downloads,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
WHERE
  file.project = 'cryptography'
```

### Query: Downloads By Filename Filtered By Project and Version

If you ship multiple release artifacts (platform/version specific wheels as
well as sdist) this query can show you their popularity.

```sql
SELECT
  file.filename,
  COUNT(*) as total_downloads,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
WHERE
  file.project = 'cryptography'
AND
  file.filename like '%1.6%'
GROUP BY
  file.filename
ORDER BY
  total_downloads DESC
LIMIT 100
```


### Query: Python 2 vs 3 For A Single Project

We broke it down by Python release previously, but what if you just want to know 2 vs 3?

```sql
SELECT
  ROUND(100 * SUM(CASE WHEN REGEXP_EXTRACT(details.python, r"^([^\.]+)") = "3" THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_3,
  COUNT(*) as download_count,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
WHERE
  file.project = 'cryptography'
ORDER BY
  download_count DESC
LIMIT 100
```

In cryptography's case Python 3 currently makes up 11% of downloads[^8].


### Query: Percentage of Downloads By Python 3 In The Top 100

Of course, a trivial modification and we can see the Python 3 percentage in the top 100 packages:

```sql
SELECT
  file.project,
  ROUND(100 * SUM(CASE WHEN REGEXP_EXTRACT(details.python, r"^([^\.]+)") = "3" THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_3,
  COUNT(*) as download_count,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
group by
  file.project
ORDER BY
  download_count DESC
LIMIT 100
```


### Query: Highest Python 3 Usage With More Than 100,000 Downloads Per 30 Days

```sql
SELECT
  file.project,
  ROUND(100 * SUM(CASE WHEN REGEXP_EXTRACT(details.python, r"^([^\.]+)") = "3" THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_3,
  COUNT(*) as download_count,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
group by
  file.project
HAVING
  download_count > 100000
ORDER BY
  percent_3 DESC
LIMIT 100
```

The top 10 is interesting here:

<table style="width:100%">
  <tr><th>Row</th><th>Date</th><th>Percentage</th><th>Total Downloads</th></tr>
  <tr><td>1</td><td>async-timeout</td><td>98.8</td><td>117724</td></tr>
  <tr><td>2</td><td>multidict</td><td>89.4</td><td>157661</td></tr>
  <tr><td>3</td><td>yarl</td><td>82.1</td><td>118081</td></tr>
  <tr><td>4</td><td>aiohttp</td><td>74.6</td><td>209035</td></tr>
  <tr><td>5</td><td>azure-servicebus</td><td>72.3</td><td>151706</td></tr>
  <tr><td>6</td><td>plumbum</td><td>60.1</td><td>131295</td></tr>
  <tr><td>7</td><td>mysqlclient</td><td>57.9</td><td>140905</td></tr>
  <tr><td>8</td><td>pkginfo</td><td>56.2</td><td>102214</td></tr>
  <tr><td>9</td><td>azure-nspkg</td><td>55.2</td><td>214821</td></tr>
  <tr><td>10</td><td>azure-storage</td><td>54.7</td><td>213129</td></tr>
</table>

### Query: Which Packages By Percentage Are Downloaded Most Often Via Python 2.6?

```sql
SELECT
  file.project,
  ROUND(100 * SUM(CASE WHEN REGEXP_EXTRACT(details.python, r"^([^\.]+\.[^\.]+)") = "2.6" THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_26,
  SUM(CASE WHEN REGEXP_EXTRACT(details.python, r"^([^\.]+\.[^\.]+)") = "2.6" THEN 1 ELSE 0 END) as total_26_downloads,
  COUNT(*) as total_downloads,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
WHERE
  details.python IS NOT NULL AND
  # Exclude things which are stdlib backports *for* Python 2.6
  file.project NOT IN ("argparse", "ordereddict")
GROUP BY
  file.project,
HAVING
  total_downloads > 5000
ORDER BY
  percent_26 DESC
LIMIT 250
```

### Query: Most Used Installers/Versions

```sql
SELECT
  details.installer.name,
  details.installer.version,
  COUNT(*) as total_downloads
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
GROUP BY
  details.installer.name,
  details.installer.version
ORDER BY
  total_downloads DESC
LIMIT 100
```

### Query: Downloads By Country

```sql
SELECT
  country_code,
  COUNT(*) as downloads,
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -31, "day"),
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "day")
  )
GROUP BY
  country_code
ORDER BY
  downloads DESC
LIMIT 100
```

### Query: Python 3 Download Percentage Across PyPI Grouped By Month

```sql
SELECT
  STRFTIME_UTC_USEC(timestamp, "%Y-%m") AS yyyymm,
  ROUND(100 * SUM(CASE WHEN REGEXP_EXTRACT(details.python, r"^([^\.]+)") = "3" THEN 1 ELSE 0 END) / COUNT(*), 1) AS percent_3,
  COUNT(*) as download_count
FROM
  TABLE_DATE_RANGE(
    [the-psf:pypi.downloads],
    DATE_ADD(CURRENT_TIMESTAMP(), -1, "year"),
    CURRENT_TIMESTAMP()
  )
group by
  yyyymm
ORDER BY
  yyyymm DESC
LIMIT 100
```

This is an interesting query for obvious reasons[^10], but also one that processes quite a bit of data. Here are the results through January 2017:

<table style="width:100%">
  <tr><th>Date</th><th>Percentage</th><th>Total Downloads</th></tr>
  <tr><td>2017-01</td><td>10.3</td><td>619254519</td></tr>
  <tr><td>2016-12</td><td>9.6</td><td>537529176</td></tr>
  <tr><td>2016-11</td><td>9.2</td><td>563208796</td></tr>
  <tr><td>2016-10</td><td>8.9</td><td>528112583</td></tr>
  <tr><td>2016-09</td><td>7.9</td><td>538323554</td></tr>
  <tr><td>2016-08</td><td>7.4</td><td>542956355</td></tr>
  <tr><td>2016-07</td><td>7.1</td><td>483143293</td></tr>
  <tr><td>2016-06</td><td>7.1</td><td>428402893</td></tr>
  <tr><td>2016-05</td><td>7.1</td><td>137138123</td></tr>
  <tr><td>2016-03</td><td>6.6</td><td>63615708</td></tr>
  <tr><td>2016-02</td><td>6.5</td><td>314275153</td></tr>
  <tr><td>2016-01</td><td>6.4</td><td>98290315</td></tr>
</table>

Data ingestion into the BigQuery data set was spotty prior to June 2016[^9], but you can see a significant uptick in Python 3 based downloads over 2016.
[If these trends continue...](https://frinkiac.com/caption/S08E11/289555)

## Decisions

These queries just scratch the surface of the data you can get about the Python ecosystem at large and your project in particular. Write your own queries and make your decisions about platform, version, and implementation with real information.

Of course, this doesn't address more granular usage, exception tracking, and other analytics web and mobile devs frequently take for granted, but that's a [separate blog post](https://alexgaynor.net/2015/sep/03/telemetry-for-open-source/).



Thanks to [Donald Stufft](https://twitter.com/dstufft) for his tireless efforts on PyPI, [Alex Gaynor](https://alexgaynor.net) for several of the queries in this blog post, and Google for generously donating capacity on BigQuery for PyPI data.


[^1]: I am legally obligated to link [Frinkiac](https://frinkiac.com) in every blog post.
[^3]: Or worse, don't complain until after you've done the removal.
[^4]: ...which can contribute significantly to project burnout.
[^5]: Queries are charged against your account, but you get 1TB free per month and cached queries won't count against it.
[^6]: It's not for cryptography. We have a deprecation warning in place but an indefinite timeline for removal of support.
[^7]: Issues for [1.0.0 removal](https://github.com/pyca/cryptography/issues/3002) and [0.9.8 removal](https://github.com/pyca/cryptography/issues/2836).
[^8]: requests is at 14%, bcrypt 29%, django 32.9%, and jupyter 33.3%. Packages can vary wildly!
[^9]: But it shouldn't be biased, so these percentages are likely to be accurate.
[^10]: To my knowledge no one has posted this information previously, so we're accidentally breaking new ground with our queries!
