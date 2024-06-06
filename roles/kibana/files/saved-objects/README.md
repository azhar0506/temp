# Elasticsearch Saved Objects

This is an attempt to have a centralised space where we can store all Elasticsearch Saved Objects (visualisations,dashboards etc) 
so we can deploy those with Ansible to keep Elasticsearch clusters in-sync


## What counts as **Saved Object**?
1. APM Indices
2. Advanced Settings
3. Canvas Element
4. Canvas Workpad
5. Cases
6. Connector
7. Dashboards
8. Data View
9. Graph Workspace
10. Graph Workspace
11. Infrastructure UI Source
12. Inventory View
13. Lens
14. Map
15. Metrics Explorer View
16. Osquery Pack
17. Osquery Saved Query
18. Query
19. Rule
20. Search
21. Tags
22. URL
23. Uptime Dynamic Settings
24. Visualisation


## How it all should work (ideally)
1. If you are creating/updating **dashboard**:
    1. Create/update Dashboard
    2. Ensure all good and working
    3. Save dashboard adding description & selecting correct `tag`, so it is easy to understand what this is used for
    4. If this dashboard is important and should be preserved:
        1. Go to `Stack Management/Saved objects`
        2. Find the dashboard you created
        3. Select that dashboard, click `Export` & select `Include related objects`
        4. Once downloaded, change file name to the name of the dashboard
        5. Git push it into this project, choosing the space where this dashboard belongs
2. If you are creating/updating any other Saved Object
    1. Create/update object, ie visualisation
    2. Ensure all good and working
    3. Save object adding description & selecting correct `tag`, so it is easy to understand what this is used for
    4. If this object is important and should be preserved:
        1. Go to `Stack Management/Saved objects`
        2. Find the object you created
        3. Select that object, click `Export` & unselect `Include related objects`
        4. Once downloaded, change file name to the name of the object
        5. Git push it into this project, choosing the space where this object belongs



## Structure
```
> ops-space/ - This is space that would be used by OPS from RADIUS Team
>> advanced-settings/
>> dashboards/
>> index-patterns/
>> kibana-tags/
>> visualisations/

> radius-space  - This is space that would be used by Non-OPS from RADIUS Team
>> advanced-settings/
>> dashboards/
>> index-patterns/
>> kibana-tags/
>> visualisations/
```

If there is another Kibana Space that exists and should have specific Saved Objects then create folder names same as Kibana Space ID at the same level as `ops-space` & `radius-space` with the same folders inside, for example
```
We have a Kibana Space with ID - operational-space

So, we create new folder with the name `operational-space` and the same folder layout as in `default` & `shared`, thus we get 

> ops-space/
>> advanced-settings/
>> dashboards/
>> index-patterns/
>> kibana-tags/
>> visualisations/

> radius-space/
>> advanced-settings/
>> dashboards/
>> index-patterns/
>> kibana-tags/
>> visualisations/

> operational-space/
>> advanced-settings/
>> dashboards/
>> index-patterns/
>> kibana-tags/
>> visualisations/
```
