---
layout: single
sitemap: true
title: "Running WRF"
excerpt: "This is my personal notebook to help me run WRF <img src='/assets/images/Wrf-workflow.png'>"
collection: portfolio
author_profile: false
classes: wide
---

`This is my personal notebook to help me run WRF`

# Downloading and setting up WRF environment.

I have already done this part, so I'll write a page on this some other day if I need to reinstall it. Meanwhile, here's a much more sophisticated link for [How to compile WRF](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/Compile/index.php) from UCAR's experts.

# Running a simulation in WRF.

<figure style="width: 700px">
  <img src="/assets/images/Wrf-workflow.png" alt="WRF Workflow">
  <figcaption>Workflow of WRF for a typical run. Source: <a href="http://www2.mmm.ucar.edu/wrf/OnLineTutorial/Basics/index.php">WRF Online Tutorial</a></figcaption>
</figure>

Here, I have discusssed Meteorological data first, and then Geographical. However, these are parallel processes and the order doesn't matter.

## Step 1a: Get Gridded Meteorological Data to guide the simulation.

* Choosing the Reanalysis dataset: [List of available GRIB datasets from NCAR](http://www2.mmm.ucar.edu/wrf/users/download/free_data.html).
  * For Paris, I used ERA by ECMWF.
    * [ERA-interim](https://rda.ucar.edu/datasets/ds627.0/) has GRIB files.
    * [ERA5 Reanalysis](https://rda.ucar.edu/datasets/ds630.0/) has NetCDF outputs only.
    * The prescribed V-table called `Vtable.ERA-interim_pl` can be found [here](http://www2.mmm.ucar.edu/wrf/users/vtables/Vtable.ERA-interim_pl).
  * For USA, [NCEP North American Mesoscale (NAM)](https://rda.ucar.edu/datasets/ds609.0/) with 12 km 6 hourly outputs (Vtable: `Vtable.NAM`) might be better. (Will update when I run USA case studies)
  * In Google Earth Engine, I usually use the [NCEP/NCAR Reanalysis dataset](https://rda.ucar.edu/datasets/ds090.0/) to visualize air temperatures. But its resolution is 209 km, not the best for intra-urban applications.
  * Mostly they use [NCEP FNL (Final)](https://rda.ucar.edu/datasets/ds083.2/). Resolution 1 degree.

* [Finding the right ERA-interim Reanalysis files](https://rda.ucar.edu/datasets/ds627.0/#!access) (Sign in required):
  * Last time, I made the mistake of downloading NetCDF formats of ERA5 and ran into an error where it showed 0 available levels of soil depth. However, WRF/ungrib is not yet adapted to handle netCDF files. Be sure to download GRIB files.
  * Use the "Web Server Holding" column to access the whole files, not the "Data Format Conversion" column.
  * **For atmospheric variables:** Use the "[ERA Interim atmospheric model analysis interpolated to pressure levels](https://rda.ucar.edu/datasets/ds627.0/index.html#!cgi-bin/datasets/getWebList?dsnum=627.0&gindex=6)" group for WRF, not the "ERA Interim atmospheric model analysis on model levels" group.
  * **For surface variables:** Use "[ERA Interim atmospheric model analysis for surface](https://rda.ucar.edu/datasets/ds627.0/index.html#cgi-bin/datasets/getWebList?dsnum=627.0&action=customize&disp=&gindex=9)" group.

* Downloading the files.
  * Pick "[Faceted Browse](https://rda.ucar.edu/datasets/ds627.0/index.html#cgi-bin/datasets/getWebList?dsnum=627.0&action=customize&disp=&gindex=6)" and enter the time period of interest. The files are huge (over 100 MB each) so only download the time period needed. Case of Paris: Time period - July 19th to August 10th, 2018.
  * Use download option 2 that generates a Unix script to read them all using wget.
  * Copy the contents and paste them in a new file created in `$RCAC_SCRATCH` using `vi <name_of_script>` where `<name_of_script> : Wget-City-sfc.sh or Wget-City-sfc.sh` (personal convention). Within the file, update the NCAR password.
  * Make the file executable using `chmod 755 <name_of_script>`. Although mine worked regardless.
  * Note that this is a `.csh` script so run it using `csh Wget-City-atm.sh` but save it as `.sh` nonetheless. When I create a `.csh` type file, the text gets pasted with a `#` comment sign in front of every line. Then move the downloaded files to a separate folder and `pwd` the location for next step.

## Step 1b: UNGRIB.

* Translating the ERA-interim GRIB files into intermediate file format the MetGrid will read. Note that it does NOT cut down the data according to the domain specification yet. Execute these steps within the folder `Build_WRF/WPS/`.

* Link the Vtable using `ln -sf ungrib/Variable_Tables/<name_of_Vtable> Vtable`. For exmaple, here `<name_of_Vtable> = Vtable.ERA-interim_pl`.
* Link the location of downloaded data using `./link_grib.csh <path_to_data>`. This should create several links of the format `GRIBFILE.AAA`.

* Edit the `&share` part of [namelist.wps](http://www2.mmm.ucar.edu/wrf/users/namelist_best_prac_wps.html) file. The current run specifications should always be stored as `namelist.wps` (in `Build_WRF/WPS/`). Therefore, backup the original and keep renaming the completed runs.

  * `&share`
    * `start_date` and `end_date`: three times for each domain.
    * `interval_seconds = 21600` (for 6 hourly ERA data).
    * leave `io_form_geogrid = 2` for NetCDF as ungrib will convert our reanalysis data to netcdf.
  * `&ungrib`
    * `prefix`: Leave it at the default option, `FILE`.
* Run `./ungrib.exe` to generate intermediate files in the format of `FILE:YYYY-MM-DD_hh` - one file for each time.
* Given the name of a singe intermediate format file on the command line, the `./util/rd_intermediate.exe` program prints information about the fields contained in the file.

<figure style="width: 400px" class="align-right">
  <img src="/assets/images/WRF-domain.png" alt="WRF">
</figure>


## Step 2a: Get Static Geographical Data.

* Download and save the highest resolution of each field from here - [Geographical Input Data Mandatory Fields Downloads](http://www2.mmm.ucar.edu/wrf/users/download/get_sources_wps_geog.html) - and save it in `$RCAC_SCRATCH`.

* Unlike meteorological data, there is no need to download Geographic data every time because this is just static.

* Use the [R script](/assets/files/WRF_domain.pdf) to visualize and configure domains. Note: It is recommended to have domains no smaller than about 100x100 each. Keep about 10 grid points (minimum of 5) on each side, in the boundary zone. If domains are too small, the solution will be determined by forcing data.

## Step 2b: GEOGRID.

* Edit the `&geogrid` part of namelist.wps file.
  *
  * `&geogrid`
    * Input from R domain designer.
    * `geog_data_res = 'default','default','default',`
    * `dx and dy = 9000`. Resolution of largest domain (in meters for Lambert and Mercator projection. Degrees in Lat-Lon projection).
    *  `map_proj = 'Lambert'` for mid-latitude European countries. Mercator will probably be better for India. [1=Lambert, 2=polar stereographic, 3=mercator, 6=lat-lon]
    * `ref_lat` and `ref_lon` are the center of largest domain. Use `geocode(City)` in R.
    * `truelat1 = ref_lat` required for Lambert, map_proj = 1, 2, 3 (defaults to 0 otherwise). `truelat2` - required for MAP_PROJ = 6 (defaults to 0 otherwise).
    * `stand_lon = ref_lon`: If this longitude is set to the same value as ref_lon, your largest domain will be centered.
    * `geog_data_path`: Location of Geographical dataset `WPS_GEOG` in RCAC_SCRATCH.

* Load ncl -- `module load ncl`. Then run `ncl util/plotgrids_new.ncl` to make sure geogrid is in order.

* Run `./geogrid.exe` to generate intermediate files in the format of `geo_em.dxx.nc` - one file for each domain saved in the WPS folder.
