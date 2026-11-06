I'm building out this documentation as I write the code because otherwise I'll have to do it all at the end and that's always awful. This post specifically is talking about the discover_L3.py script which focuses on finding new level 3 files and adding them to the processing queue, as well as creating whichever other data we might need.

## Finding when to start looking from
First, we need to know which files to add to the queue. That's why we begin by looking at the `queue` table itself and seeing when the most recently-added item was added.

We do this with:
`last_ran = models.Queue.select().order_by(models.Queue.added.desc()).first()`

## Getting the L3 files
Then we can use this most recent timestamp to look in the L3 file path and filter files that are newer than this timestamp. This gets all the files that were added since we last tried to discover L3 files.

We scan *all* folders in the given directory newer than the last run time. We look for any file that begins with `"SMILE_SXI_L3_SCIM15-SCI-TOT_"` and open it as a fits file. We look for the `DATE-OBS` and `DATE-END` headers and format them into a datetime variable, removing the microseconds (as the MySQL database doesn't handle microseconds, end we don't really need them anyway). We use these values to query for an observation ID, creating a new observation entry if we don't find one.

## Creating a record of that particular Level 3 file
Next we create the Level 3 record in the `leve3_files` table. This is so that we can track different versions of a Level 3 file that pertain to the same observation. This means that we can automatically reprocess reprocessed Level 3 files. Let's say there was an issue earlier in the chain and a certain orbit needs to reprocess from Level 2 upwards. This will automatically run in the Level 4 pipeline due to the following logic.

Example 1:
1. A new observation (like from a new orbit) creates a new level 3 file
2. This Level 3 file is newer than when we last ran the pipeline, so it gets picked up
3. The observation start and end is unrecognised, so a new observation ID is created and linked
4. A new record of the Level 3 file is created.

Example 2:
1. There was an issue with the Level 2 process and files were reprocessed, which means a new Level 3 file is added to the path.
2. This Level 3 file is newer than when we last ran the pipeline, so it gets picked up
3. The observation start and end is recognised, so it gets attached to the old observation record
4. A new record of this Level 3 file is created.

In both of these examples, a new level 3 file record is created, which is exactly what we want. However, let's say we change how we process the Level 4 files. We want to add an old selection of Level 3 files to the queue, but we don't want to create new Level 3 records. How is this done? 

Great question. Let me have a think for a moment.

![[Pasted image 20261106170833.png]]

We don't claim ownership of the Level 3 folder. It would be wrong to have duplicates in that folder *just* so we can reprocess Level 4 files. 

Therefore, what if we added IDs of Level 3 file records directly to the queue table using a separate script? This script could take a list of IDs (or an observation date range) and keep everything entirely self-contained. It should be a fairly short script as well, although date range functionality might make it a bit more complex (since there might be multiple L3 files for any single observation)

That leads us to the conclusion that this script deals with brand new, never-before-seen Level 3 files. Which is extremely appropriate given the name of the script - *discover_L3.py*.

One caveat here is that there is a very slim path to having multiple Level 3 records that pertain to a single Level 3 file. That is if an old Level 3 file gets moved or accidentally deleted and then re-added to the folder. The ideal approach would be attaching onto some unique identifier in the Level 3 headers, but there isn't really one. The closest we have is the `DATE` field. For now, let's use this as the Level 3 file ID by adding it to the database schema and checking it just before we add it as a new Level 3 file in this script, like we do with the observation record. If we find a Level 3 file we've already processed but is still somehow new to the Level 3 folder, let's put it in the log - that feels like something we should at least be aware of for now. We can review it later.

We then add a reference to that Level 3 file into the `queue` table.