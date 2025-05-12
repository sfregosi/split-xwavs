# split-xwavs
Small MATLAB toolset to split duty cycled XWAVs into inidividual raw files for analysis outside Triton (e.g., PAMGuard, Raven)

If HARP data is duty cycled, a single HARP-created XWAV will contain multiple raw files with time gaps between them. The timestamp of the XWAV is only correct for the first few raw files. The correct start time of each raw file is stored in the XWAV header, but that header is not accessible by PAMGuard or other acoustic analysis programs, which rely on timestamps in the filenames to correctly parse the file start time. PAMGuard cannot access this XWAV header info (for now!) so if these data are processed in PAMGuard, PAMGuard will not properly parse the time information and will assume the entire XWAV is continuous from the time specified in the filename. 

This code provides a workaround where each XWAV is split into multiple XWAVs made up of only continuous raw files. For example, if the duty cycle is 5 mins every 30 mins, and each raw file is 75 sec long, it will write a new XWAV made up of the four raw files for the first 5 min duty cycle (timestampled with the start time of the first raw file), then write a new XWAV containing the next four raw files (the next 5 min duty cycle, 30 mins later, timestamped with the start time of the *fifth* raw file) and so on. 

**_I've tried to test this as much as possible with a small example dataset but there may be bugs - I cannot promise that it works perfectly. Please do thorough tests to check that the outputs are what you expect. Please create an issue if you run into a bug!!_**

### Operation

Use the `workflow_splitXwavs.m` script to process a directory of files: 

- Update `path_code` on Line 47 to point to the directory where you cloned/downloaded this repository
- Update `path_xwavs` on Line 50 to point to the directory containing the original XWAVs. Alternatively you can leave this line alone and it will prompt you to select a directory.
- Update `path_split` on Line 58 to specify the output directory where the split files should be saved. If this directory doesn't exist it will be created. 
- Hit Run
- Check the Command Window for status updates. After it identifies all the gaps it will pause and you must hit ENTER to confirm the info is valid to continue

The script will first loop through all the XWAVs and generate a `fileGapSummary.mat` table that summarizes how many gaps were identified in each XWAV. This can be used to double check that the code is identifying gaps in a pattern that you would expect based on your duty cycle settings. Then, it will use that summary table to loop through all files that require splitting, read in the raw data, and write out the newly split files. 


### Some notes

- Unfortunately, this is a very data-storage-heavy process; this will create a fully copy of the original data set so you need to have enough hard drive space to create the copy

- If a single cycle of recording is split across two XWAVs (e.g., 2.5 mins recorded on one XWAV, 2.5 mins on the next as would happen in this example) those will not be merged... that 5 min period will be written as two files because this operates on an XWAV by XWAV basis

- Several functions inlucded are modified versions or based on functions included in [Triton]( https://github.com/MarineBioAcousticsRC/Triton). 
	- `wrxwavhd_split` and `rdxwavhd_sf` are slightly modified versions of those same functions and have been renamed to avoid conflicts
	- `wrSplitXwav` is a modified version of Triton's `decimatexwav` 


