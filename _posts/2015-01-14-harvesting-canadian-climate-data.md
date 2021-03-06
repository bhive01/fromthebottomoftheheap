--- 
title: "Harvesting Canadian climate data"
status: publish
layout: post
published: true
type: post
tags:
- API
- data
- climate
- download
- opendata
active: blog
category: R
alert: "As of May 11, 2016, Government Canada has updated the data download URLs such that the URLs created by <code>genURLS()</code> are no longer valid. The <a href=\"https://gist.github.com/gavinsimpson/8c13e3c5f905fd67cf85\">gist</a> contains updated functions that work with the URL and data download API."
---



In December I found myself helping one of our graduate students with a 
data problem; for one of their thesis chapters they needed a lot of 
hourly climate data for a handful of stations around Saksatchewan. All 
of this data was and is available for download from the Government of 
Canada's website, but with one catch; you had to download the hourly 
data one month at a time, manually! There is no interface to allow a 
user of the website to specify the data range they want and download 
all the data from a single station. I figured there had to be a better 
way, using R to automate the downloading. Thinking the solution I came 
up with might help other researchers needing to grab data from the 
Government of Canada's website save some time in the future, I wrote 
this post to document how we ended up doing it.

![Screenshot of Government of Canada's climate website]({{ site.url }}/assets/img/posts/screenshot-gc-climate-website.jpeg "Screenshot of Government of Canada's climate website")

The website itself is reasonably pretty but the way the web form worked 
to trigger the download of a CSV containing the data was a little 
tricky. You can see an example of the sort of data we were interested 
in 
[here](http://climate.weather.gc.ca/climateData/hourlydata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1996-01-30|2015-01-12&Year=2015&Month=1&Day=12); 
interestingly you are only shown a single day of data but when you 
click the big Download button you get the entire month containing the 
day shown in the HTML table. The web form was setting some hidden 
parameters that were added to the current page's URL once the Download 
button was clicked. Frustratingly, the same page that showed the HTML 
table also handled generating and returning the CSV download. Even more 
frustrating was that the script that they were using needed GET 
variables with almost the same names as some of the existing GET 
variables, just with different case, such as `StationID` and 
`stationID`, the latter of which is required for the CSV-creating 
script only. A further annoyance was that even though the CSV generated 
contained an entire month's worth of data, the URL still needed to 
contain the `Day` GET variable.

I'm sure I haven't whittled the URL down to the bare minimum required 
to trigger CSV generation and download, but I ended up using:

```
http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1996-01-30%7C2014-11-30&cmdB1=Go&Year=2003&Month=5&Day=27&format=csv&stationID=28011
```

which will get you the data for May 2003 from station 28011 (Regina RCS).

Having figured that out, I needed a little function that would generate 
the URLs we'd need to visit to get data covering the periods we wanted. 
Because the student needed multiple stations and the time periods of 
interest differed between stations (stations got moved and picked up 
new IDs so we needed to track those movements) I wrote a little 
function that would create a whole load of URLS if given a set of 
station IDs and start and end years.


{% highlight r %}
genURLS <- function(id, start, end) {
    years <- seq(start, end, by = 1)
    nyears <- length(years)
    years <- rep(years, each = 12)
    months <- rep(1:12, times = nyears)
    URLS <- paste0("http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=",
                   id,
                   "&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=",
                   years,
                   "&Month=",
                   months,
                   "&Day=27",
                   "&format=csv",
                   "&stationID=",
                   id)
    list(urls = URLS, ids = rep(id, nyears * 12), years = years, months = months)
}
{% endhighlight %}

The `genURLS()` function is pretty simple and just repeats each year 
integer in the sequence `start:end` 12 times, once per month, and then 
repeats the months `1:12` for as many years were requested. Then it 
builds up a character vector of URLs from these vectors `years`, 
`months` and `id`, the station ID.

If we wanted all the data for 2014 for the Regina RCS station then we 
could generate the URLs we'd need to visit as follows


{% highlight r %}
regina <- genURLS(28011, 2014, 2014)
length(regina$urls)
head(regina$urls)
{% endhighlight %}



{% highlight text %}
[1] 12
[1] "http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=2014&Month=1&Day=27&format=csv&stationID=28011"
[2] "http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=2014&Month=2&Day=27&format=csv&stationID=28011"
[3] "http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=2014&Month=3&Day=27&format=csv&stationID=28011"
[4] "http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=2014&Month=4&Day=27&format=csv&stationID=28011"
[5] "http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=2014&Month=5&Day=27&format=csv&stationID=28011"
[6] "http://climate.weather.gc.ca/climateData/bulkdata_e.html?timeframe=1&Prov=SK&StationID=28011&hlyRange=1953-01-30%7C2014-12-31&cmdB1=Go&Year=2014&Month=6&Day=27&format=csv&stationID=28011"
{% endhighlight %}

The function I used to grab all the data is a little more involved, 
partly because in a long-running job you don't want a single error due 
to a bad download to cause the entire job to end. Another reason for 
some of the complexity is that if the job did fail for some reason, as 
long as the files downloaded up to that point were OK/readable, I 
didn't want to download them again. Therefore the function downloads 
and saves all the CSV files first and only then do we try to read the 
data. The function is reasonably well-commented so I won't dwell on 
those details


{% highlight r %}
getData <- function(stations, folder, verbose = TRUE, delete = TRUE) {
    ## form URLS
    urls <- lapply(seq_len(NROW(stations)),
                   function(i, stations) {
                       genURLS(stations$StationID[i],
                               stations$start[i],
                               stations$end[i])
                   }, stations = stations)
 
    ## check the folder exists and try to create it if not
    if (!file.exists(folder)) {
        warning(paste("Directory:", folder,
                      "doesn't exist. Will create it"))
        fc <- try(dir.create(folder))
        if (inherits(fc, "try-error")) {
            stop("Failed to create directory '", folder,
                 "'. Check path and permissions.", sep = "")
        }
    }
 
    ## Extract the data from the URLs generation
    URLS <- unlist(lapply(urls, '[[', "urls"))
    sites <- unlist(lapply(urls, '[[', "ids"))
    years <- unlist(lapply(urls, '[[', "years"))
    months <- unlist(lapply(urls, '[[', "months"))
 
    ## filenames to use to save the data
    fnames <- paste(sites, years, months, "data.csv", sep = "-")
    fnames <- file.path(folder, fnames)
 
    nfiles <- length(fnames)
 
    ## set up a progress bar if being verbose
    if (isTRUE(verbose)) {
        pb <- txtProgressBar(min = 0, max = nfiles, style = 3)
        on.exit(close(pb))
    }
 
    out <- vector(mode = "list", length = nfiles)
    cnames <- c("Date/Time", "Year", "Month","Day", "Time", "Data Quality",
                "Temp (degC)", "Temp Flag", "Dew Point Temp (degC)",
                "Dew Point Temp Flag", "Rel Hum (%)", "Rel Hum Flag",
                "Wind Dir (10s deg)", "Wind Dir Flag", "Wind Spd (km/h)",
                "Wind Spd Flag", "Visibility (km)", "Visibility Flag",
                "Stn Press (kPa)", "Stn Press Flag", "Hmdx", "Hmdx Flag",
                "Wind Chill", "Wind Chill Flag", "Weather")
 
    for (i in seq_len(nfiles)) {
        curfile <- fnames[i]
 
        ## Have we downloaded the file before?
        if (!file.exists(curfile)) {    # No: download it
            dload <- try(download.file(URLS[i], destfile = curfile, quiet = TRUE))
            if (inherits(dload, "try-error")) { # If problem, store failed URL...
                out[[i]] <- URLS[i]
                if (isTRUE(verbose)) {
                    setTxtProgressBar(pb, value = i) # update progress bar...
                }
                next                             # bail out of current iteration
            }
        }
 
        ## Must have downloaded, try to read file
        ## skip first 16 rows of header stuff
        ## encoding must be latin1 or will fail - may still be problems with character set
        cdata <- try(read.csv(curfile, skip = 16, encoding = "latin1"), silent = TRUE)
 
        ## Did we have a problem reading the data?
        if (inherits(cdata, "try-error")) { # yes handle read problem
            ## try to fix the problem with dodgy characters
            cdata <- readLines(curfile) # read all lines in file
            cdata <- gsub("\x87", "x", cdata) # remove the dodgy symbol for partner data in Data Quality
            cdata <- gsub("\xb0", "deg", cdata) # remove the dodgy degree symbol in column names
            writeLines(cdata, curfile)          # write the data back to the file
            ## try to read the file again, if still an error, bail out
            cdata <- try(read.csv(curfile, skip = 16, encoding = "latin1"), silent = TRUE)
            if (inherits(cdata, "try-error")) { # yes, still!, handle read problem
                if (delete) {
                    file.remove(curfile) # remove file if a problem & deleting
                }
                out[[i]] <- URLS[i]    # record failed URL...
                if (isTRUE(verbose)) {
                    setTxtProgressBar(pb, value = i) # update progress bar...
                }
                next                  # bail out of current iteration
            }
        }
 
        ## Must have (eventually) read file OK, add station data
        cdata <- cbind.data.frame(StationID = rep(sites[i], NROW(cdata)),
                                  cdata)
        names(cdata)[-1] <- cnames
        out[[i]] <- cdata
 
        if (isTRUE(verbose)) { # Update the progress bar
            setTxtProgressBar(pb, value = i)
        }
    }
 
    out                                 # return
}
{% endhighlight %}

The main infelicity is that you have to supply the `getData()` with a 
data frame containing the station IDs and start and end years 
respectively for the data you want to collect. This suited my needs as 
we wanted to grab data from 10 stations with different start and end 
years as required to track station movements. It's not as convenient if 
you only want to grab the data for a single station, however.

One thing you'll note quickly if you start downloading data using this 
function is that the web script the Government of Canada is using on 
their climate website will quite happily generate a fully-formed file 
containing no actual data (but with all the headers, hourly time 
stamps, etc) if you ask it for data outside the window of observations 
for a given station. There are no errors, just lots of mostly empty 
files, bar the header and labels.

One other thing to note is that `getData()` returns the downloaded data 
as a list and no attempt is made to flatten the individual components 
to a single large data frame. That's because it allows for any failed 
data downloads (or reads) and records the failed URL instead of the 
data. This gives you a chance to manually check those URLs to see what 
the problem might be before re-running the job, which because we saved 
all the CSVs will run very quickly from that local cache.

To see `getData()` in action, we'll run a quick job, downloading the 
2014 data for two stations

 * Regina INTL A (51441)
 * Indian Head CDA (2925)

First we create a data frame of station information


{% highlight r %}
stations <- data.frame(StationID = c(51441, 2925),
                       start = rep(2014, 2),
                       end = rep(2014, 2))
{% endhighlight %}

Then we pass this to `getData()` with the path to the folder we wish to 
cache downloaded CSVs in


{% highlight r %}
met <- getData(stations, folder = "./csv", verbose = FALSE)
{% endhighlight %}

This will take a few minutes to run, even for just 24 files, as the 
site is not the quickest to respond to requests (or perhaps they are 
now throttling my workstation's IP?). Note I turned off the printing of 
the progress bar here, only because this doesn't play nicely with 
**knitr**'s capturing of the output. In real use, you'll want to leave 
the progress bar on (which it is by default) so you see how long you 
have to wait till the job is done.

Once this has finished, we can quickly determine if there were any 
failures


{% highlight r %}
any(failed <- sapply(met, is.character))
{% endhighlight %}



{% highlight text %}
[1] FALSE
{% endhighlight %}

If any had failed, the `failed` logical vector could be used to index 
into `met` to extract the URLs that encountered problems, e.g.


{% highlight r %}
unlist(met[failed])
{% endhighlight %}

If there were no problems, then the components of `met` can be bound 
into a data frame using `rbind()`


{% highlight r %}
met <- do.call("rbind", met)
{% endhighlight %}

The data now looks like this


{% highlight r %}
head(met)
{% endhighlight %}



{% highlight text %}
  StationID        Date.Time Year Month Day  Time Data.Quality Temp...C.
1     51441 2014-01-01 00:00 2014     1   1 00:00           **     -23.3
2     51441 2014-01-01 01:00 2014     1   1 01:00           **     -23.1
3     51441 2014-01-01 02:00 2014     1   1 02:00           **     -22.8
4     51441 2014-01-01 03:00 2014     1   1 03:00           **     -23.3
5     51441 2014-01-01 04:00 2014     1   1 04:00           **     -24.3
6     51441 2014-01-01 05:00 2014     1   1 05:00           **     -24.3
  Temp.Flag Dew.Point.Temp...C. Dew.Point.Temp.Flag Rel.Hum....
1                         -26.3                              77
2                         -26.1                              77
3                         -25.8                              77
4                         -26.3                              77
5                         -27.1                              78
6                         -27.0                              79
  Rel.Hum.Flag Wind.Dir..10s.deg. Wind.Dir.Flag Wind.Spd..km.h.
1                              13          <NA>              22
2                              12          <NA>              26
3                              12          <NA>              22
4                              13          <NA>              18
5                              13          <NA>              14
6                               9          <NA>               6
  Wind.Spd.Flag Visibility..km. Visibility.Flag Stn.Press..kPa.
1                          19.3            <NA>           95.38
2                          24.1            <NA>           95.38
3                          24.1            <NA>           95.39
4                          24.1            <NA>           95.47
5                          24.1            <NA>           95.56
6                          24.1            <NA>           95.60
  Stn.Press.Flag Hmdx Hmdx.Flag Wind.Chill Wind.Chill.Flag
1                  NA        NA        -35              NA
2                  NA        NA        -36              NA
3                  NA        NA        -35              NA
4                  NA        NA        -34              NA
5                  NA        NA        -34              NA
6                  NA        NA        -30              NA
            Weather
1 Snow,Blowing Snow
2 Snow,Blowing Snow
3 Snow,Blowing Snow
4 Snow,Blowing Snow
5              Snow
6              <NA>
{% endhighlight %}

Yep, a bit of a mess; some post processing is required if you want tidy 
`names` etc. The student was only interested in temperature and 
relative humidity so I dropped all the other met data and data quality 
columns and then only had to update a few variable names. ~~I purposely 
didn't have `getData()` fix this in case the data format on the 
Government of Canada's climate website changes.~~ **Update** I had to 
change this behaviour to allow `getData()` to process some degenerate 
CSV files with odd characters in the column name data and the data 
quality field (see the comments for details). The columns names are 
hardcoded but retain the messy names as given to them by the Government 
of Canada's webmaster. Cleaning up afterwards is advised still.

A final note, I could have run this over all the cores in my 
workstation or even on all the computers in my small computer cluster 
but I didn't, instead choosing to run on a single core overnight to get 
the data we needed. Please be a good netizen if you do use the 
functions I've discussed here as other people will no doubt want to 
access the Government of Canada's website. Don't flood the site with 
requests!

If you have any suggestions for improvements or changes, let me know in 
the comments. The latest versions of the `genURLS()` and `getData()` 
functions can be found in this Github 
[gist](https://gist.github.com/gavinsimpson/8c13e3c5f905fd67cf85).
