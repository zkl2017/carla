# Add a new map

Users can create their own maps, and run CARLA using these. The creation of the map object is quite independent from CARLA. Nonetheless, the process to ingest it has been refined to be automatic. Thus, the new map can be used in CARLA almost out-of-the-box.  

*   [__Introduction__](#introduction)  
*   [__Create a map with RoadRunner__](#map-creation-with-roadrunner)  
	*   [Export from RoadRunner](#export-from-roadrunner)  
* [__Prepare the files__](#prepare-the-files)  
	*   [Create the folder structure](#create-the-folder-structure)  
	*   [Create the JSON description](#create-the-json-description)  
*   [__Map ingestion__](#import-the-map)  
	*   [Via Docker](#via-docker)  
	*   [Via terminal](#via-terminal)  
*   [Final tips](#final-tips)  
	*   [Modify pedestrian navigation](#modify-pedestrian-navigation)  
*   [__Previous ways to import a map__](#previous-ways-to-import-a-map)  

---
## Introduction

RoadRunner is the recommended software to create a map due to its simplicity. Some basic steps on how to do it are provided in [the next section](#create-a-map-with-roadrunner). The resulting map should consist of at least two elements.  

* __`.fbx` binaries.__ The meshes needed to build the map, such as roads, lanemarkings, sidewalks, ect.
* __`.xodr` OpenDRIVE file.__ Contains the road network information necessary for vehicles to navigate the map.

These files must be prepared to be imported. The files of the map have to be stored in a certain [folder structure](#create-the-folder-structure), with a [`.json` file](#create-the-json-description) describing them.  

The process of the map ingestion has been simplified to minimize the users' intervention. For said reason, there are certains steps have been automatized.  

*   __Traffic signs are created when running the simulator.__ It will generate the traffic lights, stops, and yields according to their `.xodr` definition. The rest of landmarks present in the road map will not be physically on scene, but they can be queried using the API.  
*   __Pedestrian navigation will be generated automatically.__ The user can modify it if desired, but a `.bin` file describing the basic navigation of the map will be created during the process. 

There are two different ways to import the map into CARLA. Both grant the previously mentioned automatization, but there are some differences between them.  

*   [__Via Docker.__](#via-docker) The output will be a distribution package containing the map ready to be used. However, as the map is packaged, pedestrian navigation cannot be modified. 
*   [__Via terminal.__](#via-terminal) This will import the map into CARLA. The pedestrian navigation can be modified if needed. The map can be distributed using a standalone package. Take a look [this tutorial](tuto_A_create_standalone.md) to learn how to create it. 

After that, it is almost ready. There are some [final tips](#final-tips) to make sure that everything fits perfectly. 

There are other ways to import a map into CARLA, which are now deprecated. They require the user to manually set the map ready. Nonetheless, as they may be useful for specific cases when the user wants to customize a specific setting, they are listed in the [last section](#previous-ways-to-import-a-map) of this tutorial.  

---
## Create a map with RoadRunner

RoadRunner is an accessible and powerful software from Vector Zero to create 3D scenes. There is a trial version available at their [site](https://www.vectorzero.io/), and an [installation guide][rr_docs].

[rr_docs]: https://tracetransit.atlassian.net/wiki/spaces/VS/pages/740622413/Getting+Started

The process is quite straightforward, but there are some things to take into account.  

*   __Center the map__ in (0,0).  
*   __Create the map definition.__ Take a look at the [official tutorials](https://www.youtube.com/channel/UCAIXf4TT8zFbzcFdozuFEDg/playlists).  
*   __Check the map validation.__ Take a close look at all connections and geometries.  

![CheckGeometry](img/check_geometry.jpg)

Once the map is ready, click on the `OpenDRIVE Preview Tool` button to visualize the OpenDRIVE road network. Give one last check to everything. Once the map is exported, it cannot be modified. 

![checkopen](img/check_open.jpg)

!!! note
    _OpenDrive Preview Tool_ makes it easier to test the integrity of the map. If there is any error with junctions, click on `Maneuver Tool`, and `Rebuild Maneuver Roads`.
  
### Export from RoadRunner

__1. Export the scene using the CARLA option.__ `File/Export/CARLA(.fbx+.xml+.xodr)`

__2. Leave `Export individual Tiles` unchecked.__ This will generate only one _.fbx_ with all the pieces. It makes easier to keep track of the map.  

__3. Click `Export`.__  

This will generate a `mapname.fbx` and `mapname.xodr` files within others. There is more detailed information about how to export to CARLA in [VectorZero's documentation][exportlink].

[exportlink]: https://tracetransit.atlassian.net/wiki/spaces/VS/pages/752779356/Exporting+to+CARLA

!!! Warning
    Make sure that the .xodr and the .fbx files have the same name.  

---
## Prepare the package

### Create the folder structure

__1. Create a folder inside `carla/Import`.__ The name of the folder is not relevant.  

__2. Create different subfolders__ for each map to import.

__3. Move the files of each map to the corresponding subfolder.__ A subfolder will contain a specific set of elements.  

*   The mesh of the map in a `.fbx`.  
*   The OpenDRIVE definition in a `.xodr`.  
*   Optionally, the textures required by the asset.  


For instance, an `Import` folder with one package containing two maps should have a structure similar to the one below.

```sh
Import
│
└── Package01
  ├── Package01.json
  ├── Map01
  │   ├── Asphalt1_Diff.png
  │   ├── Asphalt1_Norm.png
  │   ├── Asphalt1_Spec.png
  │   ├── Grass1_Diff.png
  │   ├── Grass1_Norm.png
  │   ├── Grass1_Spec.png
  │   ├── LaneMarking1_Diff.png
  │   ├── LaneMarking1_Norm.png
  │   ├── LaneMarking1_Spec.png
  │   ├── Map01.fbx
  │   └── Map01.xodr
  └── Map02
      └── Map02.fbx
```

### Create the JSON description

Create a `.json` file in the root folder of the package. Name the file after the package. Note that this will be the distribution name. The content of the file will describe a JSON array of __maps__ and __props__ with basic information for each of them.  

__Maps__ need the following parameters.  

* __name__ of the map. This must be the same as the `.fbx` and `.xodr` files.  
* __source__ path to the `.fbx`.  
* __use_carla_materials__. If __True__, the map will use CARLA materials. Otherwise, it will use RoadRunner materials.  
* __xodr__ Path to the `.xodr`.  

__Props__ are not part of this tutorial. The field will be left empty. There is another tutorial on how to [add new props](tuto_A_add_props.md).  

In the end, the `.json` should look similar to the one below.

```json
{
  "maps": [
    {
      "name": "Map01",
      "source": "./Map01/Map01.fbx",
      "use_carla_materials": true,
      "xodr": "./Map01/Map01.xodr"
    },
    {
      "name": "Map02",
      "source": "./Map02/Map02.fbx",
      "use_carla_materials": false,
      "xodr": "./Map02/Map02.xodr"
    }
  ],
  "props": [
  ]
}
```
---
## Import into CARLA

!!! Warning
    Packages with the same name will produce an error.  

### Via Docker

This option will run a Docker image of Unreal Engine to import the package, and export it as a distribution package. The Docker image takes 4h and 400GB to be built. However, this is only needed the first time.

__1. Build a Docker image of Unreal Engine.__ Follow [these instructions](https://github.com/carla-simulator/carla/tree/master/Util/Docker). 

__2. Run the script to cook the map.__ In the folder `~/carla/Util/Docker` there is a script that connects with the Docker image previously created, and makes the ingestion automatically. It only needs the path for the input and output files.  

```sh
python docker_tools.py --input ~/path_to_package --output ~/path_for_output_assets
```

__3. Extract the output package__. The Docker should have generated a `.tar.gz` in the output path. This is the distribution package for the assets. Extract it in the main root CARLA folder.  

!!! Note
    There is an alternative on Linux. Move the package to the `Import` folder and run the script `Util/ImportAssets.sh` to extract the package.

### Via terminal 

This option will read the JSON file, and place the assets inside the `Content` in Unreal Engine. Furthermore, it will create a `Package1.Package.json` file inside the package's
`Config` folder. This file describes the content, and is especially useful for props.  

When everything is ready, run the command. 

```sh
make import
```

---
## Final tips

Now the map is ready to be used in CARLA. However, there are some additional settings to be done. These are recommended steps, but mostly optional besides pedestrian navigation.  

*   __Test traffic light timing.__ The traffic lights are generated when the simulator runs. Their state changes using a default time that may be inaccurate for the new map. If there is need to change it, the custom time will have to be set on every run using the API.  
<br>
*   __Place vehicle spawn points.__ These will be used in scripts such as `spawn_npc.py`. They have to be created manually. Place the spawning point 2 to 3 meters above the ground, and a Route Planner's trigger box below it. Orient both in the same direction. When the vehicle falls into the trigger box, the autopilot will be enabled, and the vehicle will be registered to a Traffic Manager.

  ![ue_vehicle_spawnpoint](img/ue_vehicle_spawnpoint.png)

*   __Add the map to the Unreal packaging system.__ This will include the map along with the rest if a CARLA package is created.  
`Edit > Project Settings > Project > Packaging > Show Advanced > List of maps to include...` <br>

  ![ue_maps_to_include](img/ue_maps_to_include.png)

### Modify pedestrian navigation

The pedestrian navigation is managed using a `.bin` file. This is generated when the map is imported. However, this navigation is made using the road definition, and it should be modified.  

*   __Crosswalk meshes are not created during the import.__ Crosswalks defined inside the `.xodr` remain in the logic of the map, but are not visible. For each of them, create a plane mesh that extends a bit over both sidewalks connected. __Place it overlapping the ground, and disable its physics and rendering__. 

!!! Note
    To generate new crosswalks, change the name of the mesh to `Road_Crosswalk`. Avoid doing so if the crosswalk is in the `.xodr`. Otherwise, it will be duplicated. 

![ue_crosswalks](img/ue_crosswalks.png)  

*   __If additional props are added, change the navigation accordingly.__ These may interfere with the navigation. This is the case for streetlights, trees and other elements on the navigation areas. Follow the steps below to create a new navigation file. 

!!! Warning
    The Docker import does not allow additional modifications. The navigation file has already been packaged and cannot be edited. 


__1.__ Select the __Skybox object__ and add a tag `NoExport` to it. Otherwise, the map will not be exported, as the size would be too big. 

![ue_skybox_no_export](img/ue_noexport.png) 

__2.__ Check the name of the meshes. By default, pedestrians will be able to walk over sidewalks, crosswalks, and grass (with minor influence over the rest).  

*   Sidewalk = `Road_Sidewalk`.  
*   Grass = `Road_Grass`.  

![ue_meshes](img/ue_meshes.png) 

__3.__ Name these planes following the common format `Road_Crosswalk_mapname`. 

__4.__ Press `G` to deselect everything, and export the map. `File > Export CARLA...`.  
__5.__ Run RecastDemo `./RecastDemo`.  
__6.__ Change the name of the `.bin` file to be the same as the `mapname.fbx`. Move it into `Content/Carla/Maps/Nav`.  

---
## Deprecated ways to import a map

There are other ways to import a map used in previous CARLA releases. These required to manually cook the map and prepare everything, so they are now deprecated. However, they are explained below in case they are needed.  

### RoadRunner plugin import

This software provides specific plugins for CARLA. Get those and follow some simple steps to get the map.  

  <details>
    <summary> Read RoadRunner plugin import guide
    </h4></summary>

!!! Warning
    These importing tutorials are deprecated. [Docker import](tuto_A_import_assets.md#via-docker) is the recommended way to import a map.  

### Plugin installation

These plugins will set everything ready to be used in CARLA. It makes the import process more simple. 

__1. Locate the plugins__ in RoadRunner's installation folder  
`/usr/bin/VectorZero/Tools/Unreal/Plugins`.

__2. Copy those folders__ to the CarlaUE4 plugins directory `/carla/Unreal/CarlaUE4/Plugins/`.

__3. Rebuild the plugin__ following the instructions below.  

*   __a) Rebuild on Windows.__  
	* Right-click the `.uproject` file and `Generate Visual Studio project files`.  
	* Open the project and build the plugins.  

*   __b) Rebuild on Linux.__  
	* Run the following command.  
```sh
> UE4_ROOT/GenerateProjectFiles.sh -project="carla/Unreal/CarlaUE4/CarlaUE4.uproject" -game -engine
```

__4. Restart Unreal Engine.__ Make sure the checkbox is on for both plugins `Edit > Plugins`. 

![rr_ue_plugins](img/rr-ue4_plugins.png)

### Import map

__1. Import the _mapname.fbx_ file__ to a new folder under `/Content/Carla/Maps` with the `Import` button.  

![ue_import](img/ue_import_mapname.png)

__2. Set `Scene > Hierarchy Type`__ to _Create One Blueprint Asset_ (selected by default).  
__3. Set `Static Meshes > Normal Import Method`__ to _Import Normals_.  

![ue_import_options](img/ue_import_options.png)

__4. Click `Import`.__  
__5. Save the current level__ `File > Save Current As...` > _mapname_.  

The new map should now appear next to the others in the Unreal Engine _Content Browser_.

![ue_level_content](img/ue_level_content.png)
</details>

!!! Note
    The tags for semantic segmentation will be assigned by the name of the asset. And the asset moved to the corresponding folder in `Content/Carla/PackageName/Static`. To change these, move them manually after imported. 


### Manual import

This process requires to go through all the process manually. From importing _.fbx_ and _.xodr_ to setting the static meshes.  

  <details>
    <summary>  Read manual import guide
    </summary>

!!! Warning
    These importing tutorials are deprecated. [Docker import](tuto_A_import_assets.md#via-docker) is the recommended way to import a map.  

This is the generic way to import maps into Unreal Engine using any _.fbx_ and _.xodr_ files. As there is no plugin to ease the process, there are many settings to be done before the map is available in CARLA.  

__1. Create a new level__ with the **Map** name in Unreal `Add New > Level` under `Content/Carla/Maps`.  
__2. Copy the illumination folder and its content__ from the BaseMap `Content/Carla/Maps/BaseMap`, and paste it in the new level. Otherwise, the map will be in the dark.

![ue_illumination](img/ue_illumination.png)

### Import binaries

__1. Import the _mapname.fbx_ file__ to a new folder under `/Content/Carla/Maps` with the `Import` button. __Make sure the following options are unchecked.__

*   Auto Generate Collision  
*   Combine Meshes  
*   Force Front xAxis  
*   Normal Import Method - _To import normals_  

__2. Check the following options.__  

*   Convert Scene Unit
*   _To import materials and textures._
    *   Material Import Method - _To create new materials_
    *   Import Textures

![ue_import_file](img/ue_import_file.png)

__3. Check that the static meshes have appeared__ in the chosen folder.

__4. Drag the meshes__ into the level.

![ue_meshes](img/ue_drag_meshes.png)

__5. Center the meshes at point (0,0,0)__ when Unreal finishes loading. 

![Transform_Map](img/transform.jpg)

__6. Generate collisions__. Otherwise, pedestrians and vehicles will fall into the abyss.

*   Select the meshes meant to have colliders.
*   Right-click `Asset Actions > Bulk Edit via Property Matrix...`.

  ![ue_selectmesh_collision](img/ue_selectmesh_collision.png)

*   Search for _collision_ in Property's Matrix search box.
*   Change `Collision complexity` from `Project Default` to `Use Complex Collision As Simple`.

  ![ue_collision_complexity](img/ue_collision_complexity.png)

*   Go to `File > Save All`.

__7. Move the static meshes__ from `Content/Carla/Maps/mapfolder` to the corresponding `Carla/Static` subsequent folder. This will be meaningful for the semantic segmentation ground truth.  

  *   `Terrain/mapname`
  *   `Road/mapname`
  *   `RoadLines/mapname`

```sh
Content
└── Carla
    ├── Blueprints
    ├── Config
    ├── Exported Maps
    ├── HDMaps
    ├── Maps
    └── Static
        ├── Terrain
        │   └── mapname
        │       └── Static Meshes
        │
        ├── Road
        │   └── mapname
        │       └── Static Meshes
        │
        ├── RoadLines  
        |   └── mapname
        |       └── Static Meshes
        └── Sidewalks  
            └── mapname
                └── Static Meshes
```

![ue__semantic_segmentation](img/ue_ssgt.png)

### Import OpenDRIVE files

__1. Copy the `.xodr` file__ inside the `Content/Carla/Maps/OpenDrive` folder.  
__2. Open the Unreal level.__ Drag the _Open Drive Actor_ inside the level. It will read the level's name. Search the Opendrive file with the same name and load it.

![ue_opendrive_actor](img/ue_opendrive_actor.png)

</details>
<br>


### Set traffic and pedestrian behaviour

This software provides specific plugins for CARLA. Get those and follow some simple steps to get the map.  

  <details>
    <summary>Read traffic and pedestrian setting guide</summary>

!!! Warning
    These importing tutorials are deprecated. [Docker import](tuto_A_import_assets.md#via-docker) is the recommended way to import a map.  

### Set traffic behavior

Once everything is loaded into the level, it is time to create traffic behavior.  

__1. Click on the _Open Drive Actor_.__  
__2. Check the following boxes in the same order.__  

*   Add Spawners.  
*   _(Optional for more spawn points)_ On Intersections.  
*   Generate Routes.  

This will generate a series of _RoutePlanner_ and _VehicleSpawnPoint_ actors. These are used for vehicle spawning and navigation.

### Traffic lights and signs

Traffic lights and signs must be placed all over the map.

__1. Drag traffic light/sign actors__ into the level and place them.  
__2. Adjust the [`trigger volume`][triggerlink]__ for each of them. This will determine their area of influence.  
  [triggerlink]: python_api.md#carla.TrafficSign.trigger_volume

![ue_trafficlight](img/ue_trafficlight.png)

__3. In junctions, drag a traffic light group actor__ into the level. Assign to it all the traffic lights involved and configure their timing. Make sure to understand [how do traffic lights work](http://127.0.0.1:8000/core_actors/#traffic-signs-and-traffic-lights).  

![ue_tl_group](img/ue_tl_group.png)

__4. Test traffic light timing and traffic trigger volumes.__ This may need trial and error to fit perfectly.

![ue_tlsigns_example](img/ue_tlsigns_example.png)

> _Example: Traffic Signs, Traffic lights and Turn based stop._

---
### Add pedestrian navigation

In order to prepare the map for pedestrian navigation, there are some settings to be done before exporting it.  

__1.__ Select the __Skybox object__ and add a tag `NoExport` to it. Otherwise, the map will not be exported, as the size would be too big. 

![ue_skybox_no_export](img/ue_noexport.png) 

__2.__ Check the name of the meshes. By default, pedestrians will be able to walk over sidewalks, crosswalks, and grass (with minor influence over the rest).

![ue_meshes](img/ue_meshes.png) 

__3.__ Crosswalks have to be manually created. For each of them, create a plane mesh that extends a bit over both sidewalks connected. __Place it overlapping the ground, and disable its physics and rendering__. 

![ue_crosswalks](img/ue_crosswalks.png) 

__4.__ Name these planes following the common format `Road_Crosswalk_mapname`. 

__5.__ Press `G` to deselect everything, and export the map. `File > Export CARLA...`.  
__6.__ Run RecastDemo `./RecastDemo`.  

  * Select `Solo Mesh` from the `Sample` parameter's box.
  * Select the _mapname.obj_ file from the `Input Mesh` parameter's box.
![recast_example](img/recast_example.png)

__7.__ Click on the `Build` button.  
__8.__ Once the build has finished, click on the `Save` button.  
__9.__ Change the **filename** of the binary file generated at `RecastDemo/Bin` to `mapname.bin`.  
__10.__ Drag the _mapname.bin_ file into the `Nav` folder under `Content/Carla/Maps`.  

</details>
<br>

---

That comprises the process to create and import a new map into CARLA. If during the process any doubts arise, feel free to post these in the forum. 

<div class="build-buttons">
<p>
<a href="https://forum.carla.org/" target="_blank" class="btn btn-neutral" title="Go to the CARLA forum">
CARLA forum</a>
</p>
</div>