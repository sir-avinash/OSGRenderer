Matlab Offscreen Renderer
=========================

[TOC]


Efficient Matlab Wrapper for Offscreen Rendering engine using [OpenSceneGraph 3](https://github.com/openscenegraph/osg) and [Mexplus](https://github.com/kyamagu/mexplus). The Matlab wrapper holds the C++ object instance and returns rendering whenever the Matlab wrapper object requests it. It is more since you do not load a CAD model every time you render it. 

There are two modes for installation. One that does not require OSG installation which is recommended and the one that works without OSG installation.

Requirement 
------------

MATLAB version 2013a or above

Install : Standard (Linux/Mac)
------------------

`Note` Prebuilt OSG library only works in Linux. If you want to use 

1. Install Open Scene Graph from https://github.com/openscenegraph/osg

    ```
    git clone https://github.com/openscenegraph/osg
    cd osg
    make all
    sudo make install
    ```

2. Locate `mexopts.sh` file which is used for compiling mex file. 
    - Type `mex -v -n` in the MATLAB command line
    - Read the output and locate `mexopts.sh` file. There are two paths: 1. your Matlab default file 2. your current configuration file. You must edit the current configuration file.

3. Add `-std=c++11` to CXXFLAGS in `mexopts.sh`
    - For Mac
        - add line `CFLAGS="$CFLAGS -std=c++11"
    - For Linux
        - if you are using G++ version < 4.7 add line `CFLAGS="$CFLAGS -std=c++0x"` 
        - if you are using G++ version >= 4.7 add line `CFLAGS="$CFLAGS -std=c++11"`

4. **Mac only** add `-framework OpenGL` to `MLIBS`

5. Clone the MatlabRenderer repo

    ```
    git clone https://github.com/chrischoy/MatlabRenderer.git
    ```

6. Go to the `MatlabRenderer` folder and run `compile.m`

Install : Prebuild (Linux Only)
-------------------------------

1. Clone the MatlabRenderer repo

    ```
    git clone https://github.com/chrischoy/MatlabRenderer.git
    ```
    
2. Add compile library path and runtime library path. Note that `LD_LIBRARY_PATH` is for application library search path and `LIBRARY_PATH` is for compiler library search path. If you open new command line session, you must these lines every time unless you add the ld library path to `.bashrc`
    
    ```
    cd MatlabRenderer
    export LIBRARY_PATH=LIBRARY_PATH:path/to/MatlabRenderer/lib/osg/:path/to/MatlabRenderer/lib/boost/
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:path/to/MatlabRenderer/lib/osg:path/to/MatlabRenderer/lib/boost/
    ```
    
3. Run matlab


Other IDE (Eclipse)
-------------------

1. Use OpenSceneGraph https://github.com/openscenegraph/osg to download latest OSG.
2. Eclipse Setting

- Library to include

    ```
    GL, GLU, osg, osgDB, osgGA, osgViewer, osgUtil, boost_program_options
    ```

- If you installed OSG on `/usr/local/lib64` (which is default)
go to `Run Configuration`, add Environment variable `LD_LIBRARY_PATH` and value `/usr/local/lib64`

- Compile
    
    ```
    g++ -o MatlabRenderer MatlabRenderer.cpp -lGL -losg -losgViewer -losgDB -losgGA -losgUtil -lboost_program_options -O3
    ```


Usage
-----

```
% Initialize the Matlab object.
renderingSizeX = 700; renderingSizeY = 700; % pixels

azimuth = 90; elevation = 45; yaw = 0;
% if you use field of view, set distance to 0
distance = 0; fieldOfView = 25; 


% Setup Renderer
renderer = Renderer();
if ~renderer.initialize({'mesh/Honda-Accord.3ds', 'mesh/road_bike.3ds',...
	 'mesh/untitled.dae'},700,700,45,0,0,0,25)
    error('Renderer initilization failed');
end



% If the output is only the rendering, it renders more efficiently
renderer.setModelIndex(1);
renderer.setViewpoint(az,el,yaw,0,fov);
[rendering]= renderer.render();



% Once you initialized the renderer, you can just set 
% the viewpoint and render without loading CAD model again.
renderer.setModelIndex(2);
renderer.setViewpoint(az,el,yaw,0,fov);
[rendering]= renderer.render();



% If you give the second output, it renders depth too.
renderer.setModelIndex(1);
renderer.setViewpoint(0,20,0,0,25);

[rendering, depth]= renderer.render();
subplot(121);imagesc(rendering); axis equal; axis off;
subplot(122);imagesc(1-depth); axis equal; axis off; colormap hot;


% You must clear the memory before you exit
renderer.delete();
```

Example 

![](https://dl.dropboxusercontent.com/u/57360783/MatlabRenderer/rendering_with_depth.png)


Input CAD format
----------------

List of available formats 
http://trac.openscenegraph.org/projects/osg//wiki/Support/UserGuides/Plugins

To get free models, use [Google 3D Warehouse](https://3dwarehouse.sketchup.com) and use Sketchup to edit models.
Rotate the model and export it to the format that you will use.

COLLADA (DAE) format support
-------------
Follow the direction of
https://github.com/openscenegraph/osg/tree/master/src/osgPlugins/dae

The precompiled library contains osgPlugin for collada format. (LINUX)


ISSUE
----------------

Warning: could not find plugin to read objects from file *.*

The reason that it fails to load is that the renderer fails to load appropriate shared library. In the prebuilt ./OSG/lib/osgPlugins-3.3.2, yoou can see several plugins. Add the plugin path to 'LD_L
IBRARY_PATH`.

You must either install complete OSG package or download from http://trac.openscenegraph.org/projects/osg//wiki/Downloads/Dependencies

TODO
----

1. Windows installation
2. Add dylib for prebuilt OS X

- http://stackoverflow.com/questions/3146274/is-it-ok-to-use-dyld-library-path-on-mac-os-x-and-whats-the-dynamic-library-s

> Written with [StackEdit](https://stackedit.io/).
