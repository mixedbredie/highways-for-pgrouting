# highways-for-pgrouting
Building OS Highways for pgRouting

1. Load the data

If you have Safe Software’s FME you can download the workbenches to create the database tables here: https://github.com/OrdnanceSurvey/OSMM-Highways-Network-Support but note that you will have to edit the workbenches to write to PostGIS instead of GeoPackage.  Not hard if you’ve used FME before.

The following tables are created by FME process:

- hw_accessrestriction*
- hw_accessrestriction_inclusion
- hw_accessrestriction_exemption
- hw_accessrestriction_networkref*
- hw_restrictionforvehicles
- hw_restrictionsforvehicles_networkref
- hw_road
- hw_road_networkref
- hw_roadlink*
- hw_roadlink_formspartof
- hw_roadlink_relatedroadarea
- hw_roadnode*
- hw_roadnode_relatedroadarea
- hw_turnrestriction*
- hw_turnrestriction_networkref*

\* Tables used in this exercise

2. Add the fields required for pgRouting

        ALTER TABLE my_schema.hw_roadlink
          ADD COLUMN source integer,
          ADD COLUMN target integer,
          ADD COLUMN one_way varchar(2),
          ADD COLUMN speed_km integer,
          ADD COLUMN cost_len double precision,
          ADD COLUMN rcost_len double precision,
          ADD COLUMN cost_time double precision,
          ADD COLUMN rcost_time double precision,
          ADD COLUMN x1 double precision,
          ADD COLUMN y1 double precision,
          ADD COLUMN x2 double precision,
          ADD COLUMN y2 double precision,
          ADD COLUMN to_cost double precision,
          ADD COLUMN rule text,
          ADD COLUMN isolated integer;

3. Create indexes for speed

        CREATE INDEX hw_roadlink_source_idx ON my_schema.hw_roadlink USING btree(source);
        CREATE INDEX hw_roadlink_target_idx ON my_schema.hw_roadlink USING btree(target);

4. Update new fields with values required for pgRouting

The start and end coordinates of the links are used in the Astar shortest path algorithmto determine the links closest to the target shortest path solution.

        UPDATE my_schema.hw_roadlink 
          SET x1 = st_x(st_startpoint(centrelinegeometry)),
            y1 = st_y(st_startpoint(centrelinegeometry)),
            x2 = st_x(st_endpoint(centrelinegeometry)),
            y2 = st_y(st_endpoint(centrelinegeometry));

Setting the one_way flags helps pgRouting analyse the road network graph for errors.

        UPDATE hw_roadlink SET one_way = 'B' WHERE directionality = 'bothDirections';
        UPDATE hw_roadlink SET one_way = 'TF' WHERE directionality = 'inOppositeDirection';
        UPDATE hw_roadlink SET one_way = 'FT' WHERE directionality = 'inDirection';

Costs are what the routing engine uses to calculate the best route across the network.  In this example we are using **distance**, based on link length, and **time**, based on average speed for particular road classification and link length.  For links with one way directionality we set the reverse cost very high to discourage use.  pgRouting uses the digitised direction of the line to help with routing and the Highways layer helpfully has a field which tells us, for one way streets, which way the line has been drawn and which way the traffic is expected to flow.  For one way streets with traffic flow in the same direction as digitised direction the directionality flag is set to **inDirection**. For traffic flow contra to line direction it is set to **inOppositeDirection**. For links with two way flow of traffic it is set to **bothDirections**.

Forward and reverse cost is the same for two way streets

        UPDATE my_schema.hw_roadlink
          SET cost_len = ST_Length(centrelinegeometry),
              rcost_len = ST_Length(centrelinegeometry)
          WHERE directionality IN ('bothDirections');

Reverse costs are increased for one way streets

        UPDATE my_schema.hw_roadlink
          SET cost_len = ST_Length(centrelinegeometry),
              rcost_len = ST_Length(centrelinegeometry)*100
          WHERE directionality IN ('inDirection');
        
        UPDATE my_schema.hw_roadlink
          SET rcost_len = ST_Length(centrelinegeometry),
              cost_len = ST_Length(centrelinegeometry)*100
          WHERE directionality IN ('inOppositeDirection');
          
To set the time cost I set an average speed based on the road classifiation and form of the road.  A tip here is to remember to set the speed for slip roads, traffic island links to be the same as the speed for the carriageway of the same class.  This prevents odd routing through complex junctions.

Check what type of roads you are using with:

        SELECT DISTINCT roadclassification, formofway
          FROM my_schema.hw_roadlink
          ORDER BY roadclassification;
          
