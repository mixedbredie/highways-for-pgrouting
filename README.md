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
