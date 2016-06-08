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
