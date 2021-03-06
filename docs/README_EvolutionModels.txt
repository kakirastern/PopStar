Matt Hosek
11/11/15

Casey Lam
11/22/18 (Updated MIST section)

=========================
Models Used
=========================
The default stellar evolution models used in the synthetic isochrones
are as follows:

For logAge < 7.4:
Baraffe et al. 2015 models from 0.08 - 0.4 M_sun
Transition region between 0.4 - 0.5 M_sun between Baraffe+15 and Pisa models
Pisa models (Tognelli et al. 2011) from 0.5 M_sun to the highest mass
available at that age (typically 5-7 M_sun), and then switching to
Geneva models (Ekstrom et al. 2012) for the rest of the way.

For logAge > 7.4:
Use the Parsec v1.2s (Bressan et al. 2012) models throughout entire
mass range


========================
Parsec version 1.2s
========================

Paper references: Bressan et al. 2012, Chen et al. 2014
Isochrones downloaded from website: http://stev.oapd.inaf.it/cgi-bin/cmd
	-parameters used: 
	n_Reimers parameter (mass loss on RGB) = 0.2
	photometric system: HST/WFC3 IR channel
	bolometric corrections OBC from Girardi+08, based on ATLAS9 ODFNEW models
	Carbon star bolometric corrections from Aringer+09
	no dust
	no extinction
	Chabrier+01 mass function
	constant metallicity: Z = 0.015, 0.04, 0.005 (+/- 0.5 dex from Z_solar)
		-no alpha enhancement
	time = log t (5.9 - 10.0), steps of delta_logt = 0.01

NOTE: Only the solar metallicity isochrones (Z = 0.015) have been processed for use as of yet

***Code procedure:

1) popstar/evolution.py --> class ParsecStellarEvolution, function format_isochrone
	-takes output*.dat file downloaded from web and organize into popstar format. Creates separate *.dat file for each age
	-can handle multiple metallicities at once, if desired

NOTE: no need to interpolate Parsec models ourselves since we can download whatever time resolution we want from the website

***File Locations:

-solar-metallicity isochrones: /g/lu/models/evolution/ParsecV1.2s/iso/z015
-sub-solar metallicity isochrones (not interpolated yet): /g/lu/models/evolution/ParsecV1.2s/iso/z005
-super-solar metallicity isochrones (not interpolated yet): /g/lu/models/evolution/ParsecV1.2s/iso/z04

-evolutionary tracks in /g/lu/models/evolution/ParsecV1.2s/tracks are not used

=================================
Geneva Models: Esktrom+12
=================================

paper reference: Ekstrom et al. 2012
Isochrones downloaded from web at http://obswww.unige.ch/Recherche/evoldb/index/Isochrone/
	-large grid (0.08 - 500 M_sun, solar metallicity (Z = 0.14, solar [alpha/Fe]), 2 rotation rates (0 and 0.4 v_crit)
	-note: website will let you download whatever time steps you want, but you are limited in the number of isochrones you can get per request. So, I had to download the 	isochrones in chunks and piece them together later

***Code Procedure

0) SED voodoo magic
	-Need to reformat the downloaded Isochrone*.dat files so they can be read by python (specifically, header lines need to be commented out). This is done using the 	following SED command:
		> sed 's/Isochrone/#Isochrone/' <oldfile> > <newfile>

1) popstar/evolution.py --> class EkstromStellarEvolution, function = create_iso
	-takes the Isochrones*.dat files downloaded online and puts them into a master iso.dat file with the proper format
		-will create tmp_*.dat files, one for each Isochrones*.dat file, which are easy to combine by hand in aquamacs

2) popstar/evolution.py --> class EkstromStellarEvolution, function = format_isochrones
	-parse the iso.dat file from create_iso function, create individual isochrone file for each given age
	
***File Locations

-Original downloaded files from website, plus tmp*.dat files after SED processed, plus master iso.dat file: /g/lu/models/evolution/Ekstrom2012/iso/z014

-rotating models: /g/lu/models/evolution/Ekstrom2012/iso/z014/rot
-non-rotating models: /g/lu/models/evolution/Ekstrom2012/iso/z014/norot

-evolutionary tracks in /g/lu/models/evolution/Ekstrom2012/ not used

============================
Pisa Models, 2011 
============================

Paper reference: Tognelli et al. 2011
Isochrones downloaded from website: http://astro.df.unipi.it/stellar-models/index.php?m=1
	-downloaded 3 metallicities: z=0.005, z = 0.015 (solar), z = 0.03
	-Y = middle value of 3 provided (changes for different metallicities)
	-mixing length = 1.68
	-Deuterium fraction: 2*10^-5 (though  4*10^-4 for 0.005)
	-time: 1- 100 Myr in steps of 1 Myr

NOTE: Only solar metallicity isochrones have been processed up to this point

***Code procedure:

1) popstar/evolution.py,  class PisaEvolutionaryModels, function = format_isochrones
	-rename isochrones downloaded from Pisa site to fit popstar iso*.dat scheme
	-can process multiple metallicities in same loop

2) popstar/evolution.py,  class PisaEvolutionaryModels, function = make_isochrone_grid
	-creates interpolated grid with the user-defined time sampling. Builds off of the online grid downloaded
		-hard-coded to create grid with delta-logAge = 0.01
		-only supports up to logAge = 8.0 (since this is the max age of the online grid)
		-can handle different metallicities if desired
	-this is a wrapper function which calls popstar/evolution.py ---> make_isochrone_pisa_interp
		-this in turn calls evolution.py --> get_orig_pisa_isochrones
	
	-paths are hard-coded, so don't need to be in any particular working directory here. Puts interpolated models in /g/lu/models/evolution/Pisa2011/iso/<metallicity>

NOTE: By design, this process interpolates over isochrones, rather than evolutionary tracks. Testing showed that interpolation over the isochrones was more accurate than interpolating over the isochrones

***File Locations:

-Original downloaded isochrones (needed for interpolation, DO NOT DELETE): /g/lu/models/evolution/Pisa2011/iso/
	-metallicity subdirectory: z015

-Interpolated solar metallicity isochrones: /g/lu/models/evolution/Pisa2011/iso/z015

-Non-interpolated non-solar metallicity isochrones: /g/lu/models/evolution/Pisa2011/iso/z005, z03

========================
STERN Models
========================

Paper reference: Brott et al. 2011
Downloaded via ftp provided in paper
	-1 - 35 Myr, steps of 0.2 Myr
	-MW metallicity, 5 - 60 M_sun


***Code Procedure:
None yet

***File Location:

Downloaded files: /g/lu/models/evolution/STERN2011

=================================
Baraffe+15 Models
=================================
Paper reference: Baraffe et al. 2015
Isochrones are solar metallicity (Asplund et al. 2009), mixing length
= 1.6 * H_p, where H_p is pressure scale height

Tracks downloaded from
http://perso.ens-lyon.fr/isabelle.baraffe/BHAC15dir/BHAC15_tracks
	0.07 M_sun - 1.4 M_sun
	5.7 < log t < 10

***Code procedure:

0) SED voodoo magic
	-Need to reformat the downloaded tracks.dat fils so they
	can be read by python (specifically, "!" must be replaced by "#"). This is done using the following SED command:
		> sed 's/\!/#/' <oldfile> > <newfile>

1) popstar/evolution.py. class Baraffe15 --> tracks_to_isochrones
   	- Generate isochrones from evolutionary tracks. Resamples at 
	desired ages (6.0 < logAge 8.0, steps of 0.01)

	-additional function to test age-interpolated isochrone:
	class Baraffe15 --> test_age_interp

***File Locations:
-Original evolutionary tracks downloaded to
/g/lu/models/evolution/Baraffe15

-Isochrones stored in /g/lu/models/evolution/Baraffe15/iso

=====================================
Mesa Isochrone and Stellar Tracks (MIST), Choi+16
=====================================
Paper reference: Choi+16
-Downloaded via web interpolator: http://waps.cfa.harvard.edu/MIST/interp_isos.html
-Apslund+09 solar abundances
-photometric system: HST WFC3/UVIS+IR


***Code procedure:

1) popstar/evolution.py --> class MISTv1, function format_isochrone
   -takes MIST_iso* files downloaded from web and organize into popstar format. Creates separate *.dat file for each age
   -can handle multiple metallicities at once, if desired

Note: No need to interpolate MIST isochrones since we can download them at any time step we want

***File Locations:
Original files are located in appropiate metallicity subdirectory under /g/lu/models/evolution/MISTv1

***Gory details on how to download isochrones below -CYL

1) Go to MIST isochrone page [http://waps.cfa.harvard.edu/MIST/interp_isos.html]
   Note: We have been downloading isochrones with the following convention:
         Version: 1.2
         Rotation: Initial v/v_crit = 0.4
         Age: Log10 Scale with spacing of 0.01
         Composition: Fe/H = 0
         Output Option: Theoretical, Full (79 columns)
    There's a limit on how many isochrones you can download at a time, so break up downloads into smaller chunks if necessary. 
    
2) Move the downloaded .iso file into the directory where you want to unpack the isochrone file.
   Note: For our current purposes this is /g/lu/models/evolution/MISTv1/v1.2/iso/z015. 
   But you should put it into the directory of the relevant version/metallicity.

3) Unpack the isochrone with format_isochrones function. An example of how to call in iPython:
   ```
   from popstar import evolution
   import numpy as np
   evo = evolution.MISTv1()
   input_iso_dir = /g/lu/models/evolution/MISTv1/v1.2/iso
   metallicity_list = np.array(['z015'])
   evo.format_isochrones(input_iso_dir, metallicity_list)
   ```
