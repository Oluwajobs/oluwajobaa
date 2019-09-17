---
layout: single
sitemap: true
title: "Running WRF"
excerpt: "This is my personal notebook to help me run WRF"
collection: portfolio
author_profile: false
classes: wide
---

`This is my personal notebook to help me run WRF

## Downloading and setting up WRF environment.

I have already done this part (hopefully correctly), so I'll write a page on this some other day if I need to reinstall it. Meanwhile, here's a [How to compile WRF link](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/Compile/index.php) from UCAR's website.

## Running a case study in WRF.

### Step 1: Get Meteorological Reanalysis Data to guide the simulation.

* Choosing the Reanalysis dataset: [List of available GRIB datasets from NCAR](http://www2.mmm.ucar.edu/wrf/users/download/free_data.html).
  * For Paris, I used ERA by ECMWF.
    * [ERA-interim](https://rda.ucar.edu/datasets/ds627.0/) has GRIB files.
    * [ERA5 Reanalysis](https://rda.ucar.edu/datasets/ds630.0/) has NetCDF outputs only.
    * The prescribed V-table called Vtable.ERA-interim_pl can be found [here](http://www2.mmm.ucar.edu/wrf/users/vtables/Vtable.ERA-interim_pl).
  * For USA, [NCEP North American Mesoscale (NAM)](https://rda.ucar.edu/datasets/ds609.0/) with 12 km 6 hourly outputs (Vtable: Vtable.NAM) might be better. (Will update when I run USA case studies)
  * In Google Earth Engine, I usually use the [NCEP/NCAR Reanalysis dataset](https://rda.ucar.edu/datasets/ds090.0/) to visualize air temperatures. But its resolution is 209 km, not the best for intra-urban applications.
  * Mostly they use [NCEP FNL (Final)](https://rda.ucar.edu/datasets/ds083.2/). Resolution 1 degree.

* [Finding the right ERA-interim Reanalysis files](https://rda.ucar.edu/datasets/ds627.0/#!access) (Sign in required): Courtesy [Dave Stepniak], UCAR
  * Last time, I made the mistake of downloading NetCDF formats of ERA5 and ran into an error where it showed 0 available levels of soil depth. However, WRF/ungrib is not yet adapted to handle netCDF files. Be sure to download GRIB files.
  * Use the "Web Server Holding" column to access the whole files, not the "Data Format Conversion" column.
  * **For atmospheric variables:** Use the "[ERA Interim atmospheric model analysis interpolated to pressure levels](https://rda.ucar.edu/datasets/ds627.0/index.html#!cgi-bin/datasets/getWebList?dsnum=627.0&gindex=6)" group for WRF, not the "ERA Interim atmospheric model analysis on model levels" group.
  * **For surface variables:** Use "[ERA Interim atmospheric model analysis for surface](https://rda.ucar.edu/datasets/ds627.0/index.html#cgi-bin/datasets/getWebList?dsnum=627.0&action=customize&disp=&gindex=9)" group.`

* Downloading the files.
  * Pick "[Faceted Browse](https://rda.ucar.edu/datasets/ds627.0/index.html#cgi-bin/datasets/getWebList?dsnum=627.0&action=customize&disp=&gindex=6)" and enter the time period of interest. The files are huge (over 100 MB each) so only download the time period needed. Case of Paris: Time period - July 19th to August 10th, 2018.
  * Select none of the variables (which basically means select all) and all the files identified within the time period.
  * Use download option 2 that generates a Unix script to read them all using wget.
  * Copy the contents and paste them in a new file created in RCAC_SCRATCH using `vi <name_of_script>` where `<name_of_script> : Wget-City-sfc.sh or Wget-City-sfc.sh` (personal convention). Within the file, update the NCAR password.
  * Make the file executable using `chmod 755 <name_of_script>`. Although mine worked regardless.
  * Note that this is a `.csh` script so run it using `csh Wget-City-atm.sh` but save it as `.sh` nonetheless. If I create a `.csh` type file, the text gets pasted with a `#` comment sign in front of every line.
  * Then move the downloaded files to a separate folder.

### Step 2: Get Geographical inputs. Domain set up.
