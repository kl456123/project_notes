
## name method
class names start with lowercase "vtk"(vtkSphereSource)
name of class members start with upper case letters (vtkSpereSource::SetResolution(int))

## usage
* Smart pointer
vtkSmartPointer
```
vtkSmartPointer<vtkSpereSource>
```

* create object
```
vtkSmartPointer<vtkSphereSource> sphereSource = 
    vtkSmartPointer<vtkSphereSource>::New();
```

## objects
vtkObjectBase and vtkObject

## data-flow architecture
vtkDataSet is parent object , vtkImageData , vtkPolyData and vtkRectilinearGrid.

### three types of processes
* Source(no input)
** take vtkSpereSource for example, It specifies the geometry and topology of a sphere and makes it available to other nodes.
** Other examples of source are all subclasses of the vtkReader class, read data from files of various formats and make them available to other ones.
* filters(have input and output)
* sinks (input but no output)
vtkRenderer and all subclasses of vtkWriter are also sinks

## How to code in cpp

```
vtkSmartPointer<vtkConeSource> coneSource = vtkSmartPointer<vtkConeSource>::New();
coneSource->Update();
vtkSmartPointer<vtkPolyDataMapper> coneMapper = vtkSmartPointer<vtkPolyDataMaper>::New();
coneMapper->SetInputData(coneSource->GetOutput());

all props in VTK are subclasses of vtkProp.
// those props that are renderable objects are called actors and are represented using vtkActor class
vtkSmartPointer<vtkActor> cone = vtkSmartPointer<vtkActor>::New();
//vtkProperty is also created
cone->SetMapper(coneMapper);

vtkSmartPointer<vtkRenderer> renderer = vtkSmartPointer<vtkRenderer>::New();
//vtkLight and vtkCamera are also created
renderer->AddActor(cone);

vtkSmartPointer<vtkRenderWindow> renderWindow = vtkSmartPointer<vtkRenderWindow>::New();
renderWindow->AddRenderer(renderer);
renderWindow->Render();

vtkSmartPointer<vtkRenderWindowInteractor> renderWindowInteractor = vtkSmartPointer<vtkRenderWindowInteractor>::New();
renderWindowInteractor->SetRenderWindow(renderWindow);
renderWindowInteractor->Start();

```

* Note that basic pipeline source mapper actor render
render.GetActivateCamera()
render.SetViewPort(xmin,ymin,xmax,ymax)
render.SetBackground(r,g,b)
cone.SetResolution(num_of_facets)

property = vtk.vtkProperty()
coneActor2.SetProperty(property)
coneActor2.SetPosition(0, 2, 0)

## interactive
iren = vtk.vtkRenderWindowInteractor()
style = vtkInteractorStyle()
iren.SetInteractorStyle(style)

position sensitive joystick
motion sensitive trackball

### Widget
boxWidget = vtk.vtkBoxWidget()
t = vtk.vtkTransform()
boxWidget.GetTransform(t)
boxWidget.GetProp3D().SetUserTransform(t)

## Light
ambient light that is being reflected or scattered from other objects.
diffuse light
specular Light

Positional lights have an associated cone angle and attenuation factors.

## Camera

### parameters
* The vector defined from the camera position to the focal point is called the direction of projection.
The camera orientation is controlled by the position and focal point plus the camera view-up vector
* the front and back clipping planes
* angle
1. azimuth, elevation, roll
2. yaw , pitch

3. dollying(distance from the focal point)
4. zooming(view angle of camerat)

### method of projection
* orthographic projection,all rays of light entering the camera are parallel to the projection vector

### coordinate system
1. model coordinate system(local)
2. world coordinate system(actor for converting from model's coordintes into world coordinates)
in which the position and orientation of camera and lights are specified.
3. view coordinate (x,y in image plane ranges between(-1,1) ,z refers distance or range from the camera. 4x4 transform matrix)
4. display coordinate system (x,y  are the actual piexl locations,z-buffer used for occulsion)
viewport 
## vtkAssembly(subclass of vtkActor)


* examples
```
renderer = vtk.vtkRenderer()
renderer.SetBackground()

renwin = vtk.vtkRenderWindow()
renwin.AddRenderer(renderer)

interactor = vtk.vtkRenderWindowInteractor()
interactor.SetRenderWindow(renwin)

style = vtk.vtkStyle()
style.SetDefaultRenderer(renderer)

interactor.SetInteractorStyle(style)
```

## Picker
* vtkPropPicker
* vtkAreaPicker
* vtkPicker
```
picker = vtk.vtkCellPicker()
picker.AddObserver("EndPickEvent",annotatePick)
iren.SetPicker(picker)
```

## graphics primitives
* Triangle strip
a series of triangles where each triangle shares its edges with its neighbors
* Polygon
a set of edges,usually in a plane, that define a closed region. Triangles and rectangles are examples of polygons.
* Polyline
* Line
* Point(vertex)
position, normal and color each of which is a three element vector

## Conclusion
1. vtkMapper defines actor geometry,more than one actor may refer to the same mapper.
3. vtkCamera defines view for each renderer,view position,focal point,and other viewing properties of the scene.
4. one or more vtkLight's illuminate the scene
5. vtkRendererWindow, manages a window on the display device; one or more renderers draw into an instance of vtkRenderWindow,store graphics specific characteristics of the display window such as size,position ,window titile ,window depth and the double buffering flag.72 bits 
6. vtkRenderer,coordinates the rendering process involving lights,cameras and actors.
7. vtkActor,represents an object rendered in the scene,including its properties and position in the world coordinate system(vtkActor is a subclass of vtkProp.)
8. vtkProperty,defines the appearance properties of actor including color, transparency, and lighting properties such as specular and diffuse. Also representational properties like wireframe and solid surface.


view point coordinates(0,0,1,1)

## Note
*class vtkActor,combines object properties(color, shading type,etc. use vtkProperty),geometric definition(vtkMapper) and orientation(vtkTransform) in the world coordinate system.

*class vtkProp(such as vtkFollower for signs or text)

*class vtkMapper refers to a table of colors(i.e., vtkLookupTable)

* vtkRenderWindowInteractor
captures these events and then triggers certain operations like camera dolly, pan and rotate,actor picking ,into/out of stereo mode,and so on.  
use SetRenderWindow() method to associated with a rendering window


window is calcualted by pixel,but viewport is calculated by rate

## Events
* "w": draws all actors in wireframe
* "s": draws the actors in surface form
* "3": toggles in and out of 3D stereo
* "r": resets camera view
* "e": exits the application
* "u": executes a user-defined function
* "p": picks the actor under the mouse pointer

Origin(Ox,Oy,Oz)
Position(Px,Py,Pz)
Scale(Sx,Sy,Sz)

EraseOff() so that we can see the effects of the rotations.

vtkTransform
Translate
RotateZ,RotateX,Scale

discrete ,regular/irregular, and data dimension,shape

## Dataset
Data objects with an organizing structure and associated data attributes form datasets.

### structure(two part)

* topology(cell) is the set of properties invariance under certain geometric transformations.
* geometry(points) is the instantiation of the topology, the specification of position in 3D space

### data attributes
* for example,temperature value at a point or the intertial mass of a cell
typical attributes include scalars,vectors,normals,texture coordinates and tensors.

Points are located where data is known and the cells allow us to interpolate between points.
* Scalar,Vectors,Normals, Texture, Coordinates, Tensor,
### Cell
Cells are defined by specifying a type in combination with an ordered list(connectivity list,combined with the type specification) of points
* the ordered list is a sequence of point ids that index into a point coordinate list
number of points is the size of the cell
* linear types
* nonlinear types
1. quadratic interpolation functions
2. ...

## Interactor Styles
1. use a subclass of vtkInteractorStyle
2. add observers that watch for events on the vtkRenderWindowInteractor and define your own set of callbacks(or commands)
3. 3D widgets are another more complex way to interact with data in scene


## level of detail actors
1. point cloud sample from the points defining the mapper's input
2. bounding box of the actor
additional levels of detail can be added using the AddLODMapper

