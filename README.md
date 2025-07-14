
# OMPL (Open Motion Planning Library) – Clean Installation Guide

This guide provides a clean and modern approach to building and installing **OMPL** from source in a Conda environment. It ensures compatibility with **Boost** and **Eigen** dependencies and is tested to work with **Python 3.10 and 3.11**.

---

## 📁 Directory Structure Assumed

```
~/Downloads/
├── boost_1_80_0/
├── ompl-1.7.0/
```

---

## ⚙️ Step 1: Setup Conda Environment

```bash
conda create -n ompl-env python=3.11
conda activate ompl-env
```

---

## 📦 Step 2: Install Eigen via Conda

```bash
conda install -c conda-forge eigen
```

---

## 🛠️ Step 3: Build and Install Boost (with Python bindings)

### 3.1 Download Boost
```bash
cd ~/Downloads
wget https://boostorg.jfrog.io/artifactory/main/release/1.80.0/source/boost_1_80_0.tar.gz
tar -xvzf boost_1_80_0.tar.gz
cd boost_1_80_0
```

### 3.2 Bootstrap with Conda Python
```bash
./bootstrap.sh --prefix=$CONDA_PREFIX --with-python=$(which python)
```

### 3.3 Install Boost
```bash
./b2 install
```

Verify:
```bash
ls $CONDA_PREFIX/lib/libboost_python*
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

---

## 🧾 Notes

- Boost must be built with the same Python version as your Conda environment.
- Python 3.11 users must patch Boost or use a version ≥1.82 to avoid GC errors with Boost.Python enums.
- This guide uses out-of-source CMake builds and Ninja for faster compilation.

---

## 🏁 Done!

You now have a clean, fully working OMPL + Boost + Eigen setup for development and research.
