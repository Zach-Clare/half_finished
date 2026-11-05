I'm building out this documentation as I write the code because otherwise I'll have to do it all at the end and that's always awful. This post specifically is talking about the discover_L3.py script which focuses on finding new level 3 files and adding them to the processing queue, as well as creating whichever other data we might need.

## Finding when to start looking from
First, we need to know which files to add to the queue. That's why we begin by looking at the `queue` table itself and seeing when the most recently-added item was added.

We do this with:
`last_ran = models.Queue.select().order_by(models.Queue.added.desc()).first()`

## Getting the L3 files
Then we can use this most recent timestamp to look in the L3 file path and filter files that are newer than this timestamp. This gets all the files that were added since we last tried to discover L3 files.

We scan *all* folders in the given directory newer than the last run time. We look for any file that begins with `"SMILE_SXI_L3_SCIM15-SCI-TOT_"` and open it as a fits file. We look for the `DATE-OBS` and `DATE-END` headers and format them into a datetime variable, removing the microseconds (as the MySQL database doesn't handle microseconds, end we don't really need them anyway). We use these values to query for an observation ID, creating a new observation entry if we don't find one.

*So far, this is where I'm up to. The next bit is my plan.*

Next we create the Level 3 record in the `leve3_files` table. We then add a reference to that Level 3 file into the `queue` table.