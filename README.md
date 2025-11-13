# HARC
data project

placeholder text; 

Some pseudocode-esque logic and/or context

Disadvantaged, as defined by the Climate and Economic Justice Map thingy along with SVI and ADI? Maybe? is including the SVI and ADI measurements necessary? Eh yes and no? bc the C&E justice map uses Census Tracts, whereas SVI and ADI uses Census Blocks. And Census Blocks are a little more pain in the ass to deal with.

I guess what we display in terms of SDOH / HRSN / health trends things or whatever, we display them to client iff the "yes" threshold is met

pseudocode for Census Tract address matching thing:

Get addresses from whichever source (idk what to do if only ZIP code is available, but idk if that's a problem we want to deal with. also idk where specifically the address will come from and will go. figure that out later i guess). Then geo-code address (link: https://github.com/tuva-health/geo-coding). Create table for the raw geocoded data, read it back (in JSON format?), then convert it to a table with defined columns. Take geocoded patient data and join it with defined Census Tract dataset with geospacial queries. 

Speaking of the geospacial queries, it looks like in order to create the geospacial tables (in order to enable the join type that I'm currently unsure of), depending on where the table is coming from, might have to create some kind of function in order to be able to load the geocoded patient data into the table. However the geospacial table is created and properly loaded, once that's done, create a stage (if necessary, or not) or do whatever in order to be able to define a path for the 2010 Census Tract shapefiles (taken from the Climate and Economic Justice Tool) to be uploaded into. Once that's done, create a table by loading the Census Tract shapefiles. Then write a query to "find the polygon that contains my geocoded point." Query from Tuva Project is below (it's assuming I'll be getting the patient data from Snowflake though):

create or replace table tuva.geocoded.patient_tracts as (
select a.patient_id::varchar patient_id, b.PROPERTIES:GEOID::STRING FIPS
from tuva.geocoded.geocoded_patients a
join tuva.reference.census_tracts b
on st_contains(GEOGRAPHY, st_point(a.longitude, a.latitude))
);

The "on st_contains..." is the part that says "find the polygon that contains my geocoded point."

Once the address is assigned to a Census Tract, do a right join with the "2.0-communities" table--the table from Climate and Economic Justice Tool that has all the Census Tract-based geocoded SDOH information. Then in a separate section (which will also be displayed), join that with the different HRSN/SDOH categories (the main 5 that have been defined by CMS and referenced in other studies for now). Idk if the zCode section would be from a join with the matched main SDOH categories (if the subcategorization is a clean one) or if it would be from the Census-Tract-geocoded address to the 2.0-communities table right join one but--however it goes, join this with the zCode table. Might have to do conditional joins for this though, since iirc neighborhood disadvantage has multiple conditions that need to be met, and also--zCodes do have vague 2/2 based codes.

If we want to add current insights, which would have to be from the 2020 Census, we would likely have to geocode the address, get the shp files or whatever that would help faciliate the geocoded insights (i.e., proximity to data centers). Regional maps of climate and environmental disasters / risks and so on can likely be defined by a polygon for the geocoded point to be matched with. 
