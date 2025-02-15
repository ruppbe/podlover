The Backstory: Podlove - a Wordpress plugin for Podcasting
----------------------------------------------------------

The **[Podlove](https://podlove.org/)** podcasting suite is an open
source toolset to help you publish and manage a podcast within a
Wordpress blog. Over the years it has become the de facto standard for
easy while flexible podcast publishing in the German-speaking podcast
community.

Podlove includes an analytics dashboard to give you an overview of how
your podcast is performing over time and various dimensions such as
media formats. While being a practical overview for everyday analytics,
it is limited when it comes to more complex or fine-grained analytics.

podlover Brings Podlove Data into R
-----------------------------------

The `podlover` package allows you to access the access data behind the
Podlove dashboard. It connects to the relevant Wordpress MySQL tables,
fetches the raw data, connects and cleans it into a tidy dataset with
one row per download attempt.

Furthermore, it allows you to:

-   plot download data for multiple episodes as point, line, area and
    ridgeline graphs
-   use options for absolute vs relative display (think: release dates
    vs. days since release) and cumulative vs. non-cumulative display.
-   compare episodes, epsiode formats, sources/contexts, podcast
    clients, and operating systems over time
-   create and compare performance data for episode launches and
    long-term performance
-   calculate and plot regressions to see if you are gaining or losing
    listeners over time.

This vignette demonstrates what `podlover` can do.

Installation
------------

This package is based on the statistical programming framework R. If
you’re a podcast producer who is new R, you will need to install R as
well as its programming environment RStudio and familiarize yourself
with it. Both of them are free open source tools. [Start here to get
RStudio and R](https://rstudio.com/products/rstudio/download/).

`podlover` is available as a package from GitHub at
<a href="https://github.com/lordyo/podlover" class="uri">https://github.com/lordyo/podlover</a>
and can be installed via `devtools`:

    # install devtools if you don't have it already
    install.packages("devtools")

    # install podlover from GitHub
    devtools::install_github("lordyo/podlover")
    # intall this fork from GitHub
    devtools::install_github("ruppbe/podlover")

Once installed, you can load the package.

    library(podlover)

Ways to Access Podlove Data
---------------------------

There are two ways to get your download data into `podlover`:

-   You either **fetch the data directly** from the Wordpress data base
    with `podlove_get_and_clean()`. This will give you the most recent
    data and, once established, is the most comfortable way of accessing
    data. Establishing the connection can be tricky though.
-   Or you **download the necessary tables** and feed it to podlover
    with `podlove_clean_stats()`. This is easier, but takes longer and
    will only give you a snapshot of the data at a certain point in
    time.

Fetching Download Data via a Database Connection
------------------------------------------------

Behind every Wordpress site is a MySQL database containing almost
everything that’s stored in the blog. When installing Podlove under
Wordpress, the plugin creates additional database tables containing
podcast-specific data. The function `podlove_get_and_clean()` fetches
those.

To make that happen, you will need:

1.  `db_name`: The Wordpress database’s name.
2.  `db_host`: The (external) hostname of the database.
3.  `db_user`: The databases’s user name (usually not the same as your
    Wordpress login)
4.  `db_password`: The user’s password for database (usually not the
    same as your Worpdress password)
5.  Permissions to access your database from an external IP address.
6.  The names of the database tables

### Database name, user and password

`db_name`, `db_user` and `db_password` can be found in the
`wp-config.php` file in the root folder of your podcast’s Wordpress
directory. Look for the following passage:

    // ** MySQL settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define( 'DB_NAME', 'lorem_wp_ipsum' );

    /** MySQL database username */
    define( 'DB_USER', 'lorem_dolor' );

    /** MySQL database password */
    define( 'DB_PASSWORD', 'my_password' );

### External Database Hostname

Note that this file also includes a hostname, but this is the internal
hostname - you’re looking for the **external hostname**. This you will
need to get from your hoster’s admin panel, usually where MySQL
databases are managed. Check your hoster’s support section if you get
stuck.

### Access permissions

MySQL databases are sensitive to hacking attacks, which is why they
usually aren’t accessible to just any visitor - even if she has the
correct access information. You will probably need to set allow an
**access permission** to your database and user from the IP address R is
running on (“whitelisting”). This is also done via your hoster’s admin
panel. Check your hoster’s support sections for more info. (Note: Some
hosters are stricter and don’t allow any access except via SSH tunnels,
which you will need ot establish first - how depends on your OS and
hoster. Check your hoster’s support section on how to do this.)

### Prefix of the Table Names

Finally, you might need to check if the **tables name prefix** in your
Wordpress database corresponds to the usual naming conventions. Most
Wordpress installations start the tables with `wp_`, but sometimes, this
prefix differs (e.g. `wp_wtig_`). For starters, you can just try to use
the default prefix built into the function. If the prefix is different
than the default, you will get an error message. If that’s the case,
access your hoster’s MySQL management tool (e.g. phpMyAdmin, PHP
Workbench), open your database and check if the tables are starting with
something else than just `wp_...` and write down the prefix. You can of
course also use a locally installed MySQL tool to do so (e.g. HeidiSQL).

### Downloading the data

Once you gathered all this, it’s time to access your data and store it
to a data frame:

    downloads <- podlove_get_and_clean()

Four input prompts will show up, asking for the database name, user,
password and host. You have the option to save these values to your
system’s keyring, so you don’t have to enter them repeatedly. Use
`?rstudioapi::askForSecret` to learn more about where these values are
stored or `?keyring` to learn more how keyrings and how they are used
within R.

After entering the information, you should see something like this:

    connection established
    fetched table wp_podlove_downloadintentclean
    fetched table wp_podlove_mediafile
    fetched table wp_podlove_useragent
    fetched table wp_podlove_episode
    fetched table wp_posts
    connection closed

### Troubleshooting

You might also get an error message, meaning something went wrong. If
you see the following error message…

    Error in .local(drv, ...) : 
      Failed to connect to database: Error: Access denied for user 'username'@'XX.XX.XX.XXX' to database 'databasename'

…then the function couldn’t access the databse. This means either that
there’s something wrong with your database name, user name, password or
host name (see sections “Database name, user name, password” or
“External Database Hostname” above) Or it could mean that access to this
database with this username is restricted, i.e. your IP is not
whitelisted (see “Access Permissions” above). If you can’t make that
work, you’re only option is to download the tables yourself (see
“Working with Local Table Downloads”).

If your error says…

    connection established
    Error in .local(conn, statement, ...) : 
      could not run statement: Table 'dbname.tablename' doesn't exist

…this means you were able to access the database (congrats!), but the
table names/prefix are incorrect. Check your table name prefix as
described under “Prefix of the table names” and try again while
specifying the prefix:

    downloads <- podlove_get_and_clean(tbl_prefix = "PREFIX")

Working with Local Table Downloads
----------------------------------

If you can’t or don’t want to work with `podlove_get_and_clean()`, you
can still analyze your data by downloading the individual database
tables yourself and feed it directly to the cleaning function
`podlove_clean_stats()`.

First, you will need to get the necessary tables. The easiest way is to
use your hoster’s database management tool, e.g. phpMyAdmin or PHP
Workbench. These can usually be accessed from your hosting
administration overview: Look for a “databases”, “MySQL” or “phpMyAdmin”
option, find a list of tables, usually starting with `wp_...`, select
the table and look for an “export” option. Export the tables to CSV. If
you get stuck, check your hoster’s support section.

**Warning: When using database tools, you can break things - i.e. your
site and your podcast. To be on the safe side, always make a backup
first, don’t change any names or options, and don’t delete anything!**

You will need the following tables, each in its own CSV file with
headings (column titles):

-   `wp_podlove_downloadintentclean`
-   `wp_podlove_episode`
-   `wp_podlove_mediafile`
-   `wp_podlove_useragent`
-   `wp_posts`

Note: The prefix of the tables (here `wp_`) might be different or longer
in your case.

Once you have downloaded the tables, you need to import them into R as
data frames: to use the `podlove_clean_stats()` function to connect the
table and clean the data:

    # replace file names with your own
    download_table <- read.csv("wp_podlove_downloadintentclean.csv", as.is = TRUE)
    episode_table <- read.csv("wp_podlove_episode.csv", as.is = TRUE)
    mediafile_table <- read.csv("wp_podlove_mediafile.csv", as.is = TRUE)
    useragent_table <- read.csv("wp_podlove_useragent.csv", as.is = TRUE)
    posts_table <- read.csv("wp_posts.csv", as.is = TRUE)

    # connect & clean the tables
    downloads <- podlove_clean_stats(df_stats = download_table,
                                         df_episode = episode_table,
                                         df_mediafile = mediafile_table,
                                         df_user = useragent_table,
                                         df_posts = posts_table)

Note: If during reading of the CSV files you notice that your data
frames consist of only one column, check your CSV files which separator
is used. Some use semicolons (`;`) instead of commas (`,`). If that’s
the case, use the option `sep = ";"` (or any other separator) inside
your `read.csv()` commands, e.g.

    download_table <- read.csv("wp_podlove_downloadintentclean.csv", as.is = TRUE, sep = ";")

Create Example Data
-------------------

`podlover` includes a number of functions to generate example download
tables. This can be useful if you want to test the package without
having real data, or to write reproducible examples for a vignette like
this. We will use an example data set for the next chapters.

Generate some random data with the function `podlove_create_example()`
with ~10.000 downloads in total. The `seed` parameter fixes the
randomization to give you the same data as in this example. The `clean`
parameter states that you want a dataframe of cleaned data, not raw
input tables.

    downloads <- podlove_create_example(total_dls = 10000, seed = 12, clean = TRUE)

Here it is:

    print(downloads)
    #> # A tibble: 9,116 x 20
    #>    ep_number title ep_num_title duration                   post_date 
    #>    <chr>     <chr> <chr>        <Duration>                 <date>    
    #>  1 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  2 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  3 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  4 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  5 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  6 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  7 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  8 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #>  9 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #> 10 01        Asht~ 01: Ashton-~ 1948.105s (~32.47 minutes) 2019-01-01
    #> # ... with 9,106 more rows, and 15 more variables: post_datehour <dttm>,
    #> #   ep_age_hours <dbl>, ep_age_days <dbl>, hours_since_release <dbl>,
    #> #   days_since_release <dbl>, source <chr>, context <chr>, dldate <date>,
    #> #   dldatehour <dttm>, weekday <ord>, hour <int>, client_name <chr>,
    #> #   client_type <chr>, os_name <chr>, dl_attempts <int>

Nice - this is all the data you need for further analysis. It contains
information about the episode (what was downloaded?), the download (when
was it downloaded?) and the user agent (how / by which agent was it
downloaded?).

By the way, if you want get access the raw data or try out the cleaning
function, you can set the `podlove_create_example()` parameter
`clean = FALSE`:

    table_list <- podlove_create_example(total_dls = 10000, seed = 12)

The result is a list of 5 named tables (`posts`, `episodes`,
`mediafiles`, `useragents`, `downloads`) wrapped in a list. Now all you
have to do is feed them to the cleaning function:

    downloads <- podlove_clean_stats(df_stats = table_list$downloads,
                                     df_mediafile = table_list$mediafiles,
                                     df_user = table_list$useragents,
                                     df_episodes = table_list$episodes,
                                     df_posts = table_list$posts)

Summary
-------

Now that you have the clean download data, it’s time to check it out.
`podlover` includes a simple summary function to give you an overview of
the data:

    podlove_podcast_summary(downloads)
    #> 'downloads': 
    #> 
    #> A podcast with 10 episodes, released between 2019-01-01 and 2019-12-05.
    #> 
    #> Total runtime:  11m 4d 22H 0M 0S.
    #> Average time between episodes: 2928240s (~4.84 weeks).
    #> 
    #> Episodes were downloaded 9116 times between 2019-01-01 and 2020-01-04.
    #> 
    #> Downloads per episode: 911.6
    #> min: 148 | 25p: 413 | med: 872 | 75p: 1330 | max: 1859
    #> 
    #> Downloads per day: 24.7
    #> min: 1 | 25p: 3 | med: 7 | 75p: 16 | max: 1095
    #> NULL

If you set the parameter `return_params` to `TRUE`, you can access the
individual indicators directly. The `verbose` parameter defines if you
want to see the printed summary.

    pod_sum <- podlove_podcast_summary(downloads, return_params = TRUE, verbose = FALSE)

    names(pod_sum)
    #>  [1] "n_episodes"                 "ep_first_date"             
    #>  [3] "ep_last_date"               "runtime"                   
    #>  [5] "ep_interval"                "n_downloads"               
    #>  [7] "dl_first_date"              "dl_last_date"              
    #>  [9] "downloads_per_episode_mean" "downloads_per_episode_5num"
    #> [11] "downloads_per_day_mean"     "downloads_per_day_5num"

    pod_sum$n_downloads
    #> [1] 9116
    pod_sum$dl_last_date
    #> [1] "2020-01-04 22:00:00 UTC"

Download curves
---------------

One of the main features of the `podlover` package is that it lets you
plot all kinds of download curves over time - aggregated and grouped,
with relative and absolute starting points. The `podlove_plot_curves()`
function (which is based on the `ggplo2` package) helps you do that.

### Parameters

`podlove_plot_curves()` accepts the following parameters for a graph:

-   `dldata`: The clean data to be analyzed, as prepared by the import
    or cleaning function.
-   `gvar`: The grouping variable. Defining one will create multiple
    curves, one for each group. This needs to be one of the variables
    (columns) in the clean data:
    -   `ep_number`: The episode’s official number
    -   `title`: The episode’s title
    -   `ep_num_title`: The episode’s title with the number in front
    -   `source`: The dowload source - e.g. “feed” for RSS, “webplayer”
        for plays on a website, “download” for file downloads
    -   `context`: The file type for feeds and downloads, “episode” for
        feed accesses
    -   `client_name`: The client application (e.g. the podcatcher’s or
        brower’s name)
    -   `client_type`: A more coarse grouping of the clients,
        e.g. “mediaplayer”, “browser”, “mobile app”.
    -   `os_name`: The operating system’s name of the client
        (e.g. Android, Linux, Mac)
    -   Any other grouping variable you create yourself from the
        existing data.
-   `hourly` (`podlove_prepare_stats_for_graph()` only): If set to
    `TRUE`, the downloads will be shown per hour, otherwise per day
-   `relative`: If set to `TRUE`, the downloads will be shown relative
    to their publishing date, i.e. all curves starting at 0. Otherwise,
    the curves will show the download on their specific dates.
-   `cumulative`: If set to `TRUE`, the downloads will accumulate and
    show the total sum over time (rising curve). Otherwise, they will
    uncumulated downloads (scattered peaks).
-   `plot_type`: What kind of plot to use - either line plots (`"line"`)
    on one graph, or individual ridgeline plots (`"ridge"`).
-   `labelmethod`: Where to attach the labels (`"first.points"` for the
    beginning of the line, `"last.points"` for the end of the line)
-   `printout`: If you want to print the plot directly from the function
    without handing it over to a ggplot object.

### Total downloads over time

Let’s say you want to see the daily total downloads of your podcast over
time, in accumulated fashion. Here, you are not specifying any `gvar`
(which means you’ll get just one curve instead of many). `hourly` is set
to `FALSE` (= daily data) and `relative` is set `FALSE` (absolute
dates).

    g_tdlacc <- podlove_plot_curves(dldata = downloads, 
                                    hourly = FALSE, 
                                    relative = FALSE, 
                                    cumulative = TRUE, 
                                    plot_type = "line",
                                    printout = FALSE)
    #> Warning in podlove_plot_curves(dldata = downloads, hourly = FALSE, relative =
    #> FALSE, : option 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Warning: Ignoring unknown parameters: hourly

    print(g_tdlacc)

![](README_files/figure-markdown_strict/unnamed-chunk-9-1.png)

If we don’t cumulate the data, we can see the individual spikes of the
episode launches:

    g_tdl <- podlove_plot_curves(dldata = downloads, 
                                  hourly = FALSE, 
                                  relative = FALSE, 
                                  cumulative = FALSE, 
                                  plot_type = "line",
                                  printout = FALSE)
    #> Warning in podlove_plot_curves(dldata = downloads, hourly = FALSE, relative =
    #> FALSE, : option 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Warning: Ignoring unknown parameters: hourly

    print(g_tdl)

![](README_files/figure-markdown_strict/unnamed-chunk-10-1.png)

### Downloads by episode

Now you want to look at the individual episodes. For this, you will need
to use the `gvar` parameter. For an episode overviewer, you can either
set it to `title`, `ep_number` or `ep_num_title`. Here, we’re using
`title` (unquoted!), and add specify the `labelmethod` to show the
labels at the beginning of the curves.


    g_ep_dlsacc <- podlove_plot_curves(dldata = downloads, 
                                       gvar = title, 
                                       hourly = FALSE, 
                                       relative = FALSE, 
                                       cumulative = TRUE, 
                                       plot_type = "line",
                                       labelmethod = "first.points",
                                       printout = FALSE)
    #> Warning in podlove_plot_curves(dldata = downloads, gvar = title, hourly =
    #> FALSE, : option 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Warning: Ignoring unknown parameters: hourly

    print(g_ep_dlsacc)

![](README_files/figure-markdown_strict/unnamed-chunk-11-1.png)

As you can see, this shows the curves spread over the calendar X axis.
But how do the episodes hold up against each other? For this, we will
use the parameter `relative = TRUE`, which lets all curves start at the
same point. The labelling paramter `labelmethod = "last.points"` works
better for this kind of curve.


    g_ep_dlsaccrel <- podlove_plot_curves(dldata = downloads, 
                                       gvar = title, 
                                       hourly = FALSE, 
                                       relative = TRUE, 
                                       cumulative = TRUE, 
                                       plot_type = "line",
                                       labelmethod = "last.points",
                                       printout = FALSE)
    #> Warning in podlove_plot_curves(dldata = downloads, gvar = title, hourly =
    #> FALSE, : option 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Warning: Ignoring unknown parameters: hourly

    print(g_ep_dlsaccrel)

![](README_files/figure-markdown_strict/unnamed-chunk-12-1.png)

If you want to look at the uncumulated data, the line plot doesn’t work
very well. For this, a ridge plot is the right choice (but only if you
don’t have too many episodes):


    g_ep_dls <- podlove_plot_curves(dldata = downloads, 
                                       gvar = ep_num_title, # better for sorting
                                       hourly = FALSE, 
                                       relative = FALSE, 
                                       cumulative = FALSE, # no cumulation
                                       plot_type = "ridge", # use a ridgeline plot
                                       printout = FALSE)
    #> Warning in podlove_plot_curves(dldata = downloads, gvar = ep_num_title, : option
    #> 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Warning: Ignoring unknown parameters: hourly

    print(g_ep_dls)

![](README_files/figure-markdown_strict/unnamed-chunk-13-1.png)

### Downloads by other parameters

You can compare not only episodes, but also aspects of episodes or
downloads. Let’s look at the parameter `source`, which lists by what way
our listeners get their episodes. The labelmethod here is set to
`angled.boxes`:


    g_source_acc <- podlove_plot_curves(dldata = downloads, 
                                    gvar = source,
                                    hourly = FALSE, 
                                    relative = FALSE, 
                                    cumulative = TRUE,
                                    plot_type = "line", 
                                    labelmethod = "angled.boxes", #looks nice
                                    printout = FALSE)
    #> Warning in podlove_plot_curves(dldata = downloads, gvar = source, hourly =
    #> FALSE, : option 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Warning: Ignoring unknown parameters: hourly

    print(g_source_acc)

![](README_files/figure-markdown_strict/unnamed-chunk-14-1.png)

### Doing More with Curves

The `podlove_plot_curves()` function is actually a wrapper function
combining the data preparation function
`podlove_prepare_stats_for_graph()` and the plotting function
`podlove_graph_download_curves()`. If you want to tweak the data behind
the plots, create your own plots or automate certain parts of the
plotting, consider using a two-step process with those two functions.
Learn more about how they work by reading their help files:

    ?podlove_prepare_stats_for_graph

    ?podlove_graph_download_curves

You can of course also go to the source of these plots - the clean data
(`downloads` in the examples above). There’s no rule against adding your
own variables to the data set and using those in the plots. For example,
you might know that episodes 2, 3, 5 and 8 had guests while the other
episodes hadn’t.


    # create a guest table showing which episodes had guests
    guest_episodes <- tibble::tibble(
      ep_number = c("01", "02", "03", "04", "05", "06", "07", "08", "09", "10"),
      guest =     c(  F,    T,    T,    F,    T,    F,    F,    T,    F,    F)
    )

    # join the guest table to the downloads table
    downloads_with_guests <- downloads %>% 
      dplyr::left_join(guest_episodes, by = "ep_number")

    # plot the downloads with your new variable "guest"
    podlove_plot_curves(downloads_with_guests, gvar = guest, printout = FALSE)

![](README_files/figure-markdown_strict/unnamed-chunk-15-1.png)

### Reducing the number of episodes

If you have a lot of episodes, the curve diagrams can become difficult
to read. To reduce the number of curves, you can use the parameter
`last_n` to show only the most recent n episodes.


    g_ep_dlsacc_red <- podlove_plot_curves(dldata = downloads, 
                                       gvar = title, 
                                       hourly = FALSE, 
                                       relative = FALSE, 
                                       cumulative = TRUE, 
                                       plot_type = "line",
                                       labelmethod = "first.points",
                                       printout = FALSE,
                                       last_n = 3)
    #> Warning in podlove_plot_curves(dldata = downloads, gvar = title, hourly =
    #> FALSE, : option 'hourly' is depreciated. Use option 'time_unit' instead.
    #> Selecting by ep_rank
    #> Warning: Ignoring unknown parameters: hourly

    print(g_ep_dlsacc_red)

![](README_files/figure-markdown_strict/unnamed-chunk-16-1.png)

If you use a negative number for `last_n`, the first n epsiodes get
shown. Of course, if you have more sophisticated filtering needs, you
can always use `dplyr::filter()` to change the original data directly.

Epsiode Performance
-------------------

Podcast episode downloads typically follow a “heavy front, long tail”
distribution: Many downloads are made over automatic downloads by
podcatchers, while fewer are downloaded in the long-term. It therefore
helps to distinguish between the episode launch period (the heavy front)
and the long term performance (the long tail). For this, you can use the
function `podlove_performance_stats()`, which creates a table of
performance stats per episode. For this, you will first need to define
how long a “launch” goes and when the long-term period starts. Those two
don’t have to be the same. Here, we’ll use 0-3 days for the launch and
30 and above for the long-term.

    perf <- podlove_performance_stats(downloads, launch = 3, post_launch = 30)

    perf
    #> # A tibble: 10 x 7
    #>    ep_number title ep_num_title   dls dls_per_day dls_per_day_at_~
    #>    <chr>     <chr> <chr>        <int>       <dbl>            <dbl>
    #>  1 01        Asht~ 01: Ashton-~  1859        5.04            404. 
    #>  2 02        Mary~ 02: Mary To~  1727        5.04            359. 
    #>  3 03        Sama~ 03: Samanth~  1330        5.03            366. 
    #>  4 04        Shap~ 04: Shapins~  1072        5.05            233. 
    #>  5 05        Gwoy~ 05: Gwoyeu ~   936        5.02            184. 
    #>  6 06        Acut~ 06: Acute m~   808        5.04            177. 
    #>  7 07        Ficu~ 07: Ficus a~   542        5.01            114. 
    #>  8 08        Cort~ 08: Cortina~   413        5.03             88.2
    #>  9 09        Whit~ 09: White-w~   281        5.01             83.8
    #> 10 10        Debo~ 10: Debora ~   148        4.93             45.9
    #> # ... with 1 more variable: dls_per_day_after_launch <dbl>

    colnames(perf)
    #> [1] "ep_number"                "title"                   
    #> [3] "ep_num_title"             "dls"                     
    #> [5] "dls_per_day"              "dls_per_day_at_launch"   
    #> [7] "dls_per_day_after_launch"

The table shows four values per episode: The overall downlaods, the
overall average downloads, the average downloads during the launch and
the average downloads during the post-launch period. If you want to see
a ranking of the best launches, you can just sort the list:

    perf %>%
      dplyr::select(title, dls_per_day_at_launch) %>% 
      dplyr::arrange(desc(dls_per_day_at_launch))
    #> # A tibble: 10 x 2
    #>    title                  dls_per_day_at_launch
    #>    <chr>                                  <dbl>
    #>  1 Ashton-under-Lyne                      404. 
    #>  2 Samantha Smith                         366. 
    #>  3 Mary Toft                              359. 
    #>  4 Shapinsay                              233. 
    #>  5 Gwoyeu Romatzyh                        184. 
    #>  6 Acute myeloid leukemia                 177. 
    #>  7 Ficus aurea                            114. 
    #>  8 Cortinarius violaceus                   88.2
    #>  9 White-winged fairywren                  83.8
    #> 10 Debora Green                            45.9

So there are episodes with different launches strengths long-term
performance. Can you plot them against each other? Yes, you can! The
function `podlove_graph_performance()` gives you a nice four-box grid.
The top right corner is showing “evergreen” episodes with strong
launches and long-term performance, the top left the “shooting stars”
with strong launches and loss of interest over time, the bottom right
shows “slow burners” which took a while to get an audience, and the
bottom left is showing… well, the rest.

    g_perf <- podlove_graph_performance(perf, label = title, printout = FALSE)

    print(g_perf)

![](README_files/figure-markdown_strict/unnamed-chunk-19-1.png)

Or, if you want to skip the step of generating data and just get the
plot, use the wrapper function `podlove_plot_performance()`:

    podlove_plot_performance(dldata = downloads, launch = 3, label = title, post_launch = 30)

Regression Analysis: Is your podcast gaining or losing listeners?
-----------------------------------------------------------------

The one question every podcast producer is asking themselves is: “Is my
podcast’s audience growing?”. This is not an easy question to answer,
because you’re dealing with lagging time series data. One approach to
deal answer the question is to calculate a regression model of downloads
against time or episode number (tip of the hat to [Bernhard Fischer, who
prototyped this
idea](https://community.podlove.org/t/implementation-of-regression-prototype/1994)).
If the number of downloads after a specified date after launch is rising
over time, the podcast is gaining listeners. If it falls, it’s losing
listeners. If it stays stable, it’s keeping listeners.

A regression analysis is usually shown as a regression model printout,
but understanding those requires some statistical knowledge. An easier
way is to plot your data and the corresponding regression line. The
function `podlove_plot_regression()` allows you to do both.

The function creates a dataset of downloads at a specific point in time.
The longer the period between launch and measuring point is, the more
valid the model will be - but you’ll also have less data points. For
this example, we’ll pick a period of 30 days after launch. You can also
choose if you want to use the `post_datehour` parameter (better if your
episodes don’t come out regularly), or `ep_rank`, which corresponds to
the episode number (it has a different name because episode numbers are
strings).

With the option `print_model`, you can display the model results:

    reg <- podlove_plot_regression(df_tidy_data = downloads, 
                                   point_in_time = 30, 
                                   predictor = ep_rank, 
                                   print_model = TRUE,
                                   print_plot = FALSE)
    #> 
    #> Call:
    #> stats::lm(formula = stats::reformulate(terms, response = "downloads"), 
    #>     data = df_regression_data)
    #> 
    #> Residuals:
    #>     Min      1Q  Median      3Q     Max 
    #> -81.218 -36.058  -4.682  41.853  81.424 
    #> 
    #> Coefficients:
    #>             Estimate Std. Error t value Pr(>|t|)    
    #> (Intercept) 1397.933     39.849   35.08 4.77e-10 ***
    #> ep_rank     -131.679      6.422  -20.50 3.35e-08 ***
    #> ---
    #> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    #> 
    #> Residual standard error: 58.33 on 8 degrees of freedom
    #> Multiple R-squared:  0.9813, Adjusted R-squared:  0.979 
    #> F-statistic: 420.4 on 1 and 8 DF,  p-value: 3.35e-08

The option `print_plot` shows the plot (which is also saved in the
object `reg`):

    reg <- podlove_plot_regression(df_tidy_data = downloads, 
                                   point_in_time = 30, 
                                   predictor = ep_rank, 
                                   print_model = FALSE,
                                   print_plot = TRUE)

![](README_files/figure-markdown_strict/unnamed-chunk-21-1.png)

Oh noes! It seems like your podcast is steadily losing listeners!

Finally
-------

There could be much more possible with Podlove data and R and
`podlover`. For example, some of the download data have not yet been
included in the import script, such as the geolocation data. Some of the
functions are still too manual and require wrapping functions to quickly
analyze data. If you want to contribute, check out the GitHub repo
under:

<a href="https://github.com/lordyo/podlover" class="uri">https://github.com/lordyo/podlover</a>

And if you want to use or contribute to the Podlove project (the
Podcasting tools), check out:

<a href="https://podlove.org" class="uri">https://podlove.org</a>
