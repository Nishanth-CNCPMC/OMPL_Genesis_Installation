
# OMPL (Open Motion Planning Library) – Clean Installation Guide

This guide provides a clean and modern approach to building and installing **OMPL** from source in a Conda environment. It ensures compatibility with **Boost** and **Eigen** dependencies and is tested to work with **Python 3.11**.

---

Download version 1.8.0 boost and eigen 3.4.0 library from: 

```https://www.boost.org/releases/1.80.0/```

```https://eigen.tuxfamily.org/index.php?title=Main_Page```


## 📁 Directory Structure Assumed

```
~/Downloads/
├── boost_1_80_0/
├── eigen-3.4.0.tar.gz
```

---

## ⚙️ Step 1: Setup Conda Environment

Use the torch.yml file in the repo and create an env using the following command:

```bash
conda env create -f torch.yml
conda activate torch-env #(If not already activated)
```

---

## 📦 Step 2: Install Eigen via Conda

```bash
cd ~/Downloads
tar -xvzf eigen-3.4.0.tar.gz
cd eigen-3.4.0
```
Build and Install:

```
mkdir -p build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX
make install

```

Verify installation:

```bash
ls $CONDA_PREFIX/include/eigen3/Eigen
```

---

## 🛠️ Step 3: Build and Install Boost (with Python bindings)


### 3.1 Bootstrap and Install in Conda Env
```bash
./bootstrap.sh
./b2
./b2 install --prefix=$CONDA_PREFIX
```


Verify:
```bash
ls $CONDA_PREFIX/lib/libboost_python*
```

Before Installing OMPL, expose the env variables:

```bash
export BOOST_ROOT=$CONDA_PREFIX 
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
export CPLUS_INCLUDE_PATH=$CONDA_PREFIX/include:$CPLUS_INCLUDE_PATH
```

---

## 🧹 Step 4: Clean Out Any Previous OMPL Installations

```bash
rm -rf $CONDA_PREFIX/include/ompl*
rm -rf $CONDA_PREFIX/lib/libompl*
rm -rf $CONDA_PREFIX/lib/python*/site-packages/ompl
```

---

## 🔧 Step 5: Build and Install OMPL

### 5.1 Download OMPL

```bash
cd ~/Downloads
wget https://github.com/ompl/ompl/archive/1.7.0.tar.gz -O ompl-1.7.0.tar.gz
tar -xvzf ompl-1.7.0.tar.gz
cd ompl-1.7.0
```

### 5.2 Configure with CMake

```bash
cmake -G Ninja -S . -B build \
  -DPYTHON_EXEC=$(which python) \
  -DOMPL_REGISTRATION=OFF \
  -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX
```

### 5.3 Build and Install

```bash
cmake --build build --target update_bindings
cmake --build build
cmake --install build
```

---

## ✅ Step 6: Test OMPL Python Binding

Create `test_ompl_rrt.py`:

```python
from ompl import base as ob
from ompl import geometric as og

space = ob.RealVectorStateSpace(2)
bounds = ob.RealVectorBounds(2)
bounds.setLow(0)
bounds.setHigh(1)
space.setBounds(bounds)

si = ob.SpaceInformation(space)
si.setStateValidityChecker(ob.StateValidityCheckerFn(lambda s: True))
si.setup()

start = ob.State(space)
start[0] = 0.0
start[1] = 0.0
goal = ob.State(space)
goal[0] = 1.0
goal[1] = 1.0

pdef = ob.ProblemDefinition(si)
pdef.setStartAndGoalStates(start, goal)

planner = og.RRT(si)
planner.setProblemDefinition(pdef)
planner.setup()

if planner.solve(1.0):
    print("Found solution:")
    print(pdef.getSolutionPath())
else:
    print("No solution found")
```

Run:
```bash
python test_ompl_rrt.py
```

Possible Error:

Python 3.11 gives SystemError: type Boost.Python.enum has the Py_TPFLAGS_HAVE_GC flag but has no traverse function

Fix: 

Step 1: 

```bash
~/Downloads/boost_1_80_0 && gedit ./libs/python/src/object/enum.cpp
```

```
#if PY_VERSION_HEX < 0x03000000
  | Py_TPFLAGS_CHECKTYPES
#endif
  | Py_TPFLAGS_HAVE_GC                    /* <============== remove this line (I think line 110)*/
  | Py_TPFLAGS_BASETYPE,                  /* tp_flags */
```
Step 2: Repeat from main step 3 again.

---

## 🧾 Notes

- Boost must be built with the same Python version as your Conda environment.
- Python 3.11 users must patch Boost or use a version ≥1.82 to avoid GC errors with Boost.Python enums.
- This guide uses out-of-source CMake builds and Ninja for faster compilation.

---

## 🏁 Done!

You now have a clean, fully working OMPL + Boost + Eigen setup for development and research.
