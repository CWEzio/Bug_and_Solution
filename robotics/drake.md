# Introduction
My notes on Drake

# Spatial Velocity and Spatial Force Notation
Drake's [spatial velocity](https://drake.mit.edu/doxygen_cxx/classdrake_1_1multibody_1_1_spatial_velocity.html) and [spatial force](https://drake.mit.edu/doxygen_cxx/classdrake_1_1multibody_1_1_spatial_force.html) are with respect to a specific point, which is slightly different with Modern Robotics book. In my opinion, Drake's definition is more physical and has more representation power. See the [Spatial Vector](https://drake.mit.edu/doxygen_cxx/group__multibody__spatial__vectors.html) page for more information.


# Usage

## Pydrake
- Pydrake's binding is defined in the /bindings/pydrake directory. You can check the source code to see whether the function you need has been bound to Python.

### Pydrake's symbolic comparison operator for Vector of `Expression` 
Related information can be found in [this issue](https://github.com/RobotLocomotion/drake/issues/8315) (most relevant), [this pull](https://github.com/RobotLocomotion/drake/pull/11171), [this question](https://stackoverflow.com/questions/64736910/using-le-or-ge-with-scalar-left-hand-side-creates-unsized-formula-array) and [this question](https://stackoverflow.com/questions/68615278/runtimeerror-you-should-not-call-bool-nonzero-on-formula).

It should be noted that, in pydrake, the comparison operator like `<=` can not operate on a vector of expressions. Instead, `lt`, `le`, `eq`, `ne`, `ge` and `gt` should be used. These operators are defined in `/binding/pydrake/_math_extra.py`. These operators are defined as a workaround way to avoid the undesired behaviors when operators like `<=` act on vector of expressions.

I think that `C++` does not have this issue. 

## Visualization
### To visualize the pose without doing simulation
Check the [`geometry_inspector.py`](https://github.com/RobotLocomotion/drake/blob/e59b7fc18dbe80b827d07e4a3283a0c87eda7021/manipulation/util/geometry_inspector.py) file and the `MultibodyPositionToGeometryPose` class.

### set `Meshcat` reconnection time
`http://localhost:7000/?reconnect_ms=100` sets reconnection time to 100ms

## Optimization

### Unbounded optimization variable will cause the solver to fail
Recently I encountered a problem to solve for the Lyapunov function for linear system $\dot{x}=Ax$. 
> Note that for a continuous linear system, it is stable if and only if all its eigen values have negative real part; while for discrete continuous linear system, it is stable if and only if its eigen values all have norm less than 1.

To find the Lyapunov function for this system, we can solve the following optimization problem

$$\text{find } P \\  \text{subject to } P \succeq 0 \text{ and } \ PA+A^TP \preceq 0 \ $$

The code for solving this problem is 

```python
# SDP for stability
# Example Linear systems
A = np.mat('-1,0;3,-2')

# A = np.mat('-1,2;1,2')
print(np.linalg.eig(A))

prog = MathematicalProgram()
P = prog.NewSymmetricContinuousVariables(2,'P')
alpha = prog.NewContinuousVariables(1,'alpha')

prog.AddCost(-alpha[0])

prog.AddPositiveSemidefiniteConstraint(P-0.01*np.eye(2))
prog.AddPositiveSemidefiniteConstraint(-A.transpose().dot(P) - P.dot(A) - alpha[0]*np.eye(2))
result = Solve(prog)
resultP = result.GetSolution(P)
resultAlpha = result.GetSolution(alpha)

print(resultP)
print(f"alpha = {resultAlpha}")
print(result.is_success())
P = result.GetSolution(P)
print(f"eig(P) = {np.linalg.eig(P)[0]}")
print("eig(Pdot) = " +
    str(np.linalg.eig(A.transpose().dot(P) + P.dot(A))[0]))
```

However, the solver fails to solve this problem. It turns out that for this optimization objective $-\alpha$, the problem is unbounded above for decision variable $\alpha$. Adding a bounding box constraint will solve the problem
```python
prog.AddBoundingBoxConstraint([0.01],  [10], alpha)
```
Note that alpha will achieve the upper bound.

> It quiet suprise me first that the solver can fail for a feasible problem. But the solver cannot return a solution say that the best solution is $\infty$. Therefore the solver will not only fail at infeasible problem, but for problem that the optimal solution is unatainable.
### Use MOSEK in a drake-bazel-external project
Add 
```
# USE MOSEK
build --define=WITH_MOSEK=ON
```
to the project `.bashrc` file.

## Visualization
### To visualze the pose without doing simulation
Check the [`geometry_inspector.py`](https://github.com/RobotLocomotion/drake/blob/e59b7fc18dbe80b827d07e4a3283a0c87eda7021/manipulation/util/geometry_inspector.py) file and the `MultibodyPositionToGeometryPose` class.

### set `Meshcat` reconnection time
`http://localhost:7000/?reconnect_ms=100` sets reconnection time to 100ms


## LCM
Drake uses `LCM` to do basic communications between process.

### `drake-lcm-spy`
In order to view the lcm messages, you can use the shipped `drake-lcm-spy`. The difference between `drake-lcm-spy` and the original `lcm-spy` is that `drake-lcm-spy` is a thin wrapper around `lcm-spy`. `drake-lcm-spy` adds the lcm types provided by drake in the `CLASSPATH` variables, so that these messages can be successfully decoded and viewed by the `lcm-spy`.

For more information, check `examples/lcm-spy` in `lcm`'s source code.

### Use `drake-lcm-spy` to visualize custom lcm types
I have a custom type `lcmt_iiwa_cartesian_command.lcm`. In order to let the drake-lcm-spy be able to decode this type message, the following steps are necessary.
1. Generate the java type message by
    ```
    lcm-gen -j lcmt_iiwa_cartesian_command.lcm
    ```
    The generated java lcm type is located in a local drake directory.
2. Pack the java lcm type into a `jar` file
    ```
    javac -cp /opt/drake/share/java/lcm.jar drake/*.java
    jar cf my_lcmtypes.jar drake/*.class 
    ```
    Then you will have a `my_lcmtypes.jar` file in current directory.
3. Move the `jar` file into the drake's java lcm-types directory
    ```
    sudo cp my_types.jar /opt/drake/share/java 
    ```
4. Modify the `drake-lcm-spy`, add `"$prefix/share/java/my_lcmtypes.jar"` to the `CLASSPATH`.

## Miscellany
- `auto` cannot deduce the correct type of `context`. (Or maybe `context` can only be constructed by a system's method?) Use compound type (reference or pointer) for construction. For example
    ```c++
    auto& station_context = station->GetMyMutableContextFromRoot(context.get());
    auto* plant_context = &plant.GetMyMutableContextFromRoot(context.get());
    ```

# Problems

## Kernel died after adding a handwritten class derived from `LeafSystem` to `DiagramBuilder`
It turns out that I forgot to add 
```python
LeafSystem.__init__(self)
```
in the class's `__init__` method. This bug is quite hard to find for that no useful debug information is given.

## Reported Algebraic Loop Detected in DiagramBuilder
Drake does not support cycle/loop in the diagram that is pure feed-through. Check this [answer](https://stackoverflow.com/questions/50812170/understanding-algebraic-loop-error-message). To solve this problem, you need to break the algebraic loop by having a plant with a state.

More subtly, by default, at least in `pydrake`, the output port depends on all sources. You need to specify the dependency explicitly. For example,
```
self.DeclareVectorOutputPort('Lam', BasicVector(2), self.CopyStateOut, 
                             prerequisites_of_calc=set([self.xc_ticket()]))
```
in this [answer](https://stackoverflow.com/questions/61600097/algebraic-loop-error-help-fixing-spurious-dependency). There are other tickets provided by the `SystemBase` like `all_state_ticket()`.

## Miscellany
- `auto` cannot deduce the correct type of `context`. (Or maybe `context` can only be constructed by a system's method?) Use compound type (reference or pointer) for construction. For example
    ```c++
    auto& station_context = station->GetMyMutableContextFromRoot(context.get());
    auto* plant_context = &plant.GetMyMutableContextFromRoot(context.get());
    ```

## Cannot build drake from source
Encounter "RuntimeError: The operating system's C++ standard library is not installed correctly" when trying to build Drake from source using bazel. 
I ask a [question](https://stackoverflow.com/questions/70604791/encounter-runtimeerror-the-operating-systems-c-standard-library-is-not-inst/70615466#70615466) in stackoverlow, which contains the detail.

### Solution
I solved this problem following @jwnimmer-tri's answer in my question. In specific, I run
```
sudo apt remove cpp-9 g++-9 gcc-9 libasan6 libgcc-9-dev libstdc++-9-dev
sudo apt remove cpp-8 g++-8 gcc-8 libasan6 libgcc-8-dev libstdc++-8-dev
```
to remove GCC 8 and GCC 9 from my system. I must have accidentally installed GCC 8 and GCC 9, but not the corresponding G++. Drake use GCC 7 in `Ubuntu 18.04` and the `install_prereqs.sh` install GCC 7 fully (with G++). Clang seems to look for GCC standard C++ library and for higher version GCC. Therefore, when Clang looks for GCC standard C++ library in my system, it looks for GCC 9 and GCC 8. However, I do not have the corresponding G++ installed. Therefore, the Clang will fails to find the include file.

Another possible way to solve this problem is to have the higher version GCC fully installed.

# TODO: MOVE Drake related terms into this note.