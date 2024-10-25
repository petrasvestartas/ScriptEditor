# Rhino Script Editor

## Anaconda in Rhino8
```python
"""
NOTE:

- Reference to RhinoCommmon.dll is added by default

- You can specify your script requirements like:

    # r: <package-specifier> [, <package-specifier>]
    # requirements: <package-specifier> [, <package-specifier>]

    For example this line will ask the runtime to install
    the listed packages before running the script:

    # requirements: pytoml, keras

    You can install specific versions of a package
    using pip-like package specifiers:

    # r: pytoml==0.10.2, keras>=2.6.0

- Use env directive to add an environment path to sys.path automatically
    # env: /path/to/your/site-packages/
"""
#! python3

#! python 3

import os
import os.path as op
import sys
import ctypes

CONDA_ENV = r'C:\Users\petrasv\AppData\Local\anaconda3\envs\wood-rhino-plugin'
COMPAS_WOOD_PATH = r'C:\brg\2_code\compas_wood\src'

# add the paths of site-packages in conda environment so other packages that compas_wood
# depend on can be found e.g. compas
sys.path.append(op.join(CONDA_ENV, r"Lib\site-packages"))

# add the location of compas_wood source so we can import this
sys.path.append(COMPAS_WOOD_PATH)

# tell python where it can find dlls. this is required to find all other .dll files that
# are installed as part of the other packages in the conda environment e.g. fblas
os.add_dll_directory(op.join(CONDA_ENV, r'Library\bin'))

# tell python where the wood_pybind11*.pyd is located
os.add_dll_directory(COMPAS_WOOD_PATH)

from compas_wood.joinery import get_connection_zones
import time
from compas_wood.joinery import filenames_of_datasets
from compas_wood.joinery import read_xml_polylines_and_properties
from compas_wood.cpp import rpc_test
import Rhino


def test_connection_detection():
    # joinery parameters
    division_length = 300
    joint_parameters = [
        division_length,
        0.5,
        9,
        division_length * 1.5,
        0.65,
        10,
        division_length * 1.5,
        0.5,
        21,
        division_length,
        0.95,
        30,
        division_length,
        0.95,
        40,
        division_length,
        0.95,
        50,
    ]


    # read the xml file and get a data-set
    foldername = (
        "C:/brg/2_code/compas_wood/src/frontend/src/wood/dataset/"
    )

    filename_of_dataset = filenames_of_datasets[32]

        
    # generate joints
    
    output_polylines, output_vectors,output_joints_types, output_three_valence_element_indices_and_instruction, output_adjacency = read_xml_polylines_and_properties(foldername, filename_of_dataset)
    

    result = get_connection_zones(
        output_polylines,
        output_vectors,
        output_joints_types,
        output_three_valence_element_indices_and_instruction,
        output_adjacency,
        joint_parameters,
        0,
        [1, 1, 1],
        4   
    )

    start_time = time.time()
    rpc_test()

    print(
         "\n______________________________________ %s ms ______________________________________"
        % round((time.time() - start_time) * 1000.0, 2)
    )


    # display
    result_flat_list = [item for sublist in result for item in sublist]
    polylines=[]
    for plines in result_flat_list:
        pline = Rhino.Geometry.Polyline()
        for p in plines:
            pline.Add(Rhino.Geometry.Point3d(p[0],p[1],p[2]))
        Rhino.RhinoDoc.ActiveDoc.Objects.AddPolyline(pline)
        

    print(result_flat_list[0][0])
    # print(result_flat_list)

test_connection_detection()
```



## Rhino CPython

Important Notes:

* compas_wood depends on wood_pybind11.cp39-win_amd64.pyd that has its own list of dependencies.
* The command dumpbin /dependents C:\Users\ein\compas_wood\src\wood_pybind11.cp39-win_amd64.pyd provides a list of dlls that the .pyd depends on. We need to make sure python can find these dlls.

```
File Type: DLL
  Image has the following dependencies:
    liblapack.dll
    libblas.dll
    python39.dll
    libcblas.dll
    KERNEL32.dll
    VCRUNTIME140.dll
    api-ms-win-crt-stdio-l1-1-0.dll
    api-ms-win-crt-heap-l1-1-0.dll
    api-ms-win-crt-runtime-l1-1-0.dll
    api-ms-win-crt-string-l1-1-0.dll
```
* .pyd is compiled against your conda env python which means that python.exe can easily find these aforementioned dlls.

* The compas_wood packages is also installed there which means Libs/site-packages/easy-install.pth file points the python runtime to where compas_wood modules is located.

* Now to get the python in Rhino to load the package installed in a conda environment, we need to:

```
#! python 3

import os
import os.path as op
import sys
import ctypes

CONDA_ENV = r'C:\Users\ein\.conda\envs\wood-dev'
COMPAS_WOOD_PATH = r'C:\Users\ein\compas_wood\src'

# add the paths of site-packages in conda environment so other packages that compas_wood
# depend on can be found e.g. compas
sys.path.append(op.join(CONDA_ENV, r"Lib\site-packages"))

# add the location of compas_wood source so we can import this
sys.path.append(COMPAS_WOOD_PATH)

# tell python where it can find dlls. this is required to find all other .dll files that
# are installed as part of the other packages in the conda environment e.g. fblas
os.add_dll_directory(op.join(CONDA_ENV, r'Library\bin'))

# tell python where the wood_pybind11*.pyd is located
os.add_dll_directory(COMPAS_WOOD_PATH)

# now we can import the module and test
from compas_wood.joinery import test

print(test)
test()
```

 * I also added a ticket to possibly improve this: RH-80486 Investigate loading a conda environment directly in Rhino






