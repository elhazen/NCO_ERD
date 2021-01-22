# NCO_ERD
NetCDF code for NCO and Panoply by Mike Jacox 1/21/2021

Links
List of netcdf resources (https://www.unidata.ucar.edu/software/netcdf/software.html)

NCO homepage (http://nco.sourceforge.net/)

NCO user guide (long - http://nco.sourceforge.net/nco.html)

Presentation with some basic NCO examples (http://research.atmos.ucla.edu/csi/GROUP/tips/NCO_basics_N.Berg2013.pdf)

Panoply user guide (https://earth.usc.edu/files/ge-labs/EdGCM/Documentation/Panoply_Manual.pdf)


Installation

Suggested steps for installation on mac (sorry, not sure for PC). 
Use the command line (Terminal app on mac)

# Install developer tools
xcode-select --install

# Set up for homebrew installation
sudo xcode-select -s /Library/Developer/CommandLineTools

# Install homebrew (this is a single line, not split as it shows up in word)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install nco
brew install nco

# Install panoply (netcdf viewer, not necessary for nco)
brew install adoptopenjdk
brew install panoply


NCO General Notes
●	General format of nco commands is: 
command switch(es) input_file(s) output_file
●	Type the name of a command (e.g., ncks) to get information about it and a list of switches


NCO Examples

# Plot metadata
ncdump -h file_in.nc

# Print the values for a single variable (e.g., depth)
ncdump -v variable_name file_in.nc

Example: Print depth values from GLORYS
ncdump -v depth glorys_sample_day.nc

# Extract a single variable to a new file
ncks -v variable_name file_in.nc file_out.nc

Example: Extract temperature (variable name thetao) from a GLORYS file to a new file (glorys_temp.nc)
ncks -v thetao glorys_sample_day.nc glorys_temp.nc

# Extract temperature for a subset region
ncks -F -v thetao -d dimension,min,max,stride -d dimension,min,max,stride file_in.nc file_out.nc 

Example: Extract temperature from GLORYS for northeast Pacific only
ncks -F -v thetao -d longitude,-140.0,-110.0,1 -d latitude,25.0,50.0,1 glorys_sample_day.nc ccs_temp.nc

Example: Extract surface temperature from GLORYS for northeast Pacific only
ncks -F -v thetao -d longitude,-140.0,-110.0,1 -d latitude,25.0,50.0,1 -d depth 1,1,1 glorys_sample_day.nc ccs_sst.nc

●	Including decimals in the dimensions will extract between the specified latitude and longitudes. If no decimals are used, then the subsetting is done by index rather than the actual lat/lon
●	-F specifies that counting starts at 1 instead of 0
●	Stride is the spacing between records being extracted. To extract everything, stride is 1, if one wanted to extract every fifth latitude, for example, stride would be 5.

# Time average across all records in file
ncra -F -d dimension,start,end,stride file_in.nc file_out.nc

Example: Create an annual average from a ROMS file with monthly data for 1990
ncra -F -d ocean_time,1,,1 roms_temp_1990.nc roms_temp_1990_avg.nc

●	Here the “end” argument is empty, so it is assumed to be the last record in the file

# Time average across all records in multiple files
ncra -F -d dimension,start,end,stride file_in1.nc file_in2.nc file_out.nc

Example: Create a 2-year average from ROMS files with monthly data for 1990 and 1991
ncra -F -d ocean_time,1,,1 roms_temp_1990.nc roms_temp_1991.nc roms_temp_1990-1991_avg.nc

Example: Create an average from multiple ROMS files of monthly data with the format roms_temp_yyyy.nc
ncra -F -d ocean_time,1,,1 roms_temp_*.nc roms_temp_avg.nc

●	* is a wildcard, so any file that has this format will be included. * can also be used for directories, e.g., if files of the above format were in multiple subdirectories, one could use */roms_temp_*.nc

Example: As above, but create an average for July only
ncra -F -d ocean_time,7,,12 roms_temp_*.nc roms_temp_july_avg.nc

# Concatenate files (i.e., stitch multiple files together in a single file)
ncrcat file_in1.nc file_in2.nc file_out.nc

Example: Concatenate monthly roms output for 1990-1999 in a single file
ncrcat roms_temp_199[0123456789].nc roms_temp_1990-1999.nc

●	The square brackets above are shorthand to specify multiple input files for 1990, 1991, 1992, etc.

# Area average
ncwa -a dimension1,dimension2 file_in.nc file_out.nc

Example: Calculate the mean monthly temperature over the whole ROMS domain
ncwa -a eta_rho,xi_rho roms_temp_1990-1999.nc roms_temp_areaavg_1990-1999.nc

●	In the ROMS output, eta_rho and xi_rho are the name of the N-S and E-W dimensions for temperature. In other files these dimensions will have different names (e.g., in the GLORYS example they are called longitude and latitude). You can see the dimension names using ncdump

# Calculate difference between files
ncdiff file_in1.nc file_in2.nc file_out.nc

Example: Calculate the difference between 1992 and 1991 in monthly ROMS output
ncdiff roms_temp_1992.nc roms_temp_1991.nc roms_temp_diff_1991_1992.nc

# Rename a variable
ncrename -v old_variable_name,new_variable_name file.nc

Example: For the example where we extracted SST from the 3D GLORYS temperature field, we could rename the variable thetao to sst
ncrename -v thetao,sst ccs_sst.nc

●	Can also use -d to rename a dimension

# Specifying a record dimension
Often a nco command requires that one of the dimensions in the file (usually time) is specified as the “record” dimension. Some files may not have a record dimension, so you can assign it:
ncks --mk_rec_dmn time file_in.nc file_out.nc

# Calling nco from within matlab or R

In matlab: system(‘command’)
In R: system(“command”)

Example: call the earlier variable renaming command from inside matlab:
system(‘ncrename -v thetao,sst ccs_sst.nc’);

●	I think the only difference between matlab and R is the single vs. double quotes. But I have not tried this in R – let me know if there are issues!
●	This is not just for nco, you can run anything on the command line from within matlab/R




